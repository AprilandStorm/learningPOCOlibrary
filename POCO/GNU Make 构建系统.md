- 为什么选择 GNU Make？因为 GNU Make 几乎随处可用
- philisophy for POCO is download — compile — go
- 对于每个库和可执行文件，都有一个小型 make 文件来描述项目的类型、输入文件和输出
- POCO 构建系统支持以下功能：\
构建库、应用程序和插件；\
调试和发布构建；\
动态和静态链接；\
32 位和 64 位构建；\
自动依赖性检测；\
易于使用；\
支持嵌入式 Linux 或 iOS 等嵌入式平台的交叉编译
- POCO 构建系统自动确定源文件和头文件之间的依赖关系
- 构建系统的核心由三种不同类型的文件组成——平台配置文件、make 规则和 shell 脚本
- 所有内容都位于 POCO C++ 库根目录内的build目录中
- 配置文件位于 build/config 目录中
- Rule文件包含独立于平台的 make 规则，用于使用配置文件中定义的工具从源文件编译和链接可执行文件或（静态和共享）库;规则文件位于 build/rules 目录中
- Script文件由构建系统的 make 文件调用，以执行 make 文件中无法表达的事情;脚本文件位于 build/script 目录中
- 构建系统要求项目的文件（头文件、源文件、Makefile）处于特定的目录布局中
- 首先，所有项目目录必须位于构建目录所在的同一目录中或之下，或者在其下面的某个目录中\
所有头文件必须位于 include 目录（或其子目录）中。所有源文件必须位于 src 目录中。最后，项目目录中必须有一个 Makefile（名为 Makefile）  
- 下面是 POCO 源目录层次结构的示例
```
poco/
  build/
  Foundation/
  Net/
  Util/
  XML/
  ...

  MyLib/
    include/
      MyLib.h
      MyClass.h

    src/
      MyClass.cpp

    Makefile
```
# 构建产品
- POCO 构建系统将最终构建产品（可执行文件和库）和中间产品（目标文件、依赖文件）放入某些目录中\
除非另有说明（通过相应地设置环境变量 POCO_BUILD - 见下文），这些目录位于源目录树中
- 所有库都放入 POCO base目录内的路径为 lib/<OSNAME>/<OSARCH> 的目录中
- 可执行文件被放入prohect目录下的 bin/<OSNAME>/<OSARCH> 目录中
- 目标文件被放置在project目录下的 obj/<OSNAME>/<OSARCH> 目录中
- 依赖文件则被放置在project目录下的 .dep/<OSNAME>/<OSARCH> 目录中
- <OSNAME> 是主机操作系统的名称，通过调用 uname （或等效函数）获得，<OSARCH> 是主机硬件架构的名称，通过调用 uname -m （或等效函数）获得
- 考虑到构建产品，典型的项目层次结构如下所示:
```
poco/
  build/
  lib/
    Linux/
      i686/
        libCppUnit.so.1
        libCppUnitd.so.1
        libCppUnit.so
        libCppUnitd.so
        libPocoFouncation.so.1
        ...

  Foundation/
    ...
    testsuite/
      ...
      bin/
        Linux/
          i686/
            testrunner
            testrunnerd
            ...

  Net/
  Util/
  XML/
  ...

  MyLib/
    .dep/
      Linux/
        i686/
          MyClass.d

    obj/
      Linux/
        i686/
          MyClass.o

    include/
      MyLib.h
      MyClass.h

    src/
      MyClass.cpp

    Makefile
```
# 构建系统参考
- 后面不明白是在干啥，先不读了
