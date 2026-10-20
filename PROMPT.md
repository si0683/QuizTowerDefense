# Промт: Quiz Tower — башенная оборона (tower defense) с образовательным квизом

> Это готовое техническое задание для нейросети. Скопируйте весь файл и отдайте его целиком.
> Документ описывает игру **в точности такой, какая она лежит в этом репозитории**. Если игра меняется, этот файл обновляется вместе с ней (см. `AGENTS.md`).
>
> Файл состоит из двух частей:
> 1. **ТЗ** — что и почему должна делать игра. Важные места и исправленные ошибки отмечены знаком ⚠️.
> 2. **Полные исходники** в конце (раздел «Исходный код») — эталонная рабочая версия.
>
> **Нейросети:** если нужна точная копия игры, создай три файла **один в один** по разделу «Исходный код» и скачай звуки по ссылкам. Если просят изменить игру, бери исходники за основу и соблюдай ТЗ.

---

## Роль и цель

Ты — опытный разработчик образовательных браузерных игр. Напиши игру «Quiz Tower» — башенная оборона (tower defense) с образовательным квизом — на **HTML, CSS и чистом JavaScript (Canvas 2D)**, без библиотек, TypeScript, React и сборки. Игру показывают детям на уроке программирования: код простой, с подробными комментариями на русском, игра красивая и работает на компьютере и телефоне.

Что в игре:
- враги идут по извилистой дороге к замку, игрок строит и улучшает башни на светлых местах у дороги;
- монеты дают за победы над врагами и за верные ответы (кнопка «❓ Вопрос»);
- после каждой волны бонусный вопрос — верный ответ даёт улучшение для всех башен (15 улучшений);
- 3 вида башен, улучшение до 3 звёзд, босс каждые 5 волн, ускорение ×2;
- 3 мира: Фэнтези, Космос, Эко-Панк; предметы: математика, русский, английский; ответ кнопками или вводом.

## Лаборатория (раздел 0 в начале `script.js`)

| Поле | По умолчанию | Что делает |
|---|---|---|
| `titleTop` | `'Quiz'` | Название в меню |
| `startCoins` | `120` | Монеты на старте |
| `baseLives` | `20` | Жизни замка |
| `questionReward` | `50` | Монеты за верный ответ |
| `towerPower` | `1` | Множитель силы башен |
| `enemySpeed` | `1` | Множитель скорости врагов |
| `waveSize` | `6` | Врагов в первой волне |
| `bossEvery` | `5` | Босс каждые N волн |
| `freeTowers` | `false` | Бесплатные башни и улучшения |
| `rainbowShots` | `false` | Радужные снаряды |

### Механика: башенная оборона
- Карта 16 × 10 клеток по 60 (960 × 600), целиком помещается в экран (поля по краям). Дорога — точки-повороты `WAYPOINTS`, места под башни — 12 клеток `PADS`. Дорога рисуется толстой линией с бегущими точками, база (🏰 / 🛰️ / 🌳) в конце.
- Состояния: `build` (между волнами) → `wave` → (`quiz`/`perk` — пауза) → `build`… → `over`.
- Башни (`TOWER_KINDS`): fast — 50 монет, урон 8, дальность 140, 2 в/с; splash — 80, 14, 125, 0.8 в/с, взрыв радиусом 60; slow — 60, 4, 115, 1.2 в/с, замедление ×0.5 на 1.5 с. Эмодзи и имена башен свои у каждого мира. Улучшение до ★★★: цена 70 и 120, урон ×1.6, дальность +15, скорострельность ×1.15 за уровень. Башня бьёт врага, который прошёл дальше всех. Снаряды летят к цели.
- Меню башни — HTML-панель рядом с местом (постройка 3 видов или улучшение), недоступные пункты серые.
- Волна N: `waveSize + 2(N−1)` врагов по одному; HP = 18·1.18^(N−1)·сложность, скорость растёт до 110. Каждая `bossEvery`-я волна — босс последним (HP ×12, отнимает 5 жизней). Дошедший враг −1 жизнь базы. Победа над врагом — монеты `5 + N` (босс 60).
- ⚠️ Кнопка «❓ Вопрос +50🪙»: доступна всегда в `build` и `wave`, ставит игру на паузу; верный ответ — монеты, неверный — кнопка «отдыхает» 3 с.
- ⚠️ Конец волны → монеты `30 + 5N`, очки, и **сразу** бонусный вопрос (не через setTimeout, иначе игрок успевает начать следующую волну и вопрос пропадает); верно → 3 улучшения из 15 (урон +25%, скорострельность, дальность, монеты +50%, +5 жизней, скидка 20%, мороз, крит, взрыв врага, +25 монет за ответ, яд, двойной выстрел, +40 монет в начале волны, урон по боссам, ремонт +10).
- Ускорение ×2 — два шага обновления за кадр.
- ⚠️ Верхняя панель — сетка `grid-template-columns: 1fr auto 1fr`: слева группа «Монеты + База», **точно по центру** «Волна», справа «Очки + пауза». Все блоки одной высоты (54, на узком экране 36). Через `space-between` центр уезжал из-за разной ширины блоков.
- ⚠️ Меню башни не заезжает на верхнюю панель и нижние кнопки (ограничение по `top`). Чипы улучшений — по центру над нижними кнопками. Всплывающие надписи переносятся (`max-width: 92%`), а не обрезаются.
- ⚠️ Экраны (меню, сложность, вопрос, улучшения, настройки, пауза, конец) — `position: fixed` на всё окно: на телефоне карта занимает узкую полосу, и окна внутри неё не помещались.
- Узкий экран: компактная панель (`@media max-width: 700px`), в меню подсказка играть горизонтально.

## Общее для всех пяти игр (Quiz Shooter, Quiz Tower, Quiz Snake, Quiz Jump, Quiz Breakout)

### Миры (`THEMES`)
| id | Название | Акцент | Враги | Босс |
|---|---|---|---|---|
| fantasy | 🏰 Фэнтези — рыцари и магия | `#FFD700` (тёмно-синий фон) | 👹 💀 | 🐉 Дракон |
| space | 🚀 Космос — лазеры и НЛО | `#00FFFF` (чёрный фон) | ☄️ 🛸 | 👾 Мега-Мозг |
| eco | 🌿 Эко-Панк — защита природы | `#39FF14` (тёмно-зелёный фон) | 🗑️ 🦗 | 🧟 Токсичный Монстр |

Герои миров: Фэнтези 🤺 🧙 🧝‍♂️, Космос 🧑‍🚀 🤖 👽, Эко-Панк 🧑‍🌾 🐼 🦉. Акцент мира пишется в CSS-переменную `--accent`.

### Квиз (раздел «Вопросы» одинаковый во всех играх)
- Предметы (`SUBJECTS`): 🧮 Математика (пул из 20 случайных примеров: сложение, вычитание без отрицательных ответов, умножение; числа растут со сложностью), 📝 Русский (28 словарных слов с пропущенной гласной: `М_ЛОКО`), 🇬🇧 Английский (26 слов для перевода: `SPACE → Космос`).
- Задания перемешиваются и идут по очереди (`questionPool`), пул пересобирается, когда кончился. 4 варианта: верный + 3 неверных (близкие числа / другие гласные / другие переводы).
- Способ ответа (`INPUT_MODES`): **Кнопки** — 4 большие кнопки (клавиши `1`–`4`); **Ввод** — поле `<input>` + Enter, ⚠️ автофокус при открытии, `inputMode = numeric` для математики. Сравнение `normalize()`: без пробелов, без регистра, Ё = Е.
- ⚠️ Пока открыт вопрос — **жёсткая пауза** (мир не двигается). Неверно — окно мигает красным, подсвечивается/пишется правильный ответ (1.1 с). Верно — зелёная вспышка (0.5 с) и награда своей игры.
- Улучшения (перки) — рогалик: 3 случайных карточки из пула, каждое можно взять один раз за игру.

### Экраны и интерфейс
- HTML-слои поверх `<canvas>`: меню (название из `LAB`, подпись «мир · предмет · способ ответа», рекорд, «Начать», «Настройки», подсказки по управлению), выбор сложности (3 карточки), вопрос, улучшения, настройки (мир, предмет, способ ответа — карточки; звук + громкость; сброс рекорда), пауза, GAME OVER (очки, рекорд, «Рестарт», «В меню»), отсчёт 3-2-1 (кроме Quiz Tower), всплывающие надписи.
- ⚠️ Верхняя панель — сетка `grid-template-columns: 1fr auto 1fr` (у змейки `1fr 1fr`, вопрос — плашкой по центру): центральный блок **точно по центру**, все блоки и кнопка паузы одной высоты и на одной линии. Через `space-between` центр уезжал (до 46 px), высоты были разные. Экраны — `position: fixed` на всё окно; всплывающие надписи переносятся (`max-width: 92%`).
- Стекло-панели `rgba(12,16,40,0.84)` + `backdrop-filter: blur`, объёмные кнопки (янтарная и бирюзовая), шрифты Russo One + Rubik через `@import` в CSS, адаптив `@media`.
- `Esc`/`P` — пауза, автопауза при скрытии вкладки, после показа экрана `blur()` активной кнопки. Пока печатают ответ — клавиши игры не работают.
- Сохранение в `localStorage` (`<игра>-settings`, `<игра>-best`) в `try/catch`.

### Звук
⚠️ Только Web Audio API (`fetch → decodeAudioData`, `createBufferSource → createGain`), `SOUND_START = { coin: 0.045 }`. Файлы в `sound/`: `jump.wav`, `coin.mp3`, `tick.wav`, `start.wav`, `crash.wav`. В CodePen (`codepen`/`cdpn.io` в адресе) звуки берутся с GitHub Pages игры.

### Файлы и CodePen
- `index.html` / `style.css` / `script.js` — строго раздельно, чистый JavaScript + Canvas 2D, графика — эмодзи и фигуры, без библиотек и сборки. Подключение с `?v=N`.
- ⚠️ `codepen/index.pen.html` — часть `index.html` между метками «Для CodePen», без `<head>`, `<link>`, `<script>`: CodePen сам подключает CSS и JS, иначе скрипт загрузится дважды (`Identifier ... has already been declared`).
- Код для детей: объекты и функции (без классов), подробные русские комментарии, нумерованные разделы, раздел 0 — «Лаборатория» `LAB` из 10 настроек (значения по умолчанию = обычная игра).

## Чек-лист

- [ ] Три отдельных файла, чистый JS без библиотек, комментарии на русском, `LAB` в начале.
- [ ] Нет ошибок в консоли; движение через `dt` (секунды, не больше 0.05).
- [ ] Вопрос ставит игру на жёсткую паузу; верный ответ даёт награду, неверный показывает правильный ответ.
- [ ] Работают оба способа ответа (кнопки и ввод с автофокусом) и все три предмета и мира.
- [ ] Настройки и рекорд сохраняются; `?v=` у CSS и JS.
- [ ] В CodePen работают `codepen/index.pen.html` + `style.css` + `script.js`, звуки грузятся с GitHub Pages.

---

## Звуковые файлы

