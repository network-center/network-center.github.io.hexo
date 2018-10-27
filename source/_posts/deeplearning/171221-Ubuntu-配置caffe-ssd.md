---
title: Ubuntu 配置caffe ssd
date: 2017-12-21
tags:
articleid: 20171221
---

caffe 是一个比较有名的深度学习框架，在视频和图像方面应用比较广泛，因为NVIDIA主要是做GPU和视觉，所以caffe和NIVEA关系比较亲密。使用的GPU驱动也是NVIDIA驱动和NVIDIA cuda8.0。

## 安装cuda 8.0

### 下载cuda 8.0和补丁

浏览器访问 https://developer.nvidia.com/cuda-80-ga2-download-archive 下载Base Installer和Patch 2

![](/images/20171221/CUDA_Toolkit_8_0.png)

### 安装cuda 所需要的库

```bash
sudo apt-get install freeglut3-dev build-essential libx11-dev libxmu-dev libxi-dev libgl1-mesa-glx libglu1-mesa
libglu1-mesa-dev libxi-dev libxmu-dev -y
```

### 安装cuda 8.0

```bash
$ sudo ./cuda_8.0.61_375.26_linux-run

Do you accept the previously read EULA?
accept/decline/quit: accept #接受

Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 375.26?
(y)es/(n)o/(q)uit: n #不安装默认驱动

Install the CUDA 8.0 Toolkit?
(y)es/(n)o/(q)uit: y #确认安装

Enter Toolkit Location
 [ default is /usr/local/cuda-8.0 ]: #回车默认

 Do you want to install a symbolic link at /usr/local/cuda?
(y)es/(n)o/(q)uit:y #创建连接

Install the CUDA 8.0 Samples?
(y)es/(n)o/(q)uit:y

Enter CUDA Samples Location
 [ default is /home/ubuntu ]:y

Installing the CUDA Toolkit in /usr/local/cuda-8.0 ...
```

### 安装补丁

```bash
$ sudo ./cuda_8.0.61.2_linux-run
Do you accept the previously read EULA?
accept/decline/quit: accept

Enter CUDA Toolkit installation directory
 [ default is /usr/local/cuda-8.0 ]:

Installation complete!
Installation directory: /usr/local/cuda-8.0
```

## 安装caffe所需要的依赖

### 安装protobuf、BLAS、Boost、hdf5等
```bash
$ sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
$ sudo apt-get install --no-install-recommends libboost-all-dev
$ sudo apt-get install libatlas-base-dev
$ sudo apt-get install the python-dev
```
## 安装OpenCV 3.0 (https://github.com/opencv/opencv/releases/tag/3.0.0)

### 安装依赖

```bash
$ sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
$ sudo apt-get install build-essential
$ sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
```

### 下载OpenCV

```bash
$ wget -c https://github.com/opencv/opencv/archive/3.0.0.tar.gz
$ tar -zxvf 3.0.0.tar.gz
$ cd opencv-3.0.0/
$ mkdir build
$ cd build
$ sudo cmake .. ## 如果没有安装cmake，使用 apt install cmake 进行安装
$ sudo make all
```

## 安装cuDNN

### 安装拓展

```bash
$ sudo apt install snappy libleveldb-dev liblmdb-dev -y
```

### 下载cudnn类库

<<<<<<< HEAD
![](/images/20171221/cudnn.png)
=======
![](/images/cudnn.png)
>>>>>>> 094473c98ed0f50d498e3e29a19ae068959031b6

### 解压安装

```bash
$ tar xvf cudnn-8.0-linux-x64-v7.tar
## 系统将解压内容放到了cuda文件夹里面了
cuda/include/cudnn.h
cuda/NVIDIA_SLA_cuDNN_Support.txt
cuda/lib64/libcudnn.so
cuda/lib64/libcudnn.so.7
cuda/lib64/libcudnn.so.7.0.4
cuda/lib64/libcudnn_static.a

## 拷贝类库
$ sudo cp cuda/lib64/lib* /usr/local/cuda/lib64/ 
## 拷贝.h文件
$ sudo cp cuda/include/cudnn.h /usr/local/cuda/include/ 

#更新软连接
$ cd /usr/local/cuda/lib64/ 
$ sudo rm -rf libcudnn.so libcudnn.so.6.5 
$ sudo ln -s libcudnn.so.6.5.48 libcudnn.so.6.5 
$ sudo ln -s libcudnn.so.6.5 libcudnn.so 
```

