# docker

Quick setup for docker containerisation

#build a dockerfile to build image
touch Dockerfile

#edit the text file just created
# syntax=docker/dockerfile:1
'''
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
'''
# go to working directory
cd /path/to/getting-started=app
# build the image
docker build -t getting-started .
# start running the container
docker run -dp 127.0.0.1:3000:3000 getting-started

# access the app
 https://localhost:3000
# list running apps
docker ps