Положи в папку `sound/` рядом с `index.html` (или скачай из репозитория https://github.com/si0683/QuizTowerDefense):

| Файл | Скачать |
|---|---|
| `jump.wav` | https://si0683.github.io/QuizTowerDefense/sound/jump.wav |
| `coin.mp3` | https://si0683.github.io/QuizTowerDefense/sound/coin.mp3 |
| `tick.wav` | https://si0683.github.io/QuizTowerDefense/sound/tick.wav |
| `start.wav` | https://si0683.github.io/QuizTowerDefense/sound/start.wav |
| `crash.wav` | https://si0683.github.io/QuizTowerDefense/sound/crash.wav |

## Исходный код

<!-- SOURCES:START -->

<!-- Этот раздел создаётся скриптом tools/build.py — не редактируйте вручную -->

### `index.html`

````html
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
  <title>Quiz Tower</title>
  <!-- Подключаем файл с оформлением.
       ?v=2 — номер версии: меняем его при каждом обновлении игры,
       чтобы браузер не брал старый файл из кэша -->
  <link rel="stylesheet" href="style.css?v=2">
</head>
<body>

<!-- ↓↓↓ Для CodePen копируйте в окно HTML всё, что ниже этой строки ↓↓↓
     Только эту часть! CodePen сам подключает style.css и script.js.
     Если скопировать строки <link> или <script src>, игра загрузится дважды
     и появится ошибка «Identifier ... has already been declared». -->

<!-- «Сцена» — рамка игры. Внутри холст (canvas) и все экраны поверх него -->
<div class="stage" id="stage">

  <!-- Холст: здесь JavaScript рисует карту, башни и врагов -->
  <canvas id="gameCanvas"></canvas>

  <!-- Панель во время игры -->
  <div class="hud hidden" id="hud">
    <div class="hud-top">
      <!-- Слева: монеты и жизни базы -->
      <div class="hud-group">
        <div class="hud-box">
          <span class="hud-label">🪙 Монеты</span>
          <b id="coins">0</b>
        </div>
        <div class="hud-box">
          <span class="hud-label" id="baseLabel">🏰 Замок</span>
          <b id="lives">20</b>
        </div>
      </div>
      <!-- По центру: номер волны -->
      <div class="hud-box hud-center">
        <span class="hud-label">Волна</span>
        <b id="wave">1</b>
      </div>
      <!-- Справа: очки и пауза -->
      <div class="hud-group hud-group-right">
        <div class="hud-box">
          <span class="hud-label">Очки</span>
          <b id="score">0</b>
        </div>
        <button class="icon-btn" id="pauseButton" type="button" aria-label="Пауза">
          <svg viewBox="0 0 24 24" width="22" height="22"><rect x="6" y="5" width="4" height="14" rx="1.5" fill="currentColor"/><rect x="14" y="5" width="4" height="14" rx="1.5" fill="currentColor"/></svg>
        </button>
      </div>
    </div>
    <!-- Улучшения, которые уже взяли -->
    <div class="weapon perk-chips" id="perkChips"></div>
    <!-- Кнопки действий -->
    <div class="td-bar">
      <button class="td-btn" id="quizButton" type="button">❓ Вопрос +50🪙</button>
      <button class="td-btn td-btn-main" id="waveButton" type="button">▶ Волна 1</button>
      <button class="td-btn" id="speedButton" type="button">⏩ ×1</button>
    </div>
  </div>

  <!-- Меню постройки / улучшения башни (создаёт JavaScript) -->
  <div class="build-menu hidden" id="buildMenu"></div>

  <!-- Всплывающая надпись -->
  <div class="toast" id="toast"></div>

  <!-- ЭКРАН 1. Стартовое меню -->
  <section class="screen" id="menuScreen">
    <div class="card">
      <h1 class="title">
        <span class="title-top">Quiz</span>
        <span class="title-bottom">Tower</span>
      </h1>
      <p class="subtitle" id="menuInfo">Отвечай на вопросы, строй башни, защищай замок!</p>
      <div class="best">🏆 Рекорд: <b id="menuBest">0</b></div>
      <div class="buttons">
        <button class="btn" id="startButton" type="button">Начать</button>
        <button class="btn btn-secondary" id="settingsButton" type="button">Настройки</button>
      </div>
      <div class="howto">
        <div>❓ Верный ответ — монеты на башни</div>
        <div>👆 Нажми на светлое место у дороги — построить башню</div>
        <div>👆 Нажми на башню — улучшить её</div>
        <div><kbd>Q</kbd> — вопрос, <kbd>Пробел</kbd> — волна, <kbd>Esc</kbd> — пауза</div>
        <div>📱 На телефоне удобнее играть горизонтально</div>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 2. Выбор сложности -->
  <section class="screen hidden" id="difficultyScreen">
    <div class="card card-wide">
      <h2 class="screen-title">Выбери сложность</h2>
      <div class="choice-grid" id="difficultyChoices"></div>
    </div>
  </section>

  <!-- ЭКРАН 3. Вопрос (квиз) -->
  <section class="screen hidden" id="quizScreen">
    <div class="card quiz-card" id="quizCard">
      <div class="quiz-subject" id="quizSubject">🧮 Математика</div>
      <div class="quiz-question" id="quizQuestion">8 × 7 = ?</div>
      <div class="quiz-options" id="quizOptions"></div>
      <form class="quiz-form hidden" id="quizForm" autocomplete="off">
        <input class="quiz-input" id="quizInput" type="text" placeholder="Ответ" autocomplete="off" autocapitalize="characters" spellcheck="false">
        <button class="btn btn-small" type="submit">Ответить ⏎</button>
      </form>
      <div class="quiz-hint" id="quizHint">Верный ответ — монеты!</div>
    </div>
  </section>

  <!-- ЭКРАН 4. Улучшения после волны -->
  <section class="screen hidden" id="perkScreen">
    <div class="card card-wide">
      <h2 class="screen-title">Верно! Выбери улучшение</h2>
      <div class="choice-grid" id="perkChoices"></div>
    </div>
  </section>

  <!-- ЭКРАН 5. Настройки -->
  <section class="screen screen-settings hidden" id="settingsScreen">
    <div class="settings-panel">
      <div class="settings-header">
        <h2>Настройки</h2>
        <button class="icon-btn" id="settingsClose" type="button" aria-label="Закрыть">✕</button>
      </div>

      <div class="setting">
        <h3>Мир</h3>
        <div class="options options-cards" id="themeOptions"></div>
      </div>

      <div class="setting">
        <h3>Предмет</h3>
        <div class="options options-cards" id="subjectOptions"></div>
      </div>

      <div class="setting">
        <h3>Как отвечать</h3>
        <div class="options options-cards" id="inputOptions"></div>
      </div>

      <div class="setting">
        <h3>Звук</h3>
        <div class="row">
          <label class="switch">
            <input type="checkbox" id="soundToggle">
            <span class="switch-slider"></span>
          </label>
          <input class="volume" type="range" id="volumeRange" min="0" max="100" step="5" aria-label="Громкость">
        </div>
      </div>

      <div class="settings-footer">
        <button class="btn btn-ghost btn-small" id="resetBestButton" type="button">Сбросить рекорд</button>
        <button class="btn btn-small" id="settingsDone" type="button">Готово</button>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 6. Пауза -->
  <section class="screen hidden" id="pauseScreen">
    <div class="card">
      <h2 class="screen-title">Пауза</h2>
      <div class="buttons">
        <button class="btn" id="resumeButton" type="button">Продолжить</button>
        <button class="btn btn-secondary" id="pauseMenuButton" type="button">В меню</button>
      </div>
    </div>
  </section>

  <!-- ЭКРАН 7. Игра окончена -->
  <section class="screen hidden" id="gameOverScreen">
    <div class="card">
      <h2 class="over-title">GAME OVER</h2>
      <div class="new-record hidden" id="newRecord">Новый рекорд!</div>
      <div class="stats">
        <div class="stat"><span>Очки</span><b id="finalScore">0</b></div>
        <div class="stat"><span>Волны</span><b id="finalWaves">0</b></div>
        <div class="stat"><span>Ответы</span><b id="finalAnswers">0</b></div>
      </div>
      <div class="best">🏆 Рекорд: <b id="finalBest">0</b></div>
      <div class="buttons">
        <button class="btn" id="restartButton" type="button">Рестарт</button>
        <button class="btn btn-secondary" id="overMenuButton" type="button">В меню</button>
      </div>
    </div>
  </section>

</div>

<!-- ↑↑↑ Для CodePen копируйте до этой строки (строку <script> ниже — НЕ копировать) ↑↑↑ -->

<!-- Подключаем файл с логикой игры (номер версии — как у style.css) -->
<script src="script.js?v=2"></script>
</body>
</html>
````

### `style.css`

````css
/* =========================================================
   QUIZ TOWER DEFENSE — оформление (цвета, шрифты, экраны, кнопки)
   ========================================================= */

/* Шрифты Google Fonts: Russo One — заголовки и цифры, Rubik — текст */
@import url('https://fonts.googleapis.com/css2?family=Russo+One&family=Rubik:wght@400;500;700&display=swap');

/* ---------- Цвета игры (переменные) ---------- */
/* --accent JavaScript меняет под выбранный мир */
:root {
  --bg: #070b1f;
  --panel: rgba(12, 16, 40, 0.84);
  --line: rgba(255, 255, 255, 0.14);
  --text: #eef1ff;
  --dim: #a7b0dd;
  --accent: #00ffff;
  --amber: #ffb84d;
  --amber-light: #ffd27a;
  --amber-dark: #d9821c;
  --cyan: #4fd6e8;
  --cyan-dark: #1f8a99;
  --good: #39d97a;
  --bad: #ff4d6d;
  --outline: #0d1030;
}

* {
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  height: 100%;
}

body {
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  background:
    radial-gradient(900px 600px at 20% -10%, #1c2560 0%, transparent 60%),
    radial-gradient(800px 600px at 100% 100%, #2a1450 0%, transparent 55%),
    var(--bg);
  color: var(--text);
  font-family: 'Rubik', sans-serif;
  user-select: none;
  -webkit-user-select: none;
  -webkit-tap-highlight-color: transparent;
}

/* ---------- Сцена и холст ---------- */
/* Размер сцены задаёт JavaScript (функция resizeGame) */
.stage {
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 80px rgba(0, 0, 0, 0.6);
  touch-action: none; /* чтобы касание не прокручивало страницу */
}

#gameCanvas {
  display: block;
  width: 100%;
  height: 100%;
}

.hidden {
  display: none !important;
}

/* ---------- Панель во время игры (HUD) ---------- */
.hud {
  position: absolute;
  inset: 0;
  padding: 12px;
  pointer-events: none; /* касания проходят сквозь панель */
}

.hud-top {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 8px;
}

.hud-box {
  display: flex;
  flex-direction: column;
  padding: 6px 12px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--panel);
  backdrop-filter: blur(8px);
}

.hud-label {
  color: var(--dim);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  white-space: nowrap;
}

.hud-box b {
  font-family: 'Russo One', sans-serif;
  font-size: 22px;
  font-weight: 400;
  line-height: 1.15;
}

.hud-hp {
  min-width: 140px;
}

.hud-hp b {
  font-size: 14px;
}

.hud-center {
  align-items: center;
}

.hud-center b {
  color: var(--accent);
  text-shadow: 0 0 12px var(--accent);
}

.hud-right {
  display: flex;
  gap: 8px;
  align-items: flex-start;
}

.bar {
  height: 10px;
  margin: 5px 0 3px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.12);
  overflow: hidden;
}

.bar-fill {
  width: 100%;
  height: 100%;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--bad), #ff9f43, var(--good));
  background-size: 300% 100%;
  transition: width 0.25s;
}

.weapon {
  position: absolute;
  left: 12px;
  bottom: 12px;
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  max-width: calc(100% - 24px);
}

.chip {
  padding: 4px 10px;
  border: 1px solid var(--line);
  border-radius: 999px;
  background: var(--panel);
  font-size: 12px;
  font-weight: 500;
  white-space: nowrap;
}

.boss-bar {
  position: absolute;
  top: 78px;
  left: 50%;
  width: min(420px, 80%);
  transform: translateX(-50%);
  text-align: center;
  font-family: 'Russo One', sans-serif;
  font-size: 14px;
  color: var(--bad);
  text-shadow: 0 0 10px var(--bad);
}

.boss-bar .bar {
  height: 12px;
  border: 1px solid rgba(255, 77, 109, 0.6);
}

.boss-bar .bar-fill {
  background: linear-gradient(90deg, #ff2a5c, #ff7a45);
}

.icon-btn {
  pointer-events: auto;
  display: grid;
  place-items: center;
  width: 44px;
  height: 44px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--panel);
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  backdrop-filter: blur(8px);
}

/* ---------- Экраны ---------- */
.screen {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 14px;
  background: radial-gradient(circle at 50% 45%, rgba(7, 11, 31, 0.3), rgba(7, 11, 31, 0.75));
  animation: fadeIn 0.2s ease;
}

.card {
  width: min(400px, 100%);
  max-height: 100%;
  overflow-y: auto;
  padding: 24px 22px;
  border: 1px solid var(--line);
  border-radius: 26px;
  background: var(--panel);
  box-shadow: 0 24px 60px -20px rgba(0, 0, 0, 0.7), 0 0 0 1px rgba(255, 255, 255, 0.02);
  backdrop-filter: blur(10px);
  text-align: center;
  animation: rise 0.3s cubic-bezier(0.2, 0.9, 0.3, 1.2);
}

.card-wide {
  width: min(720px, 100%);
}

/* Заголовок игры с неоновым свечением в цвет мира */
.title {
  margin: 0 0 6px;
  font-family: 'Russo One', sans-serif;
  font-weight: 400;
  line-height: 0.95;
}

.title span {
  display: block;
  -webkit-text-stroke: 5px var(--outline);
  paint-order: stroke fill;
}

.title-top {
  font-size: 52px;
  color: #fff;
  text-shadow: 0 0 16px var(--accent), 0 6px 0 var(--outline);
}

