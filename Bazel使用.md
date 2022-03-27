# Bazel介绍

Bazel是一个类似于Make的编译工具，是Google为其内部软件开发的特点量身定制的工具，现在Google内部大部分软件都用Bazel进行构建。

# 主要特点

- 多语言支持：Bazel支持Java，Objective-C和C++，可以扩展来支持任意的编程语言
- 高级别的构建语言：工程是通过BUILD语言来描述的。BUILD语言以简洁的文本格式，描述了由多个小的互相关联的库、二进制程序和测试程序来组成的一个项目。而与之相比，Make这类的工具需要描述各个单独的文件和编译的命令
- 多平台支持：同一套工具和同样的BUILD文件可以用来构建不同架构和不同平台的软件
- 可重现性：在BUILD文件中，每个库，测试程序，二进制文件必须明确完整地指定直接依赖。当修改源代码文件后，Bazel使用这个依赖信息就可以知道哪些必须重新构建，哪些任务可以并行执行。这意味者所有的构建都是增量形式的并能够每次都生成相同的结果
- 可扩展性：Bazel可以处理巨大的构建，并且可以扩展支持新语言和新平台
- 编译迅速：Bazel可以成倍提高构建速度，对依赖关系进行优化，支持并发执行；只重新编译修改的文件
- 可伸缩性：可处理任意大小的代码库，并且库可以任意增加或缩减

# 适用项目：
- 有庞大代码库的项目
- 用多种语言写的项目
- 在多平台上部署的项目
- 有大量测试的工程

# 项目结构

Bazel工程的顶层为根目录，即工作区间Workspace；Workspace下包含多个Package，而每个Package又包含多个编译目标Target。

- Workspace
  Bazel的编译是基于Workspace的概念，工作区存放了所有源代码和Bazel编译输出的文件。
  Bazel工程的根目录下，必须包含一个文件名为Workspace的文件，它用来指明构建的根目录，Workspace采用类似于Python的语法。
- Package
  Workspace下可以有多个Package，每个Package都可以作为一个子模块，并且每个Package下必须包含一个BUILD文件，用来指定Package编译构建的规则。
- Target
  Target是Bazel的最小组成

# Bazel安装

**用二进制安装程序安装**

## 第1步：安装所需的包

```
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python3
```

## 第2步：下载Bazel



