# 二進位制陣列

二進位制陣列（`ArrayBuffer`物件、`TypedArray`檢視和`DataView`檢視）是 JavaScript 操作二進位制資料的一個介面。這些物件早就存在，屬於獨立的規格（2011年2月釋出），ES6 將它們納入了 ECMAScript 規格，並且增加了新的方法。

這個介面的原始設計目的，與 WebGL 專案有關。所謂WebGL，就是指瀏覽器與顯示卡之間的通訊介面，為了滿足 JavaScript 與顯示卡之間大量的、實時的資料交換，它們之間的資料通訊必須是二進位制的，而不能是傳統的文字格式。文字格式傳遞一個32位整數，兩端的 JavaScript 指令碼與顯示卡都要進行格式轉化，將非常耗時。這時要是存在一種機制，可以像 C 語言那樣，直接操作位元組，將4個位元組的32位整數，以二進位制形式原封不動地送入顯示卡，指令碼的效能就會大幅提升。

二進位制陣列就是在這種背景下誕生的。它很像C語言的陣列，允許開發者以陣列下標的形式，直接操作記憶體，大大增強了JavaScript處理二進位制資料的能力，使得開發者有可能通過JavaScript與作業系統的原生介面進行二進位制通訊。

二進位制陣列由三類物件組成。

**（1）`ArrayBuffer`物件**：代表記憶體之中的一段二進位制資料，可以通過“檢視”進行操作。“檢視”部署了陣列介面，這意味著，可以用陣列的方法操作記憶體。

**（2）TypedArray檢視**：共包括9種類型的檢視，比如`Uint8Array`（無符號8位整數）陣列檢視, `Int16Array`（16位整數）陣列檢視, `Float32Array`（32位浮點數）陣列檢視等等。

**（3）`DataView`檢視**：可以自定義複合格式的檢視，比如第一個位元組是Uint8（無符號8位整數）、第二、三個位元組是Int16（16位整數）、第四個位元組開始是Float32（32位浮點數）等等，此外還可以自定義位元組序。

簡單說，`ArrayBuffer`物件代表原始的二進位制資料，TypedArray檢視用來讀寫簡單型別的二進位制資料，`DataView`檢視用來讀寫複雜型別的二進位制資料。

TypedArray檢視支援的資料型別一共有9種（`DataView`檢視支援除`Uint8C`以外的其他8種）。

資料型別 | 位元組長度 | 含義 | 對應的C語言型別
--------|--------|----|---------------
Int8|1|8位帶符號整數|signed char
Uint8|1|8位不帶符號整數|unsigned char
Uint8C|1|8位不帶符號整數（自動過濾溢位）|unsigned char
Int16|2|16位帶符號整數|short
Uint16|2|16位不帶符號整數|unsigned short
Int32|4|32位帶符號整數|int
Uint32|4|32位不帶符號的整數|unsigned int
Float32|4|32位浮點數|float
Float64|8|64位浮點數|double

注意，二進位制陣列並不是真正的陣列，而是類似陣列的物件。

很多瀏覽器操作的API，用到了二進位制陣列操作二進位制資料，下面是其中的幾個。

- File API
- XMLHttpRequest
- Fetch API
- Canvas
- WebSockets

## ArrayBuffer物件

### 概述

`ArrayBuffer`物件代表儲存二進位制資料的一段記憶體，它不能直接讀寫，只能通過檢視（TypedArray檢視和`DataView`檢視)來讀寫，檢視的作用是以指定格式解讀二進位制資料。

`ArrayBuffer`也是一個建構函式，可以分配一段可以存放資料的連續記憶體區域。

```javascript
var buf = new ArrayBuffer(32);
```

上面程式碼生成了一段32位元組的記憶體區域，每個位元組的值預設都是0。可以看到，`ArrayBuffer`建構函式的引數是所需要的記憶體大小（單位位元組）。

為了讀寫這段內容，需要為它指定檢視。`DataView`檢視的建立，需要提供`ArrayBuffer`物件實例作為引數。

```javascript
var buf = new ArrayBuffer(32);
var dataView = new DataView(buf);
dataView.getUint8(0) // 0
```

上面程式碼對一段32位元組的記憶體，建立`DataView`檢視，然後以不帶符號的8位整數格式，讀取第一個元素，結果得到0，因為原始記憶體的`ArrayBuffer`物件，預設所有位都是0。

另一種TypedArray檢視，與`DataView`檢視的一個區別是，它不是一個建構函式，而是一組建構函式，代表不同的資料格式。

```javascript
var buffer = new ArrayBuffer(12);

var x1 = new Int32Array(buffer);
x1[0] = 1;
var x2 = new Uint8Array(buffer);
x2[0]  = 2;

x1[0] // 2
```

上面程式碼對同一段記憶體，分別建立兩種檢視：32位帶符號整數（`Int32Array`建構函式）和8位不帶符號整數（`Uint8Array`建構函式）。由於兩個檢視對應的是同一段記憶體，一個檢視修改底層記憶體，會影響到另一個檢視。

TypedArray檢視的建構函式，除了接受`ArrayBuffer`實例作為引數，還可以接受普通陣列作為引數，直接分配記憶體生成底層的`ArrayBuffer`實例，並同時完成對這段記憶體的賦值。

```javascript
var typedArray = new Uint8Array([0,1,2]);
typedArray.length // 3

typedArray[0] = 5;
typedArray // [5, 1, 2]
```

上面程式碼使用TypedArray檢視的`Uint8Array`建構函式，新建一個不帶符號的8位整數檢視。可以看到，`Uint8Array`直接使用普通陣列作為引數，對底層記憶體的賦值同時完成。

### ArrayBuffer.prototype.byteLength

`ArrayBuffer`實例的`byteLength`屬性，返回所分配的記憶體區域的位元組長度。

```javascript
var buffer = new ArrayBuffer(32);
buffer.byteLength
// 32
```

如果要分配的記憶體區域很大，有可能分配失敗（因為沒有那麼多的連續空餘記憶體），所以有必要檢查是否分配成功。

