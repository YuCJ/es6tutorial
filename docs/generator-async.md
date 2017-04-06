# Generator 函式的非同步應用

非同步程式設計對 JavaScript 語言太重要。Javascript 語言的執行環境是“單執行緒”的，如果沒有非同步程式設計，根本沒法用，非卡死不可。本章主要介紹 Generator 函式如何完成非同步操作。

## 傳統方法

ES6 誕生以前，非同步程式設計的方法，大概有下面四種。

- 回呼函式
- 事件監聽
- 釋出/訂閱
- Promise 物件

Generator 函式將 JavaScript 非同步程式設計帶入了一個全新的階段。

## 基本概念

### 非同步

所謂"非同步"，簡單說就是一個任務不是連續完成的，可以理解成該任務被人為分成兩段，先執行第一段，然後轉而執行其他任務，等做好了準備，再回過頭執行第二段。

比如，有一個任務是讀取檔案進行處理，任務的第一段是向作業系統發出請求，要求讀取檔案。然後，程式執行其他任務，等到作業系統返回檔案，再接著執行任務的第二段（處理檔案）。這種不連續的執行，就叫做非同步。

相應地，連續的執行就叫做同步。由於是連續執行，不能插入其他任務，所以作業系統從硬碟讀取檔案的這段時間，程式只能乾等著。

### 回呼函式

JavaScript 語言對非同步程式設計的實現，就是回呼函式。所謂回呼函式，就是把任務的第二段單獨寫在一個函式裡面，等到重新執行這個任務的時候，就直接呼叫這個函式。回呼函式的英語名字`callback`，直譯過來就是"重新呼叫"。

讀取檔案進行處理，是這樣寫的。

```javascript
fs.readFile('/etc/passwd', 'utf-8', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

上面程式碼中，`readFile`函式的第三個引數，就是回呼函式，也就是任務的第二段。等到作業系統返回了`/etc/passwd`這個檔案以後，回呼函式才會執行。

一個有趣的問題是，為什麼 Node 約定，回呼函式的第一個引數，必須是錯誤物件`err`（如果沒有錯誤，該引數就是`null`）？

原因是執行分成兩段，第一段執行完以後，任務所在的上下文環境就已經結束了。在這以後丟擲的錯誤，原來的上下文環境已經無法捕捉，只能當作引數，傳入第二段。

### Promise

回呼函式本身並沒有問題，它的問題出現在多個回呼函式巢狀。假定讀取`A`檔案之後，再讀取`B`檔案，程式碼如下。

```javascript
fs.readFile(fileA, 'utf-8', function (err, data) {
  fs.readFile(fileB, 'utf-8', function (err, data) {
    // ...
  });
});
```

不難想象，如果依次讀取兩個以上的檔案，就會出現多重巢狀。程式碼不是縱向發展，而是橫向發展，很快就會亂成一團，無法管理。因為多個非同步操作形成了強耦合，只要有一個操作需要修改，它的上層回呼函式和下層回呼函式，可能都要跟著修改。這種情況就稱為"回呼函式地獄"（callback hell）。

Promise 物件就是為了解決這個問題而提出的。它不是新的語法功能，而是一種新的寫法，允許將回調函式的巢狀，改成鏈式呼叫。採用 Promise，連續讀取多個檔案，寫法如下。

```javascript
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function (data) {
  console.log(data.toString());
})
.then(function () {
  return readFile(fileB);
})
.then(function (data) {
  console.log(data.toString());
})
.catch(function (err) {
  console.log(err);
});
```

上面程式碼中，我使用了`fs-readfile-promise`模組，它的作用就是返回一個 Promise 版本的`readFile`函式。Promise 提供`then`方法載入回呼函式，`catch`方法捕捉執行過程中丟擲的錯誤。

可以看到，Promise 的寫法只是回呼函式的改進，使用`then`方法以後，非同步任務的兩段執行看得更清楚了，除此以外，並無新意。

Promise 的最大問題是程式碼冗餘，原來的任務被 Promise 包裝了一下，不管什麼操作，一眼看去都是一堆`then`，原來的語義變得很不清楚。

那麼，有沒有更好的寫法呢？

## Generator 函式

### 協程

傳統的程式語言，早有非同步程式設計的解決方案（其實是多工的解決方案）。其中有一種叫做"協程"（coroutine），意思是多個執行緒互相協作，完成非同步任務。

協程有點像函式，又有點像執行緒。它的執行流程大致如下。

- 第一步，協程`A`開始執行。
- 第二步，協程`A`執行到一半，進入暫停，執行權轉移到協程`B`。
- 第三步，（一段時間後）協程`B`交還執行權。
- 第四步，協程`A`恢復執行。

上面流程的協程`A`，就是非同步任務，因為它分成兩段（或多段）執行。

舉例來說，讀取檔案的協程寫法如下。

```javascript
function *asyncJob() {
  // ...其他程式碼
  var f = yield readFile(fileA);
  // ...其他程式碼
}
```

上面程式碼的函式`asyncJob`是一個協程，它的奧妙就在其中的`yield`命令。它表示執行到此處，執行權將交給其他協程。也就是說，`yield`命令是非同步兩個階段的分界線。

協程遇到`yield`命令就暫停，等到執行權返回，再從暫停的地方繼續往後執行。它的最大優點，就是程式碼的寫法非常像同步操作，如果去除`yield`命令，簡直一模一樣。

### 協程的 Generator 函式實現

Generator 函式是協程在 ES6 的實現，最大特點就是可以交出函式的執行權（即暫停執行）。

整個 Generator 函式就是一個封裝的非同步任務，或者說是非同步任務的容器。非同步操作需要暫停的地方，都用`yield`語句註明。Generator 函式的執行方法如下。

```javascript
function* gen(x) {
  var y = yield x + 2;
  return y;
}

