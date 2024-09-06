# cmake使用

## 注释

使用#进行注释行，使用#[[]]注释块

## CMakeLists.txt文件基础内容介绍

```c++
cmake_minimum_required(VERSION 3.0)
project(CALC)
add_executable(app add.c div.c main.c mult.c sub.c)
```



- cmake_minimum_required：**指定使用的 cmake 的最低版本**
- project：**定义工程名称**
- add_executable：**定义工程会生成一个可执行程序**

`add_executable(可执行程序名 源文件名称),`源文件名可以是一个也可以是多个，如有多个可用空格或;间隔

## set

用set定义一个变量，将文件名对应的字符串存储起来

```c++
# 方式1: 各个源文件之间使用空格间隔
# set(SRC_LIST add.c  div.c   main.c  mult.c  sub.c)

# 方式2: 各个源文件之间使用分号 ; 间隔
set(SRC_LIST add.c;div.c;main.c;mult.c;sub.c)
add_executable(app  ${SRC_LIST})

```

## 指定c++标准

```c++
#增加-std=c++11
set(CMAKE_CXX_STANDARD 11)
#增加-std=c++14
set(CMAKE_CXX_STANDARD 14)
#增加-std=c++17
set(CMAKE_CXX_STANDARD 17)
```

## 指定输出路径

在CMake中指定可执行程序输出的路径，也对应一个宏，叫做EXECUTABLE_OUTPUT_PATH，它的值还是通过set命令进行设置:

```c++
set(HOME /home/robin/Linux/Sort) //定义一个变量用于存储一个绝对路径
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin) //将拼接好的路径值设置给EXECUTABLE_OUTPUT_PATH宏
```

## 搜索文件

在 CMake 中使用aux_source_directory 命令可以查找某个路径下的所有源文件，命令格式为：

`aux_source_directory(< dir > < variable >)`

- dir：要搜索的目录
- variable：将从dir目录下搜索到的源文件列表存储到该变量中

```c++
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
// 搜索 src 目录下的源文件
aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)
add_executable(app  ${SRC_LIST})
```

其次还可以用file递归搜索：

`file(GLOB/GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)`

- GLOB: 将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中。

- GLOB_RECURSE：递归搜索指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量中。

  ```c++
  file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
  file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
  ```

  

`file(GLOB MAIN_HEAD "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h")`

## 包含头文件

在编译项目源文件的时候，很多时候都需要将源文件对应的头文件路径指定出来，这样才能保证在编译过程中编译器能够找到这些头文件，并顺利通过编译。在CMake中设置要包含的目录也很简单，通过一个命令就可以搞定了，他就是include_directories:

`include_directories(headpath)`

```
$ tree
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
└── src
    ├── add.cpp
    ├── div.cpp
    ├── main.cpp
    ├── mult.cpp
    └── sub.cpp

3 directories, 7 files


cmake_minimum_required(VERSION 3.0)
project(CALC)
set(CMAKE_CXX_STANDARD 11)
set(HOME /home/robin/Linux/calc)
set(EXECUTABLE_OUTPUT_PATH ${HOME}/bin/)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
add_executable(app  ${SRC_LIST})
```

## 制作动态库以及静态库

### 制作静态库

`add_library(库名称 STATIC 源文件1 [源文件2] ...)` 

在Linux中，静态库名字分为三部分：lib+库名字+.a，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。

在Windows中虽然库名和Linux格式不同，但也只需指定出名字即可。

```
.
├── build
├── CMakeLists.txt
├── include           # 头文件目录
│   └── head.h
├── main.cpp          # 用于测试的源文件
└── src               # 源文件目录
    ├── add.cpp
    ├── div.cpp
    ├── mult.cpp
    └── sub.cpp


cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc STATIC ${SRC_LIST})
```


这样最终就会生成对应的静态库文件libcalc.a。

### 制作动态库

`add_library(库名称 SHARED 源文件1 [源文件2] ...) `

在Linux中，动态库名字分为三部分：lib+库名字+.so，此处只需要指定出库的名字就可以了，另外两部分在生成该文件的时候会自动填充。

在Windows中虽然库名和Linux格式不同，但也只需指定出名字即可。

```
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
add_library(calc SHARED ${SRC_LIST})
```


这样最终就会生成对应的动态库文件libcalc.so。

### 指定输出路径

对于生成的库文件来说和可执行程序一样都可以指定输出路径。由于**在Linux下生成的动态库默认是有执行权限的**，所以可以按照生成可执行程序的方式去指定它生成的目录：

