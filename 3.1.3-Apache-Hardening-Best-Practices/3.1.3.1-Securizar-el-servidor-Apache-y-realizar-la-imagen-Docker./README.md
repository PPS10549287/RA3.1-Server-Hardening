# Práctica Final: Gold Image Apache (Geekflare)

## 1. Introducción
Se ha diseñado como una **Golden Image**, una imagen de referencia que consolida todas las capas de seguridad configuradas en las prácticas anteriores, sumando protecciones críticas a nivel de sistema operativo, usuario y protocolo.

La imagen sigue una **estratia de herencia en cascada**, garantizando que este contenedor final sea el más robusto de toda la serie:
* **P1 a P4:** Hardening básico, WAF (ModSecurity), Reglas OWASP y protección Anti-DoS (Mod_Evasive).
* **P5:** Cifrado SSL/TLS y activación real de la política **HSTS**.
* **GOLDEN IMAGE (Final):** Hardening de sistema, gestión de usuarios y blindaje de protocolos.

## 2. Justificación Técnica de la Implementación

De nuevo aprovechamos la base sólida de las prácticas anteriores y añadimos módulos específicos de seguridad que garanticen un despliege incremental y controlado:

### A. Uso de Archivo de Configuración Externo (`a2enconf`)
En lugar de inyectar decenas de líneas mediante comandos `echo` en el archivo principal, de nuevo creamos un archivo externo `geekflare-hardening.conf`. 
Manteniendo la herencia de las prácticas anteriores intacta. Al usar `a2enconf`, Apache carga estas nuevas reglas de seguridad de forma modular, permitiendo una auditoría más clara y evitando errores de sintaxis en el archivo maestro `apache2.conf`.

### B. Uso de `sed` para Variables de Entorno
Se ha utilizado el comando `sed` exclusivamente para modificar el archivo `/etc/apache2/envvars`.
La justificación se debe a que en distribuciones basadas en Debian, el usuario que ejecuta Apache se define en este archivo. El uso de `sed` es la forma más eficiente de realizar este cambio de identidad de forma persistente sin tener que sobrescribir el archivo completo del sistema.

### C. Contenido del Dockerfile

Básicamente, lo que he montado aquí es un proceso de Hardening para asegurar que nuestra instancia de Apache no salga a producción con la configuración por defecto, que suele ser bastante insegura.

**1. El principio de "Menor Privilegio"**
Lo primero que hago es crear un usuario y un grupo específicos (apache). Ya que no queremos que el servicio corra como root. Si alguien consiguiése acceder, quedaría enjaulado como un usuario sin permisos de administrador, asegurando una óptima protección de nuestro servidor.

**2. Introduciendo la configuración "Gold Image"**

Se ha preparado un archivo de configuración (geekflare-hardening.conf) que contiene todas las mejores prácticas de seguridad vistas en el blog. Se copia directamente al directorio de configuraciones disponibles de Apache para que sea la base de confianza.

**3. Activando las herramientas de defensa**

Aquí se activan un par de módulos fundamentales:

`Rewrite`: Para gestionar redirecciones (como forzar todo a HTTPS).

`Headers`: Para inyectar cabeceras de seguridad que protejan al usuario final de ataques como XSS o Clickjacking. Luego, con a2enconf, dejamos activa nuestra configuración de "Gold Image".

**4. Forzando al sistema a usar nuestro usuario**

Apache tiene un archivo de variables de entorno (`envvars`) donde se define quién lo ejecuta. Hacemos uso de un par de comandos `sed` para buscar esas variables y reemplazarlas por nuestro nuevo usuario apache. Asegurando que, al arrancar, el servidor no intente usar el usuario por defecto.

**5. Ajuste de permisos**

Finalmente, cerramos con la gestión de archivos:

Damos permisos de propietario al usuario apache de lo que realmente necesita tocar: los logs y la carpeta de la web.

Restrinjimos el acceso a la carpeta de configuración (`/etc/apache2/`) con un `chmod 750` para que nadie pueda leer o modificar archivos críticos.

> [!IMPORTANT]
> <img width="984" height="550" alt="image" src="https://github.com/user-attachments/assets/e0f3d169-53e3-4029-86c8-2a2f8534a2ec" />

## 3. El Archivo Externo: `geekflare-hardening.conf`
Este archivo de configuración centraliza las directivas de seguridad avanzada que no se cubrieron en fases previas:

* **Hardening de Protocolo:** Incluye `TraceEnable Off` y `FileETag None` para evitar ataques de Cross-Site Tracing y fugas de información del inodo del sistema.
* **Gestión de Timeouts:** Reduce el `Timeout` a 60 segundos para mitigar ataques de denegación de servicio de conexión lenta (Slow Loris).
* **Blindaje de Directorios y Métodos:** * Desactiva el listado de directorios (`-Indexes`) y las inclusiones del lado del servidor (`-Includes`).
    * Implementa `<LimitExcept>`, que actúa como una lista blanca permitiendo solo los métodos `GET`, `POST` y `HEAD`, denegando cualquier otro intento (como `PUT` o `DELETE`).
* **Forzado de HTTP 1.1:** Utiliza el motor de reescritura para rechazar peticiones que utilicen el protocolo HTTP 1.0, eliminando riesgos de seguridad heredados.
* **Seguridad de Capa de Aplicación:** Define las cabeceras `X-Frame-Options` y `X-XSS-Protection` para proteger al cliente final.

