<callout icon="😀" color="gray_bg">
	Hi，这是我学习FreeRTOS的笔记。内容为视频：韦东山应用篇
</callout>
# 01.创建项目模板 {toggle="true"}
	## 一、项目基础配置 {toggle="true"}
		1. 选用stm32F103C8T6芯片
		2. 设置Rcc  高速时钟为 陶瓷晶振
		3. 设置SYS  Debug为 Serial Wire、基准时钟为TIM4
		4. 设置HCLK为72M 
		5. 找到Middleware and Software Packs（中间件和软件包）、选择FreeRTOS、选择CMSIS V2、配置参数保持默认
		6. 设置工程名、配置IDE为MDK-ARM
		7. 在Code Generator中设置每个外设的初始代码的.c  .h 分开（Generate peripheral initialization as a pair of'c/.h' files per periphera） 、其他默认
		8. 固件包版本设置为1.85  （为了防止 在编译时出现这个错误 #include CMSIS_device_header ）
		9. 生成代码
		10. 选择下载器
		11. 设置复位后运行
		12. 关闭调试说明
		<empty-block/>
	## 二、添加基础程序 {toggle="true"}
		1. 配置pc13  板载led测试程序
			- 在CubeMX设置PC13为推挽输出。
			- 添加led测试程序.c .h文件之后 在freertos.c中的`void StartDefaultTask(void *argument)` 函数中添加测试程序
		2. 配置屏幕
			- 在Connectivity中设置I2C1、使能为I2C、参数默认
			- 添加lcd、oled的.c .h文件  添加ascii_font.c字库文件
	<empty-block/>
# 02.创建第一个多任务程序 {toggle="true"}
	cmsis.os2.c是一个统一的接口。
	因为有很多的操作系统。他们的函数又不同。
	但有一个统一的接口，就会根据底层的操作系统的不同调用不同的函数来执行操作。
	而作为用户的我们，只需要知道cmsis.os2.c中的函数就可以了。
	并且我们写出的函数就能即运行在这个操作系统中，又可以运行在别的操作系统中
	<empty-block/>
	`xTaskCreate`是 FreeRTOS 中用于动态创建任务的函数
	各参数的含义如下：
	1. `pvTaskCode`：指向任务入口函数的指针。
	2. `pcName`：任务的描述性名称。
	3. `uxStackDepth`：任务堆栈的大小（单位是字，而不是字节）。
	4. `pvParameters`：传递给创建任务的参数。
	5. `uxPriority`：创建的任务将运行的优先级。数字越大，优先级越高。
	6. `pxCreatedTask`：用于返回已创建任务的句柄。
	<empty-block/>
	在指定的地方写好函数原型之后，把函数原型丢到`xTaskCreate` 中，填写好相应参数即可，比如
	`xTaskCreate(My_Task, "MyFirstTask", 128, NULL, osPriorityNormal, NULL); ` 
	这里两个NULL为空指针，暂时不做了解
# 03.硬件架构和汇编指令 {toggle="true"}
	## 一、硬件架构 {toggle="true"}
		ARM芯片属于精简指令集计算机(RISC:Reduced Instruction Set Computing)，
		它所用的指令比较简单，有如下特点:<br>① 对内存只有读、写指令<br>② 对于数据的运算是在CPU内部实现<br>③ 使用RISC指令的CPU复杂度小一点，易于设计
		<empty-block/>
		CPU内部有寄存器 R0-R15。以及计算单元.   
		后三个为程序状态寄存器：
		- R13:别名SP(Stack Pointer)，栈指针。用来保存栈的地址<br>R14:别名LR(Link Register)，用来保存返回地址。（A函数到B函数，B函数完成后返回A）<br>R15:别名PC(Program Counter)，程序计数器，表示当前指令地址，写入新值即可跳转
	## 二、一些简单的汇编指令 {toggle="true"}
		1. 读内存：Load
			`LDR R0， [R1, #4] ;`  读地址 ”R1+4 “ ，得到的四个字节数据存入R0
		2. 写内存：Stroe
			`STR R0， [R1， #4] ;` 把R0的四个字节写入到”R1+4“中去
		可以在LDR或STR后加H 、 B 来表示读写 半字（Half） 字节（Byte）
		1. 加
			`ADD R0， R1, R2 ; `R0 = R1 + R2
			`ADD R0， R1, #1 ; `R0 = R1 + 1
		2. 减
			`SUB R0， R1, R2 ; `R0 = R1 - R2
			`SUM R0， R1, #1 ; `R0 = R1 - 1
		3. 比较
			CMP R0, R1 ;比较R0和R1，比较的结果保存在程序状态寄存器中 PSR
		4. 跳转
			- `B  ：main ;`  Branch,   直接跳转
				执行这条指令，会导致程序状态寄存器R15（PC）寄存器中被已写入一个数值。数值为main函数的地址。CPU会从这个地址开始执行程序
			- BL：main ;     Branch  and  Link,  在函数之间的跳转时，先把返回值保存在R14（LR）寄存器里再跳转
# **04. 堆与栈** {toggle="true"}
	## **一、堆** {toggle="true"}
		堆是一块动态分配的内存空间。
		- **特点**：
			- 分配和释放由程序员手动控制，相对灵活。可以在运行时根据实际需求从中分配出不同大小的内存块，例如可以使用编程语言提供的内存分配函数（如 C 语言中的 `malloc`、C++ 中的 `new` 等）来分配堆内存。当使用完这块内存后，需要显式地调用相应的释放函数（如 C 语言中的 `free`、C++ 中的 `delete`）把它放回去，否则会导致内存泄漏。
			- 内存分配和释放的时间开销相对较大，因为堆的管理通常涉及复杂的算法来寻找合适大小的空闲内存块。
			- 堆中的内存空间大小通常只受限于系统的可用物理内存和虚拟内存大小。
		- **用途**：
			- 适用于需要动态分配较大内存块且生命周期不确定的情况。比如存储大量数据结构（如链表、树等）、动态创建对象等。
	## **二、栈** {toggle="true"}
		栈是一块由系统自动管理的内存空间。
		- **特点**：
			- 先进后出（FILO）的数据结构。CPU 的栈指针寄存器（SP）始终指向栈顶。当函数被调用时，函数的参数、局部变量和返回地址等信息被压入栈中；当函数返回时，这些信息被弹出栈。
			- 内存的分配和释放由系统自动完成，速度快。在函数调用结束后，栈上的局部变量会自动被释放，无需程序员手动管理。
			- 栈的大小通常是有限的，不同的操作系统和编译器可能会设置不同的栈大小限制。如果栈空间被耗尽，可能会导致栈溢出错误。
		- **用途**：
			- 主要用于函数调用和局部变量的存储。在多任务系统中，栈还可以用于保存任务的现场，例如当一个任务被中断时，当前的寄存器值、程序计数器等信息会被压入栈中，以便在任务恢复执行时能够恢复到中断前的状态。
		<empty-block/>
	## 三、问题 {toggle="true"}
		1. 在函数调用时，LR被覆盖了怎么办？
			- 会 在C入口保存LR进栈。
		2. 局部变量在栈中是如何分配的？
			- 使用volatile 修饰的局部变量会保存在栈中。默认使用CPU寄存器来进行保存，如果寄存器不够用了才用栈
		3. 为什么每个ROTS任务都有自己的栈
			因为每个人物都有自己的调用关系和局部变量、所以在每个任务在切换出的时候要保存自己的现场
			- 保存现场
				- 在FreeRTOS运行时，内核的定时器中断函数会自动切换任务。
				- 并为需要为每个任务 保存现场，为恢复现场奠定基础
				- 保存现场当然要保存在RAM中的 这个任务分配的栈中。
				- 保存现场是保存所有的寄存器
				- 在任务A中的结构体里记录SP
			- 恢复现场
				- 找到A的结构体，得到A的栈。
				- 把栈中的值恢复到CPU中（把所有的寄存器的值恢复到硬件）
				<empty-block/>
		> R13:别名SP(Stack Pointer)，栈指针。用来保存栈的地址<br>R14:别名LR(Link Register)，用来保存返回地址。（A函数到B函数，B函数完成后返回A）<br>R15:别名PC(Program Counter)，程序计数器，表示当前指令地址，写入新值即可跳转
