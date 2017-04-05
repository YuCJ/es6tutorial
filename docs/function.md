# 函式的擴充套件

## 函式引數的預設值

### 基本用法

在ES6之前，不能直接為函式的引數指定預設值，只能採用變通的方法。

```javascript
function log(x, y) {
  y = y || 'World';
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello World
```

上面程式碼檢查函式`log`的引數`y`有沒有賦值，如果沒有，則指定預設值為`World`。這種寫法的缺點在於，如果引數`y`賦值了，但是對應的布林值為`false`，則該賦值不起作用。就像上面程式碼的最後一行，引數`y`等於空字元，結果被改為預設值。

為了避免這個問題，通常需要先判斷一下引數`y`是否被賦值，如果沒有，再等於預設值。

```javascript
if (typeof y === 'undefined') {
  y = 'World';
}
```

ES6 允許為函式的引數設定預設值，即直接寫在引數定義的後面。

```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```

可以看到，ES6 的寫法比 ES5 簡潔許多，而且非常自然。下面是另一個例子。

```javascript
function Point(x = 0, y = 0) {
  this.x = x;
  this.y = y;
}

var p = new Point();
p // { x: 0, y: 0 }
```

除了簡潔，ES6 的寫法還有兩個好處：首先，閱讀程式碼的人，可以立刻意識到哪些引數是可以省略的，不用檢視函式體或文件；其次，有利於將來的程式碼優化，即使未來的版本在對外介面中，徹底拿掉這個引數，也不會導致以前的程式碼無法執行。

引數變數是預設宣告的，所以不能用`let`或`const`再次宣告。

```javascript
function foo(x = 5) {
  let x = 1; // error
  const x = 2; // error
}
```

上面程式碼中，引數變數`x`是預設宣告的，在函式體中，不能用`let`或`const`再次宣告，否則會報錯。

使用引數預設值時，函式不能有同名引數。

```javascript
function foo(x, x, y = 1) {
  // ...
}
// SyntaxError: Duplicate parameter name not allowed in this context
```

另外，一個容易忽略的地方是，如果引數預設值是變數，那麼引數就不是傳值的，而是每次都重新計算預設值表示式的值。也就是說，引數預設值是惰性求值的。

```javascript
let x = 99;
function foo(p = x + 1) {
  console.log(p);
}

foo() // 100

x = 100;
foo() // 101
```

上面程式碼中，引數`p`的預設值是`x + 1`。這時，每次呼叫函式`foo`，都會重新計算`x + 1`，而不是預設`p`等於 100。

### 與解構賦值預設值結合使用

引數預設值可以與解構賦值的預設值，結合起來使用。

```javascript
function foo({x, y = 5}) {
  console.log(x, y);
}

foo({}) // undefined, 5
foo({x: 1}) // 1, 5
foo({x: 1, y: 2}) // 1, 2
foo() // TypeError: Cannot read property 'x' of undefined
```

上面程式碼使用了物件的解構賦值預設值，而沒有使用函式引數的預設值。只有當函式`foo`的引數是一個物件時，變數`x`和`y`才會通過解構賦值而生成。如果函式`foo`呼叫時引數不是物件，變數`x`和`y`就不會生成，從而報錯。如果引數物件沒有`y`屬性，`y`的預設值5才會生效。

下面是另一個物件的解構賦值預設值的例子。

```javascript
function fetch(url, { body = '', method = 'GET', headers = {} }) {
  console.log(method);
}

fetch('http://example.com', {})
// "GET"

fetch('http://example.com')
// 報錯
```

上面程式碼中，如果函式`fetch`的第二個引數是一個物件，就可以為它的三個屬性設定預設值。

上面的寫法不能省略第二個引數，如果結合函式引數的預設值，就可以省略第二個引數。這時，就出現了雙重預設值。

```javascript
function fetch(url, { method = 'GET' } = {}) {
  console.log(method);
}

fetch('http://example.com')
// "GET"
```

上面程式碼中，函式`fetch`沒有第二個引數時，函式引數的預設值就會生效，然後才是解構賦值的預設值生效，變數`method`才會取到預設值`GET`。

再請問下面兩種寫法有什麼差別？

```javascript
// 寫法一
function m1({x = 0, y = 0} = {}) {
  return [x, y];
}

// 寫法二
function m2({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}
```

上面兩種寫法都對函式的引數設定了預設值，區別是寫法一函式引數的預設值是空物件，但是設定了物件解構賦值的預設值；寫法二函式引數的預設值是一個有具體屬性的物件，但是沒有設定物件解構賦值的預設值。

```javascript
// 函式沒有引數的情況
m1() // [0, 0]
m2() // [0, 0]

// x和y都有值的情況
m1({x: 3, y: 8}) // [3, 8]
m2({x: 3, y: 8}) // [3, 8]

// x有值，y無值的情況
m1({x: 3}) // [3, 0]
m2({x: 3}) // [3, undefined]

// x和y都無值的情況
m1({}) // [0, 0];
m2({}) // [undefined, undefined]

m1({z: 3}) // [0, 0]
m2({z: 3}) // [undefined, undefined]
```

### 引數預設值的位置

通常情況下，定義了預設值的引數，應該是函式的尾引數。因為這樣比較容易看出來，到底省略了哪些引數。如果非尾部的引數設定預設值，實際上這個引數是沒法省略的。

```javascript
// 例一
function f(x = 1, y) {
  return [x, y];
}

f() // [1, undefined]
f(2) // [2, undefined])
f(, 1) // 報錯
f(undefined, 1) // [1, 1]

// 例二
function f(x, y = 5, z) {
  return [x, y, z];
}

f() // [undefined, 5, undefined]
f(1) // [1, 5, undefined]
f(1, ,2) // 報錯
f(1, undefined, 2) // [1, 5, 2]
```

