# Azure新创建的虚拟机如何修改时间
在Azure上新创建虚拟机默认是UTC时区的，跟国内时区不符，需要修改为CST <br>
注：UTC:Coordinated Universal Time 协调世界时，是最主要的世界时间标准<br>
    CST:China Standard Time 北京时间<br>
## 方法1：
Azure官方提供方法：https://docs.azure.cn/zh-cn/articles/azure-operations-guide/virtual-machines/aog-virtual-machines-howto-disable-time-sync<br>
备注：操作到最后一步时一直报echo write error: no device,echo写错误,没有这个设备,建议采用第二种方法
## 方法2：
直接使用shanghai时区文件覆盖	<br>
        
        [root@test ~]# cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
        [root@test ~]# hwclock
        [root@test ~]# reboot

# Ubuntu 16.04 安装 Docker
1.选择国内的云服务商，这里选择阿里云为例<br>

    curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
2.安装所需要的包<br>

    sudo apt-get install linux-image-extra-$(uname -r) linux-image-extra-virtual
3.添加使用 HTTPS 传输的软件包以及 CA 证书

    sudo apt-get update
    sudo apt-get install apt-transport-https ca-certificates
4.添加GPG密钥
    
    sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
5.添加软件源
    
    echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
6.添加成功后更新软件包缓存

    sudo apt-get update
7.安装docker
    
    sudo apt-get install docker-engine
8.启动 docker

    sudo systemctl enable docker         #设置为开机启动项
    sudo systemctl start docker
9.更改docker镜像源为阿里云
    
    sudo vi /etc/docker/daemon.json      #编辑docker配置文件、没有就新建一个
  添加如下内容：
    
    {
       "registry-mirrors": ["https://y0qd3iq.mirror.aliyuncs.com"]
    }

    
# Ubuntu 16.04 安装 Nvidia-Docker
    wget -P /tmp https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb
    sudo dpkg -i /tmp/nvidia-docker*.deb && rm /tmp/nvidia-docker*.deb

# 利用反向代理实现从外网连接到内网服务器：
1、	服务器情况描述：<br>
服务器	IP	用户名	备注<br>
内网服务器A	192.168.3.109	Hp_admin	目标服务器、处于内网<br>
Azure虚拟器B	139.217.233.190	hongpu	外网服务器，相当于跳板<br>
2、	介绍一下使用到的ssh参数:<br>
反向代理：ssh -fCNR<br>
正向代理：ssh -fCNL<br>
-f 后台执行ssh指令<br>
-C 允许压缩数据<br>
-N 不执行远程指令<br>
-R 将远程主机(服务器)的某个端口转发到本地端指定机器的指定端口<br>
-L 将本地机(客户机)的某个端口转发到远端指定机器的指定端口<br>
-p 指定远程主机的端口<br>
3、首先在A服务器上操作，建立A到B的反向代理<br>
ssh -p 22456 -fCNR 22457:localhost:22 hongpu@139.217.233.190<br>
4、	接着在B服务器上操作，建立B服务器的正向代理<br>
ssh -fCNL *:22456:localhost:22457 localhost<br>

# Ubuntu 下根据端口查进程pid 杀进程
1、根据端口查pid进程<br>
sudo lsof -i:端口号  <br>
2、杀进程<br>
sudo kill PID号<br>

# Ubuntu下安装显卡驱动
1、安装相关依赖

    sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler 
    sudo apt-get install --no-install-recommends libboost-all-dev 
    sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev 
    sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev

2、禁用 nouveau
    
    echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf build the new kernel by:

3、查询Nvidia驱动<br>
   先查询显卡型号<br>

    lspic；lspci |grep VGA；lspci |grep 3D

