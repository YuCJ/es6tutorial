# 數值的擴展

## 二進位制和八進位制表示法

ES6 提供了二進位制和八進位制數值的新的寫法，分別用字首`0b`（或`0B`）和`0o`（或`0O`）表示。

```javascript
0b111110111 === 503 // true
0o767 === 503 // true
```

從 ES5 開始，在嚴格模式之中，八進位制就不再允許使用字首`0`表示，ES6 進一步明確，要使用字首`0o`表示。

```javascript
// 非嚴格模式
(function(){
  console.log(0o11 === 011);
})() // true

// 嚴格模式
(function(){
  'use strict';
  console.log(0o11 === 011);
})() // Uncaught SyntaxError: Octal literals are not allowed in strict mode.
```

如果要將`0b`和`0o`字首的字串數值轉為十進位制，要使用`Number`方法。

```javascript
Number('0b111')  // 7
Number('0o10')  // 8
```

## Number.isFinite(), Number.isNaN()

ES6在Number物件上，新提供了`Number.isFinite()`和`Number.isNaN()`兩個方法。

`Number.isFinite()`用來檢查一個數值是否為有限的（finite）。

```javascript
Number.isFinite(15); // true
Number.isFinite(0.8); // true
Number.isFinite(NaN); // false
Number.isFinite(Infinity); // false
Number.isFinite(-Infinity); // false
Number.isFinite('foo'); // false
Number.isFinite('15'); // false
Number.isFinite(true); // false
```

ES5可以通過下面的程式碼，部署`Number.isFinite`方法。

```javascript
(function (global) {
  var global_isFinite = global.isFinite;

  Object.defineProperty(Number, 'isFinite', {
    value: function isFinite(value) {
      return typeof value === 'number' && global_isFinite(value);
    },
    configurable: true,
    enumerable: false,
    writable: true
  });
})(this);
```

`Number.isNaN()`用來檢查一個值是否為`NaN`。

```javascript
Number.isNaN(NaN) // true
Number.isNaN(15) // false
Number.isNaN('15') // false
Number.isNaN(true) // false
Number.isNaN(9/NaN) // true
Number.isNaN('true'/0) // true
Number.isNaN('true'/'true') // true
```

ES5通過下面的程式碼，部署`Number.isNaN()`。

```javascript
(function (global) {
  var global_isNaN = global.isNaN;

  Object.defineProperty(Number, 'isNaN', {
    value: function isNaN(value) {
      return typeof value === 'number' && global_isNaN(value);
    },
    configurable: true,
    enumerable: false,
    writable: true
  });
})(this);
```

它們與傳統的全域性方法`isFinite()`和`isNaN()`的區別在於，傳統方法先呼叫`Number()`將非數值的值轉為數值，再進行判斷，而這兩個新方法只對數值有效，`Number.isFinite()`對於非數值一律返回`false`, `Number.isNaN()`只有對於`NaN`才返回`true`，非`NaN`一律返回`false`。

```javascript
isFinite(25) // true
isFinite("25") // true
Number.isFinite(25) // true
Number.isFinite("25") // false

isNaN(NaN) // true
isNaN("NaN") // true
Number.isNaN(NaN) // true
Number.isNaN("NaN") // false
Number.isNaN(1) // false
```

## Number.parseInt(), Number.parseFloat()

ES6將全域性方法`parseInt()`和`parseFloat()`，移植到Number物件上面，行為完全保持不變。

```javascript
// ES5的寫法
parseInt('12.34') // 12
parseFloat('123.45#') // 123.45

// ES6的寫法
Number.parseInt('12.34') // 12
Number.parseFloat('123.45#') // 123.45
```

這樣做的目的，是逐步減少全域性性方法，使得語言逐步模組化。

```javascript
Number.parseInt === parseInt // true
Number.parseFloat === parseFloat // true
```

## Number.isInteger()

`Number.isInteger()`用來判斷一個值是否為整數。需要注意的是，在JavaScript內部，整數和浮點數是同樣的儲存方法，所以3和3.0被視為同一個值。

