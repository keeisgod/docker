# Docker Compose
Define and share multi-container application
Created using YAML file

Example - create a multi-container app
1. Create a yaml file ```touch compose.yaml```

Edit the file with the following code

```
services:
  app:
    image: node:18-alpine
    command: sh -c "yarn install && yarn run dev"
    ports:
      -127.0.0.1:3000:3000
    working_dir: /app
    volumes:
      -./:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
  mysql:
    image:mysql:8.0
    volumes:
      - todo-mysql-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: todos
volumes:
  todo-mysql-data:
```

3. After creating the yaml file, use ```compose``` command to create/run the application stack - make sure no other copies of containers are running.

   ``` docker compose up -d```

then check the log ``` docker compose logs -f```

4. you can traack the usage of an image and the layers by using ```image history``` command
   ``` docker image history getting-started``` or
   ``` docker image history --no-trunc getting-started``` for full output
   whereas the getting-started is the image name

   

   



