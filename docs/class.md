# Class

## Class基本語法

### 概述

JavaScript語言的傳統方法是通過建構函式，定義並生成新物件。下面是一個例子。

```javascript
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);
```

上面這種寫法跟傳統的面嚮物件語言（比如C++和Java）差異很大，很容易讓新學習這門語言的程式設計師感到困惑。

ES6提供了更接近傳統語言的寫法，引入了Class（類）這個概念，作為物件的模板。通過`class`關鍵字，可以定義類。基本上，ES6的`class`可以看作只是一個語法糖，它的絕大部分功能，ES5都可以做到，新的`class`寫法只是讓物件原型的寫法更加清晰、更像面向物件程式設計的語法而已。上面的程式碼用ES6的“類”改寫，就是下面這樣。

```javascript
//定義類
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
```

上面程式碼定義了一個“類”，可以看到裡面有一個`constructor`方法，這就是構造方法，而`this`關鍵字則代表實例物件。也就是說，ES5的建構函式`Point`，對應ES6的`Point`類的構造方法。

`Point`類除了構造方法，還定義了一個`toString`方法。注意，定義“類”的方法的時候，前面不需要加上`function`這個關鍵字，直接把函式定義放進去了就可以了。另外，方法之間不需要逗號分隔，加了會報錯。

ES6的類，完全可以看作建構函式的另一種寫法。

```javascript
class Point {
  // ...
}

typeof Point // "function"
Point === Point.prototype.constructor // true
```

上面程式碼表明，類的資料型別就是函式，類本身就指向建構函式。

使用的時候，也是直接對類使用`new`命令，跟建構函式的用法完全一致。

```javascript
class Bar {
  doStuff() {
    console.log('stuff');
  }
}

var b = new Bar();
b.doStuff() // "stuff"
```

建構函式的`prototype`屬性，在ES6的“類”上面繼續存在。事實上，類的所有方法都定義在類的`prototype`屬性上面。

```javascript
class Point {
  constructor(){
    // ...
  }

  toString(){
    // ...
  }

  toValue(){
    // ...
  }
}

// 等同於

Point.prototype = {
  toString(){},
  toValue(){}
};
```

在類的實例上面呼叫方法，其實就是呼叫原型上的方法。

```javascript
class B {}
let b = new B();

b.constructor === B.prototype.constructor // true
```

上面程式碼中，`b`是B類的實例，它的`constructor`方法就是B類原型的`constructor`方法。

由於類的方法都定義在`prototype`物件上面，所以類的新方法可以新增在`prototype`物件上面。`Object.assign`方法可以很方便地一次向類新增多個方法。

```javascript
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});
```

`prototype`物件的`constructor`屬性，直接指向“類”的本身，這與ES5的行為是一致的。

```javascript
Point.prototype.constructor === Point // true
```

另外，類的內部所有定義的方法，都是不可列舉的（non-enumerable）。

```javascript
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.keys(Point.prototype)
// []
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面程式碼中，`toString`方法是`Point`類內部定義的方法，它是不可列舉的。這一點與ES5的行為不一致。

```javascript
var Point = function (x, y) {
  // ...
};

Point.prototype.toString = function() {
  // ...
};

