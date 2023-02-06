## Header Files

### 使用`#define`保护声明

所有头文件都应该使用`#define` 来保护声明，格式为：`*<PROJECT>*_*<PATH>*_*<FILE>*_H_`

例如：路径 `foo/src/bar/baz.h`下的头文件

```c++
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_

...

#endif  // FOO_BAR_BAZ_H_
```

### 不要依赖传递包含

如果 foo.cc 使用来自其他文件的符号，即使 foo.h 包含 bar.h，foo.cc 也应该包含 bar.h。

### 前向声明

尽量避免使用已经另一个项目中定义的实体的前向声明。

例如:如果 #include 被 B 和 D 的前向声明替换，test() 将调用 f(void*)

```c++
// b.h:
struct B {};
struct D : B {};

// good_user.cc:
#include "b.h"
void f(B*);
void f(void*);
void test(D* x) { f(x); }  // Calls f(B*)
```

### 内联函数

- 如果一个函数超过10行，不要内联（包括析构函数的长度）
- 在循环或者switch语句中，不要内联
- 虚函数和递归函数通常不会内联，即使它们被声明为`inline`

### #include顺序

1. 相关的头文件
2. C语言头文件
3. C++标准头文件
4. 其他库头文件
5. 自己项目内的头文件

注意：各个非空部分之间应该有一个空行分隔

例如：google-awesome-project/src/foo/internal/fooserver.cc

```c++
#include "foo/server/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <string>
#include <vector>

#include "base/basictypes.h"
#include "foo/server/bar.h"
#include "third_party/absl/flags/flag.h"
```

## Scoping

### 命名空间

- 命名空间应该具有基于项目名称及其路径的唯一名称  

- 不要使用 using 指令（例如，using namespace foo） 

- 不要使用内联命名空间

- 使用注释终止命名空间

  ```c++
  namespace mynamespace {
  
  // Definition of functions is within scope of the namespace.
  void MyClass::Foo() {
    ...
  }
  
  }  // namespace mynamespace
  ```

- 命名空间应该在#include、gflags定义/声明和来自其他命名空间的类的前向声明之后包装整个源文件

### 未命名的命名空间

当 .cc 文件中的定义不需要在该文件外部引用时，通过将它们放置在未命名的命名空间中或将它们声明为静态来为它们提供内部链接。 

不要在.h的头文件中使用

例如：

```c++
namespace {
...
}  // namespace
```

