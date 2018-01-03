---
layout: post
title: Marco de fotos interactivo con Raspberry Pi
---

La [Raspberry Pi][raspberry] es la plataforma ideal para dar rienda suelta a nuestra imaginación y poder crear proyectos de todo tipo. Usarlo como [centro de control][magpi] para todo tipo de artilugios robóticos, usarlo como NAS o nube personal con [Syncthing] o [ownCloud], [compartir una impresora en red][printer], crear un altavoz inalámbrico con [Pi MusicBox][pimusicbox], convertir cualquier televisor en un mini ordenador con [rasbpian], usarlo como centro multimedia con [Kodi], o permitirte jugar a juegos retro con [RetroPie].

En mi caso por ahora se ha reducido al último caso, aprovechando un televisor que teníamos en el jardín y las Raspberry Pi como centro de juegos retro con RetroPie para las tardes de verano (Super Bomberman y Micro Machines 😍 ). 

![](/assets/superbomberman.gif)

Aunque cuando celebrábamos algún cumpleaños, no sacábamos la Raspberry Pi  para evitar vicios y destrozos con tanto niño suelto, por lo que dejábamos la tele siempre apagada. Pensando que uso le podríamos dar se nos ocurrió que podríamos mostrar una selección de fotos del cumpleañero.

## Marco de fotos

Teniendo en cuenta que la Raspberry Pi ya estaba configurada con RetroPie, y que RetroPie utiliza framebuffer en lugar de X-Windows, busqué alguna utilidad sencilla que permitiera mostrar imágenes como un carrusel de fotos. Íbamos a convertir el televisor en un gran marco de fotos. El comando en cuestión es [`fbi`][fbi] "Linux framebuffer imageviewer".

RetroPie usa la aplicación [EmulationStation] como interfaz gráfico para navegar por las distintas consolas y juegos. En una de los menús tenemos acceso a múltiples utilidades que nos permiten configurar distintos aspectos del sistemas. Todas estas opciones no son mas que scripts de configuración que existen en el directorio `/home/pi/RetroPie/retropiemenu`. Agregaremos en este menú una nueva opción para iniciar nuestro carrusel de fotos.

Crearemos un script con permisos de ejecución en este directorio con nuestro comando, por ejemplo:

~~~bash
#!/bin/bash

fbi -noverbose -blend 1000 -a -t 2 imagen1.jpg imagen2.jpg imagen3.jpg ...
~~~

El script anterior, mostrará de forma indefinida las imágenes indicadas, sin información adicional (`-noverbose`), con un fundido entre imágenes de 1 segundos de duración (`-blend`), autoajustando el tamaño de las imágenes (`-a`) y dejando 2 segundos entre cada imagen (`-t`).

[]() gif animado seleccionando la opción

## Compartir fotos

Tener un conjunto de fotos seleccionadas que se muestran está bien, pero más interesante es permitir a los invitados interactuar con lo que están viendo en la televisión y poder compartir con todos sus propias fotos, sobre todo las que puedes hacer en ese mismo momento durante la celebración!

Para ello crearemos una aplicación web en la propia Raspberry Pi, que mostrará a nuestros usuarios una simple página web que les permita subir una foto de sus móviles. Las fotos las iremos guardando en una carpeta dentro de las Raspberry Pi y con un script similar al anterior se irán mostrando en la pantalla. Por lo que nuestra solución constará de dos partes un script en bash que muestra las fotos y una aplicación web realizada en python en este caso (aprovechando que ya viene instalado).

Puesto que el comando `fbi` anterior necesita una lista fija de fotos, lo que haremos es crear un bucle y recalcular la lista de las últimas fotos e invocar al comando `fbi` con la opción `-1` para que una vez mostrada las fotos termine su ejecución. Crearemos ademas una condición para permitirnos salir de este bucle que muestra fotos por si queremos volver al menú de retropie.

