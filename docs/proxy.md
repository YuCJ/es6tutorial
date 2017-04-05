# Proxy

## 概述

Proxy 用於修改某些操作的預設行為，等同於在語言層面做出修改，所以屬於一種“超程式設計”（meta programming），即對程式語言進行程式設計。

Proxy 可以理解成，在目標物件之前架設一層“攔截”，外界對該物件的訪問，都必須先通過這層攔截，因此提供了一種機制，可以對外界的訪問進行過濾和改寫。Proxy 這個詞的原意是代理，用在這裡表示由它來“代理”某些操作，可以譯為“代理器”。

```javascript
var obj = new Proxy({}, {
  get: function (target, key, receiver) {
    console.log(`getting ${key}!`);
    return Reflect.get(target, key, receiver);
  },
  set: function (target, key, value, receiver) {
    console.log(`setting ${key}!`);
    return Reflect.set(target, key, value, receiver);
  }
});
```

上面程式碼對一個空物件架設了一層攔截，重定義了屬性的讀取（`get`）和設定（`set`）行為。這裡暫時先不解釋具體的語法，只看執行結果。對設定了攔截行為的物件`obj`，去讀寫它的屬性，就會得到下面的結果。

```javascript
obj.count = 1
//  setting count!
++obj.count
//  getting count!
//  setting count!
//  2
```

上面程式碼說明，Proxy 實際上過載（overload）了點運算子，即用自己的定義覆蓋了語言的原始定義。

ES6 原生提供 Proxy 建構函式，用來生成 Proxy 例項。

```javascript
var proxy = new Proxy(target, handler);
```

Proxy 物件的所有用法，都是上面這種形式，不同的只是`handler`引數的寫法。其中，`new Proxy()`表示生成一個`Proxy`例項，`target`引數表示所要攔截的目標物件，`handler`引數也是一個物件，用來定製攔截行為。

下面是另一個攔截讀取屬性行為的例子。

```javascript
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

上面程式碼中，作為建構函式，`Proxy`接受兩個引數。第一個引數是所要代理的目標物件（上例是一個空物件），即如果沒有`Proxy`的介入，操作原來要訪問的就是這個物件；第二個引數是一個配置物件，對於每一個被代理的操作，需要提供一個對應的處理函式，該函式將攔截對應的操作。比如，上面程式碼中，配置物件有一個`get`方法，用來攔截對目標物件屬性的訪問請求。`get`方法的兩個引數分別是目標物件和所要訪問的屬性。可以看到，由於攔截函式總是返回`35`，所以訪問任何屬性都得到`35`。

注意，要使得`Proxy`起作用，必須針對`Proxy`例項（上例是`proxy`物件）進行操作，而不是針對目標物件（上例是空物件）進行操作。

如果`handler`沒有設定任何攔截，那就等同於直接通向原物件。

```javascript
var target = {};
var handler = {};
var proxy = new Proxy(target, handler);
proxy.a = 'b';
target.a // "b"
```

上面程式碼中，`handler`是一個空物件，沒有任何攔截效果，訪問`proxy`就等同於訪問`target`。

一個技巧是將 Proxy 物件，設定到`object.proxy`屬性，從而可以在`object`物件上呼叫。

```javascript
var object = { proxy: new Proxy(target, handler) };
```

Proxy 例項也可以作為其他物件的原型物件。

```javascript
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

let obj = Object.create(proxy);
obj.time // 35
```

上面程式碼中，`proxy`物件是`obj`物件的原型，`obj`物件本身並沒有`time`屬性，所以根據原型鏈，會在`proxy`物件上讀取該屬性，導致被攔截。

同一個攔截器函式，可以設定攔截多個操作。

```javascript
var handler = {
  get: function(target, name) {
    if (name === 'prototype') {
      return Object.prototype;
    }
    return 'Hello, ' + name;
  },

  apply: function(target, thisBinding, args) {
    return args[0];
  },

  construct: function(target, args) {
    return {value: args[1]};
  }
};

var fproxy = new Proxy(function(x, y) {
  return x + y;
}, handler);

