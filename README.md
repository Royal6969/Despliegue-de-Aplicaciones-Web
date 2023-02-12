# Despliegue de Aplicaciones Web

Hola compañero, bienvenido a mi repositorio de Despliegue de Aplicaciones Web, una de las asignaturas del segundo curso de nuestro grado superior en Desarrollo de Aplicaciones Web.

Durante todo el año, he invertido un gran número de horas para desarrollar este repositorio, el cual está compuesto por todos los apuntes que tomé, tanto para dejar detallado por escrito los pasos que iba haciendo, como para estudiar y repasar antes de realizar las pruebas finales del profesor.

Esta asignatura la conforman una serie de objetivos, los cuales suponen un avance secuencial y progresivo de los conocimientos y habilidades básicas y necesarias que necesita tener un desarrollador web como nosotros para poder desplegar aplicaciones en internet.

Estas aplicaciones web serán desplegadas en un servidor Linux, el cual crearemos y se albergará en la empresa de Amazon Web Services, y al que podremos conectarnos a través de la clásica aplicación de PuTTY. Por otro lado, compraremos un dominio propio en IONOS, y con él iremos generando los respectivos subdominios para cada una de las aplicaciones que vayamos desplegando.

## Roadmap

0. AWS - Ionos
1. Nginx
2. Tomcat
3. ApacheSSL
4. NginxSSL
5. Node.js
6. WS2012 - IIS
7. Headers
8. Email - SPF
9. Firma Digital

## Disclaimer

El esquema de los objetivos a realizar de este repositorio, tiene su origen en un curso de la empresa *ldtsynergy*, la cual tiene una sección de cursos online llamada *"Laboratorio LDTS de Blended Learning"*. Esta empresa está asociada a su vez al Colegio Altair, que es el centro docente donde he cursado la asignatura de DAW, y también cabe mencionar, que ha sido enteramente desarrollada por nuestro gran profesor, el cual nos impartía esta asignatura, utilizando de guía y esquema base este curso de "servidores web avanzados" de LDTS que veníamos comentando.

Con todo esto, quiero decir que, el alumno tiene la obligación de obedecer las reglas y normas, y debe seguir en todo caso y en todo momento, las directrices y órdenes impuestas por el profesor, para todo el desarrollo y progreso de este curso en esta asignatura, y por consecuente, conseguir y alcanzar el aprobado de la misma.

Este repositorio puede servir en algún caso al alumnado, como una ayuda extra y/o guía complementaria al curso del profesor en LDTS, pero en ningún caso y bajo ninguna circunstancia este repositorio debe convertirse en una guía que reemplace o sustituya al propio curso del profesor. Por otro lado, y teniendo en cuenta todo esto, yo (Sergio Díaz) no me responsabilizo de cualquier posible suspenso de un alumno que tan sólo haya seguido este repositorio para realizar toda la asignatura.

Como referencia directa, quisiera dejar el siguiente enlace con el que podrás dirigirte al curso del profesor que compone esta asignatura, y el cual tendrás que seguir y superar obligatoriamente para aprobar: https://lns130.ldts.us/

## Consideraciones y Notas

Es importante que usted lea las siguientes anotaciones a tener en cuenta antes de utilizar este repositorio.

Nota 1: El objetivo "0. AWS - Ionos" no constituye ningún objetivo en sí que sea evaluable por el profesor, si no que es la "puesta a punto" para poder empezar la asignatura, es decir, la creación del servidor Linux en AWS, y el registro y compra de un dominio en Ionos, y una vez que hayamos hecho eso, podremos comenzar realmente la asignatura y su materia, es decir, trata sobre preparar algunas cosas para poder partir desde un punto lo más cercano y parecido posible a como acabó la asignatura de Sistemas Informáticos el año pasado (en el primer curso).

Nota 2: Al principio, cuando asocié el dominio a la IP de la instancia, lo hice con la "triple w" delante del dominio (www.tudominio.com). Esta confusión, sólo la mantuve durante los tres primeros objetivos (hasta el ApacheSSL, inclusive), pero no te preocupes, porque encontrarás las imagenes y las notas necesarias para no caer en esta novata confusión.

