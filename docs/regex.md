# 正則的擴展

## RegExp建構函式

在ES5中，RegExp建構函式的引數有兩種情況。

第一種情況是，引數是字串，這時第二個引數表示正則表示式的修飾符（flag）。

```javascript
var regex = new RegExp('xyz', 'i');
// 等價於
var regex = /xyz/i;
```

第二種情況是，引數是一個正則表示式，這時會返回一個原有正則表示式的拷貝。

```javascript
var regex = new RegExp(/xyz/i);
// 等價於
var regex = /xyz/i;
```

但是，ES5不允許此時使用第二個引數，新增修飾符，否則會報錯。

```javascript
var regex = new RegExp(/xyz/, 'i');
// Uncaught TypeError: Cannot supply flags when constructing one RegExp from another
```

ES6改變了這種行為。如果RegExp建構函式第一個引數是一個正則物件，那麼可以使用第二個引數指定修飾符。而且，返回的正則表示式會忽略原有的正則表示式的修飾符，只使用新指定的修飾符。

```javascript
new RegExp(/abc/ig, 'i').flags
// "i"
```

上面程式碼中，原有正則物件的修飾符是`ig`，它會被第二個引數`i`覆蓋。

## 字串的正則方法

字串物件共有4個方法，可以使用正則表示式：`match()`、`replace()`、`search()`和`split()`。

ES6將這4個方法，在語言內部全部呼叫RegExp的實例方法，從而做到所有與正則相關的方法，全都定義在RegExp物件上。

- `String.prototype.match` 呼叫 `RegExp.prototype[Symbol.match]`
- `String.prototype.replace` 呼叫 `RegExp.prototype[Symbol.replace]`
- `String.prototype.search` 呼叫 `RegExp.prototype[Symbol.search]`
- `String.prototype.split` 呼叫 `RegExp.prototype[Symbol.split]`

## u修飾符

ES6對正則表示式添加了`u`修飾符，含義為“Unicode模式”，用來正確處理大於`\uFFFF`的Unicode字元。也就是說，會正確處理四個位元組的UTF-16編碼。

```javascript
/^\uD83D/u.test('\uD83D\uDC2A')
// false
/^\uD83D/.test('\uD83D\uDC2A')
// true
```

上面程式碼中，`\uD83D\uDC2A`是一個四個位元組的UTF-16編碼，代表一個字元。但是，ES5不支援四個位元組的UTF-16編碼，會將其識別為兩個字元，導致第二行程式碼結果為`true`。加了`u`修飾符以後，ES6就會識別其為一個字元，所以第一行程式碼結果為`false`。

一旦加上`u`修飾符號，就會修改下面這些正則表示式的行為。

**（1）點字元**

點（`.`）字元在正則表示式中，含義是除了換行符以外的任意單個字元。對於碼點大於`0xFFFF`的Unicode字元，點字元不能識別，必須加上`u`修飾符。

```javascript
var s = '𠮷';

/^.$/.test(s) // false
/^.$/u.test(s) // true
```

上面程式碼表示，如果不新增`u`修飾符，正則表示式就會認為字串為兩個字元，從而匹配失敗。

**（2）Unicode字元表示法**

ES6新增了使用大括號表示Unicode字元，這種表示法在正則表示式中必須加上`u`修飾符，才能識別。

```javascript
/\u{61}/.test('a') // false
/\u{61}/u.test('a') // true
/\u{20BB7}/u.test('𠮷') // true
```

上面程式碼表示，如果不加`u`修飾符，正則表示式無法識別`\u{61}`這種表示法，只會認為這匹配61個連續的`u`。

**（3）量詞**

使用`u`修飾符後，所有量詞都會正確識別碼點大於`0xFFFF`的Unicode字元。

```javascript
/a{2}/.test('aa') // true
/a{2}/u.test('aa') // true
/𠮷{2}/.test('𠮷𠮷') // false
/𠮷{2}/u.test('𠮷𠮷') // true
```

另外，只有在使用`u`修飾符的情況下，Unicode表示式當中的大括號才會被正確解讀，否則會被解讀為量詞。

