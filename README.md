# Linux-Monitor

**分布式Linux性能分析监控**

* **docker模块：dockerfile**指定相应的**cmake，grpc，proto等源码和依赖**，构建整个项目环境，易于在多台服务器上部署环境，并**编写容器操作的脚本指令**，易于启动操作项目所依赖的环境。

* **monitor模块：**采用**工厂方法**，通过构造moniotr的抽象类定义接口，**实现相应的CPU状态，系统负载，软中断，mem，net监控**，易于为之后扩展更多系统监控；并为了模拟出真实的性能问题，使用**stress工具进行模拟压测**，分析相应时刻服务器的cpu状况和中断状况。

* 通过**grpc框架**，构建出相应的server，client；**server在所要监控的服务器部署**，**client生成库供monitor模块和display模块调用**，并考虑为了降低耦合性，项目每个模块相互独立，可拆解，只通过调用grpc服务来进行远程连接。

* 使用**protobuf序列化协议**，构建出整个项目的数据结构。

* **display模块**分为两大部分：**ui界面的构造，datamodel构造**；
  * ui界面使用QWidget、QTableView、QStackedLayout、QPushButton等进行构建
  * datamodel：通过继承QAbstractTableModel，构建出相应的cpu_model、softirq_model、mem_model等，**每3秒刷新一次数据**。

## 框架

![](https://cdn.jsdelivr.net/gh/clannadbing/Image-Hosting@main/20240131/1.png)

## Demo演示

![](https://cdn.jsdelivr.net/gh/clannadbing/Image-Hosting@main/20240131/2.gif)

## 环境要求

* Ubuntu18.04
* 30-40G硬盘大小
* C++11

## 快速开始

### 1. 安装docker

```
sudo apt install curl
curl -fsSL https://test.docker.com -o test-docker.sh
sudo sh test-docker.sh
```

### 2. docker加入用户组-o

```
sudo groupadd docker
sudo usermod -aG docker ${USER}
sudo systemctl restart docker
newgrp docker
docker ps
```

### 3. 下载Linux版本QT

[链接](https://download.qt.io/archive/qt/5.12/5.12.9/qt-opensource-linux-x64-5.12.9.run)

```
将下载好的文件存放在~/Linux-Monitor/docker/build/install/qt/目录下
```

​                                                            ![](https://cdn.jsdelivr.net/gh/clannadbing/Image-Hosting@main/20240131/3.png)

### 4.  通过项目中dockerfile文件，构建项目镜像

* 构建镜像

  ```
  cd ~/Linux-Monitor/docker/build
  docker build --network host -f base.dockerfile .
  ```

​       **等待时间会比较长，镜像大小大约4G.**

* 查看镜像id

  ```
  docker images
  ```

​		                                                               ![](https://cdn.jsdelivr.net/gh/clannadbing/Image-Hosting@main/20240131/4.png)

* 命名镜像id为linux:monitor

  ```
  docker tag f866f4a3e61b linux:monitor
  ```

  ​                                                               ![](https://cdn.jsdelivr.net/gh/clannadbing/Image-Hosting@main/20240131/5.png)

### 5.  进入docker容器

```
cd ~/Linux-Monitor/docker/scripts
#启动容器
./monitor_docker_run.sh 
#进入容器
./monitor_docker_into.sh
```

### 6. 编译代码

```
cd work
cd cmake
cmake ..
make -j8
```

### 7. 运行

```
cd rpc_manager/server //注意：在work/cmake/rpc_manager/server目录下
./server
```

**新建终端**

```
cd ~/Linux-Monitor/docker/scripts
./monitor_docker_into.sh
cd work/cmake/test_monitor/src
./monitor
```

**新建终端**

```
cd ~/Linux-Monitor/docker/scripts
./monitor_docker_into.sh
cd work/cmake/display_monitor
./display
```

## 致谢

感谢以下朋友的PR和帮助: [@superxiaobai-1](https://github.com/superxiaobai-1)
