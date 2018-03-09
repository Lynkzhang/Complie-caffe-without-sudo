# Complie-caffe-without-sudo
Complie-caffe-without-sudo

caffe依赖手动编译笔记

Dependencies内容:

	手动编译
		Boost
		Google-glog
		Google-gflags
		Google-protobuf
		HDF5
		Snappy
		OpenCV
		Python(Anaconda)

	已有或联系管理员
		Threads
		OpenMP
		LMDB
		LevelDB
		CUDA
		BLAS(设置 MKL,Intel)

	未安装（不影响编译）
		Matlab
		Doxygen


python相关的依赖，下载anaconda一步解决:
```
sh Anaconda2-5.1.0-Linux-x86_64.sh

pip install numpy #一般来说已经是最新的了
```

Boost:
```
wget https://dl.bintray.com/boostorg/release/1.66.0/source/boost_1_66_0.tar.gz
tar zxvf boost_1_66_0.tar.gz
cd boost_1_66_0.tar.gz
./bootstrap.sh --with-libraries=all ##--with-libraries指定编译哪些boost库，all的话就是全部编译，只想编译部分库的话就把库的名称写上，之间用, 号分隔即可。
```

命令执行完后看到如下所示即为成功:
```
Building Boost.Build engine with toolset gcc... tools/build/src/engine/bin.linuxx86_64/b2
Detecting Python version... 2.6
Detecting Python root... /usr
Unicode/ICU support for Boost.Regex?... not found.
Generating Boost.Build configuration in project-config.jam...

Bootstrapping is done. To build, run:

    ./b2

To adjust configuration, edit 'project-config.jam'.
Further information:

   - Command line help:
     ./b2 --help

   - Getting started guide: 
     http://www.boost.org/more/getting_started/unix-variants.html

   - Boost.Build documentation:
     http://www.boost.org/build/doc/html/index.html
```

编译:
```
./b2 #消耗时间很长

...

./b2 install --prefix=/home/Lynkzhang/Boost #没有sudo所以只能安装到自己的home下了
```

参考:http://blog.csdn.net/u011641865/article/details/73498533


CMake(已经是最新版本的可以跳过这一步):

在一些库编译的时候提示版本太老:
```
CMake Error at CMakeLists.txt:1 (cmake_minimum_required):
  CMake 3.0 or higher is required.  You are running version 2.8.12.2
```

cmake版本过旧，自己编译一个cmake 3.0
```
wget https://cmake.org/files/v3.11/cmake-3.11.0-rc2.tar.gz
tar zxvf cmake-3.11.0-rc2.tar.gz
cd cmake-3.11.0-rc2
mkdir build
cd build
../configure --prefix=/home/Lynkzhang/cmake #同理只能装在自己文件夹
make -j
make install
```

之后可以直接使用二进制 /home/Lynkzhang/cmake/bin/cmake 来做cmake


Google-gflags:
```
git clone https://github.com/gflags/gflags.git
cd gflags
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/home/Lynkzhang/Gflags ..  #指定安装位置
make -j
make install
```


Google-glog(需要在gflags之后安装):
```
git clone https://github.com/google/glog.git
cd glog
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/home/Lynkzhang/Glog ..  #指定安装位置
make -j
make install
```


Google-protobuf:
Protobuf不知道为什么没有CMakeList，所以用configure。
```
git clone https://github.com/google/protobuf.git
cd protobuf
mkdir build
cd build
../configure --prefix=/home/Lynkzhang/Protobuf  #指定安装位置
make -j
make install
```


HDF5:
```
tar zxvf hdf5-1.10.1.tar.gz
cd hdf5-1.10.1
mkdir build
cd build
../configure --prefix=/home/Lynkzhang/hdf5  #指定安装位置
make -j
make install
```
下载链接：https://www.hdfgroup.org/downloads/hdf5/source-code/


Snappy:
```
git clone https://github.com/google/snappy.git
cd snappy
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/home/Lynkzhang/Snappy ..  #指定安装位置
make -j
make install
```


OpenCV:
```
git clone https://github.com/opencv/opencv.git
cd opencv
mkdir build
cd build
cmake -DCMAKE_INSTALL_PREFIX=/home/Lynkzhang/openCV ..  #指定安装位置
make -j
make install
```


在编译Dependencies的时候有一个问题，有些编译默认不会出编译出 .so 文件（没有共享库），如果发现没有，就需要添加一个flag:
```
cmake -DBUILD_SHARED_LIBS=ON
```


正式开始编译caffe:

*接下来的内容应该可以有更优雅的解决方式，我直接暴力改了...*

