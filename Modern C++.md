# Modern C++

## nullptr

空指针不使用`NULL`，而使用`nullptr`

## constexpr

常量表达式，用于编译期优化。

constexpr函数在C++14后可以在内部使用局部变量、循环和分支等简单语句

```c++
constexpr int fibonacci(const int n) {
  if (n == 1) {
    return 1;
  }
  if (n == 2) {
    return 1;
  }
  return fibonacci(n - 1) + fibonacci(n - 2);
}
```

```c++
  int len = 10;
  const int len2 = len + 1;
  constexpr int len3 = 2 + 1;
  //  constexpr int len2_constexpr = len2 + 1;
  //  非法，不能使用const只读变量初始化constexpr常量
  constexpr int len2_constexpr = len3 + 1;
  // 合法，可以使用constexpr常量定义constexpr常量
```

## if或switch语句中定义变量

可以在if和switch语句中定义临时变量

```c++
  std::vector<int> v{1, 2, 3, 4, 5};
  if (auto it = std::find(v.begin(), v.end(), 3); it != v.end()) {
    *it = 30;
  }
```

## 初始化列表

传统C++中，{}构造只能用于POD类和数组类型。现代C++还可以用于类对象。

使用{}来进行初始化，可用于对象的构造以及函数的参数传入。

注意：构造函数中不用特定声明`std::initializer_list<int> list`也可以使用{}构造

```c++
#include <iostream>
#include <vector>

class A {
public:
  A(int a_, int b_, int c_, std::initializer_list<int> list)
      : a(a_), b(b_), c(c_) {
    for (int num : list) {
      vec.push_back(num);
    }
  }

  void print(std::initializer_list<int> list) {
    for (int num : list) {
      std::cout << num << std::endl;
    }
  }

private:
  int a;
  int b;
  int c;
  std::vector<int> vec;
};

int main() {
  A a(1, 2, 3, {10, 20, 30});
  A a2{4, 5, 6, {40, 50, 60}};
  a.print({7, 8, 9});
}
```

## 结构化绑定

C++17后，可以使用`auto [x,y,z]`的形式接收元组(std::tuple<>)类型

```c++
#include <iostream>
#include <tuple>

std::tuple<int, double, std::string> f() {
  return std::make_tuple(1, 2.3, "456");
}

int main() {
  auto [x, y, z] = f();
  std::cout << x << ", " << y << ", " << z << std::endl;
  return 0;
}
```

## 函数模板返回值使用`auto`进行类型推导

```c++
template <class T, class U>
auto add(T n1,U n2){
  return n1 + n2;
}

int main(){
  auto sum = add(1,2.0);
  int n1 = 10;
  double n2 = 2.55;
  auto sum2 = add(n1,n2);
}
```

可以看见编译器为我们推导出了返回值的类型

![image-20230101214849660](E:\笔记\Modern C++.assets\image-20230101214849660.png)

## `decltype(auto)`

用于参数转发

## `if constexpr`

```c++
#include <iostream>

template<class T>
auto print_type_info(const T& t) {
    if constexpr (std::is_integral<T>::value) {
        return t + 1;
    } else {
        return t + 0.01;
    }
}

int main() {
    std::cout << print_type_info(1) << std::endl;
    std::cout << print_type_info(1.5) << std::endl;
}
```

编译后，会优化为

```c++
int print_type_info(const int& t) {
    return t + 1;
}
double print_type_info(const double& t) {
    return t + 0.001;
}
int main() {
    std::cout << print_type_info(5) << std::endl;
    std::cout << print_type_info(3.14) << std::endl;
}
```

## 外部模板

模板在每一个编译单元（也就是每一个文件）中都会实例化函数，有时候会有冗余。这时可以使用`extern`使用外部模板。

例如有两个cpp源文件，都是用了函数模板fun(int)，

```c++
// test1.cpp
#include "test.h"
template void fun<int>(int); // 显式地实例化 
void test1()
{ 
    fun(1);
}
```

```c++
// test2.cpp
#include "test.h"
extern template void fun<int>(int); // 外部模板的声明
void test2()
{
    fun(2);
}
```

此时在test2.cpp中声明fun为外部模板，编译器在链接test1.cpp和test2.cpp时就不会重复实例化`fun<int>`了，可以节省开销。

只有在项目比较大的情况下，才建议进行这样的优化。总的来说，就是在既不忽视模板实例化产生的编译及链接开销的同时，也不要过分担心模板展开的开销。

## `using`替代`typedef`

typedef在模板定义中，并不合法，例如

```c++
// 不合法
template<typename T>
typedef MagicType<std::vector<T>, std::string> FakeDarkMagic;
```

使用using的话，可以解决该问题

