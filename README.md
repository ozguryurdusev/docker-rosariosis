Docker RosarioSIS
=================

A Dockerfile that installs the latest [RosarioSIS](https://www.rosariosis.org/). This file pulls from the default branch, but can be easily modified to pull from any other available branch or tagged release.

## Installation

Minimum requirements: [Docker](https://www.docker.com/) & Git working.

You can pull the image from [DockerHub](https://hub.docker.com/r/rosariosis/rosariosis) or:

```bash
git clone https://gitlab.com/francoisjacquet/docker-rosariosis.git
cd docker-rosariosis
docker build -t rosariosis .
```

## Usage

RosarioSIS uses a PostgreSQL database:
```bash
docker run -d \
	--name rosariosisdb \
	-e "POSTGRES_USER=rosario" \
	-e "POSTGRES_PASSWORD=rosariopwd" \
	-e "POSTGRES_DB=rosariosis" \
	-v ./plan/db:/var/lib/postgresql/data \
	postgres
```
This command will
1. [run](https://docs.docker.com/engine/reference/commandline/run/) the latest [postgres](https://hub.docker.com/_/postgres/) image
2. name it "rosariosisdb"
3. set database name "rosariosis", user "rosario" and password "rosariopwd"
4. a [volume](https://docs.docker.com/storage/volumes/) will persist data on your host inside `./plan/db`

Run RosarioSIS (DockerHub image) and link the PostgreSQL container:
```bash
docker run -d \
	--name rosariosis \
	-e "ROSARIOSIS_ADMIN_EMAIL=admin@example.com" \
	-e "PGHOST=rosariosisdb" \
	--link rosariosisdb:rosariosisdb \
	-p 80:80 \
	-v ./plan/rosariosis:/var/www/html \
	rosariosis/rosariosis:master
```
This command will
1. [run](https://docs.docker.com/engine/reference/commandline/run/) the latest [rosariosis/rosariosis:master](https://hub.docker.com/r/rosariosis/rosariosis) image
2. name it "rosariosis"
3. set the notification email to "admin@example.com" (see available environment variables below)
4. link the "rosariosisdb" container
5. expose port 80 of container to port 80 on host (`host:container`)
5. a [volume](https://docs.docker.com/storage/volumes/) will persist data on your host inside `./plan/rosariosis`

Port 80 will be exposed, so you can visit http://localhost/InstallDatabase.php to get started. Default username and password: `admin`.

Note: a [`docker-compose.yml`](docker-compose.yml) file is available.

Note 2: you may have to add `sudo` before the `docker` command.

Note 3: since image for RosarioSIS version 10.9, wkhtmltopdf is installed in another location, please update your `config.inc.php` file:
```php
$wkhtmltopdfPath = '/usr/local/bin/wkhtmltopdf';
```

## Environment Variables

The RosarioSIS image uses several environment variables which are easy to miss. While none of the variables are required, they may significantly aid you in using the image.

### DBTYPE

Database type: postgresql or mysql (defaults to postgresql).

### PGHOST

Host of the database.

### PGUSER

This optional environment variable is used in conjunction with PGPASSWORD to set a user and its password for the database.

### PGPASSWORD

This optional environment variable is used in conjunction with PGUSER to set a user and its password for the database.

### PGDATABASE

This optional environment variable can be used to define a different name for the database.

### PGPORT

This optional environment variable can be used to define a different port for the database.

### ROSARIOSIS_YEAR

This optional environment variable can be used to define the default school year in RosarioSIS settings.

Only change after Rollover.

### ROSARIOSIS_ADMIN_EMAIL

This optional environment variable can be used to define an email address where to send error and new administrator notifications.

### ROSARIOSIS_LANG

This optional environment variable is for RosarioSIS to show another language.

Values are `fr_FR` for French and `es_ES` for Spanish.

You must also generate the `fr_FR.utf8` (for example) locale. To do so run these commands:
```bash
sudo docker exec -it rosariosis bash
dpkg-reconfigure locales
```

### ROSARIOSIS_VERSION

This optional environment variable is used to set the required version of RosarioSIS.

## SMTP

RosarioSIS will attempt to send mail via the host's port 25. In order for this to work you must set the hostname of the rosariosis container to that of `host` (or some other hostname that can appear on a legal `FROM` line) and configure the host to accept SMTP from the container. For postfix this means adding the container IP addresses to `/etc/postfix/main.cf` as in:

```
mynetworks = 192.168.0.0/16 172.16.0.0/12 10.0.0.0/8 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
```

Note: alternatively, you can use the [Email SMTP](https://www.rosariosis.org/plugins/email-smtp/) plugin for RosarioSIS.


## Additional configuration

[Quick Setup Guide](https://www.rosariosis.org/quick-setup-guide/)

[Secure RosarioSIS](https://gitlab.com/francoisjacquet/rosariosis/-/wikis/Secure-RosarioSIS)
