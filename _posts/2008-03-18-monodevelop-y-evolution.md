---
layout: post
title: MonoDevelop y Evolution
tags: evolution, foresight, linux, mono, monodevelop
permalink: /2008/03/monodevelop-y-evolution.html
---

Después de hacer la actualización del sistema me dispuse a abrir mi proyecto con MonoDevelop, pero al realizar la actualización, la versión 0.18 que compilé e instalé localmente no funcionó. Al comprobar que ya estaba el paquete 1.0 en foresight (es una de las mejores cosas que tiene esta distribución, tiene los paquetes la mayoría de paquetes recien salidos del horno), por lo que instalé MonodDevelop 1.0.  

Pero para mi desesperación volvió a fallar, con el intuitivo mensaje:  

~~~csharp
System.InvalidOperationException: Extension node not found in path: /MonoDevelop/Ide/Commands  
  at Mono.Addins.ExtensionContext.AddExtensionNodeHandler (System.String path, Mono.Addins.ExtensionNodeEventHandler handler) [0x00000]   
  at Mono.Addins.AddinManager.AddExtensionNodeHandler (System.String path, Mono.Addins.ExtensionNodeEventHandler handler) [0x00000]   
  at MonoDevelop.Components.Commands.CommandManager.LoadCommands (System.String addinPath) [0x00000]   
  at MonoDevelop.Ide.Gui.IdeApp.Initialize (IProgressMonitor monitor) [0x00000]   
  at MonoDevelop.Ide.Gui.IdeStartup.Run (System.String[] args) [0x00000]   
~~~

Al buscar un poco en los mensajes de consola encontré este otro mensaje:  

~~~
The required addin 'MonoDevelop.Documentation,1.0.0' is not installed.  
~~~

Así que supose que faltaría este plugin. Pero al mirar en /usr/lib/monodevelop/bin, ahí estaba el MonoDevelop.Documentation.dll. Después de buscar un poco y borrar la configuración de mono.addins (que usa monodevelop para gestionar el tema de los plugins), encontré el error real:  

~~~
Assembly not found: monodoc, Version=1.0.0.0  
~~~

Una vez instalado monodoc en el sistema, monodevelop arrancó sin problemas :), eso sí, he dejado [una incidencia en el JIRA de foresight](https://issues.foresightlinux.org/browse/FL-995), para que agreguen monodoc:cil como dependencia de MonoDevelop.  

## Evolution

En cuanto a evolutión, ya funciona bien el tema de sincronizar las etiquetas de las cuentas de gmail, por lo que ya no hace falta hacer el [truco](/2008/02/gmail-etiquetas-imap-y-evolution.html) de crear un archivo dentro de .evolution. Ahora seleccionando la opción "unsubscribe" de la carpeta imap, ya no te la actuliza (en la versión anterior, la siguiente vez que abrías evolutión volvía a crear todas las carpetas por etiquetas).  

La opción de usar calendario de google calendar, no me ha funcionando (tampoco he mirado mucho), pero si me dí cuenta, que si tenía una calendario de google e intentaba abrir el calendario desde el panel, el applet se quedaba colgado...