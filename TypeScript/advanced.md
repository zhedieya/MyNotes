#### 类型别名

类型别名用来给一个类型起个新名字。

简单的例子[§](https://ts.xcatliu.com/advanced/type-aliases.html#简单的例子)

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    } else {
        return n();
    }
}
```

上例中，我们使用 `type` 创建类型别名。

类型别名常用于联合类型。

#### 字符串字面量类型

字符串字面量类型用来约束取值只能是某几个字符串中的一个。

简单的例子[§](https://ts.xcatliu.com/advanced/string-literal-types.html#简单的例子)

```ts
type EventNames = 'click' | 'scroll' | 'mousemove';
function handleEvent(ele: Element, event: EventNames) {
    // do something
}

handleEvent(document.getElementById('hello'), 'scroll');  // 没问题
handleEvent(document.getElementById('world'), 'dblclick'); // 报错，event 不能为 'dblclick'

// index.ts(7,47): error TS2345: Argument of type '"dblclick"' is not assignable to parameter of type 'EventNames'.
```

上例中，我们使用 `type` 定了一个字符串字面量类型 `EventNames`，它只能取三种字符串中的一种。

注意，**类型别名与字符串字面量类型都是使用 `type` 进行定义。**

> 字面量：字面量（literal）是用于表达[源代码](https://baike.baidu.com/item/源代码/3969?fromModule=lemma_inlink)中一个固定值的表示法     🌰： let a = 1； let b = 'string'  这儿的1和‘string’都是字面量

#### 元组

数组合并了相同类型的对象，而元组（Tuple）合并了不同类型的对象。

元组起源于函数编程语言（如 F#），这些语言中会频繁使用元组。

简单的例子[§](https://ts.xcatliu.com/advanced/tuple.html#简单的例子)

定义一对值分别为 `string` 和 `number` 的元组：

```ts
let tom: [string, number] = ['Tom', 25];
```

当赋值或访问一个已知索引的元素时，会得到正确的类型：

```ts
let tom: [string, number];
tom[0] = 'Tom';
tom[1] = 25;

tom[0].slice(1);
tom[1].toFixed(2);
```

也可以只赋值其中一项：

```ts
let tom: [string, number];
tom[0] = 'Tom';
```

但是当直接对元组类型的变量进行初始化或者赋值的时候，需要提供所有元组类型中指定的项。

```ts
let tom: [string, number];
tom = ['Tom', 25];
let tom: [string, number];
tom = ['Tom'];

// Property '1' is missing in type '[string]' but required in type '[string, number]'.
```

当添加**越界**的元素时，它的类型会被限制为元组中每个类型的联合类型：

```ts
let tom: [string, number];
tom = ['Tom', 25];
tom.push('male');
tom.push(true);

// Argument of type 'true' is not assignable to parameter of type 'string | number'.
```

#### 枚举

枚举（Enum）类型用于取值被限定在一定范围内的场景

枚举成员会被赋值为从 `0` 开始递增的数字，同时也会对枚举值到枚举名进行反向映射：

```ts
enum Days {Sun, Mon, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 0); // true
console.log(Days["Mon"] === 1); // true

console.log(Days[0] === "Sun"); // true
console.log(Days[1] === "Mon"); // true
```

我们也可以给枚举项**手动赋值**：

```ts
enum Days {Sun = 7, Mon = 1, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 7); // true
console.log(Days["Tue"] === 2); // true
```

上面的例子中，未手动赋值的枚举项会接着上一个枚举项递增。

如果未手动赋值的枚举项与手动赋值的重复了，TypeScript 是不会察觉到这一点的：

```ts
enum Days {Sun = 3, Mon = 1, Tue, Wed, Thu, Fri, Sat};

