# How to Dockerize a Django app
Official [guide](https://www.docker.com/blog/how-to-dockerize-django-app/) about Dockerizing Django app from docker.com

## I. Prerequisities  
- Docker installed on your machine.  

## II. Steps on local machine (optional, if you want to do it from scratch)  
#### 1. Create and activate virtual environment  
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

#### 2. Install Django and python-decouple
```sh
$ pip install -U pip
$ pip install django
$ pip install python-decouple
```

#### 3. Create Django project
```sh
$ django-admin startproject my_docker_django_project .
```

#### 4. Store requirements in the file
```sh
$ pip freeze > requirements.txt
```

#### 5. Modify `settings.py` file
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

#### 6. Create `.env` file for credentials
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

#### 7. Check if projects starts without errors (without Docker)
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

#### 8 . Create `Dockerfile`  
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

#### 9 . Create `.dockerignore`:  
```sh
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env
pip-log.txt
pip-delete-this-directory.txt
.tox
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.log
.git
.mypy_cache
.pytest_cache
.hypothesis
```

#### 10. Build an image:  
```sh
$ docker build -t django-docker .
```
> you may [need](https://docs.docker.com/engine/install/linux-postinstall/) to add `sudo` before the command above!

```sh
sudo docker build -t django-docker .
[sudo] password for artur:           
[+] Building 847.0s (12/12) FINISHED                                                    docker:default
 => [internal] load build definition from Dockerfile                                              0.0s
 => => transferring dockerfile: 812B                                                              0.0s
 => [internal] load metadata for docker.io/library/python:3.13                                    2.5s
 => [internal] load .dockerignore                                                                 0.0s
 => => transferring context: 243B                                                                 0.0s
 => [1/7] FROM docker.io/library/python:3.13@sha256:18634e45b29c0dd1a9a3a3d0781f9f8a221fe32ee7  823.6s
 => => resolve docker.io/library/python:3.13@sha256:18634e45b29c0dd1a9a3a3d0781f9f8a221fe32ee7a8  0.0s
 => => sha256:18634e45b29c0dd1a9a3a3d0781f9f8a221fe32ee7a853db01e9120c710ef535 10.95kB / 10.95kB  0.0s
 => => sha256:15cfe7526c99021489a9564fa1808413ac88b68ab2bc0ccc7ba34b26718d0d0f 2.32kB / 2.32kB    0.0s
 => => sha256:45fd9a3ce5de4d2b5ac35306a207bf24345ec15006cf91ef7b3458a8f78da7eb 6.28kB / 6.28kB    0.0s
 => => sha256:80b7316254b3093eb3c7ac44bb6c34bde013f27947c1ed8d8afe456b957ebf 49.28MB / 49.28MB  391.1s
 => => sha256:36e4db86de6eba33869491caa7946b80dd71c255f1940e96a9f755cc2b1f38 25.61MB / 25.61MB  141.5s
 => => sha256:8ea45766c6449310ca2fc621a7e00bedb4b9b803a7fbfe2607efce6d2e07e4 67.78MB / 67.78MB  344.9s
 => => sha256:3cb1455cf185ce395d378e9a520710caec9909f11d6f9c69d28d3f73c50f 235.80MB / 235.80MB  816.7s
 => => sha256:d622b1dca92a1bff6997b50744603cfe2c3ff963d49d1d673ba465f4a42c3fcd 6.10MB / 6.10MB  374.2s
 => => sha256:ad72fce423fc3c58b412c53eacabd1610851730252f5ee57525cdb08df25aa 27.25MB / 27.25MB  467.7s
 => => extracting sha256:80b7316254b3093eb3c7ac44bb6c34bde013f27947c1ed8d8afe456b957ebfdb         1.4s
 => => sha256:b9f8f98927f6fd501ce40652d02ecc721a637e1a8b1f8b34ca3543011e99a7ee 251B / 251B      392.1s
 => => extracting sha256:36e4db86de6eba33869491caa7946b80dd71c255f1940e96a9f755cc2b1f3829         0.6s
 => => extracting sha256:8ea45766c6449310ca2fc621a7e00bedb4b9b803a7fbfe2607efce6d2e07e435         2.0s
 => => extracting sha256:3cb1455cf185ce395d378e9a520710caec9909f11d6f9c69d28d3f73c50f2d23         5.7s
 => => extracting sha256:d622b1dca92a1bff6997b50744603cfe2c3ff963d49d1d673ba465f4a42c3fcd         0.2s
 => => extracting sha256:ad72fce423fc3c58b412c53eacabd1610851730252f5ee57525cdb08df25aa36         0.6s
 => => extracting sha256:b9f8f98927f6fd501ce40652d02ecc721a637e1a8b1f8b34ca3543011e99a7ee         0.0s
 => [internal] load build context                                                                 1.0s
 => => transferring context: 41.97MB                                                              1.0s
 => [2/7] RUN mkdir /app                                                                          0.7s
 => [3/7] WORKDIR /app                                                                            0.0s
 => [4/7] RUN pip install --upgrade pip                                                           2.4s
 => [5/7] COPY requirements.txt  /app/                                                            0.1s
 => [6/7] RUN pip install --no-cache-dir -r requirements.txt                                     16.0s
 => [7/7] COPY . /app/                                                                            0.6s
 => exporting to image                                                                            0.8s
 => => exporting layers                                                                           0.8s
 => => writing image sha256:fe47a5ac0a6811864e7531b805f00b0e48712e8ad3639baaa36fbd23b9c41901      0.0s
 => => naming to docker.io/library/django-docker-image 
 ```
 - our image is on the list:  
 ```sh
 sudo docker image list
[sudo] password for artur:           
REPOSITORY      TAG       IMAGE ID       CREATED          SIZE
django-docker-image   latest    fe47a5ac0a68   4 minutes ago   1.18GB
```

#### 11. Running the container (with given name):  
```sh
$ sudo docker run -d --name django-docker-container django-docker-image 
a26f9ba0d223897210b805df3b0b1028abc7aac707db62e219bf8ac5d8542dbc
```
- where `django-docker-container` is the name of the container we're running, and `django-docker-image` is the name of the image we've created earlier.  

- checking containers  
```sh
$ sudo docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS      NAMES
a26f9ba0d223   django-docker-image   "python manage.py ru…"   30 seconds ago   Up 30 seconds   8000/tcp   django-docker-container
```

#### 12. Interactive shell
```sh
$ sudo docker run -it django-docker-image sh
# pwd
/app
# ls -lah
total 44K
drwxr-xr-x 1 root root 4.0K Aug 24 18:17 .
drwxr-xr-x 1 root root 4.0K Aug 24 19:39 ..
-rw-rw-r-- 1 root root  201 Aug 23 18:31 .dockerignore
-rw-rw-r-- 1 root root 4.6K Aug 23 18:46 .gitignore
drwxrwxr-x 5 root root 4.0K Aug 23 18:56 .venv
-rw-rw-r-- 1 root root  773 Aug 24 18:14 Dockerfile
-rw-rw-r-- 1 root root 3.2K Aug 24 18:16 README.md
-rw-rw-r-- 1 root root    0 Aug 23 19:38 compose.yaml
-rw-r--r-- 1 root root    0 Aug 23 20:16 db.sqlite3
-rwxrwxr-x 1 root root  680 Aug 23 19:35 manage.py
drwxrwxr-x 3 root root 4.0K Aug 23 20:12 my_docker_django_project
-rw-rw-r-- 1 root root   66 Aug 23 20:00 requirements.txt
# whoami
root
# exit
$ 
```

#### 13. Running the container with Django project
```sh
$ sudo docker run -it -p 8000:8000 django-docker-image
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
August 25, 2025 - 18:11:00
Django version 5.2.5, using settings 'my_docker_django_project.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.

WARNING: This is a development server. Do not use it in a production setting. Use a production WSGI or ASGI server instead.
For more information on production servers see: https://docs.djangoproject.com/en/5.2/howto/deployment/
[25/Aug/2025 18:11:06] "GET / HTTP/1.1" 200 12068
Not Found: /favicon.ico
[25/Aug/2025 18:11:06] "GET /favicon.ico HTTP/1.1" 404 2226
```

> Although this is a great start in containerizing the application, you’ll need to make a number of improvements to get it ready for production.

#### 14. Make improvements to the Dockerfile
- change name of the first `Dockerfile` to `Dockerfile_v1`,  
- add `gunicorn` to the `requirements.txt` file,  
- create new Dockerfile`:  

```sh
# Stage 1: Base build stage
FROM python:3.13-slim AS builder
 
# Create the app directory
RUN mkdir /app
 
# Set the working directory
WORKDIR /app
 
# Set environment variables to optimize Python
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1 
 
# Upgrade pip and install dependencies
RUN pip install --upgrade pip 
 docker build -t django-docker-image-v2 .
# Copy the requirements file first (better caching)
COPY requirements.txt /app/
 
# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt
 
# Stage 2: Production stage
FROM python:3.13-slim
 
RUN useradd -m -r appuser && \
   mkdir /app && \
   chown -R appuser /app
 
# Copy the Python dependencies from the builder stage
COPY --from=builder /usr/local/lib/python3.13/site-packages/ /usr/local/lib/python3.13/site-packages/
COPY --from=builder /usr/local/bin/ /usr/local/bin/
 
# Set the working directory
WORKDIR /app
 
# Copy application code
COPY --chown=appuser:appuser . .
 
# Set environment variables to optimize Python
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1 
 
# Switch to non-root user
USER appuser
 
# Expose the application port
EXPOSE 8000 
 
# Start the application using Gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "3", "my_docker_django_app.wsgi:application"]
```
#### 15. Build new image
```sh
$ sudo docker build -t django-docker-image-v2 .
[+] Building 353.9s (16/16) FINISHED                                                    docker:default
 => [internal] load build definition from Dockerfile                                              0.0s
 => => transferring dockerfile: 1.30kB                                                            0.0s
 => [internal] load metadata for docker.io/library/python:3.13-slim                               3.2s
 => [internal] load .dockerignore                                                                 0.0s
 => => transferring context: 243B                                                                 0.0s
 => [internal] load build context                                                                 0.2s
 => => transferring context: 851.38kB                                                             0.2s
 => [builder 1/6] FROM docker.io/library/python:3.13-slim@sha256:27f90d79cc85e9b7b2560063ef44f  301.5s
 => => resolve docker.io/library/python:3.13-slim@sha256:27f90d79cc85e9b7b2560063ef44fa0e9eaae7a  0.0s
 => => sha256:9665f5d196e5856b0362a952ee582bcd28e7b790729de83f64b70975ff4481bd 1.75kB / 1.75kB    0.0s
 => => sha256:29e93e1cbf397bec486052ed53c12cfd104158418858c13a4e57329623f5e632 5.44kB / 5.44kB    0.0s
 => => sha256:396b1da7636e2dcd10565cb4f2f952cbb4a8a38b58d3b86a2cacb172fb7011 29.77MB / 29.77MB  300.5s
 => => sha256:7b1ad37e982c43c58d01111d612787fd261190d1587466b7c0469299af38debe 1.29MB / 1.29MB   70.1s
 => => sha256:31b0d2470ce12eba9b1d8d97762511adddf82bcc6f19cdc2acf5d459d6aab5 11.73MB / 11.73MB  215.6s
 => => sha256:27f90d79cc85e9b7b2560063ef44fa0e9eaae7a7c3f5a9f74563065c5477cc24 10.37kB / 10.37kB  0.0s
 => => sha256:8c141999d08327ce1db0a24fd7d25391dc2e4e0e2c14d70d0458fdf18761cfff 250B / 250B       88.0s
 => => extracting sha256:396b1da7636e2dcd10565cb4f2f952cbb4a8a38b58d3b86a2cacb172fb70117c         0.5s
 => => extracting sha256:7b1ad37e982c43c58d01111d612787fd261190d1587466b7c0469299af38debe         0.1s
 => => extracting sha256:31b0d2470ce12eba9b1d8d97762511adddf82bcc6f19cdc2acf5d459d6aab501         0.2s
 => => extracting sha256:8c141999d08327ce1db0a24fd7d25391dc2e4e0e2c14d70d0458fdf18761cfff         0.0s
 => [builder 2/6] RUN mkdir /app                                                                  0.3s
 => [stage-1 2/6] RUN useradd -m -r appuser &&    mkdir /app &&    chown -R appuser /app          0.4s
 => [builder 3/6] WORKDIR /app                                                                    0.1s
 => [builder 4/6] RUN pip install --upgrade pip                                                   2.0s
 => [builder 5/6] COPY requirements.txt /app/                                                     0.1s
 => [builder 6/6] RUN pip install --no-cache-dir -r requirements.txt                             44.4s
 => [stage-1 3/6] COPY --from=builder /usr/local/lib/python3.13/site-packages/ /usr/local/lib/py  0.7s
 => [stage-1 4/6] COPY --from=builder /usr/local/bin/ /usr/local/bin/                             0.1s
 => [stage-1 5/6] WORKDIR /app                                                                    0.0s
 => [stage-1 6/6] COPY --chown=appuser:appuser . .                                                0.7s
 => exporting to image                                                                            0.5s
 => => exporting layers                                                                           0.4s
 => => writing image sha256:90019425adf4d9dcee816e45598ba873de075a32307c29bfe19f6ff7b54f1423      0.0s
 => => naming to docker.io/library/django-docker-image-v2    

$ sudo docker image list
REPOSITORY               TAG       IMAGE ID       CREATED          SIZE
django-docker-image-v2   latest    90019425adf4   10 minutes ago   196MB
django-docker-image      latest    fe47a5ac0a68   2 hours ago      1.18GB
```
> Quite impressive difference of sizes :smiley:

#### 16. Configure the Docker Compose file  

- add new credentials to `.env` file:  
```sh
DJANGO_LOGLEVEL=info
DJANGO_ALLOWED_HOSTS=localhost
DATABASE_ENGINE=postgresql_psycopg2
DATABASE_NAME=dockerdjango
DATABASE_USERNAME=dbuser
DATABASE_PASSWORD=dbpassword
DATABASE_HOST=db
DATABASE_PORT=5432
```

- add `psycopg2-binary` to the `requirements.txt` file,  
- update `settings.py` to use PostgreSQL:  
```sh
DATABASES = {
     'default': {
         'ENGINE': f'django.db.backends.{config('DATABASE_ENGINE')}',
         'NAME': config('DATABASE_NAME'),
         'USER': config('DATABASE_USERNAME'),
         'PASSWORD': config('DATABASE_PASSWORD'),
         'HOST': config('DATABASE_HOST'),
         'PORT': config('DATABASE_PORT'),
     }
 }
```

#### 17. Build and run your new Django project
```sh
$ sudo docker compose up --build
```