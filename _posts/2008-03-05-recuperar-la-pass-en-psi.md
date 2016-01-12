---
layout: post
title: Recuperar la contraseña en PSI
tags: linux, password, pidgin, psi
permalink: /:year/:month/recuperar-la-pass-en-psi.html
---

En el trabajo, los windoseros (y algunos kderos) utilizan PSI como cliente de mensajería jabber. Hoy un compañero se iba a pasar a pidgin, pero no recordaba su contraseña en una de las cuentas jabber. Afortunadamente buscando encontramos una solución para [esto](http://blogmal.42.org/rev-eng/psi-password.story) (ventajas de ser opensource).  

Resulta que PSI lo que hace es proteger un poco la pass, en lugar de tenerlo en texto plano como hace pidgin (que no me gusta demasiado). Lo que hace PSI es realizar un XOR entre la contraseña y el JID, y guardar el hexadecimal del texto resultante en UTF-16. Gracias a este "simple" script de perl se pudo recuperar la pass:  

~~~bash
perl -le '($jid,$pw)=@ARGV;$pw=~s/..(..)/chr hex$1/ge; \  
          print substr($pw^$jid,0,length$pw)'  \  
          user@jabber.server 000100020003007e  
~~~