.title-bottom {
  font-size: 58px;
  color: var(--accent);
  text-shadow: 0 0 24px var(--accent), 0 6px 0 var(--outline);
}

.subtitle {
  margin: 12px 0 14px;
  color: var(--dim);
  font-size: 15px;
}

.best {
  display: inline-block;
  margin-bottom: 18px;
  padding: 6px 14px;
  border-radius: 999px;
  background: rgba(255, 184, 77, 0.12);
  color: var(--amber);
  font-weight: 500;
}

.screen-title {
  margin: 0 0 16px;
  font-family: 'Russo One', sans-serif;
  font-size: 28px;
  font-weight: 400;
}

.buttons {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.howto {
  display: grid;
  gap: 8px;
  margin-top: 20px;
  color: var(--dim);
  font-size: 13px;
}

kbd {
  padding: 2px 7px;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.1);
  border-bottom: 2px solid rgba(255, 255, 255, 0.22);
  color: var(--text);
  font-family: inherit;
  font-size: 12px;
}

/* ---------- Кнопки (объёмные, «нажимаются» вниз) ---------- */
.btn {
  padding: 14px 28px;
  border: 0;
  border-radius: 16px;
  background: linear-gradient(180deg, var(--amber-light), var(--amber));
  box-shadow: 0 6px 0 var(--amber-dark), 0 12px 24px -6px rgba(0, 0, 0, 0.5);
  color: #3b2300;
  font-family: 'Russo One', sans-serif;
  font-size: 20px;
  cursor: pointer;
  transition: transform 0.1s, box-shadow 0.1s, filter 0.15s;
}

.btn:hover {
  filter: brightness(1.07);
}

.btn:active {
  transform: translateY(4px);
  box-shadow: 0 2px 0 var(--amber-dark);
}

.btn-secondary {
  background: linear-gradient(180deg, #7ae9f6, var(--cyan));
  box-shadow: 0 6px 0 var(--cyan-dark), 0 12px 24px -6px rgba(0, 0, 0, 0.5);
  color: #05303a;
}

.btn-secondary:active {
  box-shadow: 0 2px 0 var(--cyan-dark);
}

.btn-ghost {
  background: rgba(255, 255, 255, 0.08);
  box-shadow: none;
  color: var(--dim);
}

.btn-ghost:active {
  box-shadow: none;
}

.btn-small {
  padding: 11px 18px;
  font-size: 15px;
}

/* ---------- Карточки выбора: герои, сложность, улучшения ---------- */
.choice-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(170px, 1fr));
  gap: 12px;
}

.choice {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 16px 12px;
  border: 2px solid var(--line);
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.05);
  color: var(--text);
  font-family: inherit;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s, box-shadow 0.15s;
}

.choice:hover,
.choice:focus-visible {
  transform: translateY(-4px);
  border-color: var(--accent);
  box-shadow: 0 0 22px -4px var(--accent);
  outline: none;
}

.choice-icon {
  font-size: 46px;
  line-height: 1.1;
}

.choice-title {
  font-family: 'Russo One', sans-serif;
  font-size: 18px;
}

.choice-desc {
  color: var(--dim);
  font-size: 13px;
  line-height: 1.35;
}

.choice-stats {
  display: grid;
  gap: 3px;
  width: 100%;
  margin-top: 4px;
  font-size: 12px;
  text-align: left;
}

.stat-row {
  display: flex;
  align-items: center;
  gap: 6px;
}

.stat-row span {
  width: 64px;
  color: var(--dim);
}

.stat-dots {
  letter-spacing: 1px;
  color: var(--accent);
}

/* ---------- Окно вопроса ---------- */
.quiz-card {
  width: min(460px, 100%);
  border-color: var(--accent);
  box-shadow: 0 0 40px -10px var(--accent), 0 24px 60px -20px rgba(0, 0, 0, 0.7);
}

.quiz-subject {
  color: var(--dim);
  font-size: 14px;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.quiz-question {
  margin: 14px 0 18px;
  font-family: 'Russo One', sans-serif;
  font-size: 40px;
  letter-spacing: 0.04em;
  word-break: break-word;
}

.quiz-question small {
  display: block;
  margin-top: 6px;
  color: var(--dim);
  font-family: 'Rubik', sans-serif;
  font-size: 15px;
  letter-spacing: 0;
}

.quiz-options {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.quiz-option {
  min-height: 64px;
  padding: 12px;
  border: 2px solid var(--line);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.07);
  color: #fff;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  cursor: pointer;
  transition: transform 0.12s, border-color 0.12s, background 0.12s;
}

.quiz-option:hover {
  transform: translateY(-3px);
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.12);
}

.quiz-option.right {
  border-color: var(--good);
  background: rgba(57, 217, 122, 0.3);
}

.quiz-option.wrong {
  border-color: var(--bad);
  background: rgba(255, 77, 109, 0.3);
}

.quiz-form {
  display: flex;
  gap: 10px;
}

.quiz-input {
  flex: 1;
  min-width: 0;
  padding: 14px 16px;
  border: 2px solid var(--accent);
  border-radius: 16px;
  outline: none;
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  text-align: center;
  text-transform: uppercase;
}

.quiz-hint {
  margin-top: 14px;
  color: var(--dim);
  font-size: 13px;
}

/* Окно мигает красным при неверном ответе и зелёным при верном */
.quiz-card.flash-bad {
  animation: flashBad 0.6s ease;
}

.quiz-card.flash-good {
  animation: flashGood 0.5s ease;
}

/* ---------- Экран настроек ---------- */
.screen-settings {
  justify-content: flex-end;
  align-items: stretch;
  background: linear-gradient(90deg, transparent 30%, rgba(7, 11, 31, 0.6));
}

.settings-panel {
  width: min(420px, 100%);
  max-height: 100%;
  overflow-x: hidden;
  overflow-y: auto;
  padding: 18px 20px 20px;
  border: 1px solid var(--line);
  border-radius: 24px;
  background: var(--panel);
  backdrop-filter: blur(10px);
  animation: slideIn 0.3s ease;
}

.settings-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.settings-header h2 {
  margin: 0;
  font-family: 'Russo One', sans-serif;
  font-size: 24px;
  font-weight: 400;
}

.setting {
  padding: 12px 0;
  border-bottom: 1px solid var(--line);
}

.setting h3 {
  margin: 0 0 10px;
  color: var(--dim);
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.06em;
  text-transform: uppercase;
}

.row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
}

.options {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.options-cards .option {
  flex: 1;
  min-width: 100px;
}

.option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 10px 8px;
  border: 2px solid transparent;
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.05);
  color: var(--dim);
  font-family: inherit;
  font-size: 12px;
  cursor: pointer;
  transition: transform 0.15s, border-color 0.15s, background 0.15s;
}

.option:hover {
  transform: translateY(-2px);
}

.option.selected {
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.option-icon {
  font-size: 26px;
}

.option-title {
  color: #fff;
  font-size: 14px;
  font-weight: 700;
}

.settings-footer {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  padding-top: 16px;
}

/* Переключатель (вкл/выкл) */
.switch {
  position: relative;
  flex-shrink: 0;
  width: 52px;
  height: 30px;
  cursor: pointer;
}

.switch input {
  position: absolute;
  opacity: 0;
}

.switch-slider {
  position: absolute;
  inset: 0;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.15);
  transition: background 0.2s;
}

.switch-slider::before {
  content: '';
  position: absolute;
  top: 4px;
  left: 4px;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #fff;
  transition: transform 0.2s;
}

.switch input:checked + .switch-slider {
  background: var(--amber);
}

.switch input:checked + .switch-slider::before {
  transform: translateX(22px);
}

.volume {
  flex: 1;
  accent-color: var(--amber);
}

.volume:disabled {
  opacity: 0.35;
}

/* ---------- Экран «Игра окончена» ---------- */
.over-title {
  margin: 0 0 14px;
  font-family: 'Russo One', sans-serif;
  font-size: 46px;
  font-weight: 400;
  color: var(--bad);
  -webkit-text-stroke: 4px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 24px var(--bad);
}

.stats {
  display: flex;
  gap: 10px;
  margin-bottom: 14px;
}

.stat {
  flex: 1;
  padding: 10px 6px;
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.07);
}

.stat span {
  display: block;
  color: var(--dim);
  font-size: 12px;
}

.stat b {
  font-family: 'Russo One', sans-serif;
  font-size: 28px;
  font-weight: 400;
}

.new-record {
  display: inline-block;
  margin-bottom: 14px;
  padding: 6px 16px;
  border-radius: 999px;
  background: linear-gradient(90deg, var(--accent), var(--amber));
  color: #10132e;
  font-weight: 700;
  animation: pulse 1s ease-in-out infinite;
}

/* ---------- Обратный отсчёт и всплывающая надпись ---------- */
.countdown {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  pointer-events: none;
  font-family: 'Russo One', sans-serif;
  font-size: 150px;
  color: #fff;
  -webkit-text-stroke: 6px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 30px var(--accent), 0 8px 0 var(--outline);
}

.countdown span {
  animation: countPop 0.7s ease-out both;
}

.countdown .go {
  font-size: 90px;
  color: var(--accent);
}

.toast {
  position: absolute;
  top: 26%;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0;
  pointer-events: none;
  white-space: nowrap;
  font-family: 'Russo One', sans-serif;
  font-size: 30px;
  color: #fff;
  -webkit-text-stroke: 4px var(--outline);
  paint-order: stroke fill;
  text-shadow: 0 0 18px var(--accent);
}

.toast.show {
  animation: toast 1.6s ease both;
}

/* ---------- Анимации ---------- */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes rise {
  from { opacity: 0; transform: translateY(24px) scale(0.96); }
  to { opacity: 1; transform: none; }
}

@keyframes slideIn {
  from { opacity: 0; transform: translateX(40px); }
  to { opacity: 1; transform: none; }
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.07); }
}

@keyframes flashBad {
  0%, 100% { background: var(--panel); }
  20%, 60% { background: rgba(255, 77, 109, 0.75); }
  40%, 80% { background: rgba(255, 77, 109, 0.25); }
}

@keyframes flashGood {
  0%, 100% { background: var(--panel); }
  40% { background: rgba(57, 217, 122, 0.6); }
}

@keyframes countPop {
  0% { opacity: 0; transform: scale(2.2); }
  30% { opacity: 1; transform: scale(1); }
  80% { opacity: 1; transform: scale(0.95); }
  100% { opacity: 0; transform: scale(0.7); }
}

@keyframes toast {
  0% { opacity: 0; transform: translate(-50%, 20px) scale(0.8); }
  15% { opacity: 1; transform: translate(-50%, 0) scale(1.05); }
  25% { transform: translate(-50%, 0) scale(1); }
  80% { opacity: 1; }
  100% { opacity: 0; transform: translate(-50%, -20px); }
}

/* ---------- Телефоны ---------- */
@media (max-width: 640px) {
  .screen-settings {
    align-items: flex-end;
    padding: 10px;
  }

  .settings-panel {
    max-height: 80%;
  }

  .hud-hp {
    min-width: 110px;
  }

  .hud-box b {
    font-size: 18px;
  }

  .title-top { font-size: 42px; }
  .title-bottom { font-size: 46px; }
  .quiz-question { font-size: 32px; }
  .quiz-option { font-size: 20px; }
  .choice-grid { grid-template-columns: 1fr; }
  .choice { flex-direction: row; text-align: left; }
  .choice-icon { font-size: 38px; }
  .countdown { font-size: 120px; }
}

/* Если в системе включено «уменьшить движение» */
@media (prefers-reduced-motion: reduce) {
  .new-record {
    animation: none;
  }
}

/* =========================================================
   ДОПОЛНЕНИЯ ДЛЯ TOWER DEFENSE
   ========================================================= */

/* Нижняя панель с кнопками действий */
.td-bar {
  position: absolute;
  left: 50%;
  bottom: 12px;
  display: flex;
  gap: 8px;
  transform: translateX(-50%);
  pointer-events: auto;
}

.td-btn {
  padding: 10px 16px;
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--panel);
  color: #fff;
  font-family: 'Russo One', sans-serif;
  font-size: 15px;
  white-space: nowrap;
  cursor: pointer;
  backdrop-filter: blur(8px);
  transition: transform 0.1s, border-color 0.15s;
}

.td-btn:hover {
  border-color: var(--accent);
}

.td-btn:active {
  transform: translateY(2px);
}

.td-btn:disabled {
  opacity: 0.4;
  cursor: default;
}