上面程式碼中，有預設值的引數都不是尾引數。這時，無法只省略該引數，而不省略它後面的引數，除非顯式輸入`undefined`。

如果傳入`undefined`，將觸發該引數等於預設值，`null`則沒有這個效果。

```javascript
function foo(x = 5, y = 6) {
  console.log(x, y);
}

foo(undefined, null)
// 5 null
```

上面程式碼中，`x`引數對應`undefined`，結果觸發了預設值，`y`引數等於`null`，就沒有觸發預設值。

### 函式的 length 屬性

指定了預設值以後，函式的`length`屬性，將返回沒有指定預設值的引數個數。也就是說，指定了預設值後，`length`屬性將失真。

```javascript
(function (a) {}).length // 1
(function (a = 5) {}).length // 0
(function (a, b, c = 5) {}).length // 2
```

上面程式碼中，`length`屬性的返回值，等於函式的引數個數減去指定了預設值的引數個數。比如，上面最後一個函式，定義了3個引數，其中有一個引數`c`指定了預設值，因此`length`屬性等於`3`減去`1`，最後得到`2`。

這是因為`length`屬性的含義是，該函式預期傳入的引數個數。某個引數指定預設值以後，預期傳入的引數個數就不包括這個引數了。同理，rest引數也不會計入`length`屬性。

```javascript
(function(...args) {}).length // 0
```

如果設定了預設值的引數不是尾引數，那麼`length`屬性也不再計入後面的引數了。

```javascript
(function (a = 0, b, c) {}).length // 0
(function (a, b = 1, c) {}).length // 1
```

### 作用域

一旦設定了引數的預設值，函式進行宣告初始化時，引數會形成一個單獨的作用域（context）。等到初始化結束，這個作用域就會消失。這種語法行為，在不設定引數預設值時，是不會出現的。

```javascript
var x = 1;

function f(x, y = x) {
  console.log(y);
}

f(2) // 2
```

上面程式碼中，引數`y`的預設值等於變數`x`。呼叫函式`f`時，引數形成一個單獨的作用域。在這個作用域裡面，預設值變數`x`指向第一個引數`x`，而不是全域性變數`x`，所以輸出是`2`。

再看下面的例子。

```javascript
let x = 1;

function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // 1
```

上面程式碼中，函式`f`呼叫時，引數`y = x`形成一個單獨的作用域。這個作用域裡面，變數`x`本身沒有定義，所以指向外層的全域性變數`x`。函式呼叫時，函式體內部的區域性變數`x`影響不到預設值變數`x`。

如果此時，全域性變數`x`不存在，就會報錯。

```javascript
function f(y = x) {
  let x = 2;
  console.log(y);
}

f() // ReferenceError: x is not defined
```

下面這樣寫，也會報錯。

```javascript
var x = 1;

function foo(x = x) {
  // ...
}

foo() // ReferenceError: x is not defined
```

上面程式碼中，引數`x = x`形成一個單獨作用域。實際執行的是`let x = x`，由於暫時性死區的原因，這行程式碼會報錯”x 未定義“。

如果引數的預設值是一個函式，該函式的作用域也遵守這個規則。請看下面的例子。

```javascript
let foo = 'outer';

function bar(func = x => foo) {
  let foo = 'inner';
  console.log(func()); // outer
}

bar();
```

上面程式碼中，函式`bar`的引數`func`的預設值是一個匿名函式，返回值為變數`foo`。函式引數形成的單獨作用域裡面，並沒有定義變數`foo`，所以`foo`指向外層的全域性變數`foo`，因此輸出`outer`。

如果寫成下面這樣，就會報錯。

```javascript
function bar(func = () => foo) {
  let foo = 'inner';
  console.log(func());
}

bar() // ReferenceError: foo is not defined
```

上面程式碼中，匿名函式裡面的`foo`指向函式外層，但是函式外層並沒有宣告變數`foo`，所以就報錯了。

下面是一個更復雜的例子。

```javascript
var x = 1;
function foo(x, y = function() { x = 2; }) {
  var x = 3;
  y();
  console.log(x);
}

foo() // 3
x // 1
```

上面程式碼中，函式`foo`的引數形成一個單獨作用域。這個作用域裡面，首先聲明瞭變數`x`，然後聲明瞭變數`y`，`y`的預設值是一個匿名函式。這個匿名函式內部的變數`x`，指向同一個作用域的第一個引數`x`。函式`foo`內部又聲明瞭一個內部變數`x`，該變數與第一個引數`x`由於不是同一個作用域，所以不是同一個變數，因此執行`y`後，內部變數`x`和外部全域性變數`x`的值都沒變。

如果將`var x = 3`的`var`去除，函式`foo`的內部變數`x`就指向第一個引數`x`，與匿名函式內部的`x`是一致的，所以最後輸出的就是`2`，而外層的全域性變數`x`依然不受影響。

```javascript
var x = 1;
function foo(x, y = function() { x = 2; }) {
  x = 3;
  y();
  console.log(x);
}

foo() // 2
x // 1
```

### 應用

利用引數預設值，可以指定某一個引數不得省略，如果省略就丟擲一個錯誤。

```javascript
function throwIfMissing() {
  throw new Error('Missing parameter');
}

function foo(mustBeProvided = throwIfMissing()) {
  return mustBeProvided;
}

foo()
// Error: Missing parameter
```

上面程式碼的`foo`函式，如果呼叫的時候沒有引數，就會呼叫預設值`throwIfMissing`函式，從而丟擲一個錯誤。

從上面程式碼還可以看到，引數`mustBeProvided`的預設值等於`throwIfMissing`函式的執行結果（即函式名之後有一對圓括號），這表明引數的預設值不是在定義時執行，而是在執行時執行（即如果引數已經賦值，預設值中的函式就不會執行），這與 Python 語言不一樣。

