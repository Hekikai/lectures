Оператор `instanceof` позволяет проверить, к какому классу принадлежит объект, с учетом наследования.

Такая проверка может потребоваться во многих случаях. Здесь мы используем её для создания _полиморфной_ функции, которая интерпретирует аргументы по-разному в зависимости от их типа.

**Оператор instanceof**

Синтаксис:
```js
obj instanceof Class
```

Оператор вернёт `true`, если `obj` принадлежит классу `Class` или наследующему от него.

Например:
```js
class Rabbit {}
let rabbit = new Rabbit();

// это объект класса Rabbit?
console.log( rabbit instanceof Rabbit ); // true
```

Также это работает с функциями-конструкторами:
```js
// вместо класса 
function Rabbit() {}

console.log( new Rabbit() instanceof Rabbit ); // true
```

…И для встроенных классов, таких как `Array`:
```js
let arr = [1, 2, 3]; 
console.log( arr instanceof Array );  // true 
console.log( arr instanceof Object ); // true
```

Пожалуйста, обратите внимание, что `arr` также принадлежит классу `Object`, потому что `Array` наследует от `Object`.

Обычно оператор `instanceof` просматривает для проверки цепочку прототипов. Но это поведение может быть изменено при помощи статического метода `Symbol.hasInstance`.

Алгоритм работы `obj instanceof Class` работает примерно так:

1. Если имеется статический метод `Symbol.hasInstance`, тогда вызвать его: `Class[Symbol.hasInstance](obj)`. Он должен вернуть либо `true`, либо `false`, и это конец. Это как раз и есть возможность ручной настройки `instanceof`.

Пример:
```js
// проверка instanceof будет полагать,
// что всё со свойством canEat - животное Animal
class Animal {
	static [Symbol.hasInstance](obj) {
		if (obj.canEat) return true;
	}
}

let obj = { canEat: true };

console.log(obj instanceof Animal); // true: 
// вызван Animal[Symbol.hasInstance](obj)
```

2. Большая часть классов не имеет метода `Symbol.hasInstance`. В этом случае используется стандартная логика: проверяется, равен ли `Class.prototype` одному из прототипов в прототипной цепочке `obj`.

Другими словами, сравнивается:
```js
obj.__proto__ === Class.prototype?
obj.__proto__.__proto__ === Class.prototype?
obj.__proto__.__proto__.__proto__ === Class.prototype?
...
// если какой-то из ответов true - возвратить true
// если дошли до конца цепочки - false
```

В примере выше `rabbit.__proto__ === Rabbit.prototype`, так что результат будет получен немедленно.

В случае с наследованием, совпадение будет на втором шаге:
```js
class Animal {}
class Rabbit extends Animal {}

let rabbit = new Rabbit();
console.log(rabbit instanceof Animal); // true

// rabbit.__proto__ === Animal.prototype (нет совпадения)
// rabbit.__proto__.__proto__ === Animal.prototype (совпадение!)
```
Вот иллюстрация того как `rabbit instanceof Animal` сравнивается с `Animal.prototype`:
![[Pasted image 20220103085915.png]]
Кстати, есть метод [objA.isPrototypeOf(objB)](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/object/isPrototypeOf), который возвращает `true`, если объект `objA` есть где-то в прототипной цепочке объекта `objB`. Так что `obj instanceof Class` можно перефразировать как `Class.prototype.isPrototypeOf(obj)`.

Забавно, но сам *конструктор `Class` не участвует в процессе проверки!* Важна только цепочка прототипов `Class.prototype`.

Это может приводить к интересным последствиям при изменении свойства `prototype` после создания объекта.

Как, например, тут:
```js
function Rabbit() {}
let rabbit = new Rabbit();

// заменяем прототип 
Rabbit.prototype = {};

// ...больше не rabbit! 
console.log( rabbit instanceof Rabbit ); // false
```

**Object.prototype.toString возвращает тип**

Мы уже знаем, что обычные объекты преобразуется к строке как `[object Object]`:
```js
let obj = {};

console.log(obj.toString()); // [object Object]
```
Так работает реализация метода `toString`. Но у `toString` имеются скрытые возможности, которые делают метод гораздо более мощным. Мы можем использовать его как расширенную версию `typeof` и как альтернативу `instanceof`.

Звучит странно? Так и есть. Давайте развеем мистику.

Согласно [спецификации](https://tc39.github.io/ecma262/#sec-object.prototype.tostring) встроенный метод `toString` может быть позаимствован у объекта и вызван в контексте любого другого значения. И результат зависит от типа этого значения.
-   Для числа это будет `[object Number]`
-   Для булева типа это будет `[object Boolean]`
-   Для `null`: `[object Null]`
-   Для `undefined`: `[object Undefined]`
-   Для массивов: `[object Array]`
-   …и т.д. (поведение настраивается).

Сразу же пример:
```js
// скопируем метод toString в переменную для удобства
let objectToString = Object.prototype.toString;

// какой это тип?
let arr = [];

console.log( objectToString.call(arr) ); // [object Array]
```
Внутри, алгоритм метода `toString` анализирует контекст вызова `this` и возвращает соответствующий результат. Больше примеров:
```js
let s = Object.prototype.toString;

console.log( s.call(123)   ); // [object Number]
console.log( s.call(null)  ); // [object Null]
console.log( s.call(alert) ); // [object Function]
```

**Symbol.toStringTag**

Поведение метода объектов `toString` можно настраивать, используя специальное свойство объекта `Symbol.toStringTag`.

Например:
```js
let user = {
	[Symbol.toStringTag]: "User"
};

console.log( {}.toString.call(user) ); // [object User]
```

Такое свойство есть у большей части объектов, специфичных для определённых окружений. Вот несколько примеров для браузера:
```js
console.log( window[Symbol.toStringTag] ); // window
console.log( XMLHttpRequest.prototype[Symbol.toStringTag] ); // XMLHttpRequest

console.log( {}.toString.call(window) ); // [object Window]
console.log( {}.toString.call(new XMLHttpRequest()) ); // [object XMLHttpRequest]
```

Как вы можете видеть, результат – это значение `Symbol.toStringTag` (если он имеется) обёрнутое в `[object ...]`.

В итоге мы получили «typeof на стероидах», который не только работает с примитивными типами данных, но также и со встроенными объектами, и даже может быть настроен.

Можно использовать `{}.toString.call` вместо `instanceof` для встроенных объектов, когда мы хотим получить тип в виде строки, а не просто сделать проверку.

 | работает для | возвращает 
------ | --------- | --------| 
примитивов | строка |  *typeof*
примитивов, встроенных объектов, объектов с `Symbol.toStringTag` | строка | *{}.toString*
объектов | true/false | *instanceof*

Как мы можем видеть, технически `{}.toString` «более продвинут», чем `typeof`.

А оператор `instanceof` – отличный выбор, когда мы работаем с иерархией классов и хотим делать проверки с учётом наследования.