Object.keys(Point.prototype)
// ["toString"]
Object.getOwnPropertyNames(Point.prototype)
// ["constructor","toString"]
```

上面程式碼採用ES5的寫法，`toString`方法就是可列舉的。

類的屬性名，可以採用表示式。

```javascript
let methodName = "getArea";
class Square{
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

上面程式碼中，`Square`類的方法名`getArea`，是從表示式得到的。

### constructor方法

`constructor`方法是類的預設方法，通過`new`命令生成物件實例時，自動呼叫該方法。一個類必須有`constructor`方法，如果沒有顯式定義，一個空的`constructor`方法會被預設新增。

```javascript
constructor() {}
```

`constructor`方法預設返回實例物件（即`this`），完全可以指定返回另外一個物件。

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false
```

上面程式碼中，`constructor`函式返回一個全新的物件，結果導致實例物件不是`Foo`類的實例。

類的建構函式，不使用`new`是沒法呼叫的，會報錯。這是它跟普通建構函式的一個主要區別，後者不用`new`也可以執行。

```javascript
class Foo {
  constructor() {
    return Object.create(null);
  }
}

Foo()
// TypeError: Class constructor Foo cannot be invoked without 'new'
```

### 類的實例物件

生成類的實例物件的寫法，與ES5完全一樣，也是使用`new`命令。如果忘記加上`new`，像函式那樣呼叫`Class`，將會報錯。

```javascript
// 報錯
var point = Point(2, 3);

// 正確
var point = new Point(2, 3);
```

與ES5一樣，實例的屬性除非顯式定義在其本身（即定義在`this`物件上），否則都是定義在原型上（即定義在`class`上）。

```javascript
//定義類
class Point {

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

上面程式碼中，`x`和`y`都是實例物件`point`自身的屬性（因為定義在`this`變數上），所以`hasOwnProperty`方法返回`true`，而`toString`是原型物件的屬性（因為定義在`Point`類上），所以`hasOwnProperty`方法返回`false`。這些都與ES5的行為保持一致。

與ES5一樣，類的所有實例共享一個原型物件。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__ === p2.__proto__
//true
```

上面程式碼中，`p1`和`p2`都是Point的實例，它們的原型都是Point.prototype，所以`__proto__`屬性是相等的。

這也意味著，可以通過實例的`__proto__`屬性為Class新增方法。

```javascript
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
```

上面程式碼在`p1`的原型上添加了一個`printName`方法，由於`p1`的原型就是`p2`的原型，因此`p2`也可以呼叫這個方法。而且，此後新建的實例`p3`也可以呼叫這個方法。這意味著，使用實例的`__proto__`屬性改寫原型，必須相當謹慎，不推薦使用，因為這會改變Class的原始定義，影響到所有實例。

### 不存在變數提升

Class不存在變數提升（hoist），這一點與ES5完全不同。

```javascript
new Foo(); // ReferenceError
class Foo {}
```

上面程式碼中，`Foo`類使用在前，定義在後，這樣會報錯，因為ES6不會把類的宣告提升到程式碼頭部。這種規定的原因與下文要提到的繼承有關，必須保證子類在父類之後定義。

```javascript
{
  let Foo = class {};
  class Bar extends Foo {
  }
}
```

上面的程式碼不會報錯，因為`Bar`繼承`Foo`的時候，`Foo`已經有定義了。但是，如果存在`class`的提升，上面程式碼就會報錯，因為`class`會被提升到程式碼頭部，而`let`命令是不提升的，所以導致`Bar`繼承`Foo`的時候，`Foo`還沒有定義。

### Class表示式

與函式一樣，類也可以使用表示式的形式定義。

```javascript
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
```

上面程式碼使用表示式定義了一個類。需要注意的是，這個類的名字是`MyClass`而不是`Me`，`Me`只在Class的內部程式碼可用，指代當前類。

```javascript
let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined
```

上面程式碼表示，`Me`只在Class內部有定義。

如果類的內部沒用到的話，可以省略`Me`，也就是可以寫成下面的形式。

```javascript
const MyClass = class { /* ... */ };
```

採用Class表示式，可以寫出立即執行的Class。

```javascript
let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('張三');

person.sayName(); // "張三"
```

上面程式碼中，`person`是一個立即執行的類的實例。

### 私有方法

私有方法是常見需求，但 ES6 不提供，只能通過變通方法模擬實現。

一種做法是在命名上加以區別。

```javascript
class Widget {

  // 公有方法
  foo (baz) {
    this._bar(baz);
  }

  // 私有方法
  _bar(baz) {
    return this.snaf = baz;
  }

  // ...
}
```

上面程式碼中，`_bar`方法前面的下劃線，表示這是一個只限於內部使用的私有方法。但是，這種命名是不保險的，在類的外部，還是可以呼叫到這個方法。

另一種方法就是索性將私有方法移出模組，因為模組內部的所有方法都是對外可見的。

```javascript
class Widget {
  foo (baz) {
    bar.call(this, baz);
  }

  // ...
}

function bar(baz) {
  return this.snaf = baz;
}
```

上面程式碼中，`foo`是公有方法，內部呼叫了`bar.call(this, baz)`。這使得`bar`實際上成為了當前模組的私有方法。

還有一種方法是利用`Symbol`值的唯一性，將私有方法的名字命名為一個`Symbol`值。

```javascript
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};
```

上面程式碼中，`bar`和`snaf`都是`Symbol`值，導致第三方無法獲取到它們，因此達到了私有方法和私有屬性的效果。

### this的指向

類的方法內部如果含有`this`，它預設指向類的實例。但是，必須非常小心，一旦單獨使用該方法，很可能報錯。

```javascript
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}

