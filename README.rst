Introduction
============
This package is meant to be used as part of a course covering several python web
frameworks being taught in person. This will be the core code for making a simple
TODO list.

NOTE: Unlike the intro to python classes, this section of the course will not have 
time to pay special attention to windows users. This part focuses on web development 
in python, which is NOT windows friendly. Of course you may join and I will help as 
much as possible but in general if you are serious about web development you should 
borrow a friends computer or pair up with someone else in the class on a nix-like 
system, which includes Mac OS X. Sorry!

Further Documentation
---------------------
* http://www.web2py.com/book/

Steps to your own noiselist
---------------------------
For this class, we will go through the process of creating a personal TODO list on 3 different frameworks. This is the first. So let's go.

**Get web2py downloaded and running with demo app using buildout**::

  > git clone git://github.com/noisebridge/web2py-noiselist.git
  > cd web2py-noiselist
  > python bootstrap.py # use the non-local python if you are on mac osx please
  > ./bin/buildout
  > ./bin/web2py start

**Create a new simple app**

  - Start at http://127.0.0.1:8000/admin/default/site
  - Create a new simple app named noiselist. web2py will setup a default folder structure for you with basic routing. This app can be viewed at http://127.0.0.1:8000/noiselist/default/index .
  - You can edit your new app through the web or on the filesystem, whichever feels better to you. The following instructions will work for either.
  - Through the web, you can edit your files at http://127.0.0.1:8000/admin/default/design/noiselist
  - On the filesystem, you can edit your app at at web2py-noiselist/web2py/applications/noiselist

**Intro to the file system and templating**

The Model section/folder will have all of your database definitions. The View section/folder will have all of your templates, css, js, and images. The controllers section/folder will have all of your business logic, or for all intensive purposes most of your .py files.

In views > default > index.html, add html to make the front page resemble a list. Replace the boiler plate code with something like::

     <ul>
           <li>Tequila</li>
           <li>Pants</li>
           <li>Pre-party</li>
     </ul>

What you add and remove is up to you. Play with adding and removing different tags to see what happens.

**Controllers and sending data**

Here we will see how to use variables in templates, and make the list dynamically populated. We aren't starting right away with database muckery because we want to learn how things fit together nicely and minimize the chance that we have multiple errors. Reload often after each code change to make sure you haven't borked things.

In controllers > default.py, add a function which returns a list::

    def get_list():
        """
        Return the current todo list
        """
        return  [
                "Go to the store and buy pants",
                "Eat breakfast",
                "Flash mob"
                ]

Still in controllers > default.py, call your new function from the index page. Note that you must return your new variable in a dictionary. This adds the variable to the scope of the template::

    def index():
        todo_list = get_list()
        return dict(todo_list=todo_list)

In views > default > index.html, update your html to use the todo_list variable to dynamically display the list::

    <ul>
        {{for todo in todo_list:}}
            <li>{{=todo}}</li>
        {{pass}}
    </ul>

Now your app is wired to get a list of todo items from the controller logic and display them. Yay!

**Hook up to a real database**

For this example, we will be working from the default SQLite database. This is a very simple relational database that lives on the filesystem. Later on in the class we will work with more robust databases. 

At the very bottom of models > db.py, let's add code to define our todo list model::

    db.define_table('todo_list',Field('todo'),)    
    db.todo_list.todo.requires=IS_NOT_EMPTY()

This code says: Create a table called "todo_list", and give it one field (or column if you like) called "todo". The second line says: make sure no one enters empty todos. You can add as few or as many restrictions as you like. We are adding the empty check here to demo the way that web2py handles data validation.

Web2py will automatically reload changed code AND database schemas every time you save a file. Neato.

Now that we have a database, we need to get info from that database. Keep in mind that your database is currently empty so the net end result will be an empty todo list on your index page.

In controllers > default.py, update the get_list function to pull from the database instead of a list ([])::

    def get_list():
        """
        Return the current todo list
        """
        return db().select(db.todo_list.ALL)  

We have one small change to make in the template. The db select above will return rows, and we really just want one item in that row - the todo. Update views > default > index.html to say::

    <ul>
        {{for todo in todo_list:}}
            <li>{{=todo.todo}}</li>
        {{pass}}
    </ul>

If you had more fields in your model like 'created_by' for example, you can access that by saying todo.created_by.

**Adding to the db**

Currently we are pulling from the db, but pulling an empty list. Let's put a form on the front page to add list items. Web2py will auto generate and validate forms for you so we will take that approach. 

To create a form, add a function in controllers > default.py::

  def add_to_list():
    """
    Render and handle response from adding to a todo form
    """
    form=SQLFORM(db.todo_list)
    message = None
    if form.accepts(request,session):
        message = 'Added to list!'
    else:
        message = 'something went wrong'  
    if message:
        response.flash=message
    return form

There are a couple things going on there. First, we see that web2py has some whacky globals lying around (SQLFORM) so be careful. Second, given a database table it will auto-generate a form for you. Third, form.accepts will do validation of the form for you. Last but not least is the introduction of flash. response.flash automatically adds a growl style notification to the resulting page which you will see when you violate the "no empty todo's" restraint that we added earlier.

Before leaving that file, make sure to send that form to the index page with::

  def index():
    """
    example action using the internationalization operator T and flash
    rendered by views/default/index.html or views/generic.html
    """
    add_form = add_to_list()
    todo_list = get_list()
    return dict(todo_list=todo_list,
                add_list_item_form=add_form)

Last but not least, add the code to render the form to your front page. In views > default > index.html::

    {{if 'add_list_item_form' in globals():}}
        {{=add_list_item_form}}
    {{pass}}

Now reload the front page and voila! You should be able to view and add items in your list! Note that if you add an empty item, a error response is flashed.

Packaging
---------
If you did all of the work through the web (or FS even), you can package up your app and redistribute with the built in tools. 
 * Go to http://127.0.0.1:8000/admin/default/site
 * Click "Pack all"
 * Move the w2p export into web2pyapps
 * Update buildout
 * Commit!

Homework
--------
If you are captivated with web2py, try to do the following at home:
* Delete an item from a list
* Configure multiple users
* Review at the beginning of next class
