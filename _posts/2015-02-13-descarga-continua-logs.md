---
layout: post
title: Descarga continua de logs
---

Si no disponemos de ningún sistema especializado en la recolección y análisis de logs centralizados, finalmente terminamos por usar simples ficheros de logs para nuestras aplicaciones. Puesto que es necesario que los desarrolladores puedan acceder a estos logs de sus aplicaciones cuando despliegan en distintos entornos, una práctica habitual suele ser aprovechar el mismo servidor web que hace de frontal para publicar los logs de la aplicación. Al hacer esto, podremos ver los logs de la aplicación simplemente navegando a una url concreta, y el navegador nos mostrará el contenido del fichero.

En la pestaña del navegador tendremos el contenido total del fichero de log en el momento que hemos introducido la url. Si la aplicación sigue escribiendo mensajes, tendremos que volver a refrescar el navegador, para volver a descargar su contenido y dirigirnos al final de la página para ver los últimos cambios.

Pero en determinados casos, estos ficheros de logs pueden ocupar varios megas, y si además estamos trabajando con una VPN donde las conexiones son lentas, la recargar completa puede llevar mucho tiempo, llegando a desesperar a algunos desarrolladores impacientes.

Un simple truco para solventar este problema es utilizar la característica que tienen los servidores web mediante la cuál nos permiten retomar una descarga que se ha interrumpido. Nuestro cliente informará al servidor que ya habíamos descargado un determinado número de bytes, y el servidor responderá con los datos a partir del punto indicado.

Usando esta característica aplicado a los logs, podremos ir solicitando sucesivamente al servidor web que continue la descarga del fichero log, que estará creciendo constantemente en el lado servidor mientras nuestra aplicación siga funcionado. De esta manera, lo que iremos será descargando incrementalmente este fichero de log.

En un terminal lanzaremos `wget -c` o `curl -C -` para descargar incrementalmente el fichero de log:

```bash
$ while true ; do wget -c http://servidor/ruta/fichero.log ; sleep 2 ; done
```

Cada dos segundos iremos incrementando el fichero de log con el nuevo contenido. Y en otro terminal podremos usar el comando `tail` como siempre para ir viendo su salida:

```bash
$ tail -f server.log
```

