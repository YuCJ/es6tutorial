# Module 的語法

## 概述

歷史上，JavaScript 一直沒有模組（module）體系，無法將一個大程式拆分成互相依賴的小檔案，再用簡單的方法拼裝起來。其他語言都有這項功能，比如 Ruby 的`require`、Python 的`import`，甚至就連 CSS 都有`@import`，但是 JavaScript 任何這方面的支援都沒有，這對開發大型的、複雜的專案形成了巨大障礙。

在 ES6 之前，社群制定了一些模組載入方案，最主要的有 CommonJS 和 AMD 兩種。前者用於伺服器，後者用於瀏覽器。ES6 在語言標準的層面上，實現了模組功能，而且實現得相當簡單，完全可以取代 CommonJS 和 AMD 規範，成為瀏覽器和伺服器通用的模組解決方案。

ES6 模組的設計思想，是儘量的靜態化，使得編譯時就能確定模組的依賴關係，以及輸入和輸出的變數。CommonJS 和 AMD 模組，都只能在執行時確定這些東西。比如，CommonJS 模組就是物件，輸入時必須查詢物件屬性。

```javascript
// CommonJS模組
let { stat, exists, readFile } = require('fs');

// 等同於
let _fs = require('fs');
let stat = _fs.stat;
let exists = _fs.exists;
let readfile = _fs.readfile;
```

上面程式碼的實質是整體載入`fs`模組（即載入`fs`的所有方法），生成一個物件（`_fs`），然後再從這個物件上面讀取3個方法。這種載入稱為“執行時載入”，因為只有執行時才能得到這個物件，導致完全沒辦法在編譯時做“靜態優化”。

ES6 模組不是物件，而是通過`export`命令顯式指定輸出的程式碼，再通過`import`命令輸入。

```javascript
// ES6模組
import { stat, exists, readFile } from 'fs';
```

上面程式碼的實質是從`fs`模組載入3個方法，其他方法不載入。這種載入稱為“編譯時載入”或者靜態載入，即 ES6 可以在編譯時就完成模組載入，效率要比 CommonJS 模組的載入方式高。當然，這也導致了沒法引用 ES6 模組本身，因為它不是物件。

由於 ES6 模組是編譯時載入，使得靜態分析成為可能。有了它，就能進一步拓寬 JavaScript 的語法，比如引入巨集（macro）和型別檢驗（type system）這些只能靠靜態分析實現的功能。

除了靜態載入帶來的各種好處，ES6 模組還有以下好處。

- 不再需要`UMD`模組格式了，將來伺服器和瀏覽器都會支援 ES6 模組格式。目前，通過各種工具庫，其實已經做到了這一點。
- 將來瀏覽器的新 API 就能用模組格式提供，不再必須做成全域性變數或者`navigator`物件的屬性。
- 不再需要物件作為名稱空間（比如`Math`物件），未來這些功能可以通過模組提供。

本章介紹 ES6 模組的語法，下一章介紹如何在瀏覽器和 Node 之中，載入 ES6 模組。

## 嚴格模式

ES6 的模組自動採用嚴格模式，不管你有沒有在模組頭部加上`"use strict";`。

嚴格模式主要有以下限制。

- 變數必須聲明後再使用
- 函式的引數不能有同名屬性，否則報錯
- 不能使用`with`語句
- 不能對只讀屬性賦值，否則報錯
- 不能使用字首0表示八進位制數，否則報錯
- 不能刪除不可刪除的屬性，否則報錯
- 不能刪除變數`delete prop`，會報錯，只能刪除屬性`delete global[prop]`
- `eval`不會在它的外層作用域引入變數
- `eval`和`arguments`不能被重新賦值
- `arguments`不會自動反映函式引數的變化
- 不能使用`arguments.callee`
- 不能使用`arguments.caller`
- 禁止`this`指向全域性物件
- 不能使用`fn.caller`和`fn.arguments`獲取函式呼叫的堆疊
- 增加了保留字（比如`protected`、`static`和`interface`）

