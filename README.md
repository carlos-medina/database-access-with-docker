# Golang's tutorial *Accessing a relational database* using docker

This project recreates the official Golang's tutorial [*Accessing a relational database*](https://go.dev/doc/tutorial/database-access) using docker, so there is no need to install either Go or MySQL.

## Downloading and installing docker

The goal of this tutorial is to replicate the Golang's tutorial but using [*docker*](https://www.docker.com/), therefore you must first download and install it.

## Set up a database

Before creating a [*MySQL database*](https://hub.docker.com/_/mysql), create a new network with docker. Docker's built-in DNS [*does not apply*](https://github.com/docker-library/mysql/issues/644) on the default **bridge** network.

```shell
docker network create mysql-net
```

Now you can run a 8.0 version of a MySQL container with the name *some-mysql*, *admin* as the root password and connecting it to the newly created network.

```shell
docker run --network mysql-net --rm --name some-mysql -e MYSQL_ROOT_PASSWORD=admin -d mysql:8.0
```