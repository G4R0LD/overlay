# 🎭 Twitch Overlay Collection

Коллекция интерактивных и декоративных оверлеев для OBS Studio. Проект собран под Twitch-стримы: часть элементов реагирует на награды канала, часть — просто украшает сцену.

Все оверлеи — статические HTML-страницы, которые раздаются через **GitHub Pages** и подключаются в OBS как источник **«Браузер» (Browser Source)**. Прозрачный фон, независимая работа каждого компонента, никакой сборки и серверной части (кроме отдельного Cloudflare Worker для Twitch-интеграции).

**Базовый URL:** `https://g4r0ld.github.io/overlay/`

---

## 📚 Коллекции

| Папка | Тема | README |
|-------|------|--------|
| `halloween/` | Хэллоуин: череп, тыквы, свечи, атмосфера, призрак | [`halloween/README.md`](halloween/README.md) |
| `new_years/` | Новый год: колокольчик, пряня, рамка | [`new_years/README.md`](new_years/README.md) |

Каждая папка самодостаточна: свой README, свои ассеты, свои ссылки для OBS. Этот файл — точка входа для всего проекта.

---

## 🚀 Как это установить (Для стримеров)

1. Откройте OBS Studio.
2. Нажмите на плюсик в панели **«Источники»** и выберите **«Браузер»** (Browser).
3. В поле «URL» вставьте ссылку на нужный оверлей (см. список ниже).
4. Установите **Ширину: 1920** и **Высоту: 1080**.
5. Поставьте галочку **«Выключить источник, когда он не виден»**, чтобы оверлей не кушал ресурсы на скрытых сценах.
6. Для интерактивных наград включите **«Управлять аудио через OBS»**, если оверлей воспроизводит звук.

### 🧩 Все оверлеи проекта

**Хэллоуин (`halloween/`)**
*   **💀 3D Череп** — награда Twitch (тряска, свечение, звук). `…/halloween/skull3d.html`
*   **🎃 3D Тыквы** — мерцающие тыквы. `…/halloween/pumpkins.html`
*   **🕯️ 2D Свечи** — анимация пламени. `…/halloween/candles/candles.html`
*   **🦇 Атмосфера** — мыши, кровь, молнии. `…/halloween/atmosphere/`
*   **👻 3D Призрак** — скример-награда. `…/halloween/ghost.html`

**Новый год (`new_years/`)**
*   **🔔 3D Колокольчик** — награда Twitch (звон). `…/new_years/bell.html`
*   **🍪 2D Пряня** — награда Twitch (анимация + «Привет!»). `…/new_years/gingy.html`
*   **🍬 3D Рамка** — декоративная рамка. `…/new_years/candy_frame.html`

Детали, скриншоты и точные ссылки — в README соответствующей папки.

*💡 После обновления кода на GitHub — ПКМ по источнику в OBS → «Обновить страницу», чтобы сбросить кеш.*

---

<details>
<summary><b>🛠️ Техническая документация (Для разработчиков и ИИ-ассистентов)</b></summary>

## 🎯 Что это за проект

Набор оверлеев для OBS, написанных как **самостоятельные статические HTML-страницы**. Каждый оверлей — независимое приложение: свой HTML, свои ассеты, своя логика. Общего «ядра», сборщика и `package.json` нет — так задумано, чтобы любой оверлей можно было скопировать в отдельную папку и подключить без зависимостей.

**Ключевые ограничения архитектуры (не нарушать):**
- Никаких сборщиков (webpack/vite/rollup) и `npm install`.
- Никаких фреймворков (React/Vue/Svelte). Только ванильный JS + Three.js/Canvas там, где нужно.
- Всё, что загружается в браузер, — статика. Единственный серверный компонент — внешний Cloudflare Worker для Twitch.
- Обязательный прозрачный фон у каждого оверлея (`alpha: true` / `background: transparent`).

## 📂 Структура репозитория

```
overlay/
├── README.md                    ← этот файл
├── halloween/
│   ├── README.md
│   ├── skull3d.html             ← 3D-череп, Twitch-награда
│   ├── pumpkins.html            ← 3D-тыквы
│   ├── ghost.html               ← 3D-призрак (скример)
│   ├── skull_model/             ← .gltf/.bin/текстуры
│   ├── pumpkins_model/
│   ├── ghost/
│   ├── candles/candles.html     ← 2D-свечи
│   ├── atmosphere/index.html    ← 2D-атмосфера
│   └── sounds/                  ← аудио (напр. skullfire.mp3)
└── new_years/
    ├── README.md
    ├── bell.html                ← 3D-колокольчик, Twitch-награда
    ├── gingy.html               ← 2D-пряня, Twitch-награда
    └── candy_frame.html         ← 3D-рамка
```

## 🛠️ Технологический стек