```javascript
if (buffer.byteLength === n) {
  // 成功
} else {
  // 失敗
}
```

### ArrayBuffer.prototype.slice()

`ArrayBuffer`實例有一個`slice`方法，允許將記憶體區域的一部分，拷貝生成一個新的`ArrayBuffer`物件。

```javascript
var buffer = new ArrayBuffer(8);
var newBuffer = buffer.slice(0, 3);
```

上面程式碼拷貝`buffer`物件的前3個位元組（從0開始，到第3個位元組前面結束），生成一個新的`ArrayBuffer`物件。`slice`方法其實包含兩步，第一步是先分配一段新記憶體，第二步是將原來那個`ArrayBuffer`物件拷貝過去。

`slice`方法接受兩個引數，第一個引數表示拷貝開始的位元組序號（含該位元組），第二個引數表示拷貝截止的位元組序號（不含該位元組）。如果省略第二個引數，則預設到原`ArrayBuffer`物件的結尾。

除了`slice`方法，`ArrayBuffer`物件不提供任何直接讀寫記憶體的方法，只允許在其上方建立檢視，然後通過檢視讀寫。

### ArrayBuffer.isView()

`ArrayBuffer`有一個靜態方法`isView`，返回一個布林值，表示引數是否為`ArrayBuffer`的檢視實例。這個方法大致相當於判斷引數，是否為TypedArray實例或`DataView`實例。

```javascript
var buffer = new ArrayBuffer(8);
ArrayBuffer.isView(buffer) // false

var v = new Int32Array(buffer);
ArrayBuffer.isView(v) // true
```

## TypedArray檢視

### 概述

`ArrayBuffer`物件作為記憶體區域，可以存放多種型別的資料。同一段記憶體，不同資料有不同的解讀方式，這就叫做“檢視”（view）。`ArrayBuffer`有兩種檢視，一種是TypedArray檢視，另一種是`DataView`檢視。前者的陣列成員都是同一個資料型別，後者的陣列成員可以是不同的資料型別。

目前，TypedArray檢視一共包括9種類型，每一種檢視都是一種建構函式。

- **`Int8Array`**：8位有符號整數，長度1個位元組。
- **`Uint8Array`**：8位無符號整數，長度1個位元組。
- **`Uint8ClampedArray`**：8位無符號整數，長度1個位元組，溢位處理不同。
- **`Int16Array`**：16位有符號整數，長度2個位元組。
- **`Uint16Array`**：16位無符號整數，長度2個位元組。
- **`Int32Array`**：32位有符號整數，長度4個位元組。
- **`Uint32Array`**：32位無符號整數，長度4個位元組。
- **`Float32Array`**：32位浮點數，長度4個位元組。
- **`Float64Array`**：64位浮點數，長度8個位元組。

這9個建構函式生成的陣列，統稱為TypedArray檢視。它們很像普通陣列，都有`length`屬性，都能用方括號運算子（`[]`）獲取單個元素，所有陣列的方法，在它們上面都能使用。普通陣列與TypedArray陣列的差異主要在以下方面。

- TypedArray陣列的所有成員，都是同一種類型。
- TypedArray陣列的成員是連續的，不會有空位。
- TypedArray陣列成員的預設值為0。比如，`new Array(10)`返回一個普通陣列，裡面沒有任何成員，只是10個空位；`new Uint8Array(10)`返回一個TypedArray陣列，裡面10個成員都是0。
- TypedArray陣列只是一層檢視，本身不儲存資料，它的資料都儲存在底層的`ArrayBuffer`物件之中，要獲取底層物件必須使用`buffer`屬性。

### 建構函式

TypedArray陣列提供9種建構函式，用來生成相應型別的陣列實例。

建構函式有多種用法。

**（1）TypedArray(buffer, byteOffset=0, length?)**

同一個`ArrayBuffer`物件之上，可以根據不同的資料型別，建立多個檢視。

```javascript
// 建立一個8位元組的ArrayBuffer
var b = new ArrayBuffer(8);

// 建立一個指向b的Int32檢視，開始於位元組0，直到緩衝區的末尾
var v1 = new Int32Array(b);

// 建立一個指向b的Uint8檢視，開始於位元組2，直到緩衝區的末尾
var v2 = new Uint8Array(b, 2);

// 建立一個指向b的Int16檢視，開始於位元組2，長度為2
var v3 = new Int16Array(b, 2, 2);
```

上面程式碼在一段長度為8個位元組的記憶體（`b`）之上，生成了三個檢視：`v1`、`v2`和`v3`。

檢視的建構函式可以接受三個引數：

- 第一個引數（必需）：檢視對應的底層`ArrayBuffer`物件。
- 第二個引數（可選）：檢視開始的位元組序號，預設從0開始。
- 第三個引數（可選）：檢視包含的資料個數，預設直到本段記憶體區域結束。

因此，`v1`、`v2`和`v3`是重疊的：`v1[0]`是一個32位整數，指向位元組0～位元組3；`v2[0]`是一個8位無符號整數，指向位元組2；`v3[0]`是一個16位整數，指向位元組2～位元組3。只要任何一個檢視對記憶體有所修改，就會在另外兩個檢視上反應出來。

注意，`byteOffset`必須與所要建立的資料型別一致，否則會報錯。

```javascript
var buffer = new ArrayBuffer(8);
var i16 = new Int16Array(buffer, 1);
// Uncaught RangeError: start offset of Int16Array should be a multiple of 2
```

上面程式碼中，新生成一個8個位元組的`ArrayBuffer`物件，然後在這個物件的第一個位元組，建立帶符號的16位整數檢視，結果報錯。因為，帶符號的16位整數需要兩個位元組，所以`byteOffset`引數必須能夠被2整除。

如果想從任意位元組開始解讀`ArrayBuffer`物件，必須使用`DataView`檢視，因為TypedArray檢視只提供9種固定的解讀格式。

**（2）TypedArray(length)**

檢視還可以不通過`ArrayBuffer`物件，直接分配記憶體而生成。

