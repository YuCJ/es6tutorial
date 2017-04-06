# Module 的載入實現

上一章介紹了模組的語法，本章介紹如何在瀏覽器和 Node 之中載入 ES6 模組，以及實際開發中經常遇到的一些問題（比如迴圈載入）。

## 瀏覽器載入

### 傳統方法

在 HTML 網頁中，瀏覽器通過`<script>`標籤載入 JavaScript 指令碼。

```html
<!-- 頁面內嵌的指令碼 -->
<script type="application/javascript">
  // module code
</script>

<!-- 外部指令碼 -->
<script type="application/javascript" src="path/to/myModule.js">
</script>
```

上面程式碼中，由於瀏覽器指令碼的預設語言是 JavaScript，因此`type="application/javascript"`可以省略。

預設情況下，瀏覽器是同步載入 JavaScript 指令碼，即渲染引擎遇到`<script>`標籤就會停下來，等到執行完指令碼，再繼續向下渲染。如果是外部指令碼，還必須加入指令碼下載的時間。

如果指令碼體積很大，下載和執行的時間就會很長，因此成瀏覽器堵塞，使用者會感覺到瀏覽器“卡死”了，沒有任何響應。這顯然是很不好的體驗，所以瀏覽器允許指令碼非同步載入，下面就是兩種非同步載入的語法。

```html
<script src="path/to/myModule.js" defer></script>
<script src="path/to/myModule.js" async></script>
```

上面程式碼中，`<script>`標籤開啟`defer`或`async`屬性，指令碼就會非同步載入。渲染引擎遇到這一行命令，就會開始下載外部指令碼，但不會等它下載和執行，而是直接執行後面的命令。

`defer`與`async`的區別是：前者要等到整個頁面正常渲染結束，才會執行；後者一旦下載完，渲染引擎就會中斷渲染，執行這個指令碼以後，再繼續渲染。一句話，`defer`是“渲染完再執行”，`async`是“下載完就執行”。另外，如果有多個`defer`指令碼，會按照它們在頁面出現的順序載入，而多個`async`指令碼是不能保證載入順序的。

### 載入規則

瀏覽器載入 ES6 模組，也使用`<script>`標籤，但是要加入`type="module"`屬性。

```html
<script type="module" src="foo.js"></script>
```

上面程式碼在網頁中插入一個模組`foo.js`，由於`type`屬性設為`module`，所以瀏覽器知道這是一個 ES6 模組。

瀏覽器對於帶有`type="module"`的`<script>`，都是非同步載入，不會造成堵塞瀏覽器，即等到整個頁面渲染完，再執行模組指令碼，等同於打開了`<script>`標籤的`defer`屬性。

```html
<script type="module" src="foo.js"></script>
<!-- 等同於 -->
<script type="module" src="foo.js" defer></script>
```

`<script>`標籤的`async`屬性也可以開啟，這時只要載入完成，渲染引擎就會中斷渲染立即執行。執行完成後，再恢復渲染。

```html
<script type="module" src="foo.js" async></script>
```

ES6 模組也允許內嵌在網頁中，語法行為與載入外部指令碼完全一致。

```html
<script type="module">
  import utils from "./utils.js";

  // other code
</script>
```

對於外部的模組指令碼（上例是`foo.js`），有幾點需要注意。

- 程式碼是在模組作用域之中執行，而不是在全域性作用域執行。模組內部的頂層變數，外部不可見。
- 模組指令碼自動採用嚴格模式，不管有沒有宣告`use strict`。
- 模組之中，可以使用`import`命令載入其他模組（`.js`字尾不可省略，需要提供絕對 URL 或相對 URL），也可以使用`export`命令輸出對外介面。
- 模組之中，頂層的`this`關鍵字返回`undefined`，而不是指向`window`。也就是說，在模組頂層使用`this`關鍵字，是無意義的。
- 同一個模組如果載入多次，將只執行一次。

下面是一個示例模組。

```javascript
import utils from 'https://example.com/js/utils.js';

const x = 1;

console.log(x === window.x); //false
console.log(this === undefined); // true

delete x; // 句法錯誤，嚴格模式禁止刪除變數
```

利用頂層的`this`等於`undefined`這個語法點，可以偵測當前程式碼是否在 ES6 模組之中。

```javascript
const isNotModuleScript = this !== undefined;
```