# 05.FreeRTOS源码描述 {toggle="true"}
	## **7.1 FreeRTOS目录结构**
	使用STM32CubeMX创建的FreeRTOS工程中，FreeRTOS相关的源码如下:
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image1.png)
	主要涉及2个目录：
	- Core
		- Inc目录下的FreeRTOSConfig.h是配置文件
		- Src目录下的freertos.c是STM32CubeMX创建的默认任务
	- Middlewares\\Third_Party\\FreeRTOS\\Source
		- 根目录下是核心文件，这些文件是通用的
		- portable目录下是移植时需要实现的文件
			- 目录名为：\[compiler\]/\[architecture\]
			- 比如：RVDS/ARM_CM3，这表示cortexM3架构在RVDS工具上的移植文件
	7.2核心文件 FreeRTOS的最核心文件只有2个：
	- FreeRTOS/Source/tasks.c
	- FreeRTOS/Source/list.c
		其他文件的作用也一起列表如下：
		![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image2.jpg)
	## [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-3-%E7%A7%BB%E6%A4%8D%E6%97%B6%E6%B6%89%E5%8F%8A%E7%9A%84%E6%96%87%E4%BB%B6)**7.3 移植时涉及的文件**
	移植FreeRTOS时涉及的文件放在 **FreeRTOS/Source/portable/\[compiler\]/\[architecture\]** 目录下，比如：RVDS/ARM_CM3，这表示cortexM3架构在RVDS或Keil工具上的移植文件。 里面有2个文件：
	- port.c
	- portmacro.h
	## [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-4-%E5%A4%B4%E6%96%87%E4%BB%B6%E7%9B%B8%E5%85%B3)**7.4 头文件相关**
	### [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-4-1-%E5%A4%B4%E6%96%87%E4%BB%B6%E7%9B%AE%E5%BD%95)**7.4.1 头文件目录**
	FreeRTOS需要3个头文件目录：
	- FreeRTOS本身的头文件：
	Middlewares\\Third_Party\\FreeRTOS\\Source\\include
	- 移植时用到的头文件：
	Middlewares\\Third_Party\\FreeRTOS\\Source\\portable\[compiler\]\[architecture\]
	- 含有配置文件FreeRTOSConfig.h的目录：Core\\Inc
	### [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-4-2-%E5%A4%B4%E6%96%87%E4%BB%B6)**7.4.2 头文件**
	列表如下：
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image3.jpg)
	## [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-5-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86)**7.5 内存管理**
	文件在Middlewares\\Third_Party\\FreeRTOS\\Source\\portable\\MemMang下，它也是放在“portable”目录下，表示你可以提供自己的函数。
	源码中默认提供了5个文件，对应内存管理的5种方法。
	后续章节会详细讲解。
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image4.jpg)
	## [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-6-%E5%85%A5%E5%8F%A3%E5%87%BD%E6%95%B0)**7.6 入口函数**
	在Core\\Src\\main.c的main函数里，初始化了FreeRTOS环境、创建了任务，然后启动调度器。源码如下：
	```c
/* Init scheduler */
  osKernelInitialize();  /* 初始化FreeRTOS运行环境 */
  MX_FREERTOS_Init();    /* 创建任务 */

  /* Start scheduler */
  osKernelStart();       /* 启动调度器 */

	```
	## [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-7-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E5%92%8C%E7%BC%96%E7%A8%8B%E8%A7%84%E8%8C%83)**7.7 数据类型和编程规范**
	### [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-7-1-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)**7.7.1 数据类型**
	每个移植的版本都含有自己的portmacro.h头文件，里面定义了2个数据类型：
	- TickType_t：
		- FreeRTOS配置了一个周期性的时钟中断：Tick Interrupt
		- 每发生一次中断，中断次数累加，这被称为tick count
		- tick count这个变量的类型就是TickType_t
		- TickType_t可以是16位的，也可以是32位的
		- FreeRTOSConfig.h中定义configUSE_16_BIT_TICKS时，TickType_t就是uint16_t
		- 否则TickType_t就是uint32_t
		- 对于32位架构，建议把TickType_t配置为uint32_t
	- BaseType_t：
		- 这是该架构最高效的数据类型
		- 32位架构中，它就是uint32_t
		- 16位架构中，它就是uint16_t
		- 8位架构中，它就是uint8_t
		- BaseType_t通常用作简单的返回值的类型，还有逻辑值，比如pdTRUE/pdFALSE
	### [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-7-2-%E5%8F%98%E9%87%8F%E5%90%8D)**7.7.2 变量名**
	变量名有前缀：
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image5.jpg)
	### [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-7-3-%E5%87%BD%E6%95%B0%E5%90%8D)**7.7.3 函数名**
	函数名的前缀有2部分：返回值类型、在哪个文件定义。
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image6.jpg)
	### [**#**](https://rtos.100ask.net/zh/freeRTOS/DShanMCU-F103/chapter7.html#_7-7-4-%E5%AE%8F%E7%9A%84%E5%90%8D)**7.7.4 宏的名**
	宏的名字是大小，可以添加小写的前缀。前缀是用来表示：宏在哪个文件中定义。
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image7.jpg)
	通用的宏定义如下：
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-7/image8.jpg)
# 06.内存管理 {toggle="true"}
	## **8.1 为什么要自己实现内存管理**
	后续的章节涉及这些内核对象：task、queue、semaphores和event group等。为了让FreeRTOS更容易使用，这些内核对象一般都是动态分配：用到时分配，不使用时释放。使用内存的动态管理功能，简化了程序设计：不再需要小心翼翼地提前规划各类对象，简化API函数的涉及，甚至可以减少内存的使用。
	内存的动态管理是C程序的知识范畴，并不属于FreeRTOS的知识范畴，但是它跟FreeRTOS关系是如此紧密，所以我们先讲解它。
	在C语言的库函数中，有mallc、free等函数，但是在FreeRTOS中，它们不适用：
	- 不适合用在资源紧缺的嵌入式系统中
	- 这些函数的实现过于复杂、占据的代码空间太大
	- 并非线程安全的(thread- safe)
	- 运行有不确定性：每次调用这些函数时花费的时间可能都不相同
	- 内存碎片化
	- 使用不同的编译器时，需要进行复杂的配置
	- 有时候难以调试
	注意：我们经常"堆栈"混合着说，其实它们不是同一个东西：
	- 堆，heap，就是一块空闲的内存，需要提供管理函数
		- malloc：从堆里划出一块空间给程序使用
		- free：用完后，再把它标记为"空闲"的，可以再次使用
	- 栈，stack，函数调用时局部变量保存在栈中，当前程序的环境也是保存在栈中
		- 可以从堆中分配一块空间用作栈
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-8/image1.png)
	## **8.2 FreeRTOS的5中内存管理方法**
	FreeRTOS中内存管理的接口函数为：pvPortMalloc 、vPortFree，对应于C库的malloc、free。 文件在FreeRTOS/Source/portable/MemMang下，它也是放在portable目录下，表示你可以提供自己的函数。
	源码中默认提供了5个文件，对应内存管理的5种方法。
	参考文章：[**FreeRTOS说明书吐血整理【适合新手+入门】在新窗口打开**](https://blog.csdn.net/qq_43212092/article/details/104845158)
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-8/image2.jpg)
	### **8.2.1 Heap_1**
	它只实现了pvPortMalloc，没有实现vPortFree。
	如果你的程序不需要删除内核对象，那么可以使用heap_1：
	- 实现最简单
	- 没有碎片问题
	- 一些要求非常严格的系统里，不允许使用动态内存，就可以使用heap_1
	它的实现原理很简单，首先定义一个大数组：
	```c
/* Allocate the memory for the heap. */
##if ( configAPPLICATION_ALLOCATED_HEAP == 1 )

/* The application writer has already defined the array used for the RTOS
* heap -  probably so it can be placed in a special segment or address. */
    extern uint8_t ucHeap[ configTOTAL_HEAP_SIZE ];
##else
    static uint8_t ucHeap[ configTOTAL_HEAP_SIZE ];
##endif /* configAPPLICATION_ALLOCATED_HEAP */

	```
	然后，对于pvPortMalloc调用时，从这个数组中分配空间。
	FreeRTOS在创建任务时，需要2个内核对象：task control block(TCB)、stack。 使用heap_1时，内存分配过程如下图所示：
	- A：创建任务之前整个数组都是空闲的
	- B：创建第1个任务之后，蓝色区域被分配出去了
	- C：创建3个任务之后的数组使用情况
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-8/image3.png)
	### **8.2.2 Heap_2**
	Heap_2之所以还保留，只是为了兼容以前的代码。新设计中不再推荐使用Heap_2。建议使用Heap_4来替代Heap_2，更加高效。
	Heap_2也是在数组上分配内存，跟Heap_1不一样的地方在于：
	- Heap_2使用最佳匹配算法(best fit)来分配内存
	- 它支持vPortFree
	最佳匹配算法：
	- 假设heap有3块空闲内存：5字节、25字节、100字节
	- pvPortMalloc想申请20字节
	- 找出最小的、能满足pvPortMalloc的内存：25字节
	- 把它划分为20字节、5字节
		- 返回这20字节的地址
		- 剩下的5字节仍然是空闲状态，留给后续的pvPortMalloc使用
	与Heap_4相比，Heap_2不会合并相邻的空闲内存，所以Heap_2会导致严重的"碎片化"问题。
	但是，如果申请、分配内存时大小总是相同的，这类场景下Heap_2没有碎片化的问题。所以它适合这种场景：频繁地创建、删除任务，但是任务的栈大小都是相同的(创建任务时，需要分配TCB和栈，TCB总是一样的)。
	虽然不再推荐使用heap_2，但是它的效率还是远高于malloc、free。
	使用heap_2时，内存分配过程如下图所示：
	- A：创建了3个任务
	- B：删除了一个任务，空闲内存有3部分：顶层的、被删除任务的TCB空间、被删除任务的Stack空间
	- C：创建了一个新任务，因为TCB、栈大小跟前面被删除任务的TCB、栈大小一致，所以刚好分配到原来的内存
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-8/image4.png)
	### **8.2.3 Heap_3**
	Heap_3使用标准C库里的malloc、free函数，所以堆大小由链接器的配置决定，配置项configTOTAL_HEAP_SIZE不再起作用。
	C库里的malloc、free函数并非线程安全的，Heap_3中先暂停FreeRTOS的调度器，再去调用这些函数，使用这种方法实现了线程安全。
	### **8.2.4 Heap_4**
	跟Heap_1、Heap_2一样，Heap_4也是使用大数组来分配内存。
	Heap_4使用 **首次适应算法(first fit)来分配内存** 。它还会把相邻的空闲内存合并为一个更大的空闲内存，这有助于较少内存的碎片问题。
	首次适应算法：
	- 假设堆中有3块空闲内存：5字节、200字节、100字节
	- pvPortMalloc想申请20字节
	- 找出第1个能满足pvPortMalloc的内存：200字节
	- 把它划分为20字节、180字节
	- 返回这20字节的地址
	- 剩下的180字节仍然是空闲状态，留给后续的pvPortMalloc使用
	Heap_4会把相邻空闲内存合并为一个大的空闲内存，可以较少内存的碎片化问题。适用于这种场景：频繁地分配、释放不同大小的内存。
	Heap_4的使用过程举例如下：
	- A：创建了3个任务
	- B：删除了一个任务，空闲内存有2部分：
	- 顶层的
	- 被删除任务的TCB空间、被删除任务的Stack空间合并起来的
	- C：分配了一个Queue，从第1个空闲块中分配空间
	- D：分配了一个User数据，从Queue之后的空闲块中分配
	- E：释放的Queue，User前后都有一块空闲内存
	- F：释放了User数据，User前后的内存、User本身占据的内存，合并为一个大的空闲内存
	![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-8/image5.png)
	Heap_4执行的时间是不确定的，但是它的效率高于标准库的malloc、free。
	### **8.2.5 Heap_5**
	Heap_5分配内存、释放内存的算法跟Heap_4是一样的。
	相比于Heap_4，Heap_5并不局限于管理一个大数组：它可以管理多块、分隔开的内存。
	在嵌入式系统中，内存的地址可能并不连续，这种场景下可以使用Heap_5。
	既然内存时分隔开的，那么就需要进行初始化：确定这些内存块在哪、多大：
	- 在使用pvPortMalloc之前，必须先指定内存块的信息
	- 使用vPortDefineHeapRegions来指定这些信息
	怎么指定一块内存？使用如下结构体：
	```c
typedef struct HeapRegion
{
    uint8_t * pucStartAddress; // 起始地址
    size_t xSizeInBytes;       // 大小
} HeapRegion_t;

	```
	怎么指定多块内存？使用一个HeapRegion_t数组，在这个数组中，低地址在前、高地址在后。 比如：
	```c
HeapRegion_t xHeapRegions[] =
{
  { ( uint8_t * ) 0x80000000UL, 0x10000 }, // 起始地址0x80000000，大小0x10000
  { ( uint8_t * ) 0x90000000UL, 0xa0000 }, // 起始地址0x90000000，大小0xa0000
  { NULL, 0 } // 表示数组结束
 };

	```
	vPortDefineHeapRegions函数原型如下：
	```c
void vPortDefineHeapRegions( const HeapRegion_t * const pxHeapRegions );
	```
