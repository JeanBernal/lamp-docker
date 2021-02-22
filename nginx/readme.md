

# Docker servidor gninx

## Comenzando 🚀

Estas instrucciones le permitirán configurar el servidor nginx que se alojaran en un contenedor .

## Pre-requisitos 📋

 - _Sistemas operativo **linux**_
       
 - _Instalar [Docker]([https://docs.docker.com/install/])_

 - _Instalar [docker-compose]([[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)])_

## Archivo docker-compose.yml
Empezaremos configurando el archivo **docker-compose.yml**

> Mediante el editor de texto deseado crean un archivo en el directorio donde se va alojar su contenedor.

```bash
user@debian:~/Documentos/Docker/nginx$ nano docker-compose.yml 
```
> El cual debe contener la siguiente información 
```yml
version: '3.7'

services:
 nginx:
  image: nginx:${NGINX_VERSION}
  restart: always
  ports:
   - ${PORT}:80
   - ${PORT_SSL}:443
  volumes:
   - ./public_html/:/usr/share/nginx/html
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
_Después de crear el archivo docker-compose.yml se debe crear el directorio config donde se almacenaran las configuraciones del servidor gninx:_
```bash
users@debian:~/Documentos/Docker/nginx$ mkdir conf
users@debian:~/Documentos/Docker/nginx$ cd conf
users@debian:~/Documentos/Docker/nginx/config$ nano default.conf
```
>El cual contiene la siguiente información por defecto

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.php index.html index.htm;
    }


    # PHP
    location ~ \.php$ {
	  root           /var/www/html;
          fastcgi_pass   php-fpm:9000;
          fastcgi_index  index.php;
          fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
          include        fastcgi_params;
    }


    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    #error_page   500 502 503 504  /50x.html;
    #location = /50x.html {
    #    root   /usr/share/nginx/html;
    #}

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           /var/www/html;
    #    fastcgi_pass   php-fpm:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```
```bash
users@debian:~/Documentos/Docker/nginx/config$ nano nginx.conf
```
>El cual contiene la siguiente información por defecto

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}


```
 Después se procede en la creación del directorio **public_html** el cual va a contener un **index.html**   que cualquier información. En esta carpeta se van almacenar todos los proyectos.
 ```bash
users@debian:~/Documentos/Docker$ mkdir public_html && cd public_html
```
 ```bash
users@debian:~/Documentos/Docker/public_html$ nano index.html
```

```html
<!DOCTYPE html>  
<html>  
<body>  
	<h1>My First Heading</h1>  
	<p>My first paragraph.</p>  
</body>  
</html>
```
 
### Variables de entorno
Como se puede observar en el archivo **docker-compose.yml** se encuentran 3 variables de entorno las cuales son :
> * NGINX_VERSION
> * PORT
> * PORT_SSL

Las variables de entorno pueden ser modificadas de la siguiente manera:

_El archivo **docker-compose.yml** puede albergar  diferentes variables de entorno las cuales permiten la configuración del contenedor ya que ellas almacenan información como la versión, puertos, host, usuarios etc. Estas variables están contenidas en  un archivo  **.env**_

_Al abrir el archivo docker-compose.yml en el directorio se pueden observar las diferentes variables de entorno_
>Ingresar al archivo docker-compose.yml
```bash
users@debian:~/Documentos/Docker$ nano docker-compose.yml
```
```yml
version: '3.7'

services:
 nginx:
  image: nginx:${NGINX_VERSION}
```
>En este caso la variables es NGINX_VERSION

_Para gestionar las variables de entorno debemos realizar las acciones sobre el archivo **.env**_
>Ingresamos al directorio donde este alojado el archivo docker-compose.yml y abrimos el archivo .env o en que caso de que no exista se crea
```bash
user@debian:~/Documentos/Docker$ nano .env
```
>las variables de entorno se definen de la siguiente manera dentro del archivo .env

```bash
# VARIABLES DE CONFIGURACIÓN DE NGINX
NGINX_VERSION=2.0
```
>Una vez creada las variables se pueden usar en nuestro archivo docker-compose.yml como mostramos anteriormente. Ejemplo:
```yml
image: nginx:${NGINX_VERSION}
```
>De este modo se estaría especificando que  la versión 2.0 de la imagen de nginx en docker hub sera la instalada. 

En el archivo **docker-compose.yml** también se pueden destacar  volúmenes  que permiten que nuestra información sea persistente en el equipo donde se aloja en contenedor (Host). Son los siguientes:
```yml
volumes:
   - ../public_html/:/usr/share/nginx/html:rw
   - ./config/default.conf:/etc/nginx/conf.d/default.conf:ro
   - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
```

> Esto quiere decir que toda la información de los directorios **public_html** y los archivos **config/default.conf** y config/nginx.conf se alojaran en los directorios del contenedor que se muestran en frente de ellas,  **/usr/share/nginx/html** , **/etc/nginx/conf.d/default.conf**  y **/etc/nginx/nginx.conf** respectivamente.

### Networks
En el archivo docker-compose.yml se pueden observar tres redes diferentes:
> * php-fpm
> * db
> * web

```yml
  networks:
   - php-fpm
   - db
   - web
   - networks:
 web:
 db:
  external:
   name: databases_databases   
 php-fpm:
  external:
   name: php_php-fpm
```

* La red **php-fpm** permite la conexion con el contenedor donde se encuentra alojado php.
* La red **db** permite la comunicación con el contenedor donde se encuentran alojadas las diferentes bases de datos (firebird, mysql, postgres)
* La red **web** es la red interna del contendor de gninx.

## Autores ✒️
* **Leonardo Seña Pimentel** 
* **Cristian Narvaez Useche** 
* **Jeaneil Bernal Sierra**





