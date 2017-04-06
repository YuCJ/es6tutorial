# Reflect

## 概述

`Reflect`物件與`Proxy`物件一樣，也是 ES6 為了操作物件而提供的新 API。`Reflect`物件的設計目的有這樣幾個。

（1） 將`Object`物件的一些明顯屬於語言內部的方法（比如`Object.defineProperty`），放到`Reflect`物件上。現階段，某些方法同時在`Object`和`Reflect`物件上部署，未來的新方法將只部署在`Reflect`物件上。也就是說，從`Reflect`物件上可以拿到語言內部的方法。

（2） 修改某些`Object`方法的返回結果，讓其變得更合理。比如，`Object.defineProperty(obj, name, desc)`在無法定義屬性時，會丟擲一個錯誤，而`Reflect.defineProperty(obj, name, desc)`則會返回`false`。

```javascript
// 老寫法
try {
  Object.defineProperty(target, property, attributes);
  // success
} catch (e) {
  // failure
}

// 新寫法
if (Reflect.defineProperty(target, property, attributes)) {
  // success
} else {
  // failure
}
```

（3） 讓`Object`操作都變成函式行為。某些`Object`操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`讓它們變成了函式行為。

```javascript
// 老寫法
'assign' in Object // true

// 新寫法
Reflect.has(Object, 'assign') // true
```

（4）`Reflect`物件的方法與`Proxy`物件的方法一一對應，只要是`Proxy`物件的方法，就能在`Reflect`物件上找到對應的方法。這就讓`Proxy`物件可以方便地呼叫對應的`Reflect`方法，完成預設行為，作為修改行為的基礎。也就是說，不管`Proxy`怎麼修改預設行為，你總可以在`Reflect`上獲取預設行為。

```javascript
Proxy(target, {
  set: function(target, name, value, receiver) {
    var success = Reflect.set(target,name, value, receiver);
    if (success) {
      log('property ' + name + ' on ' + target + ' set to ' + value);
    }
    return success;
  }
});
```

上面程式碼中，`Proxy`方法攔截`target`物件的屬性賦值行為。它採用`Reflect.set`方法將值賦值給物件的屬性，確保完成原有的行為，然後再部署額外的功能。

下面是另一個例子。

```javascript
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log('get', target, name);
    return Reflect.get(target, name);
  },
  deleteProperty(target, name) {
    console.log('delete' + name);
    return Reflect.deleteProperty(target, name);
  },
  has(target, name) {
    console.log('has' + name);
    return Reflect.has(target, name);
  }
});
```

上面程式碼中，每一個`Proxy`物件的攔截操作（`get`、`delete`、`has`），內部都呼叫對應的`Reflect`方法，保證原生行為能夠正常執行。新增的工作，就是將每一個操作輸出一行日誌。

有了`Reflect`物件以後，很多操作會更易讀。

```javascript
// 老寫法
Function.prototype.apply.call(Math.floor, undefined, [1.75]) // 1

// 新寫法
Reflect.apply(Math.floor, undefined, [1.75]) // 1
```

## 靜態方法

`Reflect`物件一共有13個靜態方法。

- Reflect.apply(target,thisArg,args)
- Reflect.construct(target,args)
- Reflect.get(target,name,receiver)
- Reflect.set(target,name,value,receiver)
- Reflect.defineProperty(target,name,desc)
- Reflect.deleteProperty(target,name)
- Reflect.has(target,name)
- Reflect.ownKeys(target)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)

上面這些方法的作用，大部分與`Object`物件的同名方法的作用都是相同的，而且它與`Proxy`物件的方法是一一對應的。下面是對它們的解釋。

### Reflect.get(target, name, receiver)

`Reflect.get`方法查詢並返回`target`物件的`name`屬性，如果沒有該屬性，則返回`undefined`。

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
}

Reflect.get(myObject, 'foo') // 1
Reflect.get(myObject, 'bar') // 2
Reflect.get(myObject, 'baz') // 3
```

如果`name`屬性部署了讀取函式（getter），則讀取函式的`this`繫結`receiver`。

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar;
  },
};

var myReceiverObject = {
  foo: 4,
  bar: 4,
};

Reflect.get(myObject, 'baz', myReceiverObject) // 8
```

如果第一個引數不是物件，`Reflect.get`方法會報錯。

```javascript
Reflect.get(1, 'foo') // 報錯
Reflect.get(false, 'foo') // 報錯
```

### Reflect.set(target, name, value, receiver)

`Reflect.set`方法設定`target`物件的`name`屬性等於`value`。

```javascript
var myObject = {
  foo: 1,
  set bar(value) {
    return this.foo = value;
  },
}

myObject.foo // 1

Reflect.set(myObject, 'foo', 2);
myObject.foo // 2

Reflect.set(myObject, 'bar', 3)
myObject.foo // 3
```

如果`name`屬性設定了賦值函式，則賦值函式的`this`繫結`receiver`。

```javascript
var myObject = {
  foo: 4,
  set bar(value) {
    return this.foo = value;
  },
};

