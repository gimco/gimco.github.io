---
layout: post
title: mime incorrectos
tags: gnome, mono, monodevelop
permalink: /:year/:month/mime-incorrectos.html
---

Desde hacía algún tiempo no sabía porque cuando desde MonoDevelop intentaba abrir un archivo .xml este no se abría dentro de monodevelop, si no que abría el firefox?!. Así que me dispuse a investigar como decidía MonoDevelop cada tipo de archivo.  

MonoDevelop tiene distintos tipos de clases IDisplayBinding, cada una encargada de detectar si es capaz de abrir un tipo determinado. Si no encuentra ninguna delega esta responsabilidad en la plataforma (en este caso gnome) y abre la aplicación que esté asociada por defecto.  

Investigando mas comprueba que es SourceEditorDisplayBinding la clase que debería encargarse de abrir estos archivos, además se supone que cualquier archivo que se pueda abrir con gedit, se puede abrir con este editor.  

~~~csharp
// If gedit can open the file, this editor also can do it  
   foreach (DesktopApplication app in IdeApp.Services.PlatformService.GetAllApplications (mimetype))  
    if (app.Command == "gedit")  
     return true;  
~~~

El caso es que estaba firefox como aplicación por defecto y especificando gedit seguia ocurriendo lo mismo... por lo que comprobé cual era el tipo que estaba tomando y era application/x-extension-xml????  

El tipo que debería haber cogido es text/xml o application/xml. Haciendo mas prueba, descubro que la pestaña de cambiar asociación en nautilus no funciona como debía. Cada vez que intento cambiar la preferencia de apertura de un archivo, este se transforma en un appliction/x-extension-EXTENSION, como por ejemplo:  

~~~
application/x-extension-mp3  
application/x-extension-zip  
~~~

si borro ~/.local/share/mime y ~/.local/share/applications vuelve todo a la normalidad, excepto que como por defecto en los archivos .xml esta el firefox. Así que como solución temporal modifiqué el archivo /usr/share/applications/default.list, especificando que para el tipo application/xml se debe abrir con gedit.desktop, por lo que MonoDevelop vuelve a abrir los archivo .xml correctamente.  

Toca abrir incidencia.