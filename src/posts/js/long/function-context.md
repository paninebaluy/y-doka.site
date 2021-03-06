---
title: Контекст выполнения функций, this
name: function-context
author: bespoyasov
co-authors:
designers:
contributors:
summary:
  - use strict
  - строгий режим
  - bind
  - call
  - apply
  - constructor конструктор
  - стрелочная функция
  - arrow function
  - new
---

## Кратко

Грубо говоря, `this` — это ссылка на некий объект, к свойствам которого можно достучаться внутри вызова функции. Этот `this` — и есть _контекст выполнения_.

Но чтобы лучше понять, что такое `this` и контекст выполнения в JavaScript, нам потребуется зайти издалека.

## Как понять

Сперва вспомним, как мы в принципе можем выполнить какую-то инструкцию в коде.

Выполнить что-то в JS можно 4 способами:

- Вызвав функцию;
- Вызвав метод объекта;
- Использовав функцию-конструктор;
- Непрямым вызовом функции.

### Функция

Первый и самый просто способ выполнить что-то — вызвать функцию.

```jsx
function hello(whom) {
  console.log(`Hello, ${whom}!`)
}

hello("World") // Hello, World!

// Чтобы выполнить функцию, мы используем выражение `hello`
// и скобки с аргументами.
```

Когда мы вызываем _функцию_, значением `this` может быть лишь _глобальный объект_ или `undefined` при использовании `'use strict'`.

#### Глобальный объект

_Глобальный объект_ — это, так скажем, корневой объект в программе.

Если мы запускаем JS-код в браузере, то глобальным объектом будет `window`. Если мы запускаем код в Node-окружении, то `global`.

#### Строгий режим

