# ECMAScript 6簡介

ECMAScript 6.0（以下簡稱 ES6）是 JavaScript 語言的下一代標準，已經在2015年6月正式釋出了。它的目標，是使得 JavaScript 語言可以用來編寫複雜的大型應用程式，成為企業級開發語言。

## ECMAScript 和 JavaScript 的關係

一個常見的問題是，ECMAScript 和 JavaScript 到底是什麼關係？

要講清楚這個問題，需要回顧歷史。1996年11月，JavaScript 的創造者 Netscape 公司，決定將 JavaScript 提交給國際標準化組織ECMA，希望這種語言能夠成為國際標準。次年，ECMA 釋出262號標準檔案（ECMA-262）的第一版，規定了瀏覽器指令碼語言的標準，並將這種語言稱為 ECMAScript，這個版本就是1.0版。

該標準從一開始就是針對 JavaScript 語言制定的，但是之所以不叫 JavaScript，有兩個原因。一是商標，Java 是 Sun 公司的商標，根據授權協議，只有 Netscape 公司可以合法地使用 JavaScript 這個名字，且 JavaScript 本身也已經被 Netscape 公司註冊為商標。二是想體現這門語言的制定者是 ECMA，不是 Netscape，這樣有利於保證這門語言的開放性和中立性。

因此，ECMAScript 和 JavaScript 的關係是，前者是後者的規格，後者是前者的一種實現（另外的 ECMAScript 方言還有 Jscript 和 ActionScript）。日常場合，這兩個詞是可以互換的。

## ES6 與 ECMAScript 2015 的關係

ECMAScript 2015（簡稱 ES2015）這個詞，也是經常可以看到的。它與 ES6 是什麼關係呢？

2011年，ECMAScript 5.1版釋出後，就開始制定6.0版了。因此，ES6 這個詞的原意，就是指 JavaScript 語言的下一個版本。

但是，因為這個版本引入的語法功能太多，而且制定過程當中，還有很多組織和個人不斷提交新功能。事情很快就變得清楚了，不可能在一個版本里麵包括所有將要引入的功能。常規的做法是先發布6.0版，過一段時間再發6.1版，然後是6.2版、6.3版等等。

但是，標準的制定者不想這樣做。他們想讓標準的升級成為常規流程：任何人在任何時候，都可以向標準委員會提交新語法的提案，然後標準委員會每個月開一次會，評估這些提案是否可以接受，需要哪些改進。如果經過多次會議以後，一個提案足夠成熟了，就可以正式進入標準了。這就是說，標準的版本升級成為了一個不斷滾動的流程，每個月都會有變動。

標準委員會最終決定，標準在每年的6月份正式釋出一次，作為當年的正式版本。接下來的時間，就在這個版本的基礎上做改動，直到下一年的6月份，草案就自然變成了新一年的版本。這樣一來，就不需要以前的版本號了，只要用年份標記就可以了。

ES6 的第一個版本，就這樣在2015年6月釋出了，正式名稱就是《ECMAScript 2015標準》（簡稱 ES2015）。2016年6月，小幅修訂的《ECMAScript 2016標準》（簡稱 ES2016）如期釋出，這個版本可以看作是 ES6.1 版，因為兩者的差異非常小（只新增了陣列例項的`includes`方法和指數運算子），基本上是同一個標準。根據計劃，2017年6月釋出 ES2017 標準。

因此，ES6 既是一個歷史名詞，也是一個泛指，含義是5.1版以後的 JavaScript 的下一代標準，涵蓋了ES2015、ES2016、ES2017等等，而ES2015 則是正式名稱，特指該年釋出的正式版本的語言標準。本書中提到 ES6 的地方，一般是指 ES2015 標準，但有時也是泛指“下一代 JavaScript 語言”。

## 語法提案的批准流程

任何人都可以向標準委員會（又稱 TC39 委員會）提案，要求修改語言標準。

一種新的語法從提案到變成正式標準，需要經歷五個階段。每個階段的變動都需要由 TC39 委員會批准。

