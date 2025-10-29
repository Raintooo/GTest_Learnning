- [gMock使用](#gmock使用)
  - [为什么需要 gMock？](#为什么需要-gmock)
  - [概念](#概念)
  - [使用](#使用)
  - [相关API](#相关api)


# gMock使用

gMock 是 Google Test（gTest）框架的扩展，专门用于创建和使用模拟对象（Mock Objects），是单元测试中隔离依赖、验证交互的强大工具。

## 为什么需要 gMock？  

在单元测试中，被测试的代码往往依赖其他组件（如数据库、网络服务、复杂算法等）。如果直接使用真实依赖，可能会导致：
* 测试速度慢（如操作数据库）；
* 测试不稳定（如依赖网络状态）；
* 无法覆盖所有场景（如模拟异常情况）。

gMock 可以创建模拟对象来替代真实依赖，通过预设 “期望”（如 “这个方法应该被调用 2 次，参数为 3 和 5”）和 “动作”（如 “调用后返回 8”），验证被测试代码与依赖的交互是否符合预期，同时隔离真实依赖的影响。

## 概念

* 模拟对象（Mock Object）：实现了待模拟接口的类，由 gMock 宏自动生成。
* 期望（Expectation）：定义 “模拟对象的某个方法应该被怎样调用”（如调用次数、参数、顺序等），通过 ```EXPECT_CALL``` 宏设置。
* 动作（Action）：定义 “当方法被调用时应该做什么”（如返回特定值、修改参数、抛出异常等），常用动作有 ```Return()、Invoke()``` 等。
* 参数匹配器（Matcher）：用于匹配方法调用的参数（如 “参数等于 5”“参数大于 10”），常用匹配器有 ```Eq()、Ge()```、```_```（任意参数）等。

## 使用

基本使用步骤
* 定义待模拟的接口：依赖的组件需抽象为接口（纯虚函数类）。
* 创建模拟类：用 gMock 宏 ```MOCK_METHOD``` 生成接口的模拟实现。
* 在测试中使用模拟对象：
    * 实例化模拟对象；
    * 用 ```EXPECT_CALL``` 设置期望和动作；
    * 将模拟对象注入被测试代码；
    * 执行测试逻辑；
    * gMock 自动验证期望是否满足（无需手动断言）。


如何创建模拟类呢？

举例子：
```C++
class MockDerive : public Base
{
public:
    MOCK_METHOD(void, test1, (), (override));
    MOCK_METHOD(void, test2, (int a), (override));
    MOCK_METHOD(void, test3, (int b, char c), (override));
    MOCK_METHOD(void, test4, (), (const, override));
};
```

* 模拟函数需要用```MOCK_METHOD```生成
* ```MOCK_METHOD``` ：
  * 第一个参数是函数返回值类型
  * 第二个参数是函数名
  * 第三个参数是函数参数列表
  * 第四个参数 如果是虚函数，就要在此标明，同时如果是```const```类型也要在此标明

创建了模拟类，怎么用呢？

```C++
#include <gmock/gmock.h>
#include <gtest/gtest.h>

using ::testing::AtLeast;                         // #1

TEST(PainterTest, CanDrawSomething) {
  MockDerive mock  ;                              // #2
  EXPECT_CALL(mock, test1())                  // #3
      .Times(AtLeast(1))
      .WillOnce(Return(1));
  Work work(&mock);                       // #4

  EXPECT_TRUE(mock.work());      // #5
}
```
这个例子定义了几个测试项：
* 期望了test1函数至少执1次
* 期望函数返回1

## 相关API

对于gMock, ```EXPECT_CALL```是最核心的宏, 我们可以利用它给函数设置一个“期望”
比如 被调用多少次，返回什么值等等，这些都称为“动作”，以下是一些常用的动作：

| Action                            | Description                                   |
| :-------------------------------- | :-------------------------------------------- |
| `Return()`                        | 返回 `void` 模拟函数.           |
| `Return(value)`                   | 返回 `value`. 如果 `value` 的类型跟模拟函数的参数不一致, `value` 就会在设置期望时被转换. |
| `ReturnArg<N>()`                  | 返回弟`N`个参数 (0-based) .         |
| `ReturnNew<T>(a1, ..., ak)`       | 返回 `new T(a1, ..., ak)`;每次返回都会创建对应实例对象. |
| `ReturnNull()`                    | 返回空指针.                        |
| `ReturnPointee(ptr)`              | 返回的值由 指针`ptr`指向.         |
| `ReturnRef(variable)`             | 返回的值由 引用`variable`指向.             |
| `ReturnRefOfCopy(value)`          | 返回一个副本由 `value`引用; |
| `ReturnRoundRobin({a1, ..., ak})` | 每次调用将返回列表中的下一个 a1，当到达列表末尾时，将从开头重新开始. |
