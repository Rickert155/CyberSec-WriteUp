# Writeup: Fools Mate

> It's mate in one. You know it, the engine knows it, my grandma knows it. The board says checkmate is one click away. The engine says no. Settle the argument.  


Нам предлагают поиграть в шахматы. Сразу откроем DevTools браузера на вкладке Network и последим за запросами, которые отправляет наш браузер. При перемещении фигур в случае удачного хода мы видим обращение к api */api/move*, к примеру, с таким запросом *{"from":"a1","to":"b1"}*  
```sh
curl --json '{"from":"a1","to":"a2"}' http://10.112.148.238/api/move
```
И получим такой ответ
```sh{"ok":true,"move":"a1a2","fen":"7k/5ppp/8/8/8/8/R4PPP/6K1 w - - 2 2","status":"ongoing","turn":"w","botMove":"g8h8"}
```

Мы можем попробовавть поставить мат шагом ладьи A1-A8 - но игра нам этого не позволит этого сделать, нам будет показана ошибка с текстом *I'll shut down your PC if you play that.*. В таком случае справедливо будет попробовать отправить запрос, к примеру, через curl с A1 на A8, возможно, фильтруются запросы только с фронтенда:
```sh
curl --json '{"from":"a1","to":"a8"}' http://10.112.148.238/api/move
```
На этот раз нам получилось поставить мат боту и получить флаг:
```sh
{"ok":true,"move":"a1a8","fen":"R5k1/5ppp/8/8/8/8/5PPP/6K1 b - - 1 1","status":"checkmate","turn":"b","winner":"white","flag":"THM{cl13nt_s1d3_ch3ckm4t3}"}
```

