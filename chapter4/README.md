# ВАШ ПЕРВЫЙ СЕРВЕР НА NODE.JS 

В этой главе я расскажу вам о том, как вы можете запустить простой HTTP-сервер на Node.js и начать обрабатывать запросы.

## Модуль `http` для вашего Node.js-сервера

Когда вы начинаете создавать HTTP-приложения в Node.js,
встоенные модули `http/https` это то, с чем вы будете взаимодействовать.

Теперь давайте создадим ваш первый HTTP-сервер на Node.js! Нам нужно будет подключить модуль `http` и привязать наш сервер к порту `3000`.

```javascript
// содежимое index.js
const http = require(‘http’)
const port = 3000

const requestHandler = (request, response) => {
    console.log(request.url)
    response.end(‘Hello Node.js Server!’)
}

const server = http.createServer(requestHandler)

server.listen(port, (err) => {
    if (err) {
        return console.log(‘something bad happened’, err)
    }

    console.log(`server is listening on ${port}`)
})
```

Вы запустить этот скрипт:

```javascript
$ node index.js
```

Что следует отметить здесь:
* `requestHandler`: **эта функция будет вызываться каждый раз, когда
на сервер приходит запрос**. Если вы откроете в своём браузере адрес `localhost:3000`, два сообщения появятся в консоли: одно для `/` и одно для `favicon.ico`
* `if (err)`: обработка ошибок — если порт уже занят, или есть какие-то другие причины, по которым сервер не может быть запущен, мы получим уведомление в этом месте.

