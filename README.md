## Проект «api_YaMDb»
***
### Описание:
Проект YaMDb собирает отзывы пользователей на различные произведения.
***
### Функционал:
* Написание отзывов к произведениям;
* Комментирование отзывов;
* Просмотр информации о произведениях:
  * название,
  * год выхода,
  * рейтинг,
  * описание,
  * жанр.
* Возможности пользователя зависят от роли:
    * Аноним — может просматривать описания произведений, читать отзывы<br/>
      и комментарии;
    * Аутентифицированный пользователь (``user``) — может, как и Аноним,<br/>
      читать всё, дополнительно он может публиковать отзывы<br/>
      и ставить оценку произведениям (фильмам/книгам/песенкам),<br/>
      может комментировать чужие отзывы; может редактировать<br/>
      и удалять свои отзывы и комментарии.<br/>
      Эта роль присваивается по умолчанию каждому новому пользователю;
    * Модератор (``moderator``) — те же права, что и у Аутентифицированного<br/>
      пользователя плюс право удалять любые отзывы и комментарии;
    * Администратор (``admin``) — полные права на управление всем контентом проекта.<br/>
      Может создавать и удалять произведения, категории и жанры.<br/>
      Может назначать роли пользователям.
***
### Самостоятельная регистрация новых пользователей:
Пользователь отправляет POST-запрос с параметрами email и username на эндпоинт<br/>
`/api/v1/auth/signup/`. Сервис YaMDB отправляет письмо с кодом подтверждения<br/>
(confirmation_code) на указанный адрес email. Пользователь отправляет POST-запрос<br/>
с параметрами username и confirmation_code на эндпоинт `/api/v1/auth/token/`,<br/>
в ответе на запрос ему приходит token (JWT-токен).<br/>
В результате пользователь получает токен и может работать с API проекта,<br/>
отправляя этот токен с каждым запросом.

### Создание пользователя администратором:
Пользователя может создать администратор — через админ-зону сайта или через<br/>
POST-запрос на специальный эндпоинт `api/v1/users/`<br/>
(описание полей запроса для этого случая — в redoc).<br/>
В этот момент письмо с кодом подтверждения пользователю отправлять не нужно.
После этого пользователь должен самостоятельно отправить свой `email`<br/>
и username на эндпоинт `/api/v1/auth/signup/` , в ответ ему должно прийти письмо<br/>
с кодом подтверждения. Далее пользователь отправляет POST-запрос с параметрами<br/>
`username` и `confirmation_code` на эндпоинт `/api/v1/auth/token/`,<br/>
в ответе на запрос ему приходит token (JWT-токен), как и при самостоятельной регистрации.
***

### Использованные технологии/пакеты
* Python 3.7
* Django 2.2.28
* PyJWT 2.1.0
* django-filter 2.4.0
* djangorestframework 3.12.4
* djangorestframework-simplejwt 4.8.0
* requests 2.26.0
* gunicorn 20.0.4
* psycopg2-binary 2.8.6
* python-dotenv 0.21.0
* pytz 2020.1
* sqlparse 0.3.1

### Установка
* Сделать fork репозитория
* В Settings установить свои данные для аутентификации (установлены по дефолту)
```
DB_ENGINE - django.db.backends.postgresql
DB_NAME - db
POSTGRES_USER - postgres
POSTGRES_PASSWORD - postgres
DB_HOST - IP-адрес или доменное имя вашего сервера
DB_PORT - 5432
```
* Остановить службу nginx
```
sudo systemctl stop nginx
```
* Установить docker
```
sudo apt install docker.io 
```
* Установить docker-compose. [Официальная документация](https://docs.docker.com/compose/install/)
* Создайте на сервере директории для файлов инфраструктуры
```
cd ~
mkdir yamdb_infra
mkdir yamdb_infra/nginx
``` 
* Скопировать файлы docker-compose.yaml и nginx/default.conf
```
scp ./infra/docker-compose.yaml <ваш_username>@<ваш_сервер>:~/yamdb_infra/
scp ./infra/nginx/default.conf <ваш_username>@<ваш_сервер>:~/yamdb_infra/nginx/
```
* Внести изменение в репозиторий и сделать push (для активации деплоя)
* После успешного деплоя подключиться к серверу и перейти в директорию инфраструктуры 
```
cd ~/yamdb_infra
```
* Произвести миграции
```
docker-compose exec web python3 manage.py migrate
```
* Создать суперпользователя
```
docker-compose exec web python3 manage.py createsuperuser
```
* Собрать статику
```
docker-compose exec web python3 manage.py collectstatic --noinput
```
### После запуска контейнеров проект доступен по адресам:
* REST API http://<ваш_сервер>/api/v1/
* ReDoc http://<ваш_сервер>/redoc/
* Администрирование Django http://<ваш_сервер>/admin/

### Полезные команды
* Смена пароля для пользователя user_name
```
docker-compose exec web python3 manage.py changepassword user_name
```
* Загрузка данных из CSV-файлов (./static/data/*.csv). После загрузки потребуется снова создать суперпользователя.
```
docker-compose exec web python3 manage.py load_data
```
* Загрузка данных без вывода в терминал. После загрузки потребуется снова содазть суперпользователя.
```
docker-compose exec web python3 manage.py load_data -v 0
```
### Ресурсы API YaMDb
[Для перехода в redoc после запуска веб-сервера](http://127.0.0.1:8000/redoc/)
* Ресурс `auth`: аутентификация:
    * `v1/auth/signup/` для регистрации пользователей и получения кода подтверждения;
    * `v1/auth/token/` получение токена для доступа к ресурсу.
<br/><br/> 
* Ресурс `users`: пользователи:
    * `users/me/` доступ пользователя к информации о себе;
    * `users/{username}` доступ админа к списку пользователей<br/>
       и при указании `username` к конкретному пользователю;
<br/><br/> 
* Ресурс `title`: произведения, к которым пишут отзывы (определённый фильм, книга или песенка):
    * `titles/{titles_id}` доступ пользователя к информации о произведениях<br/>
      и при указании `titles_id` к конкретному произведению.
<br/><br/> 
* Ресурс `categories`: категории (типы) произведений («Фильмы», «Книги», «Музыка»):
    * `categories/` доступ пользователя к списку категорий.
<br/><br/> 
* Ресурс `genres`: жанры произведений. Одно произведение может быть привязано к нескольким жанрам:
    * `genres/` доступ пользователя к списку жанров.
<br/><br/> 
* Ресурс `reviews`: отзывы на произведения. Отзыв привязан к определённому произведению:
    * `titles/{title_id}/reviews/` доступ пользователя к списку отзывов к произведению `{title_id}`;
    * `titles/{title_id}/reviews/{review_id}/` доступ пользователя к конкретному отзыву;
<br/><br/> 
* Ресурс `comments`: комментарии к отзывам. Комментарий привязан к определённому отзыву:
    * `titles/{title_id}/reviews/{review_id}/comments/`<br/>
        доступ к комментариям к отзыву;
    * `titles/{title_id}/reviews/{review_id}/comments/{comment_id}/` <br/>
        доступ к конкретному коментарию.
### Выгрузка данных в БД из csv файлов:
1. Перейти из корневой папки проекта в api_yamdb/:<br/>
    `cd api_yamdb/`
2. Выполнить команду для выгрузки:<br/>
    `python manage.py load_data`
