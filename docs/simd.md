# SIMD

## 概述

SIMD（發音`/sim-dee/`）是“Single Instruction/Multiple Data”的縮寫，意為“單指令，多資料”。它是 JavaScript 操作 CPU 對應指令的介面，你可以看做這是一種不同的運算執行模式。與它相對的是 SISD（“Single Instruction/Single Data”），即“單指令，單資料”。

SIMD 的含義是使用一個指令，完成多個數據的運算；SISD 的含義是使用一個指令，完成單個數據的運算，這是 JavaScript 的預設運算模式。顯而易見，SIMD 的執行效率要高於 SISD，所以被廣泛用於3D圖形運算、物理模擬等運算量超大的專案之中。

為了理解SIMD，請看下面的例子。

```javascript
var a = [1, 2, 3, 4];
var b = [5, 6, 7, 8];
var c = [];

c[0] = a[0] + b[0];
c[1] = a[1] + b[1];
c[2] = a[2] + b[2];
c[3] = a[3] + b[3];
c // Array[6, 8, 10, 12]
```

上面程式碼中，陣列`a`和`b`的對應成員相加，結果放入陣列`c`。它的運算模式是依次處理每個陣列成員，一共有四個陣列成員，所以需要運算4次。

如果採用 SIMD 模式，只要運算一次就夠了。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 4);
var b = SIMD.Float32x4(5, 6, 7, 8);
var c = SIMD.Float32x4.add(a, b); // Float32x4[6, 8, 10, 12]
```

上面程式碼之中，陣列`a`和`b`的四個成員的各自相加，只用一條指令就完成了。因此，速度比上一種寫法提高了4倍。

一次 SIMD 運算，可以處理多個數據，這些資料被稱為“通道”（lane）。上面程式碼中，一次運算了四個資料，因此就是四個通道。

SIMD 通常用於向量運算。

```javascript
v + w	= 〈v1, …, vn〉+ 〈w1, …, wn〉
      = 〈v1+w1, …, vn+wn〉
```

上面程式碼中，`v`和`w`是兩個多元向量。它們的加運算，在 SIMD 下是一個指令、而不是 n 個指令完成的，這就大大提高了效率。這對於3D動畫、影象處理、訊號處理、數值處理、加密等運算是非常重要的。比如，Canvas的`getImageData()`會將影象檔案讀成一個二進位制陣列，SIMD 就很適合對於這種陣列的處理。

總的來說，SIMD 是資料並行處理（parallelism）的一種手段，可以加速一些運算密集型操作的速度。將來與 WebAssembly 結合以後，可以讓 JavaScript 達到二進位制程式碼的執行速度。

## 資料型別

SIMD 提供12種資料型別，總長度都是128個二進位制位。

- Float32x4：四個32位浮點數
- Float64x2：兩個64位浮點數
- Int32x4：四個32位整數
- Int16x8：八個16位整數
- Int8x16：十六個8位整數
- Uint32x4：四個無符號的32位整數
- Uint16x8：八個無符號的16位整數
- Uint8x16：十六個無符號的8位整數
- Bool32x4：四個32位布林值
- Bool16x8：八個16位布林值
- Bool8x16：十六個8位布林值
- Bool64x2：兩個64位布林值

每種資料型別被`x`符號分隔成兩部分，後面的部分表示通道數，前面的部分表示每個通道的寬度和型別。比如，`Float32x4`就表示這個值有4個通道，每個通道是一個32位浮點數。

每個通道之中，可以放置四種資料。

- 浮點數（float，比如1.0）
- 帶符號的整數（Int，比如-1）
- 無符號的整數（Uint，比如1）
- 布林值（Bool，包含`true`和`false`兩種值）

每種 SIMD 的資料型別都是一個函式方法，可以傳入引數，生成對應的值。

```javascript
var a = SIMD.Float32x4(1.0, 2.0, 3.0, 4.0);
```

上面程式碼中，變數`a`就是一個128位、包含四個32位浮點數（即四個通道）的值。

注意，這些資料型別方法都不是建構函式，前面不能加`new`，否則會報錯。

```javascript
var v = new SIMD.Float32x4(0, 1, 2, 3);
// TypeError: SIMD.Float32x4 is not a constructor
```

## 靜態方法：數學運算

每種資料型別都有一系列運算子，支援基本的數學運算。

### SIMD.%type%.abs()，SIMD.%type%.neg()

`abs`方法接受一個SIMD值作為引數，將它的每個通道都轉成絕對值，作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(-1, -2, 0, NaN);
SIMD.Float32x4.abs(a)
// Float32x4[1, 2, 0, NaN]
```

