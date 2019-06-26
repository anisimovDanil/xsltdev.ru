---
layout: default
title: Структуры библиотек
nav_order: 2
grand_parent: Справочник Typescript
parent: Декларации Typescript
---

<!-- prettier-ignore-start -->
# Структуры библиотек
{: .no_toc }
<!-- prettier-ignore-end -->

<!-- prettier-ignore -->
1. TOC
{:toc}

## Обзор

Говоря в общем, _структура_ файла объявлений зависит от того, как используется библиотека.
В JavaScript существует много способов предоставить библиотеку для использования, и файл объявлений должен соответствовать выбранному способу.
Данное руководство описывает, как определить часто используемые шаблоны построения библиотек и создать файлы определений, соответствующие данной практике.

Для каждого распространенного типа библиотек в директории [`templates`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates) есть соответствующий файл.
Чтобы продвигаться быстрее, можно начать с этих файлов.

## Определение типов библиотек

В первую очередь рассмотрим типы библиотек, которые могут описываться файлами определений.
Мы кратко покажем, как _используется_ каждый из типов, как _пишется код_ для них и перечислим примеры таких библиотек.

Первый шаг для написания файла объявлений — определение типа библиотеки.
Мы дадим несколько советов для того, чтобы определить тип библиотеки на основании ее _кода_ и того, как она используется.
Тот или иной способ может оказаться проще в зависимости от документации и организации кода.
Используйте тот, который покажется более удобным.

### Глобальные библиотеки

_Глобальные_ библиотеки — те, которые доступны в глобальной области видимости (т. е., без импорта в какой-либо форме).
Многие библиотеки просто предоставляют для использования одну или несколько глобальных переменных.
К примеру, с [jQuery](https://jquery.com/) переменную `$` можно использовать, просто обратившись к ней:

```ts
$(() => {
  console.log('hello!')
})
```

Как правило, в документации таких библиотек описывается, как использовать ее с тегом `script`:

```html
<script src="http://a.great.cdn.for/someLib.js"></script>
```

На сегодняшний день большинство глобальных библиотек на самом деле являются UMD-библиотеками (см. ниже).
Документацию UMD-библиотеки сложно отличить от документации глобальной библиотеки.
Перед написанием файла объявлений для глобальной библиотеки убедитесь, что она на самом деле не является UMD-библиотекой.

#### Определение глобальной библиотеки по коду

Ее код, как правило, крайне прост.
Глобальная библиотека, выводящая приветствие, может выглядеть так:

```js
function createGreeting(s) {
  return 'Привет, ' + s
}
```

Или так:

```js
window.createGreeting = function(s) {
  return 'Привет, ' + s
}
```

Взглянув на код глобальной библиотеки, обычно можно увидеть:

- Конструкции `var` или `function` на верхнем уровне
- Одно или более присваиваний к `window.какоеТоИмя`
- Предположения, что существуют объекты `document` или `window`

И _нельзя_ увидеть:

- Проверки существования или использование загрузчиков модулей (`require` или `define`)
- Импортирование в стиле CommonJS/Node.js (`var fs = require("fs");`)
- Вызовы `define(...)`
- Документацию, описывающую, как использовать библиотеку с `require` или импортировать ее

#### Примеры глобальных библиотек

Поскольку, как правило, из глобальной библиотеки легко сделать UMD-библиотеку, популярных библиотек, написанных в такой форме, крайне мало.
Однако небольшие и требующие DOM (или не имеющие зависимостей) библиотеки все еще могут оказаться глобальными.

#### Шаблон глобальной библиотеки

Шаблон [`global.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/global.d.ts) описывает библиотеку `myLib`.
Обязательно прочтите [замечание о предотвращении конфликтов имен](#Предотвращение-конфликтов-имен).

### Модульные библиотеки

Некоторые библиотеки работают только в окружении с загрузчиком модулей.
Например, `express` работает только в Node.js и загружается с помощью функции `require` из CommonJS.

В ECMAScript 2015 (также называемом ES2015, ECMAScript 6 или ES6), CommonJS и RequireJS существуют схожие между собой понятия _импортирования модулей_.
Используя CommonJS (Node.JS), можно написать, например:

```ts
var fs = require('fs')
```

В TypeScript или ES6 для тех же целей служит `import`:

```ts
import fs = require('fs')
```

Как правило, документация к модульной библиотеке включает одну из следующих строк:

```js
var someLib = require('someLib')
```

или

```ts
define(..., ['someLib'], function(someLib) {

});
```

Как и в случае с глобальными модулями, такие примеры можно увидеть и в документации к UMD-модулям, поэтому хорошо проверьте код или документацию.

#### Определение модульной библиотеки по коду

В модульных библиотеках можно встретить по крайней мере что-то одно из следующего:

- Безусловные вызовы `require` или `define`
- Объявления наподобие `import * as a from 'b';` или `export c;`
- Присваивания к `exports` или `module.exports`

В них редко можно увидеть:

- Присваивания к свойствам `window` или `global`

#### Примеры модульных библиотек

Многие популярные библиотеки Node.js — модульные, например [`express`](http://expressjs.com/), [`gulp`](http://gulpjs.com/), и [`request`](https://github.com/request/request).

### _UMD_

_UMD_ модуль может использоваться либо как модуль (с помощью импортирования) либо как глобальная библиотека (если запускается в окружении без загрузчика модулей).
Многие популярные библиотеки, к примеру, [Moment.js](http://momentjs.com/), написаны именно так.
Например, в Node.js или используя RequireJS можно написать:

```ts
import moment = require('moment')
console.log(moment.format())
```

в то время как в стандартном браузерном окружении делается так:

```ts
console.log(moment.format())
```

#### Определение UMD библиотеки

[UMD-модули](https://github.com/umdjs/umd) проверяют наличие окружения с загрузчиком модулей.
Это легко заметить, увидев нечто подобное:

```js
;(function(root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(['libName'], factory)
  } else if (typeof module === 'object' && module.exports) {
    module.exports = factory(require('libName'))
  } else {
    root.returnExports = factory(root.libName)
  }
})(this, function(b) {})
```

Если в коде библиотеки есть проверки `typeof define`, `typeof window` или `typeof module`, особенно в начале файла, то, скорее всего, это UMD-библиотека.

Документация таких библиотек часто приводит пример использования в Node.js, где есть `require`, и пример использования в браузере, где для загрузки кода используется тег `<script>`.

#### Примеры UMD-библиотек

Большинство популярных библиотек доступны в виде UMD-пакетов.
Можно упомянуть [jQuery](https://jquery.com/), [Moment.js](http://momentjs.com/), [lodash](https://lodash.com/) и многие другие.

#### Шаблон

Для данного типа модулей доступно три шаблона,
[`module.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/module.d.ts), [`module-class.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/module-class.d.ts) и [`module-function.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/module-function.d.ts).

