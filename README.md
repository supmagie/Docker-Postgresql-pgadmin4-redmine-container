Softwares and services versions we use for this project:

- Redmine v5.0.5
- PostgreSQL v16
- Pgadmin4 v8.1
- Ruby v3.0.2
- Ruby On rails v6.1.7.2

# Docker Redmine/Posgresql/Pgadmin4 container

## Install Docker

First of all, you need to install Docker-Desktop and docker-compose on your computer

### Install Docker on Mac

It's pretty simple to install Docker on Apple devices.
To install Docker on Mac, please follow this official tutorial from the Docker official website:

<https://docs.docker.com/engine/install/>

Please pay attention to the version you want to download according to the CPU you have.
Docker-compose will be installed at the same time.

### Install Docker on Linux

On a Linux machine it's a little bit more complicated to install Docker.
To install Docker on Linux, please follow this official tutorial from the Docker official website:

<https://docs.docker.com/desktop/install/linux-install/>

To install the docker-compose package you need a little bit more steps. Please follow this tutorial:

<https://docs.docker.com/compose/install/linux/>

### Install Docker on Windows

Even if I don't recommend to install Docker on a Windows machine since it was originally created for Linux OS, you can still use it.
To install Docker on Windows, please follow this official tutorial from the Docker official website:

<https://docs.docker.com/compose/install/linux/>

Docker-compose will be installed in the same time.
You will need to activate WSL in order to use Docker on a Windows Machine.

## Docker-compose file

### Get the file

Next step is to download the docker-compose you can find on this repository.
You can use the GUI and download directly a .zip file or you the CLI using `git clone` command.

Once you have the docker compose file, you can open it if you want to check or change the default some settings like username and password. The best is to put theses informations in differents file like `.env` file.
You can also use the default settings to do the test but don't forget to change it when you use it in Prod.

### Further explanations

Let's check further in this Docker compose file

```version: '3.9'

services:

    # db service
    postgresql:
        image: postgres:16
        container_name: postgresql
        ports:
            - "5432:5432"
        volumes:
            - ./postgres/init:/docker-entrypoint-initdb.docker
            - ./postgres/db-data:/var/lib/postgresql/data
        environment:
            POSTGRES_DB: redminedb
            POSTGRES_USER: redmine
            POSTGRES_PASSWORD: redmine
            POSTGRES_INIDB_ARGS: "--encoding=UTF-8"
        hostname: postgres
        restart: always
        user: root

    # db management service
    pgadmin4:
        image: dpage/pgadmin4:8.1
        container_name: pgadmin4
        ports:
            - "80:80"
        volumes:
            - ./pgadmin:/var/lib/pgadmin
        environment:
            PGADMIN_DEFAULT_EMAIL: root@root.com
            PGADMIN_DEFAULT_PASSWORD: root
        hostname: pgadmin4
        restart: always

    redmine:
        depends_on:
            - postgresql
        image: redmine:5.0.5-bookworm
        container_name: redmine
        ports:
            - "8080:3000"
        volumes:
            - ./redmine/files:/usr/src/redmine/files
            - ./redmine/log:/usr/src/redmine/log
            - ./redmine/plugins:/usr/src/redmine/plugins
            - ./redmine/public/themes:/usr/src/redmine/public/themes
        restart: always
        environment:
            REDMINE_DB_POSTGRES: postgres
            REDMINE_DB_DATABASE: redminedb
            REDMINE_DB_USERNAME: redmine
            REDMINE_DB_PASSWORD: redmine
            TZ: Asia/Tokyo

volumes:
    postgres: {}
    pgadmin: {}
    redmine: {}
```
    
As you can, we have here 3 services:
- postgresql
- pgadmin4
- redmine

#### Postgresql

I chose to use Postgresql but you can also use MySQL, SQLite...
For this tutorial, I'll only present how to use Postgresql.

```
    # db service
    postgresql:
        image: postgres:16
        container_name: postgresql
        ports:
            - "5432:5432"
        volumes:
            - ./postgres/init:/docker-entrypoint-initdb.docker
            - ./postgres/db-data:/var/lib/postgresql/data
        environment:
            POSTGRES_DB: redminedb
            POSTGRES_USER: redmine
            POSTGRES_PASSWORD: redmine
            POSTGRES_INIDB_ARGS: "--encoding=UTF-8"
        hostname: postgres
        restart: always
        user: root
```

I use the Official Docker Hub image of Postgresql version 16 (`postgresql:16`) so all the tests was made with this version. If you want to use a more recent version, it may not work.

