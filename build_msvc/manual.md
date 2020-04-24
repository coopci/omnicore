写这个手工文档是因为https://github.com/OmniLayer/omnicore/tree/master/build_msvc里的步骤需要用vcpkg，但是 vcpkg根本没法用: https://github.com/microsoft/vcpkg/issues/10775

为了减少麻烦，所有依赖都以静态库形式使用。
根据https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md描述的版本来build依赖。

这里把所有依赖都作为submodule提供，其中大部分(libevent, zeromq4-x, berkeleydb)提供了vcxproj项目，但是 openssl和boost没有提供vcxproj项目，需要用命令行build。
这里只说明openssl和boost的build步骤，因为其他依赖会由 visual studio IDE自动bulid。
1. build openssl
进入 Developer command prompt for visual studio
cd openssl
vcvars64.bat #这个用来设置vc的环境变量为x64。 如果使用visual studio 2017  community，这个bat的默认位置是 C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvars64.bat

perl Configure VC-WIN64A no-shared # 生成 makefile

nmake # build，如果成功的话会得到libcrypto.lib和libssl.lib

2. build boost

项目内的boost是从https://www.boost.org/users/download/下载的boost_1_72_0.tar.bz2
下载 https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.bz2 并解压到boost_1_72_0
要这样build boost: 
cd boost_1_72_0
./bootstrap.bat
./b2 address-model=64 toolset=msvc-141 link=static --build-type=complete stage --with-filesystem --with-system --with-thread --with-date_time --with-multiprecision

如果上述步骤成功，boost_1_72_0/stage/lib下会产生需要的静态lib

3. build omnicore的bitcoind
用visual studio 2017 打开 build_msvc/bitcoin.sln。
选择x64，build bitcoind。如果成功的话，可以得到 x64\Debug\bitcoind.exe