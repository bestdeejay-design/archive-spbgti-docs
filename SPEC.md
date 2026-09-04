# SPEC — Архив СПбГТИ (archive.ti200.ru)

## 1. Назначение проекта

Система для сбора, хранения, каталогизации и публичного показа исторических материалов Санкт-Петербургского государственного технологического института (СПбГТИ). Представляет собой публичную галерею и административный интерфейс на базе Directus (REST API + PostgreSQL + файловое хранилище).

**Домен:** https://archive.ti200.ru
**Статус:** Запущен, работает в продакшене
**Версия:** v1.0.0

---

## 2. Архитектура

### 2.1. Общая схема

```
Пользователь → DNS (archive.ti200.ru → 85.143.101.97)
                    ↓ HTTPS
    univerid.ru nginx (static-site, порт 443, SSL Let's Encrypt)
              ↓                                  ↓
    / → root /var/www/archive-gallery     /api/*, /admin/*
    (gallery/index.html, upload.html,     → proxy_pass http://directus:8055
     assets, plyr, admin-overlay)
              ↓
    ┌───────── Directus ──────────────────────┐
    │  PostgreSQL (архив, пользователи, роли)   │
    │  Файловое хранилище (/uploads)           │
    │  REST API (/items/Archive, /files,       │
    │            /users, /roles, /auth)        │
    └──────────────────────────────────────────┘
              ↓
    ┌─────────── thumbnails ──────────────────┐
    │  FFmpeg-сервис                          │
    │  Периодический опрос API → извлечение   │
    │  кадра на 1 сек → загрузка → обновление │
    │  metadata.thumbnail у файла              │
    └──────────────────────────────────────────┘
```

### 2.2. Компоненты

| Компонент | Технология | Назначение |
|-----------|-----------|------------|
| **Бэкенд** | Directus 10.12.0 | REST API, авторизация, управление данными |
| **База данных** | PostgreSQL 16 | Хранение записей, пользователей, прав |
| **Фронтенд** | HTML5 + CSS3 + JS (vanilla) | Галерея, загрузка, админ-панель |
| **Веб-сервер** | nginx (alpine) | Раздача статики, прокси на Directus |
| **Генератор миниатюр** | FFmpeg (alpine) | Извлечение кадра видео для превью |
| **Сборка** | Нет (чистый HTML/JS) | Zero dependencies |
| **Авторизация** | JWT (Directus) + статический токен | Аутентификация и доступ к API |

### 2.3. Стек технологий

| Слой | Технология | Версия |
|------|-----------|--------|
| **Сервер** | Ubuntu | 24.04.4 LTS |
| **Docker** | Docker Engine + Compose | 29.6.1 / 5.2.0 |
| **API** | Directus | 10.12.0 (последняя free-версия) |
| **БД** | PostgreSQL | 16-alpine |
| **Frontend** | Vanilla JS, CSS3 | — |
| **Медиа** | FFmpeg | 6.1.1 (в контейнере) |
| **Веб-сервер** | nginx | alpine |
| **SSL** | Let's Encrypt | Срок: до сентября 2026 |

---

## 3. Файловая структура проекта

```
archive.ti200.ru/
├── .github/workflows/deploy.yml      # CI/CD pipeline
├── .gitignore
├── docker-compose.yml                # 3 сервиса
├── docker-compose.dev.yml            # Dev overrides
├── DEPLOY.md                         # Инструкция по деплою
├── VERSION                           # v1.0.0
├── CHANGELOG.md                      # История версий
├── SPEC.md                           # Настоящий документ
│
├── gallery/                          # Публичная часть
│   ├── index.html                    # Галерея (441 строка)
│   ├── upload.html                   # Форма загрузки
│   ├── dns-check.html                # Тест DNS
│   └── admin-overlay/                # Кастомизация админки Directus
│       ├── index.html                # HTML-оболочка с CSS/JS
│       ├── setup.sh / start.sh       # Скрипты установки
│       ├── original.entry.js         # Оригинальный JS (бэкап)
│       └── patched.entry.js          # Патченый JS (скрытие элементов)
│
├── thumbnails/                       # FFmpeg-сервис
│   ├── Dockerfile                    # alpine + ffmpeg + curl + jq
│   └── watch.sh                      # Скрипт периодического опроса
│
└── docker-compose.yml                # Docker Compose конфигурация
```

