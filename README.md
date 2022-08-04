# ijkPlayer编译与使用



Ijkplayer  github地址：https://github.com/bilibili/ijkplayer

ndk下载地址：https://github.com/android/ndk/wiki/Unsupported-Downloads

## 编译环境：

mac12.5

git version 2.32.1 (Apple Git-133)

yasm 1.3.0

Homebrew 3.5.7

android-ndk-r12b

## 流程：

克隆ijkplayer代码到ijkplayer-android

```bash
git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android
```

进入ijkplayer-android文件夹

```bash
cd ijkplayer-android
```

切换分支重命名为lastes

```bash
git checkout -B latest k0.8.8
```

执行初始化android脚本，需要下载很长时间如果中途出现timeout或者链接问题重复执行，直到全部下载完成

```bash
./init-android.sh
```

可选： 添加https支持，需要执行初始化openssl会下载相应文件

```bash
./init-android-openssl.sh
```

进入android/contrib

```bash
cd android/contrib
```

执行脚本编译所需文件

```ba
清理编译残留
./compile-openssl.sh clean
./compile-ffmpeg.sh clean
编译所有cpu架构
./compile-openssl.sh all
./compile-ffmpeg.sh all
```

AndroidNdk 选择10-14，本次使用12编译成功，编译所有cup架构时报错，原因不明：

*collect2: error: ld returned 1 exit status*

解决方案参考：https://github.com/bilibili/ijkplayer/issues/5113

可以分别选择cup架构编译成功，项目只需要armv7a和arm64都可以成功

```bash
./compile-openssl.sh armv7a
./compile-ffmpeg.sh armv7a
```

返回上一级目录ijkplayer-android，编译so包

```bas
cd ..
cd ..
./compile-ijk.sh all
同样这里选择分别编译armv7a和arm64
```

生成的so包位于**ijkplayer-android/android/ijkprof/ijkplayer-xxx/src/main/libs**下

## 选择编码格式

在ijkplayer-android目录下执行命令

```bas
cd config
删除默认文件
rm module.sh
选择默认编码格式，
ln -s module-default.sh module.sh
选择较少的编码格式以及较小的二进制大小包含hevc
ln -s module-lite-hevc.sh module.sh
选择较少的编码格式以及较小的二进制大小不包含hevc
ln -s module-lite.sh module.sh

```

或者自行编辑module.sh文件，自定义所需编码格式



## 工程运行

新建工程
拷贝ijkplayer-example和ijkplayer-java目录到项目中

修改setting.gradle文件

```groovy

include ':ijkplayer-java'
include ':ijkplayer-example'


```

拷贝编译好的so文件至ijkplayer-example

修改ijkVideoView删除与IjkExoMediaPlayer相关代码即可运行





## 问题总结

**使用Git克隆[github](https://so.csdn.net/so/search?q=github&spm=1001.2101.3001.7020)仓库时一直报错**

1. *HTTP/2 stream 1 was not closed cleanly before end of the underlying stream*

   通过排查发现，是 `git` 默认使用的[通信协议](https://so.csdn.net/so/search?q=通信协议&spm=1001.2101.3001.7020)出现了问题，可以通过将默认通信协议修改为 `http/1.1` 来解决该问题。

   ```bash
   git config --global http.version HTTP/1.1
   ```

2. *curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused*

   修改 hosts 文件，输入如下命令
   ```ba
   sudo vi /etc/hosts
   ```

   在文件中添加

   ``` bash
   140.82.112.4	github.com
   140.82.113.4	github.com
   140.82.112.3	github.com
   140.82.114.4	github.com
   199.232.68.133 raw.githubusercontent.com
   199.232.68.133 user-images.githubusercontent.com
   199.232.68.133 avatars2.githubusercontent.com
   199.232.68.133 avatars1.githubusercontent.com
   ```

3. *Host 'awk' tool is outdated. Please define NDK_HOST_AWK to point to Gawk or Nawk !*
   解决方案：$(NDK_ROOT)/build/core/init.mk文件中注释掉
   HOST_AWK := $(wildcard $(HOST_PREBUILT)/awk$(HOST_EXEEXT))
   参考文献：https://www.cnblogs.com/smartptr/p/15666227.html











