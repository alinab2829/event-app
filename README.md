Запустить через Docker Compose
docker compose up --build
Дождаться запуска 
docker-compose ps
# Все сервисы должны быть в статусе Up


СПИСОК endpoint'ов
Auth Service (порт 8080)
Аутентификация

Метод	Endpoint	              Описание	                        Тело запроса
POST	/api/auth/register	    Регистрация нового пользователя	   {"email": "user@mail.com", "password": "12345678"}
POST	/api/auth/login         Вход в систему	                   {"email": "user@mail.com", "password": "12345678"}
POST	/api/auth/logout	      Выход из системы	                 {"refreshToken": "token"}
POST	/api/auth/refresh	      Обновление access токена	         {"refreshToken": "token"}
GET	  /api/auth/verify-email	Подтверждение email	               ?token=xxxxx

2FA (Двухфакторная аутентификация)
Метод	 Endpoint	                Описание	                          Тело запроса
POST	/api/auth/2fa/setup	      Настройка 2FA (получение QR-кода)	  ?email=user@mail.com
POST	/api/auth/2fa/confirm	    Подтверждение и включение 2FA	      ?email=user@mail.com + {"code": "123456"}
POST	/api/auth/2fa/disable	    Отключение 2FA	                    ?email=user@mail.com
GET	  /api/auth/user/2fa-status	Проверка статуса 2FA	              ?email=user@mail.com

Администрирование
Метод	Endpoint	          Описание	                              Тело запроса
POST	/api/auth/set-admin	Назначение пользователя администратором	?email=user@mail.com

Events Service (порт 8081)
Мероприятия
Метод	  Endpoint	                          Описание	                                             Требует токен
GET	    /api/events	                        Получить список всех мероприятий	                    ✅
POST	  /api/events	                        Создать новое мероприятие (только ADMIN)	            ✅
POST	  /api/events/{eventId}/register	    Записаться на мероприятие	                            ✅
DELETE	/api/events/{eventId}/cancel	      Отменить запись на мероприятие	                      ✅
GET	    /api/events/my-registrations	      Получить мероприятия, на которые записан пользователь	✅
GET	    /api/events/{eventId}/registrations	Получить список записавшихся (только ADMIN)

Profile Service (порт 8082)
Профиль пользователя
Метод	Endpoint	                      Описание	                            Требует токен
GET	  /api/profiles/me	              Получить свой профиль	                  ✅
PUT	  /api/profiles/me	              Обновить свой профиль	                  ✅
GET	  /api/profiles/check/{userId}	  Проверить, заполнен ли профиль	        ❌ (внутренний)
GET	  /api/profiles/by-user/{userId}	Получить профиль по ID	                ❌ (внутренний)
POST	/api/profiles/internal	        Создать профиль (внутренний)	          ❌

Изображения профиля
Метод	  Endpoint	                      Описание	                        Требует токен
POST	  /api/profiles/images	          Загрузить изображение (до 5 шт.)	✅
DELETE	/api/profiles/images/{imageId}	Удалить изображение	              ✅

ЛОГИНЫ:
admin@gmail.com пароль 12345678
alinaochka1@gmail.com  12345678



Общая архитектура
Проект представляет собой микросервисную систему для управления мероприятиями, состоящую из 3 основных сервисов и фронтенда. Все компоненты контейнеризированы с помощью Docker и оркестрируются через Docker Compose.
Компоненты системы
1. Auth Service (Сервис аутентификации)
Технологии: Spring Boot, Spring Security, JWT, JJWT, TOTP

Функционал:

Регистрация пользователей с подтверждением email

Аутентификация с выдачей JWT токенов (access + refresh)

Двухфакторная аутентификация (TOTP/Google Authenticator)

Управление ролями (USER / ADMIN)

Обработка email верификации

Refresh токены для долгосрочной сессии

База данных: PostgreSQL (auth-db)

users — хранение пользователей (email, пароль, роль, статус 2FA)

refresh_token — хранение refresh токенов

verification_tokens — токены для подтверждения email

Взаимодействие с другими сервисами:

При регистрации отправляет запрос в Profile Service для создания профиля

2. Events Service (Сервис мероприятий)
Технологии: Spring Boot, Spring Data JPA, WebClient

Функционал:

CRUD операции с мероприятиями (создание — только ADMIN)

Запись пользователей на мероприятия

Отмена записи

Просмотр списка записавшихся (только ADMIN)

Проверка заполненности профиля перед записью

База данных: PostgreSQL (events-db)

events — мероприятия (название, описание, дата, место)

event_registrations — записи пользователей (user_id, event_id)

Взаимодействие с другими сервисами:

Проверяет профиль пользователя через Profile Service перед записью

3. Profile Service (Сервис профилей)
Технологии: Spring Boot, Spring Data JPA

Функционал:

Управление профилем пользователя (имя, описание)

Загрузка до 5 изображений профиля

Проверка заполненности профиля (для записи на мероприятия)

Хранение изображений в файловой системе

База данных: PostgreSQL (profile-db)

profiles — профили пользователей (user_id, email, name, about)

profile_images — изображения профиля (image_url, profile_id)

Взаимодействие с другими сервисами:

Принимает запросы от Auth Service на создание профиля

Принимает запросы от Events Service на проверку заполненности профиля

4. Frontend (Веб-интерфейс)
Технологии: HTML5, CSS3, Bootstrap 5, JavaScript (ES6), Nginx

Страницы:

index.html — авторизация (вход/регистрация/подтверждение email)

events.html — список мероприятий (с вкладками "Все мероприятия" и "Мои записи")

profile.html — профиль пользователя (редактирование имени, описания, загрузка фото, 2FA)

admin.html — админ-панель (создание мероприятий, просмотр записавшихся)

Проксирование API:
Nginx перенаправляет запросы /api/* на соответствующие сервисы:

/api/auth/* → Auth Service (8080)

/api/events/* → Events Service (8081)

/api/profiles/* → Profile Service (8082)
