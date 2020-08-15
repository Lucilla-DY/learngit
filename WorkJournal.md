ZYNQ学习入门操作工作手册（2020-7-20  first version）

NFS用于在不同机器不同操作系统之间通过网络互相分享文件的技术。由于它的目的就是在不同的系统间使用，所以它的通讯协议设计与主机和操作系统都无关。

NFS分成服务器和客户机。嵌入式开发时，常把Linux主机作为服务器，目标系统作为客户机使用。在目标系统上通过NFS直接将服务器的NFS共享目录挂载到本地，直接运行服务器上的文件，常常用于调试驱动模块以及应用程序。并且Linux还支持NFS根文件系统，能直接从远程NFS ROOT 启动系统。

### **2020-7-20 至  2020-7-22（周一至周三）**

#### 【阅读AX7Z100的 course_s1(第一至三章)以及参考AX7020的配套教程(第一至六章)】

​    **（一）第一章**    根据给出的软件包说明和开发检测迅速熟悉ZYNQ开发板的各功能部件，以及了解配套教程的使用方式。作为初学者，构建一个对ZYNQ的大体印象。

​    **（二） 第二章    提供对ZYNQ的一个简单介绍**

​    （1）ZYNQ包括两个组成部分PS和PL，PS和PL通过AXI总线协议互联。包括AXI-Lite，AXI4 和 AXI-Stream。

###### AXI4-Lite：

具有轻量级，结构简单的特点，适合小批量数据、简单控制场合。不支持批量传输，读写

时一次只能读写一个字（32bit）。主要用于访问一些低速外设和外设的控制。

###### AXI4：

接口和 AXI-Lite 差不多，只是增加了一项功能就是批量传输，可以连续对一片地址进行一次

性读写。也就是说具有数据读写的 burst 功能。

（上面两种均采用内存映射控制方式，即 ARM 将用户自定义 IP 编入某一地址进行访问，读 写时就像在读写自己的片内 RAM，编程也很方便，开发难度较低。代价就是资源占用过多，需要额外的读地址线、写地址线、读数据线、写数据线、写应答线这些信号线。）

###### AXI4-Stream：

是一种连续流接口，不需要地址线（很像 FIFO，一直读或一直写就行）。对于这类 IP，ARM不能通过上面的内存映射方式控制（FIFO 根本没有地址的概念），必须有一个转换装置，例如AXI-DMA 模块来实现内存映射到流式接口的转换。AXI-Stream 适用的场合有很多：视频流处理；通信协议转换；数字信号处理；无线通信等。其本质都是针对数值流构建的数据通路，从信源（例如 ARM 内存、DMA、无线接收前端等）到信宿（例如 HDMI 显示器、高速 AD 音频输出，等）构建起连续的数据流。这种接口适合做实时信号处理。

**（2）在ZYNQ芯片内部用硬件实现了AXI总线协议**。总共包括9个物理接口，分别为AXI-GP0~AXI-GP3，AXI-HP0~AXI-HP3，AXI-ACP 接口。

###### AXI_ACP 接口（1个）：

用来管理 DMA 之类的不带缓存的 AXI 外设，PS 端是 Slave 接口。

###### AXI_HP 接口（4个）：

是高性能/带宽的 AXI3.0 标准的接口，PL 模块作为主设备连接。主要用于 PL 访问 PS 上的存储器（DDR 和 On-Chip RAM）。

###### AXI_GP 接口（4个）：

是通用的 AXI 接口，包括两个 32 位主设备接口和两个 32 位从设备接口。

**（3）位于 PS 端的 ARM 直接有硬件支持 AXI 接口，而 PL 则需要使用逻辑实现相应的 AXI 协议。**

​     Xilinx 在 Vivado 开发环境里提供现成 IP 如 AXI-DMA，AXI-GPIO，AXI-Dataover, AXI-Stream 都实现了相应的接口，使用时直接从 Vivado 的 IP 列表中添加即可实现相应的功能。

