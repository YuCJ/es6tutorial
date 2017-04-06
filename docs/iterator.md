# Iterator和for...of迴圈

## Iterator（遍歷器）的概念

JavaScript原有的表示“集合”的資料結構，主要是陣列（Array）和物件（Object），ES6又添加了Map和Set。這樣就有了四種資料集合，使用者還可以組合使用它們，定義自己的資料結構，比如陣列的成員是Map，Map的成員是物件。這樣就需要一種統一的介面機制，來處理所有不同的資料結構。

遍歷器（Iterator）就是這樣一種機制。它是一種介面，為各種不同的資料結構提供統一的訪問機制。任何資料結構只要部署Iterator介面，就可以完成遍歷操作（即依次處理該資料結構的所有成員）。

Iterator的作用有三個：一是為各種資料結構，提供一個統一的、簡便的訪問介面；二是使得資料結構的成員能夠按某種次序排列；三是ES6創造了一種新的遍歷命令`for...of`迴圈，Iterator介面主要供`for...of`消費。

Iterator的遍歷過程是這樣的。

（1）建立一個指標物件，指向當前資料結構的起始位置。也就是說，遍歷器物件本質上，就是一個指標物件。

（2）第一次呼叫指標物件的`next`方法，可以將指標指向資料結構的第一個成員。

（3）第二次呼叫指標物件的`next`方法，指標就指向資料結構的第二個成員。

（4）不斷呼叫指標物件的`next`方法，直到它指向資料結構的結束位置。

每一次呼叫`next`方法，都會返回資料結構的當前成員的資訊。具體來說，就是返回一個包含`value`和`done`兩個屬性的物件。其中，`value`屬性是當前成員的值，`done`屬性是一個布林值，表示遍歷是否結束。

下面是一個模擬`next`方法返回值的例子。

```javascript
var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }

function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  };
}
```

上面程式碼定義了一個`makeIterator`函式，它是一個遍歷器生成函式，作用就是返回一個遍歷器物件。對陣列`['a', 'b']`執行這個函式，就會返回該陣列的遍歷器物件（即指標物件）`it`。

指標物件的`next`方法，用來移動指標。開始時，指標指向陣列的開始位置。然後，每次呼叫`next`方法，指標就會指向陣列的下一個成員。第一次呼叫，指向`a`；第二次呼叫，指向`b`。

`next`方法返回一個物件，表示當前資料成員的資訊。這個物件具有`value`和`done`兩個屬性，`value`屬性返回當前位置的成員，`done`屬性是一個布林值，表示遍歷是否結束，即是否還有必要再一次呼叫`next`方法。

總之，呼叫指標物件的`next`方法，就可以遍歷事先給定的資料結構。

對於遍歷器物件來說，`done: false`和`value: undefined`屬性都是可以省略的，因此上面的`makeIterator`函式可以簡寫成下面的形式。

```javascript
function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++]} :
        {done: true};
    }
  };
}
```

由於Iterator只是把介面規格加到資料結構之上，所以，遍歷器與它所遍歷的那個資料結構，實際上是分開的，完全可以寫出沒有對應資料結構的遍歷器物件，或者說用遍歷器物件模擬出資料結構。下面是一個無限執行的遍歷器物件的例子。

```javascript
var it = idMaker();

it.next().value // '0'
it.next().value // '1'
it.next().value // '2'
// ...

function idMaker() {
  var index = 0;

  return {
    next: function() {
      return {value: index++, done: false};
    }
  };
}
```

上面的例子中，遍歷器生成函式`idMaker`，返回一個遍歷器物件（即指標物件）。但是並沒有對應的資料結構，或者說，遍歷器物件自己描述了一個數據結構出來。

在ES6中，有些資料結構原生具備Iterator介面（比如陣列），即不用任何處理，就可以被`for...of`迴圈遍歷，有些就不行（比如物件）。原因在於，這些資料結構原生部署了`Symbol.iterator`屬性（詳見下文），另外一些資料結構沒有。凡是部署了`Symbol.iterator`屬性的資料結構，就稱為部署了遍歷器介面。呼叫這個介面，就會返回一個遍歷器物件。

