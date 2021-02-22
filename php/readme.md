

#  Documentación de dockerización del lenguaje PHP

La dockerización del lenguaje PHP consiste en la creación de contenedores que almacenan el lenguaje haciendo uso de la herramienta **Docker Compose**, la cuál, simplifica el uso de **Docker**, a través de scripts escritos en formato **Yaml** dentro de ficheros llamados "docker-compose.yml", dichos ficheros crean servicios que permiten la conexión con otros contenedores, como ejemplo: dentro del presente proyecto: "databases" , "nginx".

## Dockerfile 📋

>Creando estructura de carpetas:
```bash
user@debian:~/Documents/Docker/php$ mkdir php-fpm && cd php-fpm
```

_La carpeta **php-fpm**  contiene el archivo Dockerfile_

>Creando archivo Dockerfile:

```bash
user@debian:~/Documents/Docker/php/php-fpm$ nano Dockerfile
```

Las siguientes instrucciones corresponden a la creación de la imagen del lenguaje PHP-FPM:
```
FROM php:7.2-fpm
RUN apt-get update && apt-get install -y \
        git \
        libzip-dev \
        zip \
        unzip \
        libpq-dev \
        firebird-dev
RUN docker-php-ext-configure zip --with-libzip
RUN docker-php-ext-install zip pdo_firebird pdo_pgsql pdo_mysql
```


### Archivo docker-compose.yml
>Crear archivo docker-compose.yml
```bash
user@debian:~/Documents/Docker/php$ nano docker-compose.yml
```
```Yaml
version: '3.7'

services:
 php-fpm:
  build:
   context: ./php-fpm
  restart: always
  ports:
   - ${PORT}:${PORT}
  volumes:
   - ../nginx/public_html:/var/www/html
  networks:
   - php-fpm
   - db
networks:
 php-fpm:
 db:
  external:
   name: databases_databases 
```
_La carpeta **nginx/public_html**  nos permite ver el mismo volumen que ve el servidor **NGINX**_

_Configuramos la red externa  **databases_databases** para poder comunicarnos con las bases de datos y nuestra propia red de nombre: **php-fpm** para que el servidor **NGINX** pueda usar la configuración de **PHP**._

_Dentro del archivo **.env**  podemos gestionar las variables de configuración._
>las variables de entorno se definen de la siguiente forma dentro del archivo .env
```bash
# VARIABLES PHP-FPM

PORT=9000
```
## Herramientas utilizadas 🛠️

* [Docker](http://www.docker.com/) - Tecnología de creación de contenedores


## Autores ✒️
* **Leonardo Seña Pimentel** 
* **Cristian Narvaez Useche** 
* **Jeaneil Bernal Sierra**