把xHeapRegions数组传给vPortDefineHeapRegions函数，即可初始化Heap_5。
## 8.3 Heap相关的函数
### 8.3.1 pvPortMalloc/vPortFree
函数原型：
```c
void * pvPortMalloc( size_t xWantedSize );
void vPortFree( void * pv );

```
## **8.3 Heap相关的函数**
### **8.3.1 pvPortMalloc/vPortFree**
函数原型：
```c
void * pvPortMalloc( size_t xWantedSize );
void vPortFree( void * pv );

```
作用：分配内存、释放内存。
如果分配内存不成功，则返回值为NULL。
### **8.3.2 xPortGetFreeHeapSize**
函数原型：
```c
size_t xPortGetFreeHeapSize( void );

```
当前还有多少空闲内存，这函数可以用来优化内存的使用情况。比如当所有内核对象都分配好后，执行此函数返回2000，那么configTOTAL_HEAP_SIZE就可减小2000。
注意：在heap_3中无法使用。
### **8.3.3 xPortGetMinimumEverFreeHeapSize**
函数原型：
```c
size_t xPortGetMinimumEverFreeHeapSize( void );

```
返回：程序运行过程中，空闲内存容量的最小值。
注意：只有heap_4、heap_5支持此函数。
### **8.3.4 malloc失败的钩子函数**
在pvPortMalloc函数内部：
```c
void * pvPortMalloc( size_t xWantedSize )vPortDefineHeapRegions
{
    ......
    #if ( configUSE_MALLOC_FAILED_HOOK == 1 ){
            if( pvReturn == NULL )
            {
                extern void vApplicationMallocFailedHook( void );
                vApplicationMallocFailedHook();
            }
        }
    #endif

    return pvReturn;
}

```
所以，如果想使用这个钩子函数：
- 在FreeRTOSConfig.h中，把configUSE_MALLOC_FAILED_HOOK定义为1
- 提供vApplicationMallocFailedHook函数
- pvPortMalloc失败时，才会调用此函数
# 07.动态和静态创建任务 {toggle="true"}
	**任务：**
	- 做什么事：函数
	- 栈和 TCB
		- 可以动态分配也可以事先静态分配
	- 优先级
	**一个任务备切换出来之后， 如何才能再次找到它：**
	- 在一个链表中找到任务A、B、C
	- 在链表中存放任务控制块（TCB ）（task contol block）
	<empty-block/>
	在动态分配内存的时候 根据你输入的栈的大小自动分配栈和TCB
	在静态分配内存的时候，需要事先准备好栈和TCB结构体
	动态：
	```c
BaseType_t xTaskCreate( TaskFunction_t pxTaskCode, // 函数指针, 任务函数
const char * const pcName, // 任务的名字
const configSTACK_DEPTH_TYPE usStackDepth, // 栈大小,单位为word,10表示40字节
void * const pvParameters, // 调用任务函数时传入的参数
UBaseType_t uxPriority,    // 优先级
TaskHandle_t * const pxCreatedTask ); // 任务句柄, 以后使用它来操作这个任务
	```
	静态：
	```c
TaskHandle_t xTaskCreateStatic ( 
    TaskFunction_t pxTaskCode,   // 函数指针, 任务函数
    const char * const pcName,   // 任务的名字
    const uint32_t ulStackDepth, // 栈大小,单位为word,10表示40字节
    void * const pvParameters,   // 调用任务函数时传入的参数
    UBaseType_t uxPriority,      // 优先级
    StackType_t * const puxStackBuffer, // 静态分配的栈，就是一个buffer
    StaticTask_t * const pxTaskBuffer // 静态分配的任务结构体的指针，用它来操作这个任务
);
	```
	<empty-block/>
	测试：
	使用静态分配的时候，需要为任务提供buff 和TCB结构体
	```plain text
static StackType_t g_pucStackOfLightTask[128];  //提供Buff  g 为全局的缩写
static StaticTask_t g_TCBofLightTask;            //提供TCB任务结构体
static TaskHandle_t xLightTaskHandle;              //光任务 句柄结构体  以FreeRTOS 的规范， x表示某些结构体  


static StackType_t g_pucStackOfColorTask[128];
static StaticTask_t g_TCBofColorTask;
static TaskHandle_t xColorTaskHandle; 
	```
	使用时是这样
	```plain text
/* 创建任务：光 */
   xLightTaskHandle = xTaskCreateStatic(Led_Test, "LightTask", 128, NULL, osPriorityNormal, g_pucStackOfLightTask, &g_TCBofLightTask);

 /* 创建任务：色 */
   xColorTaskHandle = xTaskCreateStatic(ColorLED_Test, "ColorTask", 128, NULL, osPriorityNormal, g_pucStackOfColorTask, &g_TCBofColorTask);
	```
	任务句柄还没学到。 AI说  主要有以下作用：一是用于识别和管理任务，可进行挂起、恢复、删除等操作；二是在任务间通信与同步中，可指定消息的接收者或发送者，以及用于同步操作；三是能查询任务状态和获取任务属性信息。
	<empty-block/>
	**TCB的简要介绍：**
		**一、存储任务状态信息**
		1. 记录任务当前的运行状态，如就绪态、运行态、阻塞态等。
		2. 保存任务的优先级，决定任务在可运行状态下获取 CPU 时间片的优先级顺序。
		**二、管理任务资源**
		1. 可能包含任务所使用的栈信息，包括栈的起始地址和大小。
		2. 可以存储与任务相关的同步对象指针，如信号量、互斥量等，以便任务进行同步和通信操作。
		**三、支持任务调度**
		1. 操作系统在进行任务调度时，通过检查 TCB 中的信息来决定下一个要执行的任务。
		2. TCB 使得操作系统可以快速地切换任务，保存和恢复任务的上下文。
		总之，任务控制块是 FreeRTOS 管理任务的核心数据结构，它为任务的创建、运行、暂停、恢复和删除等操作提供了必要的信息和控制机制。
		<empty-block/>
	<empty-block/>
	**动态任务**在创建时会自动分配内存和初始化任务块等等。但有可能因为内存不够等原因失败，所以返回值为创建任务的成功或失败。
	**静态任务**在创建时需要用户提前准备buff和 TCB控制块 。 所以他一定能创建成功。他的参数中没有任务句柄。他的返回值是任务句柄
