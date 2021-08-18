# Docker compose instructions for WordPress in Raspberry Pi
Compose instructions for Wordpress docker image for Raspberri Pi


## What is WordPress?

> WordPress is one of the most versatile open source content management systems on the market. WordPress is built for high performance and is scalable to many servers, has easy integration via REST, JSON, SOAP and other formats, and features a whopping 15,000 plugins to extend and customize the application for just about any type of website.

[https://www.wordpress.org/](https://www.wordpress.org/)

## TL;DR

```console
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-wordpress/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

You can find the default credentials and available configuration options in the [Environment Variables](#environment-variables) section.

# How to deploy WordPress in Docker?

Deploying Bitnami applications as Helm Charts is the easiest way to get started with our applications on Kubernetes. Read more about the installation in the [Bitnami WordPress Chart GitHub repository](https://github.com/bitnami/charts/tree/master/bitnami/wordpress).

Bitnami containers can be used with [Kubeapps](https://kubeapps.com/) for deployment and management of Helm Charts in clusters.

## Why use a non-root container?

Non-root container images add an extra layer of security and are generally recommended for production environments. However, because they run as a non-root user, privileged tasks are typically off-limits. Learn more about non-root containers [in our docs](https://docs.bitnami.com/tutorials/work-with-non-root-containers/).

## Supported tags and respective `Dockerfile` links

Learn more about the Bitnami tagging policy and the difference between rolling tags and immutable tags [in our documentation page](https://docs.bitnami.com/tutorials/understand-rolling-tags-containers/).


- [`5`, `5-debian-10`, `5.8.0`, `5.8.0-debian-10-r9`, `latest` (5/debian-10/Dockerfile)](https://github.com/bitnami/bitnami-docker-wordpress/blob/5.8.0-debian-10-r9/5/debian-10/Dockerfile)

Subscribe to project updates by watching the [bitnami/wordpress GitHub repo](https://github.com/bitnami/bitnami-docker-wordpress).

## Images used in this compose

The recommended way to get the Bitnami WordPress Docker Image is to pull the prebuilt image from the [Docker Hub Registry](https://hub.docker.com/r/bitnami/wordpress).

```console
$ docker pull bitnami/wordpress:latest
```

To use a specific version, you can pull a versioned tag. You can view the [list of available versions](https://hub.docker.com/r/bitnami/wordpress/tags/) in the Docker Hub Registry.

```console
$ docker pull bitnami/wordpress:[TAG]
```

If you wish, you can also build the image yourself.

```console
$ docker build -t bitnami/wordpress:latest 'https://github.com/bitnami/bitnami-docker-wordpress.git#master:5/debian-10'
```

## How to use this image

WordPress requires access to a MySQL or MariaDB database to store information. We'll use the [MariaDB](https://hub.docker.com/_/mariadb) for the database requirements.

### Run the application using Docker Compose

The main folder of this repository contains a functional [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress/blob/master/docker-compose.yml) file. Run the application using it as shown below:

```console
$ curl -sSL https://raw.githubusercontent.com/bitnami/bitnami-docker-wordpress/master/docker-compose.yml > docker-compose.yml
$ docker-compose up -d
```

Access your application at *http://your-ip:8080/*

## Persisting your application

If you remove the container all your data will be lost, and the next time you run the image the database will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed.

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

## Configuration

### Environment variables

When you start the WordPress image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the `docker run` command line. If you want to add a new environment variable:

- For docker-compose add the variable name and value under the application section in the [`docker-compose.yml`](https://github.com/bitnami/bitnami-docker-wordpress/blob/master/docker-compose.yml) file present in this repository:

    ```yaml
    wordpress:
      ...
      environment:
        - WORDPRESS_PASSWORD=my_password
      ...
    ```

- For manual execution add a `--env` option with each variable and value:

    ```console
    $ docker run -d --name wordpress -p 80:8080 -p 443:8443 \
      --env WORDPRESS_PASSWORD=my_password \
      --network wordpress-tier \
      --volume /path/to/wordpress-persistence:/bitnami \
      bitnami/wordpress:latest
    ```

Available environment variables:

