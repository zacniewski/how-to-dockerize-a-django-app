# How to Dockerize a Django app
Official [guide](https://www.docker.com/blog/how-to-dockerize-django-app/) about Dockerizing Django app from docker.com

## Prerequisities
- Docker installed on your machine.  

### Steps on local machine (optional, if you want to do it from scratch)  
1. Create and activate virtual environment  
- on Linux
```sh

$ python3 -m venv .venv
$ source .venv/bin/activate
```
- on Windows
```sh
$ python -m venv .venv
$ cd .venv
$ cd Scripts
$ activate 
$ cd ..
$ cd ..
```

2. Install Django and python-decouple
```sh
$ pip install -U pip
$ pip install django
$ pip install python-decouple
```

3. Create Django project
```sh
$ django-admin startproject my_docker_django_project .
```

4. Store requirements in the file
```sh
$ pip freeze > requirements.txt
```

5. Modify `settings.py` file
- add import
```sh
from decouple import config
```
- change particular lines
```sh
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=lambda v: [s.strip() for s in v.split(',')])
```

6. Create `.env` file for credentials
```sh
$ touch .env
```
- use it in a way described in the [documentation](https://pypi.org/project/python-decouple/).  

- if you want to generate new SECRET_KEY:  
```sh
$ python manage.py shell

Python 3.12.3 (main, Aug 14 2025, 17:47:21) [GCC 13.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from django.core.management.utils import get_random_secret_key
>>> print(get_random_secret_key())
# copy value from here to the .env file
>>> exit()
```

7. Check if projects starts without errors 
```sh
$ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
August 23, 2025 - 20:16:35
Django version 5.2.5, using settings 'my_docker_django_project.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```
