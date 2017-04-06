# 程式設計風格

本章探討如何將ES6的新語法，運用到編碼實踐之中，與傳統的JavaScript語法結合在一起，寫出合理的、易於閱讀和維護的程式碼。

多家公司和組織已經公開了它們的風格規範，具體可參閱[jscs.info](http://jscs.info/)，下面的內容主要參考了[Airbnb](https://github.com/airbnb/javascript)的JavaScript風格規範。

## 塊級作用域

**（1）let 取代 var**

ES6提出了兩個新的宣告變數的命令：`let`和`const`。其中，`let`完全可以取代`var`，因為兩者語義相同，而且`let`沒有副作用。

```javascript
'use strict';

if (true) {
  let x = 'hello';
}

for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

上面程式碼如果用`var`替代`let`，實際上就聲明瞭兩個全域性變數，這顯然不是本意。變數應該只在其宣告的程式碼塊內有效，`var`命令做不到這一點。

`var`命令存在變數提升效用，`let`命令沒有這個問題。

```javascript
'use strict';

if(true) {
  console.log(x); // ReferenceError
  let x = 'hello';
}
```

上面程式碼如果使用`var`替代`let`，`console.log`那一行就不會報錯，而是會輸出`undefined`，因為變數宣告提升到程式碼塊的頭部。這違反了變數先聲明後使用的原則。

所以，建議不再使用`var`命令，而是使用`let`命令取代。

**（2）全域性常量和執行緒安全**

在`let`和`const`之間，建議優先使用`const`，尤其是在全域性環境，不應該設定變數，只應設定常量。

`const`優於`let`有幾個原因。一個是`const`可以提醒閱讀程式的人，這個變數不應該改變；另一個是`const`比較符合函數語言程式設計思想，運算不改變值，只是新建值，而且這樣也有利於將來的分散式運算；最後一個原因是 JavaScript 編譯器會對`const`進行優化，所以多使用`const`，有利於提供程式的執行效率，也就是說`let`和`const`的本質區別，其實是編譯器內部的處理不同。

```javascript
// bad
var a = 1, b = 2, c = 3;

// good
const a = 1;
const b = 2;
const c = 3;

// best
const [a, b, c] = [1, 2, 3];
```

`const`宣告常量還有兩個好處，一是閱讀程式碼的人立刻會意識到不應該修改這個值，二是防止了無意間修改變數值所導致的錯誤。

所有的函式都應該設定為常量。

長遠來看，JavaScript可能會有多執行緒的實現（比如Intel的River Trail那一類的專案），這時`let`表示的變數，只應出現在單執行緒執行的程式碼中，不能是多執行緒共享的，這樣有利於保證執行緒安全。

## 字串

靜態字串一律使用單引號或反引號，不使用雙引號。動態字串使用反引號。

```javascript
// bad
const a = "foobar";
const b = 'foo' + a + 'bar';

// acceptable
const c = `foobar`;

// good
const a = 'foobar';
const b = `foo${a}bar`;
const c = 'foobar';
```

## 解構賦值

使用陣列成員對變數賦值時，優先使用解構賦值。

```javascript
const arr = [1, 2, 3, 4];

// bad
const first = arr[0];
const second = arr[1];

// good
const [first, second] = arr;
```

函式的引數如果是物件的成員，優先使用解構賦值。

```javascript
// bad
function getFullName(user) {
  const firstName = user.firstName;
  const lastName = user.lastName;
}

// good
function getFullName(obj) {
  const { firstName, lastName } = obj;
}

// best
function getFullName({ firstName, lastName }) {
}
```

如果函式返回多個值，優先使用物件的解構賦值，而不是陣列的解構賦值。這樣便於以後新增返回值，以及更改返回值的順序。

```javascript
// bad
function processInput(input) {
  return [left, right, top, bottom];
}

// good
function processInput(input) {
  return { left, right, top, bottom };
}

const { left, right } = processInput(input);
```

## 物件

單行定義的物件，最後一個成員不以逗號結尾。多行定義的物件，最後一個成員以逗號結尾。

```javascript
// bad
const a = { k1: v1, k2: v2, };
const b = {
  k1: v1,
  k2: v2
};

// good
const a = { k1: v1, k2: v2 };
const b = {
  k1: v1,
  k2: v2,
};
```

物件儘量靜態化，一旦定義，就不得隨意新增新的屬性。如果新增屬性不可避免，要使用`Object.assign`方法。

```javascript
// bad
const a = {};
a.x = 3;

// if reshape unavoidable
const a = {};
Object.assign(a, { x: 3 });

// good
const a = { x: null };
a.x = 3;
```

如果物件的屬性名是動態的，可以在創造物件的時候，使用屬性表示式定義。

```javascript
// bad
const obj = {
  id: 5,
  name: 'San Francisco',
};
obj[getKey('enabled')] = true;

// good
const obj = {
  id: 5,
  name: 'San Francisco',
  [getKey('enabled')]: true,
};
```

上面程式碼中，物件`obj`的最後一個屬性名，需要計算得到。這時最好採用屬性表示式，在新建`obj`的時候，將該屬性與其他屬性定義在一起。這樣一來，所有屬性就在一個地方定義了。

另外，物件的屬性和方法，儘量採用簡潔表達法，這樣易於描述和書寫。

```javascript
var ref = 'some value';

// bad
const atom = {
  ref: ref,

  value: 1,

  addValue: function (value) {
    return atom.value + value;
  },
};

// good
const atom = {
  ref,

  value: 1,

  addValue(value) {
    return atom.value + value;
  },
};
```

## 陣列

使用展開運算子（...）拷貝陣列。

```javascript
// bad
const len = items.length;
const itemsCopy = [];
let i;

for (i = 0; i < len; i++) {
  itemsCopy[i] = items[i];
}

// good
const itemsCopy = [...items];
```

使用Array.from方法，將類似陣列的物件轉為陣列。

```javascript
const foo = document.querySelectorAll('.foo');
const nodes = Array.from(foo);
```

## 函式

立即執行函式可以寫成箭頭函式的形式。

```javascript
(() => {
  console.log('Welcome to the Internet.');
})();
```

那些需要使用函式表示式的場合，儘量用箭頭函式代替。因為這樣更簡潔，而且綁定了this。

```javascript
// bad
[1, 2, 3].map(function (x) {
  return x * x;
});

// good
[1, 2, 3].map((x) => {
  return x * x;
});

// best
[1, 2, 3].map(x => x * x);
```

箭頭函式取代`Function.prototype.bind`，不應再用self/\_this/that繫結 this。

```javascript
// bad
const self = this;
const boundMethod = function(...params) {
  return method.apply(self, params);
}

// acceptable
const boundMethod = method.bind(this);

// best
const boundMethod = (...params) => method.apply(this, params);
```

簡單的、單行的、不會複用的函式，建議採用箭頭函式。如果函式體較為複雜，行數較多，還是應該採用傳統的函式寫法。

所有配置項都應該集中在一個物件，放在最後一個引數，布林值不可以直接作為引數。

```javascript
// bad
function divide(a, b, option = false ) {
}

// good
function divide(a, b, { option = false } = {}) {
}
```

不要在函式體內使用arguments變數，使用rest運算子（...）代替。因為rest運算子顯式表明你想要獲取引數，而且arguments是一個類似陣列的物件，而rest運算子可以提供一個真正的陣列。

```javascript
// bad
function concatenateAll() {
  const args = Array.prototype.slice.call(arguments);
  return args.join('');
}

// good
function concatenateAll(...args) {
  return args.join('');
}
```

使用預設值語法設定函式引數的預設值。

```javascript
// bad
function handleThings(opts) {
  opts = opts || {};
}

// good
function handleThings(opts = {}) {
  // ...
}
```

## Map結構

注意區分Object和Map，只有模擬現實世界的實體物件時，才使用Object。如果只是需要`key: value`的資料結構，使用Map結構。因為Map有內建的遍歷機制。

```javascript
let map = new Map(arr);

for (let key of map.keys()) {
  console.log(key);
}

for (let value of map.values()) {
  console.log(value);
}

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
```

## Class

總是用Class，取代需要prototype的操作。因為Class的寫法更簡潔，更易於理解。

```javascript
// bad
function Queue(contents = []) {
  this._queue = [...contents];
}
Queue.prototype.pop = function() {
  const value = this._queue[0];
  this._queue.splice(0, 1);
  return value;
}

// good
class Queue {
  constructor(contents = []) {
    this._queue = [...contents];
  }
  pop() {
    const value = this._queue[0];
    this._queue.splice(0, 1);
    return value;
  }
}
```

使用`extends`實現繼承，因為這樣更簡單，不會有破壞`instanceof`運算的危險。

```javascript
// bad
const inherits = require('inherits');
function PeekableQueue(contents) {
  Queue.apply(this, contents);
}
inherits(PeekableQueue, Queue);
PeekableQueue.prototype.peek = function() {
  return this._queue[0];
}

// good
class PeekableQueue extends Queue {
  peek() {
    return this._queue[0];
  }
}
```

## 模組

首先，Module語法是JavaScript模組的標準寫法，堅持使用這種寫法。使用`import`取代`require`。

```javascript
// bad
const moduleA = require('moduleA');
const func1 = moduleA.func1;
const func2 = moduleA.func2;

// good
import { func1, func2 } from 'moduleA';
```

使用`export`取代`module.exports`。

```javascript
// commonJS的寫法
var React = require('react');

var Breadcrumbs = React.createClass({
  render() {
    return <nav />;
  }
});

module.exports = Breadcrumbs;

// ES6的寫法
import React from 'react';

const Breadcrumbs = React.createClass({
  render() {
    return <nav />;
  }
});

export default Breadcrumbs
```

如果模組只有一個輸出值，就使用`export default`，如果模組有多個輸出值，就不使用`export default`，不要`export default`與普通的`export`同時使用。

不要在模組輸入中使用萬用字元。因為這樣可以確保你的模組之中，有一個預設輸出（export default）。

```javascript
// bad
import * as myObject './importModule';

// good
import myObject from './importModule';
```

如果模組預設輸出一個函式，函式名的首字母應該小寫。

```javascript
function makeStyleGuide() {
}

export default makeStyleGuide;
```

如果模組預設輸出一個物件，物件名的首字母應該大寫。

```javascript
const StyleGuide = {
  es6: {
  }
};

export default StyleGuide;
```

## ESLint的使用

ESLint是一個語法規則和程式碼風格的檢查工具，可以用來保證寫出語法正確、風格統一的程式碼。

首先，安裝ESLint。

```bash
$ npm i -g eslint
```

然後，安裝Airbnb語法規則。

```bash
$ npm i -g eslint-config-airbnb
```

最後，在專案的根目錄下新建一個`.eslintrc`檔案，配置ESLint。

```javascript
{
  "extends": "eslint-config-airbnb"
}
```

現在就可以檢查，當前專案的程式碼是否符合預設的規則。

`index.js`檔案的程式碼如下。

```javascript
var unusued = 'I have no purpose!';

function greet() {
    var message = 'Hello, World!';
    alert(message);
}

greet();
```

使用ESLint檢查這個檔案。

```bash
$ eslint index.js
index.js
  1:5  error  unusued is defined but never used                 no-unused-vars
  4:5  error  Expected indentation of 2 characters but found 4  indent
  5:5  error  Expected indentation of 2 characters but found 4  indent

✖ 3 problems (3 errors, 0 warnings)
```

上面程式碼說明，原檔案有三個錯誤，一個是定義了變數，卻沒有使用，另外兩個是行首縮排為4個空格，而不是規定的2個空格。
