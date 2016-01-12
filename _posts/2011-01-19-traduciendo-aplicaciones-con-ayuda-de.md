---
layout: post
title: Traduciendo aplicaciones desde consola
tags: bash, google, linux, script, traducir
permalink: /:year/:month/traduciendo-aplicaciones-con-ayuda-de.html
---

Recientemente un compañero de trabajo, me comentó durante el desayuno que estaba harto de hacer lo mismo durante toda la mañana. Le habían encargado realizar una traducción de una de nuestras aplicaciones al euskera. El ni habla, ni entiende nada de vasco. Así que la idea era realizar una primera traducción usando algún traductor automático, para que después nuestro cliente corrigiera las frases y expresiones, sobre un trabajo ya realizado.  

Como la mayoría de los frameworks web existentes viene preparados para la internacionalización, la aplicación disponía de un archivo de propiedades con todas las frases en español que se mostraban en algún momento. Por lo que al menos, las tenía todas juntas. Su trabajo era pues, crear un archivo nuevo pero con los valores para euskera. Un ejemplo de archivo de propiedades de mensajes es el siguiente:  

~~~
messages_es_ES.properties
=========================

common.cancel=Cancelar
common.continue=Continuar
login.user=Usuario
login.pass=Contraseña
login.error.notValid=Contraseña o usuario no válido
main.welcome=Bienvenido a la aplicación X
...
~~~

Frase por frase, había que copiarla en el traductor de google por ejemplo, pulsar en traducir y copiar el resultado en un nuevo archivo, para finalmente generar un nuevo archivo de propiedades con los textos en euskera. Seguro que existen herramientas que ayudan a realizar este tedioso trabajo, pero muchos se lanzan al trabajo duro mecánico, o trabajo de monos, como me gusta llamarlo, antes de pensar si quiera en buscar herramientas que faciliten este tipo de tareas.  

Como uno de mis defectos o virtudes, es odiar el trabajo de monos, le dije que dejara de perder el tiempo ya que realizar esa tarea es un castigo para su inteligencia. Le dije que en 10 minutos tendría la aplicación traducida en inglés, euskera y catalán!  

Existen comandos en linux que facilitan realizar traducciones desde consola, pero me hacia ilusión utilizar manualmente el servicio de google translate. Para ello accedí a la página de google translate y seleccioné las opciones de traducción. Ahora quería saber que petición se realizaba para realizar la traducción. Así que usando firebug o herramienta similar, monitoricé las peticiones que se habían realizado.  

![](/assets/developertools.png)

Revisando las peticiones me encontré con la llamada al servicio de traducción:

~~~
http://translate.google.com/translate_a/t?client=t&text=Continuar&hl=es&sl=es&tl=en&multires=1&otf=2&sc=1
~~~

Al copiar esto en el navegador vemos la respuesta:  

~~~
[[["Continue","Continuar","",""]],[["verbo",["continue","keep","pursue","go on","carry on","remain","resume","keep up","keep on","persist","sustain","hold","push on","drag on","last","last out","get along with smth."]]],"es",,[["Continue",[5],1,,990,0,1,0]],[["Continuar",4,,,""],["Continuar",5,[["Continue",990,1,],["Continuing",0,1,],["Continued",0,1,],["Further",0,1,],["To continue",0,1,]],[[0,9]],"Continuar"]],,,,61]
~~~

Al ejecutarlo con wget vemos el siguiente error:  

~~~bash
$ wget  "http://translate.google.com/translate_a/t?client=t&text=Continuar&hl=es&sl=es&tl=en&multires=1&otf=2&sc=1" -O -
--22:44:27--  http://translate.google.com/translate_a/t?client=t&text=Continuar&hl=es&sl=es&tl=en&multires=1&otf=2&sc=1
           => `-'
Resolving translate.google.com... 74.125.230.169, 74.125.230.173, 74.125.230.168, ...
Connecting to translate.google.com|74.125.230.169|:80... connected.
HTTP request sent, awaiting response... 403 Forbidden
22:44:27 ERROR 403: Forbidden.
~~~

Al principio pensé que era por las cookies, pero después de probar un par de opciones, parece que el error es debido al useragent. La gente de google se habrán cansado de los scripts (como este mismo :) y habrán baneado el UserAgent de wget. Tan simple como parasar el parámetro -U al comando wget con un useragent válido.  

Solo queda leer el fichero de entrada linea a linea, obtener la frase a traducir, ejecutar el wget, extraer la traducción, generar el fichero de salida con las claves y las nuevas frases. Todo en 10 minutos y como extra, todo en una sola línea de bash :)  

~~~bash
IFS="="; while read -ra linea; do if [ -n "${linea[1]}" ] ; then echo -n ${linea[0]}= ; wget "http://translate.google.es/translate_a/t?client=t&hl=es&sl=es&tl=eu&text=${linea[1]}" -U "Mozilla/5" -qO - | cut -d\" -f 2; else echo ${linea[0]} ; fi; done < messages_es_ES.properties > messages_eu_ES.properties
~~~
