# 變數的解構賦值

## 陣列的解構賦值

### 基本用法

ES6 允許按照一定模式，從陣列和物件中提取值，對變數進行賦值，這被稱為解構（Destructuring）。

以前，為變數賦值，只能直接指定值。

```javascript
let a = 1;
let b = 2;
let c = 3;
```

ES6允許寫成下面這樣。

```javascript
let [a, b, c] = [1, 2, 3];
```

上面程式碼表示，可以從陣列中提取值，按照對應位置，對變數賦值。

本質上，這種寫法屬於“模式匹配”，只要等號兩邊的模式相同，左邊的變數就會被賦予對應的值。下面是一些使用巢狀陣列進行解構的例子。

```javascript
let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

如果解構不成功，變數的值就等於`undefined`。

```javascript
let [foo] = [];
let [bar, foo] = [1];
```

以上兩種情況都屬於解構不成功，`foo`的值都會等於`undefined`。

另一種情況是不完全解構，即等號左邊的模式，只匹配一部分的等號右邊的陣列。這種情況下，解構依然可以成功。

```javascript
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```

上面兩個例子，都屬於不完全解構，但是可以成功。

如果等號的右邊不是陣列（或者嚴格地說，不是可遍歷的結構，參見《Iterator》一章），那麼將會報錯。

```javascript
// 報錯
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```

上面的語句都會報錯，因為等號右邊的值，要麼轉為物件以後不具備 Iterator 介面（前五個表示式），要麼本身就不具備 Iterator 介面（最後一個表示式）。

對於 Set 結構，也可以使用陣列的解構賦值。

```javascript
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
```

事實上，只要某種資料結構具有 Iterator 介面，都可以採用陣列形式的解構賦值。

```javascript
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
```

上面程式碼中，`fibs`是一個 Generator 函式（參見《Generator 函式》一章），原生具有 Iterator 介面。解構賦值會依次從這個介面獲取值。

### 預設值

解構賦值允許指定預設值。

```javascript
let [foo = true] = [];
foo // true

let [x, y = 'b'] = ['a']; // x='a', y='b'
let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

注意，ES6 內部使用嚴格相等運算子（`===`），判斷一個位置是否有值。所以，如果一個數組成員不嚴格等於`undefined`，預設值是不會生效的。

```javascript
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
```

上面程式碼中，如果一個數組成員是`null`，預設值就不會生效，因為`null`不嚴格等於`undefined`。

如果預設值是一個表示式，那麼這個表示式是惰性求值的，即只有在用到的時候，才會求值。

```javascript
function f() {
  console.log('aaa');
}

let [x = f()] = [1];
```

上面程式碼中，因為`x`能取到值，所以函式`f`根本不會執行。上面的程式碼其實等價於下面的程式碼。

```javascript
let x;
if ([1][0] === undefined) {
  x = f();
} else {
  x = [1][0];
}
```

預設值可以引用解構賦值的其他變數，但該變數必須已經宣告。

```javascript
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError
```

上面最後一個表示式之所以會報錯，是因為`x`用到預設值`y`時，`y`還沒有宣告。

## 物件的解構賦值

解構不僅可以用於陣列，還可以用於物件。

```javascript
let { foo, bar } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"
```

物件的解構與陣列有一個重要的不同。陣列的元素是按次序排列的，變數的取值由它的位置決定；而物件的屬性沒有次序，變數必須與屬性同名，才能取到正確的值。

```javascript
let { bar, foo } = { foo: "aaa", bar: "bbb" };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: "aaa", bar: "bbb" };
baz // undefined
```

上面程式碼的第一個例子，等號左邊的兩個變數的次序，與等號右邊兩個同名屬性的次序不一致，但是對取值完全沒有影響。第二個例子的變數沒有對應的同名屬性，導致取不到值，最後等於`undefined`。

如果變數名與屬性名不一致，必須寫成下面這樣。

