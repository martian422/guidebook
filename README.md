# 从入门到跑路：服务器使用指南

本教程所有操作请在**你自己的**拥有管理员权限的账户和home目录下进行，**请勿使用ubuntu，root账户**。

目前本教程包括CUDA和CONDA的安装示例。-2023/6/9

新增了连接离线服务器的办法。-2023/6/20

新增了升级CUDA的办法。 -2023/8/31

## 1.如何正确安装CUDA？(以11.7为例)

### #0 查看现有CUDA及CUDA Toolkit版本

使用nvidia-smi命令可查看当前的系统CUDA版本。安装CUDA驱动时最好不要超过该版本。

**倘若存在越级，请参考#3。**

使用nvcc -V命令可查看当前CUDA编译驱动版本（一般来说要安装或者改变的就是这个）。

如果nvcc -V显示的版本与你想要的不符，建议首先使用

```bash
ls /usr/local
```

查看该目录下有无你想要的cuda-x.x文件夹，如果有，直接使用

```bash
#11.7换成你想使用且该目录下存在的版本
echo -e "PATH=\"/usr/local/cuda-11.7/bin:\$PATH\"\nLD_LIBRARY_PATH=\"/usr/local/cuda-11.7/lib64:\$LD_LIBRARY_PATH\"" >> ~/.bashrc

# 更新bashrc
source ~/.bashrc 

# 应当会显示你需要的CUDA版本了。
nvcc -V 
```

如果没有，再根据以下步骤进行操作。

### #1 下载.runfile

直接必应搜索你想下载的版本是最快的（例：“CUDA 11.7"，第一个就是）

***注意ubuntu系统版本号！使用lsb_release -a命令查询。***

运行网页中给出的wget命令即可将安装文件下载到服务器本地。

<img src="./pictures/download.png" style="zoom:50%;" />

### #2 运行.runfile

```bash
#第一步提权一般可忽略，是因为可能该文件之前已存在，所有者不是你。
#版本号与截图中略有差异，以下载版本为准。
chmod 777 cuda_11.7.1_515.65.01_linux.run
#如果服务器在nvcc时没有显示任何版本，可能它处于准裸机状态，连gcc都没有。
#这种情况下直接sh可能会报错然后浪费时间。
#为了避免这种情况，（对于未安装nvcc的机器）您应该首先执行如下命令：
sudo apt-get update
sudo apt-get install gcc
#此时遇到restart问题请全部取消勾选，不影响（常见于ubuntu2202版本）。
sudo sh ./cuda_11.7.1_515.65.01_linux.run
```

安装过程中有几处建议勾选/去勾选，具体请参考图片

<img src="./pictures/step1.png" alt="step1" style="zoom: 50%;" />

<img src="./pictures/step2.png" alt="step2" style="zoom:50%;" />

<img src="./pictures/step3.png" alt="step3" style="zoom:50%;" />

<img src="./pictures/step4.png" alt="step4" style="zoom:50%;" />

**对于nvcc有显示的这一步务必选no，否则会覆盖系统的CUDA！（11.7以下低版本中选择‘upgrade’那一项）**

<img src="./pictures/step5.png" alt="step5" style="zoom:50%;" />

之后运行下列代码

```bash
echo -e "PATH=\"/usr/local/cuda-11.7/bin:\$PATH\"\nLD_LIBRARY_PATH=\"/usr/local/cuda-11.7/lib64:\$LD_LIBRARY_PATH\"" >> ~/.bashrc

# 更新bashrc
source ~/.bashrc 

# 应当会显示你需要的CUDA版本了。
nvcc -V 
```

### #3 升级CUDA版本

倘若你要使用的项目需求的CUDA版本超过了当前nvidia-smi显示的版本，用echo强行指定高版本驱动可能出现问题。因为linux升级驱动可能（往往）需要删除之前的驱动，此时你应当**首先咨询服务器管理员**解决办法，得到许可后再进行升级操作。操作过程中建议仅使用命令行，不开启图形界面。

```bash
#此处以11.7版本为例
#删除已经安装的nvidia包
sudo apt-get purge nvidia-*

#增加下载nvidia驱动的源
sudo add-apt-repository ppa:graphics-drivers/ppa 

#这一步不一定能让你直接安装成功驱动，但能获取并配置好你安装驱动所需要的工具链（最后的数字与你要安装的驱动的版本号一致）
sudo apt-get install nvidia-driver-515

#重启，之后试试nvidia-smi，大概率它无法正常显示，继续安装
sudo reboot 

#参考1、2把runfile下载到本地并且安装

#####
#注意：与#2中不同，此时安装一定要勾选Driver（建议全部勾选），并且在选择是否update synlink时选择yes。
#####
sudo sh cuda_11.7.0_515.43.04_linux.run

#使用0中的办法更新你的CUDA路径，之后nvidia-smi应该可以显示并且为你想要的版本了。
```

### #4 事后

依赖原CUDA编译和安装的pytorch相关库可能会在运行时出一些迷惑问题，建议在更换版本/更新驱动后重新配置你的conda环境（指新建一个环境从头装起）以避免奇怪的bug。



## 2.如何安装conda？(以miniconda为例)

远程服务器一般不需要conda的完整版本，按需下载和更新，使用miniconda即可。

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-py39_23.3.1-0-Linux-x86_64.sh
# 其他版本可以在https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/查找。
chmod 777 ./Miniconda3-py39_23.3.1-0-Linux-x86_64.sh
bash ./Miniconda3-py39_23.3.1-0-Linux-x86_64.sh
source ~/.bashrc
# 接下来新建你需要的虚拟环境。记得指定环境名称NAME和python版本
conda create -n NAME python=3.8 -y && conda activate NAME
```

此时你的终端左侧应该就会出现标识当前环境的*(NAME)*了。



## 3.如何连接离线服务器？

当一台服务器掉电或迁移位置之后，可能会由于掉盘，IP变化等原因无法进行远程连接，此时我们需要携带一台有网线接口的笔记本电脑前往机房，使用网线将笔记本电脑与服务器相连后进行操作。

注：此处默认服务器ssh服务正常运作，如若异常，可使用

```bash
sudo systemctl restart sshd.service
```

重启ssh服务。此外，此处默认服务器使用静态IP，如果使用动态IP(DHCP)，步骤1中可能无法通过输入其之前IP来连接（Z可能发生变化）。

#### 1.ipv4配置。

为了让笔记本能与服务器进行通信，需要对网络进行配置。

找到“更改适配器选项”，点击配置对应以太网的属性。

<img src="./pictures/network config0.png" alt="step5" style="zoom:80%;" />

设服务器断电前能正常远程访问的IP地址为X.Y.Z.A，在此处则应该将IP地址设置为X.Y.Z.B，B可为任意1~255之间的整数（推荐设置为1）,且B≠A，以保证你的笔记本与服务器处于同一局域网内。

子网掩码默认设置为255.255.255.0即可。

<img src="./pictures/network config.png" alt="step5" style="zoom:60%;" />

#### 2.使用ssh连接服务器。

在cmd或powershell中使用惯用的ssh命令连接即可：

```bash
ssh -p PORT_NUM username@X.Y.Z.A 
```