另外，可以將引數預設值設為`undefined`，表明這個引數是可以省略的。

```javascript
function foo(optional = undefined) { ··· }
```

## rest引數

ES6 引入 rest 引數（形式為“...變數名”），用於獲取函式的多餘引數，這樣就不需要使用`arguments`物件了。rest 引數搭配的變數是一個數組，該變數將多餘的引數放入陣列中。

```javascript
function add(...values) {
  let sum = 0;

  for (var val of values) {
    sum += val;
  }

  return sum;
}

add(2, 5, 3) // 10
```

上面程式碼的`add`函式是一個求和函式，利用 rest 引數，可以向該函式傳入任意數目的引數。

下面是一個 rest 引數代替`arguments`變數的例子。

```javascript
// arguments變數的寫法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest引數的寫法
const sortNumbers = (...numbers) => numbers.sort();
```

上面程式碼的兩種寫法，比較後可以發現，rest 引數的寫法更自然也更簡潔。

rest 引數中的變數代表一個數組，所以陣列特有的方法都可以用於這個變數。下面是一個利用 rest 引數改寫陣列`push`方法的例子。

```javascript
function push(array, ...items) {
  items.forEach(function(item) {
    array.push(item);
    console.log(item);
  });
}

var a = [];
push(a, 1, 2, 3)
```

注意，rest 引數之後不能再有其他引數（即只能是最後一個引數），否則會報錯。

```javascript
// 報錯
function f(a, ...b, c) {
  // ...
}
```

函式的`length`屬性，不包括 rest 引數。

```javascript
(function(a) {}).length  // 1
(function(...a) {}).length  // 0
(function(a, ...b) {}).length  // 1
```

## 擴充套件運算子

### 含義

擴充套件運算子（spread）是三個點（`...`）。它好比 rest 引數的逆運算，將一個數組轉為用逗號分隔的引數序列。

```javascript
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]
```

該運算子主要用於函式呼叫。

```javascript
function push(array, ...items) {
  array.push(...items);
}

function add(x, y) {
  return x + y;
}

var numbers = [4, 38];
add(...numbers) // 42
```

上面程式碼中，`array.push(...items)`和`add(...numbers)`這兩行，都是函式的呼叫，它們的都使用了擴充套件運算子。該運算子將一個數組，變為引數序列。

擴充套件運算子與正常的函式引數可以結合使用，非常靈活。

```javascript
function f(v, w, x, y, z) { }
var args = [0, 1];
f(-1, ...args, 2, ...[3]);
```

### 替代陣列的apply方法

由於擴充套件運算子可以展開陣列，所以不再需要`apply`方法，將陣列轉為函式的引數了。

```javascript
// ES5的寫法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f.apply(null, args);

// ES6的寫法
function f(x, y, z) {
  // ...
}
var args = [0, 1, 2];
f(...args);
```

下面是擴充套件運算子取代`apply`方法的一個實際的例子，應用`Math.max`方法，簡化求出一個數組最大元素的寫法。

```javascript
// ES5的寫法
Math.max.apply(null, [14, 3, 77])

// ES6的寫法
Math.max(...[14, 3, 77])

// 等同於
Math.max(14, 3, 77);
```

上面程式碼表示，由於JavaScript不提供求陣列最大元素的函式，所以只能套用`Math.max`函式，將陣列轉為一個引數序列，然後求最大值。有了擴充套件運算子以後，就可以直接用`Math.max`了。

另一個例子是通過`push`函式，將一個數組新增到另一個數組的尾部。

```javascript
// ES5的寫法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
Array.prototype.push.apply(arr1, arr2);

// ES6的寫法
var arr1 = [0, 1, 2];
var arr2 = [3, 4, 5];
arr1.push(...arr2);
```

上面程式碼的ES5寫法中，`push`方法的引數不能是陣列，所以只好通過`apply`方法變通使用`push`方法。有了擴充套件運算子，就可以直接將陣列傳入`push`方法。

下面是另外一個例子。

```javascript
// ES5
new (Date.bind.apply(Date, [null, 2015, 1, 1]))
// ES6
new Date(...[2015, 1, 1]);
```

### 擴充套件運算子的應用

**（1）合併陣列**

擴充套件運算子提供了數組合並的新寫法。

```javascript
// ES5
[1, 2].concat(more)
// ES6
[1, 2, ...more]

var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['d', 'e'];

// ES5的合併陣列
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]

// ES6的合併陣列
[...arr1, ...arr2, ...arr3]
// [ 'a', 'b', 'c', 'd', 'e' ]
```

**（2）與解構賦值結合**

擴充套件運算子可以與解構賦值結合起來，用於生成陣列。

```javascript
// ES5
a = list[0], rest = list.slice(1)
// ES6
[a, ...rest] = list
```

下面是另外一些例子。

```javascript
const [first, ...rest] = [1, 2, 3, 4, 5];
first // 1
rest  // [2, 3, 4, 5]

const [first, ...rest] = [];
first // undefined
rest  // []:

const [first, ...rest] = ["foo"];
first  // "foo"
rest   // []
```

如果將擴充套件運算子用於陣列賦值，只能放在引數的最後一位，否則會報錯。

```javascript
const [...butLast, last] = [1, 2, 3, 4, 5];
// 報錯

const [first, ...middle, last] = [1, 2, 3, 4, 5];
// 報錯
```

**（3）函式的返回值**

JavaScript的函式只能返回一個值，如果需要返回多個值，只能返回陣列或物件。擴充套件運算子提供瞭解決這個問題的一種變通方法。

```javascript
var dateFields = readDateFields(database);
var d = new Date(...dateFields);
```

上面程式碼從資料庫取出一行資料，通過擴充套件運算子，直接將其傳入建構函式`Date`。

**（4）字串**

