

#complie for android 5.1 (liunx :ubuntu 16.04, ubuntu18.04.4 都测试成功):

refrence１:https://github.com/jcsilva/docker-kaldi-android/blob/master/Dockerfile

refrence２:http://jcsilva.github.io/2017/03/18/compile-kaldi-android/

refrence３：https://blog.csdn.net/qq_33335062/article/details/80918600

我的工作目录是/home/md510/w2019b，没有特殊说明都在这个目录下操作

就在此工作目录下执行：

１．下载ＮＤＫ

`$ wget -q --output-document=android-ndk.zip https://dl.google.com/android/repository/android-ndk-r16b-linux-x86_64.zip`

２．`$ unzip android-ndk.zip` 生成android-ndk-r16b文件夹

3.  生成交叉编译工具链./my-android-toolchain，
api 21　指的你要运行安卓的版本对应的ＡＰＩ,因为这个例子是想在安卓５．１编译KALDI，所以这里指定ＡＰＩ为２１　，
注意．因为咱们生成的my-android-toolchain　是在/tmp下所以ubuntu 系统重启是会消失的，直接开机再执行下面的命令即可

`$ android-ndk-r16b/build/tools/make_standalone_toolchain.py --arch arm --api 21 --stl=libc++ --install-dir ./home/md510/w2019b/my-android-toolchain`

４．Compile OpenBLAS for Android

Download source

`$ git clone https://github.com/xianyi/OpenBLAS`

Install gfortran

`$ sudo apt-get install gfortran`

５．把交叉编译工具链的路径添加到用户环境变量（~/.bashrc）里面，方便找到．

vim ~/.bashrc

把这三行添加到bashrc里面
```
export ANDROID_TOOLCHAIN_PATH=/home/md510/w2019b/my-android-toolchain
export PATH=${ANDROID_TOOLCHAIN_PATH}/bin:$PATH
export CLANG_FLAGS="-target arm-linux-androideabi -marm -mfpu=vfp -mfloat-abi=softfp --sysroot ${ANDROID_TOOLCHAIN_PATH}/sysroot -gcc-toolchain ${ANDROID_TOOLCHAIN_PATH}"
```

6.Build for ARMV7 ，ＡＲＭＶ７是安卓系统ＣＰＵ的内核指令集的名字，这个代表ＡＲＭ３２位处理器

`$ cd OpenBLAS`
`$ make TARGET=ARMV7 ONLY_CBLAS=1 AR=ar CC="clang ${CLANG_FLAGS}" HOSTCC=gcc ARM_SOFTFP_ABI=1 USE_THREAD=0 NUM_THREADS=32 -j4`

Install library

```$ make install NO_SHARED=1 PREFIX=`pwd`/install ```

７．Compile CLAPACK for Android

`$ cd ..`
`$ git clone https://github.com/simonlynen/android_libs` 

8. Comment the following four commands
`$ cd android_libs/lapack `
`$ sed -i 's/LOCAL_MODULE:= testlapack/#LOCAL_MODULE:= testlapack/g' jni/Android.mk`
`$ sed -i 's/LOCAL_SRC_FILES:= testclapack.cpp/#LOCAL_SRC_FILES:= testclapack.cpp/g' jni/Android.mk `
`$ sed -i 's/LOCAL_STATIC_LIBRARIES := lapack/#LOCAL_STATIC_LIBRARIES := lapack/g' jni/Android.mk `
`$ sed -i 's/include $(BUILD_SHARED_LIBRARY)/#include $(BUILD_SHARED_LIBRARY)/g' jni/Android.mk`
 
 At this current folder, run this command
 
`$ /home/md510/w2019b/android-ndk-r16b/ndk-build`　 

9. Copy libs from obj/local/armeabi-v7a/ to the same place you installed OpenBLAS libraries

(e.g: OpenBlas/install/lib). Kaldi 

will look at this directory for libf2c.a, liblapack.a, libclapack.a and libblas.a.
  
`$ cp obj/local/armeabi-v7a/*.a /home/md510/w2019b/OpenBLAS/install/lib`

10.Compile kaldi for Android
Download kaldi source code

`$ cd ../../ `
`$ git clone https://github.com/kaldi-asr/kaldi.git kaldi-android`

Compile OpenFST

`$ cd kaldi-android/tools`

10. compile openfst tools

