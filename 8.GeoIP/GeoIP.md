1. Установите nginx из системных (стандартных) репозиториев ubuntu.
```
apt install nginx
```

2. Установите модуль geoip из системных репозиториев, а так же geoip-database.
```
apt install libnginx-mod-http-geoip
```

gпроверяем линк
```
/etc/nginx/modules-enabled/50-mod-http-geoip.conf
```
```

root@nginx:~# ls -la /etc/nginx/modules-enabled/
total 16
drwxr-xr-x 2 root root 4096 Jan  7 12:25 .
drwxr-xr-x 8 root root 4096 Jan  7 12:23 ..
lrwxrwxrwx 1 root root   54 Jan  7 12:25 50-mod-http-geoip.conf -> /usr/share/nginx/modules-available/mod-http-geoip.conf
lrwxrwxrwx 1 root root   61 Jan  7 12:23 50-mod-http-image-filter.conf -> /usr/share/nginx/modules-available/mod-http-image-filter.conf
lrwxrwxrwx 1 root root   60 Jan  7 12:23 50-mod-http-xslt-filter.conf -> /usr/share/nginx/modules-available/mod-http-xslt-filter.conf
lrwxrwxrwx 1 root root   48 Jan  7 12:23 50-mod-mail.conf -> /usr/share/nginx/modules-available/mod-mail.conf
lrwxrwxrwx 1 root root   50 Jan  7 12:23 50-mod-stream.conf -> /usr/share/nginx/modules-available/mod-stream.conf
```

добавляем в конфигурацию

geoip_country         /usr/share/GeoIP/GeoIP.dat;

 
 
3. Создайте virtual host для host: country.{base_domain} и пропишите настройки, которые будут реализовывать следующую логику:

   1. При запросе location /country, в ответ должен добавляться header X-Country: и значение - трехбуквенный код страны, код ответа - 200
   
   add_header X-Country $geoip_country_code3
   
   
   2. При запросе location /name, в ответ должен добавляться header X-Country-Name: и значение - название страны, код ответа - 200
   
   add_header X-Country-Name $geoip_country_name;
   
   
 конфиг
 ```
 geoip_country         /usr/share/GeoIP/GeoIP.dat;
  
server {
  listen 80;
  server_name country.6227a62c2a281e4c3ffa6809316ad220.kis.im;

  location /country {
    add_header X-Country $geoip_country_code3;
    return 200;
  }

  location /name {
    add_header X-Country-Name $geoip_country_name;
    return 200;

  }
}

 ```
 
 