fproxy(1, 2) // 1
new fproxy(1,2) // {value: 2}
fproxy.prototype === Object.prototype // true
fproxy.foo // "Hello, foo"
```

下面是 Proxy 支援的攔截操作一覽。

對於可以設定、但沒有設定攔截的操作，則直接落在目標物件上，按照原先的方式產生結果。

**（1）get(target, propKey, receiver)**

攔截物件屬性的讀取，比如`proxy.foo`和`proxy['foo']`。

最後一個引數`receiver`是一個物件，可選，參見下面`Reflect.get`的部分。

**（2）set(target, propKey, value, receiver)**

攔截物件屬性的設定，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一個布林值。

**（3）has(target, propKey)**

攔截`propKey in proxy`的操作，返回一個布林值。

**（4）deleteProperty(target, propKey)**

攔截`delete proxy[propKey]`的操作，返回一個布林值。

**（5）ownKeys(target)**

攔截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`，返回一個數組。該方法返回目標物件所有自身的屬性的屬性名，而`Object.keys()`的返回結果僅包括目標物件自身的可遍歷屬性。

**（6）getOwnPropertyDescriptor(target, propKey)**

攔截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回屬性的描述物件。

**（7）defineProperty(target, propKey, propDesc)**

攔截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一個布林值。

**（8）preventExtensions(target)**

攔截`Object.preventExtensions(proxy)`，返回一個布林值。

**（9）getPrototypeOf(target)**

攔截`Object.getPrototypeOf(proxy)`，返回一個物件。

**（10）isExtensible(target)**

攔截`Object.isExtensible(proxy)`，返回一個布林值。

**（11）setPrototypeOf(target, proto)**

攔截`Object.setPrototypeOf(proxy, proto)`，返回一個布林值。

如果目標物件是函式，那麼還有兩種額外操作可以攔截。

**（12）apply(target, object, args)**

攔截 Proxy 例項作為函式呼叫的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。

**（13）construct(target, args)**

攔截 Proxy 例項作為建構函式呼叫的操作，比如`new proxy(...args)`。

## Proxy 例項的方法

下面是上面這些攔截方法的詳細介紹。

### get()

`get`方法用於攔截某個屬性的讀取操作。上文已經有一個例子，下面是另一個攔截讀取操作的例子。

```javascript
var person = {
  name: "張三"
};

var proxy = new Proxy(person, {
  get: function(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError("Property \"" + property + "\" does not exist.");
    }
  }
});

proxy.name // "張三"
proxy.age // 丟擲一個錯誤
```

上面程式碼表示，如果訪問目標物件不存在的屬性，會丟擲一個錯誤。如果沒有這個攔截函式，訪問不存在的屬性，只會返回`undefined`。

`get`方法可以繼承。

```javascript
let proto = new Proxy({}, {
  get(target, propertyKey, receiver) {
    console.log('GET '+propertyKey);
    return target[propertyKey];
  }
});

let obj = Object.create(proto);
obj.xxx // "GET xxx"
```

上面程式碼中，攔截操作定義在`Prototype`物件上面，所以如果讀取`obj`物件繼承的屬性時，攔截會生效。

下面的例子使用`get`攔截，實現陣列讀取負數的索引。

```javascript
function createArray(...elements) {
  let handler = {
    get(target, propKey, receiver) {
      let index = Number(propKey);
      if (index < 0) {
        propKey = String(target.length + index);
      }
      return Reflect.get(target, propKey, receiver);
    }
  };

  let target = [];
  target.push(...elements);
  return new Proxy(target, handler);
}

let arr = createArray('a', 'b', 'c');
arr[-1] // c
```

上面程式碼中，陣列的位置引數是`-1`，就會輸出陣列的倒數最後一個成員。

利用 Proxy，可以將讀取屬性的操作（`get`），轉變為執行某個函式，從而實現屬性的鏈式操作。

```javascript
var pipe = (function () {
  return function (value) {
    var funcStack = [];
    var oproxy = new Proxy({} , {
      get : function (pipeObject, fnName) {
        if (fnName === 'get') {
          return funcStack.reduce(function (val, fn) {
            return fn(val);
          },value);
        }
        funcStack.push(window[fnName]);
        return oproxy;
      }
    });

    return oproxy;
  }
}());

var double = n => n * 2;
var pow    = n => n * n;
var reverseInt = n => n.toString().split("").reverse().join("") | 0;

pipe(3).double.pow.reverseInt.get; // 63
```

