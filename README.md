# Книги + озвучка (edge-tts / Silero)

## Запуск
1. `npm install`
2. `pip install -r requirements.txt`  (нужен Python 3.9+; по умолчанию голоса Microsoft через edge-tts, нужен интернет. Silero по желанию: раскомментируй torch и silero в requirements.txt)
3. `npm run start:all`  — запускает сайт и сервис озвучки вместе
   (или в двух терминалах: `npm start` и `python tts_server.py`)
4. Открыть http://localhost:3000

Если сервис озвучки не запущен, читалка сама использует системный голос браузера.

Переменные окружения: EDGE_RATE ('+0%', '-5%'…), USE_SILERO (0 — не грузить Silero), FLIBUSTA_URL, PORT, TTS_PORT (5005), SILERO_MODEL (v5_5_ru),
SILERO_SR (24000), TORCH_THREADS (4), PYTHON.

Готовые куски озвучки кэшируются в папке `cache/` — повторное прослушивание мгновенное.

## Telegram-бот и мини-приложение

Бот поднимается тем же процессом, что и сайт, но только если задан `BOT_TOKEN`. На `/start` и на любое
сообщение он отвечает, что все книги полностью бесплатны, и даёт inline-кнопку с мини-приложением.

- Локально: положи токен в `.env` (файл в `.gitignore`) и запусти `npm run start:all`.
- Кнопка-мини-приложение требует публичный HTTPS-адрес. Он берётся из `WEBAPP_URL`, а на Railway —
  автоматически из `RAILWAY_PUBLIC_DOMAIN`. Если адрес не `https`, бот отдаёт обычную ссылку.
- На Railway: добавь переменную `BOT_TOKEN` (Settings → Variables) и задеплой — меню-кнопка «Книги»
  появится у бота сама.

Важно: не запускай бота одновременно локально и на Railway — два опроса одного токена конфликтуют (`409 Conflict`).

## Деплой на Railway

Всё разворачивается одним сервисом: Node.js отдаёт сайт и API, Python-процесс озвучки (edge-tts)
запускается из него же командой `node --env-file-if-exists=.env server.js --tts`. Конфиги уже в репозитории —
`nixpacks.toml` (Node + Python) и `railway.json`.

1. Запушь проект в GitHub (или `railway up` из папки проекта) и создай проект на Railway.
2. Railway соберёт образ сам: `npm install` + `pip install -r requirements.txt`, стартовая команда
   берётся из `nixpacks.toml`.
3. Порт задавать не нужно: приложение слушает `process.env.PORT`, который Railway подставляет автоматически.
4. Открой сайт: **Settings → Networking → Generate Domain**.

Переменные окружения (**Settings → Variables**):

- `BOT_TOKEN` — включает Telegram-бота (обязательно, если бот нужен)
- `FLIBUSTA_URL` — зеркало Флибусты, если `https://flibusta.is` недоступен из дата-центра Railway
- `TTS_PORT` (по умолчанию `5005`) — внутренний порт сервиса озвучки
- `WEBAPP_URL` — нужен только если домен не Railway (на Railway берётся из `RAILWAY_PUBLIC_DOMAIN`)
- `EDGE_RATE`, `USE_SILERO`, `PYTHON` — см. раздел с переменными выше

Кэш (тексты книг, обложки, готовая озвучка) пишется в `cache/`. Чтобы он переживал деплои,
добавь том: **Settings → Volumes → mount path `/app/cache`**. Без тома всё работает, но кэш
сбрасывается при каждом деплое.

## Лицензия
Модели Silero распространяются под CC BY-NC (некоммерческое использование).
