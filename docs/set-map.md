# Set和Map資料結構

## Set

### 基本用法

ES6 提供了新的資料結構 Set。它類似於陣列，但是成員的值都是唯一的，沒有重複的值。

Set 本身是一個建構函式，用來生成 Set 資料結構。

```javascript
const s = new Set();

[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));

for (let i of s) {
  console.log(i);
}
// 2 3 5 4
```

上面程式碼通過`add`方法向 Set 結構加入成員，結果表明 Set 結構不會新增重複的值。

Set 函式可以接受一個數組（或類似陣列的物件）作為引數，用來初始化。

```javascript
// 例一
var set = new Set([1, 2, 3, 4, 4]);
[...set]
// [1, 2, 3, 4]

// 例二
var items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size // 5

// 例三
function divs () {
  return [...document.querySelectorAll('div')];
}

var set = new Set(divs());
set.size // 56

// 類似於
divs().forEach(div => set.add(div));
set.size // 56
```

上面程式碼中，例一和例二都是`Set`函式接受陣列作為引數，例三是接受類似陣列的物件作為引數。

上面程式碼中，也展示了一種去除陣列重複成員的方法。

```javascript
// 去除陣列的重複成員
[...new Set(array)]
```

向Set加入值的時候，不會發生型別轉換，所以`5`和`"5"`是兩個不同的值。Set內部判斷兩個值是否不同，使用的演算法叫做“Same-value equality”，它類似於精確相等運算子（`===`），主要的區別是`NaN`等於自身，而精確相等運算子認為`NaN`不等於自身。

```javascript
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set {NaN}
```

上面程式碼向Set實例添加了兩個`NaN`，但是隻能加入一個。這表明，在Set內部，兩個`NaN`是相等。

另外，兩個物件總是不相等的。

```javascript
let set = new Set();

set.add({});
set.size // 1

set.add({});
set.size // 2
```

上面程式碼表示，由於兩個空物件不相等，所以它們被視為兩個值。

### Set實例的屬性和方法

Set結構的實例有以下屬性。

- `Set.prototype.constructor`：建構函式，預設就是`Set`函式。
- `Set.prototype.size`：返回`Set`實例的成員總數。

Set實例的方法分為兩大類：操作方法（用於操作資料）和遍歷方法（用於遍歷成員）。下面先介紹四個操作方法。

- `add(value)`：新增某個值，返回Set結構本身。
- `delete(value)`：刪除某個值，返回一個布林值，表示刪除是否成功。
- `has(value)`：返回一個布林值，表示該值是否為`Set`的成員。
- `clear()`：清除所有成員，沒有返回值。

上面這些屬性和方法的實例如下。

```javascript
s.add(1).add(2).add(2);
// 注意2被加入了兩次

s.size // 2

s.has(1) // true
s.has(2) // true
s.has(3) // false

s.delete(2);
s.has(2) // false
```

下面是一個對比，看看在判斷是否包括一個鍵上面，`Object`結構和`Set`結構的寫法不同。

```javascript
// 物件的寫法
var properties = {
  'width': 1,
  'height': 1
};

if (properties[someName]) {
  // do something
}

// Set的寫法
var properties = new Set();

properties.add('width');
properties.add('height');

if (properties.has(someName)) {
  // do something
}
```

`Array.from`方法可以將Set結構轉為陣列。

```javascript
var items = new Set([1, 2, 3, 4, 5]);
var array = Array.from(items);
```

這就提供了去除陣列重複成員的另一種方法。

```javascript
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```

### 遍歷操作

Set結構的實例有四個遍歷方法，可以用於遍歷成員。

- `keys()`：返回鍵名的遍歷器
- `values()`：返回鍵值的遍歷器
- `entries()`：返回鍵值對的遍歷器
- `forEach()`：使用回呼函式遍歷每個成員

需要特別指出的是，`Set`的遍歷順序就是插入順序。這個特性有時非常有用，比如使用Set儲存一個回呼函式列表，呼叫時就能保證按照新增順序呼叫。