上面程式碼設定 Proxy 以後，達到了將函式名鏈式使用的效果。

下面的例子則是利用`get`攔截，實現一個生成各種DOM節點的通用函式`dom`。

```javascript
const dom = new Proxy({}, {
  get(target, property) {
    return function(attrs = {}, ...children) {
      const el = document.createElement(property);
      for (let prop of Object.keys(attrs)) {
        el.setAttribute(prop, attrs[prop]);
      }
      for (let child of children) {
        if (typeof child === 'string') {
          child = document.createTextNode(child);
        }
        el.appendChild(child);
      }
      return el;
    }
  }
});

const el = dom.div({},
  'Hello, my name is ',
  dom.a({href: '//example.com'}, 'Mark'),
  '. I like:',
  dom.ul({},
    dom.li({}, 'The web'),
    dom.li({}, 'Food'),
    dom.li({}, '…actually that\'s it')
  )
);

document.body.appendChild(el);
```

如果一個屬性不可配置（configurable）和不可寫（writable），則該屬性不能被代理，通過 Proxy 物件訪問該屬性會報錯。

```javascript
const target = Object.defineProperties({}, {
  foo: {
    value: 123,
    writable: false,
    configurable: false
  },
});

const handler = {
  get(target, propKey) {
    return 'abc';
  }
};

const proxy = new Proxy(target, handler);

proxy.foo
// TypeError: Invariant check failed
```

### set()

`set`方法用來攔截某個屬性的賦值操作。

假定`Person`物件有一個`age`屬性，該屬性應該是一個不大於200的整數，那麼可以使用`Proxy`保證`age`的屬性值符合要求。

```javascript
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // 對於age以外的屬性，直接儲存
    obj[prop] = value;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

person.age // 100
person.age = 'young' // 報錯
person.age = 300 // 報錯
```

上面程式碼中，由於設定了存值函式`set`，任何不符合要求的`age`屬性賦值，都會丟擲一個錯誤，這是資料驗證的一種實現方法。利用`set`方法，還可以資料繫結，即每當物件發生變化時，會自動更新 DOM。

有時，我們會在物件上面設定內部屬性，屬性名的第一個字元使用下劃線開頭，表示這些屬性不應該被外部使用。結合`get`和`set`方法，就可以做到防止這些內部屬性被外部讀寫。

```javascript
var handler = {
  get (target, key) {
    invariant(key, 'get');
    return target[key];
  },
  set (target, key, value) {
    invariant(key, 'set');
    target[key] = value;
    return true;
  }
};
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}
var target = {};
var proxy = new Proxy(target, handler);
proxy._prop
// Error: Invalid attempt to get private "_prop" property
proxy._prop = 'c'
// Error: Invalid attempt to set private "_prop" property
```

上面程式碼中，只要讀寫的屬性名的第一個字元是下劃線，一律拋錯，從而達到禁止讀寫內部屬性的目的。

注意，如果目標物件自身的某個屬性，不可寫也不可配置，那麼`set`不得改變這個屬性的值，只能返回同樣的值，否則報錯。

### apply()

`apply`方法攔截函式的呼叫、`call`和`apply`操作。

`apply`方法可以接受三個引數，分別是目標物件、目標物件的上下文物件（`this`）和目標物件的引數陣列。

```javascript
var handler = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments);
  }
};
```

下面是一個例子。

```javascript
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);

p()
// "I am the proxy"
```

上面程式碼中，變數`p`是 Proxy 的例項，當它作為函式呼叫時（`p()`），就會被`apply`方法攔截，返回一個字串。

下面是另外一個例子。

```javascript
var twice = {
  apply (target, ctx, args) {
    return Reflect.apply(...arguments) * 2;
  }
};
function sum (left, right) {
  return left + right;
};
var proxy = new Proxy(sum, twice);
proxy(1, 2) // 6
proxy.call(null, 5, 6) // 22
proxy.apply(null, [7, 8]) // 30
```

上面程式碼中，每當執行`proxy`函式（直接呼叫或`call`和`apply`呼叫），就會被`apply`方法攔截。

