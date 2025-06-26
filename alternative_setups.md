# Videoflix - Alternative Setup Guides

If you don't want to work with Github Actions, there are **other ways to use Videoflix**, either locally or with Docker.

## Table of Contents

1. [Run Videoflix locally without Docker](#run-videoflix-locally-without-docker)
1. [Run Videoflix with Docker manually](#run-videoflix-with-docker-manually)

## Run Videoflix locally without Docker

1. [Clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) the project to your platform:
    * <ins>Example</ins>: Clone the repo e.g. using an SSH-Key:  

        ```bash
        git clone git@github.com:SarahZimmermann-Schmutzler/videoflix_deploy.git
        ```

    * Navigate to the **project directory**:

        ```bash
        cd videoflix_deploy
        ```

1. Install, start and configure **PostgreSQL**:

    ```bash
    sudo apt install postgresql postgresql-contrib
    sudo systemctl start postgresql
    ```

    * **Create User and DB** with `psql` or `pgAdmin`

1. Install and start **Redis**:

    ```bash
    sudo apt install redis-server
    sudo systemctl start redis
    ```

1. Install **ffmpeg**:

    ```bash
    sudo apt install ffmpeg
    ```

1. Navigate into the **backend directory** and create an `.env` file with the following variables:

    ```bash
    cd videoflix
    nano .env
    ```

    | Variable | Description | Default Value |
    | -------- | ----------- | ------------- |
    | DEBUG | Set to True to enable debug mode for local development/testing; Set to False in production environments | True |
    | SECRET_KEY | Essential cryptographic key used by Django to protect sensitive data and provide security-critical functionality - [create here](https://stackoverflow.com/questions/41298963/is-there-a-function-for-generating-settings-secret-key-in-django) | your-secret-key |
    | GMAIL_PWD | App password used to authenticate the Gmail account that sends emails | your app password |
    | EMAIL_SENDER | Email address used as the sender in outgoing system emails | example@gmail.com |
    | ACTIVATION_URL | URL/IP address of the Frontend | 127.0.0.1:4200 |
    | REDIS_HOST | Name of Redis service | localhost |
    | REDIS_PORT | Port number on which the Redis service is running | 6379 |
    | REDIS_PASSWORD | Password required to connect to your Redis instance | changeme123 |
    | POSTGRES_DB | Name of the PostgreSQL database to be created and used by the application | videoflix_db |
    | POSTGRES_USER | Username for connecting to the PostgreSQL database | postgres |
    | POSTGRES_PASSWORD | Password for the PostgreSQL database user defined in POSTGRES_USER | changeme123 |
    | POSTGRES_HOST | Name of PostgreSQL service | localhost |
    | POSTGRES_PORT | Port number on which the PostgreSQL service is running | 5432 |

1. Create a **virtual environment** and **install Python packages**:

    ```bash
    python -m venv env
    source venv/bin/activate
    pip install -r requirements.txt
    ```

1. **Prepare and start Django**:

    ```bash
    python manage.py migrate
    python manage.py createsuperuser
    python manage.py rqworker
    ```

    * You should eventually run the **RQ worker as a background service** using `systemd` or `supervisor`.

    * In a **second terminal**:

        ```bash
        python manage.py runserver
        ```

1. In a **new terminal window**, navigate into the **frontend directory**, **install the frontend dependencies** and **start** the frontend on http://localhost:4200/:

    ```bash
    cd ..
    cd videoflix_frontend
    npm install
    ng serve
    ```

## Run Videoflix with Docker manually

1. [Clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) the project to your platform:
    * <ins>Example</ins>: Clone the repo e.g. using an SSH-Key:  

        ```bash
        git clone git@github.com:SarahZimmermann-Schmutzler/videoflix_deploy.git
        ```

    * Navigate to the **project directory**:

        ```bash
        cd videoflix_deploy
        ```

1. Init and update the **submodules**:

    ```bash
    git submodule init
    git submodule update
    ```

1. Configure the **environment variables**:
    * Copy the content of the [`example.env`](https://github.com/SarahZimmermann-Schmutzler/kenben_deploy/blob/main/example.env) file into an .env file.

    | Variable | Description | Default Value |
    | -------- | ----------- | ------------- |
    | **For backend application** | | |
    | DEBUG | Set to True to enable debug mode for local development/testing; Set to False in production environments | True |
    | SECRET_KEY | Essential cryptographic key used by Django to protect sensitive data and provide security-critical functionality - [create here](https://stackoverflow.com/questions/41298963/is-there-a-function-for-generating-settings-secret-key-in-django) | your-secret-key |
    | IP_ADDRESS_VM | Server IP address or Domain, added to the ALLOWED_HOSTS list in settings.py | 127.0.0.1:4200 |
    | CORS_ALLOWED_ORIGINS | List of all hosts with their portnumbers, added to settings.py | http://localhost:4200 |
    | GMAIL_PWD | App password used to authenticate the Gmail account that sends emails | your-app-password |
    | EMAIL_SENDER | Email address used as the sender in outgoing system emails | example@gmail.com |
    | ACTIVATION_URL | URL/IP address of the Frontend | http://127.0.0.1:4200 |
    | REDIS_HOST | Name of Redis service | redis |
    | REDIS_PORT | Port number on which the Redis service is running | 6379 |
    | REDIS_PASSWORD | Password required to connect to your Redis instance | changeme123 |
    | POSTGRES_DB | Name of the PostgreSQL database to be created and used by the application | videoflix_db |
    | POSTGRES_USER | Username for connecting to the PostgreSQL database | postgres |
    | POSTGRES_PASSWORD | Password for the PostgreSQL database user defined in POSTGRES_USER | changeme123 |
    | POSTGRES_HOST | Name of PostgreSQL service | db |
    | POSTGRES_PORT | Port number on which the PostgreSQL service is running | 5432 |
    | **For containerization** | | |
    | BACKEND_EXTERNAL_PORT | External portnumber for backend container | 8000 |
    | FRONTEND_EXTERNAL_PORT | External portnumber for frontend container | 8080 |
    | DJANGO_SUPERUSER_USERNAME | Username to create a superuser for the admin panel | admin |
    | DJANGO_SUPERUSER_EMAIL | Email address to create a superuser for the admin panel | admin@example.com |
    | DJANGO_SUPERUSER_PASSWORD | Password to create a superuser for the admin panel | changeme123 |

1. Create a **persistent volume** for PostgreSQL on your remote server:

   * To ensure your **data is not lost after container restarts**:
  
      ```bash
      docker volume create videoflix-db-volume
      ```

1. Create a **persistent volume** for media files on your remote server:

   ```bash
   docker volume create videoflix-media-volume
   ```

1. **Build and start the containers** in the background (detached mode):

    ```bash
    docker compose up --build -d
    ```

1. Check whether the **containers are running** correctly:

    ```bash
    docker ps -a
    ```
