---

---

Material de apoyo Hacktricks Graphql
------------------
IDORs:
skf labs/graphql/idor

interceptamos la pagina de settings con burpsuite y le damos a forward y podemos ver que se hace una peticion por ppst
la pasamos al repeater y vemos un campo de id 
lo modificamos de 1 a 2 y vemos que hemos accedido a otro usuario
![[Pasted image 20250419020212.png]]

y cambiando el id podemos ver el resto de users

-----------------------
Accedemos a /graphql-instrospection

accedemos a /graphql en la url y vemos que podemos ingresar querys

Visitamos hacktricks graphql para copiar la query que utilizaremos

Por medio de instrospection es crear una query y la representamos en graphquery voyager en github
https://graphql-kit.com/graphql-voyager/

![[Pasted image 20250419021402.png]]

de hack tricks copiamos este payload y lo pegamos en la url
Una vez dentro modificamos la primera linea

	on Type a on __Type
	Hacemos lo mismo a InputValue y Type
	lo que salte como error en la linea 
	le agregamos __



![[Pasted image 20250419021747.png]]
La query que nos arroja la copiamos al visualizador
![[Pasted image 20250419021804.png]]

Regresamos al repeater de burp
![[Pasted image 20250419022059.png]]
Configuramos el payload conforme a la query que visualizamos


-----------------------------
Mutations

Vemos que podemos crear un post

Interceptaremos mediante POST la peticion
![[Pasted image 20250419022327.png]]

Ahora podemos observar que tenemos la seccion de mutation
![[Pasted image 20250419022615.png]]

Al cambiar id podemos cambiar de usuarios para crear los post desde multiples usuarios con solo cambiar el id 