```javascript
var f64a = new Float64Array(8);
f64a[0] = 10;
f64a[1] = 20;
f64a[2] = f64a[0] + f64a[1];
```

上面程式碼生成一個8個成員的`Float64Array`陣列（共64位元組），然後依次對每個成員賦值。這時，檢視建構函式的引數就是成員的個數。可以看到，檢視陣列的賦值操作與普通陣列的操作毫無兩樣。

**（3）TypedArray(typedArray)**

TypedArray陣列的建構函式，可以接受另一個TypedArray實例作為引數。

```javascript
var typedArray = new Int8Array(new Uint8Array(4));
```

上面程式碼中，`Int8Array`建構函式接受一個`Uint8Array`實例作為引數。

注意，此時生成的新陣列，只是複製了引數陣列的值，對應的底層記憶體是不一樣的。新陣列會開闢一段新的記憶體儲存資料，不會在原陣列的記憶體之上建立檢視。

```javascript
var x = new Int8Array([1, 1]);
var y = new Int8Array(x);
x[0] // 1
y[0] // 1

x[0] = 2;
y[0] // 1
```

上面程式碼中，陣列`y`是以陣列`x`為模板而生成的，當`x`變動的時候，`y`並沒有變動。

如果想基於同一段記憶體，構造不同的檢視，可以採用下面的寫法。

```javascript
var x = new Int8Array([1, 1]);
var y = new Int8Array(x.buffer);
x[0] // 1
y[0] // 1

x[0] = 2;
y[0] // 2
```

**（4）TypedArray(arrayLikeObject)**

建構函式的引數也可以是一個普通陣列，然後直接生成TypedArray實例。

```javascript
var typedArray = new Uint8Array([1, 2, 3, 4]);
```

注意，這時TypedArray檢視會重新開闢記憶體，不會在原陣列的記憶體上建立檢視。

上面程式碼從一個普通的陣列，生成一個8位無符號整數的TypedArray實例。

TypedArray陣列也可以轉換回普通陣列。

```javascript
var normalArray = Array.prototype.slice.call(typedArray);
```

### 陣列方法

普通陣列的操作方法和屬性，對TypedArray陣列完全適用。

- `TypedArray.prototype.copyWithin(target, start[, end = this.length])`
- `TypedArray.prototype.entries()`
- `TypedArray.prototype.every(callbackfn, thisArg?)`
- `TypedArray.prototype.fill(value, start=0, end=this.length)`
- `TypedArray.prototype.filter(callbackfn, thisArg?)`
- `TypedArray.prototype.find(predicate, thisArg?)`
- `TypedArray.prototype.findIndex(predicate, thisArg?)`
- `TypedArray.prototype.forEach(callbackfn, thisArg?)`
- `TypedArray.prototype.indexOf(searchElement, fromIndex=0)`
- `TypedArray.prototype.join(separator)`
- `TypedArray.prototype.keys()`
- `TypedArray.prototype.lastIndexOf(searchElement, fromIndex?)`
- `TypedArray.prototype.map(callbackfn, thisArg?)`
- `TypedArray.prototype.reduce(callbackfn, initialValue?)`
- `TypedArray.prototype.reduceRight(callbackfn, initialValue?)`
- `TypedArray.prototype.reverse()`
- `TypedArray.prototype.slice(start=0, end=this.length)`
- `TypedArray.prototype.some(callbackfn, thisArg?)`
- `TypedArray.prototype.sort(comparefn)`
- `TypedArray.prototype.toLocaleString(reserved1?, reserved2?)`
- `TypedArray.prototype.toString()`
- `TypedArray.prototype.values()`

上面所有方法的用法，請參閱陣列方法的介紹，這裡不再重複了。

注意，TypedArray陣列沒有`concat`方法。如果想要合併多個TypedArray陣列，可以用下面這個函式。

```javascript
function concatenate(resultConstructor, ...arrays) {
  let totalLength = 0;
  for (let arr of arrays) {
    totalLength += arr.length;
  }
  let result = new resultConstructor(totalLength);
  let offset = 0;
  for (let arr of arrays) {
    result.set(arr, offset);
    offset += arr.length;
  }
  return result;
}

concatenate(Uint8Array, Uint8Array.of(1, 2), Uint8Array.of(3, 4))
// Uint8Array [1, 2, 3, 4]
```

另外，TypedArray陣列與普通陣列一樣，部署了Iterator介面，所以可以被遍歷。

```javascript
let ui8 = Uint8Array.of(0, 1, 2);
for (let byte of ui8) {
  console.log(byte);
}
// 0
// 1
// 2
```

### 位元組序

位元組序指的是數值在記憶體中的表示方式。

```javascript
var buffer = new ArrayBuffer(16);
var int32View = new Int32Array(buffer);

for (var i = 0; i < int32View.length; i++) {
  int32View[i] = i * 2;
}
```

上面程式碼生成一個16位元組的`ArrayBuffer`物件，然後在它的基礎上，建立了一個32位整數的檢視。由於每個32位整數佔據4個位元組，所以一共可以寫入4個整數，依次為0，2，4，6。

如果在這段資料上接著建立一個16位整數的檢視，則可以讀出完全不一樣的結果。

```javascript
var int16View = new Int16Array(buffer);

for (var i = 0; i < int16View.length; i++) {
  console.log("Entry " + i + ": " + int16View[i]);
}
// Entry 0: 0
// Entry 1: 0
// Entry 2: 2
// Entry 3: 0
// Entry 4: 4
// Entry 5: 0
// Entry 6: 6
// Entry 7: 0
```

由於每個16位整數佔據2個位元組，所以整個`ArrayBuffer`物件現在分成8段。然後，由於x86體系的計算機都採用小端位元組序（little endian），相對重要的位元組排在後面的記憶體地址，相對不重要位元組排在前面的記憶體地址，所以就得到了上面的結果。