```javascript
/^\u{3}$/.test('uuu') // true
```

上面程式碼中，由於正則表示式沒有`u`修飾符，所以大括號被解讀為量詞。加上`u`修飾符，就會被解讀為Unicode表示式。

**（4）預定義模式**

`u`修飾符也影響到預定義模式，能否正確識別碼點大於`0xFFFF`的Unicode字元。

```javascript
/^\S$/.test('𠮷') // false
/^\S$/u.test('𠮷') // true
```

上面程式碼的`\S`是預定義模式，匹配所有不是空格的字元。只有加了`u`修飾符，它才能正確匹配碼點大於`0xFFFF`的Unicode字元。

利用這一點，可以寫出一個正確返回字串長度的函式。

```javascript
function codePointLength(text) {
  var result = text.match(/[\s\S]/gu);
  return result ? result.length : 0;
}

var s = '𠮷𠮷';

s.length // 4
codePointLength(s) // 2
```

**（5）i修飾符**

有些Unicode字元的編碼不同，但是字型很相近，比如，`\u004B`與`\u212A`都是大寫的`K`。

```javascript
/[a-z]/i.test('\u212A') // false
/[a-z]/iu.test('\u212A') // true
```

上面程式碼中，不加`u`修飾符，就無法識別非規範的K字元。

## y 修飾符

除了`u`修飾符，ES6還為正則表示式添加了`y`修飾符，叫做“粘連”（sticky）修飾符。

`y`修飾符的作用與`g`修飾符類似，也是全域性匹配，後一次匹配都從上一次匹配成功的下一個位置開始。不同之處在於，`g`修飾符只要剩餘位置中存在匹配就可，而`y`修飾符確保匹配必須從剩餘的第一個位置開始，這也就是“粘連”的涵義。

```javascript
var s = 'aaa_aa_a';
var r1 = /a+/g;
var r2 = /a+/y;

r1.exec(s) // ["aaa"]
r2.exec(s) // ["aaa"]

r1.exec(s) // ["aa"]
r2.exec(s) // null
```

上面程式碼有兩個正則表示式，一個使用`g`修飾符，另一個使用`y`修飾符。這兩個正則表示式各執行了兩次，第一次執行的時候，兩者行為相同，剩餘字串都是`_aa_a`。由於`g`修飾沒有位置要求，所以第二次執行會返回結果，而`y`修飾符要求匹配必須從頭部開始，所以返回`null`。

如果改一下正則表示式，保證每次都能頭部匹配，`y`修飾符就會返回結果了。

```javascript
var s = 'aaa_aa_a';
var r = /a+_/y;

r.exec(s) // ["aaa_"]
r.exec(s) // ["aa_"]
```

上面程式碼每次匹配，都是從剩餘字串的頭部開始。

使用`lastIndex`屬性，可以更好地說明`y`修飾符。

```javascript
const REGEX = /a/g;

// 指定從2號位置（y）開始匹配
REGEX.lastIndex = 2;

// 匹配成功
const match = REGEX.exec('xaya');

// 在3號位置匹配成功
match.index // 3

// 下一次匹配從4號位開始
REGEX.lastIndex // 4

// 4號位開始匹配失敗
REGEX.exec('xaxa') // null
```

上面程式碼中，`lastIndex`屬性指定每次搜尋的開始位置，`g`修飾符從這個位置開始向後搜尋，直到發現匹配為止。

`y`修飾符同樣遵守`lastIndex`屬性，但是要求必須在`lastIndex`指定的位置發現匹配。

```javascript
const REGEX = /a/y;

// 指定從2號位置開始匹配
REGEX.lastIndex = 2;

// 不是粘連，匹配失敗
REGEX.exec('xaya') // null

// 指定從3號位置開始匹配
REGEX.lastIndex = 3;

// 3號位置是粘連，匹配成功
const match = REGEX.exec('xaxa');
match.index // 3
REGEX.lastIndex // 4
```

進一步說，`y`修飾符號隱含了頭部匹配的標誌`^`。