---

## 4. Docker-инфраструктура

### 4.1. Сервисы

| Сервис | Образ | Имя контейнера | Память | Назначение |
|--------|-------|---------------|--------|------------|
| directus-db | postgres:16-alpine | directus-db | ~47MB | PostgreSQL |
| directus | directus/directus:10.12.0 | directus | ~177MB | API + админка |
| thumbnails | thumbnails-amd64:latest | directus-thumbs | <1MB | FFmpeg генератор превью |

### 4.2. Сети

| Сеть | Контейнеры |
|------|-----------|
| `archiveti200ru_default` | directus-db, directus, directus-thumbs |
| `univeridru_default` (external) | directus, static-site (внешний nginx) |

**Ключевое:** `directus` состоит в двух сетях — своей внутренней (для БД и thumbnails) и общей с внешним nginx (для приёма запросов с домена).

### 4.3. Переменные окружения Directus

```
DB_CLIENT: pg
DB_HOST: directus-db
CORS_ENABLED: true
CORS_ORIGIN: https://archive.ti200.ru
PUBLIC_URL: https://archive.ti200.ru
REFRESH_TOKEN_COOKIE_SAME_SITE: none
REFRESH_TOKEN_COOKIE_SECURE: true
STORAGE_LOCATIONS: local
STORAGE_LOCAL_ROOT: /var/lib/directus/uploads
TELEMETRY_DISABLED: true
MARKETPLACE_DISABLED: true
```

### 4.4. Тома Docker

| Том | Назначение |
|-----|-----------|
| `directus-db-data` | Данные PostgreSQL |
| `directus-uploads` | Загруженные файлы |
| `directus-database` | База Directus |

### 4.5. Монтирование файлов (bind mounts)

| Хост | Контейнер | Назначение |
|------|-----------|------------|
| `./gallery/admin-overlay/index.html` | `/directus/app/dist/index.html` | Замена HTML-оболочки админки |

---

## 5. Галерея (gallery/index.html)

### 5.1. Общее

