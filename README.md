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

```environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: your-name-here!!
      WORDPRESS_DB_PASSWORD: your-password-here!
```
and
```environment:
      MYSQL_DATABASE: wp-db
      MYSQL_USER: your-name-here!!
      MYSQL_PASSWORD: your-password-here!!
```

For persistence you should mount a directory at the `/bitnami/wordpress` path. If the mounted directory is empty, it will be initialized on the first run. Additionally you should [mount a volume for persistence of the MariaDB data](https://github.com/bitnami/bitnami-docker-mariadb#persisting-your-database).

The above examples define the Docker volumes named `mariadb_data` and `wordpress_data`. The WordPress application state will persist as long as volumes are not removed.

To avoid inadvertent removal of volumes, you can [mount host directories as data volumes](https://docs.docker.com/engine/tutorials/dockervolumes/). Alternatively you can make use of volume plugins to host the volume data.

### Mount host directories as data volumes with Docker Compose

This requires a minor change to the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress/blob/master/docker-compose.yml) file present in this repository:

```diff
   mariadb:
     ...
     volumes:
-      - 'mariadb_data:/bitnami/mariadb'
+      - /path/to/mariadb-persistence:/bitnami/mariadb
   ...
   wordpress:
     ...
     volumes:
-      - 'wordpress_data:/bitnami/wordpress'
+      - /path/to/wordpress-persistence:/bitnami/wordpress
   ...
-volumes:
-  mariadb_data:
-    driver: local
-  wordpress_data:
-    driver: local
```

> NOTE: As this is a non-root container, the mounted files and directories must have the proper permissions for the UID `1001`.

### Mount host directories as data volumes using the Docker command line

#### Step 1: Create a network (if it does not exist)

```console
$ docker network create wordpress-network
```

#### Step 2. Create a MariaDB container with host volume

```console
$ docker run -d --name mariadb \
  --env ALLOW_EMPTY_PASSWORD=yes \
  --env MARIADB_USER=bn_wordpress \
  --env MARIADB_PASSWORD=bitnami \
  --env MARIADB_DATABASE=bitnami_wordpress \
  --network wordpress-network \
  --volume /path/to/mariadb-persistence:/bitnami/mariadb \
  bitnami/mariadb:latest
```

> NOTE: As this is a non-root container, the mounted files and directories must have the proper permissions for the UID `1001`.

#### Step 3. Create the WordPress container with host volumes

```console
$ docker run -d --name wordpress \
  -p 8080:8080 -p 8443:8443 \
  --env ALLOW_EMPTY_PASSWORD=yes \
  --env WORDPRESS_DATABASE_USER=bn_wordpress \
  --env WORDPRESS_DATABASE_PASSWORD=bitnami \
  --env WORDPRESS_DATABASE_NAME=bitnami_wordpress \
  --network wordpress-network \
  --volume /path/to/wordpress-persistence:/bitnami/wordpress \
  bitnami/wordpress:latest
```

> NOTE: As this is a non-root container, the mounted files and directories must have the proper permissions for the UID `1001`.