另外，直接呼叫`Reflect.apply`方法，也會被攔截。

```javascript
Reflect.apply(proxy, null, [9, 10]) // 38
```

### has()

`has`方法用來攔截`HasProperty`操作，即判斷物件是否具有某個屬性時，這個方法會生效。典型的操作就是`in`運算子。

下面的例子使用`has`方法隱藏某些屬性，不被`in`運算子發現。

```javascript
var handler = {
  has (target, key) {
    if (key[0] === '_') {
      return false;
    }
    return key in target;
  }
};
var target = { _prop: 'foo', prop: 'foo' };
var proxy = new Proxy(target, handler);
'_prop' in proxy // false
```

上面程式碼中，如果原物件的屬性名的第一個字元是下劃線，`proxy.has`就會返回`false`，從而不會被`in`運算子發現。

如果原物件不可配置或者禁止擴充套件，這時`has`攔截會報錯。

```javascript
var obj = { a: 10 };
Object.preventExtensions(obj);

var p = new Proxy(obj, {
  has: function(target, prop) {
    return false;
  }
});

'a' in p // TypeError is thrown
```

上面程式碼中，`obj`物件禁止擴充套件，結果使用`has`攔截就會報錯。也就是說，如果某個屬性不可配置（或者目標物件不可擴充套件），則`has`方法就不得“隱藏”（即返回`false`）目標物件的該屬性。

值得注意的是，`has`方法攔截的是`HasProperty`操作，而不是`HasOwnProperty`操作，即`has`方法不判斷一個屬性是物件自身的屬性，還是繼承的屬性。

另外，雖然`for...in`迴圈也用到了`in`運算子，但是`has`攔截對`for...in`迴圈不生效。

```javascript
let stu1 = {name: '張三', score: 59};
let stu2 = {name: '李四', score: 99};

let handler = {
  has(target, prop) {
    if (prop === 'score' && target[prop] < 60) {
      console.log(`${target.name} 不及格`);
      return false;
    }
    return prop in target;
  }
}

let oproxy1 = new Proxy(stu1, handler);
let oproxy2 = new Proxy(stu2, handler);

'score' in oproxy1
// 張三 不及格
// false

'score' in oproxy2
// true

for (let a in oproxy1) {
  console.log(oproxy1[a]);
}
// 張三
// 59

for (let b in oproxy2) {
  console.log(oproxy2[b]);
}
// 李四
// 99
```

上面程式碼中，`has`攔截只對`in`迴圈生效，對`for...in`迴圈不生效，導致不符合要求的屬性沒有被排除在`for...in`迴圈之外。

### construct()

`construct`方法用於攔截`new`命令，下面是攔截物件的寫法。

```javascript
var handler = {
  construct (target, args, newTarget) {
    return new target(...args);
  }
};
```

`construct`方法可以接受兩個引數。

- `target`: 目標物件
- `args`：構建函式的引數物件

下面是一個例子。

```javascript
var p = new Proxy(function () {}, {
  construct: function(target, args) {
    console.log('called: ' + args.join(', '));
    return { value: args[0] * 10 };
  }
});

(new p(1)).value
// "called: 1"
// 10
```

`construct`方法返回的必須是一個物件，否則會報錯。

```javascript
var p = new Proxy(function() {}, {
  construct: function(target, argumentsList) {
    return 1;
  }
});

new p() // 報錯
```

### deleteProperty()

`deleteProperty`方法用於攔截`delete`操作，如果這個方法丟擲錯誤或者返回`false`，當前屬性就無法被`delete`命令刪除。

```javascript
var handler = {
  deleteProperty (target, key) {
    invariant(key, 'delete');
    return true;
  }
};
function invariant (key, action) {
  if (key[0] === '_') {
    throw new Error(`Invalid attempt to ${action} private "${key}" property`);
  }
}

var target = { _prop: 'foo' };
var proxy = new Proxy(target, handler);
delete proxy._prop
// Error: Invalid attempt to delete private "_prop" property
```

上面程式碼中，`deleteProperty`方法攔截了`delete`操作符，刪除第一個字元為下劃線的屬性會報錯。

注意，目標物件自身的不可配置（configurable）的屬性，不能被`deleteProperty`方法刪除，否則報錯。

