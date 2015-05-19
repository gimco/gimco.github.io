---
layout: post
title: Introducción a Clojure
---

En los últimos años, han ido apareciendo distintos frameworks y librerías que se benefician de algunas de las características propias de los lenguajes funcionales. Parece existir una tendencia a utilizar estos lenguajes y librerías tanto en lado servidor como en el lado cliente. [Twitter] y [LinkedIn] fueron unos de los primeros en admitir que utilizaban lenguajes funcionales en el lado servidor, y que estaban mas que satisfechos con el nivel de eficiencia, calidad y mantenimiento de la lógica implementada de esta manera. Play Framework, basado en Scala, es otro ejemplo de framework basado en un lenguaje funcional que se está popularizando cada vez más.

Y en el lado cliente, con la tendencia a crear aplicaciones JavaScript cada vez más complejas y reactivas, librerías como [underscore.js], [Knockout.js], [AngularJS], [ReactJS] y [Blaze], demuestran que el paradigma funcional y el “estilo” declarativo facilitan el desarrollo y el mantenimiento de las aplicaciones.

Los [lenguajes funcionales][lenguajes-funcionales] no son algo nuevo. Aunque quizás lo novedoso es poder utilizar estos lenguajes funcionales sobre plataformas actuales y poder reutilizar e interactuar con código existente. Con el objetivo de aprovechar las bondades de la programación funcional sobre las plataformas actuales nacen lenguajes como Scala y Clojure. Ambos son lenguajes funcionales que se ejecutan sobre la máquina virtual de Java, probablemente la plataforma estándar de facto para el desarrollo de aplicaciones empresariales.

![Clojure](/assets/clojure.png)

Está claro que Scala es la opción mas extendida y popular con diferencia, en parte gracias al apoyo comercial de la empresa Typesafe (fundada por el propio creador de Scala). Después de haber intentado aprender ambos, me ha convencido muchísimo más la [filosofía][rationale] y los principios en los que se basa Clojure (y desde luego [no faltan][comp1] [comparativas][comp2] [entre ambos lenguajes][comp3]).

# Características

Algunas de las [principales características][features] que hacen a Clojure un lenguaje tan único son las siguientes:

## Desarrollo dinámico

Clojure es principalmente dinámico. Esto significa que un programa Clojure no es sólo algo que compilas y ejecutas, sino que es una especie de entorno con el que puedes interactuar haciendo crecer tu aplicación, cargando datos, añadiendo funcionalidades, corrigiendo bugs, testando, y pudiendo examinar y cambiar cualquier elemento. Esto es una experiencia diferente a los ciclos de ejecutar un programa, examinar sus resultados y volver a empezar una y otra vez.

Clojure puede ser embebido en aplicaciones Java o usado como un lenguaje de scripting. El principal interfaz de programación es el REPL (Read-Eval-Print-Loop). Esto es un interfaz de consola que nos permite escribir y ejecutar comandos y examinar sus resultados.

Clojure es un lenguaje compilado, pero no tienes que ejecutar ningún compilador. Todo lo que escribes en la consola REPL o cargas desde un fichero es automáticamente compilado a bytecode JVM al vuelo.

## Programación funcional

Clojure es un lenguaje de programación funcional. Proporciona las herramientas para evitar el estado mutable, las funciones son elementos de primer nivel y enfatiza la iteración recursiva en lugar de los bucles con variables que cambian de estado.

Clojure is impuro, la filosofía tras Clojure es que la mayoría de las partes de los programas debería ser funcional, y los programas que son mas funcionales son mas robustos. Cuando creas sinónimos con `let` no estas creando variables. Una vez que creas el elemento su valor nunca puede ser cambiado.

La forma mas fácil de evitar el estado mutable es usar estructuras de datos inmutables. Clojure proporciona un conjunto de elementos inmutables como listas, vectores, conjuntos y mapas. Ya que estos elementos no pueden ser cambiados, agregar o eliminar algo en estas estructuras significa crear una nueva colección como la anterior pero con que incluya el nuevo elemento.

En ausencia de variables locales que cambian de estado, recorrer  bucles e iterar se hacen de forma distinta que en otros lenguajes donde este tipo de variables son utilizadas en los elementos `for` o `while`. En los lenguajes funcionales, los bucles y las iteraciones son implementados mediante llamadas recursivas. Para evitar que consumamos la pila de llamadas y puesto que Java no incluye soporte para este tipo optimización, Clojure proporciona un operador especial `recur` que permite reasignar y volver al bloque `loop` mas cercano.

## LISP