## ES6 模組與 CommonJS 模組的差異

討論 Node 載入 ES6 模組之前，必須瞭解 ES6 模組與 CommonJS 模組完全不同。

它們有兩個重大差異。

- CommonJS 模組輸出的是一個值的拷貝，ES6 模組輸出的是值的引用。
- CommonJS 模組是執行時載入，ES6 模組是編譯時輸出介面。

第二個差異是因為 CommonJS 載入的是一個物件（即`module.exports`屬性），該物件只有在指令碼執行完才會生成。而 ES6 模組不是物件，它的對外介面只是一種靜態定義，在程式碼靜態解析階段就會生成。

下面重點解釋第一個差異。

CommonJS 模組輸出的是值的拷貝，也就是說，一旦輸出一個值，模組內部的變化就影響不到這個值。請看下面這個模組檔案`lib.js`的例子。

```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
```

上面程式碼輸出內部變數`counter`和改寫這個變數的內部方法`incCounter`。然後，在`main.js`裡面載入這個模組。

```javascript
// main.js
var mod = require('./lib');

console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```

上面程式碼說明，`lib.js`模組載入以後，它的內部變化就影響不到輸出的`mod.counter`了。這是因為`mod.counter`是一個原始型別的值，會被快取。除非寫成一個函式，才能得到內部變動後的值。

```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  get counter() {
    return counter
  },
  incCounter: incCounter,
};
```

上面程式碼中，輸出的`counter`屬性實際上是一個取值器函式。現在再執行`main.js`，就可以正確讀取內部變數`counter`的變動了。

```bash
$ node main.js
3
4
```

ES6 模組的執行機制與 CommonJS 不一樣。JS 引擎對指令碼靜態分析的時候，遇到模組載入命令`import`，就會生成一個只讀引用。等到指令碼真正執行時，再根據這個只讀引用，到被載入的那個模組裡面去取值。換句話說，ES6 的`import`有點像 Unix 系統的“符號連線”，原始值變了，`import`載入的值也會跟著變。因此，ES6 模組是動態引用，並且不會快取值，模組裡面的變數繫結其所在的模組。

還是舉上面的例子。

```javascript
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}

// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```

上面程式碼說明，ES6 模組輸入的變數`counter`是活的，完全反應其所在模組`lib.js`內部的變化。

再舉一個出現在`export`一節中的例子。

```javascript
// m1.js
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);

// m2.js
import {foo} from './m1.js';
console.log(foo);
setTimeout(() => console.log(foo), 500);
```

上面程式碼中，`m1.js`的變數`foo`，在剛載入時等於`bar`，過了500毫秒，又變為等於`baz`。

讓我們看看，`m2.js`能否正確讀取這個變化。

```bash
$ babel-node m2.js

bar
baz
```

上面程式碼表明，ES6 模組不會快取執行結果，而是動態地去被載入的模組取值，並且變數總是繫結其所在的模組。

由於 ES6 輸入的模組變數，只是一個“符號連線”，所以這個變數是隻讀的，對它進行重新賦值會報錯。

```javascript
// lib.js
export let obj = {};

// main.js
import { obj } from './lib';

obj.prop = 123; // OK
obj = {}; // TypeError
```

上面程式碼中，`main.js`從`lib.js`輸入變數`obj`，可以對`obj`新增屬性，但是重新賦值就會報錯。因為變數`obj`指向的地址是隻讀的，不能重新賦值，這就好比`main.js`創造了一個名為`obj`的`const`變數。

最後，`export`通過介面，輸出的是同一個值。不同的指令碼載入這個介面，得到的都是同樣的實例。

```javascript
// mod.js
function C() {
  this.sum = 0;
  this.add = function () {
    this.sum += 1;
  };
  this.show = function () {
    console.log(this.sum);
  };
}

export let c = new C();
```

上面的指令碼`mod.js`，輸出的是一個`C`的實例。不同的指令碼載入這個模組，得到的都是同一個實例。

```javascript
// x.js
import {c} from './mod';
c.add();

// y.js
import {c} from './mod';
c.show();

// main.js
import './x';
import './y';
```

現在執行`main.js`，輸出的是`1`。

```bash
$ babel-node main.js
1
```

這就證明了`x.js`和`y.js`載入的都是`C`的同一個實例。

## Node 載入

### 概述