上面這些限制，模組都必須遵守。由於嚴格模式是 ES5 引入的，不屬於 ES6，所以請參閱相關 ES5 書籍，本書不再詳細介紹了。

其中，尤其需要注意`this`的限制。ES6 模組之中，頂層的`this`指向`undefined`，即不應該在頂層程式碼使用`this`。

## export 命令

模組功能主要由兩個命令構成：`export`和`import`。`export`命令用於規定模組的對外介面，`import`命令用於輸入其他模組提供的功能。

一個模組就是一個獨立的檔案。該檔案內部的所有變數，外部無法獲取。如果你希望外部能夠讀取模組內部的某個變數，就必須使用`export`關鍵字輸出該變數。下面是一個 JS 檔案，裡面使用`export`命令輸出變數。

```javascript
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

上面程式碼是`profile.js`檔案，儲存了使用者資訊。ES6 將其視為一個模組，裡面用`export`命令對外部輸出了三個變數。

`export`的寫法，除了像上面這樣，還有另外一種。

```javascript
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export {firstName, lastName, year};
```

上面程式碼在`export`命令後面，使用大括號指定所要輸出的一組變數。它與前一種寫法（直接放置在`var`語句前）是等價的，但是應該優先考慮使用這種寫法。因為這樣就可以在指令碼尾部，一眼看清楚輸出了哪些變數。

`export`命令除了輸出變數，還可以輸出函式或類（class）。

```javascript
export function multiply(x, y) {
  return x * y;
};
```

上面程式碼對外輸出一個函式`multiply`。

通常情況下，`export`輸出的變數就是本來的名字，但是可以使用`as`關鍵字重新命名。

```javascript
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

上面程式碼使用`as`關鍵字，重新命名了函式`v1`和`v2`的對外介面。重新命名後，`v2`可以用不同的名字輸出兩次。

需要特別注意的是，`export`命令規定的是對外的介面，必須與模組內部的變數建立一一對應關係。

```javascript
// 報錯
export 1;

// 報錯
var m = 1;
export m;
```

上面兩種寫法都會報錯，因為沒有提供對外的介面。第一種寫法直接輸出1，第二種寫法通過變數`m`，還是直接輸出1。`1`只是一個值，不是介面。正確的寫法是下面這樣。

```javascript
// 寫法一
export var m = 1;

// 寫法二
var m = 1;
export {m};

// 寫法三
var n = 1;
export {n as m};
```

上面三種寫法都是正確的，規定了對外的介面`m`。其他指令碼可以通過這個介面，取到值`1`。它們的實質是，在介面名與模組內部變數之間，建立了一一對應的關係。

同樣的，`function`和`class`的輸出，也必須遵守這樣的寫法。

```javascript
// 報錯
function f() {}
export f;

// 正確
export function f() {};

// 正確
function f() {}
export {f};
```

另外，`export`語句輸出的介面，與其對應的值是動態繫結關係，即通過該介面，可以取到模組內部實時的值。

```javascript
export var foo = 'bar';
setTimeout(() => foo = 'baz', 500);
```

上面程式碼輸出變數`foo`，值為`bar`，500毫秒之後變成`baz`。

這一點與 CommonJS 規範完全不同。CommonJS 模組輸出的是值的快取，不存在動態更新，詳見下文《ES6模組載入的實質》一節。

最後，`export`命令可以出現在模組的任何位置，只要處於模組頂層就可以。如果處於塊級作用域內，就會報錯，下一節的`import`命令也是如此。這是因為處於條件程式碼塊之中，就沒法做靜態優化了，違背了ES6模組的設計初衷。

```javascript
function foo() {
  export default 'bar' // SyntaxError
}
foo()
```

上面程式碼中，`export`語句放在函式之中，結果報錯。

## import 命令