`neg`方法接受一個SIMD值作為引數，將它的每個通道都轉成負值，作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(-1, -2, 3, 0);
SIMD.Float32x4.neg(a)
// Float32x4[1, 2, -3, -0]

var b = SIMD.Float64x2(NaN, Infinity);
SIMD.Float64x2.neg(b)
// Float64x2[NaN, -Infinity]
```

### SIMD.%type%.add()，SIMD.%type%.addSaturate()

`add`方法接受兩個SIMD值作為引數，將它們的每個通道相加，作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(1.0, 2.0, 3.0, 4.0);
var b = SIMD.Float32x4(5.0, 10.0, 15.0, 20.0);
var c = SIMD.Float32x4.add(a, b);
```

上面程式碼中，經過加法運算，新的SIMD值為`(6.0, 12.0, 18.0. 24.0)`。

`addSaturate`方法跟`add`方法的作用相同，都是兩個通道相加，但是溢位的處理不一致。對於`add`方法，如果兩個值相加發生溢位，溢位的二進位制位會被丟棄; `addSaturate`方法則是返回該資料型別的最大值。

```javascript
var a = SIMD.Uint16x8(65533, 65534, 65535, 65535, 1, 1, 1, 1);
var b = SIMD.Uint16x8(1, 1, 1, 5000, 1, 1, 1, 1);
SIMD.Uint16x8.addSaturate(a, b);
// Uint16x8[65534, 65535, 65535, 65535, 2, 2, 2, 2]

var c = SIMD.Int16x8(32765, 32766, 32767, 32767, 1, 1, 1, 1);
var d = SIMD.Int16x8(1, 1, 1, 5000, 1, 1, 1, 1);
SIMD.Int16x8.addSaturate(c, d);
// Int16x8[32766, 32767, 32767, 32767, 2, 2, 2, 2]
```

上面程式碼中，`Uint16`的最大值是65535，`Int16`的最大值是32767。一旦發生溢位，就返回這兩個值。

注意，`Uint32x4`和`Int32x4`這兩種資料型別沒有`addSaturate`方法。

### SIMD.%type%.sub()，SIMD.%type%.subSaturate()

`sub`方法接受兩個SIMD值作為引數，將它們的每個通道相減，作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(-1, -2, 3, 4);
var b = SIMD.Float32x4(3, 3, 3, 3);
SIMD.Float32x4.sub(a, b)
// Float32x4[-4, -5, 0, 1]
```

`subSaturate`方法跟`sub`方法的作用相同，都是兩個通道相減，但是溢位的處理不一致。對於`sub`方法，如果兩個值相減發生溢位，溢位的二進位制位會被丟棄; `subSaturate`方法則是返回該資料型別的最小值。

```javascript
var a = SIMD.Uint16x8(5, 1, 1, 1, 1, 1, 1, 1);
var b = SIMD.Uint16x8(10, 1, 1, 1, 1, 1, 1, 1);
SIMD.Uint16x8.subSaturate(a, b)
// Uint16x8[0, 0, 0, 0, 0, 0, 0, 0]