**（1）`keys()`，`values()`，`entries()`**

`keys`方法、`values`方法、`entries`方法返回的都是遍歷器物件（詳見《Iterator 物件》一章）。由於 Set 結構沒有鍵名，只有鍵值（或者說鍵名和鍵值是同一個值），所以`keys`方法和`values`方法的行為完全一致。

```javascript
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```

上面程式碼中，`entries`方法返回的遍歷器，同時包括鍵名和鍵值，所以每次輸出一個數組，它的兩個成員完全相等。

Set結構的實例預設可遍歷，它的預設遍歷器生成函式就是它的`values`方法。

```javascript
Set.prototype[Symbol.iterator] === Set.prototype.values
// true
```

這意味著，可以省略`values`方法，直接用`for...of`迴圈遍歷Set。

```javascript
let set = new Set(['red', 'green', 'blue']);

for (let x of set) {
  console.log(x);
}
// red
// green
// blue
```

**（2）`forEach()`**

Set結構的實例的`forEach`方法，用於對每個成員執行某種操作，沒有返回值。

```javascript
let set = new Set([1, 2, 3]);
set.forEach((value, key) => console.log(value * 2) )
// 2
// 4
// 6
```

上面程式碼說明，`forEach`方法的引數就是一個處理函式。該函式的引數依次為鍵值、鍵名、集合本身（上例省略了該引數）。另外，`forEach`方法還可以有第二個引數，表示繫結的this物件。

**（3）遍歷的應用**

展開運算子（`...`）內部使用`for...of`迴圈，所以也可以用於Set結構。

```javascript
let set = new Set(['red', 'green', 'blue']);
let arr = [...set];
// ['red', 'green', 'blue']
```

展開運算子和Set結構相結合，就可以去除陣列的重複成員。

```javascript
let arr = [3, 5, 2, 2, 5, 5];
let unique = [...new Set(arr)];
// [3, 5, 2]
```

而且，陣列的`map`和`filter`方法也可以用於Set了。

```javascript
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));
// 返回Set結構：{2, 4, 6}

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));
// 返回Set結構：{2, 4}
```

因此使用Set可以很容易地實現並集（Union）、交集（Intersect）和差集（Difference）。

```javascript
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 並集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```

如果想在遍歷操作中，同步改變原來的Set結構，目前沒有直接的方法，但有兩種變通方法。一種是利用原Set結構映射出一個新的結構，然後賦值給原來的Set結構；另一種是利用`Array.from`方法。

```javascript
// 方法一
let set = new Set([1, 2, 3]);
set = new Set([...set].map(val => val * 2));
// set的值是2, 4, 6

// 方法二
let set = new Set([1, 2, 3]);
set = new Set(Array.from(set, val => val * 2));
// set的值是2, 4, 6
```

上面程式碼提供了兩種方法，直接在遍歷操作中改變原來的Set結構。

## WeakSet

WeakSet結構與Set類似，也是不重複的值的集合。但是，它與Set有兩個區別。

首先，WeakSet的成員只能是物件，而不能是其他型別的值。

其次，WeakSet中的物件都是弱引用，即垃圾回收機制不考慮WeakSet對該物件的引用，也就是說，如果其他物件都不再引用該物件，那麼垃圾回收機制會自動回收該物件所佔用的記憶體，不考慮該物件還存在於WeakSet之中。這個特點意味著，無法引用WeakSet的成員，因此WeakSet是不可遍歷的。

```javascript
var ws = new WeakSet();
ws.add(1)
// TypeError: Invalid value used in weak set
ws.add(Symbol())
// TypeError: invalid value used in weak set
```

上面程式碼試圖向WeakSet新增一個數值和`Symbol`值，結果報錯，因為WeakSet只能放置物件。

WeakSet是一個建構函式，可以使用`new`命令，建立WeakSet資料結構。

```javascript
var ws = new WeakSet();
```

作為建構函式，WeakSet可以接受一個數組或類似陣列的物件作為引數。（實際上，任何具有iterable介面的物件，都可以作為WeakSet的引數。）該陣列的所有成員，都會自動成為WeakSet實例物件的成員。