使用`export`命令定義了模組的對外介面以後，其他 JS 檔案就可以通過`import`命令載入這個模組。

```javascript
// main.js
import {firstName, lastName, year} from './profile';

function setName(element) {
  element.textContent = firstName + ' ' + lastName;
}
```

上面程式碼的`import`命令，用於載入`profile.js`檔案，並從中輸入變數。`import`命令接受一對大括號，裡面指定要從其他模組匯入的變數名。大括號裡面的變數名，必須與被匯入模組（`profile.js`）對外介面的名稱相同。

如果想為輸入的變數重新取一個名字，`import`命令要使用`as`關鍵字，將輸入的變數重新命名。

```javascript
import { lastName as surname } from './profile';
```

`import`後面的`from`指定模組檔案的位置，可以是相對路徑，也可以是絕對路徑，`.js`路徑可以省略。如果只是模組名，不帶有路徑，那麼必須有配置檔案，告訴 JavaScript 引擎該模組的位置。

```javascript
import {myMethod} from 'util';
```

上面程式碼中，`util`是模組檔名，由於不帶有路徑，必須通過配置，告訴引擎怎麼取到這個模組。

注意，`import`命令具有提升效果，會提升到整個模組的頭部，首先執行。

```javascript
foo();

import { foo } from 'my_module';
```

上面的程式碼不會報錯，因為`import`的執行早於`foo`的呼叫。這種行為的本質是，`import`命令是編譯階段執行的，在程式碼執行之前。

由於`import`是靜態執行，所以不能使用表示式和變數，這些只有在執行時才能得到結果的語法結構。

```javascript
// 報錯
import { 'f' + 'oo' } from 'my_module';

// 報錯
let module = 'my_module';
import { foo } from module;

// 報錯
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

上面三種寫法都會報錯，因為它們用到了表示式、變數和`if`結構。在靜態分析階段，這些語法都是沒法得到值的。

最後，`import`語句會執行所載入的模組，因此可以有下面的寫法。

```javascript
import 'lodash';
```

上面程式碼僅僅執行`lodash`模組，但是不輸入任何值。

如果多次重複執行同一句`import`語句，那麼只會執行一次，而不會執行多次。

```javascript
import 'lodash';
import 'lodash';
```

上面程式碼載入了兩次`lodash`，但是隻會執行一次。

```javascript
import { foo } from 'my_module';
import { bar } from 'my_module';

// 等同於
import { foo, bar } from 'my_module';
```

上面程式碼中，雖然`foo`和`bar`在兩個語句中載入，但是它們對應的是同一個`my_module`實例。也就是說，`import`語句是 Singleton 模式。

## 模組的整體載入

除了指定載入某個輸出值，還可以使用整體載入，即用星號（`*`）指定一個物件，所有輸出值都載入在這個物件上面。

下面是一個`circle.js`檔案，它輸出兩個方法`area`和`circumference`。

```javascript
// circle.js

export function area(radius) {
  return Math.PI * radius * radius;
}

export function circumference(radius) {
  return 2 * Math.PI * radius;
}
```

現在，載入這個模組。

```javascript
// main.js

import { area, circumference } from './circle';

console.log('圓面積：' + area(4));
console.log('圓周長：' + circumference(14));
```

上面寫法是逐一指定要載入的方法，整體載入的寫法如下。

```javascript
import * as circle from './circle';

console.log('圓面積：' + circle.area(4));
console.log('圓周長：' + circle.circumference(14));
```

注意，模組整體載入所在的那個物件（上例是`circle`），應該是可以靜態分析的，所以不允許執行時改變。下面的寫法都是不允許的。

```javascript
import * as circle from './circle';