如果使用TypeScript的寫法，遍歷器介面（Iterable）、指標物件（Iterator）和next方法返回值的規格可以描述如下。

```javascript
interface Iterable {
  [Symbol.iterator]() : Iterator,
}

interface Iterator {
  next(value?: any) : IterationResult,
}

interface IterationResult {
  value: any,
  done: boolean,
}
```

## 資料結構的預設Iterator介面

Iterator介面的目的，就是為所有資料結構，提供了一種統一的訪問機制，即`for...of`迴圈（詳見下文）。當使用`for...of`迴圈遍歷某種資料結構時，該迴圈會自動去尋找Iterator介面。

一種資料結構只要部署了Iterator介面，我們就稱這種資料結構是”可遍歷的“（iterable）。

ES6規定，預設的Iterator介面部署在資料結構的`Symbol.iterator`屬性，或者說，一個數據結構只要具有`Symbol.iterator`屬性，就可以認為是“可遍歷的”（iterable）。`Symbol.iterator`屬性本身是一個函式，就是當前資料結構預設的遍歷器生成函式。執行這個函式，就會返回一個遍歷器。至於屬性名`Symbol.iterator`，它是一個表示式，返回`Symbol`物件的`iterator`屬性，這是一個預定義好的、型別為Symbol的特殊值，所以要放在方括號內。（參見Symbol一章）。

```javascript
const obj = {
  [Symbol.iterator] : function () {
    return {
      next: function () {
        return {
          value: 1,
          done: true
        };
      }
    };
  }
};
```

上面程式碼中，物件`obj`是可遍歷的（iterable），因為具有`Symbol.iterator`屬性。執行這個屬性，會返回一個遍歷器物件。該物件的根本特徵就是具有`next`方法。每次呼叫`next`方法，都會返回一個代表當前成員的資訊物件，具有`value`和`done`兩個屬性。

在ES6中，有三類資料結構原生具備Iterator介面：陣列、某些類似陣列的物件、Set和Map結構。

```javascript
let arr = ['a', 'b', 'c'];
let iter = arr[Symbol.iterator]();

iter.next() // { value: 'a', done: false }
iter.next() // { value: 'b', done: false }
iter.next() // { value: 'c', done: false }
iter.next() // { value: undefined, done: true }
```

上面程式碼中，變數`arr`是一個數組，原生就具有遍歷器介面，部署在`arr`的`Symbol.iterator`屬性上面。所以，呼叫這個屬性，就得到遍歷器物件。

上面提到，原生就部署Iterator介面的資料結構有三類，對於這三類資料結構，不用自己寫遍歷器生成函式，`for...of`迴圈會自動遍歷它們。除此之外，其他資料結構（主要是物件）的Iterator介面，都需要自己在`Symbol.iterator`屬性上面部署，這樣才會被`for...of`迴圈遍歷。

物件（Object）之所以沒有預設部署Iterator介面，是因為物件的哪個屬性先遍歷，哪個屬性後遍歷是不確定的，需要開發者手動指定。本質上，遍歷器是一種線性處理，對於任何非線性的資料結構，部署遍歷器介面，就等於部署一種線性轉換。不過，嚴格地說，物件部署遍歷器介面並不是很必要，因為這時物件實際上被當作Map結構使用，ES5沒有Map結構，而ES6原生提供了。

一個物件如果要有可被`for...of`迴圈呼叫的Iterator介面，就必須在`Symbol.iterator`的屬性上部署遍歷器生成方法（原型鏈上的物件具有該方法也可）。

```javascript
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }

  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    }
    return {done: true, value: undefined};
  }
}

function range(start, stop) {
  return new RangeIterator(start, stop);
}

for (var value of range(0, 3)) {
  console.log(value);
}
```

