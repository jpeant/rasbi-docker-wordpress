# Docker compose instructions for WordPress in Raspberry Pi
Non-official guide


## What is WordPress?

> WordPress is one of the most versatile open source content management systems on the market. WordPress is built for high performance and is scalable to many servers, has easy integration via REST, JSON, SOAP and other formats, and features a whopping 15,000 plugins to extend and customize the application for just about any type of website.

[https://www.wordpress.org/](https://www.wordpress.org/)

## TL;DR for test purposes only

```console
$ curl -sSL https://raw.githubusercontent.com/jpeant/rasbi-docker-wordpress/main/wordpress-stack.yml
$ docker-compose -f wordpress-stack.yml up
```

Access your application at *http://your-ip:8080/*

## Images used in this compose

The recommended way to get the WordPress Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/_/wordpress).

```console
$ docker pull wordpress:latest
```

For the database we are using MariaDB and it is equivalent for MySql. You can view the [list of available versions](https://hub.docker.com/_/mariadb) in the Docker Hub Registry.

```console
$ docker pull mariadb:latest
```

## Configuring the wordpress-stack.yml file

It is very important that you change the username and password:

```
environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: your-name-here!!
      WORDPRESS_DB_PASSWORD: your-password-here!
```
and
```
environment:
      MYSQL_DATABASE: wp-db
      MYSQL_USER: your-name-here!!
      MYSQL_PASSWORD: your-password-here!!
```

Username has to be the same in WORDPRESS_DB_USER and MYSQL_USER.
Password has to be the same in WORDPRESS_DB_PASSWORD and MYSQL_PASSWORD.

Ports can be modified at:

```
ports:
      - 8080:80
```

In this example port 8080 is for outside, that can be changed to 80 if so needed. If you are going to get a SSL certificate for the server port 443 should be opened.



## Volumes to store data

The WordPress application state will persist as long as volumes are not removed.

Compose file creates two volumes, one for WordPress instance and one for MariaDB instance. These are named as wordpress and db.

To avoid inadvertent removal of volumes, you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.