Node 對 ES6 模組的處理比較麻煩，因為它有自己的 CommonJS 模組格式，與 ES6 模組格式是不相容的。目前的解決方案是，將兩者分開，ES6 模組和 CommonJS 採用各自的載入方案。

在靜態分析階段，一個模組指令碼只要有一行`import`或`export`語句，Node 就會認為該指令碼為 ES6 模組，否則就為 CommonJS 模組。如果不輸出任何介面，但是希望被 Node 認為是 ES6 模組，可以在指令碼中加一行語句。

```javascript
export {};
```

上面的命令並不是輸出一個空物件，而是不輸出任何介面的 ES6 標準寫法。

如何不指定絕對路徑，Node 載入 ES6 模組會依次尋找以下指令碼，與`require()`的規則一致。

```javascript
import './foo';
// 依次尋找
//   ./foo.js
//   ./foo/package.json
//   ./foo/index.js

import 'baz';
// 依次尋找
//   ./node_modules/baz.js
//   ./node_modules/baz/package.json
//   ./node_modules/baz/index.js
// 尋找上一級目錄
//   ../node_modules/baz.js
//   ../node_modules/baz/package.json
//   ../node_modules/baz/index.js
// 再上一級目錄
```

ES6 模組之中，頂層的`this`指向`undefined`；CommonJS 模組的頂層`this`指向當前模組，這是兩者的一個重大差異。

### import 命令載入 CommonJS 模組

Node 採用 CommonJS 模組格式，模組的輸出都定義在`module.exports`這個屬性上面。在 Node 環境中，使用`import`命令載入 CommonJS 模組，Node 會自動將`module.exports`屬性，當作模組的預設輸出，即等同於`export default`。

下面是一個 CommonJS 模組。

```javascript
// a.js
module.exports = {
  foo: 'hello',
  bar: 'world'
};

// 等同於
export default {
  foo: 'hello',
  bar: 'world'
};
```

`import`命令載入上面的模組，`module.exports`會被視為預設輸出。

```javascript
// 寫法一
import baz from './a';
// baz = {foo: 'hello', bar: 'world'};

// 寫法二
import {default as baz} from './a';
// baz = {foo: 'hello', bar: 'world'};
```

如果採用整體輸入的寫法（`import * as xxx from someModule`），`default`會取代`module.exports`，作為輸入的介面。

```javascript
import * as baz from './a';
// baz = {
//   get default() {return module.exports;},
//   get foo() {return this.default.foo}.bind(baz),
//   get bar() {return this.default.bar}.bind(baz)
// }
```

上面程式碼中，`this.default`取代了`module.exports`。需要注意的是，Node 會自動為`baz`新增`default`屬性，通過`baz.default`拿到`module.exports`。

```javascript
// b.js
module.exports = null;

// es.js
import foo from './b';
// foo = null;

import * as bar from './b';
// bar = {default:null};
```

上面程式碼中，`es.js`採用第二種寫法時，要通過`bar.default`這樣的寫法，才能拿到`module.exports`。

下面是另一個例子。

```javascript
// c.js
module.exports = function two() {
  return 2;
};

// es.js
import foo from './c';
foo(); // 2

import * as bar from './c';
bar.default(); // 2
bar(); // throws, bar is not a function
```

上面程式碼中，`bar`本身是一個物件，不能當作函式呼叫，只能通過`bar.default`呼叫。

CommonJS 模組的輸出快取機制，在 ES6 載入方式下依然有效。

```javascript
// foo.js
module.exports = 123;
setTimeout(_ => module.exports = null);
```

上面程式碼中，對於載入`foo.js`的指令碼，`module.exports`將一直是`123`，而不會變成`null`。

由於 ES6 模組是編譯時確定輸出介面，CommonJS 模組是執行時確定輸出介面，所以採用`import`命令載入 CommonJS 模組時，不允許採用下面的寫法。

```javascript
import {readfile} from 'fs';
```

上面的寫法不正確，因為`fs`是 CommonJS 格式，只有在執行時才能確定`readfile`介面，而`import`命令要求編譯時就確定這個介面。解決方法就是改為整體輸入。

```javascript
import * as express from 'express';
const app = express.default();

import express from 'express';
const app = express();
```

### require 命令載入 ES6 模組

採用`require`命令載入 ES6 模組時，ES6 模組的所有輸出介面，會成為輸入物件的屬性。