var g = gen(1);
g.next() // { value: 3, done: false }
g.next() // { value: undefined, done: true }
```

上面程式碼中，呼叫 Generator 函式，會返回一個內部指標（即遍歷器）`g`。這是 Generator 函式不同於普通函式的另一個地方，即執行它不會返回結果，返回的是指標物件。呼叫指標`g`的`next`方法，會移動內部指標（即執行非同步任務的第一段），指向第一個遇到的`yield`語句，上例是執行到`x + 2`為止。

換言之，`next`方法的作用是分階段執行`Generator`函式。每次呼叫`next`方法，會返回一個物件，表示當前階段的資訊（`value`屬性和`done`屬性）。`value`屬性是`yield`語句後面表示式的值，表示當前階段的值；`done`屬性是一個布林值，表示 Generator 函式是否執行完畢，即是否還有下一個階段。

### Generator 函式的資料交換和錯誤處理

Generator 函式可以暫停執行和恢復執行，這是它能封裝非同步任務的根本原因。除此之外，它還有兩個特性，使它可以作為非同步程式設計的完整解決方案：函式體內外的資料交換和錯誤處理機制。

`next`返回值的value屬性，是 Generator 函式向外輸出資料；`next`方法還可以接受引數，向 Generator 函式體內輸入資料。

```javascript
function* gen(x){
  var y = yield x + 2;
  return y;
}

var g = gen(1);
g.next() // { value: 3, done: false }
g.next(2) // { value: 2, done: true }
```

上面程式碼中，第一`next`方法的`value`屬性，返回表示式`x + 2`的值`3`。第二個`next`方法帶有引數`2`，這個引數可以傳入 Generator 函式，作為上個階段非同步任務的返回結果，被函式體內的變數`y`接收。因此，這一步的`value`屬性，返回的就是`2`（變數`y`的值）。

Generator 函式內部還可以部署錯誤處理程式碼，捕獲函式體外丟擲的錯誤。

```javascript
function* gen(x){
  try {
    var y = yield x + 2;
  } catch (e){
    console.log(e);
  }
  return y;
}

var g = gen(1);
g.next();
g.throw('出錯了');
// 出錯了
```

上面程式碼的最後一行，Generator 函式體外，使用指標物件的`throw`方法丟擲的錯誤，可以被函式體內的`try...catch`程式碼塊捕獲。這意味著，出錯的程式碼與處理錯誤的程式碼，實現了時間和空間上的分離，這對於非同步程式設計無疑是很重要的。

### 非同步任務的封裝

下面看看如何使用 Generator 函式，執行一個真實的非同步任務。

```javascript
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);
  console.log(result.bio);
}
```

上面程式碼中，Generator 函式封裝了一個非同步操作，該操作先讀取一個遠端介面，然後從 JSON 格式的資料解析資訊。就像前面說過的，這段程式碼非常像同步操作，除了加上了`yield`命令。

執行這段程式碼的方法如下。

```javascript
var g = gen();
var result = g.next();

