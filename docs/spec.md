# 讀懂 ECMAScript 規格

## 概述

規格檔案是計算機語言的官方標準，詳細描述語法規則和實現方法。

一般來說，沒有必要閱讀規格，除非你要寫編譯器。因為規格寫得非常抽象和精煉，又缺乏例項，不容易理解，而且對於解決實際的應用問題，幫助不大。但是，如果你遇到疑難的語法問題，實在找不到答案，這時可以去檢視規格檔案，瞭解語言標準是怎麼說的。規格是解決問題的“最後一招”。

這對JavaScript語言很有必要。因為它的使用場景複雜，語法規則不統一，例外很多，各種執行環境的行為不一致，導致奇怪的語法問題層出不窮，任何語法書都不可能囊括所有情況。檢視規格，不失為一種解決語法問題的最可靠、最權威的終極方法。

本章介紹如何讀懂ECMAScript 6的規格檔案。

ECMAScript 6的規格，可以在ECMA國際標準組織的官方網站（[www.ecma-international.org/ecma-262/6.0/](http://www.ecma-international.org/ecma-262/6.0/)）免費下載和線上閱讀。

這個規格檔案相當龐大，一共有26章，A4列印的話，足足有545頁。它的特點就是規定得非常細緻，每一個語法行為、每一個函式的實現都做了詳盡的清晰的描述。基本上，編譯器作者只要把每一步翻譯成程式碼就可以了。這很大程度上，保證了所有ES6實現都有一致的行為。

ECMAScript 6規格的26章之中，第1章到第3章是對檔案本身的介紹，與語言關係不大。第4章是對這門語言總體設計的描述，有興趣的讀者可以讀一下。第5章到第8章是語言巨集觀層面的描述。第5章是規格的名詞解釋和寫法的介紹，第6章介紹資料型別，第7章介紹語言內部用到的抽象操作，第8章介紹程式碼如何執行。第9章到第26章介紹具體的語法。

對於一般使用者來說，除了第4章，其他章節都涉及某一方面的細節，不用通讀，只要在用到的時候，查閱相關章節即可。下面通過一些例子，介紹如何使用這份規格。

## 相等運算子

相等運算子（`==`）是一個很讓人頭痛的運算子，它的語法行為多變，不符合直覺。這個小節就看看規格怎麼規定它的行為。

請看下面這個表示式，請問它的值是多少。

```javascript
0 == null
```

如果你不確定答案，或者想知道語言內部怎麼處理，就可以去檢視規格，[7.2.12小節](http://www.ecma-international.org/ecma-262/6.0/#sec-7.2.12)是對相等運算子（`==`）的描述。

規格對每一種語法行為的描述，都分成兩部分：先是總體的行為描述，然後是實現的演算法細節。相等運算子的總體描述，只有一句話。

> “The comparison `x == y`, where `x` and `y` are values, produces `true` or `false`.”

上面這句話的意思是，相等運算子用於比較兩個值，返回`true`或`false`。

下面是演算法細節。

> 1. ReturnIfAbrupt(x).
> 1. ReturnIfAbrupt(y).
> 1. If `Type(x)` is the same as `Type(y)`, then  
>   Return the result of performing Strict Equality Comparison `x === y`.
> 1. If `x` is `null` and `y` is `undefined`, return `true`.
> 1. If `x` is `undefined` and `y` is `null`, return `true`.
> 1. If `Type(x)` is Number and `Type(y)` is String,  
>  return the result of the comparison `x == ToNumber(y)`.
> 1. If `Type(x)` is String and `Type(y)` is Number,  
>  return the result of the comparison `ToNumber(x) == y`.
> 1. If `Type(x)` is Boolean, return the result of the comparison `ToNumber(x) == y`.
> 1. If `Type(y)` is Boolean, return the result of the comparison `x == ToNumber(y)`.
> 1. If `Type(x)` is either String, Number, or Symbol and `Type(y)` is Object, then  
>  return the result of the comparison `x == ToPrimitive(y)`.
> 1. If `Type(x)` is Object and `Type(y)` is either String, Number, or Symbol, then  
>  return the result of the comparison `ToPrimitive(x) == y`.
> 1. Return `false`.

上面這段演算法，一共有12步，翻譯如下。

> 1. 如果`x`不是正常值（比如丟擲一個錯誤），中斷執行。
> 1. 如果`y`不是正常值，中斷執行。
> 1. 如果`Type(x)`與`Type(y)`相同，執行嚴格相等運算`x === y`。
> 1. 如果`x`是`null`，`y`是`undefined`，返回`true`。
> 1. 如果`x`是`undefined`，`y`是`null`，返回`true`。
> 1. 如果`Type(x)`是數值，`Type(y)`是字串，返回`x == ToNumber(y)`的結果。
> 1. 如果`Type(x)`是字串，`Type(y)`是數值，返回`ToNumber(x) == y`的結果。
> 1. 如果`Type(x)`是布林值，返回`ToNumber(x) == y`的結果。
> 1. 如果`Type(y)`是布林值，返回`x == ToNumber(y)`的結果。
> 1. 如果`Type(x)`是字串或數值或`Symbol`值，`Type(y)`是物件，返回`x == ToPrimitive(y)`的結果。
> 1. 如果`Type(x)`是物件，`Type(y)`是字串或數值或`Symbol`值，返回`ToPrimitive(x) == y`的結果。
> 1. 返回`false`。

由於`0`的型別是數值，`null`的型別是Null（這是規格[4.3.13小節](http://www.ecma-international.org/ecma-262/6.0/#sec-4.3.13)的規定，是內部Type運算的結果，跟`typeof`運算子無關）。因此上面的前11步都得不到結果，要到第12步才能得到`false`。

```javascript
0 == null // false
```

## 陣列的空位

下面再看另一個例子。

```javascript
const a1 = [undefined, undefined, undefined];
const a2 = [, , ,];

a1.length // 3
a2.length // 3

a1[0] // undefined
a2[0] // undefined

a1[0] === a2[0] // true
```

上面程式碼中，陣列`a1`的成員是三個`undefined`，陣列`a2`的成員是三個空位。這兩個陣列很相似，長度都是3，每個位置的成員讀取出來都是`undefined`。

但是，它們實際上存在重大差異。

```javascript
0 in a1 // true
0 in a2 // false

a1.hasOwnProperty(0) // true
a2.hasOwnProperty(0) // false

Object.keys(a1) // ["0", "1", "2"]
Object.keys(a2) // []

a1.map(n => 1) // [1, 1, 1]
a2.map(n => 1) // [, , ,]
```

上面程式碼一共列出了四種運算，陣列`a1`和`a2`的結果都不一樣。前三種運算（`in`運算子、陣列的`hasOwnProperty`方法、`Object.keys`方法）都說明，陣列`a2`取不到屬性名。最後一種運算（陣列的`map`方法）說明，陣列`a2`沒有發生遍歷。

為什麼`a1`與`a2`成員的行為不一致？陣列的成員是`undefined`或空位，到底有什麼不同？

規格的[12.2.5小節《陣列的初始化》](http://www.ecma-international.org/ecma-262/6.0/#sec-12.2.5)給出了答案。

> “Array elements may be elided at the beginning, middle or end of the element list. Whenever a comma in the element list is not preceded by an AssignmentExpression (i.e., a comma at the beginning or after another comma), the missing array element contributes to the length of the Array and increases the index of subsequent elements. Elided array elements are not defined. If an element is elided at the end of an array, that element does not contribute to the length of the Array.”

翻譯如下。

> "陣列成員可以省略。只要逗號前面沒有任何表示式，陣列的`length`屬性就會加1，並且相應增加其後成員的位置索引。被省略的成員不會被定義。如果被省略的成員是陣列最後一個成員，則不會導致陣列`length`屬性增加。”

上面的規格說得很清楚，陣列的空位會反映在`length`屬性，也就是說空位有自己的位置，但是這個位置的值是未定義，即這個值是不存在的。如果一定要讀取，結果就是`undefined`（因為`undefined`在JavaScript語言中表示不存在）。

這就解釋了為什麼`in`運算子、陣列的`hasOwnProperty`方法、`Object.keys`方法，都取不到空位的屬性名。因為這個屬性名根本就不存在，規格里面沒說要為空位分配屬性名(位置索引），只說要為下一個元素的位置索引加1。

至於為什麼陣列的`map`方法會跳過空位，請看下一節。

## 陣列的map方法

規格的[22.1.3.15小節](http://www.ecma-international.org/ecma-262/6.0/#sec-22.1.3.15)定義了陣列的`map`方法。該小節先是總體描述`map`方法的行為，裡面沒有提到陣列空位。

後面的演算法描述是這樣的。

> 1. Let `O` be `ToObject(this value)`.
> 1. `ReturnIfAbrupt(O)`.
> 1. Let `len` be `ToLength(Get(O, "length"))`.
> 1. `ReturnIfAbrupt(len)`.
> 1. If `IsCallable(callbackfn)` is `false`, throw a TypeError exception.
> 1. If `thisArg` was supplied, let `T` be `thisArg`; else let `T` be `undefined`.
> 1. Let `A` be `ArraySpeciesCreate(O, len)`.
> 1. `ReturnIfAbrupt(A)`.
> 1. Let `k` be 0.
> 1. Repeat, while `k` < `len`  
>     a. Let `Pk` be `ToString(k)`.  
>     b. Let `kPresent` be `HasProperty(O, Pk)`.  
>     c. `ReturnIfAbrupt(kPresent)`.  
>     d. If `kPresent` is `true`, then  
>     d-1. Let `kValue` be `Get(O, Pk)`.  
>     d-2. `ReturnIfAbrupt(kValue)`.  
>     d-3. Let `mappedValue` be `Call(callbackfn, T, «kValue, k, O»)`.  
>     d-4. `ReturnIfAbrupt(mappedValue)`.  
>     d-5. Let `status` be `CreateDataPropertyOrThrow (A, Pk, mappedValue)`.  
>     d-6. `ReturnIfAbrupt(status)`.  
>     e. Increase `k` by 1.
> 1. Return `A`.

翻譯如下。

> 1. 得到當前陣列的`this`物件
> 1. 如果報錯就返回
> 1. 求出當前陣列的`length`屬性
> 1. 如果報錯就返回
> 1. 如果map方法的引數`callbackfn`不可執行，就報錯
> 1. 如果map方法的引數之中，指定了`this`，就讓`T`等於該引數，否則`T`為`undefined`
> 1. 生成一個新的陣列`A`，跟當前陣列的`length`屬性保持一致
> 1. 如果報錯就返回
> 1. 設定`k`等於0
> 1. 只要`k`小於當前陣列的`length`屬性，就重複下面步驟  
>     a. 設定`Pk`等於`ToString(k)`，即將`K`轉為字串  
>     b. 設定`kPresent`等於`HasProperty(O, Pk)`，即求當前陣列有沒有指定屬性  
>     c. 如果報錯就返回  
>     d. 如果`kPresent`等於`true`，則進行下面步驟  
>     d-1. 設定`kValue`等於`Get(O, Pk)`，取出當前陣列的指定屬性  
>     d-2. 如果報錯就返回  
>     d-3. 設定`mappedValue`等於`Call(callbackfn, T, «kValue, k, O»)`，即執行回撥函式  
>     d-4. 如果報錯就返回  
>     d-5. 設定`status`等於`CreateDataPropertyOrThrow (A, Pk, mappedValue)`，即將回調函式的值放入`A`陣列的指定位置  
>     d-6. 如果報錯就返回  
>     e. `k`增加1
> 1. 返回`A`

仔細檢視上面的演算法，可以發現，當處理一個全是空位的陣列時，前面步驟都沒有問題。進入第10步的b時，`kpresent`會報錯，因為空位對應的屬性名，對於陣列來說是不存在的，因此就會返回，不會進行後面的步驟。

```javascript
const arr = [, , ,];
arr.map(n => {
  console.log(n);
  return 1;
}) // [, , ,]
```

上面程式碼中，`arr`是一個全是空位的陣列，`map`方法遍歷成員時，發現是空位，就直接跳過，不會進入回撥函式。因此，回撥函式裡面的`console.log`語句根本不會執行，整個`map`方法返回一個全是空位的新陣列。

V8引擎對`map`方法的[實現](https://github.com/v8/v8/blob/44c44521ae11859478b42004f57ea93df52526ee/src/js/array.js#L1347)如下，可以看到跟規格的演算法描述完全一致。

```javascript
function ArrayMap(f, receiver) {
  CHECK_OBJECT_COERCIBLE(this, "Array.prototype.map");

  // Pull out the length so that modifications to the length in the
  // loop will not affect the looping and side effects are visible.
  var array = TO_OBJECT(this);
  var length = TO_LENGTH_OR_UINT32(array.length);
  return InnerArrayMap(f, receiver, array, length);
}

function InnerArrayMap(f, receiver, array, length) {
  if (!IS_CALLABLE(f)) throw MakeTypeError(kCalledNonCallable, f);

  var accumulator = new InternalArray(length);
  var is_array = IS_ARRAY(array);
  var stepping = DEBUG_IS_STEPPING(f);
  for (var i = 0; i < length; i++) {
    if (HAS_INDEX(array, i, is_array)) {
      var element = array[i];
      // Prepare break slots for debugger step in.
      if (stepping) %DebugPrepareStepInIfStepping(f);
      accumulator[i] = %_Call(f, receiver, element, i, array);
    }
  }
  var result = new GlobalArray();
  %MoveArrayContents(accumulator, result);
  return result;
}
```
