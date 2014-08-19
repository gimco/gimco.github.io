---
layout: post
title:  "Analizando SpliveTV"
---

Hace un par de días un compañero me facilitó una página en un foro, donde daban instrucciones para poder ver todo los partidos de fútbol gratis en Android. No hace falta decir que obviamente esto es ilegal, pero me llamó la curiosidad por ver como lo estaban haciendo. En la página había una captura de la aplicación que mostraba los distintos canales a los que se tendría acceso, como GolT y Canal+.

La aplicación en cuestión se llama [SpliveTV] y actualmente está en la tienda de aplicaciones de Android. Las instrucciones para conseguir ver estos canales de forma gratuita eran simples, se instalaba la aplicación, y se agregaban unos enlaces que incluían el acceso a estos canales. De hecho los había de distintas categorías, deportes, cine, etc. Así que lo primero que hice fue ver que contenía estos enlaces. Así que probé a cargarlo en Google Chrome pero mostraba un error 404, fichero no encontrado con la siguiente url: `http://spliveapp.com/listas/Deportes.xml`

![Error 404]({{ site.url }}/assets/splivetv-404.png)

Parecía que ya no eran válidos. Se lo comenté a mi amigo pero me decía que *nada de eso*, que lo acaba de probar y que *funcionaba perfectamente*. ¿Como podría ser posible?. ¿Quizás implementaban algún tipo de seguridad?. Una de las forma mas sencillas es verificar el `User-Agent` del dispositivo que hacía la petición. De hecho, hasta Google en su momento usó el `User-Agent` del comando wget para banear al acceso a la página de Google Translate.

Aprovechando que disponía de un servidor Apache publicado en Internet, y que en el fichero `/var/log/apache2/access.log` se muestran las peticiones que se realizan además del `User-Agent`, introduje en la aplicación una url hacia mi servidor a ver que rastro dejaba la aplicación:

<pre>
80.35.XXX.XXX - - [01/Aug/2014:12:53:52 +0200] "GET /listas/prueba.xml HTTP/1.1" 404 976 "-" "Dalvik/1.6.0 (Linux; U; Android 4.4.2; Nexus 4 Build/KOT49H)"
</pre>

Ahí se puede ver el `User-Agent` que utiliza la aplicación, `Dalvik/1.6.0` mas la versión de Android y el modelo del dispositivo, en mi caso un Nexus 4. Así que solo queda probar suerte y realizar una petición con este `User-Agent`. Así que utilizando wget hacemos la siguiente petición:

{% highlight bash %}
curl -A "Dalvik/1.6.0 (Linux; U; Android 4.4.2; Nexus 4 Build/KOT49H)" http://spliveapp.com/listas/Deportes.xml
{% endhighlight %}

**Error 404 de nuevo**. Vaya, parece que hay algo más. Aprovechando que la url es http, es decir, va sin cifrar, opté por la forma mas sencilla posible, instalé un sniffer en el móvil para capturar el trafico que generaba la aplicación al cargar la url. Así que inicié el sniffer ([tPacketCapture]) y cargué de nuevo la url en la aplicación. Efectivamente la aplicación cargó los canales. Detenido la aplicación, enviado el fichero .pcap generado al ordenador y analizado con wireshark, se puede ver que la aplicación hace dos peticiones. Una ya la conocíamos, con la misma respuesta 404 que la anterior. Pero la siguiente es muy familiar, es la original con el sufijo `_Splive`.

![pcap de splivetv]({{ site.url}}/assets/splivetv-wireshark.png)

Escribiendo esta nueva url `http://spliveapp.com/listas/Deportes_Splive.xml` en el navegador ahora sí que obtenemos algo mas interesante:

{% highlight xml %}
<channels_cinema>
    ...
    <channel>
        <id_channel>0</id_channel>
        <name>Gol TV</name>
        <category>Deportes</category>
        <available>1</available>
        <rtmp>
            J3JdxEhsAo+IjEIZK39tF7432DRw+qt51hdvCukOZ19xZNA2JPnfBmkmMt+dwIiBsC3WLBDqAHwRoaVuj5hmhnuWZWTrE/a98ii7IQZp0wz/SGl8GblkRBUS87c71SVjnHQkAJCdF0RxlqEay4C94S7kj8GnSatC5FqKeSUGRIftuAc55H+b0/ifKIpCla+L4+FwW+aoglJ6iDEvJrVbg9pqfEtD5qTEf1fYs3qAYtaBYXN/qjVEGL6fz5f/4P73vDXabshefRaAVE5ESCvKLw==
        </rtmp>
        <link_logo>
            mmFla+TP7GV8cjo2Ce64MdM90Cdl+vtmOTlsrpvMP43DwT9hHaN7aBxa+7WQsva1QsTqOTJJU94=
        </link_logo>
        <url_html>
            EiHdP/BOohXfEs/5a2paJj4RAqiIuPlREjfzlPoz2iLVPEVpBnB+/3OSKElHWyBdSmUjkPfGWeRO598HdMN+0zK6+Cx4pKL5
        </url_html>
        <playpath>0</playpath>
        <token>0</token>
        <isIliveTo>0</isIliveTo>
        <isFreeLive>0</isFreeLive>
        <isFlashStreaming>0</isFlashStreaming>
        <quality>3</quality>
    </channel>
    ...
</channels_cinema>
{% endhighlight %}

Modificando otras urls también se obtiene estos xml con la información deseada. Así que parece que el sistema de "seguridad" consiste en devolver un error 404 para despistar, y agregar el sufijo `_Splive` a la url para obtener los datos.

