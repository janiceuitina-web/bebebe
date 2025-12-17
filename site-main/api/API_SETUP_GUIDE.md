# 🚀 Инструкция по установке API для RobBob Launcher

## 📋 Содержание
1. [Установка зависимостей](#установка-зависимостей)
2. [Настройка API ключа](#настройка-api-ключа)
3. [Локальное тестирование](#локальное-тестирование)
4. [Развертывание на VPS](#развертывание-на-vps)
5. [Настройка Nginx](#настройка-nginx)
6. [Настройка админ-панели](#настройка-админ-панели)
7. [Настройка лаунчера](#настройка-лаунчера)
8. [Установка фона на сайте](#установка-фона-на-сайте)

---

## 🔧 Установка зависимостей

### Шаг 1: Перейдите в папку API
```bash
cd site-main/api
```

### Шаг 2: Установите Node.js зависимости
```bash
npm install
```

Это установит:
- `express` - веб-сервер
- `cors` - поддержка кросс-доменных запросов

---

## 🔐 Настройка API ключа

### Шаг 1: Откройте файл server.js
Найдите строку 22:
```javascript
API_KEY: process.env.API_KEY || 'robbob-admin-secret-key-change-me',
```

### Шаг 2: Замените на свой секретный ключ
```javascript
API_KEY: process.env.API_KEY || 'ваш-супер-секретный-ключ-12345678',
```

**⚠️ ВАЖНО:** Используйте сложный случайный ключ! Пример генерации:
```bash
# В Linux/Mac:
openssl rand -hex 32

# В Windows PowerShell:
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Minimum 0 -Maximum 256 }))
```

---

## 🧪 Локальное тестирование

### Запустите сервер локально:
```bash
node server.js
```

Вы должны увидеть:
```
RobBob News API running on port 3000
API Key: ваш-ключ
Data file: /path/to/data/news.json
```

### Протестируйте API:
Откройте браузер и перейдите по адресу:
```
http://localhost:3000/api/health
```

Должен вернуться ответ:
```json
{"status":"ok","timestamp":"2025-12-17T00:00:00.000Z"}
```

---

## 🌐 Развертывание на VPS

### Вариант А: Ручное развертывание

#### 1. Подключитесь к серверу
```bash
ssh user@185.185.142.190
```

#### 2. Создайте папку для API
```bash
mkdir -p /var/www/robbob/api
```

#### 3. Скопируйте файлы на сервер
С вашего локального компьютера:
```bash
# Из корня проекта
scp -r site-main/api/* user@185.185.142.190:/var/www/robbob/api/
```

#### 4. На сервере установите зависимости
```bash
cd /var/www/robbob/api
npm install
```

#### 5. Установите PM2 (процесс-менеджер)
```bash
sudo npm install -g pm2
```

#### 6. Запустите API через PM2
```bash
pm2 start server.js --name "robbob-api"
pm2 save
pm2 startup
```

PM2 обеспечит:
- ✅ Автоматический перезапуск при падении
- ✅ Запуск при загрузке сервера
- ✅ Логирование

#### 7. Проверьте статус
```bash
pm2 status
pm2 logs robbob-api
```

### Вариант Б: Использование переменных окружения

Создайте файл `.env` на сервере:
```bash
cd /var/www/robbob/api
nano .env
```

Добавьте:
```env
PORT=3000
API_KEY=ваш-супер-секретный-ключ
```

Обновите [`server.js`](server.js:16) для чтения из `.env`:
```bash
npm install dotenv
```

Добавьте в начало server.js:
```javascript
require('dotenv').config();
```

---

## 🔧 Настройка Nginx (опционально)

Если вы используете Nginx как reverse proxy:

### 1. Создайте конфиг для API
```bash
sudo nano /etc/nginx/sites-available/robbob-api
```

### 2. Добавьте конфигурацию
```nginx
server {
    listen 80;
    server_name 185.185.142.190;  # или ваш домен

    # API endpoint
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_cache_bypass $http_upgrade;
    }

    # Статические файлы сайта
    location / {
        root /var/www/robbob/site-main;
        index index.html;
        try_files $uri $uri/ =404;
    }
}
```

### 3. Активируйте конфигурацию
```bash
sudo ln -s /etc/nginx/sites-available/robbob-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 👨‍💼 Настройка админ-панели

### 1. Откройте админ-панель
```
http://185.185.142.190/admin/index.html
```
или локально:
```
site-main/admin/index.html
```

### 2. Войдите в систему
- **Логин:** `lorexdd`
- **Пароль:** `ktBPHrAFyiUcv5@`

### 3. Перейдите в раздел "Новости лаунчера"

### 4. Введите настройки API

**Если используете Nginx:**
- **API URL:** `http://185.185.142.190/api`
- **API Key:** ваш секретный ключ

**Если без Nginx (прямое подключение):**
- **API URL:** `http://185.185.142.190:3000`
- **API Key:** ваш секретный ключ

### 5. Нажмите "Сохранить настройки API"

Вы увидите сообщение: ✅ Подключение успешно! API настроен.

### 6. Создайте первую новость

Нажмите "Добавить новость" и заполните:
- **Заголовок:** Например, "Добро пожаловать!"
- **Содержание:** Ваш текст
- **Emoji:** Выберите подходящий (🚀, ⭐, 🔥 и т.д.)
- **Ссылка:** (опционально)
- **Закрепить:** Отметьте если нужно

---

## 🎮 Настройка лаунчера

### Обновите URL API в лаунчере

Откройте файл [`launcher-main/src/scripts/news.js`](../../launcher-main/src/scripts/news.js) и найдите:

```javascript
const API_URL = 'http://your-server.com/api/news';
```

Замените на:
```javascript
const API_URL = 'http://185.185.142.190:3000/api/news';
// или если используете Nginx:
const API_URL = 'http://185.185.142.190/api/news';
```

Пересоберите лаунчер:
```bash
cd launcher-main
npm run build  # или ваша команда сборки
```

---

## 🎨 Установка фона на сайте

### Вариант 1: Статическое изображение

#### 1. Поместите изображение в папку
```
site-main/img/background.jpg
```

#### 2. Откройте [`site-main/css/style.css`](../css/style.css)

#### 3. Добавьте в конец файла:
```css
/* Фон сайта */
body {
  background: url('../img/background.jpg') center/cover no-repeat fixed;
  position: relative;
}

/* Затемнение поверх фона */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: -1;
}

/* Контент поверх фона */
.content {
  position: relative;
  z-index: 1;
}
```

### Вариант 2: Видео на фоне

#### 1. Поместите видео в папку
```
site-main/img/background.mp4
site-main/img/background.webm  (для совместимости)
```

#### 2. Откройте [`site-main/index.html`](../index.html)

#### 3. Добавьте перед закрывающим `</body>`:
```html
<!-- Видео фон -->
<div class="video-background">
  <video autoplay muted loop playsinline>
    <source src="img/background.mp4" type="video/mp4">
    <source src="img/background.webm" type="video/webm">
  </video>
  <div class="video-overlay"></div>
</div>
```

#### 4. Добавьте в [`style.css`](../css/style.css):
```css
/* Видео фон */
.video-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  overflow: hidden;
}

.video-background video {
  min-width: 100%;
  min-height: 100%;
  width: auto;
  height: auto;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  object-fit: cover;
}

.video-overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.4);
}

/* Контент поверх видео */
.content {
  position: relative;
  z-index: 1;
}
```

---

## 🔍 Проверка работы

### Проверьте API:
```bash
# Health check
curl http://185.185.142.190:3000/api/health

# Получить новости
curl http://185.185.142.190:3000/api/news
```

### Проверьте админ-панель:
1. Откройте админ-панель
2. Перейдите в "Новости лаунчера"
3. Создайте тестовую новость
4. Проверьте что она появилась в списке

### Проверьте лаунчер:
1. Запустите лаунчер
2. Перейдите на страницу "Новости"
3. Убедитесь что новости загружаются

---

## ❓ Решение проблем

### API не запускается
```bash
# Проверьте логи PM2
pm2 logs robbob-api

# Проверьте порт
sudo netstat -tlnp | grep 3000

# Убейте процесс если нужно
pm2 delete robbob-api
pm2 start server.js --name "robbob-api"
```

### Ошибка "Failed to fetch"
- ✅ Проверьте что API запущен
- ✅ Проверьте firewall (порт 3000 должен быть открыт)
- ✅ Проверьте URL в админ-панели
- ✅ Проверьте API ключ

### Новости не загружаются в лаунчере
- ✅ Проверьте URL в news.js
- ✅ Проверьте что API доступен извне
- ✅ Проверьте CORS настройки

---

## 🎉 Готово!

API установлен и настроен. Теперь вы можете:
- ✅ Управлять новостями через админ-панель
- ✅ Добавлять рекомендуемые игры Roblox
- ✅ Новости автоматически появляются в лаунчере
- ✅ Красивый фон на сайте

### Полезные команды PM2:
```bash
pm2 status              # Статус всех процессов
pm2 logs robbob-api     # Логи API
pm2 restart robbob-api  # Перезапуск API
pm2 stop robbob-api     # Остановить API
pm2 delete robbob-api   # Удалить процесс
```

---

## 📞 Поддержка

При возникновении проблем проверьте:
1. Логи PM2: `pm2 logs robbob-api`
2. Логи Nginx: `sudo tail -f /var/log/nginx/error.log`
3. Права доступа к папкам
4. Настройки firewall

---

**Документация обновлена: 17.12.2025**