var c = SIMD.Int16x8(-100, 0, 0, 0, 0, 0, 0, 0);
var d = SIMD.Int16x8(32767, 0, 0, 0, 0, 0, 0, 0);
SIMD.Int16x8.subSaturate(c, d)
// Int16x8[-32768, 0, 0, 0, 0, 0, 0, 0, 0]
```

上面程式碼中，`Uint16`的最小值是`0`，`subSaturate`的最小值是`-32678`。一旦運算髮生溢位，就返回最小值。

### SIMD.%type%.mul()，SIMD.%type%.div()，SIMD.%type%.sqrt()

`mul`方法接受兩個SIMD值作為引數，將它們的每個通道相乘，作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(-1, -2, 3, 4);
var b = SIMD.Float32x4(3, 3, 3, 3);
SIMD.Float32x4.mul(a, b)
// Float32x4[-3, -6, 9, 12]
```

`div`方法接受兩個SIMD值作為引數，將它們的每個通道相除，作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(2, 2, 2, 2);
var b = SIMD.Float32x4(4, 4, 4, 4);
SIMD.Float32x4.div(a, b)
// Float32x4[0.5, 0.5, 0.5, 0.5]
```

`sqrt`方法接受一個SIMD值作為引數，求出每個通道的平方根，作為一個新的SIMD值返回。

```javascript
var b = SIMD.Float64x2(4, 8);
SIMD.Float64x2.sqrt(b)
// Float64x2[2, 2.8284271247461903]
```

### SIMD.%FloatType%.reciprocalApproximation()，SIMD.%type%.reciprocalSqrtApproximation()

`reciprocalApproximation`方法接受一個SIMD值作為引數，求出每個通道的倒數（`1 / x`），作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 4);
SIMD.Float32x4.reciprocalApproximation(a);
// Float32x4[1, 0.5, 0.3333333432674408, 0.25]
```

`reciprocalSqrtApproximation`方法接受一個SIMD值作為引數，求出每個通道的平方根的倒數（`1 / (x^0.5)`），作為一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 4);
SIMD.Float32x4.reciprocalSqrtApproximation(a)
// Float32x4[1, 0.7071067690849304, 0.5773502588272095, 0.5]
```

注意，只有浮點數的資料型別才有這兩個方法。

### SIMD.%IntegerType%.shiftLeftByScalar()

`shiftLeftByScalar`方法接受一個SIMD值作為引數，然後將每個通道的值左移指定的位數，作為一個新的SIMD值返回。

```javascript
var a = SIMD.Int32x4(1, 2, 4, 8);
SIMD.Int32x4.shiftLeftByScalar(a, 1);
// Int32x4[2, 4, 8, 16]
```

如果左移後，新的值超出了當前資料型別的位數，溢位的部分會被丟棄。

```javascript
var ix4 = SIMD.Int32x4(1, 2, 3, 4);
var jx4 = SIMD.Int32x4.shiftLeftByScalar(ix4, 32);
// Int32x4[0, 0, 0, 0]
```

注意，只有整數的資料型別才有這個方法。

### SIMD.%IntegerType%.shiftRightByScalar()

`shiftRightByScalar`方法接受一個SIMD值作為引數，然後將每個通道的值右移指定的位數，返回一個新的SIMD值。

```javascript
var a = SIMD.Int32x4(1, 2, 4, -8);
SIMD.Int32x4.shiftRightByScalar(a, 1);
// Int32x4[0, 1, 2, -4]
```

如果原來通道的值是帶符號的值，則符號位保持不變，不受右移影響。如果是不帶符號位的值，則右移後頭部會補`0`。

```javascript
var a = SIMD.Uint32x4(1, 2, 4, -8);
SIMD.Uint32x4.shiftRightByScalar(a, 1);
// Uint32x4[0, 1, 2, 2147483644]
```

上面程式碼中，`-8`右移一位變成了`2147483644`，是因為對於32位無符號整數來說，`-8`的二進位制形式是`11111111111111111111111111111000`，右移一位就變成了`01111111111111111111111111111100`，相當於`2147483644`。

注意，只有整數的資料型別才有這個方法。

## 靜態方法：通道處理

### SIMD.%type%.check()

`check`方法用於檢查一個值是否為當前型別的SIMD值。如果是的，就返回這個值，否則就報錯。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 9);

SIMD.Float32x4.check(a);
// Float32x4[1, 2, 3, 9]

SIMD.Float32x4.check([1,2,3,4]) // 報錯
SIMD.Int32x4.check(a) // 報錯
SIMD.Int32x4.check('hello world') // 報錯
```

