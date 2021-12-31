## referrence site
~~~
https://ayoubb.com/technology/dockerize-apache-mysql-php-stack/
~~~

# docker-compose-website-build-with-database
##Apache MySQL PHP PHPMyAdmin Containerized with Docker


#PHP / Apache / MySQL stack has a dominant market share on internet content management systems and web applications, and with so many developers using these technologies, there is a great deal of interest in modernizing the way they use them from local development through to production. Today we’ll look at several ways to containerize and connect PHP, Apache, and MySQL together while demonstrating some tips, tricks, and best practices that will help you build and deploy your PHP applications in a modern way!

~~~
apache.conf
~~~
~~~
ServerName localhost

LoadModule deflate_module /usr/local/apache2/modules/mod_deflate.so
LoadModule proxy_module /usr/local/apache2/modules/mod_proxy.so
LoadModule proxy_fcgi_module /usr/local/apache2/modules/mod_proxy_fcgi.so

<VirtualHost *:80>
    # Proxy .php requests to port 9000 of the php-fpm container
    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://php:9000/var/www/html/$1

    DocumentRoot /var/www/html/
    <Directory /var/www/html/>
        DirectoryIndex index.php
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    
    # Send apache logs to stdout and stderr
    CustomLog /proc/self/fd/1 common
    ErrorLog /proc/self/fd/2
</VirtualHost>
~~~
~~~
apache > Dockerfile
~~~
~~~
FROM httpd:2.4.33-alpine

RUN apk update; \
    apk upgrade;

# Copy apache vhost file to proxy php requests to php-fpm container
COPY apache.conf /usr/local/apache2/conf/apache.conf
RUN echo "Include /usr/local/apache2/conf/apache.conf" \
    >> /usr/local/apache2/conf/httpd.conf
~~~
~~~

php > Dockerfile
~~~

~~~
FROM php:7.2.7-fpm-alpine3.7

RUN apk update; \
    apk upgrade;

RUN docker-php-ext-install mysqli
~~~

~~~
.env
~~~
~~~
MYSQL_VERSION=8
APACHE_VERSION=2.4.32

DB_ROOT_PASSWORD=root
DB_NAME=db
DB_USERNAME=user
DB_PASSWORD=root

PROJECT_ROOT=./public_html
~~~

~~~
docker-compose.yml
~~~

~~~
version: "3.2"
services:
  php:
    build: 
      context: './php/'
      args:
       PHP_VERSION: ${PHP_VERSION}
    networks:
      - backend
    volumes:
      - ${PROJECT_ROOT}/:/var/www/html/
    container_name: php
    links:
     - mysql
  apache:
    build:
      context: './apache/'
      args:
       APACHE_VERSION: ${APACHE_VERSION}
    depends_on:
      - php
      - mysql
    links:
      - mysql
    networks:
      - frontend
      - backend
    ports:
      - "8080:80"
    volumes:
      - ${PROJECT_ROOT}/:/var/www/html/
    container_name: apache
  mysql:
    image: mysql:${MYSQL_VERSION}
    command: --default-authentication-plugin=mysql_native_password 
    restart: always
    ports:
      - "3306:3306"
    volumes:
      - data:/var/lib/mysql
    networks:
      - backend
  # The default MySQL installation only creates the "root" administrative account
  # create new users using docker-compose exec
    environment:
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"
      MYSQL_DATABASE: "${DB_NAME}"
      MYSQL_USER: "${DB_USERNAME}"
      MYSQL_PASSWORD: "${DB_PASSWORD}"
    container_name: mysql
  phpmyadmin:
    depends_on:
      - mysql
    links:
      - mysql
    image: phpmyadmin/phpmyadmin
    restart: always
    ports:
      - '8081:80'
    environment:
      PMA_HOST: mysql
      MYSQL_USERNAME: "${DB_USERNAME}"
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"   
    networks:
      - backend
    volumes:
      - /sessions
    container_name: phpmyadmin
networks:
  frontend:
  backend:
volumes:
    data:
~~~

~~~
public_html > index.php
~~~
~~~
<h1>Hello World!!!</h1>
<h4>Attempting MySQL connection from php...</h4>
<?php
// I assume that you've created a new username 'user' with password 'user' for a demo purpose
$host = 'mysql';
$user = 'root';
$pass = 'root';
$conn = new mysqli($host, $user, $pass);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
} else {
  echo "Connected to MySQL successfully!";
}
?>
~~~


## docker-compose -f docker-compose.yaml up -d

##Access PHPMyAdmin

## You can now navigate to localhost:8081 to open PHPMyAdmin.

## The default MySQL installation only creates the “root” administrative account (user: root, pass: root)



