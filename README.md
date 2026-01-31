# Introducción 

El presente proyecto documenta el proceso de diseño, implementación y auditoría de una infraestructura web segura basada en el servidor Apache, utilizando una arquitectura de contenedores Docker. El objetivo fundamental es la creación de una Golden Image que consolide diversas capas de protección, siguiendo una metodología incremental de bastionado (hardening). 

A lo largo de las distintas fases, se han integrado medidas de seguridad críticas que abarcan desde la reducción de la superficie de exposición y la gestión de cabeceras HTTP, hasta la implementación de un WAF (Web Application Firewall) con reglas OWASP, la mitigación de ataques de Denegación de Servicio (DoS) y el cifrado de comunicaciones mediante SSL/TLS. Cada etapa hereda las configuraciones de la anterior, garantizando así un sistema robusto, resiliente y alineado con los estándares de seguridad de la industria. 

# 3.1.1-Apache-Hardening

# 3.1.2-Certificados

# 3.1.3-Apache-Hardening-Best-Practices

# 3.1-Memoria

# Bibliografía y otras fuentes consultadas 

Para el desarrollo de este proyecto se han utilizado recursos oficiales, documentación técnica y guías de referencia en ciberseguridad: 

* **Repositorio Oficial de Imágenes (Docker Hub):** 

Perfil de Usuario pps10549287 - Almacenamiento y registro de las imágenes generadas en cada práctica.
https://hub.docker.com/u/pps10549287

* **Repositorio de Código Fuente (GitHub):** 

RA3.1-Server Hardening - Código y Dockerfiles - Documentación técnica y scripts de construcción. 
https://github.com/PPS10549287/RA3.1-Server-Hardening/blob/main/README.md

* **Documentación de Referencia (Hardening y Seguridad Web):** 

Segarra, P. (2021). Hardening de Servidor Web. Ciberseguridad-PePS 
https://psegarrac.github.io/Ciberseguridad-PePS/tema3/seguridad/web/2021/03/01/Hardening-Servidor.html

Segarra, P. (2020). Implementación de SSL/TLS. Ciberseguridad-PePS 
https://psegarrac.github.io/Ciberseguridad-PePS/tema1/practicas/2020/11/08/P1-SSL.html

Geekflare. (2023). Apache Web Server Hardening & Security Guide. Geekflare Cybersecurity 
https://geekflare.com/cybersecurity/apache-web-server-hardening-security/

* **Otras herramientas: Documentación oficial de Apache HTTP Server, ModSecurity Core Rule Set (CRS) y manuales de Docker Engine.** 

# Conclusión 

Este proyecto demuestra una arquitectura de Defensa en Profundidad real. Cada práctica ha añadido una capa de robustez que la Gold Image final ha consolidado, resultando en un servidor Apache optimizado para resistir ataques modernos. 

### Autor
Javier Lluesma Aparici IES El Caminàs

Puesta en Producción Segura (Especialización Ciberseguridad)