擴充套件運算子還可以將字串轉為真正的陣列。

```javascript
[...'hello']
// [ "h", "e", "l", "l", "o" ]
```

上面的寫法，有一個重要的好處，那就是能夠正確識別32位的Unicode字元。

```javascript
'x\uD83D\uDE80y'.length // 4
[...'x\uD83D\uDE80y'].length // 3
```

上面程式碼的第一種寫法，JavaScript會將32位Unicode字元，識別為2個字元，採用擴充套件運算子就沒有這個問題。因此，正確返回字串長度的函式，可以像下面這樣寫。

```javascript
function length(str) {
  return [...str].length;
}

length('x\uD83D\uDE80y') // 3
```

凡是涉及到操作32位Unicode字元的函式，都有這個問題。因此，最好都用擴充套件運算子改寫。

```javascript
let str = 'x\uD83D\uDE80y';

str.split('').reverse().join('')
// 'y\uDE80\uD83Dx'

[...str].reverse().join('')
// 'y\uD83D\uDE80x'
```

上面程式碼中，如果不用擴充套件運算子，字串的`reverse`操作就不正確。

**（5）實現了Iterator介面的物件**

任何Iterator介面的物件，都可以用擴充套件運算子轉為真正的陣列。

```javascript
var nodeList = document.querySelectorAll('div');
var array = [...nodeList];
```

上面程式碼中，`querySelectorAll`方法返回的是一個`nodeList`物件。它不是陣列，而是一個類似陣列的物件。這時，擴充套件運算子可以將其轉為真正的陣列，原因就在於`NodeList`物件實現了Iterator介面。

對於那些沒有部署Iterator介面的類似陣列的物件，擴充套件運算子就無法將其轉為真正的陣列。

```javascript
let arrayLike = {
  '0': 'a',
  '1': 'b',
  '2': 'c',
  length: 3
};

// TypeError: Cannot spread non-iterable object.
let arr = [...arrayLike];
```

上面程式碼中，`arrayLike`是一個類似陣列的物件，但是沒有部署Iterator介面，擴充套件運算子就會報錯。這時，可以改為使用`Array.from`方法將`arrayLike`轉為真正的陣列。

**（6）Map和Set結構，Generator函式**

擴充套件運算子內部呼叫的是資料結構的Iterator介面，因此只要具有Iterator介面的物件，都可以使用擴充套件運算子，比如Map結構。

```javascript
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

let arr = [...map.keys()]; // [1, 2, 3]
```

Generator函式執行後，返回一個遍歷器物件，因此也可以使用擴充套件運算子。

```javascript
var go = function*(){
  yield 1;
  yield 2;
  yield 3;
};

[...go()] // [1, 2, 3]
```

上面程式碼中，變數`go`是一個Generator函式，執行後返回的是一個遍歷器物件，對這個遍歷器物件執行擴充套件運算子，就會將內部遍歷得到的值，轉為一個數組。

如果對沒有`iterator`介面的物件，使用擴充套件運算子，將會報錯。

```javascript
var obj = {a: 1, b: 2};
let arr = [...obj]; // TypeError: Cannot spread non-iterable object
```

## 嚴格模式

從ES5開始，函式內部可以設定為嚴格模式。

```javascript
function doSomething(a, b) {
  'use strict';
  // code
}
```

《ECMAScript 2016標準》做了一點修改，規定只要函式引數使用了預設值、解構賦值、或者擴充套件運算子，那麼函式內部就不能顯式設定為嚴格模式，否則會報錯。

```javascript
// 報錯
function doSomething(a, b = a) {
  'use strict';
  // code
}

// 報錯
const doSomething = function ({a, b}) {
  'use strict';
  // code
};

// 報錯
const doSomething = (...a) => {
  'use strict';
  // code
};

const obj = {
  // 報錯
  doSomething({a, b}) {
    'use strict';
    // code
  }
};
```

這樣規定的原因是，函式內部的嚴格模式，同時適用於函式體程式碼和函式引數程式碼。但是，函式執行的時候，先執行函式引數程式碼，然後再執行函式體程式碼。這樣就有一個不合理的地方，只有從函式體程式碼之中，才能知道引數程式碼是否應該以嚴格模式執行，但是引數程式碼卻應該先於函式體程式碼執行。

```javascript
// 報錯
function doSomething(value = 070) {
  'use strict';
  return value;
}
```

上面程式碼中，引數`value`的預設值是八進位制數`070`，但是嚴格模式下不能用字首`0`表示八進位制，所以應該報錯。但是實際上，JavaScript引擎會先成功執行`value = 070`，然後進入函式體內部，發現需要用嚴格模式執行，這時才會報錯。

雖然可以先解析函式體程式碼，再執行引數程式碼，但是這樣無疑就增加了複雜性。因此，標準索性禁止了這種用法，只要引數使用了預設值、解構賦值、或者擴充套件運算子，就不能顯式指定嚴格模式。

兩種方法可以規避這種限制。第一種是設定全域性性的嚴格模式，這是合法的。

```javascript
'use strict';

function doSomething(a, b = a) {
  // code
}
```

第二種是把函式包在一個無引數的立即執行函式裡面。

```javascript
const doSomething = (function () {
  'use strict';
  return function(value = 42) {
    return value;
  };
}());
```

## name 屬性

函式的`name`屬性，返回該函式的函式名。

```javascript
function foo() {}
foo.name // "foo"
```

這個屬性早就被瀏覽器廣泛支援，但是直到 ES6，才將其寫入了標準。

需要注意的是，ES6 對這個屬性的行為做出了一些修改。如果將一個匿名函式賦值給一個變數，ES5 的`name`屬性，會返回空字串，而 ES6 的`name`屬性會返回實際的函式名。

```javascript
var f = function () {};

// ES5
f.name // ""

// ES6
f.name // "f"
```