```javascript
/b/y.exec('aba')
// null
```

上面程式碼由於不能保證頭部匹配，所以返回`null`。`y`修飾符的設計本意，就是讓頭部匹配的標誌`^`在全域性匹配中都有效。

在`split`方法中使用`y`修飾符，原字串必須以分隔符開頭。這也意味著，只要匹配成功，陣列的第一個成員肯定是空字串。

```javascript
// 沒有找到匹配
'x##'.split(/#/y)
// [ 'x##' ]

// 找到兩個匹配
'##x'.split(/#/y)
// [ '', '', 'x' ]
```

後續的分隔符只有緊跟前面的分隔符，才會被識別。

```javascript
'#x#'.split(/#/y)
// [ '', 'x#' ]

'##'.split(/#/y)
// [ '', '', '' ]
```

下面是字串物件的`replace`方法的例子。

```javascript
const REGEX = /a/gy;
'aaxa'.replace(REGEX, '-') // '--xa'
```

上面程式碼中，最後一個`a`因為不是出現下一次匹配的頭部，所以不會被替換。

單單一個`y`修飾符對`match`方法，只能返回第一個匹配，必須與`g`修飾符聯用，才能返回所有匹配。

```javascript
'a1a2a3'.match(/a\d/y) // ["a1"]
'a1a2a3'.match(/a\d/gy) // ["a1", "a2", "a3"]
```

`y`修飾符的一個應用，是從字串提取token（詞元），`y`修飾符確保了匹配之間不會有漏掉的字元。

```javascript
const TOKEN_Y = /\s*(\+|[0-9]+)\s*/y;
const TOKEN_G  = /\s*(\+|[0-9]+)\s*/g;

tokenize(TOKEN_Y, '3 + 4')
// [ '3', '+', '4' ]
tokenize(TOKEN_G, '3 + 4')
// [ '3', '+', '4' ]

function tokenize(TOKEN_REGEX, str) {
  let result = [];
  let match;
  while (match = TOKEN_REGEX.exec(str)) {
    result.push(match[1]);
  }
  return result;
}
```

上面程式碼中，如果字串裡面沒有非法字元，`y`修飾符與`g`修飾符的提取結果是一樣的。但是，一旦出現非法字元，兩者的行為就不一樣了。

```javascript
tokenize(TOKEN_Y, '3x + 4')
// [ '3' ]
tokenize(TOKEN_G, '3x + 4')
// [ '3', '+', '4' ]
```

上面程式碼中，`g`修飾符會忽略非法字元，而`y`修飾符不會，這樣就很容易發現錯誤。

## sticky屬性

與`y`修飾符相匹配，ES6的正則物件多了`sticky`屬性，表示是否設定了`y`修飾符。

```javascript
var r = /hello\d/y;
r.sticky // true
```

## flags屬性

ES6為正則表示式新增了`flags`屬性，會返回正則表示式的修飾符。

```javascript
// ES5的source屬性
// 返回正則表示式的正文
/abc/ig.source
// "abc"

// ES6的flags屬性
// 返回正則表示式的修飾符
/abc/ig.flags
// 'gi'
```

## RegExp.escape()

字串必須轉義，才能作為正則模式。

```javascript
function escapeRegExp(str) {
  return str.replace(/[\-\[\]\/\{\}\(\)\*\+\?\.\\\^\$\|]/g, '\\$&');
}

let str = '/path/to/resource.html?search=query';
escapeRegExp(str)
// "\/path\/to\/resource\.html\?search=query"
```

上面程式碼中，`str`是一個正常字串，必須使用反斜槓對其中的特殊字元轉義，才能用來作為一個正則匹配的模式。

