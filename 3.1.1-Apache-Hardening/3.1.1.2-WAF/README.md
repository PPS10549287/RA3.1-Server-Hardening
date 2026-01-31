# Práctica 2: ModSecurity (WAF Base)

### 1. Explicación
Esta imagen hereda la configuración de la **P1 (Hardening de Apache)** y añade una capa de defensa activa mediante un **WAF (Web Application Firewall)**:

* **Estrategia en cascada:** Se utiliza la imagen `pps10549287/pps-pr1` como base, manteniendo CSP, HSTS y SSL.
* **ModSecurity:** Instalación y activación del motor en modo bloqueo (`SecRuleEngine On`).
* **Protección Activa:** El servidor ahora es capaz de interceptar peticiones maliciosas antes de que lleguen a la aplicación.

### A. Contenido del Dockerfile
En este paso, el objetivo es dejar de ser "pasivos". Disponemos de un servidor robusto por fuera (gracias al Hardening de la P1), pero además ahora instalamos un WAF (Web Application Firewall) para que analice el tráfico en tiempo real.

**1. Evolución sobre la base (Herencia)**

Como se ha indicado, partimos con toda la configuración de SSL, HSTS y las cabeceras de seguridad configuradas anteriormente. Únicamente añadimos la capa de protección de aplicación.

**2. Instalación del motor de seguridad**

Instalamos el módulo `libapache2-mod-security2`. Este es el motor que nos va a permitir inspeccionar qué viene dentro de las peticiones HTTP (como parámetros de formularios o cookies) para así detectar ataques que un firewall normal no vería.

**3. Del modo "Detección" al modo "Bloqueo"**

Este es el punto más importante del script. Por defecto, ModSecurity viene configurado para "solo mirar" (DetectionOnly), lo cual no sirve de mucho en producción.

Por ello usamos `sed` para cambiar esa directiva a `SecRuleEngine On`. Gracias a esto, si el firewall detecta algo sospechoso, no solo lo anotará en el log, sino que cortará la conexión de forma inmediata.

**4. Activación en el servidor**

Finalmente, habilitamos el módulo en Apache con `a2enmod`. Como ya tenemos los puertos 80 y 443 abiertos y el comando de arranque configurado de la práctica anterior, el contenedor está listo para filtrar ataques desde el momento en que sea desplegado.

> [!IMPORTANT]
> <img width="696" height="365" alt="image" src="https://github.com/user-attachments/assets/19258c5e-cc01-4440-bab7-6ebed65f3463" />

### 2. Guía de Despliegue
Este repositorio utiliza una imagen preconfigurada alojada en Docker Hub. Al heredar de la P1, los ajustes de endurecimiento y el firewall ya están integrados en la imagen.

**Paso 1: Descargar la imagen**

`docker pull pps10549287/pps-pr2:latest`

**Paso 2: Lanzar el contenedor**
Mapeamos el puerto 8080 para HTTP y el 8081 para HTTPS (puerto 443 interno):

`docker run -d --name pps-pr2-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr2:latest`

**Paso 3: Visualizar contenedor activo**
Comprobamos que el contenedor se encuentra corriendo y está operativo:

`docker ps`

### 3. Validación y Auditoría

Para verificar que el WAF está operativo y bloqueando amenazas, realizamos una prueba de ataque simulado:

**Comprobación de bloqueo (Path Traversal)**

`curl -I "http://localhost:8080/index.php?exec=/bin/bash"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="762" height="141" alt="image" src="https://github.com/user-attachments/assets/65c443d4-8c0f-4bca-bc45-8d6af193c268" />

> [!NOTE]
> *El código **403 Forbidden** indica que el WAF ha detectado un intento de acceso a binarios del sistema a través de la URL y lo ha bloqueado automáticamente.*

**Comprobación de protección en Formularios (POST - XSS)**

`curl -i -X POST http://localhost:8080/post.php -d "campo=<script>alert(1)</script>"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="1029" height="335" alt="image" src="https://github.com/user-attachments/assets/3851a3d5-165b-4d42-aa51-5214126e3250" />

> [!NOTE]
> *El servidor bloquea el intento de inyección de script con un código 403, protegiendo la integridad de la aplicación, indicando además que no tenemos acceso a este recurso.*

**Verificación de que el módulo está cargado**

`docker exec pps-pr2-javlluapa apache2ctl -M | grep security`

> [!IMPORTANT]
> Resultado esperado:
> <img width="1382" height="95" alt="image" src="https://github.com/user-attachments/assets/d9dbe54c-beec-4f8b-9b6c-a664b19043b5" />

> [!NOTE]
> *(Esta salida confirma que el motor ModSecurity está activo y vinculado correctamente al proceso de Apache).*

### 4. URL Docker Hub
`docker pull pps10549287/pps-pr2:latest`

### 5. Limpieza
Para detener y borrar el contenedor de prueba ejecutamos:

`docker stop pps-pr2-javlluapa`

`docker rm pps-pr2-javlluapa`

> [!IMPORTANT]
> Resultado esperado:
>
> <img width="490" height="121" alt="image" src="https://github.com/user-attachments/assets/30c034c3-a331-48d1-9230-3183eef70973" />

### Autor
Javier Lluesma Aparici IES El Caminàs

Puesta en Producción Segura (Especialización Ciberseguridad)