```javascript
Number.isInteger(25) // true
Number.isInteger(25.0) // true
Number.isInteger(25.1) // false
Number.isInteger("15") // false
Number.isInteger(true) // false
```

ES5可以通過下面的程式碼，部署`Number.isInteger()`。

```javascript
(function (global) {
  var floor = Math.floor,
    isFinite = global.isFinite;

  Object.defineProperty(Number, 'isInteger', {
    value: function isInteger(value) {
      return typeof value === 'number' && isFinite(value) &&
        value > -9007199254740992 && value < 9007199254740992 &&
        floor(value) === value;
    },
    configurable: true,
    enumerable: false,
    writable: true
  });
})(this);
```

## Number.EPSILON

ES6在Number物件上面，新增一個極小的常量`Number.EPSILON`。

```javascript
Number.EPSILON
// 2.220446049250313e-16
Number.EPSILON.toFixed(20)
// '0.00000000000000022204'
```

引入一個這麼小的量的目的，在於為浮點數計算，設定一個誤差範圍。我們知道浮點數計算是不精確的。

```javascript
0.1 + 0.2
// 0.30000000000000004

0.1 + 0.2 - 0.3
// 5.551115123125783e-17

5.551115123125783e-17.toFixed(20)
// '0.00000000000000005551'
```

但是如果這個誤差能夠小於`Number.EPSILON`，我們就可以認為得到了正確結果。

```javascript
5.551115123125783e-17 < Number.EPSILON
// true
```

因此，`Number.EPSILON`的實質是一個可以接受的誤差範圍。

```javascript
function withinErrorMargin (left, right) {
  return Math.abs(left - right) < Number.EPSILON;
}
withinErrorMargin(0.1 + 0.2, 0.3)
// true
withinErrorMargin(0.2 + 0.2, 0.3)
// false
```

上面的程式碼為浮點數運算，部署了一個誤差檢查函式。

## 安全整數和Number.isSafeInteger()

JavaScript能夠準確表示的整數範圍在`-2^53`到`2^53`之間（不含兩個端點），超過這個範圍，無法精確表示這個值。

```javascript
Math.pow(2, 53) // 9007199254740992

9007199254740992  // 9007199254740992
9007199254740993  // 9007199254740992

Math.pow(2, 53) === Math.pow(2, 53) + 1
// true
```

上面程式碼中，超出2的53次方之後，一個數就不精確了。

ES6引入了`Number.MAX_SAFE_INTEGER`和`Number.MIN_SAFE_INTEGER`這兩個常量，用來表示這個範圍的上下限。

```javascript
Number.MAX_SAFE_INTEGER === Math.pow(2, 53) - 1
// true
Number.MAX_SAFE_INTEGER === 9007199254740991
// true

Number.MIN_SAFE_INTEGER === -Number.MAX_SAFE_INTEGER
// true
Number.MIN_SAFE_INTEGER === -9007199254740991
// true
```

上面程式碼中，可以看到JavaScript能夠精確表示的極限。

`Number.isSafeInteger()`則是用來判斷一個整數是否落在這個範圍之內。

```javascript
Number.isSafeInteger('a') // false
Number.isSafeInteger(null) // false
Number.isSafeInteger(NaN) // false
Number.isSafeInteger(Infinity) // false
Number.isSafeInteger(-Infinity) // false

Number.isSafeInteger(3) // true
Number.isSafeInteger(1.2) // false
Number.isSafeInteger(9007199254740990) // true
Number.isSafeInteger(9007199254740992) // false

Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1) // false
Number.isSafeInteger(Number.MIN_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER) // true
Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1) // false
```

這個函式的實現很簡單，就是跟安全整數的兩個邊界值比較一下。

```javascript
Number.isSafeInteger = function (n) {
  return (typeof n === 'number' &&
    Math.round(n) === n &&
    Number.MIN_SAFE_INTEGER <= n &&
    n <= Number.MAX_SAFE_INTEGER);
}
```

實際使用這個函式時，需要注意。驗證運算結果是否落在安全整數的範圍內，不要只驗證運算結果，而要同時驗證參與運算的每個值。

