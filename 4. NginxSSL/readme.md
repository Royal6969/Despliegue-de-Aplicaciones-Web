# Nginx HTTPS

Nota: En este punto, en IONOS, he habilitado que mi dominio principal (tudominio.com) apunte también a mi IP (es decir, he activado verdaderamente mi dominio), ya que antes estaba apuntando a mi ip con el dominio y la triple w delante (www.tudominio.com), pero ahora lo he modificado para que mi ip apunte solo al nombre de mi dominio, sin la triple w, es decir, tudominio.com.

![](./img/1.png)

# 1. Certificado SSL

## 1.1. Descarga en Ionos

Voy a IONOS para descargar mi Certificado SSL oficial (el que me daban cuando me compré el dominio en IONOS)

![](./img/2.png)

![](./img/3.png)

![](./img/4.png)

![](./img/5.png)

![](./img/6.png)

![](./img/7.png)

![](./img/8.png)

## 1.2. Organizar los archivos

Y subimos los archivos que acabamos de descargar al servidor con FileZilla, y los tres archivos principales, los reorganizamos en una nueva carpeta.

![](./img/9.png)

## 1.3. Fusión de ficheros

Vamos a fusionar los dos archivos .cer

Y aquí viene una primera diferencia con Apache: no tenemos dos parámetros diferentes, para el certificado de la CA y el certificsdo de nuestro sitio web, si no sólo uno. Entonces, ¿cómo poner ambos?

El truco está en el archivo cerbundle.cer (que por supuesto, podría llamarse de otro modo). 

Ese archivo debe contener la concatenación de los dos certificados: el de la CA y el del sitio web.

Para crearlo basta con ejecutar:

`cat tucertificado.crt > cerbundle.cer`

`echo >> cerbundle.cer`

`cat CAcertificate.crt >> cerbundle.cer`

![](./img/10.png)

## 1.4. Nuevo server block

Ahora vamos al server block de nginx de mi portfolio, para poner debajo del
server {} que ya tenía el siguiente server {} nuevo para el ssl …

```
server {
    listen 443 ssl http2;
    server_name tudominio.com _;

    ssl on;
    ssl_certificate /etc/certs/reales/megacertificado.cer;
    ssl_certificate_key /etc/certs/reales/_.tudominio.com_private_key.key;

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

![](./img/11.png)

Nota: Para poder completar bien este objetivo, acabaremos borrando el server{} antiguo para dejar solo el nuevo server{} del ssl.

## 1.5. Resetear Nginx

Reiniciamos nginx, y si ahora ponemos en una nueva pestaña de incógnito en el
Chrome, https://tudominio.com, puedo comprobar que, efectivamente sale con el
candado, y si miro el certificado, es el que me expidió IONOS.

![](./img/12.png)

![](./img/13.png)

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}
```

- listen 80 → que escucha por el puerto 80
- default_server → si me fueran hacer un ataque de DDOS, pues todas peticiones se redirigirían al servidor principal (este mismo)
- listen [::]:80 → si me hacen un telnet, también escucharía el puerto 80
- servername _ → cualquier dominio que no tenga un serverblock (en este
puerto)
- return 301 https://$host%request_uri → lo que puedan escribir en la barra de
navegación, se redirige como https (redireccionamiento genérico)

# 2. El archivo include

## 2.1. Volvemos al server block principal

Vuelvo al server block de mi portfolio (mi server block principal, el de tudominio.com.conf) para modificarlo y dejarlo de esta manera:

![](./img/14.png)

`include /etc/nginx/include/include.conf;`

## 2.2. include.conf

Ahora vamos a crear una carpeta llamada “include” dentro del directorio de Nginx, en la que crearemos a su vez, un archivo llamado “include.conf” y en él
introduciremos las siguientes sentencias:

![](./img/15.png)

```
    ssl on;
    ssl_certificate /etc/certs/reales/megacertificado.cer;
    ssl_certificate_key /etc/certs/reales/_.tudominio.com_private_key.key;

    #esto para el OCP
    ssl_stapling on;
    ssl_stapling_verify on;

    #esto es para el catch de sesion
    ssl_session_cache shared:ssl_session_cache:10m;

    #para el preload
    add_header Strict-Transport-Security "max-age=31536000;includeSubdomains;preload";

    #todo lo relativo a la seguridad del server
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/nginx/include/dhparam.pem;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256';
```

## 2.3. listen 443 ssl http2

Ahora vamos a ir, serverblock por serverblock, cambiando lo del “listen 80” por el 443 ssl http2, e incluyendo la sentencia del include

