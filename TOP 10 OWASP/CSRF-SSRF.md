# **CSRF vs SSRF vs SSTI vs CSTI**

## **1. CSRF (Cross-Site Request Forgery) 🏹**

**Explicación con la Técnica Feynman:**  
Imagina que tienes una cuenta bancaria y cada vez que realizas una transferencia, simplemente haces clic en un botón sin necesidad de ingresar contraseña nuevamente. Un atacante podría engañarte para hacer clic en un enlace malicioso que, sin que lo notes, envía una transferencia desde tu cuenta. **Eso es CSRF**: ejecutar acciones sin tu consentimiento aprovechando que ya tienes sesión iniciada.

### **Ejemplo en la vida real:**

1. Un banco permite cambiar la contraseña con una simple petición GET:
    
    nginx
    

    
    `GET http://bank.com/change_password?new_pass=hacked`
    
2. El atacante envía a la víctima un enlace malicioso:
    
  
    
    `<img src="http://bank.com/change_password?new_pass=hacked">`
    
3. Si la víctima hace clic estando logueada, su contraseña cambiará sin darse cuenta.
    

### **Solución:**

- Usar **tokens CSRF** en formularios (un valor aleatorio único para cada sesión).
    
- Requerir verificación adicional (como ingresar la contraseña para cambios sensibles).
    
- Usar el **método POST** en lugar de GET para acciones críticas.
    

📖 **Material de apoyo:**

- OWASP CSRF Guide
    
- Ejemplo práctico de CSRF
    

---

## **2. SSRF (Server-Side Request Forgery) 🌍**

**Explicación con la Técnica Feynman:**  
Piensa en un restaurante donde solo los meseros pueden entrar a la cocina. Tú, como cliente, no puedes entrar, pero puedes pedirle al mesero que lleve un mensaje al chef. **SSRF es cuando engañamos al servidor para que haga peticiones internas por nosotros**, incluso accediendo a cosas que no deberíamos ver.

### **Ejemplo en la vida real:**

1. Un sitio permite a los usuarios ingresar una URL para obtener datos, como un visor de imágenes:
    
    bash
    
    CopiarEditar
    
    `GET http://victima.com/fetch?url=http://ejemplo.com/imagen.jpg`
    
2. Un atacante cambia la URL para acceder a información interna:
    
    pgsql
    
    CopiarEditar
    
    `GET http://victima.com/fetch?url=http://localhost/admin`
    
3. El servidor, al hacer la petición, devuelve contenido privado que normalmente no es accesible desde el exterior.
    

### **Solución:**

- Restringir las direcciones IP que el servidor puede consultar.
    
- Bloquear acceso a direcciones internas (`localhost`, `127.0.0.1`).
    
- Usar listas blancas de URLs permitidas.
    

📖 **Material de apoyo:**

- OWASP SSRF Guide
    
- SSRF en HackTricks
    

---

## **3. SSTI (Server-Side Template Injection) 📝**

**Explicación con la Técnica Feynman:**  
Imagina que un sitio web te permite personalizar un mensaje como "Hola, Juan". Pero, en lugar de texto plano, el servidor usa una plantilla que evalúa código:

jinja

CopiarEditar

`"Hola, {{ nombre }}"`

Si el servidor no filtra correctamente los datos, un atacante podría inyectar código malicioso en la plantilla:

jinja

CopiarEditar

`{{ 7 * 7 }}  →  49`

O peor aún:

jinja

CopiarEditar

`{{ __import__('os').system('whoami') }}`

Esto permitiría ejecutar comandos en el servidor.

### **Ejemplo de SSTI en Jinja2 (Python):**

1. Un sitio vulnerable evalúa directamente entradas de usuario:
    
    python
    
    CopiarEditar
    
    `from flask import request, render_template_string user_input = request.args.get("name") render_template_string("Hola, " + user_input)`
    
2. Un atacante inyecta código malicioso:
    
    ruby
    
    CopiarEditar
    
    `http://victima.com/?name={{7*7}}`
    
3. El servidor evalúa la expresión y responde:
    
    
    
    `Hola, 49`

Hacer fuerza bruta con Hydra a login paage
hydra -l admin -P /usr/share/wordlists/rockyou.txt "http-post-form://172.17.0.2/cms/admin/login.php:username=^USER^&password=^PASS^&loginsubmit=Submit:User name or password incorrect"


para ver todos los tipos de revershells 
pagina web llamada reverse shell generator
			#tratamientotty 
			 
			Tratamiento de la TTI:
			cd /
			script /dev/null -c bash
			presionamos CTRL+Z
			stty raw -echo; fg
			reset xterm
			export TERM=xterm
			export SHELL=bash
			
			
			Luego redimencionaremos el tamao
			
			stty rows 44 columns 184

Sudo bruteforce en github


