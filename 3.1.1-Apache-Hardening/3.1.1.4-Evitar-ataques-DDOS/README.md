# Práctica 4: Protección DoS con mod_evasive

### 1. Explicación
Esta imagen hereda la robustez de la **P3 (OWASP CRS)** y añade protección contra ataques de Denegación de Servicio (DoS) y fuerza bruta:

* **Estrategia en cascada:** Basada en la imagen `pps10549287/pps-pr3`, mantiene el Hardening (P1), el motor ModSecurity (P2) y las reglas de OWASP (P3).
* **mod_evasive:** Implementación de un módulo de detección activa que rastrea IPs sospechosas basándose en la frecuencia de sus peticiones a nivel de aplicación.
* **Umbrales de Bloqueo:** Configurado para detectar ráfagas de más de 5 peticiones por segundo a una misma página, bloqueando la IP automáticamente con un código **403 Forbidden** para preservar la disponibilidad del servidor.

### A. Contenido del Dockerfile
En esta etapa, reforzamos la seguridad del servidor contra ataques de denegación de servicio (`DoS`) y ataques de fuerza bruta. Es decir, preparamos al servidor para reconocer cuándo alguien está intentando saturarlo.

**1. Instalación de la defensa (mod-evasive)**

Heredamos todo el trabajo previo (`Hardening` y `ModSecurity`) y le añadimos el módulo `mod-evasive`. También instalamos Perl, necesario para ejecutar el script de auditoría diseñado que permite comprobar que el bloqueo funciona de verdad.

**2. Preparación del terreno**

Creamos un directorio específico para los logs de bloqueo en `/var/log/mod_evasive`. Le damos permisos de propietario a `www-data` para que Apache pueda escribir los registros de las IPs que decide banear en tiempo real.

**3. El script de testeo (Resolución de problemas)**

Aquí he tenido que ser resolutivo ya que el script original me causaba errores `400 (Bad Request)`. Por ello he programado un pequeño script en Perl (`test.pl`) que inyecta correctamente la cabecera `Host: localhost`. Este script permite simular una inundación de peticiones real y verificar que el módulo responde bloqueando el tráfico sospechoso.

> [!IMPORTANT]
> <img width="970" height="923" alt="image" src="https://github.com/user-attachments/assets/a9353f70-eea2-4d86-bf9e-4204530923ec" />

### B. Contenido del archivo evasive.conf

La configuración de este fichero permite al servidor decidir quién es un atacante y quién es un usuario legítimo.

Detección por página (`DOSPageCount 5`): Si una IP pide la misma página más de 5 veces en un segundo, el servidor sospecha y la marca como amenaza.

Detección por sitio (`DOSSiteCount 100`): Si una IP intenta descargar 100 recursos de cualquier parte de nuestra web en un segundo, se activa la alarma.

Baneo (`DOSBlockingPeriod 10`): En cuanto se supera el umbral configurado, la IP entra en "la lista negra" durante 10 segundos. Si el atacante sigue insistiendo durante ese tiempo, el contador se reinicia, manteniéndolo fuera más tiempo.

Memoria interna (`DOSHashTableSize`): Se configura una tabla de 2048 entradas para rastrear las IPs. Tamaño equilibrado que permite monitorizar a muchos usuarios sin consumir demasiada memoria RAM del contenedor.

> [!IMPORTANT]
> <img width="882" height="542" alt="image" src="https://github.com/user-attachments/assets/e5fedf91-3726-497c-99e7-ccf84baf512d" />

### 2. Guía de Despliegue
Este repositorio contiene un bastionado completo de seguridad activa y probada en un entorno "dockerizado".

**Paso 1: Descargar la imagen**

`docker pull pps10549287/pps-pr4:latest`

**Paso 2: Lanzar el contenedor**

Desplegamos el servicio mapeando los puertos de seguridad (8080 para HTTP y 8081 para HTTPS):

`docker run -d --name pps-pr4-javlluapa -p 8080:80 -p 8081:443 pps10549287/pps-pr4:latest`

**Paso 3: Visualizar contenedor activo**

Comprobamos que el contenedor se encuentra corriendo y está operativo:

`docker ps`

### 3. Validación y Auditoría

