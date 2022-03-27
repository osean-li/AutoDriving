##  1 引言

在网上或者官方教程中，通常使用`terminal`进入`docker`，从而进行`apollo`的调试，这对于后端(服务器)人员是很友好的。

处于算法开发初期的工程师，若能拥有图形化界面，能够给自己写的代码设置断点(需要结合log)，调试时间及调试难度会大大降低。

## 2 vscode进入docker容器

首先要在vscode里安装docker，安装成功后，如下图所示

![image-20220321215513783](D:\自动驾驶\AutoDriving\note\picture\image-20220321215513783.png)

在使用`vscode`进入`dokcer`容器之前，需要按照官方教程安装好`apollo`，需要启动`apollo`环境

注意此处步要用root权限进入docker

```text
cd apollo/
bash docker/scripts/dev_start.sh
```

1.`vscode`需要借助`Remote-Containers`插件进入`apollo`的`docker`容器，安装`Remote-Containers`插件

![image-20220321215851211](D:\自动驾驶\AutoDriving\note\picture\image-20220321215851211.png)

2.在`vscode`左侧侧边栏会显示`Remoter Explorer`图标，点击图标后会显示本机安装的容器，选择`apolloauto/apollo:dev`(即选择`apollo`的`docker`运行环境容器)，右键选择`Attach to Container`即可。

注意：图中显示两个apollo容器，一个是普通用户，一个是root用户，选择普通用户的登录，root用户一直提示权限问题

![image-20220321220014228](D:\自动驾驶\AutoDriving\note\picture\image-20220321220014228.png)

3.在新弹出来的`vscode`界面，其左下角会显示当前的`docker`容器信息，点击左上角file选择文件夹`/apollo/`，点击ok打开。

如下图显示，表示进入docker成功了

![image-20220321220520265](D:\自动驾驶\AutoDriving\note\picture\image-20220321220520265.png)

## 配置容器

在`vscode`中使用`docker`的终端环境，需要配置容器内用户和环境，按`ctrl+shift+p`，输入`open container configuration file`，打开容器配置文件，主要填写`remoteUser`和`remoteEnv`，`remoteUser`填写本机用户名。此外，在容器的`vscode`，可自行安装需要的插件，比如C/C++。

打开容器配置文件，配置容器的一些参数，如进入容器后的一些环境变量，用户等，我的配置如下：

![image-20220321220803055](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20220321220803055.png)

```json
{
    "workspaceFolder": "/",
    "extensions": [
        "ms-vscode.cpptools"
    ],
    "remoteUser": "yons",
    "remoteEnv": {
        "HISTFILE": "/apollo/.dev_bash_hist"
    }
}
```

## 开始调试

打开Apollo所在的文件夹，重新编译Apollo代码，生成带调试信息的可执行文件。

> `bash apollo.sh build_dbg`

**生成调试文件，可参考launch.json**

> {
>     // 使用 IntelliSense 了解相关属性。 
>     // 悬停以查看现有属性的描述。
>     // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
>     "version": "0.2.0",
>     "configurations": [
>         {
>             "name": "g++ - 生成和调试活动文件",
>             "type": "cppdbg",
>             "request": "launch",
>             "program": "/apollo/bazel-bin/cyber/mainboard/mainboard",
>             "args": ["-d","/apollo/modules/localization/dag/dag_streaming_msf_localization.dag"],
>             "stopAtEntry": false,
>             "cwd": "/apollo",
>             "environment": [],
>             "externalConsole": false,
>             "MIMode": "gdb",
>             "setupCommands": [
>                 {
>                     "description": "为 gdb 启用整齐打印",
>                     "text": "-enable-pretty-printing",
>                     "ignoreFailures": true
>                 }
>             ],
>             // "preLaunchTask": "C/C++: g++ 生成活动文件",
>             "miDebuggerPath": "/usr/bin/gdb"
>         }
>     ]
> }

本launch.json文件以调试localization模块为例，其中program为mainboard可执行文件的路径，用于运行dag文件，args为要调试模块的dag文件，若要调试其他模块，修改args即可。
注意：cwd必须为Apollo项目所在路径，否则调试过程中会找不到相关的代码文件。

## 容器环境调试apollo

设置一个断点,快捷键`F5`开始调试

![image-20220321221313432](D:\自动驾驶\AutoDriving\note\picture\image-20220321221313432.png)

Enjoy！

### 调试test

`apollo`每个模块都编test写有对应的单元测试文件(后缀带有test)，在很多资料介绍中，很少有介绍是怎么调试的。单元测试基于`google_test`编写，想编写单元测试代码的可自行查阅`google_test`规则。

**以control_component_test为例**

`launch.json`如下

```json
{
    "version": "0.2.0",
    "configurations": [

        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "/apollo/bazel-bin/modules/control/control_component_test",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "miDebuggerPath": "/usr/bin/gdb"

        }
    ]
}
```

同理，编写好调试脚本后，在`control_component_test.cc`设置一个断点，按F5开始调试，程序会停止在断点处。由此，可进行单元测试的调试。



##  结语

本文比较适合刚接触`apollo`的算法工程师及其他人员，对于具有相当丰富`apollo`调试经验的人员来说可能并不具有帮助作用，可根据需要自行选择是否按照本文进行`apollo`的调试。