修改 cmake/Dependencies.cmake
```

set(GFLAGS_INCLUDE_DIR "/home/Lynkzhang/Gflags/include/")
set(GFLAGS_LIBRARY "/home/Lynkzhang/Gflags/lib/libgflags.so")

set(GLOG_INCLUDE_DIR "/home/Lynkzhang/Glog/include/")
set(GLOG_LIBRARY "/home/Lynkzhang/Glog/lib64/libglog.so")

set(HDF5_LIBRARIES "/home/Lynkzhang/hdf5/lib/libhdf5.so")
set(HDF5_INCLUDE_DIRS "/home/Lynkzhang/hdf5/include/")
set(HDF5_HL_LIBRARIES "/home/Lynkzhang/hdf5/lib/libhdf5_hl.so")

set(Snappy_INCLUDE_DIR "/home/Lynkzhang/Snappy/include/")
set(Snappy_LIBRARIES "/home/Lynkzhang/Snappy/lib64/libsnappy.so")

set(OpenCV_INCLUDE_DIRS "/home/Lynkzhang/openCV/include/")

set(Boost_INCLUDE_DIRS "/home/Lynkzhang/Boost/include")

...

# ---[ Boost

#find_package(Boost 1.54 REQUIRED COMPONENTS system thrvead filesystem)
list(APPEND Caffe_INCLUDE_DIRS PUBLIC ${Boost_INCLUDE_DIRS})

set(Boost_LIBRARIES "/home/Lynkzhang/Boost/lib/libboost_system.so")
list(APPEND Caffe_LINKER_LIBS PUBLIC ${Boost_LIBRARIES})
set(Boost_LIBRARIES "/home/Lynkzhang/Boost/lib/libboost_thread.so")
list(APPEND Caffe_LINKER_LIBS PUBLIC ${Boost_LIBRARIES})
set(Boost_LIBRARIES "/home/Lynkzhang/Boost/lib/libboost_filesystem.so")
list(APPEND Caffe_LINKER_LIBS PUBLIC ${Boost_LIBRARIES})

...

# ---[ HDF5
#find_package(HDF5 COMPONENTS HL REQUIRED)

...

# ---[ OpenCV:


#find_package(OpenCV QUIET COMPONENTS core highgui imgproc imgcodecs)
#if(NOT OpenCV_FOUND) # if not OpenCV 3.x, then imgcodecs are not found
#  find_package(OpenCV REQUIRED COMPONENTS core highgui imgproc)
#endif()

#  list(APPEND Caffe_LINKER_LIBS PUBLIC ${OpenCV_LIBS})
set(OpenCV_LIBS "/home/Lynkzhang/openCV/lib64/libopencv_core.so")

list(APPEND Caffe_LINKER_LIBS PUBLIC ${OpenCV_LIBS})
set(OpenCV_LIBS "/home/Lynkzhang/openCV/lib64/libopencv_highgui.so")
list(APPEND Caffe_LINKER_LIBS PUBLIC ${OpenCV_LIBS})
set(OpenCV_LIBS "/home/Lynkzhang/openCV/lib64/libopencv_imgcodecs.so")
list(APPEND Caffe_LINKER_LIBS PUBLIC ${OpenCV_LIBS})
set(OpenCV_LIBS "/home/Lynkzhang/openCV/lib64/libopencv_imgproc.so")
list(APPEND Caffe_LINKER_LIBS PUBLIC ${OpenCV_LIBS})

...

# ---[ BLAS
if(NOT APPLE)
  set(BLAS "MKL")

 ...

```

修改cmake/ProtoBuf.cmake

```
# Finds Google Protocol Buffers library and compilers and extends
# the standard cmake script with version and python generation support
set(PROTOBUF_LIBRARY "/home/Lynkzhang/Protobuf/lib/libprotobuf.so")
set(PROTOBUF_INCLUDE_DIR "/home/Lynkzhang/Protobuf/include/")
set(PROTOBUF_PROTOC_EXECUTABLE "/home/Lynkzhang/Protobuf/bin/protoc")
find_package( Protobuf REQUIRED )
list(APPEND Caffe_INCLUDE_DIRS PUBLIC ${PROTOBUF_INCLUDE_DIR})
list(APPEND Caffe_LINKER_LIBS PUBLIC ${PROTOBUF_LIBRARIES})

...

```


一般CUDNN都会在Cuda文件夹下面，找不到的情况下只能自己下载解压，添加路径:
修改cmake/Cuda.cmake

```
...

# cudnn detection

if(USE_CUDNN)
  set(CUDNN_INCLUDE "/home/Lynkzhang/cuda/include")
  set(CUDNN_LIBRARY "/home/Lynkzhang/cuda/lib64/libcudnn.so")
  list(APPEND Caffe_DEFINITIONS PUBLIC -DUSE_CUDNN)
  list(APPEND Caffe_INCLUDE_DIRS PUBLIC ${CUDNN_INCLUDE})
  list(APPEND Caffe_LINKER_LIBS PUBLIC ${CUDNN_LIBRARY})
endif()

...



```


简单的编译:
```
mkdir build
cd build
cmake ..
make -j
make runtest
```


make runtest的时候，测试有一个测试始终过不去，在论坛里有人给出了解决方案:给NVCC加上-G
https://github.com/BVLC/caffe/issues/6164


After countless caffe compilations and tests, finally, I find a workaround to this problem: I add a line NVCCFLAGS += -G to Makefile and it changes from

```
# Debugging
ifeq ($(DEBUG), 1)
        COMMON_FLAGS += -DDEBUG -g -O0
        NVCCFLAGS += -G
else
        COMMON_FLAGS += -DNDEBUG -O2
endif
```
to

```
# Debugging
ifeq ($(DEBUG), 1)
        COMMON_FLAGS += -DDEBUG -g -O0
        NVCCFLAGS += -G
else
        COMMON_FLAGS += -DNDEBUG -O2
        NVCCFLAGS += -G
endif
```
Then, compiling caffe again... and make runtest passes without failure!


但是论坛里面的人用的是Make 和 Makeconfigure 的方法，
cmake的方法需要修改CMakeList:

CMakeList.txt
```
 # add definitions to nvcc flags directly
  set(Caffe_ALL_DEFINITIONS ${Caffe_DEFINITIONS})
  list(REMOVE_ITEM Caffe_ALL_DEFINITIONS PRIVATE PUBLIC)
  list(APPEND CUDA_NVCC_FLAGS ${Caffe_ALL_DEFINITIONS})
  list(APPEND CUDA_NVCC_FLAGS "-G") # add here
```

重新从cmake开始编译一次。


最后结果：

[==========] 2139 tests from 285 test cases ran. (619216 ms total)

[  PASSED  ] 2139 tests.

[100%] Built target runtest
