# 1. Instalar y configurar Tomcat

<span style="color: lightgreen">
Nota: Para el primer objetivo de Tomcat, he usado el puerto 8079, pero el puerto más comunmente usado para el Tomcat es el 8080.
</span>

## 1.1. Instalar Tomcat

Para instalar Tomcat y llegar a ver el Tomcat Manager en el navegador,
he seguido esta guía, hasta el paso número 4 (no incluído) https://techviewleo.com/install-tomcat-on-amazon-linux/

video extra:
https://www.youtube.com/watch?v=SMQjlcRmgiM

<span style="color: lightgreen">
Nota: Después de seguir la guía, en el momento final de probar a ver el Tomcat Manager en el Chrome, no me salía … por qué??
</span>

<span style="color: lightgreen">
Resulta que, aunque había abierto un puerto (8079) para el Tomcat a través de AWS, 
y cambiado el puerto también (8079) dentro del archivo de configuración del Tomcat de server.xml…
</span>

`sudo vi usr/share/apache-tomcat-9.0.39/conf/server.xml`

![](./img/1.png)

<span style="color: lightgreen">
Pues aún así, en el navegador, al poner IP:8079 seguía sin funcionarme…
Y todo era porque, de entre todas las cosas que ya llevaba hechas de antemano,
resulta que el servicio del firewalld estaba activo !!! y no debería de estarlo !!
</span>

<span style="color: lightgreen">
Entonces el firewalld estaba interfiriendo con el firewall de AWS …
</span>

`sudo systemctl stop firewalld`

`sudo systemctl disable firewalld`

<span style="color: lightgreen">
Ahora voy al navegador a poner IP (o dominio):8079 y puedo ver el Tomcat Manager perfectamente.
</span>

![](./img/2.png)

video extra:
https://www.youtube.com/watch?v=mBVINWl-AWo&ab_channel=LDTS

## 1.2. Configurar Tomcat

### 1.2.1. Usuarios

Empezamos creando los usuarios necesarios para entrar en Tomcat Manager.
Para ello, retomamos la guía de antes del Tomcat a partir del punto 4

https://techviewleo.com/install-tomcat-on-amazon-linux/

Abrimos el archivo de:

`vi usr/share/apache-tomcat-9.0.39/conf/tomcat-users.xml`

Añadimos al final:

```
<role rolename="admin-gui"/>
<role rolename="manager-gui"/>
<user username="tomcat" password="tupassword" fullName="Administrator"
roles="admin-gui,manager-gui"/>
```

`sudo systemctl restart tomcat`

<span style="color: lightgreen">
Nota: esto que hemos hecho no es obligatorio hacerlo ahora ya que esto sería para entrar en el gestor de Tomcat...
</span>

<span style="color: lightgreen">
Si me diese algún error Tomcat Manager en lo que hemos hecho hasta ahora …
</span>

<span style="color: lightgreen">
Abrir el archivo de configuración de Tomcat de:
</span>

`vi usr/share/apache-tomcat-9.0.39/webapps/manager/META-INF/context.xml`

<span style="color: lightgreen">
En él, comentaremos las líneas que hacen que sólo se pueda acceder al Tomcat Manager por localhost…
</span>

## 1.3. VirtualHost en Tomcat

### 1.3.1. webapps

Hacemos una copia a la carpeta webapps y le cambiamos el nombre

`sudo cp -rf usr/share/apache-tomcat-9.0.39/webapps/usr/share/apache-tomcat-9.0.39/webPrueba1`

### 1.3.2. server.xml

Ahora vamos al archivo de configuración de Tomcat de server.xml

`sudo vi usr/share/apache-tomcat-9.0.39/conf/server.xml`

y buscamos el siguiente bloque:

![](./img/3.png)

Esto es equivalente a un VirtualHost de Apache web server, o a un Server Block de Nginx (ver apartado siguiente). 
Por tanto, lo que tenemos que hacer es añadir otra sección Host
nueva, de forma que tendremos 2 Host -equivalentes a dos VirtualHost de Apache...

![](./img/4.png)

… donde webPrueba1 es el directorio que copiamos antes y seria nuestro nuevo document root (lo que en Apache equivaldría al DocumentRoot).

### 1.3.3. Crear el subdominio

Ahora vamos a IONOS para crear un nuevo subdominio 
(de la misma forma que creamos el del wordpress… wp.tudominio.com)

![](./img/5.png)

### 1.3.4. Registro tipo A

Ahora tenemos que añadirle a este subdominio el registro tipo A que apunta a mi IP

<span style="color: lightgreen">
Nota: pero primero, desactivamos el Registro A que ya viene predeterminado
con la creación del subdominio
</span>

![](./img/6.png)

![](./img/7.png)

![](./img/8.png)

