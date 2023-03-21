### 编译那些事儿
#### 前言

---

接触开源 *uix项目，需要本地源码编译安装的，都不难遇到过 `./configure ` `make ` `make install` 一键三连。
有的大型软件如google chrome等，还需要额外安装不同编译环境 GN才能进行构建。
本文从最简单的纯使用编译器进行编译，以及make、GN的使用场景，来聊聊编译那些事儿。
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657730592207-65affd69-7032-45a4-81f9-a80990872448.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=416&id=u7ec26aa8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=832&originWidth=1652&originalType=binary&ratio=1&rotation=0&showTitle=false&size=576791&status=done&style=none&taskId=u6a5c17ea-b07b-4ef2-a0a0-76694078e81&title=&width=826)


#### 1.刀耕火种-调用编译器

---

话不多说，先来一个路人皆知的helloworld，太简单也不太好。稍微复杂一丢丢，函数声明和实现分离（头文件+实现文件），main文件再调用这个函数

hello.h

```c
#if !defined(_HELLO_H_)
#define _HELLO_H_

void SayHello(void);

#endif /* if !define(_HELLO_H_) */
```

hello.c

```c
#include "hello.h"
#include <stdio.h>

void SayHello(void){
  printf("Hello makefiles!\n");
}
```

main.c

```c
#include "hello.h"
int main() {
  SayHello();
  return(0);
}
```

直接使用编译器进行编译
**GCC:**
`gcc -o makehello hello.c main.c -I.`
**Clang:**  /ˈklæŋ/
`clang -o makehello hello.c main.c -I.`

常用编译参数

- **-I** 头文件查找目录
- **-W** 编译器诊断参数]

	用于语言级别的约束。 
	比如 `-Winvalid-noreturn` 遇到函数无return则会报错

- **-isysroot**  系统库地址

	比如非常基础的基础库地址比如` #include<stdio.h>`交叉编译中经常用到

- **-F**  framework的搜索目录

	当前提示找不到framework时可以紧盯这个参数

更多参数
>
> GCC编译命令 [https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html)
>
> 
>
> Clang编译命令：[https://clang.llvm.org/docs/DiagnosticsReference.html#winvalid-noreturn](https://clang.llvm.org/docs/DiagnosticsReference.html#winvalid-noreturn)

#### 2.解放双手-make
源文件增长时，直接使用编译器进行编译变得不太现实。
❌ 每个源文件都需要编译（重复工作）
❌依赖关系不好维护  （谁先谁后）

**能否找个文件记录编译步骤/指令，并且帮我调用编译器进行编译？？**

记录编译步骤/指令的文件 = makefile
调用编译的程序       = make

打个不太好的比方，编译的过程是一个烹饪的过程。

食材     = 源文件
一盘菜 = 可执行文件（动态库、静态库）
厨具     = 编译器

简单的菜式，那直接上手直接出锅，比如简单的做个沙拉🥗
![BurritoRecipe2.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657731970924-d5e60b38-5dba-40e7-bf21-58bd48c176f4.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=276&id=u7f108412&margin=%5Bobject%20Object%5D&name=BurritoRecipe2.png&originHeight=728&originWidth=1200&originalType=binary&ratio=1&rotation=0&showTitle=true&size=308268&status=done&style=none&taskId=u6a733879-89e9-4ff1-acb0-4273020fb00&title=%E7%AE%80%E5%8D%95%E7%9A%84%E5%81%9A%E4%B8%AA%E6%B2%99%E6%8B%89&width=455 "简单的做个沙拉")

实际情况是你需要炒**满汉全席，**菜式复杂且繁复。
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657732897720-6f499740-f1db-4760-ae72-73c438e9c487.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=154&id=u32aa0354&margin=%5Bobject%20Object%5D&name=image.png&originHeight=100&originWidth=100&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10975&status=done&style=none&taskId=ubb5845ed-64cd-4591-a28c-dc48977acc4&title=&width=154)![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657732929241-3b342e28-a812-4798-a0c0-c4e294635e4c.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=152&id=u150a039c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=100&originWidth=100&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10640&status=done&style=none&taskId=ubace2921-13de-442b-8cf9-b5f50c0e6b2&title=&width=152)![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657732970254-40af6a87-c7b7-424a-894b-81e00d804544.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=154&id=u9d5f9d19&margin=%5Bobject%20Object%5D&name=image.png&originHeight=100&originWidth=100&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9273&status=done&style=none&taskId=u06fe9766-7adf-44f2-a380-d55a5c778f8&title=&width=154)
你需要炒菜的步骤（**编译步骤以及依赖关系a模块依赖b模块）,**且使用不同的厨具（编译器）有不同的说明书。
此时你需要一个记录炒菜步骤的**菜谱（recipe）makefile**

