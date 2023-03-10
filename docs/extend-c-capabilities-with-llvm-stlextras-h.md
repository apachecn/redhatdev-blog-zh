# 使用 LLVM STLExtras.h 扩展 C++功能

> 原文：<https://developers.redhat.com/blog/2019/10/18/extend-c-capabilities-with-llvm-stlextras-h>

[LLVM 编译器项目](https://llvm.org/)提供了一个名为 STLExtras.h 的[头文件，它扩展了 C++的功能，而不依赖于 LLVM 的其余部分。在本文中，我们快速浏览一下它的基本功能。](https://github.com/llvm-mirror/llvm/blob/master/include/llvm/ADT/STLExtras.h)

## 星际迷航舰级列表

对于`/usr/include/llvm/ADT/STLExtras.h`文件，您需要安装:

*   Fedora: `dnf install llvm-devel`
*   RHEL-8:  `yum install llvm-devel`
*   RHEL-7/8 带 [LLVM 工具组 7.0](https://developers.redhat.com/HW/ClangLLVM-RHEL-7/) : `yum install llvm-toolset-7.0-{clang,llvm-devel};scl enable llvm-toolset-7.0 -- clang++ -I$(scl enable llvm-toolset-7.0 -- llvm-config --includedir) -std=c++17 ...`

## llvm::反向

=反向基于范围的 for 循环

[C++11](https://en.wikipedia.org/wiki/C%2B%2B11) 带来了[基于范围的 for 循环](https://en.cppreference.com/w/cpp/language/range-for)。在最常见的情况下，我们不再需要处理迭代器。

参见[在线编译器示例代码。](https://godbolt.org/z/GgIB-R)

```
  std::vector vec{1, 2, 3, 4}; // C++11 initializer list

  // C++98 iterator: 1 2 3 4
  for (std::vector::iterator it = vec.begin(); it != vec.end(); ++it)
    std::cout << *it;

  // C++11 range-based for loop: 1 2 3 4
  for (int i : vec)
    std::cout << i; 
```

除...之外...当我们需要以相反的方式迭代时。C++11 不知何故忘记了这一点。LLVM 带着它的 [llvm::reverse](https://github.com/llvm-mirror/llvm/blob/cc0761d47c40e6b793b937d8af5c9bb517b5b7ba/include/llvm/ADT/STLExtras.h#L273) 容器适配器来了。

 参见[在线编译器示例代码。](https://godbolt.org/z/TX59Jv)

```
  std::vector vec{1, 2, 3, 4};

  // Reverse range-based for loop - llvm::reverse: 4 3 2 1
  for (int i : llvm::reverse(vec))
    std::cout << i;
} 
```

## 基于范围的算法

 = `vec.begin(),vec.end()` → `vec`

容器算法总是同时写`begin()`和`end()`有点烦吧？

```
  std::vector vec{1, 2, 3, 4};
  std::sort(vec.begin(),vec.end());
  // or:
  std::sort(vec.begin(),vec.end(), std::greater()); 
```

问题已经解决了；事情真的就这么简单:

 ```
#define LLVM_DISABLE_ABI_BREAKING_CHECKS_ENFORCING 1 // no LLVM libraries needed
#include <llvm/ADT/STLExtras.h>
  std::vector vec{1, 2, 3, 4};
  llvm::sort(vec);
  // or:
  llvm::sort(vec, std::greater()); 
```

适用于所有的 C++算法:[STD::sort](http://www.cplusplus.com/reference/algorithm/sort/)[STD::for _ each](http://www.cplusplus.com/reference/algorithm/for_each/)[STD::all _ of](http://www.cplusplus.com/reference/algorithm/all_of/)[STD::any _ of](http://www.cplusplus.com/reference/algorithm/any_of/)[STD::none _ of](http://www.cplusplus.com/reference/algorithm/none_of/)[STD::find](http://www.cplusplus.com/reference/algorithm/find/)STD::find _ if[STD::find _ if _ not](http://www.cplusplus.com/reference/algorithm/find_if_not/)STD::remove _ if[STD::copy _ if](http://www.cplusplus.com/reference/algorithm/copy_if/)[STD::copy](http://www.cplusplus.com/reference/algorithm/find/)

 在 C++20 中被期望为`std::ranges::`。

此外，它还提供:

*   [is_contained](https://github.com/llvm-mirror/llvm/blob/cc0761d47c40e6b793b937d8af5c9bb517b5b7ba/include/llvm/ADT/STLExtras.h#L1221) = [std::用`bool`结果找到](http://www.cplusplus.com/reference/algorithm/find/)
*   [is_splat](https://github.com/llvm-mirror/llvm/blob/cc0761d47c40e6b793b937d8af5c9bb517b5b7ba/include/llvm/ADT/STLExtras.h#L1306) =返回一个`bool`容器中的所有元素是否相同
*   [erase _ if](https://github.com/llvm-mirror/llvm/blob/cc0761d47c40e6b793b937d8af5c9bb517b5b7ba/include/llvm/ADT/STLExtras.h#L1324)=[STD::remove _ if](http://www.cplusplus.com/reference/algorithm/remove_if/)调用更方便(C++20 中预期的

参见[在线编译器示例代码。](https://godbolt.org/z/_WIEsA)

*Last updated: July 1, 2020*