About the container name, you can choose what you want. I choosed `postgresql`.

I also used the default ports for Postgresql: `5432`.

I created 2 volumes to stores the data on the hosts:

- The first one is to store an init file if you want to create specific settings when you start the container. It's the best idea for set you database with your own settings.
- The second one is to store the data of the databases.

Next step is to setup the environment. Let's check further:

`POSTGRES_DB: redminedb` is the name of the db we want to create.
If you don't set a name, Posgres will use the username as database name.

`POSTGRES_USER: redmine` is the user we want to set to access the db

`POSTGRES_PASSWORD: redmine` is the password of this user

`POSTGRES_INIDB_ARGS: "--encoding=UTF-8"` is to avoid issues with the init files

You can also add other environment settings. Please check the following Official Docker Hub link for more details:

<https://hub.docker.com/_/postgres>

I chose`postgres` as hostname. We will use it later.

The restart option should be `restart:always` because it's database and we want the container to restart if we have problem.

Finally, user is set to `root`.


#### Pgadmin4

Pgadmin4 allow you to access to databases from a web interface.

```
    pgadmin4:
        image: dpage/pgadmin4:8.1
        container_name: pgadmin4
        ports:
            - "80:80"
        volumes:
            - ./pgadmin:/var/lib/pgadmin
        environment:
            PGADMIN_DEFAULT_EMAIL: root@root.com
            PGADMIN_DEFAULT_PASSWORD: root
        hostname: pgadmin4
        restart: always
```

Here it's pretty the same things.

I chose the non Official image `dpage/pgadmin4:8.1`. At the time I write this tutorial, we don't have Docker Hub Official repository for pgadmin4.

I set the container name to `pgadmin4`.

I set the port to `80:80` to access using `http://localhost:80`.

I set the volumes `pgadmin/` to keep pgadmin data on the host.

I set the hostname to pgadmin4

And I set `restart:always` so Docker can restart the container is we have issue.

#### Redmine

According to the Official Redmine website: "Redmine is a flexible project management web application written using Ruby on Rails framework". If you need further explanations or the documentation, please follow this link:

<https://www.redmine.org/projects/redmine>

```
    redmine:
        depends_on:
            - postgresql
        image: redmine:5.0.5-bookworm
        container_name: redmine
        ports:
            - "8080:3000"
        volumes:
            - ./redmine/files:/usr/src/redmine/files
            - ./redmine/log:/usr/src/redmine/log
            - ./redmine/plugins:/usr/src/redmine/plugins
            - ./redmine/public/themes:/usr/src/redmine/public/themes
        restart: always
        environment:
            REDMINE_DB_POSTGRES: postgres
            REDMINE_DB_DATABASE: redminedb
            REDMINE_DB_USERNAME: redmine
            REDMINE_DB_PASSWORD: redmine
            TZ: Asia/Tokyo
```

First, I set `depends_on` so `posgresql` to redmine stack will wait for the posgresql service to start before start. It' perfect because we need the database to use Redmine. If you set a different name for the service associate to Posgresql, don't forget to change here too.

I chose the Official Docker Hub Redmine image to version 5.0.5 with bookworm install so I can access to more functionality if I want to access to the container. This image is really good because it install Ruby, Ruby on Rails, Redmine and even run the `rails server` command so when we launch the container, we can access directly to the homepage.
If you need a more recent image, you can try but it may not work.

I set the container name to `redmine`.

I set the port `8080` to access using `http://localhost:8080`. We can't use port 80 because pgadmin already using it.

I set different volumes so we can access easily to some folders from the host and keep data just in case.

I set `restart:always` so Docker can restart the container is we have issue.

I set the different environment settings:

`REDMINE_DB_POSTGRES: postgres` I tell to Redmine I want to use posgres as database management system.

`REDMINE_DATABASE: redminedb` because I set the `POSTGRES_DB` to `redminedb`. If you don't choose database name, the default one will be redmine.

`REDMINE_DB_USERNAME: redmine` because I set the `POSTGRES_USER` to `redmine`.

`REDMINE_DB_PASSWORD: redmine` because I set the `POSTGRES_PASSWORD` to `redmine`.

All theses information will be used to update automatically the config/database.yml file in the Redmine container.

And I set TZ: Asia/Tokyo because I live in Tokyo so it don't cause problem with the time.

## Launch the container

Create a folder and put the docker-compose.yml file on it. I choose `redmine` but you can choose what you want.