​    大多数情况下，当多个外设需要互相交互数据时，需要加入一个 AXI Interconnect 模块（ AXI 互联矩阵），作用是提供将一个或多个 AXI 主设备连接到一个或多个 AXI 从设备的一种交换机制。ZYNQ 内部的 AXI 接口设备就是通过互联矩阵的的方式互联起来的，既保证了传输数据的高效性，又保证了连接的灵活性。Xilinx 在 Vivado 里我们提供的 IP 核叫axi_interconnect。

**（4）ZYNQ 芯片开发流程**

**（三）第三章   vivado开发环境介绍**

​    根据提供的vivado软件安装步骤，尝试在笔记本上安装vivado软件以进行后续学习，然障碍有三：

（1）Ubuntu是18.04版本的，高于手册的要求，如果操作过程中出现问题解决比较麻烦；

（2）其二，已经安装上的Ubuntu可用的硬盘容量不够（远小于手册上要求的120G)，而扩展硬盘的操作比较复杂，不作为首选的办法；

（3）第三，重新安装好符合要求的Ubuntu系统之后，始终没有找到方法使得客户端能自适应大小，并且连不上网络，这两点若未能解决会直接影响之后的学习，所以直接使用教研室电脑进行配套教程学习。

### 2020-7-23 至  2020-7-24（周四至周五）

#### 【阅读AX7Z100的 course_s1(第十六章）】

**第十六章    固化程序**

所谓固化程序，就是改变仅通过 JTAG 调试的方式，改用把程序放在 SD 卡或者烧写到 QSPI Flash 里运行。首先 PL 必须有 PS 配置，所以不能像以往的 FPGA 烧录方法直接烧录到 Flash。

###### 步骤简述：

vivado工程和启动SDK——生成FSBL——生成本工程的BOOT.bin文件——SD卡启动测试或者QSPI启动测试。

###### vivado工程和SDK部分：

新建vivado工程（目前只是实验程序的固化过程，所以使用哪个程序或程序需要哪些功能均不考虑，随意），导出硬件信息启动SDK。

###### 生成FSBL：

在SDK中新建一个名为 fsbl 的 APP，硬件平台选择最新的那个，模板选择 Zynq FSBL。在fsbl_debug.h文件中添加调试宏定义 FSBL_DEBUG_INFO，可以在启动输出 FSBL 的一些状态信息，有利于调试，但是会导致启动时间变长。SDK 默认会自动编译，生成 fsbl.elf 文件。

###### 生成本工程的BOOT.bin文件：

一句话来说就是把 FSBL 可执行文件（fsbl.elf ）、FPGA 的配置文件（即比特流文件）和 应用程序文件（.elf文件）结合成一个 Bin 文件。

fsbl工程右键->create boot image。弹出的窗口中从上到下可以依次看到生成的 BIF 文件路径（BIF 文件是生成 BOOT 文件的配置文件），“output path”一栏中显示待生成的 BOOT.bin 文件路径（BOOT.bin 文件就是我们需要的启动文件，可以放到 SD 卡启动，也可以烧写到 QSPI Flash），最下面是 Boot image partitions 对话框（在 Boot image partitions 列表中有要合成的文件）。

点击 Add 按钮， 在弹出的 Add new boot image partition 对话框中, 点击 Browse 找到fsbl.elf，这里 Partition type 为 bootloader，fsbl.elf 作为bootloader 程序。同样地，添加system_wrapper.bit和应用程序文件 vdma_test.elf,  Partition type 均为 datafile。

点击“create image”后在之前的生成路径下可以找到BOOT.bin文件。

###### SD卡启动测试：

#### 【阅读AX7Z100的 course_s1(第二十一至二十六章)】

（一）第二十一章   安装虚拟机和Ubuntu系统

（二）第二十二章    Ubuntu安装Linux版Vivado软件

（三）第二十三章    petalinux工具安装

（四）第二十四章    NFS网络服务软件安装

（五）第二十五章    使用petalinux定制Linux系统

