# Docker
Quick setup for docker containerisation

build a dockerfile to build image
touch Dockerfile

edit the text file just created
syntax=docker/dockerfile:1

``` bash
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```


go to working directory
```cd /path/to/getting-started=app```

# Build an image for the container
build the image
```docker build -t getting-started .``` 

The build command will use the Dockerfile to create a new image

 ```.``` denotes finding the file in current working directory
# Run the app
To start running the container
``` docker run -dp 127.0.0.1:3000:3000 getting-started```

```-d``` denotes detach mode so the app runs in the background, mannual termination is required

``` -p``` denotes publishing and it creates a mapping between host and container.
Input format is ```HOST:CONTAINER```, whereas 127.0.0.1:3000 is the hosts address and 3000 is the containers port

Once the app is built, then can access the app via
 ``` https://localhost:3000```
 
To list running apps
```docker ps```

# Terminate the app
Once the host address is occupied by a container, it cannot be used again for other containers. 

In order to terminate the running of a container, you need to stop/remove it
``` docker stop <container-id>```

``` docker rm <container-id>```

or force it to stop and be removed 
``` docker rm -f <container-id>```

# Push the image to Docker Hub

Before pushing it to Docker hub, make sure an account is registered and a repo is created under the account.

Log in to Docker Hub
```docker login -u XXX -p XXXXXxxx```

Tag the image with the username
``` docker tag XXX/getting-started```

Push it to the repo
``` docker push XXX/getting-started```

# Mount a volumn to container
volumn is a sqlite database that container can access and write data to

To create a volume called 'todo-db'
```docker volume create todo-db```

Then start container with specifying the mount option
``` docker run -dp 127.0.0.1:3000:3000 --mount type=volume, src=todo-db,target=//etc/todos getting-started```

If checking the info of volume
``` docker volume inspect todo-db```

It will contain some info like below shown
``` bash
[{
 "CreatedAt": "2019-09-26T02:18:36Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local" }]
```

where ```Mountpoint``` specifies the exact location where the app data stores

# Use a bind mount for the app
In simple words, you can bind a directory on the host with a directory in the container, changes made inside the app (in the directory bound), can be seen from the host or the other way around

start off the app with bind mount
``` docker run --it --mount type=bind, src="/$(pwd)", target=/src ubuntu bash```

``` src``` specifies the current working directory

``` target ``` defines where the directory should appear inside the app (imagine container itself is another machine and has a different file system/directories to the host)

Once app is running, a bash terminal session will appear in the root directory in the container
```
root@ac1237fad8db:/# pwd
/
root@ac1237fad8db:/# ls
bin   dev  home  media  opt   root  sbin  srv  tmp  var
boot  etc  lib   mnt    proc  run   src   sys  usr
```

Press ```Ctrl``` + ```D``` to stop the session

# Run development container
Docker will pull the dependencies and tools via commands

```
docker run -dp 127.0.0.1:3000:3000
   -w //app --mount type=bind, src="/$(pwd)", target=/app
   node:18-alpine
   sh -c "yarn install && yarn run dev"
```

```-w //app ``` sets the working directory where the command will run from

``` sh -c "yarn install && yarn run dev"``` starts a shell and run ```yarn install``` command to instal packages, then ``` yarn run dev``` will start the development server


If wanting to check the scripts, it is stored in ```package.json``` in which ```dev``` script starts ```nodemon```


# Monitor the process in the log
Run ``` docker logs -f <container-id>```

Press```CTRL``` + ```C``` to turn off

# Container networking
Assuming there is one container running as database and another one as app

1. Create a network
```docker network create todo-app```

2. Start a MySQL container and attach it to the network
   ``` bash
   docker run -d
   --network todo-app --network-alias mysql
   -v todo-mysql-data:/var/lib/mysql
   -e MYSQL_ROOT_PASSWORD=secret
   -e MYSQL_DATABASE=todos
   mysql:8.0
   ```
``` -v todo-mysql-data:/var/lib/mysql``` it automatically creates a volumn and mounted to appointed path

``` -e``` denotes environment variables

3. To check if database is running,
   Use ``` docker exec -it <container-id> mysql -u root -p```
   Once the command is entered, password prompt will pop out and type in the password ```secret ```, as set in previous step
   
5. Once password is entered correctly, mysql shell will be active, use this command to list out all the existing databases
   
   ``` mysql> SHOW DATABASES ```
   
```
 | Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| todos              |
+--------------------+
```

Use ``` exit``` to return to host shell

6. Start a new container working for database 
``` docker run -it --network todo-app nicolaka/netshoot```

7. Find the IP address of Mysql database in the network
Once inside the app container, use ``` dig mysql ``` to find the IP address
OUtput:
```
; <<>> DiG 9.18.8 <<>> mysql
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 32162
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;mysql.				IN	A

;; ANSWER SECTION:
mysql.			600	IN	A	172.23.0.2

;; Query time: 0 msec
;; SERVER: 127.0.0.11#53(127.0.0.11)
;; WHEN: Tue Oct 01 23:47:24 UTC 2019
;; MSG SIZE  rcvd: 44
```

In the ``` ANSWER SECTION ```, you can find the hostname ```mysql```, and its associated IP address	```172.23.0.2```


8. Start off a new container for the app
   Run a new container and connect to todo-app network
   ```
   docker run -dp 127.0.0.1:3000:3000
   -w //app -v "/$(pwd):/app"
   --network todo-app
   -e MYSQL_HOST=mysql
   -e MYSQL_USER=root
   -e MYSQL_PASSWORD=secret
   -e MYSQL_DB=todos
   node:18-alpine
   sh -c "yarn install && yarn run dev"
   ```

9. Use logs to check connection
   ```docker logs -f <container-id>```
   
Output print:

```
nodemon src/index.js
[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching dir(s): *.*
[nodemon] starting `node src/index.js`
Connected to mysql db at host mysql
Listening on port 3000
```
10. add any changes in the browser and check changes are made effectively
    enter mysql shell
    ```docker exec -it <mysql-container-id> mysql -p todos```

Type in ```mysql> select * from todo_items;```
Output print:

```
+--------------------------------------+--------------------+-----------+
| id                                   | name               | completed |
+--------------------------------------+--------------------+-----------+
| c906ff08-60e6-44e6-8f49-ed56a0853e85 | Do amazing things! |         0 |
| 2912a79e-8486-4bc3-a4c5-460793a575ab | Be awesome!        |         0 |
+--------------------------------------+--------------------+-----------+
```