// 下面兩行都是不允許的
circle.foo = 'hello';
circle.area = function () {};
```

## export default 命令

從前面的例子可以看出，使用`import`命令的時候，使用者需要知道所要載入的變數名或函式名，否則無法載入。但是，使用者肯定希望快速上手，未必願意閱讀文件，去了解模組有哪些屬性和方法。

為了給使用者提供方便，讓他們不用閱讀文件就能載入模組，就要用到`export default`命令，為模組指定預設輸出。

```javascript
// export-default.js
export default function () {
  console.log('foo');
}
```

上面程式碼是一個模組檔案`export-default.js`，它的預設輸出是一個函式。

其他模組載入該模組時，`import`命令可以為該匿名函式指定任意名字。

```javascript
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```

上面程式碼的`import`命令，可以用任意名稱指向`export-default.js`輸出的方法，這時就不需要知道原模組輸出的函式名。需要注意的是，這時`import`命令後面，不使用大括號。

`export default`命令用在非匿名函式前，也是可以的。

```javascript
// export-default.js
export default function foo() {
  console.log('foo');
}

// 或者寫成

function foo() {
  console.log('foo');
}

export default foo;
```

上面程式碼中，`foo`函式的函式名`foo`，在模組外部是無效的。載入的時候，視同匿名函式載入。

下面比較一下預設輸出和正常輸出。

```javascript
// 第一組
export default function crc32() { // 輸出
  // ...
}

import crc32 from 'crc32'; // 輸入

// 第二組
export function crc32() { // 輸出
  // ...
};

import {crc32} from 'crc32'; // 輸入
```

上面程式碼的兩組寫法，第一組是使用`export default`時，對應的`import`語句不需要使用大括號；第二組是不使用`export default`時，對應的`import`語句需要使用大括號。

`export default`命令用於指定模組的預設輸出。顯然，一個模組只能有一個預設輸出，因此`export default`命令只能使用一次。所以，`import`命令後面才不用加大括號，因為只可能對應一個方法。

本質上，`export default`就是輸出一個叫做`default`的變數或方法，然後系統允許你為它取任意名字。所以，下面的寫法是有效的。

```javascript
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同於
// export default add;

// app.js
import { default as xxx } from 'modules';
// 等同於
// import xxx from 'modules';
```

正是因為`export default`命令其實只是輸出一個叫做`default`的變數，所以它後面不能跟變數宣告語句。

```javascript
// 正確
export var a = 1;

// 正確
var a = 1;
export default a;

// 錯誤
export default var a = 1;
```

上面程式碼中，`export default a`的含義是將變數`a`的值賦給變數`default`。所以，最後一種寫法會報錯。

同樣地，因為`export default`本質是將該命令後面的值，賦給`default`變數以後再預設，所以直接將一個值寫在`export default`之後。

```javascript
// 正確
export default 42;

// 報錯
export 42;
```

上面程式碼中，後一句報錯是因為沒有指定對外的介面，而前一句指定外對介面為`default`。

有了`export default`命令，輸入模組時就非常直觀了，以輸入 lodash 模組為例。

```javascript
import _ from 'lodash';
```

如果想在一條`import`語句中，同時輸入預設方法和其他變數，可以寫成下面這樣。

```javascript
import _, { each } from 'lodash';
```

對應上面程式碼的`export`語句如下。

```javascript
export default function (obj) {
  // ···
}

export function each(obj, iterator, context) {
  // ···
}

export { each as forEach };
```

上面程式碼的最後一行的意思是，暴露出`forEach`介面，預設指向`each`介面，即`forEach`和`each`指向同一個方法。

`export default`也可以用來輸出類。

```javascript
// MyClass.js
export default class { ... }

// main.js
import MyClass from 'MyClass';
let o = new MyClass();
```

## export 與 import 的複合寫法

如果在一個模組之中，先輸入後輸出同一個模組，`import`語句可以與`export`語句寫在一起。

```javascript
export { foo, bar } from 'my_module';

