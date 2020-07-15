# DarwinML 模型自动设计平台安装培训



[TOC]


<h2 align = "center">变更历史</h2>
| **日期时间** | **版本** | **编辑人** |        **内容说明**  |
| :----------: | :------: | :--------: | :------------------------: |
|  2019-11-27  |   0.1    |    何兵    |            初稿            |
|  2019-12-19  |   0.2    |    何兵    | 增加对/文件系统的空间要求 |
|  2020-04-20  |   0.3    |    何兵    |  修改为2.0的部署步骤  |
|  2020-06-02  |   0.4    |    何兵    |  支持darwin2.0 GPU的版本  |



## [**Chapter-1 文档说明**][#Chapter-1 文档说明]
　　本文档描述了如何利用Linux容器镜像在客户环境搭建DarwinML模型自动设计平台软件（DarwinML Studio）及简单场景的使用。



## [**Chapter-2 环境要求**][#Chapter-2 环境要求]
　　客户需要提供安装了Linux环境和运行docker的组件，环境要求：

| **项目** | **内容** |
| :---------: | :--------: |
| 硬件(GPU)    | X86 Intel® 处理器，8core以上 |
| 内存	|64G或以上 |
|Linux OS	|Ubuntu 16.04/18.04、Centos 7|
|内存	|64G或以上|
|GPU	|Nvidia GPU，默认4块卡，如果不是4块，需要在配置中修改|
| 磁盘	| 用户文件系统50G或以上，用于存放软件、日志、数据等资料/文件系统空闲30G或以上（镜像解压后12G左右）|
|可访问端口|	Host环境提供3个可以对外访问的端口，例如9030 ~ 9034|
|浏览器	|客户端浏览器支持Firefox, Chrome, IE|
|用户权限|	安装用户需要有管理容器的权限|

在上述 Hos t环境中需要安装运行 docker 的组件，详细步骤可以参考：
Ubuntu OS:
<https://docs.docker.com/install/linux/docker-ce/ubuntu/#upgrade-docker-ce>
Centos OS:
<https://docs.docker.com/install/linux/docker-ce/centos/#os-requirements>

下载DarwinML Studio Image到 Host 环境中，文件名为：studio2_gpu_v2.tar.gz



## [**Chapter-3 安装步骤**][#Chapter-3 安装步骤]


### **3.1 导入镜像**
| **命令** | **备注** |
| :---------: | :--------: |
| gunzip studio2_gpu_v2.tar.gz|	解压压缩文件|
|docker load -i studio2_gpu_v2.tar|将镜像文件导入到容器环境中|
|docker images|	查看导入镜像列表|



导入镜像示例：

```shell
darwin@rack01:~$ sudo docker load -i /rack02_space/workspace/darwin/conda_envs/ studio2_gpu_v2.tar
e5269eb13663: Loading layer [==================================================>]    628MB/628MB
Loaded image: studio2_gpu_v2:20200604
```

查看导入镜像示例：
```shell
darwin@rack01:~$ sudo docker images
REPOSITORY	TAG	IMAGE ID	CREATED 	SIZE
studio2_gpu_v2	20200602  486993806323	4 minutes ago	11.4GB
```



### **3.2 启动镜像**
| **命令** | **备注** |
| :---------: | :--------: |
|sudo nvidia-docker run -d \ <br>-e NVIDIA_VISIBLE_DEVICES=0,1,2,3 \ <br/>-p 9030:22\ -p 9031-9034:3001-3004 \ <br/>-v /sys/fs/cgroup:/sys/fs/cgroup \ <br/>--name=studio2_demo \ <br/>--hostname=studio2_gpu \ <br/>studio2_gpu_v2:20200604 \ <br/>注意1：因为这是测试环境，先不要修改hostname的值，因为修改的话，需要修改部分配置文件，如果确实需要，可以咨询探智立方；容器的名字name是可以修改的<br/>注意2：容器内部的端口（3001-3004）建议不要修改 <br/>注意3：Host环境的对应端口，可以根据实际环境进行调整| 容器内的端口使用：<br/>NVIDIA_VISIBLE_DEVICES 指定GPU的No.22    ssh访问 <br/>3001  DarwinML Studio Portal (前端)<br/>3002  NGINX反向代理端口 <br/>3003  Celery Flower监控端口 <br/>在Host环境中，需要有4个端口与上述4个端口对应，例如在本例中，对应的端口分别为：9030～9033，剩余的1个端口备用；<br/>其中9030是从Host ssh到container中<br/>其他3个用来通过浏览器访问DarwinML服务 |
|ssh -p 9030 darwin@localhost| 通过ssh登录到容器环境中<br/>默认密码是：abcd1234 |