### defineProperty()

`defineProperty`方法攔截了`Object.defineProperty`操作。

```javascript
var handler = {
  defineProperty (target, key, descriptor) {
    return false;
  }
};
var target = {};
var proxy = new Proxy(target, handler);
proxy.foo = 'bar'
// TypeError: proxy defineProperty handler returned false for property '"foo"'
```

上面程式碼中，`defineProperty`方法返回`false`，導致新增新屬性會丟擲錯誤。

注意，如果目標物件不可擴充套件（extensible），則`defineProperty`不能增加目標物件上不存在的屬性，否則會報錯。另外，如果目標物件的某個屬性不可寫（writable）或不可配置（configurable），則`defineProperty`方法不得改變這兩個設定。

### getOwnPropertyDescriptor()

`getOwnPropertyDescriptor`方法攔截`Object.getOwnPropertyDescriptor()`，返回一個屬性描述物件或者`undefined`。

```javascript
var handler = {
  getOwnPropertyDescriptor (target, key) {
    if (key[0] === '_') {
      return;
    }
    return Object.getOwnPropertyDescriptor(target, key);
  }
};
var target = { _foo: 'bar', baz: 'tar' };
var proxy = new Proxy(target, handler);
Object.getOwnPropertyDescriptor(proxy, 'wat')
// undefined
Object.getOwnPropertyDescriptor(proxy, '_foo')
// undefined
Object.getOwnPropertyDescriptor(proxy, 'baz')
// { value: 'tar', writable: true, enumerable: true, configurable: true }
```

上面程式碼中，`handler.getOwnPropertyDescriptor`方法對於第一個字元為下劃線的屬性名會返回`undefined`。

### getPrototypeOf()

`getPrototypeOf`方法主要用來攔截獲取物件原型。具體來說，攔截下面這些操作。

- `Object.prototype.__proto__`
- `Object.prototype.isPrototypeOf()`
- `Object.getPrototypeOf()`
- `Reflect.getPrototypeOf()`
- `instanceof`

下面是一個例子。

```javascript
var proto = {};
var p = new Proxy({}, {
  getPrototypeOf(target) {
    return proto;
  }
});
Object.getPrototypeOf(p) === proto // true
```

上面程式碼中，`getPrototypeOf`方法攔截`Object.getPrototypeOf()`，返回`proto`物件。

注意，`getPrototypeOf`方法的返回值必須是物件或者`null`，否則報錯。另外，如果目標物件不可擴充套件（extensible）， `getPrototypeOf`方法必須返回目標物件的原型物件。

### isExtensible()

`isExtensible`方法攔截`Object.isExtensible`操作。

```javascript
var p = new Proxy({}, {
  isExtensible: function(target) {
    console.log("called");
    return true;
  }
});

Object.isExtensible(p)
// "called"
// true
```

上面程式碼設定了`isExtensible`方法，在呼叫`Object.isExtensible`時會輸出`called`。

注意，該方法只能返回布林值，否則返回值會被自動轉為布林值。

這個方法有一個強限制，它的返回值必須與目標物件的`isExtensible`屬性保持一致，否則就會丟擲錯誤。

```javascript
Object.isExtensible(proxy) === Object.isExtensible(target)
```

下面是一個例子。

```javascript
var p = new Proxy({}, {
  isExtensible: function(target) {
    return false;
  }
});

Object.isExtensible(p) // 報錯
```

### ownKeys()

`ownKeys`方法用來攔截物件自身屬性的讀取操作。具體來說，攔截以下操作。

- `Object.getOwnPropertyNames()`
- `Object.getOwnPropertySymbols()`
- `Object.keys()`

下面是攔截`Object.keys()`的例子。

```javascript
let target = {
  a: 1,
  b: 2,
  c: 3
};

let handler = {
  ownKeys(target) {
    return ['a'];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy)
// [ 'a' ]
```

上面程式碼攔截了對於`target`物件的`Object.keys()`操作，只返回`a`、`b`、`c`三個屬性之中的`a`屬性。

下面的例子是攔截第一個字元為下劃線的屬性名。

