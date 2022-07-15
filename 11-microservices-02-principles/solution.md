# Как запускать
После написания [nginx.conf](gateway/nginx.conf) для запуска выполните команду
* повысил Flask до 2.1.0 [requirements.txt](security/requirements.txt)
```
docker-compose up --build
```

# Как тестировать

## Login
Получить токен
```
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token
```
Результат
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I
```

## Test
Использовать полученный токен для загрузки картинки
```
curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @1.png http://localhost/upload
```
Результат
```
{"filename":"b62997be-69e4-4508-bdad-2919174280ee.jpg"}
```

 ## Проверить
Загрузить картинку и проверить что она открывается
```
curl localhost/image/b62997be-69e4-4508-bdad-2919174280ee.jpg > b62997be-69e4-4508-bdad-2919174280ee.jpg
```
Результат
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   153  100   153    0     0  38250      0 --:--:-- --:--:-- --:--:-- 38250

$ ls
b62997be-69e4-4508-bdad-2919174280ee.jpg
```