`$ wget -T 10 -t 1 http://www.openslr.org/resources/2/openfst-1.6.5.tar.gz `
`$ tar -zxvf openfst-1.6.5.tar.gz`
`$ cd openfst-1.6.5/`
`$ CXX=clang++ ./configure --prefix=`pwd` --enable-static --enable-shared --enable-far --enable-ngram-fsts --host=arm-linux-androideabi LIBS="-ldl" `
` $ make -j 4 `
` $ make install `
` $ cd ..`
` $ ln -s openfst-1.6.5 openfst`
` $ make cub`

11．Compile src

`$ cd ../src`

Be sure android-toolchain is in your $PATH before the next step

`$ CXX=clang++ ./configure --static --android-incdir=/home/md510/w2019b/my-android-toolchain/sysroot/usr/include/ --host=arm-linux-androideabi --openblas-root=/home/md510/w2019b/OpenBLAS/install `

You may want to compile Kaldi without debugging symbols.

In this case, do:

` $ sed -i 's/-g # -O0 -DKALDI_PARANOID/-O3 -DNDEBUG/g' kaldi.mk `
` $ make clean -j 4 `
` $ make depend -j 4 `
` $ make -j 4 `

12. 
```
去/home/md510/w2019b/my-android-toolchain/arm-linux-androideabi/lib/armv7-a/　找到libc++_shared.so 
然后把它复制到安卓手机/system/lib　即可

最后生成so文件在kaldi/src/lib中 没找到的话要去改kaldi/src/configure中的这个
--android-incdir=*)
android=true;
threaded_math=false;
static_math=true;
static_fst=true;
dynamic_kaldi=true; #需要手动改为true，否则不能生成
MATHLIB='OPENBLAS';
ANDROIDINC=`read_dirname $1`;
shift;
```
 




# 最新版本：

complie for android 8.1(安卓系统的ＣＰＵ架构是arm35，是一个６４位的armv8a指令集的处理器，

因为实验这个安卓系统版本是8.1,它对应的系统ＡＰＩ的level是２７) (以下工作都在liunx完成 ，具体liunx系统是ubuntu 16.04):

refrence1:https://github.com/jcsilva/docker-kaldi-android/blob/master/Dockerfile

refrence2:http://jcsilva.github.io/2017/03/18/compile-kaldi-android/

refrence3：https://blog.csdn.net/qq_33335062/article/details/80918600

refrence4:https://github.com/xianyi/OpenBLAS/wiki/How-to-build-OpenBLAS-for-Android

refrence5:https://blog.csdn.net/cs123951/article/details/80054859

我的工作目录是/home/md510/Downloads，没有特殊说明都在这个目录下操作

就在此工作目录下执行：

１．下载ＮＤＫ

`$ wget -q --output-document=android-ndk.zip https://dl.google.com/android/repository/android-ndk-r18b-linux-x86_64.zip`

２．` $ unzip  android-ndk.zip ` 生成android-ndk-r18b文件夹

3. 生成交叉编译工具链/home/md510/w2019a/android-toolchain　，

api 27　指的你要运行安卓的版本对应的ＡＰＩ,因为这个例子是想在安卓8.1编译ＫＡＬＤＩ，

所以这里指定ＡＰＩ为２7　， 特别注意armv7就用–arch arm，armv8选择–arch arm64，

后面的api根据你本地的编译器版本来选择,来自refrence5提示．

`$ android-ndk-r18b/build/tools/make_standalone_toolchain.py --arch arm64 --api 27 --stl=libc++ --install-dir /home/md510/w2020/kaldi_android_8.1/my-android-toolchain`

４．Compile OpenBLAS for Android
＃# Download source

`$ git clone https://github.com/xianyi/OpenBLAS`

＃Install gfortran

`$ sudo apt-get install gfortran `

把交叉编译工具链的路径添加到用户环境变量（~/.bashrc）里面，方便找到．

vim ~/.bashrc

把这几行添加到bashrc里面
```
export ANDROID_TOOLCHAIN_PATH=/home/md510/w2019a/android-toolchain
export PATH=${ANDROID_TOOLCHAIN_PATH}/bin:$PATH
export NDK_BUNDLE_DIR=/home/md510/w2019a/android-ndk-r18b
```

然后source ~/.bashrc

`$ cd OpenBLAS `

然后执行以下两条命令