result.value.then(function(data){
  return data.json();
}).then(function(data){
  g.next(data);
});
```

上面程式碼中，首先執行 Generator 函式，獲取遍歷器物件，然後使用`next`方法（第二行），執行非同步任務的第一階段。由於`Fetch`模組返回的是一個 Promise 物件，因此要用`then`方法呼叫下一個`next`方法。

可以看到，雖然 Generator 函式將非同步操作表示得很簡潔，但是流程管理卻不方便（即何時執行第一階段、何時執行第二階段）。

## Thunk 函式

Thunk 函式是自動執行 Generator 函式的一種方法。

### 引數的求值策略

Thunk 函式早在上個世紀60年代就誕生了。

那時，程式語言剛剛起步，計算機學家還在研究，編譯器怎麼寫比較好。一個爭論的焦點是"求值策略"，即函式的引數到底應該何時求值。

```javascript
var x = 1;

function f(m){
  return m * 2;
}

f(x + 5)
```

上面程式碼先定義函式`f`，然後向它傳入表示式`x + 5`。請問，這個表示式應該何時求值？

一種意見是"傳值呼叫"（call by value），即在進入函式體之前，就計算`x + 5`的值（等於6），再將這個值傳入函式`f`。C語言就採用這種策略。

```javascript
f(x + 5)
// 傳值呼叫時，等同於
f(6)
```

另一種意見是“傳名呼叫”（call by name），即直接將表示式`x + 5`傳入函式體，只在用到它的時候求值。Haskell 語言採用這種策略。

```javascript
f(x + 5)
// 傳名呼叫時，等同於
(x + 5) * 2
```

傳值呼叫和傳名呼叫，哪一種比較好？

回答是各有利弊。傳值呼叫比較簡單，但是對引數求值的時候，實際上還沒用到這個引數，有可能造成效能損失。

```javascript
function f(a, b){
  return b;
}

f(3 * x * x - 2 * x - 1, x);
```

上面程式碼中，函式`f`的第一個引數是一個複雜的表示式，但是函式體內根本沒用到。對這個引數求值，實際上是不必要的。因此，有一些計算機學家傾向於"傳名呼叫"，即只在執行時求值。

### Thunk 函式的含義

編譯器的“傳名呼叫”實現，往往是將引數放到一個臨時函式之中，再將這個臨時函式傳入函式體。這個臨時函式就叫做 Thunk 函式。

```javascript
function f(m) {
  return m * 2;
}

f(x + 5);

// 等同於

var thunk = function () {
  return x + 5;
};

function f(thunk) {
  return thunk() * 2;
}
```

上面程式碼中，函式f的引數`x + 5`被一個函式替換了。凡是用到原引數的地方，對`Thunk`函式求值即可。

這就是 Thunk 函式的定義，它是“傳名呼叫”的一種實現策略，用來替換某個表示式。

### JavaScript 語言的 Thunk 函式

JavaScript 語言是傳值呼叫，它的 Thunk 函式含義有所不同。在 JavaScript 語言中，Thunk 函式替換的不是表示式，而是多引數函式，將其替換成一個只接受回呼函式作為引數的單引數函式。

```javascript
// 正常版本的readFile（多引數版本）
fs.readFile(fileName, callback);

// Thunk版本的readFile（單引數版本）
var Thunk = function (fileName) {
  return function (callback) {
    return fs.readFile(fileName, callback);
  };
};

var readFileThunk = Thunk(fileName);
readFileThunk(callback);
```

上面程式碼中，`fs`模組的`readFile`方法是一個多引數函式，兩個引數分別為檔名和回呼函式。經過轉換器處理，它變成了一個單引數函式，只接受回呼函式作為引數。這個單引數版本，就叫做 Thunk 函式。

任何函式，只要引數有回呼函式，就能寫成 Thunk 函式的形式。下面是一個簡單的 Thunk 函式轉換器。

```javascript
// ES5版本
var Thunk = function(fn){
  return function (){
    var args = Array.prototype.slice.call(arguments);
    return function (callback){
      args.push(callback);
      return fn.apply(this, args);
    }
  };
};

