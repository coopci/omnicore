写这个手工文档是因为https://github.com/OmniLayer/omnicore/tree/master/build_msvc里的步骤需要用vcpkg，但是 vcpkg根本没法用: https://github.com/microsoft/vcpkg/issues/10775

为了减少麻烦，所有依赖都以静态库形式使用。
根据https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md描述的版本来build依赖。

1. 从http://p-nand-q.com/programming/windows/building_openssl_with_visual_studio_2010.html下载编译好的openssl。
假设解压到 E:\src\openssl-1.1.0c-64bit-debug-static-VS2015
如果以后有心情再整理用visual studio build openssl的步骤。

2. git clone https://github.com/libevent/libevent.git
假设clone到E:\src\libevent\

2.1 用cmake为libevent生成sln文件。
cd E:\src\libevent\
set OPENSSL_ROOT_DIR=E:\src\openssl-1.1.0c-64bit-debug-static-VS2015
cmake . -A x64 -B "build64"

2.2 用vs2017打开 E:\src\libevent\build64\libevent.sln build libevent。
build好的.lib文件在E:\src\libevent\build64\lib\Debug

3 git clone https://github.com/zeromq/zeromq4-x.git
假设clone到E:\src\zeromq3-x\
用visual studio 打开 E:\src\zeromq4-x\builds\msvc\msvc11.sln

选择build x64 并把libzmq11 改成static library。

build得到 E:\src\zeromq4-x\bin\x64\libzmq_d.lib



项目内的boost是从https://www.boost.org/users/download/下载的boost_1_72_0.tar.bz2
https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.bz2 可以验证没有改变
要这样build boost: 
./b2 address-model=64 toolset=msvc-141 link=static --build-type=complete stage --with-filesystem --with-system --with-thread --with-date_time
