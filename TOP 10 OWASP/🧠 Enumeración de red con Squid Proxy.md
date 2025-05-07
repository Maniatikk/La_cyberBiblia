Tras realizar un escaneo ARP en la red, identificamos un host en `192.168.0.249`. Continuamos con un escaneo de puertos utilizando `nmap`, pero notamos que no obteníamos respuestas a través de comandos como `ping` o `curl`. Esto sugiere que los puertos podrían estar **filtrados** (filtered), impidiendo la comunicación directa.

---

### 🔀 Bypass mediante Proxy (Squid)

Probamos acceder usando el puerto `3128`, típico de Squid Proxy, con el siguiente comando:

bash

Copiar código

`curl http://192.168.0.249 --proxy http://192.168.0.249:3128`

Esta vez obtuvimos una respuesta, lo que confirmó que el proxy Squid está funcionando como puerta de entrada al host interno.

Configuramos el navegador con ese proxy para explorar el sitio de forma gráfica, lo que permitió un acceso exitoso a la interfaz web disponible en el host.

---

### 🔎 Descubrimiento de recursos

Una vez dentro, usamos herramientas como `gobuster` para descubrir rutas y posibles subdominios expuestos a través del proxy:

bash

Copiar código

`gobuster dir -u http://target-site.local --proxy http://192.168.0.249:3128 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

---

### 🛠 Enumeración de puertos internos a través del proxy

Creamos un script en Python que permite descubrir puertos internos accesibles a través del Squid Proxy. La lógica es sencilla: realizamos peticiones HTTP a una lista de los puertos TCP más comunes usando el proxy, y si el código de estado **no es 503**, asumimos que el puerto responde de alguna forma (posiblemente indicando un servicio activo).

---

### 📜 Script – Enumeración de Puertos vía Squid Proxy

 #!/usr/bin/python3                                                  14:25:33 [0/1858]
   2   │                                                                              
   3   │ import sys,signal,requests                                                   
   4   │                                                                              
   5   │ def def_handler(sig,frame):                                                  
   6   │     print("\n\n[!] Saliendo...\n")                                           
   7   │     sys.exit(1)                                                              
   8   │                                                                              
   9   │                                                                              
  10   │ signal.signal(signal.SIGINT, def_handler)                                    
  11   │                                                                              
  12   │ main_url = "http://127.0.0.1"                                                
  13   │ squid_proxy = {'http': 'http://192.168.0.249:3128'}                          
  14   │                                                                              
  15   │ def portDiscovery():                                                         
  16   │     common_tcp_ports = {20, 21, 22, 23, 25, 53, 67, 68, 69, 80, 110, 111, 123
       │ , 135, 137, 138, 139, 143, 161, 162, 179, 389, 443, 445, 465, 514, 515, 993,  
       │ 995, 1080, 1433, 1434, 1521, 1723, 2049, 3306, 3389, 5060, 5432, 5900, 5985,       
       │ 5986, 6379, 8080, 8443, 8888, 9001, 9200, 10000, 27017}                      
  17   │                                                                              
  18   │     for tcp_port in common_tcp_ports:                                        
  19   │         r = requests.get(main_url + ':' + str(tcp_port), proxies=squid_proxy)
  20   │                                                                               
  21   │         if r.status_code != 503:                                                   
  22   │             print("\n[+] Port" + str(tcp_port) + " - OPEN")                          
  23   │                                                                                     
  24   │ if __name__ == '__main__':                                                         
  25   │     portDiscovery()          