.td-btn-main {
  border-color: var(--accent);
  box-shadow: 0 0 16px -4px var(--accent);
}

/* Меню постройки и улучшения башни — появляется рядом с местом */
.build-menu {
  position: absolute;
  z-index: 5;
  display: flex;
  flex-direction: column;
  gap: 6px;
  width: 210px;
  padding: 10px;
  border: 1px solid var(--accent);
  border-radius: 16px;
  background: var(--panel);
  box-shadow: 0 0 24px -6px var(--accent);
  backdrop-filter: blur(10px);
  animation: fadeIn 0.15s ease;
}

.build-title {
  margin-bottom: 2px;
  color: var(--dim);
  font-size: 12px;
  font-weight: 700;
  text-align: center;
  text-transform: uppercase;
}

.build-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 7px 9px;
  border: 1px solid var(--line);
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.06);
  color: #fff;
  font-family: inherit;
  font-size: 13px;
  text-align: left;
  cursor: pointer;
}

.build-item:hover:not(:disabled) {
  border-color: var(--accent);
}

.build-item:disabled {
  opacity: 0.4;
  cursor: default;
}

.build-icon {
  font-size: 24px;
}

.build-name {
  flex: 1;
  line-height: 1.2;
}

.build-name small {
  display: block;
  color: var(--dim);
  font-size: 11px;
}

.build-cost {
  color: var(--amber);
  font-weight: 700;
}

@media (max-width: 640px) {
  .td-btn {
    padding: 9px 11px;
    font-size: 13px;
  }
}

/* Узкий экран: панель компактнее, чтобы было видно карту */
@media (max-width: 700px) {
  .hud {
    padding: 6px;
  }

  .hud-top {
    gap: 4px;
  }

  .hud-box {
    padding: 3px 7px;
    border-radius: 10px;
  }

  .hud-label {
    font-size: 8px;
  }

  .hud-box b {
    font-size: 14px;
  }

  .icon-btn {
    width: 34px;
    height: 34px;
  }

  .td-bar {
    bottom: 6px;
    gap: 4px;
  }

  .td-btn {
    padding: 6px 8px;
    font-size: 11px;
  }

  .build-menu {
    width: 170px;
    padding: 6px;
  }

  .build-item {
    padding: 4px 6px;
    font-size: 11px;
  }

  .build-icon {
    font-size: 18px;
  }

  .toast {
    font-size: 18px;
  }
}

/* ---------- Ровная верхняя панель ----------
   Три колонки: слева и справа одинаковой ширины (1fr),
   поэтому блок «Волна» стоит точно по центру */
.hud-top {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  align-items: start;
}

.hud-group {
  display: flex;
  gap: 8px;
  align-items: stretch;
}

.hud-group-right {
  justify-content: flex-end;
}

/* Все блоки одной высоты, текст по центру */
.hud-top .hud-box {
  justify-content: center;
  min-width: 86px;
  height: 54px;
  text-align: center;
}

.hud-top .hud-label {
  line-height: 16px;
}

.hud-top .icon-btn {
  width: 54px;
  height: 54px;
}

/* Улучшения — над нижними кнопками, по центру */
.perk-chips {
  left: 50%;
  bottom: 66px;
  justify-content: center;
  transform: translateX(-50%);
}

@media (max-width: 700px) {
  .hud-top .hud-box {
    min-width: 0;
    height: 36px;
    padding: 2px 7px;
  }

  .hud-top .hud-label {
    line-height: 11px;
  }

  .hud-top .icon-btn {
    width: 36px;
    height: 36px;
  }

  .hud-group {
    gap: 4px;
  }

  .perk-chips {
    bottom: 42px;
  }
}

/* Длинные подсказки переносятся, а не обрезаются краем экрана */
.toast {
  width: max-content;
  max-width: 92%;
  white-space: normal;
  text-align: center;
  line-height: 1.15;
}

/* Экраны (меню, вопрос, улучшения, настройки…) — на всё окно, а не только
   поверх карты: на телефоне карта узкой полосой, и окна в ней не помещались */
.screen {
  position: fixed;
}
````

### `script.js`

````javascript
/* =========================================================
   QUIZ TOWER — оборона замка + образовательный квиз
   Чистый JavaScript и Canvas, графика из эмодзи
   ---------------------------------------------------------
   Как устроен файл (читайте сверху вниз):
     0. Лаборатория: 10 настроек для экспериментов
     1. Данные: миры, башни, предметы, сложность
     2. Вопросы: математика, русский, английский
     3. Улучшения после волны
     4. Настройки игрока и рекорд
     5. Звуки
     6. Холст и карта
     7. Состояние игры, враги, башни, снаряды
     8. Рисование: карта, башни, враги, частицы
     9. Окно вопроса (квиз)
    10. Экран улучшений
    11. Игровые действия: башни, волны, урон
    12. Обновление мира за один кадр
    13. Рисование одного кадра
    14. Экраны и интерфейс
    15. Экран настроек
    16. Управление
    17. Игровой цикл и запуск
   ========================================================= */


/* ---------------------------------------------------------
   0. ЛАБОРАТОРИЯ ИГРЫ 🧪
   Поменяй одно значение, и игра изменится!
   Сейчас здесь обычные значения: игра такая, как задумано.
   --------------------------------------------------------- */

const LAB = {
  // 1. Название игры в меню: верхнее и нижнее слово (до 8 букв)
  titleTop: 'Quiz',
  titleBottom: 'Tower',

  // 2. Монеты на старте: 120 — обычно, 1000 — богатый старт
  startCoins: 120,

  // 3. Жизни замка: 20 — обычно, 50 — крепость
  baseLives: 20,

  // 4. Монеты за верный ответ: 50 — обычно, 200 — щедро
  questionReward: 50,

  // 5. Сила башен: 1 — обычная, 2 — в два раза сильнее
  towerPower: 1,

  // 6. Скорость врагов: 1 — обычная, 0.5 — медленные, 1.5 — быстрые
  enemySpeed: 1,

  // 7. Врагов в первой волне: 6 — обычно, 15 — толпа
  waveSize: 6,

  // 8. Босс приходит каждые N волн: 5 — обычно, 2 — часто
  bossEvery: 5,

  // 9. Бесплатные башни: false — за монеты, true — даром
  freeTowers: false,

  // 10. Радужные снаряды: false — цвет мира, true — радуга
  rainbowShots: false
};


/* ---------------------------------------------------------
   1. ДАННЫЕ: МИРЫ, БАШНИ, ПРЕДМЕТЫ, СЛОЖНОСТЬ
   --------------------------------------------------------- */

// Три вида башен:
//   fast   — быстро стреляет по одному врагу
//   splash — медленно, но бьёт по всем рядом с целью
//   slow   — слабый урон, зато замедляет врагов
// cost — цена, damage — урон, range — дальность (пикс),
// rate — выстрелов в секунду
const TOWER_KINDS = {
  fast:   { cost: 50, damage: 8,  range: 140, rate: 2.0, desc: 'Быстрые выстрелы' },
  splash: { cost: 80, damage: 14, range: 125, rate: 0.8, splash: 60, desc: 'Урон по площади' },
  slow:   { cost: 60, damage: 4,  range: 115, rate: 1.2, slow: 0.5, desc: 'Замедляет врагов' }
};

// Цена улучшения башни до 2 и до 3 уровня
const UPGRADE_COST = [0, 70, 120];

const THEMES = [
  {
    id: 'fantasy', icon: '🏰', label: 'Фэнтези', desc: 'Рыцари и магия',
    accent: '#FFD700',
    ground: '#17213d', groundDots: 'rgba(120,140,255,0.08)', path: '#4b4566', pathEdge: '#2e2a45',
    base: { emoji: '🏰', name: 'Замок' },
    towers: { fast: { emoji: '🏹', name: 'Лучник' }, splash: { emoji: '🔮', name: 'Маг' }, slow: { emoji: '❄️', name: 'Ледяная' } },
    enemies: ['👹', '💀'],
    boss: { emoji: '🐉', name: 'Дракон' },
    props: ['🌲', '🪨', '🌲', '🍄']
  },
  {
    id: 'space', icon: '🚀', label: 'Космос', desc: 'Лазеры и НЛО',
    accent: '#00FFFF',
    ground: '#070a1c', groundDots: 'rgba(0,255,255,0.06)', path: '#16224a', pathEdge: '#00b7c7',
    base: { emoji: '🛰️', name: 'Станция' },
    towers: { fast: { emoji: '🔫', name: 'Лазер' }, splash: { emoji: '🚀', name: 'Ракеты' }, slow: { emoji: '🧊', name: 'Крио' } },
    enemies: ['☄️', '🛸'],
    boss: { emoji: '👾', name: 'Мега-Мозг' },
    props: ['🪐', '✨', '🌑', '✨']
  },
  {
    id: 'eco', icon: '🌿', label: 'Эко-Панк', desc: 'Защита природы',
    accent: '#39FF14',
    ground: '#0c2516', groundDots: 'rgba(57,255,20,0.06)', path: '#4a3a26', pathEdge: '#2c2216',
    base: { emoji: '🌳', name: 'Древо' },
    towers: { fast: { emoji: '🌵', name: 'Кактус' }, splash: { emoji: '🍄', name: 'Гриб' }, slow: { emoji: '💧', name: 'Родник' } },
    enemies: ['🗑️', '🦗'],
    boss: { emoji: '🧟', name: 'Токсичный Монстр' },
    props: ['🌿', '🌼', '🌲', '🪵']
  }
];

const SUBJECTS = [
  { id: 'math', icon: '🧮', label: 'Математика', desc: 'Примеры' },
  { id: 'rus',  icon: '📝', label: 'Русский', desc: 'Пропущенная буква' },
  { id: 'eng',  icon: '🇬🇧', label: 'Английский', desc: 'Перевод слов' }
];

const INPUT_MODES = [
  { id: 'buttons',  icon: '👆', label: 'Кнопки', desc: '4 варианта' },
  { id: 'keyboard', icon: '⌨️', label: 'Ввод', desc: 'Напечатать ответ' }
];

// Сложность: здоровье и скорость врагов, множитель очков
const DIFFICULTIES = [
  { id: 'easy',   icon: '🙂', label: 'Легко',     desc: 'Слабые враги',            enemyHp: 0.8, speed: 0.9, score: 1 },
  { id: 'normal', icon: '😎', label: 'Нормально', desc: 'Честная битва, очки ×1.5', enemyHp: 1.2, speed: 1.05, score: 1.5 },
  { id: 'hard',   icon: '😈', label: 'Сложно',    desc: 'Толстые враги, очки ×2',   enemyHp: 1.7, speed: 1.2, score: 2 }
];

function findById(list, id) {
  return list.find(item => item.id === id) || list[0];
}

function currentTheme() {
  return findById(THEMES, settings.theme);
}

function difficulty() {
  return findById(DIFFICULTIES, settings.difficulty);
}


/* ---------------------------------------------------------
   2. ВОПРОСЫ
   Для каждого предмета — пул заданий. Задания перемешиваются
   и идут по очереди, поэтому подряд не повторяются.
   --------------------------------------------------------- */

// Русский язык: слово, номер пропущенной буквы (с нуля)
const RUS_WORDS = [
  ['МОЛОКО', 1], ['СОБАКА', 1], ['КОРОВА', 1], ['ВОРОНА', 1], ['БЕРЁЗА', 1],
  ['ЗАЯЦ', 1], ['ПЕТУХ', 1], ['ЛИСИЦА', 1], ['МЕДВЕДЬ', 1], ['КАРАНДАШ', 1],
  ['ТЕТРАДЬ', 1], ['ПЕНАЛ', 1], ['ЯГОДА', 3], ['МАГАЗИН', 1], ['ГОРОД', 1],
  ['УЧЕНИК', 2], ['ДЕЖУРНЫЙ', 1], ['РЕБЯТА', 1], ['КАПУСТА', 1], ['ПОГОДА', 1],
  ['СОРОКА', 1], ['ВЕТЕР', 1], ['МОРКОВЬ', 1], ['ПАЛЬТО', 1], ['ОГУРЕЦ', 2],
  ['ХОРОШО', 1], ['ДЕВОЧКА', 1], ['ЯЗЫК', 1]
];
const VOWELS = ['А', 'О', 'Е', 'И', 'Я', 'У'];