然后去官网找到对应的驱动下载<br>
 https://www.nvidia.cn/Download/index.aspx?lang=cn
 
  ![查看图片](https://raw.githubusercontent.com/hongpu-corp/dev_ops/dev/Picture/Findnvidia.png)

4、	开始安装<br>

    sudo sh NVIDIA-Linux-x86_64-xxx.run
可以通过以下命令确认驱动是否正确安装：<br>
    
    cat /proc/driver/nvidia/version
    nvidia-smi

# Azure云VM虚拟机无法连接解决方案
1， 在此虚拟机的同一资源组中创建同一区域的ubuntu16虚拟机；<br>
2， 删除当前无法连接的虚拟机，平台会自动保留您的托管磁盘；<br>
3， 将保留的系统磁盘附加到新创建的虚拟机上，并在新虚拟机上进行mount；<br>
4， 修改挂载磁盘中的相关home目录名称，并注意文件夹权限。修改后与该虚拟机接触附加状态；<br>
5， 通过portal中的磁盘页面通过托管磁盘直接创建新的vm。

# Azure虚拟机附加数据磁盘
1、虚拟机-选择需要附加数据磁盘的虚拟机-磁盘-添加数据磁盘-创建磁盘<br>
2、进入虚拟机查看<br>
       
       ls /dev/sd*
	   fdisk -l
3、对附加的磁盘进行分区（使用此方法分区磁盘最大容量只能为2T）<br>

    fdisk /dev/sdc
    n  #创建分区
    p  #创建主分区
    1  #创建1个分区
    #分区开始扇区
    #分区结束扇区
    p  #再次检查磁盘分区情况
    wq  #保存退出
 
4、格式化分区<br>

    sudo mkfs -t ext4 /dev/sdc
5、创建挂载点进行挂载<br>
    
    sudo mkdir /data
    sudo mount /dev/sdc1 /data
6、设置开机自动挂载<br>
    
    sudo -i blkid                  #查看磁盘UUID
    sudo vi /etc/fstab
添加以下内容：<br>
UUID=aa8988be-6fc2-46a9-8f9c-e240b6b57d88    /data     ext4    defaults             0 0	

# Ubuntu使用iptables开放端口
1、	查看系统是否安装防火墙<br>
    
    sudo whereis iptables
2、查看防火墙的配置信息<br>
	
    sudo iptables -L
3、新建规则文件（如果没有）<br>
	
    mkdir /etc/iptables 
	vi /etc/iptables/rules.v4
添加以下内容：<br>

    *filter
    :INPUT DROP [0:0]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [0:0]
    :syn-flood - [0:0]
    -A INPUT -i lo -j ACCEPT
    -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
    -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
    -A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
    -A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
    -A INPUT -p icmp -m limit --limit 100/sec --limit-burst 100 -j ACCEPT
    -A INPUT -p icmp -m limit --limit 1/s --limit-burst 10 -j ACCEPT
    -A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -j syn-flood
    -A INPUT -j REJECT --reject-with icmp-host-prohibited
    -A syn-flood -p tcp -m limit --limit 3/sec --limit-burst 6 -j RETURN
    -A syn-flood -j REJECT --reject-with icmp-port-unreachable
    COMMIT
4、使防火墙生效<br>
    
    iptables-restore < /etc/iptables/rules.v4

5、创建文件、<br>
    
    vi /etc/network/if-pre-up.d/iptables
   添加以下内容、使防火墙开机启动<br>
    
    #!/bin/bash
    iptables-restore < /etc/iptables/rules.v4

6、添加执行权限<br>
    
    chmod +x /etc/network/if-pre-up.d/iptables
7、查看规则是否生效<br>
    
    iptables -L -n  

# ModuleNotFoundError: No module named 'apt_pkg'
   升级到python3.6会导致python库的引用产生混乱<br>
   解决方法:<br>
1、先选择删除python-apt<br>
	
	apt-get remove --purge python-apt 
2、安装python-apt<br>
	
	apt-get install -f -y python-apt
3、拷贝python3.5的apt-pkg*.so 名重名为python3.6的apt-pkg*.so<br>
	
	cd /usr/lib/python3/dist-packages/
	cp apt_pkg.cpython-35m-x86_64-linux-gnu.so apt_pkg.cpython-36m-x86_64-linux-gnu.so
# ModuleNotFoundError: No module named 'cryptography.hazmat.bindings._padding'
   解决方法：<br>
	
	pip uninstall pyopenssl
	pip uninstall cryptography
	pip install pyopenssl
	pip install cryptography

# Ubuntu使用TinyProxy搭建代理服务器
1、安装

	sudo apt-get update
	sudo apt-get install tinyproxy
2、配置
	
	sudo vi /etc/tinyproxy.conf
   修改如下地方：<br>
   Port 8888 					#预设是8888 Port,你可以更改<br>
   Allow 127.0.0.1 				#将127.0.0.1改成你自己的IP<br>
   #例如你的IP 是1.2.3.4,你改成Allow 1.2.3.4,那只有你才可以连上这个Proxy<br>
   #若你想任何IP都可以连到Proxy就在Allow前面打#注释<br>
3、重启和设置开机默认启动

	sudo service tinyproxy restart
	sudo service tinyproxy enable
4、连接测试<br>
   在另一台机器上输入:<br>
	
	curl -x <IP>:<PORT> www.baidu.com

# 查看ubuntu服务器网速

	git clone https://github.com/sivel/speedtest-cli.git
	cd speedtest-sli/	
	./speedtest_cli.py

# 查看系统版本
        
        cat /etc/lsb-release

# apt-get update报错
   报错如下：

    public key is not available
    GPG error: https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64
   解决方法1：
    
    apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 3B4FE6ACC0B21F32
   解决方法2：更换更新源<br>

    sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
    sudo vim /etc/apt/sources.list
   替换为以下内容<br>
    
    deb http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
    deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
    deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
    deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
    deb http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
    deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial main restricted universe multiverse
    deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
    deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
    deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
    deb-src http://mirrors.ustc.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
   解决方法3：
    
    curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey | sudo apt-key add -

# apt NVIDIA命令相关

    sudo apt-get purge nvidia* 			# 卸载低版本驱动
    sudo add-apt-repository ppa:graphics-drivers	# 把显卡加入PPA
    sudo apt-get update		
    sudo add-apt-repository ppa:bumblebee/stable
    sudo apt-get update
    sudo apt-get install bumblebee bumblebee-nvidia
    sudo apt-cache search nvidia    		# 使用apt搜寻可以安装的nvidia驱动
    ubuntu-drivers devices			# 查看Ubuntu推荐的驱动版本
    sudo apt-get install nvidia-390 nvidia-settings nvidia-prime 	# 使用apt命令在终端安装
    nvidia-setting			# 在弹出的界面中选择独显与集显
    sudo prime-select nvidia 			# 切换nvidia显卡
    sudo prime-select intel  			# 切换intel显卡
    sudo prime-select query  			# 查看当前使用的显卡
    sudo apt-mark hold nvidia-390		# 停止当前版本驱动的本地更新

# python3.6 下提示No module named "apt_pkg"的解决方法

    sudo apt install python3-apt
    cd /usr/lib/python3/dist-packages
    sudo cp apt_pkg.cpython-35m-x86_64-linux-gnu.so apt_pkg.cpython-36m-x86_64-linux-gnu.so
    
# 服务器安装opencv-python,并解决cv2依赖环境
    pip install opencv-python
安装了opencv-python  之后，在使用 import cv2  报错如下<br>
    
    Python 3.6.5 |Anaconda, Inc.| (default, Apr 29 2018, 16:14:56) 
    [GCC 7.2.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import cv2
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
      File "/root/anaconda3/lib/python3.6/site-packages/cv2/__init__.py", line 3, in <module>
        from .cv2 import *
    ImportError: libSM.so.6: cannot open shared object file: No such file or directory
    >>>
报错原因：缺少共享库<br>
Centos解决方案<br>
    
    yum whatprovides libSM.so.6
    yum install libSM-1.2.2-2.el7.x86_64 --setopt=protected_multilib=false
Ubuntu解决方案
    
    sudo apt-get install libsm6
    sudo apt-get install libxrender1
    sudo apt-get install libxext-dev

# docker_mysql配置步骤

    docker run -it --name mysql8 \
    -p 3306:3306 \
    -v /home/hongpu/mysql/data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=123456\
    -d mysql:8
起来docker后，如遇到不能远程连接的情况请按如下方法操作:

    docker exec -it mysql8 bash                    #进入docker
    mysql -uroot -p123456                          #进入mysql，注意-u后面直接接用户名、-p后面直接输入密码，两者之间不要有空格
<br>

    mysql>alter user 'root'@'%' identified by '123456' password expire never;
    mysql>alter user 'root'@'%' identified with mysql_native_password by '123456';
    mysql>flush privileges
