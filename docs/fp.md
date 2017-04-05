# 函數語言程式設計

JavaScript語言從一誕生，就具有函數語言程式設計的烙印。它將函式作為一種獨立的資料型別，與其他資料型別處於完全平等的地位。在JavaScript語言中，你可以採用面向物件程式設計，也可以採用函數語言程式設計。有人甚至說，JavaScript是有史以來第一種被大規模採用的函數語言程式設計語言。

ES6的種種新增功能，使得函數語言程式設計變得更方便、更強大。本章介紹ES6如何進行函數語言程式設計。

## 柯里化

柯里化（currying）指的是將一個多引數的函式拆分成一系列函式，每個拆分後的函式都只接受一個引數（unary）。

```javascript
function add (a, b) {
  return a + b;
}

add(1, 1) // 2
```

上面程式碼中，函式`add`接受兩個引數`a`和`b`。

柯里化就是將上面的函式拆分成兩個函式，每個函式都只接受一個引數。

```javascript
function add (a) {
  return function (b) {
    return a + b;
  }
}
// 或者採用箭頭函式寫法
const add = x => y => x + y;

const f = add(1);
f(1) // 2
```

上面程式碼中，函式`add`只接受一個引數`a`，返回一個函式`f`。函式`f`也只接受一個引數`b`。

## 函式合成

函式合成（function composition）指的是，將多個函式合成一個函式。

```javascript
const compose = f => g => x => f(g(x));

const f = compose (x => x * 4) (x => x + 3);
f(2) // 20
```

上面程式碼中，`compose`就是一個函式合成器，用於將兩個函式合成一個函式。

可以發現，柯里化與函式合成有著密切的聯絡。前者用於將一個函式拆成多個函式，後者用於將多個函式合併成一個函式。

## 引數倒置

引數倒置（flip）指的是改變函式前兩個引數的順序。

```javascript
var divide = (a, b) => a / b;
var flip = f.flip(divide);

flip(10, 5) // 0.5
flip(1, 10) // 10

var three = (a, b, c) => [a, b, c];
var flip = f.flip(three);
flip(1, 2, 3); // => [2, 1, 3]
```

上面程式碼中，如果按照正常的引數順序，10除以5等於2。但是，引數倒置以後得到的新函式，結果就是5除以10，結果得到0.5。如果原函式有3個引數，則只顛倒前兩個引數的位置。

引數倒置的程式碼非常簡單。

```javascript
let f = {};
f.flip =
  fn =>
    (a, b, ...args) => fn(b, a, ...args.reverse());
```

## 執行邊界

執行邊界（until）指的是函式執行到滿足條件為止。

```javascript
let condition = x => x > 100;
let inc = x => x + 1;
let until = f.until(condition, inc);

until(0) // 101

condition = x => x === 5;
until = f.until(condition, inc);

until(3) // 5
```

上面程式碼中，第一段的條件是執行到`x`大於100為止，所以`x`初值為0時，會一直執行到101。第二段的條件是執行到等於5為止，所以`x`最後的值是5。

執行邊界的實現如下。

```javascript
let f = {};
f.until = (condition, f) =>
  (...args) => {
    var r = f.apply(null, args);
    return condition(r) ? r : f.until(condition, f)(r);
  };
```

上面程式碼的關鍵就是，如果滿足條件就返回結果，否則不斷遞迴執行。

## 佇列操作

佇列（list）操作包括以下幾種。

- `head`： 取出佇列的第一個非空成員。
- `last`： 取出有限佇列的最後一個非空成員。
- `tail`： 取出除了“佇列頭”以外的其他非空成員。
- `init`： 取出除了“佇列尾”以外的其他非空成員。

下面是例子。

```javascript
f.head(5, 27, 3, 1) // 5
f.last(5, 27, 3, 1) // 1
f.tail(5, 27, 3, 1) // [27, 3, 1]
f.init(5, 27, 3, 1) // [5, 27, 3]
```

這些方法的實現如下。

```javascript
let f = {};
f.head = (...xs) => xs[0];
f.last = (...xs) => xs.slice(-1);
f.tail = (...xs) => Array.prototype.slice.call(xs, 1);
f.init = (...xs) => xs.slice(0, -1);
```

## 合併操作

合併操作分為`concat`和`concatMap`兩種。前者就是將多個數組合成一個，後者則是先處理一下引數，然後再將處理結果合成一個數組。

```javascript
f.concat([5], [27], [3]) // [5, 27, 3]
f.concatMap(x => 'hi ' + x, 1, [[2]], 3) // ['hi 1', 'hi 2', 'hi 3']
```

這兩種方法的實現程式碼如下。

```javascript
let f = {};
f.concat =
  (...xs) => xs.reduce((a, b) => a.concat(b));
f.concatMap =
  (f, ...xs) => f.concat(xs.map(f));
```

## 配對操作

配對操作分為`zip`和`zipWith`兩種方法。`zip`操作將兩個佇列的成員，一一配對，合成一個新的佇列。如果兩個佇列不等長，較長的那個佇列多出來的成員，會被忽略。`zipWith`操作的第一個引數是一個函式，然後會將後面的佇列成員一一配對，輸入該函式，返回值就組成一個新的佇列。

下面是例子。

```javascript
let a = [0, 1, 2];
let b = [3, 4, 5];
let c = [6, 7, 8];

f.zip(a, b) // [[0, 3], [1, 4], [2, 5]]
f.zipWith((a, b) => a + b, a, b, c) // [9, 12, 15]
```

上面程式碼中，`zipWith`方法的第一個引數是一個求和函式，它將後面三個佇列的成員，一一配對進行相加。

這兩個方法的實現如下。

```javascript
let f = {};

f.zip = (...xs) => {
  let r = [];
  let nple = [];
  let length = Math.min.apply(null, xs.map(x => x.length));

  for (var i = 0; i < length; i++) {
    xs.forEach(
      x => nple.push(x[i])
    );

    r.push(nple);
    nple = [];
  }

  return r;
};

f.zipWith = (op, ...xs) =>
  f.zip.apply(null, xs).map(
    (x) => x.reduce(op)
  );
```

## 參考連結

- Mateo Gianolio, [Haskell in ES6: Part 1](http://casualjavascript.com/?1)