// Английский: слово и перевод
const ENG_WORDS = [
  ['APPLE', 'Яблоко'], ['DOG', 'Собака'], ['CAT', 'Кошка'], ['SPACE', 'Космос'],
  ['STAR', 'Звезда'], ['SUN', 'Солнце'], ['MOON', 'Луна'], ['HOUSE', 'Дом'],
  ['BOOK', 'Книга'], ['WATER', 'Вода'], ['TREE', 'Дерево'], ['BIRD', 'Птица'],
  ['FISH', 'Рыба'], ['SCHOOL', 'Школа'], ['FRIEND', 'Друг'], ['MOTHER', 'Мама'],
  ['FATHER', 'Папа'], ['GREEN', 'Зелёный'], ['RED', 'Красный'], ['BIG', 'Большой'],
  ['MILK', 'Молоко'], ['BREAD', 'Хлеб'], ['SKY', 'Небо'], ['EARTH', 'Земля'],
  ['FLOWER', 'Цветок'], ['CAR', 'Машина']
];

let questionPool = [];

function shuffle(list) {
  const copy = list.slice();
  for (let i = copy.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [copy[i], copy[j]] = [copy[j], copy[i]];
  }
  return copy;
}

function randInt(min, max) {
  return Math.floor(min + Math.random() * (max - min + 1));
}

// Один пример по математике. Чем сложнее игра — тем больше числа
function makeMathQuestion(level) {
  const kind = randInt(0, 2);
  let a, b, text, answer;
  if (kind === 0) {
    a = randInt(2, 10 + level * 20);
    b = randInt(2, 10 + level * 20);
    text = a + ' + ' + b + ' = ?';
    answer = a + b;
  } else if (kind === 1) {
    a = randInt(5, 15 + level * 20);
    b = randInt(1, a);           // без отрицательных ответов
    text = a + ' − ' + b + ' = ?';
    answer = a - b;
  } else {
    a = randInt(2, 5 + level * 2);
    b = randInt(2, 9);
    text = a + ' × ' + b + ' = ?';
    answer = a * b;
  }
  // Неверные варианты — близкие числа
  const wrong = new Set();
  while (wrong.size < 3) {
    const w = answer + randInt(-10, 10);
    if (w !== answer && w >= 0) wrong.add(w);
  }
  return { text: text, hint: '', answer: String(answer), options: shuffle([answer, ...wrong].map(String)), numeric: true };
}

function makeRusQuestion([word, index]) {
  const letter = word[index] === 'Ё' ? 'Е' : word[index];
  const text = word.slice(0, index) + '_' + word.slice(index + 1);
  const wrong = shuffle(VOWELS.filter(v => v !== letter)).slice(0, 3);
  return { text: text, hint: 'Какая буква пропущена?', answer: letter, options: shuffle([letter, ...wrong]) };
}

function makeEngQuestion([word, translation]) {
  const wrong = shuffle(ENG_WORDS.filter(w => w[1] !== translation)).slice(0, 3).map(w => w[1]);
  return { text: word, hint: 'Переведи на русский', answer: translation, options: shuffle([translation, ...wrong]) };
}

// Пул из 20+ заданий для выбранного предмета
function buildQuestionPool() {
  const subject = settings.subject;
  if (subject === 'rus') {
    questionPool = shuffle(RUS_WORDS).map(makeRusQuestion);
  } else if (subject === 'eng') {
    questionPool = shuffle(ENG_WORDS).map(makeEngQuestion);
  } else {
    const level = DIFFICULTIES.indexOf(findById(DIFFICULTIES, settings.difficulty));
    questionPool = [];
    for (let i = 0; i < 20; i++) questionPool.push(makeMathQuestion(level));
  }
}

function nextQuestion() {
  if (questionPool.length === 0) buildQuestionPool();
  return questionPool.pop();
}

// Сравнение ответов: без пробелов, без разницы в регистре, Ё = Е
function normalize(text) {
  return String(text).trim().toUpperCase().replace(/Ё/g, 'Е');
}


/* ---------------------------------------------------------
   3. УЛУЧШЕНИЯ ПОСЛЕ ВОЛНЫ
   Верный ответ на бонусный вопрос → 3 случайных улучшения.
   Каждое можно взять один раз за игру
   --------------------------------------------------------- */

const PERKS = [
  { id: 'dmg',    icon: '💥', title: 'Урон +25%',         desc: 'Все башни бьют сильнее',         apply: b => { b.damage *= 1.25; } },
  { id: 'rate',   icon: '⚡', title: 'Скорострельность',  desc: 'Башни стреляют на 20% чаще',     apply: b => { b.rate *= 1.2; } },
  { id: 'range',  icon: '🔭', title: 'Дальность +15%',    desc: 'Башни видят дальше',             apply: b => { b.range *= 1.15; } },
  { id: 'coins',  icon: '🪙', title: 'Богатая добыча',    desc: 'Монеты за врагов +50%',          apply: b => { b.coins *= 1.5; } },
  { id: 'lives',  icon: '❤️', title: 'Крепкие стены',     desc: '+5 жизней базы',                 apply: b => { game.lives += 5; } },
  { id: 'sale',   icon: '🏷️', title: 'Скидка 20%',        desc: 'Башни и улучшения дешевле',      apply: b => { b.cost *= 0.8; } },
  { id: 'frost',  icon: '❄️', title: 'Морозные снаряды',  desc: 'Все башни немного замедляют',    apply: b => { b.frost = true; } },
  { id: 'crit',   icon: '🍀', title: 'Крит 20%',          desc: 'Шанс двойного урона',            apply: b => { b.crit += 0.2; } },
  { id: 'blast',  icon: '💣', title: 'Взрыв врага',       desc: 'Побеждённый враг ранит соседей', apply: b => { b.blast = true; } },
  { id: 'nerd',   icon: '🎓', title: 'Отличник',          desc: '+25 монет за каждый ответ',      apply: b => { b.questionBonus += 25; } },
  { id: 'poison', icon: '☠️', title: 'Яд',                desc: 'Снаряды отравляют врагов',       apply: b => { b.poison = true; } },
  { id: 'double', icon: '🔁', title: 'Двойной выстрел',   desc: '15% шанс выстрелить дважды',     apply: b => { b.double += 0.15; } },
  { id: 'magnet', icon: '🧲', title: 'Магнит',            desc: '+40 монет в начале волны',       apply: b => { b.waveIncome += 40; } },
  { id: 'boss',   icon: '👑', title: 'Охотник на боссов', desc: 'Урон по боссам +50%',            apply: b => { b.bossDamage *= 1.5; } },
  { id: 'repair', icon: '🛠️', title: 'Ремонт',            desc: '+10 жизней базы',                apply: b => { game.lives += 10; } }
];


/* ---------------------------------------------------------
   4. НАСТРОЙКИ ИГРОКА И РЕКОРД
   --------------------------------------------------------- */

const DEFAULT_SETTINGS = {
  theme: 'fantasy',
  subject: 'math',
  inputMode: 'buttons',
  difficulty: 'easy',
  sound: true,
  volume: 0.7
};

let settings = loadSettings();
let bestScore = loadBestScore();

function loadSettings() {
  try {
    const saved = JSON.parse(localStorage.getItem('quiztower-settings'));
    return Object.assign({}, DEFAULT_SETTINGS, saved);
  } catch (error) {
    return Object.assign({}, DEFAULT_SETTINGS);
  }
}

function saveSettings() {
  try {
    localStorage.setItem('quiztower-settings', JSON.stringify(settings));
  } catch (error) {
    // Если браузер запретил сохранение — просто играем без него
  }
}

function loadBestScore() {
  try {
    return Number(localStorage.getItem('quiztower-best')) || 0;
  } catch (error) {
    return 0;
  }
}

function saveBestScore() {
  try {
    localStorage.setItem('quiztower-best', String(bestScore));
  } catch (error) {
    // ничего страшного
  }
}


/* ---------------------------------------------------------
   5. ЗВУКИ (Web Audio — звук играет мгновенно)
   В CodePen своей папки нет — звуки берутся с GitHub Pages
   --------------------------------------------------------- */

const isCodePen = location.hostname.includes('codepen') || location.hostname.includes('cdpn.io');
const SOUND_PATH = isCodePen ? 'https://si0683.github.io/QuizTowerDefense/sound/' : 'sound/';

const SOUND_FILES = {
  jump:  'jump.wav',   // башня построена
  coin:  'coin.mp3',   // верный ответ, монеты
  tick:  'tick.wav',   // начало волны
  start: 'start.wav',  // улучшение, босс
  crash: 'crash.wav'   // враг дошёл до базы, неверный ответ
};

const SOUND_START = { coin: 0.045 };

const audioContext = new (window.AudioContext || window.webkitAudioContext)();
const sounds = {};

for (const name in SOUND_FILES) {
  fetch(SOUND_PATH + SOUND_FILES[name])
    .then(response => response.arrayBuffer())
    .then(data => audioContext.decodeAudioData(data))
    .then(buffer => { sounds[name] = buffer; })
    .catch(() => {});
}

function playSound(name) {
  if (!settings.sound || !sounds[name]) return;
  if (audioContext.state === 'suspended') audioContext.resume();
  const source = audioContext.createBufferSource();
  source.buffer = sounds[name];
  const volume = audioContext.createGain();
  volume.gain.value = settings.volume;
  source.connect(volume);
  volume.connect(audioContext.destination);
  source.start(0, SOUND_START[name] || 0);
}


/* ---------------------------------------------------------
   6. ХОЛСТ И КАРТА
   Карта — сетка 16 × 10 клеток по 60 «игровых пикселей».
   Дорога задаётся точками-поворотами, места для башен — списком.
   --------------------------------------------------------- */

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const stage = document.getElementById('stage');

const CELL = 60;
const COLS = 16;
const ROWS = 10;
const W = COLS * CELL;  // 960
const H = ROWS * CELL;  // 600
let scale = 1;

// Повороты дороги (столбец, строка). Первая точка — за краем карты
const WAYPOINTS = [[-1, 3], [3, 3], [3, 7], [7, 7], [7, 2], [11, 2], [11, 7], [14, 7], [14, 4], [15, 4]];

// Места, где можно строить башни
const PADS = [[2, 2], [4, 4], [2, 6], [5, 6], [6, 4], [8, 3], [9, 1], [10, 4], [12, 4], [13, 6], [12, 8], [8, 8]];

// Центр клетки в пикселях
function cellCenter(col, row) {
  return { x: col * CELL + CELL / 2, y: row * CELL + CELL / 2 };
}

const PATH_POINTS = WAYPOINTS.map(([c, r]) => cellCenter(c, r));

// Все клетки дороги (чтобы рисовать её и не ставить на неё декор)
const PATH_CELLS = new Set();
for (let i = 0; i < WAYPOINTS.length - 1; i++) {
  let [c, r] = WAYPOINTS[i];
  const [c2, r2] = WAYPOINTS[i + 1];
  while (c !== c2 || r !== r2) {
    PATH_CELLS.add(c + ',' + r);
    c += Math.sign(c2 - c);
    r += Math.sign(r2 - r);
  }
  PATH_CELLS.add(c2 + ',' + r2);
}

// Карта целиком помещается в экран, по краям могут быть поля
function resizeGame() {
  scale = Math.min(window.innerWidth / W, window.innerHeight / H);
  stage.style.width = W * scale + 'px';
  stage.style.height = H * scale + 'px';
  const ratio = window.devicePixelRatio || 1;
  canvas.width = Math.round(W * scale * ratio);
  canvas.height = Math.round(H * scale * ratio);
  ctx.setTransform(scale * ratio, 0, 0, scale * ratio, 0, 0);
}


/* ---------------------------------------------------------
   7. СОСТОЯНИЕ ИГРЫ, ВРАГИ, БАШНИ, СНАРЯДЫ
   --------------------------------------------------------- */

// Состояния:
//   'menu'   — меню
//   'choose' — выбор сложности
//   'build'  — между волнами: строим башни и отвечаем на вопросы
//   'wave'   — идёт волна
//   'quiz'   — вопрос (игра на паузе)
//   'perk'   — выбор улучшения (пауза)
//   'paused' — пауза
//   'over'   — игра окончена
const game = {
  state: 'menu',
  stateBeforeQuiz: 'build',
  coins: 0,
  lives: 20,
  wave: 0,
  score: 0,
  answers: 0,
  time: 0,
  speed: 1,           // ×1 или ×2
  toSpawn: 0,         // сколько врагов волны ещё не вышло
  spawnTimer: 0,
  bossInWave: false,
  quizLock: 0,        // после неверного ответа кнопка «Вопрос» отдыхает
  stateBeforePause: 'build',
  usedPerks: []
};

// Общие бонусы всех башен (меняются улучшениями)
let bonus = null;

