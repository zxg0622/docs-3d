
## 语言支持

### Javascript

Cocos3D 对脚本中 Javascript 语言的支持为 ECMAScript 2015。

### Typescript

Cocos3D 对脚本中 Typescript 语言的支持为 Typescript 3.5.0。

一般情况下，标准的 Typescript 脚本由 `tsc` 编译，
`tsc` 提供了配置文件（一般名为`tsconfig.json`）以允许用户控制 `tsc` 的编译过程。
但在 Cocos3D 中，Typescript 脚本的编译完全由 Cocos3D 完成。
然而，Cocos3D 仍允许用户通过配置 `tsc` 配置文件的方式改变 Cocos3D 的编译行为，
以使得用户可以使用 Typescript 特有的某些功能，如模块解析等。

---

Cocos3D 在编译 Typescript 时就如同使用了以下 `tsc` 编译选项：
```json
{
    "compilerOptions": {
        // 必须保持以下属性使得 tsc 的检查行为和 Cocos3D 的编译行为一致
        "target": "es2015",
        "module": "es2015",
        // 可以自由地改变以下属性以约束 tsc 的检查行为：
        "experimentalDecorators": true,
        "strict": false,
        "noImplicitAny": false,
    }
}
```

除 `compilerOptions.target` 和 `compilerOptions.module`外，
你仍然可以改变上述的其它选项以实现（在 IDE 中）类型检查等功能，
但这些将不会影响 Cocos3D 对 Typescript 的编译结果。
例如，当你希望禁止你项目中所有 Typscript 脚本对隐式 `any` 的使用，
你就可以在 `tsconfig.json` 中将 `compilerOptions.noImplicitAny` 设为 `true`，
如此当你用 Visual Studio Code 等 IDE 检查该文件时就会收到相应的错误提示。

然而，请勿修改和设置 `compilerOptions.target` 和 `compilerOptions.module`，
例如，若将 `tsconfig.json` 设置为：
```json
{
    "compilerOptions": {
        "target": "es5",
        "module": "cjs"
    }
}
```
那么脚本代码：

```ts
const myModule = require("path-to-module");
```
在（使用 `tsc` 作为检查器的）IDE 中不会引起错误，因为`compilerOptions.module` 设置为了 `cjs`。
然而 Cocos3D 隐含的 `compilerOptions.module` 是 `es2015`，
因此在运行时它可能提示 "require 未定义" 等错误。

脚本代码：

```ts
const mySet = new Set();
```
对于 Cocos3D 来说是合法的，但 IDE 可能会报告错误：
因为`compilerOptions.target` 设置为了 `es5`：ECMAScript 2015 才引入 `Set`。

------

下列的 `tsc` 编译选项在 Cocos3D 中具有与 `tsc` 相同的含义，
这些选项也将影响 Cocos3D 对 Typescript 的编译：
- `compilerOptions.baseUrl`
- `compilerOptions.paths`

其余 `tsc` 选项你可以自由更改以实现类型检查、`.d.ts`文件关联等功能。

## 运行环境

Cocos3D 引擎的 API 都存在于模块 `Cocos3D`中，
使用标准的 ES6 模块导入语法将其导入：

```ts
import {
    Component, // 导入类 Component
    _decorator, // 导入命名空间 _decorator
} from "Cocos3D";
import * as cocos3d from "Cocos3D"; // 将整个 Cocos3D 模块导入为命名空间 cocos3d

@_decorator.ccclass("MyComponent")
export class MyComponent extends Component {
    public v = new cocos3d.Vec3();
}
```

## 保留标识符 cc

注意，由于历史原因，`cc` 是 Cocos3D 保留使用的标识符，
其行为*相当于*在任何模块顶部已经定义了名为 cc 的对象。
因此，你不应该将 `cc` 用作任何**全局**对象的名称：

```ts
/* const cc = {}; // 每个 Cocos3D 脚本都等价于在此处含有隐式定义 */

import * as cc from "Cocos3D"; // 错误：命名空间导入名称 cc 由 Cocos3D 保留使用

const cc = { x: 0 };
console.log(cc.x); // 错误：全局对象名称 cc 由 Cocos3D 保留使用

function f () {
    const cc = { x: 0 };
    console.log(cc.x); // 正确：cc 可以用作局部对象的名称

    const o = { cc: 0 };
    console.log(o.cc); // 正确：cc 可以用作属性名
}

console.log(cc, typeof cc); // 错误：行为是未定义的
```