已經有[提議](https://esdiscuss.org/topic/regexp-escape)將這個需求標準化，作為RegExp物件的靜態方法[RegExp.escape()](https://github.com/benjamingr/RexExp.escape)，放入ES7。2015年7月31日，TC39認為，這個方法有安全風險，又不願這個方法變得過於複雜，沒有同意將其列入ES7，但這不失為一個真實的需求。

```javascript
RegExp.escape('The Quick Brown Fox');
// "The Quick Brown Fox"

RegExp.escape('Buy it. use it. break it. fix it.');
// "Buy it\. use it\. break it\. fix it\."

RegExp.escape('(*.*)');
// "\(\*\.\*\)"
```

字串轉義以後，可以使用RegExp建構函式生成正則模式。

```javascript
var str = 'hello. how are you?';
var regex = new RegExp(RegExp.escape(str), 'g');
assert.equal(String(regex), '/hello\. how are you\?/g');
```

目前，該方法可以用上文的`escapeRegExp`函式或者墊片模組[regexp.escape](https://github.com/ljharb/regexp.escape)實現。

```javascript
var escape = require('regexp.escape');
escape('hi. how are you?');
// "hi\\. how are you\\?"
```

## s 修飾符：dotAll 模式

正則表示式中，點（`.`）是一個特殊字元，代表任意的單個字元，但是行終止符（line terminator character）除外。

以下四個字元屬於”行終止符“。

- U+000A 換行符（`\n`）
- U+000D 回車符（`\r`）
- U+2028 行分隔符（line separator）
- U+2029 段分隔符（paragraph separator）

```javascript
/foo.bar/.test('foo\nbar')
// false
```

上面程式碼中，因為`.`不匹配`\n`，所以正則表示式返回`false`。

但是，很多時候我們希望匹配的是任意單個字元，這時有一種變通的寫法。

```javascript
/foo[^]bar/.test('foo\nbar')
// true
```

這種解決方案畢竟不太符合直覺，所以現在有一個[提案](https://github.com/mathiasbynens/es-regexp-dotall-flag)，引入`/s`修飾符，使得`.`可以匹配任意單個字元。

```javascript
/foo.bar/s.test('foo\nbar') // true
```

這被稱為`dotAll`模式，即點（dot）代表一切字元。所以，正則表示式還引入了一個`dotAll`屬性，返回一個布林值，表示該正則表示式是否處在`dotAll`模式。

```javascript
const re = /foo.bar/s;
// 另一種寫法
// const re = new RegExp('foo.bar', 's');

re.test('foo\nbar') // true
re.dotAll // true
re.flags // 's'
```

`/s`修飾符和多行修飾符`/m`不衝突，兩者一起使用的情況下，`.`匹配所有字元，而`^`和`$`匹配每一行的行首和行尾。

## 後行斷言

JavaScript 語言的正則表示式，只支援先行斷言（lookahead）和先行否定斷言（negative lookahead），不支援後行斷言（lookbehind）和後行否定斷言（negative lookbehind）。

目前，有一個[提案](https://github.com/goyakin/es-regexp-lookbehind)，引入後行斷言。V8 引擎4.9版已經支援，Chrome 瀏覽器49版開啟”experimental JavaScript features“開關（位址列鍵入`about:flags`），就可以使用這項功能。

”先行斷言“指的是，`x`只有在`y`前面才匹配，必須寫成`/x(?=y)/`。比如，只匹配百分號之前的數字，要寫成`/\d+(?=%)/`。”先行否定斷言“指的是，`x`只有不在`y`前面才匹配，必須寫成`/x(?!y)/`。比如，只匹配不在百分號之前的數字，要寫成`/\d+(?!%)/`。

```javascript
/\d+(?=%)/.exec('100% of US presidents have been male')  // ["100"]
/\d+(?!%)/.exec('that’s all 44 of them')                 // ["44"]
```

上面兩個字串，如果互換正則表示式，就會匹配失敗。另外，還可以看到，”先行斷言“括號之中的部分（`(?=%)`），是不計入返回結果的。

“後行斷言”正好與“先行斷言”相反，`x`只有在`y`後面才匹配，必須寫成`/(?<=y)x/`。比如，只匹配美元符號之後的數字，要寫成`/(?<=\$)\d+/`。”後行否定斷言“則與”先行否定斷言“相反，`x`只有不在`y`後面才匹配，必須寫成`/(?<!y)x/`。比如，只匹配不在美元符號後面的數字，要寫成`/(?<!\$)\d+/`。

```javascript
/(?<=\$)\d+/.exec('Benjamin Franklin is on the $100 bill')  // ["100"]
/(?<!\$)\d+/.exec('it’s is worth about €90')                // ["90"]
```

上面的例子中，“後行斷言”的括號之中的部分（`(?<=\$)`），也是不計入返回結果。

“後行斷言”的實現，需要先匹配`/(?<=y)x/`的`x`，然後再回到左邊，匹配`y`的部分。這種“先右後左”的執行順序，與所有其他正則操作相反，導致了一些不符合預期的行為。

首先，”後行斷言“的組匹配，與正常情況下結果是不一樣的。

```javascript
/(?<=(\d+)(\d+))$/.exec('1053') // ["", "1", "053"]
/^(\d+)(\d+)$/.exec('1053') // ["1053", "105", "3"]
```

上面程式碼中，需要捕捉兩個組匹配。沒有"後行斷言"時，第一個括號是貪婪模式，第二個括號只能捕獲一個字元，所以結果是`105`和`3`。而"後行斷言"時，由於執行順序是從右到左，第二個括號是貪婪模式，第一個括號只能捕獲一個字元，所以結果是`1`和`053`。

其次，"後行斷言"的反斜槓引用，也與通常的順序相反，必須放在對應的那個括號之前。

```javascript
/(?<=(o)d\1)r/.exec('hodor')  // null
/(?<=\1d(o))r/.exec('hodor')  // ["r", "o"]
```

上面程式碼中，如果後行斷言的反斜槓引用（`\1`）放在括號的後面，就不會得到匹配結果，必須放在前面才可以。

## Unicode屬性類

目前，有一個[提案](https://github.com/mathiasbynens/es-regexp-unicode-property-escapes)，引入了一種新的類的寫法`\p{...}`和`\P{...}`，允許正則表示式匹配符合Unicode某種屬性的所有字元。

```javascript
const regexGreekSymbol = /\p{Script=Greek}/u;
regexGreekSymbol.test('π') // u
```

上面程式碼中，`\p{Script=Greek}`指定匹配一個希臘文字母，所以匹配`π`成功。

Unicode屬性類要指定屬性名和屬性值。

```javascript
\p{UnicodePropertyName=UnicodePropertyValue}
```

對於某些屬性，可以只寫屬性名。

```javascript
\p{UnicodePropertyName}
```

`\P{…}`是`\p{…}`的反向匹配，即匹配不滿足條件的字元。

注意，這兩種類只對Unicode有效，所以使用的時候一定要加上`u`修飾符。如果不加`u`修飾符，正則表示式使用`\p`和`\P`會報錯，ECMAScript預留了這兩個類。

由於Unicode的各種屬性非常多，所以這種新的類的表達能力非常強。

```javascript
const regex = /^\p{Decimal_Number}+$/u;
regex.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼') // true
```

上面程式碼中，屬性類指定匹配所有十進位制字元，可以看到各種字型的十進位制字元都會匹配成功。

`\p{Number}`甚至能匹配羅馬數字。

```javascript
// 匹配所有數字
const regex = /^\p{Number}+$/u;
regex.test('²³¹¼½¾') // true
regex.test('㉛㉜㉝') // true
regex.test('ⅠⅡⅢⅣⅤⅥⅦⅧⅨⅩⅪⅫ') // true
```

下面是其他一些例子。

```javascript
// 匹配各種文字的所有字母，等同於Unicode版的\w
[\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]

// 匹配各種文字的所有非字母的字元，等同於Unicode版的\W
[^\p{Alphabetic}\p{Mark}\p{Decimal_Number}\p{Connector_Punctuation}\p{Join_Control}]

// 匹配所有的箭頭字元
const regexArrows = /^\p{Block=Arrows}+$/u;
regexArrows.test('←↑→↓↔↕↖↗↘↙⇏⇐⇑⇒⇓⇔⇕⇖⇗⇘⇙⇧⇩') // true
```

