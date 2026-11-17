# AGENTS.md — памятка для правок shiki dusk

короткая навигация для ai-ассистентов (и для себя в будущем). устройство повторяет [Hikisey/shikimori-theme](https://github.com/Hikisey/shikimori-theme) (shiki glass): модули, сборка частями, проверка фильтра шикимори. подробности — в [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## что где лежит

исходники — модули в `src/`, порядок каскада задаёт `src/manifest.txt` (сверху вниз). `css/part_NN.css` — собранные части для `@import`, руками их не правят.

| слой | модули | что внутри |
|---|---|---|
| токены | `tokens/palette.css`, `tokens/components.css` | палитра (rgb-тройки) и родные переменные шикимори; токены `--line*`, `--plate-*`, `--drop-*`, `--heading-*`, `--btn-*`, `--chip-*`, `--field-*`, `--row-*`, `--bar-*`, `--ease-*` |
| база | `base/base.css` | текст, ссылки, таблицы, выделение, скроллбары |
| каркас | `layout/page.css`, `layout/header.css`, `layout/footer.css` | фон и обои, `.l-page`; верхнее меню и поиск; подвал и `--theme-version` |
| компоненты | `components/*.css` | `headings`, `buttons`, `forms`, `modals`, `badges`, `cards`, `stats`, `users`, `comments`, `editor`, `pagination` |
| страницы | `pages/*.css` | `anime` (тайтл, видео, плеер), `forum`, `profile`, `dashboard`, `catalog` (фильтры, сезон), `lists`, `achievements`, `moderation` |
| поздние | `overrides/late.css` | правила, которые обязаны идти после всех модулей; у каждого помечено `/* из <модуль> */` |

- **устройство модуля:** `@media screen and (min-width: 1px) { … }` — общие правила для всех ширин; затем `@media (hover: hover) and (pointer: fine) { … }` — все `:hover`; в конце раздел `/* ----- телефон и узкие экраны ----- */`.
- **токены.** новые правила пишите через токены (`background: var(--chip-bg)`, `color: rgb(var(--text-2))`), а не числами. новый общий вид — новый токен в `tokens/components.css`. оттенки — через `color-mix(in srgb, rgb(var(--x)) N%, rgb(var(--y)))`.
- **порядок — это каскад.** новые правила — в конец своего блока в своём модуле.
- `docs/my.original.css` — исходный однофайловый стиль, из которого собрана тема (для сверки, в сборку не входит).

## сборка и проверки

- **сборка:** `python3 tools/build.py [--version X.Y.Z]` — склеивает модули по манифесту в `css/part_NN.css` (≤150 КБ на часть, не больше 4 частей) и печатает строки `@import`.
- **проверка:** `python3 tools/quick_check.py [--page сохранённая.html] [--phone]` — сборка во временную папку, эмуляция фильтра шикимори, ссылки после camo, запрещённые слова и слэши, счётчик `:hover` вне медиазапроса, скриншот (нужен node + playwright-core + chrome).

## выпуск релиза

1. правки в `src/` → `python3 tools/build.py --version X.Y.Z` → `tools/quick_check.py` без ошибок.
2. в `site.css` и `README.md` заменить старый тег на новый во всех `@import`; строка `| vX.Y.Z | … |` в таблице «история» `docs/ARCHITECTURE.md`.
3. commit → push в `main` → `git tag vX.Y.Z` (или `git tag -a vX.Y.Z -m "описание"`) → `git push origin vX.Y.Z`. шикимори кеширует каждый адрес на 8 часов, новый тег — новый адрес. опубликованные теги не переносить.
4. дальше всё делает GitHub Actions (`.github/workflows/release.yml`): сверяет тег с версией в подвале и ссылками в `site.css`/`README.md`, проверяет, что `css/` собран из `src/`, гоняет `quick_check.py`, создаёт release (описание — из сообщения тега или строки истории, файлы частей и `site.css` прикреплены) и проверяет, что raw.githubusercontent отдаёт файлы тега. если проверка упала — release не создаётся: поправить, удалить тег (`git push origin :vX.Y.Z`, пока релиза нет) и поставить заново.
5. `.github/workflows/check.yml` гоняет те же проверки на каждый push в `main` и pull request.

## жёсткие правила (фильтр шикимори)

- css без `@media` шикимори оборачивает в `@media only screen and (min-width: 1024px)` — поэтому всё в `@media`.
- без минификации; `;` перед `}` после `url()`.
- нет слов `behavior`, eval, cookie, window, parent, this, script, expression, javascript, moz-binding, charset и `<` (как отдельных слов — даже в комментариях).
- нет `@view-transition`, нет обратных слэшей вне `content: "…"`, data uri только base64-картинки, без `@` в ссылках.
- `:hover` — только в `@media (hover: hover) and (pointer: fine)`; на сенсорах отклик через `:active`.
- правки только для телефона — в `@media screen and (max-width: 767px)`.
- `-webkit-backdrop-filter` — только фиксированные значения, без `var()`.
- наведение не должно сдвигать сам элемент под курсором.
- название темы — строчными: «shiki dusk».