![](./img/9.png)

### 1.3.5. Volvemos al server.xml

Tenemos que poner bien el nuevo (server)name

![](./img/10.png)

### 1.3.6. Propietarios

Y por último, cambiamos el propietario de la carpeta webPrueba1 y comprobamos los permisos:

`chown -R tomcat:tomcat usr/share/apache-tomcat-9.0.39/webPrueba1/`

![](./img/11.png)

![](./img/12.png)

## 1.4. Subir la app

Una vez hecho esto solo nos queda ir al directorio webPrueba1 y poner el el Root lo que vamos a desplegar, o entrar al tomcat manager (dns:8079) y subir la aplicación en .war.

video extra:
https://www.youtube.com/watch?v=PXozxgEMVss&ab_channel=LDTS

# 2. EXTRA. ¿ Cómo hacer un proyecto en VS y exportarlo a .war?

<span style="color: lightgreen">
Para reforzar lo aprendido, voy a realizar los mismos pasos para desplegar un proyecto en Tomcat, pero esta vez, creando un proyecto de maven en JSP con un nuevo subdominio (webprueba2.tudominio.com)
</span>

## 2.1. Extensión en VS

Vamos a Visual Studio y en el apartado de las extensiones, buscamos la de Extension Pack for Java y la instalamos

<span style="color: lightgreen">
Nota: esta extensión es como la recopilación a su vez, de más extensiones de visual studio para este propósito de desarrollo, es decir, con esta extensión, también tendremos como instaladas y habilitadas otras extensiones como, Project Manager For Java o Maven For Java …
</span>

<span style="color: lightcoral">
Importante: Es necesario instalar/tener instalado, un jdk (el jdk más actual)
</span>

https://www.oracle.com/es/java/technologies/javase/javase8-archive-downloads.html

<span style="color: lightcoral">
En mi caso, como versión de java principal en mi sistema tengo el jre1.8.0_311, y he instalado el jdk1.8.0_202
</span>

<span style="color: lightyellow">
Curiosidad: El jdk1.8.0_202, también me instaló a su compañero el jre1.8.0_202, pero si hago en el cmd el comando java -version, lo que me sale como versión principal de java en mi sistema es el jre1.8.0_311 el cual era el que yo ya tenía en mi
sistema, y que es más actual aún que el jre compañero que me ha venido del jdk1.8.0_202 … ¿cómo se explica esto, si en la variable de sistema JAVA_HOME la tengo apuntando a la carpeta del jdk1.8.0_202 ?
</span>

## 2.2. Crear el proyecto en JSP

Para crear un proyecto de Maven en JSP…

![](./img/13.png)

![](./img/14.png)

![](./img/15.png)

![](./img/16.png)

![](./img/17.png)

![](./img/18.png)

![](./img/19.png)

![](./img/20.png)

## 2.3. Exportar el proyecto

Para exportar el proyecto a un archivo .war … ¡ Antes tenemos que instalar Maven !

### 2.3.1. Instalar Maven

Para instalar maven, vamos a la web de maven apache project y descargamos el Binary zip archive

https://maven.apache.org/download.cgi

y seguimos esta guía:

https://www.youtube.com/watch?v=km3tLti4TCM&ab_channel=AmitThinks

### 2.3.2. Descompresión del zip y ubicación

Descomprimimos el archivo zip y llevamos la carpeta resultante a Program Files, y nos metemos dentro de ella, dentro de bin, y copiamos su ruta

### 2.3.3. Variable de entorno Path

Ahora vamos a las variables de entorno del sistema, para editar el path del sistema y añadirle una nueva ruta que es la que habíamos copiado

### 2.3.4. Variable de entorno Maven

Ahora vamos a crear una variable de sistema llamada MAVEN_HOME y le pegamos la ruta de antes pero sin el \bin, y ponemos ésta justa debajo del JAVA_HOME

### 2.3.5. Comprobación final

Si en el cmd hacemos el comando mvn --version (ó -v) podremos comprobar que ya lo tenemos instalado correctamente

![](./img/21.png)

## 2.4. Comando final

Una vez que tenemos ya nuestra proyecto listo, desde la terminal de VS ejecutamos el comando de maven para compilar en .war

`mvn clean package`

y se nos guardará el proyecto.war en la carpeta de Target.

![](./img/22.png)

# 3. Despliegue del proyecto de maven en JSP

## 3.1. Subir el proyecto

Subimos el proyecto.war con Filezilla al servidor de AWS

## 3.2. Autodescompresión

Lo movemos hacia dentro de la carpeta de webapps para que se auto-descomprima el .war

`mv proyecto.war /usr/share/apache-tomcat-9.0.39/webapps/`

## 3.3. Colocar el proyecto en su carpeta

