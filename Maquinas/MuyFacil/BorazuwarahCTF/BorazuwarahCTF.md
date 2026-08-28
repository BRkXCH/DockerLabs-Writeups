# Writeup: BorazuwarahCTF &mdash; Dockerlabs
- **Dificultad:** Muy Fácil
- **Plataforma:** Dockerlabs
- **IP Objetivo:** 172.17.0.2
- **Técnicas Clave:** Enumeración de Puertos, Fuzzing Web, Análisis de Metadatos, Fuerza Bruta SSH, Escalada de Privilegios

---
### Reconocimiento y Escaneo de Puertos
Se realizó un escaneo completo de puertos TCP sobre la máquina objetivo:

    $ nmap -p- --open -sS -sC -sV 172.17.0.2 -n -Pn

![Nmap Scan](img/nmapScan.png)

#### Puertos Descubiertos:
- **22/tcp**
- **80/tcp**

---
### Enumeración Web y Fuzzing de Directorios
Se ejecutó un fuzzing sobre rutas y directorios mediante *Gobuster:*

    $ gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/directory-lists-2.3-small.txt -x php,html,txt,sh,py,bak,zip --exclude-length 10701

![Gobuster Fuzzing](img/gobuster.png)

#### Resultados:
- ```/index.html``` (Status:200) &mdash; Página principal
- ```/imagen.jpeg``` &mdash; archivo de imagen expuesto tras inspeccionar el index con ```curl```:

![Curl Index](img/curl.png)

---
### Extracción de Metadatos e Información Oculta
Se descargó la imagen a través de ```wget```:

![wget](img/wget.png)

Se procedió a analizar los metadatos de dicha imagen con ```exiftool```:

![exiftool.png](img/exiftool.png)

#### Resultado Obtenido:
El análisis reveló el nombre del usuario válido incrustado en la información de la imagen:
- **Usuario:** borazuwarah

---
### Acceso Inicial mediante Fuerza Bruta SSH
Con el usuario identificado, se ejecutó una entrada de fuerza bruta con *Hydra* contra el servicio SSH utilizando el diccionario *rockyou.txt*:

    $ hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2 -t 16 -f -V -I

![hydraBruteF](img/hydraBruteF.png)

#### Credenciales Obtenidas:
- **Usuario:** ```borazuwarah```
- **Contraseña:** ```123456```

Tras obtener las credenciales, se inició una conexión remota mediante SSH

---
### Escalada de Privilegios (```borazuwarah > root```)
Una vez en el sistema, se listaron los privilegios ```sudo``` asignados al usuario:

![SSH Connection](img/sshConnection.png)

#### Resultado:
El usuario cuenta con permisos para ejecutar ```/bin/bash``` como superusuario sin requerir contraseña.

![flag](img/flag.png)

Se ejecuta el comando ```sudo su /bin/bash``` y por último se confirma el acceso como administrador.
