#SDK使用说明
***
##SDK简介
本版SDK适配数迹的TOF系列模组，提供了模组相关部分驱动，并配套提供多种系统下多种编程语言环境的测试样例，以便通过样例熟悉相关API的使用，进行二次开发。

***
##SDK如何使用
- 进行对应系统的环境配置，如windows或者linux下进行对应的配置
- 安装设备的usb驱动
- 样例的使用和相关API说明
***

##系统使用环境配置方法
###windows下配置
- 环境：win10 64位, msys2 32/64位
- 配置
1. 从官网下载msys2（这里安装的是 msys2-i686- 开头的32位）并按照官网教程安装

2. 在msys2安装gcc，gdb，make几个相关的包
  pacman -S msys/make
  pacman -S msys/gcc
3. 分别安装不同版本的对应包
>若打开使用mingw32.exe，则分别运行
	pacman -S mingw32/mingw-w64-i686-gcc
	pacman -S mingw32/mingw-w64-i686-gdb
	pacman -S mingw32/mingw-w64-i686-make
>若打开使用mingw64.exe，则分别运行
	pacman -S mingw32/mingw-w64-x86_64-gcc
	pacman -S mingw32/mingw-w64-x86_64-gdb
	pacman -S mingw32/mingw-w64-x86_64-make
4. 安装USB驱动
  双击SDK中drivers\usb_driver目录下面的install_driver.bat文件，进行USB驱动安装。
5. 测试运行
>在SDK里的DMCAM\Windows\samples\c文件夹下的CMakeList.txt 所在目录建立文件夹build， 打开msys2编译相应工程。
	1$ cd build
	2$ cmake .. -G "MSYS Makefiles"   //第一次cmake失败可以再尝试一次
	3$ make -j  //or mingw32-make
	在build目录下有相应的sample开头的.exe格式的可执行文件生成
***
###linux下环境配置
- 环境：Ubuntu 14.04/16.04 64位
- 配置：
1. 安装cmake，打开终端，输入安装命令
  sudo apt-get install cmake

  因为ubuntu14和ubuntu16下的libusb驱动自带，这里不用额外再装
2. 测试运行
> 在DMCAM\linux\samples\c文件夹下的CMakeList.txt 所在目录建立文件夹build， 打开终端编译相应工程。
	1$ cd build
	2$ cmake .. -G "Unix Makefiles"
	3$ make -j
> 在build目录下有对应的sample开头的可执行文件生成
***
###样例使用说明
在DMCAM目录下包含windows和linux两个文件夹，里面都包含sample文件夹，里面分别提供有有C、python等不同语言的样例，相关步骤说明参考位于各个样例下的readme.txt。
***
##API说明
SDK中所用的相关结构体和函数说明都位于对应系统文件夹下的dmcam.h中，如使用windows则在DMCAM\Windows\lib\include下，函数的主要功能和具体参数介绍在dmcam.h中都有介绍。

##具体例程说明
>如在DMCAM\Windows\samples\c文件夹下，包含sample_capture_frames.c;sample_set_param.c;sample_filter.c等样例文件。
***
- sample_capture_frames.c  展示获取帧数据
1. 在主程序开始时，先调用dmcam_init()函数进行初始化，必须在调用任何dmcam的api前初始化
2. 可以通过dmcam_log_cfg()设置dmcam层打印日志信息的级别
3. 利用dmcam_dev_list()的返回值获取设备数量
4. 如果有设备连接上，通过dmcam_dev_open()打开指定的设备
5. 设置发光功率
```
      wparam.param_id = PARAM_ILLUM_POWER;	//设置参数为功率
      wparam.param_val_len = 1;
      wparam.param_val.raw[0] = 30;			//功率大小为30
      assert(dmcam_param_batch_set(dev, &wparam, 1));	//调用dmcam_param_batch_set()函数来设置
```
6. 调用dmcam_cap_set_frame_buffer()设置frame buffer，这里是设为自动分配
7. 利用dmcam_cap_set_callback_on_error()设置采样出错时的回掉函数
8. 设置好后通过dmcam_cap_start()开始采样
9. 采样开始后利用dmcam_cap_get_frames获取采样数据
10. 采样数据完成后利用dmcam_cap_stop()停止采样
11. 可以使用dmcam_cap_snapshot()函数使用快照功能，如果设备在采样时会获取最新的帧数据，如果没有开启则会自动开启，然后采集再自动关闭。
12. 最后通过dmcam_dev_close()来关闭设备，调用dmcam_uninit()后退出。
***
- sample_set_param.c  展示设置和获取设备信息
1. 参照sample_capture_frames.c进行初始化,日志输出配置，设备打开
2. test_cfg_mod_freq()样例展示怎么设置和验证设置的参数是否正确，这里通过设置预先设置一组模组频率值，调用dmcam_param_batch_set()来设置一个频率值
3. 再通过dmcam_param_batch_get()来读取频率设置的频率值，判断跟预先的设置值是否相等。
4. 模组的参数如功率、设备信息等都可以通过调用dmcam_param_batch_set()和再通过dmcam_param_batch_get()来读取和设置
5. 最后通过dmcam_dev_close()来关闭设备，调用dmcam_uninit()后退出。
***
- sample_filter.c  展示相机校准相关的功能
1. 参照sample_capture_frames.c进行初始化,日志输出配置，设备打开

2. 调用dmcam_filter_enable来使能或者失能
~~~
 	dmcam_filter_args_u witem;
	witem.lens_id = 1;		//1是镜头校准 2是工厂校准 3是用户校准
	assert(dmcam_filter_enable(dev,DMCAM_FILTER_ID_LEN_CALIB, 			&witem,sizeof(dmcam_filter_args_u))); //
~~~
3. 最后通过dmcam_dev_close()来关闭设备，调用dmcam_uninit()后退出。