```javascript
var { foo: baz } = { foo: 'aaa', bar: 'bbb' };
baz // "aaa"

let obj = { first: 'hello', last: 'world' };
let { first: f, last: l } = obj;
f // 'hello'
l // 'world'
```

這實際上說明，物件的解構賦值是下面形式的簡寫（參見《物件的擴充套件》一章）。

```javascript
let { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" };
```

也就是說，物件的解構賦值的內部機制，是先找到同名屬性，然後再賦給對應的變數。真正被賦值的是後者，而不是前者。

```javascript
let { foo: baz } = { foo: "aaa", bar: "bbb" };
baz // "aaa"
foo // error: foo is not defined
```

上面程式碼中，`foo`是匹配的模式，`baz`才是變數。真正被賦值的是變數`baz`，而不是模式`foo`。

注意，採用這種寫法時，變數的宣告和賦值是一體的。對於`let`和`const`來說，變數不能重新宣告，所以一旦賦值的變數以前宣告過，就會報錯。

```javascript
let foo;
let {foo} = {foo: 1}; // SyntaxError: Duplicate declaration "foo"

let baz;
let {bar: baz} = {bar: 1}; // SyntaxError: Duplicate declaration "baz"
```

上面程式碼中，解構賦值的變數都會重新宣告，所以報錯了。不過，因為`var`命令允許重新宣告，所以這個錯誤只會在使用`let`和`const`命令時出現。如果沒有第二個`let`命令，上面的程式碼就不會報錯。

```javascript
let foo;
({foo} = {foo: 1}); // 成功

let baz;
({bar: baz} = {bar: 1}); // 成功
```

上面程式碼中，`let`命令下面一行的圓括號是必須的，否則會報錯。因為解析器會將起首的大括號，理解成一個程式碼塊，而不是賦值語句。

和陣列一樣，解構也可以用於巢狀結構的物件。

```javascript
let obj = {
  p: [
    'Hello',
    { y: 'World' }
  ]
};

let { p: [x, { y }] } = obj;
x // "Hello"
y // "World"
```

注意，這時`p`是模式，不是變數，因此不會被賦值。

```javascript
var node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

var { loc: { start: { line }} } = node;
line // 1
loc  // error: loc is undefined
start // error: start is undefined
```

上面程式碼中，只有`line`是變數，`loc`和`start`都是模式，不會被賦值。

下面是巢狀賦值的例子。

```javascript
let obj = {};
let arr = [];

({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });

obj // {prop:123}
arr // [true]
```

物件的解構也可以指定預設值。

```javascript
var {x = 3} = {};
x // 3

var {x, y = 5} = {x: 1};
x // 1
y // 5

var {x:y = 3} = {};
y // 3

var {x:y = 3} = {x: 5};
y // 5

var { message: msg = 'Something went wrong' } = {};
msg // "Something went wrong"
```

預設值生效的條件是，物件的屬性值嚴格等於`undefined`。

```javascript
var {x = 3} = {x: undefined};
x // 3

var {x = 3} = {x: null};
x // null
```

上面程式碼中，如果`x`屬性等於`null`，就不嚴格相等於`undefined`，導致預設值不會生效。

如果解構失敗，變數的值等於`undefined`。

```javascript
let {foo} = {bar: 'baz'};
foo // undefined
```

如果解構模式是巢狀的物件，而且子物件所在的父屬性不存在，那麼將會報錯。

```javascript
// 報錯
let {foo: {bar}} = {baz: 'baz'};
```

上面程式碼中，等號左邊物件的`foo`屬性，對應一個子物件。該子物件的`bar`屬性，解構時會報錯。原因很簡單，因為`foo`這時等於`undefined`，再取子屬性就會報錯，請看下面的程式碼。

```javascript
let _tmp = {baz: 'baz'};
_tmp.foo.bar // 報錯
```

如果要將一個已經宣告的變數用於解構賦值，必須非常小心。

