 Session Puzzling / Session Fixation / Session Variable Overloading

instalamos y activamos el contenedor de docker skflabs de docker sessionpuzzle y accedemos a la web

ingresamos con las credenciales admin admin
inspeccionamos y copiamos la cookie de sesion
accedemos a la pagina jason web token
jwt.io
![[Pasted image 20250416170741.png]]
nos deslogueamo y vemos que ya no tenemos ninguna cookie de sesion

Entramos al campo de forgot password una vez dentro ponemos el usuario al que queremos recuperar la contraseña e interceptamos la peticion con burpsuite

Al enviar la peticion al repeate y le damos a send podemos observar que nos setea una cookie de sesion
![[Pasted image 20250416171136.png]]

esa misma cookie la vemos en jason web token y observemos su contenido
Nos percatamos que es una cookie de admin
![[Pasted image 20250416171353.png]]
Entonces entramos a la pagina dashboard (que solo nos pertmitia entrar con usuario y contraseña) con la cookie que se genero en forgot password y vemos que hemos obtenido acceso

Lo que se trata de contemplar es que se persuada al  para secuestrar su sesion