```javascript
var a = [[1,2], [3,4]];
var ws = new WeakSet(a);
```

上面程式碼中，`a`是一個數組，它有兩個成員，也都是陣列。將`a`作為WeakSet建構函式的引數，`a`的成員會自動成為WeakSet的成員。

注意，是`a`陣列的成員成為WeakSet的成員，而不是`a`陣列本身。這意味著，陣列的成員只能是物件。

```javascript
var b = [3, 4];
var ws = new WeakSet(b);
// Uncaught TypeError: Invalid value used in weak set(…)
```

上面程式碼中，陣列`b`的成員不是物件，加入WeaKSet就會報錯。

WeakSet結構有以下三個方法。

- **WeakSet.prototype.add(value)**：向WeakSet實例新增一個新成員。
- **WeakSet.prototype.delete(value)**：清除WeakSet實例的指定成員。
- **WeakSet.prototype.has(value)**：返回一個布林值，表示某個值是否在WeakSet實例之中。

下面是一個例子。

```javascript
var ws = new WeakSet();
var obj = {};
var foo = {};

ws.add(window);
ws.add(obj);

ws.has(window); // true
ws.has(foo);    // false

ws.delete(window);
ws.has(window);    // false
```

WeakSet沒有`size`屬性，沒有辦法遍歷它的成員。

```javascript
ws.size // undefined
ws.forEach // undefined

ws.forEach(function(item){ console.log('WeakSet has ' + item)})
// TypeError: undefined is not a function
```

上面程式碼試圖獲取`size`和`forEach`屬性，結果都不能成功。

WeakSet不能遍歷，是因為成員都是弱引用，隨時可能消失，遍歷機制無法保證成員的存在，很可能剛剛遍歷結束，成員就取不到了。WeakSet的一個用處，是儲存DOM節點，而不用擔心這些節點從文件移除時，會引發記憶體洩漏。

下面是WeakSet的另一個例子。

```javascript
const foos = new WeakSet()
class Foo {
  constructor() {
    foos.add(this)
  }
  method () {
    if (!foos.has(this)) {
      throw new TypeError('Foo.prototype.method 只能在Foo的實例上呼叫！');
    }
  }
}
```

上面程式碼保證了`Foo`的實例方法，只能在`Foo`的實例上呼叫。這裡使用WeakSet的好處是，`foos`對實例的引用，不會被計入記憶體回收機制，所以刪除實例的時候，不用考慮`foos`，也不會出現記憶體洩漏。

## Map

### Map結構的目的和基本用法

JavaScript的物件（Object），本質上是鍵值對的集合（Hash結構），但是傳統上只能用字串當作鍵。這給它的使用帶來了很大的限制。

```javascript
var data = {};
var element = document.getElementById('myDiv');

data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
```

上面程式碼原意是將一個DOM節點作為物件`data`的鍵，但是由於物件只接受字串作為鍵名，所以`element`被自動轉為字串`[object HTMLDivElement]`。

為了解決這個問題，ES6提供了Map資料結構。它類似於物件，也是鍵值對的集合，但是“鍵”的範圍不限於字串，各種型別的值（包括物件）都可以當作鍵。也就是說，Object結構提供了“字串—值”的對應，Map結構提供了“值—值”的對應，是一種更完善的Hash結構實現。如果你需要“鍵值對”的資料結構，Map比Object更合適。

```javascript
var m = new Map();
var o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```

上面程式碼使用`set`方法，將物件`o`當作`m`的一個鍵，然後又使用`get`方法讀取這個鍵，接著使用`delete`方法刪除了這個鍵。

作為建構函式，Map也可以接受一個數組作為引數。該陣列的成員是一個個表示鍵值對的陣列。

```javascript
var map = new Map([
  ['name', '張三'],
  ['title', 'Author']
]);

map.size // 2
map.has('name') // true
map.get('name') // "張三"
map.has('title') // true
map.get('title') // "Author"
```

上面程式碼在新建Map實例時，就指定了兩個鍵`name`和`title`。

