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

7. Check if projects starts without errors (without Docker)
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
- if everything works fine quit with the `Ctrl + C` combination.  

8 . Create `Dockerfile`  
```sh
# Use the official Python runtime image
FROM python:3.13  
 
# Create the app directory
RUN mkdir /app
 
# Set the working directory inside the container
WORKDIR /app
 
# Set environment variables 
# Prevents Python from writing pyc files to disk
ENV PYTHONDONTWRITEBYTECODE=1
#Prevents Python from buffering stdout and stderr
ENV PYTHONUNBUFFERED=1 
 
# Upgrade pip
RUN pip install --upgrade pip 
 
# Copy the Django project  and install dependencies
COPY requirements.txt  /app/
 
# run this command to install all dependencies 
RUN pip install --no-cache-dir -r requirements.txt
 
# Copy the Django project to the container
COPY . /app/
 
# Expose the Django port
EXPOSE 8000
 
# Run Django’s development server
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

- build an image:  
```sh
$ docker build -t django-docker .
```
> you may [need](https://docs.docker.com/engine/install/linux-postinstall/) to add `sudo` before the command above!

```sh
sudo docker build -t django-docker .
[sudo] password for artur:           
[+] Building 1388.8s (12/12) FINISHED                                                   docker:default
 => [internal] load build definition from Dockerfile                                              0.0s
 => => transferring dockerfile: 812B                                                              0.0s
 => [internal] load metadata for docker.io/library/python:3.13                                    8.2s
 => [internal] load .dockerignore                                                                 0.0s
 => => transferring context: 243B                                                                 0.0s
 => [1/7] FROM docker.io/library/python:3.13@sha256:18634e45b29c0dd1a9a3a3d0781f9f8a221fe32ee  1345.9s
 => => resolve docker.io/library/python:3.13@sha256:18634e45b29c0dd1a9a3a3d0781f9f8a221fe32ee7a8  0.0s
 => => sha256:45fd9a3ce5de4d2b5ac35306a207bf24345ec15006cf91ef7b3458a8f78da7eb 6.28kB / 6.28kB    0.0s
 => => sha256:36e4db86de6eba33869491caa7946b80dd71c255f1940e96a9f755cc2b1f38 25.61MB / 25.61MB  163.4s
 => => sha256:8ea45766c6449310ca2fc621a7e00bedb4b9b803a7fbfe2607efce6d2e07e4 67.78MB / 67.78MB  555.8s
 => => sha256:80b7316254b3093eb3c7ac44bb6c34bde013f27947c1ed8d8afe456b957ebf 49.28MB / 49.28MB  384.3s
 => => sha256:18634e45b29c0dd1a9a3a3d0781f9f8a221fe32ee7a853db01e9120c710ef535 10.95kB / 10.95kB  0.0s
 => => sha256:15cfe7526c99021489a9564fa1808413ac88b68ab2bc0ccc7ba34b26718d0d0f 2.32kB / 2.32kB    0.0s
 => => sha256:3cb1455cf185ce395d378e9a520710caec9909f11d6f9c69d28d3f73c50 235.80MB / 235.80MB  1339.2s
 => => extracting sha256:80b7316254b3093eb3c7ac44bb6c34bde013f27947c1ed8d8afe456b957ebfdb         1.3s
 => => sha256:d622b1dca92a1bff6997b50744603cfe2c3ff963d49d1d673ba465f4a42c3fcd 6.10MB / 6.10MB  487.5s
 => => extracting sha256:36e4db86de6eba33869491caa7946b80dd71c255f1940e96a9f755cc2b1f3829         0.6s
 => => sha256:ad72fce423fc3c58b412c53eacabd1610851730252f5ee57525cdb08df25aa 27.25MB / 27.25MB  755.6s
 => => extracting sha256:8ea45766c6449310ca2fc621a7e00bedb4b9b803a7fbfe2607efce6d2e07e435         1.9s
 => => sha256:b9f8f98927f6fd501ce40652d02ecc721a637e1a8b1f8b34ca3543011e99a7ee 251B / 251B      556.8s
 => => extracting sha256:3cb1455cf185ce395d378e9a520710caec9909f11d6f9c69d28d3f73c50f2d23         5.4s
 => => extracting sha256:d622b1dca92a1bff6997b50744603cfe2c3ff963d49d1d673ba465f4a42c3fcd         0.2s
 => => extracting sha256:ad72fce423fc3c58b412c53eacabd1610851730252f5ee57525cdb08df25aa36         0.6s
 => => extracting sha256:b9f8f98927f6fd501ce40652d02ecc721a637e1a8b1f8b34ca3543011e99a7ee         0.0s
 => [internal] load build context                                                                 0.7s
 => => transferring context: 41.97MB                                                              0.7s
 => [2/7] RUN mkdir /app                                                                          0.7s
 => [3/7] WORKDIR /app                                                                            0.1s
 => [4/7] RUN pip install --upgrade pip                                                           2.5s
 => [5/7] COPY requirements.txt  /app/                                                            0.1s
 => [6/7] RUN pip install --no-cache-dir -r requirements.txt                                     29.8s
 => [7/7] COPY . /app/                                                                            0.6s
 => exporting to image                                                                            0.8s
 => => exporting layers                                                                           0.8s
 => => writing image sha256:9b974a8fa5aa0db3702f4895696d072496d3e9d3504041de8748f355659f79be      0.0s
 => => naming to docker.io/library/django-docker
 ```
 - our image is on the list:  
 ```sh
 sudo docker image list
[sudo] password for artur:           
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
django-docker   latest    9b974a8fa5aa   26 minutes ago   1.18GB
```