# 物件的擴充套件

## 屬性的簡潔表示法

ES6允許直接寫入變數和函式，作為物件的屬性和方法。這樣的書寫更加簡潔。

```javascript
var foo = 'bar';
var baz = {foo};
baz // {foo: "bar"}

// 等同於
var baz = {foo: foo};
```

上面程式碼表明，ES6 允許在物件之中，直接寫變數。這時，屬性名為變數名, 屬性值為變數的值。下面是另一個例子。

```javascript
function f(x, y) {
  return {x, y};
}

// 等同於

function f(x, y) {
  return {x: x, y: y};
}

f(1, 2) // Object {x: 1, y: 2}
```

除了屬性簡寫，方法也可以簡寫。

```javascript
var o = {
  method() {
    return "Hello!";
  }
};

// 等同於

var o = {
  method: function() {
    return "Hello!";
  }
};
```

下面是一個實際的例子。

```javascript
var birth = '2000/01/01';

var Person = {

  name: '張三',

  //等同於birth: birth
  birth,

  // 等同於hello: function ()...
  hello() { console.log('我的名字是', this.name); }

};
```

這種寫法用於函式的返回值，將會非常方便。

```javascript
function getPoint() {
  var x = 1;
  var y = 10;
  return {x, y};
}

getPoint()
// {x:1, y:10}
```

CommonJS模組輸出變數，就非常合適使用簡潔寫法。

```javascript
var ms = {};

function getItem (key) {
  return key in ms ? ms[key] : null;
}

function setItem (key, value) {
  ms[key] = value;
}

function clear () {
  ms = {};
}

module.exports = { getItem, setItem, clear };
// 等同於
module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
```

屬性的賦值器（setter）和取值器（getter），事實上也是採用這種寫法。

```javascript
var cart = {
  _wheels: 4,

  get wheels () {
    return this._wheels;
  },

  set wheels (value) {
    if (value < this._wheels) {
      throw new Error('數值太小了！');
    }
    this._wheels = value;
  }
}
```

注意，簡潔寫法的屬性名總是字串，這會導致一些看上去比較奇怪的結果。

```javascript
var obj = {
  class () {}
};

// 等同於

var obj = {
  'class': function() {}
};
```

上面程式碼中，`class`是字串，所以不會因為它屬於關鍵字，而導致語法解析報錯。

如果某個方法的值是一個Generator函式，前面需要加上星號。

```javascript
var obj = {
  * m(){
    yield 'hello world';
  }
};
```

## 屬性名錶達式

JavaScript語言定義物件的屬性，有兩種方法。

```javascript
// 方法一
obj.foo = true;

// 方法二
obj['a' + 'bc'] = 123;
```

上面程式碼的方法一是直接用識別符號作為屬性名，方法二是用表示式作為屬性名，這時要將表示式放在方括號之內。

但是，如果使用字面量方式定義物件（使用大括號），在 ES5 中只能使用方法一（識別符號）定義屬性。

```javascript
var obj = {
  foo: true,
  abc: 123
};
```

ES6 允許字面量定義物件時，用方法二（表示式）作為物件的屬性名，即把表示式放在方括號內。

```javascript
let propKey = 'foo';

let obj = {
  [propKey]: true,
  ['a' + 'bc']: 123
};
```

下面是另一個例子。

```javascript
var lastWord = 'last word';

var a = {
  'first word': 'hello',
  [lastWord]: 'world'
};

a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"
```

表示式還可以用於定義方法名。

```javascript
let obj = {
  ['h' + 'ello']() {
    return 'hi';
  }
};

obj.hello() // hi
```

注意，屬性名錶達式與簡潔表示法，不能同時使用，會報錯。

```javascript
// 報錯
var foo = 'bar';
var bar = 'abc';
var baz = { [foo] };

// 正確
var foo = 'bar';
var baz = { [foo]: 'abc'};
```

注意，屬性名錶達式如果是一個物件，預設情況下會自動將物件轉為字串`[object Object]`，這一點要特別小心。

```javascript
const keyA = {a: 1};
const keyB = {b: 2};

const myObject = {
  [keyA]: 'valueA',
  [keyB]: 'valueB'
};

myObject // Object {[object Object]: "valueB"}
```

上面程式碼中，`[keyA]`和`[keyB]`得到的都是`[object Object]`，所以`[keyB]`會把`[keyA]`覆蓋掉，而`myObject`最後只有一個`[object Object]`屬性。

## 方法的 name 屬性

函式的`name`屬性，返回函式名。物件方法也是函式，因此也有`name`屬性。

```javascript
const person = {
  sayName() {
    console.log('hello!');
  },
};

person.sayName.name   // "sayName"
```

上面程式碼中，方法的`name`屬性返回函式名（即方法名）。

如果物件的方法使用了取值函式（`getter`）和存值函式（`setter`），則`name`屬性不是在該方法上面，而是該方法的屬性的描述物件的`get`和`set`屬性上面，返回值是方法名前加上`get`和`set`。

