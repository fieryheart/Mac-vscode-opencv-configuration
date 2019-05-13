# macOS+vscode+opencv环境配置

## 1. 版本说明

+ macOS: 10.14.3  
+ vscode: 1.32.3
+ opencv: 3.4.0

## 2. opencv的安装

我选择的方案是下载压缩包，使用终端进行编译。

2.1. 安装cmake，`brew install cmake`

2.2 终端进入解压之后的文件夹位置，`cd /Users/$(your_user_name)/opencv`， 然后创建一个新的文件夹，`mkdir build`， 文件夹命名为build

2.3 终端进入文件夹build，`cd build`

2.4 cmake编译，`cmake -G "Unix Makefiles" ..`

2.4 然后，`make -j8`

2.6 最后，`sudo make install`  

opencv的库文件会出现在`build/lib`中，并且在`/usr/local/include`应该会出现opencv和opencv2两个文件夹，opencv也算安装完成。

## 3. vscode的c/c++执行以及调试环境搭建（非opencv）

文件目录如下：  
![Alt text](/img/directory.jpg)  
创建4个文件，分别为`c_cpp_properties.json`、`launch.json`、`tasks.json`和`setting.json`，接下来就是这个四个文件的内容，（Tips: 在这些文件中，将鼠标悬浮在属性上，会出现说明，比如你将鼠标悬浮在`c_cpp_properties.json`中的`compilePath`上会出现`Full path of the compile being used`文字说明）

### 3.1 c_cpp_properties.json

随便打开一个C++源文件，点击一下`#include`旁边的黄灯，然后点击`Add include path to settings`，VScode会弹出`c_cpp_properties.json`，`c_cpp_properties.json`时用来配置`#include`的路径环境。

```js
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [],
            "defines": [],
            "macFrameworkPath": [
                "/System/Library/Frameworks",
                "/Library/Frameworks",
                "${workspaceFolder}/**"
            ],
            "compilerPath": "/usr/bin/clang++",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64",
            "browse": {
                "path": [
                    "${workspaceFolder}/**"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            }
        }
    ],
    "version": 4
}
```

其中，关注`includePath`和`browse.path`，在后面配置opencv时会用到

### 3.2 launch.json

`launch.json`在debug模式下会自动生成，是用来配置debug环境的。

```js
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(lldb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}.out",
            "args": [],
            "cwd": "${workspaceFolder}",
            "stopAtEntry": true, // if true then stop at the main entry (function)
            "environment": [],
            "externalConsole": true,
            "MIMode": "lldb",
            "preLaunchTask": "build hello world"
        }
    ]
}
```

其中，`program`属性中文件的后缀与自身系统有关，文件默认是没有后缀名的；`"preLaunchTask":"build hello world"`表示执行文件前需要编译的任务，具体任务内容在`tasks.json`中定义。

### 3.3 tasks.json

快捷键`shift+command+p` 打开`Tasks: Configure Tasks`，选择 `Create tasks.json file from templates`，此时会蹦出一个下拉列表，在下拉列表中选择`Others`。

```js
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    // 有关 tasks.json 格式的文档，请参见
    // https://go.microsoft.com/fwlink/?LinkId=733558
    "version": "2.0.0",
    "tasks": [
        {
            "type": "shell",
            "label": "build hello world",
            "command": "clang++",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}.out"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": [
                "$gcc"
            ]
        }
    ]
}
```

其中，`label`中写上与`launch.json`的`preLaunchTask`相同的值；`command`选择你使用的编译器(`g++`或`clang++`)，注意`args`的值，在后面debug中因为要配置opencv所以会更改值。

### 3.4 setting.json

需要安装`code-runner`这个扩展文件，`setting.json`是工作区设置。vscode中进入`Code -> Preferences -> Settings`选择某个属性，然后点击`Edit in settings.json`让其自动生成`setting.json`，这个文件是在设置运行状态的环境的。

```js
{
    "code-runner.executorMap": {
        "c": "cd $dir && clang $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt",
        "cpp": "cd $dir && clang++ $fileName -o $fileNameWithoutExt && $dir$fileNameWithoutExt"
    },
    "code-runner.runInTerminal": true,
    "[makefile]": {
        "editor.insertSpaces": true
    },
    "C_Cpp.default.includePath": []
}
```

`executorMap`属性中，默认的执行器是`g++`，这样设置后，使用`control+option+N`会把执行器换成`clang++`。

## 4. 引入头文件和外链库（如opencv）

修改三个文件，分别为`c_cpp_properties.json`、`tasks.json`和`settings.json`，创建编译文件`Makefile`

