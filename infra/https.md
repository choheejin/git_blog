# https 달기 위한 여정

1. ssl 인증서를 발급 받는다.
   1.  cerbot 설치

       ```bash
       sudo apt update
       sudo apt install certbot
       ```

       * cerbot 받는 과정에서 만난 에러
         1. https://dahliachoi.tistory.com/64
         2. apache 실행중이었음
   2.  인증서 발급 요청

       ```bash
       sudo certbot certonly --standalone -d <도메인 이름>
       ```
   3.  발급 확인

       ```bash
       cat /etc/letsencrypt/live/i12e103.p.ssafy.io/fullchain.pem
       ```
   4.  권한 설정

       ```bash
       sudo chmod 644 /etc/letsencrypt/archive/i12e103.p.ssafy.io/cert1.pem
       sudo chmod 644 /etc/letsencrypt/archive/i12e103.p.ssafy.io/chain1.pem
       sudo chmod 644 /etc/letsencrypt/archive/i12e103.p.ssafy.io/fullchain1.pem
       sudo chmod 644 /etc/letsencrypt/archive/i12e103.p.ssafy.io/privkey1.pem
       ```
2.  nginx 설정

    `work_dir\conf.d\default.conf` nginx 서버 설정 파일을 생성하여 작성

    *   work\_dir\conf.d\default.conf

        ```bash
        server {
            listen 80;
            server_name i12e103.p.ssafy.io;

            location /.well-known/acme-challenge/ {
                alias /var/www/certbot/.well-known/acme-challenge/;
                allow all;
            }

            location / {
                return 301 https://$host$request_uri;
            }
        }

        server {
            listen 443 ssl;
            server_name i12e103.p.ssafy.io;

            ssl_certificate /etc/letsencrypt/live/i12e103.p.ssafy.io/fullchain.pem;
            ssl_certificate_key /etc/letsencrypt/live/i12e103.p.ssafy.io/privkey.pem;

            location / {
                root /usr/share/nginx/html;
                index index.html;
                try_files $uri $uri/ /index.html;
            }

            location /.well-known/acme-challenge/ {
                root /var/www/certbot;
                allow all;
            }

            location /api/ {
                proxy_pass http://i12e103.p.ssafy.io:8076;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }

        ```
3.  docker-compose.yaml 작성하여 컨테이너 설정

    ```docker
    services:
      frontend:
        build:
          context: ./frontend
          dockerfile: Dockerfile
        container_name: frontend
        restart: always
        ports:
          - 80:80
          - 443:443
        volumes:
          - ./work_dir/conf.d/default.conf:/etc/nginx/conf.d/default.conf
          - ./work_dir/certbot:/var/www/certbot
          - /etc/letsencrypt:/etc/letsencrypt:ro
        healthcheck:
          test: ["CMD", "nginx", "-t"]
          interval: 10s
          timeout: 10s
          retries: 3
          
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

    * 만났던 문제들
      1. nginx를 이용한 vue 배포 과정에서 만난 403 forbidden 에러
      2. work\_dir\conf.d\default.conf 설정 과정
         1. 뒤늦게 vue container 정상적이지 못한 실행으로 인해 nginx.conf 로 설정이 안되는 것을 확인…
         2. 그래서 돌고 돌아 default.conf로 설정하고 마운트하는 것으로 해결하였다.