Используйте [`module-function.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/module-function.d.ts), если модуль может _вызываться_, подобно функции:

```ts
var x = require('foo')
// Внимание: вызываем 'x' как функцию
var y = x(42)
```

Обязательно прочтите [замечание о влиянии ES6 на сигнатуры вызова модулей](#Влияние-es6-на-сигнатуры-вызова-модулей).

Используйте [`module-class.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/module-class.d.ts), если модуль может быть _сконструирован_, используя `new`:

```ts
var x = require('bar')
// Внимание: используем оператор `new` с импортированной переменной
var y = new x('hello')
```

То же самое [замечание](#Влияние-es6-на-сигнатуры-вызова-модулей) относится и к таким модулям.

Если модуль не может быть вызван или сконструирован, используйте файл [`module.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/module.d.ts).

### _Плагин модуля_ или _UMD-плагин_

_Плагин модуля_ изменяет форму другого модуля (простого модуля или UMD).
Например, в Moment.js `moment-range` добавляет новый метод `range` в объекту `moment`.

Код файла объявлений остается тем же вне зависимости от того, является ли изменяемый модуль простым модулем либо UMD.

#### Шаблон

Используйте [`module-plugin.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/module-plugin.d.ts).

### _Глобальный плагин_

_Глобальный плагин_ — это глобальный код, изменяющий форму глобальных объектов.
Так же, как и в случае с _модулями, изменяющими глобальные объекты_, в этом случае появляется вероятность конфликта имен во время выполнения.

Например, некоторые библиотеки добавляют новые функции к `Array.prototype` или `String.prototype`.

#### Определение глобальных плагинов

Глобальные плагины, как правило, легко узнать по документации.

Там можно увидеть подобные примеры:

```ts
var x = 'hello, world'
// Создает новые методы на встроенных типах
console.log(x.startsWithHello())

var y = [1, 2, 3]
// Создает новые методы на встроенных типах
console.log(y.reverseAndSort())
```

#### Шаблон

Используйте шаблон [`global-plugin.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/global-plugin.d.ts).

### _Модули, изменяющие глобальные объекты_

Модули, изменяющие глобальные объекты, при их импортировании изменяют значения в глобальной области видимости.
К примеру, может существовать библиотека, которая при импортировании добавляет новые члены к `String.prototype`.
Такая техника несколько опасна из-за возможности конфликтов во время выполнения, но написать файл объявлений для такой библиотеки все же можно.

#### Определение модулей, изменяющих глобальные объекты

Такие модули легко узнать по документации.
Как правило, они похожи на глобальные плагины, но для внесения изменений нужен вызов `require`.

В документации можно увидеть следующее:

```ts
// вызов 'require', чье возвращаемое значение не используется
var unused = require('magic-string-time')
/* или */
require('magic-string-time')

var x = 'привет, мир'
// Создает новые методы для встроенных типов
console.log(x.startsWithHello())

var y = [1, 2, 3]
// Создает новые методы для встроенных типов
console.log(y.reverseAndSort())
```

#### Шаблон

Используйте шаблон [`global-modifying-module.d.ts`](https://github.com/gooddaytoday/TypeScript-Handbook-RU/tree/master/pages/declaration%20files/templates/global-modifying-module.d.ts).

## Использование зависимостей

Встречаются несколько видов зависимостей.

### Зависимости от глобальных библиотек

Если библиотека зависит от глобальной библиотеки, используйте директиву `/// <reference types="..." />`:

```ts
/// <reference types="someLib" />

function getThing(): someLib.thing
```

### Зависимости от модулей

Если библиотека зависит от модуля, используйте `import`:

```ts
import * as moment from 'moment'

function getThing(): moment
```

### Зависимости от UMD-библиотек

#### Из глобальной библиотеки

Если глобальная библиотека зависит от UMD-модуля, используйте директиву `/// <reference types`:

```ts
/// <reference types="moment" />

function getThing(): moment
```

#### Из модуля или UMD-библиотеки

Если модуль или UMD-библиотека зависит от UMD-библиотеки, используйте `import`:

```ts
import * as someLib from 'someLib'
```

Не используйте `/// <reference` для объявления зависимости от UMD-библиотеки!

## Примечания

### Предотвращение конфликтов имен

Обратите внимание, что при написании глобального файла объявлений есть возможность описать в глобальной области видимости сразу множество типов.
Мы крайне рекомендуем воздерживаться от подобного, посколько это может привести к неразрешимым конфликтам имен при использовании в проекте нескольких файлов объявлений.

Простое правило, которому рекомендуется следовать: объявлять типы только в пространстве имен, соответствующем глобальной переменной библиотеки.
Например, если библиотека определяет глобальное значение `cats`, то следует написать:

```ts
declare namespace cats {
  interface KittySettings {}
}
```

Но _не_

```ts
// на верхнем уровне
interface CatsKittySettings {}
```

Соблюдение данного правила также гарантирует, что библиотека может быть перенесена на UMD без изменений, критичных для пользователей файла объявлений.

### Влияние ES6 на плагины модулей

Некоторые плагины добавляют или изменяют экспорты вернего уровня у существующих модулей.
CommonJS и другие загрузчики это допускают, однако в ES6 модули считаются неизменяемыми, и такое поведение невозможно.
TypeScript не зависит от конкретных загрузчиков, поэтому это не проверяется во время компиляции, но разработчики, намеревающиеся переходить на использование ES6-загрузчика, должны это знать.

### Влияние ES6 на сигнатуры вызова модулей

Многие популярные библиотеки, например, Express, при импортировании предоставляют себя в качестве доступной для вызова функции.
К примеру, Express обычно используется так:

```ts
import exp = require('express')
var app = exp()
```

При использовании загрузчика ES6 объект верхнего уровня (который в данном случае экспортируется как `exp`) может иметь только свойства; вызываемым он ни в коем случае быть не может.
Самое распространенное решение этой проблемы — объявить экспортируемый по умолчанию объект, который может быть вызываемым или конструируемым.
Некоторые эмуляторы загрузчиков модулей сами обнаруживают подобные ситуации и заменяют объект верхнего уровня объектом, экспортируемым по умолчанию.

## Ссылки

- [Структуры библиотек](http://typescript-lang.ru/docs/declaration%20files/Library%20Structures.html)