const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```

上面程式碼中，`printName`方法中的`this`，預設指向`Logger`類的實例。但是，如果將這個方法提取出來單獨使用，`this`會指向該方法執行時所在的環境，因為找不到`print`方法而導致報錯。

一個比較簡單的解決方法是，在構造方法中繫結`this`，這樣就不會找不到`print`方法了。

```javascript
class Logger {
  constructor() {
    this.printName = this.printName.bind(this);
  }

  // ...
}
```

另一種解決方法是使用箭頭函式。

```javascript
class Logger {
  constructor() {
    this.printName = (name = 'there') => {
      this.print(`Hello ${name}`);
    };
  }

  // ...
}
```

還有一種解決方法是使用`Proxy`，獲取方法的時候，自動繫結`this`。

```javascript
function selfish (target) {
  const cache = new WeakMap();
  const handler = {
    get (target, key) {
      const value = Reflect.get(target, key);
      if (typeof value !== 'function') {
        return value;
      }
      if (!cache.has(value)) {
        cache.set(value, value.bind(target));
      }
      return cache.get(value);
    }
  };
  const proxy = new Proxy(target, handler);
  return proxy;
}

const logger = selfish(new Logger());
```

### 嚴格模式

類和模組的內部，預設就是嚴格模式，所以不需要使用`use strict`指定執行模式。只要你的程式碼寫在類或模組之中，就只有嚴格模式可用。

考慮到未來所有的程式碼，其實都是執行在模組之中，所以ES6實際上把整個語言升級到了嚴格模式。

### name屬性

由於本質上，ES6的類只是ES5的建構函式的一層包裝，所以函式的許多特性都被`Class`繼承，包括`name`屬性。

```javascript
class Point {}
Point.name // "Point"
```

`name`屬性總是返回緊跟在`class`關鍵字後面的類名。

## Class的繼承

### 基本用法

Class之間可以通過`extends`關鍵字實現繼承，這比ES5的通過修改原型鏈實現繼承，要清晰和方便很多。

```javascript
class ColorPoint extends Point {}
```

上面程式碼定義了一個`ColorPoint`類，該類通過`extends`關鍵字，繼承了`Point`類的所有屬性和方法。但是由於沒有部署任何程式碼，所以這兩個類完全一樣，等於複製了一個`Point`類。下面，我們在`ColorPoint`內部加上程式碼。

```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 呼叫父類的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 呼叫父類的toString()
  }
}
```

上面程式碼中，`constructor`方法和`toString`方法之中，都出現了`super`關鍵字，它在這裡表示父類的建構函式，用來新建父類的`this`物件。

子類必須在`constructor`方法中呼叫`super`方法，否則新建實例時會報錯。這是因為子類沒有自己的`this`物件，而是繼承父類的`this`物件，然後對其進行加工。如果不呼叫`super`方法，子類就得不到`this`物件。

```javascript
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError
```

上面程式碼中，`ColorPoint`繼承了父類`Point`，但是它的建構函式沒有呼叫`super`方法，導致新建實例時報錯。

ES5的繼承，實質是先創造子類的實例物件`this`，然後再將父類的方法新增到`this`上面（`Parent.apply(this)`）。ES6的繼承機制完全不同，實質是先創造父類的實例物件`this`（所以必須先呼叫`super`方法），然後再用子類的建構函式修改`this`。

如果子類沒有定義`constructor`方法，這個方法會被預設新增，程式碼如下。也就是說，不管有沒有顯式定義，任何一個子類都有`constructor`方法。

```javascript
constructor(...args) {
  super(...args);
}
```

另一個需要注意的地方是，在子類的建構函式中，只有呼叫`super`之後，才可以使用`this`關鍵字，否則會報錯。這是因為子類實例的構建，是基於對父類實例加工，只有`super`方法才能返回父類實例。

```javascript
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正確
  }
}
```

上面程式碼中，子類的`constructor`方法沒有呼叫`super`之前，就使用`this`關鍵字，結果報錯，而放在`super`方法之後就是正確的。

下面是生成子類實例的程式碼。

```javascript
let cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint // true
cp instanceof Point // true
```

上面程式碼中，實例物件`cp`同時是`ColorPoint`和`Point`兩個類的實例，這與ES5的行為完全一致。

### 類的prototype屬性和\_\_proto\_\_屬性

大多數瀏覽器的ES5實現之中，每一個物件都有`__proto__`屬性，指向對應的建構函式的prototype屬性。Class作為建構函式的語法糖，同時有prototype屬性和`__proto__`屬性，因此同時存在兩條繼承鏈。

（1）子類的`__proto__`屬性，表示建構函式的繼承，總是指向父類。

（2）子類`prototype`屬性的`__proto__`屬性，表示方法的繼承，總是指向父類的`prototype`屬性。

```javascript
class A {
}