### SIMD.%type%.extractLane()，SIMD.%type%.replaceLane()

`extractLane`方法用於返回給定通道的值。它接受兩個引數，分別是SIMD值和通道編號。

```javascript
var t = SIMD.Float32x4(1, 2, 3, 4);
SIMD.Float32x4.extractLane(t, 2) // 3
```

`replaceLane`方法用於替換指定通道的值，並返回一個新的SIMD值。它接受三個引數，分別是原來的SIMD值、通道編號和新的通道值。

```javascript
var t = SIMD.Float32x4(1, 2, 3, 4);
SIMD.Float32x4.replaceLane(t, 2, 42)
// Float32x4[1, 2, 42, 4]
```

### SIMD.%type%.load()

`load`方法用於從二進位制陣列讀入資料，生成一個新的SIMD值。

```javascript
var a = new Int32Array([1,2,3,4,5,6,7,8]);
SIMD.Int32x4.load(a, 0);
// Int32x4[1, 2, 3, 4]

var b = new Int32Array([1,2,3,4,5,6,7,8]);
SIMD.Int32x4.load(a, 2);
// Int32x4[3, 4, 5, 6]
```

`load`方法接受兩個引數：一個二進位制陣列和開始讀取的位置（從0開始）。如果位置不合法（比如`-1`或者超出二進位制陣列的大小），就會丟擲一個錯誤。

這個方法還有三個變種`load1()`、`load2()`、`load3()`，表示從指定位置開始，只加載一個通道、二個通道、三個通道的值。

```javascript
// 格式
SIMD.Int32x4.load(tarray, index)
SIMD.Int32x4.load1(tarray, index)
SIMD.Int32x4.load2(tarray, index)
SIMD.Int32x4.load3(tarray, index)

// 例項
var a = new Int32Array([1,2,3,4,5,6,7,8]);
SIMD.Int32x4.load1(a, 0);
// Int32x4[1, 0, 0, 0]
SIMD.Int32x4.load2(a, 0);
// Int32x4[1, 2, 0, 0]
SIMD.Int32x4.load3(a, 0);
// Int32x4[1, 2, 3,0]
```

### SIMD.%type%.store()

`store`方法用於將一個SIMD值，寫入一個二進位制陣列。它接受三個引數，分別是二進位制陣列、開始寫入的陣列位置、SIMD值。它返回寫入值以後的二進位制陣列。

```javascript
var t1 = new Int32Array(8);
var v1 = SIMD.Int32x4(1, 2, 3, 4);
SIMD.Int32x4.store(t1, 0, v1)
// Int32Array[1, 2, 3, 4, 0, 0, 0, 0]

var t2 = new Int32Array(8);
var v2 = SIMD.Int32x4(1, 2, 3, 4);
SIMD.Int32x4.store(t2, 2, v2)
// Int32Array[0, 0, 1, 2, 3, 4, 0, 0]
```

上面程式碼中，`t1`是一個二進位制陣列，`v1`是一個SIMD值，只有四個通道。所以寫入`t1`以後，只有前四個位置有值，後四個位置都是0。而`t2`是從2號位置開始寫入，所以前兩個位置和後兩個位置都是0。

這個方法還有三個變種`store1()`、`store2()`和`store3()`，表示只寫入一個通道、二個通道和三個通道的值。

