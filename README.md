# Golang's tutorial *Accessing a relational database* using docker

This project recreates the official Golang's tutorial [*Accessing a relational database*](https://go.dev/doc/tutorial/database-access) using docker, so there is no need to install either Go or MySQL.

## Downloading and installing docker

The goal of this tutorial is to replicate the Golang's tutorial but using [*docker*](https://www.docker.com/), therefore you must first download and install it.

## Set up the files

You can clone this repo or manually download all its files to your computer.

```shell
git clone https://github.com/carlos-medina/database-access-with-docker.git
```

## Set up a database

In the *.env* file, we have the variable *MYSQL_ROOT_PASSWORD*. Its value is required to for both our application and our client to access the database.

```env
MYSQL_ROOT_PASSWORD="admin"
```

In the *compose.yaml* file, regarding our database, we have two important information: *services:db* and *networks:mysql-net*:

```yaml
services:
  db:
    image: mysql:8.0
    container_name: some-mysql
    env_file: .env
    ports:
      - 3306:3306
    networks:
      - mysql-net

networks:
  mysql-net:
    driver: bridge
    name: mysql-net
```

In *db*, we assign which mysql image from [dockerhub](https://hub.docker.com/) will be used, the container name, the path to the .env file so it can get the variable **MYSQL_ROOT_PASSWORD**, which port is going to be exposed and which network the container is going to be connected to. It is important to not use the default docker's *bridge* network because [docker's built-in DNS *does not apply* to this network](https://github.com/docker-library/mysql/issues/644).

In *mysql-net* we assing which driver will be used and the name of the network, which is the same used by our database.

Finally, we can head to a terminal and run our database:

```shell
docker compose up db
```

In a second terminal, we will create a container for the MySQL client:

```shell
docker run -it --network mysql-net --rm --name mysql-client mysql:8.0 mysql -hsome-mysql -uroot -p
```

In the container's terminal, we type the password atributed to the value *MYSQL_ROOT_PASSWORD* in our *.env* file, which is *admin*.

In a third terminal, we will copy the file *create-table.sql* to our database container:

```shell
docker cp ./create-tables.sql mysql-client:/create-tables.sql
```

In mysql-client's container terminal, we execute the following commands to 1. create the database 2. use it 3. execute the SQL commands from the copyied file and 4. check if the table was successfully created with the given data:

```sql
create database recordings;
use recordings;
source /create-tables.sql;
select * from album;
```

## Run the code

In the *.env* file, we also have the variables *DBUSER* and *DBPASS*. Their value is required to for our application to access the database.

```env
DBUSER="root"
DBPASS="admin"
```

In the *compose.yaml* file, regarding our application, besides *networks* that was already covered, we also have *services:app*:

```yaml
services:
  app:
    image: golang:1.21.0-alpine3.18
    container_name: go-app
    working_dir: /usr/src/app
    volumes:
      - .:/usr/src/app
    command: go run ./main.go
    env_file: .env
    networks:
      - mysql-net
```

In *app*, we assign which go image from [dockerhub](https://hub.docker.com/) will be used, the container name, from which directory the command will be executed, which files will be copied from the host to the container, which command will be executed, the path to the .env file and which network the container is going to be connected to.

Before running the app, there is only one change that you might do the the file *main.go*: On line 27, we have the struct key *Addr* and its value. If your *some-mysql* container's IP Address is different than *172.19.0.2:3306*, you have to change it to its correct value. In order for you to find its value, you can inspect it using the command *docker inspect*:

```shell
docker inspect some-mysql | grep IPAddress
```

If your container's IP Adress is, for instance, *172.28.0.2*, change the line 27 to

```go
Addr: "172.19.0.2:3306",
```

from

```go
Addr: "172.28.0.2:3306",
```

After the adjustments, run the command below in a terminal:

```shell
docker compose up app
```

You should see the following result:

```shell
[+] Running 1/0
 ✔ Container go-app  Created                                                                                                                                                                     0.1s 
Attaching to go-app
go-app  | go: downloading github.com/go-sql-driver/mysql v1.7.1
go-app  | Connected!
go-app  | Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
go-app  | Album found: {2 Giant Steps John Coltrane 63.99}
go-app  | ID of added album: 5
go-app exited with code 0
```