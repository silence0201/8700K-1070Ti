
# Install Tensorflow 1.8 On Hackintosh

Tensorflow自1.2版本后停止了对Mac版的支持,因此我们只能通过源码编译的方式来安装

> 如果有支持CUDA的显卡,还可以通过GPU加速,本文以Tensorflow-GPU版本为基础

## 环境

> 相关驱动和和编译环境要对应好,否则会编译失败

- Mac OS 10.13.6 这个不是很重要
- GPU 387.10.10.10.40.108
- CUDA9.2,CUDA驱动,确认支持上述GPU驱动
- cuDNN 7.2,与CUDA对应
- **Xcode8.2.1**,需要cc版本 
- **bazel 0.14.0**,请降级使用该版本
- **python**,最新版Python3.7存在问题

## 相关地址
1. 显卡是否支持GPU计算:[链接](https://developer.nvidia.com/cuda-gpus)
2. Mac版GPU驱动:[链接](http://www.macvidcards.com/drivers.html)
3. CUDA驱动信息:[链接](https://www.nvidia.cn/object/mac-driver-archive-cn.html)
4. CUDA ToolKit:[链接](https://developer.nvidia.com/cuda-toolkit)
5. cuDNN:[链接](https://developer.nvidia.com/rdp/cudnn-download)

## 相关资源

1. Xcode 8.2.1:[下载地址](https://developer.apple.com/download/more/Xcode_8.2.1.xip)
	
	```
	要使用xcode8.2.1的Command Line Tools,使用cc(clang)命令验证,否则编译会报错
	
	$ cc -v
	Apple LLVM version 8.0.0 (clang-800.0.42.1)
	Target: x86_64-apple-darwin17.7.0
	Thread model: posix
	InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
	
	```
2. bazel-0.14.0:[下载地址](https://github.com/bazelbuild/bazel/releases/download/0.14.0/bazel-0.14.0-installer-darwin-x86_64.sh)
3. CUDA Toolkit 9.2:[下载地址](https://developer.nvidia.com/cuda-toolkit-archive)
4. cuDNN 7.2:[下载地址](https://developer.nvidia.com/rdp/cudnn-download)
5. python3.6.5: 请使用你熟悉的方式降级

## CUDA配置

#### 第一步,确认是否支持你的显卡型号,[显卡是否支持GPU计算](https://developer.nvidia.com/cuda-gpus)

| GPU               | Compute Capability |
| --------------    | ----------------   |
| GeForce GTX 1070Ti| 6.1                |

#### 第二步,卸载旧版CUDA

如果安装老的版本的CUDA,需先卸载

```
$ sudo /usr/local/bin/uninstall_cuda_drv.pl
$ sudo /usr/local/cuda/bin/uninstall_cuda_x.x.pl
$ sudo rm -rf /Developer/NVIDIA/CUDA-x.x/
$ sudo rm -rf /Library/Frameworks/CUDA.framework
$ sudo rm -rf /usr/local/cuda/
```

### 第三步,安装CUDA

注: CUDA Driver与GPU Driver的版本必须对应,才能让CUDA找到显卡

Mac版GPU驱动: 387.10.10.10.40.108 [链接](http://www.macvidcards.com/drivers.html)  
CUDA驱动信息: 396.148 [链接](https://www.nvidia.cn/object/mac-driver-archive-cn.html)  
CUDA ToolKit:v9.2 [链接](https://developer.nvidia.com/cuda-toolkit)  

### 第四部,配置环境变量

配置环境变量编辑 ~/.bash_profile或 ~/.zshrc

```
export CUDA_HOME=/usr/local/cuda
export DYLD_LIBRARY_PATH="$CUDA_HOME/lib:$CUDA_HOME/extras/CUPTI/lib:$CUDA_HOME/bin"
export LD_LIBRARY_PATH=$DYLD_LIBRARY_PATH
export PATH=$DYLD_LIBRARY_PATH:$PATH
```

### 第五步,检查环境

完成安装并配置好后,验证安装情况:

```
$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Tue_Jun_12_23:08:12_CDT_2018
Cuda compilation tools, release 9.2, V9.2.148
```

确认CUDA驱动是否加载

```
$ kextstat | grep -i cuda.
132    0 0xffffff7f819a9000 0x2000     0x2000     com.nvidia.CUDA (1.1.0) E13478CB-B251-3C0A-86E9-A6B56F528FE8 <4 1>
```

### 第六步:安装cuDNN

[cuDNN](https://developer.nvidia.com/rdp/cudnn-download),下载合适的版本

下载好后解压缩合并到CUDA的目录下即可,并添加相关权限:

```
$ sudo cp cuda/include/cudnn.h /usr/local/cuda/include
$ sudo cp cuda/lib/libcudnn* /usr/local/cuda/lib
$ sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib/libcudnn*
$ rm -rf cuda
```

## 编译

下载 Tensorflow GPU 版本源码

### CUDA

参考上一节配置

```
$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Tue_Jun_12_23:08:12_CDT_2018
Cuda compilation tools, release 9.2, V9.2.148
```

### python

版本限制:3.6.5  

```
$ python3 --version
Python 3.6.5
```

安装依赖:six numpy wheel

```
$ pip3 install six numpy wheel
```

### Clang

确认版本:

```
$ cc -v
Apple LLVM version 8.0.0 (clang-800.0.42.1)
Target: x86_64-apple-darwin17.7.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```

一些工具:
```
$ brew install coreutils llvm cliutils/apple/libomp
```

### Bazel

安装0.14.0

```
$ curl -O https://github.com/bazelbuild/bazel/releases/download/0.14.0/bazel-0.14.0-installer-darwin-x86_64.sh
$ chmod +x bazel-0.14.0-installer-darwin-x86_64.sh
$ ./bazel-0.14.0-installer-darwin-x86_64.sh
$ bazel version
Build label: 0.14.0
```

### Build

执行配置:

```
$ ./configure
```

配置相关信息:

```
Please specify the location of python. [Default is /usr/local/opt/python@2/bin/python2.7]: /usr/local/bin/python3

Found possible Python library paths:
  /usr/local/Cellar/python/3.6.5_1/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages
Please input the desired Python library path to use.  Default is [/usr/local/Cellar/python/3.6.5_1/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages]

Do you wish to build TensorFlow with Google Cloud Platform support? [Y/n]: n
No Google Cloud Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Hadoop File System support? [Y/n]: n
No Hadoop File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Amazon S3 File System support? [Y/n]: n
No Amazon S3 File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Apache Kafka Platform support? [Y/n]: n
No Apache Kafka Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with XLA JIT support? [y/N]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with GDR support? [y/N]: n
No GDR support will be enabled for TensorFlow.

Do you wish to build TensorFlow with VERBS support? [y/N]: n
No VERBS support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: y
CUDA support will be enabled for TensorFlow.

Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 9.0]: 9.2

Please specify the location where CUDA 9.1 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:

Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7.0]: 7.2

Please specify the location where cuDNN 7 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:

Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 3.5,5.2]3.0,3.5,5.0,5.2,6.0,6.1

Do you want to use clang as CUDA compiler? [y/N]:n
nvcc will be used as CUDA compiler.

Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]:

Do you wish to build TensorFlow with MPI support? [y/N]:
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:

Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]:
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See tools/bazel.rc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
Configuration finished
```

> 一定要输入正确的版本
>
> * /usr/local/bin/python3
> * CUDA 9.2
> * cuDNN 7.2
> * compute capability 3.0,3.5,5.0,5.2,6.0,6.1 这个要去查你的显卡支持的版本，可以输入多个

生成了编译配置文件 .tf_configure.bazelrc

开始编译:
```
$ bazel clean --expunge
$ bazel build --config=opt --cxxopt="-D_GLIBCXX_USE_CXX11_ABI=0" --action_env PATH --action_env DYLD_LIBRARY_PATH //tensorflow/tools/pip_package:build_pip_package
```

### 生成PIP安装包

重编译并且替换ncclops.so:

```
$ gcc -march=native -c -fPIC tensorflow/contrib/nccl/kernels/nccl_ops.cc -o _nccl_ops.o
$ gcc _nccl_ops.o -shared -o _nccl_ops.so
$ mv _nccl_ops.so bazel-out/darwin-py3-opt/bin/tensorflow/contrib/nccl/python/ops
$ rm _nccl_ops.o
```

打包:

```
$ bazel-bin/tensorflow/tools/pip_package/build_pip_package ~/Downloads/
```

清理:

```
$ bazel clean --expunge
```

### 安装

```
$ pip3 uninstall tensorflow
$ pip3 install tensorflow-1.8.0-cp36-cp36m-macosx_10_13_x86_64.wh
```
 