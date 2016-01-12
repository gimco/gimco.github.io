---
layout: post
title: Cracking j2me
tags: j2me, java, me4se
permalink: /:year/:month/cracking-j2me.html
---

Como siempre, los periodos venariegos nos relaja demasiado, y debemos de ejercitar nuestra mente de vez en cuando. Uno de las cosas que llevo tiempo queriendo hacer es leerme algún libro en inglés. Pero claro, el vocabulario siempre es un handicap. Entonces comencé a buscar algún programilla j2me para mi nokia 6300, con traducciones de palabras.  

Pero buscando, encontré algo más interesante. ¿Por que usar un traductor de palabras ingles-español cuando no entienda una palabra, si es mejor utilizar un diccionario en inglés?. Así cuando no se entieda una palabra, buscamos su definición, que claro estará en inglés. Haciendo esto, el esfuerzo será mayor, pero también la inmersión en el inglés. Así fué como encontré la siguiente página de [Mobile Systems](http://www.mobisystems.com/), donde tienen varios diccionarios de ingles. En concreto me interesó el [MSDict Concise Oxford English Dictionary](http://nokia-6300-software.mobisystems.com/product.html?p=10&l=1&pid=9&i=1&c=1&sc=9), con mas de 240.000 palabras y frases!.  

![](/assets/j2me_dictionary.jpg)

Me lo bajé y lo probé. Pero claro, era una versión demo y sólo te dejaba consultar las palabras que comenzaran por A. Para acceder a toda la funcionalidad se debe introducir un código de licencia válido. Me entró mucha curiosidad por averiguar que mecanismo habrían utilizado para generar las licencias válidas, ya que la aplicación movil no se conecta a internet para validar el código de licencia.  

Descomprimí el .jar y descompilé todas las clases, que por supuesto estaban ofuscadas. Despues de indagar un poco mas, comencé a seguir la traza, desde que se escribe el código en la caja de texto. Pero claro, los desarrolladores intentan complicar esta parte lo máximo posible, por lo que en el proceso intervienen hilos, y muchos objetos con muchas relaciones entre ellos.  

El código de licencia consistían en dos bloques de 5 digitos separados por un guion: XXXXX-XXXXX. Despues de mucho buscar, llegué a una parte donde realizaba unas especie de hash del código de licencia. Obviamente esta es la tecnica que se usa, para poder distinguir codigo de licencia validos de los invalidos. La cosa se comenzaba a complicar, por lo que los desarrolladores habían hecho bien su trabajo de complicar esta parte tan crítica de la aplicación (y por supuesto la ayuda del ofuscador). No hay que decir que no es posible realizar realizar la función inversa de un hash.  

Pero después caí en la cuenta en que el código de licencia no era muy grande, solo habían 10.000.000.000 de licencias posibles, y como no era posible realizar el inverso del hash, porque no probar con cada una de las licencias posibles. Tan solo agregando el .jar de la aplicación j2me, la librería me4se, y un bucle donde llamar al método que valida la licencia, conseguí realizar "un ataque por fuerza bruta" obteniendo todas las claves válidas en menos de un minuto.  

Después de esta distracción llegue a dos conclusiones. Es muy dificil proteger una aplicación j2me, ya que estamos totalmente expuestos. Podemos combinar varias tecnicas, cifrado, hash, activación por internet, ... Pero depende de lo aburrido que esté la persona que intenta romper la aplicación :) lo conseguirá tarde o temprano. Y en el caso de aplicaciones J2ME no suele ser muy dificil.  

La segunda conclusión fué que la librería ME4SE es una maravilla. ME4SE es una implementación de las clases propias de J2ME sobre J2SE, lo que nos permite ejecutar una aplicación j2me, desde nuestro eclipse, sin neceisdad de instalar ningún plugin adicional. Además de que te permite hacer uso de toda la api J2SE disponible, lo cuál facilita mucho las cosas. Además tambien vale como emulador para esos jueguecillos j2me que tanto distraen.