` $ make TARGET=ARMV8 BINARY=64 HOSTCC=gcc CC=aarch64-linux-android-gcc NOFORTRAN=1 `

` $ make PREFIX=$(pwd)/install   install `

然后你就会在当前目录下发现有一个install 文件夹，里面的lib 文件夹里面有这样的文件

libopenblas_armv8p-r0.3.6.dev.so  libopenblas.so.0　　libopenblas.so　libopenblas.a　说明安装成功了

５．Compile CLAPACK for Android

`$ cd .. `

`$ git clone https://github.com/simonlynen/android_libs.git `

`$ cd android_libs/lapack ` 

 Comment the following four commands

`$ sed -i 's/LOCAL_MODULE:= testlapack/#LOCAL_MODULE:= testlapack/g' jni/Android.mk `

`$ sed -i 's/LOCAL_SRC_FILES:= testclapack.cpp/#LOCAL_SRC_FILES:= testclapack.cpp/g' jni/Android.mk ` 

`$ sed -i 's/LOCAL_STATIC_LIBRARIES := lapack/#LOCAL_STATIC_LIBRARIES := lapack/g' jni/Android.mk `

`$ sed -i 's/include $(BUILD_SHARED_LIBRARY)/#include $(BUILD_SHARED_LIBRARY)/g' jni/Android.mk `

在这个目录下保持不动

修改 jni/Application.mk,下面是修改后的内容
```
APP_STL := c++_static　　#把gnustl_static　修改为 c++_static　
APP_CPPFLAGS := -frtti -fexceptions -mfloat-abi=softfp -mfpu=neon -std=gnu++0x -Wno-deprecated \
-ftree-vectorize -ffast-math -fsingle-precision-constant
APP_ABI := arm64-v8a　　# build for android (v8 指令集)　
APP_OPTIM := release
```
```
修改AndroidManifest.xml

把＇<uses-sdk android:minSdkVersion="10" />＇　中的数字10　改为27 因为我们的安卓系统api 是27

修改project.properties

把target=android-10　改为target=android-27
```

在当前路径下执行下面命令：

`$　/home/md510/w2019a/android-ndk-r18b/ndk-build `

