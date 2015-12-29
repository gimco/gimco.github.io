---
layout: post
title: OpenKapow y Yahoo Pipes
tags: mashup, openkapow, pipes, rss, yahoo
permalink: :year/:month/openkapow-y-yahoo-pipes.html
---

Una de mis herramientas favoritas es RoboMaker de OpenKapow. RoboMaker es una herramienta que nos permite crear mashups fácilmente. Con esta herramienta gráfica podemos extraer y procesar información de cualquier página web. Una de las primeros robots que hice, fue uno que se conectaba a la página de los40.com, extraía la lista con los nombres de las 40 canciones y realizaba un búsqueda en youtube con esos nombre, para filnamente obtener un .rss con los 40 videos correspondientes a los 40 principales, un día de estos lo explicaré paso a paso :). Con este tipo de servicios puedes hacer muchas cosas.  

Hace poco instalé en la asesoría donde trabaja mi novia, google apps para empresa, para gestionar correo y demás. En la página principal de google (al estilo de igoogle), tiene la lista de las últimas noticas de los periódicos. Me comentó si se podría poner la lista de noticas de la agencia tributaria. En principio que le dije que sí, sólamente haría falta el rss de la agencia tributaria. Pero para mi desgracia no existe :S o al menos yo no lo encontré.  

Así que me dispuse a crea un robot para "robar" esta información de la agencia tributaria, crear un rss con las noticias, para que pudieran agregarla a la página de inicio de la empresa. Aquí podeis ver el RoboMaker en acción:  

![](/assets/robomaker.png)

Realmente tuve que crear dos robots distintos, uno que me creara un rss de las últimas novedades y otro para las últimas notas de empresa. El siguiente paso fué mezclar estos dos rss, y ordenarlos por fecha. Para este tipo de operaciones, es mas sencillo utilizar Yahoo Pipes, ya que éste último está mas enfocado a trabajar con xml y rss.  

Así que finalmente, después de crear la yahoo pipe que utilizaba los rss creados desde openkapow, lo puse en feedburner, para redondear la faena. Finalmente el resultado:

* [http://feeds.feedburner.com/UltimasNoticasAeat](http://feeds.feedburner.com/UltimasNoticasAeat)
* [http://www.aeat.es](http://www.aeat.es)

Lo malo de openkapow, es que una vez creas el robot, lo subes a sus servidores, que ejecutan el robot de forma gratuita, el problema es que los servidores de éstos suelen estar caidos habitualmente ...