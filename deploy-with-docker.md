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