## 2.4. dhparams

Ahora, al hacer el systemctl restart nginx, veo que me sigue dando un error, que tiene que ver con que me falta el “dhparams” (una clave para los cifrados DH)

Vamos a la carpeta del include, y ejecuto el siguiente comando:

`openssl dhparam -out dhparams.pem 4096`

![](./img/16.png)

Nota: Antes de ejecutar este comando, ten en cuenta el disponer de media hora libre, porque es casi el tiempo que tardará...

Ahora pongo bien su ruta en el archivo de include.conf

![](./img/17.png)

# 3. Definir la CAA

Para definir la Autorización de la Autoridad Certificadora en Ionos...

![](./img/18.png)

![](./img/19.png)

# 4. Preloader

Solicitar el preload (es decir, solicitar que me metan en la lista que tienen los navegadores)

https://hstspreload.org/

![](./img/20.png)

![](./img/21.png)

Nota: Después de solicitar esto por el 2X/12/2021, me lo han concedido el 04/02/2022

![](./img/22.png)

# 5. SSL Labs

Comprobamos la seguridad de nuestro servidor a través de SSL LABS

https://www.ssllabs.com/ssltest/

![](./img/23.png)
![](./img/24.png)
![](./img/25.png)
![](./img/26.png)
![](./img/27.png)
![](./img/28.png)
![](./img/29.png)
![](./img/30.png)
![](./img/31.png)
![](./img/32.png)
![](./img/33.png)
![](./img/34.png)
![](./img/35.png)
![](./img/36.png)

Nota: Pone hsts preolading not in chrome, edge y firefox, porque cuando hice esta captura, mi servidor (dominio) aún seguía estando pendiente de validarse en las listas preloaders.

![](./img/37.png)

# 6. Examen - Teoría

## 6.1. Archivos necesarios en el servidor

![](./img/38.png)

- megaserverblock.conf
- tudominio.com.conf

Dentro del megaserverblock encontramos …

![](./img/39.png)

- listen 80 → que escucha por el puerto 80
- default_server → si me fueran hacer un ataque de DDOS, pues todas peticiones se redirigirían al servidor principal (este mismo)
- listen [::]:80 → si me hacen un telnet, también escucharía el puerto 80
- servername _ → cualquier dominio que no tenga un serverblock (en este
puerto)
- return 301 https://$host%request_uri → lo que puedan escribir en la barra de
navegación, se redirige como https (redireccionamiento genérico)

Dentro de mi serverblock principal, el de tudominio.com.conf …

![](./img/40.png)

Ese include.conf que se importa, dentro contiene lo siguiente:

![](./img/41.png)

![](./img/42.png)

## 6.2. HSTS

El HSTS, cuyas siglas significan HTTP Strict Transport Security, es una política de seguridad, en la cual **un servidor declara que un navegador solamente puede interactuar con él de forma segura** (HTTP conTLS/SSL) y así evitar que un atacante intercepte la comunicación (ataques man in the middle y secuestros de sesión) y pueda comprometer los datos del cliente.

Es decir, con HSTS, podemos indicar a los navegadores que sólo se deben comunicar con
nuestro dominio (web) usando HTTPS en lugar de usar HTTP.

¿Dónde viene dicho esto?
Esto viene declarado en los server blocks, donde antes tenía Listen 80, ahora tengo listen 443 ssl http2; Aparte de que en el archivo de include.conf tengo activado el ssl con “ssl_on;”

La política HSTS es comunicada por el servidor al navegador a través de un campo de la **cabecera HTTP de respuesta denominado "Strict Transport-Security"**. (ver archivo include … “add_header”).

La política HSTS también especifica un **período de tiempo durante el cual el agente de usuario (navegador) deberá acceder al servidor sólo en forma segura**. (ver archivo include … **“max-age”**).

Viendo esto, podemos darnos cuenta de que **el HSTS es en realidad un HTTP Header** más (Response Header). 

Cuando una aplicación web envía la política HSTS al agente del usuario, los que cumplan con el estándar se comportan de la siguiente manera:

