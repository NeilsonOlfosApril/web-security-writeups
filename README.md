# web-security-writeups
Informes técnicos (write-ups) de laboratorios de ciberseguridad web en PortSwigger Web Security Academy


## Módulo 1: Control de Acceso (Broken Access Control - OWASP A01)
**Plataforma:** PortSwigger Web Security Academy  
**Herramientas utilizadas:** Navegador Web (DevTools: Elements, Console, Application/Cookies)

---

### 1. Funcionalidad de administrador desprotegida (`robots.txt`)
* **Vulnerabilidad:** Exposición de rutas sensibles mediante archivos de indexación y falta de validación de sesión en el servidor (*Seguridad por oscuridad*).
* **Pasos para reproducir:**
  1. Consultar `/robots.txt` en la raíz del dominio.
  2. Identificar la ruta restringida `Disallow: /administrator-panel`.
  3. Navegar directamente a `/administrator-panel` sin iniciar sesión y eliminar al usuario objetivo.
* **Remediación:** Implementar verificación de autenticación y autorización (roles) en el servidor antes de cargar cualquier ruta administrativa, sin depender de ocultar la URL.

---

### 2. Funcionalidad de administrador con URL impredecible (Código fuente JS)
* **Vulnerabilidad:** Divulgación de rutas internas en código JavaScript del lado del cliente sin control de acceso en el backend.
* **Pasos para reproducir:**
  1. Inspeccionar el código fuente HTML (`Ctrl + U`) de la página principal.
  2. Localizar el bloque `<script>` condicional (`if (isAdmin)`) que revela la ruta aleatoria (`/admin-xxxxxx`).
  3. Acceder directamente a dicha ruta en el navegador.
* **Remediación:** No renderizar ni incluir lógica o rutas administrativas en el código entregado a usuarios no autenticados; validar permisos en el servidor.

---

### 3. Rol de usuario controlado por parámetro de solicitud (Cookies)
* **Vulnerabilidad:** Confianza ciega en cookies manipulables por el cliente para determinar privilegios de administrador.
* **Pasos para reproducir:**
  1. Iniciar sesión con credenciales estándar (`wiener:peter`).
  2. Abrir DevTools (`F12`) > `Application` > `Cookies` y modificar la cookie `Admin` de `false` a `true`.
  3. Recargar la página para habilitar y acceder al `Admin panel`.
* **Remediación:** Almacenar los roles y permisos de sesión exclusivamente en el servidor (o mediante tokens firmados criptográficamente como JWT con firma verificada en backend).

---

### 4. Modificación de rol en perfil de usuario (Mass Assignment / JSON)
* **Vulnerabilidad:** Asignación masiva (*Mass Assignment*) al aceptar parámetros no autorizados en peticiones `POST` de actualización de perfil.
* **Pasos para reproducir:**
  1. Analizar la petición `POST /my-account/change-email` y observar que la respuesta JSON devuelve el campo `"roleid": 1`.
  2. Enviar una petición `fetch` desde la consola incluyendo `"roleid": 2` en el cuerpo JSON.
  3. Recargar la sesión con privilegios elevados y acceder al panel administrativo.
* **Remediación:** Filtrar estrictamente los campos permitidos en el backend (*allowlist*) al procesar actualizaciones de usuario, ignorando cualquier parámetro de rol enviado desde el cliente.
