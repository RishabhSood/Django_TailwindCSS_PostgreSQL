# Django - TailwindCSS - PostgreSQL Polls APP

![Django](https://img.shields.io/badge/django-%23092E20.svg?style=for-the-badge&logo=django&logoColor=white) ![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white) ![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)

## TABLE OF CONTENTS

- [Initial Setup:](#initial-setup)
  - [Create a Virtual Environment](#create-a-virtual-environment)
  - [Activate Virtual Environment](#activate-virtual-environment)
  - [Install Django](#install-django)
  - [Initialize Django Project](#initialize-django-project)
  - [Setup Database (PostgreSQL)](#setup-database-postgresql)
  - [Integrate Database](#integrate-database)
  - [That's it you're ready to start your project now!](#thats-it-youre-ready-to-start-your-project-now)
- [Notes](#notes)
  - [APPS](#apps)
  - [VIEWS](#views)
    - [TEMPLATES](#templates)
    - [GENERIC/CLASS BASED](#genericclass-based)
  - [FORMS](#forms)
  - [URLS](#urls)
  - [MODELS](#models)
    - [QUERYING & WORKING WITH MODELS](#querying--working-with-models)
  - [DJANGO ADMIN](#django-admin)
  - [TESTING](#testing)
  - [STATIC FILES](#static-files)
  - [TAILWIND CSS](#styling-via-tailwindcss)

## Initial Setup:

### Create a Virtual Environment

```console
user@device:~/projectDir$ python -m venv env
```

### Activate Virtual Environment

```console
user@device:~/projectDir$ env/Scripts/Activate
```

### Install Django

```console
user@device:~/projectDir$ pip install Django
```

### Initialize Django Project

```console
user@device:~/projectDir$ django-admin startproject projectname .
```

> A look at the general file structure post initialization:

```
rootdir/
    projectname/
        __init.py__
        asgi.py
        wsgi.py
        settings.py
        urls.py
    db.sqlite3
    manage.py
```

### Setup Database (PostgreSQL)

- Install PostgreSQL (for local db hosting/testing) : [Download Here](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)

- Open psql, you'll see the following prompt (note: the password for default user postgres, is the one you entered during the initial setup):

  > PSQL(postgresql prompt), PS: you might need to add this script to environment variables first !

  ```console
  Server [localhost]:
  Database [postgres]:
  Port [5432]:
  Username [postgres]:
  Password for user postgres:
  psql (14.1)
  WARNING: Console code page (437) differs from Windows code page (1252)
       8-bit characters might not work correctly. See psql reference
       page "Notes for Windows users" for details.
  Type "help" for help.

  postgres=#
  ```

- Create custom user and allow db creation (optional):
  ```console
  postgres=# CREATE USER username WITH PASSWORD 'password';
  postgres=# ALTER USER user CREATEDB;
  ```
- Create db for your project:
  ```console
  postgres=# CREATE DATABASE db_name WITH OWNER username ENCODING 'utf-8';
  ```

### Integrate Database

- Install psycopg2
  ```console
  user@device:~/projectDir$ pip install psycopg2
  ```
- Make changes to settings.py (present in the folder projectname):
  ```py
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.postgresql_psycopg2',
          'NAME': 'dbname',
          'USER': 'dbowner_username',
          'PASSWORD': 'dbowner_password',
          'HOST': 'localhost',
          'PORT': '',
      }
  }
  ```
- Also, change the timezone to cater to your region. For example, in India, the following setting would apply:
  ```py
  TIME_ZONE = 'Asia/Kolkata'
  ```

### That's it you're ready to start your project now!

- Make migrations
  ```console
  user@device:~/projectDir$ python ./manage.py makemigrations
  ```
- Migrate changes (create default user tables etc.)
  ```console
  user@device:~/projectDir$ python ./manage.py migrate
  ```
- Create superuser for admin access:
  ```console
  user@device:~/projectDir$ python ./manage.py createsuperuser
  ```
  > Follow the prompts to enter required credentials, you may leave some fields blank by pressing enter!
- Run server:
  ```console
  user@device:~/projectDir$ python ./manage.py runserver
  ```

## Notes

### APPS

- Creating an App(use case specific web application) within our project:
  ```console
  user@device:~/projectDir$ python ./manage.py startapp appName
  ```
- Following is the general structure of an App:
  ```
  appName/
      __init__.py
      admin.py
      apps.py
      migrations/
          __init__.py
      models.py
      tests.py
      urls.py
      views.py
  ```
- The app must then be added to INSTALLED_APPS in the settings.py file:

  ```
  INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    #app
    'appname'
  ]
  ```

### VIEWS

> A view is a “type” of web page in your Django application that generally serves a specific function and has a specific template.

- Views related to an app (pages to be rendered) are defined in the views.py file. Each view, takes in a request parameter, and other optional params:

  ```py
  def viewname(request, var1, var2, ...):
    return HttpResponse
  ```

- Some views, may depend on existence of object/s. For these, we use the get_object_or_404/get_list_or_404 shortcuts:

  ```py
  from .models import ModelName
  from django.shortcuts import get_object_or_404

  def viewname(request, id):
    obj = get_object_or_404(ModelName, pk=id)
    return HttpResponse
  ```

- While updating object fields in views, a race condition might arise(two users parallelly updating the same fields leading to an error). To resolve this, we use the "F()" function, read the docs for the same here:

  > https://docs.djangoproject.com/en/4.0/ref/models/expressions/#f-expressions

  #### TEMPLATES

  - Templates (Django-Html docs) are defined inside a templates folder local to the app directory. So as to not conflict with similar template names in other "app-template" directories, we create a subfolder inside the template directory with the name of the app. The app folder structure should look similar to this:
    ```
    appName/
      __init__.py
      admin.py
      apps.py
      migrations/
          __init__.py
      models.py
      tests.py
      templates/
          appName/
              templatename.html
      urls.py
      views.py
    ```
  - In order to render HTML templates instead of plain Http Responses, we use the following django shortcut:

    > Here, context defines the variables to be passed to the HTML template.

    ```py
    from django.shortcuts import render

    def viewname(request, var1, var2, ...):
      context = {
        "var1" : var1,
        "var2" : var2
      }
      return render(request, "template path inside local templates folder", context)
    ```

  - A sample template (here, the variable latest_question_list, was passed via context):
    ```jinja
    {% if latest_question_list %}
      <ul>
        {% for question in latest_question_list %}
            <li><a href="{% url 'polls:detail' question.id %}">{{ forloop.counter }}. {{ question.question_text }}</a></li>
        {% endfor %}
      </ul>
    {% else %}
      <p>No polls are available.</p>
    {% endif %}
    ```

  #### GENERIC/CLASS BASED

  - Views, which list objects of a particular Model, or which display/update a certain object of a model are very common. Hence, Django provides a shortcut, called the “generic views” system, which abstract common patterns.
  - For using generic views, a few changes must be made to the way we define our urls:

    > generic view expects the primary key value captured from the URL to be called "pk"

    ```py
    # appname/urls.py
    from django.urls import path
    from . import views

    app_name="appname"
    urlpatterns = [
      path('<int:pk>/viewname', views.ViewName.as_view(), name='viewname'),
      #...
    ]
    ```

  - A generic view, maybe defined as follows:

    ```py
    from django.views import generic

    class ViewName(generic.ViewType):
      template_name = "appname/temlate.html"
      model = ModelName
      context_object_name = "custom_name"

      #...
    ```

  - Refer to the following links:
    > - https://docs.djangoproject.com/en/4.0/topics/class-based-views/generic-display/
    > - https://docs.djangoproject.com/en/4.0/topics/class-based-views/generic-editing/
    > - https://docs.djangoproject.com/en/4.0/topics/class-based-views/mixins/

### FORMS

- Django provides a safe system for protection against Cross Site Reqest Forgeries. Thus, for each form defined in out templates, we use the {% csrf_token %} template tag.
  ```jinja
  <form action="{% url 'view_name' args %}" method="post/get">
  {% csrf_token %}
    <!-- form fields -->
  </form>
  ```
- The **name** attribute of each form field, defines it's access name in the post data.
  ```py
  request.POST['field_name']
  ```

### URLS

- URL wrt to each view must be added to the urls file local to the app. A url may contain path string and variables to be passed along to its corresponding view:

  ```py
  from django.urls import path

  from . import views

  app_name = "appname" # defines namespace for app urls, avoids url conflicts
  urlpatterns = [
      path('<param_type:param_name>/pathstring/', views.viewname, name='view_name'),
  ]
  ```

- These urls local to the app must be then pointed to by the root URLConf residing in the projectname folder.

  ```py
  from django.contrib import admin
  from django.urls import include, path

  urlpatterns = [
      path('appname/', include('appname.urls')),
      path('admin/', admin.site.urls),
  ]
  ```

- URLs may be accessed in the templates, as follows:

  ```jinja
  <a href="{% url 'app_name:url_name' args %}"> {{ var_link_name }} </a>
  ```

- A URL, can be generated from the name of the view it's connected to, by using the reverse function:

  ```py
  from django.http import HttpResponseRedirect
  from django.urls import reverse

  def viewname(request, args):
    # ...
    return HttpResponseRedirect(reverse('appname:viewname', args=(arg1, arg2, ...)))
  ```

### MODELS

- Models related to an app are defined in the models.py file local to the app.

  ```py
  from django.db import models

  class ModelName(models.Model):
    model_field = models.FieldName(params)
  ```

- A foreign key may be established from one model instance to the other, as follows:

  ```py
  class ModelOne(models.Model):
    # ...
    model_field = models.FieldName(params)

  class ModelTwo(models.Model):
    modelOne = models.ForeignKey(ModelOne, on_delete=models.CASCADE)
    # ...
    model_field = models.FieldName(params)
  ```

- Each model has an object representation (id by default), which can be changed by defining the \_\_str\_\_ function:
  ```py
  class ModelName(models.Model):
    # ...
    def __str__(self):
        return self.paramToRender
  ```
- Similarly, custom methods can be added to the Model, taking self as the input parameter to refer the model's fields.
  ```py
  class ModelName(models.Model):
    # ...
    def customMethod(self):
      # ...
      return result
  ```
- Once models have been defined, migrations must be made in order to update the database:
  ```console
  user@device:~/projectDir$ python ./manage.py makemigrations appName
  user@device:~/projectDir$ python ./manage.py migrate
  ```

#### QUERYING & WORKING WITH MODELS

- Post migrations, the model can then be interacted with via a shell:
  ```console
  user@device:~/projectDir$ python ./manage.py shell
  >>>
  ```
- Importing models into the shell
  ```py
  >>> from appname.models import ModelName
  ```
- Displaying all objects in a model
  ```py
  >>> ModelName.objects.all()
  ```
- Creating model objects
  ```py
  >>> obj = ModelName(param1=val1, param2=val2...)
  >>> obj.save()
  ```
- Getting objects params:
  ```py
  >>> obj.param
  ```
- Changing object param values:
  ```py
  >>> obj.param = newValue
  >>> obj.save()
  ```
- To get a particular object on basis of field value (returns Object):
  ```py
  >>> ModelName.objects.get(field=val)
  ```
- Filtering objects on basis of field value (return QuerySet):
  ```py
  >>> ModelName.objects.filter(field=val)
  ```
- Double underscores are used to seperate relationships, for example, consider the following object question:
  ```
  question:
    pub_date:
      year
      month
      day
    question_text:
  ```
  > The **year** attribute, for example can be represented as follows: **question**pub_date**year**
- Consider the following Question, Choice models:

  ```py
  class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


  class Choice(models.Model):
      question = models.ForeignKey(Question, on_delete=models.CASCADE)
      choice_text = models.CharField(max_length=200)
      votes = models.IntegerField(default=0)
  ```

  - Here, we can access the choice objects linked to a particular Question object (say q), as follows:
    ```py
    q.choice_set.all()
    # and for the count:
    q.choice_set.count()
    # similarly filtering can be done too:
    q.choice_set.filter(choice_text__startswith='Choice Value')
    ```
  - Similarly details of question related to a choice can be accessed, as if it were the object's own parameters:
    ```py
    Choice.objects.filter(question__pub_date__year=current_year)
    ```

### DJANGO ADMIN

> Customizing admin page (& custom templates) : https://docs.djangoproject.com/en/4.0/intro/tutorial07/

- In order to access created Models on the admin dashboard interface (http://127.0.0.1:8000/admin/), we must register our models on the admin site. To do this, we make the following updates in the admin.py file local to the App containing the Models we wish to register:

  ```py
  from django.contrib import admin

  from .models import ModelName

  admin.site.register(ModelName)
  ```

### TESTING

> Test are routines that check the operation of your code. They might work upon a tiny detail (unit tests) or examine the overall operation of the software (integration tests).

- Some best practices while defining tests are :

  - a separate TestClass for each model or view
  - a separate test method for each set of conditions you want to test
  - test method names that describe their function

- We conventionaly define the test cases for an app in its tests.py file. We create a class catering to the model/view we wish to test and inherit from the [django.test.TestCase Class](https://docs.djangoproject.com/en/4.0/topics/testing/tools/#testcase).

  ```py
  from django.test import TestCase

  class NameModelTests(TestCase):
    # ...
    self.assertIs(val1, val2)
    # or/and
    self.assertContains(val3, val4)
    # or/and
    response = self.client.get(url)
    self.assertQuerysetEqual(response.context['attribute'], [...])
  ```

- Read more on the same, [here](https://docs.djangoproject.com/en/4.0/intro/tutorial05/#further-testing).

### STATIC FILES

- Static files (imgs/js/css) are defined inside a static folder local to the app directory. So as to not conflict with similar static files names in other "app-static" directories, we create a subfolder inside the static directory with the name of the app. The app folder structure should look similar to this:
  ```
  appName/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    templates/
        appName/
    static/
        appName/
          staticfile
          ...
    urls.py
    views.py
  ```
- To use the static files in a template, we then use {% load static %}, to enable usage of the {% static %} template tag, which generates the absolute URL of static file.
  ```python
  {% load static %}
  <link rel="stylesheet" type="text/css" href="{% static 'appname/style.css' %}">
  ```

### Styling Via TailwindCSS

> Use the django-tailwind library to integrate tailwind development with django, read the docs [here](https://django-tailwind.readthedocs.io/en/latest/installation.html).

- Install the django-tailwind library:
  ```console
  user@device:~/projectDir$ pip install django-tailwind
  ```
- Add tailwind to the installed apps (in settings.py file):
  ```py
  INSTALLED_APPS = [
      # ...
      # TailwindCSS
      'tailwind',
  ]
  ```
- Create a theme app, which will provide tailwind styles to the whole project:
  ```console
  user@device:~/projectDir$ python manage.py tailwind init theme
  ```
- Add the theme app to the installed apps and configure the TAILWIND_APP_NAME variable.

  ```py
  INSTALLED_APPS = [
      # ...
      # TailwindCSS
      'tailwind',
      'theme',
  ]

  TAILWIND_APP_NAME = 'theme'
  ```

- Install [NodeJs](https://nodejs.org/en/)
- Get NodeJs system path:
  - Linux:
    ```console
    user@device:~/projectDir$ which npm
    /usr/local/bin/npm
    ```
  - Windows:
    ```console
    user@device::~/projectDir$ gcm npm
    CommandType     Name        Version    Source
    -----------     ----        -------    ------
    Application     npm.cmd     0.0.0.0    C:\Program Files\nodejs\npm.cmd
    ```
- Add path to settings.py file
  - Linux
    ```python
    # rest of the settings
    NPM_BIN_PATH = r"/usr/local/bin/npm" # add this line
    ```
  - Windows
    ```python
    # rest of the config
    NPM_BIN_PATH = r"C:\Program Files\nodejs\npm.cmd" # add this line
    ```
- Install TailwindCSS dependencies:
  ```console
  user@device:~/projectDir$ python manage.py tailwind install
  ```
- For using tailwind in any template of any registered app, do the following:
  ```jinja
  {% load static tailwind_tags %}
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <!-- rest of the header -->
      {% tailwind_preload_css %}
      {% tailwind_css %}
    </head>
    <body>
      <!-- body content -->
    </body>
  </html>
  ```
- For rendering, compile "used" tailwind tags by running tailwind (only required for development) prior runserver:
  ```console
  <!-- console 1 -->
  user@device:~/projectDir$ python manage.py tailwind start
  <!-- console 2 -->
  user@device:~/projectDir$ python manage.py runserver
  ```
- To create a production build of your theme, run:
  ```console
  user@device:~/projectDir$ python manage.py tailwind build
  ```