```javascript
var tarray = new Int32Array(8);
var value = SIMD.Int32x4(1, 2, 3, 4);
SIMD.Int32x4.store1(tarray, 0, value);
// Int32Array[1, 0, 0, 0, 0, 0, 0, 0]
```

### SIMD.%type%.splat()

`splat`方法返回一個新的SIMD值，該值的所有通道都會設成同一個預先給定的值。

```javascript
SIMD.Float32x4.splat(3);
// Float32x4[3, 3, 3, 3]
SIMD.Float64x2.splat(3);
// Float64x2[3, 3]
```

如果省略引數，所有整數型的SIMD值都會設定`0`，浮點型的SIMD值都會設成`NaN`。

### SIMD.%type%.swizzle()

`swizzle`方法返回一個新的SIMD值，重新排列原有的SIMD值的通道順序。

```javascript
var t = SIMD.Float32x4(1, 2, 3, 4);
SIMD.Float32x4.swizzle(t, 1, 2, 0, 3);
// Float32x4[2,3,1,4]
```

上面程式碼中，`swizzle`方法的第一個引數是原有的SIMD值，後面的引數對應將要返回的SIMD值的四個通道。它的意思是新的SIMD的四個通道，依次是原來SIMD值的1號通道、2號通道、0號通道、3號通道。由於SIMD值最多可以有16個通道，所以`swizzle`方法除了第一個引數以外，最多還可以接受16個引數。

下面是另一個例子。

```javascript
var a = SIMD.Float32x4(1.0, 2.0, 3.0, 4.0);
// Float32x4[1.0, 2.0, 3.0, 4.0]

var b = SIMD.Float32x4.swizzle(a, 0, 0, 1, 1);
// Float32x4[1.0, 1.0, 2.0, 2.0]

var c = SIMD.Float32x4.swizzle(a, 3, 3, 3, 3);
// Float32x4[4.0, 4.0, 4.0, 4.0]

var d = SIMD.Float32x4.swizzle(a, 3, 2, 1, 0);
// Float32x4[4.0, 3.0, 2.0, 1.0]
```

### SIMD.%type%.shuffle()

`shuffle`方法從兩個SIMD值之中取出指定通道，返回一個新的SIMD值。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 4);
var b = SIMD.Float32x4(5, 6, 7, 8);

SIMD.Float32x4.shuffle(a, b, 1, 5, 7, 2);
// Float32x4[2, 6, 8, 3]
```

上面程式碼中，`a`和`b`一共有8個通道，依次編號為0到7。`shuffle`根據編號，取出相應的通道，返回一個新的SIMD值。

## 靜態方法：比較運算

### SIMD.%type%.equal()，SIMD.%type%.notEqual()

`equal`方法用來比較兩個SIMD值`a`和`b`的每一個通道，根據兩者是否精確相等（`a === b`），得到一個布林值。最後，所有通道的比較結果，組成一個新的SIMD值，作為掩碼返回。`notEqual`方法則是比較兩個通道是否不相等（`a !== b`）。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 9);
var b = SIMD.Float32x4(1, 4, 7, 9);

SIMD.Float32x4.equal(a,b)
// Bool32x4[true, false, false, true]

SIMD.Float32x4.notEqual(a,b);
// Bool32x4[false, true, true, false]
```

### SIMD.%type%.greaterThan()，SIMD.%type%.greaterThanOrEqual()

`greatThan`方法用來比較兩個SIMD值`a`和`b`的每一個通道，如果在該通道中，`a`較大就得到`true`，否則得到`false`。最後，所有通道的比較結果，組成一個新的SIMD值，作為掩碼返回。`greaterThanOrEqual`則是比較`a`是否大於等於`b`。

