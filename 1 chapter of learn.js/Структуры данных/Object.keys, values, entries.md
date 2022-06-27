Перед этим стоит ознакомиться с [[Map и Set]].
Поговорим о переборе структур данных, пройденных ранее.

Методы `map.keys()`, `map.values()`, `map.entries()` - это универсальные методы, и существует общее соглашение использовать их для структур данных.
**Если бы мы делали собственную структуру данных**, то нам также следовало бы их реализовать.

А пока вспомним, для каких структур вышеперечисленные методы поддерживаются:
-   `Map`
-   `Set`
-   `Array`

**Object.keys, values, entries**

Для простых объектов доступны следующие методы:
-   [Object.keys(obj)](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) – возвращает массив ключей. ^4ff6d3
-   [Object.values(obj)](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/values) – возвращает массив значений.
-   [Object.entries(obj)](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/entries) – возвращает массив пар `[ключ, значение]`.

Стоит понимать разницу вызова у некоторого объекта и у map, например:
- Map - map.keys() - вернет **перебираемый объект**
- Object - Object.keys(obj) - вернет **реальный массив**

Почему так? Основная причина – гибкость. Помните, что объекты являются основой всех сложных структур в JavaScript. У нас может быть объект `data`, который реализует свой собственный метод `data.values()`. И мы всё ещё можем применять к нему стандартный метод `Object.values(data)`.
Второе отличие в том, что методы вида `Object.*` возвращают «реальные» массивы, а не просто итерируемые объекты. Это в основном по историческим причинам.
Например:
```js
let user = jonh {
	name: 'John',
	age: 30,
}
```
-   `Object.keys(user) = ["name", "age"]`
-   `Object.values(user) = ["John", 30]`
-   `Object.entries(user) = [ ["name","John"], ["age",30] ]`

**ВАЖНО ЗНАТЬ!**
Так же, как и цикл `for...in`, эти методы игнорируют свойства, использующие `Symbol(...)` в качестве ключей.
Обычно это удобно. Но если требуется учитывать и символьные ключи, то для этого существует отдельный метод [Object.getOwnPropertySymbols](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertySymbols), возвращающий массив только символьных ключей. Также, существует метод [Reflect.ownKeys(obj)](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Reflect/ownKeys), который возвращает _все_ ключи.

**Трансформация объекта**
У объектов нет множества методов, которые есть в массивах, например `map`, `filter` и других.

Если мы хотели бы их применить, то можно использовать `Object.entries` с последующим вызовом `Object.fromEntries`:

1.  Вызов `Object.entries(obj)` возвращает массив пар ключ/значение для `obj`.
2.  На нём вызываем методы массива, например, `map`.
3.  Используем `Object.fromEntries(array)` на результате, чтобы преобразовать его обратно в объект.
Например, у нас есть объект с ценами, и мы хотели бы их удвоить:
```js
let prices = {
	banana: 1,
	orange: 2,
	meat: 4,
};

let doublePrices = Object.fronEntries(
	//Сначала в массив, потом какой-нибудь метод массива
	//потом обратно в объект - fromEntries(array)
	Object.entries(prices).map(
	([key, value]=> [key,value * 2]))
);

alert(doublePrices.meat); // 8
```
Это может выглядеть сложным на первый взгляд, но становится лёгким для понимания после нескольких раз использования.

Можно делать и более сложные «однострочные» преобразования таким путём. Важно только сохранять баланс, чтобы код при этом был достаточно простым для понимания.