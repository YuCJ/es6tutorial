# 修飾器

## 類的修飾

修飾器（Decorator）是一個函式，用來修改類的行為。這是ES7的一個[提案](https://github.com/wycats/javascript-decorators)，目前Babel轉碼器已經支援。

修飾器對類的行為的改變，是程式碼編譯時發生的，而不是在執行時。這意味著，修飾器能在編譯階段執行程式碼。

```javascript
function testable(target) {
  target.isTestable = true;
}

@testable
class MyTestableClass {}

console.log(MyTestableClass.isTestable) // true
```

上面程式碼中，`@testable`就是一個修飾器。它修改了`MyTestableClass`這個類的行為，為它加上了靜態屬性`isTestable`。

基本上，修飾器的行為就是下面這樣。

```javascript
@decorator
class A {}

// 等同於

class A {}
A = decorator(A) || A;
```

也就是說，修飾器本質就是編譯時執行的函式。

修飾器函式的第一個引數，就是所要修飾的目標類。

```javascript
function testable(target) {
  // ...
}
```

上面程式碼中，`testable`函式的引數`target`，就是會被修飾的類。

如果覺得一個引數不夠用，可以在修飾器外面再封裝一層函式。

```javascript
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```

上面程式碼中，修飾器`testable`可以接受引數，這就等於可以修改修飾器的行為。

前面的例子是為類新增一個靜態屬性，如果想新增實例屬性，可以通過目標類的`prototype`物件操作。

```javascript
function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable // true
```

上面程式碼中，修飾器函式`testable`是在目標類的`prototype`物件上新增屬性，因此就可以在實例上呼叫。

下面是另外一個例子。

```javascript
// mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// main.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // 'foo'
```

上面程式碼通過修飾器`mixins`，把`Foo`類的方法新增到了`MyClass`的實例上面。可以用`Object.assign()`模擬這個功能。

```javascript
const Foo = {
  foo() { console.log('foo') }
};

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'
```

## 方法的修飾

修飾器不僅可以修飾類，還可以修飾類的屬性。

```javascript
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}
```

上面程式碼中，修飾器`readonly`用來修飾“類”的`name`方法。

此時，修飾器函式一共可以接受三個引數，第一個引數是所要修飾的目標物件，第二個引數是所要修飾的屬性名，第三個引數是該屬性的描述物件。

```javascript
function readonly(target, name, descriptor){
  // descriptor物件原來的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}

readonly(Person.prototype, 'name', descriptor);
// 類似於
Object.defineProperty(Person.prototype, 'name', descriptor);
```

上面程式碼說明，修飾器（readonly）會修改屬性的描述物件（descriptor），然後被修改的描述物件再用來定義屬性。

下面是另一個例子，修改屬性描述物件的`enumerable`屬性，使得該屬性不可遍歷。

```javascript
class Person {
  @nonenumerable
  get kidCount() { return this.children.length; }
}

function nonenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}
```

下面的`@log`修飾器，可以起到輸出日誌的作用。

```javascript
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function() {
    console.log(`Calling "${name}" with`, arguments);
    return oldValue.apply(null, arguments);
  };

  return descriptor;
}

const math = new Math();

// passed parameters should get logged now
math.add(2, 4);
```

上面程式碼中，`@log`修飾器的作用就是在執行原始的操作之前，執行一次`console.log`，從而達到輸出日誌的目的。

修飾器有註釋的作用。

```javascript
@testable
class Person {
  @readonly
  @nonenumerable
  name() { return `${this.first} ${this.last}` }
}
```

從上面程式碼中，我們一眼就能看出，`Person`類是可測試的，而`name`方法是隻讀和不可列舉的。

如果同一個方法有多個修飾器，會像剝洋蔥一樣，先從外到內進入，然後由內向外執行。

```javascript
function dec(id){
    console.log('evaluated', id);
    return (target, property, descriptor) => console.log('executed', id);
}

class Example {
    @dec(1)
    @dec(2)
    method(){}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```

上面程式碼中，外層修飾器`@dec(1)`先進入，但是內層修飾器`@dec(2)`先執行。

除了註釋，修飾器還能用來型別檢查。所以，對於類來說，這項功能相當有用。從長期來看，它將是JavaScript程式碼靜態分析的重要工具。

## 為什麼修飾器不能用於函式？

修飾器只能用於類和類的方法，不能用於函式，因為存在函式提升。

```javascript
var counter = 0;

var add = function () {
  counter++;
};

@add
function foo() {
}
```

上面的程式碼，意圖是執行後`counter`等於1，但是實際上結果是`counter`等於0。因為函式提升，使得實際執行的程式碼是下面這樣。

```javascript
@add
function foo() {
}

var counter;
var add;

counter = 0;

add = function () {
  counter++;
};
```

下面是另一個例子。

```javascript
var readOnly = require("some-decorator");

@readOnly
function foo() {
}
```

上面程式碼也有問題，因為實際執行是下面這樣。

```javascript
var readOnly;

@readOnly
function foo() {
}

readOnly = require("some-decorator");
```

總之，由於存在函式提升，使得修飾器不能用於函式。類是不會提升的，所以就沒有這方面的問題。

## core-decorators.js

[core-decorators.js](https://github.com/jayphelps/core-decorators.js)是一個第三方模組，提供了幾個常見的修飾器，通過它可以更好地理解修飾器。

**（1）@autobind**

`autobind`修飾器使得方法中的`this`物件，繫結原始物件。

```javascript
import { autobind } from 'core-decorators';

class Person {
  @autobind
  getPerson() {
    return this;
  }
}

let person = new Person();
let getPerson = person.getPerson;

getPerson() === person;
// true
```

**（2）@readonly**

`readonly`修飾器使得屬性或方法不可寫。

```javascript
import { readonly } from 'core-decorators';

class Meal {
  @readonly
  entree = 'steak';
}

var dinner = new Meal();
dinner.entree = 'salmon';
// Cannot assign to read only property 'entree' of [object Object]
```

**（3）@override**

`override`修飾器檢查子類的方法，是否正確覆蓋了父類的同名方法，如果不正確會報錯。

```javascript
import { override } from 'core-decorators';

class Parent {
  speak(first, second) {}
}

class Child extends Parent {
  @override
  speak() {}
  // SyntaxError: Child#speak() does not properly override Parent#speak(first, second)
}

// or

class Child extends Parent {
  @override
  speaks() {}
  // SyntaxError: No descriptor matching Child#speaks() was found on the prototype chain.
  //
  //   Did you mean "speak"?
}
```

**（4）@deprecate (別名@deprecated)**

`deprecate`或`deprecated`修飾器在控制檯顯示一條警告，表示該方法將廢除。

```javascript
import { deprecate } from 'core-decorators';

class Person {
  @deprecate
  facepalm() {}

  @deprecate('We stopped facepalming')
  facepalmHard() {}

  @deprecate('We stopped facepalming', { url: 'http://knowyourmeme.com/memes/facepalm' })
  facepalmHarder() {}
}

let person = new Person();

person.facepalm();
// DEPRECATION Person#facepalm: This function will be removed in future versions.

person.facepalmHard();
// DEPRECATION Person#facepalmHard: We stopped facepalming

person.facepalmHarder();
// DEPRECATION Person#facepalmHarder: We stopped facepalming
//
//     See http://knowyourmeme.com/memes/facepalm for more details.
//
```

**（5）@suppressWarnings**

`suppressWarnings`修飾器抑制`decorated`修飾器導致的`console.warn()`呼叫。但是，非同步程式碼發出的呼叫除外。

```javascript
import { suppressWarnings } from 'core-decorators';

class Person {
  @deprecated
  facepalm() {}

  @suppressWarnings
  facepalmWithoutWarning() {
    this.facepalm();
  }
}

let person = new Person();

person.facepalmWithoutWarning();
// no warning is logged
```

## 使用修飾器實現自動釋出事件

我們可以使用修飾器，使得物件的方法被呼叫時，自動發出一個事件。

```javascript
import postal from "postal/lib/postal.lodash";

export default function publish(topic, channel) {
  return function(target, name, descriptor) {
    const fn = descriptor.value;

    descriptor.value = function() {
      let value = fn.apply(this, arguments);
      postal.channel(channel || target.channel || "/").publish(topic, value);
    };
  };
}
```

上面程式碼定義了一個名為`publish`的修飾器，它通過改寫`descriptor.value`，使得原方法被呼叫時，會自動發出一個事件。它使用的事件“釋出/訂閱”庫是[Postal.js](https://github.com/postaljs/postal.js)。

它的用法如下。

```javascript
import publish from "path/to/decorators/publish";

class FooComponent {
  @publish("foo.some.message", "component")
  someMethod() {
    return {
      my: "data"
    };
  }
  @publish("foo.some.other")
  anotherMethod() {
    // ...
  }
}
```

以後，只要呼叫`someMethod`或者`anotherMethod`，就會自動發出一個事件。

```javascript
let foo = new FooComponent();

foo.someMethod() // 在"component"頻道釋出"foo.some.message"事件，附帶的資料是{ my: "data" }
foo.anotherMethod() // 在"/"頻道釋出"foo.some.other"事件，不附帶資料
```

## Mixin

在修飾器的基礎上，可以實現`Mixin`模式。所謂`Mixin`模式，就是物件繼承的一種替代方案，中文譯為“混入”（mix in），意為在一個物件之中混入另外一個物件的方法。

請看下面的例子。

```javascript
const Foo = {
  foo() { console.log('foo') }
};

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'
```

上面程式碼之中，物件`Foo`有一個`foo`方法，通過`Object.assign`方法，可以將`foo`方法“混入”`MyClass`類，導致`MyClass`的實例`obj`物件都具有`foo`方法。這就是“混入”模式的一個簡單實現。

下面，我們部署一個通用指令碼`mixins.js`，將mixin寫成一個修飾器。

```javascript
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list);
  };
}
```

然後，就可以使用上面這個修飾器，為類“混入”各種方法。

```javascript
import { mixins } from './mixins';

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // "foo"
```

通過mixins這個修飾器，實現了在MyClass類上面“混入”Foo物件的`foo`方法。

不過，上面的方法會改寫`MyClass`類的`prototype`物件，如果不喜歡這一點，也可以通過類的繼承實現mixin。

```javascript
class MyClass extends MyBaseClass {
  /* ... */
}
```

上面程式碼中，`MyClass`繼承了`MyBaseClass`。如果我們想在`MyClass`裡面“混入”一個`foo`方法，一個辦法是在`MyClass`和`MyBaseClass`之間插入一個混入類，這個類具有`foo`方法，並且繼承了`MyBaseClass`的所有方法，然後`MyClass`再繼承這個類。

```javascript
let MyMixin = (superclass) => class extends superclass {
  foo() {
    console.log('foo from MyMixin');
  }
};
```

上面程式碼中，`MyMixin`是一個混入類生成器，接受`superclass`作為引數，然後返回一個繼承`superclass`的子類，該子類包含一個`foo`方法。

接著，目標類再去繼承這個混入類，就達到了“混入”`foo`方法的目的。

```javascript
class MyClass extends MyMixin(MyBaseClass) {
  /* ... */
}

let c = new MyClass();
c.foo(); // "foo from MyMixin"
```

如果需要“混入”多個方法，就生成多個混入類。

```javascript
class MyClass extends Mixin1(Mixin2(MyBaseClass)) {
  /* ... */
}
```

這種寫法的一個好處，是可以呼叫`super`，因此可以避免在“混入”過程中覆蓋父類的同名方法。

```javascript
let Mixin1 = (superclass) => class extends superclass {
  foo() {
    console.log('foo from Mixin1');
    if (super.foo) super.foo();
  }
};

let Mixin2 = (superclass) => class extends superclass {
  foo() {
    console.log('foo from Mixin2');
    if (super.foo) super.foo();
  }
};

class S {
  foo() {
    console.log('foo from S');
  }
}

class C extends Mixin1(Mixin2(S)) {
  foo() {
    console.log('foo from C');
    super.foo();
  }
}
```

上面程式碼中，每一次`混入`發生時，都呼叫了父類的`super.foo`方法，導致父類的同名方法沒有被覆蓋，行為被保留了下來。

```javascript
new C().foo()
// foo from C
// foo from Mixin1
// foo from Mixin2
// foo from S
```

## Trait

Trait也是一種修飾器，效果與Mixin類似，但是提供更多功能，比如防止同名方法的衝突、排除混入某些方法、為混入的方法起別名等等。

下面採用[traits-decorator](https://github.com/CocktailJS/traits-decorator)這個第三方模組作為例子。這個模組提供的traits修飾器，不僅可以接受物件，還可以接受ES6類作為引數。

```javascript
import { traits } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') }
};

@traits(TFoo, TBar)
class MyClass { }

let obj = new MyClass();
obj.foo() // foo
obj.bar() // bar
```

上面程式碼中，通過traits修飾器，在`MyClass`類上面“混入”了`TFoo`類的`foo`方法和`TBar`物件的`bar`方法。

Trait不允許“混入”同名方法。

```javascript
import { traits } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};

@traits(TFoo, TBar)
class MyClass { }
// 報錯
// throw new Error('Method named: ' + methodName + ' is defined twice.');
//        ^
// Error: Method named: foo is defined twice.
```

上面程式碼中，TFoo和TBar都有foo方法，結果traits修飾器報錯。

一種解決方法是排除TBar的foo方法。

```javascript
import { traits, excludes } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};

@traits(TFoo, TBar::excludes('foo'))
class MyClass { }

let obj = new MyClass();
obj.foo() // foo
obj.bar() // bar
```

上面程式碼使用繫結運算子（::）在TBar上排除foo方法，混入時就不會報錯了。

另一種方法是為TBar的foo方法起一個別名。

```javascript
import { traits, alias } from 'traits-decorator';

class TFoo {
  foo() { console.log('foo') }
}

const TBar = {
  bar() { console.log('bar') },
  foo() { console.log('foo') }
};

@traits(TFoo, TBar::alias({foo: 'aliasFoo'}))
class MyClass { }

let obj = new MyClass();
obj.foo() // foo
obj.aliasFoo() // foo
obj.bar() // bar
```

上面程式碼為TBar的foo方法起了別名aliasFoo，於是MyClass也可以混入TBar的foo方法了。

alias和excludes方法，可以結合起來使用。

```javascript
@traits(TExample::excludes('foo','bar')::alias({baz:'exampleBaz'}))
class MyClass {}
```

上面程式碼排除了TExample的foo方法和bar方法，為baz方法起了別名exampleBaz。

as方法則為上面的程式碼提供了另一種寫法。

```javascript
@traits(TExample::as({excludes:['foo', 'bar'], alias: {baz: 'exampleBaz'}}))
class MyClass {}
```

## Babel轉碼器的支援

目前，Babel轉碼器已經支援Decorator。

首先，安裝`babel-core`和`babel-plugin-transform-decorators`。由於後者包括在`babel-preset-stage-0`之中，所以改為安裝`babel-preset-stage-0`亦可。

```bash
$ npm install babel-core babel-plugin-transform-decorators
```

然後，設定配置檔案`.babelrc`。

```javascript
{
  "plugins": ["transform-decorators"]
}
```

這時，Babel就可以對Decorator轉碼了。

指令碼中開啟的命令如下。

```javascript
babel.transform("code", {plugins: ["transform-decorators"]})
```

Babel的官方網站提供一個[線上轉碼器](https://babeljs.io/repl/)，只要勾選Experimental，就能支援Decorator的線上轉碼。
