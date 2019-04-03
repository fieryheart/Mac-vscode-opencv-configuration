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

## vscode的c/c++执行以及调试环境搭建

文件目录如下：  
![Alt text](/img/directory.jpg)  
创建4个文件，分别为`c_cpp_properties.json`、`launch.json`、`setting.json`和`tasks.json`