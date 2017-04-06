# async 函式

## 含義

ES2017 標準引入了 async 函式，使得非同步操作變得更加方便。

async 函式是什麼？一句話，它就是 Generator 函式的語法糖。

前文有一個 Generator 函式，依次讀取兩個檔案。

```javascript
var fs = require('fs');

var readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) reject(error);
      resolve(data);
    });
  });
};

var gen = function* () {
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

寫成`async`函式，就是下面這樣。

```javascript
var asyncReadFile = async function () {
  var f1 = await readFile('/etc/fstab');
  var f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

一比較就會發現，`async`函式就是將 Generator 函式的星號（`*`）替換成`async`，將`yield`替換成`await`，僅此而已。

`async`函式對 Generator 函式的改進，體現在以下四點。

（1）內建執行器。

Generator 函式的執行必須靠執行器，所以才有了`co`模組，而`async`函式自帶執行器。也就是說，`async`函式的執行，與普通函式一模一樣，只要一行。

```javascript
var result = asyncReadFile();
```

上面的程式碼呼叫了`asyncReadFile`函式，然後它就會自動執行，輸出最後結果。這完全不像 Generator 函式，需要呼叫`next`方法，或者用`co`模組，才能真正執行，得到最後結果。

（2）更好的語義。

`async`和`await`，比起星號和`yield`，語義更清楚了。`async`表示函式裡有非同步操作，`await`表示緊跟在後面的表示式需要等待結果。

（3）更廣的適用性。

`co`模組約定，`yield`命令後面只能是 Thunk 函式或 Promise 物件，而`async`函式的`await`命令後面，可以是Promise 物件和原始型別的值（數值、字串和布林值，但這時等同於同步操作）。

（4）返回值是 Promise。

`async`函式的返回值是 Promise 物件，這比 Generator 函式的返回值是 Iterator 物件方便多了。你可以用`then`方法指定下一步的操作。

進一步說，`async`函式完全可以看作多個非同步操作，包裝成的一個 Promise 物件，而`await`命令就是內部`then`命令的語法糖。

## 用法

### 基本用法

`async`函式返回一個 Promise 物件，可以使用`then`方法添加回調函式。當函式執行的時候，一旦遇到`await`就會先返回，等到非同步操作完成，再接著執行函式體內後面的語句。

下面是一個例子。

```javascript
async function getStockPriceByName(name) {
  var symbol = await getStockSymbol(name);
  var stockPrice = await getStockPrice(symbol);
  return stockPrice;
}

getStockPriceByName('goog').then(function (result) {
  console.log(result);
});
```

上面程式碼是一個獲取股票報價的函式，函式前面的`async`關鍵字，表明該函式內部有非同步操作。呼叫該函式時，會立即返回一個`Promise`物件。

下面是另一個例子，指定多少毫秒後輸出一個值。

```javascript
function timeout(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

上面程式碼指定50毫秒以後，輸出`hello world`。

由於`async`函式返回的是 Promise 物件，可以作為`await`命令的引數。所以，上面的例子也可以寫成下面的形式。

```javascript
async function timeout(ms) {
  await new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}

async function asyncPrint(value, ms) {
  await timeout(ms);
  console.log(value);
}

asyncPrint('hello world', 50);
```

async 函式有多種使用形式。

```javascript
// 函式宣告
async function foo() {}

// 函式表示式
const foo = async function () {};

// 物件的方法
let obj = { async foo() {} };
obj.foo().then(...)

// Class 的方法
class Storage {
  constructor() {
    this.cachePromise = caches.open('avatars');
  }

  async getAvatar(name) {
    const cache = await this.cachePromise;
    return cache.match(`/avatars/${name}.jpg`);
  }
}

const storage = new Storage();
storage.getAvatar('jake').then(…);

// 箭頭函式
const foo = async () => {};
```

## 語法

`async`函式的語法規則總體上比較簡單，難點是錯誤處理機制。

### 返回 Promise 物件

`async`函式返回一個 Promise 物件。

`async`函式內部`return`語句返回的值，會成為`then`方法回呼函式的引數。

```javascript
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```

上面程式碼中，函式`f`內部`return`命令返回的值，會被`then`方法回呼函式接收到。

`async`函式內部丟擲錯誤，會導致返回的 Promise 物件變為`reject`狀態。丟擲的錯誤物件會被`catch`方法回呼函式接收到。

```javascript
async function f() {
  throw new Error('出錯了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出錯了
```

### Promise 物件的狀態變化

`async`函式返回的 Promise 物件，必須等到內部所有`await`命令後面的 Promise 物件執行完，才會發生狀態改變，除非遇到`return`語句或者丟擲錯誤。也就是說，只有`async`函式內部的非同步操作執行完，才會執行`then`方法指定的回呼函式。

下面是一個例子。

```javascript
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

上面程式碼中，函式`getTitle`內部有三個操作：抓取網頁、取出文字、匹配頁面標題。只有這三個操作全部完成，才會執行`then`方法裡面的`console.log`。

### await 命令

正常情況下，`await`命令後面是一個 Promise 物件。如果不是，會被轉成一個立即`resolve`的 Promise 物件。

```javascript
async function f() {
  return await 123;
}

f().then(v => console.log(v))
// 123
```

上面程式碼中，`await`命令的引數是數值`123`，它被轉成 Promise 物件，並立即`resolve`。

`await`命令後面的 Promise 物件如果變為`reject`狀態，則`reject`的引數會被`catch`方法的回呼函式接收到。

```javascript
async function f() {
  await Promise.reject('出錯了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出錯了
```

注意，上面程式碼中，`await`語句前面沒有`return`，但是`reject`方法的引數依然傳入了`catch`方法的回呼函式。這裡如果在`await`前面加上`return`，效果是一樣的。

只要一個`await`語句後面的 Promise 變為`reject`，那麼整個`async`函式都會中斷執行。

```javascript
async function f() {
  await Promise.reject('出錯了');
  await Promise.resolve('hello world'); // 不會執行
}
```

上面程式碼中，第二個`await`語句是不會執行的，因為第一個`await`語句狀態變成了`reject`。

有時，我們希望即使前一個非同步操作失敗，也不要中斷後面的非同步操作。這時可以將第一個`await`放在`try...catch`結構裡面，這樣不管這個非同步操作是否成功，第二個`await`都會執行。

```javascript
async function f() {
  try {
    await Promise.reject('出錯了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// hello world
```

另一種方法是`await`後面的 Promise 物件再跟一個`catch`方法，處理前面可能出現的錯誤。

```javascript
async function f() {
  await Promise.reject('出錯了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出錯了
// hello world
```

### 錯誤處理

如果`await`後面的非同步操作出錯，那麼等同於`async`函式返回的 Promise 物件被`reject`。

```javascript
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出錯了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出錯了
```

上面程式碼中，`async`函式`f`執行後，`await`後面的 Promise 物件會丟擲一個錯誤物件，導致`catch`方法的回呼函式被呼叫，它的引數就是丟擲的錯誤物件。具體的執行機制，可以參考後文的“async 函式的實現原理”。

防止出錯的方法，也是將其放在`try...catch`程式碼塊之中。

```javascript
async function f() {
  try {
    await new Promise(function (resolve, reject) {
      throw new Error('出錯了');
    });
  } catch(e) {
  }
  return await('hello world');
}
```

如果有多個`await`命令，可以統一放在`try...catch`結構中。

```javascript
async function main() {
  try {
    var val1 = await firstStep();
    var val2 = await secondStep(val1);
    var val3 = await thirdStep(val1, val2);

    console.log('Final: ', val3);
  }
  catch (err) {
    console.error(err);
  }
}
```

下面的例子使用`try...catch`結構，實現多次重複嘗試。

```javascript
const superagent = require('superagent');
const NUM_RETRIES = 3;

async function test() {
  let i;
  for (i = 0; i < NUM_RETRIES; ++i) {
    try {
      await superagent.get('http://google.com/this-throws-an-error');
      break;
    } catch(err) {}
  }
  console.log(i); // 3
}

test();
```

上面程式碼中，如果`await`操作成功，就會使用`break`語句退出迴圈；如果失敗，會被`catch`語句捕捉，然後進入下一輪迴圈。

### 使用注意點

第一點，前面已經說過，`await`命令後面的`Promise`物件，執行結果可能是`rejected`，所以最好把`await`命令放在`try...catch`程式碼塊中。

```javascript
async function myFunction() {
  try {
    await somethingThatReturnsAPromise();
  } catch (err) {
    console.log(err);
  }
}

// 另一種寫法

async function myFunction() {
  await somethingThatReturnsAPromise()
  .catch(function (err) {
    console.log(err);
  };
}
```

第二點，多個`await`命令後面的非同步操作，如果不存在繼發關係，最好讓它們同時觸發。

```javascript
let foo = await getFoo();
let bar = await getBar();
```

上面程式碼中，`getFoo`和`getBar`是兩個獨立的非同步操作（即互不依賴），被寫成繼發關係。這樣比較耗時，因為只有`getFoo`完成以後，才會執行`getBar`，完全可以讓它們同時觸發。

```javascript
// 寫法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 寫法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

上面兩種寫法，`getFoo`和`getBar`都是同時觸發，這樣就會縮短程式的執行時間。

第三點，`await`命令只能用在`async`函式之中，如果用在普通函式，就會報錯。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  // 報錯
  docs.forEach(function (doc) {
    await db.post(doc);
  });
}
```

上面程式碼會報錯，因為`await`用在普通函式之中了。但是，如果將`forEach`方法的引數改成`async`函式，也有問題。

```javascript
function dbFuc(db) { //這裡不需要 async
  let docs = [{}, {}, {}];

  // 可能得到錯誤結果
  docs.forEach(async function (doc) {
    await db.post(doc);
  });
}
```

上面程式碼可能不會正常工作，原因是這時三個`db.post`操作將是併發執行，也就是同時執行，而不是繼發執行。正確的寫法是採用`for`迴圈。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}
```

如果確實希望多個請求併發執行，可以使用`Promise.all`方法。

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的寫法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```

## async 函式的實現原理

async 函式的實現原理，就是將 Generator 函式和自動執行器，包裝在一個函式裡。

```javascript
async function fn(args) {
  // ...
}

// 等同於

function fn(args) {
  return spawn(function* () {
    // ...
  });
}
```

所有的`async`函式都可以寫成上面的第二種形式，其中的`spawn`函式就是自動執行器。

下面給出`spawn`函式的實現，基本就是前文自動執行器的翻版。

```javascript
function spawn(genF) {
  return new Promise(function(resolve, reject) {
    var gen = genF();
    function step(nextF) {
      try {
        var next = nextF();
      } catch(e) {
        return reject(e);
      }
      if(next.done) {
        return resolve(next.value);
      }
      Promise.resolve(next.value).then(function(v) {
        step(function() { return gen.next(v); });
      }, function(e) {
        step(function() { return gen.throw(e); });
      });
    }
    step(function() { return gen.next(undefined); });
  });
}
```

## 與其他非同步處理方法的比較

我們通過一個例子，來看 async 函式與 Promise、Generator 函式的比較。

假定某個 DOM 元素上面，部署了一系列的動畫，前一個動畫結束，才能開始後一個。如果當中有一個動畫出錯，就不再往下執行，返回上一個成功執行的動畫的返回值。

首先是 Promise 的寫法。

```javascript
function chainAnimationsPromise(elem, animations) {

  // 變數ret用來儲存上一個動畫的返回值
  var ret = null;

  // 新建一個空的Promise
  var p = Promise.resolve();

  // 使用then方法，新增所有動畫
  for(var anim of animations) {
    p = p.then(function(val) {
      ret = val;
      return anim(elem);
    });
  }

  // 返回一個部署了錯誤捕捉機制的Promise
  return p.catch(function(e) {
    /* 忽略錯誤，繼續執行 */
  }).then(function() {
    return ret;
  });

}
```

雖然 Promise 的寫法比回呼函式的寫法大大改進，但是一眼看上去，程式碼完全都是 Promise 的 API（`then`、`catch`等等），操作本身的語義反而不容易看出來。

接著是 Generator 函式的寫法。

```javascript
function chainAnimationsGenerator(elem, animations) {

  return spawn(function*() {
    var ret = null;
    try {
      for(var anim of animations) {
        ret = yield anim(elem);
      }
    } catch(e) {
      /* 忽略錯誤，繼續執行 */
    }
    return ret;
  });

}
```

上面程式碼使用 Generator 函式遍歷了每個動畫，語義比 Promise 寫法更清晰，使用者定義的操作全部都出現在`spawn`函式的內部。這個寫法的問題在於，必須有一個任務執行器，自動執行 Generator 函式，上面程式碼的`spawn`函式就是自動執行器，它返回一個 Promise 物件，而且必須保證`yield`語句後面的表示式，必須返回一個 Promise。

最後是 async 函式的寫法。

```javascript
async function chainAnimationsAsync(elem, animations) {
  var ret = null;
  try {
    for(var anim of animations) {
      ret = await anim(elem);
    }
  } catch(e) {
    /* 忽略錯誤，繼續執行 */
  }
  return ret;
}
```

可以看到Async函式的實現最簡潔，最符合語義，幾乎沒有語義不相關的程式碼。它將Generator寫法中的自動執行器，改在語言層面提供，不暴露給使用者，因此程式碼量最少。如果使用Generator寫法，自動執行器需要使用者自己提供。

## 實例：按順序完成非同步操作

實際開發中，經常遇到一組非同步操作，需要按照順序完成。比如，依次遠端讀取一組 URL，然後按照讀取的順序輸出結果。

Promise 的寫法如下。

```javascript
function logInOrder(urls) {
  // 遠端讀取所有URL
  const textPromises = urls.map(url => {
    return fetch(url).then(response => response.text());
  });

  // 按次序輸出
  textPromises.reduce((chain, textPromise) => {
    return chain.then(() => textPromise)
      .then(text => console.log(text));
  }, Promise.resolve());
}
```

上面程式碼使用`fetch`方法，同時遠端讀取一組 URL。每個`fetch`操作都返回一個 Promise 物件，放入`textPromises`陣列。然後，`reduce`方法依次處理每個 Promise 物件，然後使用`then`，將所有 Promise 物件連起來，因此就可以依次輸出結果。

這種寫法不太直觀，可讀性比較差。下面是 async 函式實現。

```javascript
async function logInOrder(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    console.log(await response.text());
  }
}
```

上面程式碼確實大大簡化，問題是所有遠端操作都是繼發。只有前一個URL返回結果，才會去讀取下一個URL，這樣做效率很差，非常浪費時間。我們需要的是併發發出遠端請求。

```javascript
async function logInOrder(urls) {
  // 併發讀取遠端URL
  const textPromises = urls.map(async url => {
    const response = await fetch(url);
    return response.text();
  });

  // 按次序輸出
  for (const textPromise of textPromises) {
    console.log(await textPromise);
  }
}
```

上面程式碼中，雖然`map`方法的引數是`async`函式，但它是併發執行的，因為只有`async`函式內部是繼發執行，外部不受影響。後面的`for..of`迴圈內部使用了`await`，因此實現了按順序輸出。

## 非同步遍歷器

《遍歷器》一章說過，Iterator 介面是一種資料遍歷的協議，只要呼叫遍歷器物件的`next`方法，就會得到一個物件，表示當前遍歷指標所在的那個位置的資訊。`next`方法返回的物件的結構是`{value, done}`，其中`value`表示當前的資料的值，`done`是一個布林值，表示遍歷是否結束。

這裡隱含著一個規定，`next`方法必須是同步的，只要呼叫就必須立刻返回值。也就是說，一旦執行`next`方法，就必須同步地得到`value`和`done`這兩個屬性。如果遍歷指標正好指向同步操作，當然沒有問題，但對於非同步操作，就不太合適了。目前的解決方法是，Generator 函式裡面的非同步操作，返回一個 Thunk 函式或者 Promise 物件，即`value`屬性是一個 Thunk 函式或者 Promise 物件，等待以後返回真正的值，而`done`屬性則還是同步產生的。

目前，有一個[提案](https://github.com/tc39/proposal-async-iteration)，為非同步操作提供原生的遍歷器介面，即`value`和`done`這兩個屬性都是非同步產生，這稱為”非同步遍歷器“（Async Iterator）。

### 非同步遍歷的介面

非同步遍歷器的最大的語法特點，就是呼叫遍歷器的`next`方法，返回的是一個 Promise 物件。

```javascript
asyncIterator
  .next()
  .then(
    ({ value, done }) => /* ... */
  );
```

上面程式碼中，`asyncIterator`是一個非同步遍歷器，呼叫`next`方法以後，返回一個 Promise 物件。因此，可以使用`then`方法指定，這個 Promise 物件的狀態變為`resolve`以後的回呼函式。回呼函式的引數，則是一個具有`value`和`done`兩個屬性的物件，這個跟同步遍歷器是一樣的。

我們知道，一個物件的同步遍歷器的介面，部署在`Symbol.iterator`屬性上面。同樣地，物件的非同步遍歷器介面，部署在`Symbol.asyncIterator`屬性上面。不管是什麼樣的物件，只要它的`Symbol.asyncIterator`屬性有值，就表示應該對它進行非同步遍歷。

下面是一個非同步遍歷器的例子。

```javascript
const asyncIterable = createAsyncIterable(['a', 'b']);
const asyncIterator = asyncIterable[Symbol.asyncIterator]();

asyncIterator
.next()
.then(iterResult1 => {
  console.log(iterResult1); // { value: 'a', done: false }
  return asyncIterator.next();
})
.then(iterResult2 => {
  console.log(iterResult2); // { value: 'b', done: false }
  return asyncIterator.next();
})
.then(iterResult3 => {
  console.log(iterResult3); // { value: undefined, done: true }
});
```

上面程式碼中，非同步遍歷器其實返回了兩次值。第一次呼叫的時候，返回一個 Promise 物件；等到 Promise 物件`resolve`了，再返回一個表示當前資料成員資訊的物件。這就是說，非同步遍歷器與同步遍歷器最終行為是一致的，只是會先返回 Promise 物件，作為中介。

由於非同步遍歷器的`next`方法，返回的是一個 Promise 物件。因此，可以把它放在`await`命令後面。

```javascript
async function f() {
  const asyncIterable = createAsyncIterable(['a', 'b']);
  const asyncIterator = asyncIterable[Symbol.asyncIterator]();
  console.log(await asyncIterator.next());
  // { value: 'a', done: false }
  console.log(await asyncIterator.next());
  // { value: 'b', done: false }
  console.log(await asyncIterator.next());
  // { value: undefined, done: true }
}
```

上面程式碼中，`next`方法用`await`處理以後，就不必使用`then`方法了。整個流程已經很接近同步處理了。

注意，非同步遍歷器的`next`方法是可以連續呼叫的，不必等到上一步產生的Promise物件`resolve`以後再呼叫。這種情況下，`next`方法會累積起來，自動按照每一步的順序執行下去。下面是一個例子，把所有的`next`方法放在`Promise.all`方法裡面。

```javascript
const asyncGenObj = createAsyncIterable(['a', 'b']);
const [{value: v1}, {value: v2}] = await Promise.all([
  asyncGenObj.next(), asyncGenObj.next()
]);

console.log(v1, v2); // a b
```

另一種用法是一次性呼叫所有的`next`方法，然後`await`最後一步操作。

```javascript
const writer = openFile('someFile.txt');
writer.next('hello');
writer.next('world');
await writer.return();
```

### for await...of

前面介紹過，`for...of`迴圈用於遍歷同步的 Iterator 介面。新引入的`for await...of`迴圈，則是用於遍歷非同步的 Iterator 介面。

```javascript
async function f() {
  for await (const x of createAsyncIterable(['a', 'b'])) {
    console.log(x);
  }
}
// a
// b
```

上面程式碼中，`createAsyncIterable()`返回一個非同步遍歷器，`for...of`迴圈自動呼叫這個遍歷器的`next`方法，會得到一個Promise物件。`await`用來處理這個Promise物件，一旦`resolve`，就把得到的值（`x`）傳入`for...of`的迴圈體。

`for await...of`迴圈的一個用途，是部署了 asyncIterable 操作的非同步介面，可以直接放入這個迴圈。

```javascript
let body = '';
for await(const data of req) body += data;
const parsed = JSON.parse(body);
console.log('got', parsed);
```

上面程式碼中，`req`是一個 asyncIterable 物件，用來非同步讀取資料。可以看到，使用`for await...of`迴圈以後，程式碼會非常簡潔。

如果`next`方法返回的Promise物件被`reject`，那麼就要用`try...catch`捕捉。

```javascript
async function () {
  try {
    for await (const x of createRejectingIterable()) {
      console.log(x);
    }
  } catch (e) {
    console.error(e);
  }
}
```

注意，`for await...of`迴圈也可以用於同步遍歷器。

```javascript
(async function () {
  for await (const x of ['a', 'b']) {
    console.log(x);
  }
})();
// a
// b
```

### 非同步Generator函式

就像 Generator 函式返回一個同步遍歷器物件一樣，非同步 Generator 函式的作用，是返回一個非同步遍歷器物件。

在語法上，非同步 Generator 函式就是`async`函式與 Generator 函式的結合。

```javascript
async function* readLines(path) {
  let file = await fileOpen(path);

  try {
    while (!file.EOF) {
      yield await file.readLine();
    }
  } finally {
    await file.close();
  }
}
```

上面程式碼中，非同步操作前面使用`await`關鍵字標明，即`await`後面的操作，應該返回Promise物件。凡是使用`yield`關鍵字的地方，就是`next`方法的停下來的地方，它後面的表示式的值（即`await file.readLine()`的值），會作為`next()`返回物件的`value`屬性，這一點是於同步Generator函式一致的。

可以像下面這樣，使用上面程式碼定義的非同步Generator函式。

```javascript
for await (const line of readLines(filePath)) {
  console.log(line);
}
```

非同步 Generator 函式可以與`for await...of`迴圈結合起來使用。

```javascript
async function* prefixLines(asyncIterable) {
  for await (const line of asyncIterable) {
    yield '> ' + line;
  }
}
```

`yield`命令依然是立刻返回的，但是返回的是一個Promise物件。

```javascript
async function* asyncGenerator() {
  console.log('Start');
  const result = await doSomethingAsync(); // (A)
  yield 'Result: '+ result; // (B)
  console.log('Done');
}
```

上面程式碼中，呼叫`next`方法以後，會在`B`處暫停執行，`yield`命令立刻返回一個Promise物件。這個Promise物件不同於`A`處`await`命令後面的那個 Promise 物件。主要有兩點不同，一是`A`處的Promise物件`resolve`以後產生的值，會放入`result`變數；二是`B`處的Promise物件`resolve`以後產生的值，是表示式`'Result： ' + result`的值；二是`A`處的 Promise 物件一定先於`B`處的 Promise 物件`resolve`。

如果非同步 Generator 函式丟擲錯誤，會被 Promise 物件`reject`，然後丟擲的錯誤被`catch`方法捕獲。

```javascript
async function* asyncGenerator() {
  throw new Error('Problem!');
}

asyncGenerator()
.next()
.catch(err => console.log(err)); // Error: Problem!
```

注意，普通的 async 函式返回的是一個 Promise 物件，而非同步 Generator 函式返回的是一個非同步Iterator物件。基本上，可以這樣理解，`async`函式和非同步 Generator 函式，是封裝非同步操作的兩種方法，都用來達到同一種目的。區別在於，前者自帶執行器，後者通過`for await...of`執行，或者自己編寫執行器。下面就是一個非同步 Generator 函式的執行器。

```javascript
async function takeAsync(asyncIterable, count=Infinity) {
  const result = [];
  const iterator = asyncIterable[Symbol.asyncIterator]();
  while (result.length < count) {
    const {value,done} = await iterator.next();
    if (done) break;
    result.push(value);
  }
  return result;
}
```

上面程式碼中，非同步Generator函式產生的非同步遍歷器，會通過`while`迴圈自動執行，每當`await iterator.next()`完成，就會進入下一輪迴圈。

下面是這個自動執行器的一個使用實例。

```javascript
async function f() {
  async function* gen() {
    yield 'a';
    yield 'b';
    yield 'c';
  }

  return await takeAsync(gen());
}

f().then(function (result) {
  console.log(result); // ['a', 'b', 'c']
})
```

非同步 Generator 函數出現以後，JavaScript就有了四種函式形式：普通函式、async 函式、Generator 函式和非同步 Generator 函式。請注意區分每種函式的不同之處。

最後，同步的資料結構，也可以使用非同步 Generator 函式。

```javascript
async function* createAsyncIterable(syncIterable) {
  for (const elem of syncIterable) {
    yield elem;
  }
}
```

上面程式碼中，由於沒有非同步操作，所以也就沒有使用`await`關鍵字。

### yield* 語句

`yield*`語句也可以跟一個非同步遍歷器。

```javascript
async function* gen1() {
  yield 'a';
  yield 'b';
  return 2;
}

async function* gen2() {
  const result = yield* gen1();
}
```

上面程式碼中，`gen2`函式裡面的`result`變數，最後的值是`2`。

與同步Generator函式一樣，`for await...of`迴圈會展開`yield*`。

```javascript
(async function () {
  for await (const x of gen2()) {
    console.log(x);
  }
})();
// a
// b
```

