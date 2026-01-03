# Multer [![NPM Version][npm-version-image]][npm-url] [![NPM Downloads][npm-downloads-image]][npm-url] [![Build Status][ci-image]][ci-url] [![Test Coverage][test-image]][test-url] [![OpenSSF Scorecard Badge][ossf-scorecard-badge]][ossf-scorecard-visualizer]

Multer — це middleware для фреймворка express для обробки `multipart/form-data`, потрыбна в першу чергу при завантаженні файлів. Написана як обгортка над [busboy](https://github.com/mscdex/busboy) для її максимально эфективного використання.

**ВАЖНО**: Multer не обробляє ніяких інших тип форм, крім `multipart/form-data`.

## Переклади

Ця README також доступна на інших мовах:

- [العربية](https://github.com/expressjs/multer/blob/main/doc/README-ar.md) (арабська)
- [English](https://github.com/expressjs/multer/blob/main/README.md) (Англійська)
- [Español](https://github.com/expressjs/multer/blob/main/doc/README-es.md) (Іспанська)
- [简体中文](https://github.com/expressjs/multer/blob/main/doc/README-zh-cn.md) (Китайска)
- [한국어](https://github.com/expressjs/multer/blob/main/doc/README-ko.md) (Корейска)
- [Português](https://github.com/expressjs/multer/blob/main/doc/README-pt-br.md) (бр Португальска)

## Встановлення

```sh
$ npm install --save multer
```

## Використання

Multer додає об'єкт `body` та об'єкт `file` (або `files`) всередену об'єкта `request`. Об'єкт `body` містить значення текстових рядків форми, об'єкт `file` (`files`) містить файл чи файли, завантажені через форму.

Простий приклад використання:

Не забувайте про `enctype="multipart/form-data"` в вашій формі.

```html
<form action="/profile" method="post" enctype="multipart/form-data">
  <input type="file" name="avatar" />
</form>
```

```javascript
const express = require('express')
const multer  = require('multer')
const upload = multer({ dest: 'uploads/' })

const app = express()

app.post('/profile', upload.single('avatar'), function (req, res, next) {
  // req.file - файл `avatar`
  // req.body збереже текстові поля, якщо вони будуть
})

app.post('/photos/upload', upload.array('photos', 12), function (req, res, next) {
  // req.files - масив файлів `photos`
  // req.body збереже текстові поля, якщо вони будуть
})

const uploadMiddleware = upload.fields([{ name: 'avatar', maxCount: 1 }, { name: 'gallery', maxCount: 8 }])
app.post('/cool-profile', uploadMiddleware, function (req, res, next) {
  // req.files - объект (String -> Array), де fieldname - ключ, і значення - масив файлів
  //
  // наприклад:
  //  req.files['avatar'][0] -> File
  //  req.files['gallery'] -> Array
  //
  // req.body збереже текстові рядки, якщо вони будуть
})
```

Якщо вам потрібно обробити multipart-форму, яка містить тільки текст, використовуйте метод `.none()`:

```javascript
const express = require('express')
const app = express()
const multer  = require('multer')
const upload = multer()

app.post('/profile', upload.none(), function (req, res, next) {
  // req.body містить текстові поля
})
```

## API

### Інформація про файли

Кожен файл містить наступну інформацію :

Ключ | Опис | Примітки
--- | --- | ---
`fieldname` | Імя рядка, задане в формі |
`originalname` | Імя фала на комп'ютері користувача |
`encoding` | Кодування файла |
`mimetype` | Mime-тип файла |
`size` | Розмір файла в байтах |
`destination` | Каталог, де буде збережений файл | `DiskStorage`
`filename` | Імя файла без `destination` | `DiskStorage`
`path` | Повний шлях до завантажуємого файла | `DiskStorage`
`buffer` | `Buffer` з всього файла | `MemoryStorage`

### `multer(opts)`

Multer приймає обєкт з опціями. Базова опція `dest` вказує Multer, куди завантажувати файли. Якщо ви не вказуєте обєкт з опціями, файли будуть знаходитися в памяті і не будуть збережені на диск.

По замовченню, Multer перейменовує файли, щоб запобігти конфліктів. Це налаштовується під ваші потреби.

Наступні опції можуть бути передані Multer.

Ключ | Опис
--- | ---
`dest` чи `storage` | Де зберігати файли
`fileFilter` | Функція для контролю отримання файлів
`limits` | Ліміти по завантаженню файлів
`preservePath` | Зберегти повний шлях до файлів замість тільки базового імені

Зазвичай для веб-програми потрібно обов'язково перевизначити `dest`, як показано у прикладі нижче.

```javascript
const upload = multer({ dest: 'uploads/' })
```
Якщо вам потрібно більше можливостей для керування програмою, можна використовувати `storage` замість `dest`. Multer поставляється з двома двигунами роботи з пам'яттю, `DiskStorage` та `MemoryStorage`, інші двигуни можна знайти у сторонніх розробників.

#### `.single(fieldname)`

Приймає один файл з ім'ям `fieldname`. Файл буде збережено в `req.file`.

#### `.array(fieldname[, maxCount])`

Приймає масив файлів з ім'ям `fieldname`. Опціонально можна задати помилку при спробі завантаження більш `maxCount` файлів. Масив файлів буде збережено в `req.files`.

#### `.fields(fields)`

Приймає набір файлів, визначених у `fields`. Об'єкт з масивом файлів буде збережено в `req.files`.

`fields` має бути масивом об'єктів з полями `name` та опціональним `maxCount`.
Наприклад:

```javascript
[
  { name: 'avatar', maxCount: 1 },
  { name: 'gallery', maxCount: 8 }
]
```

#### `.none()`

Приймає лише текстові поля форми. При спробі завантаження файлу падає з помилкою "LIMIT\_UNEXPECTED\_FILE".

#### `.any()`

Приймає всі передані файли. Масив файлів буде збережено в `req.files`.

**ПОПЕРЕДЖЕННЯ:** Переконайтеся, що ваше програмне забезпечення завантаження файлів коректне. Ніколи не використовуйте Multer як middleware глобально, якщо користувач може завантажити шкідливі файли, і тим самим порушити роботу вашої програми. Використовуйте цей метод тільки якщо ви повністю керуєте процесом завантаження файлів.

### `storage`

#### `DiskStorage`

Двигун дискового простору. Дає повний контроль за розміщенням файлів на диск.

```javascript
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, '/tmp/my-uploads')
  },
  filename: function (req, file, cb) {
    cb(null, file.fieldname + '-' + Date.now())
  }
})

const upload = multer({ storage: storage })
```

Доступно дві опції, розташування `destination` та ім'я файлу `filename`. Обидві ці функції визначають, де буде файл після завантаження.

`destination` используется, чтобы задать каталог, в котором будут размещены файлы. Может быть задан строкой (например, `'/tmp/uploads'`). Если не задано расположение `destination`, операционная система воспользуется для сохранения каталогом для временных файлов.

**Важно:** Вы должны создать каталог, когда используете `destination`. При передачи в качестве аргумента строки, Multer проверяет, что каталог создан.

`filename` используется, чтобы определить, как будет назван файл внутри каталога. Если
имя файла `filename` не задано, каждому файлу будет сконфигурировано случайное имя без расширения файла.

**Важно:** Multer не добавляет никакого файлового расширения, ваша функция должна возвращать имя файла с необходимым расширением.

В аргументах каждой функции прокидывается запрос (`req`) и набор информации о файле (`file`).

Обратите внимание, что `req.body` может быть не полностью заполнено. Это зависит от порядка отправки клиентом полей и файлов на сервер.

#### `MemoryStorage`

Движок оперативной памяти сохраняет файлы в памяти как объекты типа `Buffer`. В этом случае нет никаких дополнительных опций.

```javascript
const storage = multer.memoryStorage()
const upload = multer({ storage: storage })
```
Когда вы используете этот тип передачи, информация о файле будет содержать поле `buffer`, которое содержит весь файл.

**ПРЕДУПРЕЖДЕНИЕ**: Загрузка очень больших файлов, или относительно небольших файлов в большом количестве может вызвать переполнение памяти.

### `limits`

Объект, устанавливающий ограничения. Multer прокидывает этот объект напрямую в busboy, поэтому детали можно посмотреть
[на странице с методами busboy](https://github.com/mscdex/busboy#busboy-methods).

Доступны следующие целочисленные значения:

Ключ | Описание | Значение по умолчанию
--- | --- | ---
`fieldNameSize` | Максимальный размер имени файла | 100 bytes
`fieldSize` | Максимальный размер значения поля | 1MB
`fields` | Максимальное количество не-файловых полей | Не ограничено
`fileSize` | Максимальный размер файла в байтах для multipart-форм | Не ограничен
`files` | Максимальное количество полей с файлами для multipart-форм | Не ограничено
`parts` | Максимальное количество полей с файлами для multipart-форм (поля плюс файлы) | Не ограничено
`headerPairs` | Максимальное количество пар ключ-значение key=>value для multipart-форм, которое обрабатывается | 2000

Установка ограничений может помочь защитить ваш сайт от DoS-атак.

### `fileFilter`

Задают функцию для того, чтобы решать, какие файлы будут загружены, а какие — нет. Функция может выглядеть так:

```javascript
function fileFilter (req, file, cb) {

  // Функция должна вызывать `cb` с булевым значением,
  // которое показывает, следует ли принять файл

  // Чтобы отклонить, прокиньте в аргументы `false` так:
  cb(null, false)

  // Чтобы принять файл, используется как аргумент `true` таким образом:
  cb(null, true)

  // Вы можете всегда вернуть ошибку, если что-то пошло не так:
  cb(new Error('I don\'t have a clue!'))

}
```

## Обработка ошибок

Когда выбрасывается исключение, Multer делегирует его обработку Express. Вы можете выводить страницу ошибки [стандартными для express способами](http://expressjs.com/guide/error-handling.html).

Если вы хотите отлавливать ошибки конкретно от Multer, вам нужно вызывать собственную middleware для их обработки. Еще, если вы хотите отлавливать [исключительно ошибки Multer](https://github.com/expressjs/multer/blob/main/lib/make-error.js#L1-L9), вы можете использовать класс `MulterError`, который привязан к объекту `multer` (например, `err instanceof multer.MulterError`)

```javascript
const multer = require('multer')
const upload = multer().single('avatar')

app.post('/profile', function (req, res) {
  upload(req, res, function (err) {
    if (err instanceof multer.MulterError) {
      // Случилась ошибка Multer при загрузке.
    } else {
      // При загрузке произошла неизвестная ошибка.
    }

    // Все прекрасно загрузилось.
  })
})
```

## Собственные движки для сохранения файлов

Чтобы получить информацию, как создать собственный движок для обработки загрузки файлов, смотрите страницу [Multer Storage Engine](https://github.com/expressjs/multer/blob/main/StorageEngine.md).

## Лицензия

[MIT](LICENSE)

[ci-image]: https://github.com/expressjs/multer/actions/workflows/ci.yml/badge.svg
[ci-url]: https://github.com/expressjs/multer/actions/workflows/ci.yml
[test-url]: https://coveralls.io/r/expressjs/multer?branch=main
[test-image]: https://badgen.net/coveralls/c/github/expressjs/multer/main
[npm-downloads-image]: https://badgen.net/npm/dm/multer
[npm-url]: https://npmjs.org/package/multer
[npm-version-image]: https://badgen.net/npm/v/multer
[ossf-scorecard-badge]: https://api.scorecard.dev/projects/github.com/expressjs/multer/badge
[ossf-scorecard-visualizer]: https://ossf.github.io/scorecard-visualizer/#/projects/github.com/expressjs/multer
