---
layout: post
title: Mainframes
---

Últimamente he tenido la oportunidad de conocer una parte de la historia de la informática que para mi era totalmente desconocida. Me estoy refiriendo al mundo de los [mainframes]. Estos sistemas han sido el orgullo de los centros de datos de empresas y universidades. Son ordenadores caros (muy caros), potentes y del tamaño de frigoríficos americanos. Aunque hoy en día IBM sigue comercializando estos sistemas, la fuerte competencia de las soluciones de virtualización y la popularidad de los proveedores en la nube, no los hace tan imprescindibles como antaño.

![](/assets/mainframe-390.gif)

No cabe duda de que han sido las plataformas donde se han implementado los grandes sistemas de información de los 80 y los 90, y aunque parezca sorprendente, muchos de ellos siguen dando servicio varias décadas después. Hoy en días estos sistemas siguen en la sombra, dando soporte y gestionando el negocio clave de multitud de sistemas. De hecho, según cita [una de las principales empresas software de Europa][softwareag]:

> COBOL is still a world-player in IT. In fact, 75 percent of the world's business transactions are processed by COBOL.

Podemos identificarlos con sus pantallas negras tan características (emuladores de terminal). Yo al menos los he podido identificar en sitios tan diversos como en los sistemas de gestión de garantía de Thermomix, en El Corte Inglés, Hospitales, Supermercados, Aerolíneas. El negocio de IBM con estos sistemas es increíble. Desorbitados costes de licencias, licencias que hay que pagar anualmente por "ejecución de software". Es decir, por el mero hecho de arrancar el sistema.

![](/assets/natural-adabas-example.jpg)

Son los conocidos backoffice, o sistemas legacy. Sistemas que llevan funcionando 20 años, desarrollados en los 80’s y 90’s y que por una u otra razón no han podido ser actualizados o migrados a nuevas tecnologías o nuevas plataformas. **Sistemas con millones y millones de líneas de código**, que actualmente están funcionando, sin fallos. Y todos hemos aprendido la regla de oro: *“Si funciona no lo toques”*. No es casualidad que el [libro más mencionado en Stack Overflow][dev-books] sea precisamente [“Working Effectively with Legacy Code”][working-legacy-code].

![](/assets/book-working-legacy-code.jpg)

## Acceso al sistema

Para conectarnos al mainframe se utiliza un emulador de [terminal 3270][t3270], que sería el equivalente a un telnet o ssh, aunque la interacción parece mas rica, ya que el cliente gestiona conceptos como campos de entrada, campos de solo lectura y posición del cursor. En Windows se suele utilizar los programas propietarios Entire Connection o TN3270Plus, y afortunadamente existe el proyecto [x3270] que implementa distintos clientes para todas las plataformas incluida Linux.

![](/assets/c3270.png)

Junto con los clientes gráficos y de consola se incluye el cliente s3270 que nos permite interactuar con el mainframe en forma de scripts. Podemos utilizar este cliente para automatizar ciertas acciones (al estilo de macros) o para hacer lo que podríamos llamar [Terminal scraping][scraping], para extraer información de forma sencilla obteniendo el texto en pantalla.

## Sistema operativos

Algunos de estos antiguos mainframes ejecutan sistemas operativos "extraños" a los ojos de los que estamos acostumbrado a los sistemas basados en UNIX. Recordemos que estos sistemas ya existían varias décadas antes de la popularización de Linux (Linus Torvals presentó Linux 1.0 en 1992). Uno de estos sistemas operativos con los que me he encontrado es [VSE/ESA][vse-esa].

Como ejemplo de lo "diferente" que pueden llegar a ser estos sistemas operativos veamos por ejemplo [JCL (Job Control Language)][jcl]. Esto es el lenguaje que se utiliza para interactuar con el sistema operativo. Vendría a ser el equivalente a un shell script con bash o un .bat en windows. Lo que en el mundo linux sería copiar un fichero:

```
cp oldFile newFile
```

... en el mundo mainframe con JCL equivaldría a:

```
//IS198CPY JOB (IS198T30500),'COPY JOB',CLASS=L,MSGCLASS=X
//COPY01   EXEC PGM=IEBGENER
//SYSPRINT DD SYSOUT=*
//SYSUT1   DD DSN=OLDFILE,DISP=SHR
//SYSUT2   DD DSN=NEWFILE,
//            DISP=(NEW,CATLG,DELETE),
//            SPACE=(CYL,(40,5),RLSE),
//            DCB=(LRECL=115,BLKSIZE=1150)
//SYSIN    DD DUMMY
```

Podemos hacernos una idea del aspecto que tendría un JCL que obtenga un fichero por FTP, ejecute un programa que procese los datos, genere nuevos ficheros, los transfiera por FTP a otra máquina, y envíe correos electrónicos ... Para los mas curiosos todavía se pueden encontrar [tutoriales de JCL][jcl-tutorial].

## Desarrollo de aplicaciones

Estos mainframes eran las plataformas de desarrollo de aplicaciones empresariales de su época, y como tales proporcionaban a los programadores el entorno de programación, compilación, procesamiento y acceso a bases de datos necesarios. No existía el concepto de desarrollo en local y despliegue, si no que los desarrolladores se conectaban mediante el emulador de terminal al mainframe para programar. El entorno de desarrollo era por lo tanto basado en consola (quienes hayan conocido TurboPascal o TurboC se podrán hacer una idea). Un ejemplo de este interfaz:

![](/assets/natural314.png)

Los lenguajes mas habituales eran C, COBOL y NATURAL. Yo en concreto me he encontrado con algunas aplicaciones desarrolladas en el lenguaje de programación [NATURAL] y la base de datos [ADABAS], ambos creados por la empresa Software AG.

> Natural: Cobol's ugly, brain-damaged, BABBLING IN ALL-CAPS — but regrettably still healthy and strong — cousin.

NATURAL es un lenguaje de programación de los considerados de [4ª generación][4gl] (como Clipper), que vendrían a ser lenguajes de alto nivel y de propósito específico. En estos lenguajes el acceso y manipulación de las base de datos y la interacción con interfaces de usuario e informes forman parte del propio lenguaje, al contrario que con los lenguajes de propósito general los cuáles necesitarían de distintas librerías adicionales para implementar estas características. Esto hace que la creación de pantallas, validaciones de datos, generación de listados e informes sea rápida y sencilla. 

```
DEFINE DATA
LOCAL USING LUSUARIO
LOCAL
  1 #APELLIDO (A20)
END-DEFINE
INPUT #APELLIDO
FIND LUSUARIO WITH APELLIDO = #APELLIDO
  DISPLAY NIF NOMBRE APELLIDO1
END-FIND
END
```

Como curiosidad decir que la base de datos ADABAS es NoSQL ... en el sentido de que no se utiliza SQL para consultarla. En su lugar hay que construir programas NATURAL para obtener la información que queramos. Soporta búsquedas “fonéticas” (à la Oracle Text/Lucene), grupos periódicos y campos multivaluados, similares a las estructuras jerárquicas y arrays que se utilizan con JSON y MongoDB, claro que todo esto en los años 80 😆.

![](/assets/dilbert-legacy-code.gif)

[mainframes]: https://en.wikipedia.org/wiki/Mainframe_computer
[softwareag]: https://en.wikipedia.org/wiki/Software_AG
[dev-books]: http://www.dev-books.com
[working-legacy-code]: https://www.amazon.es/Working-Effectively-Legacy-Robert-Martin/dp/0131177052
[t3270]: https://es.wikipedia.org/wiki/IBM_3270
[x3270]: http://x3270.bgp.nu/index.html
[scraping]: https://en.wikipedia.org/wiki/Web_scraping
[vse-esa]: https://en.wikipedia.org/wiki/VSE_(operating_system)
[jcl]: https://en.wikipedia.org/wiki/Job_Control_Language
[jcl-tutorial]: http://tutorialspoint.com/jcl/
[NATURAL]: https://es.wikipedia.org/wiki/Natural_(lenguaje_de_programación)
[ADABAS]: https://en.wikipedia.org/wiki/ADABAS
[4gl]: https://en.wikipedia.org/wiki/Fourth-generation_programming_language
