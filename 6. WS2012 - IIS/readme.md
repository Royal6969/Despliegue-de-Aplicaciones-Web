# Tabla de Contenidos

<!-- TOC -->

- [1. Crear la instancia](#crear-la-instancia)
- [2. Desplegar un proyecto con IIS](#desplegar-un-proyecto-con-iis)
    - [2.1. Instalación del IIS](#instalaci%C3%B3n-del-iis)
    - [2.2. Proyecto de ASP.net](#proyecto-de-aspnet)
    - [2.3. Entramos en el IIS](#entramos-en-el-iis)
    - [2.4. El dashboard del IIS](#el-dashboard-del-iis)
    - [2.5. Permisos](#permisos)
    - [2.6. Default Document](#default-document)
    - [2.7. ASP.NET Core Runtime](#aspnet-core-runtime)
    - [2.8. Comprobación local](#comprobaci%C3%B3n-local)
    - [2.9. Nuevo server block y subdominio](#nuevo-server-block-y-subdominio)
    - [2.10. Cerrar el puerto del WS](#cerrar-el-puerto-del-ws)
    - [2.11. Comprobación final](#comprobaci%C3%B3n-final)

<!-- /TOC -->

# 1. Crear la instancia

Lo primero de todo será dirigirnos a AWS para crear una instancia ec2 de un Windows Server 2012.

Nota: Aunque la creación de esta instancia está dentro de la capa gratuita de AWS, es posible que algunos factores (como superar el límite máximo de horas gratis entre las instancias) puedan acabar suponiendo un coste adicional en nuestro medio de pago en AWS, de ahí la importancia de mantener apagada esta instancia cuando no estemos haciendo este objetivo y sólo encenderla cuando nos pongamos con ello o cuando tengamos que presentarle el resultado final al profesor.

Nota: En la siguiente imagen aparece marcada la instancia de Windows Server 2022, pero la que yo usé y la que tú tienes que seleccionar, es la versión de 2012 (que es la que marca el profesor en su programa)... lo que te quiero decir, es que puedes crear la instancia con la versión de 2022 si quieres, ya que todas son prácticamente iguales sólo que con algunas ciertas mejoras, pero si te surge algún error o inconveniente que yo no haya experimentado en la versión de 2012 o que yo no haya recogido en esta guía, tendrás que arreglártelas por tí mismo...

![](./img/1.png)

![](./img/2.png)

![](./img/3.png)

![](./img/4.png)

![](./img/5.png)

![](./img/6.png)

Una vez que ya hemos creado la instancia de WS2012, le creamos también una IP elástica y se la asociamos a esta nueva instancia (a su IP pública).

Y ahora ya podemos comenzar la ejecución de la instancia, y entramos en ella (nos conectamos) con el clásico programa de nuestro windows de “Conexión a Escritorio Remoto”, para el cual necesitaremos descargar un archivo desde AWS.

![](./img/7.png)

![](./img/8.png)

![](./img/9.png)

![](./img/10.png)

![](./img/11.png)

![](./img/12.png)

![](./img/13.png)

![](./img/14.png)

# 2. Desplegar un proyecto con IIS

Internet Information Server (o IIS por sus siglas en inglés) es un servidor web extensible que provee un conjunto de servicios para sistemas operativos Windows.

## 2.1. Instalación del IIS

Para instalar el IIS he seguido los pasos de esta guía: https://enterprise.arcgis.com/es/web-adaptor/latest/install/iis/enable-iis-2012-components-server.htm

## 2.2. Proyecto de ASP.net

Lo siguiente que necesito es crear un proyecto de ASP.net, pero para crear este tipo de proyectos necesitamos instalarnos el plugin de ASP.net en el Visual Studio 2022, el cual puede llegar a ocupar 5Gb más o menos (ojo, todo esto en VisualStudio, no VisualCode).

Una vez que lo tenemos instalado, en la pantalla inicial de Visual Studio nos aparecerá la opción de crear un proyecto de ASP.net

Haremos una prueba sencilla, y a la plantilla de base que viene cuando creamos el proyecto, le añadimos un for() con algún mensaje dentro de él.

![](./img/15.png)

![](./img/16.png)

![](./img/17.png)

![](./img/18.png)

## 2.3. Entramos en el IIS

Ahora tenemos que conectarnos a nuestros WS2012 y abrir el Server Manager, para a su vez, abrir el IIS.

![](./img/19.png)

## 2.4. El dashboard del IIS

Vamos a conocer un poco más de cerca el dashboard del IIS para familiarizarnos con él.

![](./img/20.png)

- A la izquierda, tenemos la navegación del IIS, donde, si nos situamos en nuestro servidor WS2012, veremos unas opciones de configuración generales de diferentes funciones del IIS (para todos nuestros proyectos), y a la derecha, MUY IMPORTANTE, el botón de RESTART, el cual hará la función equivalente a la que en Linux fuese el “systemctl restart iss”.

- Si en la barra de navegación, le hacemos click derecho a nuestro servidor, encontraremos una opción de “add website”.

![](./img/21.png)

- Si pulsamos sobre esta opción, se nos abre un cuadro de diálogo en el cual, debemos detallar cierta configuración básica para nuestro nuevo proyecto.

![](./img/22.png)

Nota: Para haber especificado ese Phisical Path, obviamente antes hemos tenido que subir el proyecto a nuestro WS2012 y guardarlo, por ejemplo, en el disco C:\ , pero, ¿y cómo subimos un proyecto al WS2012? ... en una nota de un poco más abajo te explico el truco...

![](./img/23.png)

Aparte de saber cómo subir un proyecto al WS2012, también debemos tener en cuenta que, Internet Explorer no funciona ya en WS2012, así que... ¿cómo vamos a ver si el despliegue del proyecto va funcionando? Necesitaríamos instalar un navegador...

Es en este momento cuando aprendemos el “MotoTruco”. Resulta que el ClipBoard de windows permite copiar archivos y carpetas, así que dicho esto, copiamos la carpeta de app2 en nuestro windows original donde trabajamos y en el que creamos el proyecto de ASP.net, y vamos al WS2012 para pegarlo en el disco C:\ ... y funciona!

Del mismo modo podemos instalar Google Chrome en WS2012. En nuestro ordenador, descargamos el ejecutable (instalador) de Chrome, lo copiamos, y nos vamos al WS2012 para pegarlo en el escritorio, y allí lo ejecutamos para instalarlo... y funciona perfectamente!

Nota: Para no tener que instalar los 5 Gb del plugin de ASP.net en nuestro ordenador para VS, te dejo en un enlace de Drive, para que descargues directamente el proyecto que he usado para la prueba de toda esta explicación.

https://drive.google.com/drive/folders/1KHRxOgACAQk6mMtMP3IBgOCLMrLcWwG0?usp=sharing

## 2.5. Permisos

Un detalle muy importante que debemos tener en cuenta es, cambiar los permisos de la carpeta de nuestro proyecto (app2). Debemos incluir el grupo de IIS-USERS y otorgarle el FullControl.

![](./img/24.png)

![](./img/25.png)

Nota: A cada paso que hagamos, y por consiguiente, después de cada cambio que realicemos, debemos reiniciar el IIS, para asegurarnos que los cambios vayan surtiendo efecto.

![](./img/26.png)

## 2.6. Default Document

El siguiente paso será definir el DefaultDocument de nuestro proyecto (el archivo.dll).

Nota: ojo, lo hacemos desde las opciones del panel de app2.

![](./img/27.png)

![](./img/28.png)

Nota: es importante que tengamos en cuenta que, después de haber creado el proyecto en el IIS, siempre podremos modificar la configuración que le dimos inicialmente, a través del botón de la navegación derecha, “BasicSettings”, el cual podremos encontrar desde la carpeta Sites del IIS. (en este ejemplo, te demuestro que puedo cambiar los campos, como el Path, por ejemplo).

![](./img/29.png)

![](./img/30.png)

Nota: Empezamos haciendo algunas comprobaciones sobre si conseguimos reproducir (desplegar) nuestro proyecto en el Chrome del WS2012, y para ello, en el Firewall de AWS (en el grupo de seguridad del WS2012), le abrimos un puerto tipo TCP y elegí el 8070 (especificándolo sólo para la IP del WS).

Una vez lo hayamos conseguido de esta forma, finalmente acabaremos cerrando el puerto 8070, para hacerle un proxy_pass para que el IIS funcione a través del Nginx de nuestro servidor Linux, y de esta forma, hacerlo más seguro.

¿Te acuerdas de cuando hicimos esto con Tomcat y con Node? Pues es más de lo mismo...

![](./img/31.png)

## 2.7. ASP.NET Core Runtime

El siguiente paso es muy sencillo, ya que tan sólo consiste en que tenemos que instalar desde/en nuestro WS2012, el ASP.net Core Runtime, ya que sin ésto, nada de lo que hemos hecho funcionaría, y no se podría hacer efectivo el despliegue de nuestro proyecto.

![](./img/32.png)

![](./img/33.png)

## 2.8. Comprobación local

Ahora ya tan sólo tenemos que hacerle un Restart al IIS, y si nos vamos al Chrome del WS2012, y ponemos en la URL el localhost con el puerto 8070... vemos que funciona!

![](./img/34.png)

Nota: Aparte de que podemos manejar el IIS en general con Start, Stop, y Restart, debemos saber que estas opciones también están disponibles para cada proyecto, con lo cual podemos activar o desactivar cualquier proyecto en cualquier momento de manera fácil y rápida cuando lo necesitemos.

## 2.9. Nuevo server block y subdominio

Ahora pasamos a la parte final de este objetivo, el verdadero despliegue de nuestro proyecto en internet... con un redireccionamiento hacia Nginx...

Esto ya nos suena familiar de los objetivos anteriores, y es volver a hacer más de lo mismo, así que, resumiremos los pasos básicos a seguir:

- crear un nuevo subdominio en IONOS para el proyecto de "app2" y ajustar su destino con un registro dns tipo A que apunte a la ip de nuestra instancia ec2 Linux
- crear un nuevo server block en el directorio de configuración de Nginx en nuestra instancia de Linux
- para redirigir el despliegue del IIS hacia el Nginx, ponemos la sentencia del proxy_pass en el server block (con la ip del WS)

```
server {
   listen 443 ssl http2;
   server_name ws-iis-app2.tudominio.com;

   include /etc/nginx/include/include.conf;

   location / {
      proxy_pass http://laIpPublicaDeTuWindowsServer:80;
   }
}
```

![](./img/35.png)

![](./img/38.png)

![](./img/36.png)

Nota: No olvidarnos de reiniciar Nginx después de crear el nuevo server block.

## 2.10. Cerrar el puerto del WS

Por último, la "prueba de fuego" de siempre, cerrar el puerto 8070 del Firewall del WS2012 (a través del grupo de seguridad de la instancia ec2 del WS2012 en AWS). 

Nota: Además, debemos cambiar el origen del puerto 80 hacia la IP de nuestra instancia ec2 Linux.

El grupo de seguridad de nuestra instancia WS2012 debe quedar así:

![](./img/37.png)

## 2.11. Comprobación final

Ahora vamos a probar a poner en el Chrome de nuestro ordenador el nuevo subdominio... y comprobamos que funciona perfectamente! Proyecto desplegado con éxito y objetivo conseguido!

![](./img/39.png)