~~~bash
while [ ! -f $STOP_FILE ] ; do
    ls -1t images/* | head -n 25 > | sort -R > $IMAGE_LIST
    fbi -1 -noverbose -a -t 8 logo.png > /dev/null 2>&1
    fbi -1 -noverbose -blend 1000 -a -t 2 -u -l $IMAGE_LIST > /dev/null 2>&1
done
~~~

Lo primer que hacemos dentro del bucle es obtener las 25 fotos mas recientes y las ponemos en orden aleatorio (sort -R). Mostramos durante 8 segundos la imagen `logo.png` (que habremos preparado con información del evento, la url para subir fotos, etc.). Por último mostramos la lista de fotos.

## Servidor web en python

Python ya incluye de serie un servidor web básico que podemos utilizar para compartir la carpeta actual en la dirección `http://localhost:8080` con el comando: 

~~~bash
python -m SimpleHTTPServer 8080
~~~

Para evitar hacer uso de librerías de terceros como Flask, Django o Pyramid, podemos simplemente extender la funcionalidad de este módulo para procesar la subida de los ficheros por el método `POST`.

~~~python
class PhotoHandler(SimpleHTTPRequestHandler):
    def do_POST(self):
		    # Guardar la imagen subida por el usuario
		    ...
		    
httpd = SocketServer.TCPServer(("", 8080), PhotoHandler)
while True:
    httpd.handle_request()
~~~

Por último creamos una sencilla página web que será lo que verán nuestros invitados cuando quieran compartir una foto.

![](/assets/photoframepi-upload.gif)

## Publicarlo

A la hora de publicarlo o hacerlo accesible tenemos varias opciones según la disponibilidad de dominios y el nivel de privacidad que queramos.

- Podemos obligar a nuestros invitados a que se conecten a la red wifi del lugar y utilizar la ip local para acceder, al estilo http://192.168.0.X:8080.
- Si disponemos o registramos un dominio, podemos configurar la opción de iframe (o configurar un cname) y poner como destino la ip interna anterior. De esta manera, el acceso será mediante un dominio reconocible "cumple-de-pepito.es" pero sólo funcionará si estamos conectados a la red wifi del evento.
- Podemos mapear un puerto del router para exponer el servicio a internet y configurar nuestro dominio a la ip pública del router.

## Usar un servicio externo de imágenes

En casos de muchas peticiones, me ha ocurrido que las Raspberry Pi se ha reiniciado, probablemente debido a que el cargador utilizado no suministraba la suficiente potencia, mostrándose a veces el cuadrado multicolor en la esquina superior derecha. Además, si no queremos ofrecer ni obligar a los invitados que se conecten al wifi existe otra opción. Podemos utilizar un servicio de almacenamiento de imágenes externo como por ejemplo [cloudinary].

Cambiaremos el servidor web anterior por un proceso en python que use la api de cloudinary para descargar las nuevas fotos subidas cada 10 segundos por ejemplo:

~~~python
CLOUDINARY_API = 'https://ID:KEY@api.cloudinary.com/v1_1/gimco/resources/image?max_results=500&direction=asc&start_at=%s'
created_at = '2017-01-01'

while True:
    try:
        url = CLOUDINARY_API % created_at
        print str(datetime.now()), 'Buscando nuevos ficheros > ', created_at 
        response = urllib.urlopen(url)
        data = json.loads(response.read())
        for resource in data['resources']:
            image = 'images/' + resource['public_id'] + '.jpg'
            url = resource['url']
            created_at = resource['created_at']
            if not isfile(image):
                print 'Downlading ' + image
                urllib.urlretrieve(url, image)
    except: print 'Error!', sys.exc_info()
    print 'Fin de comprobacion, esperando 10 segundos'
    time.sleep(10)
~~~

Además, deberemos subir el contenido de la carpeta web con nuestra página html al espacio estático disponible con nuestro dominio para que sea accesible a los invitados. También deberemos cambiar la dirección a la que subir la imagen por la url del servicio de cloudinary:

~~~diff
@@ -12,13 +12,14 @@
         <p>Subiendo imagen ...</p>
         <img src="img/spinner.gif"/>
       </div>
-      <form id="image-form" ENCTYPE="multipart/form-data" method="post">
+      <form id="image-form" ENCTYPE="multipart/form-data" method="post" action="https://api.cloudinary.com/v1_1/gimco/image/upload">
         <p>Pulsa sobre el icono de abajo y selecciona una foto para subir.</p>
         <p> 😁 🎉 🎁 🎂 🍰 🎊 😛 </p>
         <p>¡En un minuto la podremos ver en la televisión!</p>
         <div class="image-upload">
           <label for="file"><img src="img/camara.png"/></label>
           <input id="file" name="file" type="file" accept="image/*" onchange="uploadImage()" />
+          <input type="hidden" name="upload_preset" value="brunito" />
         </div>
       </form>
     </div>
~~~

Podremos configurar ademas en cloudinary mas opciones a la hora de subir las imágenes (upload_preset), pudiendo redimensionar, rotar y convertir las imágenes a JPEG.

Puedes consultar los [fuentes del proyecto][github] en github.

[raspberry]: https://www.raspberrypi.org
[magpi]: https://www.raspberrypi.org/blog/build-remote-control-robot-magpi-51/
[Syncthing]: https://syncthing.net
[ownCloud]: https://owncloud.org
[pimusicbox]: http://www.pimusicbox.com
[printer]: https://www.makeuseof.com/tag/make-wireless-printer-raspberry-pi/
[rasbpian]: https://www.raspbian.org
[Kodi]: https://kodi.tv
[RetroPie]: https://retropie.org.uk
[fbi]: http://manpages.ubuntu.com/manpages/xenial/man1/fbi.1.html
[EmulationStation]: http://www.emulationstation.org
[cloudinary]: https://cloudinary.com
[github]: https://github.com/gimco/photoframepi