- **Язык:** русский
- **Тема:** тёмная (background #0a0a0f, акценты золотой #c9a96e)
- **Шрифты:** Playfair Display (заголовки), Inter (основной)
- **Адаптивность:** да (mobile, tablet, desktop)
- **Сборка:** нет (vanilla HTML/CSS/JS)
- **Зависимости:** нет (все ресурсы self-hosted)

### 5.2. Функции

| Функция | Описание |
|---------|----------|
| **Masonry-сетка** | Адаптивная колоночная верстка (4→3→2→1 колонки) |
| **Фильтры** | По категории (7 шт) + по десятилетию |
| **Лайтбокс** | Просмотр фото/видео на весь экран |
| **Слайд-шоу** | Автоматический показ (5 сек), пропускает видео |
| **Видеоплеер** | Кастомные контролы (play/pause, прогресс, время) |
| **Миниатюры видео** | Из FFmpeg-сервиса, fallback на иконку play |
| **Lazy loading** | Изображения через loading="lazy" |
| **Счётчик** | Показано N из M материалов |
| **Клавиатура** | Escape закрыть, стрелки навигация, пробел play/pause |
| **Скелетоны** | Анимация загрузки (shimmer) |

### 5.3. API-запросы

**Публичная галерея** (без авторизации):

```
GET /items/Archive?filter[status][_eq]=published
  &fields[]=*
  &fields[]=file.id
  &fields[]=file.type
  &fields[]=file.metadata
```

**Авторизованная галерея** (статический токен `pub-archive-read-2f8a3b1c`):

```
GET /items/Archive?filter[status][_eq]=published
  &fields[]=*
  &fields[]=file.id
  &fields[]=file.type
  &fields[]=file.metadata
Headers: Authorization: Bearer pub-archive-read-2f8a3b1c
```

### 5.4. Видео

- **Определение:** через `file.type` из API (video/mp4, video/webm, etc.)
- **Превью:** `file.metadata.thumbnail` (UUID миниатюры, если сгенерирована)
- **Плеер:** кастомный HTML/JS (не Plyr):
  - Кнопка play/pause
  - Прогресс-бар (перемотка)
  - Таймер (текущее / общее время)
  - Авто-скрытие контролов (3 сек бездействия)
  - Play/pause по клику на видео и пробелу
- **В слайд-шоу:** видео пропускаются, показываются только фото

### 5.5. Параметры изображений

| Параметр | Назначение |
|----------|-----------|
| `?key=card` | Превью для фото в сетке галереи (Directus Image Transform) |
| `?key=gallery` | Полноразмерное изображение в лайтбоксе |

---

## 6. Форма загрузки (gallery/upload.html)

### 6.1. Общее

- **Доступ:** только для авторизованных пользователей (roles: Uploader, Editor, Admin)
- **Авторизация:** JWT через Directus (логин/пароль)
- **Статус новых записей:** draft (требуют публикации редактором)

### 6.2. Поля записи

| Поле | Тип | Обязательное | Источник |
|------|-----|-------------|----------|
| Название | text | да | Пользователь |
| Файл | file | да | Загрузка → Directus /files |
| Категория | select | да | Выбор из списка |
| Дата съёмки | date | нет | Пользователь |
| Место | text | нет | Пользователь |
| Загрузил | uuid | да | Автоматически (userId) |
| Статус | string | да | Всегда "draft" |

### 6.3. Процесс загрузки

1. Авторизация (JWT) → получение токена и userId
2. Выбор файлов через input или drag & drop
3. Множественная загрузка (очередь)
4. POST /files → загрузка файла в Directus
5. POST /items/Archive → создание записи с привязкой к файлу

---

## 7. Административный интерфейс (admin-overlay)

### 7.1. Описание

Стандартная админка Directus используется как есть, с минимальными модификациями:

- **Замена HTML-оболочки:** кастомный `index.html` с дополнительными CSS-правилами
- **Патч entry.js:** скрытие элементов (Marketplace, Report Bug, GitHub/Discord ссылки)

### 7.2. Скрытые элементы UI

| Элемент | Селектор CSS |
|---------|-------------|
| Marketplace | `a[href*="/settings/marketplace"]` |
| GitHub/Discord/docs | `a[href*="github.com/directus"]` и др. |
| Update notification | `[class*="update"]` |
| Settings > Users | `a[href*="/users"]` (для роли Editor) |
| Settings > Permissions | `a[href*="/permissions"]` (для роли Editor) |

### 7.3. Версия

В правом нижнем углу админки отображается `Архив СПбГТИ · v1.0.0` (через CSS body::after).

---

## 8. FFmpeg — сервис миниатюр (thumbnails/)

### 8.1. Описание

Отдельный Docker-контейнер, который периодически опрашивает Directus API на предмет новых видеофайлов. Для каждого нового видео:

1. Извлекает кадр на 1-й секунде (`ffmpeg -ss 00:00:01 -vframes 1`)
2. Загружает полученное изображение в Directus (`POST /files`)
3. Обновляет metadata исходного файла: `{"thumbnail": "<uuid>"}` (`PATCH /files/:id`)

### 8.2. Конфигурация

- **Базовый образ:** alpine:3.19
- **Пакеты:** ffmpeg, curl, jq
- **Период опроса:** 30 секунд
- **Эндпоинт:** `http://directus:8055/files?limit=50&sort=-uploaded_on`
- **Фильтр:** `select(.type | startswith("video/"))`
- **Состояние:** файл `/tmp/processed_thumbs.txt` (чтобы не обрабатывать повторно)
- **Переменные окружения:** ADMIN_EMAIL, ADMIN_PASSWORD (передаются из docker-compose)

---

## 9. Роли и права

### 9.1. Роли Directus

| Роль | admin_access | app_access | Назначение |
|------|-------------|-----------|------------|
| **Administrator** | true | true | Полный доступ, управление системой |
| **Editor** | true | true | Работа с записями, без Settings/Users |
| **Uploader** | false | false | Только загрузка через upload.html |

### 9.2. Права на коллекции

**Uploader:**
- `Archive`: create (свои), read, update (свои)
- `directus_files`: create, read

**Editor:**
- `Archive`: create, read, update, delete
- `directus_files`: create, read, update, delete
- Доступ к админ-панели (app_access: true)
- Settings/Users/Permissions скрыты через CSS

**Gallery API (статический токен):**
- `Archive`: read
- `directus_files`: read
- Токен: `pub-archive-read-2f8a3b1c`
- Пользователь: `gallery@ti200.ru`

---

## 10. API Directus

### 10.1. Используемые эндпоинты

| Метод | Endpoint | Описание | Аутентификация |
|-------|----------|----------|----------------|
| POST | `/auth/login` | Вход | Нет |
| POST | `/auth/refresh` | Обновить токен | Refresh token |
| GET | `/users/me` | Текущий пользователь | JWT |
| POST | `/files` | Загрузить файл | JWT |
| GET | `/files` | Список файлов | JWT / Static Token |
| PATCH | `/files/:id` | Обновить файл | JWT |
| DELETE | `/files/:id` | Удалить файл | JWT |
| GET | `/items/Archive` | Список записей | JWT / Static Token / Public |
| GET | `/items/Archive/:id` | Одна запись | JWT / Static Token / Public |
| POST | `/items/Archive` | Создать запись | JWT |
| PATCH | `/items/Archive/:id` | Редактировать | JWT |
| DELETE | `/items/Archive/:id` | Удалить запись | JWT |
| GET | `/roles` | Список ролей | JWT (admin) |
| POST | `/roles` | Создать роль | JWT (admin) |
| PATCH | `/roles/:id` | Редактировать роль | JWT (admin) |
| GET | `/users` | Список пользователей | JWT (admin) |
| POST | `/users` | Создать пользователя | JWT (admin) |
| PATCH | `/users/:id` | Редактировать пользователя | JWT (admin) |

### 10.2. Параметры запросов

```
# Фильтрация
GET /items/Archive?filter[category][_eq]=students
GET /items/Archive?filter[status][_eq]=published
GET /items/Archive?filter[status][_eq]=draft

# Сортировка
GET /items/Archive?sort=-date_created
GET /items/Archive?sort=date_taken

# Пагинация
GET /items/Archive?limit=25&page=1

# Поиск
GET /items/Archive?search=текст

# Расширение связанных полей
GET /items/Archive?fields[]=file.id&fields[]=file.type&fields[]=file.metadata
```

---

## 11. База данных

### 11.1. Коллекция Archive

| Поле | Тип | Обязательное | По умолчанию | Описание |
|------|-----|-------------|-------------|----------|
| id | integer (auto-increment) | да | — | Первичный ключ |
| title | string | да | — | Название материала |
| file | uuid (file) | да | — | Связь с directus_files |
| description | text | нет | null | Описание |
| category | string | да | — | Категория (enum) |
| date_taken | date | нет | null | Дата съёмки |
| decade | string | нет | null | Десятилетие (для фильтрации) |
| location | string | нет | null | Место съёмки |
| people | string | нет | null | Люди на фото |
| uploaded_by | uuid (user) | нет | null | Кто загрузил |
| status | string | да | draft | draft / published |
| date_created | timestamp | да | now() | Дата создания |
| date_updated | timestamp | нет | null | Дата изменения |

### 11.2. Категории

| Значение | Лейбл |
|----------|-------|
| event | Мероприятия |
| students | Студенты |
| teachers | Преподаватели |
| buildings | Здания |
| documents | Документы |
| other | Другое |

---

## 12. Пользователи (тестовые/рабочие)

| Роль | Email | Пароль | Статический токен |
|------|-------|--------|------------------|
| Administrator | admin@axiiom.ru | admin123 | — |
| Uploader | uploader@spbgti.ru | upload123 | — |
| Editor | editor@spbgti.ru | Editor123! | — |
| API Gallery | gallery@ti200.ru | gallery123 | pub-archive-read-2f8a3b1c |

---

## 13. CI/CD

### 13.1. GitHub Actions

**Репозиторий:** `github.com/bestdeejay-design/archive-spbgti`
**Ветка:** main
**Триггер:** push на main + workflow_dispatch

**Шаги:**
1. `actions/checkout@v4`
2. Создание `.env` на сервере (из Secrets)
3. **Build Docker-образа thumbnails** (linux/amd64, сохранить в tar.gz)
4. `rsync -avz --delete` на сервер (исключая `.git*`, `.github`, `.env`, `docker-compose.dev.yml`)
5. **Загрузка Docker-образа** на сервер (через tar.gz)
6. **Load image + docker compose up -d**

### 13.2. Secrets

| Secret | Назначение |
|--------|-----------|
| SSH_PRIVATE_KEY | Ключ для подключения к серверу (user: archive) |
| ARCHIVE_DB_PASSWORD | Пароль БД |
| ARCHIVE_KEY | Ключ Directus |
| ARCHIVE_SECRET | Секрет Directus |
| ARCHIVE_ADMIN_PASSWORD | Пароль админа Directus |

### 13.3. Исключения rsync

- `.git*`, `.github`, `.env` — безопасность
- `docker-compose.dev.yml` — dev-конфиг

---

## 14. Сервер

- **Хостинг:** Выделенный сервер
- **IP:** 85.143.101.97
- **ОС:** Ubuntu 24.04.4 LTS
- **Пользователь:** archive
- **Доступ:** SSH по ключу
- **Интернет:** Исходящие соединения заблокированы
- **Docker:** v29.6.1
- **Docker Compose:** v5.2.0
- **Образы Docker:** загружаются офлайн (через OrbStack → SCP)
- **SSL:** Let's Encrypt (обновляются вручную, срок до сентября 2026)
- **Диск:** 243G (14G занято)
- **ОЗУ:** 7.8G

### 14.1. Схема DNS

| Домен | Тип | Значение |
|-------|-----|----------|
| archive.ti200.ru | A | 85.143.101.97 |
| ti200.ru | A | 85.143.101.97 |
| www.ti200.ru | CNAME | ti200.ru |
| univerid.ru | A | 85.143.101.97 |
| www.univerid.ru | CNAME | univerid.ru |

---

## 15. Версионирование

- **VERSION:** Общая версия проекта
- **CHANGELOG.md:** История изменений
- **git tag:** Каждый релиз тегируется (v1.0.0, ...)
- **Версия в галерее:** Внизу страницы (`Архив СПбГТИ · gallery v1.0.0`)

---

## 16. Выполненные работы

- [x] Установка и настройка Directus 10.12.0 (PostgreSQL, файловое хранилище)
- [x] Docker-инфраструктура (docker-compose, сети, volumes)
- [x] CI/CD GitHub Actions (rsync + build образа)
- [x] Публичная галерея с masonry-сеткой, фильтрами, лайтбоксом
- [x] Видеоподдержка: загрузка, кастомный плеер, миниатюры FFmpeg
- [x] Слайд-шоу с пропуском видео
- [x] Форма загрузки файлов (multi-file, drag & drop, категории)
- [x] Роли: Editor, Uploader, API-reader (с правами и токенами)
- [x] Admin-overlay (скрытие лишних элементов админки)
- [x] Сервис генерации миниатюр FFmpeg (Docker-контейнер)
- [x] nginx-конфигурация (прокси, статика, SSL)
- [x] Let's Encrypt SSL-сертификаты
- [x] Версионирование (VERSION, CHANGELOG, git tags)

---

## 17. Расширенная модель данных (по требованиям 03.09.2026)

### 17.1. Новые коллекции

Для поддержки 5 осей классификации и Requirements Analysis.md добавлены:

| Коллекция | Назначение | Ключевые поля |
|-----------|-----------|---------------|
| `faculties` | Факультеты и кафедры | name, abbreviation, years_active, description |
| `persons` | Карточки персон (студенты, преподаватели, выпускники) | full_name, role, faculty_id, graduation_year, bio, photo_id |
| `events` | События и мероприятия | title, date, event_type, cover_image_id, description |
| `tags` | Тематические теги | name, group_name, color |

### 17.2. Связи many-to-many

| Связь | Таблица | Назначение |
|-------|---------|------------|
| Archive ↔ Persons | `archive_persons` | Привязка записей к персонам (author/subject/mentioned) |
| Archive ↔ Events | `archive_events` | Привязка записей к событиям |
| Archive ↔ Tags | `archive_tags` | Тематическая классификация |

### 17.3. Дополнительные поля Archive

| Поле | Тип | Назначение |
|------|-----|------------|
| `faculty_id` | UUID → faculties | Основная структурная ось |
| `primary_event_id` | UUID → events | Быстрый доступ к событию |

### 17.4. Начальные данные

- 9 исторических факультетов СПбГТИ
- 8 тематических тегов

### 17.5. Представления (Views)

- `v_archive_by_faculty` — статистика по факультетам
- `v_archive_by_person` — статистика по персонам
- `v_archive_stats` — общая статистика архива

### 17.6. Миграция

Файл: `migrations/001_add_collections.sql`
Скрипт: `migrations/run_migration.sh`
Инструкция: `migrations/README.md`

---

## 18. Планируемые доработки

### 18.1. Приоритет 1 (Расширение модели данных)
- [x] Создание коллекций: faculties, persons, events, tags
- [x] Создание связей many-to-many
- [x] Начальные данные (факультеты, теги)
- [ ] Настройка коллекций в Directus Admin
- [ ] Создание ролей: Contributor, Viewer
- [ ] Настройка прав доступа

### 18.2. Приоритет 2 (Кастомный фронтенд)
- [ ] Страница просмотра (`/browse`) — галерея с фильтрами
- [ ] Страница загрузки (`/upload`) — форма с метаданными
- [ ] Админ-панель (`/admin`) — управление персонами, событиями, тегами
- [ ] Фильтры: год, факультет, тип контента, персона, событие

### 18.3. Приоритет 3 (Интерактив для выпускников)
- [ ] Личный кабинет выпускника
- [ ] Альбомы выпусков
- [ ] Раздел «Благодарности»
- [ ] Комментарии/обсуждения

### 18.4. Приоритет 4 (Продвинутые функции)
- [ ] Полнотекстовый поиск по документам (OCR)
- [ ] Поиск по лицу (Face Recognition)
- [ ] Мобильное приложение

### 18.5. Инфраструктура
- [ ] Полный административный интерфейс (альтернатива стандартной админке Directus)
- [ ] Поиск по названию и описанию в галерее
- [ ] Статистика и дашборд
- [ ] Экспорт данных (CSV, JSON)
- [ ] Бэкапы БД и файлов
- [ ] Мониторинг (healthcheck, uptime)

---

## История изменений документа

| Дата | Версия | Изменения |
|------|--------|-----------|
| 02.07.2026 | 1.0 | Первая версия |
| 03.07.2026 | 2.0 | Добавлены: видео, FFmpeg, инфраструктура, CI/CD, роли, файловая структура, выполненные работы |
| 03.09.2026 | 3.0 | Расширенная модель данных: faculties, persons, events, tags, связи many-to-many, roadmap по требованиям |
