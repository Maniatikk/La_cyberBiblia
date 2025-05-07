# 🔥 **Herramienta de Pruebas de Penetración para Aplicaciones Web**

En el ámbito del **pentesting** (pruebas de penetración), existen herramientas especializadas para detectar vulnerabilidades en aplicaciones web. Estas herramientas permiten automatizar ataques y evaluar la seguridad de los sistemas.

## 🛠️ **Tipos de Ataques en Aplicaciones Web**

Durante las pruebas, se pueden emplear diferentes tipos de ataques, cada uno con un propósito específico.

### 🎯 **1. Sniper**

El modo **Sniper** permite realizar ataques dirigidos utilizando **un solo payload**. Es especialmente útil para realizar ataques de **fuerza bruta**, como:  
✅ Probar listas de usuarios.  
✅ Intentar combinaciones de contraseñas.

🔹 **Ejemplo:** Se puede cargar una **wordlist** con posibles contraseñas y aplicarlas secuencialmente contra un campo de login.

---

### 💥 **2. Cluster Bomb**

El modo **Cluster Bomb** permite cargar **dos payloads simultáneamente**, lo que lo hace ideal para ataques combinados.

✅ Ataque en paralelo a múltiples campos (ejemplo: usuario y contraseña).  
✅ Mayor capacidad de prueba en comparación con Sniper.

🔹 **Ejemplo:**

- Un payload con posibles nombres de usuario (**admin, root, user**).
- Otro payload con contraseñas comunes (**123456, password, qwerty**).
- Se prueban todas las combinaciones posibles para encontrar credenciales válidas.