Writeup: Compromiso y Escalada de Privilegios en Entorno DockerLabs (IP: 172.17.0.3)
Autor: Raúl Lozano Fernández
Contacto: ralofe89@gmail.com
Plataforma: DockerLabs

1. Resumen Ejecutivo
Este documento detalla el proceso de auditoría de seguridad realizado sobre un objetivo Linux (172.17.0.3). La intrusión se logró encadenando una fuga de información en el código fuente web (Information Disclosure) con un ataque de fuerza bruta por diccionario, culminando en una escalada de privilegios a root explotando una mala configuración en los permisos de sudo asociados al editor de texto Vim.

2. Fase de Reconocimiento y Enumeración (Reconnaissance)
La auditoría comenzó con un escaneo de puertos que reveló dos servicios clave expuestos:

Puerto 22: SSH

Puerto 80: HTTP (Apache/2.4.58 en Ubuntu)

Análisis Web e Inteligencia de Fuentes Abiertas (OSINT)
Al interactuar con el servidor web, se descubrió una página de un entrenador personal ("Russoski Coaching"). Al inspeccionar manualmente el código fuente HTML (Ctrl+U), se detectó un comentario oculto dejado por el desarrollador en producción:

HTML
<! -- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente -->
Adicionalmente, en un enlace hacia su repositorio, se identificó una variación del nombre de usuario: russ0ski (con un cero). Esta fuga de información indicó una política de contraseñas extremadamente débil, sugiriendo la reutilización de credenciales y el uso de contraseñas idénticas al nombre de usuario.

3. Fase de Acceso Inicial (Initial Access)
Basado en la recolección de información, se determinaron dos posibles vectores de ataque para el servicio SSH: el usuario russ0ski y el usuario russoski.

Ante la ineficacia de los diccionarios personalizados y las pruebas manuales iniciales, se optó por un ataque de fuerza bruta agresivo utilizando la herramienta Hydra en conjunto con el diccionario estándar rockyou.txt, el cual contiene millones de contraseñas filtradas mundialmente.

Comando ejecutado:

Bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.3 -t 4
Resultado:
El ataque fue exitoso, revelando que el usuario empleaba una contraseña altamente predecible e insegura.

Usuario: russoski

Contraseña: iloveme

Con estas credenciales, se estableció una sesión interactiva SSH exitosa en el sistema objetivo.

4. Fase de Escalada de Privilegios (Privilege Escalation)
Una vez obtenido el acceso inicial como el usuario de bajos privilegios russoski, el objetivo principal fue comprometer el sistema en su totalidad escalando a root.

El primer paso en la enumeración interna fue verificar las configuraciones de sudo para el usuario actual:

Bash
sudo -l
Salida obtenida:

Plaintext
User russoski may run the following commands on faae9a3b4c00:
    (root) NOPASSWD: /usr/bin/vim
Explotación (GTFOBins)
El sistema permitía ejecutar el editor de texto Vim como administrador (root) sin necesidad de ingresar contraseña. Vim posee la funcionalidad de ejecutar comandos de la consola (shell) desde su interfaz. Al ejecutarse con privilegios elevados, cualquier shell invocada hereda dichos permisos.

Para evadir problemas de interfaz interactiva dentro del editor, se aplicó un vector de ataque directo automatizado pasando el comando por argumento (-c):

Bash
sudo /usr/bin/vim -c ':!/bin/bash'
Este comando forzó a Vim a abrir una sesión interactiva de Bash con los privilegios de quien lo ejecutó (root). Tras confirmar el acceso con el comando whoami, se procedió a capturar la bandera final ubicada en /root/root.txt.

5. Medidas de Mitigación y Recomendaciones (Defensa)
Para parchear estas vulnerabilidades en un entorno de producción, se deben aplicar las siguientes medidas de remediación:

Sanitización de Código (Prevención de Information Disclosure): Implementar procesos de CI/CD que eliminen los comentarios HTML y notas de desarrollo antes de desplegar el código a producción.

Políticas de Contraseñas Robustas: Auditar las credenciales del sistema para evitar el uso de claves presentes en diccionarios conocidos (como rockyou.txt) o el reciclaje de credenciales básicas. Implementar autenticación por llaves públicas (SSH Keys) y deshabilitar el acceso SSH por contraseña.

Principio de Menor Privilegio (Sudoers): Nunca asignar permisos NOPASSWD a binarios que posean funciones de escape al sistema operativo (conocidos en GTFOBins como binarios con Shell escape), tales como vim, nano, tar, awk, o less. Si un usuario necesita editar archivos como root, se debe configurar el uso estricto de sudoedit en lugar del binario del editor.
