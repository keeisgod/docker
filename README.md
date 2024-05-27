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

```docker run -dp 127.0.0.1:3000:3000
   -w //app --mount type=bind, src="/$(pwd)", target=/app
   node:18-alpine
   sh -c "yarn install && yarn run dev"```

```-w //app ``` sets the working directory where the command will run from

``` sh -c "yarn install && yarn run dev"``` starts a shell and run ```yarn install``` command to instal packages, then ``` yarn run dev``` will start the development server

``` If wanting to check the scripts, it is stored in ```package.json``` in which ```dev``` script starts ```nodemon```
```
# Monitor the process in the log
Run ``` docker logs -f <container-id>```

Press```CTRL``` + ```C``` to turn off