Para demostrar la efectividad de la protección anti-DoS y la persistencia de las capas anteriores, realizamos las siguientes pruebas:

**A. Ejecución del script de prueba de carga (Perl)**

Utilizamos el script de prueba localizado en `/root/test.pl` que simula una inundación de peticiones HTTP/1.1:

`docker exec pps-pr4-javlluapa perl /root/test.pl`

> [!IMPORTANT]
> Resultado esperado:
>
> <img width="683" height="491" alt="image" src="https://github.com/user-attachments/assets/171ac66f-875d-4d03-8f71-7f654c00bd79" />

> [!NOTE]
> *Se observa cómo las primeras peticiones devuelven **200 OK** y, tras superar el umbral de 5 peticiones, el servidor comienza a responder con **403 Forbidden**, confirmando el bloqueo activo de la IP.*

**B. Verificación de registros de bloqueo (Evidencia de DoS)**
Comprobamos que el módulo ha creado un registro físico de la IP bloqueada en el directorio de logs configurado:

`docker exec pps_p4_full ls -l /var/log/mod_evasive`

> [!IMPORTANT]
> Resultado esperado:
> <img width="762" height="78" alt="image" src="https://github.com/user-attachments/assets/5bc7e612-e924-48fb-8bed-eaec013a87fd" />


> [!NOTE]
> *La existencia del archivo `dos-127.0.0.1` con el propietario `www-data` confirma que el motor de evasión tiene permisos de escritura y está registrando los ataques detectados.*

**C. Prueba de carga masiva con Apache Bench (ab)**

Para generar un informe técnico de denegación de servicio, lanzamos 100 peticiones concurrentes desde el host:

`ab -n 100 -c 5 http://localhost:8080/`

> [!IMPORTANT]
> Resultado obtenido:
> <img width="749" height="990" alt="image" src="https://github.com/user-attachments/assets/23454331-0924-4c72-b749-4d4dd4476e9b" />

> [!TIP]
> **Informe Detallado:** Se adjunta un reporte técnico completo en el archivo [informe_apache_bench.txt](./informe_apache_bench.txt) dentro de este repositorio para la revisión detallada. Este informe no es el que se visualiza en la imagen pertenece a una segunda prueba realizada, ya que si no no se podía mostrar la ejecución tal y como se visualiza en la imagen.

> [!IMPORTANT]
> <img width="858" height="35" alt="image" src="https://github.com/user-attachments/assets/296930a2-2ebd-4f36-83d9-540159071d59" />

> [!NOTE]
> * **Análisis del informe:** Se observa que de las 100 peticiones realizadas, **90 fueron rechazadas (Non-2xx responses)**. Esto demuestra que tras las primeras peticiones exitosas, el módulo `mod_evasive` identificó el exceso de tráfico y activó el bloqueo 403, limitando el consumo de recursos del servidor de forma efectiva.

**D. Persistencia de capas de seguridad (Heredabilidad de P1, P2 y P3)**

Comprobamos que las reglas de OWASP siguen filtrando ataques de inyección (XSS) a pesar de la nueva capa de protección DoS:

`curl -I "http://localhost:8080/?test=<script>alert(1)</script>"`

> [!IMPORTANT]
> Resultado esperado:
> <img width="834" height="167" alt="image" src="https://github.com/user-attachments/assets/16776f36-7b3e-4c00-afa3-bd02dcd2ce73" />

> [!NOTE]
> *El código **403 Forbidden** y la presencia de cabeceras de seguridad de la P1 demuestran que el Hardening y el WAF siguen protegiendo el servidor.*

### 4. URL Docker Hub
`docker pull pps10549287/pps-pr4:latest`

### 5. Limpieza
Para detener y borrar el entorno completo de prueba:

`docker stop pps_p4_full`

`docker rm -f pps_p4_full`

> [!IMPORTANT]
>
> Resultado esperado:
> 
> <img width="503" height="99" alt="image" src="https://github.com/user-attachments/assets/bb29a1b6-49a0-424f-98d2-01730fea8782" />

### Autor
Javier Lluesma Aparici IES El Caminàs

Puesta en Producción Segura (Especialización Ciberseguridad)
