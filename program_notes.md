# Program guide

Kenneth Burchfiel

In this program, I will aim to show how to display interactive charts online using Python, Django, Bokeh (an interactive chart library), and Heroku. I'll also show how to incorporate user authentication into the webpage (so that users won't be able to view a given page without first logging in).

Here are the steps I took:

## 1. Creating a Heroku app and a corresponding Heroku Postgres database

I chose to create a Heroku app first (with the name 'kburchfiel-web-chart-demo') so that I would have a database to which to connect my Django app.

I did this online via https://dashboard.heroku.com, but could also have chosen to do so using the command line interface (see https://devcenter.heroku.com/articles/git). The project page showed steps for adding a Git repository for this app; these steps can also be found at https://devcenter.heroku.com/articles/git#for-an-existing-app. 


Next, within my app's page on Heroku, I went to the Resources tab and searched for PostgreSQL. I then added a free instance of Heroku PostgreSQL to the app. By going to the app's Settings page, I was able to view and download the Heroku Postgres database's URI, which will let my program connect to the database. (Heroku warns you that "these credentials are not permanent," but hardcoding the URI into my program will do for now.)

Soon enough, the file structure for my program will include two web_chart_demo folders: one contained within web_chart_app (which I'll call the 'upper' web_chart_demo folder), and one contained within the upper web_chart_app folder (which I'll call the 'lower' web_chart_demo folder).

## 2. Configuring the Django app and linking it with the Heroku Postgres database