上面程式碼是一個類部署Iterator介面的寫法。`Symbol.iterator`屬性對應一個函式，執行後返回當前物件的遍歷器物件。

下面是通過遍歷器實現指標結構的例子。

```javascript
function Obj(value) {
  this.value = value;
  this.next = null;
}

Obj.prototype[Symbol.iterator] = function() {
  var iterator = {
    next: next
  };

  var current = this;

  function next() {
    if (current) {
      var value = current.value;
      current = current.next;
      return {
        done: false,
        value: value
      };
    } else {
      return {
        done: true
      };
    }
  }
  return iterator;
}

var one = new Obj(1);
var two = new Obj(2);
var three = new Obj(3);

one.next = two;
two.next = three;

for (var i of one){
  console.log(i);
}
// 1
// 2
// 3
```

上面程式碼首先在建構函式的原型鏈上部署`Symbol.iterator`方法，呼叫該方法會返回遍歷器物件`iterator`，呼叫該物件的`next`方法，在返回一個值的同時，自動將內部指標移到下一個實例。

下面是另一個為物件新增Iterator介面的例子。

```javascript
let obj = {
  data: [ 'hello', 'world' ],
  [Symbol.iterator]() {
    const self = this;
    let index = 0;
    return {
      next() {
        if (index < self.data.length) {
          return {
            value: self.data[index++],
            done: false
          };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};
```

對於類似陣列的物件（存在數值鍵名和length屬性），部署Iterator介面，有一個簡便方法，就是`Symbol.iterator`方法直接引用陣列的Iterator介面。

```javascript
NodeList.prototype[Symbol.iterator] = Array.prototype[Symbol.iterator];
// 或者
NodeList.prototype[Symbol.iterator] = [][Symbol.iterator];

[...document.querySelectorAll('div')] // 可以執行了
```

下面是類似陣列的物件呼叫陣列的`Symbol.iterator`方法的例子。

```javascript
let iterable = {
  0: 'a',
  1: 'b',
  2: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // 'a', 'b', 'c'
}
```

注意，普通物件部署陣列的`Symbol.iterator`方法，並無效果。

```javascript
let iterable = {
  a: 'a',
  b: 'b',
  c: 'c',
  length: 3,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};
for (let item of iterable) {
  console.log(item); // undefined, undefined, undefined
}
```

如果`Symbol.iterator`方法對應的不是遍歷器生成函式（即會返回一個遍歷器物件），解釋引擎將會報錯。

```javascript
var obj = {};

obj[Symbol.iterator] = () => 1;

[...obj] // TypeError: [] is not a function
```

上面程式碼中，變數obj的Symbol.iterator方法對應的不是遍歷器生成函式，因此報錯。

有了遍歷器介面，資料結構就可以用`for...of`迴圈遍歷（詳見下文），也可以使用`while`迴圈遍歷。

```javascript
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  var x = $result.value;
  // ...
  $result = $iterator.next();
}
```

上面程式碼中，`ITERABLE`代表某種可遍歷的資料結構，`$iterator`是它的遍歷器物件。遍歷器物件每次移動指標（`next`方法），都檢查一下返回值的`done`屬性，如果遍歷還沒結束，就移動遍歷器物件的指標到下一步（`next`方法），不斷迴圈。

## 呼叫Iterator介面的場合

有一些場合會預設呼叫Iterator介面（即`Symbol.iterator`方法），除了下文會介紹的`for...of`迴圈，還有幾個別的場合。

**（1）解構賦值**

對陣列和Set結構進行解構賦值時，會預設呼叫`Symbol.iterator`方法。

```javascript
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

**（2）展開運算子**

展開運算子（...）也會呼叫預設的iterator介面。

```javascript
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

上面程式碼的展開運算子內部就呼叫Iterator介面。

實際上，這提供了一種簡便機制，可以將任何部署了Iterator介面的資料結構，轉為陣列。也就是說，只要某個資料結構部署了Iterator介面，就可以對它使用展開運算子，將其轉為陣列。

```javascript
let arr = [...iterable];
```