- Los vínculos inseguros que hacen referencia a la aplicación web se convierten automáticamente en enlaces seguros. (Por ejemplo, http://example.com/some/page/ se modificará a https://example.com/some/page/ antes de acceder al servidor)

- Si la seguridad de la conexión no se puede asegurar (por ejemplo, el certificado TLS del servidor es autofirmado), muestran un mensaje de error y no permiten al usuario acceder a la aplicación web.

Enlaces de interés:

https://hstspreload.org/

https://www.ssllabs.com/ssltest/

## 6.3. HTTP Headers

Partiendo de la base de que, un protocolo HTTP transmite contenido mediante requests y responses entre navegador y servidor … podemos decir que …

**Los HTTP headers son** la parte central de **los HTTP requests y responses**, y transmiten información acerca del navegador del cliente, de la página solicitada, del servidor, etc.

Cada vez que visitas cualquier sitio, puedes ver los headers del request enviado:

![](./img/43.png)

La primera línea es la línea del request, que contiene su información básica (método HTTP, URL y versión). Lo demás son headers HTTP.

Una vez se envía ese request al servidor, éste responde con un HTTP response:

![](./img/44.png)

La primera línea es la barra de estado (versión HTTP y código de respuesta), seguido de los HTTP headers, hasta una línea vacía. Bajo la línea vacía se encuentra el contenido (en este caso contenido html).

Cuando le damos al inspeccionar del Chrome para ver el código de algún sitio web, los headers no aparecen, pero sí que se han recibido.

Para ver los headers se puede hacer mediante Firebug en Firefox, las Developer Tools de Chrome, curl, o con funciones de PHP (getAllHeaders() o $_SERVER para obtener los request headers y _headers_list()_ para obtener los response headers enviados o por enviar).

Los HTTP requests pueden envíar y acabar recibiendo muchas otras cosas, como imágenes, archivos CSS, archivos JavaScript, etc, es decir, que cuando se carga una página web se están enviando y recibiendo HTTP Requests y HTTP Responses por cada recurso.

Un HTTP request se compone de:

- Método: GET, POST, PUT, etc. Indica que tipo de request es.
- Path: la URL que se solicita, donde se encuentra el resource.
- Protocolo: contiene HTTP y su versión, actualmente 1.1.

Una vez que el navegador envía el HTTP request, el servidor responde con un HTTP response, compuesto por:

- Protocolo. Contiene HTTP y su versión, actualmente 1.1.
- Status code. El código de respuesta, por ejemplo: 200 OK, que significa que el GET request ha sido satisfactorio y el servidor devolverá los contenidos del documento solicitado. Otro ejemplo es 404 Not Found, el servidor no ha encontrado el resource solicitado.
- Headers. Contienen información sobre el software del servidor, cuando se modificó por última vez el resource solicitado, el mime type, etc. De nuevo la mayoría son opcionales.
- Body. Si el servidor devuelve información que no sean headers ésta va en el body.

## 6.4. CAA

Realmente llamada DNS CAA (**Autorización de Autoridad Certificadora**), es otra política de seguridad en la cual **los titulares de nombres de dominio, indican a las CA si están autorizadas a emitir certificados digitales para un nombre de dominio particular**.

Me acuerdo de que en IONOS, en el apartado de "Añadir Registro DNS", tuve que pulsar la opción de Añadir Registro CAA, y así poder definir un proveedor SSL para mi dominio, que como les compré el dominio a ellos, conjunto a un Certificado SSL, tal proveedor de SSL es una empresa que trabaja con IONOS.

![](./img/45.png)

![](./img/46.png)

Por lo tanto, podemos concluir con que, **las CAA son Registros DNS**, los cuales impiden que una persona con malas intenciones, configure un dominio para hacerse pasar por el nuestro, y evitar que esa persona pueda solicitar una certificación de una CA para ese dominio que falsifica el nuestro.

## 6.5. Preloaders

Básicamente, es una **lista que tienen los navegadores, los cuales dicen “este dominio va con esta IP”**, y al hacer eso, aunque haya alguien que esté falsificando el servidor DNS, se sabría que mi dominio tudominio.com pertenece sólo y exclusivamente a mi IP

Ejemplo:

instagram.com = 157.240.221.174

La historia original de los preloaders viene de que …

**La cabecera HSTS puede ser despojada por un atacante si se trata de la primera visita del usuario** a la web en cuestión.

El navegador Chrome intenta **limitar este problema mediante la inclusión de una lista de sitios HSTS "pre-cargada"**

Esta configuración, se haya en el archivo de “include.conf” que creé en /etc/nginx/include/ y tal archivo contiene una sentencia tal que así:

`add_header Strict-Transport-Security "max-age=31536000;includeSubdomains preload";`

(por un periodo determinado de tiempo, una página solo será accesible por SSL TSL)

**La directriz max-age (edad máxima) especifica cuánto tiempo debe estar disponible una web cifrada**. El período se define en segundos. Una max-age de 31536000 segundos corresponde a un periodo de un año.

Así, cuando un usuario visita por primera vez una página web asegurada con HSTS, el navegador recibe las siguientes instrucciones en el campo de cabecera Strict-Transport-Security:

- Todos los enlaces no cifrados deben ser reescritos en enlaces cifrados (http:// se convertirá en https://)
- Si no se puede establecer la seguridad de la conexión (por ejemplo, debido a un certificado no válido), esta se debe cancelar y el usuario recibe un mensaje de error.

Opcionalmente, **existe la opción de ampliar la información HSTS de los subdominios**. En este caso, se utiliza el campo de cabecera Strict-Transport-Security en combinación con la **directriz includeSubDomains**.

Esta le **indica al navegador que el encabezado HSTS** no solo **es válido** para el host actual (p. ej., www.example.com) si no **también para todos los subdominios** del dicho dominio (p. ej., blog.example.com o adserver.example.com).

`Strict-Transport-Security: max-age=31536000; includeSubDomains`

**La directriz preload permite marcar páginas web para la llamada preloading** para lidiar con el “first-visit-problem”.

`Strict-Transport-Security: max-age=31536000; includeSubDomains; preload`

Dicho todo esto, llegamos a la conclusión de que:

**Sin el parámetro preload, HSTS solo se aplicará a futuras visitas en la página web**: una vez el navegador conoce la información del encabezado HSTS de una web, los accesos posteriores se ejecutarán de la misma forma. Este mecanismo de seguridad no toma acciones en la primera visita a la página web.

Como consecuencia, **algunos proveedores de navegadores** como Google y Mozilla **ofrecen la posibilidad de inscribir páginas web en las llamadas listas de precarga** (preload lists). Así, las webs que hayan sido registradas para precargarse, estarán disponibles exclusivamente a través de HTTPS. Las listas de precarga se ejecutan centralmente y son enviadas al navegador del usuario cuando se realizan actualizaciones.

Debido a que HSTS solo funciona si una web ha sido visitada anteriormente al menos una vez con una conexión HTTPS no manipulable, **cada visita inicial es, en principio, susceptible a ataques de SSLstripping**.

Para evitar esto, en la actualidad, la mayoría de navegadores acceden a listas basadas en un sistema de precarga HSTS que es proporcionado por Google como parte del proyecto Chromium: https://hstspreload.org/

En principio, todo operador web puede introducir su nombre en la lista de precarga HSTS sin ningún coste. Ahora bien, la inclusión de una web a dicha lista depende de ciertos requisitos básicos que tienen como finalidad asegurar la calidad del sistema de control: https://www.ssllabs.com/ssltest/

1. Todas las webs deben contar con un certificado SSL válido.
2. Las direcciones HTTP deben ser redirigidas a direcciones HTTPS del mismo host.
3. Todos los subdominios (incluyendo el subdominio www) tienen que estar disponibles a través de HTTPS.
4. La cabecera HSTS debe entregarse sobre el dominio base cumpliendo con los siguientes parámetros:
- El valor para max-age debe ser al menos de ocho semanas (4.838.400
segundos).
- La cabecera HSTS debe contener los includeSubDomains.
- El HSTS debe incluir la directriz preload.
- Cualquier redirección adicional HTTPS de la web debe incluir el encabezado
HSTS.

Algunos de los usuarios de este servicio de precarga son Google, PayPal, Twitter y LastPass

## 6.6. OCSP Stapling

- Resumen:

El OCSP es un Protocolo de estado de certificado en línea, diseñado e implementado originalmente por los navegadores y CAs, para que los navegadores, cuando lo requieran, puedan solicitar el estado de revocación de un solo certificado emitido por la CA.

- Explicación larga:

En ciertos casos, un certificado debe marcarse como no válido (y la confianza en esos certificados debe ser revocado).

En los primeros días de SSL, el método utilizado por las CA era publicar su estado de revocación de certificados en documentos de acceso público llamados listas de revocación de certificados (CRL).

Para mitigar estos problemas de escalado, los navegadores y las CA diseñaron e implementaron Protocolo de estado de certificado en línea (OCSP). Las CA de confianza pública, como SSL.com, mantienen servidores HTTP simples llamados Respondedores OCSP.

Los navegadores ahora pueden contactar a un respondedor para solicitar el estado de revocación de un solo certificado emitido por la CA, en lugar de tener que adquirir y procesar toda la CRL.

Los respondedores de OCSP firman sus respuestas con la clave de firma privada de la CA, y los navegadores pueden verificar que el estado de revocación que recibieron fue realmente generado por la CA real.