```c++
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
# 设置动态库生成路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib) //只适用于动态库
add_library(calc SHARED ${SRC_LIST})
```

对于这种方式来说，其实就是通过set命令给EXECUTABLE_OUTPUT_PATH宏设置了一个路径，这个路径就是可执行文件生成的路径。

由于**在Linux下生成的静态库默认不具有可执行权限**，所以在指定静态库生成的路径的时候就不能使用EXECUTABLE_OUTPUT_PATH宏了，而应该使用LIBRARY_OUTPUT_PATH，这个宏对应静态库文件和动态库文件都适用。

```
cmake_minimum_required(VERSION 3.0)
project(CALC)
include_directories(${PROJECT_SOURCE_DIR}/include)
file(GLOB SRC_LIST "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
# 设置动态库/静态库生成路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)
# 生成动态库
#add_library(calc SHARED ${SRC_LIST})
# 生成静态库
add_library(calc STATIC ${SRC_LIST})
```

## 简单介绍库文件

### 1.静态库

在Linux中静态库以lib作为前缀, 以.a作为后缀, 中间是库的名字自己指定即可, 即: libxxx.a
在Windows中静态库一般以lib作为前缀, 以lib作为后缀, 中间是库的名字需要自己指定, 即: libxxx.lib

在Linux中生成静态库，需要先对源文件进行汇编操作 【使用参数 -c。gcc 源文件(.c) -c】 得到二进制格式的目标文件 (.o 格式), 然后在通过 ar工具将目标文件打包【ar rcs 静态库名字（libxxx.a）原材料（.o）】就可以得到静态库文件了 (libxxx.a)。

