---
layout: post
title:  "UART and TTY"
date:   2016-08-24 16:22:37 +0800
categories: IT
---

# 串行接口描述 #

串行接口传送数据相比并行接口有很多优势

- 减少连接线的数量
- 因为端口减少了所以IC的PAD面积减小，可以大量降低芯片的成本
- 减少了传输线的数目可以大幅度降低功耗
- 串行信号更容易采取优化措施，比如屏蔽以及采用LVDS信号等


UART：通用异步串行通信

UART是一种异步，采用NZ编码的通信方式

## UART的信号逻辑 ##
在芯片间使用的UART信号通常都是TTL电平或者与TLL电平兼容的，也有设备使用1.8V 电平的标准。我们介绍信号逻辑时只分为逻辑”0”和逻辑”1”, 各种电平的判断可以参考相应的标准。

UART采用不归零电平编码，常见的波特率有以下几种

300, 600， 900， 1200， 2400， 3600， 4800， 9600， 19200， 38400， 57600， 115200， 230400， 460800， 921600， 1M, 1.5M, 2M, 4M

UART数据线在没有信号传输时保持高电平状态，传输数据有一个低电平的起始位，根据配置不同可以有7位或者8位数据。可以有校验位也可以没有校验位。（UART只有校验没有纠错，所以只能检测错误不能纠正。并且在UART规格里并没有错误重发的机制，所以UART只能作为可靠性不高的物理层通信标准，如果需要高可靠性的通信，需要在UART上再使用一套支持错误重发机制的协议，比如Slip协议等）

一个典型的UART数据结构如下（传输的数据为0x55)

![UART信号](pics/uart/signal.png)

**注意UART传输数据是地位在前(LSB),这点与I2C等不同**

## UART的比特率误差 ##
因为UART不传输串口，所以发送端与接收端都是用自己的时钟来同步信号。两个系统的时钟不可避免有误差，UART每个字节都通过起始位同步，理论上误差不超过5%就可以正常通信。但是实际传输过程中信号有畸变，所以对时钟精度的要求会更高一些。

## RS-232-C规范 ##
计算机与计算机或计算机与终端之间的数据传送可以采用串行通讯和并行通讯二种方式。由于串行通讯方式具有使用线路少、成本低，特别是在远程传输时，避免了多条线路特性的不一致而被广泛采用。

在串行通讯时，要求通讯双方都采用一个标准接口，使不同的设备可以方便地连接起来进行通讯。 RS-232-C接口（又称 EIA RS-232-C）是目前最常用的一种串行通讯接口。它是在1970年由美国电子工业协会（EIA）联合贝尔系统、调制解调器厂家及计算机终端生产厂家共同制定的用于串行通讯的标准。它的全名是“数据终端设备（DTE）和数据通讯设备（DCE）之间串行二进制数据交换接口技术标准”该标准规定采用一个25个脚的 DB25连接器，对连接器的每个引脚的信号内容加以规定，还对各种信号的电平加以规定。

###接口的信号内容###
实际上RS-232-C的25条引线中有许多是很少使用的，在计算机与终端通讯中一般只使用3-9条引线。RS-232-C最常用的9条引线的信号内容见附表

###接口的电气特性###
在RS-232-C中任何一条信号线的电压均为负逻辑关系。即：逻 辑“1”，-5— -15V；逻辑“0” +5— +15V 。噪声容限为2V。即 要求接收器能识别低至+3V的信号作为逻辑“0”，高到-3V的信号 作为逻辑“1” 