# 08.估算栈的大小 {toggle="true"}
	**栈的作用是**
	- 返回地址  比如LR寄存器、其他寄存器
		- 取决于函数的调用深度
	- 局部变量
		- 取决于代码
	- 保存现场（在任务被切换时）
		- 现场时个寄存器 16 \* 4 = 64 字节
	**如何评估栈的大小**
	- 选取最复杂的调用关系
		- n级调用 \* （被调用者寄存器R4 - R11共8个 + LR寄存器）也就是一次调用最多使用36 个字节
		- 所以 调用深度越深 所需要的栈的空间就越大
	- 选取创建局部变量最大的函数。
# 09.一个函数创建多个任务 {toggle="true"}
	定义结构体，通过在函数内定义 void\*  类型的指针变量 来接收我们定义的结构体。
	通过创建值不同的结构体，来作为创建任务创建时传入函数的参数
	通过多次创建任务，但任务在调用时传入函数的 参数不同。 可以做到同一个函数，创建多个不同的任务。
	<empty-block/>
	注意使用struct  而不能使用Typedef struct 否则任务在传参时会报错\~
	<empty-block/>
# 10.删除任务 {toggle="true"}
	删除任务 使用 vTaskDelete  函数
	**它的参数为 **任务在创建时  返回 或 传入的  **任务句柄**
	<empty-block/>
	使用这个函数可以直接删除函数，但是多次的删除和创建 ，每次创建都会去分配内存。频繁地动态分配内存很容易导致内存的碎片化，直到分配不到内存。
	<empty-block/>
	**并且在删除任务之后。这个任务就终止了。没法做一些清除的工作。**
	<empty-block/>
	所以，在删除任务时，我们并不会经常使用vTaskDelete函数。
	一般来说是让任务自己来根据（比如遥控器的按键）自己停止。自己做一些清除的工作
	<empty-block/>
# 11.优先级与阻塞 {toggle="true"}
	这里会通过提升任务的优先级来达到改善播放的效果
	可以通过修改优先级 来达到优化音乐播放，
	学习使用vTaskdelay的使用。使用自己写的会阻塞任务。如果阻塞的delay在高优先级的任务，会导致只会运行高优先级的任务。
# 12.任务状态 {toggle="true"}
	对于FreeRTOS有四种状态
	- Ready（就绪）
	- Running（运行中）
	- Blocked（等待某些Event）
		- 比如vTaskDelay函数，就会进入
	- Suspend（暂停）
	<empty-block/>
	暂停状态如何进入：
	- 自己调用`vTaskSuspend()；`  
		- 处于Running状态的自己把自己比暂停
		- 处于Running状态的别人把处于Ready或Blocked 状态的你暂停
	<empty-block/>
	**任务状态详解**
	- 一个任务在被创建之后，一定处于Ready状态
	- 只要他的优先级足够高，他会立马从Ready状态进入Running状态开始运行
	- 当他因为vTaskDelay函数进入Blocked 阻塞状态后，函数会等待某一时刻的到来。（虽然叫阻塞状态，但不同于我们自己制作的Delay，这时别的程序也可以运行）
	- 在暂停时，会让某一任务暂停，进入Suspend暂停状态。
	- 在暂停状态时，使用恢复函数`vTaskResume() `可以从暂停状态恢复到Ready状态，又因为优先级最高，会迅速进入Running状态
	<empty-block/>
# 13.任务管理与调度规则 {toggle="true"}
	1. 相同优先级的任务 轮流运行
	2. 最高优先级的任务 先运行
	**则**
	1. 高优先级的任务未执行完，低优先级的任务无法运行
	2. 一但高优先级的任务就绪，会马上运行
	3. 高优先级的任务有多个，他们会轮流运行
	<empty-block/>
	**如何管理？**
	- **核心：链表。**
	FreeRTOS 中有一个系统时钟 Tick 中断，在该中断中会执行三件事：
	1. 累加计数。
	2. 判断 DelayedTaskList 中处于阻塞态的任务是否到时间。若时间已到，则将其恢复到就绪链表中，并发起一次调度。
		- 调度会遍历就绪态链表 ReadyList，按照从高优先级到低优先级的顺序从上往下遍历（任务的最高优先级为 56）。
		- 直到找到一个非空项。
		- 从非空项中，取出下一个要运行的任务并运行它。
			- 当只有三个相同优先级的任务被创建时，会先运行最后一个创建的任务，接着是第一个任务，即按照 3、1、2、3、1、2 的顺序运行。
			- 当一个高优先级任务就绪时，低优先级任务会被打断，然后运行高优先级任务。在下一次调度时，会跳过上次被打断的任务，转而去运行下一个任务。
	3. `vTaskDelay`会把任务从就绪链表移动到 DelayTaskList 链表。
	4. `vTaskSuspend`会把任务从 Delay 或 Ready 状态移动到 xSuspendeTaskList 链表，直到我们调用 Resume 时，才把它放回 Ready 链表。
	<empty-block/>
	**任务函数如果不是死循环会发生什么？**
	如果任务不是死循环，那么在执行完毕后会进入`prvTaskExitError`这个错误函数。在该函数中，会关闭所有中断，然后进入死循环状态。这将导致所有任务都无法运行。
	**\[空闲任务\] 任务如何退出？**
	1. 自杀（`vTaskDelete(NULL)`）：
		- 自杀的任务会由优先级为 0 的空闲任务来释放栈和任务控制块（TCB）。
		- 空闲任务要么处于就绪状态，要么处于运行状态，它不会处于阻塞状态。
		- 如果一直有任务自杀，但其他高优先级任务不释放 CPU，那么就无法释放内存，可能会导致内存不足。
	2. 它杀（`vTaskDelete(任务句柄)`）：
		- 它杀可以由杀死该任务的那个任务帮忙释放栈和 TCB。
	**所以该怎么做？**
	良好的编程习惯：
	1. 事件驱动时，在使用延迟（`vTaskDelay`）的时候，不要使用死循环。
