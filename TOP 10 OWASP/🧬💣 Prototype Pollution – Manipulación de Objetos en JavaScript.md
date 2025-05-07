

### 🧠 **¿Qué es Prototype Pollution?**

**Prototype Pollution** es una vulnerabilidad que permite a un atacante **modificar el prototipo (`__proto__`) de los objetos base en JavaScript**, afectando a **todos los objetos** que heredan de él.  
Esto puede permitir:

- ⚙️ Cambiar valores por defecto
    
- 🚨 Introducir propiedades inesperadas
    
- 🔐 **Escalar privilegios**, como convertir un usuario normal en admin
    

---

### 📦 **Estructura típica de un payload malicioso**

Aquí un ejemplo donde se **eleva el privilegio del usuario a admin** usando `__proto__`:

json

`{   "email": "test@test.com",   "msg": "Hola, esto es una prueba",   "__proto__": {     "admin": true   } }`

🧨 Al inyectar esta estructura, **todos los objetos** creados después **tendrán la propiedad `admin: true`**, incluso si no la tenían definida originalmente.

---

### 🔎 **¿Por qué funciona esto?**

JavaScript usa un modelo basado en **prototipos** para herencia.

Cuando haces algo como:

js


`let user = {}; console.log(user.admin); // undefined`

...pero si antes ejecutaste un payload de pollution como:

js


`Object.prototype.admin = true;`

Entonces:

js


`let user = {}; console.log(user.admin); // true ✅`

El objeto `user` **hereda** la propiedad `admin` desde el prototipo. 😱

---

### 🧰 **Casos de uso malicioso**

- 🔐 Escalada de privilegios en aplicaciones mal validadas.
    
- ⚙️ Alteración del comportamiento interno de librerías vulnerables.
    
- 📂 Manipulación de rutas o configuraciones en objetos globales.
    

---

### 📚 **Material de apoyo (Altamente recomendado)**

👉 [Prototype Pollution – Understanding and Exploiting a Hidden JavaScript Vulnerability](https://medium.com/@albertoc_91016/prototype-pollution-understanding-and-exploiting-a-hidden-javascript-vulnerability-28ea454b1f99)  
✍️ Un excelente artículo que explica la técnica paso a paso, con ejemplos prácticos y código vulnerable para practicar.

---

### 🛡️ **¿Cómo mitigarlo?**

✔️ **Validar siempre** las entradas del usuario antes de mergear objetos.  
✔️ Bloquear claves peligrosas como `__proto__`, `constructor`, y `prototype`.  
✔️ Usar herramientas seguras como `lodash.set` en lugar de hacer merges manuales.  
✔️ Mantener las dependencias actualizadas.

---

### 🔚 **Conclusión**

**Prototype Pollution** es una vulnerabilidad silenciosa pero **extremadamente poderosa** en aplicaciones basadas en JavaScript.  
Una inyección bien colocada puede permitir **control completo de la lógica de la app** con una simple línea de JSON.

💡 **Un solo campo puede cambiarlo todo.**