Clojure es miembro de la familia de lenguajes Lisp. Muchas de la funcionalidades de Lisp han sido llevadas a otros lenguajes, pero el enfoque de `code-as-data` y el sistema de macros son cosa aparte. En Lisp no hay diferencia entre los datos y el código, ambos tienen la misma representación, lo que permite poder manipular o generar funciones en tiempo de ejecución de forma muy sencilla. Es decir, es muy sencillo crear programas que generen programas, mucho mas allá de la simple reflexión.

## Polimorfismo

Clojure soporta polimorfismo de diferentes formas. La mayoría de las estructuras de datos están definidas mediante interfaces Java. Pero Clojure proporciona capacidades polimórficas mas abstractas, avanzadas y flexibles mediante el uso de proxys, multimétodos y protocolos, permitiendo implementar tus propias estrategias y reglas. De hecho, el paradigma de Orientación a Objetos (con las correspondientes reglas de visibilidad, instanciación y herencia) es solo una de las posibles implementaciones que puedes construir con Clojure!

## Programación concurrente

Los sistemas actuales tienen que trabajar con multitud de tareas simultáneas para poder aprovechar las potencia de las actuales CPU (de múltiples núcleos). Intentar con hilos puede ser muy difícil debido a las complejidades de la sincronización. Clojure simplifica la programación multi-hilo de diversas formas. Puesto que las estructuras de datos son inmutables pueden ser compartidas en solo lectura entre los hilos. Sin embargo, es frecuente necesitar cambiar el estado en los programas. Clojure permite cambiar estados pero haciendo uso de un mecanismo que asegure que se mantiene un estado consistente, evitando que los desarrolladores tengan que evitar los posibles conflictos manualmente usando candados y bloques sincronizados.

## JVM

Clojure comparte el sistema de tipos de la máquina virtual de Java, el recolector de basuras, los hilos, etc. Todas las funciones son compiladas a bytecode de la JVM. Clojure puede consumir cualquier librería Java estándar y los programas de Clojure pueden ser utilizados desde aplicaciones Java como cualquier otra librería.

# Hola mundo

Ninguna introducción que se valga puede prescindir del obligado ejemplo `hola mundo`, y no íbamos a ser menos. El único requisito es disponer de alguna máquina virtual de Java instalada y seguir los siguientes pasos:

- El primer paso es descargar Clojure, para lo cuál accedemos a la [página de descarga de Clojure][download], nos bajamos la última versión y descomprimimos el zip. Toda la implementación del lenguaje está en una única librería de apenas un mega, `clojure.jar`.

- Como dijimos, Clojure es dinámico, por lo que comenzaremos por iniciar una sesión en la consola REPL. Solo tenemos que ejecutar el siguiente comando:

~~~bash
$ java -jar clojure.jar
Clojure 1.6.0
user=>
~~~

- Escribamos nuestra primer código Clojure:

~~~clojure
user=> (println "Hola mundo!")
hola mundo
nil
~~~

- Por último escribiremos un ejemplo un poco mas educado, para lo cual crearemos una función que nos salude con nuestro nombre:

~~~clojure
user=> (defn saludar [nombre] (println "Hola" nombre "!"))
#'user/saludar

user=> (saludar "Bruno")
"Hola Bruno!"
nil
~~~

Clojure prácticamente no tiene sintaxis, y la mayoría del lenguaje se reduce a la forma `(función parámetro1 parámetro2 ...)`. En la familia de lenguajes LISP, se utiliza los paréntesis para limitar las expresiones y funciones, y es normal que al principio solo veamos paréntesis por doquier ...

![xkcd: Lisp Cycles](http://imgs.xkcd.com/comics/lisp_cycles.png)

[Twitter]: http://es.slideshare.net/al3x/the-how-and-why-of-scala-at-twitter
[LinkedIn]: http://www.scala-lang.org/old/node/6436
[underscore.js]: http://underscorejs.org
[Knockout.js]: http://knockoutjs.com
[AngularJS]: https://angularjs.org
[ReactJS]: http://facebook.github.io/react
[Blaze]: https://www.meteor.com/blaze
[lenguajes-funcionales]: http://es.wikipedia.org/wiki/Programaci%C3%B3n_funcional#Lenguajes_funcionales
[comp1]: http://www.bestinclass.dk/blog/scala-vs-clojure-lets-get-down-to-business
[comp2]: http://www.quora.com/Why-is-Scala-more-popular-than-Clojure-despite-the-simplicity-of-the-latter
[comp3]: http://programming-puzzler.blogspot.com.es/2013/12/clojure-vs-scala.html
[rationale]: http://clojure.org/rationale
[features]: http://clojure.org/features
[download]: http://clojure.org/downloads