```javascript
Number.isSafeInteger(9007199254740993)
// false
Number.isSafeInteger(990)
// true
Number.isSafeInteger(9007199254740993 - 990)
// true
9007199254740993 - 990
// 返回結果 9007199254740002
// 正確答案應該是 9007199254740003
```

上面程式碼中，`9007199254740993`不是一個安全整數，但是`Number.isSafeInteger`會返回結果，顯示計算結果是安全的。這是因為，這個數超出了精度範圍，導致在計算機內部，以`9007199254740992`的形式儲存。

```javascript
9007199254740993 === 9007199254740992
// true
```

所以，如果只驗證運算結果是否為安全整數，很可能得到錯誤結果。下面的函式可以同時驗證兩個運算數和運算結果。

```javascript
function trusty (left, right, result) {
  if (
    Number.isSafeInteger(left) &&
    Number.isSafeInteger(right) &&
    Number.isSafeInteger(result)
  ) {
    return result;
  }
  throw new RangeError('Operation cannot be trusted!');
}

trusty(9007199254740993, 990, 9007199254740993 - 990)
// RangeError: Operation cannot be trusted!

trusty(1, 2, 3)
// 3
```

## Math物件的擴展

ES6在Math物件上新增了17個與數學相關的方法。所有這些方法都是靜態方法，只能在Math物件上呼叫。

### Math.trunc()

`Math.trunc`方法用於去除一個數的小數部分，返回整數部分。

```javascript
Math.trunc(4.1) // 4
Math.trunc(4.9) // 4
Math.trunc(-4.1) // -4
Math.trunc(-4.9) // -4
Math.trunc(-0.1234) // -0
```

對於非數值，`Math.trunc`內部使用`Number`方法將其先轉為數值。

```javascript
Math.trunc('123.456')
// 123
```

對於空值和無法擷取整數的值，返回NaN。

```javascript
Math.trunc(NaN);      // NaN
Math.trunc('foo');    // NaN
Math.trunc();         // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.trunc = Math.trunc || function(x) {
  return x < 0 ? Math.ceil(x) : Math.floor(x);
};
```

### Math.sign()

`Math.sign`方法用來判斷一個數到底是正數、負數、還是零。

它會返回五種值。

- 引數為正數，返回+1；
- 引數為負數，返回-1；
- 引數為0，返回0；
- 引數為-0，返回-0;
- 其他值，返回NaN。

```javascript
Math.sign(-5) // -1
Math.sign(5) // +1
Math.sign(0) // +0
Math.sign(-0) // -0
Math.sign(NaN) // NaN
Math.sign('foo'); // NaN
Math.sign();      // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.sign = Math.sign || function(x) {
  x = +x; // convert to a number
  if (x === 0 || isNaN(x)) {
    return x;
  }
  return x > 0 ? 1 : -1;
};
```

### Math.cbrt()

`Math.cbrt`方法用於計算一個數的立方根。

```javascript
Math.cbrt(-1) // -1
Math.cbrt(0)  // 0
Math.cbrt(1)  // 1
Math.cbrt(2)  // 1.2599210498948734
```

對於非數值，`Math.cbrt`方法內部也是先使用`Number`方法將其轉為數值。

```javascript
Math.cbrt('8') // 2
Math.cbrt('hello') // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.cbrt = Math.cbrt || function(x) {
  var y = Math.pow(Math.abs(x), 1/3);
  return x < 0 ? -y : y;
};
```

### Math.clz32()

JavaScript的整數使用32位二進位制形式表示，`Math.clz32`方法返回一個數的32位無符號整數形式有多少個前導0。

```javascript
Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1000) // 22
Math.clz32(0b01000000000000000000000000000000) // 1
Math.clz32(0b00100000000000000000000000000000) // 2
```

上面程式碼中，0的二進位制形式全為0，所以有32個前導0；1的二進位制形式是`0b1`，只佔1位，所以32位之中有31個前導0；1000的二進位制形式是`0b1111101000`，一共有10位，所以32位之中有22個前導0。

