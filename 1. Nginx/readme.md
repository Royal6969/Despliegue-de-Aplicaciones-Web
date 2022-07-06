# 1. Configurar Nginx

## 1.1. Vamos al archivo php.ini

`sudo vi /etc/php.ini`

## 1.2. Buscamos la línea en la que pone

`;cgi.fix_pathinfo=1`

## 1.3. La descomentamos y la igualamos a 0

`cgi.fix_pathinfo=0`

![](./img/1.png)

# 2. Configurar php-fpm

Vamos al archivo /etc/php-fpm.d/www.conf
## 2.1. Modificamos Listen... ponemos 

`listen = /var/run/php-fpm/php-fpm.sock`

![](./img/2.png)

Más adelante le daremos la ruta del socket a Nginx.
Una vez hecho esto, no necesitamos que php-fpm escuche por localhost (127.0.0.1).

## 2.2. Usuario con el que se ejecuta php-fpm
Decimos a php-fpm con qué usuario ha de ejecutarse su proceso.
Dado que php-fpm necesitará entrar en los directorios web de Nginx, el ajuste es el siguiente:

```
user = nginx
group = nginx
```

![](./img/3.png)

## 2.3. Lista de usuarios a los que php-fpm escuchará
Un usuario o proceso que no esté en esta lista llamada acl_users será
ignorado y no se le atenderá.
Por eso mismo, descomentamos la siguiente línea, y nos aseguramos que en la lista de usuarios esté el usuario que use Nginx:

`listen.acl_users = apache,nginx`

![](./img/4.png)

<span style="color:lightgreen">Nota: Esto da permisos a los procesos nginx para conectar con el socket de php-fpm. 
Ojo porque un espacio después de la coma puede hacer que falle.</span>

## 2.4. Configurar el servicio php-fpm

`systemctl start php-fpm`
`systemctl enable php-fpm`

<span style="color:lightpink;">
Curiosidad: La primera vez que hice esta parte, antes de hacer en este punto un systemctl
restart php-fpm, me doy cuenta de que tengo que ir al archivo de /etc/php-fpm.d/www.conf y
en él cambiar la línea de listen = /run/php-fpm/www.conf a listen =
/var/run/php-fpm/php-fpm.sock
</span>

<span style="color:lightpink">
La cosa fue que, al hacer este par de cosas en ese orden, es decir, primero cambié aquella
ruta del archivo para que cogiese un archivo que aún no existe ... y cuando después le hice
el systemctl start/enable/restart,  me daba un error de que no se podía activar ni hacer nada
con el servicio php-fpm, y en el error que me daba al hacer systemctl status -l php-fpm, el
error venía precisamente de la ruta que apunta a ese archivo de php-fpm.sock el cual aún
no existe...
</span>

<span style="color:lightpink">
La curiosidad de esto está en que, haciendo los pasos justamente al revés, es decir, si
hiciese un yum remove php-fpm y después un yum install php-fpm, y reiniciase el archivo de
www.conf para volver hacer estos pasos al revés, puedo comprobar que, si primero le hago
el systemctl start/enable/restart al php-fpm, y después modifico la ruta esa del archivo que
apunta al socket que aún no existe, me ocurre que ahora sí funciona el servicio de php-fpm,
y simplemente he hecho los mismos pasos pero en diferente orden, y ya no me da aquel
error, y es que se trata de un archivo que se genera automáticamente.
</span>

# 3. Configuración del server block

Configuramos el server block para que incluya la conexión con php-fpm

## 3.1. Vamos a crear un archivo en la ruta /etc/nginx/conf.d/

`sudo vi /etc/nginx/conf.d/tudominio.com.conf`

## 3.2. Dentro de él, vamos a poner lo siguiente:

```
server {
  listen 80;
  server_name tudominio.com;
  root /usr/share/nginx/html;
  index index.php index.html index.htm;
  charset UTF-8;
  location / {
    try_files $uri $uri/ =404;
  }
  error_page 404 /404.html;
  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
  location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME
    $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
  include fastcgi_params;
  fastcgi_param  SCRIPT_FILENAME
  $document_root$fastcgi_script_name;
  fastcgi_split_path_info ^(.+\.php)(/.+)$;
  fastcgi_index index.php;
  # fastcgi_intercept_errors on;
  # fastcgi_keep_conn on;
  # fastcgi_read_timeout 300;
  # fastcgi_pass   127.0.0.1:9000;
  #fastcgi_pass  unix:/var/run/php-fpm/www.conf;
  #for ubuntu unix:/var/run/php/php8.0-fpm.sock;
  ##
  # FastCGI cache config
  ##
  # fastcgi_cache_path /var/cache/nginx levels=1:2
  keys_zone=WORDPRESS:10m max_size=1000m inactive=60m;
  # fastcgi_cache_key $scheme$host$request_uri$request_method;
  # fastcgi_cache_use_stale updating error timeout invalid_header http_500;
  fastcgi_cache_valid any 30m;
}
```