Map建構函式接受陣列作為引數，實際上執行的是下面的演算法。

```javascript
var items = [
  ['name', '張三'],
  ['title', 'Author']
];
var map = new Map();
items.forEach(([key, value]) => map.set(key, value));
```

下面的例子中，字串`true`和布林值`true`是兩個不同的鍵。

```javascript
var m = new Map([
  [true, 'foo'],
  ['true', 'bar']
]);

m.get(true) // 'foo'
m.get('true') // 'bar'
```

如果對同一個鍵多次賦值，後面的值將覆蓋前面的值。

```javascript
let map = new Map();

map
.set(1, 'aaa')
.set(1, 'bbb');

map.get(1) // "bbb"
```

上面程式碼對鍵`1`連續賦值兩次，後一次的值覆蓋前一次的值。

如果讀取一個未知的鍵，則返回`undefined`。

```javascript
new Map().get('asfddfsasadf')
// undefined
```

注意，只有對同一個物件的引用，Map結構才將其視為同一個鍵。這一點要非常小心。

```javascript
var map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined
```

上面程式碼的`set`和`get`方法，表面是針對同一個鍵，但實際上這是兩個值，記憶體地址是不一樣的，因此`get`方法無法讀取該鍵，返回`undefined`。

同理，同樣的值的兩個實例，在Map結構中被視為兩個鍵。

```javascript
var map = new Map();

var k1 = ['a'];
var k2 = ['a'];

map
.set(k1, 111)
.set(k2, 222);

map.get(k1) // 111
map.get(k2) // 222
```

上面程式碼中，變數`k1`和`k2`的值是一樣的，但是它們在Map結構中被視為兩個鍵。

由上可知，Map的鍵實際上是跟記憶體地址繫結的，只要記憶體地址不一樣，就視為兩個鍵。這就解決了同名屬性碰撞（clash）的問題，我們擴展別人的庫的時候，如果使用物件作為鍵名，就不用擔心自己的屬性與原作者的屬性同名。

如果Map的鍵是一個簡單型別的值（數字、字串、布林值），則只要兩個值嚴格相等，Map將其視為一個鍵，包括`0`和`-0`。另外，雖然`NaN`不嚴格相等於自身，但Map將其視為同一個鍵。

```javascript
let map = new Map();

map.set(NaN, 123);
map.get(NaN) // 123

map.set(-0, 123);
map.get(+0) // 123
```

### 實例的屬性和操作方法

Map結構的實例有以下屬性和操作方法。

**（1）size屬性**

`size`屬性返回Map結構的成員總數。

```javascript
let map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
```

**（2）set(key, value)**

`set`方法設定`key`所對應的鍵值，然後返回整個Map結構。如果`key`已經有值，則鍵值會被更新，否則就新生成該鍵。

```javascript
var m = new Map();

m.set("edition", 6)        // 鍵是字串
m.set(262, "standard")     // 鍵是數值
m.set(undefined, "nah")    // 鍵是undefined
```

`set`方法返回的是Map本身，因此可以採用鏈式寫法。

```javascript
let map = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');
```

**（3）get(key)**

`get`方法讀取`key`對應的鍵值，如果找不到`key`，返回`undefined`。

```javascript
var m = new Map();

var hello = function() {console.log("hello");}
m.set(hello, "Hello ES6!") // 鍵是函式

m.get(hello)  // Hello ES6!
```

**（4）has(key)**

`has`方法返回一個布林值，表示某個鍵是否在Map資料結構中。

```javascript
var m = new Map();

m.set("edition", 6);
m.set(262, "standard");
m.set(undefined, "nah");

m.has("edition")     // true
m.has("years")       // false
m.has(262)           // true
m.has(undefined)     // true
```

**（5）delete(key)**

`delete`方法刪除某個鍵，返回true。如果刪除失敗，返回false。

```javascript
var m = new Map();
m.set(undefined, "nah");
m.has(undefined)     // true

m.delete(undefined)
m.has(undefined)       // false
```
**（6）clear()**

