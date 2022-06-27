TypeScript берет весь JavaScript и "покрывает" его *статической системой типов*, которые предоставляет сам JS. (ну и парочка типов из TypeScript, но об этом ниже).

Было бы замечательно, если бы у нас был инструмент, который позволяет находить ошибки в коде ПЕРЕД тем, как наш код выполнится.
И да, такой инструмент у нас есть: это TypeScript.

Статическая типизация описывает форму и поведение наших переменных, когда мы запускаем наш код.

**Типы с помощью интерфейсов**

Чтобы создавать объекты, которые содержат определенные свойства, например, стоит прибегать к *интерфейсам*:
```ts
interface User {
	name: string,
	id: number
}

const user: User = {
	name: 'Artem',
	id: 0
}

```

Сделал интерфейс и сказал своему объекту быть похожим, написав первый через двоеточие после него =) Соответственно, этот способ работает и на все инстансы классов.

Также мы можем использовать интерфейсы для аннотации параметров и возвращаемого значения функции:
```ts
function getAdminUser(): User {

	//...

}

function deleteUser(user: User) {

	// ...

}
```

Вспоминаем, какие типы предоставляем нам JS: `number`, `string`, `boolean`, `undefined`, `bigint`, `null`, `symbol`. Эти типы мы можем использовать в интерфейсе. 
TypeScript же немного расширяет пул используемых типов:
- `any` (позволяет быть вообще всему, что есть на этой земле)
- `unknown` (нужно убедиться, что кто-то, использующий этот тип, объявляет, что это за тип)
- `never` (пока не понимаю)
- `void` (если функция возвращает `undefined` или не возвращает вообще ничего)

**Композиция типов**

В TypeScript композиции типов можно добиться двумя способами: с помощью юнионов и с помощью женериков.

Разберём на примере:

*Unions*
Тут вообще все просто: мы говорим переменной, что она может быть тем, тем или тем.
```ts
type MyBool = true | false;
```

Популярный юз-кейс юнионов - это описание множества строк или чисел, которые могут быть как значения переменной:
```ts
type WindowStates = "open" | "closed" | "minimized";

type LockStates = "locked" | "unlocked";

type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;
```

Также можно комбинировать некоторые типы:
```ts
function getLength(obj: string | string[]) {
	return obj.length;
}
```

*Generics*
Дженерики предоставляют переменные к типам! Типичный пример: работа с массивом. Массив без дженерика может содержать что-угодно, а с дженериком - только то, что описано в последнем.

```ts
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{name: string}>
```

Мы можем комбинировать интерфейсы с дженериками:
```ts
interface Backpack<Type> {
	add: (obj: Type) => void;
	get: () => Type;
}

// Шорткат для того, чтобы не использовать каждый раз Backpack<string>
declare const backpack: Backpack<string>

// typeof object - string
const object = backpack.get();

// Ай-ай-ай, в declare написано другое! 
backpack.add(23);
```

**Структурная система типов**

Один из кор-принципов TypeScript'а - утиная типизация или же структурная типизация.
"Если два объекта имеют одну и ту же форму, то они - одного типа".

Например:
```ts
interface Point {
	x: number;
	y: number;
}

function logPoint(p: Point): void {
	console.log(`${p.x}, ${p.y}`);
}


const point = { x: 12, y: 26 };
// logs "12, 26"
logPoint(point);
```

Как мы видим, переменной `point` мы не указали тип `Point`! Но тем не менее, TypeScript сравнивает "форму" `point` с формой `Point` в проверке типа. Так как одинаковая структура, то код не ругается на нас!

Сопоставление формы требует соответствия только подмножеству полей объекта.
Смотрим код:
```ts
const point3 = { x: 12, y: 26, z: 89 };

logPoint(point3); // logs "12, 26"

const rect = { x: 33, y: 3, width: 30, height: 80 };

logPoint(rect); // logs "33, 3"

const color = { hex: "#187ABF" };

logPoint(color);
// Argument of type '{ hex: string; }' is not assignable to parameter of type 'Point'. 
// Type '{ hex: string; }' is missing the following properties from type 'Point': x, y
```

Ну соотвественно, разницы между литером объекта и инстансом класса нет никакого:
```ts
class VirtualPoint {
	x: number;
	y: number;

	constructor(x: number, y: number) {
		this.x = x;
		this.y = y;
	}

}

const newVPoint = new VirtualPoint(13, 56);
logPoint(newVPoint); // logs "13, 56"
```