```javascript
const obj = {
  get foo() {},
  set foo(x) {}
};

obj.foo.name
// TypeError: Cannot read property 'name' of undefined

const descriptor = Object.getOwnPropertyDescriptor(obj, 'foo');

descriptor.get.name // "get foo"
descriptor.set.name // "set foo"
```

有兩種特殊情況：`bind`方法創造的函式，`name`屬性返回`bound`加上原函式的名字；`Function`建構函式創造的函式，`name`屬性返回`anonymous`。

```javascript
(new Function()).name // "anonymous"

var doSomething = function() {
  // ...
};
doSomething.bind().name // "bound doSomething"
```

如果物件的方法是一個 Symbol 值，那麼`name`屬性返回的是這個 Symbol 值的描述。

```javascript
const key1 = Symbol('description');
const key2 = Symbol();
let obj = {
  [key1]() {},
  [key2]() {},
};
obj[key1].name // "[description]"
obj[key2].name // ""
```

上面程式碼中，`key1`對應的 Symbol 值有描述，`key2`沒有。

## Object.is()

ES5比較兩個值是否相等，只有兩個運算子：相等運算子（`==`）和嚴格相等運算子（`===`）。它們都有缺點，前者會自動轉換資料型別，後者的`NaN`不等於自身，以及`+0`等於`-0`。JavaScript缺乏一種運算，在所有環境中，只要兩個值是一樣的，它們就應該相等。

ES6提出“Same-value equality”（同值相等）演算法，用來解決這個問題。`Object.is`就是部署這個演算法的新方法。它用來比較兩個值是否嚴格相等，與嚴格比較運算子（===）的行為基本一致。

```javascript
Object.is('foo', 'foo')
// true
Object.is({}, {})
// false
```

不同之處只有兩個：一是`+0`不等於`-0`，二是`NaN`等於自身。

```javascript
+0 === -0 //true
NaN === NaN // false

Object.is(+0, -0) // false
Object.is(NaN, NaN) // true
```

ES5可以通過下面的程式碼，部署`Object.is`。

```javascript
Object.defineProperty(Object, 'is', {
  value: function(x, y) {
    if (x === y) {
      // 針對+0 不等於 -0的情況
      return x !== 0 || 1 / x === 1 / y;
    }
    // 針對NaN的情況
    return x !== x && y !== y;
  },
  configurable: true,
  enumerable: false,
  writable: true
});
```

## Object.assign()

### 基本用法

`Object.assign`方法用於物件的合併，將源物件（source）的所有可列舉屬性，複製到目標物件（target）。

```javascript
var target = { a: 1 };

var source1 = { b: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

`Object.assign`方法的第一個引數是目標物件，後面的引數都是源物件。

注意，如果目標物件與源物件有同名屬性，或多個源物件有同名屬性，則後面的屬性會覆蓋前面的屬性。

```javascript
var target = { a: 1, b: 1 };

var source1 = { b: 2, c: 2 };
var source2 = { c: 3 };

Object.assign(target, source1, source2);
target // {a:1, b:2, c:3}
```

如果只有一個引數，`Object.assign`會直接返回該引數。

```javascript
var obj = {a: 1};
Object.assign(obj) === obj // true
```

如果該引數不是物件，則會先轉成物件，然後返回。

```javascript
typeof Object.assign(2) // "object"
```

由於`undefined`和`null`無法轉成物件，所以如果它們作為引數，就會報錯。

```javascript
Object.assign(undefined) // 報錯
Object.assign(null) // 報錯
```

如果非物件引數出現在源物件的位置（即非首引數），那麼處理規則有所不同。首先，這些引數都會轉成物件，如果無法轉成物件，就會跳過。這意味著，如果`undefined`和`null`不在首引數，就不會報錯。

```javascript
let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true
```

其他型別的值（即數值、字串和布林值）不在首引數，也不會報錯。但是，除了字串會以陣列形式，拷貝入目標物件，其他值都不會產生效果。

```javascript
var v1 = 'abc';
var v2 = true;
var v3 = 10;