`clear`方法清除所有成員，沒有返回值。

```javascript
let map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
map.clear()
map.size // 0
```

### 遍歷方法

Map原生提供三個遍歷器生成函式和一個遍歷方法。

- `keys()`：返回鍵名的遍歷器。
- `values()`：返回鍵值的遍歷器。
- `entries()`：返回所有成員的遍歷器。
- `forEach()`：遍歷Map的所有成員。

需要特別注意的是，Map的遍歷順序就是插入順序。

下面是使用實例。

```javascript
let map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);

for (let key of map.keys()) {
  console.log(key);
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value);
}
// "no"
// "yes"

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"

// 或者
for (let [key, value] of map.entries()) {
  console.log(key, value);
}

// 等同於使用map.entries()
for (let [key, value] of map) {
  console.log(key, value);
}
```

上面程式碼最後的那個例子，表示Map結構的預設遍歷器介面（`Symbol.iterator`屬性），就是`entries`方法。

```javascript
map[Symbol.iterator] === map.entries
// true
```

Map結構轉為陣列結構，比較快速的方法是結合使用展開運算子（`...`）。

```javascript
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```

結合陣列的`map`方法、`filter`方法，可以實現Map的遍歷和過濾（Map本身沒有`map`和`filter`方法）。

```javascript
let map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

let map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// 產生Map結構 {1 => 'a', 2 => 'b'}

let map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// 產生Map結構 {2 => '_a', 4 => '_b', 6 => '_c'}
```

此外，Map還有一個`forEach`方法，與陣列的`forEach`方法類似，也可以實現遍歷。

```javascript
map.forEach(function(value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
```

`forEach`方法還可以接受第二個引數，用來繫結`this`。

```javascript
var reporter = {
  report: function(key, value) {
    console.log("Key: %s, Value: %s", key, value);
  }
};

map.forEach(function(value, key, map) {
  this.report(key, value);
}, reporter);
```

上面程式碼中，`forEach`方法的回呼函式的`this`，就指向`reporter`。

### 與其他資料結構的互相轉換

**（1）Map轉為陣列**

前面已經提過，Map轉為陣列最方便的方法，就是使用展開運算子（...）。

```javascript
let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
[...myMap]
// [ [ true, 7 ], [ { foo: 3 }, [ 'abc' ] ] ]
```

**（2）陣列轉為Map**

將陣列轉入Map建構函式，就可以轉為Map。

```javascript
new Map([[true, 7], [{foo: 3}, ['abc']]])
// Map {true => 7, Object {foo: 3} => ['abc']}
```

**（3）Map轉為物件**

如果所有Map的鍵都是字串，它可以轉為物件。

```javascript
function strMapToObj(strMap) {
  let obj = Object.create(null);
  for (let [k,v] of strMap) {
    obj[k] = v;
  }
  return obj;
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToObj(myMap)
// { yes: true, no: false }
```

**（4）物件轉為Map**

```javascript
function objToStrMap(obj) {
  let strMap = new Map();
  for (let k of Object.keys(obj)) {
    strMap.set(k, obj[k]);
  }
  return strMap;
}

objToStrMap({yes: true, no: false})
// [ [ 'yes', true ], [ 'no', false ] ]
```

**（5）Map轉為JSON**

Map轉為JSON要區分兩種情況。一種情況是，Map的鍵名都是字串，這時可以選擇轉為物件JSON。

```javascript
function strMapToJson(strMap) {
  return JSON.stringify(strMapToObj(strMap));
}

let myMap = new Map().set('yes', true).set('no', false);
strMapToJson(myMap)
// '{"yes":true,"no":false}'
```

另一種情況是，Map的鍵名有非字串，這時可以選擇轉為陣列JSON。

```javascript
function mapToArrayJson(map) {
  return JSON.stringify([...map]);
}

let myMap = new Map().set(true, 7).set({foo: 3}, ['abc']);
mapToArrayJson(myMap)
// '[[true,7],[{"foo":3},["abc"]]]'
```

**（6）JSON轉為Map**

