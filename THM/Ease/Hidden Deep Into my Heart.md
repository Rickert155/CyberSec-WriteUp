# Writeup: Hidden Deep Into my Heart

> Cupid's Vault was designed to protect secrets meant to stay hidden forever. Unfortunately, Cupid underestimated how determined attackers can be.
Intelligence indicates that Cupid may have unintentionally left vulnerabilities in the system. With the holiday deadline approaching, you've been tasked with uncovering what's hidden inside the vault before it's too late.  

Что же, поможем Купидону найти язву и принести флаг, подтверждающий ее наличие. Начнем с изучения главной страницы:
```sh
curl http://10.114.180.129:5000 -I
```
```sh
HTTP/1.1 200 OK
Server: Werkzeug/3.1.5 Python/3.10.12
Date: Tue, 17 Nov 2026 02:24:24 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 1367
Connection: close
```
В ответе ничего особого не видно, как и ссылок на другие страницы. Стоит проверить, какие есть директории и страницы вообще:
```sh
python3 -m cyberwarn fuzz-dirs --url=http://10.114.180.129:5000 --wordlist=data/wordlist/web/top-10k-web-directories_from_10M_urlteam_links.txt --workers=100
```
В результате я получил ответ, где содержалась следующая информация:
```sh
http://10.114.180.129:5000/console 400

Server: Werkzeug/3.1.5 Python/3.10.12
Date: Tue, 17 Nov 2026 02:26:31 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 167
Connection: close
```
Интересно само тело ответа:
```sh
curl http://10.114.180.129:5000/console
```
```sh
<!doctype html>
<html lang=en>
<title>400 Bad Request</title>
<h1>Bad Request</h1>
<p>The browser (or proxy) sent a request that this server could not understand.</p>
```
Такс, а вот в robots.txt тоже есть кое что интересное:
```sh
curl http://10.114.180.129:5000/robots.txt
```
```sh
User-agent: *
Disallow: /cupids_secret_vault/*

# cupid_arrow_2026!!!
```

Для нас становится доступен еще один URL: http://10.114.180.129:5000/cupids_secret_vault/, но от отдает следующий ответ:
```html
            <h1>Cupid's Secret Vault</h1>
            <p>You've found the secret vault, but there's more to discover...</p>
```
Думаю, что имеет смысл запустить сканнер директорий и от сюда:
```sh
python3 -m cyberwarn fuzz-dirs --url=http://10.114.180.129:5000/cupids_secret_vault/ --wordlist=data/wordlist/web/top-10k-web-directories_from_10M_urlteam_links.txt --workers=100
```
И не зря, так как получилось обнаружить новую директорию: *http://10.114.180.129:5000/cupids_secret_vault/administrator*.  
Разумеется, она по авторизацией, по этой причине нам нужно собрать данные, которые у нас уже есть и применить их для атаки на форму авторизации:
- cupid_arrow_2026!!! - нашли в robots.txt  
После нескольких попыток подобрать учетные данные было установлено, что валидны следующие учетные данные:
- username: *admin*
- password: *cupid_arrow_2026!!!*  

После ввода аутентификационных данных мы получили флаг!
