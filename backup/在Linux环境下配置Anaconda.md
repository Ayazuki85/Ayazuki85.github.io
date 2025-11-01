# 1.软件下载

[[Anaconda下载地址](https://repo.anaconda.com/archive/index.html)](https://repo.anaconda.com/archive/index.html "https://repo.anaconda.com/archive/index.html")

<img width="1466" height="1239" alt="Image" src="https://github.com/user-attachments/assets/2df07d75-8c21-4a73-9016-bb83f97961bf" />

按照自己的Linux架构按需下载，不知道自己Linux架构的可以在终端输入 `uname -m` 查询。

<img width="749" height="215" alt="Image" src="https://github.com/user-attachments/assets/f6a47328-9649-4796-8ed7-6256a7a1631f" />

我这里下载`Anaconda3-2025.06-1-Linux-x86_64.sh`

将下载后的文件放置在Linux里，我这里使用的是WSL。如何使用WSL可以参考[[超详细的WSL教程：Windows上的Linux子系统](https://www.bilibili.com/video/BV1tW42197za/?vd_source=40bb5e54b66eaedaf85bb6bb9ba32dad)](https://www.bilibili.com/video/BV1tW42197za/?vd_source=40bb5e54b66eaedaf85bb6bb9ba32dad)

文件下载好后我选择将文件放置在/home/下

<img width="1176" height="558" alt="Image" src="https://github.com/user-attachments/assets/c0facad5-e3aa-4a0c-90c3-40d7ef16b66c" />

# 2.安装Anaconda

使用`bash Anaconda3-2025.06-1-Linux-x86_64.sh`运行该文件

依次回车，输入yes，回车。

<img width="891" height="762" alt="Image" src="https://github.com/user-attachments/assets/b215735f-7796-461d-af76-be2b21c2df12" />

解压后会问一个问题，说人话就是你如果希望在启动Linux终端时自动激活conda的base环境就输入yes，不想这么做就输入no，这里我选择输入no。

<img width="1200" height="540" alt="Image" src="https://github.com/user-attachments/assets/9971398e-1a2b-45e7-9920-1832b0ddaa0b" />

至此，Anaconda安装完成。

# 3.配置环境变量

此时在终端输入`conda`会提示`conda: command not found`，这是因为conda还没有被配置到环境变量中。如果出现`Could not find command-not-found database. Run 'sudo apt update' to populate it.`说明你安装Linux后还没有进行过包索引更新。依次输入`sudo apt update` `sudo apt upgrade` 同步本地软件包索引与远程仓库的最新信息，升级系统中所有已安装的软件包到新版本。

接下来进行配置环境变量，输入`source ~/anaconda3/etc/profile.d/conda.sh`，再输入`conda --version`,如果终端返回conda的版本号，说明成功将 conda 的路径添加到环境变量中。

# 4.基本命令示例

**创建一个名为 "myenv" 的新环境:**

`conda create --name myenv`

**创建指定版本的环境**：

`conda create --name myenv python=3.8`

以上代码创建一个名为 "myenv" 的新环境，并指定 Python 版本为 3.8。

**激活环境：**

`conda activate myenv`

以上代码激活名为 "myenv" 的环境。

**要退出当前环境使用以下命令：**

`conda deactivate`

**查看所有环境：**

`conda env list`

以上代码查看所有已创建的环境。

**复制环境：**

`conda create --name myclone --clone myenv`

以上代码通过克隆已有环境创建新环境。

**删除环境：**

`conda env remove --name myenv`

以上代码删除名为 "myenv" 的环境。

**安装包：**

`conda install package_name`

以上代码安装名为 "package_name" 的软件包。

**安装指定版本的包：**

`conda install package_name=1.2.3`

以上代码安装 "package_name" 的指定版本。

**更新包：**

`conda update package_name`

以上代码更新已安装的软件包。

**卸载包：**

`conda remove package_name`

以上代码卸载已安装的软件包。

**查看已安装的包：**

`conda list`

查看当前环境下已安装的所有软件包及其版本。

**查看帮助：**

`conda --help`

以上代码获取 conda 命令的帮助信息。

**查看 conda 版本：**

`conda --version`

以上代码查看安装的 conda 版本。

**搜索包：**

`conda search package_name`

以上代码在 conda 仓库中搜索指定的软件包。

**清理不再需要的包：**

`conda clean --all`

以上代码清理 conda 缓存，删除不再需要的软件包。

# 参考资料

[[超详细的linux-conda环境安装教程CSDN博客](https://blog.csdn.net/Alex_81D/article/details/135692506)](https://blog.csdn.net/Alex_81D/article/details/135692506)

[[Linux安装Conda教程](https://0xdadream.github.io/2025/04/10/linux-an-zhuang-conda-jiao-cheng/)](https://0xdadream.github.io/2025/04/10/linux-an-zhuang-conda-jiao-cheng/)

[[Anaconda教程](https://www.runoob.com/python-qt/anaconda-tutorial.html)](https://www.runoob.com/python-qt/anaconda-tutorial.html)