```javascript
var a = SIMD.Float32x4(1, 6, 3, 11);
var b = SIMD.Float32x4(1, 4, 7, 9);

SIMD.Float32x4.greaterThan(a, b)
// Bool32x4[false, true, false, true]

SIMD.Float32x4.greaterThanOrEqual(a, b)
// Bool32x4[true, true, false, true]
```

### SIMD.%type%.lessThan()，SIMD.%type%.lessThanOrEqual()

`lessThan`方法用來比較兩個SIMD值`a`和`b`的每一個通道，如果在該通道中，`a`較小就得到`true`，否則得到`false`。最後，所有通道的比較結果，會組成一個新的SIMD值，作為掩碼返回。`lessThanOrEqual`方法則是比較`a`是否等於`b`。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 11);
var b = SIMD.Float32x4(1, 4, 7, 9);

SIMD.Float32x4.lessThan(a, b)
// Bool32x4[false, true, true, false]

SIMD.Float32x4.lessThanOrEqual(a, b)
// Bool32x4[true, true, true, false]
```

### SIMD.%type%.select()

`select`方法通過掩碼生成一個新的SIMD值。它接受三個引數，分別是掩碼和兩個SIMD值。

```javascript
var a = SIMD.Float32x4(1, 2, 3, 4);
var b = SIMD.Float32x4(5, 6, 7, 8);

var mask = SIMD.Bool32x4(true, false, false, true);

SIMD.Float32x4.select(mask, a, b);
// Float32x4[1, 6, 7, 4]
```

上面程式碼中，`select`方法接受掩碼和兩個SIMD值作為引數。當某個通道對應的掩碼為`true`時，會選擇第一個SIMD值的對應通道，否則選擇第二個SIMD值的對應通道。

這個方法通常與比較運算子結合使用。

```javascript
var a = SIMD.Float32x4(0, 12, 3, 4);
var b = SIMD.Float32x4(0, 6, 7, 50);

var mask = SIMD.Float32x4.lessThan(a,b);
// Bool32x4[false, false, true, true]

var result = SIMD.Float32x4.select(mask, a, b);
// Float32x4[0, 6, 3, 4]
```

上面程式碼中，先通過`lessThan`方法生成一個掩碼，然後通過`select`方法生成一個由每個通道的較小值組成的新的SIMD值。

### SIMD.%BooleanType%.allTrue()，SIMD.%BooleanType%.anyTrue()

`allTrue`方法接受一個SIMD值作為引數，然後返回一個布林值，表示該SIMD值的所有通道是否都為`true`。

```javascript
var a = SIMD.Bool32x4(true, true, true, true);
var b = SIMD.Bool32x4(true, false, true, true);

SIMD.Bool32x4.allTrue(a); // true
SIMD.Bool32x4.allTrue(b); // false
```

`anyTrue`方法則是隻要有一個通道為`true`，就返回`true`，否則返回`false`。

```javascript
var a = SIMD.Bool32x4(false, false, false, false);
var b = SIMD.Bool32x4(false, false, true, false);

SIMD.Bool32x4.anyTrue(a); // false
SIMD.Bool32x4.anyTrue(b); // true
```

注意，只有四種布林值資料型別（`Bool32x4`、`Bool16x8`、`Bool8x16`、`Bool64x2`）才有這兩個方法。

這兩個方法通常與比較運算子結合使用。

```javascript
var ax4    = SIMD.Float32x4(1.0, 2.0, 3.0, 4.0);
var bx4    = SIMD.Float32x4(0.0, 6.0, 7.0, 8.0);
var ix4    = SIMD.Float32x4.lessThan(ax4, bx4);
var b1     = SIMD.Int32x4.allTrue(ix4); // false
var b2     = SIMD.Int32x4.anyTrue(ix4); // true
```

### SIMD.%type%.min()，SIMD.%type%.minNum()

`min`方法接受兩個SIMD值作為引數，將兩者的對應通道的較小值，組成一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(-1, -2, 3, 5.2);
var b = SIMD.Float32x4(0, -4, 6, 5.5);
SIMD.Float32x4.min(a, b);
// Float32x4[-1, -4, 3, 5.2]
```