// ES6版本
var Thunk = function(fn) {
  return function (...args) {
    return function (callback) {
      return fn.call(this, ...args, callback);
    }
  };
};
```

使用上面的轉換器，生成`fs.readFile`的 Thunk 函式。

```javascript
var readFileThunk = Thunk(fs.readFile);
readFileThunk(fileA)(callback);
```

下面是另一個完整的例子。

```javascript
function f(a, cb) {
  cb(a);
}
let ft = Thunk(f);

let log = console.log.bind(console);
ft(1)(log) // 1
```

### Thunkify 模組

生產環境的轉換器，建議使用 Thunkify 模組。

首先是安裝。

```bash
$ npm install thunkify
```

使用方式如下。

```javascript
var thunkify = require('thunkify');
var fs = require('fs');

var read = thunkify(fs.readFile);
read('package.json')(function(err, str){
  // ...
});
```

Thunkify 的原始碼與上一節那個簡單的轉換器非常像。

```javascript
function thunkify(fn) {
  return function() {
    var args = new Array(arguments.length);
    var ctx = this;

    for (var i = 0; i < args.length; ++i) {
      args[i] = arguments[i];
    }

    return function (done) {
      var called;

      args.push(function () {
        if (called) return;
        called = true;
        done.apply(null, arguments);
      });

      try {
        fn.apply(ctx, args);
      } catch (err) {
        done(err);
      }
    }
  }
};
```

它的原始碼主要多了一個檢查機制，變數`called`確保回呼函式只執行一次。這樣的設計與下文的 Generator 函式相關。請看下面的例子。

```javascript
function f(a, b, callback){
  var sum = a + b;
  callback(sum);
  callback(sum);
}

var ft = thunkify(f);
var print = console.log.bind(console);
ft(1, 2)(print);
// 3
```

上面程式碼中，由於`thunkify`只允許回呼函式執行一次，所以只輸出一行結果。

### Generator 函式的流程管理

你可能會問， Thunk 函式有什麼用？回答是以前確實沒什麼用，但是 ES6 有了 Generator 函式，Thunk 函式現在可以用於 Generator 函式的自動流程管理。

Generator 函式可以自動執行。

```javascript
function* gen() {
  // ...
}

var g = gen();
var res = g.next();

while(!res.done){
  console.log(res.value);
  res = g.next();
}
```

上面程式碼中，Generator 函式`gen`會自動執行完所有步驟。

但是，這不適合非同步操作。如果必須保證前一步執行完，才能執行後一步，上面的自動執行就不可行。這時，Thunk 函式就能派上用處。以讀取檔案為例。下面的 Generator 函式封裝了兩個非同步操作。

```javascript
var fs = require('fs');
var thunkify = require('thunkify');
var readFileThunk = thunkify(fs.readFile);

var gen = function* (){
  var r1 = yield readFileThunk('/etc/fstab');
  console.log(r1.toString());
  var r2 = yield readFileThunk('/etc/shells');
  console.log(r2.toString());
};
```

上面程式碼中，`yield`命令用於將程式的執行權移出 Generator 函式，那麼就需要一種方法，將執行權再交還給 Generator 函式。

這種方法就是 Thunk 函式，因為它可以在回呼函式裡，將執行權交還給 Generator 函式。為了便於理解，我們先看如何手動執行上面這個 Generator 函式。

```javascript
var g = gen();

var r1 = g.next();
r1.value(function (err, data) {
  if (err) throw err;
  var r2 = g.next(data);
  r2.value(function (err, data) {
    if (err) throw err;
    g.next(data);
  });
});
```

上面程式碼中，變數`g`是 Generator 函式的內部指標，表示目前執行到哪一步。`next`方法負責將指標移動到下一步，並返回該步的資訊（`value`屬性和`done`屬性）。

仔細檢視上面的程式碼，可以發現 Generator 函式的執行過程，其實是將同一個回呼函式，反覆傳入`next`方法的`value`屬性。這使得我們可以用遞迴來自動完成這個過程。

### Thunk 函式的自動流程管理

Thunk 函式真正的威力，在於可以自動執行 Generator 函式。下面就是一個基於 Thunk 函式的 Generator 執行器。

```javascript
function run(fn) {
  var gen = fn();

  function next(err, data) {
    var result = gen.next(data);
    if (result.done) return;
    result.value(next);
  }

  next();
}