// 等同於
import { foo, bar } from 'my_module';
export { foo, bar };
```

上面程式碼中，`export`和`import`語句可以結合在一起，寫成一行。

模組的介面改名和整體輸出，也可以採用這種寫法。

```javascript
// 介面改名
export { foo as myFoo } from 'my_module';

// 整體輸出
export * from 'my_module';
```

預設介面的寫法如下。

```javascript
export { default } from 'foo';
```

具名介面改為預設介面的寫法如下。

```javascript
export { es6 as default } from './someModule';

// 等同於
import { es6 } from './someModule';
export default es6;
```

同樣地，預設介面也可以改名為具名介面。

```javascript
export { default as es6 } from './someModule';
```

下面三種`import`語句，沒有對應的複合寫法。

```javascript
import * as someIdentifier from "someModule";
import someIdentifier from "someModule";
import someIdentifier, { namedIdentifier } from "someModule";
```

為了做到形式的對稱，現在有[提案](https://github.com/leebyron/ecmascript-export-default-from)，提出補上這三種複合寫法。

```javascript
export * as someIdentifier from "someModule";
export someIdentifier from "someModule";
export someIdentifier, { namedIdentifier } from "someModule";
```

## 模組的繼承

模組之間也可以繼承。

假設有一個`circleplus`模組，繼承了`circle`模組。

```javascript
// circleplus.js

export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
  return Math.exp(x);
}
```

上面程式碼中的`export *`，表示再輸出`circle`模組的所有屬性和方法。注意，`export *`命令會忽略`circle`模組的`default`方法。然後，上面程式碼又輸出了自定義的`e`變數和預設方法。

這時，也可以將`circle`的屬性或方法，改名後再輸出。

```javascript
// circleplus.js

export { area as circleArea } from 'circle';
```

上面程式碼表示，只輸出`circle`模組的`area`方法，且將其改名為`circleArea`。

載入上面模組的寫法如下。

```javascript
// main.js

import * as math from 'circleplus';
import exp from 'circleplus';
console.log(exp(math.e));
```

上面程式碼中的`import exp`表示，將`circleplus`模組的預設方法載入為`exp`方法。

## 跨模組常量

本書介紹`const`命令的時候說過，`const`宣告的常量只在當前程式碼塊有效。如果想設定跨模組的常量（即跨多個檔案），或者說一個值要被多個模組共享，可以採用下面的寫法。

```javascript
// constants.js 模組
export const A = 1;
export const B = 3;
export const C = 4;

// test1.js 模組
import * as constants from './constants';
console.log(constants.A); // 1
console.log(constants.B); // 3

// test2.js 模組
import {A, B} from './constants';
console.log(A); // 1
console.log(B); // 3
```

如果要使用的常量非常多，可以建一個專門的`constants`目錄，將各種常量寫在不同的檔案裡面，儲存在該目錄下。

```javascript
// constants/db.js
export const db = {
  url: 'http://my.couchdbserver.local:5984',
  admin_username: 'admin',
  admin_password: 'admin password'
};