上面程式碼中，變數`f`等於一個匿名函式，ES5 和 ES6 的`name`屬性返回的值不一樣。

如果將一個具名函式賦值給一個變數，則 ES5 和 ES6 的`name`屬性都返回這個具名函式原本的名字。

```javascript
const bar = function baz() {};

// ES5
bar.name // "baz"

// ES6
bar.name // "baz"
```

`Function`建構函式返回的函式例項，`name`屬性的值為`anonymous`。

```javascript
(new Function).name // "anonymous"
```

`bind`返回的函式，`name`屬性值會加上`bound `字首。

```javascript
function foo() {};
foo.bind({}).name // "bound foo"

(function(){}).bind({}).name // "bound "
```

## 箭頭函式

### 基本用法

ES6允許使用“箭頭”（`=>`）定義函式。

```javascript
var f = v => v;
```

上面的箭頭函式等同於：

```javascript
var f = function(v) {
  return v;
};
```

如果箭頭函式不需要引數或需要多個引數，就使用一個圓括號代表引數部分。

```javascript
var f = () => 5;
// 等同於
var f = function () { return 5 };

var sum = (num1, num2) => num1 + num2;
// 等同於
var sum = function(num1, num2) {
  return num1 + num2;
};
```

如果箭頭函式的程式碼塊部分多於一條語句，就要使用大括號將它們括起來，並且使用`return`語句返回。

```javascript
var sum = (num1, num2) => { return num1 + num2; }
```

由於大括號被解釋為程式碼塊，所以如果箭頭函式直接返回一個物件，必須在物件外面加上括號。

```javascript
var getTempItem = id => ({ id: id, name: "Temp" });
```

箭頭函式可以與變數解構結合使用。

```javascript
const full = ({ first, last }) => first + ' ' + last;

// 等同於
function full(person) {
  return person.first + ' ' + person.last;
}
```

箭頭函式使得表達更加簡潔。

```javascript
const isEven = n => n % 2 == 0;
const square = n => n * n;
```

上面程式碼只用了兩行，就定義了兩個簡單的工具函式。如果不用箭頭函式，可能就要佔用多行，而且還不如現在這樣寫醒目。

箭頭函式的一個用處是簡化回撥函式。

```javascript
// 正常函式寫法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭頭函式寫法
[1,2,3].map(x => x * x);
```

另一個例子是

```javascript
// 正常函式寫法
var result = values.sort(function (a, b) {
  return a - b;
});

// 箭頭函式寫法
var result = values.sort((a, b) => a - b);
```

下面是rest引數與箭頭函式結合的例子。

```javascript
const numbers = (...nums) => nums;

numbers(1, 2, 3, 4, 5)
// [1,2,3,4,5]

const headAndTail = (head, ...tail) => [head, tail];

headAndTail(1, 2, 3, 4, 5)
// [1,[2,3,4,5]]
```

### 使用注意點

箭頭函式有幾個使用注意點。

（1）函式體內的`this`物件，就是定義時所在的物件，而不是使用時所在的物件。

（2）不可以當作建構函式，也就是說，不可以使用`new`命令，否則會丟擲一個錯誤。

（3）不可以使用`arguments`物件，該物件在函式體內不存在。如果要用，可以用Rest引數代替。

（4）不可以使用`yield`命令，因此箭頭函式不能用作Generator函式。

上面四點中，第一點尤其值得注意。`this`物件的指向是可變的，但是在箭頭函式中，它是固定的。

```javascript
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
// id: 42
```

上面程式碼中，`setTimeout`的引數是一個箭頭函式，這個箭頭函式的定義生效是在`foo`函式生成時，而它的真正執行要等到100毫秒後。如果是普通函式，執行時`this`應該指向全域性物件`window`，這時應該輸出`21`。但是，箭頭函式導致`this`總是指向函式定義生效時所在的物件（本例是`{id: 42}`），所以輸出的是`42`。

箭頭函式可以讓`setTimeout`裡面的`this`，繫結定義時所在的作用域，而不是指向執行時所在的作用域。下面是另一個例子。

```javascript
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭頭函式
  setInterval(() => this.s1++, 1000);
  // 普通函式
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();

setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2), 3100);
// s1: 3
// s2: 0
```

上面程式碼中，`Timer`函式內部設定了兩個定時器，分別使用了箭頭函式和普通函式。前者的`this`繫結定義時所在的作用域（即`Timer`函式），後者的`this`指向執行時所在的作用域（即全域性物件）。所以，3100毫秒之後，`timer.s1`被更新了3次，而`timer.s2`一次都沒更新。

箭頭函式可以讓`this`指向固定化，這種特性很有利於封裝回調函式。下面是一個例子，DOM事件的回撥函式封裝在一個物件裡面。

```javascript
var handler = {
  id: '123456',

  init: function() {
    document.addEventListener('click',
      event => this.doSomething(event.type), false);
  },

  doSomething: function(type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
};
```

上面程式碼的`init`方法中，使用了箭頭函式，這導致這個箭頭函式裡面的`this`，總是指向`handler`物件。否則，回撥函式執行時，`this.doSomething`這一行會報錯，因為此時`this`指向`document`物件。

`this`指向的固定化，並不是因為箭頭函式內部有繫結`this`的機制，實際原因是箭頭函式根本沒有自己的`this`，導致內部的`this`就是外層程式碼塊的`this`。正是因為它沒有`this`，所以也就不能用作建構函式。

所以，箭頭函式轉成ES5的程式碼如下。

```javascript
// ES6
function foo() {
  setTimeout(() => {
    console.log('id:', this.id);
  }, 100);
}

// ES5
function foo() {
  var _this = this;

  setTimeout(function () {
    console.log('id:', _this.id);
  }, 100);
}
```

