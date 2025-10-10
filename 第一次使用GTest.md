- [第一次使用GTest](#第一次使用gtest)
  - [介绍](#介绍)
  - [基本概念](#基本概念)
  - [最简单Test](#最简单test)
    - [相关API](#相关api)
  - [Test Fixtures(测试夹具)](#test-fixtures测试夹具)
  - [如何创建 fixture](#如何创建-fixture)
  - [如何使用 fixture](#如何使用-fixture)
  - [疑问](#疑问)


# 第一次使用GTest

## 介绍
GoogleTest 是由测试技术团队开发的测试框架，充分考虑了 Google 特定的需求和限制。无论您使用 Linux、Windows 还是 Mac，只要编写 C++ 代码，GoogleTest 都能为您提供帮助。它支持各种类型的测试，而不仅仅是单元测试。

## 基本概念

* Test Suite(测试套件): 是GTest中的一个术语, 一个Test Suite可以包含多个测试用例, 我们可以使用Test Suit作为不同测试类型进行分类
* fixture(夹具): 如果说多个 Test Suit要使用同一种object/配置, 那么可以把这些Test Test归到一个 fixture上, 这样多个Test Suit就可以共享同一个object


## 最简单Test

### 相关API
* ```ASSERT_*```: 跟Linux的assert一个道理, 只要语句为false就会中断当前测试函数执行, 跳到下一个测试函数
* ```EXPECT_*```: 如果与等式为false, 就会在运行时提示, 但是不会中断运行
* ```TEST(TestSuiteName, TestName)```  : 
  * 这个宏用于定义一个void类型函数, 该函数用于写测试内容
  * 第一个参数是声明所属的Test Suit
  * 第二个参数是定义当前测试函数名

例子:
```C++
TEST(FactorialTest, HandlesZeroInput) {
  EXPECT_EQ(Factorial(0), 1);
}

TEST(FactorialTest, HandlesIfEqual) {
  ASSERT_EQ(Factorial(0), 2);
}
```

上面2个测试函数 同属于FactorialTest


## Test Fixtures(测试夹具)

如果你写的很多测试用例都要使用到同一个配置，那么可以使用Test fixture, 
目的在于复用, 提高可维护性

## 如何创建 fixture

1. 定义一个fixture类, 需要继承自```testing::Test```, 并且所有成员都是```protected```访问属性，因为后续我们可以直接调用这些成员
2. 类中定义被共享的数据
3. 初始化数据: 可以使用类的构造函数或者重写```void SetUp()```虚函数(记住大小写不要写错, 怕写错可以加关键字```override```由编译器来检查), 
4. 析构释放: 可以使用类的析构函数或者重写```void TearDown()```

## 如何使用 fixture

需要使用 ```TEST_F()```

```C++
TEST_F(TestFixtureClassName, TestName) {
  ... test body ...
}
```

注意:
* 不同于```TEST()```, ```TEST_F()```第一个参数必须是 fixture类名, 否则编译器报错
* 定义```TEST_F()```前, 必须先声明对应的 fixture类, 因为每次调用```TEST_F()```前都会先定义对应类类型的变量供函数使用, 函数退出就会销毁

例子:

```C++
namespace MyGTest
{
    class FooTest : public testing::Test
    {
    protected:
        void SetUp() override
        {
            q1_.push(1);
            q1_.push(2);
            q2_.push(3);
            q2_.push(4);
        }

        void TearDown() override
        {}

        std::queue<int> q1_;
        std::queue<int> q2_;
    public:
        ~FooTest()
        {
            printf("~FooTest()\n");
        }
    };
    TEST_F(FooTest, sizeTest)
    {
        ASSERT_EQ(q1_.size(), 2)<< "err ? ";
        ASSERT_EQ(q2_.size(), 1);
        EXPECT_GT(q1_.size(), 3)<< "q1 greater than 2";
        EXPECT_EQ(q1_.size() == q2_.size(), true);
    }

    TEST_F(FooTest, sizeTest2)
    {
        ASSERT_EQ(q1_.size(), 2)<< "err ? ";
        ASSERT_EQ(q2_.size(), 2);
        EXPECT_EQ(q1_.size() == q2_.size(), true);
        EXPECT_GT(q1_.size(), 3)<< "q1 greater than 2";
    }
}
```

## 疑问

**说了这么多测试用法, 那么如何让这些函数运行起来呢？**

答案: 调用
```RUN_ALL_TESTS()```

那么```RUN_ALL_TESTS()```做了什么动作呢？
1. 保存传入的所有参数
2. 创建fixture
3. 并调用初始化函数```SetUp()```
4. 运行所有fixture
5. 调用```TearDown()```清理
6. 重复2 3 4 5


当然, 你是否会有这样的疑问: **我写了这么多测试用例, 还要我调用运行API让测试跑起来, 是不是太麻烦了？**

确实，这里有个更方便的做法无需写```main```函数直接让测试跑起来:  

**在编译时候链接上 ```libgtest_main```**, 由GTest自己将测试跑起来