就会在当前路径下生成 obj/local/arm64-v8a/*.a　＃*表示.a 的名字，由于太多所以用* 代替来展示

把编译生成的库复制到 /home/md510/w2019a/OpenBLAS/install/lib　，这里/home/md510/w2019a/OpenBLAS/　下面的install 文件夹是在第四步．
Compile OpenBLAS for Android中最后一个命令　make PREFIX=$(pwd)/install  TARGET=ARMV8 install 指定的，你也可指定别的目录．

`$ cp obj/local/arm64-v8a/*.a /home/md510/w2019a/OpenBLAS/install/lib `


６．Compile kaldi for Android
Download kaldi source code

`$ cd ../../ `

`$ git clone https://github.com/kaldi-asr/kaldi.git kaldi-android  `

vim ~/.bashrc

添加以下命令　为了编译OpenFST
Tell configure what tools to use.
```
target_host=aarch64-linux-android
export AR=$target_host-ar
export AR=ar
export AS=${target_host}-clang
export CC=${target_host}-clang
export CXX=$target_host-clang++
export LD=${target_host}-ld
export STRIP=${target_host}-strip
```

Compile OpenFST

` $ cd kaldi-android/tools `
` $ wget -T 10 -t 1 http://www.openslr.org/resources/2/openfst-1.6.7.tar.gz `
` $ tar -zxvf openfst-1.6.7.tar.gz `
` $ cd openfst-1.6.7/ `
` $ CXX=clang++ ./configure --prefix=`pwd` --enable-static --enable-shared  --enable-ngram-fsts --host=aarch64-linux-android `
` $ make -j 4 `
` $ make install `
` $ cd .. `
` $ ln -s openfst-1.6.7 openfst `

７．Compile src
`$ cd ../src`

执行下面这条命令

`$ CXX=clang++  ./configure --static --android-incdir=/home/md510/w2019a/android-toolchain/sysroot/usr/include/ --host=aarch64-linux-android --openblas-root=/home/md510/w2019a/OpenBLAS/install `

```
然后才能生成一个kaldi.mk
然后可能提示:
***configure failed: Could not find file /cub/cub.cuh:
  you may not have installed cub.  Go to ../tools/ and type
  e.g. 'make cub'; cub is a new requirement. ***

```
然后你执行：
`$ cd ..`
`$ cd tools `
`$ make cub `
这样就把cub 安装成功了
然后再执行
`$ cd ../src `
`$ CXX=clang++  ./configure --static --android-incdir=/home/md510/w2019a/android-toolchain/sysroot/usr/include/ --host=aarch64-linux-android --openblas-root=/home/md510/w2019a/OpenBLAS/install `

You may want to compile Kaldi without debugging symbols.
In this case, do:
`$ sed -i 's/-g # -O0 -DKALDI_PARANOID/-O3 -DNDEBUG/g' kaldi.mk `

```
修改kaldi.mk
删除
-mfloat-abi=softfp -mfpu=neon 
添加
-march=arm64v8-a
```
```
make clean -j 12
make depend -j 12
 make -j 12
```
8. 去/home/md510/w2019a/android-toolchain/aarch64-linux-android/lib找到libc++_shared.so 
然后把它复制到安卓手机/system/lib　即可


# 2020-5-17 updated 
complie for android 8.1(安卓系统的ＣＰＵ架构是arm35，是一个６４位的armv8a指令集的处理器，

因为实验这个安卓系统版本是8.1,它对应的系统ＡＰＩ的level是２７) (以下工作都在liunx完成 ，具体liunx系统是ubuntu 18.04.4):

refrence1:https://github.com/jcsilva/docker-kaldi-android/blob/master/Dockerfile

refrence2:http://jcsilva.github.io/2017/03/18/compile-kaldi-android/

refrence3：https://blog.csdn.net/qq_33335062/article/details/80918600

refrence4:https://github.com/xianyi/OpenBLAS/wiki/How-to-build-OpenBLAS-for-Android

refrence5:https://blog.csdn.net/cs123951/article/details/80054859

我的工作目录是/home/md510/w2020/kaldi_android_8.1，没有特殊说明都在这个目录下操作

就在此工作目录下执行：

１．下载ＮＤＫ
`$ wget -q --output-document=android-ndk.zip https://dl.google.com/android/repository/android-ndk-r18b-linux-x86_64.zip`

２．` $ unzip  android-ndk.zip ` 生成android-ndk-r18b文件夹

3. 生成交叉编译工具链/home/md510/w2020/kaldi_android_8.1/my-android-toolchain　，

api 27　指的你要运行安卓的版本对应的ＡＰＩ,因为这个例子是想在安卓8.1编译ＫＡＬＤＩ，

所以这里指定ＡＰＩ为２7　， 特别注意armv7就用–arch arm，armv8选择–arch arm64，

后面的api根据你本地的编译器版本来选择,来自refrence5提示．

`$ android-ndk-r18b/build/tools/make_standalone_toolchain.py --arch arm64 --api 27 --stl=libc++ --install-dir /home/md510/w2020/kaldi_android_8.1/my-android-toolchain`

４．Compile OpenBLAS for Android
Download source

`$ git clone https://github.com/xianyi/OpenBLAS`

Install gfortran

`$ sudo apt-get install gfortran `

把交叉编译工具链的路径添加到用户环境变量（~/.bashrc）里面，方便找到．
vim ~/.bashrc
把这几行添加到bashrc里面
```
export ANDROID_TOOLCHAIN_PATH=/home/md510/w2020/kaldi_android_8.1/my-android-toolchain
export PATH=${ANDROID_TOOLCHAIN_PATH}/bin:$PATH
export NDK_BUNDLE_DIR=/home/md510/w2020/kaldi_android_8.1/android-ndk-r18b
```

然后source ~/.bashrc

`$ cd OpenBLAS `

然后执行以下两条命令

` $ make TARGET=ARMV8 BINARY=64 HOSTCC=gcc CC=aarch64-linux-android-gcc NOFORTRAN=1 `
` $ make PREFIX=$(pwd)/install   install `

我在ubuntu 18.04.4 上进行编译时，在当前目录下你会发现一个install 文件夹，在install/lib包含以下文件：

libopenblas.a  libopenblas_armv8p-r0.3.9.dev.a  libopenblas_armv8p-r0.3.9.dev.so  libopenblas.so  libopenblas.so.0 说明安装成功了

５．Compile CLAPACK for Android
`$ cd .. `
`$ git clone https://github.com/simonlynen/android_libs.git `
`$ cd android_libs/lapack ` 

Comment the following four commands
`$ sed -i 's/LOCAL_MODULE:= testlapack/#LOCAL_MODULE:= testlapack/g' jni/Android.mk `
`$ sed -i 's/LOCAL_SRC_FILES:= testclapack.cpp/#LOCAL_SRC_FILES:= testclapack.cpp/g' jni/Android.mk ` 
`$ sed -i 's/LOCAL_STATIC_LIBRARIES := lapack/#LOCAL_STATIC_LIBRARIES := lapack/g' jni/Android.mk `
`$ sed -i 's/include $(BUILD_SHARED_LIBRARY)/#include $(BUILD_SHARED_LIBRARY)/g' jni/Android.mk `

在这个目录下保持不动

修改 jni/Application.mk,下面是修改后的内容
```
APP_STL := c++_static　　#把gnustl_static　修改为 c++_static，目的是生成静态库　你还可以选择这四个中其中一个 ：none system c++_static c++_shared 
APP_CPPFLAGS := -frtti -fexceptions -mfloat-abi=softfp -mfpu=neon -std=gnu++0x -Wno-deprecated \
-ftree-vectorize -ffast-math -fsingle-precision-constant
APP_ABI := arm64-v8a　　# build for android (v8 指令集)　
APP_OPTIM := release
```
```
修改AndroidManifest.xml
把＇<uses-sdk android:minSdkVersion="10" />＇　中的数字10　改为27 因为我们的安卓系统api 是27
修改project.properties
把target=android-10　改为target=android-27
```

在当前路径下执行下面命令：

`$　/home/md510/w2020/kaldi_android_8.1/android-ndk-r18b/ndk-build  `

就会在当前路径下生成 obj/local/arm64-v8a/*.a　说明成功这一步成功了

libblas.a  libcholrl.a  libcholtop.a  libclapack.a  libf2c.a  liblapack.a  liblucr.a  liblull.a  liblurec.a  libqrll.a

把编译生成的库复制到 OpenBLAS/install/lib

` $ cp obj/local/arm64-v8a/*.a /home/md510/w2020/kaldi_android_8.1/OpenBLAS/install/lib `

６．Compile kaldi for Android

Download kaldi source code

`$ cd ../../ `
`$ git clone https://github.com/kaldi-asr/kaldi.git kaldi-android  `

7. vim ~/.bashrc

添加以下命令　为了编译OpenFST

Tell configure what tools to use.
```
target_host=aarch64-linux-android
export AR=$target_host-ar
export AR=ar
export AS=${target_host}-clang
export CC=${target_host}-clang
export CXX=$target_host-clang++
export LD=${target_host}-ld
export STRIP=${target_host}-strip
```
8. Compile OpenFST
` $ cd kaldi-android/tools `
` $ wget -T 10 -t 1 http://www.openslr.org/resources/2/openfst-1.6.7.tar.gz `
` $ tar -zxvf openfst-1.6.7.tar.gz `
` $ cd openfst-1.6.7/ `
` $ CXX=clang++ ./configure --prefix=`pwd` --enable-static --enable-shared  --enable-ngram-fsts --host=aarch64-linux-android `
` $ make -j 4 `
` $ make install `
` $ cd .. `
` $ ln -s openfst-1.6.7 openfst `

记得这时编译以下cub

`$ make cub `

9．Compile src

`$ cd ../src `

执行下面这条命令

`$ CXX=clang++  ./configure --static --android-incdir=/home/md510/w2020/kaldi_android_8.1/my-android-toolchain/sysroot/usr/include/ --host=aarch64-linux-android --openblas-root=/home/md510/w2020/kaldi_android_8.1/OpenBLAS/install `

然后才能生成一个kaldi.mk

10.修改kaldi.mk
```
删除
-mfloat-abi=softfp -mfpu=neon 
添加
-march=arm64v8-a
```
11. compile openblas with android-toolchain,current folder is  kaldi_android/src

`$ make clean -j 4 `
`$ make depend -j 4 `
`$ make -j 4 `

12. 去/home/md510/w2020/kaldi_android_8.1/my-android-toolchain/aarch64-linux-android/lib找到libc++_shared.so 
然后把它复制到安卓手机/system/lib　即可


 
