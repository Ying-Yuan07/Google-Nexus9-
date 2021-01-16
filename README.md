# Google-Nexus9-compile AOSP and kernel    
write by 涂涂, commit by Lihua    

A aosp上的f2fs   
============
官方文档
---------
https://source.android.google.cn/source/building-kernels?hl=zh-cn     
https://source.android.google.cn/setup/build/building-kernels-deprecated     

版本选择
--------
AOSP:    
android8.0不能刷nexus9,所以下载版本选择android-7.1.1_r39    
内核：     
nexus 9 代号volantis(flounder)     
user_debug选择flounder    
内核代码下载选择kernel/tegra     

下载AOSP  
-------
参考地址：https://lug.ustc.edu.cn/wiki/mirrors/help/aosp   
mkdir ~/bin   
PATH= ~/bin:$PATH    
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo    
##如果上述 URL 不可访问，可以用下面的：    
##curl -sSL  'https://gerrit-googlesource.proxy.ustclug.org/git-repo/+/master/repo?format=TEXT' |base64 -d > ~/bin/repo     

chmod a+x ~/bin/repo  
然后建立一个工作目录（名字任意）    
mkdir WORKING_DIRECTORY     
cd WORKING_DIRECTORY    
初始化仓库：     
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest      
#如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：    
#REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'     
如果需要某个特定的 Android 版本：   
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-7.1.1_r39     
同步源码树（以后只需执行这条命令来同步）：     
repo sync       

下载tegra-kernel
===========
#下载tegra代码，由于google下不了，去ustc下载      
git clone git://mirrors.ustc.edu.cn/aosp/kernel/tegra.git/    

编译内核 
=======
只需要第一次切换       
git checkout -b android-tegra-flounder-3.10-nougat origin/android-tegra-flounder-3.10-nougat    

cd /mnt/tsy/WORKING_DIRECTORY    

source build/envsetup.sh    
lunch aosp_flounder-userdebug    
export ARCH=arm64     
export CROSS_COMPILE=aarch64-linux-android-     
cd /mnt/tsy/tegra    
make flounder_defconfig    
make -j24     

#将编译结果替换aosp中的Image.gz-dtb    
cp /mnt/tsy/tegra/arch/arm64/boot/Image.gz-dtb /mnt/tsy/WORKING_DIRECTORY/device/htc/flounder-kernel    

参考linaro: https://source.android.google.cn/source/devices?hl=zh-cn     
编译aosp
==========
cd /mnt/tsy/WORKING_DIRECTORY     
source build/envsetup.sh    
lunch aosp_flounder-userdebug    
export ARCH=arm64    
export CROSS_COMPILE=aarch64-linux-android-    
make -j24     


生成的结果在/mnt/tsy/WORKING_DIRECTORY/out/target/product/flounder中    

刷机
=====
解锁   
进入开发者模式   
开启usb权限   
#第一次刷机，在WORKING_DIRECTORY下刷所有的镜像   
adb reboot bootloader    
fastboot flashall -w    


后续的拷贝到笔记本上，只刷boot.img    
adb reboot bootloader   
fastboot flash boot boot.img   
fastboot reboot   

交叉编译
========
./arm/bin/clang -pie test.c -o test    
刷完重启，测试   
adb push sourceFile distDir      
