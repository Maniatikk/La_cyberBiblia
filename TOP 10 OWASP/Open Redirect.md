Tenemos una pagina que tiene un boton de redireccionamiento por lo tanto abriremos burpsuite para interceptar la peticion
![[Pasted image 20250414230851.png]]
al darle al boton te redirecciona a la pagina /newsite
Interceptaremos con Burpsuite

Vemos la peticion por post que se ha generado
![[Pasted image 20250414231204.png]]

la copiamos e intentamos agregar esto en la url
http://localhost:5000/redirect?newurl=https://google.com
Al redirigirnos a google podemos ver que es vulnerable a open redirect

Esta vulnerabilidad es sencilla

BOTNET UFONET DE EPSILON
Abusa los openredirects para montarse una botnet
En vez de infectar maquinas una por una se monta una base de datos de servidores vulnerables a openredirect  y crea ataques DDOS
					**Github: UFONET**


Ahora lo complicamos un poco mas:

En este caso nos sale una alerta de que no podemos utilizar . en la url
![[Pasted image 20250414232135.png]]

Para ello buscamos alternativas al punto "."
Entonces urlencodeamos el punto "." -> %2e
pero aun sigue detectandolo como punto
Entonces para ello urlencodeamos el % -> %25

Tal que quedaria asi
http://localhost:5000/redirect?newurl=https://google%252ecom

SUBIMOS DE NIVEL
Ahora tenemos de restriccion las barras "//" y el punto "."

PARA HTTPS:// NO HACE FALTA PONER LAS BARRAS
SOLO HTTPS:    (No es necesario poner las barras)
Los open redirects se pueden utilizar para phishing
para robar las credenciales que nos proporcionen


