# heroku-docker-django
Deployment Guide



1. In project directory-

	    sudo pip3 insatll pipenv

2. `pipenv install django gunicorn`

3.  Activate it

	    pipenv shell

4. Add "gunicorn" to requirements.txt

5. Copy or Create Djnago Project here and up and run with requirements.txt

6. DEBUG can be True/False or 1/0

	In settings.py-

       DEBUG = int(os.environ.get('DEBUG', default=1)) 

8. `Create .env` and add following-

    DEBUG=1
   
9. Test Gunicorn
   

		gunicorn blurit.wsgi:application --bind 0.0.0.0:8000

----------------------------------------------------------------
10. `nano Dockerfile`

11. paste below conetent

	    # Base Image
		FROM python:3.6

		# create and set working directory
		RUN mkdir /app
		WORKDIR /app

		# Add current directory code to working directory
		ADD . /app/

		# set default environment variables
		ENV PYTHONUNBUFFERED 1
		ENV LANG C.UTF-8
		ENV DEBIAN_FRONTEND=noninteractive 

		# set project environment variables
		# grab these via Python's os.environ
		# these are 100% optional here
		ENV PORT=8888

		# Install system dependencies
		RUN apt-get update && apt-get install -y --no-install-recommends \
		        tzdata \
		        python3-setuptools \
		        python3-pip \
		        python3-dev \
		        python3-venv \
		        git \
		        && \
		    apt-get clean && \
		    rm -rf /var/lib/apt/lists/*


		# install environment dependencies
		RUN pip3 install --upgrade pip 
		RUN pip3 install pipenv

		# Install project dependencies
		RUN pipenv install --skip-lock --system --dev

		EXPOSE 8888
		CMD gunicorn cfehome.wsgi:application --bind 0.0.0.0:$PORT

12. ##### Build your Docker Image

	```
	$ cd path/to/your/dev/folder
	$ cd docker-djnago0project
	$ ls
	Dockerfile
	```
	    docker build -t bluritai-docker -f Dockerfile .

13. Docker check image-
	  
		docker images

14. Run image-

	For testing on local-

	    docker run -it -p 80:8888 bluritai-docker

	For testing on local (Background)-

		docker run -it -d -p 80:8888 bluritai-docker

	Check-

		docker ps
		
15. **Go to browser-**

"http://localhost"	, "https://0.0.0.0", "http://localhost:8888"	, "https://0.0.0.0:8888"

15. **Clean up**

List running containers

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                    PORTS               NAMES
5966a461fa9d        simple-django-on-docker         "/bin/sh -c 'gunicorn …"   3 seconds ago       Exited (0) 1 second ago                       thirsty_cori
```

Stop your container

Copy

```
docker stop 5966a461fa9d
docker rm 5966a461fa9d
```

Remove Image-

    docker images
 Then-
    
    docker image srm <id>


-------------------

## Deploy Django with Docker & Heroku

1. Sign up for  [Heroku](https://www.heroku.com/)

2. Download the  [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)

3. RUN-


	   heroku create blurai-docker
	   
	   heroku container:login

	   heroku addons:create heroku-postgresql:hobby-dev -a blurai-docker


4. Update Django Database Settings
In  `settings.py`  uncomment the following lines:

	    import dj_database_url
	    db_from_env = dj_database_url.config()
	    DATABASES['default'].update(db_from_env)
	    DATABASES['default']['CONN_MAX_AGE'] = 500

	***A. In requirements.txt add-***

	    dj_database_url

	In Dockerfile add-

	    RUN apt-get install python3-psycopg2	
	    
	***B. If above (A) not works then-***

	In requirements.txt add-

	    dj_database_url
	    psycopg2-binary

	In Dockerfile add nothing
	    
5. Allowed Host in setings.py add- `["*"]`

6. Test Gunicorn
   
		gunicorn blurit.wsgi:application --bind 0.0.0.0:8000


## Modify Dockerfile for Heroku-

1. Create a backup of original Dockerfile

2. `nano Dockerfile`

3. Heroku handle port automatically-

	comment-

	    # EXPOSE 8888

4. Just test-

	    docker build -t bluritai-docker -f Dockerfile .

5. Push to heroku-

		heroku container:push web -a blurai-docker

6. Release-

	```
	heroku container:release web -a blurai-docker
	```
	 
7. Open App

	    heroku open -a blurai-docker



8.  Run  `migrate`  &  `createsuperuser`

With heroku you can just simply run:

**Migrate**

```
heroku run python3 manage.py migrate
```

or


```
heroku run python3 manage.py migrate -a blurai-docker
```

**collectstatic** 

```
heroku run python3 manage.py collectstatic -a blurai-docker
```

**Create super user**


```
heroku run python3 manage.py createsuperuser
```

or


```
heroku run python3 manage.py createsuperuser -a blurai-docker
```

**Bash**


```
heroku run bash
```

or


```
heroku run bash -a blurai-docker
```


