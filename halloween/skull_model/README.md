# 💀 3D Skull Overlay (OBS + Twitch Direct Integration)

Этот проект представляет собой интерактивный 3D-оверлей для OBS, написанный на HTML/Three.js. 
Оверлей отображает левитирующий 3D-череп с зелеными светящимися глазами и искрами. 
Череп реагирует на активацию награды за баллы канала на Twitch: он трясется, наклоняется, глаза вспыхивают ярче, ускоряются искры, и проигрывается звук.

## 🏗 Архитектура и принцип работы (Новая версия)

Система состоит из трех частей:
1. **OBS Browser Source / Браузер:** Открывает `skull3d.html` с GitHub Pages.
2. **Cloudflare Worker (Только как поставщик токенов):** Выступает в роли безопасного хранилища токенов и API для их выдачи.
3. **Twitch EventSub API:** Источник событий (наград), к которому HTML подключается **напрямую**.

### Схема работы:
1. `skull3d.html` делает GET-запрос к Cloudflare Worker по адресу `/token`.
2. Worker возвращает актуальный `Access Token`, `Client ID` и `Broadcaster ID` (токены защищены от утечки).
3. `skull3d.html` открывает **прямой WebSocket** к Twitch (`wss://eventsub.wss.twitch.tv/ws`).
4. При получении `session_welcome` от Twitch, `skull3d.html` сам отправляет POST-запрос в Twitch API для подписки на событие `channel.channel_points_custom_reward_redemption.add`.
5. Когда награда активируется, Twitch присылает событие в `skull3d.html`, и запускается анимация.

**Почему так сделано?**
Ранее WebSocket проходил через Cloudflare Worker. Из-за особенностей Cloudflare (изоляты) и лимитов Twitch на подписки, при открытии двух вкладок (OBS и Браузер) они начинали «отбирать» друг у друга подписки, из-за чего награды переставали приходить. Теперь каждый клиент подключается к Twitch независимо.

---

## ⚙️ Настройка Cloudflare Worker (Бэкенд)

Воркеру больше не нужно держать WebSocket-соединения. Он только раздает токены и обновляет их по расписанию.

**Переменные окружения (Variables and Secrets) в Cloudflare:**
*   `TWITCH_CLIENT_ID` (Variable)
*   `TWITCH_ACCESS_TOKEN` (Secret)
*   `TWITCH_REFRESH_TOKEN` (Secret)
*   `TWITCH_CLIENT_SECRET` (Secret)
*   `BROADCASTER_ID` (Variable)

**Привязки (Bindings):**
*   `TOKENS_KV` (KV Namespace) — хранилище `TWITCH_TOKENS` для сохранения обновленных токенов.

**Триггеры (Triggers):**
*   Cron Trigger: `0 0 */5 * *` (каждые 5 дней) для вызова функции `scheduled` (автообновление токенов).

**Что делает код Worker'а:**
1. Отдает CORS-заголовки для работы с GitHub Pages.
2. На эндпоинте `/token` возвращает JSON с `access_token`, `client_id` и `broadcaster_id`.
3. В функции `scheduled` обновляет токены через API Twitch и сохраняет их в KV.

---

## 💻 Настройка HTML (`skull3d.html`)

1. **Получение токена:** Функция `connectTwitch()` делает `fetch` к `https://<твой_воркер>.workers.dev/token`.
2. **Прямое подключение:** Открывает `wss://eventsub.wss.twitch.tv/ws`.
3. **Подписка:** При получении `session_welcome` отправляет POST-запрос на создание подписки на награды.
4. **Анимация (`triggerEyeFlash`):**
   *   Устанавливает `targetExcitement = 1` и таймер `donationTimer = 3.5` сек.
   *   Запускает воспроизведение `<audio id="donationSound" src="sounds/skullfire.mp3">`.
   *   В цикле `animate()` переменная `currentExcitement` плавно меняется, вызывая наклон, тряску, вспышку глаз и ускорение искр.
   *   Через 3.5 секунды `targetExcitement` сбрасывается до 0, и анимация затухает.
5. **Защита от сбоев OBS:**
   *   `delta = clock.getDelta(); if (delta > 0.1) delta = 0.1;` — не дает анимации «сломаться», если вкладка OBS засыпала и таймеры остановились.
   *   Автоматическое переподключение WebSocket при обрыве.

---

## 🛠 Как вносить изменения

**Изменить звук:**
Заменить файл `sounds/skullfire.mp3` в репозитории на новый (с тем же именем) или изменить путь в теге `<audio src="...">`.

**Изменить название награды:**
В коде `skull3d.html` (в функции `ws.onmessage`, внутри проверки `if (messageType === 'notification')`) изменить проверку `if (rewardTitle === 'Пощекотать череп')` на новое название.

**Изменить цвет глаз:**
В `skull3d.html` найти `const leftEyeLight = new THREE.PointLight(0x00ff33, ...)` и заменить `0x00ff33` на нужный HEX-цвет.

---

## ⚠️ Важные нюансы (Troubleshooting)

1. **Кеш OBS:** После любых изменений в `skull3d.html` или настройках Cloudflare, в OBS нужно нажать ПКМ по источнику -> **"Обновить страницу" (Refresh)**.
2. **Срок жизни токенов:** Если автообновление упадет, нужно сгенерировать новые токены на `twitchtokengenerator.com` (права `channel:read:redemptions`) и обновить секреты в Cloudflare.
3. **Звук в OBS:** OBS по умолчанию блокирует автовоспроизведение звука в браузерных источниках. В настройках источника `skull3d.html` в OBS нужно поставить галочку **"Управлять аудио через OBS"** (Control audio via OBS).
4. **Правила `Client Secret`:** Если ты пересоздавал приложение на Twitch (Dev Console), не забудь обновить `TWITCH_CLIENT_SECRET` в Cloudflare, иначе автообновление токенов перестанет работать.