Nota 3: Al principio, en vez de poner el portfolio en mi dominio principal, lo puse en un subdominio. Aunque en las imágenes, notas, y textos, lo he especificado lo suficiente para que tú no caigas en esta equivocación. El portfolio debe ir en nuestro dominio principal, y el wordpress en un subdominio (al igual que todas las aplicaciones que despleguemos, que cada una tendrá su subdominio).

Nota 4: Al principio, tanto el subdominio y serverblock del wordpress, los tenía nombrados como "wordpress" y a partir del cuarto objetivo del NginxSSL lo cambié a "wp" para ambas cosas.

Nota 5: Es importante, sobretodo para el objetivo del Tomcat, deshabilitar el firewall interno del servidor. Esto es debido a que el el servidor Linux, al estar albergado por AWS, ya cuenta con un tipo de firewall que se controla y administra a través de una interfaz de AWS llamada "grupo de seguridad", y si dentro del servidor Linux, activamos el firewall con el clásico CLI de "firewall-cmd", ambos entrarán en conflicto y tendremos algunos inconvenientes repetidas veces apara desplegar correctamente las apliaciones. El único firewall que se debe usar y que siempre está activo, es la propia interfaz del "grupo de seguridad" de la instancia de nuestro servidor Linux en AWS.

Nota 6: El objetivo del Tomcat está separado en dos partes (dos objetivos), el primero sería desplegar una app por el puerto del tomcat (tusubdominio.tudominio.com:8080), y el segundo objetivo sería desplegar otra vez la app, pero con el Nginx proxy y cerrando el puerto 8080 del Tomcat, de modo que el cliente no tenga que buscar nuestra web añadiendo el puerto al final (tusubdominio.tudominio.com). OJO, el puerto 8080 está censurado dentro de la red del Colegio Altair, de modo que para el Tomcat utilizaremos otro puerto como por ejemplo el 8079.

Nota 7: Cuando alguna vez experimentemos algún error que impida ver el despliegue de alguno de los objetivos tras hacer todos los pasos de la guía, recuerda comprobar los permisos y propietarios de los directorios y archivos que componen el proyecto del objetivo en cuestión, así como reiniciar el programa utilizado para el despliegue con el CLI de systemctl (systemctl restart *[programa]*).

Nota 8: Cuando alguna vez experimentemos algún error que impida ver el despliegue de alguno de los objetivos cuando ponemos su URL en nuestro navegador tras algunos diás sin repasar la asignatura, recuerda comprobar con el CLI de *systemctl* que los servicios que intervienen en dicho despliegue estén funcionando correctamente.

Nota 9: El objetivo final de la Firma Digital, también está subdividido en dos objetivos diferentes, por un lado, realizamos la firma digital de un documento pdf mediante un certificado "falso" que nosotros mismos habremos creado previamentea través de los comandos aprendidos en el objetivo del ApacheSSL, y por otro lado, realizamos la firma digital de un documento pdf mediante un certificado avalado por el Gobierno a través de la Fábrica Nacional de Moneda y Timbre (para el cual tendremos que realizar previamente una serie de pasos en la Administración Pública para acreditar nuestra identidad como persona física).

Nota 10: A medida que se vaya avanzando y progresando en los diferentes objetivos del curso, y sobretodo a partir del objetivo del NginxSSL (que es donde se empieza a tocar el tema de la ciberseguridad), podremos encontrar con más frecuencia, algunos apartados que contienen solamente teoría. Es muy importante que usted no prescinda de la lectura, comprensión y estudio de estos apartados de teoría, ya que también es una información muy valiosa, la cual ha sido investigada, cotejada y extraída de múltiples fuentes web, y mediante la necesidad de superar sin problemas las pruebas finales del profesor en cada uno de los objetivos.

## GNU Free Documentation License

Copyright (C)  2023  Sergio Díaz Campos.
    Permission is granted to copy, distribute and/or modify this document
    under the terms of the GNU Free Documentation License, Version 1.3
    or any later version published by the Free Software Foundation;
    with no Invariant Sections, no Front-Cover Texts, and no Back-Cover Texts.
    A copy of the license is included in the section entitled "GNU
    Free Documentation License".