### **3.3 生成hostkey，申请license文件**
|  | **操作** |
| :---------: | :--------: |
| 1	|以 darwin 用户登录|
|2	|运行 source platform_env|
|3	|运行 python cmd/cli.py get_host_key<br/>会得到一个 host key 和主机名的字符串，例如 <br/> ["55917890141cb1c89accc5e53a929acb1867c42c", "studio2_gpu"]|
|4	|将该字符串通过公司邮箱发到 support@iqubic.net 邮箱中，我们会为每个客户生成相应的 license 文件，并回复邮件|
|5	|收到 license 文件后，将文件传到/home/darwin/studio/darwin-platform/conf 目录中，文件名为 license.dat|

注 1：如果用 image 新创建一个 container，每个 container 环境中的 host key 不同，需要单独申请
注 2：如果一个 container 关闭并重启，host key 有可能会变化（因为容器获得的IP地址有可能会有变化），这样的话当前license文件就会失效，服务就会启动不了， 需要重新申请；或者在启动容器的时候，指定IP地址（--ip string）



### **3.4. 修改DarwinML GUI服务的IP/端口**
|   **需要修改的文件及修改方法** |
| :--------- |
| 3 <font color=#008000 >DEFAULT_HOST_IP</font>=202.112.23.252     			    #根据实际环境进行调整<br>4 <font color=#008000 >DEFAULT_PORTAL_LISTEN_PORT</font>=3001<br>5 <font color=#008000 >DEFAULT_HOST_PORTAL_LISTEN_PORT</font>=9031   #根据实际环境进行调整<br>7 <font color=#008000 >DEFAULT_WSGI_LISTEN_PORT</font>=3002<br>8 <font color=#008000 >DEFAULT_HOST_WSGI_LISTEN_PORT</font>=9032  	  #根据实际环境进行调整 |
|<font color=#008000 >DEFAULT_HOST_IP</font> 是Host环境的IP地址，用户通过该地址来访问DarwinML的WEB服务<br><font color=#008000 >DEFAULT_HOST_PORTAL_LISTEN_PORT</font> 是Host环境的端口，对应到容器中的3001端口<br><font color=#008000 >DEFAULT_HOST_WSGI_LISTEN_PORT</font> 是Host环境的端口，对应到容器中的3002端口<br>用户根据实际的环境进行调整|

注：如果GPU/CPU数量与默认值不同，需要修改conf目录下的 slurm.conf.template.$PROFILE 文件，这一步骤后续再补



### **3.5. 启动后台服务**
| **命令** | **备注** |
| :---------: | :--------: |
|Login with darwin 用户	|	|
|source platform_env| |
|sudo ls|需要输入darwin用户的密码，默认是abcd1234|
|bash cmd/setup.sh cmd=setup_cgroup|激活darwin用户的cgroup功能，出现<br>+----------------------------------------------------------- Setup memory cgroup<br>+-----------------------------------------------------------<br>表示运行成功|
|sh start.sh|启动过程无需人工干预|


