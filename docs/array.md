# 陣列的擴充套件

## Array.from()

`Array.from`方法用於將兩類物件轉為真正的陣列：類似陣列的物件（array-like object）和可遍歷（iterable）的物件（包括ES6新增的資料結構Set和Map）。

下面是一個類似陣列的物件，`Array.from`將它轉為真正的陣列。

```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5的寫法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ES6的寫法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

實際應用中，常見的類似陣列的物件是DOM操作返回的NodeList集合，以及函式內部的`arguments`物件。`Array.from`都可以將它們轉為真正的陣列。

```javascript
// NodeList物件
let ps = document.querySelectorAll('p');
Array.from(ps).forEach(function (p) {
  console.log(p);
});

// arguments物件
function foo() {
  var args = Array.from(arguments);
  // ...
}
```

上面程式碼中，`querySelectorAll`方法返回的是一個類似陣列的物件，只有將這個物件轉為真正的陣列，才能使用`forEach`方法。

只要是部署了Iterator介面的資料結構，`Array.from`都能將其轉為陣列。

```javascript
Array.from('hello')
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']
```

上面程式碼中，字串和Set結構都具有Iterator介面，因此可以被`Array.from`轉為真正的陣列。

如果引數是一個真正的陣列，`Array.from`會返回一個一模一樣的新陣列。

```javascript
Array.from([1, 2, 3])
// [1, 2, 3]
```

值得提醒的是，擴充套件運算子（`...`）也可以將某些資料結構轉為陣列。

```javascript
// arguments物件
function foo() {
  var args = [...arguments];
}

// NodeList物件
[...document.querySelectorAll('div')]
```

擴充套件運算子背後呼叫的是遍歷器介面（`Symbol.iterator`），如果一個物件沒有部署這個介面，就無法轉換。`Array.from`方法則是還支援類似陣列的物件。所謂類似陣列的物件，本質特徵只有一點，即必須有`length`屬性。因此，任何有`length`屬性的物件，都可以通過`Array.from`方法轉為陣列，而此時擴充套件運算子就無法轉換。

```javascript
Array.from({ length: 3 });
// [ undefined, undefined, undefined ]
```

上面程式碼中，`Array.from`返回了一個具有三個成員的陣列，每個位置的值都是`undefined`。擴充套件運算子轉換不了這個物件。

對於還沒有部署該方法的瀏覽器，可以用`Array.prototype.slice`方法替代。

```javascript
const toArray = (() =>
  Array.from ? Array.from : obj => [].slice.call(obj)
)();
```

`Array.from`還可以接受第二個引數，作用類似於陣列的`map`方法，用來對每個元素進行處理，將處理後的值放入返回的陣列。

```javascript
Array.from(arrayLike, x => x * x);
// 等同於
Array.from(arrayLike).map(x => x * x);

Array.from([1, 2, 3], (x) => x * x)
// [1, 4, 9]
```

下面的例子是取出一組DOM節點的文字內容。

```javascript
let spans = document.querySelectorAll('span.name');

// map()
let names1 = Array.prototype.map.call(spans, s => s.textContent);

// Array.from()
let names2 = Array.from(spans, s => s.textContent)
```

下面的例子將陣列中布林值為`false`的成員轉為`0`。

```javascript
Array.from([1, , 2, , 3], (n) => n || 0)
// [1, 0, 2, 0, 3]
```

另一個例子是返回各種資料的型別。

```javascript
function typesOf () {
  return Array.from(arguments, value => typeof value)
}
typesOf(null, [], NaN)
// ['object', 'object', 'number']
```

如果`map`函式裡面用到了`this`關鍵字，還可以傳入`Array.from`的第三個引數，用來繫結`this`。

`Array.from()`可以將各種值轉為真正的陣列，並且還提供`map`功能。這實際上意味著，只要有一個原始的資料結構，你就可以先對它的值進行處理，然後轉成規範的陣列結構，進而就可以使用數量眾多的陣列方法。

```javascript
Array.from({ length: 2 }, () => 'jack')
// ['jack', 'jack']
```

上面程式碼中，`Array.from`的第一個引數指定了第二個引數執行的次數。這種特性可以讓該方法的用法變得非常靈活。

`Array.from()`的另一個應用是，將字串轉為陣列，然後返回字串的長度。因為它能正確處理各種Unicode字元，可以避免JavaScript將大於`\uFFFF`的Unicode字元，算作兩個字元的bug。

```javascript
function countSymbols(string) {
  return Array.from(string).length;
}
```

## Array.of()

`Array.of`方法用於將一組值，轉換為陣列。

```javascript
Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1
```

這個方法的主要目的，是彌補陣列建構函式`Array()`的不足。因為引數個數的不同，會導致`Array()`的行為有差異。

```javascript
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```

上面程式碼中，`Array`方法沒有引數、一個引數、三個引數時，返回結果都不一樣。只有當引數個數不少於2個時，`Array()`才會返回由引數組成的新陣列。引數個數只有一個時，實際上是指定陣列的長度。

`Array.of`基本上可以用來替代`Array()`或`new Array()`，並且不存在由於引數不同而導致的過載。它的行為非常統一。

```javascript
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```

`Array.of`總是返回引數值組成的陣列。如果沒有引數，就返回一個空陣列。

`Array.of`方法可以用下面的程式碼模擬實現。

```javascript
function ArrayOf(){
  return [].slice.call(arguments);
}
```

## 陣列例項的copyWithin()

陣列例項的`copyWithin`方法，在當前陣列內部，將指定位置的成員複製到其他位置（會覆蓋原有成員），然後返回當前陣列。也就是說，使用這個方法，會修改當前陣列。

```javascript
Array.prototype.copyWithin(target, start = 0, end = this.length)
```

它接受三個引數。

- target（必需）：從該位置開始替換資料。
- start（可選）：從該位置開始讀取資料，預設為0。如果為負值，表示倒數。
- end（可選）：到該位置前停止讀取資料，預設等於陣列長度。如果為負值，表示倒數。

這三個引數都應該是數值，如果不是，會自動轉為數值。

```javascript
[1, 2, 3, 4, 5].copyWithin(0, 3)
// [4, 5, 3, 4, 5]
```

上面程式碼表示將從3號位直到陣列結束的成員（4和5），複製到從0號位開始的位置，結果覆蓋了原來的1和2。

下面是更多例子。

```javascript
// 將3號位複製到0號位
[1, 2, 3, 4, 5].copyWithin(0, 3, 4)
// [4, 2, 3, 4, 5]

