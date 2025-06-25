<!-- > ⚠️ **This repository has been archived**
> 
> All efforts to provide Docker images for PKP tools, along with specialized Docker Compose setups (production, development, etc.), have been consolidated in a new repository:  
> 👉 [pkp/docker-ojs](https://github.com/pkp/docker-ojs)  
>  
> If you have any questions or suggestions, please use the issue tracker in the new repository:  
> https://github.com/pkp/docker-ojs/issues
>
> **easyOJS** instructions are still valid and will be moved soon to the new respository.

# easyOJS - OJS in a docker
-->

| **IMPORTANT:** |
|:---------------------------------------------------------|
| In the Hannover sprint, we decided to mutate this repository into an **easy howto for running OJS** with the official dockerHub containers.<br /> This is now the recommended way to run dockerized OJS in test and production.  |
| To be safe in case dockerHub changes it's usage terms and to test alternative CI/CD tools, we also created a [temporary gitLab repository](https://gitlab.com/pkp-org/docker), that is the one we use to build the images you can find in dockerHub. This is a temporary change as most likely in the near future, we will move back to this same repository and use gitHubActions. Sorry for the inconvenience. |
| Anyway, generated in gitLab or gitHub, as long as it is a free service, we will keep pushing images to dockerHub. |

Open Journal Systems (OJS) is a journal management and publishing system that has been developed by the [Public Knowledge Project](https://pkp.sfu.ca/) through its federally funded efforts to expand and improve access to research.

The images in this repository are built on top of [Alpine Linux](https://alpinelinux.org/) and come in several variants (see [Versions](#versions)).

This repository is a fork of the work formerly done by [Lucas Dietrich](https://github.com/lucasdiedrich/ojs) and [Marc Bria](https://github.com/marcbria/docker-ojs).

This new project is maintained by [Marc Bria](https://github.com/marcbria).


## How to use

### TL;DR;

Be sure you properly installed `docker` and `docker-compose` and the `docker` service is running.

```
git clone https://github.com/pkp/docker-ojs.git
mv docker-ojs journalName && cd journalName
mv .env.TEMPLATE .env
vim .env                         					# Set environment variables as you wish (ojs version, ports, url...)
source .env && wget "https://github.com/pkp/ojs/raw/${OJS_VERSION}/config.TEMPLATE.inc.php" -O ./volumes/config/ojs.config.inc.php
sudo chown 100:101 ./volumes -R && sudo chown 999:999 ./volumes/db -R	# Ensure folders got the propper permissions
docker compose up -d
# Visit your new site and complete the installation as usual (Read about DB access credentials below, in step 5).

```

### Extended version

If you want to run it locally (or in your own server), first you need to install
[docker](https://docs.docker.com/get-docker/) (even [docker-compose](https://docs.docker.com/compose/install/) it's also recommended).

You can have it all up and running in less than 10 minutes following this brief howto:
https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-debian-10

After this, notice that for all available versions, we provide a **docker-compose** configuration file so
you will be able to start a full OJS stack (web app + database containers) in 4 easy steps:

1. Clone this repository in your machine (if youprefer, you can also [download](https://github.com/pkp/docker-ojs/archive/master.zip) and unzip it):

    ```bash
    $ git clone https://github.com/pkp/docker-ojs.git
    $ mv docker-ojs journalName && cd journalName
    ```

   Replace "journalName" with a short name of your journal (probably you will like to set the same value you use for COMPOSE_PROJECT_NAME variable).

2. Set your environment variables

    ```bash
    $ mv .env.TEMPLATE .env
    ```

    Edit your .env file to fit your need to fit your needs.
    You will probably like to chage your OJS_VERSION, ports, and names.
    For a detailed description of all the environment variables take a look to ["Environment Variables"](#environment-variables) sectionj.

3. Download the ojs config file related to your desired version

    ```bash
    $ source .env && wget "https://github.com/pkp/ojs/raw/${OJS_VERSION}/config.TEMPLATE.inc.php" -O ./volumes/config/ojs.config.inc.php
    ```

    If your are running docker on windows (with Powershell), specify the version you like to download:

   ```bash
    $ wget "https://github.com/pkp/ojs/raw/3_3_0-21/config.TEMPLATE.inc.php" -O ./volumes/config/ojs.config.inc.php
    ```

4. Run the stack:
    ```bash
    $ docker compose up
    ```

    Docker-compose will pull images from dockerHub and do all the hard work for you to rise a full functional OJS stack.
    If all goes as expected you will see your app_container informing apache is RUNNING successfully.

    ```
    INFO success: apache entered RUNNING state, process has stayed up for > than 1 seconds (startsecs) 
    ```

    You can add the "-d" parameter to the call if you like to run it detached.

4. Access **http://127.0.0.1:8081** and continue through web installation process.

    Note that the database connection needs the following options:

    - **Database driver**: `mysqli` (or "mysql" if your php is lower than 7.3)
    - **Host**: `db` (which is the name of the container in the internal Docker network)
    - **Username**: `ojs`
    - **Password**: `setYourPass` (change with the password you set in your environment variables)
    - **Database name**: `ojs`
    - _Uncheck_ "Create new database"
    - _Uncheck_ "Beacon"

    And the  "Directory for uploads:" acording to your docker-compose.yml "/var/www/files"

    | **TIP:**             |
    |:---------------------|
    | To go through the OJS installation process automatically, set the environment variable `OJS_CLI_INSTALL=1`, and use the other .env variables to automatize the process. |

That's all. Easy peasy, isn't it?

Ok, let's talk talk about more complex concepts and scenarios.

<!-- 
## Building local images

The official image will work for 90% of the people but, if you don't want external dependencies or you like to modify our official Dockerfiles to fit your specific needs you will need to build your images in your machine.

Each version folder also includes an alternative yml file called `docker-compose-local.yml`.

This compose won't ask dockerHub for the required images, it will build a docker image locally.

To do this...

1. Go to your preferred version folder and and build the image as follows:
    ```bash
    $ docker build -t local/ojs:3_2_1-4 .
    ```

    If something goes wrong, double-check if you ran the former command with the right version number or in a folder without the local Dockerfile.

2. Once the image is built, you can run the stack locally telling compose to use the local yaml file with the `-f`/`--file` option as follows:
    ```bash
    $ docker compose --file docker-compose-local.yml up
    ```
-->

## Versions

Different OJS versions are combined with different versions of PHP (5 to 8...). In future we are planning to add variants with different web servers ([Apache HTTP Server](https://httpd.apache.org/), [nginx](https://nginx.org/)) and tools.

_Currently, not all these combinations work! We are mostly focused in Apache2. PR are welcome_

All version tags can be found at [Docker Hub Tags tab](https://hub.docker.com/r/pkpofficial/ojs/tags/).

(If no webserver is mentioned in the tag, then Apache is used).


## Environment Variables

The image understand the following environment variables:

| NAME                   | Default      | Info                 |
|:----------------------:|:------------:|:---------------------|
| SERVERNAME             | localhost    | Used to generate httpd.conf and certificate            |
| OJS_IMAGE              | ojs          | PKP tool to be used (ojs, omp, ops). Only OJS images avaliable right now |
| OJS_VERSION            | 3_3_0-21     | OJS version to be deployed |
| COMPOSER_PROJECT_NAME  | journal      | 
| OJS_CLI_INSTALL        | 0            | Used to install ojs automatically when container starts |
| OJS_DB_HOST            | db           | Database host        |
| OJS_DB_USER            | ojs          | Database             |
| OJS_DB_PASSWORD        | setYourPass  | Database password    |
| OJS_DB_NAME            | ojs          | Database name        |
| HTTP_PORT              | 8081         | Http port            |
| HTTPS_PORT             | 8481         | Https port           |


_**Note:** OJS_CLI_INSTALL and certificate features are under construction._

## Special Volumes

Docker content is efimerous by design, but in some situations you would like
to keep some stuff **persistent** between docker restarts (ie: database content,
upload files, plugin development...)

By default we include an structure of directories in the version folders
(see ./volumes), but they are empty and disabled by default in your compose.
To enable them, **you only need to uncomment the volume lines in
your docker-compose.yml** and fill the folders properly.

When you run the `docker-compose` it will mount the volumes with persistent
data and will let you share files from your host with the container.

This is the usual volumes you will probably like to map:


| Host                                    | Container  | Volume                                | Description                    |
|:----------------------------------------|:----------:|:--------------------------------------|:-------------------------------|
| ./volumes/public                        | ojs        | /var/www/html/public                  | All public files               |
| ./volumes/private                       | ojs        | /var/www/files                        | All private files (uploads)    |
| ./volumes/config/db.charset.conf        | db         | /etc/mysql/conf.d/charset.cnf         | mariaDB config file            |
| ./volumes/config/ojs.config.inc.php     | ojs        | /var/www/html/config.inc.php          | OJS config file                |
| ./volumes/config/php.custom.ini         | ojs        | /usr/local/etc/php/conf.d/custom.ini  | PHP custom.init                |
| ./volumes/config/apache.htaccess        | ojs        | /var/www/html/.htaccess               | Apache2 htaccess               |
| ./volumes/logs/app                      | ojs        | /var/log/apache2                      | Apache2 Logs                   |
| ./volumes/logs/db                       | db         | /var/log/mysql                        | mariaDB Logs                   |
| ./volumes/db                            | db         | /var/lib/mysql                        | mariaDB database content       |
| ./volumes/db-import                     | db         | /docker-entrypoint-initdb.d           | DB init folder (with SQLs)     |
| ./volumes/plugins                       | ojs        | /var/www/html/plugins                 | Ensure host plugins are sync with the ojs version |
| /etc/localtime                          | ojs        | /etc/localtime                        | Sync clock with the host one.  |
| TBD                                     | ojs        | /etc/ssl/apache2/server.pem           | SSL **crt** certificate        |
| TBD                                     | ojs        | /etc/ssl/apache2/server.key           | SSL **key** certificate        |

In this image we use "bind volumes" with relative paths because it will 
give you a clear view where your data is stored.

The down sides of those volumes is they can not be "named" and docker will 
store them with an absolute path (that it’s annoying to make stuff portable) 
but I prefer better control about where data is stored than leave it in docker hands.

And remember this is just an image, so feel free to modify to fit your needs.

You can add your own volumes. For instance, make sense for a plugin developer
or a themer to create a volume with his/her work, to keep a persistent copy in
the host of the new plugin or theme (see plugins mapping in former table).

An alternative way of working for developers is working with his/her own local
Dockerfile that will be build to pull the plugin for his/her own repository...
but this will be significantly slower than the volume method.

Last but not least, those storage folders need to exist with the right permissions
before you run your `docker compose` command or it will fail.

To be sure your volumes have the right permissions, you can run those commands:

   ```bash
   $ sudo chown 100:101 ./volumes -R
   $ sudo chown 999:999 ./volumes/db -R
   $ sudo chown 999:999 ./volumes/logs/db -R
   ```

So, permissions for volumes folders are... all the content will be owned by apache2 
user and group (uid 100 and gid 101 inside the container), execpt for db and logs/db
folders that will be owned by mysql user and group (uid and gid 999).

    | **TIP:**             |
    |:---------------------|
    | The mysql/mariadb images include a nice entrypoint that let you run the scripts and sql dumps *if your DB is not yet created*. This feature is great for sites migrations, testing, demos, development dummy data, etc. You only need to include you mysqldump file in your `volumes/db-import` folder, map this folder as volume in your docker-compose.yml and start your container to enjoy the magic. |

## Built in scripts

The Dockerfile includes some scritps at "/usr/local/bin" to facilitate common opperations:

| Script               | Container  | Description                                                                                                   |
|:---------------------|:----------:|:--------------------------------------------------------------------------------------------------------------|
| ojs-run-scheduled    | ojs        | Runs "php tools/runScheduledTasks.php". Called by cron every hour.                                            |
| ojs-cli-install      | ojs        | Uses curl to call the ojs install using pre-defined variables (beta).                                         |
| ojs-pre-start        | ojs        | Enforces some config variables and generates a self-signed cert based on ServerName.                          |
| ojs-upgrade          | ojs        | Runs "php tools/upgrade.php upgrade". (beta: issue when config.inc.php is a volume)                           |
| ojs-variable         | ojs        | Replaces the variable value in config.inc.php (ie: ojs-variable variable newValue)                            |
| ojs-migrate          | ojs        | Takes a dump.sql, public and private files from "migration" folder and builds and builds a docker site (beta) |

Most of those scripts are still beta, you be careful when you use them.

You can call the scripts outside the container as follows:

   ```bash
   $ docker exec -it ojs_app_journalname /usr/local/bin/ojs-variable session_check_ip Off
   ```

## Upgrading OJS

Thanks to docker, the ugrade process is easy and straightforward.

1. **Stop the stack** with the old OJS version (for instance "pkpofficial/ojs:2_4_8-5").
   ```bash
   docker compose stop
   ```
2. **Set the new version** in docker-compose.yml.

     Replace the old version: ```image: pkpofficial/ojs:2_4_5-2```

     with the new one:        ```image: pkpofficial/ojs:3_2_1-4```

3. **Start the container**. As you changed the version, docker-compose will automatically pull a new image with an updated version of the OJS code that will replace the old one (remember that containers are not persistent).
   ```bash
   $ docker compose up
   ```
4. **Run the upgrade script** to upgrade the OJS database and files. Easiest was is connecting to your OJS container with [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec/) and run the `ojs-upgrade` built in script with this single line:
   ```bash
   $ docker exec -it ojs_app_journalname /usr/local/bin/ojs-upgrade
   ```
   | **TIP:** Discover your container name? |
   |:---------------------------------------|
   | You can see the name of all your containers with `docker ps -a`. The ones related with OJS will be something like `ojs_app_journalname`. |
   | Use grep to filter as follows: `$ docker ps -a | grep ojs_app` |

Before the upgrade you will like to [diff](https://linux.die.net/man/1/diff) your `config.inc.php` with the version of the new OJS version to learn about new configuration variables. Be specially carefully with the charsets.

   | **WARNING:** May I upgrade directly to the last OJS stable version?                                             |
   |:----------------------------------------------------------------------------------------------------------------|
   | It depends on your initial version. The recommended upgrade route is:<br/> **2.x > 2.4.8-5 > 3.1.2-4 > 3.2.1-4 > 3.3.x-x** |


## Apache2

As said, right now the only avaliable stack is Apache2, so configuration files
and volumes are thought you will work over an apache.

If you want to know the fastest method to set your own config jump to the next
section **["Easy way to change config stuff"](#easy-way-to-change-config-stuff)**.

For those who like to understand what happens behind the courtains, you need to
know that during the image building we ask the Dockerfile to copy the content of
`./templates/webServers/apache/phpVersion/root` folder in the root
of your container.

It means, if you like to add your specific php settings you can do it
creating a PHP configuration in `./templates/webServers/php{5,7,73}/conf.d/0-ojs.ini`
and build you own image locally.

There are some optimized variables already, you can check them within each
version directory, e.g. `/root/etc/apache2/conf.d/ojs.conf` with your virtualhost
or `/etc/supervisor.conf`... or, now you know what Dockerfile will do, you
can add your own.

**Again:** Note that template's /root folder is copied into the Docker image
**at build time** so, if you change it, you must rebuild the image for changes
to take effect.

### Easy way to change config stuff

If you don't want to keep you own local images, you can use the ones we
build in dockerHub and map your config files in your `docker-compose.yml`.

So if you like to change something in (for instance) your php settings,
you only need to created create a `./volumes/config/php.custom.ini`
outside your container and uncomment the volume in your `docker-compose.yml`.

Check the volumes section for a list of folders and files we think could
be useful to overwrite or extend to fit your needs.

### Restful URLs (aka. clean URLs)

By default the restful_url are enabled and Apache is already configured,
so there is no need to use index.php over url.

### SSL

By default at the start of Apache one script will check if the SSL certificate
is valid and its CN matches your SERVERNAME, if don't it will generate a new one.
The certificate can be overwritten using a volume mount (see `docker-compose.yml` file).

_**Note:** This feature is under reveiw and could change in future._

### SSL handled by an external service

If you have an external service in front handling SSL connections (often referred as
*SSL offloading* or *SSL termination*) you need to add a new line containing
`PassENV HTTPS` in `ojs.conf`, inside the main `<VirtualHost *:80>` section.

# How could I help?

## Update the compose configurations and Dockerfiles

If you want to join the docker-ojs (aka. docker4ojs) team you will like to
contribute with new Dockerfiles or docker-composes. To do this, you will need
to clone this project and send us a PR with your contributions.

Before we follow, let us clarify something:
Versions of this project fit with OJS tag versions, BUT...
if you take a look to PKP's repositories, you will notice the tag naming
syntax changed at version 3.1.2-0. We moved from "ojs-3_1_2-0" to "3_1_2-0".

So, to avoid keeping in mind at what momment we changed the syntax, in this
project we only use the modern syntax, so you cant/must refer every version
with the same notation.

Said this, let's talk about how we generate all this versions:

The files for the different OJS versions and software stacks are generated based
on a number of template files, see directory `templates`.

You can re-generate all `Dockerfile`, `docker-compose(-local).yml`, configuration
files, etc. calling the `build.sh` script.

```bash
# generate a specific version
$ ./build.sh 3_1_2-4

# generate _all_ versions
$ ./build.sh
```

## Add a new stack to the list

Let's say we like us to distribute a "nginx" stack. We have our hands full but
you know docker and nginx so... ¿how could you contribute?

1. Be sure the version number is in the versions.list file (ie: 3_2_1-0).
2. Create the required files and folders in the templates folder.

   Basically you will need:

   - A Dockerfile template for each php version you publish (ie: dockerfile-alpine-nginx-php73.template)
   - A generic `docker-compose.yml` template (templates/dockerComposes/docker-compose-nginx.template)
   - A folder with all the specific configurations (templates/webServers/nginx/php73/root)
   - Extend exclude.list with the stuff you want to be removed.

   This is the hard work. Take apache as a reference and contact us if you need indications.

3. Edit build.sh to add your version to the proper variables.

   For the nginx exemple (over alpine) it should be enough extending the webServers array:
   ```
   webServers=(  'apache' 'nginx' )
   ```
   Modify the script if you need it to be smarter.

4. Run build script to generate all versions again:

   ```bash
   $ ./build.sh
   ```

5. Test your work running `docker compose` with local dockerfiles.

6. Commit a PR with your new build.sh and templates (ignore the versions folder).

## Troubleshooting 

#### **I have trouble with Mac**
In general with docker, there are some known issues with the new Mac’s ARM architecture : https://stackoverflow.com/questions/73294020/docker-couldnt-create-the-mpm-accept-mutex . Alternative solution (other than hardcoding mutex settings) might be to build docker image also for arm64 platform (https://github.com/bitnami/containers/issues/4679). Some work was started in this line in gitLab building pipelines with promising preliminary results.

#### **I have trouble with Windows**
Instructions to run are for linux (as fas as linux is the natural platform for docker and servers) but is also possible to run it in windows. The wget instructions use variables defined in the env-file but this is not compatible with windows powershell, so would be nice to find an alternative that works all platforms. As a temporary solution we add clear instructions for windows users, that need to modify the inliner to get the right version of the config.TEMPLATE file.

#### **May I get an image based on nginx?**
A nice addition for docker images would be offer nginx image to replace the existing apache2.

#### **How could I deal with plugins?**
One thing you always will need to deal with is plugins. This is now possible but could be improved with a few ideas that appear during the sprint as: 
- Use volumes managed with git
- Create new ojs-plugins script helper that backups and download the essential release plugins for your version. 

#### **Is there any roadmap?**
The project is build based on the needs of the participants. If you like to join, contact marc.bria@uab.cat.
There is no formal roadmap, but we like to implement all the suggestion we made in the [Containers for PKP](https://docs.google.com/document/d/1AoGn1K4ep4vf7ylIS7wU2ybCLHdJNpkDRND7OhfRG-I/edit#heading=h.tpkz1jmp2yzm) document.
Priorities right now are (by order):
1. Create new images based on "[Containers for PKP](https://docs.google.com/document/d/1AoGn1K4ep4vf7ylIS7wU2ybCLHdJNpkDRND7OhfRG-I/edit#heading=h.tpkz1jmp2yzm)" proposal.
2. Fixing Mac image issues.
3. Automatize docker images building and pushing to different repositories.

#### **When I try to install I got an error...**
> Errors occurred during installation A database error has occurred: SQLSTATE[HY000] [2002] No such file or directory (SQL: create table announcement_types (type_id bigint not null auto_increment primary key, assoc_type smallint not null, assoc_id bigint not null) default character set utf8 collate 'utf8_general_ci')

When you run the docker-compose.yml, you will be creating different containers with different names.
In this project, the database container will be named as "db" so you can refer it in the "app" container to reach the DB.
So, nevermind if you use the web installer, or you set it manually in the config.inc.php, or you ask the Dockerimage to do it in your behalf... in all cases, you need to be sure you to set "db" when you are asked about the hostname of the database.

## License

GPL3 © [PKP](https://github.com/pkp)