`clz32`這個函式名就來自”count leading zero bits in 32-bit binary representations of a number“（計算32位整數的前導0）的縮寫。

左移運算子（`<<`）與`Math.clz32`方法直接相關。

```javascript
Math.clz32(0) // 32
Math.clz32(1) // 31
Math.clz32(1 << 1) // 30
Math.clz32(1 << 2) // 29
Math.clz32(1 << 29) // 2
```

對於小數，`Math.clz32`方法只考慮整數部分。

```javascript
Math.clz32(3.2) // 30
Math.clz32(3.9) // 30
```

對於空值或其他型別的值，`Math.clz32`方法會將它們先轉為數值，然後再計算。

```javascript
Math.clz32() // 32
Math.clz32(NaN) // 32
Math.clz32(Infinity) // 32
Math.clz32(null) // 32
Math.clz32('foo') // 32
Math.clz32([]) // 32
Math.clz32({}) // 32
Math.clz32(true) // 31
```

### Math.imul()

`Math.imul`方法返回兩個數以32位帶符號整數形式相乘的結果，返回的也是一個32位的帶符號整數。

```javascript
Math.imul(2, 4)   // 8
Math.imul(-1, 8)  // -8
Math.imul(-2, -2) // 4
```

如果只考慮最後32位，大多數情況下，`Math.imul(a, b)`與`a * b`的結果是相同的，即該方法等同於`(a * b)|0`的效果（超過32位的部分溢位）。之所以需要部署這個方法，是因為JavaScript有精度限制，超過2的53次方的值無法精確表示。這就是說，對於那些很大的數的乘法，低位數值往往都是不精確的，`Math.imul`方法可以返回正確的低位數值。

```javascript
(0x7fffffff * 0x7fffffff)|0 // 0
```

上面這個乘法算式，返回結果為0。但是由於這兩個二進位制數的最低位都是1，所以這個結果肯定是不正確的，因為根據二進位制乘法，計算結果的二進位制最低位應該也是1。這個錯誤就是因為它們的乘積超過了2的53次方，JavaScript無法儲存額外的精度，就把低位的值都變成了0。`Math.imul`方法可以返回正確的值1。

```javascript
Math.imul(0x7fffffff, 0x7fffffff) // 1
```

### Math.fround()

Math.fround方法返回一個數的單精度浮點數形式。

```javascript
Math.fround(0)     // 0
Math.fround(1)     // 1
Math.fround(1.337) // 1.3370000123977661
Math.fround(1.5)   // 1.5
Math.fround(NaN)   // NaN
```