`mkdir redmine`

`mv DOCKER-COMPOSE.YML-PATH REDMINE-FOLDER`

Go to the new folder you just created:

`cd REDMINE-FOLDER`

Once you are in the folder, check you have the docker-compose.yml file

`ls`

Now you are ready to launch the container:

`docker-compose up -d`

With this command, you launch docker compose and you have directly access to the terminal when the containers are running.
This step may take time because Docker needs to pull the images if it's the first you are running this command for theses images.

If you want you can also launch:

`docker-compose up`  to access to the logs when you launch the containers.

If you want to access to the logs after launched the container, just tap:

`docker-compose logs -f ` from the REDMINE-FOLDER

So you can follow the logs.
If you got an error, please check your docker-compose.yml file.
You can also launch:

`docker-compose config` to check if the syntax of your file is correct. The indentation is really important here.

Next, check the containers are running with

`docker-compose ps`

If you can see the 3 services, that means they are running. If not, please double check your docker-compose.yml file.

## Access to the services

Now, your container are running, you should be able to access to your services.

### Pgadmin4

To access to pgadmin web interface, go to <http://localhost:80>.

To log in, use the username/password you chose in the docker-compose.yml file. For me it's `root@root.com` as username and `root` as password.
From here you can access to the main interface.

Click on `Add New Server` to access to your Postgresql database.
Choose the name you want, for example `posgresql`.

After that, go to the Connection tab on the top and add the information according to your docker-compose.yml file. For me it's:

```
Hostname: postgres
Port: 5432
Maintenance database: postgres
Username: redmine
Password: redmine
```

And press `Save`.

You should be able to see the database on the left.

### Redmine

To access to redmine web interface, go to <http://localhost:8080>.

To log in the default username/password is admin/admin.
You should be able to access to the default interface and start to use it.

## Set Redmine

### Install Dependencies

In the following steps, I'll adapt to my case.
First, we need to add a Line in the GemFile.

Open a CMD and tap
`Docker ps`
to get the container id. For me is something like ede8798yev

Tap
`docker exec -it ede bash` in order to access to the container.

Once you are inside, open the file call `GemFile` with Vim

`vim GemFile`

You may need to install Vim first so tap:

`apt-get update -y && apt-get install vim`

And add the following line at the end of the dependency list:

`gem 'matrix', '~> 0.4.2'`

Once it's done, tap

`gem install bundler`

`bundle install`

If you don't have error, you should have all the dependencies you need.

### Install plugins

Redmine propose a lot of different plugins. Some are free and some you need to pay for but they have a pretty big list.
If you decide to install a plugin, please follow the following steps.

You can do the first steps on the Host.
Open you Redmine folder (created by the volume) and go to `plugins/`
Once you are in this folder, create a folder name `plugin_assets/`

Paste the plugin(s) you have (Each plugin is a folder so copy directly the folder).

Once it's done, open a CMD and tap:

`docker exec -it ede bash` in order to access to the container.

Tap:
`chown -R 755 plugins/plugin_assets/` to set the correct right to the new folder

Once it's done, tap:

`bundle exec rake redmine:plugins:migrate RAILS_ENV=production` to install the plugins.

If you want to check, you can open Redmine on the browser (<http://localhost:8080>), log in, go to **Administration** > **Plugins**

You shoud see the list of plugins.

### Change theme

Redmine software propose also to change the theme.
If you want to change it, please follow the following steps:

On the host, Open your `Redmine/` folder and go to `public/themes/`
You now may need to restart Redmine container in order to see the theme.

`docker restart ede`

Once it's done, open Redmine on the browser (<http://localhost:8080>), log in, go to **Administration** > **Settings** > **Display** and select the theme you want. Don't forget to **Save**.

You should see the new Theme.

### Migrate database

#### Dbeaver preparation

First, download a software to manage database.
For my case, I will use **DBeaver** on Mac

Once you have the software, if it's the first time you use it, you need to create a new local database client.

Using Homebrew, install libpq package for PostgreSQl

`brew install libpq`

For MySQL, install mysql-client package

`brew install mysql-client`

To find the installation path for the local client of a database in Dbeaver for PosgreSQL, open CMD and tap:

find / -name "pg_dump" 2>/dev/null

You should get the path so copy this path and open Dbeaver.



=== ADD HERE ===


For more information about this, please check the following link: <https://dbeaver.com/docs/dbeaver/Local-Client-Configuration/>

#### Migration



=== ADD HERE ===
