---
layout: post
title: GMail, Etiquetas, Imap y Evolution
permalink: :year/:month/gmail-etiquetas-imap-y-evolution.html
---

Hace tiempo que gmail permite el acceso al correo a través de IMAP, permitiendo tener una copia de todos tus correos (o al menos los títulos) en cualquier ordenador con tu programa de correo. La configuración es muy sencilla y todo funciona de maravilla ... excepto las etiquetas.  

Gmail utiliza el concepto de etiqueta para agrupar tu correo, en lugar de carpetas, permitiendo que un correo tenga varias etiquetas. Desde el interfaz de gmail, esto es facil de utilizar y gestionar. El problema viene cuando lo integras con evolution por ejemplo. La forma de emular este compartamiento es crear un directorio por etiqueta.  

El inconveniente viene en que se crean en tu evolution las carpeta "Inbox", "Spam", y "All mail". Cuando recibes un correo y se le asigna a una etiqueta a través de un filtro (por ejemplo las listas de correos) ese correo aparece tanto dentro de la carpeta que reprsenta la etiqueta, como también dentro de "All mail". Por lo que aparecen duplicados los correos sin leer!  

Por lo que además de que tenemos siempre el correo duplicado, además la actualización de la carpeta "All mail" tarda bastante. Buscando encontré un truquillo, que si bien no es una solución definitiva a mi me viene de maravilla. El truco consiste en sustituir en los archivos de configuración de Evolution la carpeta "All mail" donde se guarda toda la configuración de la carpeta por un simple fichero. Ejemplo:  

~~~bash
cd ~/.evolution/mail/imap/CORREO@gmail.com@imap.gmail.com/folders/[Gmail]/subfolders  
rm "All mail" -rf  
touch "All mail"  
~~~

Con lo que sustituimos la carpeta interna de evolution "All mail" por un fichero con ese mismo nombre. Evolution al inciar e intentar actualizar esta carpeta no pordrá hacerlo, puesto que no es un directorio válido (ahora es un fichero!), por lo que no actualizará esta carpeta. Aqui vemos un ejemplo de correo en un carpeta/etiqueta pero no en "All mail":  

![](/assets/old_evolution_screenshot.png)

Si quisieramos volver a tener nuestra carpeta "All mail" simplemente borramos el fichero y reiniciamos Evolution. No es una solución muy elegante, pero al menos funciona bien ;)

