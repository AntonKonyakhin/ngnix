1. Установите nginx из системных (стандартных) репозиториев ubuntu.  
```
apt install nginx
```

2. Сгенерируйте корневой CA сертификат и ключ, а также клиентский сертификат и разместите их в папке /opt/ssl со следующими именами (ключи должны быть формата RSA):  
- ca.crt - корневой CA сертификат;
- ca.key - ключ корневого сертификата;
- client.crt - клиентский сертификат;
- client.key - клиентский ключ.

```
mkdir /opt/ssl
cd /opt/ssl
```
генерируем приватный ключ, который будет использоваться для подписи
```
openssl genrsa -out ca.key 2048
```
создаем корневой сертификат CA, которым мы будем проверять клиентские сертификаты
```
openssl req -x509 -new -key ca.key -days 1000 -out ca.crt
```

создаем клиентский ключ
```
openssl genrsa -out client.key 2048
```

создаем запрос на сертификат
```
openssl req -new -key client.key -out client.csr
```
подписываем

```
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 500
```

добавляем в конфиг server

```
ssl_client_certificate /opt/ssl/ca.crt
ssl_verify_client on;

```

3. Создайте virtual host для host tls.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:
- tls.{base_domain} должен быть доступен по HTTPS с валидным сертификатом, выданным letsencrypt.
- На все запросы должна происходить проверка клиентского сертификата, и в случае успеха nginx должен возвращать стандартный index.html, входящий в поставку.


```
apt install letsencrypt
mkdir -p /opt/www/acme

```
получим сертификат в эту папку (тестовый с ключом --test-cert)

```
letsencrypt certonly --webroot -w /opt/www/acme/ -d ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im -m antonioxxxl@bk.ru --test-cert --agree-tos
```

далее создадим конфигурацию хостов
```
vim /etc/nginx/sites-available/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im

ln -s /etc/nginx/sites-available/ssl.df28d3b0df921d83c82d8f54240d5a11.kis.im /etc/nginx/sites-enabled/ssl.site
```

```
server {
  listen 80;
  server_name tls.d8853c7a9a2b76d8345729b547fa0de2.kis.im;

  location /.well-known {
    root /opt/www/acme;
  }

  location / {
    return 301 https://tls.d8853c7a9a2b76d8345729b547fa0de2.kis.im$request_uri;
  }
}

server {
  listen 443 ssl;

  ssl_certificate /etc/letsencrypt/live/tls.d8853c7a9a2b76d8345729b547fa0de2.kis.im/cert.pem;
  ssl_certificate_key /etc/letsencrypt/live/tls.d8853c7a9a2b76d8345729b547fa0de2.kis.im/privkey.pem;
 
  
  ssl_client_certificate /opt/ssl/ca.crt;
  ssl_verify_client on;

  location /index.html {
    alias /tmp/index.html;
  }

  server_name tls.d8853c7a9a2b76d8345729b547fa0de2.kis.im;
}


```


проверить можно так

```
curl --cert client.crt --key client.key -D - -s https://tls.d8853c7a9a2b76d8345729b547fa0de2.kis.im/index.html -v -k
```

