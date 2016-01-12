---
layout: post
title: DBus y Pidgin
tags: dbus, linux, mono, pidgin
permalink: /2008/02/dbus-y-pidgin.html
---

Hoy toca hablar de [DBus](http://www.freedesktop.org/wiki/Software/dbus). DBus es un bus de comunicaciones que las aplicaciones utilizan para intercambiar mensajes. Es la forma mas sencilla para controlar el comportamiento de muchas de nuestras aplicaciones de escritorio. En este caso vamos a controlar pidgin desde una aplicación hecha con mono.  

En nuestro sistema tenemos varias comandos de consola que nos ayudan a probar cosas con dbus. Por ejemplo, "dbus-monitor" monitoriza los mensajes enviados a traves de dbus. Si lanzamos esto en una consola y utilizamos abrimos por ejemplo pidgin, vemos que aparecen multitud de mensajes por consola:  

~~~bash
[gimco@localhost]$ dbus-monitor  
...  
signal sender=:1.16 -> dest=(null destination) path=/im/pidgin/purple/PurpleObject;   
interface=im.pidgin.purple.PurpleInterface; member=JabberSendingText  
uint32 3875  
...  
~~~

Aqui podemos ver eventos que son notificados por pidgin. Cualquier aplicación podrá registrar estos eventos y realizar acciones en consecuencia. Ademas de poder registrar eventos producidos por pidgin, podemos invocar varios métodos de su api con dbus. Algunos de estos métodos vienen explicados en la página [DbusHowto](http://developer.pidgin.im/wiki/DbusHowto) de la documentación de pidgin. Hagamos una prueba, obteniendo un listado de todas nuestras cuentas activas:  

~~~bash
[gimco@localhost ~]$ dbus-send --print-reply  
     --dest=im.pidgin.purple.PurpleService /im/pidgin/purple/PurpleObject  
     im.pidgin.purple.PurpleInterface.PurpleAccountsGetAllActive  

method return sender=:1.16 -> dest=:1.148 reply_serial=2  
array [  
int32 970  
int32 1046  
int32 1056  
]  
~~~

En este caso este método devuelve un array (enteros) con las referencias a los objetos que representan las cuentas pidgin. En este caso, las cuentas de gtalk, irc y bonjour. Consultemos por ejemplo cual es protocolo de la segunda cuenta:  

~~~bash
[gimco@localhost ~]$ dbus-send --print-reply  
      --dest=im.pidgin.purple.PurpleService /im/pidgin/purple/PurpleObject  
      im.pidgin.purple.PurpleInterface.PurpleAccountGetProtocolName int32:1046  

method return sender=:1.16 -> dest=:1.149 reply_serial=2  
string "Bonjour"  
~~~

Vemos que cuando utilizamos el comando dbus-send, tenemos que especificar el tipo del parametro. En este caso la cuenta con identificador 1046 usa el protocolo Bonjour.  

Con esta utilidad podemos realizar pruebas y llamar a estos eventos. El siguiente paso es utilizar dbus con mono. Para esto utilizaré la librería [ndesk-dbus](http://www.ndesk.org/DBusSharp), que es una implementación en C# de DBus. Los pasos para utlizar dbus en nuestras aplicaciones con mono son muy simples. Declaramos un interfaz que será el que utilizaremos para realizar las llamadas. Obtendremos el objeto que implementa este interfaz directamente de DBus.  

~~~csharp
[Interface ("im.pidgin.purple.PurpleInterface")]  
public interface PurpleInterface : Introspectable  
{  
}  
~~~

En este interfaz iremos agregando los métodos que vayamos necesitando. Lo primero que haré es obtener un listado de todos los métodos disponibles. Así que nos conectamos a DBus, obtenemos el objeto y vemos su definición en xml:  

~~~csharp
Bus bus = Bus.Session;  
PurpleInterface pi = bus.GetObject <PurpleInterface>("im.pidgin.purple.PurpleService",  
                                     new ObjectPath ("/im/pidgin/purple/PurpleObject"));  
Console.WriteLine (pi.Introspect ());
~~~

Recordar importar NDesk.DBus. La ejecución de este código devolverá la definición de todos los interfaces y métodos disponibles por el objeto indicado (PurpleObject). Por ejemplo, fijemosnos en la siguiente:

![](/assets/ejemplo-dbus-pidgin.png)

De todo ese código podemos deducir cual sería el método a agregar en nuestro interfaz:  

~~~csharp
    [Interface ("im.pidgin.purple.PurpleInterface")]  
    public interface PurpleInterface : Introspectable  
    {  
        int PurpleAccountsFindAny (string name, string protocol);  
    }  
~~~

Por lo que ya podríamos realizar esta llamada utilizando el mismo ejemplo que antes. Jugando un poco con los métodos que aparecen podemos enviar mensajes a nuestros contacto fácilmente:  

~~~csharp
    public class MainClass  
    {  
        public static void Main(string[] args)  
        {  
        Bus bus = Bus.Session;  
            PurpleInterface pi = bus.GetObject <PurpleInterface>("im.pidgin.purple.PurpleService",  
                                                                 new ObjectPath ("/im/pidgin/purple/PurpleObject"));  

            int account = pi.PurpleAccountsFindAny ("MICORREO@gmail.com/Home", "prpl-jabber");  
            int conversation = pi.PurpleConversationNew (1, account, "DESTINATARIO@gmail.com");  
            int im = pi.PurpleConvIm (conversation);  
            pi.PurpleConvImSend (im, "hola **Mundo!**");          
        }  
    }  

    [Interface ("im.pidgin.purple.PurpleInterface")]  
    public interface PurpleInterface : Introspectable  
    {  
        int PurpleAccountsFindAny (string name, string protocol);  
        int PurpleConversationNew (uint type, int account, string name);  
        int PurpleConvIm (int converstation);  
        void PurpleConvImSend (int im, string message);  
    }
~~~

![](/assets/ejemplo-dbus-pidgin-mensaje.png)

En este caso, no quería molestar a nadie con mis pruebas y me envié el mensaje a mi mismo :(, es un poco triste, pero que se le va a hacer ...  

Dentro de poco pondré un ejemplo de utilización de un servicio web REST, combinado con este ejemplo de hoy, para resolver un problemilla que tenía con una página web de ventas ;)