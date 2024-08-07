
## Environment Setup for the Latest Version of Django (Python, Nginx, Postgres) Using Docker

### Project Structure

- `docker` - Folder for Docker configuration files
- `src` - Folder where the project code will be stored
- `.env` - Configuration values for the project
- `docker-compose.yml` - Docker compose configuration file
- `requirements.txt` - a list of packages or libraries needed to work on a project

### Step-by-Step Guide

#### 1. Environment Setup

- Create a `.env` file based on `.env.example` using the following command:

  ```
  cp .env.example .env
  ```
  
- Enter or leave the default port values for your project:

    ```
    DOCKER_HTTP_PORT=80
    DJANGO_PROJECT_NAME=project  # Project name
    NGINX_CONF_PATH='./docker/nginx.dev.conf'  # Path to the nginx configuration file,
                                          # leave as is for the development environment
    ```

- Fill in the database values:

    ```
    DB_USER
    DB_PASSWORD
    DB_DATABASE
    DB_HOST
    DB_PORT
    ```

#### 2. Build the Project Using Docker Compose

- Run this command
  
    ```
    docker compose build
    ```

#### 3. Create a Django Project

-  Create a Django project where the project name is "project" (if you change the project name, update the DJANGO_PROJECT_NAME in the .env file):

    ```
    docker compose run --rm web django-admin startproject project .
    ```

- After running this command, the project code should appear in the src folder.

- You can verify if the project is working by opening the browser and adding the port specified in DOCKER_HTTP_PORT. For example, if it’s set to 80:

  ```
  http://localhost
  ```

#### 4. Configure Django 

- Import the os library for environment variable handling in settings.py located in the project folder /src/project:

  ```
  import os
  ```

- Add STATIC_ROOT in settings.py:

  ```
  STATIC_ROOT = os.path.join(BASE_DIR, 'static')
  ```
  
- Configure Postgres in settings.py:

  ```
  DATABASES = {
      'default': {
          'ENGINE': 'django.db.backends.postgresql',
          'HOST': os.environ.get('DB_HOST'),
          'PORT': os.environ.get('DB_PORT'),
          'NAME': os.environ.get('DB_DATABASE'),
          'USER': os.environ.get('DB_USER'),
          'PASSWORD': os.environ.get('DB_PASSWORD'),
      }
  }
  ```

#### 5. Run Migrations

- Run this command
  
  ```
  docker compose run --rm web python manage.py migrate
  ```

- After running this command, you can access the admin panel at:

  ```
  http://localhost/admin
  ```

#### 6. Handle Static Files

- Uncomment the lines for the ENTRYPOINT in the python.Dockerfile, which runs the command for static files:

  ```
  COPY ./docker/entrypoint.sh /entrypoint.sh
  RUN chmod +x /entrypoint.sh

  ENTRYPOINT ["/entrypoint.sh"]
  ```

- Then, restart Docker:

  ```
  docker compose restart web
  ```
- Alternatively, you can run the following command:

  ```
  docker compose run --rm web python manage.py collectstatic
  ```

#### 7. If you have a domain for your site and want a free SSL certificate for HTTPS

- Uncomment the service in the docker-compose.yml file:

  ```
  certbot:
      image: certbot/certbot    
      volumes:
          - ./docker/certbot/www/:/var/www/certbot/:rw
          - ./docker/certbot/conf/:/etc/letsencrypt/:rw
  ```

- Uncomment volumes in nginx service in the docker-compose.yml file:

  ```
  - ./docker/certbot/www:/var/www/certbot:ro
  - ./docker/certbot/conf:/etc/letsencrypt
  ```
- Rebuild and restart the Docker containers using the command:

  ```
  docker compose up -d --build
  ```
- Then run the following command, replacing example.com with your domain:

  ```
  docker compose run --rm certbot certonly --webroot --webroot-path=/var/www/certbot -d example.com
  ```

- After this, you need to configure Nginx for HTTPS. To do this, change the value of NGINX_CONF_PATH in the .env file, replacing in file /docker/nginx.prod.conf localhost with your domain:

  ```
  NGINX_CONF_PATH='./docker/nginx.prod.conf'
  ```