// constants/user.js
export const users = ['root', 'admin', 'staff', 'ceo', 'chief', 'moderator'];
```

然後，將這些檔案輸出的常量，合併在`index.js`裡面。

```javascript
// constants/index.js
export {db} from './db';
export {users} from './users';
```

使用的時候，直接載入`index.js`就可以了。

```javascript
// script.js
import {db, users} from './constants';
```

## import()

### 簡介

前面介紹過，`import`命令會被 JavaScript 引擎靜態分析，先於模組內的其他模組執行（叫做”連線“更合適）。所以，下面的程式碼會報錯。

```javascript
// 報錯
if (x === 2) {
  import MyModual from './myModual';
}
```

上面程式碼中，引擎處理`import`語句是在編譯時，這時不會去分析或執行`if`語句，所以`import`語句放在`if`程式碼塊之中毫無意義，因此會報句法錯誤，而不是執行時錯誤。也就是說，`import`和`export`命令只能在模組的頂層，不能在程式碼塊之中（比如，在`if`程式碼塊之中，或在函式之中）。

這樣的設計，固然有利於編譯器提高效率，但也導致無法在執行時載入模組。從語法上，條件載入就不可能實現。如果`import`命令要取代 Node 的`require`方法，這就形成了一個障礙。因為`require`是執行時載入模組，`import`命令無法取代`require`的動態載入功能。

```javascript
const path = './' + fileName;
const myModual = require(path);
```

上面的語句就是動態載入，`require`到底載入哪一個模組，只有執行時才知道。`import`語句做不到這一點。

因此，有一個[提案](https://github.com/tc39/proposal-dynamic-import)，建議引入`import()`函式，完成動態載入。

```javascript
import(specifier)
```

上面程式碼中，`import`函式的引數`specifier`，指定所要載入的模組的位置。`import`命令能夠接受什麼引數，`import()`函式就能接受什麼引數，兩者區別主要是後者為動態載入。

`import()`返回一個 Promise 物件。下面是一個例子。

```javascript
const main = document.querySelector('main');

import(`./section-modules/${someVariable}.js`)
  .then(module => {
    module.loadPageInto(main);
  })
  .catch(err => {
    main.textContent = err.message;
  });
```

`import()`函式可以用在任何地方，不僅僅是模組，非模組的指令碼也可以使用。它是執行時執行，也就是說，什麼時候執行到這一句，也會載入指定的模組。另外，`import()`函式與所載入的模組沒有靜態連線關係，這點也是與`import`語句不相同。

`import()`類似於 Node 的`require`方法，區別主要是前者是非同步載入，後者是同步載入。

### 適用場合

下面是`import()`的一些適用場合。

（1）按需載入。

`import()`可以在需要的時候，再載入某個模組。

```javascript
button.addEventListener('click', event => {
  import('./dialogBox.js')
  .then(dialogBox => {
    dialogBox.open();
  })
  .catch(error => {
    /* Error handling */
  })
});
```

上面程式碼中，`import()`方法放在`click`事件的監聽函式之中，只有使用者點選了按鈕，才會載入這個模組。

（2）條件載入

`import()`可以放在`if`程式碼塊，根據不同的情況，載入不同的模組。

```javascript
if (condition) {
  import('moduleA').then(...);
} else {
  import('moduleB').then(...);
}
```

上面程式碼中，如果滿足條件，就載入模組 A，否則載入模組 B。

（3）動態的模組路徑

`import()`允許模組路徑動態生成。

```javascript
import(f())
.then(...);
```

上面程式碼中，根據函式`f`的返回結果，載入不同的模組。

### 注意點

`import()`載入模組成功以後，這個模組會作為一個物件，當作`then`方法的引數。因此，可以使用物件解構賦值的語法，獲取輸出介面。

```javascript
import('./myModule.js')
.then(({export1, export2}) => {
  // ...·
});
```

上面程式碼中，`export1`和`export2`都是`myModule.js`的輸出介面，可以解構獲得。

如果模組有`default`輸出介面，可以用引數直接獲得。

```javascript
import('./myModule.js')
.then(myModule => {
  console.log(myModule.default);
});
```

上面的程式碼也可以使用具名輸入的形式。

```javascript
import('./myModule.js')
.then(({default: theDefault}) => {
  console.log(theDefault);
});
```

如果想同時載入多個模組，可以採用下面的寫法。

```javascript
Promise.all([
  import('./module1.js'),
  import('./module2.js'),
  import('./module3.js'),
])
.then(([module1, module2, module3]) => {
   ···
});
```

`import()`也可以用在 async 函式之中。

```javascript
async function main() {
  const myModule = await import('./myModule.js');
  const {export1, export2} = await import('./myModule.js');
  const [module1, module2, module3] =
    await Promise.all([
      import('./module1.js'),
      import('./module2.js'),
      import('./module3.js'),
    ]);
}
main();
```

