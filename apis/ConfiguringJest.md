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