對於整數來說，`Math.fround`方法返回結果不會有任何不同，區別主要是那些無法用64個二進位制位精確表示的小數。這時，`Math.fround`方法會返回最接近這個小數的單精度浮點數。

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.fround = Math.fround || function(x) {
  return new Float32Array([x])[0];
};
```

### Math.hypot()

`Math.hypot`方法返回所有引數的平方和的平方根。

```javascript
Math.hypot(3, 4);        // 5
Math.hypot(3, 4, 5);     // 7.0710678118654755
Math.hypot();            // 0
Math.hypot(NaN);         // NaN
Math.hypot(3, 4, 'foo'); // NaN
Math.hypot(3, 4, '5');   // 7.0710678118654755
Math.hypot(-3);          // 3
```

上面程式碼中，3的平方加上4的平方，等於5的平方。

如果引數不是數值，`Math.hypot`方法會將其轉為數值。只要有一個引數無法轉為數值，就會返回NaN。

### 對數方法

ES6新增了4個對數相關方法。

**（1） Math.expm1()**

`Math.expm1(x)`返回e<sup>x</sup> - 1，即`Math.exp(x) - 1`。

```javascript
Math.expm1(-1) // -0.6321205588285577
Math.expm1(0)  // 0
Math.expm1(1)  // 1.718281828459045
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.expm1 = Math.expm1 || function(x) {
  return Math.exp(x) - 1;
};
```

**（2）Math.log1p()**

`Math.log1p(x)`方法返回`1 + x`的自然對數，即`Math.log(1 + x)`。如果`x`小於-1，返回`NaN`。

```javascript
Math.log1p(1)  // 0.6931471805599453
Math.log1p(0)  // 0
Math.log1p(-1) // -Infinity
Math.log1p(-2) // NaN
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.log1p = Math.log1p || function(x) {
  return Math.log(1 + x);
};
```

**（3）Math.log10()**

`Math.log10(x)`返回以10為底的`x`的對數。如果`x`小於0，則返回NaN。

```javascript
Math.log10(2)      // 0.3010299956639812
Math.log10(1)      // 0
Math.log10(0)      // -Infinity
Math.log10(-2)     // NaN
Math.log10(100000) // 5
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.log10 = Math.log10 || function(x) {
  return Math.log(x) / Math.LN10;
};
```

**（4）Math.log2()**

`Math.log2(x)`返回以2為底的`x`的對數。如果`x`小於0，則返回NaN。

```javascript
Math.log2(3)       // 1.584962500721156
Math.log2(2)       // 1
Math.log2(1)       // 0
Math.log2(0)       // -Infinity
Math.log2(-2)      // NaN
Math.log2(1024)    // 10
Math.log2(1 << 29) // 29
```

對於沒有部署這個方法的環境，可以用下面的程式碼模擬。

```javascript
Math.log2 = Math.log2 || function(x) {
  return Math.log(x) / Math.LN2;
};
```

### 三角函式方法

ES6新增了6個三角函式方法。

- `Math.sinh(x)` 返回`x`的雙曲正弦（hyperbolic sine）
- `Math.cosh(x)` 返回`x`的雙曲餘弦（hyperbolic cosine）
- `Math.tanh(x)` 返回`x`的雙曲正切（hyperbolic tangent）
- `Math.asinh(x)` 返回`x`的反雙曲正弦（inverse hyperbolic sine）
- `Math.acosh(x)` 返回`x`的反雙曲餘弦（inverse hyperbolic cosine）
- `Math.atanh(x)` 返回`x`的反雙曲正切（inverse hyperbolic tangent）

## Math.signbit()

`Math.sign()`用來判斷一個值的正負，但是如果引數是`-0`，它會返回`-0`。

```javascript
Math.sign(-0) // -0
```

這導致對於判斷符號位的正負，`Math.sign()`不是很有用。JavaScript 內部使用64位浮點數（國際標準IEEE 754）表示數值，IEEE 754規定第一位是符號位，`0`表示正數，`1`表示負數。所以會有兩種零，`+0`是符號位為`0`時的零值，`-0`是符號位為`1`時的零值。實際程式設計中，判斷一個值是`+0`還是`-0`非常麻煩，因為它們是相等的。

```javascript
+0 === -0 // true
```

目前，有一個[提案](http://jfbastien.github.io/papers/Math.signbit.html)，引入了`Math.signbit()`方法判斷一個數的符號位是否設定了。

```javascript
Math.signbit(2) //false
Math.signbit(-2) //true
Math.signbit(0) //false
Math.signbit(-0) //true
```

可以看到，該方法正確返回了`-0`的符號位是設定了的。

該方法的演算法如下。

- 如果引數是`NaN`，返回`false`
- 如果引數是`-0`，返回`true`
- 如果引數是負值，返回`true`
- 其他情況返回`false`

## 指數運算子

ES2016 新增了一個指數運算子（`**`）。

```javascript
2 ** 2 // 4
2 ** 3 // 8
```

指數運算子可以與等號結合，形成一個新的賦值運算子（`**=`）。

```javascript
let a = 1.5;
a **= 2;
// 等同於 a = a * a;

let b = 4;
b **= 3;
// 等同於 b = b * b * b;
```

注意，在 V8 引擎中，指數運算子與`Math.pow`的實現不相同，對於特別大的運算結果，兩者會有細微的差異。

```javascript
Math.pow(99, 99)
// 3.697296376497263e+197

99 ** 99
// 3.697296376497268e+197
```

上面程式碼中，兩個運算結果的最後一位有效數字是有差異的。