![img](https://subingwen.cn/linux/library/static-lib.png)



发布静态库

	1. 提供头文件 **.h
	2. 提供制作出来的静态库 libxxx.a

[静态库举例点击](https://subingwen.cn/linux/library/#1-1-%E7%94%9F%E6%88%90%E9%9D%99%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93)

### 2.动态库

动态链接库是程序运行时加载的库，当动态链接库正确部署之后，运行的多个程序可以使用同一个加载到内存中的动态库，因此在Linux中动态链接库也可称之为共享库。

动态链接库是目标文件的集合，目标文件在动态链接库中的组织方式是按照特殊方式形成的。库中函数和变量的地址使用的是相对地址（静态库中使用的是绝对地址），其真实地址是在应用程序加载动态库时形成的。

在Linux中动态库以lib作为前缀, 以.so作为后缀, 中间是库的名字自己指定即可, 即: libxxx.so
在Windows中动态库一般以lib作为前缀, 以dll作为后缀, 中间是库的名字需要自己指定, 即: libxxx.dll

生成动态链接库是直接使用gcc命令并且需要添加-fPIC（-fpic） 以及-shared 参数。

- -fPIC 或 -fpic 参数的作用是使得 gcc 生成的代码是与位置无关的，也就是使用相对位置。

- -shared参数的作用是告诉编译器生成一个动态链接库。

  ![img](https://subingwen.cn/linux/library/dll-0.png)

  1..将源文件进行汇编操作, 需要使用参数 -c, 还需要添加额外参数 -fpic / -fPIC

  `gcc 源文件(*.c) -c -fpic`

  2.将得到的.o文件打包成动态库, 还是使用gcc, 使用参数 -shared 指定生成动态库(位置没有要求)

  `gcc -shared 与位置无关的目标文件(*.o) -o 动态库(libxxx.so)`

  3.发布动态库和头文件

   	1. 提供头文件: xxx.h
   	2. 提供动态库: libxxx.so

[动态库举例点击](https://subingwen.cn/linux/library/#1-1-%E7%94%9F%E6%88%90%E9%9D%99%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93)

### 两种库工作原理

**静态库如何被加载**

在程序编译的最后一个阶段也就是链接阶段，提供的静态库会被打包到可执行程序中。当可执行程序被执行，静态库中的代码也会一并被加载到内存中，因此不会出现静态库找不到无法被加载的问题。

**动态库如何被加载**

*在程序编译的最后一个阶段也就是链接阶段：*

在gcc命令中虽然指定了库路径(使用参数 -L ), 但是这个路径并没有记录到可执行程序中，只是检查了这个路径下的库文件是否存在。
同样对应的动态库文件也没有被打包到可执行程序中，只是在可执行程序中记录了库的名字。

*可执行程序被执行起来之后:*

程序执行的时候会先检测需要的动态库是否可以被加载，加载不到就会提示上边的错误信息
当动态库中的函数在程序中被调用了, 这个时候动态库才加载到内存，如果不被调用就不加载
动态库的检测和内存加载操作都是由动态连接器来完成的

**动态连接器**

动态链接器是一个独立于应用程序的进程, 属于操作系统, 当用户的程序需要加载动态库的时候动态连接器就开始工作了，很显然动态连接器根本就不知道用户通过 gcc 编译程序的时候通过参数 -L指定的路径。

那么动态链接器是如何搜索某一个动态库的呢，在它内部有一个默认的搜索顺序，按照优先级从高到低的顺序分别是：

1.可执行文件内部的 DT_RPATH 段

2.系统的环境变量 LD_LIBRARY_PATH

3.系统动态库的缓存文件 /etc/ld.so.cache

4.存储动态库/静态库的系统目录 /lib/, /usr/lib等

按照以上四个顺序, 依次搜索, 找到之后结束遍历, 最终还是没找到, 动态连接器就会提示动态库找不到的错误信息

[无法加载问题解决方案](https://subingwen.cn/linux/library/#1-1-%E7%94%9F%E6%88%90%E9%9D%99%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93)

### 优缺点对比

**静态库**

优点：

静态库被打包到应用程序中加载速度快
发布程序无需提供静态库，移植方便
缺点：

相同的库文件数据可能在内存中被加载多份, 消耗系统资源，浪费内存
库文件更新需要重新编译项目文件, 生成新的可执行程序, 浪费时间。

![img](https://subingwen.cn/linux/library/dll-1.png)

**动态库**

优点：

可实现不同进程间的资源共享
动态库升级简单, 只需要替换库文件, 无需重新编译应用程序
程序猿可以控制何时加载动态库, 不调用库函数动态库不会被加载
缺点：

加载速度比静态库慢, 以现在计算机的性能可以忽略
发布程序需要提供依赖的动态库

![img](https://subingwen.cn/linux/library/dll-2.png)

## CMake链接库

### 链接静态库

在编写程序的过程中，可能会用到一些系统提供的动态库或者自己制作出的动态库或者静态库文件，cmake中也为我们提供了相关的加载动态库的命令。

```
$ tree 
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h
├── lib
│   └── libcalc.a     # 制作出的静态库的名字
└── src
    └── main.cpp

4 directories, 4 files
```

在cmake中，链接静态库的命令如下：

`link_libraries(<static lib> [<static lib>...])`

- **参数1：指定出要链接的静态库的名字**
- 可以是全名 libxxx.a
- 也可以是掐头（lib）去尾（.a）之后的名字 xxx

- **参数2-N：要链接的其它静态库的名字**

**指定链接库路径**：如果该静态库不是系统提供的（自己制作或者使用第三方提供的静态库）可能出现静态库找不到的情况，此时可以将静态库的路径也指定出来：

`link_directories(<lib path>)`

```
cmake_minimum_required(VERSION 3.0)
project(CALC)
# 搜索指定目录下源文件
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
# 包含头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 包含静态库路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 链接静态库
link_libraries(calc)
add_executable(app ${SRC_LIST})
```

### 链接动态库

在程序编写过程中，除了在项目中引入静态库，好多时候也会使用一些标准的或者第三方提供的一些动态库

在cmake中链接动态库的命令如下:

```
target_link_libraries(
    <target> 
    <PRIVATE|PUBLIC|INTERFACE> <item>... 
    [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```

- target：指定要加载动态库的文件的名字

  该文件可能是一个源文件
  该文件可能是一个动态库文件
  该文件可能是一个可执行文件

- PRIVATE|PUBLIC|INTERFACE：动态库的访问权限，默认为PUBLIC


​	如果各个动态库之间没有依赖关系，无需做任何设置，三者没有没有区别，一般无需指定，使用默认的 PUBLIC 即可。

​	动态库的链接具有传递性，如果动态库 A 链接了动态库B、C，动态库D链接了动态库A，此时动态库D相当于也链接了动态库B、C，并可以使用动态库B、C中定义的方法。

`target_link_libraries(A B C)`

`target_link_libraries(D A)`

**PUBLIC：**在public后面的库会被Link到前面的target中，并且里面的符号也会被导出，提供给第三方使用。

**PRIVATE：**在private后面的库仅被link到前面的target中，并且终结掉，第三方不能感知你调了啥库

**INTERFACE：**在interface后面引入的库不会被链接到前面的target中，只会导出符号。

#### 链接系统动态库

- 动态库的链接和静态库是完全不同的：

  静态库会在生成可执行程序的链接阶段被打包到可执行程序中，所以可执行程序启动，静态库就被加载到内存中了。
  动态库在生成可执行程序的链接阶段不会被打包到可执行程序中，当可执行程序被启动并且调用了动态库中的函数的时候，动态库才会被加载到内存
  因此，在cmake中指定要链接的动态库的时候，应该将命令写到生成了可执行文件之后：

  ```
  cmake_minimum_required(VERSION 3.0)
  project(TEST)
  file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
  # 添加并指定最终生成的可执行程序名
  add_executable(app ${SRC_LIST})
  # 指定可执行程序要链接的动态库名字
  target_link_libraries(app pthread)
  # app: 对应的是最终生成的可执行程序的名字
  # pthread：这是可执行程序要加载的动态库，这个库是系统提供的线程库，全名为libpthread.so，在指定的时候一般会掐头（lib）去尾（.so）。
  ```


  

#### 链接第三方动态库

```
$ tree 
.
├── build
├── CMakeLists.txt
├── include
│   └── head.h            # 动态库对应的头文件
├── lib
│   └── libcalc.so        # 自己制作的动态库文件
└── main.cpp              # 测试用的源文件

3 directories, 4 files
```

假设在测试文件main.cpp中既使用了自己制作的动态库libcalc.so又使用了系统提供的线程库，此时CMakeLists.txt文件可以这样写：

```
cmake_minimum_required(VERSION 3.0)
project(TEST)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
# 指定源文件或者动态库对应的头文件路径
include_directories(${PROJECT_SOURCE_DIR}/include)
# 指定要链接的动态库的路径
link_directories(${PROJECT_SOURCE_DIR}/lib)
# 添加并生成一个可执行程序
add_executable(app ${SRC_LIST})
# 指定要链接的动态库
target_link_libraries(app pthread calc)
```

## 变量操作

有时候项目中的源文件并不一定都在同一个目录中，但是这些源文件最终却需要一起进行编译来生成最终的可执行文件或者库文件。如果我们通过file命令对各个目录下的源文件进行搜索，最后还需要做一个字符串拼接的操作，关于字符串拼接可以使用set命令也可以使用list命令。

`set(变量名1 ${变量名1} ${变量名2} ...)`

关于上面的命令其实就是将从第二个参数开始往后所有的字符串进行拼接，最后将结果存储到第一个参数中，如果第一个参数中原来有数据会对原数据就行覆盖。

```
cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
# 追加(拼接)
set(SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
message(STATUS "message: ${SRC_1}")
```



`list(APPEND <list> [<element> ...])`

list命令的功能比set要强大，字符串拼接只是它的其中一个功能，所以需要在它第一个参数的位置指定出我们要做的操作，APPEND表示进行数据追加，后边的参数和set就一样了。

```
cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/src1/*.cpp)
file(GLOB SRC_2 ${PROJECT_SOURCE_DIR}/src2/*.cpp)
# 追加(拼接)
list(APPEND SRC_1 ${SRC_1} ${SRC_2} ${TEMP})
message(STATUS "message: ${SRC_1}")
```

在CMake中，使用set命令可以创建一个list。一个在list内部是一个由分号;分割的一组字符串。例如，set(var a b c d e)命令将会创建一个list:a;b;c;d;e，但是最终打印变量值的时候得到的是abcde。



list剔除：

在当前这么目录有五个源文件，其中main.cpp是一个测试文件。如果我们想要把计算器相关的源文件生成一个动态库给别人使用，那么只需要add.cpp、div.cp、mult.cpp、sub.cpp这四个源文件就可以了。此时，就需要将main.cpp从搜索到的数据中剔除出去，想要实现这个功能，也可以使用list

`list(REMOVE_ITEM <list> <value> [<value> ...])`

```
cmake_minimum_required(VERSION 3.0)
project(TEST)
set(TEMP "hello,world")
file(GLOB SRC_1 ${PROJECT_SOURCE_DIR}/*.cpp)
# 移除前日志
message(STATUS "message: ${SRC_1}")
# 移除 main.cpp
list(REMOVE_ITEM SRC_1 ${PROJECT_SOURCE_DIR}/main.cpp)
# 移除后日志
message(STATUS "message: ${SRC_1}")
```





[整体架构](https://subingwen.cn/cmake/CMake-advanced/)