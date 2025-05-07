
#### 🚀 1. **Descarga del entorno vulnerable**

Ejecutamos el siguiente comando para descargar la imagen oficial desde Docker Hub:

bash

Copiar código

`docker pull bkimminich/juice-shop`

> 🐳 ¡Juice Shop listo para la acción!

---

#### 🕵️‍♂️ 2. **Intercepción del registro con Burp Suite**

Durante el proceso de **registro de usuario**, interceptamos la solicitud con **Burp Suite** y observamos lo siguiente en el cuerpo de la petición:

json

Copiar código

`{   "email": "usuario@correo.com",   "password": "password123",   "role": "customer" }`

➡️ El parámetro `role` se establece por defecto como `customer`.

---

#### 💥 3. **Explotación: Cambio de rol a `admin`**

Desde **Burp Repeater**, modificamos el valor del rol:

json

Copiar código

`{   "email": "usuario@correo.com",   "password": "password123",   "role": "admin" }`

🔁 Al reenviar la solicitud, el servidor **acepta el nuevo rol** y se crea la cuenta con permisos de **administrador**.

✅ **Resultado:** Acceso exitoso como **admin** sin autenticación adicional.  
🧨 ¡Vulnerabilidad de _Mass Assignment_ explotada con éxito!

📸

---

### 🧪 **Entorno 2: OWASP SKF - Parameter Binding**

---

#### 📥 1. **Clonamos un segundo entorno para más práctica**

bash

Copiar código

`docker pull blabla1337/owasp-skf-lab:parameter-binding`

> 🎯 Laboratorio enfocado en vulnerabilidades de _parameter binding_.

---

#### 🛠️ 2. **Repetición del ataque con otro parámetro**

Durante el análisis, identificamos un nuevo campo manipulable. Añadimos el parámetro:

ini

Copiar código

`is_admin=true`

➡️ El servidor **no valida adecuadamente** los campos recibidos, permitiendo otra vez una elevación de privilegios sin autorización.

🔓 Resultado: acceso como **admin** simplemente agregando un parámetro adicional.

![[Pasted image 20250414170333.png]]

