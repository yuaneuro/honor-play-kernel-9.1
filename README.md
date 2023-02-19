# honor-play-kernel-9.1-kindle4jerry
官方开源内核9.1小改  
## 功能
解除ptrace禁用

解锁SELinux

解锁一些玄学调度器 

试图用不同的编译器  

## 注意
已经测试可以用的系统 9.1.0.312以及基于312刷的其他第三方系统  
## 编译教程
### 第零步 环境配置
gcc version 9.4.0
```shell
apt install -y gcc
apt install -y bison 
apt install -y libssl-dev
wget https://developer.arm.com/-/media/Files/downloads/gnu-a/9.2-2019.12/binrel/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu.tar.xz
tar -xvf gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu.tar.xz
```
然后记得python软连接到你的python3上
```shell
ln -s /usr/bin/python3.8 /usr/bin/python
python -v
```
编辑/kernel/build.sh，里面开头有环境配置，里面有注释。  
除了内核文件之外，还需要一个编译工具在注释里面有提到  
### 第一步 编译
根据自己的路径配置，不要一股脑复制
```shell
cd honor-play-kernel-9.1
export PATH=$PATH:/root/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu/bin
export CROSS_COMPILE=/root/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
export GCC_COLORS=auto
export ARCH=arm64
mkdir ../out
mkdir ../build_out
make ARCH=arm64 O=../out merge_kirin970_defconfig
make ARCH=arm64 O=../out -j4
cp -f ../out/arch/arm64/boot/Image.gz tools
BUILD_DATE=`date +%Y%m%d`
#permissive版打包
cd tools/
./mkbootimg --kernel Image.gz --base 0x0 --cmdline "loglevel=4 initcall_debug=n page_tracker=on unmovable_isolate1=2:192M,3:224M,4:256M printktimer=0xfff0a000,0x534,0x538 androidboot.selinux=permissive buildvariant=user" --tags_offset 0x07A00000 --kernel_offset 0x00080000 --ramdisk_offset 0x07C00000 --header_version 1 --os_version 9 --os_patch_level 2019-05-05  --output kernel-$BUILD_DATE-permissive.img
#enforcing版打包
./mkbootimg --kernel Image.gz --base 0x0 --cmdline "loglevel=4 initcall_debug=n page_tracker=on unmovable_isolate1=2:192M,3:224M,4:256M printktimer=0xfff0a000,0x534,0x538 androidboot.selinux=enforcing buildvariant=user" --tags_offset 0x07A00000 --kernel_offset 0x00080000 --ramdisk_offset 0x07C00000 --header_version 1 --os_version 9 --os_patch_level 2019-05-05  --output kernel-$BUILD_DATE-enforcing.img
#把输出放到build_out并清理
cp -f *.img ../../build_out
rm -f Image.gz
rm -f *.img
cd ..
```

## 关于菊花如何解除ptrace禁用
arch/arm64/configs/merge_kirin970_defconfig中添加`CONFIG_HUAWEI_PTRACE_POKE_ON=y`即可

## 关于菊花如何解锁selinux
arch/arm64/configs/merge_kirin970_defconfig中找到并且设置CONFIG_SECURITY_SELINUX_DEVELOP=y  
在没用上述命令编译内核之前，慎用cmdline的androidboot.selinux=permissive那个方法停用selinux，菊花会让你开不开机的。  
在用了上述命令编译之后，就可以用cmdline的androidboot.selinux=permissive的方法了，我已经在打包脚本/Kernel/tools/mk1.sh这个脚本里体现了  
 
## 关于调度
CPU：Schedtutil+EAS在原生的表现比较好，但是在官方emui可能就不是那么好了  
GPU：拿出了公版mali的调度器mali_ondemand参数方面有待调整，不过默认参数还可以  
