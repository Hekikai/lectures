В классе `Promise` есть 5 статических методов. Давайте познакомимся с ними.

**Promise.all**

Допустим, нам нужно запустить множество промисов параллельно и дождаться, пока все они выполнятся.

Например, параллельно загрузить несколько файлов и обработать результат, когда он готов.

Для этого как раз и пригодится `Promise.all`.

Синтаксис:
```js
let promise = Promise.all([...promises...]);
```

Метод `Promise.all` принимает массив промисов (может принимать любой перебираемый объект, но обычно используется массив) и возвращает новый промис.

Новый промис завершится, когда завершится весь переданный список промисов, и его результатом будет массив их результатов.

Например, `Promise.all`, представленный ниже, выполнится спустя 3 секунды, его результатом будет массив `[1, 2, 3]`:
```js
Promise.all([
	new Promise(resolve => setTimeout(() => resolve(1),3000)),
	new Promise(resolve => setTimeout(() => resolve(2),2000)),
	new Promise(resolve => setTimeout(() => resolve(3),1000))
]).then(console.log); // когда все промисы выполнятся, 
// результат будет 1, 2, 3
// каждый промис даёт элемент массива
```

Обратите внимание, что порядок элементов массива в точности соответствует порядку исходных промисов. Даже если первый промис будет выполняться дольше всех, его результат всё равно будет первым в массиве.

Часто применяемый трюк – пропустить массив данных через map-функцию, которая для каждого элемента создаст задачу-промис, и затем обернёт получившийся массив в `Promise.all`.

Например, если у нас есть массив ссылок, то мы можем загрузить их вот так:
```js
let urls = [
	'https://api.github.com/users/iliakan',
	'https://api.github.com/users/remy',
	'https://api.github.com/users/jeresig'
];

// Преобразуем каждый URL в промис, возвращённый fetch
let requests = urls.map(url => fetch(url));

// Promise.all будет ожидать выполнения всех промисов
Promise.all(requests)
	.then(responses => responses.forEach(
		response => {
	console.log(`${response.url}: ${response.status}`)
}
	));
```

А вот пример побольше, с получением информации о пользователях GitHub по их логинам из массива (мы могли бы получать массив товаров по их идентификаторам, логика та же):
```js
let names = ['iliakan', 'remy', 'jeresig'];

let requests = names.map(name => fetch(`https://api.github.com/users/${name}`));

Promise.all(requests)
	.then(responses => {
		// все промисы успешно завершены
		for(let response of responses) {
			console.log(`${response.url}: ${response.status}`)
		// покажет 200 для каждой ссылки
		}

		return responses;
	})
	// преобразовать массив ответов response в response.json(),
	// чтобы прочитать содержимое каждого
	.then(responses => Promise.all(responses.map(r => r.json())))
	// все JSON-ответы обработаны, users - массив с результатами
	.then(users => users.forEach(user => console.log(user.name)));
```

**Если любой из промисов завершится с ошибкой, то промис, возвращённый `Promise.all`, немедленно завершается с этой ошибкой.**

Например:
```js
Promise.all([
	new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
	new Promise((resolve, reject) => setTimeout(() => reject(new Error("Ошибка!")), 2000)),
	new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
]).catch(console.log); // Error: Ошибка!
```

Здесь второй промис завершится с ошибкой через 2 секунды. Это приведёт к немедленной ошибке в `Promise.all`, так что выполнится `.catch`: ошибка этого промиса становится ошибкой всего `Promise.all`.
****
**В случае ошибки, остальные результаты игнорируются**
Если один промис завершается с ошибкой, то весь `Promise.all` завершается с ней, полностью забывая про остальные промисы в списке. Их результаты игнорируются.

Например, если сделано несколько вызовов `fetch`, как в примере выше, и один не прошёл, то остальные будут всё ещё выполняться, но `Promise.all` за ними уже не смотрит. Скорее всего, они так или иначе завершатся, но их результаты будут проигнорированы.

`Promise.all` ничего не делает для их отмены, так как в промисах вообще нет концепции «отмены». В главе [Fetch: прерывание запроса](https://learn.javascript.ru/fetch-abort) мы рассмотрим `AbortController`, который помогает с этим, но он не является частью Promise API.
****
**`Promise.all(iterable)` разрешает передавать не-промисы в `iterable` (перебираемом объекте)**

Обычно, `Promise.all(...)` принимает перебираемый объект промисов (чаще всего массив). Но если любой из этих объектов не является промисом, он передаётся в итоговый массив «как есть».

Например, здесь результат: `[1, 2, 3]`
```js
Promise.all([
	new Promise((resolve, reject) => {
		setTimeout(() => resolve(1), 1000)
	}),
	2,
	3
]).then(console.log); // 1, 2, 3
```

Таким образом, мы можем передавать уже готовые значения, которые не являются промисами, в `Promise.all`, иногда это бывает удобно.
****
**Promise.allSettled**

Эта возможность была добавлена в язык недавно. В старых браузерах может понадобиться полифил.

`Promise.all` завершается с ошибкой, если она возникает в любом из переданных промисов. Это подходит для ситуаций «всё или ничего», когда нам нужны _все_ результаты для продолжения:
```js
Promise.all([
	fetch('/template.html'),
	fetch('/style.css'),
	fetch('/data.json')
]).then(render); // методу render нужны результаты всех fetch
```

Метод `Promise.allSettled` всегда ждёт завершения всех промисов. В массиве результатов будет

-   `{status:"fulfilled", value:результат}` для успешных завершений,
-   `{status:"rejected", reason:ошибка}` для ошибок.

Например, мы хотели бы загрузить информацию о множестве пользователей. Даже если в каком-то запросе ошибка, нас всё равно интересуют остальные.

Используем для этого `Promise.allSettled`:
```js
let urls = [ 
	'https://api.github.com/users/iliakan', 
	'https://api.github.com/users/remy',
	'https://no-such-url'
];