```javascript
// es.js
let foo = {bar:'my-default'};
export default foo;
foo = null;

// cjs.js
const es_namespace = require('./es');
console.log(es_namespace.default);
// {bar:'my-default'}
```

上面程式碼中，`default`介面變成了`es_namespace.default`屬性。另外，由於存在快取機制，`es.js`對`foo`的重新賦值沒有在模組外部反映出來。

下面是另一個例子。

```javascript
// es.js
export let foo = {bar:'my-default'};
export {foo as bar};
export function f() {};
export class c {};

// cjs.js
const es_namespace = require('./es');
// es_namespace = {
//   get foo() {return foo;}
//   get bar() {return foo;}
//   get f() {return f;}
//   get c() {return c;}
// }
```

## 迴圈載入

“迴圈載入”（circular dependency）指的是，`a`指令碼的執行依賴`b`指令碼，而`b`指令碼的執行又依賴`a`指令碼。

```javascript
// a.js
var b = require('b');

// b.js
var a = require('a');
```

通常，“迴圈載入”表示存在強耦合，如果處理不好，還可能導致遞迴載入，使得程式無法執行，因此應該避免出現。

但是實際上，這是很難避免的，尤其是依賴關係複雜的大專案，很容易出現`a`依賴`b`，`b`依賴`c`，`c`又依賴`a`這樣的情況。這意味著，模組載入機制必須考慮“迴圈載入”的情況。

對於JavaScript語言來說，目前最常見的兩種模組格式CommonJS和ES6，處理“迴圈載入”的方法是不一樣的，返回的結果也不一樣。

### CommonJS模組的載入原理

介紹ES6如何處理"迴圈載入"之前，先介紹目前最流行的CommonJS模組格式的載入原理。

CommonJS的一個模組，就是一個指令碼檔案。`require`命令第一次載入該指令碼，就會執行整個指令碼，然後在記憶體生成一個物件。

```javascript
{
  id: '...',
  exports: { ... },
  loaded: true,
  ...
}
```

上面程式碼就是Node內部載入模組後生成的一個物件。該物件的`id`屬性是模組名，`exports`屬性是模組輸出的各個介面，`loaded`屬性是一個布林值，表示該模組的指令碼是否執行完畢。其他還有很多屬性，這裡都省略了。

以後需要用到這個模組的時候，就會到`exports`屬性上面取值。即使再次執行`require`命令，也不會再次執行該模組，而是到快取之中取值。也就是說，CommonJS模組無論載入多少次，都只會在第一次載入時執行一次，以後再載入，就返回第一次執行的結果，除非手動清除系統快取。

### CommonJS模組的迴圈載入

CommonJS模組的重要特性是載入時執行，即指令碼程式碼在`require`的時候，就會全部執行。一旦出現某個模組被"迴圈載入"，就只輸出已經執行的部分，還未執行的部分不會輸出。

