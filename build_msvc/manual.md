因为数字货币固有的安全性要求，所以写了这个完全从源代码build的手册。

写这个纯手工形式的文档是因为https://github.com/OmniLayer/omnicore/tree/master/build_msvc里的步骤需要用vcpkg，但是 vcpkg根本没法儿用: https://github.com/microsoft/vcpkg/issues/10775


为了减少麻烦，所有依赖都以静态库形式使用。
根据https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md描述的版本来build依赖。

这个repo已经把所有依赖都作为submodule提供，其中大部分(libevent, zeromq4-x, berkeleydb)提供了vcxproj项目，但是 openssl和boost没有提供vcxproj项目，需要用命令行build。
这里只说明openssl和boost的build步骤，因为其他依赖会由 visual studio IDE自动bulid。

1. build openssl
git submodule update --init openssl
进入 Developer command prompt for visual studio 好像普通的命令行是不行的。
cd openssl
vcvars64.bat #这个用来设置vc的环境变量为x64。 如果使用visual studio 2017  community，这个bat的默认位置是 C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\VC\Auxiliary\Build\vcvars64.bat

perl Configure VC-WIN64A no-shared # 生成 makefile

nmake # build，如果成功的话会得到libcrypto.lib和libssl.lib

2. build boost

项目内的boost是从https://www.boost.org/users/download/下载的boost_1_72_0.tar.bz2
下载 https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.bz2 并解压到boost_1_72_0
要这样build boost: 

git submodule update --init boost_1_72_0 # 不要用recursive，否则会很慢。
cd boost_1_72_0

git submodule update --init # 下载 boost的子模块 这个步骤可能很慢，可以尝试https://dl.bintray.com/boostorg/release/1.72.0/source/boost_1_72_0.tar.bz2 并替并将其中的libs目录和tools目录解压到当前目录。

# 只下载需要的boost submodule:
git submodule update --init libs/filesystem
git submodule init libs/filesystem
git submodule update libs/filesystem

git submodule update --init libs/system

git submodule update --init libs/thread

git submodule update --init libs/chrono
git submodule update --init libs/date_time

git submodule update --init libs/multiprecision

git submodule update --init tools/build

git submodule update --init tools/boost_install

git submodule update --init libs/config
git submodule update --init libs/headers
git submodule update --init libs/static_assert
git submodule update --init libs/assert
git submodule update --init libs/type_traits
git submodule update --init libs/winapi
git submodule update --init libs/predef
git submodule update --init libs/core

.\bootstrap.bat
.\b2 headers
.\b2 address-model=64 toolset=msvc-141 link=static --build-type=complete stage --with-filesystem --with-system --with-thread --with-date_time --with-multiprecision


./b2 address-model=64 link=static --build-type=complete stage --with-filesystem --with-system --with-thread --with-date_time --with-multiprecision


如果上述步骤成功，boost_1_72_0/stage/lib下会产生需要的静态lib

3. build omnicore的bitcoind
git submodule update --init 其它的依赖 .gitmodules里面记录了所有依赖的路径。
用visual studio 2017 打开 build_msvc/bitcoin.sln。
选择x64，build bitcoind。如果成功的话，可以得到 x64\Debug\bitcoind.exe