var myReceiverObject = {
  foo: 0,
};

Reflect.set(myObject, 'bar', 1, myReceiverObject);
myObject.foo // 4
myReceiverObject.foo // 1
```

如果第一個引數不是物件，`Reflect.set`會報錯。

```javascript
Reflect.set(1, 'foo', {}) // 報錯
Reflect.set(false, 'foo', {}) // 報錯
```

注意，`Reflect.set`會觸發`Proxy.defineProperty`攔截。

```javascript
let p = {
  a: 'a'
};

let handler = {
  set(target,key,value,receiver) {
    console.log('set');
    Reflect.set(target,key,value,receiver)
  },
  defineProperty(target, key, attribute) {
    console.log('defineProperty');
    Reflect.defineProperty(target,key,attribute);
  }
};

let obj = new Proxy(p, handler);
obj.a = 'A';
// set
// defineProperty
```

上面程式碼中，`Proxy.set`攔截中使用了`Reflect.set`，導致觸發`Proxy.defineProperty`攔截。

### Reflect.has(obj, name)

`Reflect.has`方法對應`name in obj`裡面的`in`運算子。

```javascript
var myObject = {
  foo: 1,
};

// 舊寫法
'foo' in myObject // true

// 新寫法
Reflect.has(myObject, 'foo') // true
```

如果第一個引數不是物件，`Reflect.has`和`in`運算子都會報錯。

### Reflect.deleteProperty(obj, name)

`Reflect.deleteProperty`方法等同於`delete obj[name]`，用於刪除物件的屬性。

```javascript
const myObj = { foo: 'bar' };

// 舊寫法
delete myObj.foo;

// 新寫法
Reflect.deleteProperty(myObj, 'foo');
```

該方法返回一個布林值。如果刪除成功，或者被刪除的屬性不存在，返回`true`；刪除失敗，被刪除的屬性依然存在，返回`false`。

### Reflect.construct(target, args)

`Reflect.construct`方法等同於`new target(...args)`，這提供了一種不使用`new`，來呼叫建構函式的方法。

```javascript
function Greeting(name) {
  this.name = name;
}

// new 的寫法
const instance = new Greeting('張三');

// Reflect.construct 的寫法
const instance = Reflect.construct(Greeting, ['張三']);
```

### Reflect.getPrototypeOf(obj)

`Reflect.getPrototypeOf`方法用於讀取物件的`__proto__`屬性，對應`Object.getPrototypeOf(obj)`。

```javascript
const myObj = new FancyThing();

// 舊寫法
Object.getPrototypeOf(myObj) === FancyThing.prototype;

// 新寫法
Reflect.getPrototypeOf(myObj) === FancyThing.prototype;
```

`Reflect.getPrototypeOf`和`Object.getPrototypeOf`的一個區別是，如果引數不是物件，`Object.getPrototypeOf`會將這個引數轉為物件，然後再執行，而`Reflect.getPrototypeOf`會報錯。

```javascript
Object.getPrototypeOf(1) // Number {[[PrimitiveValue]]: 0}
Reflect.getPrototypeOf(1) // 報錯
```

### Reflect.setPrototypeOf(obj, newProto)

`Reflect.setPrototypeOf`方法用於設定物件的`__proto__`屬性，返回第一個引數物件，對應`Object.setPrototypeOf(obj, newProto)`。

```javascript
const myObj = new FancyThing();

// 舊寫法
Object.setPrototypeOf(myObj, OtherThing.prototype);

// 新寫法
Reflect.setPrototypeOf(myObj, OtherThing.prototype);
```

如果第一個引數不是物件，`Object.setPrototypeOf`會返回第一個引數本身，而`Reflect.setPrototypeOf`會報錯。

```javascript
Object.setPrototypeOf(1, {})
// 1

