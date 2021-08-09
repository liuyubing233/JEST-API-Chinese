# Mock Functions（模拟函数）

Mock Functions（模拟函数）也被称为“spies（间谍）”，因为它们让你可以监视由其他代码间接调用的函数行为，而不仅仅是测试输出。你可以使用 `jest.fn()` 创建一个模拟函数。如果未给出实现，则模拟函数在调用时将返回 `undefined`。

## methods - 方法

- [Reference - 参考](#reference---参考)
  - [`mockFn.getMockName()`](#mockfngetmockname)
  - [`mockFn.mock.calls`](#mockfnmockcalls)
  - [`mockFn.mock.results`](#mockfnmockresults)
  - [`mockFn.mock.instances`](#mockfnmockinstances)
  - [`mockFn.mockClear()`](#mockfnmockclear)
  - [`mockFn.mockReset()`](#mockfnmockreset)
  - [`mockFn.mockRestore()`](#mockfnmockrestore)
  - [`mockFn.mockImplementation(fn)`](#mockfnmockimplementationfn)
  - [`mockFn.mockImplementationOnce(fn)`](#mockfnmockimplementationoncefn)
  - [`mockFn.mockName(value)`](#mockfnmocknamevalue)

---

### Reference - 参考

下面的 `mockFn` 是模拟函数的名称

#### `mockFn.getMockName()`

返回通过调用 `mockFn.mockName(value)` 设置的模拟名称字符串。

#### `mockFn.mock.calls`

包含对此模拟函数进行的调用的所有回调参数的数组。数组中的每一项都是调用时传递的参数数组。

例如：一个模拟函数 `f` 被调用了两次，一次 `f('arg1', 'arg2')`，然后 `f('arg3', 'arg4')`，`mock.calls` 数组看起来像这样：

```javascript
[
  ["arg1", "arg2"],
  ["arg3", "arg4"],
];
```

#### `mockFn.mock.results`

一个包含对此模拟函数所有回调结果的数组。该数组中的每一项都是一个包含 `type` 属性和 `value` 属性的对象。`type` 将是以下之一：

- `'return'`：表示调用正常返回开完成调用。
- `'throw'`：表示调用抛出值来完成调用。
- `'incomplete'`：表示呼叫尚未完成。如果从模拟函数本身或从模拟调用的函数中测试结果，则会发生这种情况。

`value` 属性是包含 return 或者 throw 的值，如果 `type === 'incomplete'`，那么 `value` 为 undefined。

举个例子，一个模拟函数 `f` 被调用了三次，返回 `'result1'`，抛出一个错误然后返回 `'result2'`，`mock.results` 数组看起来像这样：

```javascript
[
  {
    type: "return",
    value: "result1",
  },
  {
    type: "throw",
    value: {
      /* 错误内容 */
    },
  },
  {
    type: "return",
    value: "result2",
  },
];
```

#### `mockFn.mock.instances`

一个数组包含使用 `new` 从此模拟函数实例化的所有对象实例。

举个例子，一个模拟函数已经被实例化两次，那么 `mock.instances` 看起来像这样：

```javascript
const mockFn = jest.fn();

const a = new mockFn();
const b = new mockFn();

mockFn.mock.instances[0] === a; // true
mockFn.mock.instances[1] === b; // true
```

#### `mockFn.mockClear()`

重置所有存储在 `mockFn.mock.calls` 和 `mockFn.mock.instances` 数组中的信息。

当想要清理两个断言之间的模拟数据时，这通常很有用。

请注意，`mockClear` 将取代 `mockFn.mock`，而不仅仅是 `mockFn.mock.calls` 和 `mockFn.mock.instances`。所以无论是不是临时分配，模拟函数应该避免将 `mockFn.mock` 分配给其他变量。以确保模拟函数不会访问过时的数据。

[`clearMocks`](/apis/ConfiguringJest.md) 配置选项可用于在测试之间自动清除模拟。

#### `mockFn.mockReset()`

执行 `mockFn.mockClear()` 的所有方法，并删除所有模拟的返回值或实现。

当你想将模拟完全重置回初始状态时，这很有用。（注意：重置 spy 将导致函数没有返回值）。

请注意，`mockReset` 将取代 `mockFn.mock`，而不仅仅是 `mockFn.mock.calls` 和 `mockFn.mock.instances`。所以无论是不是临时分配，模拟函数应该避免将 `mockFn.mock` 分配给其他变量。以确保模拟函数不会访问过时的数据。

#### `mockFn.mockRestore()`

执行 `mockFn.mockRestore()` 的所有方法，并恢复了所有原始实现（非模拟的）。

当你想在使用模拟函数在某些测试用例中，并在其他测试用例中恢复原始实现时，这很有用。

请注意，`mockFn.mockRestore` 仅在使用 `jest.spyOn` 创建模拟时才有效。因此，你必须在手动分配 `jest.fn()` 时自行处理恢复。

[`restoreMocks`](/apis/ConfiguringJest.md) 配置选项可用于在测试之间自动恢复模拟。

#### `mockFn.mockImplementation(fn)`

接受一个应用用作模拟实现的函数。模拟本身仍然会记录所有进入的调用和来自自身的实例——唯一的区别是调用模拟时也会执行实现。

_注意： `jest.fn(implementation)` 是 `jest.fn().mockImplementation(implementation)` 的简写。_

```javascript
const mockFn = jest.fn().mockImplementation((scalar) => 42 + scalar);
// or: jest.fn(scalar => 42 + scalar);

const a = mockFn(0);
const b = mockFn(1);

a === 42; // true
b === 43; // true

mockFn.mock.calls[0][0] === 0; // true
mockFn.mock.calls[1][0] === 1; // true
```

`mockImplementation` 也可用于模拟类构造函数：

```javascript
// SomeClass.js
module.exports = class SomeClass {
  m(a, b) {}
};

// OtherModule.test.js
jest.mock("./SomeClass"); // 这会通过 automocking 自动发生
const SomeClass = require("./SomeClass");
const mMock = jest.fn();
SomeClass.mockImplementation(() => {
  return {
    m: mMock,
  };
});

const some = new SomeClass();
some.m("a", "b");
console.log("Calls to m: ", mMock.mock.calls);
```

#### `mockFn.mockImplementationOnce(fn)`

接受一个函数，该函数将用作对模拟函数的一次调用的实现。可以进行链接，以便多个函数调用产生不同的结果。

```javascript
const myMockFn = jest
  .fn()
  .mockImplementationOnce((cb) => cb(null, true))
  .mockImplementationOnce((cb) => cb(null, false));

myMockFn((err, val) => console.log(val)); // true

myMockFn((err, val) => console.log(val)); // false
```

当模拟函数用完用 `mockImplementationOnce` 定义的实现时，如果调用 `jest.fn(() => defaultValue)` 或 `.mockImplementation(() => defaultValue)`，它将执行默认实现：

```javascript
const myMockFn = jest
  .fn(() => "default")
  .mockImplementationOnce(() => "first call")
  .mockImplementationOnce(() => "second call");

// 'first call', 'second call', 'default', 'default'
console.log(myMockFn(), myMockFn(), myMockFn(), myMockFn());
```

#### `mockFn.mockName(value)`

在测试结果中使用字符串代替 `jest.fn()` 以指示引用的模拟函数。

```javascript
const mockFn = jest.fn().mockName("mockedFunction");
// mockFn();
expect(mockFn).toHaveBeenCalled();
```

会导致这个错误：

```zsh
expect(mockedFunction).toHaveBeenCalled()

Expected mock function "mockedFunction" to have been called, but it was not called.
```
