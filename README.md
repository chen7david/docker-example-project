# docker-example-project

##### docker-compose - nginx app

```yaml
version: '3.2'

services:
    nginx-file-server:
        image: nginx:latest
        restart: always
        container_name: file-server
        volumes: 
            - ./public:/usr/share/nginx/html
        ports:
            - 9000:80

```
##### Dockerfile - node app

```yaml
# build stage
FROM node:lts-alpine as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# production stage
FROM nginx:stable-alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY prod_nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

##### Docker - .dockerignore

```dockerignore
node_modules
npm-debug.log
```

##### Dockerfile - Vue app

```yaml
# build stage
FROM node:lts-alpine as build-stage
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# production stage
FROM nginx:stable-alpine as production-stage
COPY --from=build-stage /app/dist /usr/share/nginx/html
COPY prod_nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

##### conf - vue prod_nginx.conf
```conf
user                    nginx;
worker_processes        1;
error_log               /var/log/nginx/error.log warn;
pid                     /var/run/nginx.pid;
events {
    worker_connections  1024;
}

http {
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    log_format          main '$remote_addr - $remote_user [$time_local] "$request" '
                             '$status $body_bytes_sent "$http_referer"'
                             '"$http_user_agent" "$http_x_forwarded_for"';
    access_log          /var/log/nginx/access.log main;
    sendfile            on;
    keepalive_timeout   65;
    server {
        listen          80;
        server_name     _ default_server;
        index           index.html;
        location / {
            root        /usr/share/nginx/html;
            index       index.html;
            try_files   $uri $uri/ /index.html;
        }
    }
}
```

##### docker-compose example

```yaml
version: '3.2'

services:
    file-server:
        image: nginx:latest
        restart: always
        container_name: file-server
        volumes: 
            - ./server/public:/usr/share/nginx/html
        ports:
            - 9000:80
    reverse-proxy:
        image: nginx:latest
        container_name: reverse-proxy
        volumes: 
            - ./conf.d:/etc/nginx/conf.d
        ports: 
            - 7000:80
    meta-client:
        build: ./client
        container_name: meta-client
        ports:
            - 8080:80
    meta-server:
        build: ./server
        restart: always
        container_name: meta-server
        working_dir: /usr/src/app
        volumes:
            - ./server/public:/usr/src/app/public
        ports:
            - "8000:8000"
```