Можно сказать, что _строгий режим_ — неказистый [способ борьбы с легаси](https://learn.javascript.ru/strict-mode).

Включается строгий режим с помощью директивы `'use strict'` в начале блока, который должен выполняться в строгом режиме:

```jsx
function nonStrict() {
  // Будет выполняться в нестрогом режиме.
}

function strict() {
  "use strict"
  // Будет выполняться в строгом режиме.
}

// Также можно настроить строгий режим для всего файла,
// если указать 'use strict' в начале.
```

#### Значение `this`

Вернёмся к `this`. В нестрогом режиме при выполнении в браузере `this` при вызове функции будет равен `window`:

```jsx
function whatsThis() {
  console.log(this === window)
}

whatsThis() // true
```

То же — если функция объявлена внутри функции:

```jsx
function whatsThis() {
  function whatInside() {
    console.log(this === window)
  }

  whatInside()
}

whatsThis() // true
```

И то же — если функция будет анонимной и, например, вызвана немедленно:

```jsx
;(function () {
  console.log(this === window)
})() // true
```

В строгом режиме — значение будет равно `undefined`:

```jsx
"use strict"

function whatsThis() {
  console.log(this === undefined)
}

whatsThis() // true
```

### Метод объекта

Если функция хранится в объекте — это _метод этого объекта_.

```jsx
const user = {
  name: "Alex",
  greet() {
    console.log("Hello, my name is Alex")
  },
}

user.greet() // Hello, my name is Alex
// user.greet — это метод объекта user.
```

В этом случае значение `this` — этот объект.

```jsx
const user = {
  name: "Alex",
  greet() {
    console.log(`Hello, my name is ${this.name}`)
  },
}

user.greet() // Hello, my name is Alex
```

Обратите внимание, что если записать функцию в отдельную переменную, `this` переопределится.

```jsx
const user = {
  name: "Alex",
  greet() {
    console.log(`Hello, my name is ${this.name}`)
  },
}

const greet = user.greet
greet() // Hello, my name is undefined

// При вызове через точку user.greet
// значение this равняется объекту до точки (user).
// Без этого объекта this === undefined.
```

Чтобы такого не происходило, [следует использовать `.bind()`](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Function/bind), о котором мы поговорим чуть позже.

## Вызов конструктора

_Конструктор_ — это функция, которую мы используем, чтобы создавать однотипные объекты. Такие функции похожи на печатный станок, который создаёт детали LEGO. однотипные объекты — детальки, а конструктор — станок. Он как бы конструирует эти объекты, отсюда название.

По соглашениям конструкторы вызывают с помощью ключевого слова `new`, а также называют с большой буквы, причём обычно не глаголом, а существительным. Существительное — это та сущность, которую создаёт конструктор.

Например, если конструктор будет создавать объекты пользователей, мы можем назвать его `User`, а использовать вот так:

```jsx
function User() {
  this.name = "Alex"
}

const firstUser = new User()
firstUser.name === "Alex" // true
```

При вызове _конструктора_ `this` равен свежесозданному объекту.

В примере с `User` значением `this` будет объект, который конструктор создаёт:

```jsx
function User() {
  console.log(this instanceof User) // true
  this.name = "Alex"
}

const firstUser = new User()
firstUser instanceof User // true
```

На самом деле, многое происходит «за кулисами»:

- При вызове сперва создаётся новый пустой объект, и он присваивается `this`.
- Выполняется код функции. (Обычно он модифицирует `this`, добавляет туда новые свойства.)
- Возвращается значение `this`.

Если расписать все неявные шаги, то:

```jsx
function User() {
  // Происходит неявно:
  // this = {};

  this.name = "Alex"

  // Происходит неявно:
  // return this;
}
```

То же происходит и в ES6-классах, узнать о них больше можно в [статье про объектно-ориентированное программирование](/posts/js/long/oop).

```jsx
class User {
  constructor() {
    this.name = "Alex"
  }

  greet() {
    /*...*/
  }
}

const firstUser = new User()
```

#### Как не забыть о `new`

При работе с конструкторами легко вызвать их неправильно:

```jsx
const firstUser = new User() // Правильно.
const secondUser = User() // Неправильно,
// хотя работает будто бы верно.
```

На первый взгляд разницы нет:

```jsx
firstUser.name === "Alex" // true
secondUser.name === "Alex" // true
```

Но значение `this` у второго равняется `window`, потому что новый объект не был создан из-за отсутствия `new`:

```jsx
secondUser === window // true
```

Чтобы не попадаться в такую ловушку, в конструкторе можно прописать проверку на то, что новый объект создан:

```jsx
function User() {
  if (!(this instanceof User)) {
    throw Error("Error: Incorrect invocation!")
  }

  this.name = "Alex"
}

const secondUser = User() // Error: Incorrect invocation!
```

### Непрямой вызов

_Непрямым вызовом_ называют вызов функций через `[.call()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)` или `[.apply()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)`.

Оба первым аргументом принимают `this`. То есть они позволяют настроить контекст снаружи, к тому же — явно.

```jsx
function greet() {
  console.log(`Hello, ${this.name}`)
}

const user1 = { name: "Alex" }
const user2 = { name: "Ivan" }

greet.call(user1) // Hello, Alex
greet.call(user2) // Hello, Ivan

greet.apply(user1) // Hello, Alex
greet.apply(user2) // Hello, Ivan

// В обоих случаях
// в первом вызове this === user1,
// во втором — user2.
```

Разница между `.call()` и `.apply()` — в том, как они принимают аргументы для самой функции после `this`.

```jsx
function greet(greetWord, emoticon) {
  console.log(`${greetWord} ${this.name} ${emoticon}`)
}

const user1 = { name: "Alex" }
const user2 = { name: "Ivan" }

// .call() принимает аргументы списком через запятую:
greet.call(user1, "Hello,", ":-)") // Hello, Alex :-)
greet.call(user2, "Good morning,", ":-D") // Good morning, Ivan :-D

// .apply() же — принимает массив аргументов:
greet.apply(user1, ["Hello,", ":-)"]) // Hello, Alex :-)
greet.apply(user2, ["Good morning,", ":-D"]) // Good morning, Ivan :-D

// В остальном они идентичны.
```

#### Связывание функций

Особняком стоит `.bind()`. Это метод, который позволяет связывать контекст выполнения с функцией, чтобы «заранее и точно» определить, какое именно значение будет у `this`.

```jsx
function greet() {
  console.log(`Hello, ${this.name}`)
}

const user1 = { name: "Alex" }

const greetAlex = greet.bind(user1)
greetAlex() // Hello, Alex
```

Обратите внимание, что `.bind()` в отличие от `.call()` и `.apply()` не вызывает функцию сразу. Вместо этого он возвращает другую функцию — связанную с указанным контекстом.

#### Стрелочные функции

У _стрелочных функций_ собственного контекста выполнения нет. Они связываются с ближайшим по иерархии контекстом, в котором они определены.

Это удобно, когда нам нужно передать в стрелочную функцию, например, родительский контекст без использования `.bind()`.

```jsx
function greetWaitAndAgain() {
  console.log(`Hello, ${this.name}!`)
  setTimeout(() => {
    console.log(`Hello again, ${this.name}!`)
  })
}

const user = { name: "Alex" }

greetWaitAndAgain.call(user)

// Hello, Alex!
// Hello again, Alex!
```

При использовании обычной функции внутри, контекст бы потерялся, и чтобы добиться того же результата, нам бы пришлось использовать `.call()`, `.apply()` или `.bind()`.

## В работе

Гибкий, нефиксированный контекст в JS — это одновременно и удобно и опасно.

Удобно это тем, что мы можем писать очень абстрактные функции, которые будут использовать контекст выполнения, для своей работы. Так мы можем добавиться [полиморфизма](/posts/js/long/oop#полиморфизм).

Однако в то же время гибкий `this` может быть и причиной ошибки, например, если мы используем конструктор без `new`, или просто спутаем контекст выполнения.

### 🛠 Всегда используйте `'use strict'`

Это относится даже скорее не конкретно к контексту, а в целом рекомендация для написания 🙂

Однако и с контекстом строгий режим позволит раньше обнаружить закравшуюся ошибку. Например:

```jsx
// В нестрогом режиме, если мы забудем new,
// name станет полем на глобальном объекте.

function User() {
  this.name = "Alex"
}

const user = User()
// window.name === 'Alex';
// user === window

// В строгом мы получим ошибку,
// потому что изначально контекст внутри функции
// в строгом режиме — undefined.

function User() {
  "use strict"
  this.name = "Alex"
}

const user = User()
// Uncaught TypeError: Cannot set property 'name' of undefined.
```

### 🛠 Всегда используйте `new` и ставьте проверки в конструкторе

При использовании конструкторов всегда используйте `new`. Это обезопасит вас от ошибок, и не будет вводить в заблуждение разработчиков, которые будут читать код после.

А для защиты «от дурака» желательно ставить проверки внутри конструктора:

```jsx
function User() {
  if (!(this instanceof User)) {
    throw Error("Error: Incorrect invocation!")
  }

  this.name = "Alex"
}

const secondUser = User() // Error: Incorrect invocation!
```

### 🛠 Авто-байнд для методов класса

В ES6 появились классы, но они не работают в старых браузерах. Обычно разработчики [транспилируют](https://ru.wikipedia.org/wiki/Транспайлер) код — то есть переводят его с помощью разных инструментов в ES5.

Может случиться так, что при транспиляции, если она настроена неправильно, методы класса не будут распознавать `this`, как экземпляр класса.

```jsx
class User {
  name: "Alex"
  greet() {
    console.log(`Hello ${this.name}`)
  }
}

// this.name может быть undefined;
// this может быть undefined.
```

Чтобы от этого защититься, можно использовать стрелочные функции, чтобы создать [поля классов](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Classes/Class_fields).