var obj = Object.assign({}, v1, v2, v3);
console.log(obj); // { "0": "a", "1": "b", "2": "c" }
```

上面程式碼中，`v1`、`v2`、`v3`分別是字串、布林值和數值，結果只有字串合入目標物件（以字元陣列的形式），數值和布林值都會被忽略。這是因為只有字串的包裝物件，會產生可列舉屬性。

```javascript
Object(true) // {[[PrimitiveValue]]: true}
Object(10)  //  {[[PrimitiveValue]]: 10}
Object('abc') // {0: "a", 1: "b", 2: "c", length: 3, [[PrimitiveValue]]: "abc"}
```

上面程式碼中，布林值、數值、字串分別轉成對應的包裝物件，可以看到它們的原始值都在包裝物件的內部屬性`[[PrimitiveValue]]`上面，這個屬性是不會被`Object.assign`拷貝的。只有字串的包裝物件，會產生可列舉的實義屬性，那些屬性則會被拷貝。

`Object.assign`拷貝的屬性是有限制的，只拷貝源物件的自身屬性（不拷貝繼承屬性），也不拷貝不可列舉的屬性（`enumerable: false`）。

```javascript
Object.assign({b: 'c'},
  Object.defineProperty({}, 'invisible', {
    enumerable: false,
    value: 'hello'
  })
)
// { b: 'c' }
```

上面程式碼中，`Object.assign`要拷貝的物件只有一個不可列舉屬性`invisible`，這個屬性並沒有被拷貝進去。

屬性名為Symbol值的屬性，也會被`Object.assign`拷貝。

```javascript
Object.assign({ a: 'b' }, { [Symbol('c')]: 'd' })
// { a: 'b', Symbol(c): 'd' }
```

### 注意點

`Object.assign`方法實行的是淺拷貝，而不是深拷貝。也就是說，如果源物件某個屬性的值是物件，那麼目標物件拷貝得到的是這個物件的引用。

```javascript
var obj1 = {a: {b: 1}};
var obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
obj2.a.b // 2
```

上面程式碼中，源物件`obj1`的`a`屬性的值是一個物件，`Object.assign`拷貝得到的是這個物件的引用。這個物件的任何變化，都會反映到目標物件上面。

對於這種巢狀的物件，一旦遇到同名屬性，`Object.assign`的處理方法是替換，而不是新增。

```javascript
var target = { a: { b: 'c', d: 'e' } }
var source = { a: { b: 'hello' } }
Object.assign(target, source)
// { a: { b: 'hello' } }
```

上面程式碼中，`target`物件的`a`屬性被`source`物件的`a`屬性整個替換掉了，而不會得到`{ a: { b: 'hello', d: 'e' } }`的結果。這通常不是開發者想要的，需要特別小心。

有一些函式庫提供`Object.assign`的定製版本（比如Lodash的`_.defaultsDeep`方法），可以解決淺拷貝的問題，得到深拷貝的合併。

注意，`Object.assign`可以用來處理陣列，但是會把陣列視為物件。

```javascript
Object.assign([1, 2, 3], [4, 5])
// [4, 5, 3]
```

上面程式碼中，`Object.assign`把陣列視為屬性名為0、1、2的物件，因此源陣列的0號屬性`4`覆蓋了目標陣列的0號屬性`1`。

### 常見用途

`Object.assign`方法有很多用處。

**（1）為物件新增屬性**

```javascript
class Point {
  constructor(x, y) {
    Object.assign(this, {x, y});
  }
}
```

上面方法通過`Object.assign`方法，將`x`屬性和`y`屬性新增到`Point`類的物件實例。

**（2）為物件新增方法**

```javascript
Object.assign(SomeClass.prototype, {
  someMethod(arg1, arg2) {
    ···
  },
  anotherMethod() {
    ···
  }
});

// 等同於下面的寫法
SomeClass.prototype.someMethod = function (arg1, arg2) {
  ···
};
SomeClass.prototype.anotherMethod = function () {
  ···
};
```

上面程式碼使用了物件屬性的簡潔表示法，直接將兩個函式放在大括號中，再使用assign方法新增到SomeClass.prototype之中。

**（3）克隆物件**

```javascript
function clone(origin) {
  return Object.assign({}, origin);
}
```

上面程式碼將原始物件拷貝到一個空物件，就得到了原始物件的克隆。

不過，採用這種方法克隆，只能克隆原始物件自身的值，不能克隆它繼承的值。如果想要保持繼承鏈，可以採用下面的程式碼。

```javascript
function clone(origin) {
  let originProto = Object.getPrototypeOf(origin);
  return Object.assign(Object.create(originProto), origin);
}
```

**（4）合併多個物件**

將多個物件合併到某個物件。

```javascript
const merge =
  (target, ...sources) => Object.assign(target, ...sources);
```

如果希望合併後返回一個新物件，可以改寫上面函式，對一個空物件合併。

```javascript
const merge =
  (...sources) => Object.assign({}, ...sources);
```

**（5）為屬性指定預設值**

```javascript
const DEFAULTS = {
  logLevel: 0,
  outputFormat: 'html'
};

function processContent(options) {
  options = Object.assign({}, DEFAULTS, options);
  console.log(options);
  // ...
}
```

上面程式碼中，`DEFAULTS`物件是預設值，`options`物件是使用者提供的引數。`Object.assign`方法將`DEFAULTS`和`options`合併成一個新物件，如果兩者有同名屬性，則`option`的屬性值會覆蓋`DEFAULTS`的屬性值。

注意，由於存在深拷貝的問題，`DEFAULTS`物件和`options`物件的所有屬性的值，最好都是簡單型別，不要指向另一個物件。否則，`DEFAULTS`物件的該屬性很可能不起作用。

```javascript
const DEFAULTS = {
  url: {
    host: 'example.com',
    port: 7070
  },
};

