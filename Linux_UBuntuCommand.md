### UBuntu Commands

> *$ gnome-tweaks*                      打开乌邦图的gnemo桌面的tweaks(优化)工具
> 
> *$sudo apt update*                        更新包管理系统apt的信息，装卸软件都必须的指令

> - 删除缓存
> 1. s非常有用的清理命令： 
>    sudo apt-get autoclean                清理旧版本的软件缓存 
>    sudo apt-get clean                    清理所有软件缓存 
>    sudo apt-get autoremove             删除系统不再使用的孤立软件 
>    这三个命令主要清理升级缓存以及无用包的

-----------------------------

> - Linux系统中安装.deb软件包(以百度网盘举例)：
>   
>   1. ```shell
>      $ sudo dpkg -i baidunetdisk_linux_2.0.1.deb
>      $ sudo apt-get -f install
>      ```

> - 2. `$ sudo dpkg -i <file_name>`            用来处理安装编译包文件
>   3. `$ sudo apt-get -f install `            修复依赖包,系统上有某个package不满足依赖条件，这个命令就会自动修复，安装程序包所依赖的包.使用命令“ aptitude -f install ”，实现相同的效果 .

> #### 在Linux中运行.sh文件（Shell脚本文件）:
> 
> 1. 确保.sh文件具有执行权限：在终端中，使用ls-l命令查看文件权限。如果.sh文件没有执行权限，使用`chmod +x <filename.sh>`命令为文件添加执行权限
> 
> 2. 目录下执行 ./filename.sh

------------------------------

> ##### LINUX下tar.gz包的安装方法
> 
> 源码大多以tar.gz 和tar.bz2打包软件:
> 
> - *解压targz文件*
>   
>   ```shell
>   tar -zxvf file.tar.gz -C path/..../....
>   ```
>   
>   > 这里的参数含义为：
>   > 
>   > -z：表示使用gzip进行压缩和解压缩；
>   > 
>   > -x：表示解压缩；
>   > 
>   > -v：表示显示详细信息；
>   > 
>   > -f：表示指定要解压缩的文件。
>   > 
>   > -C: 表示解压所后的目标路径

-------

- [x] 补充：rar和zip格式的解压缩：

- 解压缩.rar格式的压缩文件

假设我们有一个名为bar.rar的压缩文件，我们想要把它解压缩到/tmp/bar目录中，可以使用以下命令：

```shell
$ rar x bar.rar /tmp/bar 
```

以上命令中的x选项表示对bar.rar文件进行解压缩，并将解压缩后的文件放到指定的目录中，这里指定为/tmp/bar目录。

- 解压缩.zip格式的压缩文件

假设我们有一个名为baz.zip的压缩文件，我们想要把它解压缩到/tmp/baz目录中，可以使用以下命令：

```shell
$ unzip baz.zip -d /tmp/baz
```

----------

##### **关于配置环境出错**：

```shell
$ source ./bashrc
报错：
.bashrc:16: command not found: shopt
....
```

>  查看系统默认shell:     `echo $SHEL`  发现为 `zsh `不是`bash` ,故而报错

- 因此，配置环境变量与资源更新都因该为：

- ```shell
  $ vim ./.zshrc
  $ source ./.zshrc
  ```

----

UBuntu中使用apt安装了很多包，太多记不住指定包是否安装过，怎么办？

```shell
apt-cache search package 搜索包

eg:$ sudo apt-cache search package jdk 
openjdk-21-dbg - Java runtime based on OpenJDK (debugging symbols)
openjdk-21-doc - OpenJDK Development Kit (JDK) documentation
openjdk-21-jre-zero - Alternative JVM for OpenJDK, using Zero
openjdk-21-source - OpenJDK Development Kit (JDK) source files
```

--------------

- [x] ### 重点：

##### Linux制作桌面快捷方式的创建

1. 法一：在`/usr/share/applications`这个目录下可以找到很多已经安装的软件的.desktop文件，把它们复制粘贴到桌面即可

2. 法二：如果在`/usr/share/applications`下没有该软件的快捷方式（找不到对应的.desktop文件），也可以手动创建并编辑一个：

> ```shell
> cd /usr/share/applications    ##必须是这个目录才有用
> sudo vim Steam++.desktop   ##创建Stem++.desktop文件并进入vim编辑界面
> ```
> 
> > 点击i进入编辑模式，在vim编辑界面中输入:
> > 
> >     [Desktop Entry]    (这行必须存在，笔者亲测把它注释掉后就会提示Broken Desktop File，即已损坏的桌面文件。）
> >     Name=steam++    (该快捷方式的名称）
> >     Comment=steam++ shortcut    （对该文件的注释）
> >     Exec=~/APPlication/Stem++/Steam++.sh   (执行文件的绝对路径，这里注意后缀.sh必须要写上，不然的话会报和之前一样的错误）
> >     Type=Application   （类型，应用）
> >     Terminal=false       （这个字段表示是否在执行时打开终端）
> >     Icon= ~/APPlication/Steam++/Icons/Watt-Toolkit.png（指定应用图标文件）

> > 然后按esc键退出编辑，再输入:wq退出，在桌面上就会出现这个快捷方式，如果点击Allow Launching后没反应（图标还是灰色，带红色叉号）或者弹出错误窗口，一般是该文件没有可执行权限造成的。给它添加可执行权限：
> > 
> > ```shell
> > $ sudo chmod u+x Steam++.desktop #文件名
> > ```
> 
> - 最后一步：复制`/usr/share/applications` 的`Steam++.desktop`粘贴到桌面，再然后允许运行即可.

----------

- [x] ##### 卸载与清理各类软件包

自动检测清理无用的安装包——新安装往往会提示

```shell
$ sudo apt autoremove
```

手动卸载无用的安装包

```shell
$ sudo apt-get remove <package_name>
```
