# Kittygram

Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

В проекте Kittygram доступны следующие возможности: регистрация/авторизация пользователя, добавление нового питомца (также дополнительной информации о нем- фото, год рождения, цвет и достижения), просмотр других питомцев.

---
# Запуск проекта на сервере

Подключаемся к удаленному серверу:

    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 

Останавливаем Gunicorn и удаляем юнит gunicorn:

    sudo systemctl stop gunicorn_kittygram
    sudo rm /etc/systemd/system/gunicorn_kittygram.service 

Устанавливаем Docker Compose на сервер:

    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt install docker-compose-plugin

Далее создаем на сервере пустой файл docker-compose.production.yml и через редактор nano 
добавляем в него содержимое docker-compose.production.yml с GitHub.

Создаем на сервере файл .env с необходимыми переменными (пример .env.example)

Запускаем Docker Compose в режиме демона:
    
    sudo docker compose -f docker-compose.production.yml up -d

Проверяем, что все нужные контейнеры запущены:

    sudo docker compose -f docker-compose.production.yml ps

Выполняем миграции и собераем статические файлы бэкенда, копируем их в /backend_static/static/:

    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/

Изменяем конфиг Nginx:

    # Всё до этой строки оставляем как было.
    location / {
        proxy_pass http://127.0.0.1:9000;
    }
    # Ниже ничего менять не нужно.

Выполняем команду проверки конфигурации и перезагружаем Nginx:

    sudo nginx -t
    sudo service nginx reload

## Автоматизация деплоя: CI/CD 

Создаем в корневой папке директорию .github/workflows, а в ней — файл main.yml.

Копируем в main.yml содержимое файла kittygram_workflow.yml

Создаем переменные для workflow. В настройках репозитория — Settings, выбираем на панели слева Secrets and Variables, 
затем - New repository secret.
    
    DOCKER_USERNAME - логин Docker Hub
    DOCKER_PASSWORD - пароль Docker Hub
    SSH_KEY - закрытый SSH-ключ 
    SSH_PASSPHRASE - passphrase SSH-ключа
    USER - имя пользователя
    HOST - IP-адрес сервера
    TELEGRAM_TO - ID telegram-аккаунта
    TELEGRAM_TOKEN - токен telegram-бота

Запускаем workflow:

    git add .
    git commit -m 'Add Actions'
    git push

# Технологии:

	• Python 3.9
	• Django 3.2.3
	• gunicorn 20.1.0
	• Доменное имя: https://www.noip.com

Автор: Backend/Frontend - Яндекс Практикум; Деплой - Исхаков Камиль











    
