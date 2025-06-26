# Videoflix

![videoflix](https://raw.githubusercontent.com/SarahZimmermann-Schmutzler/videoflix_deploy/main/img_github/videoflix.png)

**Videoflix** is a video streaming platform in the style of [Netflix](https://www.netflix.com/de/).

The project was created as part of my training at the Developer Akademie. This is the **final project of the Backend course**.

## Table of Contents

1. [Prerequisites](#prerequisites)
1. [Quickstart](#quickstart)
1. [Usage](#usage)
   * [Setup Guide](#setup-guide)
   * [Description](#description)
     * [Backend](#backend)
     * [Frontend](#frontend)

## Prerequisites

* **Angular CLI** 15.2.0, [More Information](https://github.com/angular/angular-cli)
* **Python** 3.12.2, [More Information](https://www.python.org/)
  * **Django** 5.0, [More Information](https://www.djangoproject.com/)
    * **Django REST framework** 3.14.0, [More Information](https://www.django-rest-framework.org/)
    * **Django RQ** 2.10.2 [More Information](https://django-rq.readthedocs.io/)
* **PostgreSQL** via psycog2 2.9.9. [More Information](https://pypi.org/project/psycopg2/)
* * **Redis** 5.0.4 [More Information](https://redis.io/)
* **ffmpeg** [More Information](https://ffmpeg.org/)
* **Docker** 24.0.7, [More Information](https://www.docker.com/)
  * **Compose** v2.32.4, [More Information](https://docs.docker.com/compose/)

## Quickstart

This section provides a **minimal setup guide**. For detailed instructions and alternative setups without GitHub Actions, see [Usage](#usage) section.  

1. [Fork](https://docs.github.com/de/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) the project to your namespace.

1. Set the required **Actions secrets and variables**
   * See the full list [here](#usage)

1. Create a **persistent volume** for PostgreSQL data on your remote server:

   ```bash
   docker volume create videoflix-db-volume
   ```

1. Create a **persistent volume** for media files on your remote server:

   ```bash
   docker volume create videoflix-media-volume
   ```

1. **Manually trigger** the deployment workflow
   * In the GitHub UI, you can find it under:
     * **Actions > Build & Deploy Videoflix > Run workflow**

## Usage

### Setup Guide

**Videoflix** can be run in three ways:

* [**For local development without Docker**](https://github.com/SarahZimmermann-Schmutzler/videoflix_deploy/blob/main/alternative_setups.md#run-videoflix-locally-without-docker)
* [**Using Docker manually**](https://github.com/SarahZimmermann-Schmutzler/videoflix_deploy/blob/main/alternative_setups.md#run-videoflix-with-docker-manually)
* **Automatically on a remote server via GitHub Actions** – follow the steps below:

1. [Fork](https://docs.github.com/de/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) the project to your namespace.

1. Set the required **Actions secrets and variables**:

   * To run the **GitHub Actions workflow** and generate the **.env file**, you must configure the following secrets and variables under:
      * **Settings > Security > Secrets and variables > Actions**

    | Key | Description |
    | --- | ----------- |
    | **Secrets for workflow** | |
    | REMOTE_HOST | IP address of remote server |
    | REMOTE_USER | Username on remote server |
    | SSH_PORT | SSH Port |
    | SSH_PRIVATE_KEY | Private SSH key of remote server |
    | DEPLOY_DIR | Path to the folder in which the project should be published |
    | **Secrets for application used in .env** | |
    | SECRET_KEY | Essential cryptographic key used by Django to protect sensitive data and provide security-critical functionality |
    | GMAIL_PWD | App password used to authenticate the Gmail account that sends emails |
    | EMAIL_SENDER | Email address used as the sender in outgoing system emails |
    | REDIS_PASSWORD | Password required to connect to your Redis instance |
    | POSTGRES_USER | Username for connecting to the PostgreSQL database |
    | POSTGRES_PASSWORD | Password for the PostgreSQL database user defined in POSTGRES_USER |
    | **Secrets for containerization used in .env** | |
    | DJANGO_SUPERUSER_USERNAME | Username to create a superuser for the admin panel |
    | DJANGO_SUPERUSER_EMAIL | Email address to create a superuser for the admin panel |
    | DJANGO_SUPERUSER_PASSWORD | Password to create a superuser for the admin panel |
    | **Variables for workflow** | |
    | IMAGE_OWNER | Specifies the namespace under which the Docker image will be published to the GitHub Container Registry (ghcr.io) |
    | **Variables for application used in .env** | |
    | DEBUG | Set to True to enable debug mode for local development/testing; Set to False in production environments |
    | IP_ADDRESS_VM | Server IP address or Domain, added to the ALLOWED_HOSTS list in settings.py |
    | CORS_ALLOWED_ORIGINS | List of all hosts with their portnumbers, added to settings.py |
    | ACTIVATION_URL | URL/IP address of the Frontend (similiar to value of IP_ADDRESS_VM) |
    | REDIS_HOST | Name of Redis service. The default is "redis" |
    | REDIS_PORT | Port number on which the Redis service is running. The default is 6379 |
    | POSTGRES_DB | Name of the PostgreSQL database to be created and used by the application |
    | POSTGRES_HOST | Name of PostgreSQL service. The default is "db" |
    | POSTGRES_PORT | Port number on which the PostgreSQL service is running. The default is 5432 |
    | **Variables for containerization used in .env** | |
    | BACKEND_EXTERNAL_PORT | External portnumber for backend container |
    | FRONTEND_EXTERNAL_PORT | External portnumber for frontend container |

1. Create a **persistent volume** for PostgreSQL on your remote server:

   * To ensure your **data is not lost after container restarts**:
  
      ```bash
      docker volume create videoflix-db-volume
      ```

1. Create a **persistent volume** for media files on your remote server:

   ```bash
   docker volume create videoflix-media-volume
   ```

1. **Manually trigger** the deployment workflow

   * In the `deployment.yml`, the `workflow_dispatch` trigger is enabled, so the workflow must be triggered manually:
     * In the GitHub UI, you can find it under:
       * **Actions > Build & Deploy Kenben > Run workflow**

   * <ins>Alternative</ins>: Enable **Auto-Deploy on Push** if you'd prefer automatic deployment when pushing to the main branch:
     * [Clone](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository) the project to your platform:
       * <ins>Example</ins>: Clone the repo e.g. using an SSH-Key:  

          ```bash
          git clone git@github.com:Your-Name/videoflix_deploy.git
          ```

       * Navigate to the **project directory**:

          ```bash
          cd videoflix_deploy
          ```

     * Enable **lines 4-5** in [`deployment.yml`](https://github.com/SarahZimmermann-Schmutzler/videoflix_deploy/blob/main/.github/workflows/deployment.yml)

     * **Commit and push** the change to the `main branch` - the workflow will run automatically

### Description

**Videoflix** is designed as a **fully separated frontend and backend architecture** that communicates via a **RESTful API**

#### <ins>Backend</ins>

The **main goals** were to:

* Use **Django** and the **Django REST Framework (DRF)** to deepen knowledge of RESTful backend development
* Implement an **authentication system** to manage secure user access
* Configure **Redis** as an in-memory caching layer
  * All available videos and thumbnails are cached for **15 minutes** to improve performance and reduce database load
* Automatically convert uploaded videos in the background into 360p, 720p and 1080p resolutions using **ffmpeg**
  * These conversions are handled asynchronously via **Django RQ** as background task runner
* Use a **PostgreSQL** database as the main data store, replacing the default SQLite setup
* Deploy the backend on a **virtual machine (VM)** for greater control and scalability

The backend is devided into **two main areas**:

* **Authentication and Authorization**
  * Manages user registration, login, token-based authentication, email confirmation, account activation prior to first login, password reset, and logout

* **Videoflix Module**
  * Contains the core logic and the **Video model**, which includes the following fields:
    * `created_at`
    * `title`
    * `description`
    * `video_file`
    * `preview_img`

#### <ins>Frontend</ins>

The frontend was developed using **Angular** with the goal of replicating the **look and feel of Netflix**.
  
**Key features** of the frontend include:

* **Authentication**:
  * **Homepage** - Entry point with links to registration, login, and legal information
  ![homepage](https://raw.githubusercontent.com/SarahZimmermann-Schmutzler/videoflix_deploy/main/img_github/homepage.png)
  * **Login page** - Simple login form using email and password
  ![login](https://raw.githubusercontent.com/SarahZimmermann-Schmutzler/videoflix_deploy/main/img_github/login.png)
  * **Register page** - Allows new users to sign up with email and password
  ![register](https://raw.githubusercontent.com/SarahZimmermann-Schmutzler/videoflix_deploy/main/img_github/register.png)
  * **Password Reset page** - Enables users to request a password reset via email
  ![password_reet](https://raw.githubusercontent.com/SarahZimmermann-Schmutzler/videoflix_deploy/main/img_github/password_reset.png)

* **Video overview**:
  * Displays a grid of **all available videos**
  ![videoflix](https://raw.githubusercontent.com/SarahZimmermann-Schmutzler/videoflix_deploy/main/img_github/videoflix.png)

* **Video detail view**:
  * Shows **detailed information** about the selected video
  * Allows the user to **choose between available resolutions**
  * Embedded **video player** for seamless streaming
  ![video_detail](https://raw.githubusercontent.com/SarahZimmermann-Schmutzler/videoflix_deploy/main/img_github/video_detial.png)