比如，一個佔據四個位元組的16進位制數`0x12345678`，決定其大小的最重要的位元組是“12”，最不重要的是“78”。小端位元組序將最不重要的位元組排在前面，儲存順序就是`78563412`；大端位元組序則完全相反，將最重要的位元組排在前面，儲存順序就是`12345678`。目前，所有個人電腦幾乎都是小端位元組序，所以TypedArray陣列內部也採用小端位元組序讀寫資料，或者更準確的說，按照本機作業系統設定的位元組序讀寫資料。

這並不意味大端位元組序不重要，事實上，很多網路裝置和特定的作業系統採用的是大端位元組序。這就帶來一個嚴重的問題：如果一段資料是大端位元組序，TypedArray陣列將無法正確解析，因為它只能處理小端位元組序！為了解決這個問題，JavaScript引入`DataView`物件，可以設定位元組序，下文會詳細介紹。

下面是另一個例子。

```javascript
// 假定某段buffer包含如下位元組 [0x02, 0x01, 0x03, 0x07]
var buffer = new ArrayBuffer(4);
var v1 = new Uint8Array(buffer);
v1[0] = 2;
v1[1] = 1;
v1[2] = 3;
v1[3] = 7;

var uInt16View = new Uint16Array(buffer);

// 計算機採用小端位元組序
// 所以頭兩個位元組等於258
if (uInt16View[0] === 258) {
  console.log('OK'); // "OK"
}

// 賦值運算
uInt16View[0] = 255;    // 位元組變為[0xFF, 0x00, 0x03, 0x07]
uInt16View[0] = 0xff05; // 位元組變為[0x05, 0xFF, 0x03, 0x07]
uInt16View[1] = 0x0210; // 位元組變為[0x05, 0xFF, 0x10, 0x02]
```

下面的函式可以用來判斷，當前檢視是小端位元組序，還是大端位元組序。

```javascript
const BIG_ENDIAN = Symbol('BIG_ENDIAN');
const LITTLE_ENDIAN = Symbol('LITTLE_ENDIAN');

function getPlatformEndianness() {
  let arr32 = Uint32Array.of(0x12345678);
  let arr8 = new Uint8Array(arr32.buffer);
  switch ((arr8[0]*0x1000000) + (arr8[1]*0x10000) + (arr8[2]*0x100) + (arr8[3])) {
    case 0x12345678:
      return BIG_ENDIAN;
    case 0x78563412:
      return LITTLE_ENDIAN;
    default:
      throw new Error('Unknown endianness');
  }
}
```

總之，與普通陣列相比，TypedArray陣列的最大優點就是可以直接操作記憶體，不需要資料型別轉換，所以速度快得多。

### BYTES_PER_ELEMENT屬性

每一種檢視的建構函式，都有一個`BYTES_PER_ELEMENT`屬性，表示這種資料型別佔據的位元組數。

```javascript
Int8Array.BYTES_PER_ELEMENT // 1
Uint8Array.BYTES_PER_ELEMENT // 1
Int16Array.BYTES_PER_ELEMENT // 2
Uint16Array.BYTES_PER_ELEMENT // 2
Int32Array.BYTES_PER_ELEMENT // 4
Uint32Array.BYTES_PER_ELEMENT // 4
Float32Array.BYTES_PER_ELEMENT // 4
Float64Array.BYTES_PER_ELEMENT // 8
```

這個屬性在TypedArray實例上也能獲取，即有`TypedArray.prototype.BYTES_PER_ELEMENT`。

### ArrayBuffer與字串的互相轉換

`ArrayBuffer`轉為字串，或者字串轉為`ArrayBuffer`，有一個前提，即字串的編碼方法是確定的。假定字串採用UTF-16編碼（JavaScript的內部編碼方式），可以自己編寫轉換函式。

```javascript
// ArrayBuffer轉為字串，引數為ArrayBuffer物件
function ab2str(buf) {
  return String.fromCharCode.apply(null, new Uint16Array(buf));
}

// 字串轉為ArrayBuffer物件，引數為字串
function str2ab(str) {
  var buf = new ArrayBuffer(str.length * 2); // 每個字元佔用2個位元組
  var bufView = new Uint16Array(buf);
  for (var i = 0, strLen = str.length; i < strLen; i++) {
    bufView[i] = str.charCodeAt(i);
  }
  return buf;
}
```

### 溢位

不同的檢視型別，所能容納的數值範圍是確定的。超出這個範圍，就會出現溢位。比如，8位檢視只能容納一個8位的二進位制值，如果放入一個9位的值，就會溢位。

TypedArray陣列的溢位處理規則，簡單來說，就是拋棄溢位的位，然後按照檢視型別進行解釋。

```javascript
var uint8 = new Uint8Array(1);

uint8[0] = 256;
uint8[0] // 0

uint8[0] = -1;
uint8[0] // 255
```

上面程式碼中，`uint8`是一個8位檢視，而256的二進位制形式是一個9位的值`100000000`，這時就會發生溢位。根據規則，只會保留後8位，即`00000000`。`uint8`檢視的解釋規則是無符號的8位整數，所以`00000000`就是`0`。

負數在計算機內部採用“2的補碼”表示，也就是說，將對應的正數值進行否運算，然後加`1`。比如，`-1`對應的正值是`1`，進行否運算以後，得到`11111110`，再加上`1`就是補碼形式`11111111`。`uint8`按照無符號的8位整數解釋`11111111`，返回結果就是`255`。

一個簡單轉換規則，可以這樣表示。

- 正向溢位（overflow）：當輸入值大於當前資料型別的最大值，結果等於當前資料型別的最小值加上餘值，再減去1。
- 負向溢位（underflow）：當輸入值小於當前資料型別的最小值，結果等於當前資料型別的最大值減去餘值，再加上1。

上面的“餘值”就是模運算的結果，即 JavaScript 裡面的`%`運算子的結果。

```javascript
12 % 4 // 0
12 % 5 // 2
```

上面程式碼中，12除以4是沒有餘值的，而除以5會得到餘值2。

請看下面的例子。

```javascript
var int8 = new Int8Array(1);

int8[0] = 128;
int8[0] // -128

int8[0] = -129;
int8[0] // 127
```