function resetBonus() {
  bonus = {
    damage: LAB.towerPower, rate: 1, range: 1, coins: 1, cost: 1,
    frost: false, crit: 0, blast: false, questionBonus: 0, poison: false,
    double: 0, waveIncome: 0, bossDamage: 1
  };
}

let enemies = [];
let towers = [];      // { pad, kind, level, cooldown }
let shots = [];       // снаряды
let particles = [];
let selectedPad = -1; // на каком месте открыто меню

function towerCost(kind) {
  if (LAB.freeTowers) return 0;
  return Math.round(TOWER_KINDS[kind].cost * bonus.cost);
}

function upgradeCost(level) {
  if (LAB.freeTowers) return 0;
  return Math.round(UPGRADE_COST[level] * bonus.cost);
}

// Характеристики башни с учётом уровня и улучшений
function towerStats(tower) {
  const base = TOWER_KINDS[tower.kind];
  const levelBoost = Math.pow(1.6, tower.level - 1);
  return {
    damage: base.damage * levelBoost * bonus.damage,
    range: (base.range + (tower.level - 1) * 15) * bonus.range,
    rate: base.rate * Math.pow(1.15, tower.level - 1) * bonus.rate,
    splash: base.splash || 0,
    slow: base.slow || (bonus.frost ? 0.8 : 0)
  };
}

function spawnEnemy(isBoss) {
  const theme = currentTheme();
  const d = difficulty();
  const waveHp = 18 * Math.pow(1.18, game.wave - 1) * d.enemyHp;
  const hp = Math.round(isBoss ? waveHp * 12 : waveHp);
  const start = PATH_POINTS[0];
  enemies.push({
    emoji: isBoss ? theme.boss.emoji : theme.enemies[Math.floor(Math.random() * theme.enemies.length)],
    boss: isBoss,
    x: start.x,
    y: start.y,
    target: 1,                  // к какой точке-повороту идёт
    travelled: 0,               // сколько прошёл (чтобы башни били первого)
    hp: hp,
    maxHp: hp,
    speed: Math.min(110, (48 + game.wave * 1.5) * d.speed * LAB.enemySpeed) * (isBoss ? 0.6 : 1),
    slowTimer: 0,
    slowFactor: 1,
    poison: 0,
    hit: 0
  });
}


/* ---------------------------------------------------------
   8. РИСОВАНИЕ: КАРТА, БАШНИ, ВРАГИ, ЧАСТИЦЫ
   --------------------------------------------------------- */

const TAU = Math.PI * 2;

function clamp(value, min, max) {
  return Math.max(min, Math.min(max, value));
}

function seeded(n) {
  const x = Math.sin(n * 127.1 + 311.7) * 43758.5453;
  return x - Math.floor(x);
}

function drawCircle(x, y, r, color) {
  ctx.fillStyle = color;
  ctx.beginPath();
  ctx.arc(x, y, r, 0, TAU);
  ctx.fill();
}

function drawGlow(x, y, r, color) {
  const g = ctx.createRadialGradient(x, y, 0, x, y, r);
  g.addColorStop(0, color);
  g.addColorStop(1, 'rgba(0,0,0,0)');
  ctx.fillStyle = g;
  ctx.fillRect(x - r, y - r, r * 2, r * 2);
}

function drawEmoji(emoji, x, y, size) {
  ctx.font = size + 'px "Apple Color Emoji", "Segoe UI Emoji", "Noto Color Emoji", sans-serif';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(emoji, x, y);
}

function hexToRgba(color, alpha) {
  if (!color.startsWith('#')) return color.replace('hsl(', 'hsla(').replace(')', ', ' + alpha + ')');
  const n = parseInt(color.slice(1), 16);
  return 'rgba(' + ((n >> 16) & 255) + ',' + ((n >> 8) & 255) + ',' + (n & 255) + ',' + alpha + ')';
}

function drawMap() {
  const theme = currentTheme();
  ctx.fillStyle = theme.ground;
  ctx.fillRect(0, 0, W, H);

  // Узор на земле
  for (let i = 0; i < 160; i++) {
    drawCircle(seeded(i) * W, seeded(i + 99) * H, 1 + seeded(i + 7) * 2, theme.groundDots);
  }

  // Дорога: широкая линия по точкам-поворотам
  ctx.lineCap = 'round';
  ctx.lineJoin = 'round';
  for (const [color, width] of [[theme.pathEdge, CELL * 0.92], [theme.path, CELL * 0.78]]) {
    ctx.strokeStyle = color;
    ctx.lineWidth = width;
    ctx.beginPath();
    PATH_POINTS.forEach((p, i) => (i === 0 ? ctx.moveTo(p.x, p.y) : ctx.lineTo(p.x, p.y)));
    ctx.stroke();
  }
  // Бегущие светящиеся точки по дороге — видно, куда идут враги
  drawPathFlow(theme.accent);

  // Декорации в пустых клетках
  for (let c = 0; c < COLS; c++) {
    for (let r = 1; r < ROWS; r++) {
      if (PATH_CELLS.has(c + ',' + r) || PADS.some(p => p[0] === c && p[1] === r)) continue;
      const n = c * 31 + r * 7;
      if (seeded(n) < 0.2) {
        const p = cellCenter(c, r);
        ctx.globalAlpha = 0.55;
        drawEmoji(theme.props[Math.floor(seeded(n + 1) * theme.props.length)], p.x + (seeded(n + 2) - 0.5) * 20, p.y + (seeded(n + 3) - 0.5) * 20, 26);
        ctx.globalAlpha = 1;
      }
    }
  }

  // База в конце дороги
  const end = PATH_POINTS[PATH_POINTS.length - 1];
  drawGlow(end.x, end.y, 70, hexToRgba(theme.accent, 0.35));
  drawEmoji(theme.base.emoji, end.x, end.y, 58);
}

function drawPathFlow(color) {
  ctx.fillStyle = hexToRgba(color, 0.35);
  let total = 0;
  for (let i = 0; i < PATH_POINTS.length - 1; i++) {
    const a = PATH_POINTS[i];
    const b = PATH_POINTS[i + 1];
    const length = Math.hypot(b.x - a.x, b.y - a.y);
    for (let d = (40 - (game.time * 40 - total) % 40) % 40; d < length; d += 40) {
      const t = d / length;
      ctx.beginPath();
      ctx.arc(a.x + (b.x - a.x) * t, a.y + (b.y - a.y) * t, 3, 0, TAU);
      ctx.fill();
    }
    total += length;
  }
}

function drawPads() {
  PADS.forEach(([c, r], index) => {
    const p = cellCenter(c, r);
    const tower = towers.find(t => t.pad === index);
    const selected = index === selectedPad;
    if (!tower) {
      ctx.fillStyle = selected ? hexToRgba(currentTheme().accent, 0.35) : 'rgba(255,255,255,0.08)';
      ctx.strokeStyle = hexToRgba(currentTheme().accent, selected ? 0.9 : 0.35);
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.roundRect(p.x - 24, p.y - 24, 48, 48, 12);
      ctx.fill();
      ctx.stroke();
      ctx.fillStyle = hexToRgba(currentTheme().accent, 0.7);
      ctx.font = 'bold 22px Rubik, sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText('+', p.x, p.y + 1);
    }
  });
}

function drawTowers() {
  const theme = currentTheme();
  towers.forEach(tower => {
    const [c, r] = PADS[tower.pad];
    const p = cellCenter(c, r);
    const stats = towerStats(tower);
    if (tower.pad === selectedPad) {
      // Круг дальности у выбранной башни
      ctx.strokeStyle = hexToRgba(theme.accent, 0.6);
      ctx.fillStyle = hexToRgba(theme.accent, 0.08);
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.arc(p.x, p.y, stats.range, 0, TAU);
      ctx.fill();
      ctx.stroke();
    }
    drawCircle(p.x, p.y + 4, 25, 'rgba(0,0,0,0.35)');
    drawCircle(p.x, p.y, 25, hexToRgba(theme.accent, 0.25 + tower.level * 0.12));
    const pop = tower.flash > 0 ? 1.15 : 1;
    drawEmoji(theme.towers[tower.kind].emoji, p.x, p.y - 2, 32 * pop);
    // Звёзды уровня
    ctx.fillStyle = '#ffd84a';
    ctx.font = 'bold 12px Rubik, sans-serif';
    ctx.textAlign = 'center';
    ctx.fillText('★'.repeat(tower.level), p.x, p.y + 25);
  });
}

function drawEnemies() {
  for (const e of enemies) {
    const size = e.boss ? 62 : 34;
    if (e.slowTimer > 0) drawGlow(e.x, e.y, size * 0.8, 'rgba(150,220,255,0.45)');
    if (e.poison > 0) drawGlow(e.x, e.y, size * 0.8, 'rgba(140,255,90,0.35)');
    drawEmoji(e.emoji, e.x, e.y, size * (e.hit > 0 ? 1.12 : 1));
    const w = e.boss ? 70 : 36;
    ctx.fillStyle = 'rgba(0,0,0,0.6)';
    ctx.fillRect(e.x - w / 2, e.y - size / 2 - 10, w, 5);
    ctx.fillStyle = e.hp / e.maxHp > 0.4 ? '#39d97a' : '#ff4d6d';
    ctx.fillRect(e.x - w / 2, e.y - size / 2 - 10, w * Math.max(0, e.hp / e.maxHp), 5);
  }
}

function drawShots() {
  const accent = currentTheme().accent;
  for (const s of shots) {
    const color = LAB.rainbowShots ? 'hsl(' + Math.round((game.time * 300 + s.x) % 360) + ', 100%, 60%)' : s.color || accent;
    drawGlow(s.x, s.y, 14, hexToRgba(color, 0.6));
    drawCircle(s.x, s.y, s.kind === 'splash' ? 6 : 4, color);
  }
}

function spawnParticles(x, y, count, color, power) {
  for (let i = 0; i < count; i++) {
    const angle = Math.random() * TAU;
    const speed = (0.3 + Math.random()) * power;
    particles.push({ x: x, y: y, vx: Math.cos(angle) * speed, vy: Math.sin(angle) * speed, life: 0.4 + Math.random() * 0.4, size: 2 + Math.random() * 3, color: color });
  }
}

function updateParticles(dt) {
  for (const p of particles) {
    p.x += p.vx * dt;
    p.y += p.vy * dt;
    p.vx *= 0.94;
    p.vy *= 0.94;
    p.life -= dt;
  }
  particles = particles.filter(p => p.life > 0);
}

function drawParticles() {
  for (const p of particles) {
    ctx.globalAlpha = Math.max(0, p.life * 2);
    drawCircle(p.x, p.y, p.size, p.color);
  }
  ctx.globalAlpha = 1;
}


/* ---------------------------------------------------------
   9. ОКНО ВОПРОСА (КВИЗ)
   Пока открыт вопрос, игра стоит на паузе.
   onCorrect — что сделать после верного ответа
   --------------------------------------------------------- */

let currentQuestion = null;
let quizLocked = false;
let quizReward = null;   // функция-награда за верный ответ

function openQuiz(hintText, onCorrect) {
  closeBuildMenu();
  game.stateBeforeQuiz = game.state;
  game.state = 'quiz';
  currentQuestion = nextQuestion();
  quizLocked = false;
  quizReward = onCorrect;
  const subject = findById(SUBJECTS, settings.subject);

  document.getElementById('quizSubject').textContent = subject.icon + ' ' + subject.label;
  document.getElementById('quizQuestion').innerHTML = currentQuestion.text +
    (currentQuestion.hint ? '<small>' + currentQuestion.hint + '</small>' : '');
  document.getElementById('quizHint').textContent = hintText;
  document.getElementById('quizCard').classList.remove('flash-bad', 'flash-good');

  const options = document.getElementById('quizOptions');
  const form = document.getElementById('quizForm');
  const inputEl = document.getElementById('quizInput');

  if (settings.inputMode === 'keyboard') {
    options.classList.add('hidden');
    form.classList.remove('hidden');
    inputEl.value = '';
    inputEl.inputMode = currentQuestion.numeric ? 'numeric' : 'text';
    showScreen('quiz');
    setTimeout(() => inputEl.focus(), 50);
  } else {
    form.classList.add('hidden');
    options.classList.remove('hidden');
    options.innerHTML = '';
    currentQuestion.options.forEach((text, i) => {
      const button = document.createElement('button');
      button.type = 'button';
      button.className = 'quiz-option';
      button.textContent = text;
      button.title = 'Клавиша ' + (i + 1);
      button.addEventListener('click', () => answerQuiz(text, button));
      options.appendChild(button);
    });
    showScreen('quiz');
  }
}