// -2相當於3號位，-1相當於4號位
[1, 2, 3, 4, 5].copyWithin(0, -2, -1)
// [4, 2, 3, 4, 5]

// 將3號位複製到0號位
[].copyWithin.call({length: 5, 3: 1}, 0, 3)
// {0: 1, 3: 1, length: 5}

// 將2號位到陣列結束，複製到0號位
var i32a = new Int32Array([1, 2, 3, 4, 5]);
i32a.copyWithin(0, 2);
// Int32Array [3, 4, 5, 4, 5]

// 對於沒有部署TypedArray的copyWithin方法的平臺
// 需要採用下面的寫法
[].copyWithin.call(new Int32Array([1, 2, 3, 4, 5]), 0, 3, 4);
// Int32Array [4, 2, 3, 4, 5]
```

## 陣列例項的find()和findIndex()

陣列例項的`find`方法，用於找出第一個符合條件的陣列成員。它的引數是一個回撥函式，所有陣列成員依次執行該回調函式，直到找出第一個返回值為`true`的成員，然後返回該成員。如果沒有符合條件的成員，則返回`undefined`。

```javascript
[1, 4, -5, 10].find((n) => n < 0)
// -5
```

上面程式碼找出陣列中第一個小於0的成員。

```javascript
[1, 5, 10, 15].find(function(value, index, arr) {
  return value > 9;
}) // 10
```

上面程式碼中，`find`方法的回撥函式可以接受三個引數，依次為當前的值、當前的位置和原陣列。

陣列例項的`findIndex`方法的用法與`find`方法非常類似，返回第一個符合條件的陣列成員的位置，如果所有成員都不符合條件，則返回`-1`。

```javascript
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```

這兩個方法都可以接受第二個引數，用來繫結回撥函式的`this`物件。

另外，這兩個方法都可以發現`NaN`，彌補了陣列的`IndexOf`方法的不足。

```javascript
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
// 0
```

上面程式碼中，`indexOf`方法無法識別陣列的`NaN`成員，但是`findIndex`方法可以藉助`Object.is`方法做到。

## 陣列例項的fill()

`fill`方法使用給定值，填充一個數組。

```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
```

上面程式碼表明，`fill`方法用於空陣列的初始化非常方便。陣列中已有的元素，會被全部抹去。

`fill`方法還可以接受第二個和第三個引數，用於指定填充的起始位置和結束位置。

```javascript
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']
```

上面程式碼表示，`fill`方法從1號位開始，向原陣列填充7，到2號位之前結束。

## 陣列例項的entries()，keys()和values()

ES6提供三個新的方法——`entries()`，`keys()`和`values()`——用於遍歷陣列。它們都返回一個遍歷器物件（詳見《Iterator》一章），可以用`for...of`迴圈進行遍歷，唯一的區別是`keys()`是對鍵名的遍歷、`values()`是對鍵值的遍歷，`entries()`是對鍵值對的遍歷。

```javascript
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

如果不使用`for...of`迴圈，可以手動呼叫遍歷器物件的`next`方法，進行遍歷。

```javascript
let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']
```