- Stage 0 - Strawman（展示階段）
- Stage 1 - Proposal（徵求意見階段）
- Stage 2 - Draft（草案階段）
- Stage 3 - Candidate（候選人階段）
- Stage 4 - Finished（定案階段）

一個提案只要能進入 Stage 2，就差不多肯定會包括在以後的正式標準裡面。ECMAScript 當前的所有提案，可以在 TC39 的官方網站[Github.com/tc39/ecma262](https://github.com/tc39/ecma262)檢視。

本書的寫作目標之一，是跟蹤 ECMAScript 語言的最新進展，介紹5.1版本以後所有的新語法。對於那些明確或很有希望，將要列入標準的新語法，都將予以介紹。

## ECMAScript 的歷史

ES6 從開始制定到最後釋出，整整用了15年。

前面提到，ECMAScript 1.0 是1997年釋出的，接下來的兩年，連續釋出了 ECMAScript 2.0（1998年6月）和 ECMAScript 3.0（1999年12月）。3.0版是一個巨大的成功，在業界得到廣泛支援，成為通行標準，奠定了 JavaScript 語言的基本語法，以後的版本完全繼承。直到今天，初學者一開始學習 JavaScript，其實就是在學3.0版的語法。

2000年，ECMAScript 4.0 開始醞釀。這個版本最後沒有通過，但是它的大部分內容被 ES6 繼承了。因此，ES6 制定的起點其實是2000年。

為什麼 ES4 沒有通過呢？因為這個版本太激進了，對 ES3 做了徹底升級，導致標準委員會的一些成員不願意接受。ECMA 的第39號技術專家委員會（Technical Committee 39，簡稱TC39）負責制訂 ECMAScript 標準，成員包括 Microsoft、Mozilla、Google 等大公司。

2007年10月，ECMAScript 4.0 版草案發布，本來預計次年8月釋出正式版本。但是，各方對於是否通過這個標準，發生了嚴重分歧。以 Yahoo、Microsoft、Google 為首的大公司，反對 JavaScript 的大幅升級，主張小幅改動；以 JavaScript 創造者Brendan Eich為首的Mozilla公司，則堅持當前的草案。

2008年7月，由於對於下一個版本應該包括哪些功能，各方分歧太大，爭論過於激烈，ECMA 開會決定，中止 ECMAScript 4.0 的開發，將其中涉及現有功能改善的一小部分，釋出為 ECMAScript 3.1，而將其他激進的設想擴大範圍，放入以後的版本，由於會議的氣氛，該版本的專案代號起名為 Harmony（和諧）。會後不久，ECMAScript 3.1 就改名為 ECMAScript 5。

2009年12月，ECMAScript 5.0 版正式釋出。Harmony 專案則一分為二，一些較為可行的設想定名為 JavaScript.next 繼續開發，後來演變成 ECMAScript 6；一些不是很成熟的設想，則被視為 JavaScript.next.next，在更遠的將來再考慮推出。TC39 委員會的總體考慮是，ES5 與 ES3 基本保持相容，較大的語法修正和新功能加入，將由 JavaScript.next 完成。當時，JavaScript.next 指的是ES6，第六版釋出以後，就指 ES7。TC39 的判斷是，ES5 會在2013年的年中成為 JavaScript 開發的主流標準，並在此後五年中一直保持這個位置。

2011年6月，ECMAscript 5.1 版釋出，並且成為 ISO 國際標準（ISO/IEC 16262:2011）。

2013年3月，ECMAScript 6 草案凍結，不再新增新功能。新的功能設想將被放到 ECMAScript 7。

2013年12月，ECMAScript 6 草案發布。然後是12個月的討論期，聽取各方反饋。

2015年6月，ECMAScript 6 正式通過，成為國際標準。從2000年算起，這時已經過去了15年。

## 部署進度

各大瀏覽器的最新版本，對 ES6 的支援可以檢視[kangax.github.io/es5-compat-table/es6/](http://kangax.github.io/es5-compat-table/es6/)。隨著時間的推移，支援度已經越來越高了，超過90%的 ES6 語法特性都實現了。

Node 是 JavaScript 的伺服器執行環境（runtime）。它對 ES6 的支援度更高。除了那些預設開啟的功能，還有一些語法功能已經實現了，但是預設沒有開啟。使用下面的命令，可以檢視 Node 已經實現的 ES6 特性。

```bash
$ node --v8-options | grep harmony
```

上面命令的輸出結果，會因為版本的不同而有所不同。

我寫了一個工具 [ES-Checker](https://github.com/ruanyf/es-checker)，用來檢查各種執行環境對 ES6 的支援情況。訪問[ruanyf.github.io/es-checker](http://ruanyf.github.io/es-checker)，可以看到您的瀏覽器支援 ES6 的程度。執行下面的命令，可以檢視你正在使用的 Node 環境對 ES6 的支援程度。

```bash
$ npm install -g es-checker
$ es-checker

=========================================
Passes 24 feature Dectations
Your runtime supports 57% of ECMAScript 6
=========================================
```

## Babel 轉碼器

[Babel](https://babeljs.io/) 是一個廣泛使用的 ES6 轉碼器，可以將 ES6 程式碼轉為 ES5 程式碼，從而在現有環境執行。這意味著，你可以用 ES6 的方式編寫程式，又不用擔心現有環境是否支援。下面是一個例子。

```javascript
// 轉碼前
input.map(item => item + 1);

// 轉碼後
input.map(function (item) {
  return item + 1;
});
```

上面的原始程式碼用了箭頭函式，Babel 將其轉為普通函式，就能在不支援箭頭函式的 JavaScript 環境執行了。

### 配置檔案`.babelrc`

Babel 的配置檔案是`.babelrc`，存放在專案的根目錄下。使用 Babel 的第一步，就是配置這個檔案。

該檔案用來設定轉碼規則和外掛，基本格式如下。

```javascript
{
  "presets": [],
  "plugins": []
}
```

`presets`欄位設定轉碼規則，官方提供以下的規則集，你可以根據需要安裝。

```bash
# 最新轉碼規則
$ npm install --save-dev babel-preset-latest

# react 轉碼規則
$ npm install --save-dev babel-preset-react

# 不同階段語法提案的轉碼規則（共有4個階段），選裝一個
$ npm install --save-dev babel-preset-stage-0
$ npm install --save-dev babel-preset-stage-1
$ npm install --save-dev babel-preset-stage-2
$ npm install --save-dev babel-preset-stage-3
```

然後，將這些規則加入`.babelrc`。

```javascript
  {
    "presets": [
      "latest",
      "react",
      "stage-2"
    ],
    "plugins": []
  }
```

注意，以下所有 Babe l工具和模組的使用，都必須先寫好`.babelrc`。

### 命令列轉碼`babel-cli`

Babel提供`babel-cli`工具，用於命令列轉碼。

它的安裝命令如下。

```bash
$ npm install --global babel-cli
```

基本用法如下。

```bash
# 轉碼結果輸出到標準輸出
$ babel example.js

# 轉碼結果寫入一個檔案
# --out-file 或 -o 引數指定輸出檔案
$ babel example.js --out-file compiled.js
# 或者
$ babel example.js -o compiled.js

# 整個目錄轉碼
# --out-dir 或 -d 引數指定輸出目錄
$ babel src --out-dir lib
# 或者
$ babel src -d lib

# -s 引數生成source map檔案
$ babel src -d lib -s
```

上面程式碼是在全域性環境下，進行 Babel 轉碼。這意味著，如果專案要執行，全域性環境必須有 Babel，也就是說專案產生了對環境的依賴。另一方面，這樣做也無法支援不同專案使用不同版本的 Babel。

一個解決辦法是將`babel-cli`安裝在專案之中。

```bash
# 安裝
$ npm install --save-dev babel-cli
```

然後，改寫`package.json`。

```javascript
{
  // ...
  "devDependencies": {
    "babel-cli": "^6.0.0"
  },
  "scripts": {
    "build": "babel src -d lib"
  },
}
```

轉碼的時候，就執行下面的命令。

```javascript
$ npm run build
```

### babel-node

`babel-cli`工具自帶一個`babel-node`命令，提供一個支援ES6的REPL環境。它支援Node的REPL環境的所有功能，而且可以直接執行ES6程式碼。

它不用單獨安裝，而是隨`babel-cli`一起安裝。然後，執行`babel-node`就進入REPL環境。

```bash
$ babel-node
> (x => x * 2)(1)
2
```

`babel-node`命令可以直接執行ES6指令碼。將上面的程式碼放入指令碼檔案`es6.js`，然後直接執行。

```bash
$ babel-node es6.js
2
```

`babel-node`也可以安裝在專案中。

```bash
$ npm install --save-dev babel-cli
```

然後，改寫`package.json`。

```javascript
{
  "scripts": {
    "script-name": "babel-node script.js"
  }
}
```

上面程式碼中，使用`babel-node`替代`node`，這樣`script.js`本身就不用做任何轉碼處理。

### babel-register

`babel-register`模組改寫`require`命令，為它加上一個鉤子。此後，每當使用`require`載入`.js`、`.jsx`、`.es`和`.es6`字尾名的檔案，就會先用Babel進行轉碼。

```bash
$ npm install --save-dev babel-register
```

使用時，必須首先載入`babel-register`。

```bash
require("babel-register");
require("./index.js");
```

然後，就不需要手動對`index.js`轉碼了。

需要注意的是，`babel-register`只會對`require`命令載入的檔案轉碼，而不會對當前檔案轉碼。另外，由於它是實時轉碼，所以只適合在開發環境使用。

### babel-core

如果某些程式碼需要呼叫 Babel 的 API 進行轉碼，就要使用`babel-core`模組。

安裝命令如下。

```bash
$ npm install babel-core --save
```

然後，在專案中就可以呼叫`babel-core`。

```javascript
var babel = require('babel-core');

// 字串轉碼
babel.transform('code();', options);
// => { code, map, ast }

// 檔案轉碼（非同步）
babel.transformFile('filename.js', options, function(err, result) {
  result; // => { code, map, ast }
});

// 檔案轉碼（同步）
babel.transformFileSync('filename.js', options);
// => { code, map, ast }

// Babel AST轉碼
babel.transformFromAst(ast, code, options);
// => { code, map, ast }
```

配置物件`options`，可以參看官方文件[http://babeljs.io/docs/usage/options/](http://babeljs.io/docs/usage/options/)。

下面是一個例子。

```javascript
var es6Code = 'let x = n => n + 1';
var es5Code = require('babel-core')
  .transform(es6Code, {
    presets: ['latest']
  })
  .code;
// '"use strict";\n\nvar x = function x(n) {\n  return n + 1;\n};'
```

上面程式碼中，`transform`方法的第一個引數是一個字串，表示需要被轉換的ES6程式碼，第二個引數是轉換的配置物件。

### babel-polyfill

Babel 預設只轉換新的 JavaScript 句法（syntax），而不轉換新的 API，比如`Iterator`、`Generator`、`Set`、`Maps`、`Proxy`、`Reflect`、`Symbol`、`Promise`等全域性物件，以及一些定義在全域性物件上的方法（比如`Object.assign`）都不會轉碼。

舉例來說，ES6在`Array`物件上新增了`Array.from`方法。Babel 就不會轉碼這個方法。如果想讓這個方法執行，必須使用`babel-polyfill`，為當前環境提供一個墊片。

安裝命令如下。

```bash
$ npm install --save babel-polyfill
```

然後，在指令碼頭部，加入如下一行程式碼。

```javascript
import 'babel-polyfill';
// 或者
require('babel-polyfill');
```

Babel 預設不轉碼的 API 非常多，詳細清單可以檢視`babel-plugin-transform-runtime`模組的[definitions.js](https://github.com/babel/babel/blob/master/packages/babel-plugin-transform-runtime/src/definitions.js)檔案。

### 瀏覽器環境

Babel 也可以用於瀏覽器環境。但是，從 Babel 6.0 開始，不再直接提供瀏覽器版本，而是要用構建工具構建出來。如果你沒有或不想使用構建工具，可以使用[babel-standalone](https://github.com/Daniel15/babel-standalone)模組提供的瀏覽器版本，將其插入網頁。

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.4.4/babel.min.js"></script>
<script type="text/babel">
// Your ES6 code
</script>
```

注意，網頁實時將 ES6 程式碼轉為 ES5，對效能會有影響。生產環境需要載入已經轉碼完成的指令碼。

下面是如何將程式碼打包成瀏覽器可以使用的指令碼，以`Babel`配合`Browserify`為例。首先，安裝`babelify`模組。

```bash
$ npm install --save-dev babelify babel-preset-latest
```

然後，再用命令列轉換 ES6 指令碼。

```bash
$  browserify script.js -o bundle.js \
  -t [ babelify --presets [ latest ] ]
```

上面程式碼將ES6指令碼`script.js`，轉為`bundle.js`，瀏覽器直接載入後者就可以了。

在`package.json`設定下面的程式碼，就不用每次命令列都輸入引數了。

```javascript
{
  "browserify": {
    "transform": [["babelify", { "presets": ["latest"] }]]
  }
}
```

### 線上轉換

Babel 提供一個[REPL線上編譯器](https://babeljs.io/repl/)，可以線上將 ES6 程式碼轉為 ES5 程式碼。轉換後的程式碼，可以直接作為 ES5 程式碼插入網頁執行。

### 與其他工具的配合

許多工具需要 Babel 進行前置轉碼，這裡舉兩個例子：ESLint 和 Mocha。

ESLint 用於靜態檢查程式碼的語法和風格，安裝命令如下。

```bash
$ npm install --save-dev eslint babel-eslint
```

然後，在專案根目錄下，新建一個配置檔案`.eslintrc`，在其中加入`parser`欄位。

```javascript
{
  "parser": "babel-eslint",
  "rules": {
    ...
  }
}
```

再在`package.json`之中，加入相應的`scripts`指令碼。

```javascript
  {
    "name": "my-module",
    "scripts": {
      "lint": "eslint my-files.js"
    },
    "devDependencies": {
      "babel-eslint": "...",
      "eslint": "..."
    }
  }
```

Mocha 則是一個測試框架，如果需要執行使用 ES6 語法的測試指令碼，可以修改`package.json`的`scripts.test`。

```javascript
"scripts": {
  "test": "mocha --ui qunit --compilers js:babel-core/register"
}
```

上面命令中，`--compilers`引數指定指令碼的轉碼器，規定字尾名為`js`的檔案，都需要使用`babel-core/register`先轉碼。

## Traceur 轉碼器

Google公司的[Traceur](https://github.com/google/traceur-compiler)轉碼器，也可以將 ES6 程式碼轉為 ES5 程式碼。

### 直接插入網頁

Traceur 允許將 ES6 程式碼直接插入網頁。首先，必須在網頁頭部載入 Traceur 庫檔案。

```html
<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
<script src="https://google.github.io/traceur-compiler/bin/BrowserSystem.js"></script>
<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
<script type="module">
  import './Greeter.js';
</script>
```

上面程式碼中，一共有4個`script`標籤。第一個是載入 Traceur 的庫檔案，第二個和第三個是將這個庫檔案用於瀏覽器環境，第四個則是載入使用者指令碼，這個腳本里面可以使用ES6程式碼。

注意，第四個`script`標籤的`type`屬性的值是`module`，而不是`text/javascript`。這是 Traceur 編譯器識別 ES6 程式碼的標誌，編譯器會自動將所有`type=module`的程式碼編譯為 ES5，然後再交給瀏覽器執行。

除了引用外部 ES6 指令碼，也可以直接在網頁中放置 ES6 程式碼。

```javascript
<script type="module">
  class Calc {
    constructor() {
      console.log('Calc constructor');
    }
    add(a, b) {
      return a + b;
    }
  }

  var c = new Calc();
  console.log(c.add(4,5));
</script>
```

正常情況下，上面程式碼會在控制檯打印出`9`。

如果想對 Traceur 的行為有精確控制，可以採用下面引數配置的寫法。

```javascript
<script>
  // Create the System object
  window.System = new traceur.runtime.BrowserTraceurLoader();
  // Set some experimental options
  var metadata = {
    traceurOptions: {
      experimental: true,
      properTailCalls: true,
      symbols: true,
      arrayComprehension: true,
      asyncFunctions: true,
      asyncGenerators: exponentiation,
      forOn: true,
      generatorComprehension: true
    }
  };
  // Load your module
  System.import('./myModule.js', {metadata: metadata}).catch(function(ex) {
    console.error('Import failed', ex.stack || ex);
  });
</script>
```

上面程式碼中，首先生成Traceur的全域性物件`window.System`，然後`System.import`方法可以用來載入 ES6。載入的時候，需要傳入一個配置物件`metadata`，該物件的`traceurOptions`屬性可以配置支援 ES6 功能。如果設為`experimental: true`，就表示除了 ES6 以外，還支援一些實驗性的新功能。

### 線上轉換

Traceur也提供一個[線上編譯器](http://google.github.io/traceur-compiler/demo/repl.html)，可以線上將 ES6 程式碼轉為 ES5 程式碼。轉換後的程式碼，可以直接作為 ES5 程式碼插入網頁執行。

上面的例子轉為 ES5 程式碼執行，就是下面這個樣子。

```javascript
<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
<script src="https://google.github.io/traceur-compiler/bin/BrowserSystem.js"></script>
<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
<script>
$traceurRuntime.ModuleStore.getAnonymousModule(function() {
  "use strict";

  var Calc = function Calc() {
    console.log('Calc constructor');
  };

  ($traceurRuntime.createClass)(Calc, {add: function(a, b) {
    return a + b;
  }}, {});

  var c = new Calc();
  console.log(c.add(4, 5));
  return {};
});
</script>
```

### 命令列轉換

作為命令列工具使用時，Traceur 是一個 Node 的模組，首先需要用 Npm 安裝。

```bash
$ npm install -g traceur
```

安裝成功後，就可以在命令列下使用 Traceur 了。

Traceur 直接執行 ES6 指令碼檔案，會在標準輸出顯示執行結果，以前面的`calc.js`為例。

```bash
$ traceur calc.js
Calc constructor
9
```

如果要將 ES6 指令碼轉為 ES5 儲存，要採用下面的寫法。

```bash
$ traceur --script calc.es6.js --out calc.es5.js
```

上面程式碼的`--script`選項表示指定輸入檔案，`--out`選項表示指定輸出檔案。

為了防止有些特性編譯不成功，最好加上`--experimental`選項。

```bash
$ traceur --script calc.es6.js --out calc.es5.js --experimental
```

命令列下轉換生成的檔案，就可以直接放到瀏覽器中執行。

### Node 環境的用法

Traceur 的 Node用法如下（假定已安裝`traceur`模組）。

```javascript
var traceur = require('traceur');
var fs = require('fs');

// 將ES6指令碼轉為字串
var contents = fs.readFileSync('es6-file.js').toString();

var result = traceur.compile(contents, {
  filename: 'es6-file.js',
  sourceMap: true,
  // 其他設定
  modules: 'commonjs'
});

if (result.error)
  throw result.error;

// result物件的js屬性就是轉換後的ES5程式碼
fs.writeFileSync('out.js', result.js);
// sourceMap屬性對應map檔案
fs.writeFileSync('out.js.map', result.sourceMap);
```

