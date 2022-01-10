# Creating a Django Project with PostgreSQL Database

![Django](https://img.shields.io/badge/django-%23092E20.svg?style=for-the-badge&logo=django&logoColor=white) ![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)

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
