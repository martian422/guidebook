# 从入门到跑路：服务器使用指南

本教程所有操作请在**你自己的**拥有管理员权限的账户和home目录下进行，**请勿使用ubuntu，root账户**。

目前本教程包括CUDA和CONDA的安装示例。-2023/6/9

新增了连接离线服务器的办法。-2023/6/20

## 1.如何正确安装CUDA？(以11.7为例)

### #1 下载.runfile

直接必应搜索你想下载的版本是最快的（例：“CUDA 11.7"，第一个就是）

***注意系统版本号！！！***

运行网页中给出的wget命令即可将安装文件下载到服务器本地。

<img src="./pictures/download.png" style="zoom:50%;" />

### #2 运行.runfile

```bash
#此步修改文件权限，有时可忽略，为了保险就加上了。
#版本号与截图中略有差异，以下载版本为准（输入./cuda按tab补全）
chmod 777 cuda_11.7.1_515.65.01_linux.run 
sudo ./cuda_11.7.1_515.65.01_linux.run
```

安装过程中有几处建议勾选/去勾选，具体请参考图片

<img src="./pictures/step1.png" alt="step1" style="zoom: 50%;" />

<img src="./pictures/step2.png" alt="step2" style="zoom:50%;" />

<img src="./pictures/step3.png" alt="step3" style="zoom:50%;" />

<img src="./pictures/step4.png" alt="step4" style="zoom:50%;" />

**这一步选no，否则会覆盖系统的CUDA！**

<img src="./pictures/step5.png" alt="step5" style="zoom:50%;" />

之后运行下列代码

```bash
echo -e "PATH=\"/usr/local/cuda-11.7/bin:\$PATH\"\nLD_LIBRARY_PATH=\"/usr/local/cuda-11.7/lib64:\$LD_LIBRARY_PATH\"" >> ~/.bashrc
# 更新bashrc
source ~/.bashrc 
nvcc -V 
# 应当会显示11.7，此时你的账户使用的CUDA版本就是这个了。
```

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
ssh -p PORT_NUM username@X.Y.Y.A 
```