接下来，`bazel-<version>-installer-linux-x86_64.sh` 从[GitHub上](https://github.com/bazelbuild/bazel/releases)的[Bazel发布页面](https://github.com/bazelbuild/bazel/releases)下载Bazel二进制安装程序。例如``bazel-6.0.0-pre.20220310.1-installer-linux-x86_64.sh``

![image-20220319162120922](D:\自动驾驶\AutoDriving\note\picture\image-20220319162120922.png)

![image-20220319162239150](D:\自动驾驶\AutoDriving\note\picture\image-20220319162239150.png)

## 第3步：运行安装程序

按如下方式运行Bazel安装程序：

先改一下权限，

```xml
chmod +x bazel-<version>-installer-linux-x86_64.sh
```

其中，<version>--是我们下载的版本号，如：

`` chmod +x bazel-6.0.0-pre.20220310.1-installer-linux-x86_64.sh``

然后运行Bazel安装程序：

```xml
./bazel-<version>-installer-linux-x86_64.sh --user
```

<version>--是我们下载的版本号

如：

``./bazel-6.0.0-pre.20220310.1-installer-linux-x86_64.sh --user**``

该`--user`标志将Bazel安装到`$HOME/bin`系统上的目录并设置`.bazelrc`路径`$HOME/.bazelrc`。使用该`--help`命令可以查看其他安装选项。

## 第4步：设置您的环境

如果您使用`--user`上面的标志运行Bazel安装程序，则Bazel可执行文件将安装在您的`$HOME/bin`目录中。将此目录添加到默认路径是个好主意，如下所示：

```bash
export PATH="$PATH:$HOME/bin"
```

您也可以将此命令添加到您的`~/.bashrc`文件中。

## ![image-20220319162716472](D:\自动驾驶\AutoDriving\note\picture\image-20220319162716472.png)



# 在Windows系统上安装配置Graphviz

可以不安装

**Graphviz 能干啥？**

“绘图工具”有很多种，Graphviz 主要是用来绘制【关系图】。所以它更类似于微软的 Visio。但是它与 Visio 有一个【本质上】的差异：

- 用 Visio 画图是【手动】的 — — 你需要动用你的肉眼和手指头。
- 用 Graphviz 画图是【自动】的 — — 你只需要告诉 Graphviz 这张图包含哪些元素，元素之间有啥关系，然后 Graphviz 可以【自动】帮你画出来。

**安装方法**

1. 首先下载安装包**graphviz-2.38.msi**，下载地址为https://graphviz.gitlab.io/_pages/Download/windows/graphviz-2.38.msi
2. 双击msi文件，然后一直选择next（默认安装路径为C:\Program Files (x86)\Graphviz2.38\），安装完成之后，会在windows开始菜单创建快捷信息。
3. 配置环境变量：计算机→属性→高级系统设置→高级→环境变量→系统变量→path，在path中加入路径：C:\Program Files (x86)\Graphviz2.38\bin
4. 验证：在windows命令行界面，输入dot -version，然后按回车，如果显示如下图所示的graphviz相关版本信息，则安装配置成功。



# Bazel编译Demo

bazel 在**编译速度、编译规则编写复杂性、跨平台编译** 上相比 **手写Makefile + [Automake](https://link.zhihu.com/?target=https%3A//www.gnu.org/software/automake/)** 都有很大优势。

Bazel提供了一些编译的例子，在Github上可以找到：https://github.com/bazelbuild/examples/。现选取其中的examples/cpp-tutorial进行演示：

<font color=Blue></font>

<font color=Blue>**examples**</font>
└── <font color=Blue>cpp-tutorial</font>
    ├──**stage1**
    │  └── main
    │      ├── BUILD
    │      ├── hello-world.cc
    │  └── WORKSPACE
    ├──**stage2**
    │  ├── main
    │  │   ├── BUILD
    │  │   ├── hello-world.cc
    │  │   ├── hello-greet.cc
    │  │   ├── hello-greet.h
    │  └── WORKSPACE
    └──**stage3**
       ├── main
       │   ├── BUILD
       │   ├── hello-world.cc
       │   ├── hello-greet.cc
       │   └── hello-greet.h
       ├── lib
       │   ├── BUILD
       │   ├── hello-time.cc
       │   └── hello-time.h
       └── WORKSPACE

example一共分成了三组文件，分别对应以下列举的三个例子。

>  第一个例子：学习如何构建单个Package中的单个Target；
>
> 第二个例子：将整个项目拆分成单个Package的多个Target。
>
> 第三个例子：将项目拆分成多个Package，用多个Target编译。

# 理解工作区

工作区是一个存放了所有源代码和Bazel编译输出文件的目录，也是整个项目的根目录。同时它包含一些bazel认识的文件：

- WORKSPACE文件，用于指定当前文件夹就是一个Bazel的工作区，所以WORKSPACE文件总是存在于项目的根目录下；

- 一个或多个BUILD文件，告诉Bazel怎么构建项目的不同部分。（如果工作区中的一个目录包含BUILD文件，那么它就是一个package）。

那么要指定一个目录为Bazel的工作区，就只要在该目录下创建一个空的WORKSPACE文件即可 

```c++
touch WORKSPACE //创建文件
bazel info workspace //显示WORKSPACE路径
```

# 理解BUILD文件

一个BUILD文件包含多种指令。其中最重要的是编译指令，它告诉Bazel如何编译想要的输出，比如可执行二进制文件或库。BUILD文件中的每一条编译指令被称为一个Target，它指向一系列的源文件和依赖，一个Target也可以指向别的Target。

`cc_binary(`
    `name = "hello-world",`
    `srcs = ["hello-world.cc"],`
`)`

一个名为“**hello-world**”的Target使用了Bazel内置的**cc_binary**编译指令，从**hello-world.cc**源文件（没有其他依赖）构建了一个**可执行二进制文件**。

## 第一个例子

进入 examples/cpp-tutorial/stage1 路径下，首先通过下面的命令来编译该例子：

`bazel build //main:hello-world`

Target中的`//main:`是BUILD文件相对于WORKSPACE文件的位置，`hello-world`是我们在BUILD文件中命名号的Target名字。

输出以下信息表示编译成功：

> INFO: Analyzed target //main:hello-world (1 packages loaded, 1 target configured).
> INFO: Found 1 target...
> Target //main:hello-world up-to-date:
>   bazel-bin/main/hello-world
> INFO: Elapsed time: 0.240s, Critical Path: 0.18s
> INFO: 4 processes: 2 internal, 2 linux-sandbox.
> INFO: Build completed successfully, 4 total actions

Bazal将编译的输出放在项目根目录下``examples/cpp-tutorial/stage1/bazel-bin/main``目录.

BUILD文件内容

`load("@rules_cc//cc:defs.bzl", "cc_binary")`

`cc_binary(`
    `name = "hello-world",`
    `srcs = ["hello-world.cc"],`
`)`

### 查看依赖图

成功地构建一个项目，需要将项目所有的依赖都定义在BUILD文件内，Bazel可以通过命令生成可视化关系图，来检查这些依赖项.
以下是检查该项目的依赖命令:

> bazel query --nohost_deps --noimplicit_deps 'deps(//main:hello-world)' \
>   --output graph

输出结果为输出图的文字描述,通过粘贴文字描述到[GraphViz](http://www.webgraphviz.com/),就可以查看到如下的依赖图:

![image-20220319210512621](D:\自动驾驶\AutoDriving\note\picture\image-20220319210512621.png)

# 第二个例子

编译方法 bazel build //main:hello-world

单个Target编译对于小项目来说非常高效,但是对于大项目而言,将项目拆分成多个Target和多个Package更加高效,因为Bazel能够实现快速增量的编译.
第二个例子将项目拆分成两个Target,cpp-tutorial/stage2/main目录下的BUILD文件如下所示:

> load("@rules_cc//cc:defs.bzl", "cc_binary", "cc_library")
>
> cc_library(
>     name = "hello-greet",
>     srcs = ["hello-greet.cc"],
>     hdrs = ["hello-greet.h"],
> )
>
> cc_binary(
>     name = "hello-world",
>     srcs = ["hello-world.cc"],
>     deps = [
>         ":hello-greet",
>     ],
> )

从BUILD文件可以看出Bazel首先编译了hello-greet这个库（利用Bazel内置的cc_library编译指令），然后编译hello-world这个二进制文件。hello-world这个Target的deps属性告诉Bazel，要构建hello-world这个二进制文件需要hello-greet这个库。

接下来再来看看项目生成的依赖图:

> bazel query --nohost_deps --noimplicit_deps 'deps(//main:hello-world)' \
>   --output graph

![image-20220319210404669](D:\自动驾驶\AutoDriving\note\picture\image-20220319210404669.png)

`hello-world`在编译时候的结构和例子1有所不同，现在是有两个Targets。`hello-world`这个Target从一个源文件编译而来，同时依赖于另一个Target`//main:hello-greet`，这个Target又是从两个源文件编译而来。

# 第三个例子

编译方法 bazel build //main:hello-world

第三个例子将项目拆分成多个Package,`cpp-tutorial/stage3`目录如下:

> └──stage3
>    ├── main
>    │   ├── BUILD
>    │   ├── hello-world.cc
>    │   ├── hello-greet.cc
>    │   └── hello-greet.h
>    ├── lib
>    │   ├── BUILD
>    │   ├── hello-time.cc
>    │   └── hello-time.h
>    └── WORKSPACE



注意到我们现在有两个子目录了，每个子目录中都包含了BUILD文件。因此，对于Bazel来说，整个工作区现在就包含了两个Package：`lib`和`main`,两个Package的BUILD文件如下:
`lib/BUILD:`

>cc_library(
>    name = "hello-time",
>    srcs = ["hello-time.cc"],
>    hdrs = ["hello-time.h"],
>    visibility = ["//main:__pkg__"],
>)

注意到这个Target显式可见了（通过`visibility`属性）。这是因为默认情况下，Targets只对同一个BUILD文件里的其他Targets可见（Bazel使用Target visibility来防止像公有API中库的实现细节的泄露等情况）。

main/BUILD:

> load("@rules_cc//cc:defs.bzl", "cc_binary", "cc_library")
>
> cc_library(
>     name = "hello-greet",
>     srcs = ["hello-greet.cc"],
>     hdrs = ["hello-greet.h"],
> )
>
> cc_binary(
>     name = "hello-world",
>     srcs = ["hello-world.cc"],
>     deps = [
>         ":hello-greet",
>         "//lib:hello-time",
>     ],
> )

接下来再来看看项目生成的依赖图:

> bazel query --nohost_deps --noimplicit_deps 'deps(//main:hello-world)' \
>   --output graph

![image-20220319210133797](D:\自动驾驶\AutoDriving\note\picture\image-20220319210133797.png)