Guardamos y salimos del archivo, para hacer un sudo systemctl restart nginx

## 3.3. Ahora vamos a la ruta que hemos definido como Document Root de Nginx

Vamos a /usr/share/nginx/html/

Aquí vamos a crear un archivo llamado index.php 
(porque en el server block que habíamos creado antes hemos dicho que a través del Document Root de Nginx, 
sólo se pueda acceder a los archivos llamados index index.php index.html index.htm

sudo vi /usr/share/nginx/html/index.php

3.4. Dentro de este index.php definiremos la clásica función que nos informa acerca de todas las características de nuestro php

```
<?php
  phpinfo();
?>
```

## 3.5. De nuevo hacemos un

`sudo systemctl restart nginx`

`sudo systemctl restart php-fpm`

Abrimos una nueva pestaña en nuestro navegador, y ponemos nuestra IP, 
o mejor aún, nuestro dominio (www.tudominio.com), y añadimos /index.php

Y efectivamente nos sale la lectura de la función infophp() mostrándonos toda la información acerca de nuestro php, 
lo cual quiere decir que nuestro php funciona perfectamente.

![](./img/5.png)

<span style="color:violet">Para el server block, he seguido esta guía:</span>

https://awswithatiq.com/setup-php-nginx-in-amazon-linux-2/

# 4. Despliegue en Nginx

> VirtualHosts, Server blocks y Hosts.
Cómo dar servicio web de varios dominios diferentes con un servidor y una sola IP

Esta parte trata de volver a conseguir sacar por el navegador, 
el portfolio a través de mi dominio principal, 
y el wordpress a través de un subdominio

Nota: Antes de nada, recordar que,
- los server blocks están en: /etc/nginx/conf.d/
- el DirectoryRoot de Nginx está en: /usr/share/nginx/html/

## 4.1. Vamos primero a por el portfolio

Copiamos la carpeta del portfolio que ya hicimos desde el DirectoryRoot de Apache hacia el DirectoryRoot de Nginx:

`sudo cp -rf /var/www/html/portfolio /usr/share/nginx/html/`

## 4.2. Cambiamos los permisos del grupo 

Tanto para el directorio de /usr/share/nginx/html/ como para el directorio del /usr/share/nginx/html/portfolio

`sudo chgrp -R nginx /usr/share/nginx/html/`

`sudo chgrp -R nginx /usr/share/nginx/html/portfolio/`

## 4.3. Server block

Ahora entramos en el server block que ya configuramos para el PHP de Nginx

`sudo vi /etc/nginx/conf.d/tudominio.com.conf`

y para añadir la ruta del portfolio, cambiamos solo la parte del principio a...

```
server {
  listen 80;
  server_name tudominio.com;
  root /usr/share/nginx/html/portfolio;
  index index.php index.html index.htm;
  charset UTF-8;
```

## 4.4. Reseteamos Nginx

`sudo systemctl restart nginx`

y vamos al navegador y si ponemos www.tudominio.com ...

FUNCIONA !! ya sale el portfolio a través de Nginx !!

<span style="color:lightgreen">
Nota: ATENCIÓN! previamente a esto, configuramos el server block de tudominio.com.conf
para que enrutara hacia el DirectoryRoot de Nginx y allí pudiese acceder precísamente al
archivo de index.php, el cual albergaba la función de phpinfo(), la cual si en el navegador
poníamos www.tudominio.com/index.php nos llevaba a toda la información de nuestro PHP
... pero ahora NO, porque en ese mismo server block, ahora hemos enrutado hacia dentro
del DirectoryRoot dentro de portfolio ...
</span>

<span style="color:lightgreen">
Para que vuelva a funcionar y vuelva a poder verse la información de PHP poniendo en el
navegador la misma dirección que antes... debo mover el archivo de index.php del
DirectoryRoot de Nginx hacia el directorio del portfolio, cambiándole el nombre a info.php ...
</span>

## 5. Wordpress

Vamos a recordar y proceder con la instalación de wordpress a través de la guía que ya seguimos en su momento cuando lo hicimos todo esto con Apache...

https://docs.aws.amazon.com/es_es/AWSEC2/latest/UserGuide/hosting-wordpress.html

## 5.1. Una vez dentro del DirectoryRoot de Nginx...

- `sudo wget https://wordpress.org/latest.tar.gz`
- `sudo tar -xzf latest.tar.gz`
- `sudo rm -rf latest.tar.gz`
- `sudo chown root:nginx -R wordpress` 

<span style="color:lightgreen">
Nota: al igual que ya hicimos en su momento con el portfolio, es importante cambiar el propietario porque cuando descomprimes el archivo, los propietarios por defectos son ”nobody”, y hay que ponerlo root:nginx y vemos que todos los archivos se ponen en verde... esto no lo hice al principio y cuando fui a instalar wordpress en el subdominio, no me salía wordpress, si no que siempre me redirigía al servidor principal (portfolio)
</span>

![](./img/6.png)

## 5.2. Entramos en MySQL 

Vamos a volver a crear un usuario y una BBDD para el nuevo wordpress de Nginx

`mysql -u root -p`

`CREATE USER 'wp'@'localhost' IDENTIFIED WITH` 

`mysql_native_password BY 'xxx';`

`CREATE DATABASE wp;`

`GRANT ALL PRIVILEGES ON wp.* TO "wp"@"localhost";`

`FLUSH PRIVILEGES;`

## 5.3. Ahora vamos a crear y configurar un nuevo server block para wordpress

- Vamos hacia /etc/nginx/conf.d/
- Creamos el archivo wordpress.conf

`sudo vi etc/nginx/conf.d/wordpress`

<span style="color:lightgreen">
Nota: Como ya tenemos el server block de mi servidor principal, tan sólo habría que cambiarle un par de palabras... 
Para ello vamos a copiar éste y a cambiarle el nombre a wordpress.conf
</span>

`sudo cp /etc/nginx/conf.d/tudominio.com.conf /etc/nginx/conf.d/wordpress.conf`

- Entramos en el nuevo archivo de wordpress.conf y le cambiamos el principio a:

```
server {
  listen 80;
  server_name wordpress.tudominio.com;
  root /usr/share/nginx/html/wordpress;
  index index.php index.html index.htm;
  charset UTF-8;
```

- Paralelamente a esto, tenemos que ir ahora a IONOS 

Vemos si sigue creado mi subdominio de www.wordpress.tudominio.com , y de ser así, si sigue conectado a nuestra IP (servidor principal) a través del registro A de DNS ...

De no ser así, tendríamos que volver a crearlo y volver a asociarlo a nuestra IP ...

![](./img/7.png)

![](./img/8.png)

![](./img/9.png)

![](./img/10.png)

![](./img/11.png)

![](./img/12.png)

![](./img/13.png)

![](./img/14.png)

![](./img/15.png)

- Hacemos un sudo systemctl restart nginx

Y si ahora vamos al navegador y ponemos www.wordpress.tudominio.com ,
nos sale la intro de la instalación de wordpress para seleccionar el idioma...

```
db-wordpress: wp
user-db-wordpress: wp@localhost
pass-db-wordpress: tupassword
```

Para acceder a WordPress:

```
user: tuuser
pass: tucontraseña
email: tuemail
```

Procedemos a poner los datos necesarios y le damos al botón de “run installation”... 
y con esto ya estaría todo listo ! ...
¡¡ YA TENEMOS WORDPRESS también !!

![](./img/16.png)

![](./img/17.png)

# Preguntas y Cuestiones:

> P: ¿Qué relación existe entre Nginx y PHP-fpm?

> R: La relación que hay entre Nginx y PHP-fpm es que, Nginx necesita de PHP para leer archivos y funciones, y así “proyectarlos” en el navegador.
Nginx necesita poder ejecutar y controlar a PHP-fpm, ya que le manda a éste solicitudes para ejecutarlo y usarlo.

Para que PHP-fpm escuche las peticiones de Nginx:

- PHP-fpm escuchará a Nginx gracias a listen.acl_users = apache,nginx
Un usuario o proceso que no esté en esta lista llamada acl_users será ignorado y no se le
atenderá.

- Nginx se comunica con PHP-fpm con el socket de PHP-fpm el cual es:
fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;

- Este socket lo ha creado php-fpm al activarse

Nota: El resumen de los server blocks en este objetivo es:

Para el portfolio (dominio principal):

```
server {
    listen 80;
    server_name tudominio.com _;

    root /usr/share/nginx/html/portfolio;
    index index.html index.htm;
    charset UTF-8;

    location / {
      try_files $uri $uri/ =404;
   }

    error_page 404 /404.html;

     include fastcgi_params;
     fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;

     fastcgi_cache_valid any 30m;
}

```

Para el wordpress:

```
server {
   listen 80;
   server_name wp.tudominio.com;
   # note that these lines are originally from the "location /" block

   root /usr/share/nginx/html/wordpress;
   index index.php index.html index.htm;

   location / {
      try_files $uri $uri/ =404;
   }

   error_page 404 /404.html;

    location ~ \.php$ {
      try_files $uri =404;
      fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      include fastcgi_params;
   }

     include fastcgi_params;
     fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
     fastcgi_split_path_info ^(.+\.php)(/.+)$;
     fastcgi_index index.php;

}

```
