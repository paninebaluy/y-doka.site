---
title: async/await
name: async-await
author: N_Lopin
co-authors:
designers:
contributors:
summary:
  - асинхронный вызов
  - api
---

Эта документация связана с понятием асинхронности в JavaScript. Зачем нужен асинхронный код и как он работает в деталях описано в обзорной статье [Асинхронность в JS](/posts/js/long/async-in-js) .

## Кратко

Добавленное перед определением функции ключевое слово `async` делает функцию асинхронной. Возвращаемое значение такой функции автоматически оборачивается в Promise:

```jsx
async function getStarWarsMovies() {
  return 1
}

console.log(getStarWarsMovies()) // Promise { <state>: "fulfilled", <value>: 1 }
```

Асинхронные функции нужны для выполнения асинхронных операций: работой с API, базами данных, чтения файлов и т.д.

Асинхронные операции выполняются не сразу: код отправил запрос к API и ждет, пока сервер пришлет ответ. Ключевое слово `await` используется, чтобы дождаться выполнения асинхронной операции:

```jsx
async function getStarWarsMovie(id) {
  const response = await fetch(`https://swapi.dev/api/films/${id}/`)
  console.log("ответ получен", response) // *1
  return response.json()
}

const movies = getStarWarsMovie(1).then((movie) => {
  console.log(movie.title)
}) // *2
console.log("результат вызова функции", movies) // *3
```

Движок JavaScript при этом не блокируется и может выполнять другой код. Как только ответ получен, выполнение кода продолжается.

Вывод на экран будет следующий:

```jsx
"результат вызова функции" Promise // вызвали функцию, она начала выполнять асинхронную операцию и вернула Promise (*3)
"ответ получен" Response // получили ответ API, продолжаем выполнение функции (*1)
"A New Hope" // сработал callback (*2)
```

## Как понять

Ключевые слова `async/await` не привносят в JavaScript что-то новое. Они только упрощают работу с Promise.

Вместо кода с цепочкой вызовов:

```jsx
function getMainActorProfileFromMovie(id) {
  return fetch(`https://swapi.dev/api/films/${id}/`)
    .then((movieResponse) => {
      return movieResponse.json()
    })
    .then((movie) => {
      const characterUrl = movie.characters[0].split("//")[1]
      return fetch(`https://${characterUrl}`)
    })
    .then((charactrerResponse) => {
      return charactrerResponse.json()
    })
    .catch((err) => {
      console.error("Произошла ошибка!", err)
    })
}

getMainActorProfileFromMovie(1).then((profile) => {
  console.log(profile)
})
```

Можно записать с `async/await`:

```jsx
function getMainActorProfileFromMovie(id) {
  try {
    const movieResponse = await fetch(`https://swapi.dev/api/films/${id}/`);
    const movie = await movieResponse.json();

    const characterUrl = movie.characters[0].split('//')[1];
    const charactrerResponse = await fetch(`https://${characterUrl}`);
    return charactrerResponse.json();
  } catch(err) {
    console.error('Произошла ошибка!', err);
  }
}

getMainActorProfileFromMovie(1).then((profile) => {console.log(profile)});
```

Такой код проще понимать:

- он плоский
- выглядит, как синхронный
- использует стандартный try...catch блок для обработки ошибок

☝️ Ключевое слово `await` работает только внутри асинхронных функций. Вызвов его вне функции будет синтаксической ошибкой:

```jsx
function getMainActorProfileFromMovie(id) {
  // код примера выше
}

await getMainActorProfileFromMovie(1)
```

Код упадет с ошибкой `SyntaxError: await is only valid in async functions and async generators`.

## В работе

🛠 Всегда используйте `async/await` вместо цепочек `then` и колбэков.

Этот подход проще читается, легче отлаживается, пользуется стандартными способами обработки ошибок.

🛠 `await` нельзя использовать вне асинхронных функций. Если нужно выполнить асинхронную операцию в глобальной области видимости, придется воспользоваться `then`.