上面例子中，`int8`是一個帶符號的8位整數檢視，它的最大值是127，最小值是-128。輸入值為`128`時，相當於正向溢位`1`，根據“最小值加上餘值（128除以127的餘值是1），再減去1”的規則，就會返回`-128`；輸入值為`-129`時，相當於負向溢位`1`，根據“最大值減去餘值（-129除以-128的餘值是1），再加上1”的規則，就會返回`127`。

`Uint8ClampedArray`檢視的溢位規則，與上面的規則不同。它規定，凡是發生正向溢位，該值一律等於當前資料型別的最大值，即255；如果發生負向溢位，該值一律等於當前資料型別的最小值，即0。

```javascript
var uint8c = new Uint8ClampedArray(1);

uint8c[0] = 256;
uint8c[0] // 255

uint8c[0] = -1;
uint8c[0] // 0
```

上面例子中，`uint8C`是一個`Uint8ClampedArray`檢視，正向溢位時都返回255，負向溢位都返回0。

### TypedArray.prototype.buffer

TypedArray實例的`buffer`屬性，返回整段記憶體區域對應的`ArrayBuffer`物件。該屬性為只讀屬性。

```javascript
var a = new Float32Array(64);
var b = new Uint8Array(a.buffer);
```

上面程式碼的`a`檢視物件和`b`檢視物件，對應同一個`ArrayBuffer`物件，即同一段記憶體。

### TypedArray.prototype.byteLength，TypedArray.prototype.byteOffset

`byteLength`屬性返回TypedArray陣列佔據的記憶體長度，單位為位元組。`byteOffset`屬性返回TypedArray陣列從底層`ArrayBuffer`物件的哪個位元組開始。這兩個屬性都是隻讀屬性。

```javascript
var b = new ArrayBuffer(8);

var v1 = new Int32Array(b);
var v2 = new Uint8Array(b, 2);
var v3 = new Int16Array(b, 2, 2);

v1.byteLength // 8
v2.byteLength // 6
v3.byteLength // 4

v1.byteOffset // 0
v2.byteOffset // 2
v3.byteOffset // 2
```

### TypedArray.prototype.length

`length`屬性表示TypedArray陣列含有多少個成員。注意將`byteLength`屬性和`length`屬性區分，前者是位元組長度，後者是成員長度。

```javascript
var a = new Int16Array(8);

a.length // 8
a.byteLength // 16
```

### TypedArray.prototype.set()

TypedArray陣列的`set`方法用於複製陣列（普通陣列或TypedArray陣列），也就是將一段內容完全複製到另一段記憶體。

```javascript
var a = new Uint8Array(8);
var b = new Uint8Array(8);

b.set(a);
```

上面程式碼複製`a`陣列的內容到`b`陣列，它是整段記憶體的複製，比一個個拷貝成員的那種複製快得多。

`set`方法還可以接受第二個引數，表示從`b`物件的哪一個成員開始複製`a`物件。

```javascript
var a = new Uint16Array(8);
var b = new Uint16Array(10);

b.set(a, 2)
```

上面程式碼的`b`陣列比`a`陣列多兩個成員，所以從`b[2]`開始複製。

### TypedArray.prototype.subarray()

`subarray`方法是對於TypedArray陣列的一部分，再建立一個新的檢視。

```javascript
var a = new Uint16Array(8);
var b = a.subarray(2,3);

a.byteLength // 16
b.byteLength // 2
```

`subarray`方法的第一個引數是起始的成員序號，第二個引數是結束的成員序號（不含該成員），如果省略則包含剩餘的全部成員。所以，上面程式碼的`a.subarray(2,3)`，意味著b只包含`a[2]`一個成員，位元組長度為2。

### TypedArray.prototype.slice()

TypeArray實例的`slice`方法，可以返回一個指定位置的新的TypedArray實例。

```javascript
let ui8 = Uint8Array.of(0, 1, 2);
ui8.slice(-1)
// Uint8Array [ 2 ]
```

上面程式碼中，`ui8`是8位無符號整數陣列檢視的一個實例。它的`slice`方法可以從當前檢視之中，返回一個新的檢視實例。

`slice`方法的引數，表示原陣列的具體位置，開始生成新陣列。負值表示逆向的位置，即-1為倒數第一個位置，-2表示倒數第二個位置，以此類推。

### TypedArray.of()

TypedArray陣列的所有建構函式，都有一個靜態方法`of`，用於將引數轉為一個TypedArray實例。

```javascript
Float32Array.of(0.151, -8, 3.7)
// Float32Array [ 0.151, -8, 3.7 ]
```

下面三種方法都會生成同樣一個TypedArray陣列。

```javascript
// 方法一
let tarr = new Uint8Array([1,2,3]);

// 方法二
let tarr = Uint8Array.of(1,2,3);

// 方法三
let tarr = new Uint8Array(3);
tarr[0] = 1;
tarr[1] = 2;
tarr[2] = 3;
```

### TypedArray.from()

靜態方法`from`接受一個可遍歷的資料結構（比如陣列）作為引數，返回一個基於這個結構的TypedArray實例。

```javascript
Uint16Array.from([0, 1, 2])
// Uint16Array [ 0, 1, 2 ]
```

這個方法還可以將一種TypedArray實例，轉為另一種。

```javascript
var ui16 = Uint16Array.from(Uint8Array.of(0, 1, 2));
ui16 instanceof Uint16Array // true
```

`from`方法還可以接受一個函式，作為第二個引數，用來對每個元素進行遍歷，功能類似`map`方法。

```javascript
Int8Array.of(127, 126, 125).map(x => 2 * x)
// Int8Array [ -2, -4, -6 ]

Int16Array.from(Int8Array.of(127, 126, 125), x => 2 * x)
// Int16Array [ 254, 252, 250 ]
```

上面的例子中，`from`方法沒有發生溢位，這說明遍歷不是針對原來的8位整數陣列。也就是說，`from`會將第一個引數指定的TypedArray陣列，拷貝到另一段記憶體之中，處理之後再將結果轉成指定的陣列格式。

## 複合檢視

