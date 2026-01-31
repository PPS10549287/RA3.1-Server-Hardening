# Práctica 3: ModSecurity + OWASP CRS

### 1. Explicación
Con esta imagen aplicamos un nivel más de seguridad en la serie, heredando la configuración de la **P2 (WAF Base)** e integrando el conjunto de reglas más reconocido de la industria:

* **Estrategia en cascada:** Se utiliza la imagen `pps10549287/pps-pr2` como base, manteniendo el Hardening de la P1 y el motor ModSecurity de la P2.
* **OWASP Core Rule Set (CRS):** Integración de reglas avanzadas para proteger contra el "Top 10" de riesgos de seguridad (SQLi, XSS, Local File Inclusion, etc.).
* **Protección Inteligente:** El servidor ahora cuenta con una lógica de detección mucho más profunda y precisa gracias al repositorio oficial de `coreruleset`.

### A. Contenido del Dockerfile
En la fase anterior instalamos el motor del firewall (ModSecurity), en esta lo configuramos haciendo uso de las reglas estándar de la industria.

**1. Preparación del entorno y persistencia**

Heredamos la base de Hardening y ModSecurity, pero ahora añadiendo un directorio de caché (`/var/cache/modsecurity`).

Esto es clave ya que ModSecurity necesita un lugar donde guardar datos temporales de las sesiones y las IPs. En caso de no crear este directorio con los permisos correctos para `www-data`, el firewall daría error intentando rastrear ataques complejos.

**2. Integración de OWASP CRS (Core Rule Set)**

Hacemos uso de `git` para descargar directamente desde el repositorio oficial el conjunto de reglas de OWASP.

En lugar de usar versiones obsoletas, clonamos la última versión de `coreruleset`. A continuación clonamos el archivo de configuración base (`crs-setup.conf`) y todas las reglas dinámicas a la carpeta de `ModSecurity`. Con esto, el servidor ya sabe reconocer ataques de Inyección SQL, Cross-Site Scripting (XSS) y mucho más.

**3. Orquestación de las reglas**

No basta con descargar las reglas, debemos indicar a Apache en qué orden leerlas. Para ello sobreescribimos el archivo `security2.conf`. Gracias a la herencia de Docker, la configuración anterior (`SSL, HSTS, Hardening`) sigue funcionando sin tener que escribir ni una línea extra de código.

> [!IMPORTANT]
> <img width="939" height="697" alt="image" src="https://github.com/user-attachments/assets/0b51fe63-ef18-46ee-9c46-8a204176047d" />

### B. Contenido del fichero security2.conf
Este archivo indica al firewall como debe preoceder, definiendo el orden lógico para que no haya conflictos:

SecRuleEngine On: Aseguramos que el motor no solo esté instalado, sino activo. Sin esto, el firewall estaría en modo "solo lectura" o apagado.

Configuración Base: Primero cargamos `modsecurity.conf` donde están las reglas genéricas y la configuración de logs definida en la práctica anterior.

Configuración de OWASP (`crs-setup.conf`): Antes de aplicar las reglas de ataque, cargamos este archivo que define los umbrales de puntuación (la gravedad del comportamiento para ser bloqueado).

Inyección de Reglas (`rules/*.conf`): Finalmente, cargamos las reglas de OWASP. Es el último paso debido a que estas reglas utilizan toda la configuración cargada en los puntos anteriores para decidir si bloquean o permiten una petición.

> [!IMPORTANT]
> <img width="696" height="365" alt="image" src="https://github.com/user-attachments/assets/ecca2b90-5552-4438-9b48-e98b420b201d" />

### 2. Guía de Despliegue
Este repositorio utiliza la imagen final con el conjunto de reglas OWASP ya preconfigurado y optimizado para su carga.

**Paso 1: Descargar la imagen**

`docker pull pps10549287/pps-pr3:latest`

**Paso 2: Lanzar el contenedor**
Mapeamos los puertos de servicio (8080 para HTTP y 8081 para HTTPS):

`docker run -d --name pps-pr3-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr3:latest`

**Paso 3: Visualizar contenedor activo**
Comprobamos que el contenedor se encuentra corriendo y está operativo:

`docker ps`

### 3. Validación y Auditoría

Al contar con el Core Rule Set de OWASP, realizamos pruebas de intrusión para verificar la respuesta del firewall:

**A. Bloqueo de ataque avanzado (Path Traversal)**

`curl -I "http://localhost:8080/?exec=/../../"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="651" height="144" alt="image" src="https://github.com/user-attachments/assets/4f59283e-f207-4cc6-acca-a10288c97785" />

> [!NOTE]
> *El código **403 Forbidden** confirma que las reglas de OWASP han identificado el patrón de navegación prohibida por directorios.*

**B. Bloqueo de ataque avanzado (Command Injection)**

`curl -I "http://localhost:8080/?exec=/bin/bash"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="672" height="144" alt="image" src="https://github.com/user-attachments/assets/d66dad38-4a9b-48cb-bd29-e995f972446a" />

> [!NOTE]
> *El código **403 Forbidden** confirma que las reglas de OWASP han identificado el patrón de navegación prohibida por directorios.*

**C. Visualización de Logs (Auditoría de OWASP)**
Para ver cómo el servidor "caza" el ataque en tiempo real:

`docker exec pps-p3-javlluapa tail -f /var/log/apache2/error.log`

> [!IMPORTANT]
> Resultado esperado (Ataque 1):
> <img width="1185" height="1081" alt="image" src="https://github.com/user-attachments/assets/71b3a817-8b37-427e-8ec0-aeae86497458" />

> [!NOTE]
> *En los logs se puede observar el ID de la regla de OWASP activada y la descripción detallada del ataque bloqueado.*

> [!IMPORTANT]
> Resultado esperado (Ataque 2):
> <img width="1177" height="410" alt="image" src="https://github.com/user-attachments/assets/d418a5b5-4750-4392-9279-dbca3567aca6" />

> [!NOTE]
> *En los logs se puede observar el ID de la regla de OWASP activada y la descripción detallada del ataque bloqueado.*

**D. Verificación de persistencia del Hardening (Capa 1)**

`curl -I http://localhost:8080`

> [!IMPORTANT]
> Resultado esperado:
> <img width="1084" height="282" alt="image" src="https://github.com/user-attachments/assets/51c5574b-8309-4286-ad45-a3c5cd7adf1a" />

> [!NOTE]
> *Se comprueba que la ocultación del servidor y las cabeceras de seguridad de la P1 siguen vigentes gracias a la herencia de imágenes.*

### 4. URL Docker Hub
`docker pull pps10549287/pps-pr3:latest`

### 5. Limpieza
Para detener y borrar el contenedor de prueba ejecutamos:

`docker stop pps-pr3-javlluapa`

`docker rm pps-pr3-javlluapa`

> [!IMPORTANT]
> Resultado esperado:
>
> <img width="498" height="92" alt="image" src="https://github.com/user-attachments/assets/4b601ee2-a2f1-4c90-b389-62c3d71b866a" />

### Autor
Javier Lluesma Aparici IES El Caminàs

Puesta en Producción Segura (Especialización Ciberseguridad)