console.log(Days["Sun"] === 3); // true
console.log(Days["Wed"] === 3); // true
console.log(Days[3] === "Sun"); // false
console.log(Days[3] === "Wed"); // true
```

上面的例子中，递增到 `3` 的时候与前面的 `Sun` 的取值重复了，但是 TypeScript 并没有报错，导致 `Days[3]` 的值先是 `"Sun"`，而后又被 `"Wed"` 覆盖了

手动赋值的枚举项可以不是数字，此时需要使用类型断言来让 tsc 无视类型检查 (编译出的 js 仍然是可用的)：

```ts
enum Days {Sun = 7, Mon, Tue, Wed, Thu, Fri, Sat = <any>"S"};
```

枚举项有两种类型：**常数项（constant member）和计算所得项（computed member）**

```ts
enum Color {Red, Green, Blue = "blue".length};
```

上面的例子中，`"blue".length` 就是一个计算所得项。

上面的例子不会报错，但是**如果紧接在计算所得项后面的是未手动赋值的项，那么它就会因为无法获得初始值而报错**：

```ts
enum Color {Red = "red".length, Green, Blue};

// index.ts(1,33): error TS1061: Enum member must have initializer.
// index.ts(1,40): error TS1061: Enum member must have initializer.
```

- **常数枚举**

  常数枚举是使用 `const enum` 定义的枚举类型：

  ```ts
  const enum Directions {
      Up,
      Down,
      Left,
      Right
  }
  
  let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
  ```

  常数枚举与普通枚举的区别是，它**会在编译阶段被删除**，并且不能包含计算成员。

  上例的编译结果是：

  ```js
  var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
  ```

  常数枚举如果包含了计算成员，则会在编译阶段报错：

  ```ts
  const enum Color {Red, Green, Blue = "blue".length};
  
  // index.ts(1,38): error TS2474: In 'const' enum declarations member initializer must be constant expression.
  ```

- **外部枚举**

  外部枚举（Ambient Enums）是使用 `declare enum` 定义的枚举类型：

  ```ts
  declare enum Directions {
      Up,
      Down,
      Left,
      Right
  }
  
  let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
  ```

  之前提到过，`declare` 定义的类型只会用于编译时的检查，编译结果中会被删除。

  上例的编译结果是：

  ```js
  var directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
  ```

  外部枚举与声明语句一样，常出现在声明文件`d.ts`中。

  同时使用 `declare` 和 `const` 也是可以的：

  ```ts
  declare const enum Directions {
      Up,
      Down,
      Left,
      Right
  }
  
  let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right];
  ```

  编译结果：

  ```ts
  var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
  ```



#### 类

```ts
// 使用 class 定义类，使用 constructor 定义构造函数。  
// constructor 是一种用于创建和初始化class创建的对象的特殊方法。
class Animal {
  public name
  constructor(name: string) {
    this.name = name
  }
  sayHi() {
    console.log(`My name is ${this.name}`)
  }
}

let dog = new Animal('Dog')
dog.sayHi() // My name is Dog
```

**继承 存取器getter和setter**

```ts
// 使用 extends 关键字实现继承，子类中使用 super 关键字来调用父类的构造函数和方法。
class Cat extends Animal {
  public _sex!: string
  constructor(name: string) {
    super(name)
  }
  // 使用 static 修饰符修饰的方法称为静态方法，它们不需要实例化，而是直接通过类来调用 静态方法是在创建类的时候就已经存在内存，直到程序结束才销毁，不用在每一个实例化中反复创建，节约内存空间
  static eat() : void {
    console.log('need to eat')
  }
  sayHi(): void {
    super.sayHi()
  }
  get sex(): string {
    return this._sex
  }
  set sex(value: string) {
    this._sex = value
  }
}
let cat : Cat = new Cat('Cat')
cat.sayHi() // Meeoww My name is Cat
// 如果get写死的话，set就不会生效
cat.sex = "female"
console.log(cat.sex) // female
// 直接通过类来调用
Cat.eat() // need to eat  
```

这里有个小bug，在定义sex的时候，若只单纯的`public _sex : string`这样定义，会报以下错误：

```
error TS2564: Property '_sex' has no initializer and is not definitely assigned in the constructor.
```

问题原因：

1.可能是属性的类型不对

2.可能是没有初始化

3.可能为undefined或者null(在ts中，这两个是单独的类型，是其他类型的[子类](https://so.csdn.net/so/search?q=子类&spm=1001.2101.3001.7020)型)

看了官网，大概有一下几种解决办法：

1.`tsconfig.json`配置以下设置，简单粗暴，但是超级不推荐。

```javascript
{
  "compilerOptions": {
    // 严格属性初始化
    "strictPropertyInitialization": false
  }

}
```

2.使用**非空断言**

```javascript
@Prop() option!: String;
```

3.使用联合类型

```javascript
@Prop() option: Object | undefined | null;
```

4.使用可选属性 属性后加“ ? ”

```javascript
@Prop() option?: Object;
```

**ES7 中有一些关于类的提案**，TypeScript 也实现了它们，做一个简单的介绍。

实例属性

ES6 中实例的属性只能通过构造函数中的 `this.xxx` 来定义，ES7 提案中可以直接在类里面定义：

```js
class Animal {
  name = 'Jack';