```javascript
let target = {
  _bar: 'foo',
  _prop: 'bar',
  prop: 'baz'
};

let handler = {
  ownKeys (target) {
    return Reflect.ownKeys(target).filter(key => key[0] !== '_');
  }
};

let proxy = new Proxy(target, handler);
for (let key of Object.keys(proxy)) {
  console.log(target[key]);
}
// "baz"
```

注意，使用`Object.keys`方法時，有三類屬性會被`ownKeys`方法自動過濾，不會返回。

- 目標物件上不存在的屬性
- 屬性名為 Symbol 值
- 不可遍歷（`enumerable`）的屬性

```javascript
let target = {
  a: 1,
  b: 2,
  c: 3,
  [Symbol.for('secret')]: '4',
};

Object.defineProperty(target, 'key', {
  enumerable: false,
  configurable: true,
  writable: true,
  value: 'static'
});

let handler = {
  ownKeys(target) {
    return ['a', 'd', Symbol.for('secret'), 'key'];
  }
};

let proxy = new Proxy(target, handler);

Object.keys(proxy)
// ['a']
```

上面程式碼中，`ownKeys`方法之中，顯式返回不存在的屬性（`d`）、Symbol 值（`Symbol.for('secret')`）、不可遍歷的屬性（`key`），結果都被自動過濾掉。

`ownKeys`方法還可以攔截`Object.getOwnPropertyNames()`。

```javascript
var p = new Proxy({}, {
  ownKeys: function(target) {
    return ['a', 'b', 'c'];
  }
});

Object.getOwnPropertyNames(p)
// [ 'a', 'b', 'c' ]
```

`ownKeys`方法返回的陣列成員，只能是字串或 Symbol 值。如果有其他型別的值，或者返回的根本不是陣列，就會報錯。

```javascript
var obj = {};

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return [123, true, undefined, null, {}, []];
  }
});

Object.getOwnPropertyNames(p)
// Uncaught TypeError: 123 is not a valid property name
```

上面程式碼中，`ownKeys`方法雖然返回一個數組，但是每一個數組成員都不是字串或 Symbol 值，因此就報錯了。

如果目標物件自身包含不可配置的屬性，則該屬性必須被`ownKeys`方法返回，否則報錯。

```javascript
var obj = {};
Object.defineProperty(obj, 'a', {
  configurable: false,
  enumerable: true,
  value: 10 }
);

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return ['b'];
  }
});

Object.getOwnPropertyNames(p)
// Uncaught TypeError: 'ownKeys' on proxy: trap result did not include 'a'
```

上面程式碼中，`obj`物件的`a`屬性是不可配置的，這時`ownKeys`方法返回的陣列之中，必須包含`a`，否則會報錯。

另外，如果目標物件是不可擴充套件的（non-extensition），這時`ownKeys`方法返回的陣列之中，必須包含原物件的所有屬性，且不能包含多餘的屬性，否則報錯。

```javascript
var obj = {
  a: 1
};

Object.preventExtensions(obj);

var p = new Proxy(obj, {
  ownKeys: function(target) {
    return ['a', 'b'];
  }
});

Object.getOwnPropertyNames(p)
// Uncaught TypeError: 'ownKeys' on proxy: trap returned extra keys but proxy target is non-extensible
```

上面程式碼中，`Obj`物件是不可擴充套件的，這時`ownKeys`方法返回的陣列之中，包含了`obj`物件的多餘屬性`b`，所以導致了報錯。

### preventExtensions()

`preventExtensions`方法攔截`Object.preventExtensions()`。該方法必須返回一個布林值，否則會被自動轉為布林值。

這個方法有一個限制，只有目標物件不可擴充套件時（即`Object.isExtensible(proxy)`為`false`），`proxy.preventExtensions`才能返回`true`，否則會報錯。

```javascript
var p = new Proxy({}, {
  preventExtensions: function(target) {
    return true;
  }
});

Object.preventExtensions(p) // 報錯
```

上面程式碼中，`proxy.preventExtensions`方法返回`true`，但這時`Object.isExtensible(proxy)`會返回`true`，因此報錯。

為了防止出現這個問題，通常要在`proxy.preventExtensions`方法裡面，呼叫一次`Object.preventExtensions`。