Reflect.setPrototypeOf(1, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

如果第一個引數是`undefined`或`null`，`Object.setPrototypeOf`和`Reflect.setPrototypeOf`都會報錯。

```javascript
Object.setPrototypeOf(null, {})
// TypeError: Object.setPrototypeOf called on null or undefined

Reflect.setPrototypeOf(null, {})
// TypeError: Reflect.setPrototypeOf called on non-object
```

### Reflect.apply(func, thisArg, args)

`Reflect.apply`方法等同於`Function.prototype.apply.call(func, thisArg, args)`，用於繫結`this`物件後執行給定函式。

一般來說，如果要繫結一個函式的`this`物件，可以這樣寫`fn.apply(obj, args)`，但是如果函式定義了自己的`apply`方法，就只能寫成`Function.prototype.apply.call(fn, obj, args)`，採用`Reflect`物件可以簡化這種操作。

```javascript
const ages = [11, 33, 12, 54, 18, 96];

// 舊寫法
const youngest = Math.min.apply(Math, ages);
const oldest = Math.max.apply(Math, ages);
const type = Object.prototype.toString.call(youngest);

// 新寫法
const youngest = Reflect.apply(Math.min, Math, ages);
const oldest = Reflect.apply(Math.max, Math, ages);
const type = Reflect.apply(Object.prototype.toString, youngest, []);
```

### Reflect.defineProperty(target, propertyKey, attributes)

`Reflect.defineProperty`方法基本等同於`Object.defineProperty`，用來為物件定義屬性。未來，後者會被逐漸廢除，請從現在開始就使用`Reflect.defineProperty`代替它。

```javascript
function MyDate() {
  /*…*/
}

// 舊寫法
Object.defineProperty(MyDate, 'now', {
  value: () => new Date.now()
});

// 新寫法
Reflect.defineProperty(MyDate, 'now', {
  value: () => new Date.now()
});
```

如果`Reflect.defineProperty`的第一個引數不是物件，就會丟擲錯誤，比如`Reflect.defineProperty(1, 'foo')`。

### Reflect.getOwnPropertyDescriptor(target, propertyKey)

`Reflect.getOwnPropertyDescriptor`基本等同於`Object.getOwnPropertyDescriptor`，用於得到指定屬性的描述物件，將來會替代掉後者。

```javascript
var myObject = {};
Object.defineProperty(myObject, 'hidden', {
  value: true,
  enumerable: false,
});

// 舊寫法
var theDescriptor = Object.getOwnPropertyDescriptor(myObject, 'hidden');

// 新寫法
var theDescriptor = Reflect.getOwnPropertyDescriptor(myObject, 'hidden');
```

`Reflect.getOwnPropertyDescriptor`和`Object.getOwnPropertyDescriptor`的一個區別是，如果第一個引數不是物件，`Object.getOwnPropertyDescriptor(1, 'foo')`不報錯，返回`undefined`，而`Reflect.getOwnPropertyDescriptor(1, 'foo')`會丟擲錯誤，表示引數非法。

### Reflect.isExtensible (target)

`Reflect.isExtensible`方法對應`Object.isExtensible`，返回一個布林值，表示當前物件是否可擴展。

```javascript
const myObject = {};

// 舊寫法
Object.isExtensible(myObject) // true

// 新寫法
Reflect.isExtensible(myObject) // true
```

如果引數不是物件，`Object.isExtensible`會返回`false`，因為非物件本來就是不可擴展的，而`Reflect.isExtensible`會報錯。

```javascript
Object.isExtensible(1) // false
Reflect.isExtensible(1) // 報錯
```

### Reflect.preventExtensions(target)

`Reflect.preventExtensions`對應`Object.preventExtensions`方法，用於讓一個物件變為不可擴展。它返回一個布林值，表示是否操作成功。

```javascript
var myObject = {};

// 舊寫法
Object.isExtensible(myObject) // true

// 新寫法
Reflect.preventExtensions(myObject) // true
```

如果引數不是物件，`Object.isExtensible`在 ES5 環境報錯，在 ES6 環境返回這個引數，而`Reflect.preventExtensions`會報錯。

```javascript
// ES5
Object.preventExtensions(1) // 報錯

// ES6
Object.preventExtensions(1) // 1

// 新寫法
Reflect.preventExtensions(1) // 報錯
```

### Reflect.ownKeys (target)

`Reflect.ownKeys`方法用於返回物件的所有屬性，基本等同於`Object.getOwnPropertyNames`與`Object.getOwnPropertySymbols`之和。

```javascript
var myObject = {
  foo: 1,
  bar: 2,
  [Symbol.for('baz')]: 3,
  [Symbol.for('bing')]: 4,
};

// 舊寫法
Object.getOwnPropertyNames(myObject)
// ['foo', 'bar']

Object.getOwnPropertySymbols(myObject)
//[Symbol.for('baz'), Symbol.for('bing')]

// 新寫法
Reflect.ownKeys(myObject)
// ['foo', 'bar', Symbol.for('baz'), Symbol.for('bing')]
```

## 實例：使用 Proxy 實現觀察者模式

觀察者模式（Observer mode）指的是函式自動觀察資料物件，一旦物件有變化，函式就會自動執行。

```javascript
const person = observable({
  name: '張三',
  age: 20
});

function print() {
  console.log(`${person.name}, ${person.age}`)
}

observe(print);
person.name = '李四';
// 輸出
// 李四, 20
```

上面程式碼中，資料物件`person`是觀察目標，函式`print`是觀察者。一旦資料物件發生變化，`print`就會自動執行。

下面，使用 Proxy 寫一個觀察者模式的最簡單實現，即實現`observable`和`observe`這兩個函式。思路是`observable`函式返回一個原始物件的 Proxy 代理，攔截賦值操作，觸發充當觀察者的各個函式。

```javascript
const queuedObservers = new Set();

const observe = fn => queuedObservers.add(fn);
const observable = obj => new Proxy(obj, {set});

function set(target, key, value, receiver) {
  const result = Reflect.set(target, key, value, receiver);
  queuedObservers.forEach(observer => observer());
  return result;
}
```

上面程式碼中，先定義了一個`Set`集合，所有觀察者函式都放進這個集合。然後，`observable`函式返回原始物件的代理，攔截賦值操作。攔截函式`set`之中，會自動執行所有觀察者。

