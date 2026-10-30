# 💀 3D Skull Overlay (OBS + Twitch Integration)

Этот проект представляет собой интерактивный 3D-оверлей для OBS, написанный на HTML/Three.js. 
Оверлей отображает левитирующий 3D-череп с зелеными светящимися глазами и искрами. 
Череп реагирует на активацию награды за баллы канала на Twitch: он трясется, наклоняется, глаза вспыхивают ярче, ускоряются искры, и проигрывается звук.

## 🏗 Архитектура и принцип работы

Система состоит из трех частей:
1. **OBS Browser Source:** Открывает `skull3d.html` с GitHub Pages.
2. **Cloudflare Worker (Бэкенд):** Выступает в роли безопасного моста между Twitch и OBS. 
3. **Twitch EventSub API:** Источник событий (наград).

### Как это работает (схема):
`Twitch` -> *(WebSocket)* -> `Cloudflare Worker` -> *(WebSocket)* -> `skull3d.html` в OBS.

**Безопасность:** Токены Twitch (Access, Refresh) и Client Secret не хранятся в HTML-коде (так как репозиторий публичный). Они хранятся в защищенных переменных (Secrets) в Cloudflare.
**Автономность:** Cloudflare Worker использует Cron-триггер (таймер) и KV-хранилище для автоматического обновления токенов Twitch каждые 5 дней. Вмешательство человека не требуется.

---

## 📂 Структура проекта

*   `skull3d.html` — основной файл оверлея (Three.js, WebGL, анимации).
*   `skull_model/` — папка с 3D-моделью черепа.
    *   `scene.gltf` / `scene.bin` — 3D-модель.
    *   `textures/` — текстуры черепа.
*   `sounds/skullfire.mp3` — звуковой файл, который проигрывается при активации награды.
*   *Cloudflare Worker (находится на серверах Cloudflare, не в GitHub).*

---

## ⚙️ Настройка Cloudflare Worker (Бэкенд)

Воркер настроен на прослушивание награды **"Пощекотать череп"**.

**Переменные окружения (Variables and Secrets) в Cloudflare:**
*   `TWITCH_CLIENT_ID` (Variable) — ID приложения на Twitch.
*   `TWITCH_ACCESS_TOKEN` (Secret) — Токен доступа.
*   `TWITCH_REFRESH_TOKEN` (Secret) — Токен для обновления.
*   `TWITCH_CLIENT_SECRET` (Secret) — Секретный ключ приложения Twitch.
*   `BROADCASTER_ID` (Variable) — Цифровой ID канала Twitch.

**Привязки (Bindings):**
*   `TOKENS_KV` (KV Namespace) — Хранилище `TWITCH_TOKENS`, куда Worker сохраняет обновленные токены.

**Триггеры (Triggers):**
*   Cron Trigger: `0 0 * * *` (каждые 5 дней) для вызова функции `scheduled`, которая обновляет токены через API Twitch.

**Код Worker'а:**
Worker принимает WebSocket-подключение от OBS. Параллельно он подключается к `wss://eventsub.wss.twitch.tv/ws`. При получении `session_welcome` он отправляет POST-запрос в Twitch API для подписки на ивент `channel.channel_points_custom_reward_redemption.add`. Когда награда активируется, Worker пересылает JSON-сообщение `{"type": "reward", "title": "Пощекотать череп"}` в OBS.

---

## 💻 Настройка HTML (`skull3d.html`)

1. **Подключение к Worker'у:** 
   В нижней части скрипта функция `connectTwitch()` создает WebSocket-соединение по адресу `wss://<имя_твоего_воркера>.workers.dev`.
2. **Обработка события:**
   При получении сообщения с типом `reward` и названием `"Пощекотать череп"`, вызывается функция `triggerEyeFlash()`.
3. **Анимация (`triggerEyeFlash`):**
   *   Устанавливает `targetExcitement = 1` и таймер `donationTimer = 3.5` сек.
   *   Запускает воспроизведение `<audio id="donationSound" src="sounds/skullfire.mp3">`.
   *   В цикле `animate()` переменная `currentExcitement` плавно меняется, вызывая:
       *   Наклон черепа назад.
       *   Высокочастотную тряску (Sin/Cos с высокой частотой).
       *   Увеличение яркости света из глазниц и размера спрайтов пламени.
       *   Ускорение частиц (искр) в глазах.
   *   Через 3.5 секунды `targetExcitement` сбрасывается до 0, и анимация плавно затухает.

---

## 🛠 Как вносить изменения

**Изменить звук:**
Заменить файл `sounds/skullfire.mp3` в репозитории на новый (с тем же именем) или изменить путь в теге `<audio src="...">` в `skull3d.html`.

**Изменить название награды:**
В коде `skull3d.html` (в функции `ws.onmessage`) и в коде Cloudflare Worker (в функции `subscribeToRewards`) изменить проверку `data.title === 'Пощекотать череп'` на новое название.

**Изменить цвет глаз:**
В `skull3d.html` найти `const leftEyeLight = new THREE.PointLight(0x00ff33, ...)` и заменить `0x00ff33` на нужный HEX-цвет.

---

## ⚠️ Важные нюансы (Troubleshooting)

1. **Кеш OBS:** После любых изменений в `skull3d.html` или настройках Cloudflare, в OBS нужно нажать ПКМ по источнику -> **"Обновить страницу" (Refresh)**.
2. **Срок жизни токенов:** Если автообновление по какой-то причине упадет (например, истек срок действия Refresh Token), нужно сгенерировать новые токены на `twitchtokengenerator.com` (с правами `channel:read:redemptions`) и обновить секреты в Cloudflare.
3. **Звук в OBS:** OBS по умолчанию блокирует автовоспроизведение звука в браузерных источниках, пока не будет взаимодействия. В настройках источника `skull3d.html` в OBS нужно поставить галочку **"Управлять аудио через OBS"** (Control audio via OBS).