使用 Petalinux 可以非常方便地定制嵌入式 Linux 系统，只需要 Vivado 软件把硬件信息导出（“../design_1_wrapper_hw_platform_0”目录就是 vivado 导出的硬件信息），然后 Petalinux 根据这些信息来配置 uboot，内核、文件系统等。

步骤：建立工程并配置——编译生成 bit 文件——导出硬件信息（在 vivado 的工程目录下会有一个*.sdk 的目录，下面有一个“*.hdf”文件，这个文件就包含了 petalinux 使用的文件）——把“design_1_wrapper_hw_platform_0”目录复制到 Linux 主机中——打开终端，进入工作目录——打开终端，进入工作目录——依次设置petalinux和vivado环境变量——新建petalinux工程ax_peta并进入该工程工作目录——配置硬件信息（petalinux-config --get-hw-description ../design_1_wrapper_hw_platform_0）——配置Linux内核——配置根文件——编译（编译uboot、内核、根文件系统、设备树等）——生成BOOT文件——测试Linux（SD卡启动测试）（将工程目录 images -> linux 目录中的 BOOT.BIN 和 image.ub 复制到 sd 卡，复制前最好先格式化一下 sd 卡，然后插到开发板上，开发板设置到 sd 卡启动）（使用 root 登录，默认密码 root，ETH1（PS）插上网线后（路由器支持自动获取 IP），使用ifconfig 命令可以看到网络状态。ETH2（PL）目前还不能工作，在后续的章节中会通过修改设备树的方式让 PL 端以太网也能工作起来。）

**注意：**

（1）在生成BOOT文件时使用以下命令：petalinux-package --boot --fsbl ./images/linux/zynq_fsbl.elf --fpga ./images/linux/design_1_wrapper.bit --u-boot --force

包括三个部分即fsbl.elf、design_1_wrapper.bit、u-boot。

（2）Linux工作路径是确定的一个路径吗？为什么只要不是在peta_prj路径下就会在建立petalinux工程的最后一步出现错误？是任意一个peta_prj路径下都行吗？

解释：工作目录应该是petalinux安装的目录，即../peta_prj