**（3）yield* **

yield*後面跟的是一個可遍歷的結構，它會呼叫該結構的遍歷器介面。

```javascript
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

**（4）其他場合**

由於陣列的遍歷會呼叫遍歷器介面，所以任何接受陣列作為引數的場合，其實都呼叫了遍歷器介面。下面是一些例子。

- for...of
- Array.from()
- Map(), Set(), WeakMap(), WeakSet()（比如`new Map([['a',1],['b',2]])`）
- Promise.all()
- Promise.race()

## 字串的Iterator介面

字串是一個類似陣列的物件，也原生具有Iterator介面。

```javascript
var someString = "hi";
typeof someString[Symbol.iterator]
// "function"

var iterator = someString[Symbol.iterator]();

iterator.next()  // { value: "h", done: false }
iterator.next()  // { value: "i", done: false }
iterator.next()  // { value: undefined, done: true }
```

上面程式碼中，呼叫`Symbol.iterator`方法返回一個遍歷器物件，在這個遍歷器上可以呼叫next方法，實現對於字串的遍歷。

可以覆蓋原生的`Symbol.iterator`方法，達到修改遍歷器行為的目的。

```javascript
var str = new String("hi");

[...str] // ["h", "i"]

str[Symbol.iterator] = function() {
  return {
    next: function() {
      if (this._first) {
        this._first = false;
        return { value: "bye", done: false };
      } else {
        return { done: true };
      }
    },
    _first: true
  };
};

[...str] // ["bye"]
str // "hi"
```

上面程式碼中，字串str的`Symbol.iterator`方法被修改了，所以展開運算子（`...`）返回的值變成了`bye`，而字串本身還是`hi`。

## Iterator介面與Generator函式

`Symbol.iterator`方法的最簡單實現，還是使用下一章要介紹的Generator函式。

```javascript
var myIterable = {};

myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};
[...myIterable] // [1, 2, 3]

// 或者採用下面的簡潔寫法

let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x);
}
// hello
// world
```

上面程式碼中，`Symbol.iterator`方法幾乎不用部署任何程式碼，只要用yield命令給出每一步的返回值即可。

## 遍歷器物件的return()，throw()

遍歷器物件除了具有`next`方法，還可以具有`return`方法和`throw`方法。如果你自己寫遍歷器物件生成函式，那麼`next`方法是必須部署的，`return`方法和`throw`方法是否部署是可選的。

`return`方法的使用場合是，如果`for...of`迴圈提前退出（通常是因為出錯，或者有`break`語句或`continue`語句），就會呼叫`return`方法。如果一個物件在完成遍歷前，需要清理或釋放資源，就可以部署`return`方法。

```javascript
function readLinesSync(file) {
  return {
    next() {
      if (file.isAtEndOfFile()) {
        file.close();
        return { done: true };
      }
    },
    return() {
      file.close();
      return { done: true };
    },
  };
}
```

上面程式碼中，函式`readLinesSync`接受一個檔案物件作為引數，返回一個遍歷器物件，其中除了`next`方法，還部署了`return`方法。下面，我們讓檔案的遍歷提前返回，這樣就會觸發執行`return`方法。

```javascript
for (let line of readLinesSync(fileName)) {
  console.log(line);
  break;
}
```

注意，`return`方法必須返回一個物件，這是Generator規格決定的。

`throw`方法主要是配合Generator函式使用，一般的遍歷器物件用不到這個方法。請參閱《Generator函式》一章。

## for...of迴圈

ES6 借鑑 C++、Java、C# 和 Python 語言，引入了`for...of`迴圈，作為遍歷所有資料結構的統一的方法。

一個數據結構只要部署了`Symbol.iterator`屬性，就被視為具有iterator介面，就可以用`for...of`迴圈遍歷它的成員。也就是說，`for...of`迴圈內部呼叫的是資料結構的`Symbol.iterator`方法。

`for...of`迴圈可以使用的範圍包括陣列、Set 和 Map 結構、某些類似陣列的物件（比如`arguments`物件、DOM NodeList 物件）、後文的 Generator 物件，以及字串。

### 陣列

陣列原生具備`iterator`介面（即預設部署了`Symbol.iterator`屬性），`for...of`迴圈本質上就是呼叫這個介面產生的遍歷器，可以用下面的程式碼證明。

```javascript
const arr = ['red', 'green', 'blue'];

