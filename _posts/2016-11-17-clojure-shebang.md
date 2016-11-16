---
layout: post
title: Clojure shebang
---

Uno de las grandes ventajas de los lenguajes de scripting como bash, python, perl, awk, ruby, php, nodejs, etc, es poder editarlos fácilmente desde consola, sin necesidad de tener que recompilar los fuentes para obtener un binario ejecutable. Esto hace que estos lenguajes sean muy populares para labores de mantenimientos y automatización, o crear pequeñas utilidades.

Los sistemas [*nix][unix-like] implementan un mecanismo denominado [shebang][shebang] mediante el cual podemos informar al sistema operativo que intérprete se debe utilizar para ejecutar un determinado fichero. Tan solo debemos escribir al principio del fichero los símbolos `#!` seguido de la ruta del programa que interpretará el fichero. Probablemente te sonará si has creado alguna vez ficheros de script de bash y has tenido que comenzar el fichero con `#!/bin/bash`.

Así pues, si creamos un fichero con código Python llamado `hola-mundo.py` con el siguiente contenido:

```python
print "hello world"
```

le damos permiso de ejecución e intentamos ejecutarlos obtendremos el siguiente error:

```bash
$ chmod +x hello-world.py
$ ./hello-world.py
Failed to execute process './hello-world.py'. Reason:
exec: Exec format error
The file './hello-world.py' is marked as an executable but could not be run by the operating system.
```

Nuestro sistema operativo *no sabe* como ejecutar el programa aunque le hayamos puesto la extension `.py`. En estos casos tendremos que utilizar  la expresión **shebang** para indicar cómo ejecutar el fichero. Sólo debemos agregar en la primera línea los caracteres `#!` seguido del intérprete:

```python
#!/usr/bin/python

print "hello world"
```

De este modo el sistema operativo ya sabe cómo ejecutar el fichero. De hecho, ejecutar `./hello-world.py` es exactamente lo mismo que ejecutar `/usr/bin/python hello-world.py`.

Es posible que los distintos intérpretes de comandos pueden estar instalados en distintas rutas según el sistema operativo. Para evitar introducir la ruta absoluta del intérprete se suele utilizar el comando `/usr/bin/env` que inicializa las variables de entorno del usuario y podemos indicar solamente el nombre del intérprete:

```python
#!/usr/bin/env python

print "hello world"
```

## Clojure

Todos los lenguajes de scripting instalan un binario que es el propio intérprete, lo que nos permite crear los ficheros con la correspondiente línea shebang. En el caso de Clojure no es así. Al contrario que con otros lenguajes, no disponemos de un instalador “oficial” con el que se instale un comando “clojure”. Lo mas parecido a este instalador oficial son las herramientas que gestionan la compilación de proyectos clojure como son [lein][lein] o [boot][boot], siendo el primero el estándar de facto.

Estas herramientas son las equivalentes a maven o ant en el mundo Java, y como ellas, también disponen de su correspondiente fichero de definición (`pom.xml` / `build.xml`) y de su estructura de proyecto por defecto. De hecho, si instalamos `lein` (tan solo debemos descargar el script de lein y ubicarlo en un directorio dentro del PATH) y creamos un proyecto básico obtenemos lo siguiente:

```bash
$ lein new app hello-world
Generating a project called hello-world based on the 'app' template.
$ find hello-world
hello-world
hello-world/.gitignore
hello-world/.hgignore
hello-world/CHANGELOG.md
hello-world/doc
hello-world/doc/intro.md
hello-world/LICENSE
hello-world/project.clj
hello-world/README.md
hello-world/resources
hello-world/src
hello-world/src/hello_world
hello-world/src/hello_world/core.clj
hello-world/test
hello-world/test/hello_world
hello-world/test/hello_world/core_test.clj
```

Si entramos en el directorio hello-world podremos ejecutar el proyecto clojure usando `lein run` pero que crear esta estructura de directorios y todos estos ficheros para ejecutar pequeños programa clojure que sólo consisten en un único fichero no es demasiado cómodo.

## Creando el comando clj

Como solución podemos crear un fichero que desempeñe la labor de intérprete. Para ello creamos un fichero de nombre `clj` y lo agregamos a algún directorio del PATH. El fichero contendrá el siguiente código:

```bash
#!/bin/sh

CLJ_JAR=$(ls -t ~/.lein/self-installs | head -1)

exec java -Xmx1g -server -cp $CLJ_JAR clojure.main $@
```

Básicamente lo que hacemos es buscar la versión mas actual de lein que contiene las librerías de clojure e iniciar una máquina virtual de Java pasando todos los parámetro recibidos. Si no tenemos lein podemos especificar en la variable `CLJ_JAR` la ruta al fichero `clojure-1.X.jar` que queramos usar. Después de crear este fichero podremos crear scripts con clojure que sean ejecutable utilizando el siguiente shebang:

```clojure
#!/usr/bin/env cli

(println "hola mundo")
```
    
## Importando dependencias

El sistema anterior tiene un inconveniente: no podemos usar librerías adicionales en nuestro fichero de clojure. Para solucionarlo podemos utilizar un plugin [lein-exec][lein-exec]. Para ello debemos agregar el plugin a nuestro fichero `profiles.clj`:

~~~clojure
; Agregar el plugin de lein-exe en ~/.lein/profiles.clj:

{:user {:plugins [[lein-exec "0.3.5"] }}
~~~

Y agregar el fichero [lein-exec][lein-exec-file] a nuestro path. A partir de entonces, en los script que usemos con el shebang lein-exec tendremos una instancia de clojure con el plugin cargado con el que podremos indicar que librerías adicionales necesita el fichero para funcionar. Por ejemplo:

```clojure
#!/usr/bin/env lein-exec

(use '[leiningen.exec :only (deps)])
(deps '[[org.clojure/data.codec "0.1.0"]])

(require '[clojure.data.codec.base64 :as b64])

(-> "SGVsbG8gd29ybGQh"
    .getBytes
    b64/decode
    String.
    println)
```


[unix-like]: https://en.wikipedia.org/wiki/Unix-like
[shebang]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[lein-exec]: https://github.com/kumarshantanu/lein-exec
[lein]: http://leiningen.org/
[boot]: http://boot-clj.com/
[lein-exec]: https://github.com/kumarshantanu/lein-exec
[lein-exec-file]: https://raw.github.com/kumarshantanu/lein-exec/master/lein-exec