Promise.allSettled(urls.map(url => fetch(url)))
	.then(results => { // (*)
		results.forEach((result, num) => {
			if(result.status == "fulfulled") {
				console.log(`${urls[num]}: ${result.value.status}`)
			}
			if (result.status == "rejected") {
				console.log(`${urls[num]}: ${result.reason}`);
			}
		});
	});
```

Массив `results` в строке `(*)` будет таким:
```js
[
	{status: 'fulfilled', value: ...объект ответа...},
	{status: 'fulfilled', value: ...объект ответа...},
	{status: 'rejected', reason: ...объект ошибки...}
]
```
То есть, для каждого промиса у нас есть его статус и значение/ошибка.

**Promise.race**

Метод очень похож на `Promise.all`, но ждёт только первый выполненный промис, из которого берёт результат (или ошибку).

Синтаксис:
```js
let promie = Promise.race(iterable);
```

Например, тут результат будет `1`:
```js
Promise.race([
	new Promise((resolve, reject) => setTimeout(() => resolve(1), 1000)),
	new Promise((resolve, reject) => setTimeout(() => reject(new Error("Ошибка!")), 2000)),
	new Promise((resolve, reject) => setTimeout(() => resolve(3), 3000))
])
```

Быстрее всех выполнился первый промис, он и дал результат. После этого остальные промисы игнорируются.

**Promise.resolve/reject**

Методы `Promise.resolve` и `Promise.reject` редко используются в современном коде, так как синтаксис `async/await` (мы рассмотрим его чуть позже) делает его, в общем-то, не нужным.

Мы рассмотрим их здесь для полноты картины, а также для тех, кто по каким-то причинам не может использовать `async/await`.

-   `Promise.resolve(value)` создаёт успешно выполненный промис с результатом `value`.

То же самое, что:
```js
let promise = new Promise(resolve => resolve(value));
```

Этот метод используют для совместимости: когда ожидается, что функция возвратит именно промис.

Например, функция `loadCached` ниже загружает URL и запоминает (кеширует) его содержимое. При будущих вызовах с тем же URL он тут же читает предыдущее содержимое из кеша, но использует `Promise.resolve`, чтобы сделать из него промис, для того, чтобы возвращаемое значение всегда было промисом:
```js
let cache = new Map();

function loadCached(url) {
	if(cache.has(url)) {
		return Promise.resolve(cache.get(url)); // (*)
	}

	return fetch(url)
		.then(response => respinse.text())
		.then(text => {
			cache.set(url, text);
			return text;
		});
}
```

Мы можем писать `loadCached(url).then(…)`, потому что функция `loadCached` всегда возвращает промис. Мы всегда можем использовать `.then` после `loadCached`. Это и есть цель использования `Promise.resolve` в строке `(*)`.

-   `Promise.reject(error)` создаёт промис, завершённый с ошибкой `error`.

То же самое, что:
```js
let promise = new Promise((resolve, reject) => reject(error));
```

На практике этот метод почти никогда не используется.