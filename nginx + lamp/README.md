# Установка LAMP + Nginx на сервер

* [Пример для якорной ссылки](https://github.com/путь/#Якорь)
* [Пример для якорной ссылки](https://github.com/путь/#Якорь)
* [Пример для якорной ссылки](https://github.com/путь/#Якорь)

## Установка LAMP

### Легкий способ

1. sudo apt-get install tasksel
2. sudo tasksel install lamp-server (При установке будет запрошен пароль для создания администратора БД MySQL.)


### Правильный способ (но долгий) 

Тут ссылка или полное описание установки LAMP на сервер, спизженное с диджиталокеана


## Подготовка Apache к получению сертификата ssl

Создадим файл index.html в директории нашего сайта
```
sudo nano /var/www/директория_сайта/index.html
``` 
И добавим в него простой код:
```
	<html>
		<head>
			<title>Тайтл</title>
		</head>
		<body>
			<h1>Название сайта</h1>
		</body>
	</html>
```
Далее выдаем права www-data директории сайта:

Для всех вложенных директорий
```
sudo chown -R www-data:www-data /var/www/*
```
```
sudo chmod 664 /var/www/* или sudo chmod 777 /var/www/*
```
Или конкретно для нашего сайта
```
sudo chown -R www-data:www-data /var/www/директория_сайта
```
```
sudo chmod 664 /var/www/директория_сайта или sudo chmod 777 /var/www/директория_сайта
```

## Конфигурация Apache

Создаем конфигурационный файл нашего сайта
```
sudo nano /etc/apache2/sites-enabled/домен.conf
```

И добавляем в него следующий код:
```
<VirtualHost *:80>
        ServerName your_site.ru
        ServerAlias www.your_site.ru *.your_site.ru
        ServerAdmin webmaster@your_site.ru
        Redirect permanent https://your_site/ # Раскомментируйте строку если Ваш сайт будет с поддержкой SSL
        <Directory /var/www/директория_сайта/>
        AllowOverride All
        </Directory>

        DocumentRoot /var/www/директория_сайта
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
## Настройка портов и установка сертификата

Выполняем следущие команды
```
sudo ufw allow OpenSSH
```
```
sudo ufw enable
```
```
sudo add-apt-repository ppa:certbot/certbot
```
```
sudo apt install python-certbot-apache
```
```
sudo ufw allow 'Apache Full'
```
```
sudo ufw delete allow 'Apache'
```
```
sudo certbot --apache -d ваш_сайт.ru -d www.ваш_сайт.ru (Далее ENTER и (1 или 2))
```
Далее ENTER и (1 или 2)

## Установка и настройка NGinx
```
sudo apt-get install nginx
```
```
sudo nano /etc/apache2/apache2.conf
```
добавить в начало файла: ServerName 127.0.1.1
```
sudo nano /etc/apache2/ports.conf
```
Приводим файл к виду: 
```
	Listen 8080

<IfModule ssl_module>
        Listen 444
</IfModule>

<IfModule mod_gnutls.c>
        Listen 444
</IfModule>
```
```
sudo nano /etc/apache2/sites-enabled/ваш_сайт.ru.conf
```
Изменить <VirtualHost *:80> на <VirtualHost *:8080>
```
sudo nano /etc/apache2/sites-enabled/ваш_сайт.ru-ssl.conf
```
```
	<IfModule mod_ssl.c>
        <VirtualHost *:444>
                ServerName ваш_сайт.ru
                ServerAdmin webmaster@ваш_сайт.ru
                ServerAlias *.ваш_сайт.ru
                DocumentRoot /var/www/ваш_сайт.ru

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                SSLEngine on
                SSLProtocol all -SSLv2

                SSLCertificateFile      /etc/letsencrypt/live/ваш_сайт.ru/cert.pem
                SSLCertificateKeyFile /etc/letsencrypt/live/ваш_сайт.ru/privkey.pem
                SSLCACertificateFile /etc/letsencrypt/live/ваш_сайт.ru/fullchain.pem
                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /var/www/ваш_сайт.ru/>
                                SSLOptions +StdEnvVars
                                Options Indexes FollowSymLinks MultiViews
                                AllowOverride All
                                Order allow,deny
                                allow from all
                </Directory>

        </VirtualHost>
```
```
sudo nano /etc/nginx/nginx.conf
```
```	
user www-data;
error_log /var/log/nginx/error.log debug;
pid /var/run/nginx.pid;
worker_rlimit_nofile 80000;

events {
worker_connections 2048;
}

http {
include /etc/nginx/mime.types;
default_type application/octet-stream;
log_format main ‘$remote_addr – $remote_user [$time_local] $status ‘
‘»$request» $body_bytes_sent «$http_referer» ‘
‘»$http_user_agent» «http_x_forwarded_for»‘;
access_log /var/log/nginx/access.log main;

# немного тюнинга
sendfile on;
tcp_nopush on;
server_tokens off;
keepalive_timeout 65;

# включим сжатие данных до отправки
gzip on;
gzip_static on;
gzip_vary on;
gzip_min_length 1100;
gzip_buffers 64 8k;
gzip_comp_level 6;
gzip_http_version 1.1;
gzip_proxied any;
gzip_types text/plain application/xml application/x-javascript text/javascript text/css text/xml application/xml+rss application/json;
gzip_disable "MSIE [1-6]\.(?!.*SV1)";

# Подключаем директорию для всех наших конфигов
include /etc/nginx/conf.d/*.conf;
}
```
```
sudo nano /etc/nginx/conf.d/ваш_сайт.ru.conf
```
``` 	
server {
        listen       *:80;
        server_name  ваш_сайт.ru www.ваш_сайт.ru;
        server_name_in_redirect off;
        access_log  /var/log/nginx/host.access.log  main;

        location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|bmp|swf|js|html|txt)$ {
        root /var/www/ваш_сайт.ru;
        index  index.php;
        }

        location / {
        proxy_pass   http://127.0.0.1:8080;
        }

        location ~ /\.ht {
        deny  all;
        }
}
```
```
sudo nano /etc/nginx/conf.d/ваш_сайт.ru-ssl.conf
```
``` 
upstream websocket {
  server ваш_сайт.ru:444;
}

server {
        listen  *:443;
        server_name ваш_сайт.ru *.ваш_сайт.ru;
        access_log  /var/log/nginx/host.access.log  main;

        location / {
        proxy_pass   https://127.0.0.1:444;
        }

        location ~* ^.+\.(jpg|jpeg|gif|png|ico|css|bmp|swf|js|html|txt)$ {
        root /var/www/ваш_сайт.ru;
        index  index.php index.html;
        }

        location ~ /\.ht {
        deny  all;
        }

        ssl on;
        ssl_certificate /etc/letsencrypt/live/ваш_сайт.ru/cert.pem;
        ssl_certificate_key /etc/letsencrypt/live/ваш_сайт.ru/privkey.pem;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
        ssl_prefer_server_ciphers on;

        # see http://nginx.com/blog/improve-seo-https-nginx/
        ssl_session_cache shared:SSL:100m;
        ssl_session_timeout 12h;
}
```
```
sudo nano /etc/nginx/conf.d/proxy.conf
```
```	
proxy_redirect off;
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
client_max_body_size 100m;
client_body_buffer_size 128k;
proxy_connect_timeout 90;
proxy_send_timeout 90;
proxy_read_timeout 90;
proxy_buffer_size 32k;
proxy_buffers 32 32k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
```
```
sudo apt install libapache2-mod-rpaf && a2enmod rpaf
```
```
sudo nano /etc/apache2/mods-enabled/rpaf.conf
```
Проверьте, чтобы параметры совпадали:
```
RPAFenable On
RPAFsethostname Off
RPAFproxy_ips 127.0.0.1
RPAFheader X-Real-IP
```
```
sudo /etc/init.d/apache2 restart && sudo /etc/init.d/nginx restart
```
## Поздравляю, вы успешно настроили связку nginx + lamp