# 14.两个Delay函数 {toggle="true"}
	1. **vTaskDelay：**
		- 这个函数会使任务进入阻塞态。它会在当前的系统时钟节拍（tick）基础上加上指定的 “n” 个 tick 之后，任务才会恢复为就绪状态。
		- 例如：假设当前系统时钟节拍为 100，使用`vTaskDelay(10)`，那么任务会在系统时钟节拍变为 110 的时候恢复就绪状态。
		- 另一个例子：如果任务需要等待一段时间执行下一个操作，可以使用`vTaskDelay`来实现简单的定时功能。比如在一个数据采集任务中，每采集一次数据后等待 50 个 tick 再进行下一次采集。
	2. **vTaskDelayUntil：**
		- 此函数同样会使任务进入阻塞态。它是在某个特定时间点 “T1” 的基础上加上 “n” 个 tick 之后，任务恢复就绪。并且会将 “T1” 赋值为 “T1 加 n”。可以通过`vTaskGetTickCount()`函数获得当前的 tick 值。
		- 例如：假设有一个周期性执行的任务，需要每 20 个 tick 执行一次，但每次执行任务时用时不定。可以设置一个初始时间点 “T1”，然后每次使用`vTaskDelayUntil`传入 “T1” 和 20，确保任务以固定的周期执行。
		- 使用 `vTaskDelayUntil`来实现更精确的定时任务，避免累积误差。对于需要在固定时间间隔执行的任务，优先考虑 `vTaskDelayUntil`。
	<empty-block/>
# 15.同步互斥与通信的基本概念和例子 {toggle="true"}
	**同步的概念：**
	1. 我**等**你上厕所，等就是一个同步操作
	2. 经理B必须等同时A完成表单。才能去找领导汇报。经理B对员工A有依赖，B必须放慢脚步等A。称为同步
	**互斥的概念：**
	1. 同事A在用会议室，经理B也想用。即使B是领导，但是必须等同事A用完会议室才能去用。
	2. OLED屏幕的使用。
	**同一时间只能有一个任务来使用它，这个资源就叫做临界资源**
	<empty-block/>
	**举几个例子来实现**
	1. 同步的例子：有缺陷
	创建任务A 花费一段时间计算一个数，创建任务B不断判断A完成后置的标志位。
	这样B虽然不做事情，但同样也会消耗CPU资源，在AB中不断切换。也是耗时的。每次任务切换到B都是在浪费时间。
	1. 互斥的例子：有缺陷
	使用全局变量来保护..虽然短时间可以，但是时间漫长，会有可能在程序运行的空缺中  被打断。
	**如果使用**关闭tick中断后再修改全局变量，虽然不会被打断，但是会导致CPU利用率低。
	也就是A打印一半。B来掺一脚。A继续，B又来。真是烦人啊B
	**所以可以**让B在判断没法打印的时候就阻塞自己。这样B就只会掺一小小小脚了。
	在A忙完之后再叫一下B，让B醒来。
	1. 通信的例子：有缺陷
	同样是使用全局变量。 任务A计算 赋值给全局变量。
	有可能任务B在使用的时候，A还没赋值，或只弄了一部分。这就导致通信错误了。
	<empty-block/>
	FReeRTOS的解决方案：
	- 正确性
	- 效率：等待着要进入阻塞态
	- 多种解决方案
	1. 引入队列（queue，FIFO）
	谁先做好 就往队列上放，那个产品现生产好，消费者就先用那个产品
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/2cca4849-8e87-4bcc-8fc3-61f1eae558db.webp)
	1. 事件组（event group）
	事件的组合，每一位代表一个事件，可以在做完某个事情之后设置某个bit’为1。消费者可以等待某个事件、或某几个事件、或者等待若干个事件中的某一个事件
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/d7dbcd39-1de4-411d-b344-15b4e301f8a5.webp)
	1. 信号量（semaphore）
	生产者让计数+1.消费者让计数 -1
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/5c05e3a8-5a2f-4a66-8cda-dc9cc098d3e5.webp)
	1. 互斥量（nutex）
	当信号量的计数值只为 1 或 0 时，信号量就变为了互斥量，可以使用互斥量来保护一些临界资源
	使用互斥量会引入其他问题，比如优先级反转等、提出了优先级继承来解决这些问题，
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/1e0224d7-dd52-4150-b9f5-2f57f8515f27.webp)
	1. 任务通知（task notificatin）
	任务通知是多对一的关系。可以通知数值或者通知某个事件。
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/46962f28-c32f-43fd-b052-a7e4f261790a.webp)
	<empty-block/>
# 16.队列的本质 {toggle="true"}
	<empty-block/>
	环形缓冲区可以被看作是一个首尾相接的环形内存区域。有一个读指针和一个写指针。写指针指向新数据写入的位置，读指针指向要读取的数据位置。
	当写指针到达缓冲区末尾时，会回到起始位置继续写入，就像在一个环形轨道上循环。同样，读指针在读取数据后也会相应移动。
	为了防止在判断是否满的时候出问题（没有读，但一直写）
	定义当写的下一个位置等于0时，就代表满。
	这样就可以避免读写指针重合。
	<empty-block/>
	队列中，数据的读写本质就是环形缓冲区，在这个基础上增加了互斥措施、阻塞-唤醒机制。<br> 
	如果这个队列不传输数据，只调整"数据个数"，它就是**信号量(semaphore)。**<br>如果信号量中，限定"数据个数"最大值为1，它就是**互斥量(mutex)。**<br>
	队列的本职其实就是 环形区+两个链表
	两个链表 分别放发送者 （Sender List）和 接受者（Receiver List）
	<empty-block/>
	一个任务B 在 读链表，如果读不到链表并且它愿意等待的话。它会把自己从就绪（Ready）链表中删除。并放入队列的接收链表（Queue.recv_list）中（这里存放的是想读队列但读不到数据的任务）。并放DelayedyList中（用于超时后的处理）
	另一个任务A如果写了队列。那么就会唤醒任务B，把任务B重新放入Ready列表，在有机会运行时，就会得到写入的数据，去处理。如果一直没有写，到了超时时间也会唤醒任务B。同样到Ready列表，但会返回错误。
	当B在写入时，如果满了，可以阻塞或者直接返回错误。 当A取走之后也可以告诉B，我已经取走了，你可以继续放了
	<empty-block/>
# 17.队列实验，多设备玩游戏 {toggle="true"}
	**一般步骤**
	- 创建队列
	- 写队列
	- 读队列
	<empty-block/>
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/569973e8-805d-4bd8-934c-a0828e844748.webp)
	注意：
	要注意在多个任务同时对一个队列写入数据的时候，要数据格式一样。
	可以创建任务来对某个队列的数据处理之后 再写入 最终控制的程序
# 18.队列集是什么 {toggle="true"}
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/f3a88457-4a6f-4382-b8f5-78c2c72e95d4.webp)
	可以看左边，如果在硬件相关的程序中 判断和转化数据，是不利于程序移植的
	右边的虽然方便移植，但如果每一个底层硬件都去写一个自己的队列，然后用任务去解析再写入另一个队列。  会创建多个任务，会消耗栈资源，对系统资源有很大的浪费。
	<empty-block/>
	所以应该把解析任务创建为一个。他可以同时解析多个 队列中的数据。再写入我们想写入的队列。这样就不会浪费系统资源。
	- 这样写可以使用**轮询**或者**队列集**的方式。
		- 阻塞方式可能会导致任务的推迟和延迟，会浪费CPU资源。所以不能指定超时时间。（一般可以读到，但是效果不好就是了）
	<empty-block/>
	<empty-block/>
	**队列集 也是一个队列， 不过队列集中放的是不同队列的句柄**
	- 在一个队列被写入时，他会判断这个队列是否在一个队列集中。如果在，他就会在队列写入的同时，在队列集中写入这个队列的句柄。
	- 那么我们只需要创建一个任务，来读取队列集中有哪些句柄，然后读取句柄中的信息再 处理和写入对应的队列就OK  啦\~
	<empty-block/>
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/87dacfec-f2f0-418f-b02a-e58209624cd6.webp)
# 19.队列集实验，修改之前的游戏 {toggle="true"}
	<empty-block/>
	大概的流程
	1. 写n个硬件驱动，每个硬件驱动有一个对应的队列。
		- 在硬件驱动中把数据添加到对应的队列中
	2. 创建数据处理任务
		- 在任务函数中创建队列集，把n个硬件驱动添加到队列集中
			- 注意 需要 写一个函数来专门复制 硬件驱动中的 队列句柄。 
		- 读队列集，判断队列集返回的 队列句柄。
		- 在if分支中，写数据处理及写入最终的队列的函数。
			<empty-block/>
	<empty-block/>
	出现这个错误，意思是队列集相关的函数没有定义，需要在`FreeRTOSConfig.h`配置文件中设置
	```c++
/* USER CODE BEGIN Includes */
#define configUSE_QUEUE_SETS 1

/* Section where include file can be added */
	```
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/36a1d5fd-bbef-4c26-8747-dc7b3c483154.webp)
	<empty-block/>
	<empty-block/>
	如果发现有些东西只加载了一半或者有一些元素没有加载
	可能是因为堆空间不够了。 因为我们创建了许多的队列。可以在CubeMX中设置堆的大小heap
	队列是在堆中分配的
