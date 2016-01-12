---
layout: post
title: gstreamer
tags: gstreamer, linux, mono, nintendods
permalink: /:year/:month/gstreamer.html
---

gstreamer es un framework multimedia que nos permite construir cadenas de componentes para procesar audio y video. Podemos crear un "pipeline" que reproduzca un mp3, o que visualice un video. La sentencia siguiente es capaz de reproducir un fichero .flv  

~~~bash
gst-launch-0.10 filesrc location=video.flv ! ffdemux_flv name=demuxer demuxer. ! queue \  
    ! ffdec_flv ! ffmpegcolorspace ! autovideosink demuxer. ! mad ! autoaudiosink  
~~~

Algunas de las usos que he hecho de gstreamer son:  

*   Script para convertir videos .avi y .flv a .dpg, formato de video utilizado en la NintendoDS por el moonshell. Bueno, realmente eran dos script, uno que transformaba el video y el audio, y otro que los mezclaba en ese formato especial ".dpg".

*   Un experimento de streaming y WII :). Donde desde mi portatil se obtenía un archivo en .avi, se transformaba en tiempo real a .flv con gstreamer, y se servía via thttpd para que se pudiera ver desde la wii. Con esto conseguíamos utilizando el navegador Opera, poder ver cualquier serie o película que se tuviera en el PC.

Tengo pendiente revisar la librería gstreamer-sharp, para realizar en lugar de scripts en bash, programas en mono, además de buscar algún componente que permite reproducir y convertir los videos a formato 3gp para los móviles.