Note: Before continuing on, you should first have the [Heroku CLI (command line interface)](https://devcenter.heroku.com/categories/command-line) installed on your computer.

Having [installed Python's Django library](https://docs.djangoproject.com/en/4.1/intro/install/) previously, I next followed the steps in [this guide](https://docs.djangoproject.com/en/4.1/intro/tutorial01/) to create my Django app within my web_chart_app folder. I named the folder for this app web_chart_demo.

(Note: I found that some of the Windows command line entries shown within this tutorial did not work for me. For instance, instead of entering 'py manage.py runserver', I needed to enter 'python manage.py runserver.')

Using [Part 2](https://docs.djangoproject.com/en/4.1/intro/tutorial02/) of the Django setup tutorial as a reference, I connected my app to my Heroku Postgres database. (Some of the steps for doing so can be found within the [Database setup](https://docs.djangoproject.com/en/4.1/intro/tutorial02/#database-setup) section of Part 2, although I did not follow them verbatim, as I'll explain.) I already had psycopg2 installed, which is necessary for PostgreSQL and Django to work together per [the documentation](https://docs.djangoproject.com/en/4.1/topics/install/#database-installation).

[This resource from Heroku.com](https://devcenter.heroku.com/articles/connecting-heroku-postgres#connecting-in-python) and [this resource from Djangoproject.com](https://docs.djangoproject.com/en/4.0/ref/databases/) helped me connect my Heroku Postgres database to my Django app.

In order for my Django app to connect to my PostgreSQL database, it needed access to the database's credentials. Therefore, in order to store these credentials, I created a .env file on the same directory level as manage.py. (See the [python-dotenv](https://pypi.org/project/python-dotenv/) documentation for more information about this step.) I then added the following line:

DATABASE_URL = uri_from_heroku_postgres_settings_page_goes_here

The URI that I inserted after DATABASE_URL = begins with postgres:// and contains the configuration information that Django needs to establish a database connection.

I also removed the original SECRET_KEY value from settings.py. Instead, I created a new secret key using [this guide](https://humberto.io/blog/tldr-generate-django-secret-key/) and pasted it as another entry within my .env file. By this point, the .env file appeared as follows:

SECRET_KEY = my_secret_key_went_here_without_any_quotes
DATABASE_URL = my_database_url_went_here_without_any_quotes

Note that you don't need to put quotes around your secret key or database URL.

Note that Heroku needs access to SECRET_KEY also. (It already has access to DATABASE_URL--after all, we got that URL from Heroku in the first place--so we don't need to worry about that one.) To give it the SECRET_KEY info, I went to the Settings page for my web app, then opened up the Config Vars section and added a new row with SECRET_KEY as the key and my key as the value.

Once the database URL was present in the .env file, I parsed its information using the dj_database_url library and passed the output to the default DATABASES variable (as suggested by [Heroku's documentation](https://devcenter.heroku.com/articles/connecting-heroku-postgres#connecting-in-python)).

The final database configuration can be found in the settings.py file.

## Heroku Deployment

It's important to have navigated to the right folder within your project before executing the following steps. (This will of course be the case for later steps as well.) Currently, the project directory is as follows:

web_chart_app

--->web_chart_demo [Note: I'll refer to this as the 'upper' web_chart_demo folder]

--------.env

--------.gitignore

--------db.sqlite3

--------manage.py

--------program_notes.md (the file that you're currently reading)

------->web_chart_demo [Note: I'll refer to this as the 'lower' web_chart_demo folder]

------------\_\_pycache\_\_

------------asgi.py

------------settings.py

------------urls.py

------------wsgi.py

------->chart_app

------------\_\_pycache\_\_

----------->migrations

----------->\_\_init\_\_.py

------------admin.py

------------apps.py

------------models.py

------------tests.py

------------urls.py

------------views.py

It's easy to get confused by this directory. After all, there are two web_chart_demo folders, and both the chart_app folder and the lower web_chart_demo folder have a urls.py file. However, it will become easier to follow as you gain more experience with Django. 

After navigating to the upper web_chart_demo folder in my command prompt and running python manage.py migrate, I arrived at the ['Creating models'](https://docs.djangoproject.com/en/4.1/intro/tutorial02/#creating-models) section of Part 2 of the Django tutorial. However, before continuing onward, I first deployed my app to Heroku using Heroku's [Deploying to Git](https://devcenter.heroku.com/articles/git#for-an-existing-app) and [Depolying Python and Django Apps on Heroku](https://devcenter.heroku.com/articles/deploying-python) documentation pages as a reference.

The Heroku documentation explains how to use your command line to initialize and update a project's Git repo. However, since I'm more familiar with using Visual Studio Code's Git extension, I used that tool to update my project's repo. 

Before I could successfully deploy my app to Python, I needed to first add some additional files to my project. I used Heroku's [Delpoying Python and Django Apps on Heroku](https://devcenter.heroku.com/articles/deploying-python) as a reference for these changes. 

Specifically, I needed to add the following 3 files within my upper web_chart_demo folder (so that the files were at the same level as manage.py):

1. Procfile (not Procfile.txt!)

I used the following template for the Procfile:
https://devcenter.heroku.com/articles/django-app-configuration
The template is "web: gunicorn myproject.wsgi" (in this case, 'myproject' equals web_chart_demo)


2. requirements.txt

This file contains the names of various Python libraries that are necessary in order to successfully run the web app.

3. runtime.txt

This library specifies the Python version that the web app will use.

I based my version of this file off of the one in Heroku's [python-getting-started GitHub project](https://github.com/heroku/python-getting-started/blob/main/runtime.txt). 


After navigating to my upper web_chart_demo folder, I entered [heroku login](https://devcenter.heroku.com/articles/heroku-cli#get-started-with-the-heroku-cli) within my command prompt and then logged in via the webpage that popped up.

Since I already created my app (named kburchfiel-web-chart-demo), I didn't need to enter heroku create within the command prompt. Instead, after updating my Git repository using Visual Studio Code, I entered:

heroku git:remote -a kburchfiel-web-chart-demo

This line (based on [this section](https://devcenter.heroku.com/articles/git#for-an-existing-app) of Heroku's documentation) linked my local Git repository with my Heroku web app.

At this point, I also needed to add a STATIC_ROOT value to settings.py in order to avoid an error message when attempting to push my app to Heroku.
Therefore, I entered in the STATIC_ROOT found within Heroku's getting started project: https://github.com/heroku/python-getting-started/blob/main/gettingstarted/settings.py

I could now 'push' my web app to Heroku! In order to do so, I entered git push heroku master into my command prompt within the upper web_chart_demo directory. The URL for the app (which could be found within its page on Heroku) is:

https://kburchfiel-web-chart-demo.herokuapp.com/

In order to view the app online, however, I needed to enter this URL into the ALLOWED_HOSTS list within settings.py. (I also added http://127.0.0.1:8000/, the local server for viewing Heroku pages, into this list.) I then needed to update my Git repository and run git push heroku master once again in order to upload these changes to Heroku.

I confirmed the app was working by going to https://kburchfiel-web-chart-demo.herokuapp.com/chart_app/ and seeing the message: 

"This is the index page for my web charts app."

This came from the following code within web_chart_demo/chart_app/views.py:

def index(request):
    return HttpResponse("This is the index page for my web charts app.")

Now that the app's framework and database were in place, I could turn my attention to building the app's functionality. 


## Adding 


[Note: try going through the rest of the tutorial in order to review template creation and the request/response cycle. But before that, you can create code that will take flight information from the .csv file and convert it into a format that can be used by the app. (This might involve grouping departures counts by the top 20 airports, airlines, and airplanes so that you end up with a roughly 8000-row DataFrame--just large enough to fit within your Heroku database! Or, to be on the safer side, you could limit the departures counts to the top 10 airports, airlines, and airplanes.)

This wouldn't have to be your only table--but it could be a good way to demonstrate filtered graph capabilities within Bokeh (assuming it has them lol)

I could also show how to create line charts within Bokeh by creating a table showing flight departures by month (perhaps for multiple airlines)