class B extends A {
}

B.__proto__ === A // true
B.prototype.__proto__ === A.prototype // true
```

上面程式碼中，子類`B`的`__proto__`屬性指向父類`A`，子類`B`的`prototype`屬性的`__proto__`屬性指向父類`A`的`prototype`屬性。

這樣的結果是因為，類的繼承是按照下面的模式實現的。

```javascript
class A {
}

class B {
}

// B的實例繼承A的實例
Object.setPrototypeOf(B.prototype, A.prototype);
const b = new B();

// B的實例繼承A的靜態屬性
Object.setPrototypeOf(B, A);
const b = new B();
```

《物件的擴展》一章給出過`Object.setPrototypeOf`方法的實現。

```javascript
Object.setPrototypeOf = function (obj, proto) {
  obj.__proto__ = proto;
  return obj;
}
```

因此，就得到了上面的結果。

```javascript
Object.setPrototypeOf(B.prototype, A.prototype);
// 等同於
B.prototype.__proto__ = A.prototype;

Object.setPrototypeOf(B, A);
// 等同於
B.__proto__ = A;
```

這兩條繼承鏈，可以這樣理解：作為一個物件，子類（`B`）的原型（`__proto__`屬性）是父類（`A`）；作為一個建構函式，子類（`B`）的原型（`prototype`屬性）是父類的實例。

```javascript
Object.create(A.prototype);
// 等同於
B.prototype.__proto__ = A.prototype;
```

### Extends 的繼承目標

`extends`關鍵字後面可以跟多種型別的值。

```javascript
class B extends A {
}
```

上面程式碼的`A`，只要是一個有`prototype`屬性的函式，就能被`B`繼承。由於函式都有`prototype`屬性（除了`Function.prototype`函式），因此`A`可以是任意函式。

下面，討論三種特殊情況。

第一種特殊情況，子類繼承Object類。

```javascript
class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true
```

這種情況下，`A`其實就是建構函式`Object`的複製，`A`的實例就是`Object`的實例。

第二種特殊情況，不存在任何繼承。

```javascript
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

這種情況下，A作為一個基類（即不存在任何繼承），就是一個普通函式，所以直接繼承`Funciton.prototype`。但是，`A`呼叫後返回一個空物件（即`Object`實例），所以`A.prototype.__proto__`指向建構函式（`Object`）的`prototype`屬性。

第三種特殊情況，子類繼承`null`。

```javascript
class A extends null {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === undefined // true
```

這種情況與第二種情況非常像。`A`也是一個普通函式，所以直接繼承`Funciton.prototype`。但是，A呼叫後返回的物件不繼承任何方法，所以它的`__proto__`指向`Function.prototype`，即實質上執行了下面的程式碼。

```javascript
class C extends null {
  constructor() { return Object.create(null); }
}
```

### Object.getPrototypeOf()

`Object.getPrototypeOf`方法可以用來從子類上獲取父類。

```javascript
Object.getPrototypeOf(ColorPoint) === Point
// true
```

因此，可以使用這個方法判斷，一個類是否繼承了另一個類。

### super 關鍵字

`super`這個關鍵字，既可以當作函式使用，也可以當作物件使用。在這兩種情況下，它的用法完全不同。

第一種情況，`super`作為函式呼叫時，代表父類的建構函式。ES6 要求，子類的建構函式必須執行一次`super`函式。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
  }
}
```

上面程式碼中，子類`B`的建構函式之中的`super()`，代表呼叫父類的建構函式。這是必須的，否則 JavaScript 引擎會報錯。

注意，`super`雖然代表了父類`A`的建構函式，但是返回的是子類`B`的實例，即`super`內部的`this`指的是`B`，因此`super()`在這裡相當於`A.prototype.constructor.call(this)`。

```javascript
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
```

上面程式碼中，`new.target`指向當前正在執行的函式。可以看到，在`super()`執行時，它指向的是子類`B`的建構函式，而不是父類`A`的建構函式。也就是說，`super()`內部的`this`指向的是`B`。

作為函式時，`super()`只能用在子類的建構函式之中，用在其他地方就會報錯。

```javascript
class A {}

class B extends A {
  m() {
    super(); // 報錯
  }
}
```

上面程式碼中，`super()`用在`B`類的`m`方法之中，就會造成句法錯誤。

第二種情況，`super`作為物件時，在普通方法中，指向父類的原型物件；在靜態方法中，指向父類。

```javascript
class A {
  p() {
    return 2;
  }
}