如果有一個通道的值是`NaN`，則會優先返回`NaN`。

```javascript
var c = SIMD.Float64x2(NaN, Infinity)
var d = SIMD.Float64x2(1337, 42);
SIMD.Float64x2.min(c, d);
// Float64x2[NaN, 42]
```

`minNum`方法與`min`的作用一模一樣，唯一的區別是如果有一個通道的值是`NaN`，則會優先返回另一個通道的值。

```javascript
var ax4 = SIMD.Float32x4(1.0, 2.0, NaN, NaN);
var bx4 = SIMD.Float32x4(2.0, 1.0, 3.0, NaN);
var cx4 = SIMD.Float32x4.min(ax4, bx4);
// Float32x4[1.0, 1.0, NaN, NaN]
var dx4 = SIMD.Float32x4.minNum(ax4, bx4);
// Float32x4[1.0, 1.0, 3.0, NaN]
```

### SIMD.%type%.max()，SIMD.%type%.maxNum()

`max`方法接受兩個SIMD值作為引數，將兩者的對應通道的較大值，組成一個新的SIMD值返回。

```javascript
var a = SIMD.Float32x4(-1, -2, 3, 5.2);
var b = SIMD.Float32x4(0, -4, 6, 5.5);
SIMD.Float32x4.max(a, b);
// Float32x4[0, -2, 6, 5.5]
```

如果有一個通道的值是`NaN`，則會優先返回`NaN`。

```javascript
var c = SIMD.Float64x2(NaN, Infinity)
var d = SIMD.Float64x2(1337, 42);
SIMD.Float64x2.max(c, d)
// Float64x2[NaN, Infinity]
```

`maxNum`方法與`max`的作用一模一樣，唯一的區別是如果有一個通道的值是`NaN`，則會優先返回另一個通道的值。

```javascript
var c = SIMD.Float64x2(NaN, Infinity)
var d = SIMD.Float64x2(1337, 42);
SIMD.Float64x2.maxNum(c, d)
// Float64x2[1337, Infinity]
```

## 靜態方法：位運算

### SIMD.%type%.and()，SIMD.%type%.or()，SIMD.%type%.xor()，SIMD.%type%.not()

`and`方法接受兩個SIMD值作為引數，返回兩者對應的通道進行二進位制`AND`運算（`&`）後得到的新的SIMD值。

```javascript
var a = SIMD.Int32x4(1, 2, 4, 8);
var b = SIMD.Int32x4(5, 5, 5, 5);
SIMD.Int32x4.and(a, b)
// Int32x4[1, 0, 4, 0]
```

上面程式碼中，以通道`0`為例，`1`的二進位制形式是`0001`，`5`的二進位制形式是`01001`，所以進行`AND`運算以後，得到`0001`。

`or`方法接受兩個SIMD值作為引數，返回兩者對應的通道進行二進位制`OR`運算（`|`）後得到的新的SIMD值。

```javascript
var a = SIMD.Int32x4(1, 2, 4, 8);
var b = SIMD.Int32x4(5, 5, 5, 5);
SIMD.Int32x4.or(a, b)
// Int32x4[5, 7, 5, 13]
```

`xor`方法接受兩個SIMD值作為引數，返回兩者對應的通道進行二進位制”異或“運算（`^`）後得到的新的SIMD值。

```javascript
var a = SIMD.Int32x4(1, 2, 4, 8);
var b = SIMD.Int32x4(5, 5, 5, 5);
SIMD.Int32x4.xor(a, b)
// Int32x4[4, 7, 1, 13]
```

`not`方法接受一個SIMD值作為引數，返回每個通道進行二進位制”否“運算（`~`）後得到的新的SIMD值。