由於檢視的建構函式可以指定起始位置和長度，所以在同一段記憶體之中，可以依次存放不同型別的資料，這叫做“複合檢視”。

```javascript
var buffer = new ArrayBuffer(24);

var idView = new Uint32Array(buffer, 0, 1);
var usernameView = new Uint8Array(buffer, 4, 16);
var amountDueView = new Float32Array(buffer, 20, 1);
```

上面程式碼將一個24位元組長度的`ArrayBuffer`物件，分成三個部分：

- 位元組0到位元組3：1個32位無符號整數
- 位元組4到位元組19：16個8位整數
- 位元組20到位元組23：1個32位浮點數

這種資料結構可以用如下的C語言描述：

```c
struct someStruct {
  unsigned long id;
  char username[16];
  float amountDue;
};
```

## DataView檢視

如果一段資料包括多種型別（比如伺服器傳來的HTTP資料），這時除了建立`ArrayBuffer`物件的複合檢視以外，還可以通過`DataView`檢視進行操作。

`DataView`檢視提供更多操作選項，而且支援設定位元組序。本來，在設計目的上，`ArrayBuffer`物件的各種TypedArray檢視，是用來向網絡卡、音效卡之類的本機裝置傳送資料，所以使用本機的位元組序就可以了；而`DataView`檢視的設計目的，是用來處理網路裝置傳來的資料，所以大端位元組序或小端位元組序是可以自行設定的。

`DataView`檢視本身也是建構函式，接受一個`ArrayBuffer`物件作為引數，生成檢視。

```javascript
DataView(ArrayBuffer buffer [, 位元組起始位置 [, 長度]]);
```

下面是一個例子。

```javascript
var buffer = new ArrayBuffer(24);
var dv = new DataView(buffer);
```

`DataView`實例有以下屬性，含義與TypedArray實例的同名方法相同。

- `DataView.prototype.buffer`：返回對應的ArrayBuffer物件
- `DataView.prototype.byteLength`：返回佔據的記憶體位元組長度
- `DataView.prototype.byteOffset`：返回當前檢視從對應的ArrayBuffer物件的哪個位元組開始

`DataView`實例提供8個方法讀取記憶體。

- **`getInt8`**：讀取1個位元組，返回一個8位整數。
- **`getUint8`**：讀取1個位元組，返回一個無符號的8位整數。
- **`getInt16`**：讀取2個位元組，返回一個16位整數。
- **`getUint16`**：讀取2個位元組，返回一個無符號的16位整數。
- **`getInt32`**：讀取4個位元組，返回一個32位整數。
- **`getUint32`**：讀取4個位元組，返回一個無符號的32位整數。
- **`getFloat32`**：讀取4個位元組，返回一個32位浮點數。
- **`getFloat64`**：讀取8個位元組，返回一個64位浮點數。

這一系列`get`方法的引數都是一個位元組序號（不能是負數，否則會報錯），表示從哪個位元組開始讀取。

```javascript
var buffer = new ArrayBuffer(24);
var dv = new DataView(buffer);

// 從第1個位元組讀取一個8位無符號整數
var v1 = dv.getUint8(0);

// 從第2個位元組讀取一個16位無符號整數
var v2 = dv.getUint16(1);

// 從第4個位元組讀取一個16位無符號整數
var v3 = dv.getUint16(3);
```

上面程式碼讀取了`ArrayBuffer`物件的前5個位元組，其中有一個8位整數和兩個十六位整數。

如果一次讀取兩個或兩個以上位元組，就必須明確資料的儲存方式，到底是小端位元組序還是大端位元組序。預設情況下，`DataView`的`get`方法使用大端位元組序解讀資料，如果需要使用小端位元組序解讀，必須在`get`方法的第二個引數指定`true`。

```javascript
// 小端位元組序
var v1 = dv.getUint16(1, true);

// 大端位元組序
var v2 = dv.getUint16(3, false);

// 大端位元組序
var v3 = dv.getUint16(3);
```

DataView檢視提供8個方法寫入記憶體。

- **`setInt8`**：寫入1個位元組的8位整數。
- **`setUint8`**：寫入1個位元組的8位無符號整數。
- **`setInt16`**：寫入2個位元組的16位整數。
- **`setUint16`**：寫入2個位元組的16位無符號整數。
- **`setInt32`**：寫入4個位元組的32位整數。
- **`setUint32`**：寫入4個位元組的32位無符號整數。
- **`setFloat32`**：寫入4個位元組的32位浮點數。
- **`setFloat64`**：寫入8個位元組的64位浮點數。

這一系列`set`方法，接受兩個引數，第一個引數是位元組序號，表示從哪個位元組開始寫入，第二個引數為寫入的資料。對於那些寫入兩個或兩個以上位元組的方法，需要指定第三個引數，`false`或者`undefined`表示使用大端位元組序寫入，`true`表示使用小端位元組序寫入。

```javascript
// 在第1個位元組，以大端位元組序寫入值為25的32位整數
dv.setInt32(0, 25, false);

// 在第5個位元組，以大端位元組序寫入值為25的32位整數
dv.setInt32(4, 25);

// 在第9個位元組，以小端位元組序寫入值為2.5的32位浮點數
dv.setFloat32(8, 2.5, true);
```

如果不確定正在使用的計算機的位元組序，可以採用下面的判斷方式。

```javascript
var littleEndian = (function() {
  var buffer = new ArrayBuffer(2);
  new DataView(buffer).setInt16(0, 256, true);
  return new Int16Array(buffer)[0] === 256;
})();
```

如果返回`true`，就是小端位元組序；如果返回`false`，就是大端位元組序。

## 二進位制陣列的應用

大量的Web API用到了`ArrayBuffer`物件和它的檢視物件。

### AJAX

傳統上，伺服器通過AJAX操作只能返回文字資料，即`responseType`屬性預設為`text`。`XMLHttpRequest`第二版`XHR2`允許伺服器返回二進位制資料，這時分成兩種情況。如果明確知道返回的二進位制資料型別，可以把返回型別（`responseType`）設為`arraybuffer`；如果不知道，就設為`blob`。

