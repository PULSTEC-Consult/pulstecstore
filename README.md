# pulstecstore

# Pulstec plugins

[Русская версия](README.ru.md)

The plugin store for the Pulstec office suite, and the plugins themselves.

A plugin here is **one JSON file**, usually between two and twenty kilobytes. No build
step, no bundler, no `node_modules`, nothing to install on the developer's machine. You
write the file, drop it into the application, and it runs.

---

## What is in this repository

```
store.json                              store index: what this store is and what it offers
plugins/<name>/<name>.plsplug.json      the plugin itself, a single file
plugins/<name>/icon.svg                 store icon
plugins/<name>/screen-1.png             screenshots — shown in the store, never shipped in the plugin
plugins/<name>/data/…                   large resources, when a plugin needs them
docs/                                   the plugin format and the store format
tools/build_store.py                    rebuilds store.json from the plugin files
```

`store.json` is generated, not written by hand. It carries the size and the SHA-256 of
every plugin file, and the application verifies both before installing anything.

---

## A plugin in full

This is a complete, working plugin — the whole file, not an excerpt:

```json
{
  "id": "hello",
  "name": "Hello",
  "version": "1.0",
  "about": "Writes a greeting into the current cell.",
  "apps": ["cell"],
  "panel": {
    "title": "Hello",
    "sections": [
      {
        "name": "Greeting",
        "fields": [
          {"key": "who", "name": "Address to", "type": "text", "default": "world"}
        ],
        "buttons": [
          {"name": "Insert", "function": "Insert", "primary": true}
        ]
      }
    ]
  },
  "code": "function Insert(s){ cell().value = 'Hello, ' + s.who + '!'; }"
}
```

Every key has a Russian name as well as an English one — `имя` / `name`, `код` / `code`,
`панель` / `panel`. Pick either; the application reads both.

---

## What a plugin can do

| | |
|---|---|
| **Interface** | A side panel, free-standing windows, its own group on the ribbon, items in the cell context menu. All of it is *described* in the file — the application draws it, so a plugin never injects markup of its own. |
| **Forms** | Nineteen kinds of field: text, password, multiline, source code with highlighting, number, slider, date, colour, checkbox, radio, dropdown, list, tree, table, cards, preview, progress, label, link. Fields can show and hide each other by condition. |
| **Data** | Read and write cells, ranges, sheets; paragraphs and formatted text in documents; slides and shapes; tasks and schedules in projects. Large workbooks are read on request, in chunks or as an aggregate, rather than copied wholesale. |
| **Events** | On start, on selection change, on edit, on double click, on menu open. A background plugin keeps running while it is installed. |
| **Formulas** | A plugin can add its own worksheet functions, including asynchronous ones: the cell shows a pending marker, and the value arrives when it is ready. |
| **Drawing** | A canvas in the panel or in a window, pictures inserted into the sheet, and plugin objects — a picture that carries the plugin's own data and opens for editing on a double click. |
| **Outside world** | HTTP requests to addresses the user has allowed, files chosen and saved by the user, electronic signature through the system provider, and storage that lives outside the document. |

Everything a plugin does to a document arrives as a single undo step, so one click is one
`Ctrl+Z`.

---

## Security

Plugin code is not trusted, and the design assumes it never will be.

- It runs in a worker inside a sandboxed frame with an opaque origin — no access to the
  page, its storage, or the document except through messages.
- Before the plugin's own code starts, a lock strips network access, dynamic import and
  storage from the global object and its prototypes.
- Network requests go out through the application, checked against the list of addresses
  the user approved. Requests to anywhere else are refused.
- Rights are shown on the store card **before** anything is downloaded: which addresses
  the plugin wants, which worksheet functions it adds, what it reads from the document,
  whether it uses files, signature or persistent storage.
- What is actually requested at install time is read from the plugin file, not from the
  index: the index sits next to the file and could lie about it.
- Size and SHA-256 are verified against the index, and the `id` inside the file must match
  the `id` in the index. Any mismatch stops the installation and says which check failed.
- Secrets a plugin stores — service keys, tokens — never enter the document and never
  enter an exported copy of the plugin.

---

## Connecting this store

The application takes a store from any of four places, so a machine with no internet
access is as well served as one with:

- **GitHub** — `owner/repository`, with a branch if you need one.
- **Your own server** — any web server holding a copy. In a browser it must send
  `Access-Control-Allow-Origin: *`; the Windows application fetches the files itself and
  needs no such header.
- **A folder** — a clone or a copy on disk or on a network share. No network at all.
- **An archive** — a zip of the store, for isolated machines.

---

## Contributing a plugin

1. Fork the repository.
2. Put your file in `plugins/<name>/<name>.plsplug.json`, with an `icon.svg` beside it.
   Keep the folder name in Latin letters — it ends up in a URL — even if the plugin `id`
   is not.
3. Run `python3 tools/build_store.py`. It fills in the size, the SHA-256 and the rights,
   and refuses to build if anything disagrees with the plugin file.
4. Open a pull request.

Keep it small: a plugin description up to 100 KB (two to twenty is normal), an icon up to
4 KB. Screenshots stay in the store and are never part of the plugin.

