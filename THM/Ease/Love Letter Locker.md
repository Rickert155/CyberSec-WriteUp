# Writeup: Love Letter Locker

> Welcome to LoverLetterLocker, where you can safely write and store your Valentine's letters. For your eyes only?  

Перейдем сразу к веб-приложению: http://10.114.147.77:5000  
Стоит проверить возможные директории/файлы, я это сделал через свой [CyberWarn](https://github.com/rickert155/CyberWarn). В конечном итоге можно отметить эти страницы:
```sh
http://10.114.147.77:5000/register
http://10.114.147.77:5000/login
http://10.114.147.77:5000/letters
```
Для теста я принял решение зарегистрировать учетную запись с данными:
- username: *admin*
- password: *tester123*  

Приложение начало ругаться на то, что такое имя пользователя уже существует. В таком случае методом подбора в рамках CTF мы сможем подбирать никнеймы пользователей веб-приложения, если нам доведется это делать. В итоге для теста была успешно зарегистрирована учетка со следующими данными:
- username: *test*
- password: *tester123*  

Из функционала сразу бросается кнопка *New Letter*, ведущая на *http://10.114.147.77:5000/letters/new*. Давайте попробуем создать письмо и проверить, что доступно пользователю при создании такого письма.  

После того, как мы создали письмо, нас перекидывает на */letter*, где отображаеются наши письма, и у каждого письма есть своя ссылка такого типа: *http://10.114.147.77:5000/letter/3*  
Попробуем манипулировать URL-адресом для получения доступа к письмам, не относящимся к нашем пользователю.  
По URL *http://10.114.147.77:5000/letter/2* нам становится доступно для просмотра письмо другого пользователя:
```sh
This city is a neon fever dream, but you’re the only thing that feels real. My heart’s doing 120 in a rented convertible with the check-engine light on, and somehow you’re the steering wheel.
Love you. Terrifyingly. 💘
```
Что же, в письме *http://10.114.147.77:5000/letter/1* содержится сам флаг:
```sh
My dearest...

THM{1_c4n_r3ad_4ll_l3tters_w1th_th1s_1d0r}

Forever yours,
Gonz0
```
