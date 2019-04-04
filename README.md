# macOS+vscode+opencv环境配置

## 版本说明

+ macOS: 10.14.3  
+ vscode: 1.32.3
+ opencv: 3.4.0

## opencv的安装

我选择的方案是下载压缩包，使用终端进行编译。

1. 安装cmake，`brew install cmake`

2. 终端进入解压之后的文件夹位置，`cd /Users/$(your_user_name)/opencv`， 然后创建一个新的文件夹，`mkdir build`， 文件夹命名为build

3. 终端进入文件夹build，`cd build`

4. cmake编译，`cmake -G "Unix Makefiles" ..`

5. 然后，`make -j8`

6. 最后，`sudo make install`  

opencv的库文件会出现在`build/lib`中，并且在`/usr/local/include`应该会出现opencv和opencv2两个文件夹，opencv也算安装完成。

## vscode的c/c++执行以及调试环境搭建（非opencv）

文件目录如下：  
![Alt text](/img/directory.jpg)  
创建4个文件，分别为`c_cpp_properties.json`、`launch.json`、`tasks.json`和`setting.json`，接下来就是这个四个文件的内容，（Tips: 在这些文件中，将鼠标悬浮在属性上，会出现说明，比如你将鼠标悬浮在`c_cpp_properties.json`中的`compilePath`上会出现`Full path of the compile being used`文字说明）

### c_cpp_properties.json

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

### launch.json

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

### tasks.json

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

### setting.json

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