```shell
(darwinml_gpu2) [darwin@studio2_gpu darwin-platform]$ sh start.sh
[N]: Using production build at "/home/darwin/studio/darwin-platform"
[N]: Source profile "testcluster" from /home/darwin/studio/darwin-platform/conf/profile.testcluster successfully
[N]: Using conda virtual environment at "darwinml_gpu2"
Starting redis[  OK  ]
Starting flower
Starting mongodb2020-06-04T08:15:25.386+0800 I CONTROL  [main] Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'
about to fork child process, waiting until server is ready for connections.
forked process: 2002
child process started successfully, parent exiting
[  OK  ]
Starting fsm_manager
Starting darwin_wsgi
Starting slurmctld[  OK  ]
Starting slurmproxy
Starting file_downloader
Starting resource_collector
[W]: Celery queue "gpu" was routed to slurm and will NOT take "start" action to the corresponding celery worker
Starting worker@defaultcelery multi v4.4.2 (cliffs)
> Starting nodes...
        > default@studio2_gpu: OK
[  OK  ]
[W]: Celery queue "gpu" was routed to slurm and will NOT take "start" action to the corresponding celery worker
Starting worker@longcelery multi v4.4.2 (cliffs)
> Starting nodes...
        > long@studio2_gpu: OK
[  OK  ]
[W]: Celery queue "long_gpu" was routed to slurm and will NOT take "start" action to the corresponding celery worker
[W]: Celery queue "long_gpu" was routed to slurm and will NOT take "start" action to the corresponding celery worker
Starting worker@prioritycelery multi v4.4.2 (cliffs)
> Starting nodes...
        > priority@studio2_gpu: OK
[  OK  ]
Starting worker@tensorboardcelery multi v4.4.2 (cliffs)
> Starting nodes...
        > tensorboard@studio2_gpu: OK
[  OK  ]
Starting slurmd[  OK  ]
```



### **3.6. 启动GUI服务**
| **命令** | **备注** |
| :---------: | :--------: |
|Login with darwin 用户	| |
|cd /home/darwin/gui/dist/install<br>bash gui_start.sh| 启动的时候会有交互式提示，是读取了 /home/darwin/gui/dist/install/conf/install.config 中配置的内容，每项直接回车即可。|

启动DawinML Studio后台服务example:
```shell
(darwinml_gpu2) [darwin@studio2_gpu install]$ bash gui_start.sh
/home/darwin/gui/dist/install
Please check used nginx settings: /etc/nginx. If not, please set the variable SYSTEM_NGINX_CONFIG before installation.
Current common nginx settings: /etc/nginx
请输入nginx的目录(/home/darwin/.local/ui/nginx): 
(此处输入回车）
/home/darwin/.local/ui/nginx
请输入GUI所在的目录(/home/darwin/.local/ui/webapp): 
(此处输入回车）
/home/darwin/.local/ui/webapp
请输入配置GUI访问的ip(202.112.23.252): 
(此处输入回车）
202.112.23.252
请输入配置GUI访问的端口号(3001): 
(此处输入回车）
3001
如果是容器，请输入GUI访问的端口号对应的主机端口号(9031): 
(此处输入回车）
9031
请输入配置GUI下载的共享目录全路径(/home/darwin/.share/gui_integration): 
(此处输入回车）
/home/darwin/.share/gui_integration
请输入后端反向代理的端口号(3002): 
(此处输入回车）
3002
如果是容器，请输入后端反向代理端口号对应的主机端口号(9032): 
(此处输入回车）
9032
请输入配置后端的内网IP(127.0.0.1): 
(此处输入回车）
127.0.0.1
请输入配置后端的内网PORT(5034): 
(此处输入回车）
5034
是否需要配置认证服务器(N)[Y/N]: 
(此处输入回车）
Current user: darwin
Existed nginx master process: 16371
Stop the nginx master process: 16371
Existed nginx worker process: 16372 16373 16374 16375 16376 16377 16379 16380 16381 16382
Stop the nginx worker process: 16372 16373 16374 16375 16376 16377 16379 16380 16381 16382
Clean and create webapps folder /home/darwin/.local/ui/webapp
 Start nginx as a service...
nginx: [alert] could not open error log file: open() "/usr/local/openresty/nginx/logs/error.log" failed (13: Permission denied)
2020/06/04 08:39:37 [warn] 18342#18342: the "user" directive makes sense only if the master process runs with super-user privileges, ignored in /home/darwin/.local/ui/nginx/nginx.conf:1
Nginx service is running... Nginx worker number is 10
Please try to access following link:
http://202.112.23.252:9031
```



