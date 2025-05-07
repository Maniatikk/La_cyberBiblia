### 🧠 **¿Qué es una transferencia de zona?**

Una **transferencia de zona** es una **funcionalidad legítima** del protocolo DNS (Domain Name System) que permite a los **servidores DNS autoritativos compartir información** entre sí para mantenerse sincronizados.

🔁 Esta transferencia puede realizarse mediante dos tipos de solicitudes:

- **AXFR**: Transferencia completa de zona (todos los registros).
    
- **IXFR**: Transferencia incremental (solo los cambios desde la última copia).
    

---

### ❗ **¿Dónde está el problema?**

Cuando **la transferencia de zona no está restringida** correctamente, un atacante externo puede aprovechar esta característica para **extraer toda la base de datos DNS de una organización**.  
Eso incluye:

🔍 **Información privilegiada como**:

- Subdominios ocultos (paneles internos, sistemas de pruebas)
    
- IPs internas o públicas de infraestructura
    
- Servidores de correo (MX)
    
- Nameservers secundarios
    
- Posibles vectores de entrada para otros ataques
    

---

### 🧰 **Herramientas utilizadas**

#### 📌 `dig` (Domain Information Groper)

Una poderosa herramienta para realizar consultas DNS desde consola.

---

### 🚀 **Fase 1: Recolección de información**

#### 🔍 **Listar servidores de nombres (NS):**

bash

Copiar código

`dig ns @127.0.0.1 kencorp.local`

🧠 Esto nos da los **nameservers autoritativos** del dominio objetivo.

---

#### 📧 **Listar servidores de correo (MX):**

bash

Copiar código

`dig mx @127.0.0.1 kencorp.local`

💡 Útil para enumerar posibles vectores de ataque hacia servicios de correo electrónico.

---

### 💥 **Fase 2: Ataque de transferencia de zona (AXFR)**

bash

Copiar código

`dig axfr @127.0.0.1 kencorp.local`

✅ Si el servidor **no está correctamente configurado**, devolverá **todos los registros DNS** asociados al dominio.

📂 Ejemplo de registros obtenibles:

- A (Direcciones IP de hosts)
    
- CNAME (Aliases)
    
- MX (Correo)
    
- TXT (posible información sensible como configuraciones SPF)
    
- SRV (Servicios activos en la red)
    

---

### 🧨 **¿Qué puedes hacer con esa información?**

1. **Enumeración de subdominios** 🕵️‍♂️  
    ➤ Identificar hosts internos como `dev.kencorp.local`, `vpn.kencorp.local`, `intranet.kencorp.local`, etc.
    
2. **Ampliar la superficie de ataque** 🧱  
    ➤ Al conocer más servicios expuestos, puedes probar otras técnicas como:
    
    - Escaneo de puertos
        
    - Fuerza bruta de login
        
    - LFI/RFI/SQLi si hay paneles web
        
3. **Ingeniería social** 🎭  
    ➤ Conocer nombres internos o correos puede ayudarte a construir correos de phishing mucho más realistas.
    
4. **Pivoting en redes internas** 🔀  
    ➤ Si la información revela hosts internos, podrías usarla al escalar privilegios en una intranet o red comprometida.
    

---

### 🔒 **Recomendaciones para mitigar este ataque**

✅ **Restringir las transferencias de zona** solo a servidores DNS autorizados.  
✅ Utilizar firewalls para limitar quién puede hacer consultas DNS internas.  
✅ Habilitar monitoreo de intentos de transferencia sospechosos.  
✅ Utilizar DNSSEC y segmentar la información sensible en zonas separadas.

---

### 🧩 **Resumen visual**

|Acción|Herramienta|Comando|
|---|---|---|
|Listar nameservers|`dig`|`dig ns @127.0.0.1 kencorp.local`|
|Listar servidores MX|`dig`|`dig mx @127.0.0.1 kencorp.local`|
|Transferencia de zona|`dig`|`dig axfr @127.0.0.1 kencorp.local`|