# 20.分发数据给对个队列 {toggle="true"}
	在这里是 有三个小车任务，他们有各自的队列。
	在底层硬件中解析出数据后写入三个任务的队列中。
	<empty-block/>
	在底层硬件中 对多个队列写入数值时
	这样不美观，而且每次都要修改底层代码
	```plain text
extern QueueHandle_t g_xQueueCar1;
extern QueueHandle_t g_xQueueCar2;
extern QueueHandle_t g_xQueueCar3;
extern QueueHandle_t g_xQueueCar4;

xQueueSendToBackFromISR(g_xQueueCar1, pidata, NULL);
xQueueSendToBackFromISR(g_xQueueCar2, pidata, NULL);
xQueueSendToBackFromISR(g_xQueueCar3, pidata, NULL);
xQueueSendToBackFromISR(g_xQueueCar4, pidata, NULL);

	```
	可以写一个函数用于 “注册队列” 一个用于写入已注册的队列
	```c
//注册队列
void RegisterQueueHandle(QueueHandle_t queueHandle)
{
    if(g_queue_cnt < 10)
    {
        g_xQueues[g_queue_cnt++] = queueHandle;
    }
}

//分发数据到多个队列
static void DispatchKey(struct ir_data* pidata)
{
    int i = 0;
    for(i = 0; i< g_queue_cnt; i++)
    {
        xQueueSendFromISR(g_xQueues[i], pidata, NULL);  //把数据写入已注册的队列中
    }
    
    
//    extern QueueHandle_t g_xQueueCar1;
//    extern QueueHandle_t g_xQueueCar2;
//    extern QueueHandle_t g_xQueueCar3;
//    extern QueueHandle_t g_xQueueCar4;
//    
//    xQueueSendToBackFromISR(g_xQueueCar1, pidata, NULL);
//    xQueueSendToBackFromISR(g_xQueueCar2, pidata, NULL);
//    xQueueSendToBackFromISR(g_xQueueCar3, pidata, NULL);
//    xQueueSendToBackFromISR(g_xQueueCar4, pidata, NULL);

}
	```
	<empty-block/>
	然后就可以调用DispatchKey 函数来分发数据了
# 21.信号量和互斥量 {toggle="true"}
	## 信号量实验 {toggle="true"}
		以三个小车同时向前走的任务为举例
		如果不加以限制，在程序运行后，三辆小车会几乎同时到达终点
		<empty-block/>
		**但如果用计数型信号量**
		设置初始值和最大值。
		可以放两张票，先让两辆车同时开始，当一辆结束卮再释放票（give），让另一辆开始
		<empty-block/>
		**如果使用二进制信号量**
		默认值为0，所以需要手动给个票。最大值为1
		<empty-block/>
		**在排队等待时。**
		高优先级的任务，在等待链表时，排在最前边。 同等优先级的任务先来先排
	## 优先级反转 {toggle="true"}
		任务1 优先级为低
		任务2 优先级为中
		任务3 优先级为高
		<empty-block/>
		任务1 在程序运行时抢到了信号量。
		任务3 需要信号量才能运行，于是阻塞。
		任务2 优先级比任务1高，任务2运行时任务1无法运行
		<empty-block/>
		导致了任务2把任务3  低优先级反而把高优先级的阻塞了。
		**可以使用互斥量解决**
	## 互斥量（解决优先级反转） {toggle="true"}
		互斥量的默认值为1 
		**优先级继承**
			- 当一个高优先级任务等待一个被低优先级任务持有的互斥量时，低优先级任务的优先级会被**临时提升**到与高优先级任务相同的优先级，以**避免优先级反转**问题。
		参考代码：
		```c
#include "semphr.h"

		static SemaphoreHandle_t g_xI2CMutex;  //所有信号量都是这个句柄

g_xI2CMutex = xSemaphoreCreateMutex();   //创建互斥量


void GetI2C(void)
{
    /* 等待互斥量 */
    xSemaphoreTake(g_xI2CMutex,portMAX_DELAY);
}

void PutI2C(void)
{
    /* 释放互斥量 */
    xSemaphoreGive(g_xI2CMutex);
}
		```
# 22.事件组的本质 {toggle="true"}
	对于队列、信号量 他们都只能 唤醒一个任务
	而事件组是可以唤醒多个任务的 （event group）
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/d529a270-d8b9-42e9-b103-7b1139d6e284.webp)
	- 事件组中有一个整数、高八位不用（用于表示事件 或还是与的关系）。
	- 后面的数据位每一位都表示一个事件。= 1表示对应事件产生
	- 在每个任务的结构体中，都会有一个整数：表示自己想等待那个（那些）事件。是或的关系，还是与的关系
	- 在一个任务写入事件之后，会遍历 链表的所有任务，查询有哪些任务的事件得到满足。然后唤醒。
	<empty-block/>
	一般有三种使用情况
	1. 广播
		- 一个任务设置事件组，其他n个任务等待事件组。
	2. or
		- 两个事件任意一个有1 就ok
		```plain text
//等待事件 bit 0 or bit 1                                  pdFALSE 为 或
xEventGroupWaitBits(g_xEventCar, (1<<0) | (1<<1), pdTRUE, pdTRUE, portMAX_DELAY);

		```
	3. and
		- 两个事件同时为1 就ok
		```plain text
//等待事件 bit 0 and bit 1                                 pdFALSE 为 或
xEventGroupWaitBits(g_xEventCar, (1<<0) | (1<<1), pdTRUE, pdTRUE, portMAX_DELAY);

		```
		**参数分别为创建事件组的句柄， 要检查那n位， 等到之后要恢复为0吗？  ， 是and（pdtrue）还是or（pdfalse）， 等多久 ？**
		<empty-block/>
	**一般在编写程序时，可以用中断设置事件组，然后任务等待事件，这样可以避免浪费CPU资源**
# 23.任务通知的本质 {toggle="true"}
	- 对于队列、信号量/互斥量、信号组。两边的任务都要通过他们（通信对象）来进行通信（或者互斥同步操作）。 他们都不知道是谁提供的数据或者读取的数据
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/22a39e34-446f-4c9b-814b-6f7f82cdfa70.webp)
	- 而任务通知，是明确的知道我要通知谁
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/54f7375d-2145-41b3-8108-966a77d367d5.webp)
	- 任务需两情相悦方可被唤醒（一个因等待通知而阻塞的任务与一个发出通知的任务），除此之外均不可。每个任务存在三种通知状态：
		- taskNOT_WAITING_NOTIFICATION：任务未在等待通知。
		- taskWAITING_NOTIFICATION：任务正在等待通知。
		- taskNOTIFICATION_RECEIVED：任务已接收到通知，亦称为 pending（有数据待处理）。
		当我们调用或收到期望的通知时，会修改 TCB 结构体，将其中的值++为新值，并将状态调整为上述三种状态之一。
		若任务 A 向任务 B 发送通知，即便任务 B 初始状态为未等待通知，其状态也会变为已接收到通知 且 使值递增。倘若此时任务 B 调用函数并开始等待通知，那么它会立即得到通知并再次运行，随后将状态重新设为不再等待通知，并使值递减。
# 24.软件定时器的本质 {toggle="true"}
	## **运行于中断上下文** {toggle="true"}
		1. 在硬件定时器的中断中，每隔一段时间计数值 + 1。
		2. 如果在当前时刻 n 时刻创建定时器，定时时长为 10，那么他的超时时间就是 n+10。
		3. 硬件定时器每进入一次中断，就判断当前那个定时器到达时间。
		4. 从而执行函数或其他操作。
		5. 注意：中断要尽快退出，别弄过于复杂的程序。
	## **运行于任务上下文** {toggle="true"}
		1. 定时器到达时间后，去通知任务，（唤醒任务）。
		2. 这样超时函数（定时器到达后执行的函数）是运行在某个任务之中的，就没有那么多的限制了。
		3. **在 FreeRTOS 中就是按照任务上下文的来执行的。**
		4. 定时的启动、停止、复位、删除等等等等，都是去写队列。
	## 注意事项 {toggle="true"}
		1. 保证定时器的优先级足够高，或者让别的任务运行一段时间后 `vTaskDelay`一段时间。
# 25.任务和中断的两套函数 {toggle="true"}
	<table>