processContent({ url: {port: 8000} })
// {
//   url: {port: 8000}
// }
```

上面程式碼的原意是將`url.port`改成8000，`url.host`不變。實際結果卻是`options.url`覆蓋掉`DEFAULTS.url`，所以`url.host`就不存在了。

## 屬性的可列舉性

物件的每個屬性都有一個描述物件（Descriptor），用來控制該屬性的行為。`Object.getOwnPropertyDescriptor`方法可以獲取該屬性的描述物件。

```javascript
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```

描述物件的`enumerable`屬性，稱為”可列舉性“，如果該屬性為`false`，就表示某些操作會忽略當前屬性。

ES5有三個操作會忽略`enumerable`為`false`的屬性。

- `for...in`迴圈：只遍歷物件自身的和繼承的可列舉的屬性
- `Object.keys()`：返回物件自身的所有可列舉的屬性的鍵名
- `JSON.stringify()`：只序列化物件自身的可列舉的屬性

ES6新增了一個操作`Object.assign()`，會忽略`enumerable`為`false`的屬性，只拷貝物件自身的可列舉的屬性。

這四個操作之中，只有`for...in`會返回繼承的屬性。實際上，引入`enumerable`的最初目的，就是讓某些屬性可以規避掉`for...in`操作。比如，物件原型的`toString`方法，以及陣列的`length`屬性，就通過這種手段，不會被`for...in`遍歷到。

```javascript
Object.getOwnPropertyDescriptor(Object.prototype, 'toString').enumerable
// false

Object.getOwnPropertyDescriptor([], 'length').enumerable
// false
```

上面程式碼中，`toString`和`length`屬性的`enumerable`都是`false`，因此`for...in`不會遍歷到這兩個繼承自原型的屬性。

另外，ES6規定，所有Class的原型的方法都是不可列舉的。

```javascript
Object.getOwnPropertyDescriptor(class {foo() {}}.prototype, 'foo').enumerable
// false
```

總的來說，操作中引入繼承的屬性會讓問題複雜化，大多數時候，我們只關心物件自身的屬性。所以，儘量不要用`for...in`迴圈，而用`Object.keys()`代替。

## 屬性的遍歷

ES6一共有5種方法可以遍歷物件的屬性。

**（1）for...in**

`for...in`迴圈遍歷物件自身的和繼承的可列舉屬性（不含Symbol屬性）。

**（2）Object.keys(obj)**

`Object.keys`返回一個數組，包括物件自身的（不含繼承的）所有可列舉屬性（不含Symbol屬性）。

**（3）Object.getOwnPropertyNames(obj)**

`Object.getOwnPropertyNames`返回一個數組，包含物件自身的所有屬性（不含Symbol屬性，但是包括不可列舉屬性）。

**（4）Object.getOwnPropertySymbols(obj)**

`Object.getOwnPropertySymbols`返回一個數組，包含物件自身的所有Symbol屬性。

**（5）Reflect.ownKeys(obj)**

`Reflect.ownKeys`返回一個數組，包含物件自身的所有屬性，不管是屬性名是Symbol或字串，也不管是否可列舉。

以上的5種方法遍歷物件的屬性，都遵守同樣的屬性遍歷的次序規則。

- 首先遍歷所有屬性名為數值的屬性，按照數字排序。
- 其次遍歷所有屬性名為字串的屬性，按照生成時間排序。
- 最後遍歷所有屬性名為Symbol值的屬性，按照生成時間排序。

```javascript
Reflect.ownKeys({ [Symbol()]:0, b:0, 10:0, 2:0, a:0 })
// ['2', '10', 'b', 'a', Symbol()]
```

上面程式碼中，`Reflect.ownKeys`方法返回一個數組，包含了引數物件的所有屬性。這個陣列的屬性次序是這樣的，首先是數值屬性`2`和`10`，其次是字串屬性`b`和`a`，最後是Symbol屬性。

## `__proto__`屬性，Object.setPrototypeOf()，Object.getPrototypeOf()

### `__proto__`屬性

`__proto__`屬性（前後各兩個下劃線），用來讀取或設定當前物件的`prototype`物件。目前，所有瀏覽器（包括 IE11）都部署了這個屬性。

```javascript
// es6的寫法
var obj = {
  method: function() { ... }
};
obj.__proto__ = someOtherObj;

// es5的寫法
var obj = Object.create(someOtherObj);
obj.method = function() { ... };
```

該屬性沒有寫入 ES6 的正文，而是寫入了附錄，原因是`__proto__`前後的雙下劃線，說明它本質上是一個內部屬性，而不是一個正式的對外的 API，只是由於瀏覽器廣泛支援，才被加入了 ES6。標準明確規定，只有瀏覽器必須部署這個屬性，其他執行環境不一定需要部署，而且新的程式碼最好認為這個屬性是不存在的。因此，無論從語義的角度，還是從相容性的角度，都不要使用這個屬性，而是使用下面的`Object.setPrototypeOf()`（寫操作）、`Object.getPrototypeOf()`（讀操作）、`Object.create()`（生成操作）代替。

在實現上，`__proto__`呼叫的是`Object.prototype.__proto__`，具體實現如下。

```javascript
Object.defineProperty(Object.prototype, '__proto__', {
  get() {
    let _thisObj = Object(this);
    return Object.getPrototypeOf(_thisObj);
  },
  set(proto) {
    if (this === undefined || this === null) {
      throw new TypeError();
    }
    if (!isObject(this)) {
      return undefined;
    }
    if (!isObject(proto)) {
      return undefined;
    }
    let status = Reflect.setPrototypeOf(this, proto);
    if (!status) {
      throw new TypeError();
    }
  },
});
function isObject(value) {
  return Object(value) === value;
}
```

如果一個物件本身部署了`__proto__`屬性，則該屬性的值就是物件的原型。

```javascript
Object.getPrototypeOf({ __proto__: null })
// null
```

### Object.setPrototypeOf()

`Object.setPrototypeOf`方法的作用與`__proto__`相同，用來設定一個物件的`prototype`物件，返回引數物件本身。它是 ES6 正式推薦的設定原型物件的方法。

```javascript
// 格式
Object.setPrototypeOf(object, prototype)

