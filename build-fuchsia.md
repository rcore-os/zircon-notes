
## from
https://fuchsia-china.com/guide-of-fuchsia-os-building/

## mirror addr
https://fuchsia-china.com/fuchsia-source-code-china-mirror/

## pre requirement
```
#ubuntu
sudo apt-get install build-essential curl git python unzip
```

## 获取源代码

```
curl -s "https://fuchsia.googlesource.com/fuchsia/+/master/scripts/bootstrap?format=TEXT" | base64 --decode | bash
```

执行上面的命令后,终端窗口黄色提示消息让你把 .jiri_root/bin 路径添加到变量,以便使用Fuchsia的工具链(jiri fx什么的)，可在终端输入如下命令：

export PATH="/home/flomen/fuchsia/.jiri_root/bin:$PATH"

或者直接将 PATH="/home/flomen/fuchsia/.jiri_root/bin:$PATH" 添加到.bashrc

此时在你的home目录就有个fuchsia文件夹，cd 进入fuchsia文件夹，可使用 jiri update 更新源码。


## 编译Fuchsia
这时需要用到fx命令，以前是 fx set x64 或者 fx set arm64 。现在源码有了些变化，需要使用：
```
fx set [PRODUCT].[BOARD]
```

而这个[PRODUCT]输入 fx list-products 查看候选项，[BOARD]输入 fx list-boards 查看候选项。


如下：
```
fx set workstation.x64 # x64 调试版
fx set core.arm64 # arm64 调试版
fx set core.x64 --release # x64 正式版
```
默认 fx set core.x64 

等待一会儿，此时会在~/fuchsia/out目录生成一些必要文件，用于后续编译构建。

接下来使用
```
fx build
```
开始构建你的Fuchsia！！
正在编译系统文件


## 运行Fuchsia
QEMU不支持Vulkan,因此无法运行Fuchsia的图形堆栈。所以没有UI。
此时输入
```
fx run -g
```

```
-m 用MB设置QEMU内存大小。
-g 启用图形界面。
-N 启用网络。
-k 启用KVM加速。
```
然而这时弹出了



## 分析源码

fuchsia的源码使用clion IDE 进行查看。可以在fuchsia根目录运行
```
fx compdb
```
生成compile_commands.json文件。启动clion.sh，选择fuchsia根目录导入项目，然后经过符号解析过程后（比较慢），就可以使用clion来查看代码了。

## 常见错误1：cipd auth-login
使用下载脚本下载到一半时，有些朋友会出现如下报错：

Some packages are skipped by cipd due to lack of access, you might want to run "cipd auth-login" and try again
ERROR: context deadline exceeded

解决方法是：

在浏览器中打开下方网页：
https://accounts.google.com/o/oauth2/auth?access_type=offline&approval_prompt=force&client_id=446450136466-2hr92jrq8e6i4tnsa56b52vacp7t3936.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email

登入 Google 账号，自动获得一个授权码，下图示例

在命令行中输入
cipd auth-login
这时命令行会弹出：
Authorization code:
贴上从第2步获取的授权码即可。

进入 fuchsia 源码目录，输入
jiri update
更新源代码，问题解决。



## 运行编译好的hello_world
https://www.robotshell.com/2019/10/14/os/HelloWorld/

### 去掉中文显示
```
export LANGUAGE="en_US.UTF-8"
export LANG="en_US.UTF-8"
export LC_ALL="en_US.UTF-8"
```
### 编译
```
fx set ... --with //examples/hello_world
fx build
fx serve
```
### 起到虚拟机
在另外一个终端
```
sudo ip tuntap add dev qemu mode tap user chyyuu && sudo ip link set qemu up
fx run -N -u scripts/start-dhcp-server.sh  -g
```
### 运行helloworld
在另外一个终端开始运行编译好的hello_world
```
$ fx shell run fuchsia-pkg://fuchsia.com/hello_world_cpp#meta/hello_world_cpp.cmx

WARNING: Please opt in or out of fx metrics collection.
You will receive this warning until an option is selected.
To check what data we collect, run `fx metrics`
To opt in or out, run `fx metrics <enable|disable>

Hello, world!

$ fx shell run fuchsia-pkg://fuchsia.com/hello_world_rust#meta/hello_world_rust.cmx

WARNING: Please opt in or out of fx metrics collection.
You will receive this warning until an option is selected.
To check what data we collect, run `fx metrics`
To opt in or out, run `fx metrics <enable|disable>

Hello, world!


```

## 常见错误2：fx run
```
fx run -g
....
Could not extend fvm, unable to stat fvm image
```

这样的错误。解决方法：
用文本编辑器打开~/fuchsia/tools/devshell/lib/fvm.sh，将

stat_output=$(stat "${stat_flags[@]}" "${fvmraw}") 改为
stat_output=$(LC_ALL=C stat "${stat_flags[@]}" "${fvmraw}")

这是似乎是因为系统语言导致的相关问题。在

size="${BASH_REMATCH[1]}" 的后面下一行
echo $size

然后保存，此时再 fx run -g 就能运行了。


## 常见错误3：fx run -N -u scripts/start-dhcp-server.sh  -g
且
``` 
fx shell run fuchsia-pkg://fuchsia.com/hello_world_cpp#meta/hello_world_cpp.cmx
```
方法
```
取消中文输出
export LANGUAGE="en_US.UTF-8"
export LANG="en_US.UTF-8"
export LC_ALL="en_US.UTF-8"
设置虚拟网络
add file: /etc/network/interfaces.d/qemu.conf
内容为：iface qemu inet manual
sudo service network-manager restart
sudo ip tuntap add dev qemu mode tap user chyyuu && sudo ip link set qemu up

然后再执行
fx shell run fuchsia-pkg://fuchsia.com/hello_world_cpp#meta/hello_world_cpp.cmx
就可以了
```
