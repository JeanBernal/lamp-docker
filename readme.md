
# Dockerizando ambiente de trabajo

## Comenzando 🚀

Estas instrucciones le permitirán configurar un ambiente de trabajo con **MYSQL**, **POSTGRES**, **MONGO**, **REDIS**,  **FIREBIRD**, **PHP-FPM** y **NGINX**.

### Pre-requisitos 📋

_- Sistemas operativo **linux**_
_- Instalar  [docker](https://docs.docker.com/install/)_
_- Instalar [docker compose](https://docs.docker.com/compose/)_

### ¡IMPORTANTE!
_**Las configuraciones de las redes determinan el orden de arranque de los servicios**._


### Archivo docker-compose.yml de bases de datos.
_Comenzaremos levantando los contenedores de bases de datos y su red._
>Crear archivo docker-compose.yml
```bash
user@debian:~/Documents/Docker/databases$ nano docker-compose.yml
```
```yaml
version: '3.7'


services:
 
 mysql:
  image: mysql:${MYSQL_VERSION}
  restart: always
  ports:
   - ${MYSQL_PORT_LOCAL}:${MYSQL_PORT_DEFAULT}
  environment:
   MYSQL_ROOT_PASSWORD: ${MY_MYSQL_ROOT_PASSWORD}
   MYSQL_USER: ${MY_MYSQL_USER}
   MYSQL_PASSWORD: ${MY_MYSQL_USER_PASSWORD}
   MYSQL_DATABASE: ${MY_MYSQL_DB}
  volumes:
   - ./mysql/db:/var/lib/mysql
   - ./mysql/log:/var/lib/log
  networks:
   - databases

 postgres:
  image: postgres:${POSTGRES_VERSION}
  restart: always
  ports:
   - ${POSTGRES_PORT_LOCAL}:${POSTGRES_PORT_DEFAULT}
  environment:
   POSTGRES_ROOT_PASSWORD: ${MY_POSTGRES_ROOT_PASSWORD}
   POSTGRES_USER: ${MY_POSTGRES_USER}
   POSTGRES_PASSWORD: ${MY_POSTGRES_USER_PASSWORD}
   POSTGRES_DB: ${MY_POSTGRES_DB}
  volumes:
   - ./postgres/db:/var/lib/postgresql/data
   - ./postgres/log:/var/lib/postgresql/log
  networks:
   - databases

 firebird:
    image: kpsys/firebird
    volumes:
     - ./firebird/db:/usr/local/firebird/data
    ports:
      - ${FIREBIRD_PORT_LOCAL}:${FIREBIRD_PORT_DEFAULT}
    networks:
      - databases

 mongo:
  image: mongo:${MONGO_VERSION}
  restart: always
  ports:
   - ${MONGO_PORT_LOCAL}:${MONGO_PORT_DEFAULT}
  volumes:
   - ./mongo/db:/data/db
  networks:
   - databases

 redis:
  image: redis:${REDIS_VERSION}
  restart: always
  ports:
   - ${REDIS_PORT_LOCAL}:${REDIS_PORT_DEFAULT}
  volumes:
   - ./redis/db:/data:rw
  networks:
   - databases
   
networks:
 databases:

```
_Dentro del archivo **.env**  podemos gestionar las variables de configuración._

>Creando archivo .env:

```bash
user@debian:~/Documents/Docker/databases$ nano .env
```
>las variables de entorno se definen de la siguiente forma dentro del archivo .env
```bash
# VARIABLES MYSQL
MY_MYSQL_ROOT_PASSWORD=123456
MY_MYSQL_USER=demo
MY_MYSQL_USER_PASSWORD=demo
MY_MYSQL_DB=demo
MYSQL_VERSION=5.7
MYSQL_PORT_DEFAULT=3306
MYSQL_PORT_LOCAL=3306

# VARIABLES POSTGRES
MY_POSTGRES_ROOT_PASSWORD=123456
MY_POSTGRES_USER=demo
MY_POSTGRES_USER_PASSWORD=demo
MY_POSTGRES_DB=demo
POSTGRES_VERSION=9.4
POSTGRES_PORT_DEFAULT=5432
POSTGRES_PORT_LOCAL=5432

# VARIABLES MONGO
MONGO_VERSION=latest
MONGO_PORT_DEFAULT=27017-27019
MONGO_PORT_LOCAL=27017-27019

# VARIABLES REDIS
REDIS_VERSION=latest
REDIS_PORT_DEFAULT=6379
REDIS_PORT_LOCAL=6379

# VARIABLES FIREBIRD
FIREBIRD_PORT_DEFAULT=3050
FIREBIRD_PORT_LOCAL=3050
```
### Comandos para levantar los servicios.
_Podemos levantar todos los servicios que están dentro del archivo **docker-compose.yml** o elegir manualmente qué servicio queremos levantar._
>Levantando todos los servicios
```bash
user@debian:~/Documents/Docker/databases$ docker-compose up -d
```
>Levantando un servicio manualmente
```bash
user@debian:~/Documents/Docker/databases$ docker-compose up -d nombre_servicio
```

### Archivo docker-compose.yml de php.
_Levantando servicio de **PHP-FPM**._
>Crear archivo docker-compose.yml
```bash
user@debian:~/Documents/Docker/PHP$ nano docker-compose.yml
```
```yaml
version: '3.7'

services:
 php-fpm:
  build:
   context: ./php-fpm
  restart: always
  ports:
   - ${PORT}:${PORT}
  volumes:
   - ../public_html:/var/www/html:rw
  networks:
   - php-fpm
   - db
networks:
 php-fpm:
 db:
  external:
   name: databases_databases 

```
_Dentro del archivo **.env**  podemos gestionar las variables de configuración._

>Creando archivo .env:

```bash
user@debian:~/Documents/Docker/php$ nano .env
```
>las variables de entorno se definen de la siguiente forma dentro del archivo .env
```bash
# VARIABLES PHP
PORT=9000
```
### Comandos para levantar los servicios.
_Podemos levantar todos los servicios que están dentro del archivo **docker-compose.yml** o elegir manualmente qué servicio queremos levantar._
_-Nota: en este caso solo hay un servicio._
>Levantando todos los servicios
```bash
user@debian:~/Documents/Docker/php$ docker-compose up -d
```

### Archivo docker-compose.yml de nginx.
_Levantando servicio de **NGINX**._
>Crear archivo docker-compose.yml
```bash
user@debian:~/Documents/Docker/nginx$ nano docker-compose.yml
```
```yaml
version: '3.7'

services:
 nginx:
  image: nginx:${NGINX_VERSION}
  restart: always
  ports:
   - ${PORT}:80
   - ${PORT_SSL}:443
  volumes:
   - ../public_html/:/usr/share/nginx/html
   - ./config/default.conf:/etc/nginx/conf.d/default.conf
   - ./config/nginx.conf:/etc/nginx/nginx.conf
  networks:
   - php-fpm
   - db
   - web
networks:
 web:
 db:
  external:
   name: databases_databases   
 php-fpm:
  external:
   name: php_php-fpm
```
_Dentro del archivo **.env**  podemos gestionar las variables de configuración._

>Creando archivo .env:

```bash
user@debian:~/Documents/Docker/nginx$ nano .env
```
>las variables de entorno se definen de la siguiente forma dentro del archivo .env
```bash
# VARIABLES NGINX
NGINX_VERSION=latest
PORT=80
PORT_SSL=443
```
### Comandos para levantar los servicios.
_Podemos levantar todos los servicios que están dentro del archivo **docker-compose.yml** o elegir manualmente qué servicio queremos levantar._
_-Nota: en este caso solo hay un servicio._
>Levantando todos los servicios
```bash
user@debian:~/Documents/Docker/php$ docker-compose up -d
```

### Gestionando un ejemplo practico con laravel.
_Para verificar que nuestros contenedores estén correctamente funcionando usamos el siguiente comando:_
>Podemos ver los contenedores activos.
```bash
user@debian:~/Documents/Docker$ docker ps
```
_Para el ejercicio hemos elegido levantar el servicio de bases de datos de nombre: **mysql**_

>Instrucción para el servicio de mysql
```bash
user@debian:~/Documents/Docker/databases$ docker-compose up -d mysql
```
>Instrucción para el servicio de php-fpm
```bash
user@debian:~/Documents/Docker/php$ docker-compose up -d
```
### Configurando nginx.
_Dentro de la carpeta **nginx/conf** encontramos el archivo **default.conf**_
>Configuración inicial antes de encender el servicio

```bash
user@debian:~/Documents/Docker/nginx/conf$ nano default.conf
```

```bash
server {
	listen 80;

	server_name test.demo;
        
         index  index.php index.html index.htm;

	location / {
          root   /usr/share/nginx/html/test/public;
         
          try_files $uri $uri/ /index.php$is_args$args; 
        }

        # PHP
	location ~ \.php$ {
	  root           /var/www/html/test/public;
          fastcgi_pass   php-fpm:9000;
          fastcgi_index  index.php;
          fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
          include        fastcgi_params;
        }
}
```
### Configurando hosts.

```bash
user@debian:~/Documents/Docker/php$ sudo nano /etc/hosts
```
```bash
127.0.0.1	localhost
127.0.1.1	debian.user	debian
127.0.0.1	test.demo
```
>Instrucción para el servicio de nginx
```bash
user@debian:~/Documents/Docker/php$ docker-compose up -d
```
### Configurando proyecto con laravel.
_Bajamos el proyecto del repositorio oficial de [laravel](https://github.com/laravel/laravel) dentro de la carpeta **nginx/public_html**_
>Bajando repositorio con git
```bash
user@debian:~/Documents/Docker/nginx/public_html$ git clone https://github.com/laravel/laravel.git test && cd test
```
_Luego instalamos las dependencias necesarias con **[composer](https://hub.docker.com/_/composer)**_
>Comando para instalar dependencias

```bash
user@debian:~/Documents/Docker/public_html/test$ docker run --rm --interactive --tty --volume $PWD:/app --volume ${COMPOSER_HOME:-$HOME/.composer}:/tmp composer install
```

_Después de instalar las dependencias, procedemos a configurar el archivo **.env** que necesita laravel, los permisos necesarios dentro de la carpeta **storage** **bootstrap/cache** y el **key** cómo lo indica la [documentación oficial](https://laravel.com/docs/5.8#installing-laravel)._
>Pasos
```bash
user@debian:~/Documents/Docker/public_html/test$ cp -r .env.example .env && sudo chomod 777 -R storage && sudo chomod 777 -R bootstrap/cache
```
>Nota: **php_php-fpm_1** es el contenedor de php.
```bash
user@debian:~/Documents/Docker/public_html/test$ docker exec --user $(id -u):$(id -g) -it php_php-fpm_1 php test/artisan key:generate
```

```bash
user@debian:~/Documents/Docker/public_html/test$ nano .env
```
```bash
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

# Aquí configuramos la base de datos
# Podemos usar la dirección del contenedor o el nombre de servicio
# Los contenedores están dentro de la misma red.

DB_CONNECTION=mysql
DB_HOST=mysql # Nombre del servicio mysql o la ip del contenedor
DB_PORT=3306
DB_DATABASE=demo #Variables configuradas previamente
DB_USERNAME=demo #Variables configuradas previamente
DB_PASSWORD=demo #Variables configuradas previamente

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

```


_Finalmente podemos comprobar que nuestro proyecto esté funcionando correctamente
>Ingresando en el navegador
```bash
http://test.demo/
```

## Autores ✒️

* **Jeaneil Bernal Sierra**
* **Cristian Narvaez Useche** 
* **Leonardo Seña Pimentel** 




