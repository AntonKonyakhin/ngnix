1. Установите nginx из системных (стандартных) репозиториев ubuntu.
```
apt install nginx
```

2. Установите пакет letsencrypt.

```
apt install letsencrypt
```
3. Создайте 2 virtual host для: ssl.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
- 1-ый Virtualhost должен слушать на 80 порту для протокола http.
- Все запросы на /.well-known должны идти в папку /opt/www/acme.
- Все остальные запросы должны возвращать редирект на https-версию.
- Второй virtualhost должен слушать на 443 порту для протокола https.
- Для этого сервера должны быть прописаны полученные SSL сертификаты.
- Сервер должен поддерживать TLS версии 1.3 и только.
- Сервер https должен возвращать код 201 на все запросы.

создадим сначала папку для сертификатов

```
mkdir -p /opt/www/acme
```

получим сертификат в эту папку

```
letsencrypt certonly --webroot -w /opt/www/acme/ -d ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im -m antonioxxxl@bk.ru --test-cert --agree-tos
```

проверяем
```
root@nginx:/etc/letsencrypt/live/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im# ls -la
total 28
drwxr-xr-x 2 root root 4096 Jan  2 09:15 .
drwx------ 3 root root 4096 Jan  2 09:15 ..
-rw-r--r-- 1 root root  692 Jan  2 09:15 README
lrwxrwxrwx 1 root root   67 Jan  2 09:15 cert.pem -> ../../archive/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im/cert1.pem
lrwxrwxrwx 1 root root   68 Jan  2 09:15 chain.pem -> ../../archive/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im/chain1.pem
lrwxrwxrwx 1 root root   72 Jan  2 09:15 fullchain.pem -> ../../archive/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im/fullchain1.pem
lrwxrwxrwx 1 root root   70 Jan  2 09:15 privkey.pem -> ../../archive/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im/privkey1.pem

```

далее создадим конфигурацию хостов
```
vim /etc/nginx/sites-available/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im
```
```
ln -s /etc/nginx/sites-available/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im /etc/nginx/sites-enabled/ssl.site
```

содержимое конфигурационного файла

```
server {
  listen 80;
  server_name ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im;

  location /.well-known {
    root /opt/www/acme;
  }

  location / {
    return 301 https://ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im$request_uri;
  }

}

server {
  listen 443 ssl;

  ssl_certificate /etc/letsencrypt/live/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im/cert.pem;
  ssl_certificate_key /etc/letsencrypt/live/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im/privkey.pem;
  ssl_protocols TLSv1.3;

  location / {
    return 201;
}
  server_name ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im;


}

```