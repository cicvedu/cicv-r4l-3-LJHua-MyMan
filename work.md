# 作业1：编译Linux内核

**作业说明：**

进入Linux文件夹，使用如下命令编译：

```shell
make x86_64_defconfig
make LLVM=1 menuconfig
#set the following config to yes
General setup
        ---> [*] Rust support
make LLVM=1 -j$(nproc)
```

编译成功后会在Linux文件夹下得到一个vmlinux文件，如下图所示：

![](work/作业1.png)

# 作业2：对Linux内核进行一些配置

**Q: 在该文件夹中调用make LLVM=1，该文件夹内的代码将编译成一个内核模块。请结合你学到的知识，回答以下两个问题：**

1、编译成内核模块，是在哪个文件中以哪条语句定义的？

**答**：Linux内核使用的是```Kconfig/Kbuild```来配置和编译内核，以```obj-m += 模块名.o```语句定义。

2、该模块位于独立的文件夹内，却能编译成Linux内核模块，这叫做out-of-tree module，请分析它是如何与内核代码产生联系的？

 **答**：在编译时，使用了内核源码树中的`Kbuild`系统。   `Makefile`中会引用内核源码路径，使用如下命令进行编译：

```shell
# SPDX-License-Identifier: GPL-2.0

KDIR ?= ../linux

default:
	$(MAKE) -C $(KDIR) M=$$PWD
```

这条命令告诉`make`在指定的内核源码目录下进行编译，并将当前目录作为模块源代码目录。这样，尽管模块位于独立文件夹，仍能利用内核的构建系统进行编译。

**具体步骤及结果**：

1. 重新编译Linux内核，并禁用默认的C版本`e1000`网卡驱动
2. 启动脚本，打开`qemu`并安装内核模块，在未安装e1000驱动前，使用`ifconfig`命令结果为空

![](work/作业2-1.png)

3. 安装`e1000`驱动模块，并执行`ip link set eth0 up`连接对应网卡

![](work/作业2-2.png)

4. 执行`ip addr add 10.0.2.15/255.255.255.0 brd + dev eth0 `为网卡配置`ip`地址和广播地址；为网卡添加一条新的路由`ip route add default via 10.0.2.1`，之后`ping 10.0.2.2`结果如下：

![](work/作业2-3.png)

5. 执行`ifconfig`验证

![](work/作业2-4.png)

# 作业 3：使用 rust 编写一个简单的内核模块并运行

1. 进入Linux目录下的samples/rust文件夹，添加一个`rust_helloworld.rs`文件，并添加以下内容

![](work/作业3-1.png)

2. 在`linux/sample/rust/Kconfig`添加配置

![](work/作业3-2.png)

3. 在`linux/sample/rust/Makefile`添加配置

![](work/作业3-3.png)

4. 重新编译内核

![](work/作业3-4.png)

5. 将在`samples/rust`下看到一份`rust_helloworld.ko`的文件，将该文件复制到仓库中`src_e1000/rootfs`目录下，然后重新跑`build_image.sh`

![](work/作业3-6.png)

6. 进入内核，并执行`insmod rust_helloworld.ko`

![](work/作业3-5.png)

# 作业5：注册字符设备

**Q：作业5中的字符设备/dev/cicv是怎么创建的？它的设备号是多少？它是如何与我们写的字符设备驱动关联上的？**

**答**：通过命令`mknod /dev/cicv c 248 0`可以创建一个设备文件，其中参数c表示为字符设备，248为主设备号，0为次设备号，设备驱动通过系统动态分配设备号的方式关联。

1. 修改`Linux/samples/rust/rust_chrdev.rs`文件，具体如下：

![](work/作业5-1.png)

2. 重新编译内核，并如下更新配置

```shell
Kernel hacking
  ---> Sample Kernel code
      ---> Rust samples
              ---> <*>Character device (NEW)
```

3. 进入内核进行测试，测试结果如下：

![](work/作业5-2.png)