<tr>
<td>**类型**</td>
<td>**在任务中**</td>
<td>**在ISR中**</td>
</tr>
<tr>
<td>队列(queue)</td>
<td>xQueueSendToBack</td>
<td>xQueueSendToBackFromISR</td>
</tr>
<tr>
<td></td>
<td>xQueueSendToFront</td>
<td>xQueueSendToFrontFromISR</td>
</tr>
<tr>
<td></td>
<td>xQueueReceive</td>
<td>xQueueReceiveFromISR</td>
</tr>
<tr>
<td></td>
<td>xQueueOverwrite</td>
<td>xQueueOverwriteFromISR</td>
</tr>
<tr>
<td></td>
<td>xQueuePeek</td>
<td>xQueuePeekFromISR</td>
</tr>
<tr>
<td>信号量(semaphore)</td>
<td>xSemaphoreGive</td>
<td>xSemaphoreGiveFromISR</td>
</tr>
<tr>
<td></td>
<td>xSemaphoreTake</td>
<td>xSemaphoreTakeFromISR</td>
</tr>
<tr>
<td>事件组(event group)</td>
<td>xEventGroupSetBits</td>
<td>xEventGroupSetBitsFromISR</td>
</tr>
<tr>
<td></td>
<td>xEventGroupGetBits</td>
<td>xEventGroupGetBitsFromISR</td>
</tr>
<tr>
<td>任务通知(task notification)</td>
<td>xTaskNotifyGive</td>
<td>vTaskNotifyGiveFromISR</td>
</tr>
<tr>
<td></td>
<td>xTaskNotify</td>
<td>xTaskNotifyFromISR</td>
</tr>
<tr>
<td>软件定时器(software timer)</td>
<td>xTimerStart</td>
<td>xTimerStartFromISR</td>
</tr>
<tr>
<td></td>
<td>xTimerStop</td>
<td>xTimerStopFromISR</td>
</tr>
<tr>
<td></td>
<td>xTimerReset</td>
<td>xTimerResetFromISR</td>
</tr>
<tr>
<td></td>
<td>xTimerChangePeriod</td>
<td>xTimerChangePeriodFromISR</td>
</tr>
	</table>
	### **17.1.1 为什么需要两套API** {toggle="true"}
		在任务函数中，我们可以调用各类API函数，比如队列操作函数：xQueueSendToBack。但是在ISR中使用这个函数会导致问题，应该使用另一个函数：xQueueSendToBackFromISR，它的函数名含有后缀"FromISR"，表示"从ISR中给队列发送数据"。
		FreeRTOS中很多API函数都有两套：一套在任务中使用，另一套在ISR中使用。后者的函数名含有"FromISR"后缀。
		为什么要引入两套API函数？
		- 很多API函数会导致任务计入阻塞状态：
			- 运行这个函数的 **任务** 进入阻塞状态
			- 比如写队列时，如果队列已满，可以进入阻塞状态等待一会
		- ISR调用API函数时，ISR不是"任务"，ISR不能进入阻塞状态
		- 所以，在任务中、在ISR中，这些函数的功能是有差别的
		为什么不使用同一套函数，比如在函数里面分辨当前调用者是任务还是ISR呢？示例代码如下：
		```plain text
BaseType_t xQueueSend(...)
{
    if (is_in_isr())
    {
        /* 把数据放入队列 */

        /* 不管是否成功都直接返回 */
    }
    else /* 在任务中 */
    {
        /* 把数据放入队列 */
        /* 不成功就等待一会再重试 */
    }
}
		```
		FreeRTOS使用两套函数，而不是使用一套函数，是因为有如下好处：
		- 使用同一套函数的话，需要增加额外的判断代码、增加额外的分支，是的函数更长、更复杂、难以测试
		- 在任务、ISR中调用时，需要的参数不一样，比如：
			- 在任务中调用：需要指定超时时间，表示如果不成功就阻塞一会
			- 在ISR中调用：不需要指定超时时间，无论是否成功都要即刻返回
			- 如果强行把两套函数揉在一起，会导致参数臃肿、无效
		- 移植FreeRTOS时，还需要提供监测上下文的函数，比如 **is_in_isr()**
		- 有些处理器架构没有办法轻易分辨当前是处于任务中，还是处于ISR中，就需要额外添加更多、更复杂的代码
		使用两套函数可以让程序更高效，但是也有一些缺点，比如你要使用第三方库函数时，即会在任务中调用它，也会在ISR总调用它。这个第三方库函数用到了FreeRTOS的API函数，你无法修改库函数。这个问题可以解决：
		- 把中断的处理推迟到任务中进行(Defer interrupt processing)，在任务中调用库函数
		- 尝试在库函数中使用"FromISR"函数：
			- 在任务中、在ISR中都可以调用"FromISR"函数
			- 反过来就不行，非FromISR函数无法在ISR中使用
		- 第三方库函数也许会提供OS抽象层，自行判断当前环境是在任务还是在ISR中，分别调用不同的函数
	### 中断中怎么切换任务 {toggle="true"}
		低优先级的任务正在运行时，
		突然发生了一个中断，
		在中断中会  唤醒了更高优先级的任务B ，并在中断退出之前发起调度，使B开始运行，
		否则高优先级的任务B会在低优先级的任务A运行之后才会运行（b被延迟了）。
		这就违背了FreeRTOS的任务调度原则。
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-stm32-SwissArmyKnife-FreeRTOS/f6ff3f33-e77d-4024-81f2-23b849908ca0.webp)
		<empty-block/>
		FreeRTOS的ISR函数中，可以使用两个宏进行任务切换：
		```c
portEND_SWITCHING_ISR( xHigherPriorityTaskWoken );
或
portYIELD_FROM_ISR( xHigherPriorityTaskWoken );
		```
		这两个宏做的事情是完全一样的，在老版本的FreeRTOS中，
		- **portEND_SWITCHING_ISR** 使用汇编实现
		- **portYIELD_FROM_ISR** 使用C语言实现
		新版本都统一使用**portYIELD_FROM_ISR**。
		使用示例如下：
		```c
void XXX_ISR()
{
    int i;
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    
    for (i = 0; i < N; i++)
    {
        xQueueSendToBackFromISR(..., &xHigherPriorityTaskWoken); /* 被多次调用 */
    }
	
    /* 最后再决定是否进行任务切换 
     * xHigherPriorityTaskWoken为pdTRUE时才切换
     */
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}
		```
		<br>在中断服务函数中，如果唤醒了更高优先级的任务，那么xHigherPriorityTaskWoken变量会变为pdTRUE。也就是再发起一次调度，来调度这个任务。
# 26.互斥操作的本质 {toggle="true"}
	在前面讲解互斥量时，引入过临界资源的概念。在前面课程里，已经实现了临界资源的互斥访问。
	本章节的内容比较少，只是引入两个功能：屏蔽/使能中断、暂停/恢复调度器。
	要独占式地访问临界资源，有3种方法：
	- 公平竞争：比如使用互斥量，谁先获得互斥量谁就访问临界资源，这部分内容前面讲过。
	- 谁要跟我抢，我就灭掉谁：
		- 中断要跟我抢？我屏蔽中断
		- 其他任务要跟我抢？我禁止调度器，不运行任务切换
	## **18.1 屏蔽中断**
	屏蔽中断有两套宏：任务中使用、ISR中使用：
	- 任务中使用：**taskENTER_CRITICA()/taskEXIT_CRITICAL()**
	- ISR中使用：**taskENTER_CRITICAL_FROM_ISR()/taskEXIT_CRITICAL_FROM_ISR()**
	### **18.1.1 在任务中屏蔽中断**
	在任务中屏蔽中断的示例代码如下：
	```plain text
/* 在任务中，当前时刻中断是使能的
 * 执行这句代码后，屏蔽中断
 */
taskENTER_CRITICAL();
​
/* 访问临界资源 */
​
/* 重新使能中断 */
taskEXIT_CRITICAL();
	```
	在 **taskENTER_CRITICA()/taskEXIT_CRITICAL()** 之间：
	- 低优先级的中断被屏蔽了：优先级低于、等于 **configMAX_SYSCALL_INTERRUPT_PRIORITY**
	- 高优先级的中断可以产生：优先级高于 **configMAX_SYSCALL_INTERRUPT_PRIORITY**
		- 但是，这些中断ISR里，不允许使用FreeRTOS的API函数
	- 任务调度依赖于中断、依赖于API函数，所以：这两段代码之间，不会有任务调度产生
	这套 **taskENTER_CRITICA()/taskEXIT_CRITICAL()** 宏，是可以递归使用的，它的内部会记录嵌套的深度，只有嵌套深度变为0时，调用 **taskEXIT_CRITICAL()** 才会重新使能中断。
	使用 **taskENTER_CRITICA()/taskEXIT_CRITICAL()** 来访问临界资源是很粗鲁的方法：
	- 中断无法正常运行
	- 任务调度无法进行
	- 所以，之间的代码要尽可能快速地执行
	### **18.1.2 在ISR中屏蔽中断**
	要使用含有"FROM_ISR"后缀的宏，示例代码如下：
	```plain text
void vAnInterruptServiceRoutine( void )
{
    /* 用来记录当前中断是否使能 */
    UBaseType_t uxSavedInterruptStatus;

    /* 在ISR中，当前时刻中断可能是使能的，也可能是禁止的
     * 所以要记录当前状态, 后面要恢复为原先的状态
     * 执行这句代码后，屏蔽中断
     */
    uxSavedInterruptStatus = taskENTER_CRITICAL_FROM_ISR();

    /* 访问临界资源 */

    /* 恢复中断状态 */
    taskEXIT_CRITICAL_FROM_ISR( uxSavedInterruptStatus );
    /* 现在，当前ISR可以被更高优先级的中断打断了 */
}
	```
	在 **taskENTER_CRITICA_FROM_ISR()/taskEXIT_CRITICAL_FROM_ISR()** 之间：
	- 低优先级的中断被屏蔽了：优先级低于、等于 **configMAX_SYSCALL_INTERRUPT_PRIORITY**
	- 高优先级的中断可以产生：优先级高于 **configMAX_SYSCALL_INTERRUPT_PRIORITY**
		- 但是，这些中断ISR里，不允许使用FreeRTOS的API函数
	- 任务调度依赖于中断、依赖于API函数，所以：这两段代码之间，不会有任务调度产生
	## **18.2 暂停调度器**
	如果有别的任务来跟你竞争临界资源，你可以把中断关掉：这当然可以禁止别的任务运行，但是这代价太大了。它会影响到中断的处理。
	如果只是禁止别的任务来跟你竞争，不需要关中断，暂停调度器就可以了：在这期间，中断还是可以发生、处理。
	使用这2个函数来暂停、恢复调度器：
	```plain text
/* 暂停调度器 */
void vTaskSuspendAll( void );

/* 恢复调度器
 * 返回值: pdTRUE表示在暂定期间有更高优先级的任务就绪了
 *        可以不理会这个返回值
 */
BaseType_t xTaskResumeAll( void );
	```
	示例代码如下：
	```plain text
vTaskSuspendScheduler();

/* 访问临界资源 */

xTaskResumeScheduler();
	```
	这套 **vTaskSuspendScheduler()/xTaskResumeScheduler()** 宏，是可以递归使用的，它的内部会记录嵌套的深度，只有嵌套深度变为0时，调用 **taskEXIT_CRITICAL()** 才会重新使能中断。