El xml parece simple, una serie de canales con distintas propiedades, y entre las mas llamativas la dirección rtmp. [RTMP] es un protocolo utilizado para la emisión de video en tiempo real, por lo que estas direcciones son las que contienen la pista de desde donde se está emitiendo estos canales.

A simple vista parece que algunas de las propiedades que vienen en el xml (las mas interesantes) están en formato Base64 por los caracteres que son utilizados para codificarlos. Al decodificar las cadenas en Base64 solo obtenemos datos binarios sin sentido, lo que hace pensar que la aplicación protege esta información cifrando estos con algún sistema.

Llegados a este punto y teniendo solamente estos datos binarios no podemos deducir que sistema se utiliza, así que para buscar pistas se opta por intentar analizar la aplicación. Para ello obtenemos el APK directamente desde el móvil (O desde aptoide) y utilizando la herramienta [apktool] descomprimimos y desensamblamos la aplicación en busca de nuevas pistas.

{% highlight bash %}
apktool d splivetv.apk
{% endhighlight %}

Explorando entre los ficheros creados se puede ver el nombre `Jasypt` . [Jasypt] es una librería Java con diversas utilidades para encriptar información, por lo que es mas que probable que sea esta librería la utilizada para cifrar y descifrar los datos interesantes. Buscando en el código descompilado donde se utiliza la librería jasypt podremos saber que tipo de cifrado se utiliza.

{% highlight bash %}
grep jasypt * -R | grep -v 'smali/org/jasypt'
{% endhighlight %}

Entre los resultados podemos ver que se utiliza dentro de algunas clases como es el caso de `smali/com/tdt/spliveTV/AsyncTasks/ChannelsParserDOM.smali`. Aquí podemos ver que `com.tdt.spliveTV` es el paquete Java utilizado por el código de la aplicación, y la clase tiene un nombre muy revelador `ChannelsParserDOM`. Explorando esta clase y viendo donde se utiliza las clases Jasypt podemos ver claramente que se crea un objeto de tipo [BasicTextEncryptor] y que posteriormente se estable la clave de cifrado con una llamada al método setPassword. La cadena utilizada como contraseña aparece en este mismo fichero. Solo queda probar a descifrar los datos con esta clave usando la misma técnica que utilice Jasypt. Después de leer un poco [parece que con OpenSSL no lo podemos descifrar fácilmente][openssl] ya que la configuración del algoritmo DES utilizado para el cifrado es diferente en OpenSSL y Jasypt, así que lo mas fácil es hacerse un pequeño programa en java que utilizando la misma librería, nos descifre la información que le pasemos por parámetro:

{% highlight java %}
import org.jasypt.util.text.BasicTextEncryptor;

public class Splive {
    public static void main(String[] args) {
        BasicTextEncryptor bte = new BasicTextEncryptor();
        bte.setPassword("CLAVE");
        System.out.println(bte.decrypt(args[0]));
    }
}
{% endhighlight %}

Y para compilarlo, descargamos la librería [jasypt-1.9.2.jar] y ejecutamos lo siguiente:

{% highlight bash %}
javac -cp jasypt-1.9.2.jar:. Splive.java
{% endhighlight %}

Una vez compilada la clase, la ejecutamos pasando como parámetros los trozos de Base64 que vimos en el XML anterior, y obtenemos la siguiente información:

{% highlight bash %}
java -cp jasypt-1.9.2.jar:. Splive "Chorizo Base64 del xml"
{% endhighlight %}

Así que podemos ver por fin los datos que estábamos buscando:

<pre>
rtmp:
    rtmp://50.7.69.170/iguide playpath=XXXXXX swfUrl=http://cdn1.iguide.to/player/secure_player_iguide_embed_token.swf live=1 pageUrl=http://www.iguide.to/ token=YYYYYY

link_logo:
    http://spliveapp.com/logos/logo_goltv.png

url_html:
    http://servicios.elpais.com/programacion-tv/canal/goltv/
</pre>

Parece que por lo tanto, se está utilizando la página iguide.to como fuente de la emisión, y vemos el enlace RTMP junto con los parámetros utilizados para asegurar la conexión, y podemos utilizar algún reproductor que soporte RTMP (como [VLC]) para comprobar que efectivamente funciona, con lo que podríamos reproducirlo desde el ordenador, o transformar la emisión a MP4 usando rtmpdump y enviar la emisión a tu propia televisión por ejemplo.

Así que *¡¡misterio resuelto!!*, otro caso de gente que se lucran mediante la publicidad del robo de la señal de los canales de pago.



[SpliveTV]: https://play.google.com/store/apps/details?id=com.tv.spliveTV
[RTMP]: http://en.wikipedia.org/wiki/Real_Time_Messaging_Protocol
[tPacketCapture]: https://play.google.com/store/apps/details?id=jp.co.taosoftware.android.packetcapture
[apktool]: https://code.google.com/p/android-apktool/
[Jasypt]: http://www.jasypt.org/
[BasicTextEncryptor]: http://www.jasypt.org/api/jasypt/1.8/org/jasypt/util/text/BasicTextEncryptor.html
[jasypt-1.9.2.jar]: http://search.maven.org/#search%7Cga%7C1%7Cjasypt
[openssl]: https://plus.google.com/112848582821882568317/posts/UX26v8cF2vq
[VLC]: https://wiki.videolan.org/Real_Time_Messaging_Protocol/