// 用法
var o = Object.setPrototypeOf({}, null);
```

該方法等同於下面的函式。

```javascript
function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```

下面是一個例子。

```javascript
let proto = {};
let obj = { x: 10 };
Object.setPrototypeOf(obj, proto);

proto.y = 20;
proto.z = 40;

obj.x // 10
obj.y // 20
obj.z // 40
```

上面程式碼將`proto`物件設為`obj`物件的原型，所以從`obj`物件可以讀取`proto`物件的屬性。

如果第一個引數不是物件，會自動轉為物件。但是由於返回的還是第一個引數，所以這個操作不會產生任何效果。

```javascript
Object.setPrototypeOf(1, {}) === 1 // true
Object.setPrototypeOf('foo', {}) === 'foo' // true
Object.setPrototypeOf(true, {}) === true // true
```

由於`undefined`和`null`無法轉為物件，所以如果第一個引數是`undefined`或`null`，就會報錯。

```javascript
Object.setPrototypeOf(undefined, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined
```

### Object.getPrototypeOf()

該方法與`Object.setPrototypeOf`方法配套，用於讀取一個物件的原型物件。

```javascript
Object.getPrototypeOf(obj);
```

下面是一個例子。

```javascript
function Rectangle() {
  // ...
}

var rec = new Rectangle();

Object.getPrototypeOf(rec) === Rectangle.prototype
// true

Object.setPrototypeOf(rec, Object.prototype);
Object.getPrototypeOf(rec) === Rectangle.prototype
// false
```

如果引數不是物件，會被自動轉為物件。

```javascript
// 等同於 Object.getPrototypeOf(Number(1))
Object.getPrototypeOf(1)
// Number {[[PrimitiveValue]]: 0}

// 等同於 Object.getPrototypeOf(String('foo'))
Object.getPrototypeOf('foo')
// String {length: 0, [[PrimitiveValue]]: ""}

// 等同於 Object.getPrototypeOf(Boolean(true))
Object.getPrototypeOf(true)
// Boolean {[[PrimitiveValue]]: false}

Object.getPrototypeOf(1) === Number.prototype // true
Object.getPrototypeOf('foo') === String.prototype // true
Object.getPrototypeOf(true) === Boolean.prototype // true
```

如果引數是`undefined`或`null`，它們無法轉為物件，所以會報錯。

```javascript
Object.getPrototypeOf(null)
// TypeError: Cannot convert undefined or null to object

Object.getPrototypeOf(undefined)
// TypeError: Cannot convert undefined or null to object
```

## Object.keys()，Object.values()，Object.entries()

### Object.keys()

ES5 引入了`Object.keys`方法，返回一個數組，成員是引數物件自身的（不含繼承的）所有可遍歷（enumerable）屬性的鍵名。

```javascript
var obj = { foo: 'bar', baz: 42 };
Object.keys(obj)
// ["foo", "baz"]
```

ES2017 [引入](https://github.com/tc39/proposal-object-values-entries)了跟`Object.keys`配套的`Object.values`和`Object.entries`，作為遍歷一個物件的補充手段，供`for...of`迴圈使用。

```javascript
let {keys, values, entries} = Object;
let obj = { a: 1, b: 2, c: 3 };

for (let key of keys(obj)) {
  console.log(key); // 'a', 'b', 'c'
}

for (let value of values(obj)) {
  console.log(value); // 1, 2, 3
}

for (let [key, value] of entries(obj)) {
  console.log([key, value]); // ['a', 1], ['b', 2], ['c', 3]
}
```

### Object.values()

`Object.values`方法返回一個數組，成員是引數物件自身的（不含繼承的）所有可遍歷（enumerable）屬性的鍵值。

```javascript
var obj = { foo: 'bar', baz: 42 };
Object.values(obj)
// ["bar", 42]
```

返回陣列的成員順序，與本章的《屬性的遍歷》部分介紹的排列規則一致。

```javascript
var obj = { 100: 'a', 2: 'b', 7: 'c' };
Object.values(obj)
// ["b", "c", "a"]
```

上面程式碼中，屬性名為數值的屬性，是按照數值大小，從小到大遍歷的，因此返回的順序是`b`、`c`、`a`。

`Object.values`只返回物件自身的可遍歷屬性。

```javascript
var obj = Object.create({}, {p: {value: 42}});
Object.values(obj) // []
```

上面程式碼中，`Object.create`方法的第二個引數新增的物件屬性（屬性`p`），如果不顯式宣告，預設是不可遍歷的，因為`p`的屬性描述物件的`enumerable`預設是`false`，`Object.values`不會返回這個屬性。只要把`enumerable`改成`true`，`Object.values`就會返回屬性`p`的值。

```javascript
var obj = Object.create({}, {p:
  {
    value: 42,
    enumerable: true
  }
});
Object.values(obj) // [42]
```

`Object.values`會過濾屬性名為 Symbol 值的屬性。

```javascript
Object.values({ [Symbol()]: 123, foo: 'abc' });
// ['abc']
```

如果`Object.values`方法的引數是一個字串，會返回各個字元組成的一個數組。

```javascript
Object.values('foo')
// ['f', 'o', 'o']
```

上面程式碼中，字串會先轉成一個類似陣列的物件。字串的每個字元，就是該物件的一個屬性。因此，`Object.values`返回每個屬性的鍵值，就是各個字元組成的一個數組。

如果引數不是物件，`Object.values`會先將其轉為物件。由於數值和布林值的包裝物件，都不會為實例新增非繼承的屬性。所以，`Object.values`會返回空陣列。

```javascript
Object.values(42) // []
Object.values(true) // []
```

### Object.entries

`Object.entries`方法返回一個數組，成員是引數物件自身的（不含繼承的）所有可遍歷（enumerable）屬性的鍵值對陣列。

```javascript
var obj = { foo: 'bar', baz: 42 };
Object.entries(obj)
// [ ["foo", "bar"], ["baz", 42] ]
```

除了返回值不一樣，該方法的行為與`Object.values`基本一致。

如果原物件的屬性名是一個 Symbol 值，該屬性會被忽略。

```javascript
Object.entries({ [Symbol()]: 123, foo: 'abc' });
// [ [ 'foo', 'abc' ] ]
```

上面程式碼中，原物件有兩個屬性，`Object.entries`只輸出屬性名非 Symbol 值的屬性。將來可能會有`Reflect.ownEntries()`方法，返回物件自身的所有屬性。

`Object.entries`的基本用途是遍歷物件的屬性。

```javascript
let obj = { one: 1, two: 2 };
for (let [k, v] of Object.entries(obj)) {
  console.log(
    `${JSON.stringify(k)}: ${JSON.stringify(v)}`
  );
}
// "one": 1
// "two": 2
```

`Object.entries`方法的另一個用處是，將物件轉為真正的`Map`結構。

```javascript
var obj = { foo: 'bar', baz: 42 };
var map = new Map(Object.entries(obj));
map // Map { foo: "bar", baz: 42 }
```

自己實現`Object.entries`方法，非常簡單。

```javascript
// Generator函式的版本
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

// 非Generator函式的版本
function entries(obj) {
  let arr = [];
  for (let key of Object.keys(obj)) {
    arr.push([key, obj[key]]);
  }
  return arr;
}
```

## 物件的擴充套件運算子

《陣列的擴充套件》一章中，已經介紹過擴充套件運算子（`...`）。

```javascript
const [a, ...b] = [1, 2, 3];
a // 1
b // [2, 3]
```

ES2017 將這個運算子[引入](https://github.com/sebmarkbage/ecmascript-rest-spread)了物件。

**（1）解構賦值**

物件的解構賦值用於從一個物件取值，相當於將所有可遍歷的、但尚未被讀取的屬性，分配到指定的物件上面。所有的鍵和它們的值，都會拷貝到新物件上面。

```javascript
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
x // 1
y // 2
z // { a: 3, b: 4 }
```

上面程式碼中，變數`z`是解構賦值所在的物件。它獲取等號右邊的所有尚未讀取的鍵（`a`和`b`），將它們連同值一起拷貝過來。

由於解構賦值要求等號右邊是一個物件，所以如果等號右邊是`undefined`或`null`，就會報錯，因為它們無法轉為物件。

```javascript
let { x, y, ...z } = null; // 執行時錯誤
let { x, y, ...z } = undefined; // 執行時錯誤
```

解構賦值必須是最後一個引數，否則會報錯。

```javascript
let { ...x, y, z } = obj; // 句法錯誤
let { x, ...y, ...z } = obj; // 句法錯誤
```

上面程式碼中，解構賦值不是最後一個引數，所以會報錯。

注意，解構賦值的拷貝是淺拷貝，即如果一個鍵的值是複合型別的值（陣列、物件、函式）、那麼解構賦值拷貝的是這個值的引用，而不是這個值的副本。

```javascript
let obj = { a: { b: 1 } };
let { ...x } = obj;
obj.a.b = 2;
x.a.b // 2
```

上面程式碼中，`x`是解構賦值所在的物件，拷貝了物件`obj`的`a`屬性。`a`屬性引用了一個物件，修改這個物件的值，會影響到解構賦值對它的引用。

另外，解構賦值不會拷貝繼承自原型物件的屬性。

```javascript
let o1 = { a: 1 };
let o2 = { b: 2 };
o2.__proto__ = o1;
let o3 = { ...o2 };
o3 // { b: 2 }
```

上面程式碼中，物件`o3`是`o2`的拷貝，但是隻複製了`o2`自身的屬性，沒有複製它的原型物件`o1`的屬性。

下面是另一個例子。

```javascript
var o = Object.create({ x: 1, y: 2 });
o.z = 3;

let { x, ...{ y, z } } = o;
x // 1
y // undefined
z // 3
```

上面程式碼中，變數`x`是單純的解構賦值，所以可以讀取繼承的屬性；解構賦值產生的變數`y`和`z`，只能讀取物件自身的屬性，所以只有變數`z`可以賦值成功。

解構賦值的一個用處，是擴充套件某個函式的引數，引入其他操作。

```javascript
function baseFunction({ a, b }) {
  // ...
}
function wrapperFunction({ x, y, ...restConfig }) {
  // 使用x和y引數進行操作
  // 其餘引數傳給原始函式
  return baseFunction(restConfig);
}
```

上面程式碼中，原始函式`baseFunction`接受`a`和`b`作為引數，函式`wrapperFunction`在`baseFunction`的基礎上進行了擴充套件，能夠接受多餘的引數，並且保留原始函式的行為。

**（2）擴充套件運算子**

擴充套件運算子（`...`）用於取出引數物件的所有可遍歷屬性，拷貝到當前物件之中。

```javascript
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
```

這等同於使用`Object.assign`方法。

```javascript
let aClone = { ...a };
// 等同於
let aClone = Object.assign({}, a);
```

擴充套件運算子可以用於合併兩個物件。

```javascript
let ab = { ...a, ...b };
// 等同於
let ab = Object.assign({}, a, b);
```

如果使用者自定義的屬性，放在擴充套件運算子後面，則擴充套件運算子內部的同名屬性會被覆蓋掉。

```javascript
let aWithOverrides = { ...a, x: 1, y: 2 };
// 等同於
let aWithOverrides = { ...a, ...{ x: 1, y: 2 } };
// 等同於
let x = 1, y = 2, aWithOverrides = { ...a, x, y };
// 等同於
let aWithOverrides = Object.assign({}, a, { x: 1, y: 2 });
```

上面程式碼中，`a`物件的`x`屬性和`y`屬性，拷貝到新物件後會被覆蓋掉。

這用來修改現有物件部分的部分屬性就很方便了。

```javascript
let newVersion = {
  ...previousVersion,
  name: 'New Name' // Override the name property
};
```

上面程式碼中，`newVersion`物件自定義了`name`屬性，其他屬性全部複製自`previousVersion`物件。

如果把自定義屬性放在擴充套件運算子前面，就變成了設定新物件的預設屬性值。

```javascript
let aWithDefaults = { x: 1, y: 2, ...a };
// 等同於
let aWithDefaults = Object.assign({}, { x: 1, y: 2 }, a);
// 等同於
let aWithDefaults = Object.assign({ x: 1, y: 2 }, a);
```

擴充套件運算子的引數物件之中，如果有取值函式`get`，這個函式是會執行的。

```javascript
// 並不會丟擲錯誤，因為x屬性只是被定義，但沒執行
let aWithXGetter = {
  ...a,
  get x() {
    throws new Error('not thrown yet');
  }
};

// 會丟擲錯誤，因為x屬性被執行了
let runtimeError = {
  ...a,
  ...{
    get x() {
      throws new Error('thrown now');
    }
  }
};
```

如果擴充套件運算子的引數是`null`或`undefined`，這個兩個值會被忽略，不會報錯。

```javascript
let emptyObject = { ...null, ...undefined }; // 不報錯
```

## Object.getOwnPropertyDescriptors()

ES5有一個`Object.getOwnPropertyDescriptor`方法，返回某個物件屬性的描述物件（descriptor）。

```javascript
var obj = { p: 'a' };

Object.getOwnPropertyDescriptor(obj, 'p')
// Object { value: "a",
//   writable: true,
//   enumerable: true,
//   configurable: true
// }
```

ES2017 引入了`Object.getOwnPropertyDescriptors`方法，返回指定物件所有自身屬性（非繼承屬性）的描述物件。

```javascript
const obj = {
  foo: 123,
  get bar() { return 'abc' }
};

Object.getOwnPropertyDescriptors(obj)
// { foo:
//    { value: 123,
//      writable: true,
//      enumerable: true,
//      configurable: true },
//   bar:
//    { get: [Function: bar],
//      set: undefined,
//      enumerable: true,
//      configurable: true } }
```

上面程式碼中，`Object.getOwnPropertyDescriptors`方法返回一個物件，所有原物件的屬性名都是該物件的屬性名，對應的屬性值就是該屬性的描述物件。

該方法的實現非常容易。

```javascript
function getOwnPropertyDescriptors(obj) {
  const result = {};
  for (let key of Reflect.ownKeys(obj)) {
    result[key] = Object.getOwnPropertyDescriptor(obj, key);
  }
  return result;
}
```

該方法的引入目的，主要是為了解決`Object.assign()`無法正確拷貝`get`屬性和`set`屬性的問題。

```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};

