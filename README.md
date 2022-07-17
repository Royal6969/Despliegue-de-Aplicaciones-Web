# Despliegue_de_Aplicaciones_Web
https://lab.ldts.es/course/servidores-web-avanzado/

AWS - Amazon Linux 2

Descargar e instalar Putty
Descargar e instalar filezilla
AWS - crear instancia
Ionos - crear dominio

Censuras en imágenes - IP y dominio

Nota: Al principio, cuando asocié el dominio a la ip de la instancia, lo hice con la triple w delante del dominio (www.tudominio.com)

Nota: Es importante, sobretodo para el objetivo del Tomcat, deshabilitar el firewall interno del servidor

Nota: El objetivo del Tomcat está separado en dos partes (dos objetivos), el primero sería sacar un ejemplo por el puerto del tomcat (tusubdominio.tudominio.com:8080), y el segundo objetivo sería sacarlo otra vez con el Nginx proxy y cerrando el puerto 8080 (tusubdominio.tudominio.com)

Nota: cuando alguna vez experimentemos algún error que impida ver el despliegue de alguno de los objetivos tras hacer todos los pasos de la guía, recuerda comprobar los permisos y propietarios de los directorios y archivos que componen el proyecto del objetivo en cuestión.

Nota: cuando alguna vez experimentemos algún error que impida ver el despliegue de alguno de los objetivos cuando ponemos su URL en nuestro navegador tras algunos diás sin tocar la asignatura, recuerda comprobar con systemctl que los servicios que intervienen en dicho despliegue estén funcionando correctamente.