> [!IMPORTANT]
> <img width="729" height="679" alt="image" src="https://github.com/user-attachments/assets/18aba9a7-51a2-47f5-a568-9a12704b97ee" />


## 4. Mejoras de Seguridad Implementadas (Capa Final)
* **Usuario sin privilegios:** El servicio Apache se ejecuta bajo el usuario **apache**, aislándolo de `root`.
* **Hardening de Permisos:** Aplicación de permisos restrictivos (`750`) en directorios de configuración.
* **Ocultación de Identidad:** Forzado de `ServerTokens Prod` para enmascarar la infraestructura.



### 5. Guía de Despliegue

**Paso 1: Descargar la imagen**

`docker pull pps10549287/pps-pr-gold:latest`

**Paso 2: Lanzar el contenedor**

`docker run -d --name pps-pr-gold-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr-gold:latest`

**Paso 3: Visualizar contenedor activo**
Comprobamos que el contenedor se encuentra corriendo y está operativo:

`docker ps`

## 6. Validación de la Seguridad (Evidencias)

### A. Prueba Maestra de Herencia y Prioridad de Capas
Para validar que la Gold Image respeta la herencia de las prácticas anteriores y aplica correctamente el endurecimiento final, realizamos una búsqueda recursiva de directivas críticas en todo el árbol de configuración de Apache.

**Comando:** `docker exec pps-pr-gold-javlluapa grep -rE "ServerTokens|ServerSignature|FileETag|TraceEnable" /etc/apache2/`

> [!IMPORTANT]
> **Captura de evidencia (Capas):**
> <img width="1291" height="318" alt="image" src="https://github.com/user-attachments/assets/acd30926-723a-48ff-bd42-0783764a58c3" />

> [!NOTE]
> **Interpretación:** El resultado del escaneo confirma la arquitectura de Defensa en Profundidad mediante la coexistencia de todas las capas:

Capa de Herencia (P1): Se detectan las directivas en `security-hardened.conf` (`ServerTokens ProductOnly` y `ServerSignature Off`), demostrando que el endurecimiento inicial persiste tras todas las fases del proyecto.

Capa Gold Image: Se observan las nuevas restricciones en `geekflare-hardening.conf` (`FileETag None` y `TraceEnable Off`), introducidas específicamente en esta fase final.

Prioridad de Carga: Al utilizar archivos .conf dentro de `conf-available`, Apache aplica los valores más restrictivos de las capas superiores sobre los valores por defecto del sistema, garantizando la robustez del servidor.

### B. Verificación de Identidad y Cabeceras Globales
Realizamos una petición HTTPS para validar el stack completo de seguridad desde el punto de vista del cliente.

**Comando:** `curl -I -k https://www.midominioseguro.com:8081`

> [!IMPORTANT]
> **Captura de evidencia (Cabeceras):**
> <img width="1087" height="294" alt="image" src="https://github.com/user-attachments/assets/e0c6196a-8f1e-42af-a8ef-3101b1b1c51b" />

> [!NOTE]
> **Interpretación:** El servidor responde con el banner oculto (`Server: Apache`), el mecanismo de transporte seguro activo (`Strict-Transport-Security`) y las nuevas protecciones contra Clickjacking y XSS. Se confirma que el servidor no responde ante intentos de reconocimiento de versión.

### C. Verificación de Usuario No Privilegiado
Auditamos los procesos en ejecución para asegurar que un hilo de ejecución comprometido no otorgue acceso de root al atacante.

**Comando:** `docker exec pps-pr-gold-javlluapa ps -ef | grep apache`

> [!IMPORTANT]
> **Captura de evidencia (Procesos):**
> <img width="956" height="125" alt="image" src="https://github.com/user-attachments/assets/020215c4-c64f-4edb-9415-bfbe08cc5681" />

> [!NOTE]
> **Interpretación:** Se observa cómo el proceso padre corre como `root` para gestionar los sockets de red, mientras que los procesos trabajadores (`workers`) que procesan el tráfico externo han cambiado su identidad al usuario **apache**, cumpliendo el principio de mínimo privilegio.

### D. Persistencia de la Herencia (WAF y SSL)
Validamos que las defensas activas de las prácticas anteriores (`ModSecurity`) siguen operando correctamente tras el endurecimiento del sistema.

**Comando (Simulación de ataque):**
`curl -I -k "https://www.midominioseguro.com:8081/?exec=/bin/bash"`

> [!IMPORTANT]
> **Captura de evidencia (Persistencia):**
> <img width="865" height="161" alt="image" src="https://github.com/user-attachments/assets/4ad1d5a8-8e99-4d1e-9c0c-02fdf3b2fa09" />

> [!NOTE]
> **Interpretación:** El servidor devuelve un **403 Forbidden**. Esto confirma que el motor ModSecurity (P2) y las reglas OWASP (P3) siguen inspeccionando el tráfico una vez descifrado por la capa SSL (P5), bloqueando el intento de intrusión.


## 6. URL Docker Hub (Golden Image)
`docker pull pps10549287/pps-pr-gold:latest`

## 7. Conclusión
Este proyecto demuestra una arquitectura de **Defensa en Profundidad** real. Cada práctica ha añadido una capa de robustez que la Gold Image final ha consolidado, resultando en un servidor Apache optimizado para resistir ataques modernos.

### Autor
Javier Lluesma Aparici IES El Caminàs

Puesta en Producción Segura (Especialización Ciberseguridad)
