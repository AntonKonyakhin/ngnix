1. Установите nginx из системных (стандартных) репозиториев ubuntu.
```
apt install nginx
```
2. Создайте два виртуальных хоста:
- Один слушает на порту 81, другой на 82 (должны быть доступны извне)
- При запросе location / они должны отдавать код 200 и текст "Backend 1" или "Backend 2"
3. Создайте третий virtual host reverse.${base_domain} на порту 80:
- При запросе /backend, nginx должен проксировать запрос на бекенды на порту 81 и 82
- При запросе /rebrain, nginx должен проксировать запрос на https://rebrainme.com/


конфиг  
вместо external_ip можно подставить 127.0.0.1, но тогда сервис не будет доступен извне

```
upstream backend {
  server external_ip:81;
  server external_ip:82;

}
server {
  listen external_ip:81;
  server_name _;

  location / {
    return 200 "Backend 1";
  }
}

server {
  listen external_ip:82;
  server_name _;

  location / {
    return 200 "Backend 2";
  }
}

server {
  listen 80;
  server_name reverse.7f9c292de9530033d052fb4afdf0a357.kis.im;

  location /backend {
    proxy_pass http://backend;
  }

  location /rebrain {
    proxy_pass https://rebrainme.com/;

  }


}



```
