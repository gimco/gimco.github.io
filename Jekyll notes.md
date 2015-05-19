# Chuleta de jekyll

Hay que lanzar el comando desde la raiz del proyecto jekyll. Iniciar el servidor embebido y que procese automáticamente los cambios detectados (no recarga el navegador automáticamente):

jekyll server -w --drafts


Los post están creados en markdown, con extensión .md, y poseen una cabecera delimitada por --- donde se puede establecer varias propiedades. Al menos las mas interesantes son "layout: post" y "title: XXXX".

Los enlaces los podemos coleccionar al final del documento a modo de pie de página y hacer referencia a ellos en el texto usando corchetes [enlace1]. Si queremos utilizar un texto alternativo en lugar del alias usamos dos contrucciones consecutivas de corchestes como por ejemplo [una descripción mas larga][enlace1]. Por ultimo escribimos la anotación siguiente para hacer referencia al enlace real:

[enlace1]: http://ENLACE_REAL_AL_DOCUMENTO


Para palabras y codigo en linea se ponene entre acentos `palabra`

Para formatear bloques se encierran con ```LENGUAJE y ```, o ~~~LENGUAJE y ~~~

Para incrustar imágenes ![texto alternativo](url)