```javascript
// 錯誤的寫法
let x;
{x} = {x: 1};
// SyntaxError: syntax error
```

上面程式碼的寫法會報錯，因為JavaScript引擎會將`{x}`理解成一個程式碼塊，從而發生語法錯誤。只有不將大括號寫在行首，避免JavaScript將其解釋為程式碼塊，才能解決這個問題。

```javascript
// 正確的寫法
({x} = {x: 1});
```

上面程式碼將整個解構賦值語句，放在一個圓括號裡面，就可以正確執行。關於圓括號與解構賦值的關係，參見下文。

解構賦值允許，等號左邊的模式之中，不放置任何變數名。因此，可以寫出非常古怪的賦值表示式。

```javascript
({} = [true, false]);
({} = 'abc');
({} = []);
```

上面的表示式雖然毫無意義，但是語法是合法的，可以執行。

物件的解構賦值，可以很方便地將現有物件的方法，賦值到某個變數。

```javascript
let { log, sin, cos } = Math;
```

上面程式碼將`Math`物件的對數、正弦、餘弦三個方法，賦值到對應的變數上，使用起來就會方便很多。

由於陣列本質是特殊的物件，因此可以對陣列進行物件屬性的解構。

```javascript
let arr = [1, 2, 3];
let {0 : first, [arr.length - 1] : last} = arr;
first // 1
last // 3
```

上面程式碼對陣列進行物件解構。陣列`arr`的`0`鍵對應的值是`1`，`[arr.length - 1]`就是`2`鍵，對應的值是`3`。方括號這種寫法，屬於“屬性名錶達式”，參見《物件的擴充套件》一章。

## 字串的解構賦值

字串也可以解構賦值。這是因為此時，字串被轉換成了一個類似陣列的物件。

```javascript
const [a, b, c, d, e] = 'hello';
a // "h"
b // "e"
c // "l"
d // "l"
e // "o"
```

類似陣列的物件都有一個`length`屬性，因此還可以對這個屬性解構賦值。

```javascript
let {length : len} = 'hello';
len // 5
```

## 數值和布林值的解構賦值

解構賦值時，如果等號右邊是數值和布林值，則會先轉為物件。

```javascript
let {toString: s} = 123;
s === Number.prototype.toString // true

let {toString: s} = true;
s === Boolean.prototype.toString // true
```

上面程式碼中，數值和布林值的包裝物件都有`toString`屬性，因此變數`s`都能取到值。

解構賦值的規則是，只要等號右邊的值不是物件或陣列，就先將其轉為物件。由於`undefined`和`null`無法轉為物件，所以對它們進行解構賦值，都會報錯。

```javascript
let { prop: x } = undefined; // TypeError
let { prop: y } = null; // TypeError
```

## 函式引數的解構賦值

函式的引數也可以使用解構賦值。

