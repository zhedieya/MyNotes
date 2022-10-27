# 代码检查

2019 年 1 月，[TypeScirpt 官方决定全面采用 ESLint](https://www.oschina.net/news/103818/future-typescript-eslint) 作为代码检查的工具，并创建了一个新项目 [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint)，提供了 TypeScript 文件的解析器 [@typescript-eslint/parser](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/parser) 和相关的配置选项 [@typescript-eslint/eslint-plugin](https://github.com/typescript-eslint/typescript-eslint/blob/master/packages/eslint-plugin) 等。而之前的两个 lint 解决方案都将弃用：

- [typescript-eslint-parser](https://github.com/eslint/typescript-eslint-parser) 已停止维护
- [TSLint](https://palantir.github.io/tslint/) 将提供迁移工具，并在 typescript-eslint 的功能足够完整后停止维护 TSLint（Once we consider ESLint feature-complete w.r.t. TSLint, we will deprecate TSLint and help users migrate to ESLint[1](https://medium.com/palantir/tslint-in-2019-1a144c2317a9)）

综上所述，目前以及将来的 TypeScript 的代码检查方案就是 [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint)。

## 什么是代码检查[§](https://ts.xcatliu.com/engineering/lint.html#什么是代码检查)

代码检查主要是用来发现代码错误、统一代码风格。

在 JavaScript 项目中，我们一般使用 [ESLint](https://eslint.org/) 来进行代码检查，它通过插件化的特性极大的丰富了适用范围，搭配 [typescript-eslint](https://github.com/typescript-eslint/typescript-eslint) 之后，甚至可以用来检查 TypeScript 代码。

## 为什么需要代码检查[§](https://ts.xcatliu.com/engineering/lint.html#为什么需要代码检查)

有人会觉得，JavaScript 非常灵活，所以需要代码检查。而 TypeScript 已经能够在编译阶段检查出很多问题了，为什么还需要代码检查呢？

因为 TypeScript 关注的重心是类型的检查，而不是代码风格。当团队的人员越来越多时，同样的逻辑不同的人写出来可能会有很大的区别：

- 缩进应该是四个空格还是两个空格？
- 是否应该禁用 `var`？
- 接口名是否应该以 `I` 开头？
- 是否应该强制使用 `===` 而不是 `==`？

这些问题 TypeScript 不会关注，但是却影响到多人协作开发时的效率、代码的可理解性以及可维护性。

下面来看一个具体的例子：

```ts
var myName = 'Tom';

console.log(`My name is ${myNane}`);
console.log(`My name is ${myName.toStrng()}`);
```

以上代码你能看出有什么错误吗？

分别用 tsc 编译和 eslint 检查后，报错信息如下：

```ts
var myName = 'Tom';
// eslint 报错信息：
// Unexpected var, use let or const instead.eslint(no-var)

console.log(`My name is ${myNane}`);
// tsc 报错信息：
// Cannot find name 'myNane'. Did you mean 'myName'?
// eslint 报错信息：
// 'myNane' is not defined.eslint(no-undef)
console.log(`My name is ${myName.toStrng()}`);
// tsc 报错信息：
// Property 'toStrng' does not exist on type 'string'. Did you mean 'toString'?
```

| 存在的问题                             | `tsc` 是否报错 | `eslint` 是否报错 |
| :------------------------------------- | :------------- | :---------------- |
| 应该使用 `let` 或 `const` 而不是 `var` | ❌              | ✅                 |
| `myName` 被误写成了 `myNane`           | ✅              | ✅                 |
| `toString` 被误写成了 `toStrng`        | ✅️              | ❌                 |

上例中，我们使用了 `var` 来定义一个变量，但其实 ES6 中有更先进的语法 `let` 和 `const`，此时就可以通过 `eslint` 检查出来，提示我们应该使用 `let` 或 `const` 而不是 `var`。

对于未定义的变量 `myNane`，`tsc` 和 `eslint` 都可以检查出来。

由于 `eslint` 无法识别 `myName` 存在哪些方法，所以对于拼写错误的 `toString` 没有检查出来。

由此可见，`eslint` 能够发现出一些 `tsc` 不会关心的错误，检查出一些潜在的问题，所以代码检查还是非常重要的。

## 在 TypeScript 中使用 ESLint[§](https://ts.xcatliu.com/engineering/lint.html#在-typescript-中使用-eslint)

### 安装 ESLint[§](https://ts.xcatliu.com/engineering/lint.html#安装-eslint)

ESLint 可以安装在当前项目中或全局环境下，因为代码检查是项目的重要组成部分，所以我们一般会将它安装在当前项目中。可以运行下面的脚本来安装：

```bash
npm install --save-dev eslint
```

由于 ESLint 默认使用 [Espree](https://github.com/eslint/espree) 进行语法解析，无法识别 TypeScript 的一些语法，故我们需要安装 [`@typescript-eslint/parser`](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/parser)，替代掉默认的解析器，别忘了同时安装 `typescript`：

```bash
npm install --save-dev typescript @typescript-eslint/parser
```

接下来需要安装对应的插件 [@typescript-eslint/eslint-plugin](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin) 它作为 eslint 默认规则的补充，提供了一些额外的适用于 ts 语法的规则。

```bash
npm install --save-dev @typescript-eslint/eslint-plugin
```

### 创建配置文件[§](https://ts.xcatliu.com/engineering/lint.html#创建配置文件)

ESLint 需要一个配置文件来决定对哪些规则进行检查，配置文件的名称一般是 `.eslintrc.js` 或 `.eslintrc.json`。

当运行 ESLint 的时候检查一个文件的时候，它会首先尝试读取该文件的目录下的配置文件，然后再一级一级往上查找，将所找到的配置合并起来，作为当前被检查文件的配置。

我们在项目的根目录下创建一个 `.eslintrc.js`，内容如下：

```js
module.exports = {
    parser: '@typescript-eslint/parser',
    plugins: ['@typescript-eslint'],
    rules: {
        // 禁止使用 var
        'no-var': "error",
        // 优先使用 interface 而不是 type
        '@typescript-eslint/consistent-type-definitions': [
            "error",
            "interface"
        ]
    }
}
```

以上配置中，我们指定了两个规则，其中 `no-var` 是 ESLint 原生的规则，`@typescript-eslint/consistent-type-definitions` 是 `@typescript-eslint/eslint-plugin` 新增的规则。

规则的取值一般是一个数组（上例中的 `@typescript-eslint/consistent-type-definitions`），其中第一项是 `off`、`warn` 或 `error` 中的一个，表示关闭、警告和报错。后面的项都是该规则的其他配置。

如果没有其他配置的话，则可以将规则的取值简写为数组中的第一项（上例中的 `no-var`）。

关闭、警告和报错的含义如下：

- 关闭：禁用此规则
- 警告：代码检查时输出错误信息，但是不会影响到 exit code
- 报错：发现错误时，不仅会输出错误信息，而且 exit code 将被设为 1（一般 exit code 不为 0 则表示执行出现错误）

### 检查一个 ts 文件[§](https://ts.xcatliu.com/engineering/lint.html#检查一个-ts-文件)

创建了配置文件之后，我们来创建一个 ts 文件看看是否能用 ESLint 去检查它。

创建一个新文件 `index.ts`，将以下内容复制进去：

```ts
var myName = 'Tom';

type Foo = {};
```

然后执行以下命令：

```bash
./node_modules/.bin/eslint index.ts
```

则会得到如下报错信息：

```bash
/path/to/index.ts
  1:1  error  Unexpected var, use let or const instead  no-var
  3:6  error  Use an `interface` instead of a `type`    @typescript-eslint/consistent-type-definitions

✖ 2 problems (2 errors, 0 warnings)
  2 errors and 0 warnings potentially fixable with the `--fix` option.
```

上面的结果显示，刚刚配置的两个规则都生效了：禁止使用 `var`；优先使用 `interface` 而不是 `type`。

需要注意的是，我们使用的是 `./node_modules/.bin/eslint`，而不是全局的 `eslint` 脚本，这是因为代码检查是项目的重要组成部分，所以我们一般会将它安装在当前项目中。

可是每次执行这么长一段脚本颇有不便，我们可以通过在 `package.json` 中添加一个 `script` 来创建一个 npm script 来简化这个步骤：

```json
{
    "scripts": {
        "eslint": "eslint index.ts"
    }
}
```

这时只需执行 `npm run eslint` 即可。

### 检查整个项目的 ts 文件[§](https://ts.xcatliu.com/engineering/lint.html#检查整个项目的-ts-文件)

我们的项目源文件一般是放在 `src` 目录下，所以需要将 `package.json` 中的 `eslint` 脚本改为对一个目录进行检查。由于 `eslint` 默认不会检查 `.ts` 后缀的文件，所以需要加上参数 `--ext .ts`：

`--ext`: 指定文件扩展名

```json
{
    "scripts": {
        "eslint": "eslint src --ext .ts"
    }
}
```

此时执行 `npm run eslint` 即会检查 `src` 目录下的所有 `.ts` 后缀的文件。