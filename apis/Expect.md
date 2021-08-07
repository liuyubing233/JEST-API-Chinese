# Expect（预期）

在编写测试时，您经常需要检查值是否满足特定条件。`expect` 让您可以访问许多“matchers（匹配器）”，让您验证不同的东西。

对于 Jest 社区维护的其他 Jest 匹配器，请查看[Jest-extended](https://github.com/jest-community/jest-extended)

## methods - 方法

- [`expect(value)`](#expectvalue)
- [`expect.extend(matchers)`](#expectextendmatchers)
- [`expect.anything()`](#expectanything)
- [`expect.any(constructor)`](#expectanyconstructor)
- [`expect.arrayContaining(array)`](#expectarraycontainingarray)
- [`expect.assertions(number)`](#expectassertionsnumber)
- [`expect.hasAssertions()`](#expecthasassertions)
- [`expect.not.arrayContaining(array)`](#expectnotarraycontainingarray)
- [`expect.not.objectContaining(object)`](#expectnotobjectcontainingobject)
- [`expect.not.stringContaining(string)`](#expectnotstringcontainingstring)
- [`expect.not.stringMatching(string | regexp)`](#expectnotstringmatchingstring--regexp)
- [`expect.objectContaining(object)`](#expectobjectcontainingobject)
- [`expect.stringContaining(string)`](#expectstringcontainingstring)
- [`expect.stringMatching(string | regexp)`](#expectstringmatchingstring--regexp)
- [`expect.addSnapshotSerializer(serializer)`](#expectaddsnapshotserializerserializer)
- [`.not`](#not)
- [`.resolves`](#resolves)
- [`.rejects`](#rejects)
- [`.toBe(value)`](#tobevalue)

---

## Reference - 参考

### `expect(value)`

每次你想测试一个值时都会使用 `expect` 函数。您很少会单独调用 `expect` 。相反，您将使用 `expect` 和“matcher（匹配器）”函数来断言某个值。

举个例子更容易理解。假设您有一个方法 `bestLaCroixFlavor()`，它应该返回字符串 `"grapefruit"`。以下是测试的方法：

```javascript
test("the best flavor is grapefruit", () => {
  expect(bestLaCroixFlavor()).toBe("grapefruit");
});
```

在这个例子中，`toBe` 是匹配器函数，有许多不同的匹配器函数，以帮助您测试不同的东西。

`expect` 的参数是你的代码产生的值，并且匹配器的任何参数都应该是正确的值。如果你把它们混在一起，虽然测试仍可以工作，但是测试失败的错误消息看起来会很奇怪。

### `expect.extend(matchers)`

您可以使用 `expect.extend` 将您自己的匹配器添加到 Jest。例如，假设正在测试一个数字实用程序库，并且您经常断言数字出现在其他数字的特定范围内。您可以将其抽象为 `toBeWithinRange` 匹配器：

```javascript
expect.extend({
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be within range ${floor} - ${ceiling}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be within range ${floor} - ${ceiling}`,
        pass: false,
      };
    }
  },
});

test("numeric ranges", () => {
  expect(100).toBeWithinRange(90, 110);
  expect(101).not.toBeWithinRange(0, 100);
  expect({ apples: 6, bananas: 3 }).toEqual({
    apples: expect.toBeWithinRange(1, 10),
    bananas: expect.not.toBeWithinRange(11, 20),
  });
});
```

_注意_：在 TypeScript 中，例如使用 `@types/jest` 时，您可以像这样在导入的模块中声明新的 `toBeWithinRange` 匹配器：

```typescript
declare global {
  namespace jest {
    interface Matchers<R> {
      toBeWithinRange(a: number, b: number): R;
    }
  }
}
```

**_Async Matchers（异步匹配器）_**

`expect.extend` 还支持异步匹配器。异步匹配器返回一个 Promise，因此您需要等待返回的值。让我们使用一个示例匹配器来说明它们的用法。我们将实现一个名为 `toBeDivisibleByExternalValue` 的匹配器，其中的可整除数将从外部源中提取。

```javascript
expect.extend({
  async toBeDivisibleByExternalValue(received) {
    const externalValue = await getExternalValueFromRemoteSource();
    const pass = received % externalValue == 0;
    if (pass) {
      return {
        message: () =>
          `expected ${received} not to be divisible by ${externalValue}`,
        pass: true,
      };
    } else {
      return {
        message: () =>
          `expected ${received} to be divisible by ${externalValue}`,
        pass: false,
      };
    }
  },
});

test("is divisible by external value", async () => {
  await expect(100).toBeDivisibleByExternalValue();
  await expect(101).not.toBeDivisibleByExternalValue();
});
```

**_Custom Matchers API（自定义匹配器 API）_**

匹配器应该返回一个带有两个键的对象（或一个对象的 Promise）。`pass` 代表是否存在匹配，并且 `message` 提供了一个没有参数的函数，如果失败则返回错误消息。因此，当 `pass` 为 false 时， `message` 应该返回 `expect(x).yourMatcher()` 失败时的错误消息。当 `pass` 为 true 时，`message` 应该返回 `expect(x).not.yourMatcher()` 失败时的错误消息。

```javascript
expect.extend({
  yourMatcher(x, y, z) {
    return {
      pass: true,
      message: () => "",
    };
  },
});
```

这些辅助函数和属性可以在自定义匹配器 `this` 中找到：

**`this.isNot`**

布尔值，让您知道此匹配器是使用否定的 `.not` 修饰符调用的，可以显示清晰正确的匹配器提示（请参阅示例代码）。

**`this.promise`**

字符串，可以显示清晰正确的匹配器提示：

- 如果使用 promise `.rejects` 修饰符调用 `'rejects'` 匹配器
- 如果使用 promise `.resolves` 修饰符调用 `'resolves'` 匹配器
- 如果没有使用承诺修饰符调用 `''` 匹配器

**`this.equals(a, b)`**

这是一个深度相等函数，如果两个对象（a 和 b）具有相同的值（递归地），它将返回 `true`。

**`this.expand`**

布尔值，让你知道这个匹配器是用 `expand` 选项调用的。当使用 `--expand` 标志调用 Jest 时，`this.expand` 可用于确定 Jest 是否应显示完整的差异和错误。

**`this.utils`**

`this.utils` 上公开了许多有用的工具，主要由 [jest-matcher-utils](https://github.com/facebook/jest/tree/master/packages/jest-matcher-utils) 的导出组成。

最有用的是 `matcherHint`、`printExpected` 和 `printReceived` 来很好地格式化错误消息。举个例子，看一下 `toBe` 匹配器的实现：

```javascript
const { diff } = require("jest-diff");
expect.extend({
  toBe(received, expected) {
    const options = {
      comment: "Object.is equality",
      isNot: this.isNot,
      promise: this.promise,
    };

    const pass = Object.is(received, expected);

    const message = pass
      ? () =>
          this.utils.matcherHint("toBe", undefined, undefined, options) +
          "\n\n" +
          `Expected: not ${this.utils.printExpected(expected)}\n` +
          `Received: ${this.utils.printReceived(received)}`
      : () => {
          const diffString = diff(expected, received, {
            expand: this.expand,
          });
          return (
            this.utils.matcherHint("toBe", undefined, undefined, options) +
            "\n\n" +
            (diffString && diffString.includes("- Expect")
              ? `Difference:\n\n${diffString}`
              : `Expected: ${this.utils.printExpected(expected)}\n` +
                `Received: ${this.utils.printReceived(received)}`)
          );
        };

    return { actual: received, message, pass };
  },
});
```

这将打印这样的东西：

```javascript
  expect(received).toBe(expected)

    Expected value to be (using Object.is):
      "banana"
    Received:
      "apple"
```

当断言失败时，错误消息应向用户提供尽可能多的信息，以便他们能够快速解决问题。您应该编写精确的失败消息，以确保用户使用您的自定义断言时拥有良好的开发体验。

**_Custom snapshot matchers（自定义快照匹配器）_**

要在自定义匹配器中使用快照测试，您可以导入 `jest-snapshot` 并从匹配器中使用它。

举个例子，这是一个修剪字符串以存储指定长度功能的快照匹配器，`.toMatchTrimmedSnapshot(length)`：

```javascript
const { toMatchSnapshot } = require("jest-snapshot");

expect.extend({
  toMatchTrimmedSnapshot(received, length) {
    return toMatchSnapshot.call(
      this,
      received.substring(0, length),
      "toMatchTrimmedSnapshot"
    );
  },
});

it("stores only 10 characters", () => {
  expect("extra long string oh my god").toMatchTrimmedSnapshot(10);
});

/*
存储的快照看起来像这样：

exports[`stores only 10 characters: toMatchTrimmedSnapshot 1`] = `"extra long"`;
*/
```

它也可以为内联快照创建自定义匹配器，快照将正确添加到自定义匹配器中。但是，当第一个参数是属性匹配器时，内联快照将始终尝试附加到第一个参数或第二个参数，因此无法在自定义匹配器中接受自定义参数。

```javascript
const { toMatchInlineSnapshot } = require("jest-snapshot");

expect.extend({
  toMatchTrimmedInlineSnapshot(received, ...rest) {
    return toMatchInlineSnapshot.call(this, received.substring(0, 10), ...rest);
  },
});

it("stores only 10 characters", () => {
  expect("extra long string oh my god").toMatchTrimmedInlineSnapshot();
  /*
  快照将像这样被内联添加：
  expect('extra long string oh my god').toMatchTrimmedInlineSnapshot(
    `"extra long"`
  );
  */
});
```

**_async（异步）_**

如果您的自定义内联快照匹配器是异步的，即使用 `async` - `await`，您可能会遇到“Multiple inline snapshots for the same call are not supported（不支持同一调用的多个内联快照）”之类的错误。Jest 需要额外的上下文信息来查找自定义内联快照匹配器来正确的更新快照位置。

```javascript
const { toMatchInlineSnapshot } = require("jest-snapshot");

expect.extend({
  async toMatchObservationInlineSnapshot(fn, ...rest) {
    // 必须在 `await` 之前创建错误（及堆栈跟踪）
    this.error = new Error();

    // `observe` 的实现无关紧要
    // 重要的是自定义快照匹配器是异步的
    const observation = await observe(async () => {
      await fn();
    });

    return toMatchInlineSnapshot.call(this, recording, ...rest);
  },
});

it("observes something", async () => {
  await expect(async () => {
    return "async action";
  }).toMatchTrimmedInlineSnapshot();
  /*
    快照将像这样被内联添加：
    await expect(async () => {
      return 'async action';
    }).toMatchTrimmedInlineSnapshot(`"async action"`);
  */
});
```

**_Bail out（救助?）_**

通常 `jest` 会尝试匹配测试中预期的每个快照。

有时候，如果先前的快照失败，那么继续测试可能没有意义。比如当您在各种转换后制作状态机的快照时，一旦一个转换产生了错误的状态，您就可以中止测试。

在这种情况下，您可以实现一个自定义快照匹配器，它会抛出第一个不匹配的项而不是收集每个不匹配的内容。

```javascript
const { toMatchInlineSnapshot } = require("jest-snapshot");

expect.extend({
  toMatchStateInlineSnapshot(...args) {
    this.dontThrow = () => {};

    return toMatchInlineSnapshot.call(this, ...args);
  },
});

let state = "initial";

function transition() {
  // 执行中的错别字会导致测试失败
  if (state === "INITIAL") {
    state = "pending";
  } else if (state === "pending") {
    state = "done";
  }
}

it("transitions as expected", () => {
  expect(state).toMatchStateInlineSnapshot(`"initial"`);

  transition();
  // 已经产生了错误匹配，继续执行测试没有意义
  expect(state).toMatchStateInlineSnapshot(`"loading"`);

  transition();
  expect(state).toMatchStateInlineSnapshot(`"done"`);
});
```

### `expect.anything()`

`expect.anything()` 可以匹配除 `null` 或者 `undefined` 之外的任何内容。您可以在 `isEqual` 或者 `toBeCalledWith` 中使用它而不是使用文字值。举个例子，如果要检查是否使用了非空参数调用了模拟函数：

```javascript
test("map calls its argument with a non-null argument", () => {
  const mock = jest.fn();
  [1].map((x) => mock(x));
  expect(mock).toBeCalledWith(expect.anything());
});
```

### `expect.any(constructor)`

`expect.any(constructor)` 匹配使用给定构造函数创建的任何内容。您可以在 `isEqual` 或者 `toBeCalledWith` 中使用它而不是使用文字值。举个例子，如果要检查是否使用数字调用了模拟函数：

```javascript
function randomCall(fn) {
  return fn(Math.floor(Math.random() * 6 + 1));
}

test("randomCall calls its callback with a number", () => {
  const mock = jest.fn();
  randomCall(mock);
  expect(mock).toBeCalledWith(expect.any(Number));
});
```

### `expect.arrayContaining(array)`

`expect.arrayContaining(array)` 匹配接收到的数组，该数组包含预期数组中的所有元素。也就是说，预期数组是接收数组的**子集**。因此，接受数组中含有元素**不在**预期数组中它也匹配。

您可以使用它代替文字值：

- 在 `isEqual` 或者 `toBeCalledWith`
- 在 `objectContaining` 或者 `toMatchObject` 匹配属性

```javascript
describe("arrayContaining", () => {
  const expected = ["Alice", "Bob"];
  // 即使接收数组包含其它元素也匹配
  it("matches even if received contains additional elements", () => {
    expect(["Alice", "Bob", "Eve"]).toEqual(expect.arrayContaining(expected));
  });

  // 没有预期的 Alice 项则不匹配
  it("does not match if received does not contain expected elements", () => {
    expect(["Bob", "Eve"]).not.toEqual(expect.arrayContaining(expected));
  });
});
```

```javascript
describe("Beware of a misunderstanding! A sequence of dice rolls", () => {
  const expected = [1, 2, 3, 4, 5, 6];

  // 即使有其它的数字 7 也匹配
  it("matches even with an unexpected number 7", () => {
    expect([4, 1, 6, 7, 3, 5, 2, 5, 4, 6]).toEqual(
      expect.arrayContaining(expected)
    );
  });

  // 没有预期的数字 2 不匹配
  it("does not match without an expected number 2", () => {
    expect([4, 1, 6, 7, 3, 5, 7, 5, 4, 6]).not.toEqual(
      expect.arrayContaining(expected)
    );
  });
});
```

### `expect.assertions(number)`

`expect.assertions(number)` 用来验证在测试期间调用了一定数量的断言。它经常被用来测试异步代码，以确保回调中的断言确实被调用。

举个例子，我们有一个函数 `doAsync`，它接受两个回调 `callback1` 和 `callback2`，它将以未知的顺序异步地回调它们，我们可以用以下方法测试：

```javascript
test("doAsync calls both callbacks", () => {
  expect.assertions(2);
  function callback1(data) {
    expect(data).toBeTruthy();
  }
  function callback2(data) {
    expect(data).toBeTruthy();
  }

  doAsync(callback1, callback2);
});
```

`expect.assertions(2)` 用来确保两个回调都被实际调用。

### `expect.hasAssertions()`

`expect.hasAssertions()` 用来验证在测试期间至少调用了一个断言。它经常被用来测试异步代码，以确保回调中的断言确实被调用。

举个例子，假设我们有一些函数都处理状态，`prepareState` 使用状态对象调用回调，`validateState` 在该状态对象上运行，`waitOnState` 等待所有的 `prepareState` 完成返回一个 promise，我们可以用以下方法测试：

```javascript
test("prepareState prepares a valid state", () => {
  expect.hasAssertions();
  prepareState((state) => {
    expect(validateState(state)).toBeTruthy();
  });
  return waitOnState();
});
```

`expect.hasAssertions()` 用来确保实际调用了 `prepareState`。

### `expect.not.arrayContaining(array)`

`expect.not.arrayContaining(array)` 匹配接收到的数组不包含预期数组中所有元素。也就是说，预期数组不是接收数组的子集。

它是 `expect.arrayContaining` 的反函数。

```javascript
describe("not.arrayContaining", () => {
  const expected = ["Samantha"];

  it("matches if the actual array does not contain the expected elements", () => {
    expect(["Alice", "Bob", "Eve"]).toEqual(
      expect.not.arrayContaining(expected)
    );
  });
});
```

### `expect.not.objectContaining(object)`

`expect.not.objectContaining(object)` 匹配接收对象不递归匹配预期属性。也就是说，预期对象不是接收对象的子集。因此它匹配包含不在预期对象中的属性的接收对象。

它是 `expect.objectContaining` 的反函数。

```javascript
describe("not.objectContaining", () => {
  const expected = { foo: "bar" };

  it("matches if the actual object does not contain expected key: value pairs", () => {
    expect({ bar: "baz" }).toEqual(expect.not.objectContaining(expected));
  });
});
```

### `expect.not.stringContaining(string)`

`expect.not.stringContaining(string)` 匹配接收到的值**不是一个字符串**或者**是一个与预期值不匹配字符串**

它是 `expect.stringContaining` 的反函数。

```javascript
describe("not.stringContaining", () => {
  const expected = "Hello world!";

  it("matches if the received value does not contain the expected substring", () => {
    expect("How are you?").toEqual(expect.not.stringContaining(expected));
  });
});
```

### `expect.not.stringMatching(string | regexp)`

`expect.not.stringMatching(string | regexp)` 匹配接收到的值**不是一个字符串**或者**是一个与预期值不匹配的字符串或正则表达式**

它是 `expect.stringMatching` 的反函数。

```javascript
describe("not.stringMatching", () => {
  const expected = /Hello world!/;

  it("matches if the received value does not match the expected regex", () => {
    expect("How are you?").toEqual(expect.not.stringMatching(expected));
  });
});
```

### `expect.objectContaining(object)`

`expect.objectContaining(object)` 匹配接收对象递归匹配预期属性。也就是说，预期对象是接收对象的子集。因此它匹配包含存在于预期对象中的属性的接收对象。

您可以使用匹配器、`expect.anything()` 等来代替预期对象中的文字属性值。

举个例子，假设我们期望使用 `Event` 对象调用 `onPress` 函数，并且我们需要验证的是该事件是否具有 `event.x` 和 `event.y` 属性。我们可以这样做：

```javascript
test("onPress gets called with the right thing", () => {
  const onPress = jest.fn();
  simulatePresses(onPress);
  expect(onPress).toBeCalledWith(
    expect.objectContaining({
      x: expect.any(Number),
      y: expect.any(Number),
    })
  );
});
```

### `expect.stringContaining(string)`

`expect.stringContaining(string)` 匹配包含确切的值的预期字符串。

### `expect.stringMatching(string | regexp)`

`expect.stringMatching(string | regexp)` 匹配包含确切值的预期字符串和正则表达式。

您可以使用它代替文字值：

- 在 `isEqual` 或者 `toBeCalledWith`
- 在 `arrayContaining` 中匹配元素
- 在 `objectContaining` 或者 `toMatchObject` 匹配属性

这个例子展示了如何嵌套多个非对称匹配器，在 `expect.arrayContaining` 中使用 `expect.stringMatching`。

```javascript
describe("stringMatching in arrayContaining", () => {
  const expected = [
    expect.stringMatching(/^Alic/),
    expect.stringMatching(/^[BR]ob/),
  ];
  it("matches even if received contains additional elements", () => {
    expect(["Alicia", "Roberto", "Evelina"]).toEqual(
      expect.arrayContaining(expected)
    );
  });
  it("does not match if received does not contain expected elements", () => {
    expect(["Roberto", "Evelina"]).not.toEqual(
      expect.arrayContaining(expected)
    );
  });
});
```

### `expect.addSnapshotSerializer(serializer)`

您可以调用 `expect.addSnapshotSerializer` 来添加格式化特定于应用程序的数据结构模块。

对于单个测试文件，添加的模块位于 `snapshotSerializers（快照序列化器）` 配置中的任何模块之前，后者位于内置 JavaScript 类型和 React 元素的默认快照序列化程序之前。添加的最后一个模块是测试的第一个模块。

```javascript
import serializer from "my-serializer-module";
expect.addSnapshotSerializer(serializer);

// 影响测试文件中的 expect(value).toMatchSnapshot() 断言
```

如果您在单个测试文件中添加**快照序列化程序**而不是将其添加到 `snapshotSerializers` 配置中

- 您将使依赖显示引用而不是隐式引用
- 您避免了可能从 [create-react-app](https://github.com/facebook/create-react-app) 中弹出的配置限制。

更多信息请参阅 [configuring Jest](/apis/ConfiguringJest.md)。

### `.not`

如果你知道如何测试某些东西，`.not` 会让你测试它相反的值。例如，此代码测试 `best La Croix` 不是 `coconut`：

```javascript
test("the best flavor is not coconut", () => {
  expect(bestLaCroixFlavor()).not.toBe("coconut");
});
```

### `.resolves`

使用 `resolves` 来展开已经完成的的 promise 的值，以便链接任何其他匹配器。如果 promise 失败，则断言失败。

举个例子，此代码测试 promise 成功 resolves 并且返回值是 `'lemon'`

```javascript
test("resolves to lemon", () => {
  // 确保添加了 return 语句
  return expect(Promise.resolve("lemon")).resolves.toBe("lemon");
});
```

请注意，由于您仍在测试 promise，因此测试仍然是异步的。因此，您需要[告诉 Jest 等待](https://www.jestjs.cn/docs/asynchronous#promises)来通过返回未包装的断言。

或者，您可以将 `async/await` 和 `.resolves` 结合使用：

```javascript
test("resolves to lemon", async () => {
  await expect(Promise.resolve("lemon")).resolves.toBe("lemon");
  await expect(Promise.resolve("lemon")).resolves.not.toBe("octopus");
});
```

### `.rejects`

使用 `.rejects` 来展开 promise 失败的原因，以便链接任何其他匹配器。如果 promise 被实现，则断言失败。

举个例子，此代码测试 promise 是否以 `'octopus'` 为原因失败：

```javascript
test("rejects to octopus", () => {
  return expect(Promise.reject(new Error("octopus"))).rejects.toThrow(
    "octopus"
  );
});
```

请注意，由于您仍在测试 promise，因此测试仍然是异步的。因此，您需要[告诉 Jest 等待](https://www.jestjs.cn/docs/asynchronous#promises)来通过返回未包装的断言。

或者，您可以将 `async/await` 和 `.rejects` 结合使用：

```javascript
test("rejects to octopus", async () => {
  await expect(Promise.reject(new Error("octopus"))).rejects.toThrow("octopus");
});
```

### `.toBe(value)`
