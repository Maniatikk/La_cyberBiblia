
docker pull ubuntu latest
instalamos php y apache2

Supongamos que hemos vulnerado un servidor y logrado meter una webshell

crearemos el cmd.php

http://localhost/cmd.php?cmd=whoami
tenemos esta url
haremos ahora un netstat -nat
para ver los puertos internos abiertos
http://localhost/cmd.php?cmd=netstat nat

Nos copiamos el cmd.php a /tmp/
levantamos un servicio web con php -S 0.0.0.0:80 (esto hace que desde fuera se pueda detectar con nmap)

Para hacerlo privado lo que hacemos es cambiar el 0.0.0.0:80 a 127.0.0.1:80
tal que asi:

php -S 127.0.0.1:80

ahora hacemos un http://localhost/cmd.php?cmd=ps -faux

![[Pasted image 20250421232246.png]]

Aqui nos aparece el  servicio corriendo

sabiendo eso y que existe el recurso cmd porque lo  hemos ido listando lo que podemos hacer es meternos desde la url dentro de la maquina de la siguiente manera

 http://localhost/cmd.php?cmd=curl localhost:8000/cmd.php?cmd=id

Al estar el servicio corriendo de forma privilegiada lo podemos explotar y conectarnos como root


Caso 2:
Nos vamos a cd /etc/systemd/system

creaermos un servicio que cada cierto tiempo se ejecute un apt update

Creamos un archivo apt-update.service

con nvim creamos apt-update.timer
      │ File: apt-update.timer
───────┼─────────────────────────────────────────────────────────────────────────────────────
   1   │ [Unit]
   2   │ Description=Update package list every 30 seconds
   3   │ 
   4   │ [Timer]
   5   │ OnUnitActiveSec=30s
   6   │ Unit=apt-update.service
   7   │ 
   8   │ [Install]
   9   │ WantedBy=timers.target

y creamos un apt-update.service
─────────────────────────────────────────
   1   │ [Unit]
   2   │ Description=Update package list
   3   │ 
   4   │ [Service]
   5   │ Type=oneshot
   6   │ ExecStart=/usr/bin/apt update

Ejecutamos  los siguientes comandos:
root@parrot /etc/systemd/system $ systemctl daemon-reload         
root@parrot /etc/systemd/system $ systemctl enable apt-update.timer
Created symlink /etc/systemd/system/timers.target.wants/apt-update.timer → /etc/systemd/system/apt-update.timer.
root@parrot /etc/systemd/system $ systemctl start apt-update.timer 

------------
Para listar los servicios que se estahn corriendo podemos hacer un `systemctl list-timers

ahora hacemos systemctl start apt-update.service
Podemos jugar con pspy para detectarlo

find / -name apt-update.timer 2>/dev/null
Unaa vez encontrado el servicio podemos hacerle cat y ver en que consiste
en este caso hace un apt update


si tenemos capacidad de escritura en una ruta como etc/apt/apt.conf

podemos definir politicas para indicar que tareas o comandos queremos que se ejecuten previos a la acutalizacion del sistema


Dentro del directorio etc/apt/apt.conf
creamos un script llamado 01privesc

Buscamos en el navegador apt update Pre Invoke y Post Invoke

en el archivo creado "01privesc" pegamos lo siguiente
![[Pasted image 20250421235456.png]]
hacemos un watch -n 1 list-timers
y  en otra ventana watch -n 1 ls -l /bin/bash
 Como podemos apreciar en automatico se nos otorga privilegios de suid en la bash
 ![[Pasted image 20250421235745.png]]
Solo hacemos bash -p 