Before sending, check that the plugin installs from a local folder source and that
removing it leaves nothing behind — no worksheet functions, no menu items, no storage.

---

## Licence

MIT — see [LICENSE](LICENSE). A translation into Russian, for reading rather than for
legal use, is in [LICENSE.md](LICENSE.md).

The name Pulstec and the Pulstec logo are not covered by this licence.

# Плагины Пульстек

[In English](README.md)

-----------------------------------------------------------------------------------------------

Магазин плагинов офисного пакета Пульстек и сами плагины.

Плагин здесь — **один файл JSON**, обычно от двух до двадцати килобайт. Ни сборки, ни
упаковщика, ни `node_modules`, ничего не нужно ставить на машину разработчика. Написали
файл, подключили в приложение — он работает.

---

## Содержание репозитория

```
store.json                              индекс магазина: что это за магазин и что в нём есть
plugins/<имя>/<имя>.plsplug.json        сам плагин, одним файлом
plugins/<имя>/icon.svg                  значок витрины
plugins/<имя>/screen-1.png              снимки — показываются в витрине, в плагин не попадают
plugins/<имя>/data/…                    большие ресурсы, если плагину они нужны
docs/                                   формат плагина и формат магазина
tools/build_store.py                    пересобирает store.json по файлам плагинов
```

`store.json` собирается, а не пишется руками. В нём лежат размер и sha256 каждого файла
плагина, и приложение сверяет то и другое перед установкой.

---

## Плагин целиком

Это готовый работающий плагин — файл полностью:

```json
{
  "id": "hello",
  "имя": "Привет",
  "версия": "1.0",
  "описание": "Пишет приветствие в текущую клетку.",
  "приложения": ["таблицы"],
  "панель": {
    "заголовок": "Привет",
    "разделы": [
      {
        "имя": "Приветствие",
        "поля": [
          {"ключ": "кому", "имя": "Кого приветствовать", "тип": "текст", "умолчание": "мир"}
        ],
        "кнопки": [
          {"имя": "Вставить", "функция": "Вставить", "главная": true}
        ]
      }
    ]
  },
  "код": "function Вставить(н){ ячейка().значение = 'Привет, ' + н.кому + '!'; }"
}
```

У каждого ключа есть и русское имя, и английское — `имя` / `name`, `код` / `code`,
`панель` / `panel`. Пишите как удобнее, приложение понимает оба.

---

## Что умеет плагин

| | |
|---|---|
| **Интерфейс** | Боковая панель, отдельные окна, своя группа на ленте, пункты в контекстном меню клетки. Всё это плагин *описывает*, а рисует приложение — поэтому своей разметки плагин создать не может. |
| **Формы** | Девятнадцать видов полей: текст, пароль, многострочный, код с подсветкой, число, ползунок, дата, цвет, флажок, переключатель, выпадающий список, список, дерево, таблица, карточки, предпросмотр, ход выполнения, надпись, ссылка. Поля умеют "показывать" и "прятать" друг друга по условию. |
| **Данные** | Чтение и запись клеток, диапазонов, листов; абзацы и форматированный текст в документах; слайды и фигуры; задачи и расписание в проектах. Большие книги читаются по запросу — кусками или итогом, а не полным копированием. |
| **События** | При запуске, при смене выделения, при правке, по двойному щелчку, при открытии меню. Фоновый плагин живёт, пока установлен. |
| **Формулы** | Плагин может добавить свои функции, в том числе асинхронные. |
| **Рисование** | Холст в панели или в окне, картинки на листе и объекты плагина — картинка со своими данными, которая открывается на правку двойным щелчком. |
| **Наружу** | Запросы по HTTP на адреса, которые разрешил пользователь; файлы, выбранные и сохранённые им же; электронная подпись через системное средство; хранилище, вне документа. |


---

## Как подключить этот магазин

Приложение берёт магазин любым из четырёх вариантов:

- **GitHub** — «владелец/репозиторий» и, если нужно, ветка.
- **Свой сервер** — любой веб-сервер с копией. В браузере он обязан отдавать файлы с
  заголовком `Access-Control-Allow-Origin: *`; приложение для Windows забирает файлы само,
  и заголовок ему не нужен.
- **Папка** — клон или копия на диске либо на сетевом ресурсе. Совсем без сети.
- **Архив** — zip магазина, для изолированных машин.

---

## Как прислать свой плагин

1. Сделайте форк репозитория.
2. Положите файл в `plugins/<имя>/<имя>.plsplug.json`, рядом — `icon.svg`. Имя папки
   пишите латиницей (оно уходит в ссылку), даже если `id` плагина русский.
3. Запустите `python3 tools/build_store.py`. Он проставит размер, sha256 и права и
   откажется собирать индекс, если что-то расходится с файлом плагина.
4. Откройте запрос на слияние.

Старайтесь держать вес небольшим: описание плагина до 100 КБ (обычно два-двадцать), значок до 4 КБ.

---

## Лицензия

MIT — см. [LICENSE](LICENSE). Перевод на русский, для чтения, а не для юридического
применения, — в [LICENSE.md](LICENSE.md).

Имя Пульстек и логотип Пульстек этой лицензией не передаются.
