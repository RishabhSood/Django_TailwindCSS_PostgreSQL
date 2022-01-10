# Creating a Django Polls App with PostgreSQL Database

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

- Create custom user (optional):
  ```console
  postgres=# CREATE USER username WITH PASSWORD 'password';
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

### VIEWS & URLS

- Views related to an app (pages to be rendered) are defined in the views.py file.
  ```py
  def viewname(request):
    return HttpResponse/Template
  ```
- URL wrt to each view must be added to the urls file local to the app:

  ```py
  from django.urls import path

  from . import views

  urlpatterns = [
      path('', views.viewname, name='view_name'),
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