class B extends A {
  constructor() {
    super();
    console.log(super.p()); // 2
  }
}

let b = new B();
```

上面程式碼中，子類`B`當中的`super.p()`，就是將`super`當作一個物件使用。這時，`super`在普通方法之中，指向`A.prototype`，所以`super.p()`就相當於`A.prototype.p()`。

這裡需要注意，由於`super`指向父類的原型物件，所以定義在父類實例上的方法或屬性，是無法通過`super`呼叫的。

```javascript
class A {
  constructor() {
    this.p = 2;
  }
}

class B extends A {
  get m() {
    return super.p;
  }
}

let b = new B();
b.m // undefined
```

上面程式碼中，`p`是父類`A`實例的屬性，`super.p`就引用不到它。

如果屬性定義在父類的原型物件上，`super`就可以取到。

```javascript
class A {}
A.prototype.x = 2;

class B extends A {
  constructor() {
    super();
    console.log(super.x) // 2
  }
}

let b = new B();
```

上面程式碼中，屬性`x`是定義在`A.prototype`上面的，所以`super.x`可以取到它的值。

ES6 規定，通過`super`呼叫父類的方法時，`super`會繫結子類的`this`。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
  print() {
    console.log(this.x);
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
  }
  m() {
    super.print();
  }
}

let b = new B();
b.m() // 2
```

上面程式碼中，`super.print()`雖然呼叫的是`A.prototype.print()`，但是`A.prototype.print()`會繫結子類`B`的`this`，導致輸出的是`2`，而不是`1`。也就是說，實際上執行的是`super.print.call(this)`。

由於繫結子類的`this`，所以如果通過`super`對某個屬性賦值，這時`super`就是`this`，賦值的屬性會變成子類實例的屬性。

```javascript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

上面程式碼中，`super.x`賦值為`3`，這時等同於對`this.x`賦值為`3`。而當讀取`super.x`的時候，讀的是`A.prototype.x`，所以返回`undefined`。

如果`super`作為物件，用在靜態方法之中，這時`super`將指向父類，而不是父類的原型物件。

```javascript
class Parent {
  static myMethod(msg) {
    console.log('static', msg);
  }

  myMethod(msg) {
    console.log('instance', msg);
  }
}

class Child extends Parent {
  static myMethod(msg) {
    super.myMethod(msg);
  }

  myMethod(msg) {
    super.myMethod(msg);
  }
}

Child.myMethod(1); // static 1

var child = new Child();
child.myMethod(2); // instance 2
```

上面程式碼中，`super`在靜態方法之中指向父類，在普通方法之中指向父類的原型物件。

注意，使用`super`的時候，必須顯式指定是作為函式、還是作為物件使用，否則會報錯。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // 報錯
  }
}
```

上面程式碼中，`console.log(super)`當中的`super`，無法看出是作為函式使用，還是作為物件使用，所以 JavaScript 引擎解析程式碼的時候就會報錯。這時，如果能清晰地表明`super`的資料型別，就不會報錯。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super.valueOf() instanceof B); // true
  }
}

let b = new B();
```

上面程式碼中，`super.valueOf()`表明`super`是一個物件，因此就不會報錯。同時，由於`super`繫結`B`的`this`，所以`super.valueOf()`返回的是一個`B`的實例。

最後，由於物件總是繼承其他物件的，所以可以在任意一個物件中，使用`super`關鍵字。

```javascript
var obj = {
  toString() {
    return "MyObject: " + super.toString();
  }
};

obj.toString(); // MyObject: [object Object]
```

### 實例的\_\_proto\_\_屬性

子類實例的\_\_proto\_\_屬性的\_\_proto\_\_屬性，指向父類實例的\_\_proto\_\_屬性。也就是說，子類的原型的原型，是父類的原型。

```javascript
var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__ === p1.__proto__ // false
p2.__proto__.__proto__ === p1.__proto__ // true
```

上面程式碼中，`ColorPoint`繼承了`Point`，導致前者原型的原型是後者的原型。

因此，通過子類實例的`__proto__.__proto__`屬性，可以修改父類實例的行為。

```javascript
p2.__proto__.__proto__.printName = function () {
  console.log('Ha');
};

