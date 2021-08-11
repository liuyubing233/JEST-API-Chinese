# Configuring Jest（配置）

Jest 的配置可以在项目的 `package.json` 文件中定义，也可以通过 `jest.config.js` 或 `jest.config.ts` 文件或通过 `--config <path/to/file.js|ts|cjs|mjs|json>` 选项。如果你想使用 `package.json` 来存储 Jest 的配置，则应在顶层使用 `"jest"` 键，以便 Jest 知道如何找到你的设置：

```json
{
  "name": "my-project",
  "jest": {
    "verbose": true
  }
}
```

或者通过 JavaScript

```javascript
// jest.config.js
// 同步对象
/** @type {import('@jest/types').Config.InitialOptions} */
const config = {
  verbose: true,
};

module.exports = config;

// 或通过异步函数
module.exports = async () => {
  return {
    verbose: true,
  };
};
```

或者通过 TypeScript（如果已安装了 `ts-node`）

```ts
// jest.config.ts
import type { Config } from "@jest/types";

// 同步对象
const config: Config.InitialOptions = {
  verbose: true,
};
export default config;

// 或通过异步函数
export default async (): Promise<Config.InitialOptions> => {
  return {
    verbose: true,
  };
};
```

请记住，生成的配置必须是 JSON 可序列化的。

使用 `--config` 选项时，JSON 文件不能包含 `"jest"` 键：

```json
{
  "bail": 1,
  "verbose": true
}
```

## Options - 选项

这些选项可让你在 `package.json` 文件中控制 Jest 的行为。 Jest 的理念是默认良好运行，有时只需要配置额外的功能。

## Defaults - 默认

如果需要的话，你可以检索 Jest 的默认选项并扩展它们：

```js
// jest.config.js
const { defaults } = require("jest-config");
module.exports = {
  // ...
  moduleFileExtensions: [...defaults.moduleFileExtensions, "ts", "tsx"],
  // ...
};
```

// TODO: 最后创建目录

---

## Reference - 参考

### `automock` [boolean]

默认值： `false`

这个选项告诉 Jest 测试中所有导入的模块都应该自动模拟。测试中使用的所有模块都将具有替换实现，保证 API 在表面上。

举个例子：

```js
// utils.js
export default {
  authorize: () => {
    return "token";
  },
  isAuthorized: (secret) => secret === "wizard",
};
```

```js
//__tests__/automocking.test.js
import utils from "../utils";

test("if utils mocked automatically", () => {
  // `utils` 的公共方法现在是模拟函数
  expect(utils.authorize.mock).toBeTruthy();
  expect(utils.isAuthorized.mock).toBeTruthy();

  // 你可以提供自己的实现或传递预期的返回值
  utils.authorize.mockReturnValue("mocked_token");
  utils.isAuthorized.mockReturnValue(true);

  expect(utils.authorize()).toBe("mocked_token");
  expect(utils.isAuthorized("not_wizard")).toBeTruthy();
});
```