![BurritoRecipe.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657733246903-988ca2c7-a663-4eab-9b8c-21e4d6490980.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=364&id=udba70607&margin=%5Bobject%20Object%5D&name=BurritoRecipe.png&originHeight=728&originWidth=1200&originalType=binary&ratio=1&rotation=0&showTitle=true&size=632185&status=done&style=none&taskId=ua8432e53-e148-4abe-8cf7-caaffa77e6f&title=%E8%8F%9C%E8%B0%B1%EF%BC%88recipe%EF%BC%89&width=600 "菜谱（recipe）")

| 编译角色 | 类比 | 备注 |
| --- | --- | --- |
| 源文件 | ![Tomato_Icon.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657732815077-c8900f94-58b6-45d4-9c5c-f3e54c800843.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=63&id=ud5f75ce6&margin=%5Bobject%20Object%5D&name=Tomato_Icon.png&originHeight=125&originWidth=125&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29977&status=done&style=none&taskId=u741554c4-0387-4d72-b34e-725df27a7ae&title=&width=62.5)食材|  |
| 编译器 | ![guo.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657733195234-b5b29879-1814-48fc-bdab-aa8d1fcc3267.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=85&id=uaeebeafc&margin=%5Bobject%20Object%5D&name=guo.png&originHeight=169&originWidth=170&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41328&status=done&style=none&taskId=u1bc1aa89-c5dc-435b-9665-f3fde4aac74&title=&width=85)厨具 |  |
| make | ![images.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657733322861-37f938c4-df54-46c4-b05c-32ac0bcaf020.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=84&id=u3a213f49&margin=%5Bobject%20Object%5D&name=images.png&originHeight=225&originWidth=225&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4353&status=done&style=none&taskId=u0af7a92b-7074-4a68-af5b-717dbdbf6e3&title=&width=83.5)厨师 |  |
| makefile | ![O2_Recipe_Burger.webp](https://intranetproxy.alipay.com/skylark/lark/0/2022/webp/311452/1657733359987-dfd951d7-26cd-42c2-ab6b-b5a4312d8f30.webp#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=268&id=uda8b1ada&margin=%5Bobject%20Object%5D&name=O2_Recipe_Burger.webp&originHeight=536&originWidth=883&originalType=binary&ratio=1&rotation=0&showTitle=false&size=72204&status=done&style=none&taskId=u74fb8f5a-04c9-4b03-82ee-02fc5d478ec&title=&width=441.5)食谱 | 记录编译参数、步骤、依赖 |
| 编译结果，可执行文件 | ![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657732897720-6f499740-f1db-4760-ae72-73c438e9c487.png#clientId=ua33f9f48-6149-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=126&id=sVKPW&margin=%5Bobject%20Object%5D&name=image.png&originHeight=100&originWidth=100&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10975&status=done&style=none&taskId=ubb5845ed-64cd-4591-a28c-dc48977acc4&title=&width=126)菜 |  |

##### ![工作机制.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657773646311-5cfaaab8-8ea1-477c-9a0d-40747f829445.png#clientId=ud55a19f2-682a-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=uc7895cf5&margin=%5Bobject%20Object%5D&name=%E5%B7%A5%E4%BD%9C%E6%9C%BA%E5%88%B6.png&originHeight=485&originWidth=926&originalType=binary&ratio=1&rotation=0&showTitle=true&size=128158&status=done&style=none&taskId=ua45f051b-900b-480e-b767-1c1b874d01b&title=%E8%A7%92%E8%89%B2 "角色")
厨子(make)看菜谱(makefile)使用原材料(source code)，使用厨具(compiler)炒菜
##### make
[GNU里面在重要构建套件](https://www.gnu.org/software/make/manual/html_node/Reading-Makefiles.html#Reading-Makefiles)，用于
##### makefile
>
> 官方文档：[https://www.gnu.org/software/make/manual/make.html#Introduction](https://www.gnu.org/software/make/manual/make.html#Introduction)

###### makefile  语法
```c
target: prerequisites
<TAB> recipe
```

- target: 指目标产物
- prerequisites：生成该产物所依赖的产物
- recipe: 执行的命令

>
> 官方文档：[https://www.gnu.org/software/make/manual/make.html#toc-An-Introduction-to-Makefiles](https://www.gnu.org/software/make/manual/make.html#toc-An-Introduction-to-Makefiles)


符合前面的例子的makefile，写一个最简单makefile
###### 最简单makefile
```makefile
hellomake: main.c hello.c
	gcc -o make_hello main.c hello.c -I.
```
上面两个makefile都可以执行make命令进行编译
```bash
➜✗ make                
gcc -o make_hello main.c hello.c -I.
```
###### 添加变量以及依赖makefile
```makefile
CC=gcc
CFLAGS= -c -Wall -I.

all: make_hello
make_hello: main.o hello.o
	$(CC) main.o hello.o -o make_hello
main.o: main.c
	$(CC) $(CFLAGS) main.c
hello.o: hello.c
	$(CC) $(CFLAGS) hello.c
clean:
	rm -rf *.o

```

上面两个makefile都可以执行make命令进行编译

```bash
➜✗ make   
gcc -c -Wall -I. main.c
gcc -c -Wall -I. hello.c
gcc main.o hello.o -o make_hello
```

更省人力，添加内置匹配符
makefile
```makefile
.PHONY = all clean

CC=gcc
CFLAGS= -c -Wall -I.
LINKERFLAG = -lm

SRCS := $(wildcard *.c)
BINS := $(SRCS:%.c=%)

all: ${BINS}

# %  为BIN值中的任意值
# $@ 为target
# $< 依赖

#  等同于 gcc -lm foo.o -o foo
%: %.o
	$(CC) $(LINKERFLAG) $< -o $@

# 等同于  gcc -c foo.c
%.o: %.c
	$CC $(CFLAGS) %<

clean:
	rm -rf *.o
	rm -rf ${BINS}

```

不想要每次新增文件都需要改动一次makefile，需要用到内置的通配符

`%@` : 上文语法中的target

`$<` : 上文语法中的依赖

#### 3.更简单的配置-CMake

---

##### 更简单的makefile生成工具
我还是很懒，虽然make+makefile 把繁重的编译命令都给节省了，但是我连makefile都不想写。**cmake就是使用更加直白的语言，帮我们生成makefile的工具。**
![cmake+角色.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657779222980-dac760c5-577e-48e0-8d4d-631955745dcb.png#clientId=ub24cf619-5120-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=u20d40f60&margin=%5Bobject%20Object%5D&name=cmake%2B%E8%A7%92%E8%89%B2.png&originHeight=701&originWidth=926&originalType=binary&ratio=1&rotation=0&showTitle=false&size=160910&status=done&style=none&taskId=uf62c20f9-a251-48c8-96b5-adab3de468b&title=)
##### CMake
cmake是一个程序，需要独立安装，安装地址：https://cmake.org/download/

>
> 官方使用文档：[https://cmake.org/cmake/help/latest/guide/tutorial/index.html](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

##### CMakeLists.txt
类似于makefile，编译配置会写到CMakeLists.txt中，cmake运行时读取CMakeLists.txt文件进行编译。
再次以文章开头的工程进行编译 main.c，hello.h，hello.c

文件结构

```bash
  |--CMakeLists.txt
  |--main.c
  |--hello.c
  └--hello.h
```

CMakeLists.txt

```cmake
# CMake 最低版本号要求
cmake_minimum_required (VERSION 3.21.4)
# 项目信息
project (Demo1)
# 头文件搜素目录 相当于 -I参数的作用
include_directories(.)
# 指定生成为可执行文件
add_executable(Demo main.c hello.c)
```



- project	 				项目名称
- include_directories	   	头文件搜素目录 相当于 -I参数的作用
- add_executable  		 	生成可执行文件



>
> **官方文档:**[https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#id14](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html#id14)



**执行一下**

命令：`cmake + path`  会搜索path底下的CMakeLists.txt并执行编译。
PS:注意此处是生成makefile，并不会帮你生成可执行文件
```bash
➜  cmake .
-- The C compiler identification is AppleClang 13.1.6.13160021
-- The CXX compiler identification is AppleClang 13.1.6.13160021
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/xiaoxiang/Documents/code/learn_make/cmake/simplest
```

此时已生成了**makefile文件**
![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657775638550-456415fe-61c2-4830-ba59-db6d3f5100a3.png#clientId=ub24cf619-5120-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=254&id=udfef0dde&margin=%5Bobject%20Object%5D&name=image.png&originHeight=254&originWidth=223&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20126&status=done&style=none&taskId=u965b3275-1b33-4247-9558-367b07ba4c3&title=&width=223)

再次执行make   ，并执行 Demo可执行文件

```bash
➜  make 
[ 33%] Building C object CMakeFiles/Demo.dir/main.c.o
[ 66%] Building C object CMakeFiles/Demo.dir/hello.c.o
[100%] Linking C executable Demo
[100%] Built target Demo
➜ ./Demo 
Hello makefiles!
```


回头仔细看看cmake帮我们做了什么。

###### 隐秘的Configure + Genterate

```bash
cmake -S . -B build 
-- The C compiler identification is AppleClang 13.1.6.13160021
-- The CXX compiler identification is AppleClang 13.1.6.13160021
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/xiaoxiang/Documents/code/learn_make/cmake/simplest/build
```

![image.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657776459104-d31049d9-c6c5-44ab-976b-05d5b3657c07.png#clientId=ub24cf619-5120-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=477&id=u41e80f29&margin=%5Bobject%20Object%5D&name=image.png&originHeight=477&originWidth=1600&originalType=binary&ratio=1&rotation=0&showTitle=false&size=345635&status=done&style=none&taskId=ubbfecfa9-da71-498b-a225-6e3d01f9a81&title=&width=1600)

**Configure:** 检测本地机器编译工具，查找依赖等信息
**Generate::** 生成编译缓存文件** **CMakeCache.txt，安装相关文件cmake_install.cmake

```cmake

//CXX compiler 编译器地址
CMAKE_CXX_COMPILER:FILEPATH=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/c++

...

//The product will be built against the headers and libraries located
// inside the indicated SDK.
/  系统sdk地址
CMAKE_OSX_SYSROOT:PATH=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX12.3.sdk

...
```

#### 4.换一个大厨炒菜

---

##### ![ninja+gn副本.png](https://intranetproxy.alipay.com/skylark/lark/0/2022/png/311452/1657780007305-72275f4b-05f2-46a5-8023-2465cc9d2c07.png#clientId=u44171a42-aacb-4&crop=0&crop=0&crop=1&crop=1&from=drop&height=497&id=u9e0d117c&margin=%5Bobject%20Object%5D&name=ninja%2Bgn%E5%89%AF%E6%9C%AC.png&originHeight=713&originWidth=926&originalType=binary&ratio=1&rotation=0&showTitle=false&size=183871&status=done&style=none&taskId=u07577f8d-809f-4218-b71d-5c99c93d3e2&title=&width=645)

#### Ninja
Ninja == make 角色
ninja的初衷是替换make，它比make更加快。单ninja可读性很差，手写比较困难，通常的用法是使用构建工具（gn,cmake）去生成

>
> ninja官网：https://ninja-build.org/

#### GN

```
|--build
|   └--toolchain	
|  	|		└--BUILD.gn	    #工具链声明
|   └--BUILD.gn					
|   └--BUILDCONFIG.gn
|--.gn									#GN构建的根目录    	
|--BUILD.gn              
|--main.c
|--hello.c
└--hello.h
```

BUILD.gn
```bash
executable("hello") {
  include_dirs = [.]
  sources = [ "main.c","hello.c" ]
}
```

使用 `gn gen out` 生成build.ninja文件

```bash
➜  gn gen out
Done. Made 1 targets from 4 files in 5ms
```

使用`ninja -C out` 编译

```bash
➜ ninja -C out 
ninja: Entering directory `out'
[3/3] LINK hello
```
##### 主要文件

- .gn  GN构建的根目录    

 文件中引用"//"指向目录

- BUILDCONFIG.gn	

.gn文件中指向的配置文件，用于声明一些toolchain、全局变量等

- BUILD.gn   具体的编译配置文件
##### 语法

二进制/模块声明
```
static_library("base") {
  sources = [
    “a.cc”,
    “b.cc”,
  ]
}
```
内置target类型	
target可以是可执行文件，也可以是执行一段脚本，移动文件。

- executable, shared_library, static_library     **二进制**
- source_set: compiles source files with no intermediate library  **源文件集合**
- group: a named group of targets (deps but no sources)  **target组**
- copy   **文件拷贝**
- action, action_foreach: run a script   **执行脚本**
- bundle_data, create_bundle: Mac & iOS ** iOSbudnle打包**

模块依赖引用：

```
static_library(“base”) {
  sources = [
    “a.cc”,
    “b.cc”,
  ]

  deps = [
    “//fancypants”,
    “//foo/bar:baz”,
 ]
}
```

编译配置

```
executable(“doom_melon”) {
  sources = [ “doom_melon.cc” ]

  cflags = [ “-Wall” ]
  defines = [ “EVIL_BIT=1” ]
  include_dirs = [ "." ]

  deps = [ “//base” ]
}

```
变量
可使用$引用字符拼接
```
target_out_dir =/Users/xiaoxiang/Documents/code
    
"$target_out_dir/output.txt"
```
路径

```
/#根目录/chrome/browse/BUILD.gn名为version的模块
//chrome/browser:version

#当前文件下的模块
:baz
```

条件语句

```
component(“base”) {
  sources = [
    “a.cc”,
    “b.cc”,
  ]

  if (is_win || is_linux) {
    sources += [ “win_helper.cc” ]
  } else {
    sources -= [ “a.cc” ]
  }
}
```

单独配置文件 *.gni
```
import(“//foo/build.gni”)
```

自定义模板
```
template("grit") {
  …
}

grit("components_strings") {
  source = “components.grd”
  outputs = [ … ]
}

```

##### 交叉编译
为不同于宿主host设备编译程序。

例子：
添加**target_os**指定目标操作系统

```bash
gn gen out/Default --args='target_os="android"' 
```

根据不同操作系统定义不同的toolchain，封装不同的**编译器工具集合**
当进行编译时，会调用工具集中程序进行编译

```bash
if(target_os == "android"){
    set_default_toolchain("//build/toolchain/mac:clang_android_$current_cpu")
}else if (target_os == "ios"){
    set_default_toolchain("//build/toolchain/mac:clang_ios_$current_cpu")
}else{
    set_default_toolchain("//build/toolchain/gcc:gcc")
}
```

如何定义不同**编译器工具集合**

```bash
toolchain("gcc") {
  tool("cc") {
    command = "gcc -MMD -MF $depfile {{defines}} {{include_dirs}} {{cflags}} {{cflags_c}} -c {{source}} -o {{output}}"
  }

  tool("cxx") {
    depfile = "{{output}}.d"
    command = "g++ -MMD -MF $depfile {{defines}} {{include_dirs}} {{cflags}} {{cflags_cc}} -c {{source}} -o {{output}}"
   }

  tool("alink") {
    command = "rm -f {{output}} && ar rcs {{output}} {{inputs}}"
  }

  tool("solink") {
    command = "g++ -shared {{ldflags}} -o $sofile $os_specific_option @$rspfile"
  }

  tool("link") {
    command = "g++ {{ldflags}} -o $outfile -Wl,--start-group @$rspfile {{solibs}} -Wl,--end-group {{libs}}"
  }

  tool("stamp") {
    command = "touch {{output}}"
    description = "STAMP {{output}}"
  }

  tool("copy") {
    command = "cp -af {{source}} {{output}}"
    description = "COPY {{source}} {{output}}"
  }
}

```


**编译相关工具集**

- **cc  C compiler**
- **cxx: C++ compiler**     
- cxx_module": C++ compiler used for Clang .modulemap files      
- **objc Objective C compiler**
- **objcxx: Objective C++ compiler**
-  rc:Resource compiler (Windows .rc files)     
-  asm : Assembler       
- swift : Swift compiler driver

**链接相关工具集**

- **alink**: Linker for static libraries (archives)       静态链接
- **solink**: Linker for shared libraries           	 动态链接
- **link**: Linker for executables			        可执行文件之间链接


不同操作系统使用不同toolchain

### 5. 探究flutter的GN编译脚本

---

把flutter源码下载后，以iOS为例子，在src目录下执行 命令，便可以的执行构建。

`./flutter/tools/gn --ios --unoptimized` 
`ninja -C out/ios_debug_unopt`

我没接来下详细看看这里的整个执行构建
##### 此gn非彼gn
./flutter/tools/gn  其实是一个python脚本，他的作用是根据入参来调整最终传递给gn可执行文件的gn参数
比如传入 `./flutter/tools/gn --ios`后会翻译成gn参数 target_os = ios 

```python
def parse_args(args):
    parser.add_argument('--ios', dest='target_os', action='store_const', const='ios')
  .....
```

转换完需要的参数后，最后调用才是位于 `flutter/third_party/gn/gn` 的可执行文件

```python
def main(argv):
    args = parse_args(argv)
    
    exe = '.exe' if sys.platform.startswith(('cygwin', 'win')) else ''
    
    command = [
        '%s/flutter/third_party/gn/gn%s' % (SRC_ROOT, exe),
        'gen',
        '--check',
        '--export-compile-commands',
  ]
```

最后执行的是 `flutter/third_party/gn/gn gen  GN参数`

##### flutter源码结构
先来看看flutter源码的大致结构，使用depot_tools把源码拉下来后大致的文件夹结构如下

```bash
.gclient           				depot_tools文件，声明下载的引擎分支
src
  ├---flutter							flutter源码库https://github.com/flutter/engine.git
  ┆ 		├---fml						基础库
  ┆ 		┆	├---XXX.cpp
  ┆ 		┆	└---BUILD.gn 
  ┆ 		├---lib						dart类封装
  ┆ 		├---flow					渲染相关
  ┆ 		┆	├---XXX.cpp
  ┆ 		┆	└---BUILD.gn
  ┆ 		└---BUILD.gn			
  ├---build								flutter构建库https://github.com/flutter/buildroot.git
  ├---third_party  				三方库
  ├---tools								工具
  └---.gn									gn根配置文件
  
```

首先看到.gn 文件，文件所在的目录构建的根目录，也是脚本中使用//代码的位置，比如 `//flutter` 则代表 `src/flutter` 目录

**.gn  **

```bash
buildconfig = "//build/config/BUILDCONFIG.gn"

```

其中buildconfig是系统变量，这里的//build/config/BUILDCONFIG.gn指定了构建的初始变量以及

flutter的构建由一个个相互依赖的target组成，而每个target都非常规整的写在了每个文件夹的BUILD.gn里面
比如`src/flutter/flow/BUILD.gn` 就恰好声明了一名为flow的target，这是用于声明源文件的source_set 

```bash
source_set("flow") {
    "compositor_context.cc",
    ...
}
```

比如`src/flutter/shell/platform/embedder/BUILD.gn`中会名为`embedder_source_set`引用这个`flow` target

```python
template("embedder_source_set") {
    ...
    
    deps = [
 
      "//flutter/flow",
    ]
}
```

##### flutter源码的target依赖图
依次类推，通过deps层层依赖，下面是整个flutter大致的target依赖图

```bash
src/flutter																												src/flutter/BUILD.gn
    ├┈//flutter/sky
    └┈//flutter/shell/platform/embedder:flutter_engine     	   	  flutter/shell/platform/embedder/BUILD.gn
        ├┈:copy_headers																						拷贝头文件
        └┈:flutter_engine_library								 									flutter/shell/platform/embedder/BUILD.gn							
          └┈:embedder									 														flutter/shell/platform/embedder/BUILD.gn
            ├┈embedder_gpu_configuration
            ├┈//flutter/assets
            ├┈//flutter/common
            ├┈//flutter/common/graphics
            ├┈//flutter/flow
            ├┈//flutter/fml
            ├┈//flutter/lib/ui
            ├┈//flutter/runtime:libdart
            ├┈//flutter/shell/common
            ├┈//flutter/third_party/tonic
            ├┈//third_party/dart/runtime/bin:dart_io_api
            ├┈//third_party/dart/runtime/bin:elf_loader
            └┈//third_party/skia																third_party/skia/BUILD.gn
```

自此，当执行`./flutter/tools/gn --ios --unoptimized` 后，会生成从src/flutter/BUILD.gn开始找到target名为"flutter"的target，并以此通过deps来组织层层添加依赖。最终把整个源码联系起来。

```cpp
group("flutter") {
  testonly = true

  public_deps = [
    "//flutter/shell/platform/embedder:flutter_engine",
    "//flutter/sky",
  ]
  ...
 }
```

### 6.总结
本文从最原始的调用编译器，到使用大型工程的编译工具cmake或gn，最后聊聊flutter中的代码是如何组织代码结构，以及编译系统。其中一些概念为易于理解用词不太准确，如有纰漏，很高兴能读者批评指出一起探讨。


引用资料
>
> [https://thoughtbot.com/blog/the-magic-behind-configure-make-make-install](https://thoughtbot.com/blog/
the-magic-behind-configure-make-make-install)
>
> GN Reference: [https://gn.googlesource.com/gn/+/main/docs/reference.md](https://gn.googlesource.com/gn/+/main/docs/reference.md)
>
> [https://opensource.com/article/18/8/what-how-makefile](https://opensource.com/article/18/8/what-how-makefile)
>
> GCC编译命令 [https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html)
>
> Clang编译命令：[https://clang.llvm.org/docs/DiagnosticsReference.html#winvalid-noreturn](https://clang.llvm.org/docs/DiagnosticsReference.html#winvalid-noreturn)
>
> makefil超简单入门：[https://www3.nd.edu/~zxu2/acms60212-40212/Makefile.pdf](https://www3.nd.edu/~zxu2/acms60212-40212/Makefile.pdf)
>
> morden buid system:[https://julienjorge.medium.com/an-overview-of-build-systems-mostly-for-c-projects-ac9931494444](https://julienjorge.medium.com/an-overview-of-build-systems-mostly-for-c-projects-ac9931494444)
>
> CMAKE:交叉编译[https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html](https://cmake.org/cmake/help/book/mastering-cmake/chapter/Cross%20Compiling%20With%20CMake.html)
>
> 官方网站：[https://gn.googlesource.com/gn/](https://gn.googlesource.com/gn/)
>
> quick start：[https://gn.googlesource.com/gn/+/main/docs/quick_start.md](https://gn.googlesource.com/gn/+/main/docs/quick_start.md)
>
> 文档：[https://gn.googlesource.com/gn/+/main/docs/reference.md](https://gn.googlesource.com/gn/+/main/docs/reference.md)
>
> presentation:[https://docs.google.com/presentation/d/15Zwb53JcncHfEwHpnG_PoIbbzQ3GQi_cpujYwbpcbZo/edit#slide=id.g4eba7c1434_142_0](https://docs.google.com/presentation/d/15Zwb53JcncHfEwHpnG_PoIbbzQ3GQi_cpujYwbpcbZo/edit#slide=id.g4eba7c1434_142_0)
>
> GN Cross Compile: [https://gn.googlesource.com/gn/+/HEAD/docs/cross_compiles.md](https://gn.googlesource.com/gn/+/HEAD/docs/cross_compiles.md)

