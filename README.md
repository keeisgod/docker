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
Once the host addressis occupied by a container, it cannot be used again for other containers, in order to terminate the running of a container, you need to stop/remove it
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


