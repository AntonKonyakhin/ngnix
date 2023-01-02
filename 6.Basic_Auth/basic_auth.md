1. Установите nginx из системных (стандартных) репозиториев ubuntu.

```
apt install nginx
```

2. Создайте virtual host для host: auth.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
- Все запросы на / должны быть закрыты аутентификацией с логином / паролем - установите логин user, пароль test
- Все запросы на /auth/off (и вложенные директории) должны проходить без аутентификации
- Все запросы после успешной аутентификации или при отключенной аутентификации должны возвращать код 200


создать user/пароль user/test

```
root@nginx:~# htpasswd -n user
New password: 
Re-type new password: 
```

далее хэш записываем в файл в виде user:password  
/etc/nginx/.htpasswd



```
server {
  listen 80;
  server_name auth.539aa3f760e846a207e3df8a31092a01.kis.im;

  location / {
    auth_basic "TEST";
    auth_basic_user_file /etc/nginx/.htpasswd;
    try_files DAMMY @return200;

  }

  location @return200 {
    return 200;
  }

  location /auth/off/ {
    auth_basic off;
    return 200;

  }
}

```