| Слой | Технология |
|------|-----------|
| Хостинг | GitHub Pages (статика, HTTPS, CDN) |
| 3D-рендеринг | Three.js `0.160.0` через `importmap` с unpkg, `WebGLRenderer({alpha: true, premultipliedAlpha: false})`, `ACESFilmicToneMapping`, `GLTFLoader` |
| 2D-рендеринг | Canvas API (`globalCompositeOperation = 'lighter'` для свечения), DOM/CSS-анимации |
| Twitch-интеграция | Twitch EventSub (WebSocket) + Helix API для подписки, OAuth-токены через Cloudflare Worker |
| Аудио | HTML5 `<audio>` (`preload="auto"`), файлы в `sounds/` |

## 🔌 Архитектура Twitch-интеграции

Референсная реализация — `halloween/skull3d.html`, раздел `// ========== ПРЯМОЕ ПОДКЛЮЧЕНИЕ К TWITCH ==========`. Все интерактивные оверлеи (`bell.html`, `gingy.html`, `ghost.html`) используют **тот же паттерн**.

### Cloudflare Worker (внешний)

- **Домен:** `https://skull-overlay-worker.notingem-notingem.workers.dev`
- **Эндпоинт:** `GET /token` → `{ access_token, client_id, broadcaster_id }`
- Worker **не лежит в этом репозитории** — развёрнут отдельно. Нужен, чтобы не хранить `CLIENT_SECRET` в статике.
- **Переменные окружения (на стороне Worker):** `TWITCH_CLIENT_ID`, `TWITCH_ACCESS_TOKEN` (refresh), `BROADCASTER_ID`.
- **Хранилище:** Cloudflare KV (refresh-токены).

> ⚠️ **Не менять `WORKER_URL` в HTML без синхронного обновления во всех файлах.** Ищите по репозиторию `skull-overlay-worker.notingem-notingem.workers.dev`.

### Полный handshake (по `skull3d.html`)

```
1. fetch(WORKER_URL + '/token')            → access_token, client_id, broadcaster_id
2. new WebSocket('wss://eventsub.wss.twitch.tv/ws')
3. onmessage: message_type === 'session_welcome'
   → POST https://api.twitch.tv/helix/eventsub/subscriptions
     headers: { 'Client-ID', 'Authorization: Bearer …', 'Content-Type' }
     body: {
       type: 'channel.channel_points_custom_reward_redemption.add',
       version: '1',
       condition: { broadcaster_user_id },
       transport: { method: 'websocket', session_id }
     }
4. onmessage: message_type === 'notification'
   → data.metadata.subscription_type === 'channel.channel_points_custom_reward_redemption.add'
   → data.payload.event.reward.title  ← ФИЛЬТР ПО НАЗВАНИЮ
   → если title совпал — запускаем анимацию
5. ws.onclose → setTimeout(connectTwitch, 5000)   // автопереподключение
6. ws.onerror / catch → setTimeout(connectTwitch, 5000)
```

**Тип события:** `channel.channel_points_custom_reward_redemption.add`, version `1`.

**Фильтрация:** каждый оверлей реагирует **только на свою награду** — сравнивает `data.payload.event.reward.title` с эталонной строкой. У черепа эталон — `'Пощекотать череп'`. У нового оверлея должна быть своя строка.

**Точки отказа, которые надо сохранять при копировании:**
- `ws.onclose` **обязателен** — без него после первого разрыва оверлей «умирает» до перезагрузки источника.
- `try/catch` вокруг `fetch('/token')` — если Worker лёг, страница не должна падать.
- `sound.play().catch(...)` — браузер блокирует автозвук до первого пользовательского жеста; ошибку глушим, а не ломаем оверлей.

## 🎨 Компоненты по типам

### 3D (Three.js)
- Прозрачный фон: `new THREE.WebGLRenderer({ alpha: true, premultipliedAlpha: false })` + `setClearColor(0x000000, 0)`.
- `toneMapping = ACESFilmicToneMapping`.
- Модели грузятся через `GLTFLoader` из папок `*_model/` или `ghost/`.
- Автоцентровка и автоскейл модели: `Box3().setFromObject()` → `position.sub(center)` → `scale = N / maxDim`.
- **CORS:** `file://` не работает. Только через локальный сервер (см. ниже).

### 2D (Canvas API)
- Свечи: послойная отрисовка классом `CandleGroup`, свечение через `globalCompositeOperation = 'lighter'`.
- Атмосфера: классы `Bat`, `Blood`, `Ember`, `Spider`, поддержка URL-параметров для отключения эффектов (`?bats=0&blood=0`).

### Скример (`ghost.html`)
- Машина состояний: `IDLE → FLY_IN → SHAKE → FLY_OUT`.
- Плавность: квадратичная кривая Безье + `smoothstep` для ускорения/замедления.

## 💻 Локальная разработка

Из-за CORS браузеры блокируют загрузку `.gltf`/`.bin` по `file://`. Нужен локальный сервер:

* **VS Code:** расширение «Live Server».
* **Python:** `python -m http.server 8000`, затем `http://localhost:8000/halloween/skull3d.html`.

**Тестирование Twitch-наград без стрима:** Twitch CLI (`twitch event trigger channel.channel_points_custom_reward_redemption.add`) или тестовая награда на своём канале. Название тестовой награды должно **точно совпадать** с эталоном в коде оверлея (регистр и пробелы важны).

## 🤖 Для ИИ-ассистентов

> Этот раздел — контракт. Если вы (ИИ) читаете этот README, чтобы продолжить работу над проектом, следуйте правилам ниже. Они описывают **как проект устроен и что от вас ожидается**.

### Что можно и чего нельзя
- ✅ **Можно:** добавлять новые HTML-оверлеи в новую папку по теме (например, `easter/`), создавать для неё `README.md` по шаблону `halloween/README.md`, править существующие оверлеи.
- ✅ **Можно:** подключать Three.js/Canvas через CDN (`<script src="https://…">`), если это уже используется в соседних файлах.
- ❌ **Нельзя:** вводить сборщики, `package.json`, `node_modules`, транспиляцию, фреймворки.
- ❌ **Нельзя:** делать непрозрачный фон или `body { background: … }` без `transparent`.
- ❌ **Нельзя:** хардкодить `CLIENT_SECRET` / access-токены в HTML. Только через Worker.
- ❌ **Нельзя:** переименовывать существующие файлы без обновления ссылок в README и в OBS-инструкциях.

### Как добавить новый интерактивный оверлей (Twitch-награда)
1. Создайте папку по теме (например, `easter/`) или файл в существующей, если тема совпадает.
2. Скопируйте блок `// ========== ПРЯМОЕ ПОДКЛЮЧЕНИЕ К TWITCH ==========` **целиком** из `halloween/skull3d.html`. Не переписывайте handshake «по памяти» — там учтены `session_welcome`, переподписка, автопереподключение.
3. Замените **только** эталон названия награды:
   ```js
   if (rewardTitle === 'ВАШЕ_НАЗВАНИЕ_НАГРАДЫ') { trigger(); }
   ```
   Строка должна совпадать с названием награды в панели Twitch **посимвольно**.
4. Реализуйте `trigger()` — функцию, которая запускает вашу анимацию/звук.
5. Создайте `README.md` в папке по шаблону `halloween/README.md`: витрина для стримера + свёрнутая `<details>` с техничкой.
6. Добавьте строку в таблицу «Коллекции» этого README и в раздел «Все оверлеи проекта».
7. Проверьте локально через `python -m http.server`, затем запушьте — GitHub Pages подхватит автоматически.

### Как добавить статичный оверлей (декорация)
То же самое, но без блока Twitch — просто Three.js/Canvas с прозрачным фоном.

### Соглашения по коду
- Имена файлов — **латиница, snake_case** (`bell.html`, `candy_frame.html`). Кириллица в именах запрещена (проблемы с URL-кодированием).
- Имена переменных/классов — английский, `camelCase` / `PascalCase`.
- Комментарии — можно на русском, коротко и по делу.
- Один оверлей = один HTML-файл. Общие ассеты — в подпапке рядом.
- URL-параметры для фичей — по образцу `atmosphere/index.html` (`?bats=0`).
- Точка входа анимации — `requestAnimationFrame` + `THREE.Clock`, `delta` кэпится (`if (delta > 0.1) delta = 0.1`) — сохраняйте, иначе после сворачивания вкладки анимация «телепортируется».

### Что проверять после правок
1. Прозрачный фон (открыть в браузере, фон должен быть прозрачным).
2. Отсутствие ошибок в консоли (`F12`).
3. Для 3D — что модели грузятся (нет 404 на `.gltf`/`.bin`).
4. Для интерактивных — что награда триггерит анимацию (тестовая награда / Twitch CLI с **точным** названием).
5. Ссылки в README ведут на существующие файлы.

### Известные нюансы
- Worker один на все оверлеи: `https://skull-overlay-worker.notingem-notingem.workers.dev`. Если он недоступен — все интерактивные оверлеи молчат (но не падают, благодаря `try/catch` + retry).
- Событие подписки живёт 10 дней (лимит Twitch); после — подписка слетит. Оверлей это не замечает: соединение остаётся, но `notification` перестают приходить. При «странном поведении через N дней» — перезагрузить источник.
- Браузер блокирует автозвук, пока пользователь не взаимодействовал со страницей. В OBS звук обычно идёт, но при тестировании в браузере — кликните по вкладке.

### Границы ответственности
- **Не трогать** чужой README без запроса — они самодостаточны.
- **Не менять** URL GitHub Pages (`g4r0ld.github.io/overlay/`) — на них завязаны OBS-источники у стримера.
- Если правка ломает обратную совместимость ссылок — предупредить в ответе.

</details>