```javascript
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

上面程式碼中，函式`add`的引數表面上是一個數組，但在傳入引數的那一刻，陣列引數就被解構成變數`x`和`y`。對於函式內部的程式碼來說，它們能感受到的引數就是`x`和`y`。

下面是另一個例子。

```javascript
[[1, 2], [3, 4]].map(([a, b]) => a + b);
// [ 3, 7 ]
```

函式引數的解構也可以使用預設值。

```javascript
function move({x = 0, y = 0} = {}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

上面程式碼中，函式`move`的引數是一個物件，通過對這個物件進行解構，得到變數`x`和`y`的值。如果解構失敗，`x`和`y`等於預設值。

注意，下面的寫法會得到不一樣的結果。

```javascript
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

上面程式碼是為函式`move`的引數指定預設值，而不是為變數`x`和`y`指定預設值，所以會得到與前一種寫法不同的結果。

`undefined`就會觸發函式引數的預設值。

```javascript
[1, undefined, 3].map((x = 'yes') => x);
// [ 1, 'yes', 3 ]
```

## 圓括號問題

解構賦值雖然很方便，但是解析起來並不容易。對於編譯器來說，一個式子到底是模式，還是表示式，沒有辦法從一開始就知道，必須解析到（或解析不到）等號才能知道。

由此帶來的問題是，如果模式中出現圓括號怎麼處理。ES6的規則是，只要有可能導致解構的歧義，就不得使用圓括號。

但是，這條規則實際上不那麼容易辨別，處理起來相當麻煩。因此，建議只要有可能，就不要在模式中放置圓括號。

### 不能使用圓括號的情況

以下三種解構賦值不得使用圓括號。

（1）變數宣告語句中，不能帶有圓括號。

```javascript
// 全部報錯
let [(a)] = [1];

let {x: (c)} = {};
let ({x: c}) = {};
let {(x: c)} = {};
let {(x): c} = {};

let { o: ({ p: p }) } = { o: { p: 2 } };
```

上面三個語句都會報錯，因為它們都是變數宣告語句，模式不能使用圓括號。

（2）函式引數中，模式不能帶有圓括號。

函式引數也屬於變數宣告，因此不能帶有圓括號。

```javascript
// 報錯
function f([(z)]) { return z; }
```

（3）賦值語句中，不能將整個模式，或巢狀模式中的一層，放在圓括號之中。

```javascript
// 全部報錯
({ p: a }) = { p: 42 };
([a]) = [5];
```

上面程式碼將整個模式放在圓括號之中，導致報錯。

```javascript
// 報錯
[({ p: a }), { x: c }] = [{}, {}];
```

上面程式碼將巢狀模式的一層，放在圓括號之中，導致報錯。

### 可以使用圓括號的情況

可以使用圓括號的情況只有一種：賦值語句的非模式部分，可以使用圓括號。

```javascript
[(b)] = [3]; // 正確
({ p: (d) } = {}); // 正確
[(parseInt.prop)] = [3]; // 正確
```

上面三行語句都可以正確執行，因為首先它們都是賦值語句，而不是宣告語句；其次它們的圓括號都不屬於模式的一部分。第一行語句中，模式是取陣列的第一個成員，跟圓括號無關；第二行語句中，模式是p，而不是d；第三行語句與第一行語句的性質一致。

## 用途

變數的解構賦值用途很多。

**（1）交換變數的值**

```javascript
let x = 1;
let y = 2;

[x, y] = [y, x];
```

上面程式碼交換變數`x`和`y`的值，這樣的寫法不僅簡潔，而且易讀，語義非常清晰。

**（2）從函式返回多個值**

函式只能返回一個值，如果要返回多個值，只能將它們放在陣列或物件裡返回。有了解構賦值，取出這些值就非常方便。

```javascript
// 返回一個數組

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一個物件

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

**（3）函式引數的定義**

解構賦值可以方便地將一組引數與變數名對應起來。

```javascript
// 引數是一組有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 引數是一組無次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

**（4）提取JSON資料**

解構賦值對提取JSON物件中的資料，尤其有用。

```javascript
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

上面程式碼可以快速提取 JSON 資料的值。

**（5）函式引數的預設值**

```javascript
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
}) {
  // ... do stuff
};
```

指定引數的預設值，就避免了在函式體內部再寫`var foo = config.foo || 'default foo';`這樣的語句。

**（6）遍歷Map結構**

任何部署了Iterator介面的物件，都可以用`for...of`迴圈遍歷。Map結構原生支援Iterator介面，配合變數的解構賦值，獲取鍵名和鍵值就非常方便。

```javascript
var map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

如果只想獲取鍵名，或者只想獲取鍵值，可以寫成下面這樣。

```javascript
// 獲取鍵名
for (let [key] of map) {
  // ...
}

// 獲取鍵值
for (let [,value] of map) {
  // ...
}
```

**（7）輸入模組的指定方法**

載入模組時，往往需要指定輸入哪些方法。解構賦值使得輸入語句非常清晰。

```javascript
const { SourceMapConsumer, SourceNode } = require("source-map");
```