上面程式碼中，轉換後的ES5版本清楚地說明了，箭頭函式裡面根本沒有自己的`this`，而是引用外層的`this`。

請問下面的程式碼之中有幾個`this`？

```javascript
function foo() {
  return () => {
    return () => {
      return () => {
        console.log('id:', this.id);
      };
    };
  };
}

var f = foo.call({id: 1});

var t1 = f.call({id: 2})()(); // id: 1
var t2 = f().call({id: 3})(); // id: 1
var t3 = f()().call({id: 4}); // id: 1
```

上面程式碼之中，只有一個`this`，就是函式`foo`的`this`，所以`t1`、`t2`、`t3`都輸出同樣的結果。因為所有的內層函式都是箭頭函式，都沒有自己的`this`，它們的`this`其實都是最外層`foo`函式的`this`。

除了`this`，以下三個變數在箭頭函式之中也是不存在的，指向外層函式的對應變數：`arguments`、`super`、`new.target`。

```javascript
function foo() {
  setTimeout(() => {
    console.log('args:', arguments);
  }, 100);
}

foo(2, 4, 6, 8)
// args: [2, 4, 6, 8]
```

上面程式碼中，箭頭函式內部的變數`arguments`，其實是函式`foo`的`arguments`變數。

另外，由於箭頭函式沒有自己的`this`，所以當然也就不能用`call()`、`apply()`、`bind()`這些方法去改變`this`的指向。

```javascript
(function() {
  return [
    (() => this.x).bind({ x: 'inner' })()
  ];
}).call({ x: 'outer' });
// ['outer']
```

上面程式碼中，箭頭函式沒有自己的`this`，所以`bind`方法無效，內部的`this`指向外部的`this`。

長期以來，JavaScript語言的`this`物件一直是一個令人頭痛的問題，在物件方法中使用`this`，必須非常小心。箭頭函式”繫結”`this`，很大程度上解決了這個困擾。

### 巢狀的箭頭函式

箭頭函式內部，還可以再使用箭頭函式。下面是一個ES5語法的多重巢狀函式。

```javascript
function insert(value) {
  return {into: function (array) {
    return {after: function (afterValue) {
      array.splice(array.indexOf(afterValue) + 1, 0, value);
      return array;
    }};
  }};
}

insert(2).into([1, 3]).after(1); //[1, 2, 3]
```

上面這個函式，可以使用箭頭函式改寫。

```javascript
let insert = (value) => ({into: (array) => ({after: (afterValue) => {
  array.splice(array.indexOf(afterValue) + 1, 0, value);
  return array;
}})});

insert(2).into([1, 3]).after(1); //[1, 2, 3]
```

下面是一個部署管道機制（pipeline）的例子，即前一個函式的輸出是後一個函式的輸入。

```javascript
const pipeline = (...funcs) =>
  val => funcs.reduce((a, b) => b(a), val);

const plus1 = a => a + 1;
const mult2 = a => a * 2;
const addThenMult = pipeline(plus1, mult2);

addThenMult(5)
// 12
```

如果覺得上面的寫法可讀性比較差，也可以採用下面的寫法。

```javascript
const plus1 = a => a + 1;
const mult2 = a => a * 2;

mult2(plus1(5))
// 12
```

箭頭函式還有一個功能，就是可以很方便地改寫λ演算。

```javascript
// λ演算的寫法
fix = λf.(λx.f(λv.x(x)(v)))(λx.f(λv.x(x)(v)))

// ES6的寫法
var fix = f => (x => f(v => x(x)(v)))
               (x => f(v => x(x)(v)));
```

上面兩種寫法，幾乎是一一對應的。由於λ演算對於電腦科學非常重要，這使得我們可以用ES6作為替代工具，探索電腦科學。

## 繫結 this