## 陣列例項的includes()

`Array.prototype.includes`方法返回一個布林值，表示某個陣列是否包含給定的值，與字串的`includes`方法類似。該方法屬於ES7，但Babel轉碼器已經支援。

```javascript
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, NaN].includes(NaN); // true
```

該方法的第二個引數表示搜尋的起始位置，預設為0。如果第二個引數為負數，則表示倒數的位置，如果這時它大於陣列長度（比如第二個引數為-4，但陣列長度為3），則會重置為從0開始。

```javascript
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
```

沒有該方法之前，我們通常使用陣列的`indexOf`方法，檢查是否包含某個值。

```javascript
if (arr.indexOf(el) !== -1) {
  // ...
}
```

`indexOf`方法有兩個缺點，一是不夠語義化，它的含義是找到引數值的第一個出現位置，所以要去比較是否不等於-1，表達起來不夠直觀。二是，它內部使用嚴格相當運算子（===）進行判斷，這會導致對`NaN`的誤判。

```javascript
[NaN].indexOf(NaN)
// -1
```

`includes`使用的是不一樣的判斷演算法，就沒有這個問題。

```javascript
[NaN].includes(NaN)
// true
```

下面程式碼用來檢查當前環境是否支援該方法，如果不支援，部署一個簡易的替代版本。

```javascript
const contains = (() =>
  Array.prototype.includes
    ? (arr, value) => arr.includes(value)
    : (arr, value) => arr.some(el => el === value)
)();
contains(["foo", "bar"], "baz"); // => false
```

另外，Map和Set資料結構有一個`has`方法，需要注意與`includes`區分。

- Map結構的`has`方法，是用來查詢鍵名的，比如`Map.prototype.has(key)`、`WeakMap.prototype.has(key)`、`Reflect.has(target, propertyKey)`。
- Set結構的`has`方法，是用來查詢值的，比如`Set.prototype.has(value)`、`WeakSet.prototype.has(value)`。

## 陣列的空位

陣列的空位指，陣列的某一個位置沒有任何值。比如，`Array`建構函式返回的陣列都是空位。

```javascript
Array(3) // [, , ,]
```

上面程式碼中，`Array(3)`返回一個具有3個空位的陣列。

注意，空位不是`undefined`，一個位置的值等於`undefined`，依然是有值的。空位是沒有任何值，`in`運算子可以說明這一點。

```javascript
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
```

上面程式碼說明，第一個陣列的0號位置是有值的，第二個陣列的0號位置沒有值。

ES5對空位的處理，已經很不一致了，大多數情況下會忽略空位。

- `forEach()`, `filter()`, `every()` 和`some()`都會跳過空位。
- `map()`會跳過空位，但會保留這個值
- `join()`和`toString()`會將空位視為`undefined`，而`undefined`和`null`會被處理成空字串。

```javascript
// forEach方法
[,'a'].forEach((x,i) => console.log(i)); // 1

// filter方法
['a',,'b'].filter(x => true) // ['a','b']

// every方法
[,'a'].every(x => x==='a') // true

// some方法
[,'a'].some(x => x !== 'a') // false

// map方法
[,'a'].map(x => 1) // [,1]

// join方法
[,'a',undefined,null].join('#') // "#a##"

// toString方法
[,'a',undefined,null].toString() // ",a,,"
```

ES6則是明確將空位轉為`undefined`。

`Array.from`方法會將陣列的空位，轉為`undefined`，也就是說，這個方法不會忽略空位。

```javascript
Array.from(['a',,'b'])
// [ "a", undefined, "b" ]
```

擴充套件運算子（`...`）也會將空位轉為`undefined`。

```javascript
[...['a',,'b']]
// [ "a", undefined, "b" ]
```

`copyWithin()`會連空位一起拷貝。

```javascript
[,'a','b',,].copyWithin(2,0) // [,"a",,"a"]
```

`fill()`會將空位視為正常的陣列位置。

```javascript
new Array(3).fill('a') // ["a","a","a"]
```

`for...of`迴圈也會遍歷空位。

```javascript
let arr = [, ,];
for (let i of arr) {
  console.log(1);
}
// 1
// 1
```

上面程式碼中，陣列`arr`有兩個空位，`for...of`並沒有忽略它們。如果改成`map`方法遍歷，空位是會跳過的。

`entries()`、`keys()`、`values()`、`find()`和`findIndex()`會將空位處理成`undefined`。

```javascript
// entries()
[...[,'a'].entries()] // [[0,undefined], [1,"a"]]

// keys()
[...[,'a'].keys()] // [0,1]

// values()
[...[,'a'].values()] // [undefined,"a"]

// find()
[,'a'].find(x => true) // undefined

// findIndex()
[,'a'].findIndex(x => true) // 0
```

由於空位的處理規則非常不統一，所以建議避免出現空位。