  constructor() {
    // ...
  }
}

let a = new Animal();
console.log(a.name); // Jack
```

静态属性

ES7 提案中，可以使用 `static` 定义一个静态属性：

```js
class Animal {
  static num = 42;

  constructor() {
    // ...
  }
}

console.log(Animal.num); // 42
```

**public private 和 protected**

TypeScript 可以使用三种访问修饰符（Access Modifiers），分别是 `public`、`private` 和 `protected`。

- `public` 修饰的属性或方法是公有的，可以在任何地方被访问到，默认所有的属性和方法都是 `public` 的
- `private` 修饰的属性或方法是私有的，不能在声明它的类的外部访问
- `protected` 修饰的属性或方法是受保护的，它和 `private` 类似，区别是它在**子类中**也是允许被访问的

> 👀 需要注意的是，TypeScript 编译之后的代码中，并没有限制 `private` 属性在外部的可访问性

修饰符和`readonly`还可以使用在构造函数参数中，等同于类中定义该属性同时给该属性赋值，使代码更简洁。

```ts
class Animal {
  // public name: string;
  public constructor(public name) {
    // this.name = name;
  }
}
```

**readonly **只读属性关键字，只允许出现在属性声明或索引签名(即任意属性)或构造函数中。

```ts
class Animal {
  public readonly name; // 如果 `readonly` 和其他访问修饰符同时存在的话，需要写在其后面。
  public constructor(name) {
    this.name = name;
  }
}

let a : Animal = new Animal('Jack');
console.log(a.name); // Jack
a.name = 'Tom';

// index.ts(10,3): TS2540: Cannot assign to 'name' because it is a read-only property.
```

**抽象类**

抽象类是不允许被实例化的，必须有子类继承抽象类，抽象类中的抽象方法必须被子类实现

```ts
// 抽象类
abstract class Abanimal {
  public name
  public constructor(name: string) {
    this.name = name
  }
  // 抽象方法
  public abstract sayHi(): void
}

class AbCat extends Abanimal {
  public sayHi(): void {
    console.log(this.name)
  }
}