Модуль `http` очень низкоуровневый — создание сложного веб-приложения с использованием вышеприведенного фрагмента кода очень трудоемко. Именно по этой причине мы обычно выбираем фреймворки для работы с нашими проектами. Есть множество фреймворков, вот самые популярные:
* [express](http://expressjs.com/)
* [hapi](https://hapijs.com/)
* [koa](http://koajs.com/)
* [restify](http://restify.com/)

*В этой и следующих главах мы будем использовать Express, так как вы можете найти множество модулей для Express в NPM.*

## Express

> Быстрый, гибкий, минималистичный веб-фреймворк для Node.js — [http://expressjs.com/](http://expressjs.com/)

Добавление Express в ваш проект - это просто установка через NPM:

```javascript
$ npm install express --save
```

После того, как вы установили Express, давайте посмотрим, как создать
приложение аналогичное тому, что мы написали ранее:

```javascript
const express = require(‘express’)
const app = express()
const port = 3000

app.get(‘/’, (request, response) => {
    response.send(‘Hello from Express!’)
})

app.listen(port, (err) => {
    if (err) {
        return console.log(‘something bad happened’, err)
    }

    console.log(`server is listening on ${port}`)
})
```

Самое большое различие которое вы можете здесь заметить: Express по умолчанию дает вам роутер. Вам не нужно вручную разбирать URL, чтобы решить, что делать, вместо этого вы определяете маршрутизацию приложения с помощью `app.get`, `app.post`, `app.put` и т.д. Они транслируются в соответствующие HTTP-запросы.

Одна из самых мощных концепций, которую реализует Express — это паттерн Middleware.

## Middleware — промежуточный обработчик

Вы можете думать о промежуточных обработчиках как о конвейерах Unix, но для HTTP-запросов.

![](middlewares.png)

На диаграмме вы можете увидеть, как запрос идёт через условное Express-приложение. Он проходит через три промежуточных обработчика. Каждый обработчик может изменить этот запрос, а затем на основе бизнес-логики третий middleware может отправить ответ, либо запрос попадёт в обработчик соответствующего роута.

На практике вы можете сделать это следующим образом:

```javascript
const express = require(‘express’)
const app = express()

app.use((request, response, next) => {
    console.log(request.headers)
    next()
})

app.use((request, response, next) => {
    request.chance = Math.random()
    next()
})

app.get(‘/’, (request, response) => {
    response.json({
        chance: request.chance
    })
})

app.listen(3000)
```

Что следует здесь отметить:

* `app.use`: это то, как вы можете описать middleware — этот метод принимает функцию с тремя параметрами, первый из которых является запросом, второй — ответом, а третий — коллбеком `next`. Вызов `next` сигнализирует Express о том, что он может переходить к следующему промежуточному обработчику.
* Первый промежуточный обработчик только логирует заголовки и мгновенно вызывает следующий.
* Второй добавляет дополнительное свойство к запросу - **это одна из самых мощных функций шаблона  middleware**. Ваши промежуточные обработчики могут добавлять дополнительные данные к объекту запроса, который могут считывать/изменять middleware расположенные ниже.

## Обработка ошибок

Как и во всех фреймворках, получение правильной обработки ошибок имеет решающее значение. В Express вы должны создать специальный промежуточный обработчик - middleware с четырьмя входными параметрами:

```javascript
const express = require(‘express’)
const app = express()

app.get(‘/’, (request, response) => {
    throw new Error(‘oops’)
})

app.use((err, request, response, next) => {
    // логирование ошибки, пока просто console.log
    console.log(err)
    response.status(500).send(‘Something broke!’)
})
```

Что следует здесь отметить:

* Обработчик ошибок должен быть последней функцией, добавленной с помощью `app.use`.
* Обработчик ошибок принимает коллбек `next` — он может использоваться для объединения нескольких обработчиков ошибок.

## Рендеринг HTML

Ранее мы рассмотрели, как отправлять JSON-ответы — пришло время узнать, как отрендерить HTML простым способом. Для этого мы собираемся использовать пакет [handlebars](http://handlebarsjs.com/) с обёрткой [express-handlebars](https://www.npmjs.com/package/express-handlebars).

Сначала создадим следующую структуру каталогов:

```
├── index.js
└── views
    ├── home.hbs
    └── layouts
        └── main.hbs
```

После этого заполните index.js следующим кодом:

```javascript
// index.js
const path = require(‘path’)
const express = require(‘express’)
const exphbs = require(‘express-handlebars’)

app.engine(‘.hbs’, exphbs({
    defaultLayout: ‘main’,
    extname: ‘.hbs’,
    layoutsDir: path.join(__dirname, ‘views/layouts’)
}))
app.set(‘view engine’, ‘.hbs’)
app.set(‘views’, path.join(__dirname, ‘views’)) 
```

Приведенный выше код инициализирует движок handlebars и устанавливает каталог шаблонов в `views/layouts`. Это каталог, в котором будут храниться ваши шаблоны.

После того, как вы сделали эту настройку, вы можете поместить свой начальный `html` в `main.hbs` — чтобы все было проще, давайте сразу перейдем к этому:

```html
<html>
    <head>
        <title>Express handlebars</title>
    </head>
    <body>
        {{{body}}}
    </body>
</html>
```

Вы можете заметить метку `{{{body}}}` — здесь будет размещен ваш контент. Давайте создадим `home.hbs`!

```html
<h2>Hello {{name}}<h2>
```

Последнее, что мы должны сделать, чтобы заставить всё это работать, - добавить обработчик маршрута в наше приложение Express:

```javascript
app.get(‘/’, (request, response) => {
    response.render(‘home’, {
        name: ‘John’
    })
})
```

Метод `render` принимает два параметра:
* Первый — это имя шаблона
* Второй — данные, которые необходимы для рендеринга.

Как только вы сделаете запрос по этому адресу, вы получите что-то вроде этого:

```html
<html>
    <head>
        <title>Express handlebars</title>
    </head>
    <body>
        <h2>Hello John</h2>
    </body>
</html>
```

Это всего лишь верхушка айсберга — чтобы узнать, как добавить больше шаблонов (и даже частичных), обратитесь к официальной документации (express-handlebars)[https://www.npmjs.com/package/express-handlebars].

## Отладка Express

В некоторых случаях вам может потребоваться выяснить, что происходит с Express, когда приложение работает. Для этого вы можете передать следующую переменную окружения в Express: `DEBUG=express*`.

Вы должны запустить свой Node.js HTTP-сервер, используя:

```
$ DEBUG=express* node index.js
```

## Резюме

Вот как вы можете настроить свой первый HTTP-сервер на Node.js с нуля. Я рекомендую Express для начала, а затем поэкспериментируйте.

---

В следующей главе вы узнаете, **как получать информацию из баз данных**.