for(let v of arr) {
  console.log(v); // red green blue
}

const obj = {};
obj[Symbol.iterator] = arr[Symbol.iterator].bind(arr);

for(let v of obj) {
  console.log(v); // red green blue
}
```

上面程式碼中，空物件`obj`部署了陣列`arr`的`Symbol.iterator`屬性，結果`obj`的`for...of`迴圈，產生了與`arr`完全一樣的結果。

`for...of`迴圈可以代替陣列實例的`forEach`方法。

```javascript
const arr = ['red', 'green', 'blue'];

arr.forEach(function (element, index) {
  console.log(element); // red green blue
  console.log(index);   // 0 1 2
});
```

JavaScript原有的`for...in`迴圈，只能獲得物件的鍵名，不能直接獲取鍵值。ES6提供`for...of`迴圈，允許遍歷獲得鍵值。

```javascript
var arr = ['a', 'b', 'c', 'd'];

for (let a in arr) {
  console.log(a); // 0 1 2 3
}

for (let a of arr) {
  console.log(a); // a b c d
}
```

上面程式碼表明，`for...in`迴圈讀取鍵名，`for...of`迴圈讀取鍵值。如果要通過`for...of`迴圈，獲取陣列的索引，可以藉助陣列實例的`entries`方法和`keys`方法，參見《陣列的擴展》章節。

`for...of`迴圈呼叫遍歷器介面，陣列的遍歷器介面只返回具有數字索引的屬性。這一點跟`for...in`迴圈也不一樣。

```javascript
let arr = [3, 5, 7];
arr.foo = 'hello';

for (let i in arr) {
  console.log(i); // "0", "1", "2", "foo"
}

for (let i of arr) {
  console.log(i); //  "3", "5", "7"
}
```

上面程式碼中，`for...of`迴圈不會返回陣列`arr`的`foo`屬性。

### Set和Map結構

Set 和 Map 結構也原生具有 Iterator 介面，可以直接使用`for...of`迴圈。

```javascript
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
  console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

上面程式碼演示瞭如何遍歷 Set 結構和 Map 結構。值得注意的地方有兩個，首先，遍歷的順序是按照各個成員被新增進資料結構的順序。其次，Set 結構遍歷時，返回的是一個值，而 Map 結構遍歷時，返回的是一個數組，該陣列的兩個成員分別為當前 Map 成員的鍵名和鍵值。

```javascript
let map = new Map().set('a', 1).set('b', 2);
for (let pair of map) {
  console.log(pair);
}
// ['a', 1]
// ['b', 2]

for (let [key, value] of map) {
  console.log(key + ' : ' + value);
}
// a : 1
// b : 2
```

### 計算生成的資料結構

有些資料結構是在現有資料結構的基礎上，計算生成的。比如，ES6的陣列、Set、Map 都部署了以下三個方法，呼叫後都返回遍歷器物件。

- `entries()` 返回一個遍歷器物件，用來遍歷`[鍵名, 鍵值]`組成的陣列。對於陣列，鍵名就是索引值；對於 Set，鍵名與鍵值相同。Map 結構的 Iterator 介面，預設就是呼叫`entries`方法。
- `keys()` 返回一個遍歷器物件，用來遍歷所有的鍵名。
- `values()` 返回一個遍歷器物件，用來遍歷所有的鍵值。

這三個方法呼叫後生成的遍歷器物件，所遍歷的都是計算生成的資料結構。

```javascript
let arr = ['a', 'b', 'c'];
for (let pair of arr.entries()) {
  console.log(pair);
}
// [0, 'a']
// [1, 'b']
// [2, 'c']
```