function answerQuiz(text, button) {
  if (quizLocked || game.state !== 'quiz') return;
  quizLocked = true;
  const correct = normalize(text) === normalize(currentQuestion.answer);
  const card = document.getElementById('quizCard');
  if (button) button.classList.add(correct ? 'right' : 'wrong');
  restartAnimation(card, correct ? 'flash-good' : 'flash-bad');

  if (correct) {
    game.answers++;
    game.score += 20;
    playSound('coin');
    setTimeout(() => {
      closeQuiz();
      if (quizReward) quizReward();
    }, 500);
  } else {
    playSound('crash');
    game.quizLock = 3; // кнопка «Вопрос» отдыхает 3 секунды
    for (const b of document.querySelectorAll('.quiz-option')) {
      if (normalize(b.textContent) === normalize(currentQuestion.answer)) b.classList.add('right');
    }
    document.getElementById('quizHint').textContent = 'Правильно: ' + currentQuestion.answer;
    setTimeout(closeQuiz, 1100);
  }
}

function closeQuiz() {
  if (game.state !== 'quiz') return;
  game.state = game.stateBeforeQuiz;
  showScreen(null);
  updateHud();
}

document.getElementById('quizForm').addEventListener('submit', (event) => {
  event.preventDefault();
  const value = document.getElementById('quizInput').value;
  if (value.trim()) answerQuiz(value, null);
});


/* ---------------------------------------------------------
   10. ЭКРАН УЛУЧШЕНИЙ
   --------------------------------------------------------- */

let perkOffer = [];

function openPerks() {
  const free = PERKS.filter(perk => !game.usedPerks.includes(perk.id));
  if (free.length === 0) {
    game.coins += 100;
    showToast('Все улучшения собраны! +100🪙');
    return;
  }
  game.stateBeforeQuiz = game.state;
  game.state = 'perk';
  perkOffer = shuffle(free).slice(0, 3);
  const box = document.getElementById('perkChoices');
  box.innerHTML = '';
  perkOffer.forEach((perk, i) => {
    box.appendChild(makeChoice(perk.icon, perk.title, perk.desc, () => pickPerk(perk), i + 1));
  });
  showScreen('perk');
}

function pickPerk(perk) {
  if (game.state !== 'perk') return;
  perk.apply(bonus);
  game.usedPerks.push(perk.id);
  playSound('start');
  showToast(perk.icon + ' ' + perk.title);
  game.state = game.stateBeforeQuiz;
  showScreen(null);
  updateHud();
}


/* ---------------------------------------------------------
   11. ИГРОВЫЕ ДЕЙСТВИЯ: БАШНИ, ВОЛНЫ, УРОН
   --------------------------------------------------------- */

// Кнопка «Вопрос»: верный ответ даёт монеты
function askForCoins() {
  if (!['build', 'wave'].includes(game.state) || game.quizLock > 0) return;
  const reward = LAB.questionReward + bonus.questionBonus;
  openQuiz('Верный ответ — +' + reward + ' монет!', () => {
    game.coins += reward;
    showToast('+' + reward + ' 🪙');
    updateHud();
  });
}

function buildTower(padIndex, kind) {
  const cost = towerCost(kind);
  if (game.coins < cost || towers.some(t => t.pad === padIndex)) return;
  game.coins -= cost;
  towers.push({ pad: padIndex, kind: kind, level: 1, cooldown: 0, flash: 0.3 });
  const [c, r] = PADS[padIndex];
  const p = cellCenter(c, r);
  spawnParticles(p.x, p.y, 16, currentTheme().accent, 160);
  playSound('jump');
  closeBuildMenu();
  updateHud();
}

function upgradeTower(tower) {
  if (tower.level >= 3) return;
  const cost = upgradeCost(tower.level);
  if (game.coins < cost) return;
  game.coins -= cost;
  tower.level++;
  tower.flash = 0.3;
  playSound('start');
  closeBuildMenu();
  updateHud();
}

function startWave() {
  if (game.state !== 'build') return;
  game.wave++;
  game.toSpawn = LAB.waveSize + (game.wave - 1) * 2;
  game.bossInWave = game.wave % LAB.bossEvery === 0;
  game.spawnTimer = 0.5;
  game.state = 'wave';
  if (bonus.waveIncome) game.coins += bonus.waveIncome;
  closeBuildMenu();
  playSound('tick');
  showToast(game.bossInWave ? 'Волна ' + game.wave + ': БОСС!' : 'Волна ' + game.wave);
  updateHud();
}

// Волна закончилась: награда и бонусный вопрос
function finishWave() {
  game.state = 'build';
  const prize = 30 + game.wave * 5;
  game.coins += prize;
  game.score += Math.round(100 * game.wave * difficulty().score);
  showToast('Волна пройдена! +' + prize + '🪙');
  updateHud();
  // Сразу бонусный вопрос: верный ответ — улучшение для всех башен
  openQuiz('Бонус за волну: верный ответ — улучшение для всех башен!', openPerks);
}

function damageEnemy(enemy, damage) {
  if (enemy.boss) damage *= bonus.bossDamage;
  if (Math.random() < bonus.crit) damage *= 2;
  enemy.hp -= damage;
  enemy.hit = 0.1;
}

function killEnemy(enemy) {
  enemy.dead = true;
  const reward = Math.round((enemy.boss ? 60 : 5 + game.wave) * bonus.coins);
  game.coins += reward;
  game.score += Math.round(enemy.maxHp / 2 * difficulty().score);
  spawnParticles(enemy.x, enemy.y, enemy.boss ? 50 : 16, currentTheme().accent, enemy.boss ? 260 : 170);
  if (bonus.blast) {
    for (const other of enemies) {
      if (!other.dead && Math.hypot(other.x - enemy.x, other.y - enemy.y) < 70) damageEnemy(other, enemy.maxHp * 0.3);
    }
  }
  if (enemy.boss) {
    showToast('Босс побеждён! +' + reward + '🪙');
    playSound('start');
  }
  updateHud();
}

function gameOver() {
  game.state = 'over';
  closeBuildMenu();
  const isRecord = game.score > bestScore;
  if (isRecord) {
    bestScore = game.score;
    saveBestScore();
  }
  document.getElementById('finalScore').textContent = game.score;
  document.getElementById('finalWaves').textContent = game.wave - 1;
  document.getElementById('finalAnswers').textContent = game.answers;
  document.getElementById('finalBest').textContent = bestScore;
  document.getElementById('newRecord').classList.toggle('hidden', !isRecord || game.score === 0);
  setTimeout(() => showScreen('over'), 600);
}

function resetGame() {
  resetBonus();
  game.coins = LAB.startCoins;
  game.lives = LAB.baseLives;
  game.wave = 0;
  game.score = 0;
  game.answers = 0;
  game.speed = 1;
  game.quizLock = 0;
  game.usedPerks = [];
  enemies = [];
  towers = [];
  shots = [];
  particles = [];
  selectedPad = -1;
  buildQuestionPool();
  updateHud();
}

function openDifficultySelect() {
  game.state = 'choose';
  const box = document.getElementById('difficultyChoices');
  box.innerHTML = '';
  DIFFICULTIES.forEach((d, i) => {
    box.appendChild(makeChoice(d.icon, d.label, d.desc, () => {
      settings.difficulty = d.id;
      saveSettings();
      startGame();
    }, i + 1));
  });
  showScreen('difficulty');
}

function startGame() {
  if (audioContext.state === 'suspended') audioContext.resume();
  resetGame();
  game.state = 'build';
  showScreen(null);
  showToast('Строй башни и жми «Волна»!');
  playSound('start');
}

function pauseGame() {
  if (!['build', 'wave'].includes(game.state)) return;
  game.stateBeforePause = game.state;
  game.state = 'paused';
  closeBuildMenu();
  showScreen('pause');
}

function resumeGame() {
  if (game.state !== 'paused') return;
  game.state = game.stateBeforePause;
  showScreen(null);
}

function goToMenu() {
  game.state = 'menu';
  closeBuildMenu();
  showScreen('menu');
}


/* ---------------------------------------------------------
   12. ОБНОВЛЕНИЕ МИРА ЗА ОДИН КАДР
   --------------------------------------------------------- */

function update(dt) {
  if (['quiz', 'perk', 'paused'].includes(game.state)) return; // жёсткая пауза
  game.time += dt;
  updateParticles(dt);
  if (game.quizLock > 0) {
    game.quizLock -= dt;
    if (game.quizLock <= 0) updateHud();
  }
  if (!['build', 'wave'].includes(game.state)) return;

  // Ускорение ×2: просто считаем два маленьких шага вместо одного
  for (let step = 0; step < game.speed; step++) {
    updateSpawning(dt);
    updateEnemies(dt);
    updateTowers(dt);
    updateShots(dt);
    if (game.state === 'over') return;
  }

  if (game.state === 'wave' && game.toSpawn === 0 && !game.bossInWave && enemies.length === 0) finishWave();
}

function updateSpawning(dt) {
  if (game.state !== 'wave') return;
  game.spawnTimer -= dt;
  if (game.spawnTimer > 0) return;
  if (game.toSpawn > 0) {
    spawnEnemy(false);
    game.toSpawn--;
    game.spawnTimer = Math.max(0.45, 1.1 - game.wave * 0.03);
  } else if (game.bossInWave) {
    spawnEnemy(true);           // босс выходит последним
    game.bossInWave = false;
    showToast('Босс: ' + currentTheme().boss.name + '!');
    playSound('start');
  }
}

function updateEnemies(dt) {
  for (const e of enemies) {
    if (e.hit > 0) e.hit -= dt;
    if (e.slowTimer > 0) e.slowTimer -= dt;
    if (e.poison > 0) {
      e.poison -= dt;
      e.hp -= 4 * dt;
    }
    const speed = e.speed * (e.slowTimer > 0 ? e.slowFactor : 1);
    let move = speed * dt;
    // Идём к следующей точке-повороту
    while (move > 0 && e.target < PATH_POINTS.length) {
      const goal = PATH_POINTS[e.target];
      const dist = Math.hypot(goal.x - e.x, goal.y - e.y);
      if (dist <= move) {
        e.x = goal.x;
        e.y = goal.y;
        e.target++;
        move -= dist;
        e.travelled += dist;
      } else {
        e.x += (goal.x - e.x) / dist * move;
        e.y += (goal.y - e.y) / dist * move;
        e.travelled += move;
        move = 0;
      }
    }
    // Дошёл до базы
    if (e.target >= PATH_POINTS.length) {
      e.dead = true;
      game.lives -= e.boss ? 5 : 1;
      playSound('crash');
      spawnParticles(e.x, e.y, 20, '#ff4d6d', 200);
      if (game.lives <= 0) {
        game.lives = 0;
        updateHud();
        gameOver();
        return;
      }
      updateHud();
    }
    if (e.hp <= 0 && !e.dead) killEnemy(e);
  }
  enemies = enemies.filter(e => !e.dead);
}

// Башня бьёт первого врага (дальше всех прошедшего) в радиусе
function updateTowers(dt) {
  for (const tower of towers) {
    if (tower.flash > 0) tower.flash -= dt;
    tower.cooldown -= dt;
    if (tower.cooldown > 0) continue;
    const stats = towerStats(tower);
    const [c, r] = PADS[tower.pad];
    const p = cellCenter(c, r);
    let target = null;
    for (const e of enemies) {
      if (Math.hypot(e.x - p.x, e.y - p.y) <= stats.range && (!target || e.travelled > target.travelled)) target = e;
    }
    if (!target) continue;
    fire(tower, p, target, stats);
    if (Math.random() < bonus.double) fire(tower, p, target, stats);
    tower.cooldown = 1 / stats.rate;
  }
}

function fire(tower, from, target, stats) {
  shots.push({
    x: from.x, y: from.y - 10, target: target, kind: tower.kind,
    damage: stats.damage, splash: stats.splash, slow: stats.slow,
    color: tower.kind === 'slow' ? '#9fe6ff' : tower.kind === 'splash' ? '#ff9f43' : null
  });
}

function updateShots(dt) {
  for (const s of shots) {
    const t = s.target;
    if (t.dead) { s.dead = true; continue; }   // цель уже побеждена
    const dist = Math.hypot(t.x - s.x, t.y - s.y);
    const step = 460 * dt;
    if (dist <= step) {
      s.dead = true;
      hitByShot(s, t);
    } else {
      s.x += (t.x - s.x) / dist * step;
      s.y += (t.y - s.y) / dist * step;
    }
  }
  shots = shots.filter(s => !s.dead);
}

