# RoboMaster视觉笔记CMake（五）build-type



本CMake系列是依据github上的cmake-examples进行翻译总结。同时对于不懂的地方进行总结与标注。希望本系列能节省你学习CMake的时间。

学习方式是在实践中利用github上的example学习，同时对于不懂的地方主要通过翻译官方手册学习，其次是查找博客上的私人理解。

[英文github地址](https://github.com/ttroy50/cmake-examples)：https://github.com/ttroy50/cmake-examples

CMake英文官方教程： https://cmake.org/cmake/help/latest/guide/tutorial/index.html 

[TOC]

## 一  文件树

```tree
├── CMakeLists.txt
├── main.cpp
```

### 1.1  main.cpp

```c
#include <iostream>

int main(int argc, char *argv[])
{
   std::cout << "Hello Build Type!" << std::endl;
   return 0;
}

```

### 1.2  CMakeLists.txt

```cmake

cmake_minimum_required(VERSION 3.5)
#如果没有指定则设置默认编译方式
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  #在命令行中输出message里的信息
  message("Setting build type to 'RelWithDebInfo' as none was specified.")
  #不管CACHE里有没有设置过CMAKE_BUILD_TYPE这个变量，都强制赋值这个值为RelWithDebInfo
  set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
  
  # 当使用cmake-gui的时候，设置构建级别的四个可选项
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()


project (build_type)
add_executable(cmake_examples_build_type main.cpp)

#命令的具体解释在二  CMake解析中，这里的注释只说明注释后每一句的作用
```

## 二  CMake解析

### 2.1  构建级别

CMake具有许多内置的构建配置，可用于编译工程。 这些配置指定了代码优化的级别，以及调试信息是否包含在二进制文件中。 

这些优化级别，主要有：

* Release —— 不可以打断点调试，程序开发完成后发行使用的版本，占的体积小。 它对代码做了优化，因此速度会非常快， 

	在编译器中使用命令： `-O3 -DNDEBUG` 可选择此版本。

* Debug ——调试的版本，体积大。

	在编译器中使用命令： `-g` 可选择此版本。

* MinSizeRel—— 最小体积版本

	在编译器中使用命令：`-Os -DNDEBUG`可选择此版本。

* RelWithDebInfo—— 既优化又能调试。

	在编译器中使用命令：`-O2 -g -DNDEBUG`可选择此版本。

### 2.2 设置级别的方式

 

####  2.2.1  CMake图形界面

![cmake-gui-build-type](D:\improve\微信公众号文章\6\RoboMaster视觉笔记CMake（五）build-type.assets\cmake-gui-build-type.png)

#### 2.2.2  CMake命令行中

在命令行运行CMake的时候， 使用cmake命令行的-D选项配置编译类型 

```cmake
cmake .. -DCMAKE_BUILD_TYPE=Release
```

#### 2.2.3  CMake中设置默认的构建级别

 CMake提供的默认构建类型是不进行优化的构建级别。 对于某些项目，需要自己设置默认的构建类型，以便不必记住进行设置。 

具体语法接下来介绍

####  2.2.4  set()命令

该命令可以为普通变量、缓存变量、环境变量赋值。

 <value>处可以设置零个或多个参数。多个参数将以[分号分隔的列表](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#cmake-language-lists)形式加入，以形成要设置的实际变量值。零参数将导致未设置普通变量。见[`unset()`](https://cmake.org/cmake/help/latest/command/unset.html#command:unset) 命令显式取消设置变量。 

所以此处学习SET命令需要分为设置普通变量，缓存变量以及环境变量三种类别来学习。

#####  正常变量

```cmake
set(<variable> <value>... [PARENT_SCOPE])
```

设置的变量值 作用域属于整个 CMakeLists.txt 文件。(一个工程可能有多个CMakeLists.txt)

当这个语句中加入PARENT_SCOPE后，表示要设置的变量是父目录中的CMakeLists.txt设置的变量。

比如有如下目录树：

```cmake
├── CMakeLists.txt
└── src
    └── CMakeLists.txt
```

并且在 顶层的CMakeLists.txt中包含了src目录：`add_subdirectory(src)`

那么，顶层的CMakeLists.txt就是父目录，

如果父目录中有变量`Bang`,在子目录中可以直接使用（比如用message输出`Bang`，值是父目录中设置的值）并且利用set()修改该变量`Bang`的值，但是如果希望在出去该子CMakeLists.txt对该变量做出的修改能够得到保留，那么就需要在set()命令中加入Parent scope这个变量。当然，如果父目录中本身没有这个变量，子目录中仍然使用了parent scope，那么出了这个作用域后，该变量仍然不会存在。

#####  CACHE变量

完整语句如下：

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
```

* 首先什么是CACHE变量，就是在运行cmake的时候，变量的值可能会被缓存到一份文件里即build命令下的CMakeCache.txt，当你重新运行cmake的时候，那些变量会默认使用这个缓存里的值。这个变量是全局变量，整个CMake工程都可以使用该变量。

* 在这个文件里，只要运行cmake ..命令，自动会出现一些值，比如 CMAKE_INSTALL_PREFIX ，如果设置 set(CMAKE_INSTALL_PREFIX "/usr") ，虽然CACHE缓存文件里还有这个CMAKE_INSTALL_PREFIX 变量，但是因为我们显示得设置了一个名为CMAKE_INSTALL_PREFIX 的正常变量，所以之后使用CMAKE_INSTALL_PREFIX ，值是我们设置的正常变量的值。

* 如果加上CACHE关键字，则设置的这个变量会被写入缓存文件中（但如果本身缓存文件中有这个变量，则不会覆盖缓存中的变量）。只有加上FORCE关键字，这个被写入文件的值会覆盖之前文件中存在的同名变量。
* 加上CACHE关键字，<type>和<docstring>是必需的。 

<type>被 CMake GUI 用来选择一个窗口，让用户设置值。<type>可以有5种选项。其中一个是STRING  ，弹出提示消息

* <type>为BOOL，则<docstring>为布尔`ON/OFF`值。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供一个复选框。

*  <type>为FILEPATH，则<docstring>为磁盘上文件的路径。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供一个文件对话框。

* <type>为 PATH ，则<docstring>为磁盘上目录的路径。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供一个文件对话框。

*  <type>为 STRING ，则<docstring>为一行文字。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1)) 提供文本字段或下拉选择（如果 [`STRINGS`](https://cmake.org/cmake/help/latest/prop_cache/STRINGS.html#prop_cache:STRINGS) 设置了缓存条目属性。）

* <type>为 INTERNAL ，则<docstring>为一行文字。 [`cmake-gui(1)`](https://cmake.org/cmake/help/latest/manual/cmake-gui.1.html#manual:cmake-gui(1))不显示内部条目。它们可用于在运行之间持久存储变量。使用此类型暗含`FORCE`。

比如

```cmake
set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
```

这句话，就是强制在缓存文件中覆盖CMAKE_BUILD_TYPE这个变量，将这个变量设置为RelWithDebInfo。而`STRING "Choose the type of build."`参数在使用cmake-gui的时候起作用，在界面上会出现一个下拉框供给用户选择来设置CMAKE_BUILD_TYPE变量。<docstring>里的一行文字作为提示。

但是这个下拉框里的内容，需要使用随后的`set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")`这个命令来设置。也就是所谓的设置string缓存条目属性。

界面显示如本节2.2.1

官方文档： https://cmake.org/cmake/help/latest/command/set.html 

参考博客： https://blog.csdn.net/Zhanganliu/article/details/99851352 

​					 https://www.cnblogs.com/ncuneugcj/p/9756324.html 

#####  环境变量

```cmake
set(ENV{<variable>} [<value>])
```

设置一个 [`Environment Variable`](https://cmake.org/cmake/help/latest/manual/cmake-env-variables.7.html#manual:cmake-env-variables(7)) 到给定值。随后的调用`$ENV{<varible>}`将返回此新值。

此命令仅影响当前的CMake进程，不影响调用CMake的进程，也不影响整个系统环境，也不影响后续构建或测试过程的环境。

如果在空字符串之后`ENV{}`或如果没有参数``，则此命令将清除环境变量的任何现有值。

之后``的参数将被忽略。如果发现其他参数，则会发出作者警告。

## 三  构建示例

```bash
$ mkdir build

$ cd build/

$ cmake ..
Setting build type to 'RelWithDebInfo' as none was specified.
-- The C compiler identification is GNU 4.8.4
-- The CXX compiler identification is GNU 4.8.4
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build

$ make VERBOSE=1
/usr/bin/cmake -H/home/matrim/workspace/cmake-examples/01-basic/F-build-type -B/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build --check-build-system CMakeFiles/Makefile.cmake 0
/usr/bin/cmake -E cmake_progress_start /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/progress.marks
make -f CMakeFiles/Makefile2 all
make[1]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/depend
make[2]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
cd /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/matrim/workspace/cmake-examples/01-basic/F-build-type /home/matrim/workspace/cmake-examples/01-basic/F-build-type /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake --color=
Dependee "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake" is newer than depender "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Dependee "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/CMakeDirectoryInformation.cmake" is newer than depender "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Scanning dependencies of target cmake_examples_build_type
make[2]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/build
make[2]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
/usr/bin/cmake -E cmake_progress_report /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles 1
[100%] Building CXX object CMakeFiles/cmake_examples_build_type.dir/main.cpp.o
/usr/bin/c++    -O2 -g -DNDEBUG   -o CMakeFiles/cmake_examples_build_type.dir/main.cpp.o -c /home/matrim/workspace/cmake-examples/01-basic/F-build-type/main.cpp
Linking CXX executable cmake_examples_build_type
/usr/bin/cmake -E cmake_link_script CMakeFiles/cmake_examples_build_type.dir/link.txt --verbose=1
/usr/bin/c++   -O2 -g -DNDEBUG    CMakeFiles/cmake_examples_build_type.dir/main.cpp.o  -o cmake_examples_build_type -rdynamic
make[2]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
/usr/bin/cmake -E cmake_progress_report /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles  1
[100%] Built target cmake_examples_build_type
make[1]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
/usr/bin/cmake -E cmake_progress_start /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles 0$ mkdir build
$ cd build/
/build$ cmake ..
Setting build type to 'RelWithDebInfo' as none was specified.
-- The C compiler identification is GNU 4.8.4
-- The CXX compiler identification is GNU 4.8.4
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build
/build$ make VERBOSE=1
/usr/bin/cmake -H/home/matrim/workspace/cmake-examples/01-basic/F-build-type -B/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build --check-build-system CMakeFiles/Makefile.cmake 0
/usr/bin/cmake -E cmake_progress_start /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/progress.marks
make -f CMakeFiles/Makefile2 all
make[1]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/depend
make[2]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
cd /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/matrim/workspace/cmake-examples/01-basic/F-build-type /home/matrim/workspace/cmake-examples/01-basic/F-build-type /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake --color=
Dependee "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/DependInfo.cmake" is newer than depender "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Dependee "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/CMakeDirectoryInformation.cmake" is newer than depender "/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles/cmake_examples_build_type.dir/depend.internal".
Scanning dependencies of target cmake_examples_build_type
make[2]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
make -f CMakeFiles/cmake_examples_build_type.dir/build.make CMakeFiles/cmake_examples_build_type.dir/build
make[2]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
/usr/bin/cmake -E cmake_progress_report /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles 1
[100%] Building CXX object CMakeFiles/cmake_examples_build_type.dir/main.cpp.o
/usr/bin/c++    -O2 -g -DNDEBUG   -o CMakeFiles/cmake_examples_build_type.dir/main.cpp.o -c /home/matrim/workspace/cmake-examples/01-basic/F-build-type/main.cpp
Linking CXX executable cmake_examples_build_type
/usr/bin/cmake -E cmake_link_script CMakeFiles/cmake_examples_build_type.dir/link.txt --verbose=1
/usr/bin/c++   -O2 -g -DNDEBUG    CMakeFiles/cmake_examples_build_type.dir/main.cpp.o  -o cmake_examples_build_type -rdynamic
make[2]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
/usr/bin/cmake -E cmake_progress_report /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles  1
[100%] Built target cmake_examples_build_type
make[1]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/F-build-type/build'
/usr/bin/cmake -E cmake_progress_start /home/matrim/workspace/cmake-examples/01-basic/F-build-type/build/CMakeFiles 0
```



## 微信公众号

欢迎大家关注我的个人公众号，现阶段主要总结Robomaster相关的计算机视觉知识：Qt，C++，CMake,OpenCV等等
公众号名称：三丰杂货铺



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200529103009878.gif#pic_center)

