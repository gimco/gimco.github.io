---
layout: post
title: Docker en máquinas de 32bits
---

Si dispones de una máquina y/o una distribución de Linux de 32 bits (Como por ejemplo Ubuntu 14.04 32bits) e intentas instalar Docker te puedes encontrar con el siguiente mensaje:

![Error: you are not using a 64bit platform. Docker currently only supports 64bit platforms.](/assets/docker-dont-run-on-32.png)

En la propia página web podemos encontrar los requisitos referentes a [Ubuntu Linux][ubuntu-requisitos]:

> Docker requires a 64-bit installation regardless of your Ubuntu version. Additionally, your kernel must be 3.10 at minimum.

Aunque nuestro sistema no cumpla con estos requisitos todavía tenemos una alternativa. Usar la misma técnica que se utilizaba en MacOSX y Windows para poder instalar docker, [boot2docker].

Las herramientas de docker utilizan el paradigma cliente/servidor, por lo que cuando lo instalamos tenemos por un lado la parte cliente, que sería el comando de consola `docker`, y por otro lado la parte servidor o servicio docker que utiliza ciertas características del kernel de Linux para crear y administrar los contenedores.

En el caso de MacOSX y Windows, no se dispone de estas características en el kernel, por lo que la solución es instalar una máquina virtual con un Linux que proporcione esta parte servidor. Esta es la misión de boot2docker, proporcionar el mecanismo de crear e iniciar esta máquina virtual (basada en VirtualBox) para poder usar docker en otros sistemas operativos no soportados. El rendimiento será peor obviamente (ya que el “docker servicio” no se ejecutará de forma nativa como en un Linux de 64bits, si no que se ejecutará dentro del Linux de la máquina virtual de VirtualBox), pero será mas que suficiente para poder trastear.

Puesto que no existe boot2docker para sistema Linux de 32 bits tendremos que compilarlo a mano. Si echamos un vistazo al [código fuente][boot2docker-cli] del proyecto veremos que nos encontramos con un problema recursivo: necesitamos docker para poder compilar esta aplicación, que es la que necesitamos precisamente para poder tener docker! :P 

Si analizamos el [Makefile] y el [Dockerfile] del proyecto vemos que sencillamente se instala el compilador de [go][golang] (lenguaje con el que está programado este cliente) y algunas dependencias. Afortunadamente se puede instalar el compilador de `go` con los binarios de la página oficial y podemos compilar automáticamente boot2docker-cli como cualquier otra librería:

~~~
cd /tmp
wget http://...../go1.5.3.linux-386.tar.gz
tar zxvf go1.5.3.linux-386.tar.gz
mkdir gopath && export GOPATH=/tmp/gopath
export GOROOT=/tmp/go
/tmp/go/bin/go get -v github.com/boot2docker/boot2docker-cli
sudo mv /tmp/gocode/bin/boot2docker-cli /usr/bin/boot2docker
~~~

En segundo lugar necesitamos el comando docker, que sería la parte cliente. Desde la [sección de instalación de la página oficial][docker-install] se ofrece un enlace para descargar [el binario para 32 bits][docker-i386]:

 https://get.docker.com/builds/Linux/i386/docker-latest

Ya tendríamos todo lo necesario en nuestra máquina de 32 bits. La primera vez necesitaremos que boot2docker se descargue y configure la máquina virtual de VirtualBox que proporcionará la parte servicio de docker:

~~~
boot2docker init
~~~

Cada vez que queramos usar docker deberemos iniciar este servicio:

~~~
boot2docker start
~~~

Una vez se inicie se nos mostrará las variables de entorno que informan al comando docker con qué servidor docker hablar. Los copiamos y ejecutamos en nuestra sesión y listo, ya podremos usar docker normalmente en nuestra máquina de 32 bits:

~~~
docker run ubuntu:14 echo "hola mundo!"
~~~

Happy dockering!

P.D.: Si abrimos un nuevo terminal hay que volver a establecer las variables de entorno. Podemos ejecutar `boot2docker shellinit` para volver a mostrarlas.


[ubuntu-requisitos]: https://docs.docker.com/installation/ubuntulinux/
[boot2docker]: http://boot2docker.io/
[docker-install]: https://docs.docker.com/installation/binaries/
[boot2docker-cli]: https://github.com/boot2docker/boot2docker-cli
[Dockerfile]: https://github.com/boot2docker/boot2docker-cli/blob/master/Dockerfile
[Makefile]: https://github.com/boot2docker/boot2docker-cli/blob/master/Makefile
[docker-i386]: https://get.docker.com/builds/Linux/i386/docker-latest
[golang]: https://golang.org/