```c++
template<typename T>
using TrueDarkMagic = MagicType<std::vector<T>, std::string>;
```

此外，在用`using`命名指针时，更加直观。

```c++
typedef int (*process)(void *);  // typedef命名指针
using NewProcess = int(*)(void *);
```

## 变长参数模板

变长模板类和变长函数模板参数(变长参数解包较为麻烦)

```c++
#include <iostream>
#include <tuple>
#include <vector>

// 变长模板类
template<typename... Ts>
class Magic {};

Magic<int, std::vector<int>, std::tuple<int, int>> magic;

// 变长函数模板参数
// 解包方式为：变参模板展开
template<typename T0, typename... T>
void printf2(T0 t0, T... t) {
    std::cout << t0 << std::endl;
    if constexpr (sizeof...(t) > 0) printf2(t...);
}

// 初始化列表展开
// 下列式子的详解见
// https://blog.csdn.net/qq_44721831/article/details/127865419
template<typename T, typename... Ts>
auto printf3(T value, Ts... args) {
    std::cout << value << std::endl;
    (void) std::initializer_list<T>{([&args] {
        std::cout << args << std::endl;
    }(),
                                     value)...};
}

int main() {
    printf2(1, 2.0, 3.0l, "123");
    printf3(1, 2.0, 3.0l, "123");
}
```

## `override` 和 `final`

子类重载父类虚函数时，可以显式声明override，告诉编译器进行重载，编译器将检查基函数是否存在这样的虚函数，否则将无法通过编译。

```c++
#include <iostream>

class Base {
    virtual void foo();
};

class SubClass : public Base {
    void foo() override {
        std::cout << "foo" << std::endl;
    }

//    void foo2() override;  // 报错
};
```

`final` 则是为了防止类被继续继承以及终止虚函数继续重载引入的。

```c++
struct Base {
    virtual void foo() final;
};
struct SubClass1 final : Base {
};// 合法
struct SubClass2 : SubClass1 {
};// 非法, SubClass1 已final
struct SubClass3 : Base {
    void foo();// 非法, foo 已final
};
```

## `default` 和 `delete`

允许显式的声明采用或拒绝编译器自带的函数。

```c++
class A {
public:
    A() = default;// 显式声明使用默认构造函数
    A& operator=(const A&) = delete;  // 显式声明拒绝编译器生成构造
};
```

## Lambda表达式

Lambda 表达式的基本语法如下：
\[捕获列表](参数列表) mutable(可选) 异常属性-> 返回类型{
// 函数体
}

注意：捕获列表与参数列表不相同，捕获列表是获取函数外的数据，而参数列表就是函数所需的参数。

```c++
auto test() {
    auto ptr = std::make_unique<int>(5);
    auto value = [v1 = 1, v2 = std::move(ptr)](auto a1, auto a2) {
        std::cout << v1 + *v2 + a1 + a2 << std::endl;
    };
    value(1, 2);  // 1 + 5 + 1 + 2 = 9
}
```

注意：捕获列表中可以传入右值。

## 函数对象包装器

### `std::function`

`std::function` 是一种通用、多态的函数封装，它的实例可以对任何可以调用的目标实体进行存储、复制和调用操作，它也是对C++ 中现有的可调用实体的一种类型安全的包裹（相对来说，函数指针的调用不是类型安全的），换句话说，就是函数的容器。

```c++
#include <functional>
#include <iostream>

int foo(int para) {
    return para;
}

int main() {
    // std::function 包装了一个返回值为int, 参数为int 的函数
    std::function<int(int)> func = foo;
    int important = 10;
    std::function<int(int)> func2 = [&important](int value) -> int {
        return 1 + value + important;
    };
    std::cout << important << std::endl;
    std::cout << func(10) << std::endl;
    std::cout << func2(10) << std::endl;
}
```

### `std::bind` 和 `std::placeholder`

std::bind 是用来绑定函数调用的参数的，它解决的需求是我们有时候可能并不一定能够一次性获得调用某个函数的全部参数，通过这个函数，我们可以将部分调用参数提前绑定到函数身上成为一个新的对象，然后在参数齐全后，完成调用。

std::placeholder则是用来对参数进行占位。

```c++
#include <functional>
#include <iostream>

void foo(int a, int b, int c) {
    std::cout << "a: " << a << std::endl;
    std::cout << "b: " << b << std::endl;
    std::cout << "c: " << c << std::endl;
}
int main() {
    // 将参数1 绑定到函数foo 上，
    // 但使用std::placeholders::_1 来对第一个参数进行占位,
    // 使用std::placeholders::_2 来对第二个参数进行占位。
    // 下划线后表示的是调用函数时，传入的参数的位置（从1开始）
    auto bindFoo = std::bind(foo, 1, std::placeholders::_1, std::placeholders::_2);
    // 此时，std::placeholders::_1 = 5, std::placeholders::_2 = 6
    bindFoo(5, 6);
}
```