p1.printName() // "Ha"
```

上面程式碼在`ColorPoint`的實例`p2`上向`Point`類新增方法，結果影響到了`Point`的實例`p1`。

## 原生建構函式的繼承

原生建構函式是指語言內建的建構函式，通常用來生成資料結構。ECMAScript的原生建構函式大致有下面這些。

- Boolean()
- Number()
- String()
- Array()
- Date()
- Function()
- RegExp()
- Error()
- Object()

以前，這些原生建構函式是無法繼承的，比如，不能自己定義一個`Array`的子類。

```javascript
function MyArray() {
  Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: true
  }
});
```

上面程式碼定義了一個繼承Array的`MyArray`類。但是，這個類的行為與`Array`完全不一致。

```javascript
var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"
```

之所以會發生這種情況，是因為子類無法獲得原生建構函式的內部屬性，通過`Array.apply()`或者分配給原型物件都不行。原生建構函式會忽略`apply`方法傳入的`this`，也就是說，原生建構函式的`this`無法繫結，導致拿不到內部屬性。

ES5是先新建子類的實例物件`this`，再將父類的屬性新增到子類上，由於父類的內部屬性無法獲取，導致無法繼承原生的建構函式。比如，Array建構函式有一個內部屬性`[[DefineOwnProperty]]`，用來定義新屬性時，更新`length`屬性，這個內部屬性無法在子類獲取，導致子類的`length`屬性行為不正常。

下面的例子中，我們想讓一個普通物件繼承`Error`物件。

```javascript
var e = {};

Object.getOwnPropertyNames(Error.call(e))
// [ 'stack' ]

Object.getOwnPropertyNames(e)
// []
```

上面程式碼中，我們想通過`Error.call(e)`這種寫法，讓普通物件`e`具有`Error`物件的實例屬性。但是，`Error.call()`完全忽略傳入的第一個引數，而是返回一個新物件，`e`本身沒有任何變化。這證明了`Error.call(e)`這種寫法，無法繼承原生建構函式。

ES6允許繼承原生建構函式定義子類，因為ES6是先新建父類的實例物件`this`，然後再用子類的建構函式修飾`this`，使得父類的所有行為都可以繼承。下面是一個繼承`Array`的例子。

```javascript
class MyArray extends Array {
  constructor(...args) {
    super(...args);
  }
}

var arr = new MyArray();
arr[0] = 12;
arr.length // 1

arr.length = 0;
arr[0] // undefined
```

上面程式碼定義了一個`MyArray`類，繼承了`Array`建構函式，因此就可以從`MyArray`生成陣列的實例。這意味著，ES6可以自定義原生資料結構（比如Array、String等）的子類，這是ES5無法做到的。

上面這個例子也說明，`extends`關鍵字不僅可以用來繼承類，還可以用來繼承原生的建構函式。因此可以在原生資料結構的基礎上，定義自己的資料結構。下面就是定義了一個帶版本功能的陣列。

```javascript
class VersionedArray extends Array {
  constructor() {
    super();
    this.history = [[]];
  }
  commit() {
    this.history.push(this.slice());
  }
  revert() {
    this.splice(0, this.length, ...this.history[this.history.length - 1]);
  }
}

var x = new VersionedArray();

x.push(1);
x.push(2);
x // [1, 2]
x.history // [[]]

x.commit();
x.history // [[], [1, 2]]
x.push(3);
x // [1, 2, 3]

x.revert();
x // [1, 2]
```

上面程式碼中，`VersionedArray`結構會通過`commit`方法，將自己的當前狀態存入`history`屬性，然後通過`revert`方法，可以撤銷當前版本，回到上一個版本。除此之外，`VersionedArray`依然是一個數組，所有原生的陣列方法都可以在它上面呼叫。

下面是一個自定義`Error`子類的例子。

```javascript
class ExtendableError extends Error {
  constructor(message) {
    super();
    this.message = message;
    this.stack = (new Error()).stack;
    this.name = this.constructor.name;
  }
}

class MyError extends ExtendableError {
  constructor(m) {
    super(m);
  }
}

var myerror = new MyError('ll');
myerror.message // "ll"
myerror instanceof Error // true
myerror.name // "MyError"
myerror.stack
// Error
//     at MyError.ExtendableError
//     ...
```

注意，繼承`Object`的子類，有一個[行為差異](http://stackoverflow.com/questions/36203614/super-does-not-pass-arguments-when-instantiating-a-class-extended-from-object)。

```javascript
class NewObj extends Object{
  constructor(){
    super(...arguments);
  }
}
var o = new NewObj({attr: true});
console.log(o.attr === true);  // false
```

上面程式碼中，`NewObj`繼承了`Object`，但是無法通過`super`方法向父類`Object`傳參。這是因為ES6改變了`Object`建構函式的行為，一旦發現`Object`方法不是通過`new Object()`這種形式呼叫，ES6規定`Object`建構函式會忽略引數。

## Class的取值函式（getter）和存值函式（setter）

與ES5一樣，在Class內部可以使用`get`和`set`關鍵字，對某個屬性設定存值函式和取值函式，攔截該屬性的存取行為。

```javascript
class MyClass {
  constructor() {
    // ...
  }
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}