const target1 = {};
Object.assign(target1, source);

Object.getOwnPropertyDescriptor(target1, 'foo')
// { value: undefined,
//   writable: true,
//   enumerable: true,
//   configurable: true }
```

上面程式碼中，`source`物件的`foo`屬性的值是一個賦值函式，`Object.assign`方法將這個屬性拷貝給`target1`物件，結果該屬性的值變成了`undefined`。這是因為`Object.assign`方法總是拷貝一個屬性的值，而不會拷貝它背後的賦值方法或取值方法。

這時，`Object.getOwnPropertyDescriptors`方法配合`Object.defineProperties`方法，就可以實現正確拷貝。

```javascript
const source = {
  set foo(value) {
    console.log(value);
  }
};

const target2 = {};
Object.defineProperties(target2, Object.getOwnPropertyDescriptors(source));
Object.getOwnPropertyDescriptor(target2, 'foo')
// { get: undefined,
//   set: [Function: foo],
//   enumerable: true,
//   configurable: true }
```

上面程式碼中，將兩個物件合併的邏輯提煉出來，就是下面這樣。

```javascript
const shallowMerge = (target, source) => Object.defineProperties(
  target,
  Object.getOwnPropertyDescriptors(source)
);
```

`Object.getOwnPropertyDescriptors`方法的另一個用處，是配合`Object.create`方法，將物件屬性克隆到一個新物件。這屬於淺拷貝。

```javascript
const clone = Object.create(Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj));