## 安装NVIDIA驱动(可选)

这次没有遇到驱动版本问题。但是之前也遇到过一个坑，笔记本自带的驱动不兼容，后来安装了nvdia-375.66结束

## 编译caffe-ssd

$ git clone https://github.com/weiliu89/caffe.git    
$ cd caffe    
$ git checkout ssd
$ mkdir build
$ sudo cmake ..     
$ sudo make -j8    
$ sudo make runtest -j8
$ sudo make install

## 运行作者样例

在caffe/models下新建一个文件夹VGGNet
下载预训练模型 链接：http://pan.baidu.com/s/1miDE9h2 密码：0hf2，将其中的`VGG_ILSVRC_16_layers_fc_reduced.caffemodel` 放入`caffe/models/VGGNet/`目录下
下载VOC2007和VOC2012数据集，放到 `~/data`下。下载数据集并解压
```bash
$ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar   
$ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtrainval_06-Nov-2007.tar   
$ wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtest_06-Nov-2007.tar
$ tar -xvf VOCtrainval_11-May-2012.tar   
$ tar -xvf VOCtrainval_06-Nov-2007.tar    
$ tar -xvf VOCtest_06-Nov-2007.tar
```

## 将图片转化为LMDB文件

```bash
$ ./data/VOC0712/create_list.sh     
$ ./data/VOC0712/create_data.sh
```

这里用的脚本实现批处理，可能会出现:no module named caffe等错误，这是由于caffe的Python环境变量未配置好，可按照下面方法解决：
```bash
$ echo "export PYTHONPATH=/home/$USER/caffe/python" >> ~/.profile    
$ source ~/.profile
$ echo $PYTHONPATH #检查环境变量的值
```

## 训练模型

在下载的caffe根目录执行如下命令训练，在examples/ssd下存在几个.py文件，训练的时间较长.

```bash
$ python examples/ssd/ssd_pascal.py
```
## 在摄像头上测试

```bash
$ python examples/ssd/ssd_pascal_webcam.py
```
## 错误处理

#### fatal error: boost/shared_ptr.hpp: 没有那个文件或目录

    ```bash
    $ sudo apt install libboost1.58* -y
    ```
#### fatal error: gflags/gflags.h: 没有那个文件或目录

    ```bash
    $ sudo apt install libgflags-dev -y
    ```
#### fatal error: glog/logging.h: 没有那个文件或目录

    ```bash
    sudo apt install libgoogle-glog-dev -y
    ```
#### Check failed: error == cudaSuccess (10 vs. 0)  invalid device ordinal

    更改examples/ssd/ssd_pascal.py，把gpus = "0,1,2,3" 改为gpus = "0"

#### heck failed: error == cudaSuccess (2 vs. 0)  out of memory

    更改examples/ssd/ssd_pascal.py，把 batch_size = 32 改为batch_size = 16 或依次改小

#### 搭建caffe环境时“error: hdf5.h”找不到的解决方法

    所以相应的需要更改"Makefile.config"文件中的包含目录
    INCLUDE_DIRS:=$(PYTHON_INCLUDE) /usr/local/include
    然后在后面加上"serial"的包含目录，即：
    INCLUDE_DIRS:=$(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial/
    接着需要更改相应的"Makefile"文件，找到
    LIBRARIES +=glog gflags protobuf boost_system boost_filesystem m hdf5_hl hdf5
    更改最后两项为：
    LIBRARIES +=glog gflags protobuf boost_system boost_filesystem m hdf5_serial_hl hdf5_serial
    就可以了，继续make了。

#### 如果出现 `‘NppiGraphcutState’ has not been declared`是因为cuda和OpenCV的版本兼容问题

    找到 `$CAFFE/modules/cudalegacy/src/graphcuts.cpp` 然后进行修改

    ```cpp
    # if !defined (HAVE_CUDA) || defined (CUDA_DISABLER) 

    改为

    # if !defined (HAVE_CUDA) || defined (CUDA_DISABLER) || (CUDART_VERSION >= 8000)
    ```

    然后继续编译安装
    
    ```bash
    $ sudo make
    $ sudo make install
    ```

feilong
2017.12.21