讓我們來看，Node[官方文件](https://nodejs.org/api/modules.html#modules_cycles)裡面的例子。指令碼檔案`a.js`程式碼如下。

```javascript
exports.done = false;
var b = require('./b.js');
console.log('在 a.js 之中，b.done = %j', b.done);
exports.done = true;
console.log('a.js 執行完畢');
```

上面程式碼之中，`a.js`指令碼先輸出一個`done`變數，然後載入另一個指令碼檔案`b.js`。注意，此時`a.js`程式碼就停在這裡，等待`b.js`執行完畢，再往下執行。

再看`b.js`的程式碼。

```javascript
exports.done = false;
var a = require('./a.js');
console.log('在 b.js 之中，a.done = %j', a.done);
exports.done = true;
console.log('b.js 執行完畢');
```

上面程式碼之中，`b.js`執行到第二行，就會去載入`a.js`，這時，就發生了“迴圈載入”。系統會去`a.js`模組對應物件的`exports`屬性取值，可是因為`a.js`還沒有執行完，從`exports`屬性只能取回已經執行的部分，而不是最後的值。

`a.js`已經執行的部分，只有一行。

```javascript
exports.done = false;
```

因此，對於`b.js`來說，它從`a.js`只輸入一個變數`done`，值為`false`。

然後，`b.js`接著往下執行，等到全部執行完畢，再把執行權交還給`a.js`。於是，`a.js`接著往下執行，直到執行完畢。我們寫一個指令碼`main.js`，驗證這個過程。

```javascript
var a = require('./a.js');
var b = require('./b.js');
console.log('在 main.js 之中, a.done=%j, b.done=%j', a.done, b.done);
```

執行`main.js`，執行結果如下。

```bash
$ node main.js

在 b.js 之中，a.done = false
b.js 執行完畢
在 a.js 之中，b.done = true
a.js 執行完畢
在 main.js 之中, a.done=true, b.done=true
```

上面的程式碼證明了兩件事。一是，在`b.js`之中，`a.js`沒有執行完畢，只執行了第一行。二是，`main.js`執行到第二行時，不會再次執行`b.js`，而是輸出快取的`b.js`的執行結果，即它的第四行。

```javascript
exports.done = true;
```

總之，CommonJS輸入的是被輸出值的拷貝，不是引用。

另外，由於CommonJS模組遇到迴圈載入時，返回的是當前已經執行的部分的值，而不是程式碼全部執行後的值，兩者可能會有差異。所以，輸入變數的時候，必須非常小心。

```javascript
var a = require('a'); // 安全的寫法
var foo = require('a').foo; // 危險的寫法

exports.good = function (arg) {
  return a.foo('good', arg); // 使用的是 a.foo 的最新值
};

exports.bad = function (arg) {
  return foo('bad', arg); // 使用的是一個部分載入時的值
};
```

上面程式碼中，如果發生迴圈載入，`require('a').foo`的值很可能後面會被改寫，改用`require('a')`會更保險一點。

### ES6模組的迴圈載入

ES6處理“迴圈載入”與CommonJS有本質的不同。ES6模組是動態引用，如果使用`import`從一個模組載入變數（即`import foo from 'foo'`），那些變數不會被快取，而是成為一個指向被載入模組的引用，需要開發者自己保證，真正取值的時候能夠取到值。

請看下面這個例子。

```javascript
// a.js如下
import {bar} from './b.js';
console.log('a.js');
console.log(bar);
export let foo = 'foo';

// b.js
import {foo} from './a.js';
console.log('b.js');
console.log(foo);
export let bar = 'bar';
```

上面程式碼中，`a.js`載入`b.js`，`b.js`又載入`a.js`，構成迴圈載入。執行`a.js`，結果如下。

```bash
$ babel-node a.js
b.js
undefined
a.js
bar
```

上面程式碼中，由於`a.js`的第一行是載入`b.js`，所以先執行的是`b.js`。而`b.js`的第一行又是載入`a.js`，這時由於`a.js`已經開始執行了，所以不會重複執行，而是繼續往下執行`b.js`，所以第一行輸出的是`b.js`。

接著，`b.js`要列印變數`foo`，這時`a.js`還沒執行完，取不到`foo`的值，導致打印出來是`undefined`。`b.js`執行完，開始執行`a.js`，這時就一切正常了。

再看一個稍微複雜的例子（摘自 Dr. Axel Rauschmayer 的[《Exploring ES6》](http://exploringjs.com/es6/ch_modules.html)）。

```javascript
// a.js
import {bar} from './b.js';
export function foo() {
  console.log('foo');
  bar();
  console.log('執行完畢');
}
foo();

// b.js
import {foo} from './a.js';
export function bar() {
  console.log('bar');
  if (Math.random() > 0.5) {
    foo();
  }
}
```

按照CommonJS規範，上面的程式碼是沒法執行的。`a`先載入`b`，然後`b`又載入`a`，這時`a`還沒有任何執行結果，所以輸出結果為`null`，即對於`b.js`來說，變數`foo`的值等於`null`，後面的`foo()`就會報錯。

但是，ES6可以執行上面的程式碼。

```bash
$ babel-node a.js
foo
bar
執行完畢

// 執行結果也有可能是
foo
bar
foo
bar
執行完畢
執行完畢
```

上面程式碼中，`a.js`之所以能夠執行，原因就在於ES6載入的變數，都是動態引用其所在的模組。只要引用存在，程式碼就能執行。

下面，我們詳細分析這段程式碼的執行過程。

```javascript
// a.js

// 這一行建立一個引用，
// 從`b.js`引用`bar`
import {bar} from './b.js';

export function foo() {
  // 執行時第一行輸出 foo
  console.log('foo');
  // 到 b.js 執行 bar
  bar();
  console.log('執行完畢');
}
foo();

// b.js

// 建立`a.js`的`foo`引用
import {foo} from './a.js';

export function bar() {
  // 執行時，第二行輸出 bar
  console.log('bar');
  // 遞迴執行 foo，一旦隨機數
  // 小於等於0.5，就停止執行
  if (Math.random() > 0.5) {
    foo();
  }
}
```

我們再來看ES6模組載入器[SystemJS](https://github.com/ModuleLoader/es6-module-loader/blob/master/docs/circular-references-bindings.md)給出的一個例子。

```javascript
// even.js
import { odd } from './odd'
export var counter = 0;
export function even(n) {
  counter++;
  return n == 0 || odd(n - 1);
}

// odd.js
import { even } from './even';
export function odd(n) {
  return n != 0 && even(n - 1);
}
```

上面程式碼中，`even.js`裡面的函式`even`有一個引數`n`，只要不等於0，就會減去1，傳入載入的`odd()`。`odd.js`也會做類似操作。

執行上面這段程式碼，結果如下。

```javascript
$ babel-node
> import * as m from './even.js';
> m.even(10);
true
> m.counter
6
> m.even(20)
true
> m.counter
17
```

上面程式碼中，引數`n`從10變為0的過程中，`even()`一共會執行6次，所以變數`counter`等於6。第二次呼叫`even()`時，引數`n`從20變為0，`even()`一共會執行11次，加上前面的6次，所以變數`counter`等於17。

這個例子要是改寫成CommonJS，就根本無法執行，會報錯。

```javascript
// even.js
var odd = require('./odd');
var counter = 0;
exports.counter = counter;
exports.even = function(n) {
  counter++;
  return n == 0 || odd(n - 1);
}

// odd.js
var even = require('./even').even;
module.exports = function(n) {
  return n != 0 && even(n - 1);
}
```

上面程式碼中，`even.js`載入`odd.js`，而`odd.js`又去載入`even.js`，形成“迴圈載入”。這時，執行引擎就會輸出`even.js`已經執行的部分（不存在任何結果），所以在`odd.js`之中，變數`even`等於`null`，等到後面呼叫`even(n-1)`就會報錯。

```bash
$ node
> var m = require('./even');
> m.even(10)
TypeError: even is not a function
```

## ES6模組的轉碼

瀏覽器目前還不支援ES6模組，為了現在就能使用，可以將轉為ES5的寫法。除了Babel可以用來轉碼之外，還有以下兩個方法，也可以用來轉碼。

### ES6 module transpiler

[ES6 module transpiler](https://github.com/esnext/es6-module-transpiler)是 square 公司開源的一個轉碼器，可以將 ES6 模組轉為 CommonJS 模組或 AMD 模組的寫法，從而在瀏覽器中使用。

首先，安裝這個轉碼器。

```bash
$ npm install -g es6-module-transpiler
```

然後，使用`compile-modules convert`命令，將 ES6 模組檔案轉碼。

```bash
$ compile-modules convert file1.js file2.js
```

`-o`引數可以指定轉碼後的檔名。

```bash
$ compile-modules convert -o out.js file1.js
```

### SystemJS

另一種解決方法是使用 [SystemJS](https://github.com/systemjs/systemjs)。它是一個墊片庫（polyfill），可以在瀏覽器內載入 ES6 模組、AMD 模組和 CommonJS 模組，將其轉為 ES5 格式。它在後臺呼叫的是 Google 的 Traceur 轉碼器。

使用時，先在網頁內載入`system.js`檔案。

```html
<script src="system.js"></script>
```

然後，使用`System.import`方法載入模組檔案。

```html
<script>
  System.import('./app.js');
</script>
```

上面程式碼中的`./app`，指的是當前目錄下的app.js檔案。它可以是ES6模組檔案，`System.import`會自動將其轉碼。

需要注意的是，`System.import`使用非同步載入，返回一個 Promise 物件，可以針對這個物件程式設計。下面是一個模組檔案。

```javascript
// app/es6-file.js:

export class q {
  constructor() {
    this.es6 = 'hello';
  }
}
```

然後，在網頁內載入這個模組檔案。

```html
<script>

System.import('app/es6-file').then(function(m) {
  console.log(new m.q().es6); // hello
});

</script>
```

上面程式碼中，`System.import`方法返回的是一個 Promise 物件，所以可以用`then`方法指定回呼函式。

