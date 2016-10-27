---
layout: post
title: "Retropie: Crea tu retroconsola"
---

Hace unos meses Nintendo anunciaba una nueva consola destinada a tocar la fibra sensible de todos los ochenteros que hemos tenido la suerte de haber conocido la maravillosa generación de consolas Atari/MasterSystem/Nes/MegaDrive/SuperNintendo. Se trata de la [Nintendo Classic Mini][nintendo-classic] una réplica en miniatura de la [NES][NES] con una selección de 30 juegos, simples, espartanos, pero super adictivos.

![](/assets/nes-classic-edition-in-hand.png)

No tengo la menor duda de que será un superventas estas navidades. Aunque podríamos señalar dos inconvenientes: Solo permiten dos jugadores y los juegos son exclusivamente los 30 elegidos.

- ¿Habrá que esperar a una Super Nintendo Classic Mini para poder volver disfrutar del [Yoshi’s Island][yoshi-island] o del Street Fighter II?
- ¿No podré utilizar mi [mando bluetooth retro de NES][8bitdo-nes]? 
- ¿Tampoco tardes de cuatro jugadores con Super Bomberman, Micro Machines o Mario Kart?

Para resolver estos inconvenientes siempre podemos crear nuestra propio sistema de emulación, [algo pequeñito][algo-pequenito] del estilo de la Nintendo Classic Mini, algo sencillo, para enchufar y jugar. Y nada como una RaspberryPi para ajustarse a estos requisitos.

## Raspberry

De los modelos de Raspberry mas actuales que podemos elegir, tenemos la [Raspberry Pi Zero][pi-zero] que es la mas pequeña y barata, pero que no dispone de WI-FI, ni Bluetooth, ni salida RCA/Euroconector, ni puertos USB “de los normales”. Aunque si eres manitas siempre puedes [añadir una salida RCA][zero-rca], [añadir un módulo wifi][zero-wifi], y buscar algún HUB USB.

Y la opción mas recomendable y sencilla es usar la [Raspberry Pi 3 Model B][pi-3], que es mas cara, mas potente, tiene WI-FI, puerto ethernet, bluetooth, salida RCA y 4 puertos USB listos para usar. Solo necesitaremos un par de cosas mas:

- [Raspberry 3 Model B](https://www.amazon.es/dp/B01CCOXV34) 39 €
- [Carcasa](https://www.amazon.es/dp/B01F1PSFY6/) 8.50 €
- [2 Mandos USB de estilo retro](https://www.amazon.es/dp/B00PL271Y0) 12 €
- Y buscar por casa una tarjeta microSD, un cable HDMI y un cargador de móvil.

![](/assets/raspberry-amazon.png)

## Instalación

El siguiente paso sería instalar el software necesario. Tendremos que instalar un sistema operativo para Raspberry (cualquier de los muchos sabores de linux que hay), instalar y configurar cada uno de los emuladores con los que querríamos jugar (FCEUmm, Snes9x, DOSBox, ScummVM, Mupen, PPSSPP, y la lista sigue ...), configurar los mandos USB que queramos utilizar. Pero, una vez que lo hayamos hecho, ¿como hacemos para elegir el emulador con el que queremos jugar cada vez? Necesitaríamos también un teclado y raton para poder iniciar cada emulador por separado, y poder cerrarlo y elegir el siguiente. Cada emulador tiene su propio interfaz, sus opciones de configuración, su proceso de mapeo de teclas y botones del mando, etc.

Afortunadamente existe un proyecto llamado [libretro][libretro] que simplifica este problema. Este proyecto implementa una librería que unifica el acceso a distintos emuladores, proporcionado un interfaz único para usarlos. Esto se traduce en que una aplicación que utilice libretro podrá ser capaz de iniciar cualquier juego de forma transparente al emulador que finalmente lo ejecute, lo que nos permitirá configurar y ejecutar las distintas plataformas de forma sencilla.

Ahora necesitaríamos una aplicación que nos facilite la gestión de las distintas roms, opciones de configuración y permite iniciar los distintos juegos de forma transparente al emulador sin necesidad de estar usando teclado y raton. Y [EmulationStation][emustation] es la aplicación perfecta para esto.

EmulationStation es la aplicación que querremos iniciar por defecto. Nos permite navegar entre las distintas plataformas de forma gráfica, seleccionar los juegos, descargarse metadatos para que aparezca la carátula, nombre, año, etc. Y nos permite por supuesto iniciar los juegos, configurar opciones globales, configura los mandos de forma global. Y todo esto con una usabilidad y experiencia de usuario digna de alabar. [Chapó!][chapo]

<iframe width="640" height="360" src="https://www.youtube.com/embed/AVQVmsFOclM" frameborder="0" allowfullscreen></iframe>

## Retropie

Afortunadamente existe un proyecto que se ocupa de todo lo anterior. [Retropie] es un proyecto que se encarga de prepararnos una distribución Debian para Raspberry junto con libretro/retroarch, con todos los emuladores y con EmulationStation de inicio. **¡Así que todo se reduce a copiar la distribución de Retropie a nuestra microSD, y listo!**

Los [instrucciones de instalación][retropie-install] son bien sencillas: [descargar la imagen][retropie-download] e instalarla en la microSD con [Windows][retropie-win], [Linux][retropie-lin] o [Mac][retropie-mac].

Una vez iniciemos nuestra Raspberry Pi con la microSD de Retropie nos aparecerá EmulatioStation. Cada vez se detecte un nuevo tipo de mando USB, tendremos que configurar los botones. Si el mando no dispone de alguno de los botones, dejaremos pulsado durante unos segundos cualquier botón ya mapeado anteriormente.

Entre las plataformas que muestra EulationStation veremos que existe una que es Retropie desde la que podremos iniciar distintos scripts de utilidades y configuración de Retropie. Uno de ellos nos permite configurar el WI-FI (necesitaremos enchufar un teclado momentáneamente) o consultar la IP actual.

![](/assets/retropie-pixel.png)

Lo único que nos queda copiar [nuestras roms][roms] en la correspondiente carpeta de la raspberry. Para eso tenemos [varias opciones][retropie-copy-roms], copiarlas con scp (accediendo con el usuario pi y contraseña raspberry) o usando una memoria USB. Para ello retropie tiene configurado un mecanismos de sincronización con rsync. Solo creamos una carpeta  retropie en la raíz de la memoria USB y la enchufamos a la raspberry. Tras unos segundos se habrá sincronizado la estructura de carpetas de roms que se utiliza. Ya podremos volver a enchufar el USB en un PC y copiar las roms que queramos en las carpetas correspondientes.

> *Así que solo nos queda disfrutar de divertidas tardes de juegos retro.*

P.D.: Para salir de los juegos y volver a EmulationStation hay que pulsar SELECT + START. Si aparece un [cuadrado multicolor][rainbow] en la esquina es porque el cargador no suministra suficiente potencia y deberíamos probar con otro.

[nintendo-classic]: https://www.nintendo.com/nes-classic
[NES]: https://es.wikipedia.org/wiki/Nintendo_Entertainment_System
[8bitdo-nes]: http://www.nes30.com
[arcade]: http://taximaxi.co/p/12335/beautiful-building-an-arcade-cabinet-on-build-your-own-nes-mini-arcade-cabinet-and-win-at-life-building-an-arcade-cabinet/
[yoshi-island]: https://en.wikipedia.org/wiki/Yoshi%27s_Island
[algo-pequenito]: https://www.youtube.com/watch?v=IGlKLUujURk&feature=youtu.be&t=57
[pi-zero]: https://www.raspberrypi.org/blog/raspberry-pi-zero/
[zero-rca]: https://www.modmypi.com/blog/how-to-add-an-rca-tv-connector-to-a-raspberry-pi-zero
[zero-wifi]: https://youtu.be/Kbv5mqv32tc
[pi-3]: https://www.raspberrypi.org/blog/raspberry-pi-3-on-sale/
[libretro]: http://www.libretro.com/
[emustation]: http://www.emulationstation.org/
[retropie]: https://retropie.org.uk/
[retropie-install]: https://github.com/RetroPie/RetroPie-Setup/wiki/First-Installation#installation
[retropie-download]: https://retropie.org.uk/download
[retropie-win]: http://sourceforge.net/projects/win32diskimager
[retropie-lin]: https://unetbootin.github.io
[retropie-mac]: http://www.tweaking4all.com/hardware/raspberry-pi/macosx-apple-pi-baker
[roms]: http://www.freeroms.com/
[retropie-copy-roms]: https://github.com/RetroPie/RetroPie-Setup/wiki/Transferring-Roms
[rainbow]: https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=82373
[chapo]: http://gph.is/1U3YLbG