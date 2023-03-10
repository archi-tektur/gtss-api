# General train, station and stop API

## PHP, Symfony and the database

This project is using PHP in version **8.1.\** and Symfony framework with version **^6.1**. In the future, I plan to
update this repository to match next PHP and Symfony versions. Also, the MariaDB 10.6 is used.

## Required tools

Before we start, let's make sure that our host device has required programs. We will use `docker` and `docker compose`
to manage containers.

To create this bootstrap I used the tools in version listed below. I cannot guarantee that this configuration will work
automatically with newer versions.

```shell
docker --version
> Docker version 20.10.17, build 100c701

docker compose version
> Docker Compose version v2.6.1
```
## Building environment for development

First things first you have to copy the development environment template. It' located in `.devcontainer`, I'd reccomend
to leave it there and create a symbolic link.

```shell
ln -s ./.devcontainer/docker-compose.override.yml .
```

Now we'll use `docker` to build our image locally, with local architecture:

```shell
docker compose build
```

It may take few seconds, when it's completed proceed with running the container:

```shell
docker-compose up --detach
```

Remember that you have installed the vendors in an image, however while creating container you've replaced built app
folder with empty one (repository has no `vendor` folder intentionally). So, we have to proceed once again with app
configuration:

```shell
docker-compose exec app bash -ce "
    composer install
    composer build
    chown -R $(id -u):$(id -g) .
  "
```

Now you're all set, you can visit the [localhost with port 38080](http://localhost:38080), you should
see the Symfony default application web page.

Also, a GET endpoint [/api/hello-world](http://localhost:38080/api/hello-world) was added to configure and show the functional
test environment. It will always return static data:
```json
{
  "message": "Hello, world"
}
```

If for some reason you'd like to enter the container, use the command below.

```shell
docker-compose exec app bash
```

## Removing local environment

You can remove local environment using the command below:

```shell
docker-compose down --remove-orphans
```

## Running service for production use

This section will be expanded soon.

## Assumptions

### Custom PHP image

In the main Dockerfile I used [caddy-php](https://github.com/archi-tektur/caddy-php-image): my own high-performance PHP
image that uses Caddy as a runner and php-fpm as a daemon.

```dockerfile
FROM ghcr.io/archi-tektur/caddy-php:1.0.3 AS app
```

You're free to change it to any image and configuration you'd like. You may read
[here](https://github.com/archi-tektur/caddy-php-image/README.md) what's bundled inside my image and create your own
with my approach as a guidelines.

### Application logs

Due to extensive configuration you will find all the logs stored in `./app/var/log` folder, including Caddy, php-fpm
access and error logs and xdebug outputs. Also files like `dev.log`, `prod.log` will be stored with all the messages
logged with PSR `LoggerInterface` and `monolog` package.

### Testing

Modern web applications must be well tested. To keep things simple I prepared ready to use suite for unit and functional
tests.

#### Unit
Use the command below to run unit tests. Initially this project contains
[one test](https://github.com/archi-tektur/symfony-bootstrap/app/tests/NullTest.php) with one assertion
(NullTest pattern).

```
docker compose exec app composer run-unit-tests
```

#### Functional

Use the command below to run functional tests:

```
docker compose exec app composer run-functional-tests
```

Functional test is testing here the hello-world endpoint.

## Troubleshooting

This section will be expanded as first problem will come.

## Inspiration