```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', someUrl);
xhr.responseType = 'arraybuffer';

xhr.onload = function () {
  let arrayBuffer = xhr.response;
  // ···
};

xhr.send();
```

如果知道傳回來的是32位整數，可以像下面這樣處理。

```javascript
xhr.onreadystatechange = function () {
  if (req.readyState === 4 ) {
    var arrayResponse = xhr.response;
    var dataView = new DataView(arrayResponse);
    var ints = new Uint32Array(dataView.byteLength / 4);

    xhrDiv.style.backgroundColor = "#00FF00";
    xhrDiv.innerText = "Array is " + ints.length + "uints long";
  }
}
```

### Canvas

網頁`Canvas`元素輸出的二進位制畫素資料，就是TypedArray陣列。

```javascript
var canvas = document.getElementById('myCanvas');
var ctx = canvas.getContext('2d');

var imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
var uint8ClampedArray = imageData.data;
```

需要注意的是，上面程式碼的`uint8ClampedArray`雖然是一個TypedArray陣列，但是它的檢視型別是一種針對`Canvas`元素的專有型別`Uint8ClampedArray`。這個檢視型別的特點，就是專門針對顏色，把每個位元組解讀為無符號的8位整數，即只能取值0～255，而且發生運算的時候自動過濾高位溢位。這為影象處理帶來了巨大的方便。

舉例來說，如果把畫素的顏色值設為`Uint8Array`型別，那麼乘以一個gamma值的時候，就必須這樣計算：

```javascript
u8[i] = Math.min(255, Math.max(0, u8[i] * gamma));
```

因為`Uint8Array`型別對於大於255的運算結果（比如`0xFF+1`），會自動變為`0x00`，所以影象處理必須要像上面這樣算。這樣做很麻煩，而且影響效能。如果將顏色值設為`Uint8ClampedArray`型別，計算就簡化許多。

```javascript
pixels[i] *= gamma;
```

`Uint8ClampedArray`型別確保將小於0的值設為0，將大於255的值設為255。注意，IE 10不支援該型別。

### WebSocket

`WebSocket`可以通過`ArrayBuffer`，傳送或接收二進位制資料。

```javascript
var socket = new WebSocket('ws://127.0.0.1:8081');
socket.binaryType = 'arraybuffer';

// Wait until socket is open
socket.addEventListener('open', function (event) {
  // Send binary data
  var typedArray = new Uint8Array(4);
  socket.send(typedArray.buffer);
});

// Receive binary data
socket.addEventListener('message', function (event) {
  var arrayBuffer = event.data;
  // ···
});
```

### Fetch API

Fetch API取回的資料，就是`ArrayBuffer`物件。

```javascript
fetch(url)
.then(function(request){
  return request.arrayBuffer()
})
.then(function(arrayBuffer){
  // ...
});
```

### File API

如果知道一個檔案的二進位制資料型別，也可以將這個檔案讀取為`ArrayBuffer`物件。

```javascript
var fileInput = document.getElementById('fileInput');
var file = fileInput.files[0];
var reader = new FileReader();
reader.readAsArrayBuffer(file);
reader.onload = function () {
  var arrayBuffer = reader.result;
  // ···
};
```

下面以處理bmp檔案為例。假定`file`變數是一個指向bmp檔案的檔案物件，首先讀取檔案。

```javascript
var reader = new FileReader();
reader.addEventListener("load", processimage, false);
reader.readAsArrayBuffer(file);
```

然後，定義處理影象的回呼函式：先在二進位制資料之上建立一個`DataView`檢視，再建立一個`bitmap`物件，用於存放處理後的資料，最後將影象展示在`Canvas`元素之中。

```javascript
function processimage(e) {
  var buffer = e.target.result;
  var datav = new DataView(buffer);
  var bitmap = {};
  // 具體的處理步驟
}
```

具體處理影象資料時，先處理bmp的檔案頭。具體每個檔案頭的格式和定義，請參閱有關資料。

```javascript
bitmap.fileheader = {};
bitmap.fileheader.bfType = datav.getUint16(0, true);
bitmap.fileheader.bfSize = datav.getUint32(2, true);
bitmap.fileheader.bfReserved1 = datav.getUint16(6, true);
bitmap.fileheader.bfReserved2 = datav.getUint16(8, true);
bitmap.fileheader.bfOffBits = datav.getUint32(10, true);
```

接著處理影象元資訊部分。

```javascript
bitmap.infoheader = {};
bitmap.infoheader.biSize = datav.getUint32(14, true);
bitmap.infoheader.biWidth = datav.getUint32(18, true);
bitmap.infoheader.biHeight = datav.getUint32(22, true);
bitmap.infoheader.biPlanes = datav.getUint16(26, true);
bitmap.infoheader.biBitCount = datav.getUint16(28, true);
bitmap.infoheader.biCompression = datav.getUint32(30, true);
bitmap.infoheader.biSizeImage = datav.getUint32(34, true);
bitmap.infoheader.biXPelsPerMeter = datav.getUint32(38, true);
bitmap.infoheader.biYPelsPerMeter = datav.getUint32(42, true);
bitmap.infoheader.biClrUsed = datav.getUint32(46, true);
bitmap.infoheader.biClrImportant = datav.getUint32(50, true);
```

最後處理影象本身的畫素資訊。

```javascript
var start = bitmap.fileheader.bfOffBits;
bitmap.pixels = new Uint8Array(buffer, start);
```

至此，影象檔案的資料全部處理完成。下一步，可以根據需要，進行影象變形，或者轉換格式，或者展示在`Canvas`網頁元素之中。

## SharedArrayBuffer

JavaScript 是單執行緒的，web worker 引入了多程序，每個程序的資料都是隔離的，通過`postMessage()`通訊，即通訊的資料是複製的。如果資料量比較大，這種通訊的效率顯然比較低。

```javascript
var w = new Worker('myworker.js');
```

上面程式碼中，主程序新建了一個 Worker 程序。該程序與主程序之間會有一個通訊渠道，主程序通過`w.postMessage`向 Worker 程序發訊息，同時通過`message`事件監聽 Worker 程序的迴應。

