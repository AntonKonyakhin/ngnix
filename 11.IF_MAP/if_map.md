1. Установите nginx из системных (стандартных) репозиториев ubuntu.
```
apt install nginx
```

2. Для выполнения задания используйте динамический модуль geoip первой версии (как было в 8 задании).

```
apt install libnginx-mod-http-geoip
```
добавляем в конфиг(http {})
```
geoip_country         /usr/share/GeoIP/GeoIP.dat;
```

3. Используя if & map, настройте виртуальный хост access.${base_domain} следующим образом:
- Создайте map - $blocked, который зависит от страны пользователя (двухбуквенный код страны) следующим образом:
- Все запросы из стран ru / ua / us должны быть разрешены (возвращать 200)
- При запросе на /index.html, nginx должен возвращать стандартый index.html, входящий в поставку, для стран ru / ua / us.
- Все запросы из остальных стран должны быть запрещены (возвращать 403)
- При проверке условия в if запрещается использовать сравнение (знак =)
- Ваш сервер должен обслуживать все запросы на 80 порту, как сервер по умолчанию
- Не забудьте выключить default сервер и удалить его конфигурационный файл

map $geoip_country_code $blocked {
  default block;
  RU allow;
  UA allow;
  US allow;
}




server {
  if ($blocked != allow) {
    return 403;
  
  }
  
  location / {
    return 200;
  
  }
  

}