不过更推荐使用auto 配合 lambda 的方式取代std::bind

## 右值引用

右值：表达式结束后就不再存在的临时对象

右值又分为 纯右值和将亡值

### 纯右值

纯右值(prvalue, pure rvalue)，纯粹的右值，要么是纯粹的字面量，例如10, true；要么是求值结果相当于字面量或匿名临时对象，例如1+2。非引用返回的临时变量、运算表达式产生的临时变量、原始字面量、Lambda 表达式都属于纯右值。

注意：字面量除了字符串字面量以外，均为纯右值。而字符串字面量是一个左值，类型为const char 数组。

```c++
// 正确，"01234" 类型为const char [6]，因此是左值
const char (&left)[6] = "01234";
```

数组可以被隐式转换成相对应的指针类型，而转换表达式的结果（如果不是左值引用）则一定是个右值（右值引用为将亡值，否则为纯右值）

```c++
// 正确，"01234" 被隐式转换为const char*
const char* p = "01234";

// 正确，"01234" 被隐式转换为const char*，该转换的结果是纯右值
const char*&& pr = "01234";

// const char*& pl = "01234"; // 错误，此处不存在const char* 类型的左值
```

### 将亡值

将亡值(xvalue, expiring value)，是C++11 为了引入右值引用而提出的概念（因此在传统C++中，纯右值和右值是同一个概念），也就是即将被销毁、却能够被移动的值。

将亡值主要作用是用于移动语义，也就是移动构造，移动赋值。

### 右值引用

要拿到一个将亡值，就需要用到右值引用：T &&，其中T 是类型。右值引用的声明让这个临时值的生命周期得以延长、只要变量还活着，那么将亡值将继续存活。

```c++
#include <iostream>
#include <string>

void reference(std::string& str) {
    std::cout << " 左值" << std::endl;
}

void reference(std::string&& str) {
    std::cout << " 右值" << std::endl;
}
int main() {
    std::string lv1 = "string,";// lv1 是一个左值
    // std::string&& r1 = lv1; // 非法, 右值引用不能引用左值
    std::string&& rv1 = std::move(lv1);// 合法, std::move 可以将左值转移为右值
    std::cout << rv1 << std::endl;// string,
    const std::string& lv2 =
            lv1 + lv1;// 合法, 常量左值引用能够延长临时变量的生命周期
    // lv2 += "Test"; // 非法, 常量引用无法被修改
    std::cout << lv2 << std::endl;// string,string,
    std::string&& rv2 = lv1 + lv2;// 合法, 右值引用延长临时对象(lv1 + lv2)生命周期
    rv2 += "Test";// 合法, 非常量引用能够修改临时变量
    std::cout << rv2 << std::endl;// string,string,string,Test
    reference(rv2);               // 输出左值
    return 0;
}
```

需要注意的是：rv2 虽然引用了一个右值，但由于它是一个引用，所以rv2 依然是一个左值。

### 移动语义

区分与拷贝构造，移动语义也就是移动构造，不需要重新复制一份资源，只需移动，大大提升了效率。

```c++
#include <iostream>

class A {
public:
    int* pointer;
    A() : pointer(new int(1)) {
        std::cout << "构造:" << pointer << std::endl;
    }
    A(A& a) : pointer(new int(*a.pointer)) {
        std::cout << "拷贝::" << pointer << std::endl;
    }// 无意义的对象拷贝
    A(A&& a) : pointer(a.pointer) {
        a.pointer = nullptr;
        std::cout << "移动:" << pointer << std::endl;
    }
    ~A() {
        std::cout << "析构:" << pointer << std::endl;
        delete pointer;
    }
};
// 防止编译器优化
A return_rvalue(bool test) {
    A a, b;
    if (test)
        return a;// 等价于static_cast<A&&>(a);
    else
        return b;// 等价于static_cast<A&&>(b);
}

int main() {
    A obj = return_rvalue(false);
    std::cout << "obj: " << std::endl;
    std::cout << obj.pointer << std::endl;
    std::cout << *obj.pointer << std::endl;
    return 0;
}
```

1. 首先会在return_rvalue 内部构造两个A 对象，于是获得两个构造函数的输出；
2. 函数返回后，产生一个将亡值，被A 的移动构造（A(A&&)）引用，从而延长生命周期，并将这个右值中的指针拿到，保存到了obj 中，而将亡值的指针被设置为nullptr，防止了这块内存区域被销毁。

另外一个例子：使用std::move来进行移动，可以节省开销