JSON轉為Map，正常情況下，所有鍵名都是字串。

```javascript
function jsonToStrMap(jsonStr) {
  return objToStrMap(JSON.parse(jsonStr));
}

jsonToStrMap('{"yes":true,"no":false}')
// Map {'yes' => true, 'no' => false}
```

但是，有一種特殊情況，整個JSON就是一個數組，且每個陣列成員本身，又是一個有兩個成員的陣列。這時，它可以一一對應地轉為Map。這往往是陣列轉為JSON的逆操作。

```javascript
function jsonToMap(jsonStr) {
  return new Map(JSON.parse(jsonStr));
}

jsonToMap('[[true,7],[{"foo":3},["abc"]]]')
// Map {true => 7, Object {foo: 3} => ['abc']}
```

## WeakMap

`WeakMap`結構與`Map`結構基本類似，唯一的區別是它只接受物件作為鍵名（`null`除外），不接受其他型別的值作為鍵名，而且鍵名所指向的物件，不計入垃圾回收機制。

```javascript
var map = new WeakMap()
map.set(1, 2)
// TypeError: 1 is not an object!
map.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key
```

上面程式碼中，如果將`1`和`Symbol`作為WeakMap的鍵名，都會報錯。

`WeakMap`的設計目的在於，鍵名是物件的弱引用（垃圾回收機制不將該引用考慮在內），所以其所對應的物件可能會被自動回收。當物件被回收後，`WeakMap`自動移除對應的鍵值對。典型應用是，一個對應DOM元素的`WeakMap`結構，當某個DOM元素被清除，其所對應的`WeakMap`記錄就會自動被移除。基本上，`WeakMap`的專用場合就是，它的鍵所對應的物件，可能會在將來消失。`WeakMap`結構有助於防止記憶體洩漏。

下面是`WeakMap`結構的一個例子，可以看到用法上與`Map`幾乎一樣。

```javascript
var wm = new WeakMap();
var element = document.querySelector(".element");

wm.set(element, "Original");
wm.get(element) // "Original"

element.parentNode.removeChild(element);
element = null;
wm.get(element) // undefined
```

上面程式碼中，變數`wm`是一個`WeakMap`實例，我們將一個`DOM`節點`element`作為鍵名，然後銷燬這個節點，`element`對應的鍵就自動消失了，再引用這個鍵名就返回`undefined`。

WeakMap與Map在API上的區別主要是兩個，一是沒有遍歷操作（即沒有`key()`、`values()`和`entries()`方法），也沒有`size`屬性；二是無法清空，即不支援`clear`方法。這與`WeakMap`的鍵不被計入引用、被垃圾回收機制忽略有關。因此，`WeakMap`只有四個方法可用：`get()`、`set()`、`has()`、`delete()`。

```javascript
var wm = new WeakMap();

wm.size
// undefined

wm.forEach
// undefined
```

前文說過，WeakMap應用的典型場合就是DOM節點作為鍵名。下面是一個例子。

```javascript
let myElement = document.getElementById('logo');
let myWeakmap = new WeakMap();

myWeakmap.set(myElement, {timesClicked: 0});

myElement.addEventListener('click', function() {
  let logoData = myWeakmap.get(myElement);
  logoData.timesClicked++;
}, false);
```

上面程式碼中，`myElement`是一個 DOM 節點，每當發生`click`事件，就更新一下狀態。我們將這個狀態作為鍵值放在 WeakMap 裡，對應的鍵名就是`myElement`。一旦這個 DOM 節點刪除，該狀態就會自動消失，不存在記憶體洩漏風險。

WeakMap 的另一個用處是部署私有屬性。

```javascript
let _counter = new WeakMap();
let _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    if (counter < 1) return;
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}

let c = new Countdown(2, () => console.log('DONE'));

c.dec()
c.dec()
// DONE
```

上面程式碼中，Countdown類的兩個內部屬性`_counter`和`_action`，是實例的弱引用，所以如果刪除實例，它們也就隨之消失，不會造成記憶體洩漏。