箭頭函式可以繫結`this`物件，大大減少了顯式繫結`this`物件的寫法（`call`、`apply`、`bind`）。但是，箭頭函式並不適用於所有場合，所以ES7提出了“函式繫結”（function bind）運算子，用來取代`call`、`apply`、`bind`呼叫。雖然該語法還是ES7的一個[提案](https://github.com/zenparsing/es-function-bind)，但是Babel轉碼器已經支援。

函式繫結運算子是並排的兩個雙冒號（::），雙冒號左邊是一個物件，右邊是一個函式。該運算子會自動將左邊的物件，作為上下文環境（即this物件），繫結到右邊的函式上面。

```javascript
foo::bar;
// 等同於
bar.bind(foo);

foo::bar(...arguments);
// 等同於
bar.apply(foo, arguments);

const hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return obj::hasOwnProperty(key);
}
```

如果雙冒號左邊為空，右邊是一個物件的方法，則等於將該方法繫結在該物件上面。

```javascript
var method = obj::obj.foo;
// 等同於
var method = ::obj.foo;

let log = ::console.log;
// 等同於
var log = console.log.bind(console);
```

由於雙冒號運算子返回的還是原物件，因此可以採用鏈式寫法。

```javascript
// 例一
import { map, takeWhile, forEach } from "iterlib";

getPlayers()
::map(x => x.character())
::takeWhile(x => x.strength > 100)
::forEach(x => console.log(x));

// 例二
let { find, html } = jake;

document.querySelectorAll("div.myClass")
::find("p")
::html("hahaha");
```

## 尾呼叫優化

### 什麼是尾呼叫？

尾呼叫（Tail Call）是函數語言程式設計的一個重要概念，本身非常簡單，一句話就能說清楚，就是指某個函式的最後一步是呼叫另一個函式。

```javascript
function f(x){
  return g(x);
}
```

上面程式碼中，函式f的最後一步是呼叫函式g，這就叫尾呼叫。

以下三種情況，都不屬於尾呼叫。

```javascript
// 情況一
function f(x){
  let y = g(x);
  return y;
}

// 情況二
function f(x){
  return g(x) + 1;
}

// 情況三
function f(x){
  g(x);
}
```

上面程式碼中，情況一是呼叫函式g之後，還有賦值操作，所以不屬於尾呼叫，即使語義完全一樣。情況二也屬於呼叫後還有操作，即使寫在一行內。情況三等同於下面的程式碼。

```javascript
function f(x){
  g(x);
  return undefined;
}
```

尾呼叫不一定出現在函式尾部，只要是最後一步操作即可。

```javascript
function f(x) {
  if (x > 0) {
    return m(x)
  }
  return n(x);
}
```

上面程式碼中，函式m和n都屬於尾呼叫，因為它們都是函式f的最後一步操作。

### 尾呼叫優化

尾呼叫之所以與其他呼叫不同，就在於它的特殊的呼叫位置。

我們知道，函式呼叫會在記憶體形成一個“呼叫記錄”，又稱“呼叫幀”（call frame），儲存呼叫位置和內部變數等資訊。如果在函式A的內部呼叫函式B，那麼在A的呼叫幀上方，還會形成一個B的呼叫幀。等到B執行結束，將結果返回到A，B的呼叫幀才會消失。如果函式B內部還呼叫函式C，那就還有一個C的呼叫幀，以此類推。所有的呼叫幀，就形成一個“呼叫棧”（call stack）。

尾呼叫由於是函式的最後一步操作，所以不需要保留外層函式的呼叫幀，因為呼叫位置、內部變數等資訊都不會再用到了，只要直接用內層函式的呼叫幀，取代外層函式的呼叫幀就可以了。

```javascript
function f() {
  let m = 1;
  let n = 2;
  return g(m + n);
}
f();

// 等同於
function f() {
  return g(3);
}
f();

// 等同於
g(3);
```

上面程式碼中，如果函式g不是尾呼叫，函式f就需要儲存內部變數m和n的值、g的呼叫位置等資訊。但由於呼叫g之後，函式f就結束了，所以執行到最後一步，完全可以刪除 f(x) 的呼叫幀，只保留 g(3) 的呼叫幀。

這就叫做“尾呼叫優化”（Tail call optimization），即只保留內層函式的呼叫幀。如果所有函式都是尾呼叫，那麼完全可以做到每次執行時，呼叫幀只有一項，這將大大節省記憶體。這就是“尾呼叫優化”的意義。

注意，只有不再用到外層函式的內部變數，內層函式的呼叫幀才會取代外層函式的呼叫幀，否則就無法進行“尾呼叫優化”。

```javascript
function addOne(a){
  var one = 1;
  function inner(b){
    return b + one;
  }
  return inner(a);
}
```

上面的函式不會進行尾呼叫優化，因為內層函式`inner`用到了外層函式`addOne`的內部變數`one`。

### 尾遞迴

函式呼叫自身，稱為遞迴。如果尾呼叫自身，就稱為尾遞迴。

遞迴非常耗費記憶體，因為需要同時儲存成千上百個呼叫幀，很容易發生“棧溢位”錯誤（stack overflow）。但對於尾遞迴來說，由於只存在一個呼叫幀，所以永遠不會發生“棧溢位”錯誤。

```javascript
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}

factorial(5) // 120
```

上面程式碼是一個階乘函式，計算n的階乘，最多需要儲存n個呼叫記錄，複雜度 O(n) 。

如果改寫成尾遞迴，只保留一個呼叫記錄，複雜度 O(1) 。

```javascript
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5, 1) // 120
```

還有一個比較著名的例子，就是計算fibonacci 數列，也能充分說明尾遞迴優化的重要性

如果是非尾遞迴的fibonacci 遞迴方法

```javascript
function Fibonacci (n) {
  if ( n <= 1 ) {return 1};

  return Fibonacci(n - 1) + Fibonacci(n - 2);
}

Fibonacci(10); // 89
// Fibonacci(100)
// Fibonacci(500)
// 堆疊溢位了
```

如果我們使用尾遞迴優化過的fibonacci 遞迴演算法

```javascript
function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
  if( n <= 1 ) {return ac2};

  return Fibonacci2 (n - 1, ac2, ac1 + ac2);
}

Fibonacci2(100) // 573147844013817200000
Fibonacci2(1000) // 7.0330367711422765e+208
Fibonacci2(10000) // Infinity
```

由此可見，“尾呼叫優化”對遞迴操作意義重大，所以一些函數語言程式設計語言將其寫入了語言規格。ES6也是如此，第一次明確規定，所有ECMAScript的實現，都必須部署“尾呼叫優化”。這就是說，在ES6中，只要使用尾遞迴，就不會發生棧溢位，相對節省記憶體。

### 遞迴函式的改寫

尾遞迴的實現，往往需要改寫遞迴函式，確保最後一步只調用自身。做到這一點的方法，就是把所有用到的內部變數改寫成函式的引數。比如上面的例子，階乘函式 factorial 需要用到一箇中間變數 total ，那就把這個中間變數改寫成函式的引數。這樣做的缺點就是不太直觀，第一眼很難看出來，為什麼計算5的階乘，需要傳入兩個引數5和1？

兩個方法可以解決這個問題。方法一是在尾遞迴函式之外，再提供一個正常形式的函式。

```javascript
function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

function factorial(n) {
  return tailFactorial(n, 1);
}

factorial(5) // 120
```

上面程式碼通過一個正常形式的階乘函式 factorial ，呼叫尾遞迴函式 tailFactorial ，看起來就正常多了。

函數語言程式設計有一個概念，叫做柯里化（currying），意思是將多引數的函式轉換成單引數的形式。這裡也可以使用柯里化。

```javascript
function currying(fn, n) {
  return function (m) {
    return fn.call(this, m, n);
  };
}

function tailFactorial(n, total) {
  if (n === 1) return total;
  return tailFactorial(n - 1, n * total);
}

const factorial = currying(tailFactorial, 1);

factorial(5) // 120
```

上面程式碼通過柯里化，將尾遞迴函式 tailFactorial 變為只接受1個引數的 factorial 。

第二種方法就簡單多了，就是採用ES6的函式預設值。

```javascript
function factorial(n, total = 1) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}

factorial(5) // 120
```

上面程式碼中，引數 total 有預設值1，所以呼叫時不用提供這個值。

總結一下，遞迴本質上是一種迴圈操作。純粹的函數語言程式設計語言沒有迴圈操作命令，所有的迴圈都用遞迴實現，這就是為什麼尾遞迴對這些語言極其重要。對於其他支援“尾呼叫優化”的語言（比如Lua，ES6），只需要知道迴圈可以用遞迴代替，而一旦使用遞迴，就最好使用尾遞迴。

### 嚴格模式

ES6的尾呼叫優化只在嚴格模式下開啟，正常模式是無效的。

這是因為在正常模式下，函式內部有兩個變數，可以跟蹤函式的呼叫棧。

- `func.arguments`：返回呼叫時函式的引數。
- `func.caller`：返回呼叫當前函式的那個函式。

尾呼叫優化發生時，函式的呼叫棧會改寫，因此上面兩個變數就會失真。嚴格模式禁用這兩個變數，所以尾呼叫模式僅在嚴格模式下生效。

```javascript
function restricted() {
  "use strict";
  restricted.caller;    // 報錯
  restricted.arguments; // 報錯
}
restricted();
```

### 尾遞迴優化的實現

尾遞迴優化只在嚴格模式下生效，那麼正常模式下，或者那些不支援該功能的環境中，有沒有辦法也使用尾遞迴優化呢？回答是可以的，就是自己實現尾遞迴優化。

它的原理非常簡單。尾遞迴之所以需要優化，原因是呼叫棧太多，造成溢位，那麼只要減少呼叫棧，就不會溢位。怎麼做可以減少呼叫棧呢？就是採用“迴圈”換掉“遞迴”。

下面是一個正常的遞迴函式。

```javascript
function sum(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1);
  } else {
    return x;
  }
}

sum(1, 100000)
// Uncaught RangeError: Maximum call stack size exceeded(…)
```

上面程式碼中，`sum`是一個遞迴函式，引數`x`是需要累加的值，引數`y`控制遞迴次數。一旦指定`sum`遞迴100000次，就會報錯，提示超出呼叫棧的最大次數。

蹦床函式（trampoline）可以將遞迴執行轉為迴圈執行。

```javascript
function trampoline(f) {
  while (f && f instanceof Function) {
    f = f();
  }
  return f;
}
```

上面就是蹦床函式的一個實現，它接受一個函式`f`作為引數。只要`f`執行後返回一個函式，就繼續執行。注意，這裡是返回一個函式，然後執行該函式，而不是函式裡面呼叫函式，這樣就避免了遞迴執行，從而就消除了呼叫棧過大的問題。

然後，要做的就是將原來的遞迴函式，改寫為每一步返回另一個函式。

```javascript
function sum(x, y) {
  if (y > 0) {
    return sum.bind(null, x + 1, y - 1);
  } else {
    return x;
  }
}
```

上面程式碼中，`sum`函式的每次執行，都會返回自身的另一個版本。

現在，使用蹦床函式執行`sum`，就不會發生呼叫棧溢位。

```javascript
trampoline(sum(1, 100000))
// 100001
```

蹦床函式並不是真正的尾遞迴優化，下面的實現才是。

```javascript
function tco(f) {
  var value;
  var active = false;
  var accumulated = [];

  return function accumulator() {
    accumulated.push(arguments);
    if (!active) {
      active = true;
      while (accumulated.length) {
        value = f.apply(this, accumulated.shift());
      }
      active = false;
      return value;
    }
  };
}

var sum = tco(function(x, y) {
  if (y > 0) {
    return sum(x + 1, y - 1)
  }
  else {
    return x
  }
});

sum(1, 100000)
// 100001
```

上面程式碼中，`tco`函式是尾遞迴優化的實現，它的奧妙就在於狀態變數`active`。預設情況下，這個變數是不啟用的。一旦進入尾遞迴優化的過程，這個變數就激活了。然後，每一輪遞迴`sum`返回的都是`undefined`，所以就避免了遞迴執行；而`accumulated`陣列存放每一輪`sum`執行的引數，總是有值的，這就保證了`accumulator`函式內部的`while`迴圈總是會執行。這樣就很巧妙地將“遞迴”改成了“迴圈”，而後一輪的引數會取代前一輪的引數，保證了呼叫棧只有一層。

## 函式引數的尾逗號

ES2017 [允許](https://github.com/jeffmo/es-trailing-function-commas)函式的最後一個引數有尾逗號（trailing comma）。

此前，函式定義和呼叫時，都不允許最後一個引數後面出現逗號。

```javascript
function clownsEverywhere(
  param1,
  param2
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar'
);
```

上面程式碼中，如果在`param2`或`bar`後面加一個逗號，就會報錯。

如果像上面這樣，將引數寫成多行（即每個引數佔據一行），以後修改程式碼的時候，想為函式`clownsEverywhere`新增第三個引數，或者調整引數的次序，就勢必要在原來最後一個引數後面新增一個逗號。這對於版本管理系統來說，就會顯示新增逗號的那一行也發生了變動。這看上去有點冗餘，因此新的語法允許定義和呼叫時，尾部直接有一個逗號。

```javascript
function clownsEverywhere(
  param1,
  param2,
) { /* ... */ }

clownsEverywhere(
  'foo',
  'bar',
);
```

這樣的規定也使得，函式引數與陣列和物件的尾逗號規則，保持一致了。