function* g() {
  // ...
}

run(g);
```

上面程式碼的`run`函式，就是一個 Generator 函式的自動執行器。內部的`next`函式就是 Thunk 的回呼函式。`next`函式先將指標移到 Generator 函式的下一步（`gen.next`方法），然後判斷 Generator 函式是否結束（`result.done`屬性），如果沒結束，就將`next`函式再傳入 Thunk 函式（`result.value`屬性），否則就直接退出。

有了這個執行器，執行 Generator 函式方便多了。不管內部有多少個非同步操作，直接把 Generator 函式傳入`run`函式即可。當然，前提是每一個非同步操作，都要是 Thunk 函式，也就是說，跟在`yield`命令後面的必須是 Thunk 函式。

```javascript
var g = function* (){
  var f1 = yield readFile('fileA');
  var f2 = yield readFile('fileB');
  // ...
  var fn = yield readFile('fileN');
};

run(g);
```

上面程式碼中，函式`g`封裝了`n`個非同步的讀取檔案操作，只要執行`run`函式，這些操作就會自動完成。這樣一來，非同步操作不僅可以寫得像同步操作，而且一行程式碼就可以執行。

Thunk 函式並不是 Generator 函式自動執行的唯一方案。因為自動執行的關鍵是，必須有一種機制，自動控制 Generator 函式的流程，接收和交還程式的執行權。回呼函式可以做到這一點，Promise 物件也可以做到這一點。

## co 模組

### 基本用法

[co 模組](https://github.com/tj/co)是著名程式設計師 TJ Holowaychuk 於2013年6月釋出的一個小工具，用於 Generator 函式的自動執行。

下面是一個 Generator 函式，用於依次讀取兩個檔案。

```javascript
var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

co 模組可以讓你不用編寫 Generator 函式的執行器。

```javascript
var co = require('co');
co(gen);
```

上面程式碼中，Generator 函式只要傳入`co`函式，就會自動執行。

`co`函式返回一個`Promise`物件，因此可以用`then`方法添加回調函式。

```javascript
co(gen).then(function (){
  console.log('Generator 函式執行完成');
});
```

上面程式碼中，等到 Generator 函式執行結束，就會輸出一行提示。

### co 模組的原理

為什麼 co 可以自動執行 Generator 函式？

前面說過，Generator 就是一個非同步操作的容器。它的自動執行需要一種機制，當非同步操作有了結果，能夠自動交回執行權。

兩種方法可以做到這一點。

（1）回呼函式。將非同步操作包裝成 Thunk 函式，在回呼函式裡面交回執行權。

（2）Promise 物件。將非同步操作包裝成 Promise 物件，用`then`方法交回執行權。

co 模組其實就是將兩種自動執行器（Thunk 函式和 Promise 物件），包裝成一個模組。使用 co 的前提條件是，Generator 函式的`yield`命令後面，只能是 Thunk 函式或 Promise 物件。如果陣列或物件的成員，全部都是 Promise 物件，也可以使用 co，詳見後文的例子。（co v4.0版以後，`yield`命令後面只能是 Promise 物件，不再支援 Thunk 函式。）

上一節已經介紹了基於 Thunk 函式的自動執行器。下面來看，基於 Promise 物件的自動執行器。這是理解 co 模組必須的。

### 基於 Promise 物件的自動執行

還是沿用上面的例子。首先，把`fs`模組的`readFile`方法包裝成一個 Promise 物件。

```javascript
var fs = require('fs');

var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) return reject(error);
      resolve(data);
    });
  });
};

var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

然後，手動執行上面的 Generator 函式。

```javascript
var g = gen();

g.next().value.then(function(data){
  g.next(data).value.then(function(data){
    g.next(data);
  });
});
```

手動執行其實就是用`then`方法，層層添加回調函式。理解了這一點，就可以寫出一個自動執行器。

```javascript
function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