let abCat : AbCat = new AbCat('Tom')
abCat.sayHi() // Tom
```



#### 类与接口

- **类实现接口**

  实现（implements）是面向对象中的一个重要概念。一般来讲，一个类只能继承自另一个类，有时候不同类之间可以有一些共有的特性，这时候就可以把特性提取成接口（interfaces），用 `implements` 关键字来实现。这个特性大大提高了面向对象的灵活性。

  ```ts
  interface Alarm {
      alert(): void;
  }
  
  interface Light {
      lightOn(): void;
      lightOff(): void;
  }
  
  // 一个类可以实现多个接口
  class Car implements Alarm, Light {
      alert() {
          console.log('Car alert');
      }
      lightOn() {
          console.log('Car light on');
      }
      lightOff() {
          console.log('Car light off');
      }
  }
  ```

- **接口继承接口**

  接口与接口之间可以是继承关系：

  ```ts
  interface Alarm {
      alert(): void;
  }
  
  interface LightableAlarm extends Alarm {
      lightOn(): void;
      lightOff(): void;
  }
  ```

  这很好理解，`LightableAlarm` 继承了 `Alarm`，除了拥有 `alert` 方法之外，还拥有两个新方法 `lightOn` 和 `lightOff`。

- **接口继承类**

  常见的面向对象语言中，接口是不能继承类的，但是在 TypeScript 中却是可以的：

  ```ts
  class Point {
      x: number;
      y: number;
      constructor(x: number, y: number) {
          this.x = x;
          this.y = y;
      }
  }
  
  interface Point3d extends Point {
      z: number;
  }
  
  let point3d: Point3d = {x: 1, y: 2, z: 3};
  ```

  当我们在声明 `class Point` 时，除了会创建一个名为 `Point` 的类之外，同时也创建了一个名为 `Point` 的类型（实例的类型）。

  所以我们既可以将 `Point` 当做一个类来用（使用 `new Point` 创建它的实例）：

  ```ts
  class Point {
      x: number;
      y: number;
      constructor(x: number, y: number) {
          this.x = x;
          this.y = y;
      }
  }
  
  const p = new Point(1, 2);
  ```

  也可以将 `Point` 当做一个**类型**来用（使用 `: Point` 表示参数的类型）：

  ```ts
  class Point {
      x: number;
      y: number;
      constructor(x: number, y: number) {
          this.x = x;
          this.y = y;
      }
  }
  
  function printPoint(p: Point) {
      console.log(p.x, p.y);
  }
  
  printPoint(new Point(1, 2));
  ```

  这个例子实际上可以等价于：

  ```ts
  class Point {
      x: number;
      y: number;
      constructor(x: number, y: number) {
          this.x = x;
          this.y = y;
      }
  }
  // Point的实例类型
  interface PointInstanceType {
      x: number;
      y: number;
  }
  
  function printPoint(p: PointInstanceType) {
      console.log(p.x, p.y);
  }
  
  printPoint(new Point(1, 2));
  ```

  上例中我们新声明的 `PointInstanceType` 类型，与声明 `class Point` 时创建的 `Point` 类型是等价的。

  所以回到 `Point3d` 的例子中，我们就能很容易的理解为什么 TypeScript 会支持接口继承类了：

  ```ts
  class Point {
      x: number;
      y: number;
      constructor(x: number, y: number) {
          this.x = x;
          this.y = y;
      }
  }
  
  interface PointInstanceType {
      x: number;
      y: number;
  }
  
  // 等价于 interface Point3d extends PointInstanceType
  interface Point3d extends Point {
      z: number;
  }
  
  let point3d: Point3d = {x: 1, y: 2, z: 3};
  ```

  当我们声明 `interface Point3d extends Point` 时，`Point3d` 继承的实际上是类 `Point` 的实例的类型，换句话说，可以理解为定义了一个接口 `Point3d` 继承另一个接口 `PointInstanceType`。

  所以「接口继承类」和「接口继承接口」没有什么本质的区别。

  > 声明 `Point` 类时创建的 `Point` 类型只包含其中的实例属性和实例方法,在接口继承类的时候，也只会继承它的实例属性和实例方法



#### 声明合并

> 如果定义了两个相同名字的函数、接口或类，那么它们会合并成一个类型：

- **函数的合并**

  我们可以使用重载定义多个函数类型：

  ```ts
  function reverse(x: number): number;
  function reverse(x: string): string;
  function reverse(x: number | string): number | string {
      if (typeof x === 'number') {
          return Number(x.toString().split('').reverse().join(''));
      } else if (typeof x === 'string') {
          return x.split('').reverse().join('');
      }
  }
  ```

- **接口的合并**

  接口中的属性在合并时会简单的合并到一个接口中：

  ```ts
  interface Alarm {
      price: number;
  }
  interface Alarm {
      weight: number;
  }
  ```

  相当于：

  ```ts
  interface Alarm {
      price: number;
      weight: number;
  }
  ```

  注意，**合并的属性的类型必须是唯一的**：

  ```ts
  interface Alarm {
      price: number;
  }
  interface Alarm {
      price: number;  // 虽然重复了，但是类型都是 `number`，所以不会报错
      weight: number;
  }
  interface Alarm {
      price: number;
  }
  interface Alarm {
      price: string;  // 类型不一致，会报错
      weight: number;
  }
  
  // index.ts(5,3): error TS2403: Subsequent variable declarations must have the same type.  Variable 'price' must be of type 'number', but here has type 'string'.
  ```

  接口中方法的合并，与函数的合并一样：

  ```ts
  interface Alarm {
      price: number;
      alert(s: string): string;
  }
  interface Alarm {
      weight: number;
      alert(s: string, n: number): string;
  }
  ```

  相当于：

  ```ts
  interface Alarm {
      price: number;
      weight: number;
      alert(s: string): string;
      alert(s: string, n: number): string;
  }
  ```

- **类的合并**

  类的合并与接口的合并规则一致。
