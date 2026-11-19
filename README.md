# Deployment API Lab — практична робота №11

Express API для демонстрації налаштування середовища, автоматичного тестування, хмарного розгортання та оновлення версій.

## Встановлення та запуск

Потрібні Node.js 24, npm та Git.

```sh
npm ci
npm test
```

Створіть `.env` на основі `.env.example`:

```env
PORT=4100
NODE_ENV=production
APP_ENV=staging
APP_MESSAGE="Ваше повідомлення"
```

Локальний запуск:

```sh
npm run dev
```

Команда завантажує змінні з `.env`. Для запуску без цього файла використовується `npm start`, який отримує конфігурацію зі змінних середовища процесу.

## Конфігурація

| Параметр | Призначення |
|---|---|
| `PORT` | Порт сервера (за замовчуванням 3000) |
| `NODE_ENV` | Режим роботи бібліотек |
| `APP_ENV` | Середовище: development, staging або production |
| `APP_MESSAGE` | Повідомлення у відповіді API |
| `DEPLOY_REVISION` | Ідентифікатор коміту під час ручного розгортання |

Версія застосунку задається у `src/release.js`. Для `revision` використовується `DEPLOY_REVISION`, `RENDER_GIT_COMMIT` або `local`.

## API та перевірка

Основні маршрути:

- `GET /health` — перевірка доступності сервера.
- `GET /ready` — перевірка готовності застосунку.
- `GET /api/v1/info` — інформація про конфігурацію та версію.
- Інші маршрути повертають JSON зі статусом 404.

Запуск тестів:

```sh
npm test
```

Перевірка локального API:

```sh
node scripts/check-api.mjs http://localhost:4100 1.0.0 staging
```

Формат команди:

```sh
node scripts/check-api.mjs <BASE_URL> <VERSION> <ENVIRONMENT> [REVISION]
```

Сценарій перевіряє чотири маршрути, HTTP-статуси, структуру відповідей, версію та середовище.

## Автоматичні перевірки та розгортання

GitHub Actions автоматично запускає перевірки під час Pull Request та після передавання змін до `main`: встановлює залежності через `npm ci` і виконує `npm test`.

Застосунок розгортається на [Render](https://render.com/) як Web Service (Free) із GitHub-репозиторію.

Основні параметри:

| Параметр | Значення |
|---|---|
| Branch | `main` |
| Build Command | `npm ci` |
| Start Command | `npm start` |
| Instance Type | Free |
| Health Check Path | `/ready` |
| Auto-Deploy | After CI Checks Pass |

`NODE_ENV`, `APP_ENV` та `APP_MESSAGE` задаються в налаштуваннях сервісу Render.

Публічна адреса: `https://lab11-sophiabaranova-api.onrender.com/`

Перевірка опублікованого API:

```sh
node scripts/check-api.mjs https://lab11-sophiabaranova-api.onrender.com/ <VERSION> production
```

Безкоштовний сервіс Render може автоматично призупинятися за відсутності запитів. Після простою перший запит може потребувати додаткового часу на запуск.

## Оновлення версії

Змініть версію у `src/release.js` (наприклад, з `1.0.0` на `1.1.0`), виконайте тести та передайте зміни до GitHub:

```sh
npm test
git add src/release.js
git commit -m "Publish API version 1.1.0"
git push
```

Після успішного CI Render автоматично розгорне нову версію. Перевірте результат:

```sh
node scripts/check-api.mjs https://lab11-sophiabaranova-api.onrender.com/ 1.1.0 production
```

Успішний результат — `API checks passed`, `version: 1.1.0` та `revision`, що відповідає опублікованому коміту.

## Репозиторій

https://github.com/SophiaBaranova/node-deployment-lab11.git
