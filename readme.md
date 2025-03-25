0. Установи Node и npm (Node Package manager). Они нужны для запуска приложения и установки пакетов (в том числе
   фреймворка express).
   Можно сделать это с официального сайта: https://nodejs.org/. При установке Node.js установится и npm.

1. Перейди в репозиторий с приложением, затем в папку `express-app`, и выполни команду

```bash
npm install
```

Она установит необходимые зависимости.

Затем запусти приложение. Способ запуска зависит от твоих ОС и терминала:

- MacOS/Linux:

```bash
DEBUG=myapp:* npm run dev
```

- Windows Command Prompt:

```cmd
set DEBUG=myapp:* & npm run dev
```

- Windows PowerShell:

```powershell
$env:DEBUG='myapp:*'; npm run dev
```

После этого открой http://localhost:3000/ и посмотри приветствие от Express. Обрати внимание: в консоли логируются запросы к твоему приложению.

2. Посмотри на структуру app.js. Давай разберём, что там происходит.

- Сначала мы подключаем необходимые библиотеки. Делается это с помощью директивы require(), которая присуща Node.js.
- Затем мы создаём объект приложения

```js
var app = express();
```

- Затем мы подключаем разные middleware и обработку запросов (`app.use`).
- Затем экспортируем настроенный объект приложения

```js
module.exports = app;
```

Главный интерес для нас представляем блок с подключением обработки запросов. С ним мы и будем в дальнейшем работать.

3. Давай допишем обработку запроса `/users`. Она подключается строчкой

```js
app.use("/users", usersRouter);
```

Выполни запрос http://localhost:3000/users в браузере. Посмотри на ответ.

Открой файл `routes/users.js`. В нём есть функция, которая принимает объект запроса `req` (от "request"), объект ответа
`res` (от "response") и коллбэк `next`,
который позволяет не обрабатывать запрос прямо тут же, а передать управление дальше. Такой паттерн называется
`Chain of responsibility`, а функция в таком случае
называется "middleware".

Сейчас фукнция возвращает строку, давай сделаем, чтобы она возвращала объект, внутри которого будет лежат массив
пользователей.
Пусть в этом объекте будет поле items с массиво объектов вида

```json
{
  "id": 1,
  "name": "name"
}
```

Данные придумай самостоятельно.

Приложение перезапускать при этом не нужно - помогает пакет nodemon.

Выполни запрос за пользователями заново, посмотри на ответ. Должен быть твой объект в виде json.

4. Давай научимся отправлять запросы в бэкенд не из браузера. Для этого воспользуемся http-файлами (файлы с раширением
   `.http`).

- В WebStorm клиент для них уже встроен (Tools -> HTTP Client -> Create request in HTTP client).
- Для Visual Studio Code есть расширение "REST Client", которое позволит писать и отправлять запросы точно так же, как в
  WebStorm.
  Для его установки открой меню `Extensions` (в левой панели это иконка с кубиками, затем в поисковике найдт расширение
  `REST Client`.

Открой HTTP-клиент, создай файл запроса (с расширением `.http`) и напиши следующий запрос:

```http
GET http://localhost:3000/users
Accept: application/json
```

Проверь, что в ответе будет тот же JSON, что в браузере.

5. Давай добавим метод создания пользователя: POST /users. Сделай сохранение пользователя в какой-нибудь объект,
   принимай его в виде JSON.
   Для того, чтобы обратиться к объекту внутри тела, используй `req.body` - там лежит уже распарсенный json (благодаря
   тому, что парсинг подключен строчкой
   `app.use(express.json());`).

6. Теперь сделаем псевдо-аутентификацию. Для того, чтобы обращаться к методам бэкенда, будем принимать в query string
   параметр `auth`
   со значением `true`. Чтобы не проверять аутентификацию для каждого обработчика (он же "роут") отдельно, напишем
   middleware,
   которая будет делать это универсально.
   Для этого напиши обработчик в `app.js` вида

```js
app.use((req, res, next) => {...
});
```

В обработчике возьми свойство `query` у `req` и возьми из него нужный параметр (можно через стандартный индексатор
объекта).
Подключи его до обработчиков запросов, в которых есть указание пути. Обрати внимание: middleware подключаются в порядке
использования.

Таким же образом можно проверять аутентификацию другими способами (по cookie, header'у и т.д.)

7. Мы сделали сохранение пользователей в оперативную память, но при перезапуске приложения все данные будут теряться.
   Подключим базу данных SQLite.
   Останови приложение и выполни в папке с ним команду

```bash
npm install sqlite3
```

Затем добавь код создания таблицы в файл с обработкой запросов с пользователями:

```js
const sqlite3 = require('sqlite3').verbose()
const db = new sqlite3.Database('mydb.db');
db.run(`CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name text)`);
```

Обрати внимание - id теперь будет присваиваться в базе, в запросе можно его не передавать.
После этого в обработчик создания пользователя добавь сохранение в БД:

```js
const insert = "INSERT INTO users (name) VALUES (?)";
db.run(insert, [name]);
```

В методе получения пользователей сделай получение их из БД с помощью

```js
db.all("SELECT id, name FROM users", [], (err, rows) => {
    if (err) {
        console.log(err);
    } else {
        res.send(rows);
    }
});
```

Обрати внимание, обработки ошибки всегда в коллбэке, передаваемом третьим параметром.
