---
layout: post
title: Compartiendo archivos entre linux y Leopard
tags: apple, compartir, ficheros, finder, linux, macosx
permalink: /:year/:month/compartiendo-archivos-entre-linux-y.html
---

La forma más sencilla de compartir archivos entre un sistema linux y MacOS X, parece ser AppleTalk, que es el protocolo utilizado por Apple para compartir archivos e impresoras en redes locales. Vendría a ser equivalente de SMB en redes windows. De la misma forma que tenemos Samba, para acceder a redes SMB, existe "netatalk" para comunicarnos con redes AppleTalk. Solo debemos instalar dicho paquete:  

~~~bash
$ sudo apt-get install netatalk  
~~~

En mi caso, quería conectar mi portatil con mi iMac, a través de un cable de red. Por lo que por seguridad solo habilité este servicio a través del interfaz de red, en mi caso eth0\. Para hacer esto solo hay que añadir el interfaz "eth0" en el fichero /etc/netatalk/atalkd.conf. Después de esto reiniciamos el servicio:  

~~~bash
$ sudo /etc/init.d/netatalk restart  
~~~

Una vez hecho esto, nos vamos a Leopard, y desde Finder seleccionamos Ir->Conectar al servidor. Introducimos la IP de nuestra máquina linux, y nos preguntará por usuario y contraseña. La introducimos y lo más probable es que no funcione :)  

Parece ser que esto es debido a que desde Leopard, no se permite la autenticación en plano en el protocolo AppleTalk, si no que la autenticación se hace ahora por defecto mediante SSL. El problema es que la mayoría de las distribuciones, no activan en el paquete "netatalk" dicha opción. Así que tenemos dos opciones, o recompilamos el paquete con opción SSL en la autenticación, o lo más fácil, desactivar esta opción en Leopard, y obligar a Leopard que utilice autenticación en plano. En mi caso no hay ningún problema de seguridad, puesto que era una conexión directa desde el portatil al iMac. En otro caso, si que habría que recompilar netatalk por seguridad.  

Para obligar a Leopard a que utilice autenticación en plano para el protocolo AppleTalk, solo debemos ejecutar esto desde una consola en nuestro MacOS X:  

~~~bash
$ defaults write com.apple.AppleShareClient afp_cleartext_allow -bool true  
~~~

Volvemos a ir a Finder -> Ir -> Conectar al servidor. Introducimos un usuario y contraseña del sistema linux y ya estaríamos dentro. Por defecto se comparte la carpeta hogar del usuario. Podemos especificar más volúmenes a compartir editando el fichero /etc/netatalk/AppleVolumes.default.  

También podemos acceder a los recursos compartidos, accediendo al entorno de red desde Finder (Finder -> Ir -> Red). Aunque tarda un poco, en que se visualice los recursos compartidos.  

También existen otros métodos para compartir archivos, por ejemplo utilizando Samba, o sshfs. Para más referencias [http://sethbc.org/2008/02/24/leopard-afp-and-the-hardy-heron/](http://sethbc.org/2008/02/24/leopard-afp-and-the-hardy-heron/)