### 4.1 c_cpp_properties.json修改

```js
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**",
                "/usr/local/include"
            ],
            "defines": [],
            "macFrameworkPath": [
                "/System/Library/Frameworks",
                "/Library/Frameworks",
                "${workspaceFolder}/**"
            ],
            "compilerPath": "/usr/bin/clang++",
            "cStandard": "c11",
            "cppStandard": "c++17",
            "intelliSenseMode": "clang-x64",
            "browse": {
                "path": [
                    "${workspaceFolder}/**"，
                    "/usr/local/include"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": ""
            }
        }
    ],
    "version": 4
}
```

修改`"includePath"`和`"browser.path"`

### 4.2 tasks.json修改

```js
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  // 有关 tasks.json 格式的文档，请参见
  // https://go.microsoft.com/fwlink/?LinkId=733558
  "tasks": [
    {
      "type": "shell",
      "label": "build",
      "command": "clang++",
      "args": [
        "${file}",
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}.out",
        "-I",
        "/usr/local/include/opencv",
        "-I",
        "/usr/local/include/",
        "-L",
        "/usr/local/lib",
        "-l",
        "opencv_dnn",
        "-l",
        "opencv_ml",
        "-l",
        "opencv_objdetect",
        "-l",
        "opencv_shape",
        "-l",
        "opencv_stitching",
        "-l",
        "opencv_superres",
        "-l",
        "opencv_videostab",
        "-l",
        "opencv_calib3d",
        "-l",
        "opencv_features2d",
        "-l",
        "opencv_highgui",
        "-l",
        "opencv_videoio",
        "-l",
        "opencv_imgcodecs",
        "-l",
        "opencv_video",
        "-l",
        "opencv_photo",
        "-l",
        "opencv_imgproc",
        "-l",
        "opencv_flann",
        "-l",
        "opencv_core",
        "-g"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "problemMatcher": [
        "$gcc"
      ]
    }
  ],
  "version": "2.0.0"
}
```

其中，`command`使用`clang++`, `args`添加了opencv库的路径，可以在终端中输入`pkg-config opencv --libs —-cflags`，来依次添加参数，如我的是这样的：
![Alt text](/img/opencv_path.jpg)

### 4.3 settings.json修改

```js
{
    "code-runner.executorMap": {
        "c": "cd $dir && make && ./$fileNameWithoutExt && make clean",
        "cpp": "cd $dir && make && ./$fileNameWithoutExt && make clean",
    },
    "code-runner.runInTerminal": true,
    "code-runner.enableAppInsights": false,
    "[makefile]": {
        "editor.insertSpaces": true
    },
    "C_Cpp.default.includePath": [
        "/usr/local/include"
    ]
}
```

重要的是`code-runner.executorMap`中的修改，使用了`make`

### 4.4 创建Makefile

```make
TARGET = ./main

SRCS := $(wildcard ./src/*.cpp ./*.cpp)

OBJS := $(patsubst %cpp,%o,$(SRCS))

CFLG = -g -Wall -I/usr/local/include -Iinc -I./ -std=c++11

LDFG = -Wl, $(shell pkg-config opencv --cflags --libs)

CXX = clang++

$(TARGET) : $(OBJS)
	$(CXX) -o $(TARGET) $(OBJS) $(LDFG)


%.o:%.cpp
	$(CXX) $(CFLG) -c $< -o $@ 

.PHONY : clean
clean:
	-rm ./*.o
```

Makefile的语法请自行了解。

## 5. 总结

> 本来是可以使用visual studio的，但由于我个人比较喜欢轻量的编辑器，而且vscode优秀的extensions，让我想在vscode中开发c/c++项目，但这搭建的过程可真是让我煞费苦心，参考了很多资料，但都不详细，还要一个一个实验这些配置文件都是用来做什么的，在最后配置opencv的时候还要纠结是用brew安装还是编译压缩包，都是填完一坑又出新的坑...不过也算皇天不负有心人，最后是完整地搭建好了环境...希望我的这篇环境搭建对使用这有帮助...

## 6. 存在的问题

> 调试的时候还不能进行多文件编译

## 7. 参考

### 7.1 opencv压缩包编译链接

https://www.jianshu.com/p/a36d41241ae8

### 7.2 相关配置文件 部分介绍

http://blog.biochen.com/archives/858

### 7.3 vscode中c/c++执行和调试单个程序

https://blog.csdn.net/qq_22073849/article/details/88895786

### 7.4 vscode搭建需要引入头文件和链接库的中型或大型项目

https://blog.csdn.net/qq_22073849/article/details/88893201