```javascript
w.postMessage('hi');
w.onmessage = function (ev) {
  console.log(ev.data);
}
```

上面程式碼中，主程序先發一個訊息`hi`，然後在監聽到 Worker 程序的迴應後，就將其打印出來。

Worker 程序也是通過監聽`message`事件，來獲取主程序發來的訊息，並作出反應。

```javascript
onmessage = function (ev) {
  console.log(ev.data);
  postMessage('ho');
}
```

主程序與 Worker 程序之間，可以傳送各種資料，不僅僅是字串，還可以傳送二進位制資料。很容易想到，如果有大量資料要傳送，留出一塊記憶體區域，主程序與 Worker 程序共享，兩方都可以讀寫，那麼就會大大提高效率。

ES2017 引入[`SharedArrayBuffer`](https://github.com/tc39/ecmascript_sharedmem/blob/master/TUTORIAL.md)，允許多個 Worker 程序與主程序共享記憶體資料。`SharedArrayBuffer`的 API 與`ArrayBuffer`一模一樣，唯一的區別是後者無法共享。

```javascript
// 新建 1KB 共享記憶體
var sharedBuffer = new SharedArrayBuffer(1024);

// 主視窗傳送資料
w.postMessage(sharedBuffer);

// 本地寫入資料
const sharedArray = new Int32Array(sharedBuffer);
```

上面程式碼中，`postMessage`方法的引數是`SharedArrayBuffer`物件。

Worker 程序從事件的`data`屬性上面取到資料。

```javascript
var sharedBuffer;
onmessage = function (ev) {
   sharedBuffer = ev.data;  // 1KB 的共享記憶體，就是主視窗共享出來的那塊記憶體
};
```

共享記憶體也可以在 Worker 程序建立，發給主程序。

`SharedArrayBuffer`與`SharedArray`一樣，本身是無法讀寫，必須在上面建立檢視，然後通過檢視讀寫。

```javascript
// 分配 10 萬個 32 位整數佔據的記憶體空間
var sab = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * 100000);

// 建立 32 位整數檢視
var ia = new Int32Array(sab);  // ia.length == 100000

// 新建一個質數生成器
var primes = new PrimeGenerator();

// 將 10 萬個質數，寫入這段記憶體空間
for ( let i=0 ; i < ia.length ; i++ )
  ia[i] = primes.next();

// 向 Worker 程序傳送這段共享記憶體
w.postMessage(ia);
```

Worker 程序收到資料後的處理如下。

```javascript
var ia;
onmessage = function (ev) {
  ia = ev.data;
  console.log(ia.length); // 100000
  console.log(ia[37]); // 輸出 163，因為這是第138個質數
};
```

多個程序共享記憶體，最大的問題就是如何防止兩個程序同時修改某個地址，或者說，當一個程序修改共享記憶體以後，必須有一個機制讓其他程序同步。SharedArrayBuffer API 提供`Atomics`物件，保證所有共享記憶體的操作都是“原子性”的，並且可以在所有程序內同步。

```javascript
// 主程序
var sab = new SharedArrayBuffer(Int32Array.BYTES_PER_ELEMENT * 100000);
var ia = new Int32Array(sab);

for (let i = 0; i < ia.length; i++) {
  ia[i] = primes.next(); // 將質數放入 ia
}

// worker 程序
ia[112]++; // 錯誤
Atomics.add(ia, 112, 1); // 正確
```

上面程式碼中，Worker 程序直接改寫共享記憶體是不正確的。有兩個原因，一是可能發生兩個程序同時改寫該地址，二是改寫以後無法同步到其他 Worker 程序。所以，必須使用`Atomics.add()`方法進行改寫。

下面是另一個例子。

```javascript
// 程序一
console.log(ia[37]);  // 163
Atomics.store(ia, 37, 123456);
Atomics.wake(ia, 37, 1);

// 程序二
Atomics.wait(ia, 37, 163);
console.log(ia[37]);  // 123456
```

上面程式碼中，共享記憶體`ia`的第37號位置，原來的值是`163`。程序二使用`Atomics.wait()`方法，指定只要`ia[37]`等於`163`，就處於“等待”狀態。程序一使用`Atomics.store()`方法，將`123456`放入`ia[37]`，然後使用`Atomics.wake()`方法將監視`ia[37]`的一個程序喚醒。

`Atomics`物件有以下方法。

- `Atomics.load(array, index)`：返回`array[index]`的值。
- `Atomics.store(array, index, value)`：設定`array[index]`的值，返回這個值。
- `Atomics.compareExchange(array, index, oldval, newval)`：如果`array[index]`等於`oldval`，就寫入`newval`，返回`oldval`。
- `Atomics.exchange(array, index, value)`：設定`array[index]`的值，返回舊的值。
- `Atomics.add(array, index, value)`：將`value`加到`array[index]`，返回`array[index]`舊的值。
- `Atomics.sub(array, index, value)`：將`value`從`array[index]`減去，返回`array[index]`舊的值。
- `Atomics.and(array, index, value)`：將`value`與`array[index]`進行位運算`and`，放入`array[index]`，並返回舊的值。
- `Atomics.or(array, index, value)`：將`value`與`array[index]`進行位運算`or`，放入`array[index]`，並返回舊的值。
- `Atomics.xor(array, index, value)`：將`vaule`與`array[index]`進行位運算`xor`，放入`array[index]`，並返回舊的值。
- `Atomics.wait(array, index, value, timeout)`：如果`array[index]`等於`value`，程序就進入休眠狀態，必須通過`Atomics.wake()`喚醒。`timeout`指定多少毫秒之後，進入休眠。返回值是三個字串（ok、not-equal、timed-out）中的一個。
- `Atomics.wake(array, index, count)`：喚醒指定數目在某個位置休眠的程序。
- `Atomics.isLockFree(size)`：返回一個布林值，表示`Atomics`物件是否可以處理某個`size`的記憶體鎖定。如果返回`false`，應用程式就需要自己來實現鎖定。