```c++
#include <iostream>// std::cout
#include <string>  // std::string
#include <utility> // std::move
#include <vector>  // std::vector

int main() {
    std::string str = "Hello world.";
    std::vector<std::string> v;
    // 将使用push_back(const T&), 即产生拷贝行为
    v.push_back(str);
    // 将输出"str: Hello world."
    std::cout << "str: " << str << std::endl;
    // 将使用push_back(const T&&), 不会出现拷贝行为
    // 而整个字符串会被移动到vector 中，所以有时候std::move 会用来减少拷贝出现的开销
    // 这步操作后, str 中的值会变为空
    v.push_back(std::move(str));
    // 将输出"str: "
    std::cout << "str: " << str << std::endl;
    return 0;
}
```

### 完美转发

前面我们提到了，一个声明的右值引用其实是一个左值。这就为我们进行参数转发（传递）造成了问题

引用坍缩规则：

| 函数形参类型 | 实参参数类型 | 推导后函数形参类型 |
| :----------: | :----------: | :----------------: |
|      T&      |    左引用    |         T&         |
|      T&      |    右引用    |         T&         |
|     T&&      |    左引用    |         T&         |
|     T&&      |    右引用    |        T&&         |

所谓完美转发，就是为了让我们在传递参数的时候，保持原来的参数类型（左引用保持左引用，右引用保持右引用）。为了解决这个问题，我们应该使用std::forward来进行参数的转发（传递)

```c++
#include <iostream>
#include <utility>
void reference(int& v) {
    std::cout << " 左值引用" << std::endl;
}
void reference(int&& v) {
    std::cout << " 右值引用" << std::endl;
}
template<typename T>
void pass(T&& v) {
    std::cout << " 普通传参: ";
    reference(v);
    std::cout << " std::move 传参: ";
    reference(std::move(v));
    std::cout << " std::forward 传参: ";
    reference(std::forward<T>(v));
    std::cout << "static_cast<T&&> 传参: ";
    reference(static_cast<T&&>(v));
}
int main() {
    std::cout << " 传递右值:" << std::endl;
    pass(1);
    std::cout << " 传递左值:" << std::endl;
    int v = 1;
    pass(v);
    return 0;
}
```

输出结果为：

```
传递右值:
普通传参: 左值引用
std::move 传参: 右值引用
std::forward 传参: 右值引用
static_cast<T&&> 传参: 右值引用
传递左值:
普通传参: 左值引用
std::move 传参: 右值引用
std::forward 传参: 左值引用
static_cast<T&&> 传参: 左值引用
```

#### std::move

无论实参是左值还是右值，都会将实参作为右值传递

#### std::forward

既没有造成任何多余的拷贝，同时完美转发(传递) 了函数的实参给了内部调用的其他函数。

#### static_cast

static_cast<T&&>和std::forward的效果是一样的，二者也是等效的

见std::forward具体实现

```c++
template<typename _Tp>
constexpr _Tp&& forward(
        typename std::remove_reference<_Tp>::type& __t) noexcept {
    return static_cast<_Tp&&>(__t);
}

template<typename _Tp>
constexpr _Tp&& forward(
        typename std::remove_reference<_Tp>::type&& __t) noexcept {
    static_assert(!std::is_lvalue_reference<_Tp>::value,
                  "template argument"
                  " substituting _Tp is an lvalue reference type");
    return static_cast<_Tp&&>(__t);
}
```

当传入左值时，由于引用折叠（坍塌），& + && 返回 &

当传入右值时，& + && 返回 &&

这也是为什么在使用循环语句的过程中，**auto&&** 是最安全的方式？因为当auto 被推导为不同的左右引用时，与&& 的坍缩组合是完美转发。

## std::array

std::array 对象的大小是固定的，如果容器大小是固定的，那么可以优先考虑使用std::array 容器。另外由于std::vector 是自动扩容的，当存入大量的数据后，并且对容器进行了删除操作，容器并不会自动归还被删除元素相应的内存，这时候就需要手动运行shrink_to_fit() 释放这部分内存。

```c++
std::array<int, 4> arr = {1, 2, 3, 4};
arr.empty(); // 检查容器是否为空
arr.size(); // 返回容纳的元素数
// 迭代器支持
for (auto &i : arr)
{
// ...
}
// 用lambda 表达式排序
std::sort(arr.begin(), arr.end(), [](int a, int b) {
return b < a;
});
// 数组大小参数必须是常量表达式
constexpr int len = 4;
std::array<int, len> arr = {1, 2, 3, 4};
// 非法, 不同于C 风格数组，std::array 不会自动退化成T*
// int *arr_p = arr;
```

## std::unordered_map

当不需要排序时，unordered_map的效率高于map
