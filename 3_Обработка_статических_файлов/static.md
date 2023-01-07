Обработка статических файлов

1. Установите nginx из системных (стандартных) репозиториев ubuntu.

```
apt install nginx
```

2. Создайте директорию /opt/www/files и внутри создайте два файла - test1.html и test2.html, внутри которых должна находиться строка test1 и test2, соответственно.

```
mkdir -p /opt/www/files && echo test1 > /opt/www/files/test1.html
mkdir -p /opt/www/files && echo test2 > /opt/www/files/test2.html
```
3. Создайте файл /tmp/test3.html со следующим содержимым: test3.

```
echo test3 > /tmp/test3.html

```

4. Создайте virtual host для host files.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
- Все запросы на / должны отдавать файлы из директории /opt/www/files.{base_domain}.
- Если файл не найден - то должен отдаваться код ответа 204.
- При запросе /heya.html должен отдаваться файл /tmp/test3.html.

напоминаю, что при установки nginx из ситемного репозитория ubuntu, конфиг сайтов хранится в папках   
/etc/nginx/sites-available
/etc/nginx/sites-enabled - здесь надо создать ссылку


создание ссылки  

```
ln -s /etc/nginx/sites-available/files /etc/nginx/sites-enabled/files

```
не забывать перезапускать nginx при изменнии

проверить доступность сайта

```
curl -D - -s http://files/heya.html
```



файл конфигурации

```
server {
  listen 80;
  server_name files;

  location / {
    root /opt/www/files;
        try_files $uri @notfound;
  }

  location @notfound {
    return 204;
  }

  location = /heya.html {
    rewrite heya.html$ /test3.html break;
    root /tmp;
}

}

```