// 或者

const shallowClone = (obj) => Object.create(
  Object.getPrototypeOf(obj),
  Object.getOwnPropertyDescriptors(obj)
);
```

上面程式碼會克隆物件`obj`。

另外，`Object.getOwnPropertyDescriptors`方法可以實現一個物件繼承另一個物件。以前，繼承另一個物件，常常寫成下面這樣。

```javascript
const obj = {
  __proto__: prot,
  foo: 123,
};
```

ES6 規定`__proto__`只有瀏覽器要部署，其他環境不用部署。如果去除`__proto__`，上面程式碼就要改成下面這樣。

```javascript
const obj = Object.create(prot);
obj.foo = 123;

// 或者

const obj = Object.assign(
  Object.create(prot),
  {
    foo: 123,
  }
);
```

有了`Object.getOwnPropertyDescriptors`，我們就有了另一種寫法。

```javascript
const obj = Object.create(
  prot,
  Object.getOwnPropertyDescriptors({
    foo: 123,
  })
);
```

`Object.getOwnPropertyDescriptors`也可以用來實現 Mixin（混入）模式。

```javascript
let mix = (object) => ({
  with: (...mixins) => mixins.reduce(
    (c, mixin) => Object.create(
      c, Object.getOwnPropertyDescriptors(mixin)
    ), object)
});

