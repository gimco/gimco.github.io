---
layout: post
title: Aplicaciones como churros
tags: ares, churroapp, hp, palm, webos
permalink: /:year/:month/aplicaciones-como-churros.html
---

En estas fechas navideñas, y sobre todo el día de año nuevo, casi es una tradición felicitar a familiares y amigos con algún mensaje. Supongo que las operadoras estarán muy contentas con la cantidad de mensajes de texto enviados por sus usuarios el día 1 de Enero. Aunque últimamente, con el auge de las redes sociales, los smartphones y las tarifas planas de datos, muchos optan por el mensaje privado via tuenti, facebook, twitter o email. No recuerdo cuando fue la última vez que envié un SMS, normalmente siempre hablo via gtalk o email con mis contatos.  

Todavía me resisto a felicitar por via electrónica, ya que no todo el mundo está en las redes sociales, ni todos están mirando éstos sitios el día 1 Enero. Así pues, la opción SMS en estos casos sigue siendo más instantáneo, además que lo veo mas personal (espero que quien recibe un SMS de felicitación, piense al menos que me ha costado dinero acordarme de él y querer felicitarle).  

Despues está el problema, de qué mensaje enviar. Si miramos en google trends, podeis apostar a que búsquedas del tipo "mensaje de felicitación navidad" se hacen muy populares. Hay incluso quien busca cosas como "mensaje original de feilicitación", aunque claro, si buscas en internet el mensaje, muy original no lo vas a encontrar. La mayoría no se preocupan en pensar un mensaje de filicitación del nuevo año. También tenemos al típico que espera a recibir algún mensaje gracioso para copiarlo y reenviarlo, por lo que más de una vez he recibido el mismo mensaje de distintas personas, o incluso mi propio mensaje!  

A mi me gusta enviar mensajes simples, pero escritos de forma diferente. Normalmente me invento un mecanismo de "cifrar" mis mensaje navideños. Siempre envio un pequeño puzzle, que mis contactos deben resolver, para poder saber el mensaje de felicitación. Todos los años me invento un mecanismo nuevo. Algunos ejemplos con el mensaje "Felix año nuevo 2011" son:  

* 1102oveunoñazilef *(invirtiendo el mensaje)*
* 333.33.555.444.9999.2.6666.666.66.88.33.888.666 2011 *(simulando las pulsaciones al escribir un sms en un teclado numerico)*
* edkhy znñ ltduñ 1900 *(hay que sustituir cada letra por la siguiente en el abecedario)*

Uno de los mensajes que envié este año, con el método de "cifrado" fué el siguiente:  

> uQ te ne ag si am ss la du id en or ma ro my ne so ue ir ob er pl or ix om ña o *(Esta vez he invertido cada dos letras del mensaje que pensé enviar)*

Suelo enviar tres o cuatro mensajes diferentes, a cada grupo de contactos. Un mensaje para mis amigos "normales", otro para mis amigos informáticos, otros para los familiares, y otro para los compañeros de trabajo, y algunos más para personas en concreto. Todos los años, cojo papel y lápiz, y me dispongo a pensar felicitaciones y cifrar a mano cada uno de los mensajes que enviaré el dia de año nuevo.  

Mucho trabajo cada año preparar todo esto. Así que aprovechando que esta vez tenía una Palm Pre con sistema operativo WebOS, me animé a aprovecharme de su tristemente poco conocida potente plataforma. WebOS es un sistema operativo que está basado en tecnologías web básicas (HTML, CSS y JavaScript fundamentalmente), lo que permite crear aplicaciones sea fácil, muy fácil y rápido y muy rápido. Y más rápido aún, si usamos Ares. Ares es un entorno de desarrollo de aplicaciones para WebOS, completamente desarrollado sobre tu navegador. Es decir, nada de descargar máquinas virtuales, ni kits de desarrollo de 200 megas, ni instalar IDEs como XCode, NetBeans, o Eclipse. Todo lo que necesitas para crear aplicaciones para WebOS es un navegador web, nada más.  

Así que me fui al ordenador de mi hermana, abrí firefox, y entré en [ares.palm.com](http://ares.palm.com/). Ya tenía todo lo necesario para poder crear una aplicación!. Creé un nuevo proyecto, con el diseñador de pantallas de Ares, agregué una par de cajas de textos, un boton a la pantalla, y añadí un evento sobre el botón. Jugué un rato con las expresiones regulares en la consola de javascript del navegador, hasta que saqué la expresión regular que me cifraba mi mensaje. Este es todo el código javascript que tuve que escribir:  

~~~js
var tmp = this.textoOriginal.value.replace(/ /g, '') + ' ';
this.textoFinal.value = tmp.replace(/(.)(.)/g, '$2$1 ');
this.controller.modelChanged(this.textoFinal);
~~~

La primera línea elimina los espacios en la frase. ¡La segunda linea no es ascii-art-obsceno!, es una expresión regular para invertir las posiciones de dos letras consecutivas. Y por último notificamos al framework que hemos modificado el objeto textoFinal, que está asociado a la segunda caja de texto. Aquí podeis ver los pantallazos de Ares de la mini aplicación:  

![](/assets/ares1.png)

![](/assets/ares2.png)

En 5 minutos, tenía la aplicación lista. Le dí a exportar, y Ares generó un archivo IPK con la aplicación. Lo puse en una carpeta pública del dropbox, accedí desde el navegador del movil, y la instalé en mi Palm Pre. Todo listo para fastidiar a conocidos con mensajes extraños el día de año nuevo :)  

![](/assets/felici1.png)

![](/assets/felici2.png)

Obviamente llamar a esto aplicación es casi un insulto al resto de aplicaciones móviles existentes, pero lo que me encanta de WebOS es que es muy fácil crear aplicaciones con su plataforma, y la curva de aprendizaje es casi inexistente, ya que lo primero que se aprende prácticamente antes de desarrollar para la web, son las tecnologías básicas, HTML, CSS y Javascript. WebOS es el sistema operativo perfecto para desarrolladores, diseñadores web, hackers y a todo aquel que le guste trastear. Esperemos que con el impulso de HP, logren abrir los ojos a más de uno, y este magnífico sistema operativo consigan el lugar que se merece.