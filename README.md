# ArmNN_Build_x86_64_Linux
ArmNN v21.08 and PyArmNN build steps for x86_64_Linux System

OS : Ubuntu 18.04 <br>
ArmNN Version : 21.08 <br>
PyArmNN Version : 26.0.0

## Steps to build ArmNN: 

1) Start by creating a directory to contain all components:<br>
```
mkdir $HOME/armnn-devenv cd $HOME/armnn-devenv
```
2) Get protobuf v3.12.0 only
```git clone -b v3.12.0 https://github.com/google/protobuf.git protobuf
cd protobuf
git submodule update --init --recursive
./autogen.sh
```
3) Build a native (x86_64) version of the protobuf libraries and compiler (protoc): <br>(Requires cUrl, autoconf, llibtool, and other build dependencies if not previously installed: ```sudo apt install curl autoconf libtool build-essential g++```) 
```mkdir x86_64_build
cd x86_64_build
../configure --prefix=$HOME/armnn-devenv/google/x86_64_pb_install
make install -j16
cd ..
```
4) Building the Arm Compute Library: (Requires g++-arm-linux-gnueabihf, if not installed, ```sudo apt-get install g++-arm-linux-gnueabihf```)<br>
```cd $HOME/armnn-devenv
git clone https://github.com/ARM-software/ComputeLibrary.git
cd ComputeLibrary/
git checkout v21.08
scons arch=x86_64 opencl=1 embed_kernels=1 extra_cxx_flags="-fPIC" -j4 internal_only=0
```
5) Download ArmNN:
```cd $HOME/armnn-devenv
git clone https://github.com/ARM-software/armnn.git
cd armnn
git checkout branches/armnn_21_08
git pull
```
6) Building Flatbuffer version 1.12.0 :
```cd $HOME/armnn-devenv
wget -O flatbuffers-1.12.0.tar.gz https://github.com/google/flatbuffers/archive/v1.12.0.tar.gz
tar xf flatbuffers-1.12.0.tar.gz
cd flatbuffers-1.12.0
rm -f CMakeCache.txt
mkdir build
cd build
CXXFLAGS="-fPIC" cmake .. -DFLATBUFFERS_BUILD_FLATC=1 \
     -DCMAKE_INSTALL_PREFIX:PATH=$HOME/armnn-devenv/flatbuffers \
     -DFLATBUFFERS_BUILD_TESTS=0
make all install
```
7) Building Onnx:
```cd $HOME/armnn-devenv
git clone https://github.com/onnx/onnx.git
cd onnx
git fetch https://github.com/onnx/onnx.git 553df22c67bee5f0fe6599cff60f1afc6748c635 && git checkout FETCH_HEAD
LD_LIBRARY_PATH=$HOME/armnn-devenv/google/x86_64_pb_install/lib:$LD_LIBRARY_PATH \
$HOME/armnn-devenv/google/x86_64_pb_install/bin/protoc \
onnx/onnx.proto --proto_path=. --proto_path=../google/x86_64_pb_install/include --cpp_out $HOME/armnn-devenv/onnx
```
8) Building TfLite(v2.3.1):
```cd $HOME/armnn-devenv
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow/
git checkout fcc4b966f1265f466e82617020af93670141b009
cd ..
mkdir tflite
cd tflite
cp ../tensorflow/tensorflow/lite/schema/schema.fbs .
../flatbuffers-1.12.0/build/flatc -c --gen-object-api --reflect-types --reflect-names schema.fbs
```
9) Compile Arm NN for x86_64:
```cd $HOME/armnn-devenv/armnn
mkdir build
cd build
```
Build configurations :
```
CXX=x86_64-linux-gnu-g++ CC=x86_64-linux-gnu-gcc CXXFLAGS="-fPIC" cmake .. \
-DARMCOMPUTE_ROOT=$HOME/armnn-devenv/ComputeLibrary \
-DARMCOMPUTE_BUILD_DIR=$HOME/armnn-devenv/ComputeLibrary/build/ \
-DARMCOMPUTECL=1 -DARMNNREF=1 \
-DONNX_GENERATED_SOURCES=$HOME/armnn-devenv/onnx \
-DBUILD_ONNX_PARSER=1 \
-DBUILD_TF_LITE_PARSER=1 \
-DBUILD_ARMNN_SERIALIZER=1 \
-DBUILD_PYTHON_SRC=1 \
-DBUILD_PYTHON_WHL=1 \
-DTF_LITE_GENERATED_PATH=$HOME/armnn-devenv/tflite \
-DFLATBUFFERS_ROOT=$HOME/armnn-devenv/flatbuffers \
-DFLATC_DIR=$HOME/armnn-devenv/flatbuffers-1.12.0/build \
-DPROTOBUF_ROOT=$HOME/armnn-devenv/google/x86_64_pb_install \
-DPROTOBUF_ROOT=$HOME/armnn-devenv/google/x86_64_pb_install/ \
-DPROTOBUF_LIBRARY_DEBUG=$HOME/armnn-devenv/google/x86_64_pb_install/lib/libprotobuf.so.23.0.0 \
-DPROTOBUF_LIBRARY_RELEASE=$HOME/armnn-devenv/google/x86_64_pb_install/lib/libprotobuf.so.23.0.0 \
-DPYTHON_INCLUDE_DIR=$(python3 -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())")  \
-DPYTHON_LIBRARY=$(python3 -c "import distutils.sysconfig as sysconfig; print(sysconfig.get_config_var('LIBDIR'))")
```
Run the build : 
```
make -j4
```

### Note : Following the above steps will build ArmNN v21.08 for x86_64_linux system with no CpuAcc, deserializer & tflite support.

## Steps to build PyArmNN :

Before, proceeding to the next steps, make sure that:
1) You have Python 3.6+ installed system-side. The package is not compatible with older Python versions.
2) You have python3.6-dev installed system-side. This contains header files needed to build PyArmNN extension module.
3) Install SWIG 4.x. Only 3.x version is typically available in Linux package managers, so you will have to build it and install it from sources.To install it follow the guide on [SWIG GitHub](https://github.com/swig/swig/wiki/Getting-Started) or [here](https://www.dev2qa.com/how-to-install-swig-on-macos-linux-and-windows/).

## PyArmNN installation : 
### Installing from source package : 
```cd $HOME/armnn-devenv/armnn/build/python/pyarmnn/dist
export  ARMNN_LIB=$HOME/armnn-devenv/armnn/build
export  ARMNN_INCLUDE=$HOME/armnn-devenv/armnn/include:$HOME/armnn-devenv/armnn/profiling/common/include
```
### Install PyArmNN as follows :
```
pip3 install pyarmnn-26.0.0.tar.gz
```
### You can now verify that PyArmNN library is installed and check PyArmNN version using:
```
pip3 show pyarmnn
```

## References : 
1) For ArmNN Build : https://arm-software.github.io/armnn/21.05/md__build_guide_cross_compilation.xhtml
2) For PyArmNN Build : https://github.com/ARM-software/armnn/tree/branches/armnn_21_08/python/pyarmnn