run(gen);
```

上面程式碼中，只要 Generator 函式還沒執行到最後一步，`next`函式就呼叫自身，以此實現自動執行。

### co 模組的原始碼

co 就是上面那個自動執行器的擴展，它的原始碼只有幾十行，非常簡單。

首先，co 函式接受 Generator 函式作為引數，返回一個 Promise 物件。

```javascript
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
  });
}
```

在返回的 Promise 物件裡面，co 先檢查引數`gen`是否為 Generator 函式。如果是，就執行該函式，得到一個內部指標物件；如果不是就返回，並將 Promise 物件的狀態改為`resolved`。

```javascript
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.call(ctx);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);
  });
}
```

接著，co 將 Generator 函式的內部指標物件的`next`方法，包裝成`onFulfilled`函式。這主要是為了能夠捕捉丟擲的錯誤。

```javascript
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.call(ctx);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);

    onFulfilled();
    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }
  });
}
```

最後，就是關鍵的`next`函式，它會反覆呼叫自身。

```javascript
function next(ret) {
  if (ret.done) return resolve(ret.value);
  var value = toPromise.call(ctx, ret.value);
  if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
  return onRejected(
    new TypeError(
      'You may only yield a function, promise, generator, array, or object, '
      + 'but the following object was passed: "'
      + String(ret.value)
      + '"'
    )
  );
}
```

上面程式碼中，`next`函式的內部程式碼，一共只有四行命令。

第一行，檢查當前是否為 Generator 函式的最後一步，如果是就返回。

第二行，確保每一步的返回值，是 Promise 物件。

第三行，使用`then`方法，為返回值加上回調函式，然後通過`onFulfilled`函式再次呼叫`next`函式。

第四行，在引數不符合要求的情況下（引數非 Thunk 函式和 Promise 物件），將 Promise 物件的狀態改為`rejected`，從而終止執行。

### 處理併發的非同步操作

co 支援併發的非同步操作，即允許某些操作同時進行，等到它們全部完成，才進行下一步。

這時，要把併發的操作都放在陣列或物件裡面，跟在`yield`語句後面。

```javascript
// 陣列的寫法
co(function* () {
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res);
}).catch(onerror);

// 物件的寫法
co(function* () {
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2),
  };
  console.log(res);
}).catch(onerror);
```

下面是另一個例子。

```javascript
co(function* () {
  var values = [n1, n2, n3];
  yield values.map(somethingAsync);
});

function* somethingAsync(x) {
  // do something async
  return y
}
```

上面的程式碼允許併發三個`somethingAsync`非同步操作，等到它們全部完成，才會進行下一步。

### 實例：處理 Stream

Node 提供 Stream 模式讀寫資料，特點是一次只處理資料的一部分，資料分成一塊塊依次處理，就好像“資料流”一樣。這對於處理大規模資料非常有利。Stream 模式使用 EventEmitter API，會釋放三個事件。

- `data`事件：下一塊資料塊已經準備好了。
- `end`事件：整個“資料流”處理“完了。
- `error`事件：發生錯誤。

使用`Promise.race()`函式，可以判斷這三個事件之中哪一個最先發生，只有當`data`時間最先發生時，才進入下一個資料塊的處理。從而，通過一個`while`迴圈，完成所有資料的讀取。

```javascript
const co = require('co');
const fs = require('fs');

const stream = fs.createReadStream('./les_miserables.txt');
let valjeanCount = 0;

co(function*() {
  while(true) {
    const res = yield Promise.race([
      new Promise(resolve => stream.once('data', resolve)),
      new Promise(resolve => stream.once('end', resolve)),
      new Promise((resolve, reject) => stream.once('error', reject))
    ]);
    if (!res) {
      break;
    }
    stream.removeAllListeners('data');
    stream.removeAllListeners('end');
    stream.removeAllListeners('error');
    valjeanCount += (res.toString().match(/valjean/ig) || []).length;
  }
  console.log('count:', valjeanCount); // count: 1120
});
```

上面程式碼採用 Stream 模式讀取《悲慘世界》的文字檔案，對於每個資料塊都使用`stream.once`方法，在`data`、`end`、`error`三個事件上新增一次性回呼函式。變數`res`只有在`data`事件發生時，才有值。然後，累加每個資料塊之中`valjean`這個詞出現的次數。

