1.Установите nginx из системных (стандартных) репозиториев ubuntu.
```
apt install nginx
```

2. Создайте virtual host для host regex.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
- Все запросы на /admin/* должны возвращать 301 редирект на страницу /adminochka/* (путь запроса должен сохраняться в новом url).
- Все запросы на /user/* должны уходить на скрипт /index.php?user=* (все, что идет после /user/, должно стать аргументом для index.php?user=).
- Все запросы, которые идут на /index.php, должны возвращать код 200.
- Проверьте, что все location возвращают необходимые значения и отправляйте задание на проверку.

конфигурационный файл находится в папке /etc/nginx/sites-available  
далее нужно сделать ссылку на него в папке /etc/nginx/sites-enabled  
перезапустить nginx  



```
server {
  listen 80;
  server_name ;
  
  location /admin {
    rewrite ^/admin(.*)$ /adminochka/$1 permanent;
	
  }
  
  location /user/ {
    rewrite ^/user/(.*)$ /index.php?user=$1?;
	
  }
  
  location =/index.php {
    return 200;
  }

}
```

