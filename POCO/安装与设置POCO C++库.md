# 安装
本人采用VS结合虚拟机的方式在Linux上进行学习与开发

## 第1步 安装Conan `pip install conan`
- 需要在虚拟机里事先下载python3
- 如果ubuntu系统较老，`pip3 install conan`并不会将Conan安装上，须输入`sudo pip3 install conan`才会安装成功
- `conan --version` 检查安装是否成功。成功则返回版本号

## 第2步 构建项目结构
my_poco_project/\
├── conanfile.txt\
├── CMakeLists.txt\
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
-----长久地等待------
