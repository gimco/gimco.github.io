---
layout: post
title: foresight y gnome 2.22 (2)
tags: foresight, gnome, linux
permalink: :year/:month/foresight-y-gnome-222-2.html
---

Pues todo ha funcionado a la perfección, para pasar de foresight 1.4 a 2.0 tuve que ejecutar el siguiente comando:  

~~~bash 
sudo conary migrate --no-deps group-gnome-dist=@fl:2  
~~~

Ahora bien, debido al parametro --no-deps, se produjeron algunos desperfectos que pude corregir (principalmente arrelgar grub). Desgraciadamente, la migración no se realizó de una vez (como era de esperar) aunque la verdad es que el sistema de paquetes y componente conary parece muy robusto. Tras volver a ejecutar el comando y dejar que actualizara todo. En definitiva, que ya tengo funcionando otra vez foresight 2.0 y esta vez con gnome 2.22, que dicho sea de paso es una gozada.  

Tendré que investigar más adelante las herramientas para contrucción de paquetes, para algún día crear paquetes de gconta para esta distribución tambien.