### 類似陣列的物件

類似陣列的物件包括好幾類。下面是`for...of`迴圈用於字串、DOM NodeList 物件、`arguments`物件的例子。

```javascript
// 字串
let str = "hello";

for (let s of str) {
  console.log(s); // h e l l o
}

// DOM NodeList物件
let paras = document.querySelectorAll("p");

for (let p of paras) {
  p.classList.add("test");
}

// arguments物件
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
```

對於字串來說，`for...of`迴圈還有一個特點，就是會正確識別32位 UTF-16 字元。

```javascript
for (let x of 'a\uD83D\uDC0A') {
  console.log(x);
}
// 'a'
// '\uD83D\uDC0A'
```

並不是所有類似陣列的物件都具有 Iterator 介面，一個簡便的解決方法，就是使用`Array.from`方法將其轉為陣列。

```javascript
let arrayLike = { length: 2, 0: 'a', 1: 'b' };

// 報錯
for (let x of arrayLike) {
  console.log(x);
}

// 正確
for (let x of Array.from(arrayLike)) {
  console.log(x);
}
```

### 物件

對於普通的物件，`for...of`結構不能直接使用，會報錯，必須部署了 Iterator 介面後才能使用。但是，這樣情況下，`for...in`迴圈依然可以用來遍歷鍵名。

```javascript
let es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};

for (let e in es6) {
  console.log(e);
}
// edition
// committee
// standard

for (let e of es6) {
  console.log(e);
}
// TypeError: es6 is not iterable
```

上面程式碼表示，對於普通的物件，`for...in`迴圈可以遍歷鍵名，`for...of`迴圈會報錯。

一種解決方法是，使用`Object.keys`方法將物件的鍵名生成一個數組，然後遍歷這個陣列。

```javascript
for (var key of Object.keys(someObject)) {
  console.log(key + ': ' + someObject[key]);
}
```

另一個方法是使用 Generator 函式將物件重新包裝一下。

```javascript
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

for (let [key, value] of entries(obj)) {
  console.log(key, '->', value);
}
// a -> 1
// b -> 2
// c -> 3
```

### 與其他遍歷語法的比較

以陣列為例，JavaScript 提供多種遍歷語法。最原始的寫法就是`for`迴圈。

```javascript
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```

這種寫法比較麻煩，因此陣列提供內建的`forEach`方法。

```javascript
myArray.forEach(function (value) {
  console.log(value);
});
```

這種寫法的問題在於，無法中途跳出`forEach`迴圈，`break`命令或`return`命令都不能奏效。

`for...in`迴圈可以遍歷陣列的鍵名。

```javascript
for (var index in myArray) {
  console.log(myArray[index]);
}
```

`for...in`迴圈有幾個缺點。

- 陣列的鍵名是數字，但是`for...in`迴圈是以字串作為鍵名“0”、“1”、“2”等等。
- `for...in`迴圈不僅遍歷數字鍵名，還會遍歷手動新增的其他鍵，甚至包括原型鏈上的鍵。
- 某些情況下，`for...in`迴圈會以任意順序遍歷鍵名。

總之，`for...in`迴圈主要是為遍歷物件而設計的，不適用於遍歷陣列。

`for...of`迴圈相比上面幾種做法，有一些顯著的優點。

```javascript
for (let value of myArray) {
  console.log(value);
}
```

- 有著同`for...in`一樣的簡潔語法，但是沒有`for...in`那些缺點。
- 不同用於`forEach`方法，它可以與`break`、`continue`和`return`配合使用。
- 提供了遍歷所有資料結構的統一操作介面。

下面是一個使用break語句，跳出`for...of`迴圈的例子。

```javascript
for (var n of fibonacci) {
  if (n > 1000)
    break;
  console.log(n);
}
```

上面的例子，會輸出斐波納契數列小於等於1000的項。如果當前項大於1000，就會使用`break`語句跳出`for...of`迴圈。