```javascript
var p = new Proxy({}, {
  preventExtensions: function(target) {
    console.log('called');
    Object.preventExtensions(target);
    return true;
  }
});

Object.preventExtensions(p)
// "called"
// true
```

### setPrototypeOf()

`setPrototypeOf`方法主要用來攔截`Object.setPrototypeOf`方法。

下面是一個例子。

```javascript
var handler = {
  setPrototypeOf (target, proto) {
    throw new Error('Changing the prototype is forbidden');
  }
};
var proto = {};
var target = function () {};
var proxy = new Proxy(target, handler);
Object.setPrototypeOf(proxy, proto);
// Error: Changing the prototype is forbidden
```

上面程式碼中，只要修改`target`的原型物件，就會報錯。

注意，該方法只能返回布林值，否則會被自動轉為布林值。另外，如果目標物件不可擴充套件（extensible），`setPrototypeOf`方法不得改變目標物件的原型。

## Proxy.revocable()

`Proxy.revocable`方法返回一個可取消的 Proxy 例項。

```javascript
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

`Proxy.revocable`方法返回一個物件，該物件的`proxy`屬性是`Proxy`例項，`revoke`屬性是一個函式，可以取消`Proxy`例項。上面程式碼中，當執行`revoke`函式之後，再訪問`Proxy`例項，就會丟擲一個錯誤。

`Proxy.revocable`的一個使用場景是，目標物件不允許直接訪問，必須通過代理訪問，一旦訪問結束，就收回代理權，不允許再次訪問。

## this 問題

雖然 Proxy 可以代理針對目標物件的訪問，但它不是目標物件的透明代理，即不做任何攔截的情況下，也無法保證與目標物件的行為一致。主要原因就是在 Proxy 代理的情況下，目標物件內部的`this`關鍵字會指向 Proxy 代理。

```javascript
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m() // false
proxy.m()  // true
```

上面程式碼中，一旦`proxy`代理`target.m`，後者內部的`this`就是指向`proxy`，而不是`target`。

下面是一個例子，由於`this`指向的變化，導致 Proxy 無法代理目標物件。

```javascript
const _name = new WeakMap();

class Person {
  constructor(name) {
    _name.set(this, name);
  }
  get name() {
    return _name.get(this);
  }
}

const jane = new Person('Jane');
jane.name // 'Jane'

const proxy = new Proxy(jane, {});
proxy.name // undefined
```

上面程式碼中，目標物件`jane`的`name`屬性，實際儲存在外部`WeakMap`物件`_name`上面，通過`this`鍵區分。由於通過`proxy.name`訪問時，`this`指向`proxy`，導致無法取到值，所以返回`undefined`。

此外，有些原生物件的內部屬性，只有通過正確的`this`才能拿到，所以 Proxy 也無法代理這些原生物件的屬性。

```javascript
const target = new Date();
const handler = {};
const proxy = new Proxy(target, handler);

proxy.getDate();
// TypeError: this is not a Date object.
```

上面程式碼中，`getDate`方法只能在`Date`物件例項上面拿到，如果`this`不是`Date`物件例項就會報錯。這時，`this`繫結原始物件，就可以解決這個問題。

```javascript
const target = new Date('2015-01-01');
const handler = {
  get(target, prop) {
    if (prop === 'getDate') {
      return target.getDate.bind(target);
    }
    return Reflect.get(target, prop);
  }
};
const proxy = new Proxy(target, handler);

proxy.getDate() // 1
```

## 例項：Web 服務的客戶端

Proxy 物件可以攔截目標物件的任意屬性，這使得它很合適用來寫 Web 服務的客戶端。

```javascript
const service = createWebService('http://example.com/data');

service.employees().then(json => {
  const employees = JSON.parse(json);
  // ···
});
```

上面程式碼新建了一個 Web 服務的介面，這個介面返回各種資料。Proxy 可以攔截這個物件的任意屬性，所以不用為每一種資料寫一個適配方法，只要寫一個 Proxy 攔截就可以了。

```javascript
function createWebService(baseUrl) {
  return new Proxy({}, {
    get(target, propKey, receiver) {
      return () => httpGet(baseUrl+'/' + propKey);
    }
  });
}
```

同理，Proxy 也可以用來實現資料庫的 ORM 層。