// multiple mixins example
let a = {a: 'a'};
let b = {b: 'b'};
let c = {c: 'c'};
let d = mix(c).with(a, b);
```

上面程式碼中，物件`a`和`b`被混入了物件`c`。

出於完整性的考慮，`Object.getOwnPropertyDescriptors`進入標準以後，還會有`Reflect.getOwnPropertyDescriptors`方法。

## Null 傳導運算子

程式設計實務中，如果讀取物件內部的某個屬性，往往需要判斷一下該物件是否存在。比如，要讀取`message.body.user.firstName`，安全的寫法是寫成下面這樣。

```javascript
const firstName = (message
  && message.body
  && message.body.user
  && message.body.user.firstName) || 'default';
```

這樣的層層判斷非常麻煩，因此現在有一個[提案](https://github.com/claudepache/es-optional-chaining)，引入了“Null 傳導運算子”（null propagation operator）`?.`，簡化上面的寫法。

```javascript
const firstName = message?.body?.user?.firstName || 'default';
```

上面程式碼有三個`?.`運算子，只要其中一個返回`null`或`undefined`，就不再往下運算，而是返回`undefined`。

“Null 傳導運算子”有四種用法。

- `obj?.prop`  // 讀取物件屬性
- `obj?.[expr]`  // 同上
- `func?.(...args)` // 函式或物件方法的呼叫
- `new C?.(...args)`  // 建構函式的呼叫

傳導運算子之所以寫成`obj?.prop`，而不是`obj?prop`，是為了方便編譯器能夠區分三元運算子`?:`（比如`obj?prop:123`）。

下面是更多的例子。

```javascript
// 如果 a 是 null 或 undefined, 返回 undefined
// 否則返回 a.b.c().d
a?.b.c().d

// 如果 a 是 null 或 undefined，下面的語句不產生任何效果
// 否則執行 a.b = 42
a?.b = 42

// 如果 a 是 null 或 undefined，下面的語句不產生任何效果
delete a?.b
```