function hitByShot(shot, target) {
  const victims = shot.splash
    ? enemies.filter(e => Math.hypot(e.x - target.x, e.y - target.y) <= shot.splash)
    : [target];
  for (const e of victims) {
    damageEnemy(e, shot.damage);
    if (shot.slow) {
      e.slowTimer = 1.5;        // замедлен полторы секунды
      e.slowFactor = shot.slow;
    }
    if (bonus.poison) e.poison = 3;
  }
  spawnParticles(target.x, target.y, shot.splash ? 14 : 5, shot.color || currentTheme().accent, shot.splash ? 180 : 110);
}


/* ---------------------------------------------------------
   13. РИСОВАНИЕ ОДНОГО КАДРА
   --------------------------------------------------------- */

function draw() {
  drawMap();
  if (game.state !== 'menu') {
    drawPads();
    drawTowers();
    drawEnemies();
    drawShots();
  }
  drawParticles();
}


/* ---------------------------------------------------------
   14. ЭКРАНЫ И ИНТЕРФЕЙС
   --------------------------------------------------------- */

const screens = {
  menu: document.getElementById('menuScreen'),
  difficulty: document.getElementById('difficultyScreen'),
  quiz: document.getElementById('quizScreen'),
  perk: document.getElementById('perkScreen'),
  settings: document.getElementById('settingsScreen'),
  pause: document.getElementById('pauseScreen'),
  over: document.getElementById('gameOverScreen')
};

function showScreen(name) {
  for (const key in screens) {
    screens[key].classList.toggle('hidden', key !== name);
  }
  const inGame = name === null || ['pause', 'quiz', 'perk'].includes(name);
  document.getElementById('hud').classList.toggle('hidden', !inGame);
  document.getElementById('menuBest').textContent = bestScore;
  if (document.activeElement && document.activeElement.blur) document.activeElement.blur();
}

function isVisible(el) {
  return !el.classList.contains('hidden');
}

function makeChoice(icon, title, desc, onPick, key) {
  const button = document.createElement('button');
  button.type = 'button';
  button.className = 'choice';
  button.innerHTML = '<span class="choice-icon">' + icon + '</span><span><span class="choice-title">' + title + '</span>' +
    (desc ? '<br><span class="choice-desc">' + desc + '</span>' : '') + '</span>';
  button.title = 'Клавиша ' + key;
  button.addEventListener('click', onPick);
  return button;
}

function updateHud() {
  document.getElementById('coins').textContent = game.coins;
  document.getElementById('lives').textContent = game.lives;
  document.getElementById('wave').textContent = Math.max(1, game.wave);
  document.getElementById('score').textContent = game.score;
  const quizButton = document.getElementById('quizButton');
  quizButton.disabled = game.quizLock > 0;
  quizButton.textContent = '❓ Вопрос +' + (LAB.questionReward + (bonus ? bonus.questionBonus : 0)) + '🪙';
  const waveButton = document.getElementById('waveButton');
  waveButton.disabled = game.state !== 'build';
  waveButton.textContent = game.state === 'wave' ? '⚔️ Идёт волна ' + game.wave : '▶ Волна ' + (game.wave + 1);
  document.getElementById('speedButton').textContent = '⏩ ×' + game.speed;
  document.getElementById('perkChips').innerHTML = game.usedPerks
    .map(id => '<span class="chip">' + PERKS.find(p => p.id === id).icon + '</span>').join('');
}

// Меню постройки или улучшения возле места
function openBuildMenu(padIndex) {
  selectedPad = padIndex;
  const theme = currentTheme();
  const menu = document.getElementById('buildMenu');
  const tower = towers.find(t => t.pad === padIndex);
  let html = '';
  if (!tower) {
    html += '<div class="build-title">Построить башню</div>';
    ['fast', 'splash', 'slow'].forEach((kind, i) => {
      const cost = towerCost(kind);
      html += '<button class="build-item" data-kind="' + kind + '"' + (game.coins < cost ? ' disabled' : '') + '>' +
        '<span class="build-icon">' + theme.towers[kind].emoji + '</span>' +
        '<span class="build-name">' + (i + 1) + '. ' + theme.towers[kind].name + '<small>' + TOWER_KINDS[kind].desc + '</small></span>' +
        '<span class="build-cost">' + cost + '🪙</span></button>';
    });
  } else {
    const name = theme.towers[tower.kind].name;
    if (tower.level < 3) {
      const cost = upgradeCost(tower.level);
      html += '<div class="build-title">' + name + ' · уровень ' + tower.level + '</div>' +
        '<button class="build-item" data-upgrade="1"' + (game.coins < cost ? ' disabled' : '') + '>' +
        '<span class="build-icon">⬆️</span><span class="build-name">1. Улучшить<small>Урон ×1.6, дальность +15</small></span>' +
        '<span class="build-cost">' + cost + '🪙</span></button>';
    } else {
      html += '<div class="build-title">' + name + ' · максимум ★★★</div>';
    }
  }
  menu.innerHTML = html;
  menu.querySelectorAll('[data-kind]').forEach(b => b.addEventListener('click', () => buildTower(padIndex, b.dataset.kind)));
  menu.querySelectorAll('[data-upgrade]').forEach(b => b.addEventListener('click', () => upgradeTower(tower)));

  // Ставим меню рядом с местом, но не за краем экрана
  const [c, r] = PADS[padIndex];
  const p = cellCenter(c, r);
  menu.classList.remove('hidden');
  const left = clamp(p.x * scale + 34 * scale, 4, W * scale - menu.offsetWidth - 4);
  // Сверху меню не заезжает на панель, снизу — на кнопки
  const hudBottom = document.querySelector('.hud-top').getBoundingClientRect().bottom - stage.getBoundingClientRect().top + 6;
  const top = clamp(p.y * scale - menu.offsetHeight / 2, hudBottom, H * scale - menu.offsetHeight - 60);
  menu.style.left = left + 'px';
  menu.style.top = top + 'px';
}

function closeBuildMenu() {
  selectedPad = -1;
  document.getElementById('buildMenu').classList.add('hidden');
}

function showToast(text) {
  const el = document.getElementById('toast');
  el.textContent = text;
  restartAnimation(el, 'show');
}

function restartAnimation(el, className) {
  el.classList.remove(className);
  void el.offsetWidth;
  el.classList.add(className);
}

function applyTheme() {
  const theme = currentTheme();
  const subject = findById(SUBJECTS, settings.subject);
  document.documentElement.style.setProperty('--accent', theme.accent);
  document.getElementById('baseLabel').textContent = theme.base.emoji + ' ' + theme.base.name;
  document.getElementById('menuInfo').textContent =
    theme.icon + ' ' + theme.label + ' · ' + subject.icon + ' ' + subject.label + ' · ' + findById(INPUT_MODES, settings.inputMode).label;
}


/* ---------------------------------------------------------
   15. ЭКРАН НАСТРОЕК
   --------------------------------------------------------- */

function buildOptions(containerId, items, settingName) {
  const container = document.getElementById(containerId);
  container.innerHTML = '';
  for (const item of items) {
    const button = document.createElement('button');
    button.type = 'button';
    button.className = 'option' + (settings[settingName] === item.id ? ' selected' : '');
    button.innerHTML = '<span class="option-icon">' + item.icon + '</span><span class="option-title">' + item.label + '</span><span>' + item.desc + '</span>';
    button.addEventListener('click', () => {
      settings[settingName] = item.id;
      saveSettings();
      applyTheme();
      buildSettings();
    });
    container.appendChild(button);
  }
}

function buildSettings() {
  buildOptions('themeOptions', THEMES, 'theme');
  buildOptions('subjectOptions', SUBJECTS, 'subject');
  buildOptions('inputOptions', INPUT_MODES, 'inputMode');
  document.getElementById('soundToggle').checked = settings.sound;
  document.getElementById('volumeRange').value = Math.round(settings.volume * 100);
  document.getElementById('volumeRange').disabled = !settings.sound;
}

document.getElementById('soundToggle').addEventListener('change', (event) => {
  settings.sound = event.target.checked;
  saveSettings();
  buildSettings();
  playSound('coin');
});

document.getElementById('volumeRange').addEventListener('change', (event) => {
  settings.volume = Number(event.target.value) / 100;
  saveSettings();
  playSound('coin');
});

document.getElementById('resetBestButton').addEventListener('click', () => {
  bestScore = 0;
  saveBestScore();
  showToast('Рекорд сброшен');
});


/* ---------------------------------------------------------
   16. УПРАВЛЕНИЕ
   --------------------------------------------------------- */

// Нажатие на карту: место для башни → меню, иначе меню закрывается
canvas.addEventListener('pointerdown', (event) => {
  if (!['build', 'wave'].includes(game.state)) return;
  const rect = canvas.getBoundingClientRect();
  const x = (event.clientX - rect.left) / rect.width * W;
  const y = (event.clientY - rect.top) / rect.height * H;
  const padIndex = PADS.findIndex(([c, r]) => {
    const p = cellCenter(c, r);
    return Math.abs(p.x - x) < 30 && Math.abs(p.y - y) < 30;
  });
  if (padIndex >= 0 && padIndex !== selectedPad) openBuildMenu(padIndex);
  else closeBuildMenu();
});

document.addEventListener('keydown', (event) => {
  if (document.activeElement && document.activeElement.id === 'quizInput') return;
  const code = event.code;
  if (code === 'Space') event.preventDefault();

  // Цифры 1–4: вариант ответа, сложность, улучшение или пункт меню башни
  const digit = Number(event.key);
  if (digit >= 1 && digit <= 4) {
    let list = null;
    if (game.state === 'quiz') list = document.querySelectorAll('.quiz-option');
    else if (isVisible(screens.difficulty)) list = document.querySelectorAll('#difficultyChoices .choice');
    else if (game.state === 'perk') list = document.querySelectorAll('#perkChoices .choice');
    else if (selectedPad >= 0) list = document.querySelectorAll('#buildMenu .build-item');
    if (list && list[digit - 1]) list[digit - 1].click();
  }

  if (code === 'KeyQ') askForCoins();
  if (code === 'KeyF') toggleSpeed();
  if (code === 'Space' || code === 'Enter') {
    if (isVisible(screens.menu)) openDifficultySelect();
    else if (isVisible(screens.over)) startGame();
    else if (game.state === 'build') startWave();
  }
  if (code === 'Escape' || code === 'KeyP') {
    if (selectedPad >= 0) closeBuildMenu();
    else if (['build', 'wave'].includes(game.state)) pauseGame();
    else if (game.state === 'paused') resumeGame();
    else if (isVisible(screens.settings)) showScreen('menu');
  }
});

function toggleSpeed() {
  if (!['build', 'wave'].includes(game.state)) return;
  game.speed = game.speed === 1 ? 2 : 1;
  updateHud();
}

document.getElementById('quizButton').addEventListener('click', askForCoins);
document.getElementById('waveButton').addEventListener('click', startWave);
document.getElementById('speedButton').addEventListener('click', toggleSpeed);
document.getElementById('startButton').addEventListener('click', openDifficultySelect);
document.getElementById('settingsButton').addEventListener('click', () => {
  buildSettings();
  showScreen('settings');
});
document.getElementById('settingsClose').addEventListener('click', () => showScreen('menu'));
document.getElementById('settingsDone').addEventListener('click', () => showScreen('menu'));
document.getElementById('pauseButton').addEventListener('click', pauseGame);
document.getElementById('resumeButton').addEventListener('click', resumeGame);
document.getElementById('pauseMenuButton').addEventListener('click', goToMenu);
document.getElementById('restartButton').addEventListener('click', startGame);
document.getElementById('overMenuButton').addEventListener('click', goToMenu);

document.addEventListener('visibilitychange', () => {
  if (document.hidden) pauseGame();
});

window.addEventListener('resize', () => {
  resizeGame();
  closeBuildMenu();
});


/* ---------------------------------------------------------
   17. ИГРОВОЙ ЦИКЛ И ЗАПУСК
   --------------------------------------------------------- */

let lastTime = performance.now();

function loop(now) {
  let dt = (now - lastTime) / 1000;
  lastTime = now;
  if (dt > 0.05) dt = 0.05;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}

document.querySelector('.title-top').textContent = LAB.titleTop;
document.querySelector('.title-bottom').textContent = LAB.titleBottom;
document.title = LAB.titleTop + ' ' + LAB.titleBottom;
resetBonus();
resizeGame();
applyTheme();
showScreen('menu');
requestAnimationFrame(loop);
````

<!-- SOURCES:END -->
