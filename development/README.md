<h1 align="center">Развёртывание сиcтемы на Ubuntu 22.04</h1>

<h2 align="center">Шаг 1: Установка необходимых компонентов</h2>

Для развертывания нашей системы необходимо установить следующие компоненты: Node.js, npm, PostgreSQL, PM2, Nginx и Let's Encrypt.
```bash
# Обновляем систему
sudo apt update && sudo apt upgrade -y

# Устанавливаем Node.js и npm
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install -y nodejs

# Устанавливаем PostgreSQL
sudo apt install -y postgresql postgresql-contrib

# Устанавливаем PM2 для управления процессами Node.js
sudo npm install -g pm2

# Устанавливаем Nginx
sudo apt install -y nginx

# Устанавливаем Certbot для получения SSL-сертификатов от Let's Encrypt
sudo apt install -y certbot python3-certbot-nginx
```

<h2 align="center">Шаг 2: Настройка PostgreSQL</h2>

Для обеспечения работы нашего веб-сервиса необходимо настроить PostgreSQL. 

Изменим пароль пользователя **postgres** и создадим базу данных с именем **secure_cloud_storage** (пользователь, пароль, имя БД должны соответсвовать переменным окружения файла `.env` папки `server`):
```bash
# Входим в PostgreSQL от имени пользователя postgres
sudo -i -u postgres

# Меняем пароль пользователя postgres
psql -c "ALTER USER postgres WITH PASSWORD '1425';"

# Создаем базу данных
createdb secure_cloud_storage

# Выходим из учетной записи postgres
exit
```

<h2 align="center">Шаг 3: Клонирование репозитория с сервисом</h2>

Клонируем репозиторий с нашим сервисом с GitHub (я использую `/home`, но обычно используется `/var/www`):
```bash
# Переходим в директорию, где будем хранить проект
cd /home

# Клонируем репозиторий
git clone https://github.com/FroLaCocytus/secure-cloud-storage.git

# Переходим в директорию проекта
cd secure-cloud-storage
```

<h2 align="center">Шаг 4: Запуск серверной части</h2>

Теперь запустим серверную часть приложения с использованием pm2:
```bash
# Переходим в директорию серверной части
cd server

# Устанавливаем зависимости
npm install

# Запускаем сервер с использованием pm2
pm2 start server.js --name my-app

# Сохраняем конфигурацию pm2 для автозапуска
pm2 startup
pm2 save
```

<h2 align="center">Шаг 5: Запуск клиентской части</h2>


Сначала выполним сборку проекта React:
```bash
# Переходим в директорию клиентской части
cd ../client

# Устанавливаем зависимости
npm install

# Выполняем сборку проекта
npm run build
```

Теперь настроим nginx для обслуживания нашего React приложения и настройки SSL с помощью Let's Encrypt.
В данном примере используется домен `crypt-cloud.ru`, однако можно использовать любой другой!

1. Создаем конфигурационный файл nginx для нашего сайта:
```bash
# Переходим в директорию для конфигурационных файлов nginx
cd /etc/nginx/sites-available

# Создаем файл конфигурации
sudo nano crypt-cloud.ru
```
2. Вставляем следующий контент в файл конфигурации:
```nginx
server {
    server_name crypt-cloud.ru www.crypt-cloud.ru;

    location / {
        root /home/secure-cloud-storage/client/build; # путь до билда нашего React
        try_files $uri /index.html;
    }

    location /api {
        proxy_pass http://localhost:8000; # Express запущен на порту 8000
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 80;
}
```

3. Активируем конфигурацию nginx:
```bash
# Создаем символическую ссылку для активации конфигурации
sudo ln -s /etc/nginx/sites-available/crypt-cloud.ru /etc/nginx/sites-enabled/

# Проверяем конфигурацию nginx на наличие синтаксических ошибок
sudo nginx -t

# Перезапускаем nginx для применения изменений
sudo systemctl restart nginx

```

4. Настроим SSL с использованием Let's Encrypt и certbot:
```bash
# Запускаем certbot для автоматической настройки SSL
sudo certbot --nginx -d crypt-cloud.ru -d www.crypt-cloud.ru

# Следуем инструкциям certbot для завершения настройки
```

После настройки certbot конфигурация nginx стала выглядеть следующим образом:

```nginx
server {
    server_name crypt-cloud.ru www.crypt-cloud.ru;

    location / {
        root /home/secure-cloud-storage/client/build; # путь до билда нашего React
        try_files $uri /index.html;
    }

    location /api {
        proxy_pass http://localhost:8000; # Express запущен на порту 8000
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/crypt-cloud.ru/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/crypt-cloud.ru/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}

server {
    if ($host = crypt-cloud.ru) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    server_name crypt-cloud.ru www.crypt-cloud.ru;
    return 404; # managed by Certbot


}
```