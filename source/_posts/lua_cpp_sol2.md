---
title: Sol2实现Cpp和Lua绑定
date: 2018-06-20 12:29:01
tags:
- CPP
- Lua
categories:
- CPP
---



# Sol2简介

Sol是一个用于C++绑定Lua脚本的库，仅由头文件组成，方便集成，并提供了大量易用的API接口，可以便利地将Lua脚本与C++代码绑定起来，而不必去关心如何使用那些晦涩的Lua C API。正如其作者所言，Sol的目的就是提供极其简洁的API，并能高效到与C语言媲美，极大地来方便人们使用。

# 编译条件

Sol支持Lua的绝大多数版本，包括 5.1、5.2、5.3和LuaJit等，但由于代码中用到了许多C++11/14特性，因此编译时需要编译器支持C++14标准甚至C++17标准, 本人测试g++4.8.2无法编译过Sol2.20+的版本, 用g++6.2方能编过.

# 安装方法

去 https://github.com/ThePhD/sol2 下载一个sol.hpp , 然后放到 ` /usr/local/include ` 里即可

# 为什么强大

只需要包含一个sol.hpp头文件即可,需要任何其他的东西, 没有什么静态库/动态库之类的东西, 也不需要像tolua++一样那么麻烦每个类都要写pkg文件. 只需要稍微学习一下Sol2的导出API即可.

**. . .**<!--more -->


# 基础使用

从Sol的Github仓库clone下代码后，我们发现其目录下很多test开头的cpp/hpp文件，这些文件里面有着大量的Sol的使用示例以及各种特性的展示，而在example目录下的cpp文件都仅仅是一些最基础的使用示例。为了方便测试和体验Sol，你也可以自己建立一些自己的test.cpp文件，首先你要在源文件中include引用sol.hpp头文件，这样才能使用Sol提供的接口。而在使用gcc编译的时候，需要指定关联头文件的路径，可以使用类似于如下命令：

g++ test.cpp -Isolpath/single/sol -llua -std=c++1z


其中solpath是你Sol2的具体路径，在Sol2的项目目录下，有一个single/sol/sol.hpp头文件，这个头文件集成了所有的相关代码到一起，所以编译时 -I 后仅指定这一个路径就可以了，同时要保证你的gcc编译器支持C++14或17标准。


# 一个简单例子

例子目录结构如下 : 

    ├─test_sol2.cpp
    ├─assert.hpp
    ├─test_sol2.lua
    ├─sol.hpp

编译命令 : ` g++ *.cpp -llua -std=c++1z `

``` c++ test_sol2.cpp
#define SOL_CHECK_ARGUMENTS 1
#include "sol.hpp"

#include <iostream>
#include "assert.hpp"

int main()
{
	std::cout << "=== namespacing ===" << std::endl;

	struct my_class
	{
		int b = 24;

		int f() const
		{
			return 24;
		}

		void g()
		{
			++b;
		}
	};

	sol::state lua;
	lua.open_libraries();

	// "bark" namespacing in Lua
	// namespacing is just putting things in a table
	sol::table bark = lua.create_named_table("bark");
	bark.new_usertype<my_class>("my_class",
								"f", &my_class::f,
								"g", &my_class::g); // the usual

	// can add functions, as well (just like the global table)
	bark.set_function("print_my_class", [](my_class &self) { 
        std::cout << "my_class { b: " << self.b << " }" << std::endl; 
    });

	// // this works
	// lua.script("obj = bark.my_class.new()");
	// lua.script("obj:g()");

	// // calling this function also works
	// lua.script("bark.print_my_class(obj)");

	// load and execute from file
	lua.script_file("test_sol2.lua");

	my_class &obj = lua["obj"];
	c_assert(obj.b == 25);

	std::cout << (obj.b == 25 ? "assert success" : "assert fail") << std::endl;

	return 0;
}

```

``` c++ assert.hpp
#ifndef EXAMPLES_ASSERT_HPP
#define EXAMPLES_ASSERT_HPP

#ifdef SOL2_CI
struct pre_main {
        pre_main() {
                #ifdef _MSC_VER
                _set_abort_behavior(0, _WRITE_ABORT_MSG);
                #endif
        }
} pm;
#endif // Prevent lockup when doing Continuous Integration

#ifndef NDEBUG
#include <exception>
#include <iostream>
#include <cstdlib>

#   define m_assert(condition, message) \
    do { \
        if (! (condition)) { \
            std::cerr << "Assertion `" #condition "` failed in " << __FILE__ \
                      << " line " << __LINE__ << ": " << message << std::endl; \
            std::terminate(); \
        } \
    } while (false)

#   define c_assert(condition) \
    do { \
        if (! (condition)) { \
            std::cerr << "Assertion `" #condition "` failed in " << __FILE__ \
                      << " line " << __LINE__ << std::endl; \
            std::terminate(); \
        } \
    } while (false)
#else
#   define m_assert(condition, message) do { if (false) { (void)(condition); \
    (void)sizeof(message); } } while (false)
#   define c_assert(condition) do { if (false) { (void)(condition); } } while (false)
#endif

#endif // EXAMPLES_ASSERT_HPP
```

``` lua test_sol2.lua
obj = bark.my_class.new()
obj:g()
bark.print_my_class(obj)
```


## 打印结果

    === namespacing ===
    my_class { b: 25 }
    assert success
