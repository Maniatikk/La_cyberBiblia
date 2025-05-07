
Vemos que podemos modificar el /etc/passwd 
Generamos una contraseña hardcodeada con openssl 
`openssl passwd`
	el hash que nos entrega lo podemos hardcodear al /etc/passwd debido a que podemos modificarlo
		`nano /etc/passwd`
Como podemos saber que podemos modificar ese archivo?

Podemos realizar una busqueda de archivos editable
`find / -writable 2>/dev/null | grep -vE "python3.10|proc"`

![[Pasted image 20250420214734.png]]

Aqui modificamos el etc/passwd y agregamos la password hardcodeada con openssl
Una vez hecho eso podemos loggearnos como root utilizando la contraseña que modificamos

Asi que la lee directamente con el etc/passwd y no con el shadow


Tenemos otro caso  TAREAS CRON
hay un archivo que ejecuta un comando whoami en nuestro user ken pero no podemos modificar su contenido para poder ejecutar comandos , pero una manera alternativa de un archivo que se encuentre en nuestro directorio de trabajo es

 Tenemos el archivo example que ejecuta el comando whoami desde el usuario root pero dentro del directorio del usuario Ken
Entonces vemos que dentro de /home/ken existe example
lo que podemos hacer es que eliminemos dentro del directorio ken el documento y creemos uno nuevo con el mismo nombre pero con diferente contenido (malicioso)

![[Pasted image 20250420215556.png]]

![[Pasted image 20250420215616.png]]
Se crea un scritp que otorgue privilegios suid a la bash
Entonces eso lo guardamos y al ejecutarse de nuevo la tarea cron se convertira en privilegio suid y podemos escalar a root

BINARIO PARA PODER IDENTIFICAR DE MANERA AUTOMATICA FORMAS POTENCIALES DE ELEVAR PRIVILEGIOS

Github: ==lse.sh==  -  ==linpeas==


./lse.sh --help
./lse.sh -l 1 (para listar lo critico)