_注意：当你手动模拟时，节点模块会自动模拟（例如：`__mocks__/lodash.js`）。在[这里](https://jestjs.io/docs/manual-mocks#mocking-node-modules)查看更多信息。_

_注意：核心模块，如 `fs`，默认不会被模拟。它们可以被显式地模拟，比如 `jest.mock('fs')`。_

### `bail` [number | boolean]

默认值： `0`

默认情况下，Jest 运行所有测试并在完成后将所有错误生成到控制台中。在这里可以使用 `bail` 选项让 Jest 在 `n` 次失败后停止运行测试。将 `bail` 设置为 `true` 等于设置为 `1`。

### `cacheDirectory` [string]

默认值： `"/tmp/<path>"`

Jest 存储缓存的依赖信息的目录。

Jest 会预先尝试扫描你的依赖树一次并缓存它，以减少在运行测试时发生的一些文件系统倾斜。此配置选项可让你自定义 Jest 缓存数据在磁盘上的位置。

### `clearMocks` [boolean]

默认值：`false`

在每次测试之前自动清除模拟调用和实例。相当于在每次测试之前调用 `jest.clearAllMocks()`。这个方法不会删除所有可能已经完成的模拟任务。

### `collectCoverage` [boolean]

默认值：`false`

表示在执行测试时是否收集**覆盖率**信息。因为这会用**覆盖率收集语句**改造所有已执行的文件，所以它可能会显著减慢你的测试速度。

### `collectCoverageFrom` [array]

默认值：`undefined`

表示收集覆盖信息的一组文件的 [glob pattern](https://github.com/micromatch/micromatch)数组。如果文件与 glob pattern 模式匹配，即便是这个文件不在测试中并且测试文件也从不需要它，也会为其收集覆盖率信息。

举个例子：

```json
{
  "collectCoverageFrom": [
    "**/*.{js,jsx}",
    "!**/node_modules/**",
    "!**/vendor/**"
  ]
}
```

除了匹配 `**/node_modules/**` 或 `**/vendor/**` 中的文件，这也收集项目中所有文件（`.js` 和 `.jsx`）的覆盖率信息。

_注意：每个 glob pattern 都按照在配置中的顺序被应用。（例如 `["!**/__tests__/**", "**/*.js"]` 不会排除 `__tests__` 因为否定的方法被第二个模式覆盖。为了使否定的 glob 在这个例子中生效，顺序必须在 `**/*.js` 之后。）_

_注意：此选项需要将 `collectCoverage` 设置为 true 或使用 `--coverage` 调用 Jest。_

$\Downarrow$ Help:

如果你看到这样的输出...

```zsh
=============================== Coverage summary ===============================
Statements   : Unknown% ( 0/0 )
Branches     : Unknown% ( 0/0 )
Functions    : Unknown% ( 0/0 )
Lines        : Unknown% ( 0/0 )
================================================================================
Jest: Coverage data for global was not found.
```

很可能你的 glob pattern 模式与任何文件都不匹配。请参阅 [micromatch](https://github.com/micromatch/micromatch) 文档以匹配你的 glob。

### `coverageDirectory` [string]

默认值：`undefined`

Jest 输出包含测试文件范围的目录。

### `coveragePathIgnorePatterns` [array\<string\>]

默认值：`["/node_modules/"]`

格式为与文件路径匹配的正则格式的字符串数组，在执行测试之前进行匹配，如果文件路径与任何模式匹配，则将跳过该文件。

这些字符串与完整路径匹配。使用 `<rootDir>` 字符串标记包含项目根目录的路径，以防止它忽略不同环境中可能具有不同根目录的所有文件。示例：`["<rootDir>/build/", "<rootDir>/node_modules/"]`。

### `coverageProvider` [string]

指示应使用哪个提供程序来检测代码。允许的值为 `babel`（默认）或 `v8`。

请注意，`v8` 是实验性的。这使用了 V8 而不是基于 Babel 的代码覆盖率。它没有经过很好的测试，在 Node.js 的最后几个版本中也得到了改进。使用最新版本的 node（原文编写时为 v14）效果会更好。

### `coverageReporters` [array\<string | [string, options]\>]

默认值：`["json", "lcov", "text", "clover"]`

Jest 在生产报告时使用的 reporter 列表。可以使用任何 [istanbul reporter](https://github.com/istanbuljs/istanbuljs/tree/master/packages/istanbul-reports/lib)。

_注意：设置此选项会覆盖默认值。添加 `"text"` 或 `"text-summary"` 以在控制台输出中查看覆盖范围摘要。_

_注意：你可以使用元组形式将其他选项传递给 istanbul 报告器。例如：_

```json
["json", ["lcov", { "projectRoot": "../../" }]]
```

有关选项对象结构的其他信息，你可以参考[类型定义](https://github.com/facebook/jest/blob/master/packages/jest-types/src/Config.ts)中的 `CoverageReporterWithOptions` 类型。

### `coverageThreshold` [object]

默认值：`undefined`

这用于配置覆盖结果的最低阈值强制执行。阈值可以指定为 `global`、[glob](https://github.com/isaacs/node-glob#glob-primer) 以及目录或文件路径。如果未达到阈值，jest 将失败。指定为正数的阈值被视为所需的最小百分比。指定为负数的阈值表示允许的未覆盖实体的最大数量。

例如使用以下配置，如果分支、行和函数覆盖率低于 80%，或者有 10 个以上未覆盖的语句，jest 将失败：

```json
{
  ...
  "jest": {
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": -10
      }
    }
  }
}
```

如果 globs 或路径与 `global` 一起指定，则匹配路径的覆盖数据将从整体覆盖中减去，并且阈值将独立应用。 glob 的阈值适用于所有与 glob 匹配的文件。如果找不到路径指定的文件，则返回错误。

例如，使用以下配置：

```json
{
  ...
  "jest": {
    "coverageThreshold": {
      "global": {
        "branches": 50,
        "functions": 50,
        "lines": 50,
        "statements": 50
      },
      "./src/components/": {
        "branches": 40,
        "statements": 40
      },
      "./src/reducers/**/*.js": {
        "statements": 90
      },
      "./src/api/very-important-module.js": {
        "branches": 100,
        "functions": 100,
        "lines": 100,
        "statements": 100
      }
    }
  }
}
```

如果出现以下情况，Jest 将失败：

- `./src/components` 目录的分支或语句覆盖率不到 40%。
- 匹配 `./src/reducers/**/*.js` glob 的文件之一的语句覆盖率低于 90%。
- `./src/api/very-important-module.js` 文件的覆盖率低于 100%。
- 每个剩余文件的总覆盖率低于 50%（全局）。

### `dependencyExtractor` [string]

此选项允许使用自定义依赖项提取器。它必须是一个使用 `extract` 功能导出对象的节点模块。例如：\

```js
const crypto = require("crypto");
const fs = require("fs");

module.exports = {
  extract(code, filePath, defaultExtract) {
    const deps = defaultExtract(code, filePath);
    // 扫描文件并在`deps`（这是一个`Set`）中添加依赖项
    return deps;
  },
  getCacheKey() {
    return crypto
      .createHash("md5")
      .update(fs.readFileSync(__filename))
      .digest("hex");
  },
};
```

`extract` 函数应该返回一个可迭代对象（`Array`、`Set` 等），其中包含在代码中找到的依赖项。

该模块还可以包含一个 `getCacheKey` 函数来生成一个缓存键，以确定逻辑是否已更改，并且任何依赖于它的缓存工件都应该被丢弃。

### `displayName` [string, object]

默认值：`undefined`

允许在测试运行时在测试旁边打印标签。这在可能有许多 jest 配置文件的多项目存储库中变得更加有用。能直观地告诉我们测试属于哪个项目。以下是示例有效值：

```js
module.exports = {
  displayName: "CLIENT",
};
```

或

```js
module.exports = {
  displayName: {
    name: "CLIENT",
    color: "blue",
  },
};
```

作为次要选项，可以传递具有属性 `name` 和 `color` 的对象。这允许自定义配置 `displayName` 的背景颜色。当其值为字符串时，displayName 默认为白色。 Jest 使用 [chalk](https://github.com/chalk/chalk) 提供颜色。因此，jest 也支持 chalk 支持的所有颜色的有效选项。

### `errorOnDeprecated` [boolean]

默认值：`false`

调用已弃用的 API，抛出有用的错误消息。有助于简化升级过程。

### `extensionsToTreatAsEsm` [array\<string\>]

默认值：`[]`

Jest 将运行 `.mjs` 和 `.js` 文件，其中最近的 `package.json` 的 `type` 字段设置为 `module` 作为 ECMAScript 模块。如果你有任何其他文件应该使用本机 ESM 运行，你需要在这里指定它们的文件扩展名。

> 注意：Jest 的 ESM 支持仍处于试验阶段，有关更多详细信息，请参阅其[文档](https://jestjs.io/docs/ecmascript-modules)。

```json
{
  ...
  "jest": {
    "extensionsToTreatAsEsm": [".ts"]
  }
}
```

### `extraGlobals` [array\<string\>]

默认值： `undefined`

测试文件在 [vm](https://nodejs.org/api/vm.html) 内运行，这会减慢对全局属性（例如 `Math`）的调用。使用此选项时，你可以指定在 vm 内定义的额外属性，以加快查找速度。

例如，如果你的测试经常调用 `Math`，则可以通过设置 `extraGlobals` 来传递它。

```json
{
  ...
  "jest": {
    "extraGlobals": ["Math"]
  }
}
```

### `forceCoverageMatch` [array\<string\>]

默认值：`['']`

收集代码覆盖率时通常会忽略测试文件。使用此选项，你可以覆盖此行为并在代码覆盖率中包含其他被忽略的文件。

例如，如果你在以 `.t.js` 扩展名命名的源文件中有测试，如下所示：

```js
// sum.t.js

export function sum(a, b) {
  return a + b;
}

if (process.env.NODE_ENV === "test") {
  test("sum", () => {
    expect(sum(1, 2)).toBe(3);
  });
}
```

你可以通过设置 `forceCoverageMatch` 从这些文件中收集覆盖率。

```json
{
  ...
  "jest": {
    "forceCoverageMatch": ["**/*.t.js"]
  }
}
```

### `globals` [object]

默认值：`{}`

一组需要在所有测试环境中可用的全局变量。

例如，以下将创建一个在所有测试环境中设置为 `true` 的全局 `__DEV__` 变量：

```json
{
  ...
  "jest": {
    "globals": {
      "__DEV__": true
    }
  }
}
```

请注意，如果你在此处指定全局引用值（如对象或数组），并且某些代码在运行测试期间更改该值，则该更改将**不会**在其他测试文件的测试运行中持续存在。另外，`globals` 对象必须是 json-serializable，因此不能用于指定全局函数。如果想要实现该功能，请 `setupFiles`。

### `globalSetup` [string]

默认值：`undefined`

此选项允许使用自定义全局设置模块，该模块导出一个在所有测试套件之前触发一次的异步函数。该函数获取 Jest 的 `globalConfig` 对象作为参数。

_注意：只有当你从该项目运行至少一个测试时，才会触发在项目中配置的全局设置模块（使用多项目运行器）。_

_注意：任何通过 `globalSetup` 定义的全局变量只能在 `globalTeardown` 中读取。你无法在测试套件中检索此处定义的全局变量。_

_注意：当代码转换应用于链接的安装文件时，Jest **不会**转换 `node_modules` 中的任何代码。这是因为需要加载实际的转换器（例如 `babel` 或 `typescript`）来执行转换。_

举个例子：

```js
// setup.js
module.exports = async () => {
  // ...
  // 设置对 mongod 的引用，以便在拆解过程中关闭服务器
  global.__MONGOD__ = mongod;
};
```

```js
// teardown.js
module.exports = async function () {
  await global.__MONGOD__.stop();
};
```

### `globalTeardown` [string]

默认值：`undefined`

此选项允许使用自定义全局拆解模块，该模块导出一个在所有测试套件后触发一次的异步函数。该函数获取 Jest 的 `globalConfig` 对象作为参数。

_注意：在项目中配置的全局拆卸模块（使用多项目运行器）只有在你从该项目运行至少一个测试时才会被触发。_

_注意：关于 `node_modules` 转换与 `globalSetup` 相同的警告适用于 `globalTeardown`。_

### `haste` [object]

默认值：`undefined`

这将用于配置 [`jest-haste-map`](https://www.npmjs.com/package/jest-haste-map?activeTab=versions) 的行为，Jest 的内部文件爬虫/缓存系统。支持以下选项：

```js
type HasteConfig = {
  /** 是否使用 SHA-1 hash 文件 */
  computeSha1?: boolean,
  /** 用作默认的平台：例如 'ios' */
  defaultPlatform?: string | null,
  /** 强制使用 Node 的 `fs` API，而不是去寻找 `find` */
  forceNodeFilesystemAPI?: boolean,
  /**
   * 在抓取文件时是否遵循符号链接。
   * 此选项不能用于使用 watchman 的项目。
   * 如果此选项设置为 true，`watchman` 设置为 true 的项目将出错。
   */
  enableSymlinks?: boolean,
  /** Haste 自定义实现的路径 */
  hasteImplModulePath?: string,
  /** 要定位的所有平台, 例如 ['ios', 'android'] */
  platforms?: Array<string>,
  /** 是否在模块冲突时抛出错误 */
  throwOnModuleCollision?: boolean,
  /** 自定义 HasteMap 模块 */
  hasteMapModulePath?: string,
};
```

### `injectGlobals` [boolean]

默认值：`true`

将 Jest 的全局变量（`expect`、 `test`、 `describe`、 `beforeEach` 等）插入到全局环境中。如果将此设置为 `false`，则应从 `@jest/globals` 导入，例如：

```js
import { expect, jest, test } from "@jest/globals";

jest.useFakeTimers();

test("some test", () => {
  expect(Date.now()).toBe(0);
});
```

_注意：只有使用默认的 `jest-circus` 才支持此选项。_

### `maxConcurrency` [number]

默认值：`5`

限制使用 `test.concurrent` 时同时运行的测试数量。一旦插槽被释放，任何超过此限制的测试都将排队执行。

### `maxWorkers` [number | string]

指定工作池将为运行测试生成的最大工作线程数。在单次运行模式下，这默认占用你一个机器上可用内核的一个主线程。在监视模式下，这默认占用你机器上可用内核的一半，以确保 Jest 不引人注目并且不会使你的机器停止运行。在 CI 等资源有限的环境中进行调整可能很有用，但对于大多数情况来说，默认值应该足够了。

对于具有可变 CPU 可用的环境，你可以使用基于百分比的配置：`"maxWorkers"："50%"`

### `moduleDirectories` [array\<string\>]

默认值：`["node_modules"]`

要从所需模块的位置向上递归搜索的目录名称数组。设置此选项将覆盖默认值，如果你仍希望在 `node_modules` 中搜索包含它的包以及其他选项就这么设置：`["node_modules", "bower_components"]`

### `moduleFileExtensions` [array\<string\>]

默认值：`["js", "jsx", "ts", "tsx", "json", "node"]`

你的模块使用的文件扩展名数组。如果没有指定文件扩展名，这些是 Jest 将按从左到右的顺序查找的扩展名。

我们建议将项目中最常用的扩展放在左侧，因此如果你使用的是 TypeScript，你可能需要考虑将 `"ts"` 或 `"tsx"` 移动到数组的开头。

### `moduleNameMapper` [object<string, string | array\<string\>>]

默认值：`null`

从**正则表达式到模块名称**或**模块名称数组**的映射，允许使用单个模块提取资源，例如图像或样式。

映射到别名的模块在默认情况下是不进行模拟的的，无论是否启用 `automock`。

如果要使用文件路径，请使用 `<rootDir>` 字符串标记来引用 [rootDir](https://jestjs.io/docs/configuration#rootdir-string) 值。

此外，你可以使用反向引用的编号替换捕获的正则表达式组。

例如：

```json
{
  "moduleNameMapper": {
    "^image![a-zA-Z0-9$_-]+$": "GlobalImageStub",
    "^[./a-zA-Z0-9$_-]+\\.png$": "<rootDir>/RelativeImageStub.js",
    "module_name_(.*)": "<rootDir>/substituted_module_$1.js",
    "assets/(.*)": [
      "<rootDir>/images/$1",
      "<rootDir>/photos/$1",
      "<rootDir>/recipes/$1"
    ]
  }
}
```

定义映射的顺序很重要。模型被依次检查，直到匹配到。应该首先列出最具体的匹配规则。对于模块名称数组也是如此。

_注意：如果你提供没有边界的模块名称 `^$` 可能会导致难以发现错误。例如 `relay` 将替换所有包含的 `relay` 字符串的模块：`relay`、`react-relay` 和 `graphql-relay` 都将指向你的存根。_

### `modulePathIgnorePatterns` [array\<string\>]

默认值：`[]`

在这些路径之前所有与模块路径匹配的正则字符串数组对模块加载器被认为是“可见的”。如果给定的模块路径与任何模式匹配，则它在测试环境中将不支持 `require()`。

这些模式字符串与完整路径匹配。使用 `<rootDir>` 字符串标记包含项目根目录的路径，以防止它意外忽略不同环境中可能具有不同根目录的所有文件。示例：`["<rootDir>/build/"]`。

### `modulePaths` [array\<string\>]

默认值：`[]`

设置 `NODE_PATH` 环境变量的替代 API，`modulePaths` 是解析模块时要搜索的其他位置的绝对路径数组。使用 `<rootDir>` 字符串标记来包含项目根目录的路径。示例：`["<rootDir>/app/"]`。

### `notify` [boolean]

默认值：`false`

激活测试结果通知。

**注意**：Jest 使用 [node-notifier](https://github.com/mikaelbr/node-notifier) 来显示桌面通知。在 Windows 上，它会在第一次使用时创建一个新的开始菜单条目，并且不显示通知。通知将在后续运行中正确显示。

### `notifyMode` [string]

默认值：`failure-change`

指定通知的模式。需要设置 `notify: true`。

模式种类：

- `always`：总是发送通知。
- `failure`：测试失败时发送通知。
- `success`：测试成功时发送通知。
- `change`：状态改变时发送通知。
- `success-change`：测试失败一次或者成功时发送通知。
- `failure-change`：测试成功一次或者失败时发送通知。

### `preset` [string]

默认值：`undefined`

Jest 基础配置的预设。预设应该指向一个 npm 模块，该模块在根目录下有 `jest-preset.json`、`jest-preset.js`、`jest-preset.cjs` 或 `jest-preset.mjs` 文件。

例如，这个预设的 `foo-bar/jest-preset.js` 将配置如下：

```json
{
  "preset": "foo-bar"
}
```

预设也可能与文件系统路径有关。

```json
{
  "preset": "./node_modules/foo-bar/jest-preset.js"
}
```

### `prettierPath` [string]

默认值：`prettier`

设置用于更新内联快照的 [`prettier`](https://prettier.io/) 节点模块的路径。

### `projects` [array\<string | ProjectConfig\>]

默认值：`undefined`

当项目配置提供了一系列路径或全局模式时，Jest 将同时在所有指定的项目中运行测试。这对于单一仓库或同时处理多个项目时非常有用。

```json
{
  "projects": ["<rootDir>", "<rootDir>/examples/*"]
}
```

此示例配置将在根目录以及示例目录中的每个文件夹中运行 Jest。你可以在同一个 Jest 实例中运行无限数量的项目。

项目功能还可用于运行多个配置或多个运行程序。为此，你可以传递一组配置对象。例如，要在同一个 Jest 调用中同时运行测试和 ESLint（通过 [jest-runner-eslint](https://github.com/jest-community/jest-runner-eslint)）：

```json
{
  "projects": [
    {
      "displayName": "test"
    },
    {
      "displayName": "lint",
      "runner": "jest-runner-eslint",
      "testMatch": ["<rootDir>/**/*.js"]
    }
  ]
}
```

_注意：使用多项目运行器时，建议为每个项目添加一个 `displayName`。这将在测试旁边显示项目的 `displayName`。_

### `reporters` [array\<moduleName | [moduleName, options]\>]

默认值：`undefined`

使用此配置选项将自定义报告器添加到 Jest。自定义报告器是一个实现 `onRunStart`、`onTestStart`、`onTestResult`、`onRunComplete` 方法的类，这些方法将在这些事件发生时调用。

如果指定了自定义报告器，默认的 Jest 报告器将被覆盖。为了保留默认报告器，`default` 可以作为模块名称传递。

这将覆盖默认报告器：

```json
{
  "reporters": ["<rootDir>/my-custom-reporter.js"]
}
```

除了 Jest 提供的默认报告器之外，这还将使用自定义报告器：

```json
{
  "reporters": ["default", "<rootDir>/my-custom-reporter.js"]
}
```

此外，可以通过将选项对象作为第二个参数传递来配置自定义报告器：

```json
{
  "reporters": [
    "default",
    ["<rootDir>/my-custom-reporter.js", { "banana": "yes", "pineapple": "no" }]
  ]
}
```

自定义报告器模块必须定义一个类，该类将 `GlobalConfig` 和报告器选项作为构造函数参数：

例如这个报告器：

```js
// my-custom-reporter.js
class MyCustomReporter {
  constructor(globalConfig, options) {
    this._globalConfig = globalConfig;
    this._options = options;
  }

  onRunComplete(contexts, results) {
    console.log("Custom reporter output:");
    console.log("GlobalConfig: ", this._globalConfig);
    console.log("Options: ", this._options);
  }
}

module.exports = MyCustomReporter;
// 或 export default MyCustomReporter;
```

自定义报告器还可以通过用 `getLastError()` 强制 Jest 以非 0 代码返回错误

```js
class MyCustomReporter {
  // ...
  getLastError() {
    if (this._shouldFail) {
      return new Error("my-custom-reporter.js reported an error");
    }
  }
}
```

有关方法和参数类型的完整列表，请参阅 [packages/jest-reporters/src/types.ts](https://github.com/facebook/jest/blob/master/packages/jest-reporters/src/types.ts) 中的 `Reporter` 接口

### `resetMocks` [boolean]

默认值：`false`

每次测试前自动重置模拟状态。相当于在每次测试之前调用 `jest.resetAllMocks()`。这将导致任何模拟都会删除其虚假实现，但不会恢复其初始实现。

### `resetModules` [boolean]

默认值：`false`

默认情况下，每个测试文件都有自己独立的模块注册表。启用 `resetModules` 在运行每个单独的测试之前重置模块注册表。这对于隔离每个测试的模块非常有用，这样本地模块状态就不会在测试之间发生冲突。也可以使用 [`jest.resetModules()`](https://github.com/superPufferfish/JEST-API-Chinese/blob/main/apis/TheJestObject.md#jestresetmodules) 以编程方式完成。

### `resolver` [string]

默认值：`undefined`

此选项允许使用自定义解析器。这个解析器必须是一个节点模块，它导出一个函数，第一个参数为要解析的路径的字符串，第二个参数具有以下结构的对象：

```json
{
  "basedir": string,
  "defaultResolver": "function(request, options)",
  "extensions": [string],
  "moduleDirectory": [string],
  "paths": [string],
  "packageFilter": "function(pkg, pkgdir)",
  "rootDir": [string]
}
```

该函数应该返回解析的模块的路径，或者抛出找不到模块的错误。

注意：作为选项传递的 `defaultResolver` 是 Jest 默认解析器，这在你编写自定义解析器时可能很有用。它采用与你的自定义参数相同的参数，例如 `(request, options)`。

例如，如果要遵守 Browserify 的 [`browser` 字段](https://github.com/browserify/browserify-handbook/blob/master/readme.markdown#browser-field)，则可以使用以下配置：

```json
{
  ...
  "jest": {
    "resolver": "<rootDir>/resolver.js"
  }
}
```

```js
// resolver.js
const browserResolve = require("browser-resolve");

module.exports = browserResolve.sync;
```

通过结合 `defaultResolver` 和 `packageFilter` 我们可以实现 `package.json` “预处理器”，它允许我们更改默认解析器解析模块的方式。例如，假设如果存在 `"module"` 字段就使用它，否则回退到 `"main"`：

```json
{
  ...
  "jest": {
    "resolver": "my-module-resolve"
  }
}
```

```js
// my-module-resolve package

module.exports = (request, options) => {
  // 调用 defaultResolver，以便我们利用其缓存、错误处理等。
  return options.defaultResolver(request, {
    ...options,
    // 解析前使用 packageFilter 对解析后的 `package.json` 进行处理（https://www.npmjs.com/package/resolve#resolveid-opts-cb）
    packageFilter: pkg => {
      return {
        ...pkg,
        // 在解析包之前改变 `main` 的值
        main: pkg.module || pkg.main,
      };
    },
  });
};
```
