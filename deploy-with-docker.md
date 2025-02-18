# Пошаговое инструкция деплой проекта на сервер с помощю Docker




## **Шаг первый**

* Создать инстнас на AWS: [по ссылке](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances:instanceState=running)
* В нём же сразу генерить ssh ключ и сохранить это нам понадобится для подключения к инстансу через терминал 
* Создать базу данных Postgresql AWS: [по ссылке](https://us-east-1.console.aws.amazon.com/rds/home?region=us-east-1#databases:)
* Сохранить credentials где нибудь в надежном месте.





## **Шаг второй**

* Войти в удаленный сервер с помощью команды  
```bash
ssh -i <путь_к_ssh_ключу.pem> <instance_user>@<ip_адрес_instance>
```
* Если выдало ошибку по типу Доступ огрраничен восползуйся коммандой ниже 
```bash
chmod 600 путь_к_ssh_ключу.pem   # это изменить права ключа на только для владельца
```
* Обновим **ОС** и устновим **Docker**      
``` bash
sudo yum update -y
sudo yum install -y docker 
sudo systemctl start docker
sudo systemctl enable docker
```
* Добавить пользователя в группу **Docker** (чтобы каждый раз не использовать `sudo`)
``` bash
sudo usermod -aG docker $USER   # добавить User
newgrp docker   # применяет изменения

# проверка
docker --version
>>> Docker version 27.4.0

docker run hello-world
>>> Hello from Docker!
```

* Устанавливаем `docker-compose`
  
```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
>>> Docker Compose version v...
```
* Установим **GIT**:
```py
sudo yum install -y git

git --version
>>> git version 2.39.3
```




## **Шаг третий**



* Клонируем проект из **Github**
`git clone <ссыка на репозиторий>`
``` bash
# переходим в папку проекта
cd <путь к папке проекта>

# создадим Dockerfile
touch Dockerfile

#  Откроем его в редакторе Vim
vim Dockerfile
```
ставим код ниже 
```docker
# Используем официальный образ Python
FROM python:3.9-slim

# Устанавливаем рабочую директорию
WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

RUN pip install --upgrade pip

# Копируем зависимости
COPY requirements.txt .

# Устанавливаем зависимости
RUN pip install -r requirements.txt

# Копируем зависимости
COPY . .

# Открываем порт 8000
EXPOSE 8000

# Запуск сервера Django
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "config.wsgi:application"]
```
и выйдем сохранив изменения

* Вернемся в корневую директорию `cd ..`
* Cоздадим папку для **Nginx**
```bash
mkdir Nginx

# переходим на папку Nginx
cd Nginx/

# Cоздадим файл конфигураций Nginx
touch nginx.conf

# Откроем файл с помощью Vim
vim nginx.conf
```
ставим код ниже:
```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;

    types_hash_max_size 2048;

    upstream django {
        server django-backend:8000;  # Имя сервиса из docker-compose
    }
    listen 80;
         server_name <укажем доменное имя>;
    
    location = /favicon.ico { 
        access_log off; log_not_found off; 
        }

    location /static/ {
        alias /home/admin/assistant/static/;
    } # для быстрой доставки статика

    location /.well-known/acme-challenge/ {
        root /var/www/certbot; # понадобится для получения ssl сертификатов
        autoindex on;
        }

    location / {
        proxy_pass http://django; # указали из параметра upstream
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

}
```
выйдем из **Vim** сохранив изменения и вернемся корневую директорию `cd ..`

* Создание `docker-compose.yml` и настройка
 ```bash
# Создаем docker-compose.yml
touch docker-compose.yml

# Откроем с помощью Vim
vim docker-compose.yml
 ```
ставим код ниже
```py
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./data/certbot/www:/var/www/certbot
      - /etc/letsencrypt:/etc/letsencrypt:ro
    depends_on:
      - django
    networks:
      - backend

  django:
    build:
      context: ./project-aurora/backend
      dockerfile: Dockerfile
    container_name: django-backend
    restart: unless-stopped
    volumes:
      - ./project-aurora/backend:/app
    expose:
      - "8000"
    environment:
      - DEBUG=1
      - DB_HOST=<эндпоинт базы данных>
      - DB_PORT=5432
      - DB_NAME=<имя базы данны>
      - DB_USER=<user для базы данных>
      - DB_PASSWORD=<пароль для базы данных>
    networks:
      - backend

networks:
  backend:
    driver: bridge
```
выйдем из редактора **Vim**.

