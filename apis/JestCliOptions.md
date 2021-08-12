# Jest CLI Options（Jest 命令行选项）

`jest` 命令行运行程序有许多有用的选项。您可以运行 `jest --help` 来查看所有可用选项。下面显示的许多选项也可以一起使用，以完全按照您想要的方式运行测试。 Jest 的每一个[配置选项](/apis/ConfiguringJest.md)也可以通过 CLI 指定。

以下是简要概述：

## Running from the command line（命令行运行）

运行测试（默认）：

```zsh
jest
```

仅运行指定模式或文件的测试：

```zsh
jest my-test #或者
jest path/to/my-test.js
```

基于 hg/git（未提交的文件）运行与更改文件相关的测试：

```zsh
jest -o
```

运行与 `path/to/fileA.js` 文件和 `path/to/fileB.js` 文件相关的测试：

```zsh
jest --findRelatedTests path/to/fileA.js path/to/fileB.js
```

运行与此规范名称匹配的测试（基本上与 `describe` 或 `test` 中的名称匹配）。

```zsh
jest -t name-of-spec
```

运行监视模式：

```zsh
jest --watch #默认运行 run jest -o
jest --watchAll #运行所有测试
```

监视模式还允许指定文件的名称或路径，以集中于一组特定的测试。

## Using with yarn（使用 yarn）

如果您通过 `yarn test` 运行 Jest，您可以将命令行参数直接作为 Jest 参数传递。

例如：

```zsh
jest -u -t="ColorPicker"
```

可以使用以下方法代替：

```zsh
yarn test -u -t="ColorPicker"
```

## Using with npm scripts（与 npm 脚本一起使用）

如果您通过 `npm test` 运行 Jest，您仍然可以通过在 `npm test` 和 Jest 参数之间插入 `--` 来使用命令行参数。

例如：

```zsh
jest -u -t="ColorPicker"
```

可以使用以下方法代替：

```zsh
npm test -- -u -t="ColorPicker"
```

## Camelcase & dashed args support（驼峰和虚线写法）

Jest 支持驼峰格式和虚线 arg 格式。以下示例结果相同：

```zsh
jest --collect-coverage
jest --collectCoverage
```

参数也可以混合使用：

```zsh
jest --update-snapshot --detectOpenHandles
```

## Options（选项）

_注意：CLI 选项优先于[配置](/apis/ConfiguringJest.md)中的值。_

- [`jest <regexForTestFiles>`](#jest-regexfortestfiles)
- [`--bail`](#--bail)
- [`--cache`](#--cache)
- [`--changedFilesWithAncestor`](#--changedfileswithancestor)
- [`--changedSince`](#--changedsince)
- [`--ci`](#--ci)
- [`--clearCache`](#--clearcache)
- [`--collectCoverageFrom=<glob>`](#--collectcoveragefromglob)
- [`--colors`](#--colors)
- [`--config=<path>`](#--configpath)
- [`--coverage[=<boolean>]`](#--coverageboolean)
- [`--coverageProvider=<provider>`](#--coverageproviderprovider)
- [`--debug`](#--debug)

---

## Reference（参考）

### `jest <regexForTestFiles>`

当您使用一个参数运行 `jest` 时，该参数将被视为正则表达式以匹配项目中的文件。可以通过提供模式来运行测试套件。只有模式匹配的文件才会被选取并执行。根据您的终端，您可能需要引用此参数：`jest "my.*(complex)?pattern"`。在 Windows 上，您需要使用 `/` 作为路径分隔符或将 `\` 转义为 `\\`。

### `--bail`

别名：`-b`。

在出现 `n` 个失败的测试套件时立即退出测试套件。默认为 `1`。

### `--cache`

是否使用缓存。默认为 true。使用 `--no-cache` 禁用缓存。注意：只有在遇到缓存相关问题时才应禁用缓存。一般来说，禁用缓存会使 Jest 至少慢两倍。

如果要检查缓存，请使用 `--showConfig` 并查看 `cacheDirectory` 值。如果需要清除缓存，请使用 `--clearCache`。

### `--changedFilesWithAncestor`

运行与当前更改和上次提交中所做更改相关的测试。行为类似于 `--onlyChanged`。

### `--changedSince`

运行提供的分支或提交散列的更改相关的测试。如果当前分支与给定分支有分歧，则只会测试本地所做的更改。行为类似于 `--onlyChanged`。

### `--ci`

提供此选项时，Jest 将假定它在 CI 环境中运行。这会在遇到新快照时更改行为。与自动存储新快照的常规行为不同，它会使测试失败并需要使用 `--updateSnapshot` 运行 Jest。

### `--clearCache`

删除 Jest 缓存目录，然后退出而不运行测试。如果该选项运行通过，将删除 `cacheDirectory（缓存目录）` 或 Jest 的默认缓存目录。默认缓存目录可以通过调用 `jest --showConfig` 找到。注意：清除缓存会降低性能。

### `--collectCoverageFrom=<glob>`

相对于 `rootDir` 的 glob 模式，匹配需要从中收集覆盖率信息的文件。

### `--colors`

强制测试结果输出高亮显示，即使标准输出不是 [TTY](https://baike.baidu.com/item/TTY)。

### `--config=<path>`

别名：`-c`。

指定查找和执行测试的配置文件的路径的规则。如果在配置中没有设置 `rootDir`，那么会假定包含配置文件的目录是项目的 `rootDir` 目录（根目录）。这也可以是 json 编码的值，Jest 将使用它作为配置。

### `--coverage[=<boolean>]`

别名：`--collectCoverage`

指示应该输出测试覆盖率信息的收集和报告。可选地传递 `<boolean>` 以覆盖配置中设置的选项。

### `--coverageProvider=<provider>`

指示应使用哪个提供程序来检测代码。允许的值为 `babel`（默认）或 `v8`。

请注意，`v8` 是实验性的。这使用了 V8 而不是基于 Babel 的代码覆盖率。它没有经过很好的测试，在 Node.js 的最后几个版本中也得到了改进。使用最新版本的 node（原文编写时为 v14）效果会更好。

### `--debug`

打印有关 Jest 配置的调试信息。