Una vez descomprimido, eliminamos el .war y llevamos la carpeta
resultante a su carpeta correspondiente la cual ya habíamos creado de antes

`mv proyecto /usr/share/apache-tomcat-9.0.39/webPrueba1/`

<span style="color: lightgreen">
Nota: Antes de mover todo lo que hay dentro de /usr/share/apache-tomcat-9.0.39/webPrueba1/proyecto/ hacia /usr/share/apache-tomcat-9.0.39/webPrueba1/ROOT/ debemos eliminar en esa carpeta de ROOT/ todos los archivos los cuales sus nombres coincidan de antemano con alguno de los nuestros, de la carpeta
proyecto/ (por si no se sobreescriben, que no se entorpezcan los unos con otros)
</span>

Movemos el contenido de /usr/share/apache-tomcat-9.0.39/webPrueba1/proyecto/ hacia /usr/share/apache-tomcat-9.0.39/webPrueba1/ROOT/

## 3.5. Propietarios

Cambiamos los propietarios de forma recursiva para Tomcat

`chown -R tomcat:tomcat /usr/share/apache-tomcat-9.0.39/webPrueba1/`

Reseteamos tomcat

`systemctl restart tomcat`

## 3.6. Resultado en navegador

Vamos al navegador y ponemos webprueba1.tudominio.com:8079

y funciona !!!

# 4. Nginx Proxy

Esta parte trata sobre cómo hacer que un servicio escuche a través de otro servicio (pareciendo que dos servicios escuchan por el mismo puerto). Debemos hacer que Tomcat escuche a través de Nginx (puerto 80) y después para asegurarnos, eliminaremos el puerto 8079 del grupo de segurrar de AWS, que previamente habíamos abierto y utilizado para el primer objetivo.

## 4.1. Netstat

Para ver qué puertos están escuchando usamos el netstat:

`nestat -ntl`

`netstat -ntlp`

`netstat -n4a`

![](./img/23.png)

## 4.2. Server block

Creamos un nuevo server block para Nginx

`vi etc/nginx/conf.d/tudominio.com.tomcat.conf`

```
server {
  listen 80;
  server_name webprueba2.tudominio.com;
  # note that these lines are originally from the "location /"block
  #root /usr/share/nginx/html/portfolio;
  #index index.php index.html index.htm;
  #location / {
    #try_files $uri $uri/ =404;
  #}
  error_page 404 /404.html;
  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
  location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
  }
  location / {
    #proxy_pass http://127.0.0.1:8079;
    #proxy_pass http://tuip:8079;
    proxy_pass http://webprueba2.tudominio.com:8079;
  }
}
```


<span style="color: lightgreen">
Nota: podemos apreciar que este server block NO tiene DocumentRoot, si no un nuevo location/ que redirige al subdominio de webprueba2.tudomninio.com
</span>


<span style="color: lightyellow">
Curiosidad: tanto si redirecciono con el proxy_pass al localhost(127.0.0.1) como a mi IP pública (tuip), se redirige al Tomcat Manager, así que necesito poner en el proxy_pass el subdominio:8079 para que realmente se vaya al subdominio webprueba2.tudominio.com
</span>


<span style="color: lightyellow">
Curiosidad: También puedo hacer que se vea en weprueba2.tudominio.com mi proyecto de de maven en JSP, y que si pongo weprueba2.tudoninio.com:8079 salga el webprueba1 … Esto se conseguiría si en el server block de webprueba2.tudominio.com.conf en la parte de arriba de servername pongo webprueba2.tudominio.com, y en la parte de abajo del proxy_pass pongo webprueba1.tudominio.com:8079
</span>


<span style="color: lightcoral">
Importante: en el video del profesor dice que puedo cerrar el puerto 8080, porque como el Tomcat está escuchando a través de Nginx, ya no haría falta el 8080, tan solo el 80 para Nginx … pues yo necesito tenerlo abierto, si no, no ve funciona!!
</span>


<span style="color: lightgreen">
Nota: Para asegurarnos de que el proxy_pass (redireccionamiento de escucha) efectivamente está funcionando, debemos cerrar (eliminar en AWS) el puerto 8079 (o el que se hubiera utilizado para el primer objetivo del tomcat)
</span>


<span style="color: lightgreen">
Nota: El profesor me ha encargado hacer que el server block de mi portfolio (dominio principal) sea el server block por default, para lo cual, he tenido que añadir a su server_name, un espacio y una barra_baja después del nombre de mi dominio principal... server_name tudominio.com _;
</span>


<span style="color: lightyellow">
El resumen del server block del webprueba2 en este objetivo es:  
</span>

```
server {
   listen 80;
   server_name webprueba2.tudominio.com;

   location / {
      proxy_pass http://127.0.0.1:8079;
   }
}
```
