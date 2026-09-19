# 🎯 DockerLabs CTF: The Aesthetic Dream

Este directorio contiene la documentación técnica y el proceso de explotación paso a paso de una máquina Linux vulnerable dentro del entorno de **DockerLabs**. 

El objetivo de este proyecto es demostrar una metodología estructurada de pruebas de penetración (*pentesting*), combinando el análisis manual de vulnerabilidades web con técnicas automatizadas de fuerza bruta y escalada de privilegios local.

## 🛠️ Tecnologías y Herramientas Destacadas

*   **Nmap:** Utilizado en la fase inicial de reconocimiento de red para la enumeración de puertos abiertos y servicios (SSH, HTTP).
*   **Hydra:** Implementado para ejecutar un ataque de fuerza bruta contra el servicio SSH. El ataque se fundamentó en inteligencia de fuentes abiertas (OSINT) recolectada en la web y el uso táctico del diccionario masivo `rockyou.txt`.
*   **GTFOBins:** Metodología clave aplicada para la fase de *Privilege Escalation*. Se explotó una vulnerabilidad de configuración en `sudoers` (permisos `NOPASSWD` sobre `/usr/bin/vim`), forzando un escape de la interfaz (*shell escape*) para obtener acceso interactivo total como `root`.
*   **Análisis Manual (OSINT):** Inspección de código fuente HTML para identificar fugas de información (*Information Disclosure*) que revelaron patrones de reutilización de credenciales.

## 📖 Contenido del Directorio

*   `writeup.md`: El informe técnico completo que detalla las fases de Reconocimiento, Acceso Inicial, Escalada de Privilegios y las estrategias de Defensa/Mitigación recomendadas.
*   *(Opcional) Puedes añadir aquí una carpeta `/img` si decides incluir capturas de pantalla de la intrusión.*

## 👨‍💻 Autor

**Raúl Lozano Fernández**
*   **Contacto:** ralofe89@gmail.com
*   **Perfil:** Orientado a la ciberseguridad, análisis de datos y desarrollo de soluciones automatizadas.