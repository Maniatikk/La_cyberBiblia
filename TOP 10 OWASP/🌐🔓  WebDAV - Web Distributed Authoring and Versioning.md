### 🧠 ¿Qué es WebDAV?

**WebDAV** (Web Distributed Authoring and Versioning) es una **extensión del protocolo HTTP** que permite a los usuarios:

📂 **Acceder**, 📤 **subir**, 📥 **descargar** y ✏️ **manipular archivos** directamente desde un navegador o herramientas compatibles, como si fuese un sistema de archivos remoto.

---

## 🐳 1. **Entorno de práctica - Docker WebDAV**

Usaremos una imagen de Docker que simula un servidor WebDAV vulnerable.

📦 **Descarga e instalación**:

bash

Copiar código

`docker pull bytemark/webdav`

🚀 Luego puedes ejecutarlo con:

bash

Copiar código

`docker run -d -p 80:80 -e AUTH_TYPE=Basic -e USERNAME=admin -e PASSWORD=admin bytemark/webdav`

🔐 Usuario y contraseña por defecto: `admin / admin`

---

## 🕵️‍♂️ 2. **Reconocimiento - ¿Es realmente WebDAV?**

🔍 Utilizamos la herramienta **WhatWeb** para identificar tecnologías del sitio:

bash

Copiar código

`whatweb http://localhost`

🔁 **Salida esperada**:

css

Copiar código

`HTTPServer[Apache], WebDAV`

📌 Si ves "WebDAV" en los resultados, ¡es