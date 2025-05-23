# nginx를 이용한 vue 배포 과정에서 만난 403 forbidden 에러 해결 과정

내가 헤매던 과정을 설명하기 위해 엉망이었던 나의 docker-compose.yaml 파일과, Dockerfile 부터 적어보겠다.

## 오류 범벅 코드

1.  docker-compose.yaml

    ```docker
    version: "3.8"

    services:
      frontend:
        build:
          context: ./frontend
          dockerfile: Dockerfile
        container_name: frontend

      web:
        image: nginx:latest
        container_name: nginx
        restart: always
        ports:
          - 80:80
          - 443:443
        volumes:
          - ./work_dir/conf.d/default.conf:/etc/nginx/conf.d/default.conf  # conf 파일 마운트
          - ./work_dir/certbot:/var/www/certbot  # Certbot 인증 파일 경로 마운트
          - ./frontend/dist:/usr/share/nginx/html  # Vue 앱 배포 파일 마운트
          - /etc/letsencrypt:/etc/letsencrypt:ro
        networks:
          - my-sql-network
        healthcheck:
          test: ["CMD", "nginx", "-t"]
          interval: 10s
          timeout: 10s
          retries: 3
        depends_on:
          - frontend
          
      spring-boot:
        build:
          context: ./backend
          dockerfile: Dockerfile
        container_name: spring-boot
        restart: unless-stopped
        ports:
          - 8076:8076
        environment:
          - SPRING_PROFILES_ACTIVE=prod
          - JAVA_OPTS=-Xms512m -Xmx1024m
        networks:
          - redis-network
          - my-sql-network
        depends_on:
          mysql:
            condition: service_healthy

      mysql:
        image: mysql:8.0
        container_name: mysql
        restart: unless-stopped
        environment:
          MYSQL_ROOT_PASSWORD: "root"
          MYSQL_DATABASE: "E103_DB"
        ports:
          - 3307:3306
        volumes:
          - my-sql-volume:/var/lib/mysql
          - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
        networks:
          - my-sql-network
        healthcheck:
          test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
          interval: 10s
          retries: 5
          start_period: 10s

    volumes:
      my-sql-volume:

    networks:
      redis-network:
        driver: bridge
        external: true
      my-sql-network:
        driver: bridge
        external: true
    ```
2. frontend/Dockerfile
   1.  1차

       ```docker
       # 'AS build-stage'로 스테이지 이름을 지정
       FROM node:18 AS build-stage  
       WORKDIR /app
       COPY package.json ./
       COPY package-lock.json ./
       RUN npm install
       COPY . .
       RUN npm run build
       ```
   2.  2차

       ```docker
       # 'AS build-stage'로 스테이지 이름을 지정
       FROM node:18 AS build-stage  
       WORKDIR /app
       COPY package.json ./
       COPY package-lock.json ./
       RUN npm install
       COPY . .
       RUN npm run build

       # Serve stage
       FROM nginx:alpine
       # Copy dist to Nginx
       COPY --from=build-stage /app/dist /usr/share/nginx/html  
       ```

## 해결 과정

* 문제 상황
  1. vue를 빌드하고 빌드된 폴더를 nginx에서 사용하면 될 줄 알았다.
  2. 1번이 안되니 nginx 서버를 결과적으로 두 개 생성해버렸다.
* 문제 이유
  1. vue 컨테이너, nginx 컨테이너가 데이터를 주고 받기 위해선 공유 볼륨이 존재해야한다.
     1. 이미 공유 볼륨을 지정해줬었는데도 문제가 발생했었다. 아마 파일 접근 권한이 없었는가 싶기도 하다.
     2. 또한, 이 구조는 매우 비효율적인 구조라고 볼 수 있다.
  2. nginx 서버를 두개를 생성해버려 최종적으로 설정되는 컨테이너는 docker-compose에 있는 nginx 서버였다.
     1. 기존의 /app 디렉토리가 최종 이미지에 포함되지 않게 되었다.
