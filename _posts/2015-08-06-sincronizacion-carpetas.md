---
layout: post
title: Sincronización de carpetas
---

En ocasiones nos vemos en la necesidad de tener que modificar ficheros en servidores remotos. Servidores de los que sólo disponemos de conexión `ssh` para acceder normalmente. Cuando se trata de realizar modificaciones puntuales sobre unos pocos archivos, como pueden ser ficheros de configuración o propiedades, es suficiente con iniciar una sesión `ssh`, y utilizar el editor de consola disponible en el servidor, como puede ser `vi`, `nano` o `emacs`. El problema se complica cuando necesitamos realizar estas modificaciones constantemente, por ejemplo, cuando queremos desarrollar contra un servidor remoto.

Imaginemos que en el equipo del trabajo, tenemos una carpeta con algún proyecto web escrito en `php`, `python` o `Ruby on Rails`. Si en otro momento, estamos trabajando remotamente desde nuestras casas y nos vemos en la necesidad de seguir desarrollando en este proyecto tenemos varias opciones:

1. Conectarnos gráficamente al ordenador remoto y seguir utilizando nuestro entorno de desarrollo habitual. Esto lo conseguimos con herramientas como *VNC*, *TeamViewer*, redirigiendo las X, usando *[Xpra]*, etc. El problema con esta opción es que suele ser lento (por limitación del ancho de banda, latencia, uso de *VPN's*). Cuando programamos o usamos un IDE queremos que responda instantáneamente a nuestras pulsaciones y atajos de teclado, pero con estos sistema siempre se suele incluir un pequeño retardo que nos puede desesperar.
1. Montar el proyecto en el ordenador de casa. Si no disponemos de sistemas como *Dockers* o *Valgrant*, esto requerirá tener que instalar todas las dependencias que necesite el proyecto web. Esto puede ser complejo y requerirá instalar en el ordenador de casa las dependencias que se necesite (módulos python, gemas de ruby, librerías del sistema, etc). Si ademas dependemos de otros servicios (bases de datos, servicios web, comandos de consola) tendremos que instarlos localmente o comenzar a mapear puertos para poder acceder a ellos. Esto se complica aún mas si los sistemas operativos del trabajo y de casa son distintos (Linux vs Mac o incluso Debian 8 vs Ubuntu 14.10).
1. Conectarnos por `ssh`. Nuestro equipo del trabajo (o servidor remoto) ya está configurado y preparado para hacer funcionar el proyecto. Sólo debemos conectarnos, acceder a la carpeta del proyecto y realizar las modificaciones desde consola. El problema es que si no somos unos expertos en `vi` o `emacs`, si tenemos que estar modificando varios ficheros (abriendo y cerrando `vi`), navegar o buscar dentro del proyecto (usando comandos `grep` y `find`), volvemos a encontrarnos con la lentitud a la hora de programar con un entorno de desarrollo que no es el nuestro habitual.
1. Montar remotamente el directorio por cifs/samba/sftp/sshfs. En esta opción seguimos accediendo por `ssh` para lanzar la compilación o reiniciar el servidor web, pero la modificación y navegación de los ficheros del proyecto la podemos hacer con alguna aplicación desde nuestro ordenador de casa. Podríamos montar remotamente la carpeta del proyecto y abrirla con [Aptana], [Eclipse] o [SublimeText] de nuestro ordenador de casa. Pero una vez mas la lentitud es el gran impedimento. Suelen ocurrir retardos al abrir los ficheros y sobre todo si intentamos buscar en el contenido de estos.

## RSYNC

Existe otra opción adicional, que es la que mas utilizo últimamente. Por un lado tenemos la magnífica herramienta `rsync`. Este comando permite mantener sincronizados dos estructuras de carpetas de forma eficiente. Esta herramienta tiene multitud de opciones, ya que puede que nos interese o no, sincronizar también las fechas de los ficheros, los permisos, los propietarios, los enlaces simbólicos, borrar ficheros del destino, guardar los cambios en una tercera carpeta, etc. Así que por ejemplo, si en una misma máquina queremos tener sincronizada dos directorios escribiríamos lo siguiente:

~~~bash
$ rsync -navh /path/origen /path/destino
~~~

Siempre debemos tener mucho cuidado con este comando, y prestar mucha atención a la carpeta destino. De hecho la opción `-n` del ejemplo es la opción `--dry-run` que realmente no hace modificaciones, solo nos informa sobre que operaciones que se van a realizar y que siempre deberíamos probar antes de lanzar el comando de forma final ... para evitar dolorosas equivocaciones.

También existe la posibilidad de utilizar este comando para sincronizar ficheros entre dos máquinas, para ellos debemos indicar con la opción `-e` qué comando se debe utilizar para llegar a la máquina remota. Lo mas habitual será utilizar `ssh` por lo que el comando quedaría de la siguiente forma:

~~~bash
$ rsync -navh -e ssh /path/origen usuario@servidor1:/path/destino
~~~

Llegados a este punto, lo único que tendríamos que hacer es copiarnos inicialmente la carpeta remote a nuestro ordenador local, y comenzar a editarlo con nuestra herramienta favorita, como puede ser [SublimeText]. Una vez hayamos realizado modificaciones podemos ejecutar el comando anterior y enviaremos las modificaciones realizadas en nuestro equipo local, hacia el servidor en el directorio destino.

## Mejorando el flujo de trabajo

Con lo detallado anteriormente, la forma de trabajar por ahora sería la siguiente:

1. Copiamos en nuestro ordenador local el proyecto remoto con el que queremos trabajar (con un simple scp o utilizando rsync en el sentido inverso).
2. Comenzamos a editar de forma local el proyecto con nuestro IDE.
3. Cada vez que queramos probar algo, ejecutamos el comando rsync.
4. Una vez terminada la ejecución de rsync, reiniciamos el servidor remoto (webrick, apache, tomcat, jboss, node o lo que sea) si es necesario.

Pero aún podemos mejorar esta forma de trabajar, eliminando el tercer paso, en el que somos nosotros los que manualmente tenemos que lanzar el comando rsync periódicamente.

Para esto podemos utilizar la utilidad [fswatch-rsync]. Esto es un script en bash, que combina el uso de rsync con [fswatch]. fswatch es una utilidad que utiliza inotify/kqueue/poll que permite monitorizar los eventos producidos en un directorio. De esta forma, este script bash está preparado para detectar los cambios realizados en un directorio (que será nuestro proyecto), e invocará rsync indicándole además qué ficheros son los que han cambiado, permitiendo a rsync funcionar de forma aún más rápida y optima, ya que no necesitará analizar los ficheros del directorio origen y destino para decidir qué cambios hay que realizar. Además permite ignorar algunos eventos, como pueden ser los producidos en los directorios .git o .svn.

De esta forma, podemos trabajar de forma muy cómoda con proyectos en servidores remotos implementando un [sistema de sincronización de baja latencia][low-latency-rsync].


[Xpra]: https://www.xpra.org
[Aptana]: http://www.aptana.com
[Eclipse]: http://www.eclipse.org
[SublimeText]: http://www.sublimetext.com/3
[fswatch-rsync]: https://github.com/aalto-ics-kepaco/fswatch-rsync
[fswatch]: https://github.com/alandipert/fswatch
[low-latency-rsync]: http://www.danplanet.com/blog/2012/05/09/low-latency-continuous-rsync/
