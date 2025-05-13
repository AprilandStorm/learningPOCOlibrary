# 安装
本人采用VS结合虚拟机的方式在Linux上进行学习与开发

## 第1步 安装Conan `pip install conan`
- 需要在虚拟机里事先下载python3
- 如果ubuntu系统较老，`pip3 install conan`并不会将Conan安装上，须输入`sudo pip3 install conan`才会安装成功
- `conan --version` 检查安装是否成功。成功则返回版本号
- 注意：使用 Conan 安装 POCO 默认只能获得已编译好的库和头文件，不能直接看到 POCO 的完整源码；想查看源码须克隆GitHub代码

## 第2步 构建项目结构
my_poco_project/\
├── conanfile.txt\
├── CMakeLists.txt\
├── build\
├── CMakeUserPresets.json\
└── src/\
    └── main.cpp\

## 第3步 创建conanfile.txt

- `nano conanfile.txt` 然后在打开的编辑器里输入:
```
[requires]
poco/1.13.1

[generators]
CMakeDeps
CMakeToolchain
```
编辑完后：按 Ctrl + O 保存（然后按回车）;再按 Ctrl + X 退出
- [requires]这一部分声明了 项目所依赖的第三方库
- [generators]这一部分定义了 如何将 Conan 管理的依赖库信息传递给你的构建系统；cmake 表示：Conan 会自动生成一个 conanbuildinfo.cmake 文件，你只需在 CMakeLists.txt 中引用它，就能使用 POCO 的头文件和库路径

## 第4步 创建CMake配置文件CMakeLists.txt
```
cmake_minimum_required(VERSION 3.15)
project(MyPocoApp)

# 引入 Conan 工具链
include(${CMAKE_BINARY_DIR}/conan_toolchain.cmake)

# 添加源文件
add_executable(MyPocoApp src/main.cpp)

# 添加 POCO 依赖
find_package(Poco REQUIRED COMPONENTS Foundation)
target_link_libraries(MyPocoApp PRIVATE Poco::Foundation)
```
## 第5步 使用 Conan 安装依赖
```
conan profile detect  # 初始化 Conan 配置（首次执行）
conan install . --output-folder=build --build=missing
```
这一步会：下载 POCO 和它的依赖;在 build/ 文件夹下生成 CMake 构建所需文件\
注意：这里要想安装成功，需要将 Conan 编译器配置更新为 C++17\
- 手动打开配置文件修改
用文本编辑器打开配置文件：`nano ~/.conan2/profiles/default`\
找到 [settings] 部分，修改或添加如下内容：
```
[settings]
os=Linux
arch=x86_64
compiler=gcc
compiler.version=11
compiler.libcxx=libstdc++11
cppstd=17
```
保存退出（Ctrl+O 保存，Enter，再按 Ctrl+X 退出）\
-----长久地等待------\
请问为什么连接远程库经常出问题🤬\
好了，安装成功了😊

## 第6步 安装第三方库
- 安装OpenSSL `sudo apt-get install openssl libssl-dev`
- 安装iODBC/unixODBC `sudo apt-get install libiodbc2 libiodbc2-dev`\
然而，Makefile 可能无法找到头文件和库文件。在这种情况下，请相应地编辑 Data/ODBC 和/或 Data/MySQL 中的 Makefile 文件。
- 安装MySQL Client
```
sudo apt-get update
sudo apt-get install libmysqlclient-dev
```

# 配置VS2022与ubuntu进行远程连接
- VS2022下新建CMake文件
- [远程连接](https://blog.csdn.net/qq_45009309/article/details/130149429)

# 使用CMake构建
- CMake 可用于在任何平台上使用任何编译器构建 POCO C++ 库
- POCO C++ 库需要 CMake 3.15 或更高版本
- CMake 支持源代码构建，这是使用 CMake 构建 POCO C++ 库的推荐方法
- 假设你已经下载了 POCO C++ Libraries 的源码，并将其放在了 /path/to/poco 目录下，如果你想编译（build）这个库，只需要在终端中依次输入以下两条命令：
```
$ cmake -H/path/to/poco -B/path/to/poco-build
$ cmake --build /path/to/poco-build
```
这两条命令的含义如下：\
`cmake -H/path/to/poco -B/path/to/poco-build`
这是 CMake 的一种命令写法：
-H 表示源码目录（即包含 CMakeLists.txt 的目录）是 /path/to/poco;
-B 表示构建目录（即所有中间文件、编译结果等生成的地方）是 /path/to/poco-build;
简单说，这是“从 /path/to/poco 编译源码，到 /path/to/poco-build 目录中去”\
`cmake --build /path/to/poco-build`
这条命令会在前一步准备好的构建目录中，实际开始执行构建操作（调用 make、ninja 或其他生成的构建系统）\
本人用的是 Conan 来管理和获取 POCO，这意味着不需要手动去 /path/to/poco 下载和构建源码，Conan 会自动搞定这些事情\
不需要去运行 cmake -H/path/to/poco 这类命令了,只需要：
```
cd build
cmake ..
cmake --build .
```
或者使用 preset：
```
cmake --preset <your-preset-name>
cmake --build --preset <your-preset-name>
```

# 后面看不太懂
- 如果第三方库未安装在默认位置，cmake 将无法运行
- 要在 cmake 项目中使用 POCO C++ 库，要在项目中添加以下行，例如使用加密：
```
find_package(Poco REQUIRED Crypto)
....
target_link_libraries(yourTargetName ... Poco::Crypto)
```
- 如果收到“由于未提供“FindPoco.cmake”而导致的错误”，则应将 CMAKE_PREFIX_PATH 设置为 POCO C++ 库的安装目录
```
 cmake -H/path/to/yourProject -B/path/to/yourProject-build -DCMAKE_PREFIX_PATH=/path/to/installationOf/poco
```
##  在Unix/Linux/macOS上构建
- 要提取源代码并构建所有库、测试套件和示例，只需
```
$ gunzip poco-X.Y.tar.gz
$ tar -xf poco-X.Y.tar
$ cd poco-X.Y
$ ./configure
$ gmake -s
```
如需帮助，请调用
```
$ ./configure --help`
```
- 可以阅读 configure 脚本源代码以获取可用的选项列表。对于初学者，建议使用 `--no-tests` 和 `--no-samples`，以缩短构建时间。在多核或多处理器机器上，使用并行 make 可以加快构建速度`（make -j4）`
- 成功构建 POCO 后，您可以将其安装到 /usr/local（或作为 configure `--prefix=<path> `参数指定的其他目录）：
```
$ sudo gmake -s install
```
- 重要提示：请确保构建目录的路径不包含符号链接,否则，将收到错误消息“当前工作目录不在 $PROJECT_BASE 下”

# 下载源码并进行配置
- 使用 CMake 来构建 POCO（需要 CMake 3.5 或更高版本），在 POCO 源代码树中，运行以下命令：
```
$ mkdir cmake-build//创建一个名为cmake-build的新目录 
$ cd cmake-build
$ cmake .. && cmake --build .//运行CMake,读取上一级目录中的CMakeLists.txt文件生成构建配置；编译项目
```
- 还可以运行 `$ sudo cmake --build . --target install` 在 cmake-build 目录中将 POCO 安装到 /usr/local，可以通过在配置步骤中将 -DCMAKE_INSTALL_PREFIX=/path/to/directory 选项传递给 CMake 来更改安装目录
- 可以通过在配置步骤中将 -DCMAKE_INSTALL_PREFIX=/path/to/directory 选项传递给 CMake 来更改安装目录
- 