let inst = new MyClass();

inst.prop = 123;
// setter: 123

inst.prop
// 'getter'
```

上面程式碼中，`prop`屬性有對應的存值函式和取值函式，因此賦值和讀取行為都被自定義了。

存值函式和取值函式是設定在屬性的descriptor物件上的。

```javascript
class CustomHTMLElement {
  constructor(element) {
    this.element = element;
  }

  get html() {
    return this.element.innerHTML;
  }

  set html(value) {
    this.element.innerHTML = value;
  }
}

var descriptor = Object.getOwnPropertyDescriptor(
  CustomHTMLElement.prototype, "html");
"get" in descriptor  // true
"set" in descriptor  // true
```

上面程式碼中，存值函式和取值函式是定義在`html`屬性的描述物件上面，這與ES5完全一致。

## Class 的 Generator 方法

如果某個方法之前加上星號（`*`），就表示該方法是一個 Generator 函式。

```javascript
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
// hello
// world
```

上面程式碼中，`Foo`類的`Symbol.iterator`方法前有一個星號，表示該方法是一個 Generator 函式。`Symbol.iterator`方法返回一個`Foo`類的預設遍歷器，`for...of`迴圈會自動呼叫這個遍歷器。

## Class 的靜態方法

類相當於實例的原型，所有在類中定義的方法，都會被實例繼承。如果在一個方法前，加上`static`關鍵字，就表示該方法不會被實例繼承，而是直接通過類來呼叫，這就稱為“靜態方法”。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

Foo.classMethod() // 'hello'

var foo = new Foo();
foo.classMethod()
// TypeError: foo.classMethod is not a function
```

上面程式碼中，`Foo`類的`classMethod`方法前有`static`關鍵字，表明該方法是一個靜態方法，可以直接在`Foo`類上呼叫（`Foo.classMethod()`），而不是在`Foo`類的實例上呼叫。如果在實例上呼叫靜態方法，會丟擲一個錯誤，表示不存在該方法。

父類的靜態方法，可以被子類繼承。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod(); // 'hello'
```

上面程式碼中，父類`Foo`有一個靜態方法，子類`Bar`可以呼叫這個方法。

靜態方法也是可以從`super`物件上呼叫的。

```javascript
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
  static classMethod() {
    return super.classMethod() + ', too';
  }
}

Bar.classMethod();
```

## Class的靜態屬性和實例屬性

靜態屬性指的是Class本身的屬性，即`Class.propname`，而不是定義在實例物件（`this`）上的屬性。

```javascript
class Foo {
}

Foo.prop = 1;
Foo.prop // 1
```

上面的寫法為`Foo`類定義了一個靜態屬性`prop`。

目前，只有這種寫法可行，因為ES6明確規定，Class內部只有靜態方法，沒有靜態屬性。

```javascript
// 以下兩種寫法都無效
class Foo {
  // 寫法一
  prop: 2

  // 寫法二
  static prop: 2
}

Foo.prop // undefined
```

ES7有一個靜態屬性的[提案](https://github.com/jeffmo/es-class-properties)，目前Babel轉碼器支援。

這個提案對實例屬性和靜態屬性，都規定了新的寫法。

（1）類的實例屬性

類的實例屬性可以用等式，寫入類的定義之中。

```javascript
class MyClass {
  myProp = 42;

  constructor() {
    console.log(this.myProp); // 42
  }
}
```

上面程式碼中，`myProp`就是`MyClass`的實例屬性。在`MyClass`的實例上，可以讀取這個屬性。

以前，我們定義實例屬性，只能寫在類的`constructor`方法裡面。

```javascript
class ReactCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
}
```

上面程式碼中，構造方法`constructor`裡面，定義了`this.state`屬性。

有了新的寫法以後，可以不在`constructor`方法裡面定義。

```javascript
class ReactCounter extends React.Component {
  state = {
    count: 0
  };
}
```

這種寫法比以前更清晰。

為了可讀性的目的，對於那些在`constructor`裡面已經定義的實例屬性，新寫法允許直接列出。

```javascript
class ReactCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  state;
}
```

（2）類的靜態屬性

類的靜態屬性只要在上面的實例屬性寫法前面，加上`static`關鍵字就可以了。

```javascript
class MyClass {
  static myStaticProp = 42;