# 27.优化系统-精细调整栈大小 {toggle="true"}
	## 原理 {toggle="true"}
		栈来自堆，任务多，堆就可能不够。我们需要精确计算每个任务用到的堆是多大
		<empty-block/>
		对于每个任务，会分得一块内存。
		整块内存都会会初始化为0xA5A5A5A5
		初始化完毕后让栈指向第一个位置，
		在函数调用过程中，栈指针会从上往下移动
		在程序中就会用到栈，里面会填入一些数值。数值大概率不会是A5A5A5A5
		于是这里面的数据就被破坏了，就不等于0xA5A5A5A5
		这样我们就可以从栈的最低端，往上去遍历栈，数一下没有被破坏的空间有多大，这一块空间就是空闲栈。
		<empty-block/>
		这样就可以根据这个值，来调整栈大小。
		`UBaseType_t uxTaskGetStackHighWaterMark( TaskHandle_t xTask );`  返回值为4字节
		<empty-block/>
		<empty-block/>
		<empty-block/>
		```plain text
TaskHandle_t xTask_Handle;  //任务句柄
UBaseType_t freeNum;        //获得剩余值


/* 获得当前任务的句柄 */
xTask_Handle =  xTaskGetCurrentTaskHandle();
/* 获得当前任务栈剩余空间 */
freeNum = uxTaskGetStackHighWaterMark(xTask_Handle);
/* 打印 */
printf("FreeStack of Task  %s : %d\\n\\r" , pcTaskGetName(xTask_Handle), freeNum);

		```
	## 使用**vtasklist**获得任务的统计信息 {toggle="true"}
		使用vtask list 需要使用CubeMX 工具来配置FreeRTOS 的 USE STATS FORMATTING FUNCTIONS配置项，为Enable
		<empty-block/>
		### **使用钩子函数的前提**
		在FreeRTOS\\Source\\tasks.c中，可以看到如下代码，所以前提就是：
		- 把这个宏定义为1：configUSE_IDLE_HOOK
		- 实现vApplicationIdleHook函数
# 28.优化系统-统计CPU占比 找出有问题的程序 {toggle="true"}
	例如MPU6050在运行时，会不断产生中断，而在中断中如果我们进行了写队列操作，那么这个任务就会占用很多的CPU资源。
	那么我们可以使用
	## ** 任务运行时间统计** {toggle="true"}
		需要再CubeMX中配置FreeRTOS 的GENERATE RUN TIME STATS为Eeable
		然后需要配置FreeRTOS.c文件中 的 需要自己实现：`weak unsigned long getRunTimeCounterValue (void)` ，自己编写一个强声明的函数，返回系统启动后过了多少时间，单位微妙。（一个函数，自己编写）
		使用vTaskGetRunTimeStats(…);函数，可以把信息存储在buff中，
		<empty-block/>
		对于同优先级的任务，它们按照时间片轮流运行：你执行一个Tick，我执行一个Tick。
		是否可以在Tick中断函数中，统计当前任务的累计运行时间？
		不行！很不精确，因为有更高优先级的任务就绪时，当前任务还没运行一个完整的Tick就被抢占了。
		我们需要比Tick更快的时钟，比如Tick周期时1ms，我们可以使用另一个定时器，让它发生中断的周期时0.1ms甚至更短。
		使用这个定时器来衡量一个任务的运行时间，原理如下图所示：
		![](http://photos.100ask.net/rtos-docs/FreeRTOS/DShanMCU-F103/chapter-19/image3.png)
		- 切换到Task1时，使用更快的定时器记录当前时间T1
		- Task1被切换出去时，使用更快的定时器记录当前时间T4
		- (T4-T1)就是它运行的时间，累加起来
		- 关键点：在 **vTaskSwitchContext** 函数中，使用 **更快的定时器** 统计运行时间
# 99.小技巧 {toggle="true"}
	**在 FreeRTOS 中后创建的同优先级任务先运行的原因**
	- FreeRTOS 中有一个 `pxCurrentTCB` 指针，当创建新的任务时，如果该任务的优先级最高（包括与已存在的任务优先级相同的情况），那么该指针就会指向该任务。启动任务的时候，会从 `pxCurrentTCB` 指针所指的任务开始执行，这导致后创建的同优先级任务先运行。
	<empty-block/>
	<empty-block/>
	<empty-block/>
	Ctrl+B 删除所有断点
	中断应该是唤醒一个任务，让任务去读取I2C。否则太慢了。会卡一下
	<empty-block/>
	队列长度一般设置为宏定义，这样方便在队列集的时候相加
	<empty-block/>
	#include CMSIS_device_header 错误 把版本调到1.8.5就OK 了
	<empty-block/>
	CubeMX重新生成代码会使中文注释覆盖。需要在系统环境变量中新建变量名为JAVA_TOOL_OPTIONS    变量值为-Dfile.encoding=UTF-8  的 系统变量  这样再生成就没问题了
	<empty-block/>
	<empty-block/>
	<empty-block/>
	```c
static const byte roadMarking[] PROGMEM ={
0x01,0x01,0x01,0x01,0x01,0x01,0x01,0x01,
};
	```
	- `static const byte`：声明一个静态的常量数据类型为`byte`（通常可能是一个 8 位无符号整数类型）。
	- `roadMarking`：数组的名称。
	- `[]`：表示这是一个数组。
	- `PROGMEM`：指示编译器将这个数组存储在程序存储器中，而不是常规的数据存储器，这样可以节省宝贵的 RAM（随机存取存储器）空间，尤其是在资源受限的嵌入式系统中。
	- `{0x01,0x01,0x01,0x01,0x01,0x01,0x01,0x01}`：初始化这个数组，这里有 8 个值为`0x01`的元素。每个`0x01`表示一个十六进制的数值，对应的十进制值为 1。
	<empty-block/>
	<empty-block/>
	**函数创建任务踩的坑**
	在使用函数创建任务时，不要单独写个任务去创建了。 直接初始化里调用函数就OK 了
	如果非要用任务， 记得在调用完后最后一行 自杀。
	（或者在任务的最后添加while循环 和延时……….）
<empty-block/>
<empty-block/>
<callout icon="💡" color="gray_bg">
	有关其他FreeRTOS的问题，欢迎在底部评论区留言，一起交流\~
</callout>
<empty-block/>