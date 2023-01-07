
1.  Установите nginx из официальных репозиториев  
инструкции на сайте http://nginx.org/en/linux_packages.html  
гуглится по запросу apt repo nginx  

Например, для ubuntu ветка mainline  ставится так

Install the prerequisites:

```
sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
```

Import an official nginx signing key so apt could verify the packages authenticity. Fetch the key:
```
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

Verify that the downloaded file contains the proper key:
```
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
```

If you would like to use mainline nginx packages, run the following command instead:

```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

Set up repository pinning to prefer our packages over distribution-provided ones:
```
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

To install nginx, run the following commands:
```
sudo apt update
sudo apt install nginx
```

2. Установка количества обработчиков запросов (workers) равное 4 и максимальное количество соединений для одного обработчика - 2048. (рекомендовано кол-во обрабочиков равное кол-во ядер)

Открываем файл /etc/nginx/nginx.conf  

```
### Секция main
worker_processes 4;
```
```
### Секция events
events {
	worker_connections 2048;
}
```

3. максимальный размер файла, отправляемого клиентами, равным 50 Mb.  

```
### Секция http
http {
    client_max_body_size 50M;
    }
```
4. три виртуальных хоста:  
 - alpha - слушать запросы на 80 порту, должен возвращать код 201 и текст "ALPHA".
 - beta - слушать запросы на 80 порту, по умолчанию этот сервер должен отвечать на запросы. Если не указан Host, должен возвращать код 202 и текст "BETA".
 - gamma - слушать запросы на 8080 порту, должен возвращать редирект на "https://kuda-nibud.com" (код ответа 301).  

открываем файл /etc/nginx/conf.d/default  
удаляем из него все через vim (gg > dG)  

прописываем конфиг
```
server {
  listen 80;
  server_name alpha;
  return 201 "ALPHA\n";

}

server {
  listen 80 default_server;
  server_name beta;
  return 202 "BETA\n";
}

server {
  listen 8080;
  server_name gamma;
  return  301 https://kuda-nibud.com;
}
```