  constructor() {
    console.log(MyClass.myStaticProp); // 42
  }
}
```

同樣的，這個新寫法大大方便了靜態屬性的表達。

```javascript
// 老寫法
class Foo {
}
Foo.prop = 1;

// 新寫法
class Foo {
  static prop = 1;
}
```

上面程式碼中，老寫法的靜態屬性定義在類的外部。整個類生成以後，再生成靜態屬性。這樣讓人很容易忽略這個靜態屬性，也不符合相關程式碼應該放在一起的程式碼組織原則。另外，新寫法是顯式宣告（declarative），而不是賦值處理，語義更好。

## 類的私有屬性

目前，有一個[提案](https://github.com/tc39/proposal-private-fields)，為`class`加了私有屬性。方法是在屬性名之前，使用`#`表示。

```javascript
class Point {
  #x;

  constructor(x = 0) {
    #x = +x;
  }

  get x() { return #x }
  set x(value) { #x = +value }
}
```

上面程式碼中，`#x`就表示私有屬性`x`，在`Point`類之外是讀取不到這個屬性的。還可以看到，私有屬性與實例的屬性是可以同名的（比如，`#x`與`get x()`）。

私有屬性可以指定初始值，在建構函式執行時進行初始化。

```javascript
class Point {
  #x = 0;
  constructor() {
    #x; // 0
  }
}
```

之所以要引入一個新的字首`#`表示私有屬性，而沒有采用`private`關鍵字，是因為 JavaScript 是一門動態語言，使用獨立的符號似乎是唯一的可靠方法，能夠準確地區分一種屬性是私有屬性。另外，Ruby 語言使用`@`表示私有屬性，ES6 沒有用這個符號而使用`#`，是因為`@`已經被留給了 Decorator。

該提案只規定了私有屬性的寫法。但是，很自然地，它也可以用來寫私有方法。

```javascript
class Foo {
  #a;
  #b;
  #sum() { return #a + #b; }
  printSum() { console.log(#sum()); }
  constructor(a, b) { #a = a; #b = b; }
}
```

## new.target屬性

`new`是從建構函式生成實例的命令。ES6為`new`命令引入了一個`new.target`屬性，（在建構函式中）返回`new`命令作用於的那個建構函式。如果建構函式不是通過`new`命令呼叫的，`new.target`會返回`undefined`，因此這個屬性可以用來確定建構函式是怎麼呼叫的。

```javascript
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必須使用new生成實例');
  }
}

// 另一種寫法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必須使用new生成實例');
  }
}

var person = new Person('張三'); // 正確
var notAPerson = Person.call(person, '張三');  // 報錯
```

上面程式碼確保建構函式只能通過`new`命令呼叫。

Class內部呼叫`new.target`，返回當前Class。

```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}

var obj = new Rectangle(3, 4); // 輸出 true
```

需要注意的是，子類繼承父類時，`new.target`會返回子類。

```javascript
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    // ...
  }
}

class Square extends Rectangle {
  constructor(length) {
    super(length, length);
  }
}

var obj = new Square(3); // 輸出 false
```

上面程式碼中，`new.target`會返回子類。

利用這個特點，可以寫出不能獨立使用、必須繼承後才能使用的類。

```javascript
class Shape {
  constructor() {
    if (new.target === Shape) {
      throw new Error('本類不能實例化');
    }
  }
}

class Rectangle extends Shape {
  constructor(length, width) {
    super();
    // ...
  }
}

var x = new Shape();  // 報錯
var y = new Rectangle(3, 4);  // 正確
```

上面程式碼中，`Shape`類不能被實例化，只能用於繼承。

注意，在函式外部，使用`new.target`會報錯。

## Mixin模式的實現

Mixin模式指的是，將多個類的介面“混入”（mix in）另一個類。它在ES6的實現如下。

```javascript
function mix(...mixins) {
  class Mix {}

  for (let mixin of mixins) {
    copyProperties(Mix, mixin);
    copyProperties(Mix.prototype, mixin.prototype);
  }

  return Mix;
}

function copyProperties(target, source) {
  for (let key of Reflect.ownKeys(source)) {
    if ( key !== "constructor"
      && key !== "prototype"
      && key !== "name"
    ) {
      let desc = Object.getOwnPropertyDescriptor(source, key);
      Object.defineProperty(target, key, desc);
    }
  }
}
```

上面程式碼的`mix`函式，可以將多個物件合成為一個類。使用的時候，只要繼承這個類即可。

```javascript
class DistributedEdit extends mix(Loggable, Serializable) {
  // ...
}
```