```javascript
var a = SIMD.Int32x4(1, 2, 4, 8);
SIMD.Int32x4.not(a)
// Int32x4[-2, -3, -5, -9]
```

上面程式碼中，`1`的否運算之所以得到`-2`，是因為在計算機內部，負數採用”2的補碼“這種形式進行表示。也就是說，整數`n`的負數形式`-n`，是對每一個二進位制位取反以後，再加上1。因此，直接取反就相當於負數形式再減去1，比如`1`的負數形式是`-1`，再減去1，就得到了`-2`。

## 靜態方法：資料型別轉換

SIMD提供以下方法，用來將一種資料型別轉為另一種資料型別。

- `SIMD.%type%.fromFloat32x4()`
- `SIMD.%type%.fromFloat32x4Bits()`
- `SIMD.%type%.fromFloat64x2Bits()`
- `SIMD.%type%.fromInt32x4()`
- `SIMD.%type%.fromInt32x4Bits()`
- `SIMD.%type%.fromInt16x8Bits()`
- `SIMD.%type%.fromInt8x16Bits()`
- `SIMD.%type%.fromUint32x4()`
- `SIMD.%type%.fromUint32x4Bits()`
- `SIMD.%type%.fromUint16x8Bits()`
- `SIMD.%type%.fromUint8x16Bits()`

帶有`Bits`字尾的方法，會原封不動地將二進位制位拷貝到新的資料型別；不帶字尾的方法，則會進行資料型別轉換。

```javascript
var t = SIMD.Float32x4(1.0, 2.0, 3.0, 4.0);
SIMD.Int32x4.fromFloat32x4(t);
// Int32x4[1, 2, 3, 4]

SIMD.Int32x4.fromFloat32x4Bits(t);
// Int32x4[1065353216, 1073741824, 1077936128, 1082130432]
```

上面程式碼中，`fromFloat32x4`是將浮點數轉為整數，然後存入新的資料型別；`fromFloat32x4Bits`則是將二進位制位原封不動地拷貝進入新的資料型別，然後進行解讀。

`Bits`字尾的方法，還可以用於通道數目不對等的拷貝。

```javascript
var t = SIMD.Float32x4(1.0, 2.0, 3.0, 4.0);
SIMD.Int16x8.fromFloat32x4Bits(t);
// Int16x8[0, 16256, 0, 16384, 0, 16448, 0, 16512]
```

上面程式碼中，原始SIMD值`t`是4通道的，而目標值是8通道的。

如果資料轉換時，原通道的資料大小，超過了目標通道的最大寬度，就會報錯。

## 例項方法

### SIMD.%type%.prototype.toString()

`toString`方法返回一個SIMD值的字串形式。

```javascript
var a = SIMD.Float32x4(11, 22, 33, 44);
a.toString() // "SIMD.Float32x4(11, 22, 33, 44)"
```

## 例項：求平均值

正常模式下，計算`n`個值的平均值，需要運算`n`次。

```javascript
function average(list) {
  var n = list.length;
  var sum = 0.0;
  for (var i = 0; i < n; i++) {
    sum += list[i];
  }
  return sum / n;
}
```

使用SIMD，可以將計算次數減少到`n`次的四分之一。

```javascript
function average(list) {
  var n = list.length;
  var sum = SIMD.Float32x4.splat(0.0);
  for (var i = 0; i < n; i += 4) {
    sum = SIMD.Float32x4.add(
      sum,
      SIMD.Float32x4.load(list, i)
    );
  }
  var total = SIMD.Float32x4.extractLane(sum, 0) +
              SIMD.Float32x4.extractLane(sum, 1) +
              SIMD.Float32x4.extractLane(sum, 2) +
              SIMD.Float32x4.extractLane(sum, 3);
  return total / n;
}
```

上面程式碼先是每隔四位，將所有的值讀入一個 SIMD，然後立刻累加。然後，得到累加值四個通道的總和，再除以`n`就可以了。