（3）在二十五章使用Petalinux定制Linux系统时，对Linux内核、根文件系统等均进行了配置，然后petalinux软件只需使用一次命令就可编译uboot、内核、根文件系统、设备树，并将编译好的各文件(BOOT.bin、image.ub）放在新建的petalinux工程子目录下（peta_prj/ax_peta/images/linux）。

（4）在生成BOOT文件时，输入的命令应和给出的命令一致，截图中的命令中--uboot是错误的（缺少短横杠，正确的应该是u-boot）。

（六）第二十六章    使用SDK开发Linux程序 

#### 【阅读AX7Z100的 course_s1(第二十八、二十九章)】

**第二十八章**     **Petalinux** **下的** **HDMI** **显示**

**第二十九章**     **使用** **Debian 8** **桌面系统**

**通用步骤：**

（1）petalinux配置（包括三个部分：设置petalinux和vivado环境变量，新建petalinux工程、配置硬件信息）

（2）配置Linux内核

（3）编译测试petalinux工程（包括三个部分：编译、生成BOOT文件、测试（sd卡简单启动测试、制作sd卡文件系统）

注意：在二十五章使用Petalinux定制Linux系统时，对Linux内核、根文件系统等均进行了配置，然后petalinux软件只需使用一次命令就可编译uboot、内核、根文件系统、设备树，并将编译好的各文件(BOOT.bin、image.ub）放在新建的petalinux工程子目录下（peta_prj/ax_peta/images/linux）。

**手动制作SD卡文件系统（不采用petalinux）的详细步骤（内容整理自AX7020第十八、十九、二十章）：**

###### 引入：

u-boot 是德国 DENX 小组开发的用于多种嵌入式 CPU 的bootloader 程序，在 Zynq7000 系统中，u-boot 主要用于引导Ubuntu 操作系统。

开发平台 u-boot 跑起来后，想进入 Linux，则需要三大法宝：

 Linux image (uImag or zImage) 

 Device tree image(arm linux3.0 以后需要 device tree)

 文件系统 (Linux app 都是存储在文件系统里的)

###### 过程：

（1）u-boot编译得到BOOT.bin文件

在第 16 章"程序固化和启动"里已经介绍过应用程序的 BOOT.BIN 文件的生成，而此时应用程序变成了 u-boot 程序。此时 BOOT.BIN 文件包括：第一级启动代码 FSBL、硬件比特流文件和 u-boot 文件。

（2）内核编译得到UImag文件

（3）设备树编译

设备树的描述在.dts 文件(device tree source)里，它是一种 ASCII 文本格式，此文本格式非常人性化，适合人类的阅读习惯。基本上，在 ARM Linux 在，一个.dts 文件对应一个 ARM的 machine，一般放置在内核的 arch/arm/boot/dts/目录(目录下AX7020 开发板的设备树文件为为 AX7020.dts)。对.dts 设备树文件进行编译生成.dtb 格式的设备树文件。

（4）Ubuntu文件系统

有了 Image/uImage, 和 devicetree.dtb，还需要文件系统。Linux 内核启劢阶段，待所有的驱动和外设都初始化好了以后，最后要加载一种文件系统Linux 扄可以正常启动。ZYNQ7000 开发平台支持的三种类型文件系统中，本实验选用Ubuntu文件系统，linaro-precise-ubuntu-desktop-20121124-560.tar.gz。

（5）制作SD卡文件系统

要在 AX7010/AX7020 开发板上运行 Linux 操作系统，需要制作特殊格式的 SD 卡，把Ubuntu 操作系统所需要的文件均保存到 SD 卡中。

将SD卡进行重新分区（FAT、EXT），将BOOT.bin、UImg、devicetree.dtb拷贝到FAT扇区，将文件系统拷贝到EXT分区。再进行几步操作SD卡就制作好了。

把 SD 卡插入开发板的SD 卡插槽内。连接 USB 串口线，连接 HDMI 显示器，开发板上电后在 HDMI 显示器上会显示 Ubuntu 的操作系统的界面。

### 2020-7-27 至  2020-7-30（周一至周四）

#### 【阅读AX7020的配套教程(第七章)】

**AX7020第七章    单独使用FPGA部分点亮LED灯**

（1）新建vivado工程。

（2）由于不使用PS端，则不需要添加block design，对比使用PS的情况下block design右键选择“generate output products”生成HDL源文件（.v文件）和引脚约束文件（.xdc文件），此处需要使用Add sources手动生成.v文件和.xdc文件。点亮LED灯的程序在.v文件中编写。

（3）然后Run synthesis综合运行并生成网表文件，在弹出窗口中选择Run implementation开始布局布线。

（4）PC连接JTAG，开发板上电，点击“Open hardware manager”图标选择Program device，最后点击Program进行程序的烧写，此时观察LED灯的亮灭情况。

#### 【阅读AX7Z100的 course_s1(第九章)】

**AX7Z100第九章    单独使用ARM即PS端进行裸机输出“Hello World”**

###### 步骤简述：

 新建vivado工程——对ZYNQ进行配置——硬件导入SDK——SDK中进行软件编程（helloworld.c文件是选择模板之后自带的，此时不需修改，只用其输出helloworld功能即可）——下载和调试。

###### 补充“下载和调试过程”如下：

（1）连接UART串口和JTAG，开发板上电，使用sudo putty打开串口终端调试工具putty，选择Serial项并配置串口号为ttyUSB0，speed为115200（这些数据均可以使用minicom查询本机串口信息）。

（2）右键helloworld文件->Run as->Launch on hardware。

（3）观察putty终端中输出hello world。

######  Windows下装虚拟机使用Ubuntu和Ubuntu单系统使用上有一些区别。

教程均使用虚拟机，而我所使用的是已经安装好的Ubuntu系统，目前在学习过程中出现的问题及解决如下：

（1）在进行course_1第九章时需连接uart串口进行调试，本机的串口号在不同操作系统上查看方式不同。Ubuntu单系统需要安装minicom软件进行查看和修改。名称是 ttyUSB0（最后一位为数字零）。

（2）开发板启动方式有三：SD，QSPI，JTAG。本实验中使用JTAG启动方式，需参考开发板的用户手册中各启动方式切换的方法，进行手动切换。

（3）使用不同的板子时切记要选择对应的教程，否则在版型选择，芯片配置，引脚使能和配置等等方面均有出入。（适时地查询不同zynq开发板的芯片手册，切记过于依赖教程的内容）

#### 【阅读AX7020的配套教程(第十章)】

**AX7020第十章     PS 点亮 PL 的 LED 灯**

###### 步骤简述：

 新建vivado工程——对ZYNQ进行配置——添加引脚约束文件（.xdc）——生成.bit文件——硬件导入SDK——SDK中进行软件编程（仍然是helloworld.c，是创建APP工程时选择模板之后自带的，此时需修改其内容实现点亮LED灯的功能）——下载和调试。

###### 补充“下载和调试过程”如下：

（1）连接JTAG，开发板上电。

（2）下载.bit文件到开发板上。Xilinx  tools(新版vivado改成Xilinx了）->Program FPGA->在Bitstream一栏中选择system_wrapper.bit文件->点击Program下载。

（3）右键pl_led_test工程文件->Run as->Launch on hardware。

（3）观察开发板LED灯点亮情况。

#### 【阅读AX7Z100的 course_s1(第十章)】

对比**AX7020**第十章，区别之处在于SDK编程部分。在.mss文件中找到“axi_gpio_0”，点击“Import Examples”（选第一个“xgpio_example”）直接用Xilinx提供的例程来修改。（已经产生的helloworld.c不需要管，之后Launch on hardware时选择导入的例程文件来运行）

### 2020-8-3 至  2020-8-5（周一至周三）

#### 【阅读AX7Z100的 course_s2(第七章)】

DMA（direct memory access）直接内存存取模块。外部设备可以不通过CPU就与系统内存交换数据的接口技术。一般来说是CPU采用查询和中断的方式来控制外设和内存的数据传输过程，如之前的BRAM实验。CPU只需要提供地址和长度给DMA，DMA就可接管总线来访问内存，结束之后再交出总线控制权。

#### 【阅读AX7020 (第十四、十五章)】

注意：

（1）打开配套程序给出的vivado工程文件后,生成比特流文件时出现以下错误,则表示变量名字和约束文件里的引脚名字不匹配.(很多时候是由于直接拷贝教程的约束文件内容,从而导致自己工程中引脚名字和约束文件不符)(可以从.v文件中查看接口名字)

ERROR: [DRC 23-20] Rule violation (NSTD-1) Unspecified I/O Standard - 4 out of 134 logical ports use I/O standard .......... implementation run. Problem ports: led4b_tri_o[3:0]. 

ERROR: [DRC 23-20] Rule violation (UCIO-1) Unconstrained Logical Port - 4 out of 134 logical ports have no user assigned specific location constraint (LOC). This may cause I/O contention or incompatibility with ......... implementation run.  Problem ports: led4b_tri_o[3:0].
ERROR: [Vivado 12-1345] Error(s) found during DRC. Bitgen not run.

(2)在实现第十五章部分时,进行到最后一步,终止的原因是,笔记本电脑的HDMI只作为输出,即只可以将电脑的显示屏输出到其他设备如电视上,而不能连接开发板的HDMI接口接收数据.因此本实验成功的前提是,找到一台HDMI显示器!!!!!!!

### 2020-8-10 （周一）

#### 【阅读AX7020 (第十八、十九、二十章)】u-boot的编译和启动，内核编译，文件系统，设备树，SD卡制作

（1）在下载/u-boot-xlnx-master目录下，经过u-boot编译（make CROSS_COMPILE=arm-xilinx-linux-gnueabi- zynq_ax7020_defconfig"命令）后可以找到u-boot.bin、u-boot.srec文件和u-boot.elf文件，u-boot.elf则是用来制作最后启动的BOOT文件。

（2）fsbl文件是通过SDK生成的。名称写fsbl，选择ZYNQ FSBL模板即可。在 fsbl_debug.h 文件里添加一条语句，定义一下“FSBL_DEBUG_INFO”常量。修改后保存，重新编译一下 fsbl 项目。

（3）实验中使用的vivado过程文件是我自己利用配套资源08_vdma_test复制成linux_hw保存在/asd目录下的，在重新编译的过程中出现不能生成比特流文件的情况，解决无果，于是直接采用配套资源10_linux_hw来继续进行实验。

（4）生成 BOOT.BIN 文件的方法跟前面"程序固化和启动"那章内容一样，接下去我们要把FSBL 可执行文件，FPGA 的比特流文件和 u-boot 结合成一个 BOOT.BIN 文件。

1. 这里需要先把 Ubuntu 操作系统中生成的 u-boot（步骤（1）生成的那个 elf 文件)拷贝到 Vivado 的工程目录下（AX7020/...source/...10_linux_hw），并添加后缀为 u-boot.elf。 

2. 在 SDK 里选择菜单 Xilinx Tools->Create Boot Image

3. 选择 Zynq Boot Image 文件.bif 的存放地址（默认为vivado过程文件下，xxxx.sdk / fsbl / bootimage / BOOT.bin），然后在 Boot image partitions 里添加 3 个文件：分别是 fsbl.elf、system_wrapper.bit 和 u-boot.elf。点击 Create Image 挄钮，生成 BOOT.BIN。

（5）连接串口，开发板启动方式改成SD卡启动，复制BOOT.BIN到SD卡（复制到扇区FAT），上电后，putty串口终端成功显示开发板启动过程（也可以直接用Minicom软件，根本不需要配置串口，更加方便）。

（6）内核编译完成后，uImage.ub文件在/下载/kernel/linux/arch/arm/boot目录下。

（7）在 ARM Linux 在，一个.dts 文件对应一个 ARM的 machine，一般放置在内核的arch/arm/boot/dts/ 目录(AX7020 开发板为 AX7020.dts)。我们需要对.dts 设备树文件进行编译生成.dtb 格式的设备树文件。而设备树文件devecetree.dtb在之前u-boot编译的时候就已经生成了，位置在下载/kernel/linux目录下。

（8）把 Downloads 目录下的文件系统拷贝到 SD 卡的 EXT 分区时，直接复制粘贴不了，于是采用右键文件系统->复制到->选择EXT区，复制成功。

（9）在命令窗口输入"rsync -av ./ /media/alinx/EXT"，开始同步当前目录（即解压后的文件系统目录，内部包含几十个需要的文件）下 SD 卡的 EXT 分区根目录，同步可能需要十几分钟的时间。此过程中的命令改为rsync -av ./ /media/asd/EXT。

（10）开发板连接串口，HDMI显示器，上电。可以在串口终端看到Ubuntu系统启动过程，并且在显示器上显示了一个简化版的Ubuntu操作系统界面，连接鼠标到开发板USB口上，即可像PC上一样进行操作。

#### 【阅读AX7020 (第二十一、二十二章)】Linux系统下的helloworld实验

（1）在/home/asd/work/test目录下新建.c文件时，使用vim helloworld.c命令，提示vim包含在下列软件包中，尝试apt install <选定的软件包>，此时选择vim软件包进行下载。

（2）NFS网络服务器安装和配置

1.sudo apt-get install nfs-kernel-server安装NFS

2.创建 work 文件夹，用于目标机挂载。

3.输入命令"/etc/init.d/nfs-kernel-server start"启动 nfs server。

4.输入"showmount -e"命令可以查看  work  路径。

5.输入命令"sudo mount -t nfs localhost:/home/asd/work    /mnt", 把  work  目录下的内容同步到/mnt 目录下。（localhost也可以用网络IP地址代替）

6.开发板同步。连接开发板Ubuntu系统启动后，确认网络连接正确（如ping  baidu.com）后在串口终端输入命令"apt-get install nfs-common"安装 nfs系统文件。

7.安装后，输入命令 mount  -t  nfs  192.168.2.190:/home/asd/work  /mnt   把虚拟机上的Ubuntu 操作系统中的 work目录同步到开发板上的/mnt目录下（ IP(192.168.1.27)是PC机 Ubuntu 操作系统的网络 IP 地址，可通过ifconfig  -a 查看）。

8.这时输入"ls /mnt"查看，可以看到目录下有一个 test_nfs.dat 文件了，说明虚拟机上的 work 目录下的文件系统已经通过网络映射到开发板的 /mnt 文件夹了。

（3）应用程序实现步骤如下：

1.首先，在本地work/test文件夹下，进入终端，输入vim  gpio.c，进而在编辑器里修改代码保存。

2.然后，"source /opt/Xilinx/SDK/2017.4/settings64.sh"定位到交叉编译器，用arm-linux-gnueabi-gcc gpio.c -o gpio -static 编译刚刚新增的.c文件。

3.开发板连接上电后，在串口终端minicom中输入命令 mount  -t  nfs  192.168.2.190:/home/asd/work  /mnt   把虚拟机上的Ubuntu 操作系统中的 work目录同步到开发板上的/mnt目录下。

4.之后，进入/mnt/test 目录，再输入 ls 查看，我们可以看前面编译的 gpio 可执行文件。

5.最后，输入./gpio 运行。若代码中有输出内容则在串口终端中显示。开发板上LED灯间隔一秒点亮。

### 2020-8-11至2020-8-12（周二至周三）

#### 【周老师嵌入式课件Section3部分】

目标板启动过程中，应用程序设计流程如下:

(1)编写Linux APP应用程序如hello.c文件

（2）使用GNU交叉编译工具来编译应用程序

（3）将编译好的文件下载到目标机开发板

（4）采用串口终端在目标机上运行程序，串口终端可打印信息

应用程序设计过程根据程序下载方式不同分为两种：

（1）TFTP方式：用串口线和网线连接宿主机和目标机，宿主机运行tftp服务器程序，配置服务器主目录，将应用程序复制到主目录中。在宿主机运行串口终端，配置目标机ip 地址为与宿主机同一网段的地址，并用ping命令检测是否连通  ifconfig eth0 192.168.2.190 以及 ping  192.168.2.190。最后，在串口终端中，使用tftp命令下载程序  tftp –g 宿主机IP –r ./程序名。

<img src="C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20200813094957626.png" alt="image-20200813094957626" style="zoom:67%;" />

（2）NFS方式：将宿主机的文件系统挂载到目标机文件系统下，即可通过串口终端在目标机直接访问宿主机文件系统中的应用程序，省去下载环节。

<img src="C:\Users\asus\AppData\Roaming\Typora\typora-user-images\image-20200813095144112.png" alt="image-20200813095144112" style="zoom:67%;" />

#### 【阅读AX7Z100的 course_s4(第一章)】

（一）QT Creator安装

在 Ubuntu 中安装 QT Creator，安装文件名称为“qt-opensource-linux-x64-5.7.1.run”（紫色菱形），从文件名我们可以看出 QT 版本为 5.7.1，安装目录为/opt/Qt5.7.1。

（二）QT交叉库编译

（1）为了使得前面安装的QT能够建立ZYNQ能运行的程序，需要对QT的源代码进行交叉编译后再在ZYNQ上运行。采用用xlinx提供的源代码包alinx_qt_5.7.1.tar.gz。

 （2）alinx_qt_5.7.1.tar.gz解压在AX7020/QT目录下，解压后的文件夹alinx_qt_5.7.1中包括几个部分：

**fonts：** QT 应用程序显示需要用到的字体文件。

**qt-everywhere-opensource-src-5.7.1：** QT 源代码文件夹。

**make_img.sh：**把编译的库打包成 img 镜像文件。

**qt_env_set.sh：**环境变量设置脚本。

**build.sh：**编译脚本，指定了 XSDK 的安装目录，主要使用 XSDK 自带的交叉编译器，同时指定 **ZYNQ 版本 QT 库**的安装路径为**“/opt/alinx/zynq_qt5.7.1”**，编译完成以后可以在这个路径里找到交叉编译过的 QT 库，不建议修改此路径。

（3）运行./build.sh 脚本。到目录“/opt/alinx/zynq_qt5.7.1”下我们可以找到 bin 目录（里面有我们要用到的 qmake文件）和 lib 目录（我们需要的 QT 库）。

（4）运行 ./make_img.sh。在当前目录下（AX7020/QT/alinx-qt_5.7.1）得到**qt_lib.img** 可以在 zynq 板上挂载。

（三）配置ZYNQ版QT Kits

（四）创建QT测试工程

经过一系列的新建blabla之后，选择 ZYNQ Kit，点击锤子图标，只编译程序，不运行。在 build- qt_test（工程名）-ZYNQ-Debug 目录可以看到生成了一个 qt_test （工程名）的文件，这个文件要在 ZYNQ上运行。

（五）运行 ZYNQ版本QT 程序

（1）准备好开发板，连接好显示器，将例程给的 BOOT.BIN，image.ub 复制到 sd 卡（sd_boot 目录）。接下来均在串口终端进行操作。

（2）挂载 Ubuntu 主机 NFS 服务。

（3）进入/mnt 目录，建立一个目录用于挂载 QT 库  mkdir  /tmp/qt。

（4）mount **qt_lib.img**   /tmp/qt  挂载 QT 库。

（5）进入 qt 库目录 cd /tmp/qt，运行环境变量设置脚本 source ./qt_env_set.sh

（6）进入/mnt 目录，再进入 build-qt_test-ZYNQ-Debug 目录。

（7）运行 ZYNQ 版本 QT 测试  **./qt_test**。

### 2020-8-13 （周四）

#### 【Windows下安装Ubuntu系统】

https://www.cnblogs.com/Duane/p/6776302.html

首先，找到一个空的U盘，将.ios映像文件拷贝到U盘中（如果U盘中本来有其他文件，则安装时会自动将之覆盖掉即文件丢失）。插入U盘，在虚拟机Ubuntu下查找启动盘创建器，自动识别出Ubuntu16.04的映像文件，点击“制作启动盘”完成。重启Windows迅速按住F2，选择blablabla后进入Ubuntu安装界面，选择中文，在安装类型界面选择“其他选项”进行手动分区，最后输入用户名密码那些，搞定。

#### 【尝试zynq版Qt编程第一章】

整个下午陷入了ZYNQ下锤子图标编译出现“找不到IGLESv2"的沼泽之中，泪奔ing........

#### 【大致浏览course_4的所有章节】

看过一遍后，发现重点还是第一章ZYNQ版Qt的安装。其他的blablabla没什么用。

### 2020-8-14（周五）

仍然陷入了ZYNQ下锤子图标编译出现“找不到IGLESv2"的沼泽之中，泪奔ing........根据学长的提示重新对套件、Qt版本、编译器进行配置，仍然报错，痛苦ing............煎熬ing............没有实际进展的一天................

## 接下来的安排（持续更新删改中）：

1. 继续微信读书的操作系统哲学原理部分，最好是赶紧结束，然后再拉一遍，开始手写笔记。
2. 查看涛哥学Linux。
3. 看菜鸟教程的C++部分，各个概念要搞清楚，最好是手写笔记并归纳和口头复述。
4. 周老师的课件第三章部分，对比ZYNQ教程查看具体过程有哪些重合部分和不同的部分。
6. 第二十五章剩余最后Linux测试部分，留待XCZ7100板子来试验。截止8-10，仍然不行，所以采取020的方法来测试。