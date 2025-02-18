# Пошаговое инструкция деплой проекта на сервер с помощю Docker


### **–– Шаг первый ––**
1. Создать инстнас на AWS: [По ссылке](https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Instances:instanceState=running)
2. В нём же сразу генерить ssh ключ и сохранить это нам понадобится для подключения к инстансу через терминал 


### **–– Шаг второй ––**
1. Создать базу данных Postgresql AWS: [По ссылке](https://us-east-1.console.aws.amazon.com/rds/home?region=us-east-1#databases:)
2. Сохранить credentials где нибудь в надежном месте.


### **–– Шаг третий ––**
1. Войти в удаленный сервер с помощью команды  `ssh -i путь_к_ssh_ключу.pem instance_user@ip_адрес_instance`
1. Если выдало ошибку по типу Доступ огрраничен восползуйся коммандой ниже 
```py
chmod 600 путь_к_ssh_ключу.pem   # это изменить права ключа на только для владельца
```
2. Выполнить команды      
``` py
sudo yum update -y
sudo yum install -y docker 
sudo systemctl start docker
sudo systemctl enable docker
```
4. Добавить пользователя в группу Docker (чтобы каждый раз не использовать `sudo`)
``` py
sudo usermod -aG docker $USER   # добавить User
newgrp docker   # применяет изменения

# проверка
docker --version
>>> Docker version 27.4.0

docker run hello-world

>>> Hello from Docker!
```
5. Устанавливаем `docker-compose`
```py
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker-compose --version
>>> Docker Compose version v...
```
6. Установим GIT:
```py
sudo yum install -y git

git --version
>>> git version 2.39.3
```