### **3.7. 访问Web服务**
　　如果服务启动成功，可以通过浏览器访问DarwinML Studio Web界面，例如(<http://202.112.23.252:9031>)，如果出现下面的界面，输入用户名（当前安装版本没有安装鉴权管理模块，输入任意字符串都可以）：
![DarwinLogin.png](/Users/chenmeng/Desktop/DarwinML/DarwinML文档/图片_DarwinML模型自动设计平台安装培训/DarwinLogin.png)


因为是试用环境，未启用鉴权功能，直接点击**“登录”**，会看到首页信息，就说明安装成功。![DarwinHome.png](/Users/chenmeng/Desktop/DarwinML/DarwinML文档/图片_DarwinML模型自动设计平台安装培训/DarwinHome.png)

<center>（图-登录首页）</center>

注：因为测试环境未启用鉴权，未能获取用户计算资源的使用情况，所以看到都是空的信息。

点击左侧导航栏项目管理，可以看到有3个项目，这是预置在系统里的3个项目，并对应3个数据集：![ProjectList.png](/Users/chenmeng/Desktop/DarwinML/DarwinML文档/图片_DarwinML模型自动设计平台安装培训/ProjectList.png)

<center>（图-项目列表）</center>



## [**Chapter-4 注意事项**][#Chapter-4 注意事项]


### **4.1. 空间问题**

　　默认安装后，DarwinML软件运行时的日志文件会存储到$HOME/.local目录下，与模型相关的文件会存储到$HOME/.share目录下，如果创建的container能用到的存储空间小，在运行一段时间之后，就会出现空间不足的情况。
　　可以在创建Container的时候，可以分给container更多的空间，至少20G以上；或者在container内部创建NFS Client，从host上共享一个目录给container，将这两个目录移到NFS中，并在$HOME/下创建两个软连接就行。



### **4.2. 装载镜像文件遇到空间不够的问题**

​		在用docker load命令装载镜像文件的时候，有时候会遇到空间不够的错误提示，如下图所示：
![DockerError.png](/Users/chenmeng/Desktop/DarwinML/DarwinML文档/图片_DarwinML模型自动设计平台安装培训/DockerError.png)

<center>（图-镜像文件装载过程中的报错信息）</center>

　　实际上，根文件系统的空间是足够的，这个错误原因是由于当前docker的版本比较低，默认的容器根分区大小是10G，镜像文件超过了10G，所以会产生上述的问题，解决办法就是修改docker的默认参数，可以参考下面链接中的方法：
<https://blog.csdn.net/weixin_42847874/article/details/83748736>



### **4.3. license过期问题**
如果License文件过期，需要申请新的license文件，需要将该文件拷贝到目录：```/home/darwin/studio/darwin-platform/conf ```中，然后需要重新启动服务。



### **FAQ**

安装手册：

```shell
>> docker run -d --name=iqubic_centos_gpu_1 --hostname=iqubic_centos_gpu_1 --env=NVIDIA_VISIBLE_DEVICES=0,1 --volume=/home/darwin:/home/darwin --volume=/workspace:/workspace -v /sys/fs/cgroup:/sys/fs/cgroup  -p 9060:22 -p 9061:9061  -p 9062:9062 -p 9063:9063 -p 9064:9064 -p 9065:9065 -p 9066:9066  fuzhiwen/platform-dev-x86_64:10.1-cudnn7-rh7 bash -c '/usr/sbin/sshd-keygen && /usr/sbin/sshd -D'
```


注意事项：

- 添加limited参数

- share和local文件夹不要太深

- 如果是中文系统，需要在你用的profile.<PROFILE>里加上下面几行：

  ```shell
  export LANG=en_US.utf8
  export LANGUAGE=en_US:en
  export LC_CTYPE="en_US.utf8"
  export LC_ALL=en_US.utf8
  ```

-----------------------------------------------------------------------------------------------------------------------------------------