![](https://github.com/renesas/diary/raw/master/train_doc/uart_train/table.png)

###接口的物理结构###
RS-232-C接口连接器一般使用型号为DB-25的25芯插头座,通常插头在DCE端,插座在DTE端. 一些设备与PC机连接的RS-232-C接口,因为不使用对方的传送控制信号,只需三条接口线,即“发送数据”、“接收数据”和“信号地”。所以采用DB-9的9芯插头座，传输线采用屏蔽双绞线。
DCE与DTE的连接线是直通线，一端为针一端为孔。但是我们在调试中经常用到的传输线是用于两个计算机设备间的数据传输的，两头都是孔，线序是交叉线。

![](https://github.com/renesas/diary/raw/master/train_doc/uart_train/rtl.png)

###串口的应用###
串口一般用于两个系统间的通信，比如很多芯片的开发板就是通过串口与PC连接的。 串口也用于系统内部设备间的通信，最常见使用UART通信的设备是GPS和蓝牙模块。 另外比较老的Modem （GPRS或者Edge）也使用串口与CPU通信。

#终端#
在Linux中，很多进程都会绑定一个终端，进程组的成员会继承组长的终端常见的终端设备包括：

- 控制台终端: /dev/tty*
- 串口设备 /dev/ttyS*
- 伪终端设备 /dev/pts/* (不同的发行版下面可能是/dev/ptsp*等）
- 其他终端， 比如USB串口(/dev/ttyUSB*), ISDN模拟终端(/dev/ttyin)等

tty一词源于teletypes，或者teletypewriters，原来指的是电传打字机，是通过串行线用打印机键盘通过阅读和发送信息的东西，后来这东西被键盘与显示器取代，所以现在一般都叫做终端。

##串口终端##
Linux串口设备是一个标准的字符设备，一般的名称是/dev/ttyS*, 在嵌入式系统上也有不同的设备名称（比如FreeScale是ttyMXCx, Renesas是ttySCx)

串行端口终端(Serial Port Terminal)是使用计算机串行端口连接的终端设备。计算机把每个串行端口都看作是一个字符设备。有段时间这些串行端口设备通常被称为终端设备，因为 那时它的最大用途就是用来连接终端。这些串行端口所对应的设备是串口设备比如/dev/ttyS0或 /dev/ttyS1等，设备号分别是(4,0), (4,1)等。 串口设备不作为终端时，访问串口可以使用特定的程序比如minicom, ckermit等。

下面是在不同的设备上看到的终端的例子

PC上

	[root@Kendo ~]# ls -l /dev/ttyS*
	crw-rw---- 1 root uucp 4, 64 Jan  8 13:39 /dev/ttyS0
    crw-rw---- 1 root uucp 4, 65 Jan  8 13:39 /dev/ttyS1
    crw-rw---- 1 root uucp 4, 66 Jan  8 13:39 /dev/ttyS2
    crw-rw---- 1 root uucp 4, 67 Jan  8 13:39 /dev/ttyS3

R-Car

    # ls -lh /dev/ttyS*
    crw-------1 root root  204,   8 Jan  1 00:00 /dev/ttySC0
    crw-------1 root root  204,  13 Jan  1 00:00 /dev/ttySC5
    crw-------1 root root  204,  14 Jan  1 00:00 /dev/ttySC6

##控制台终端##
在Linux系统中，计算机显示器通常被称为控制台终端（Console,注意与后面提到的Console不同）。它仿真了类型为Linux的一种终端（TERM=Linux），并且有一些设备特殊文件（内核驱动能够）与之相关联比如：tty0、tty1、tty2……。

当用户从控制台上登录时，使用的是tty1。使用Alt+[F1—F6]组合键时，我们就可以切换到tty2、tty3……上面去（在图形界面切换到字符终端需要使用组合键Ctrl-Alt-Fn, 在大部分发行版中，1~6是字符终端，7是图形终端）。tty1 –tty6等称为虚拟终端，而tty0则是当前所使用虚拟终端的一个别名，系统所产生的信息会发送到该终端上。因此不管当前正在使用哪个虚拟终端，系统信息都会发送到控制台终端上。用户可以登录到不同的虚拟终端上去，因而可以让系统同时有几个不同的会话期存在。只有系统或超级用户root可以向/dev/tty0进行写操作。

控制终端（/dev/tty） 这是个在应用程序中的一个概念，前台进程有个控制终端，就对应这个。不过它并不指任何物理意义上的终端，/dev/tty会映射到当前的设备（通过 tty命令可以看到），如果在控制台界面下(即字符界面下）那么dev/tty就是映射到dev/tty1-6之间的一个（取决于你当前的控制台号），但是如果在是在图形界面（Xwindows），那么/dev/tty映射到/dev/pts的伪终端上。

tty命令可以用于查看当前进程连接的终端名称

	liuxu@Ubuntu-PC ~> tty
	/dev/tty1

##伪终端##
伪终端（/dev/pts/）是一种特殊的虚拟设备，用于支持网络，图形界面登录等要求。

伪终端（Pseudo Terminal）分为“伪终端主设备(/dev/ptyMN)”和“伪终端从设备”。(/dev/ttyMN)。比较新的内核版本采用了另外一种方式，使用一个主设备，动态生成从设备。 主设备设备名是/dev/pts/ptmx，从设备的名称是/dev/pts/*。它们与实际物理设备并不直接相关。如果一个程序把/dev/pts/3看作是一个串行端口设备，则它对该端口的读/写操作会反映在该逻辑终端设备对的另一个上面（/dev/pts/ptmx的某个文件句柄）这时打开主设备的进程就可以通过读写ptmx设备与打开从设备的进程交互。这样，两个程序就可以通过这种逻辑设备进行互相交流，而其中使用从设备的程序则认为自己正在与一个串行端口进行通信。这很象是逻辑设备对之间的管道操作。

对于设备，任何设计成使用一个串行端口设备的程序都可以使用伪终端的从设备。但对于使用主设备的程序，则需要专门设计来使用伪终端。

    liuxu@Ubuntu-PC ~/b/E/I/Kernel> ls -l /dev/pts/
    total 0
    crw--w---- 1 liuxu tty  136, 0 Aug  5 12:06 0
    crw--w---- 1 liuxu tty  136, 1 Aug  2 21:33 1
    crw--w---- 1 liuxu tty  136, 2 Aug  5 06:05 2
    crw--w---- 1 liuxu tty  136, 3 Aug  3 14:08 3
    crw--w---- 1 liuxu tty  136, 4 Aug  3 14:08 4
    c--------- 1 root  root   5, 2 Jul 26 13:01 ptmx

例如，如果某人在网上使用telnet程序连接到你的计算机上，则telnet程序就可能会开始连接到设备/dev/pts/ptmx上（一个伪终端端口上）并且打开一个对应于/dev/pts/2设备的句柄(m2)。此时一个getty程序就应该运行在对应的/dev/pts/2（s2）端口上。当telnet从远端获取了一个字符时，该字符就会通过m2、s2传递给getty程序，而getty程序就会通过s2、m2和telnet程序往网络上返回”login:”字符串信息。这样，登录程序与telnet程序就通过“伪终端”进行通信。通过使用适当的软件，就可以把两个甚至多个伪终端设备连接到同一个物理串行端口上。

任何写入到伪终端主设备的输入，都会作为伪终端从设备的输入，反之亦然。类似于管道，如下图：

![](https://github.com/renesas/diary/raw/master/train_doc/uart_train/flow.png)

这张图的关键在于：如果把伪终端从设备想像为传统的终端设备，把主设备看成进程读写数据的一个“接口”，那么它的工作原理，就跟传统终端一样了。

上述只是一个本地进程，把网络引入进来，对应到telnetd上面来，应该是下面这个样子：

![](https://github.com/renesas/diary/raw/master/train_doc/uart_train/telnet.png)


同样的登录方式，就变成了这样：

1. 如果某人在网上使用telnet程序连接到本地服务器，则telnetd程序就可能会开始连接到设备/dev/pts/ptmx（m2）主设备上，并且打开一个句柄，同时生成/dev/pts/2从设备。
2. telnetd产生一个子进程，运行getty程序，其打开一个对应的从设备对应的/dev/pts/2（s2），并设置stdin\stdout\stderr
3. telnetd通过内核tcp/ip协议栈从远端获取了一个字符时，该字符就会通过m2、s2传递给getty程序，而getty程序就会通过s2、m2和telnetd程序往网络上返回”login:”字符串信息
4. 这样，登录程序与telnetd程序就通过“伪终端”进行通信

伪终端有两种设备命名方式，一种是传统的BSD格式，还有一种是UNIX98_PTYS （来源与SVR4)。上面的介绍都是基于UNIX98_PTYS的定义。 BSD格式中的不同点在于，主设备与从设备是成对出现的，并不动态生成。所以打开设备时需要查找空闲的pty设备。

Linux内核中对于UNIX98_PTYS的解释是这样的：

	  Linux has traditionally used the BSD-like names /dev/ptyxx 
	for masters and /dev/ttyxx for slaves of pseudo terminals. 
	This scheme has a number of problems. The GNU C library 
	glibc 2.1 and later, however, supports the Unix98 naming 
	standard: in order to acquire a pseudo terminal, a process 
	opens /dev/ptmx; the number of the pseudo terminal is then 
	made available to the process and the pseudo terminal slave 
	can be accessed as /dev/pts/<number>. What was traditionally 
	/dev/ttyp2 will then be /dev/pts/2, for example.
	  All modern Linux systems use the Unix98 ptys.  Say Y unless
	you're on an embedded system and want to conserve memory.

关于伪终端的更多用法，可以参考《Advanced Programming under Unix Environment》第19章
##Console##
console是一个缓冲的概念，其实是为内核提供打印的。我们的pc，终端常用的是显示 器和键盘构成，我们用户打印和内核打印都从这个终端反 映给用户。所以，这里，/dev/console是连接到/dev/tty0的，其实这里有2个概念，console和tty这2个咚咚，怎么实现，其实 console这个结构中有个device，这里其实就是tty0对应的一个虚拟终端设备。 如果，我们来个专门打印内核的设备（比如通过串口），我们把那个串口register_console，那么/dev/console就到这个串口设备 了。这时，内核打印就到这个串口设备了，而用户的打印还是和上面的/dev/tty相关，如果/dev/tty对应/dev/tty0,那么用户打印还在 窗口中出现。所以说/dev/console是用来外接控制台的。

在Console的数据结构中保存了Console使用的设备名称，可以在kernel命令行中通过console=来指定console使用的设备，如果不指定，使用的设备是/dev/tty0

嵌入式设备一般会在kernel命令行中通过console=参数指定console使用的设备。 例如在R-Car的开发板上，可以看到"console=ttySC0,115200"的设置。

#终端应用举例#
##Linux系统初始化##
（这里只介绍标准的System V init方式，Upstart不是这样的）

Linux系统启动后使用终端的流程设这样的

Init --(/etc/inittab) --> getty ----> login --(/etc/passwd)--> shell

下面详细介绍对终端操作的流程

###init进程是系统的1号进程，启动后读取inittab配置文件。###
配置文件中通常有类似于

	tty1::respawn:/sbin/getty 38400 tty1
	ttyS0::respawn:/sbin/getty 115200 ttyS0

这样的内容， 这个内容使init进程启动getty命令，并且传递设备名给getty

###getty程序###
Getty命令初始化设备后，把这个设备设置为自己的终端，显示登录提示符并且等待输入用户名

当getty命令获取用户名后，通过exec调用运行login程序

###login程序###
login程序继承了getty的终端设置，并且要求用户输入密码（不回显）。校验密码成功后，login修改自身的权限为用户，并且根据/etc/passwd中的定义，通过exec调用shell
（注意，getty调用login以及login调用shell都是通过exec系统调用，保持PID不变，否则init会认为这个进程结束了并重启启动getty）

###退出###
当用户退出shell后，init检测到子进程结束，根据inittab中的定义，在终端上重启启动getty进程。

##图形界面下创建终端模拟器窗口##
本节重点介绍终端相关的操作，并不详细描述窗口管理等信息

窗口管理器启动一个终端模拟器命令（比如xterm) 并且为这个命令创建了一个窗口，这个进程可以在窗口中绘制文字。

Xerm进程打开/dev/pts/ptmx设备，这时ptmx驱动会在/dev/pts目录下创新一个新的从设备节点（比如是/dev/pts/1)

Xerm通过伪终端的调用获取从设备的设备名，并且fork出一个子进程，把子进程的终端设备设置为/dev/pts/1，并且修改标准输入输出设备为/dev/pts/1

子进程通过exec调用启动shell

在shell进程把/dev/pts/1作为一个串口终端使用，向终端的读写操作反应在Xerm主进程打开的/dev/pts/ptmx设备上，xerm进程把ptmx设备的输出显示在窗口中，并且把用户对窗口的输入发送到ptmx设备中。



