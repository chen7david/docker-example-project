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
##### Docker - node app

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