<callout icon="😀" color="gray_bg">
	看江协的视频，板子是STM32F103C8T6最小系统板。
	看江协科技的视频为主，这是我在学完c语言之后首次接触嵌入式。
	以下是我的笔记\~
</callout>
# **0.  资料在哪准备** {toggle="true"}
	1. [芯片包的下载](https://www.keil.arm.com/devices/)
	2. [数据手册和参考手册](https://www.st.com/en/microcontrollers-microprocessors/stm32-32-bit-arm-cortex-mcus.html)
	3. [STM32标准外设库下载](https://www.st.com/en/embedded-software/stm32-standard-peripheral-libraries.html)
# **1.   创建标准库的工程模板** {toggle="true"}
	这一部分只是创建了一个基础的标准库模板。并没有Hardware等文件
	## 1.解压库函数的压缩包并打开 {toggle="true"}
		库函数为：`STM32F10x_StdPeriph_Lib_V3.5.0`，这个可以去官网下载。
		解压后文件夹内容如下
		- Libraries是库函数的文件
		- Project是官方提供的工程示例和模板
		- Utilities是官方做的一个小电路板，存放的用来测评stm32的程序
		- Release_Notes.html是发布文档
		- stm32f10x_stdperiph_lib_um.chm为库函数使用手册
	## 2.建立工程模板，尝试使用寄存器/标准库来点灯 {toggle="true"}
		**本章建立了工程模板**。**并尝试用寄存器和库函数来进行点灯操作**
		建立工程模板是为了配置好STM32的环境，并且方便下次去建立工程。不用每次都新建一次。
		1. 建立工程文件是为了方便管理。可以起名为`STM32Project`
		2. 在Keil上新建工程，起名后  选择芯片型号（这里为STM32F103C8）
		### 一、添加Start启动文件夹 {toggle="true"}
			1. 新建Start文件，用来存放STM32的启动文件，STM32运行时会先运行它们
				启动文件的存放位置位于库函数的压缩包中的Libraries文件中。下面的位置可以参考用
				`\STM32F10x_StdPeriph_Lib_V3.5.0\Libraries\CMSIS\CM3\`<span color="yellow">`DeviceSupport`</span>`\ST\STM32F10x\startup\arm`
				**我们把这里面的所有文件都粘贴到Start文件中。**
				往回走，可以在这个目录下找到一些头文件
				`\STM32F10x_StdPeriph_Lib_V3.5.0\Libraries\CMSIS\CM3\`<span color="yellow">**`DeviceSupport`**</span>`\ST\STM32F10x`
				- stm32f10x.h  -  STM32寄存器外设描述文件，是用来描述stm32有哪些寄存器，和他对应的地址
				- system_stm32f10x.c    -    这两个`system_stm32f10x`文件主要是用来配置时钟的（STM32的72MHZ主频就是在这里配置的）
				- system_stm32f10x.h    。
				**上面三个同上，也是全部放到Start文件中**
				因为STM32是内核和内核外围的设备组成的，并且内核寄存器描述和外围寄存器描述是不在一起的，所以要添加内核寄存器的描述文件
				这次就不是打开<span color="yellow">`D`</span><span color="yellow">**`eviceSupport`**</span>** **文件了，而是打开内核的<span color="blue">**`CoreSupport`**</span>文件
				- core_cm3.c  -  这两个就是内核的寄存器描述文件.h以及配置文件.c
				- core_cm3.h
			2. Keil添加启动文件Start。`.s`的启动文件只需要添加一个。但他有分类
				<empty-block/>
				所以我们选择`_md.s`后缀的文件
				添加后再把其他的.c  .h后缀的文件都添加进来（要选择筛选为ALL，否则看不全）
			3. 在工程选项中添加文件的路径，否则找不到。<br>并且这样也有利于移动文件，因为添加后工程是以目录所在的文件位置去找文件，而不是使用绝对路径去找。可以很方便的打包工程发给别人
				1. 打开魔术棒，在C/C++选项中找到Include Paths的框框，点三个点找到Start的文件夹。然后确定。
		### 二、添加User用户文件夹 {toggle="true"}
			1. 在工程模板的文件夹下添加User文件夹，用来存放用户写的代码
			2. 在Keil软件的Start的上一个目录Target 1上右键，点击添加组。然后重命名为Start。<br>并在魔术棒→  C/C++中，添加好头文件路径 Include Path 
			3. Keil软件上右键Start添加新文件，选择.c输入ain.c文件然后选择存放路径到User文件中，否则会默认放到文件夹外的工程目录中。
				（这里我已经在魔术棒上设置好Tab=4个空格和显示空格等配置了。）<br>（并且魔术棒按钮的Target块中选择的是ARMCode编译器为v5.06）<br>（在扳手工具处已经设置字号为14，编码格式为：GB2312）
			4. 在里边输入
				```c
#include "stm32f10x.h"  //头文件

int main()
{
    while(1)
    {
    
    }
}
//注意最后一行得多一行空格，不然编译会报错。
				```
				此时编译（编译分为全局编译和单文件编译，全局编译是对所有文件编译，单文件则是对当下的文件进行编译。这里先选择单文件就够了）就可以看到0错误0警告。
			5. 把STINK和系统板插好，然后连接电脑。
			- 3.3V对应3.3V
			- GND对应GND
			- SWDIO对应SWDIO
			- SWCLK对应SWCLK
			1. 点击魔术棒选择Debug，设置Use为STink调试器（ST-Link Debugger），然后点击右边的设置按钮。在flash下载的选项中，把Reset and Run 勾选上，这样每次下载程序后会自动复位并执行，不需要按复位按键了
				（这里我已经下载好虚拟串口的驱动CH340了，不下是发现不了设备的）
			2. 编译一下，可以看到0警告0错误
		### 三、尝试用寄存器点亮LED灯 {toggle="true"}
			**当弄到现在，创建好User文件夹对于寄存器开发者来说，已经建立好工程模板了。**
			> **可以先尝试一下寄存器点灯**
			这里我点的灯位于GPIOC，
			寄存器点灯只需要配置三个寄存器。
			1. **首先是配置RCC寄存器，使能GPIOC的时钟，**<br>要使能GPIOC的时钟。而GPIO是挂载到APB2的总线上的。
				打开STM32F10xxx参考手册（中文）手册后可以看到IOPC是在RCC_APB2ENR总线上（偏移地址：0x18）的第4位控制。并且下面有说明这一项为1就开启。
				所以把这一项写1就可以开启他的时钟了。
				整个寄存器的2进制数据换成16进制就是0 0 0 0    0 0 1 0
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e8ef9717-f47b-41cb-8f54-6941a5915a4e.webp)
				**所以写上代码****`RCC->APB2ENR = 0x00000010;`****就可以打开GPIOC的时钟了。**
			2. 然后第二个寄存器，要配置PC13的端口的模式
				在手册中的通用和复用I/O中可以找到GOIO寄存器的一小节。在里面找到**端口配置高寄存器（GPIOx_CRH）**的一小节。<br>(0-7的前八位是在低寄存器里，8-15是在高寄存器里配置)<br>其中的CNF13 和MODE13就是用来配置端口PC13的
				这里我们需要将端口配置为 通用推挽输出模式也就是CNF13的两位要为**00**，<br>MODE则是设置为输出模式，最大速度为50MHZ,也就是MODE两位要为**11**
				最后按照16进制，把32位二进制位转换为16进制的数字<br>也就是把0 0 3 0   0 0 0 0 
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8b8cefe7-98f5-4d1f-83b3-f4f7f5a0f4b3.webp)
				**所以写上代码****`GPIOC->CRH = 0x00300000;`****就配置PC13口为推挽输出模式，速度为50MHz了**
			3. 下一步就是配置端口输出数据寄存器
				在手册中的通用和复用I/O中可以找到GOIO寄存器的一小节。<br>在里面找到**端口输出数据寄存器（GPIOx_ODR）**的一小节。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/867ba4fd-50fa-4ada-be2d-59a1fe05b877.webp)
				这一位写一就是把PC13的输出设置为高电平了。
				把这32位2进制换算为16进制，就是0 0 0 0   2 0 0 0 
				**所以写上代码****`GPIOC->ODR = 0x00002000;`****就配置PC13口为高电平**
				因为这个板子的灯是低电平点亮，所以配置为全0就能点亮灯了**`GPIOC->ODR = 0x00000000;<br>`**下边那个灯就是我点亮的。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1b2c26ea-45b3-4723-90f7-923987d7e2f3.webp)
				> 可以看到寄存器点灯十分的麻烦，得去手册查寄存器。而寄存器那么多，根本不可能记得完。并且我刚才点灯是把除了PC13之外的位都设置成了0。,这样会影响其他端口的正常配置，如果要不影响的话还得用&=或者\|=。就会更加麻烦。所以寄存器的操作方式，虽然代码简洁，但是操作起来很不方便<br><br>下面我就要去学习库函数的使用，来用库函数点灯！！
		### \*\*\*、尝试用库函数来点亮LED灯 {toggle="true"}
			<empty-block/>
		### 四、添加Library文件 {toggle="true"}
			**本章添加了库函数到Library文件，并在Keil软件中定义了库函数头文件。**
			Library是用来存放库函数的，让我们可以使用。
			1. 在工程目录新建Library的文件夹。
			2. 添加库函数到新建的Library文件
				库函数位于库函数压缩包的目录为：**`\STM32F10x_StdPeriph_Lib_V3.5.0\Libraries\STM32F10x_StdPeriph_Driver\src`** 中了。其中`STM32F10x_StdPeriph_Lib_V3.5.0` 为STM公司推出的标准外设库。
				其中的文件就是库函数的源文件了。
				misc.c  为内核的库函数。其余的都是内核外的外设库函数。<br>这里就Ctrl+A全选，全部复制到Library文件夹中
				然后再打开固件库的inc文件夹，目录就在上一级。<br>`\STM32F10x_StdPeriph_Lib_V3.5.0\Libraries\STM32F10x_StdPeriph_Driver\inc`
				这个文件夹中放的是库函数的头文件，也把他复制到Library文件夹中。
			3. 接着回到keil软件，在target 1 处右键，添加组，重命名，然后添加文件到组
				> 但是到这里，对于库函数来说还是不能直接使用的，需要再添加三个文件<br>它位于固件库目录的：**`\STM32F10x_StdPeriph_Lib_V3.5.0\Project\STM32F10x_StdPeriph_Template`**
				可以看到几个文件
				- stm32f10x_conf  -  用来配置库函数头文件的包含关系，还有用来参数检查的函数定义，这是所有库函数都需要的
				- 剩下的两个it后缀的是用来存放中断函数的
				把这三个文件放到User的目录中。接着在keil软件的User组中添加这三个文件。**并在魔术棒→  C/C++中，添加好头文件路径 Include Path **
			4. 完成后，就需要添加库函数的头文件了。
				先对`#include "stm32f10x.h" `的`stm32f10x.h` 右键，选择打开`stm32f10x.h` 滑到最下边可以看到这样一段代码：
				```c
#ifdef USE_STDPERIPH_DRIVER
  #include "stm32f10x_conf.h"
#endif
				```
				这是C语言中的条件编译语句，意思是如果我定义了`USE_STDPERIPH_DRIVER`。下面的这个`#include "stm32f10x_conf.h"`才有效
				这需要我们点击魔术棒按钮，在C/C++的选项中，在Define框框中输入`USE_STDPERIPH_DRIVER` 这样才能包含标准库函数。
				（记得把每个组（文件）的路径添加好头文件路径 Include Path，在魔术棒的C/C++ 中）
				可以在KEil软件看到。<br>User中的文件，我们是可以修改的，而Start和Library文件中的东西都带有小钥匙<br>是改不了的。我们可以点击魔术棒旁边的小箱子来拖动挑中左边的组的排序位置。
				比如把User放到最下边，其他放上边，这样的话平时不用就可以收起来。
				<empty-block/>
				下面再编译一下就可以看到成功了。<br>（单文件编译，如果新添加了文件会默认使用全局编译一次。所以比较慢）
				<empty-block/>
		### 五、使用库函数进行点灯操作 {toggle="true"}
			**本小节使用库函数进行点灯。**
			库函数本质还是配置寄存器，不过是间接的了。
			1. 第一步还是使能GOIOC的时钟<br>它位于APB2的外设上。
				库函数中用来开启时钟的函数是`RCC_APB2PeriphClockCmd` <br>大概翻译一下就是RCC_APB2  外设  时钟  命令（控制）
				输入完后会提示出来这个函数要输入两个参数。
				**可以在编译后按F12或者右键跳到函数定义来看他的参数需要填什么，都会有介绍。**
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1151c5fb-22ac-41c1-b82d-fe532e18a79c.webp)
				简介中会说，这个函数是让APB2的时钟使能或者失能的。他的参数可以是下面这些。
				**我们这里用的是****`RCC_APB2Periph_GPIOC`**** 这一项，然后填写到函数的第一个参数就OK**
				**第二个参数是****`ENABLE`**** 使能。**
				**这样就可以使能GPIOC的时钟了。**
				通过F12下这个函数，来查看这个函数的内部代码。其实他只是包装了一下。实际上还是配置寄存器的。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/120c373e-6625-475b-9b6e-d52fce0dda55.webp)
			2. 第二步还是配置端口的模式。
				这里用的库函数是`GPIO_Init`，F12查看定义，可以知道他的两个参数
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e0671a4e-cedc-4bdb-b1f4-6e1123bd0991.webp)
				第一个参数是选择GPIO。x可以是A-G中的数字<br>所以第一个参数就是GPIOC。
				第二个参数是一个`GPIO_InitTypeDef `的结构体**的指针（是指针！）**。我们需要先定义一个结构体。结构体的名字根据官方的推荐，最好叫`GPIO_InitStructure。`
				代码：`GPIO_InitTypeDef GPIO_InitStructure;`
				然后复制结构体的名字，用` . ` 操作符来配置结构体内部的变量。
				```c
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = 
    GPIO_InitStructure.GPIO_Pin = 
    GPIO_InitStructure.GPIO_Speed = 
				```
				> 这里可以现在Mode的一行按F12 ，跳转到Mode参数的定义。
				他说：Mode的值在`GPIOMode_TypeDef`（这是一个枚举变量）里可以找到。那么就可以在当前文件Ctrl+F搜一下。找到GPIOMode_TypeDef这个枚举变量的位置。<br>这里我们用到的是`GPIO_Mode_Out_PP` ：通用推挽输出。<br>然后把这个参数放到Mode = 后就可以了。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8f0af559-9de8-461c-9493-f82b7a6ff94f.webp)
				> 下面就是配置GPIO_Pin的参数了,
				在使用F12跳转的时候，会显示一个框，这是因为他的定义后很多个。<br>这里需要的是member这一项，双击就可跳转过去了。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/acbcd95a-53bd-476e-afb8-6d30641450dc.webp)
				（其实还是刚才GPIO_Mode跳转过去的那个位置）
				他说Pin的值在`GPIO_pins_define`（这是一个枚举变量）里可以找到
				仍然是Ctrl+F搜一下。
				可以看到这里是#define的宏定义列表
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5aa73325-30b2-45d1-b331-db9de92bb11b.webp)
				这里我们需要用到的是GPIO_Pin_13
				仍然是复制粘贴过去
				> Speed的参数也是同理
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9de17bfd-b38d-4f26-82c5-f181be312828.webp)
				这里选择`GPIO_Speed_50MHz`
				致此，我们定义的结构体变量`GPIO_InitTypeDef GPIO_InitStructure;`的参数就配置完全了。<br>下面就可以把结构体变量的**地址（是传地址！！）**放到配置GPIO模式的函数`GPIO_Init` 的第二个参数中了。
				```c
    //配置PC13的端口为推挽输出模式，速度为50MHz
    GPIO_InitTypeDef GPIO_InitStructure;                //定义GPIO_Init的第二个参数：结构体变量
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOC,&GPIO_InitStructure);//GPIO_Init，配置GPIO模式
				```
			3. 第三步：配置GPIO口为高/低电平
				这里用到的函数是<br>`GPIO_SetBits`  - 设置GPIO为高电平<br>`GPIO_ResetBits`  -  设置GPIO为低电平
				参数则都是GPIOx，GPIO_Pin_x<br>要看定义也是同上F12
			4. 编译！！
				代码如下，如果编译的时候报错：
				```c
User\main.c(15): error:  #268: declaration may not appear after executable statement in block
GPIO_InitTypeDef GPIO_InitStructure;                //定义GPIO_Init的第二个参数：结构体变量
				```
				他的意思这一行的声明位置。C 语言要求变量声明通常应在可执行语句之前。这违反了 C 语言的语法规则，想让我把`GPIO_InitTypeDef GPIO_InitStructure;` 放到第一行声明。我能听他的？
				**解决办法就是：在魔术棒→ C/C++处勾选C99Mode（C语言的C99标准）**，这样就不会报错了。
				```c
    //库函数点灯
    
    //打开GPIOC的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC,ENABLE);
    
    //配置PC13的端口为推挽输出模式，速度为50MHz
    GPIO_InitTypeDef GPIO_InitStructure;                //定义GPIO_Init的第二个参数：结构体变量
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOC,&GPIO_InitStructure);//GPIO_Init，配置GPIO模式
    //设置PC 13为低电平。点灯。
    GPIO_ResetBits(GPIOC,GPIO_Pin_13);
				```
				点灯如下：
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1415407c-e6b1-464f-a4b4-eea2e8b2b6f7.webp)
# **2.   GPIO的工作原理** {toggle="true"}
	GPIO如果只是单独的进行输入输出就太浪费了。所以芯片的GPIO口还可以复用为外设功能引脚（比如串口）
	GPIO的4种输入模式
	- 输入浮空 **GPIO_Mode_IN_FLOATING**
	- 输入上拉 **GPIO_Mode_IPU**
	- 输入下拉 **GPIO_Mode_IPD**
	- 模拟输入 **GPIO_Mode_AIN**
	GPIO的4种输出模式
	- 开漏输出（带上拉或者下拉）   **GPIO_Mode_Out_OD**
	- 开漏复用功能（带上拉或者下拉）**GPIO_Mode_AF_OD**
	- 推挽式输出（带上拉或者下拉） **GPIO_Mode_Out_PP**
	- 推挽式复用功能（带上拉或者下拉）**GPIO_Mode_AF_PP**
	4种最大输出速度（这里与F1不同）
	- 2M
	- 25M
	- 50M
	- 100M
	**F4的**芯片手册中，GPIo表格只要有FT标识，就代表他可以容忍5V的输入
	**F4与F1的不同就是这里的上下拉电阻被移动到了保护二极管那边，而不是在里边。**
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5150ccda-2f2c-4e24-b2c6-17150d75f15d.webp)
	1. 浮空输入模式：
		既不上拉又不下拉，输入信号直接经过施密特触发器，然后存入寄存器，让芯片读取
	2. 输入上/下拉模式
		区别就是输入电平会经过上拉或者下拉电阻拉高或拉低
	3. 模拟输入
		此时不**经过施密特触发器**来转换为高低电平。是跳过触发器直接到AD（A是模拟，D是数字）这里
	4. 开漏输出
		输出强低电平。
		输出时，首先是操作位设置寄存器（间接去操作输出数据寄存器）或者输出数据寄存器。然后通过输出控制电路来控制Nmos管是否接地，当输出控制电路输出1时，Nmos为断开，此时的GPIO为高阻态，如果想输出高电平。，就需要配置电阻为上拉。简单来说，开漏输出只可以输出强低电平，高电平得靠外部电阻拉高。显然，这种输出方式就有一个优点，由于高电平完全由外部电阻控制，那此模式下的输出电平是可以通过改变电阻而改变的
	5. 开漏复用功能
		通过复用功能外设来控制输入输出，不是那俩寄存器。其他的都是一样的
	6. 推挽输出
		输出强高低电平。
		输出时，首先是操作位设置寄存器（间接去操作输出数据寄存器）或者输出数据寄存器。然后通过输出控制电路来控制两个NMOS管的通断，来控制输出的高低电平。他也可以去配置上下拉电阻，
	7. 推挽式复用功能
		通过复用功能外设来控制输入输出，不是那俩寄存器。其他的都是一样的
		**每组（16个IO口）GPIO端口的寄存器包括:**
		- 4个32位配置寄存器
			- 1个端口模式寄存器(GPIOx MODER)
			- 1个端口输出类型寄存器(GPIOx OTYPER)
			- 1个端口输出速度寄存器(GPIOx OSPEEDR)
			- 1个端口上拉下拉寄存器(GPIOx PUPDR)
		- 2个32位数据寄存器
			- 1个端口输入数据寄存器(GPIOx IDR)
			- 1个端口输出数据寄存器(GPIOx ODR)
		- 1个端口置位/复位寄存器(GPIOx BSRR)
		- 1个端口配置锁存寄存器(GPIOx LCKR)
		- 两个复位功能寄存器(低位GPIOx AFRL & GPIOx AFRH)  （这个比较重要）
		<empty-block/>
		所有的IO口都可以用作中断的输入
		<empty-block/>
	<empty-block/>
# **3.   标准库的LED跑马灯** {toggle="true"}
	这里的工程文件，我是用的是江协的工程文件，在其基础上添加了Hardware文件夹和Delay文件夹
	并在Keil中添加和保存文件相对路径。
	代码如下：
	`main.c：`
	```c
#include "stm32f10x.h"                  // Device header
#include "led.h"
#include "delay.h"//直接拿的延时函数
int main()
{

    
    LED_Init();//初始化LED


    while(1)
    {
        GPIO_SetBits(GPIOB,GPIO_Pin_15);//PB15为高电平，熄灭
        GPIO_ResetBits(GPIOB,GPIO_Pin_12);//PB12为低电平，点亮
        Delay_ms(500);//延时500ms
        GPIO_SetBits(GPIOB,GPIO_Pin_12);//PB12为高电平，熄灭
        GPIO_ResetBits(GPIOB,GPIO_Pin_13);//PB13为低电平，点亮
        Delay_ms(500);//延时500ms
        GPIO_SetBits(GPIOB,GPIO_Pin_13);//PB13为高电平，熄灭
        GPIO_ResetBits(GPIOB,GPIO_Pin_14);//PB14为低电平，点亮
        Delay_ms(500);//延时500ms
        GPIO_SetBits(GPIOB,GPIO_Pin_14);//PB14为高电平，熄灭
        GPIO_ResetBits(GPIOB,GPIO_Pin_15);//PB15为低电平，点亮
        Delay_ms(500);//延时500ms
    }
}

	```
	`LED.c`：
	```c
#include "led.h"
#include "stm32f10x.h"

void LED_Init(void)
{
    //跑马灯引脚为PB 12 13 14 15,LED为低电平有效
    
        
    //F4的芯片在这里还需要配置上下拉电阻PuPd 以及 推挽或者开漏Otype
    
    //打开GPIOB的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
    //定义GPIO_Init的第二个参数：结构体变量
    GPIO_InitTypeDef GPIO_InitStructure;                
    //↑在C99下不需要放到第第一行
    
    
    //下面的GPIO是分开配置的，其实可以写成A||B.这样就不用写那么多次了。这里我就不改了
    
    
    //PB12
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_12;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    //配置PC13的端口为推挽输出模式，速度为50MHz
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);//GPIO_Init，初始化GPIO B，PB12的GPIO
    GPIO_SetBits(GPIOB,GPIO_Pin_12);//设置GPIO为高电平。LED为熄灭
    //
    
   //PB13
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);
    GPIO_SetBits(GPIOB,GPIO_Pin_13);
    
   //PB14
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_14;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);
    GPIO_SetBits(GPIOB,GPIO_Pin_14);
    
    
   //PB15
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_15;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);
    GPIO_SetBits(GPIOB,GPIO_Pin_15);

}

	```
	`LED.h`
	```c
#ifndef __LED_H //条件编译

#define __LED_H

void LED_Init(void);//初始化LED

#endif

	```
	Delay.c就不写上去了。
	结果如图
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9832ac50-c383-410b-9a2c-173a00a5ca96.gif)
	<empty-block/>
# **4.   标准库的按键点灯** {toggle="true"}
	按键检测使用的是gpio的输入功能。函数为
	`KEY1 GPIO_ReadInputDataBit(GPIO_B,GPIO_PIN_11)//读取 GPIOB 11引脚的电平`
	## 按键支持连续按的一般思路 {toggle="true"}
		支持连续按，每次检测只要是按下的状态都会返回有效值<br>思路如下：
		```c
uint8_t KEY_Scan(void)
{
    if(KEY按下)
    {
        delay(10)//延迟10ms，消抖
        if(KEY按下)//确定按下
        {
            return 有效值
        }
    }
    return 0;//否则返回无效值
}
		```
	## 按键不支持连续按的一般思路 {toggle="true"}
		不支持连续按，按下按键后只要不松开就不会返回有效值<br>思路如下:<br>使用了static静态修饰变量
		1. 存储持续性：`static` 修饰的局部变量具有静态存储持续性，它在程序的整个运行期间都存在，而不是在函数调用结束时被销毁。
			- 例如，每次函数调用结束后，`static` 局部变量的值会被保留，下次函数调用时会继续使用上次修改后的值。
		2. 初始化：只在第一次函数调用时进行初始化。
		- 假设一个函数被多次调用，但`static` 局部变量只会在第一次调用时被初始化为指定的值，后续调用不会再次初始化。
		1. 作用域：作用域仍然局限在声明它的函数内部，在函数外部无法直接访问。
		```c
uint8_t KEY_Scan(void)
{
    static key_up = 1//=1表示上一个状态为未被按下
    if(key_up && KEY按下)//如果上个状态未被按下，并且现在KEY按下，那么就进入。
    {
        delay(10)//延迟10ms，消抖
       
        if(KEY按下)//确定按下
        {
            key_up = 0;//标记KEY按下
            return 有效值
        }
    }
    else if (KEY没有按下)
    {
        key_up = 1;//记录上个状态没有按下
    }
    return 0;//否则返回无效值
		```
		1. 存储持续性：`static` 修饰的局部变量具有静态存储持续性，它在程序的整个运行期间都存在，而不是在函数调用结束时被销毁。
			- 例如，每次函数调用结束后，`static` 局部变量的值会被保留，下次函数调用时会继续使用上次修改后的值。
		2. 初始化：只在第一次函数调用时进行初始化。
			- 假设一个函数被多次调用，但`static` 局部变量只会在第一次调用时被初始化为指定的值，后续调用不会再次初始化。
		3. 作用域：作用域仍然局限在声明它的函数内部，在函数外部无法直接访问。
	## 两种模式合二为一的思路 {toggle="true"}
		根据传入的mode值来强制使Key up为1
		```c
uint8_t KEY_Scan(uint8_t mode)
{
    static key_up = 1//=1表示上一个状态为未被按下
    if(mode == 1)//如果mode为1
    {
        key_up = 1;//那么key强制为1.支持连续按。
    }
    if(key_up && KEY按下)//如果上个状态未被按下，并且现在KEY按下，那么就进入。
    {
        delay(10)//延迟10ms，消抖
       
        if(KEY按下)//确定按下
        {
            key_up = 0;//标记KEY按下
            return 有效值
        }
    }
    else if (KEY没有按下)
    {
        key_up = 1;//记录上个状态没有按下
    }
    return 0;//否则返回无效值
}

		```
	## 实现按键点亮对应LED灯 {toggle="true"}
		这里已经在头文件内定义了宏，
		`#define KEY1 GPIO_ReadInputDataBit(GPIO_B,GPIO_PIN_11)//读取 GPIOB 11引脚的宏`
		点灯图如下：
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1171b38b-6dbd-4b18-8725-43d81ba01adf.gif)
		key.文件如下
		```c
#include "key.h"
#include "stm32f10x.h"

//初始化GPIO
//按键1 2是低电平检测。所以模式配置为上拉输入
void KEY_Init(void)
{
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);//使能APB2总线的GPIOC 时钟
    
    GPIO_InitTypeDef GPIO_InitStructure;
    
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;//上拉输入
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_11 | GPIO_Pin_0;//KEY 1  2 对应引脚
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB,&GPIO_InitStructure);//初始化GPIOB 11  0引脚
    
}

//按键检测
uint8_t KEY_Scan(uint8_t mode)
{
    static uint8_t Key_up = 1;//初始化keyup = 1，未被按下
    if(mode == 1)
    {
        Key_up = 1;//如果启动了连续按下模式，那么就强制定义上一次状态为未被按下
    }
    if(Key_up && (KEY1||KEY2))//上一次未被按下，现在KEY1或2按下。那么就有效
    {
        Delay_ms(50);//去抖动
        Key_up = 0;//赋值为0,标记按下
        if     (KEY1)
        {
            return 1;
        }
        else if(KEY2)
        {
            return 2;
        }    
        }
    else if(KEY1 == 0 && KEY2 == 0)
    {
        Key_up = 1;//未被按下，那么赋值为1，标记未被按下。
    }
    return 0;
}

		```
		Key.h文件如下
		```c
#ifndef __KEY_H
#define  __KEY_H

#include "stm32f10x.h"
#include "Delay.h"

//低电平有效，读取到低电平返回1
#define KEY1 (!(GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_11) ))//读取 GPIOB 11引脚的宏
#define KEY2 (!(GPIO_ReadInputDataBit(GPIOB,GPIO_Pin_0)  ))//读取 GPIOB 11引脚的宏

//初始化KEY引脚
void KEY_Init(void);

//按键扫描  1为支持连续按，0为不支持
uint8_t KEY_Scan(uint8_t mode);

#endif

		```
		LED.c文件如下
		```c
#include "led.h"


//初始化LED
void LED_Init(void)
{
    //跑马灯引脚为PB 12 13 14 15,LED为低电平有效
    
        
    //F4的芯片在这里还需要配置上下拉电阻PuPd 以及 推挽或者开漏Otype
    
    //打开GPIOB的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);

    //定义GPIO_Init的第二个参数：结构体变量
    GPIO_InitTypeDef GPIO_InitStructure;                
    //↑在C99下不需要放到第第一行
    
    
    //下面的GPIO是分开配置的，其实可以写成A||B.这样就不用写那么多次了。这里我就不改了
    
    
    //PB12
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_12;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    //配置PC13的端口为推挽输出模式，速度为50MHz
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);//GPIO_Init，初始化GPIO B，PB12的GPIO
    GPIO_SetBits(GPIOB,GPIO_Pin_12);//设置GPIO为高电平。LED为熄灭
    //
    
   //PB13
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);
    GPIO_SetBits(GPIOB,GPIO_Pin_13);
    
   //PB14
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_14;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);
    GPIO_SetBits(GPIOB,GPIO_Pin_14);
    
    
   //PB15
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_15;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_Init(GPIOB,&GPIO_InitStructure);
    GPIO_SetBits(GPIOB,GPIO_Pin_15);
    
}


//熄灭LED
void LED_OFF(uint8_t num)
{
    if(num == 0)
    {
        return;
    }
    if(num == 1)
    {
        GPIO_SetBits(GPIOB,GPIO_Pin_12);
    }
    else if(num == 2)
    {
        GPIO_SetBits(GPIOB,GPIO_Pin_13);
    }
    else if(num == 3)
    {
        GPIO_SetBits(GPIOB,GPIO_Pin_14);
    }
    else if(num == 4)
    {
        GPIO_SetBits(GPIOB,GPIO_Pin_15);
    }
}

//点亮LED
void LED_ON(uint8_t num)
{
    if(num == 0)
    {
        return;
    }
    if(num == 1)
    {
        GPIO_ResetBits(GPIOB,GPIO_Pin_12);
    }
    else if(num == 2)
    {
        GPIO_ResetBits(GPIOB,GPIO_Pin_13);
    }
    else if(num == 3)
    {
        GPIO_ResetBits(GPIOB,GPIO_Pin_14);
    }
    else if(num == 4)
    {
        GPIO_ResetBits(GPIOB,GPIO_Pin_15);
    }
}

//反转LED
void LED_Flip(uint8_t num)
{
    if(num == 0)
    {
        return;
    }
    else if(num == 1)
    {
        if(GPIO_ReadOutputDataBit(GPIOB, GPIO_Pin_12) == Bit_SET)
        {
            GPIO_ResetBits(GPIOB,GPIO_Pin_12);
        }
        else
        {
            GPIO_SetBits(GPIOB,GPIO_Pin_12);
        }
    }
    else if(num == 2)
    {
        if(GPIO_ReadOutputDataBit(GPIOB, GPIO_Pin_13) == Bit_SET)
        {
            GPIO_ResetBits(GPIOB,GPIO_Pin_13);
        }
        else
        {
            GPIO_SetBits(GPIOB,GPIO_Pin_13);
        }
    }
    else if(num == 3)
    {
        if(GPIO_ReadOutputDataBit(GPIOB, GPIO_Pin_14) == Bit_SET)
        {
            GPIO_ResetBits(GPIOB,GPIO_Pin_14);
        }
        else
        {
            GPIO_SetBits(GPIOB,GPIO_Pin_14);
        }
    }
    else if(num == 4)
    {
        if(GPIO_ReadOutputDataBit(GPIOB, GPIO_Pin_15) == Bit_SET)
        {
            GPIO_ResetBits(GPIOB,GPIO_Pin_15);
        }
        else
        {
            GPIO_SetBits(GPIOB,GPIO_Pin_15);
        }
    }
}

		```
		LED.h文件如下
		```c
#ifndef __LED_H //条件编译
#define __LED_H


#include "stm32f10x.h"

//初始化LED
void LED_Init(void);

//点亮LED
void LED_ON(uint8_t num);

//熄灭LED
void LED_OFF(uint8_t num);

//反转LED
void LED_Flip(uint8_t num);
#endif


		```
# **5.  MDK中寄存器地址名称映射的再理解** {toggle="true"}
	举个栗子就比较明白了。
	比如（这里的外设是不准确的，具体要看手册里的。）
	外设基地址为              `0x01000000`<br>APB1的外设基地址为`0x00100000`
		- GPIOA外设是挂载在AHB1上的。<br>GPIOA地址为 `0x00110000`
	APB2的外设基地址为`0x00200000`
		- I2C外设是挂载在AHB2上的。<br>I2c的地址为   `0x00210000`
	看图大概就是这样子
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e19c867e-bc94-43d1-bd93-6940ff73088c.webp)
	每一级往下都会有一个偏移量。是相对于他的母目录的地址的偏移量
# **6.   时钟树系统的了解** {toggle="true"}
	## STM32（ARM）的时钟为什么要设置的那么复杂 {toggle="true"}
		为什么ARM的芯片不能像51单片机就一样，全部都弄成一个时钟呢？
		1. 有利于省电<br>51单片机是很早之前比较广泛应用，当时对芯片的功耗高低的要求不高<br>而STM32作为新型的芯片，他的内核是Cortex - M3的内核。他对于功耗的要求就比较高。这里的高不是功耗要变高，而是对功耗控制的精度变高。不同的外设用不同的频率。因为频率越高。功耗也就越高
		2. 不同外设的频率不相同<br>使用不同外设，如果用相同的频率。那么外设的抗干扰能力就会变弱。功耗也会变大。
	## 分析时钟树的方法（这里是对F4系列芯片） {toggle="true"}
		### 时钟分析方法 {toggle="true"}
			要分析之前，首先要知道**梯形符号**是什么意思.<br>梯形符号叫做**“选择器”  **他可以选择多个时钟的其中一条来输出
			1. LSI 低速内部时钟<br>L - low低速  S  - speed 速度   I - interior内部
				频率为32KHZ，是内部的LC振荡器，不太稳定。一般是给独立看门狗做时钟的。（看门狗也需要使能看门狗的使能位。）
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/7aeb90f8-d6bc-4733-b001-225b8a6e9c2b.webp)
			2. LSE 低速外部时钟
				E  -  external 外部<br>这里是我们外接的（低速）晶振时钟。它的稳定性就很高了。
			3. `HSI `高速内部时钟  16Mhz  。 内部的，不稳定
			4. `PLLCLK `锁相环时钟输出<br>有俩，第一个是主用的，第二个是专用的。 其中的xN是倍频器。  /P是分频器。 /Q是USB模块用的。/R主用的没用到。
				为什么要有一个专用的呢？因为对于I2S的时钟对频率要求非常高，所以是专用的。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9803ae72-abd9-4c6a-bae4-725596a95da0.webp)
				可以看到，主PLL锁相环输出的系统时钟。可以经过选择器。<br>AHB与分频器给很多的外设<br>AHB的信号，又能经过APBX的与分频器，又能产生很多时钟。<br>APBX就是给相关外设用的。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/d147a047-0439-4019-a58f-f387229e10eb.webp)
			5. `HSE `高速外部时钟(可以接4-26M的外部时钟)
			6. `MCO1 `和 `MCO2`  这两个是可以经过分频器和选择器来输出到芯片引脚上的
		### 总结： {toggle="true"}
			1. STM32 有5个时钟源:HSI、HSE、LSI、LSE、PLL.
				1. HSI是高速内部时钟，RC振荡器，频率为16MHZ，精度不高。可以直接作为系统时钟或者用作PLL时钟输入。
				2. HSE是高速外部时钟，可接石英/陶瓷谐振器，或者接外部时钟源，频率范围为4MHz\~26MHZ。
				3. LSI是低速内部时钟，RC振荡器，频率为32KH2，提供低功耗时钟。主要供独立看门狗和自动唤醒单元使用。
				4. LSE是低速外部时钟，接频率为32.768kHz的石英晶体。RTCPLL为锁相环倍频输出。STM32F4有两个PLL
					1. 主PLL(PLL)由HSE或者HSI提供时钟信号，并具有两个不同的输出时钟
						1. 第一个输出PLLP用于生成高速的系统时钟(最高168MHz)
						2. 第二个输出PLLQ用于生成USBOTGFS的时钟(48MHZ)，随机数发生器的时钟和SDIO时钟。
					2. 专用PLL(PLLI2S)用于生成精确时钟，从而在12S接口实现高品质音频性能
			2. .系统时钟SYSCLK可来源于三个时钟源:
				1. HSI振荡器时钟
				2. HSE振荡器时钟
				3. PLL时钟
		### 常用时钟配置寄存器 {toggle="true"}
			- RCC时钟控制寄存器` RCC_CR`
				主要用来配置、使能时钟源。就绪时钟源的
			- RCC PLL配置寄存器  `RCC_PLLCFGR`
				是用来配置PLL锁相环输出中的P、Q、R等值。
			- RCC 时钟配置寄存器 `RCC_CFGR` 
				用来配置分频系数以及时钟源。（梯形以及方块里的/M的配置）
			- RCC AHB**X**外设时钟使能寄存器 `RCC_ AHBXENR`
				使能一些外设的时钟
			- RCC  APB1、APB2 外设时钟使能寄存器 RCC_APBXENR
				使能一些外设的时钟
# **7.   SystemInit.c时钟系统初始化文件了解** {toggle="true"}
	在系统初始化之后，是先调用Systemlnit函数，然后才调用main函数的。（这点在.s的启动文件哪里，可大概看到汇编指令是先执行sys再执行main的）
	这个文件会打开时钟以及复位一些东西，具体可以去手册的RCC和SystemInit.c文件中对应的去看。把16进制翻译为2进制后，找到寄存器的对应位就OK。这样就知道这些是什么作用了。
	这个建议新手不要改
# **8.   Systick定时器** {toggle="true"}
	## Systick定时器基础知识了解 {toggle="true"}
		- Systick定时器，是一个简单的定时器，对于CM3.CM4内核芯片，都有Systick定时器
		- **Systick定时器常用来做延时，或者实时系统的心跳时钟**。**这样可以节省MCU资源，不用浪费一个定时器**。比如UCOS中，分时复用，需要一个最小的时间戳，一般在STM32+UCOS系统中，都采用Systick做UCOS心跳时钟:
		- Svstick定时器就是系统滴答定器一个**24 位的倒计数定时器**，<br>计到0时，**将从RELOAD寄存器中自动重装载定时初值**。
		- 只要不把它在SvsTick控制及状态寄存器中的使能位清除，就永不停息，**即使在睡眠模式下也能工作:**
		- SysTick定时器被捆绑在NVIC中，用于产生SYSTICK异常(异常号:15)<br>**意思就是它能够产生中断。**<br>Systick中断的优先级也可以设置
		<empty-block/>
	## Systick定时器的四个寄存器 {toggle="true"}
		- SysTick控制和状态寄存器    -   `CRTR  `
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/fe33aa3d-882a-4f06-b459-f9621f5d3c95.webp)
			可以配置使能、是否产生中断（异常请求）、配置内外部时钟源<br>以及在一次循环之后。可以读取的位段16的值。（读取后复位）
			配置的函数：`SysTick_CLKSourceConfig();`
		- SysTick 自动重装载初值寄存器    -  ` LOAD`<br>放重装时放的值
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f42087a6-1662-452f-ac2b-526e6598747a.webp)
		- SysTick  当前值寄存器     - ` VAL `<br>把重装值复制过来，然后每个时钟周期-1<br>减到零就再次重装
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ed892175-1091-4386-bcba-1596b4ba8bca.webp)
		- SysTick   校准值寄存器 
			不太重要。
	## Systick相关函数 {toggle="true"}
		固件库中在 misc.c文件中
		`SysTick_CLKSourceConfig();`   时钟源选择  
		`SysTick_Config(uint32_t ticks);` 初始化Systick，时钟为HCLK。并开启中断
		SYstick中断服务函数
		`void SysTick_Handler(void);` 
	## 尝试使用Systick写一个延时函数 {toggle="true"}
		1. 把定义好Systick的重装值。
			这里使用到了中断与Systick的` LOAD` 寄存器
		2. `volatile` 告知编译器，每次读取 `TimingDelay` 的值时都要从内存中获取，而不能依赖于可能过时的缓存值，从而保证了在多线程或硬件中断等环境下，对该变量的操作能够反映其最新的状态。<br>        如用volatile:<br>         volatile u8int_8 test;<br>         test = 1;<br>         test = 2;<br>         test = 3;<br>       则所有语句都会被编译。test先后被设置成1、2、3
		```c
// 延时函数初始化
void Delay_Init(void)
{
    // 配置 SysTick 为 1us 中断一次
    if (SysTick_Config(SystemCoreClock / 1000000))
    {
        while (1);  // 配置错误，进入死循环
    }
}
		```
		`SysTick_Config`是SysTick的时钟源选择函数。<br>起内部的代码是
		```c
static __INLINE uint32_t SysTick_Config(uint32_t ticks)
{ 
  if (ticks > SysTick_LOAD_RELOAD_Msk)  return (1);            /* Reload value impossible */
                                                               
  SysTick->LOAD  = (ticks & SysTick_LOAD_RELOAD_Msk) - 1;      /* set reload register */
  NVIC_SetPriority (SysTick_IRQn, (1<<__NVIC_PRIO_BITS) - 1);  /* set Priority for Cortex-M0 System Interrupts */
  SysTick->VAL   = 0;                                          /* Load the SysTick Counter Value */
  SysTick->CTRL  = SysTick_CTRL_CLKSOURCE_Msk | 
                   SysTick_CTRL_TICKINT_Msk   | 
                   SysTick_CTRL_ENABLE_Msk;                    /* Enable SysTick IRQ and SysTick Timer */
  return (0);                                                  /* Function successful */
}
		```
		在判断是否超过最大值后，这个函数会把你传入的值放到SYStick的LOAD寄存器（重装寄存器）中。<br>然后设置优先级为最高<br>并且将VAL寄存器（当前值寄存器）置零。<br>噔噔蹬蹬….
		`SystemCoreClock` 是 STM32 标准库中定义的一个全局变量，用于表示系统的核心时钟频率。使用了条件编译的命令。这里默认为F1的72M
		那么`SysTick_Config(SystemCoreClock / 1000000)` 的意思就是设置系统滴答定时器的重装值为为7200（72 000 000000/1000000）。
		因为晶振的速度为72M，也就是一秒钟有72M个高电平。
		所以晶振震荡7200次花费的时间就是1us。
		所以可以这样去判断
		```c
// 延时函数
void Delay_us(uint32_t us)
{
    TimingDelay = us;
    while (TimingDelay!= 0);
}

// SysTick 中断处理函数
void SysTick_Handler(void)
{
    if (TimingDelay!= 0)
    {
        TimingDelay--;
    }
}
		```
		其中的中断函数`void SysTick_Handler(void)`，是库函数固定的，必须要用这个名字。他是SysTick的固定中断服务函数。
		`void Delay_us(uint32_t us)` 是定义的us函数
		举例子说，想`Delay_us` 函数中传参：50.
		那么全局变量`TimingDelay` == 50
		然后就会进入到`while (TimingDelay!= 0);` 命令中一直等待。此时就在等待中断处理函数把全局变量`TimingDelay` 减为0.然后跳出函数。此时程序就能继续运行了。 
		在中断处理函数中，每当重装值7200 减 到 0 之后。就会触发一次中断，（也就是执行一次` SysTick_Handler()`函数。）全局变量`TimingDelay`就会减一。当他捡到0之后就不会再减了。
		要实现1ms延时。也很简单。只需要把传入的值x1000就OK啦
		```c
// 毫秒级延时函数
void Delay_ms(uint32_t ms)
{
    Delay_us(ms * 1000);
}
		```
		这次用这个Delay函数来实现一下闪烁灯试试.完全Ok。
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a7bfd8ed-5824-41f4-a22b-c97a3c26d9d8.gif)
		(如果在编译的时候报错，根据报错信息。大概率要去stm32f103x_it.h找到`void SysTick_Handler(void)`函数然后注释掉。不然会出现多次定义。头文件。)
		```c
int main()
{

    KEY_Init();//初始化KEY
    LED_Init();//初始化LED
    Delay_Init();//初始化延时函数
    
    while(1)
    {
        LED_ON(1);
        Delay_ms(500);
        LED_OFF(1);
        Delay_ms(500);
    }
}
		```
# **9.   IO引脚的复用和映射** {toggle="true"}
	看这个之前可以先去把NVIC中断优先级了解一下。在第10大标题
	## **什么是端口复用?** {toggle="true"}
		STM32有很多的内置外设，这些外设的外部引脚都是与GPIO复用的。<br>也就是说，**一个GPIO如果可以复用为内置外设的功能引脚，那么当这个GPIO作为内置外设使用的时候，就叫做复用。**
		> 举个栗子：<br>比如串口1的俩引脚是PA9和PA10。那么如果PA9和PA10端口不用做串口（USART），二用作复用功能的串口1的发送接收引脚的时候，就叫做端口复用
		查端口的表在手册可以看到。
		每个引脚都会有一个复用器（其实是时钟树那种的梯形选择器），选择器可以选择连接对应的复用功能引脚。
		端口复用映射是通过AFRL或者AFRH来配置的
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/77d4d043-1447-4065-ab23-da44bbfd36d5.webp)
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8461578c-daf6-4904-9ba7-b053801fa1f0.webp)
		0-7八个寄存器是寄存器AFRL控制的。每四位控制一个GPIO口。 同理AFRH也是一样
		首先参考复用端口示意图，然后去配置对应的端口寄存器。
	## 芯片的复用功能映射配置 {toggle="true"}
		引脚有不同的作用，
		1. 系统功能：<br>将 I/O 连接到 AF0，然后根据所用功能进行配置（比如）：
			- JTAG/SWD:在各器件复位后，会将这些引脚指定为专用引脚，可供片上调试模块立即使用(不受 GPIO 控制器控制)。
			- RTC REFIN:此引脚应配置为输入浮空模式。
			- MCO1 和 MCO2:这些引脚必须配置为复用功能模式
		2. GPIO<br>在 GPIOX_MODER 寄存器中将所需 I/0 配置为输出或输入。
		3. 外设复用功能
			对于ADC和DAC，需要再GPIOx_MODER寄存器把IO配置为模拟通道，
			对于其他外设
			- 在 GPIOx_MODER 寄存器中将**所需 I/0 配置为复用块能**
			- 通过 GPIOX OTYPER、GPIOX PUPDR 和 GPIOx OSPEEDER 寄存器，分别选择类型、上拉/下拉以及输出速度
			- 在 GPIOX AFRL 或 GPIOX AFRH 寄存器中，将 I/0 连接到所需 AFx
	## \[江协\]什么是通信 {toggle="true"}
		通信的目的：将一个设备的数据传到另一个设备，扩展硬件系统
		通信协议：定制通信的规则，通信双方按照协议规则进行数据收发 
	## \[江协\]常用通信方式以及不同 {toggle="true"}
		<table header-row="true">
		<colgroup>
		<col width="66">
		<col width="199">
		<col width="84">
		<col width="92">
		<col width="66">
		<col>
		</colgroup>
<tr>
<td>名称</td>
<td>引脚</td>
<td>双工</td>
<td>时钟</td>
<td>电平</td>
<td>设备</td>
</tr>
<tr>
<td>USART</td>
<td>TX、RX</td>
<td>全双工</td>
<td>异步</td>
<td>单端</td>
<td>点对点</td>
</tr>
<tr>
<td>I2C</td>
<td>SCL、SDA</td>
<td>半双工</td>
<td>同步</td>
<td>单端</td>
<td>多设备</td>
</tr>
<tr>
<td>SPI</td>
<td>SCLK、MOSI、MISO、CS</td>
<td>全双工</td>
<td>同步</td>
<td>单端</td>
<td>多设备</td>
</tr>
<tr>
<td>CAN</td>
<td>CAN_N、CAN_L</td>
<td>半双工</td>
<td>异步</td>
<td>差分</td>
<td>多设备</td>
</tr>
<tr>
<td>USB</td>
<td>DP、DM</td>
<td>半双工</td>
<td>异步</td>
<td>差分</td>
<td>点对点</td>
</tr>
		</table>
	## \[江协\]什么是串口通信USART {toggle="true"}
		### 串口是什么 {toggle="true"}
			- 串口是一种应用十分广泛的通讯接口，串口成本低、容易使用、通信线路简单，可实现两个设备的互相通信
			- 单片机的串口可以使单片机与单片机、单片机与电脑、单片机与名式各样的模块互相通信，极大地扩展了单片机的应用范围，增强了单片机系统的硬件实力
			- 串口一般使用异步通信，所以一般要双方约定一个通信速率
		### 串口的连接注意事项及分类 {toggle="true"}
			- 简单双向串口通信有两根通信线(发送端TX和接收端RX)
			- TX与RX要交叉连接
			- 当只需单向的数据传输时，可以只接一根通信线
			- 当电平标准不一致时，需要加电平转换芯片
				- 电平标准是数据1和数据0的表达方式，是传输线缆中人为规定的电压与数据的对应关系,1串口常用的电平标准有如下三种:
					- TTL电平:+3.3V或+5V表示1，0V表示0
					- RS232电平:-3\~-15V表示1，+3\~+15V表示0（一般在大型机器上使用）
					- RS485电平**:两线压差**+2\~+6V表示1，-2\~-6V表示0(差分信号) （可以传输上千米。上面俩就几十米）
			<empty-block/>
		### 串口的参数 {toggle="true"}
			1. 波特率:   串口通信的速率<br>串口一般使用异步通信，所以一般要双方约定一个通信速率
			2. 起始位:   标志一个数据帧的开始，固定为低电平<br>高速接收设备我要发送数据了。
			3. 数据位:  数据帧的有效载荷，1为高电平，0为低电平，**低位先行**
			4. 校验位:  用于数据验证，根据数据位计算得来<br>无校验、奇校验和偶校验<br>奇校验就是通过校验位保证数据位和校验位为**奇数个1.**<br>偶校验就是通过校验位保证数据位和校验位为**偶数个1.<br>（更复杂的话可以去了解CRC校验）**
			5. 停止位:   用于数据帧间隔，固定为高电平
		### USART简介及框图介绍 {toggle="true"}
			USART通用同步/异步收发器<br>Universal Synchronous/Asynchronous Receiver/Transmitter
			UART是异步收发器。一般很少用。S只是多了一个时钟输出而已。
			它只支持时钟输出，不支持时钟输入。（或许是为了别的通信协议设计）
			**我们学习的主要是异步通信。**
			<empty-block/>
			USART是STM32内部集成的硬件外设。
			STM32F103C8T6的USART资源有 USART1、USART2、USART3
			其中USART1为APB1总线上、USART2、USART3在APB2总线上挂载
			USART框图
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/2eae46e4-6a07-4f6c-ba98-60ad9cb1a699.webp)
			左边的IRDA之类的是别的通信协议要用的，这里不做了解
			只了解一下串口通信。 
			TX数据发送，是由发送移位寄存器发送的
			同理，RX数据接收，是通过接受移位寄存器接受的。
			最上边的发送（TDR）\\接收（RDR）数据寄存器。发送或者接受的字节数据都存在这里。<br>**这是俩寄存器**，共用同一个地址，在程序上就用一个数据寄存器DR表示。
			TDR是只写的，RDR是只读的。
			他们下边的发送移位寄存器，和接收移位寄存器，是把一个字节的数据，一位一位的移出去。
			如果此时进行了写操作，写入的字节会存放在发送寄存器TDR中，在等待发送移位寄存器的数据移位完毕后，会立刻把这个数据赋值（送）到移位寄存器中。**然后置一个标志位 1 ，**我们只需要检查这个发送数据寄存器TDR置的**TEX**标志位，**TEX**是否为1.只要为1 就可以在TDR写入下一个数据了。<br>移位寄存器会收到发送端控制的控制，向右一位一位的把数据输出到TX引脚。向右移位。也就是**低位先行** 
			接收端也是类似，也是向右移位。当接收移位寄存器满了之后，就会一下子把数据转移到接受数据寄存器RDR。转移的过程中，也会 置一个**RXNE**标志位，。当**RXNE**为1时，就可以把数据读走了 。
			剔除帧头帧尾是电路帮我们自动删除了。
			唤醒单元是用多设备串口通信用的。<br>硬件流控也是交叉连接，用来控制双方数据停止继续的，
			右边的SCLK是会每发送一个字节就输出一下高电平。可以在别的硬件上做自适应的波特率检测。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/78a6b3bb-51d5-4bd3-817f-fd3a5aeb2152.webp)
			再往下就是波特率发生装置，其实就是个分频器。他们的频率不同。
			简化后其实就是这样子
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/526f3154-2506-42a6-af8d-a056530d8ef1.webp)
		### STM32在接受端的天才设计 {toggle="true"}
			STM32在接收时，为了应对噪声。有一个很好的处理方法。对高电平时一次采样，但如果突然检测到下降沿（也就是起始位）
			就会立马开始进行16次采样。3次一组。两组一次。这两组如果全都是0，就代表起始位确实开始了。不是噪声。就可以开始采样了。<br>但是如果发现某组有一个1，俩0.就代表他有些许的噪声，但是不会做处理，这时如果有俩组通过，也算他检测到起始位了。不过会有一个噪声标志位，NE。会提醒你一下，数据我收到了。但是你悠着点用。
			再其次就是有某组为俩0，那么这组就废了。继续开始下一组，直到连着两组通过。就算成功！
			如果通过，他就会在第7  8  9 次采样的时候为有效采样。也是遵循2:1的原则。 7 8 9次检测也是刚刚好在最中间，非常好！
	## \[正点（但是F103板）\]端口的复用功能配置过程（串口举例） {toggle="true"}
		**以PA9、PA10配置为串口为例**
		1. 使能端口GPIOA的时钟
			`RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);`
		2. 使能复用外设时钟
			`RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);`
		3. 把端口模式配置为复用功能（除了ADC和DAC都得这么配置）<br>F1这里与F4的不太一样<br>F4的是GPIO_PinAFConfig（GPIOA,GPIO_PinSource9）。并且每个端口都需要配置复用。<br>**但是在F103中，**只需要把输入引脚RX配置成上拉输入或者浮空输入。<br>把输出引脚TX配置成推挽输出就可以了。
		4. 配置GPIOX_AFRL寄存器或者GPIO_AFRH寄存器，把IO连接到所需要的外设<br>这里也与F4不同，这里只需要直接配置USART就可以了。然后使能USART的串口
			参考代码如下：
			```c
//初始化
void Serial_Init(void)
{
    //使能A9 A10所在的GPIO
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    
    //使能A9 A10的复用外设时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);
    
    //配置PA9端口为复用推挽输出
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;//复用推挽模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;       //9
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);//初始化
    //这里对于F4芯片可以单独 分开的设置复用以及上下拉。不用分开配置，
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;//复用推挽模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;       //9
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);//初始化
    
    USART_InitTypeDef USART_InitStructure;
    USART_InitStructure.USART_BaudRate = 9600;//波特率。会自动算好填入BRR寄存器
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;//硬件流控制、不使用.
    USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;//用| 使用两个功能
    USART_InitStructure.USART_Parity = USART_Parity_No; //校验位。
    USART_InitStructure.USART_StopBits = USART_StopBits_1;//停止位1位。
    USART_InitStructure.USART_WordLength = USART_WordLength_8b; //不需要校验，所以字长选择8位字长。
    USART_Init(USART1,&USART_InitStructure);
    
    USART_Cmd(USART1,ENABLE);
}
			```
# **10.   NVIC中断优先级管理** {toggle="true"}
	## NVIC中断优先级分组 {toggle="true"}
		不同的系列芯片，他的中断个数和中断可编程优先级的是不一样的。<br>中断分为内核 中断和可屏蔽中断。
		在手册中可以在“中断和事件”中查找到中断的列表
		> 分组配置是在寄存器SCB - \>AIRCR中配置的。
		这个寄存器是位于8  9  10位的。可以有以下分类
		<table header-row="true" header-column="true">
		<colgroup>
		<col width="49">
		<col>
		<col>
		<col>
		</colgroup>
<tr>
<td>组</td>
<td>AIRCR\[10:8\]</td>
<td>IP  Bit\[7:4\]分配情况</td>
<td>分配结果</td>
</tr>
<tr>
<td>0</td>
<td>111</td>
<td>0：4</td>
<td>0位抢占优先级，4位相应优先级</td>
</tr>
<tr>
<td>1</td>
<td>110</td>
<td>1：3</td>
<td>1位抢占优先级，3位相应优先级</td>
</tr>
<tr>
<td>2</td>
<td>101</td>
<td>2：2</td>
<td>2位抢占优先级，2位相应优先级</td>
</tr>
<tr>
<td>3</td>
<td>100</td>
<td>3：1</td>
<td>3位抢占优先级，1位相应优先级</td>
</tr>
<tr>
<td>4</td>
<td>001</td>
<td>4：0</td>
<td>4位抢占优先级，0位相应优先级</td>
</tr>
		</table>
		每个中断有一个寄存器， ip bit  他的定位是4  5  6  7四个位
		他决定了有几个位是用来设置抢占优先级、有几个是用来响应优先级。
		- 抢占优先级：数字越低级别越高。**高级别的**抢占优先级的中断可以**打断**正在进行中断的**低级别的**抢占优先级的中断
		- 相应优先级：数字越低级别越高，！**在抢占优先级相同的情况才会生效。**此时若两个中断同时发生，那么高响应优先级的会优先执行。但若已经有相同抢占优先级的中断在运行了。是不会去打搅的。（也就是说在相同抢占优先级，并且同时请求中断的时候。响应优先级才会生效。）
		### 抢占优先级和响应优先级 的区别 {toggle="true"}
			- 高优先级的抢占优先级是可以打断正在进行的低抢占优先级中断的。
			- 抢占优先级粕同的中断，高响应优先级不可以打断低响应优先级的中断。
			- 当两个中断同时发生的情况抢占优先级相同的中断，下，哪个响应优先级高，哪个先执行。
			- 如果两个中断的抢占优先级和响应优先级都是一样的话，则看哪个中断先发生就先执行;
			一般系统只会设置一次分组。 不然会发生混乱
			<empty-block/>
	## NVIC中断优先级设置 {toggle="true"}
		> 中断优先级分组函数：`void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup)`<br>在misc.c的文件中<br>他实际操作的就是SCB-\>AIRCR 寄存器
		可以在misc.h的头文件看到这个
		```c
#define NVIC_PriorityGroup_0         ((uint32_t)0x700) /*!< 0 bits for pre-emption priority
                                                            4 bits for subpriority */
#define NVIC_PriorityGroup_1         ((uint32_t)0x600) /*!< 1 bits for pre-emption priority
                                                            3 bits for subpriority */
#define NVIC_PriorityGroup_2         ((uint32_t)0x500) /*!< 2 bits for pre-emption priority
                                                            2 bits for subpriority */
#define NVIC_PriorityGroup_3         ((uint32_t)0x400) /*!< 3 bits for pre-emption priority
                                                            1 bits for subpriority */
#define NVIC_PriorityGroup_4         ((uint32_t)0x300) /*!< 4 bits for pre-emption priority
                                                            0 bits for subpriority */
		```
		中断参数初始化函数`void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct)`
		```c
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//设置中断优先级
    NVIC_InitTypeDef NCIC_InitStructure;
    NCIC_InitStructure.NVIC_IRQChannel = USART1_IRQn;//设置中断通道
    NCIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NCIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;//设置抢占优先级
    NCIC_InitStructure.NVIC_IRQChannelSubPriority = 1;//设置响应优先级
    NVIC_Init(&NCIC_InitStructure);//使能中断通道
		```
		如果后续需要挂起、解挂，查看中断当前状态，分别调用相应的函数就OK
# 11.**   **STM32单片机中断详解 {toggle="true"}
	## 1  中断相关概念 {toggle="true"}
		### 1.1  什么是中断？ {toggle="true"}
			- 中断是单片机正在执行程序时，由于内部或外部事件的触发，打断当前程序，转而去处理这一事件，当处理完成后再回到原来被打断的地方继续执行原程序的过程。
			- 在ARM体系结构中，中断通常由外设或外部输入产生，有时也可以由软件触发。中断是单片机系统处理紧急或突发事件的重要方式，如定时器溢出、按键输入、串口数据到达等。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5f8351d9-f85d-48c8-93eb-3001101e8f09.webp)
			-
		### 1.2  中断的意义 {toggle="true"}
			- 中断的主要意义在于**提高CPU的效率**，而不会一直占用CPU，实现对突发事件的实时处理，以及实现程序的并行化和嵌入式系统进程之间的切换。
			- 相较于轮询方式（即按照一定的频率和周期不断地检测某些事件的发生），**中断在处理一些偶然发生的事情时效率更高。**
		### 1.3  中断优先级 {toggle="true"}
			STM32的Cortex-M3内核中有16个内部中断、240个外部中断和可编程的256级优先级设置。
			目前支持84（16内部+68外部）个，和16级可编程中断优先级设置
			优先级冲突处理
			- STM32中中断通道可配置为响应优先级和抢占优先级
			- 具有高抢占式优先级的中断 可以在具有低抢占式优先级的中断处理过程中被响应。<br>（即中断的嵌套）（或者说高抢占式优先级的中断可以嵌套低抢占式优先级的中断）
			- 当两个中断源的抢占式优先级相同时，这两个中断将没有嵌套关系。当一个中断到来后，如果**正在处理另一个中断**，这个后到来的中断就要等到前一个中断处理完之后才能被处理。
			- 如果上述两个中断**同时到达,**并且抢占优先级相同<br>则中断控制器会根据它们的响应优先级的高低来决定先处理哪一个;<br>如果它们的抢占式优先级和响应式优先级都相等，则根据它们在中断表中的排位顺序决定先处理哪一个。
			数字越小优先级越高。
		### 1.4  中断嵌套 {toggle="true"}
			如果一个高优先级的中断发生，它会立即打断当前正在处理的中断（如果其优先级较低），并首先处理这个 高优先级的中断，这就是所谓的中断嵌套。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8ae0d5fe-3e23-4520-8471-cc0629348f75.webp)
		### 1.5  中断执行流程 {toggle="true"}
			当中断发生时，STM32的执行流程如下：首先，由外设发出**中断请求**；然后，处理器暂停当前执行的任务，**保护现场**（如将当前位置的PC地址压栈）；接着，程序跳转到对应的中断服务程序（ISR）并执行；**中断服务程序执行**完毕后，**恢复现场**（如将栈顶的值送回PC）；最后，处理器**返回到被中断的位置**，继续执行下一个指令。
	## 2  STM32中断 {toggle="true"}
		### 2.1  中断数量（以SMT32F103C8T6为例） {toggle="true"}
			SMT32F103C8T6支持的中断共有70个，其中包括10个内核中断和60个外部中断。其中外部中断包含EXTI、TIM、ADC、I2C、SPI等等。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/bf91a2f9-b195-4c6a-9d25-99625c706da7.webp)
		### 2.2  中断向量表 {toggle="true"}
			- STM32的中断向量表是一个存储中断处理函数地址的数组，位于Flash区的起始位置。每个数组元素对应一个中断源，其地址指向相应的中断服务程序。<br>当中断发生时，处理器会根据中断号查找向量表，然后跳转到对应的中断服务程序执行。
			- 中断向量表的主要作用是解决中断函数地址不固定与中断必须跳转到固定地方执行程序之间的矛盾。由于编译器每次编译都会为中断函数随机分配地址，但硬件要求中断必须跳转到固定的位置，因此，**中断向量表就作为这样一个固定的地址列表，其中包含了中断函数的地址以及跳转到这些地址的程序。**当中断发生时，处理器会跳转到这个固定的中断向量表，然后根据其中的信息跳转到相应的中断处理函数从而执行中断。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f5db54cf-f5dc-4da5-8d34-e2b134c314b1.webp)
		### 2.3  SMT32中断框图 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e98f16ac-df6f-4ff4-8ccb-9f50f786faf8.webp)
	## 3  NVIC（嵌套向量中断控制器） {toggle="true"}
		### 3.1  NVIC（嵌套向量中断控制器） {toggle="true"}
			- NVIC，即Nested Vectored Interrupt Controller（嵌套向量中断控制器），是STM32中的中断控制器。它负责管理和协调处理器的中断请求，是STM32中处理异步事件的重要机制。
			- NVIC提供了灵活、高效、可扩展的中断处理机制，支持多级优先级、多向中断、嵌套向量中断等特性。
			- 当一个中断请求到达时，NVIC会确定其优先级并决定是否应该中断当前执行的程序，以便及时响应和处理该中断请求。这种设计有助于提高系统的响应速度和可靠性，特别是在需要处理大量中断请求的实时应用程序中。
			- NVIC 支持：256个中断（16内核+240外部），支持：256个优先级，允许裁剪。
		### 3.2  NVIC工作原理 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/52932fe6-30b7-4a69-9f3c-38634fea0473.webp)
		### 3.3  中断优先级基本概念 {toggle="true"}
			- NVIC可以管理多个中断请求，并按优先级处理它们。
			- 在STM32中，中断优先级被划分为抢占式优先级和响应 优先级，可以根据具体的应用需求进行配置。不同的优先级分组方式会影响中断的响应和处理顺序。
			- 抢占优先级<br>如果一个中断的抢占优先级高于当前正在执行的中断，那么它可以打断当前中断，优先得到执行。数值越小，优先级越高。
			- 应优先级<br>如果两个中断同时到达，且它们的抢占优先级相同，那么响应优先级高的中断将首先得到响应。数值越小，优先级越高。
			- 自然优先级<br>自然优先级是由硬件固定并预先设定的，用户无法更改。当抢占优先级和响应优先级都相同时，自然优先级将决定哪个中断先得到处理。
			- 优先级执行顺序<br>当多个中断同时发生时，执行顺序首先由抢占优先级决定。如果抢占优先级相同，则进一步由响应优先级决。如果响应优先级也相同，则最终由自然优先级决定。
			- 在中断嵌套的情况下，<br>**高抢占**优先级的中断**可以**打断**低抢占**优先级的中断，<br>但**高响应**优先级的中断**不能**打断**低响应**优先级的中断（当它们具有相同的抢占优先时）。
			- 优先级分组<br>优先级寄存器 IPR 有 8 位，但实际只使用到高 4 位，用于决定抢占优先级、响应优先级的等级。<br>具体这 4 位如何切割？由又由 AIRCR 寄存器控制。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3ddbfd11-1aa1-47d7-b92a-3d825afc0303.webp)
		### 3.4  NVIC寄存器 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4921e265-d816-41e6-897f-265e88405cbf.webp)
		### 3.5  NVIC相关函数介绍 {toggle="true"}
			- 在contex.c中可以查看到以下函数：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/217fb950-0eeb-4de2-a5db-5e909ceb536a.webp)
			- 其中，只有DisableIRQ(中断失能函数）、EnableIRQ（中断使能函数）、GetPriority（中断优先级函数）、GetPriorityGrouping（中断优先级分组函数）经常使用。
	## 4  EXTI（外部中断） {toggle="true"}
		### 4.1  EXTI简介 {toggle="true"}
			- EXTI 是 External Interrupt 的缩写，表示外部中断事件控制器。
			- EXTI 可以监测指定 GPIO 口的电平信号变化，并在检测到指定条件时，向内核的中断控制器 NVIC 发出中断申请。
			- NVIC 在裁决后，如果满足条件，会中断CPU的主程序，使 CPU 转而执行EXTI 对应的中断服务程序。
			- EXTI 支持的触发方式：上升沿、下降沿、双边沿或软件触发。
			- EXTI 支持所有的 GPIO 口，但需要注意的是，**相同的 Pin 不能同时触发中断。**例如，PA0 和 PB0 不能同时被配置为中断源。
			- EXTI 提供了 16 个 GPIO_Pin 的中断线，以及额外的中断线如 PVD 输出、RTC 闹钟、USB 唤醒和以太网唤醒。
			- 通过适当的配置，EXTI可以实现丰富多样的功能，如响应按键的按下、传感器的状态变化等外部事件。
		### 4.2  中断/事件 {toggle="true"}
			- 中断会**打断**CPU当前正在执行的程序，转而去执行中断服务程序，待中断服务程序执行**完毕后，**CPU会**返回到原来的程序执行点继续执行。**
			- 而事件只是简单地表示某个动作或状态的变化，而不会打断CPU当前正在执行的程序。当事件发生时，它会**根据配置来决定是否触发相应的中断**。如果开放了对应的中断屏蔽位，事件就可以触发相应的中断，否则事件只会作为一个信号存在，不会被CPU处理。
		### 4.3  EXTI基本结构 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e44d585f-a423-42ea-a5db-07a20398c11b.webp)
		### 4.4 EXTI基本框图 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8caaf516-e349-4297-ab67-59f7094b5d2b.webp)
		### 4.5  EXTI寄存器（结合EXTI基本框图进行理解） {toggle="true"}
			- 中断屏蔽寄存器（EXTI_IMR)结合EXTI基本框图进行理解
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/2c4aa56f-f481-4805-a29e-2085c8bf39ba.webp)
			- 事件屏蔽寄存器（EXTI_EMR)
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/6873e7c5-688a-4db8-9773-18e9b7ffe229.webp)
			- 上升沿触发选择寄存器（EXTI_RTSR)
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c0d3a907-403d-4a10-b753-7739cbb5e612.webp)
			- 下降沿触发选择寄存器（EXTI_FTSR)
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/22d896ff-f5ff-4ab3-8b04-647ef3d34bbf.webp)
			- 软件中断事件寄存器（EXTI_SWIER）
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3e06b588-fc0a-454a-8277-6ba050656ec6.webp)
			- 挂起寄存器（EXTI_PR）
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e0f6655e-bd26-401a-a10a-18dd588049ef.webp)
		### 4.6  EXTI相关函数 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c0cc24c3-822a-41d2-a896-072d2f613e9c.webp)
			这两个函数在gpio.c文件中，Callback函数是回调函数，IRQHandler函数是中断处理函数。
	## 5  AFIO（复用功能IO） {toggle="true"}
		### 5.1  AFIO简介 {toggle="true"}
			- AFIO 是 Alternate Function Input/Output 的缩写，表示复用功能 IO，主要用于实现 I/O 端口的复用功能以及外部中断的控制。
			- STM32上有很多 I/ O口以及内置外设（如I2C、ADC、ISP、USART等）。<br>为了节省引出管脚的数量，这些内置外设通常与 I/O 口共用管脚，即 I/O 管脚具有复用功能。<br>例如，一个 GPIO 管脚除了可以作为普通的 I/O端口外，还可以被复用为某个内置外设的功能引脚。
			- 然而，为了优化64脚或100脚封装的外设数量，有时需要将一些复用功能重新映射到其他引脚上。这时，就可以使用AFIO的复用**重映射**功能。通过设置复用重映射和调试I/O配置寄存器（AFIO_MAPR），可以实现引脚的重新映射，使得复用功能不再映射到它们的原始分配上。
			- 此外，AFIO 还用于控制外部中断，用来配置 EXTI 中断线 0\~15 对应哪个具体 IO 口。
			- 当需要使能外部中断线或进行外部中断线的映射时，通常需要开启AFIO的时钟。
		### 5.2  AFIO与IO对应关系 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/cca8d6ed-ec76-4f55-9d2d-c50545dea93a.webp)
		### 5.3  AFIO寄存器 {toggle="true"}
			外部中断配置寄存器（AFIO_EXTICR)
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/75e7c303-482a-4dd6-8513-1f825717e3bb.webp)
		### 5.4  EXTI配置流程 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/35a3ba0c-d340-4a6a-bf14-d23ffdc136de.webp)
	<empty-block/>
# 12.**   **STM32单片机串口 {toggle="true"}
	## 通信接口概述 {toggle="true"}
		通信的目的：将一个设备的数据传送到另一个设备，扩展硬件系统
		通信协议：制定通信的规则，通信双方按照协议规则进行数据收发
		<table>
<tr>
<td>名称</td>
<td>引脚</td>
<td>双工</td>
<td>时钟</td>
<td>电平</td>
<td>设备</td>
</tr>
<tr>
<td>USART</td>
<td>TX, RX</td>
<td>全双工</td>
<td>异步</td>
<td>单端</td>
<td>点对点</td>
</tr>
<tr>
<td>I2C</td>
<td>SCL, SDA</td>
<td>半双工</td>
<td>同步</td>
<td>单端</td>
<td>多设备</td>
</tr>
<tr>
<td>SPI</td>
<td>SCLK, MOSI, MISO,CS</td>
<td>全双工</td>
<td>同步</td>
<td>单端</td>
<td>多设备</td>
</tr>
<tr>
<td>CAN</td>
<td>CAN_H, CAN_L</td>
<td>半双工</td>
<td>异步</td>
<td>差分</td>
<td>多设备</td>
</tr>
<tr>
<td>USB</td>
<td>DP, DM</td>
<td>半双工</td>
<td>异步</td>
<td>差分</td>
<td>点对点</td>
</tr>
		</table>
	## **通信方式分类** {toggle="true"}
		设备之间的通信方式可以分为**串行通信**和**并行通信**，
		- 串行通信是按照将数据按位顺序传输，这样做的优势是占用的引脚资源少，对于引脚资源紧张的MCU来大有益处，但是由于每次只能传输一个数据，传输速度较慢。
		- 并行通信就是数据的各个位同时传输，优点是数据传输快，缺点是占用引脚资源较多。
	## 串行通信传输方向分类 {toggle="true"}
		串行通信分为**单工、半双工和全双工。**
		- **单工 是**只能向着一个方向传输数据
		- **半双工 **是可以双向传输，但每次只能有一个传输方向
		- **全双工 **是既可以双向传输，它又可以同时有两个传输方向。
	## 同步通信和异步通信 {toggle="true"}
		- 同步通信需要时钟线（用作时钟信号的同步）的参与，例如SPI和IIC通信接口，
		- 而异步通信就是不带时钟线的，例如UART和one-wire。
		**同步是：按照任务的顺序执行任务，前一个任务没有执行结束，下一个任务不会执行，要等待上一个任务执行结束。** <br>在同步通信时，有一根时钟线。 这约定了两个设备何时开始工作， 决定了传输速率。以及工作时间（比如单片机突然进中断，就可以先暂时时让时钟失能，两边的设备都先暂停到这里，不能动）
		> 同步可以理解为打电话。 你没说完之前 我是不能说话的。
		**异步是：这两件事在同时进行**，**而不是一方等待另一方**，**因此这就是为什么一般来说异步比同步高效的本质所在。<br>**在异步通信时，两遍是没有约定的。如果我突然有事，但是你还在哇哇哇说个不停，这些东西就不被我接收到了。
		> 异步可以理解为发邮件。你发了邮件我就看。你不发我可以继续忙我的事情。
		（[单总线（one-wire](https://blog.csdn.net/weixin_46897065/article/details/136679108)）是美国 DALLAS 公司推出的外围串行扩展总线技术，与 SPI、I2C 等串行数据通信方式不同。<br>**它采用单根信号线，既传输时钟又传输数据，而且数据传输是双向的。<br>**它具有节省 I/O口线资源、结构简单、成本低廉、便于总线扩展和维护等诸多优点。）
	## 串口通信 {toggle="true"}
		STM32提供了UART（Universal Asynchronous Receiver/Transmitter，通用异步收发器）<br>USART（Universal Synchronous/Asynchronous Receiver/Transmitter，通用同步异步收发器）<br>这两种串口通信接口。于实现与其他设备之间的数据交换。
		- **UART是一种异步通信协议**，它使用<span color="blue">起始位、数据位、校验位和停止位</span>来定义**一个字符**的传输格式。
		- **USART则是一种同步/异步通信协议**，它支持全双工通信，并具备更高的数据传输速率和更好的抗干扰能力。他也可以作为UART使用。
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/057f54d3-4d6a-48a5-90c1-f8de74560c30.webp)
	## 数据帧的组成 {toggle="true"}
		在串口通信中，**每一个字节的数据**都装载在一个数据帧中。
		> 每个数据帧由以下几个部分组成：<br>**起始位：**标志数据帧的开始。<br>**数据位：**实际传输的数据，通常为8位。<br>**校验位（可选）**：用于错误检测。<br>**停止位：**标志数据帧的结束。
		1. **起始位：**固定位低电平
		2. **数据位长度**：可配置为8位或9位数据长度。
		3. **停止位长度**：可选0.5、1、1.5或2个停止位。
		4. **校验位**：可选无校验、奇校验或偶校验。
		5. **停止位：**恢复高电平持续几个周期、
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/16a96b56-7fc2-429f-858c-311478906b53.webp)
		> **起始位**<br>作用：标志一个数据帧的开始。<br>电平：固定为低电平（0）。<br>机制：串口的空闲状态是高电平（1），没有数据传输时，信号线处于高电平。发送数据时，起始位从高电平跳变到低电平，产生一个下降沿，通知接收设备数据传输的开始。
		> **数据位**<br>组成：数据帧的有效载荷。<br>长度：通常为8位，代表一个字节的8位二进制数据。<br>传输顺序：低位先行（在STM32中发送移位寄存器就是这样工作的）。<br>电平表示：1为高电平，0为低电平。
		> **校验位**<br>作用：用于数据验证，检测数据传输中的错误。<br>类型：奇校验和偶校验。<br>奇校验：确保数据帧中的1的数量是奇数。<br>偶校验：确保数据帧中的1的数量是偶数。<br>计算：根据数据位计算得来，并添加到数据位的最后。<br>例如，数据为00001111，有4个1（偶数）<br>那么偶校验之后校验位就补0，成为000011110。让整个串串的1保持偶数，否则补1<br>奇校验时，校验位为1，使1的数量变为奇数（000011111）。让整个串串的1保持奇数。
		> **停止位**<br>作用：标志一个数据帧的结束，同时为下一个数据帧的起始位做准备。<br>电平：固定为高电平（1）。<br>机制：在一个字节数据发送完成后，必须有一个停止位，用于数据帧间隔。如果没有停止位，当数据的最后一位是0时，下一个起始位的低电平无法形成下降沿，接收设备无法识别数据帧的开始。
		> **波特率**<br>定义：规定串口通信的速率，**即每秒传输的码元数**，**单位是波特（Baud）。**<br>**比特率：每秒传输的比特数，单位是比特每秒（bps）。**<br>**在二进制调制情况下，1个码元等于1个比特**，**所以波特率等于比特率。**<br>设定：发送和接收设备必须约定相同的波特率，以确保数据正确接收。<br>例如，波特率为1000bps，表示1秒传输1000位，每位的时间间隔为1毫秒（ms）。发送设备每隔1ms发送一位，接收设备也每隔1ms接收一位。
	## 采样策略等 {toggle="true"}
		> 起始位侦测<br>检测起始位：输入电路检测到一个数据帧的起始位后，会以波特率的频率连续采样一帧数据。**采样位置必须从起始位开始对齐到位的正中间**。**只要第一位对齐，后面的位也会对齐。**<br>16倍采样率：为了实现精确采样，**输入电路以波特率的16倍频率进行采样，**即在一位的时间内可以进行16次采样。
		> 采样策略<br>**初始采样：**在空闲状态下，采样结果一直为高电平（1）。<br>一旦在某个位置采集到低电平（0），说明出现了下降沿，可能是起始位。<br>连续采样：**在起始位期间进行16次连续采样。如果没有噪声，这16次采样结果应全为0。**<br>如果有噪声<br>噪声处理：实际电路中可能存在噪声。因此，在检测到下降沿后，额外进行几次采样：<br>**第3、5、7次采样。**<br>**第8、9、10次采样。**<br>噪声判断：在两批采样中，每批3次采样中至少有2个0，才能确认是起始位。如果采样结果满足条件**但有噪声，状态寄存器会设置NE（Noise Error）标志位，提醒数据有噪声**。如果3次采样中只有一个0，则认为不是起始位，电路忽略前面的数据，重新捕捉下降沿。
		> 中间采样<br>如果通过了起始位检测，接收状态从空闲变为接收状态，并且第8、9、10次采样的位置**正好是起始位的正中间**。**之后的数据位也在第8、9、10次进行采样，**确保采样位置在每个位的正中间。
	## STM32串口通信 {toggle="true"}
		在进行STM32串口通信之前，需要对串口通信参数进行配置。这些参数包括波特率、数据位、停止位、校验位等。<br>其中，<br><span color="blue">波特率表示每秒钟传输的bit</span>，在STM32F103系列中，最高可达4.5Mbps；<br><span color="blue">数据位表示每个字符的数据长度(8位或者9位)</span>，<span color="blue">停止位用于表示字符的结束（1bit或者2bit）</span>，<br><span color="blue">校验位用于检查数据传输的正确性（无校验、奇校验或者偶校验）</span>。
		在STM32中，这些参数可以通过配置相应的寄存器来实现。
		<empty-block/>
	## {toggle="true"}
# **13.   串口的学习** {toggle="true"}
	## \[江协\]F1芯片编写串口收发数据 {toggle="true"}
		这里使用的是STM32F103C8T6的最小系统板
		他的USART的串口1是PA9和PA10。
		PA9为TXD、PA10为RXD。
		### 准备工作 {toggle="true"}
			1.接线
			使用到的是USB转串口模块。用到的引脚有RXD、TXD、GND
			RXD是发送数据到STM32的TXD（PA9）
			TXD是接受数据从STM32的RXD（PA10）
			GND是为了有电平的参考。要共地。
			2.新建工程
			这里不详细说了。我添加了一个新的文件给USART用。
			这里的编码格式（在右上角的扳手哪里），选择的个GB2312.这个格式在串口收发的时候可以直接看到汉字。而UTF-8需要配置一些东西（在魔术棒里写一行字）
			并且开启了魔术邦德Use MCicroLIB。<br>这是Keil为嵌入式优化的一个精简库。是用来移植Printf用的。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ace06a65-2f8b-459d-ac30-aa0991a9a705.webp)
		### 编写程序 {toggle="true"}
			由端口的复用功能配置大概可以了解。
			其实对于F103端口，他的开启串口1的方式和GPIO的差不太多。
			**用到的函数在rcc.h和usart.h里。都可以找到。、**
			<empty-block/>
			> 这里是先说明了如何接收，然后再说了如何发送。<br>发送部分分为两种：查询法，中断法<br>查询则是通过读之前说过的输入寄存器的RXNE寄存器的值来说是够了。<br><br>但是对于中断法，需要去配置NVIC代码前期不全。到最后我会补全针对中断的代码。
			<empty-block/>
			<empty-block/>
			> 主模块serial.c
			1. 使能串口所在的GPIO时钟`<br>RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);`
			2. 使能想开启的外设时钟：USART1。（他挂载在APB2总线）
				`RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);`
			3. 分别配置PA9（TXD，发送）、PA10（RXD，接收）为推挽输出和浮空输入（或上拉输入）。
				```c
 		//配置PA9端口为复用推挽输出
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;//复用推挽模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;       //9
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);//初始化

    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;//复用推挽模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;       //9
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);//初始化
				```
			4. 指定启用USART的某个外设（这点跟GPIO那部分的结构体配置很像）<br>这里需要注意，指定USART的模式的时候，需要用 \|  来开启RX和TX两个功能
				```c
    USART_InitTypeDef USART_InitStructure;
    USART_InitStructure.USART_BaudRate = 9600;//波特率。会自动算好填入BRR寄存器
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;//硬件流控制、不使用.
    USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;//用| 使用两个功能
    USART_InitStructure.USART_Parity = USART_Parity_No; //校验位。
    USART_InitStructure.USART_StopBits = USART_StopBits_1;//停止位1位。
    USART_InitStructure.USART_WordLength = USART_WordLength_8b; //不需要校验，所以字长选择8位字长。
    USART_Init(USART1,&USART_InitStructure);
    
    USART_Cmd(USART1,ENABLE);
				```
			5. 发送字节
				在usart.h文件中，有一条发送字节的指令`USART_SendData(USART1,Byte);`
				可以这样去使用他
				其中的`USART_GetFlagStatus` 函数，是读取某个串口中，TXE寄存器（发送移位寄存器移位完成之后，会将寄存器的TXE标志位记为1，这点在\[第九大章、\[江协\]什么是串口通信USART。中有详细提及。）是否为0.如果为0就等待，等待完成之后才发送下一字节。
				```c
void Serial_SendByte(uint16_t Byte)
{
    USART_SendData(USART1,Byte);
    while(USART_GetFlagStatus(USART1,USART_FLAG_TXE) == RESET);//如果写标志位没有置1（没有发送完），那么就等
}
				```
			6. 发送数组
				这点其实是在发送字节的函数上再次包装， 通过for循环来输出数组中的每位。
				因为数组在函数中不能求长度（因为此时的数组是一个地址。储存他的是指针变量）
				```c
void Serial_SendArray(uint8_t* Array,uint16_t len)
{
    uint16_t i = 0;
    for(i = 0; i < len; i++)
    {
        Serial_SendByte(Array[i]);
    }
}
				```
			7. 发送字符串。
				也是在发送字节 的函数上包装，不同的是不用长度了。因为字符串有\\0结束标志位
				```c
//发送字符串
void Serial_SendString(char* String)//有结束的\0 所以不用再传长度了，
{
    uint16_t i = 0;
    for(i = 0; (String[i] != '\0'); i++)
    {
        Serial_SendByte(String[i]);
    }
}
				```
			8. 发送字符形式的数字。
				这点用到的是num先/ 再 %求出个十百千万的位。然后逐字输出。
				先求最大的位。然后+上‘0‘的ASCII码。就能输出字符形式的数字啦。
				这里自己封装了一个pow求次方的函数
				```c

//求次方函数
uint32_t Serial_Pow(uint32_t X,uint8_t Y)
{
    int Num = 1;
    while(Y--)
    {
        Num *= X;
    }
    return Num;
}


//发送字符形式的数字
void Serial_SendNumber(uint32_t Number,uint8_t len)
{
    //从高位像低位取数字然后输出
    uint8_t i = 0;
    for(i = 0; i < len; i++)
    {
        Serial_SendByte(Number / Serial_Pow(10,len - i - 1) %10 + '0');
    
    }
}
				```
			9. 移植printf。。
				这里需要包含两个头文件。“stdio.h”   “stdarg.h”
				用到了**可变参数**这玩意
				不太懂，没学过。直接抄过来吧..
				```c
//下边没学过，直接搬过来的。
/**
  * 函    数：使用printf需要重定向的底层函数
  * 参    数：保持原始格式即可，无需变动
  * 返 回 值：保持原始格式即可，无需变动
  */
int fputc(int ch, FILE *f)
{
	Serial_SendByte(ch);			//将printf的底层重定向到自己的发送字节函数
	return ch;
}

/**
  * 函    数：自己封装的prinf函数
  * 参    数：format 格式化字符串
  * 参    数：... 可变的参数列表
  * 返 回 值：无
  */
void Serial_Printf(char *format, ...)
{
	char String[100];				//定义字符数组
	va_list arg;					//定义可变参数列表数据类型的变量arg
	va_start(arg, format);			//从format开始，接收参数列表到arg变量
	vsprintf(String, format, arg);	//使用vsprintf打印格式化字符串和参数列表到字符数组中
	va_end(arg);					//结束变量arg
	Serial_SendString(String);		//串口发送字符数组（字符串）
}


				```
			> 头文件serial.h
			```c
#ifndef __SERIAL_H 

#include "stm32f10x.h"
#include "stdio.h"
#include "stdarg.h"

#define __SERIAL_H 


//初始化串口
void Serial_Init(void);

//发送字节
void Serial_SendByte(uint16_t Byte);

//发送数组
void Serial_SendArray(uint8_t* Array,uint16_t len);

//发送字符串
void Serial_SendString(char* String);

//发送字符形式的数字
void Serial_SendNumber(uint32_t Number,uint8_t len);

//移植printf
void Serial_Printf(char *format, ...);

#endif

			```
			这时候，我们只需要在主函数中初始化后，就可以发送数据了
			**如果要接受数据，有两种方法。**
			> **第一种方法：查询法**
			则需要读取RXNE（接收移位寄存器，当收的东西都移位完成之后，会把RXNE位 置1.在被读取时清零。(RXNE是位5)<br>那么我们就判断他是否为1.然后存起来输出就OK 拉。
			代码为：
			```c
uint8_t RxData;
    while(1)
    {
        if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE) == SET)
        {
            RxData = USART_ReceiveData(USART1);
            Serial_SendByte(RxData);
        }
    }
}
			```
			这个就是查询法的串口接收程序，程序比较简单是可以考虑这个的。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f425ad55-5f34-41c0-8bd8-1214787c556a.webp)
			> 第二种方法：中断法
			如果要使用这个方法，我们要在初始化函数中添加中断的代码
			流程是：
			1. 开启USART1的RXNE标志位到NVIC的输出。
				`USART_ITConfig(USART1,USART_IT_RXNE,ENABLE);`
			2. 配置NVIC
				```c
    USART_ITConfig(USART1,USART_IT_RXNE,ENABLE);
    
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    NVIC_InitTypeDef NCIC_InitStructure;
    NCIC_InitStructure.NVIC_IRQChannel = USART1_IRQn;
    NCIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NCIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
    NCIC_InitStructure.NVIC_IRQChannelSubPriority = 1;//优先级
    NVIC_Init(&NCIC_InitStructure);
    
				```
			之后我们就可以在中断函数中使用这个中断了。在IT.h中可以找到。是：`USART1_IRQHandler`
			那么函数内可以这么写
			当中断触发，就再次判断是否是RXNE位为1，为真则读取输入寄存器的值，然后赋值给全局变量Serial_RxData，然后把全局变量标志位定义为1.然后手动清除标志位。并自动把全局标志位恢复为0`Serial_GetRxfalg`
			对于获取寄存器的值，也封装了一个函数。`Serial_GetRxData` 。方便获取
			```c
uint8_t Serial_RxData;
uint8_t Serial_RxFlag;

uint8_t Serial_GetRxfalg(void)
{
    if(Serial_RxFlag == 1)
    {
        Serial_RxFlag = 0;
        return 1;
    }
    return 0;
}

uint8_t Serial_GetRxData(void)
{
    return Serial_RxData;
}


void USART1_IRQHandler(void)//中断
{
    if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE) == SET)
    {
        Serial_RxData = USART_ReceiveData(USART1);//读取
        Serial_RxFlag = 1;
        USART_ClearITPendingBit(USART1,USART_FLAG_RXNE);//手动清除标志位
    }

}
			```
			最后的主函数也几乎没变，
			```c
    uint8_t RxData;
    while(1)
    {
        if(Serial_GetRxfalg() == 1)
        {
            RxData = Serial_GetRxData();
            Serial_SendByte(RxData);
        }
    }
			```
			结果如下：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/92591913-cc15-472d-ac47-04b4da12ffe8.webp)
	## \[江协\]什么是串口通信USART {toggle="true"}
		### 串口是什么 {toggle="true"}
			- 串口是一种应用十分广泛的通讯接口，串口成本低、容易使用、通信线路简单，可实现两个设备的互相通信
			- 单片机的串口可以使单片机与单片机、单片机与电脑、单片机与名式各样的模块互相通信，极大地扩展了单片机的应用范围，增强了单片机系统的硬件实力
			- 串口一般使用异步通信，所以一般要双方约定一个通信速率
		### 串口的连接注意事项及分类 {toggle="true"}
			- 简单双向串口通信有两根通信线(发送端TX和接收端RX)
			- TX与RX要交叉连接
			- 当只需单向的数据传输时，可以只接一根通信线
			- 当电平标准不一致时，需要加电平转换芯片
				- 电平标准是数据1和数据0的表达方式，是传输线缆中人为规定的电压与数据的对应关系,1串口常用的电平标准有如下三种:
					- TTL电平:+3.3V或+5V表示1，0V表示0
					- RS232电平:-3\~-15V表示1，+3\~+15V表示0（一般在大型机器上使用）
					- RS485电平**:两线压差**+2\~+6V表示1，-2\~-6V表示0(差分信号) （可以传输上千米。上面俩就几十米）
			<empty-block/>
		### 串口的参数 {toggle="true"}
			1. 波特率:   串口通信的速率<br>串口一般使用异步通信，所以一般要双方约定一个通信速率
			2. 起始位:   标志一个数据帧的开始，固定为低电平<br>高速接收设备我要发送数据了。
			3. 数据位:  数据帧的有效载荷，1为高电平，0为低电平，**低位先行**
			4. 校验位:  用于数据验证，根据数据位计算得来<br>无校验、奇校验和偶校验<br>奇校验就是通过校验位保证数据位和校验位为**奇数个1.**<br>偶校验就是通过校验位保证数据位和校验位为**偶数个1.<br>（更复杂的话可以去了解CRC校验）**
			5. 停止位:   用于数据帧间隔，固定为高电平
		### USART简介及框图介绍 {toggle="true"}
			USART通用同步/异步收发器<br>Universal Synchronous/Asynchronous Receiver/Transmitter
			UART是异步收发器。一般很少用。S只是多了一个时钟输出而已。
			它只支持时钟输出，不支持时钟输入。（或许是为了别的通信协议设计）
			**我们学习的主要是异步通信。**
			<empty-block/>
			USART是STM32内部集成的硬件外设。
			STM32F103C8T6的USART资源有 USART1、USART2、USART3
			其中USART1为APB1总线上、USART2、USART3在APB2总线上挂载
			USART框图
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/2eae46e4-6a07-4f6c-ba98-60ad9cb1a699.webp)
			左边的IRDA之类的是别的通信协议要用的，这里不做了解
			只了解一下串口通信。 
			TX数据发送，是由发送移位寄存器发送的
			同理，RX数据接收，是通过接受移位寄存器接受的。
			最上边的发送（TDR）\\接收（RDR）数据寄存器。发送或者接受的字节数据都存在这里。<br>**这是俩寄存器**，共用同一个地址，在程序上就用一个数据寄存器DR表示。
			TDR是只写的，RDR是只读的。
			他们下边的发送移位寄存器，和接收移位寄存器，是把一个字节的数据，一位一位的移出去。
			如果此时进行了写操作，写入的字节会存放在发送寄存器TDR中，在等待发送移位寄存器的数据移位完毕后，会立刻把这个数据赋值（送）到移位寄存器中。**然后置一个标志位 1 ，**我们只需要检查这个发送数据寄存器TDR置的**TEX**标志位，**TEX**是否为1.只要为1 就可以在TDR写入下一个数据了。<br>移位寄存器会收到发送端控制的控制，向右一位一位的把数据输出到TX引脚。向右移位。也就是**低位先行** 
			接收端也是类似，也是向右移位。当接收移位寄存器满了之后，就会一下子把数据转移到接受数据寄存器RDR。转移的过程中，也会 置一个**RXNE**标志位，。当**RXNE**为1时，就可以把数据读走了 。
			剔除帧头帧尾是电路帮我们自动删除了。
			唤醒单元是用多设备串口通信用的。<br>硬件流控也是交叉连接，用来控制双方数据停止继续的，
			右边的SCLK是会每发送一个字节就输出一下高电平。可以在别的硬件上做自适应的波特率检测。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/78a6b3bb-51d5-4bd3-817f-fd3a5aeb2152.webp)
			再往下就是波特率发生装置，其实就是个分频器。他们的频率不同。
			简化后其实就是这样子
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/526f3154-2506-42a6-af8d-a056530d8ef1.webp)
		### STM32在接受端的天才设计 {toggle="true"}
			STM32在接收时，为了应对噪声。有一个很好的处理方法。对高电平时一次采样，但如果突然检测到下降沿（也就是起始位）
			就会立马开始进行16次采样。3次一组。两组一次。这两组如果全都是0，就代表起始位确实开始了。不是噪声。就可以开始采样了。<br>但是如果发现某组有一个1，俩0.就代表他有些许的噪声，但是不会做处理，这时如果有俩组通过，也算他检测到起始位了。不过会有一个噪声标志位，NE。会提醒你一下，数据我收到了。但是你悠着点用。
			再其次就是有某组为俩0，那么这组就废了。继续开始下一组，直到连着两组通过。就算成功！
			如果通过，他就会在第7  8  9 次采样的时候为有效采样。也是遵循2:1的原则。 7 8 9次检测也是刚刚好在最中间，非常好！
	## \[正点\]F4芯片串口收发步骤 {toggle="true"}
		我没F4芯片，所以就记一下步骤<br>F4多了一个引脚复用映射的函数。把引脚映射为串口<br>并且在GPIO初始化时，使用的模式不是F1的推挽输出和输入了。而是启用了复用功能<br>其他的都一样哒.<br>
		1. 串口时钟使能:   RCC APBxPerphClockCmd();<br>GPIO时钟使能:   RCCAHB1PerphClockcmd();
		2. **引脚复用映射:    GPIO PinAFConfig();**
		3. GPIO端口模式设置:    GPI0 Init();   **模式设置为GPIO Mode AF4**
		4. 串口参数初始化:USART Init();
		5. 开启中断并且初始化NVIC<br>(如果需要开启中断才需要这个步骤）<br>NvIc Init(),<br>USART ITConfig0;
		6. 使能串曰:USART Cmd();
		7. 编写中断处理函数:USARTx IRQHandler();
		8. 串口数据收发:void USART SendData();//发送数据到串口，
			DRuint16 tUSART ReceiveData();/接受数据从DR读取接受到的数据
		9. 串口传输状态获取:<br>FlagStatus USART GetFlagStatus(),<br>void USART CleanTPendingBit();
		<empty-block/>
	## \[江协\]F1芯片串口收发数据包理解与思路 {toggle="true"}
		### 数据包的使用理解 {toggle="true"}
			数据包的作用：把一个个字节打包起来,方便我们进行多字节的数据通信
			比如传输位置的XYZ，如果不封装起来。接收方就不知道那个是X，Y ，Z
			- **对于Hex数据包来说，**如果载荷会出现和包头包尾重复的情况。应选择固定包长。这样可以避免接受错误，不然很容易乱套。<br>如果确定不会出现包头和尾与载荷的重复，那么可以使用可变的包长。因为包头包尾是唯一的
			- Hex的**优点是**，传输最直接，解析数据简单。适合用类似于串口通信的陀螺仪，温湿度传感器等等。<br>**缺点**则是灵活性不好。载荷容易和包头包尾重复
			- **对于文本数据包**（其实每个文本字符的背后，都是一个字节的Hex数据包。不过是把他们编码和译码了）<br>由于译码后，产生的字符。所以我们可以用很多的字符来作为包头和包尾。所以文本是比较灵活的
			- 文本数据包的**优点是**，数据只管易理解。非常灵活。适合输入指令进行人机交互的场合。比如蓝牙模块常用的AT指令、CNC和 3D打印机常用的G代码。<br>**缺点**则是解析效率低，比如发送个数据包100.看着是100但实际上是文本数据1   0   0  还要把文本转换成数据才能得到100.。
				所以，我们要根据实际场景来选择和设计数据包的格式
		### 数据包发送思路 {toggle="true"}
			对于数据包发送，我们是自主可控的。是很简单的，这里大概写个代码
		### 数据包接受思路 {toggle="true"}
			数据包接收这里相较于发送比较难
			这里尝试的是**使用的是固定包长HEX数据包的接收方法。**
			以及尝试使用**可变包长的文本数据包的接受方法。**
			> **HEX固定包长数据包接收**
			在接受数据的时候 ，数据是一个字节一个字节传输进来的。
			每接受一次字符，程序就会进一次中断，在中断函数中可以拿到这个字节。
			所以每拿到一个数据都是一个独立的过程。
			但对于数据包来说，数据是有前后关联性的。 
			所以。要制作一个数据包接受，就要有 **状态机 **的思维
			设计数据包，画一个状态转移图是必要的
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e17025ef-30ae-4418-ae15-6d285babd18f.webp)
			对于上图，固定包长的HEX数据包。
			我们定义了三个状态，等待包头、收到数据、等待包尾
			用标志位S来表示当前在什么状态
			这里的包头定义的是0xFF  。包尾定义的是0xFE。
			在不同的状态，执行不同的程序。
			> **文本的可变包长数据包接收**
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/00da6aff-2d87-4fcc-9259-0e83a6a1d2f0.webp)
			如上图，这里定义的包尾是\\r和\\n。如果不用俩包尾也行，这样更保险。 @是包头
		<empty-block/>
	## \[江协\]F1芯片编写串口收发数据包 {toggle="true"}
		### HEX数据包的发送与接收 {toggle="true"}
			在串口收发数据的中断函数部分上进行更改
			定义包头和包尾为EE和EF
			1. 删除了读取单字节的函数，因为中断部分要用来读取数据包的每个字节
			2. 添加了两个数组：<br>`uint8_t Serial_TxPacket[4] = {0};  `  定义发送数据包数组<br>`uint8_t Serial_RxPacket[4] = {0};   ` 定义接收数据包数组<br>`uint8_t Serial_RxFlag; ` 仍是接收到数据（包）的标志位
			3. 发送数据包部分很简单，只需要这段代码<br>先发送包头，再调用发送数组的函数发送数据包。再发送包尾
				```c
//串口发送数据包
void Serial_SendPacket(void)
{
    Serial_SendByte(0xFF);
    Serial_SendArray(Serial_TxPacket, 4);
    Serial_SendByte(0xFE);
}
				```
			4. 接收部分则用到了 状态机 机的思想。
				`static uint8_t RxState = 0;` 是状态机的状态位置。他的值决定了下次进入中断（收到字节）后对数据的处理
				 ` static uint8_t pRxPacket = 0;` 是定义下次接收的数据放到缓存数组的那个下标的位置。（其实跟指针差不多，但他不是指针。他就是个记录下标的）
				大概流程为
				1. 若接收到包头：0xFF。则进入第二状态（置标志位为1）：存放4位数据<br>然后重置数据缓存存放位置的下标变量`pRxPacket` 准备放东西
				2. 开始放，直到放了4个之后，进入下一状态（置标志位为2）。
				3. 检查是否接收到0xFE，然后重新把标志位置0，然后把接收到数据包标志位置1<br>告诉大家，我的数据包填满了。可以来取了。
				```c
//USART1中断函数
void USART1_IRQHandler(void)
{
    static uint8_t RxState = 0;     //定义表示当前状态机状态的静态变量
    static uint8_t pRxPacket = 0;   //定义表示当前接收数据位置的静态变量
    if (USART_GetITStatus(USART1, USART_IT_RXNE) == SET)        //判断是否是USART1的接收事件触发的中断
    {
        uint8_t RxData = USART_ReceiveData(USART1);             //读取数据寄存器，存放在接收的数据变量
        
        //使用状态机的思路，依次处理数据包的不同部分
        
        //当前状态为0，接收数据包包头
        if (RxState == 0)
        {
            if (RxData == 0xFF)         //如果数据确实是包头
            {
                RxState = 1;            //置下一个状态
                pRxPacket = 0;          //数据包的位置归零。准备开始放东西
            }
        }
        //当前状态为1，接收数据包数据
        else if (RxState == 1)
        {
            Serial_RxPacket[pRxPacket] = RxData;    //将数据存入接收缓存数组的指定位置
            pRxPacket ++;               //数据包的位置自增
            if (pRxPacket >= 4)         //如果收够4个数据
            {
                RxState = 2;            //置下一个状态
            }
        }
        //当前状态为2，接收数据包包尾
        else if (RxState == 2)
        {
            if (RxData == 0xFE)         //如果数据确实是包尾部
            {
                RxState = 0;            //状态归0
                Serial_RxFlag = 1;      //接收数据包标志位置1，成功接收一个数据包
            }
        }
        
        USART_ClearITPendingBit(USART1, USART_IT_RXNE);     //清除移位寄存器移位完成的RXNE标志位
    }
}
				```
				这里我在main函数的测试程序是：
				使用按键，每次按下数组位每位+1.然后显示在OLED屏幕上，
				每次发送，也会被接受，然后显示在OLED上。
				```c
#include "stm32f10x.h"                  // Device header
#include "led.h"
#include "delay.h"
#include "Key.h"
#include "Serial.h"
#include "oled.h"
int main()
{

    KEY_Init();//初始化KEY
    LED_Init();//初始化LED
    Delay_Init();//初始化延时函数
    OLED_Init();//初始化OLED;
    
    Serial_Init();//初始化串口收发数据包
    
    OLED_ShowString(1,1,"TX Packet:");//在OLED1行2列显示发送的数据包
    OLED_ShowString(3,1,"RX Packet:");//在OLED1行4列显示接受的数据包
    
    //初始化数组内的内容
    Serial_TxPacket[0] = 0x01;
    Serial_TxPacket[1] = 0x02;
    Serial_TxPacket[2] = 0x03;
    Serial_TxPacket[3] = 0x04;
    
    while(1)
    {
        if(KEY_Scan(0) == 2)//如果按键2按下，那么对发送的数据包所有字节++。然后显示发送
        {
            
            Serial_TxPacket[0]++;
            Serial_TxPacket[1]++;
            Serial_TxPacket[2]++;
            Serial_TxPacket[3]++;
            //显示
            OLED_ShowHexNum(2,1,Serial_TxPacket[0],2);
            OLED_ShowHexNum(2,4,Serial_TxPacket[1],2);
            OLED_ShowHexNum(2,7,Serial_TxPacket[2],2);
            OLED_ShowHexNum(2,10,Serial_TxPacket[3],2); 
            //发送数据包
            Serial_SendPacket();
        }

        
        
        //接收到的数据放到OLED上显示
        if(Serial_GetRxFlag() == 1)
        {
            OLED_ShowHexNum(4,1,Serial_RxPacket[0],2);
            OLED_ShowHexNum(4,4,Serial_RxPacket[1],2);
            OLED_ShowHexNum(4,7,Serial_RxPacket[2],2);
            OLED_ShowHexNum(4,10,Serial_RxPacket[3],2);
        }
    

    }
}


				```
				程序结果如下：
				俩是可以同时显示的
				<columns>
					<column>
						![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/2a226cad-b2a3-4399-ad79-84e891797955.gif)
					</column>
					<column>
						<empty-block/>
						![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/96276b57-6385-41c5-a000-be104c53afe1.gif)
					</column>
				</columns>
		### 文本数据包的接收 {toggle="true"}
			文本数据包的接收是在HEX数据包的基础上改进的。
			因为发送文本数据包使用printf或者自己写的SendString就可以了<br>不然挺麻烦的。所以这里就写一个文本数据包的接受就可以了。
			这里的数据头为@，数据结尾是\\r \\n 
			为什么是\\r和\\n呢。我个人觉得是因为在你发送一段文本的时候，他后面默认带的就是\\r\\n。
			要制作文本接受。要先处理一下HEX的代码
			首先把不用的函数和定义的常量都删掉。
			比如`uint8_t Serial_TxPacket[4] = {0};    定义发送数据包数组` 这行话（记得把头文件里的声明也去除掉。），<br>还有这段函数
			```c
//串口发送数据包
void Serial_SendPacket(void)
{
    Serial_SendByte(0xFF);
    Serial_SendArray(Serial_TxPacket, 4);
    Serial_SendByte(0xFE);
}
			```
			然后就要开始改造HEX数据包了！
			首先，我们接收的是字符串了。不是数组，并且字符串的长度是可变的。所以可以把原本的接收数组缓存区的`uint8_t Serial_RxPacket[4] = {0};   ` 改为char `Serial_RxPacket[100]= {0};` 这样我们就有足够的空间去存放未知的字符串了。
			剩下的就是去修改中断函数里的状态机部分了
			在接受字符串包的时候。首先要判断@。然后需要先判断是否遇到了结束标志。再进行存放字符，因为我们不知道现在这个字符是不是包尾标志。在遇到\\r之后，需要再判断一下。\\n 如果\\n有效，。那么就可以把接收到包的位置1。
			代码如下，几乎是没怎么变的。
			```c
//USART1中断函数
void USART1_IRQHandler(void)
{
    static uint8_t RxState = 0;     //定义表示当前状态机状态的静态变量
    static uint8_t pRxPacket = 0;   //定义表示当前接收数据位置的静态变量
    if (USART_GetITStatus(USART1, USART_IT_RXNE) == SET)        //判断是否是USART1的接收事件触发的中断
    {
        uint8_t RxData = USART_ReceiveData(USART1);             //读取数据寄存器，存放在接收的数据变量
        
        //使用状态机的思路，依次处理数据包的不同部分
        
        //当前状态为0，接收数据包包头
        if (RxState == 0)
        {
            if (RxData == '@')         //如果数据确实是包头
            {
                RxState = 1;            //置下一个状态
                pRxPacket = 0;          //数据包的位置归零。准备开始放东西
            }
        }

        
        //当前状态为1，接收数据包数据
        else if (RxState == 1)
        {
            if(RxData == '\r')
            {
                RxState = 2;
            }
            else
            {
                Serial_RxPacket[pRxPacket] = RxData;    //将数据存入接收缓存数组的指定位置
                pRxPacket ++;               //数据包的位置自增
            }
            
        }
        //当前状态为2，接收数据包包尾
        else if (RxState == 2)
        {
            if (RxData == '\n')         //如果数据确实是包尾部
            {
                RxState = 0;            //状态归0
                Serial_RxPacket[pRxPacket] = '\0';//添加字符串结束标志位\0
                Serial_RxFlag = 1;      //接收数据包标志位置1，成功接收一个数据包
            }
        }
        
        USART_ClearITPendingBit(USART1, USART_IT_RXNE);     //清除移位寄存器移位完成的RXNE标志位
    }
}

			```
			验证仍然是同上，就不演示了
			现在既然接收到了字符串数据，我们就可以利用C语言自带库中的String.h这个头文件中的strcmp这个字符串对比的指令（相同为0）
			就可以实现点灯操作了！
			代码参考：
			```c
#include "stm32f10x.h"                  // Device header
#include "led.h"
#include "delay.h"
#include "Key.h"
#include "Serial.h"
#include "oled.h"
#include "string.h"
int main()
{

    KEY_Init();//初始化KEY
    LED_Init();//初始化LED
    Delay_Init();//初始化延时函数
    OLED_Init();//初始化OLED;
    
    Serial_Init();//初始化串口收发数据包
    
    OLED_ShowString(1,1,"Result:");//在OLED1行2列显示当前命令执行结果
    OLED_ShowString(3,1,"RX Packet:");//在OLED1行4列显示接受的数据包
    
    //初始化数组内的内容
    
    while(1)
    {
        //接收到的数据放到OLED上显示
        if(Serial_GetRxFlag() == 1)//接收到数据
        {
            OLED_ShowString(4, 1, "                 ");
            OLED_ShowString(4, 1,Serial_RxPacket);
            
            
            if( strcmp(Serial_RxPacket,"LED_ON(1)" ) == 0)//如果=0则代表字符串相等
            {
                LED_ON(1);//执行命令并显示
                OLED_ShowString(2,1,"LED1_ON_OK");
            }
            else if( strcmp(Serial_RxPacket,"LED_ON(2)" ) == 0)//如果=0则代表字符串相等
            {
                LED_ON(2);//执行命令并显示
                OLED_ShowString(2,1,"LED2_ON_OK");
            }
            else if( strcmp(Serial_RxPacket,"LED_OFF(1)" ) == 0)//如果=0则代表字符串相等
            {
                LED_OFF(1);//执行命令并显示
                OLED_ShowString(2,1,"LED1_OFF_OK");
            }
            else if( strcmp(Serial_RxPacket,"LED_OFF(2)" ) == 0)//如果=0则代表字符串相等
            {
                LED_OFF(2);//执行命令并显示
                OLED_ShowString(2,1,"LED2_OFF_OK");
            }
            else
            {
                OLED_ShowString(2,1,"ERROR_CMD");
            }
        }
    }
}



			```
			结果如图：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c3e56017-758d-46e5-b7bd-f50ef812957d.gif)
# **14.   外部中断** {toggle="true"}
	## \[正点&江协\]什么是外部中断，他的特点是什么 {toggle="true"}
		在主程序运行过程中，出现了特定的中断触发条件(中断源)，使得CPU暂停当前正在运行的程序，转而去处理中断程序，处理完成后又返回原来被暂停的位置继续运行
		中断分为内核的中断和外设的中断
		**中断是由NVIC根据优先级。去判断然后根据优先级告诉CPU**
		**stm32的GPIO口可以当做外部中断的输入**
		STM32F4的中断控制器支持22个外部中断/事件请求
		**那么外部中断就是EXTI了**
		**EXTI可以检测电平信号。（上升，下降。双边沿、软件触发）**
		EXTI线0-15:对应外部IQ口的输入中断<br>EXTI线16:连接到PVD输出。<br>EXTI线17:连接到RTC闹钟事件。<br>EXTI线18:连接到USB OTG FS唤醒事件。<br>EEXTI线19:连接到以太网唤醒事件<br>XTI线20:连接到USB OTG HS(在FS中配置)唤醒事件:XTI
		线21:连接到RTC入侵和时间戳事件。<br>EEXTI线22:连接到RTC唤醒事件。
		由此我们大概可以了解到，STM32的供GPIo使用的中断线是有限的。
		需要有终端线才能产生中断请求。
		GPIO与中断线的映射关系就是这次的重点
		**同一时间只能有一个pin口映射到中断线**
		**因为GPIOA 的1和GPIOB的1….他们的引脚是在一个映射器上的。**
		对于每个中断线，可以设置相应的触发方式，比如上升沿下降沿边沿触发之类的。还包是否要开启等等。
		那么，16个中断线就要有16个中断服务函数呢？  不<br>在F4系列中，只有7个可使用的中断服务函数。<br>F4中，外部中断线5-9分配分配一个、10-15分配一个中断向量。他们都是共用一个中断函数，
		所以我们只能是用EXTI 0 1 2 3 4 9_5 15_10这七个。
		也就是说，gpio0只能映射到EXTI0    GPIO3 只能映射到EXTI3   而 GPIO6需要映射到EXTI9_5
		对于F103c8t6.同一个pin他的连接是下边这样（图中都是pin16）所以也由此可知，对于同一个pin、是不能同时发生中断请求的。
		**AFIO是一个数据选择器。**它可以把前边16个GPIO口的**其中一个**连接到EXTI外部中断中
		通道数量可以看到。一共有16个GPIO_Pin口+PVD+RTC+USB+ETH。其余四个其实是来蹭网的。PVD是外加PVD输出，RTC为闹钟、USB唤醒USB、以太网唤醒ETH。
		所以EXTI有20个输入信号。
		如图，再EXTI外部中断连接到NVIC时，把9-5   15-10  goio放到了一个通道里。 **所以在用的时候需要判断一下到底是谁弄的**
		其他相应，则是之前说的事件响应。
		**AFIO的作用不止于此，我们之前用的 端口重映射（把普通的gpio映射为他复用的功能）。也是他干的。**
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/da444e37-b48a-48ac-8a1f-007fccd967fa.webp)
	## \[江协\]外部中断的再理解 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/53b0b279-aca5-4483-90d5-65fd29233915.webp)
		如图，是EXTI的框图，在触发终端的二时候，**请求挂起寄存器就会置一。就相当于中断的标志位**
		再往后走，可以看到 请求挂起寄存器和中断屏蔽寄存器，共同进入一个与门。 所以当中断屏蔽寄存器为0时，中断就失能了。如果他们都为1，则进入NVIC进行下一步
		事件的也是一样。（二十根线都是这样子的。）
	## \[江协\]外部中断的适用场景 {toggle="true"}
		外部中断的应用场合 是 ** “单片机事先不知道我们要去操作，但如果一操作就需要处理的突发事件,并且信号的来源是外部的，stm32不知道什么时候来，并且信号速度很快，要迅速执行”**
		<empty-block/>
	## \[江协\]外部中断EXTI函数作用介绍 {toggle="true"}
		- `void EXTI_DeInit(void);`<br>把EXEI配置清除，恢复成上电默认状态
		- `void EXTI_Init(EXTI_InitTypeDef* EXTI_InitStruct);`<br>根据结构体中的参数配置EXTIi外设
		- v`oid EXTI_StructInit(EXTI_InitTypeDef* EXTI_InitStruct);`<br>把参数传递的结构体变量赋一个默认值
		- `void EXTI_GenerateSWInterrupt(uint32_t EXTI_Line);`<br>软件触发外部中断
		- F`lagStatus EXTI_GetFlagStatus(uint32_t EXTI_Line);`<br>获取指定的标志位是否被置一(建议主函数中使用)
		- `void EXTI_ClearFlag(uint32_t EXTI_Line);`<br>对指定的标志位清除(议主函数中使用)
		- `ITStatus EXTI_GetITStatus(uint32_t EXTI_Line);`
			获取中断标志位是否被置一(只能在中断中使用)
		- `void EXTI_ClearITPendingBit(uint32_t EXTI_Line);`
			对中断标志位是清除(只能在中断中使用)
	## \[江协\]NVIC函数作用介绍 {toggle="true"}
		- `void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup);`
			中断分组用，参数为中断分组的方式
		- `void NVIC_Init(NVIC_InitTypeDef* NVIC_InitStruct);`
			根据结构体里的配置来初始化NVIC
		- `void NVIC_SetVectorTable(uint32_t NVIC_VectTab, uint32_t Offset);`
			设置中断向量表，
		- `void NVIC_SystemLPConfig(uint8_t LowPowerMode, FunctionalState NewState);`
			系统低功耗配置
		- `void SysTick_CLKSourceConfig(uint32_t SysTick_CLKSource);`
			之前讲滴答定时器用过了。
	## \[江协\]编写：红外传感器计次 {toggle="true"}
		代码如下，已经解释的很全了
		```c
#include "stm32f10x.h"                  // Device header

//引脚为A2  

//计次变量
uint16_t CountSensor_Count = 0;

//初始化
void CountSensor_Init(void)
{
    //开启A2的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    //开启挂载在APB2外设的AFIO外设时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO,ENABLE);
    //EXTI时钟和NVIC的时钟不用手动打开。
    
    //配置GPIO
    GPIO_InitTypeDef GPIO_InitStructrue;
    GPIO_InitStructrue.GPIO_Mode = GPIO_Mode_IPU;        //上拉输入模式
    GPIO_InitStructrue.GPIO_Pin = GPIO_Pin_2;
    GPIO_InitStructrue.GPIO_Speed = GPIO_Speed_50MHz;    
    GPIO_Init(GPIOA,&GPIO_InitStructrue);
    
    //配置AFIO （F1中在GPIO.h文件中）（目的是为了把GPIOA的PIN2引脚映射到AFIO中。）
    GPIO_EXTILineConfig(GPIO_PortSourceGPIOA,GPIO_PinSource2);//这里需要根据PIn和GPIOx来选择

    
    //配置EXTI
    EXTI_InitTypeDef EXTI_InitStructure;
    EXTI_InitStructure.EXTI_Line = EXTI_Line2;          //选择中断线路，这里是PIN2 所以为2   
    EXTI_InitStructure.EXTI_LineCmd = ENABLE;           //是否使能指定的中断线路
    EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt; //中断或响应模式   
    EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;//上升或下降或边沿触发
    EXTI_Init(&EXTI_InitStructure);
    
    //配置NVIC（因为NVIC属于内核，所以被分配到内核的杂项中去了，在misc.c）
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//配置抢占和响应优先级
     
    NVIC_InitTypeDef NVIC_InitStructure;
    NVIC_InitStructure.NVIC_IRQChannel = EXTI2_IRQn;       //在stm32f10x.h文件里。让你找IRQn_Type里的一个中断通道。这里使用的是md的芯片（如果引脚是15-10或者9-5则需要去找对应的那个）这里我是PIN2.所以找2
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;         //是否使能指定的中断通道
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;//抢占优先级（这里可以看表。看范围，）
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;       //指定响应优先级
    NVIC_Init(&NVIC_InitStructure);    
    
   
    
}



//获取当前的计数
uint16_t CountSenSor_Get(void)
{
    return CountSensor_Count;
}


//在STM32中，中断的函数都是固定的。他们在启动文件中存放xxx.s 
//以IRQHandler结尾的就是中断函数的名字。
//在这里需要找到对应的中断函数，我这里是2
void EXTI2_IRQHandler(void)//中断函数是无参无返回值的。中断函数必须写对，写错就进不去
{
    //在进入中断后，一般要判断一下这个是不是我们想要的那个中断源触发的中断。
    //但是在这里。我是GPIOA的PIN2引脚，所以不用写。
    //如果是5-9 10-15的引脚。他们EXTI到NVIC是几个共用的。
    //所以需要根据EXTI输入时的16根引脚。来判断是16根引脚的那一根发送的中断请求。
    //这里规范写的话需要加上去
    //查找标志位函数在exit.h中。
    if(EXTI_GetITStatus(EXTI_Line2) == SET)//第一个参数是行数.判断这个线的标志位是不是== SET。是则是我们想要的
    {
    
        CountSensor_Count++;
        EXTI_ClearITPendingBit(EXTI_Line2);//中断结束后，要调用清除标志位的函数。如果你不清除，程序会一直进入中断
    }
}



		```
		.h文件如下
		```c
#ifndef __COUNTSENSOR_H
#define __COUNTSENSOR_H


//初始化旋转编码计次
void CountSensor_Init(void);

//获取当前计数值
uint16_t CountSenSor_Get(void);

#endif

		```
		结果：
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/08fe931a-e533-423c-ac88-6d19a8681102.gif)
# **15.   STM32时钟树理解** {toggle="true"}
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/cf65c762-58dc-4c23-b08e-66fa91139214.webp)
	如图，是STM32的时钟树。可以从中间划个界限
	左边是时钟产生电路， 右边是时钟分配电路
	中间的SYSCLK 就是系统时钟72MHZ
	在时钟产生电路
	**有四个震荡源：**分别是  内部 高速 8MHZ RC振荡器、外部 高速 4-16MHZ 石英晶体振荡器（一般为8M）、内部40KHZ 低速 RC振荡器（一般给看门狗提供时钟）、 外部 32.768KHZ 低速晶振（一般给RTC提供时钟）
	**在启动时钟时：**（在Systemlinit函数中）他首先会启动8MHZ的内部时钟。选择8MHZ为系统的时钟。然后选择外部的8Mhz时钟。配置外部时钟到PLL锁相环进行倍频9倍、得到72MHZ.等到锁相环稳定之后，才使用外部的时钟。
	**CSS是时钟检测电路**，他会检测外部时钟的运行状态，如果外部不行就自动切换回内部。保证系统时钟的运行
	系统的时钟会先进入AHB总线。
	AHB后边有个分频器,在Systemlinit函数中配置的分配系数为1.<br>**所以AHB总线的系统时钟就是72MHZ、**<br>然后进入ABP1的与分频器，他的配置系数为2<br>**所以APB1总线的系统时钟就是36MHZ<br>**但再往下看，可以看到，APB1总线的36Mhz又兵分两路，一路直接给到了APB1外设。另一路又经过一个**倍频器**（分频器设置的是：如果APB1分频器分频系数为1，也就是没分频，那么我也不分频。如果他分频了。那么我再把他的频率x2），又把频率拉回了72MHZ。**所以所有的定时器都是72MHZ的频率。**（定时器1 8 为高级定时器。直接在APB2总线上。频率为72M、2-7为通用和基本定时器。经过32M倍频后又回到72MHZ）这里的倍频器输出到的就是Tm 2 - 7<br>然后AHB时钟再进入APB2.APB2的预分频器为1。<br>所以APB2总线频率为72MHZ <br> （同理也是有一个倍频器、分频器设置的是：如果APB1分频器分频系数为1，也就是没分频，那么我也不分频。如果他分频了。那么我再把他的频率x2。因为APB2没分频。所以这里的倍频器也没起作用。）这个倍频器输出到的就是TIM1 和 8的高级定时器
	<empty-block/>
	> 可以看到，不管是APB1还是APB2或者TIM的时钟输出部分。**前边都有一个与门**.这里就是供我们去控制的地方。<br>也就是我们输入的RCC_APB1/2PeriphClockCmd作用的地方，我们实际控制的就是这么个与门
	<empty-block/>
# **16.   定时器部分** {toggle="true"}
	## 三种定时器介绍 {toggle="true"}
		### 什么是定时器 {toggle="true"}
			- 定时器TIM(Timer)定时器
			- 可以对输入的时钟进行计数，并在计数值达到设定值时触发中断
			- **16位计数器**、**预分频器**、**自动重装寄存器**的**时基单元**，在72MHz计数时钟下可以实现最大59.65s的定时（72M/65536/65536）
			- 不仅具备基本的定时中断功能，而且还包含内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等多种功能
			- 根据复杂度和应用场景**分为了高级定时器、通用定时器、基本定时器三种类型**
			<table header-row="true">
			<colgroup>
			<col width="96">
			<col width="96">
			<col width="65">
			<col width="371">
			</colgroup>
<tr>
<td>类型</td>
<td>编号</td>
<td>总线</td>
<td>功能</td>
</tr>
<tr>
<td>高级定时器</td>
<td>TIM1、TIM8</td>
<td>APB‘2</td>
<td>拥有通用定时器全部功能，并额外具有重复计数器、死区生成、互补输出、刹车输入等功能</td>
</tr>
<tr>
<td>通用定时器</td>
<td>TIm2-5</td>
<td>APB‘1</td>
<td>拥有基本定时器全部功能，并额外具有内外时钟源选择输入捕获、输出比较、编码器接口、主从触发模式等功能</td>
</tr>
<tr>
<td>基本定时器</td>
<td>TIM6、TIM7</td>
<td>APB‘1</td>
<td>拥有定时中断、主模式触发DAC的功能</td>
</tr>
			</table>
		### 基本定时器框图讲解 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/93e724e1-a1e4-449d-bf9e-e73b59b8b05a.webp)
			对于基本定时器，他的时钟信号来源只能是内部时钟RCC的72MHZ，<br>时钟经过预16位的分频器PSC、最大可以分频65536。当我们输入0，即为不分频（或者1分频）<br>输入1则为2分频。所以预分频器PSC的实际分频系数比我们输入的多1。
			然后预分频器的信号输入到计数器CNT中，CNT也是一个16位的计数器。**每来一个上升沿CNT计数器就会自增+1（基本定时器仅支持向上计数）**，当**CNT计数器与自动重装在寄存器的值一样时，**就会触发一个**外部中断响应和事件响应。<br>**可以在图中看到，向下的箭头是事件响应，他的响应可以选择直接映射到DAC。这样就可以不通过中断，靠硬件电路来实现DAC的控制
			预分频器、自动重装栽寄存器、CNT计数器组成了基础的**时基单元**
		### 通用定时器框图讲解 {toggle="true"}
			对于通用定时器，计数的模式不止有向上计数。还有向下计数、中央对其模式**三种模式**
			向上模式可以理解为：↗↓↗↓
			向下计数可以理解为：↘↑↘↑
			中央对其模式可以理解为：↗↘↗↘
			这时通用定时器的框图，上半部分是定时器的时钟来源。当前的红线是，**外部时钟模式2<br>**他的信号是由外部引脚TIMx_ETR（这个是由引脚复用之后得到的功能）接入信号。然后经过处理（极性选择、边沿检测和与分频、再经过滤波电路）。输出的信号兵分两路，一路进入下方的**信号选择器**、另一路通过**ETRF**进入了**触发控制器。**这时信号就正儿八经进入到了刚才所说的基本定时器的时基单元处。其他都一样了。
			再往下看，刚才另一路的信号经过**信号选择器**可以进入到TRGI通道、这个主要是用作触发定时器的从模式（从模式这一部分后续再来补充，现在没学），现在可以暂时把他当做定时器的外部输入来理解。当他是外部输入的时候，这个模式也叫做定时器的**外部时钟模式1**
			再往下看，刚才的**信号选择器**的输入除了刚才讲过的，还有TRC信号。这个TRC信号又有一个选择器来选择。
			先看左边的ITR信号。他们是来自其他的计时器。（可以看到在触发控制器的输出部分，有一个通道叫TRDO，他可以输出信号到其他定时器。可以把计数器计数后的更新时间映射到这个位置）。这四个ITR信号，分别来自其他的四个定时器信号（具体可以查手册的表（从模式控制器小节）来找）。这样就可以实现定时器的级联。
			再往下看，TRC左边的选择器的输入还可以由TI1F_ED（ED是边沿的意思，上升下降均有效）这里连接的是输入捕获的CH1引脚
			再往右上角看，TRGI通道的左边的选择器，它的输入还可以由TI1FP1输入，或者TI2FP2输入。他们分别是和输入补货CH1、CH2引脚链接（中间经过了输入滤波器和边沿检测器）。
			> 到这里，外部时钟模式1 的输入就介绍完了。总结一下就是。外部输入模式1 的输入可以是ETR引脚、其他定时器、CH1引脚的 边沿、CH1引脚、以及CH2引脚，<br>一般情况下，外部时钟通过ETR引脚进入就ok了（也就是时钟模式2）。
			再往右上角看，可以看到一个编码器接口。它的输入是TI1FP1和TI2FP2。这个可以读取正交编码器的波形，后续会讲
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ca0b6259-e3cf-4526-9cb0-8ac4363421f3.webp)
			下面看下面的一部分，下边的部分可以看成两块。
			右边的是输出比较电路，总共有四个通道。分别对应CH1到CH4。可以用于输出PWM波、驱动电机
			左边的是输入捕获电路，总共有四个通道，分别对应CH1到CH4的引脚，可以用来测量输入方波的频率等等
			在最中间（其实是靠右）的CNT计数器下方，这四个是输入捕获/输出比较电路共用的（因为他俩不能同时用）。
		### 高级定时器框图大概介绍 {toggle="true"}
			高级定时器相比于通用定时器就加了这些东西
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9e790623-aed4-4993-92c6-007741229263.webp)
			在出发中断和事件响应的线路中，多了一个重复次数寄存器。他可以实现几个周期才输出一次。相当于又对他进行了分频（还是16位的。最大值为65536）
			其他看就是DTG寄存器和DTG 是死区生成电路（这里是为了防止直通的现象。在驱动三项无刷电机的时候。在切换的时候使用）
			右边的输出，从通用定时器的一个，变为了俩互补的输出， 可以输出两对互补的PWM波（这是用来驱动三项无刷电机）第四路还是一个。
			最左下角。则是如果输入信号TIMx_BKIN（刹车信号）如果他使能了，那么控制电路就会自动切断电机的输出，防止意外的发生
			还有CSS，这个是为了防止外部时钟失效，如果外部时钟失效，就会立刻停止，
	## 通用定时中断基本结构 {toggle="true"}
		如下为框图
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4726d093-6fe8-4228-8604-1fe428b96ccc.webp)
		运行控制，是控制时基单元用的
		左边是为时基单元提供时钟的部分
		右侧则是中断输出控制。因为定时器会产生很多的中断。（在框图中的小箭头就是）<br>这里就需要我们手动的允许一下某些中断的通过。
	## 预分频器的时序 {toggle="true"}
		计数器计数频率  :    CK_CNT=CK_PSC/(PSC+1)<br>时钟频率/预分频器值+1
		看完这个图之后，其实计数器的**框图中， 框框带影子的，其实就是影子寄存器：**（
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ce0cc798-02a0-44ff-8aa7-82ac7e8cbeaa.webp)
	## 计数器的时序图 {toggle="true"}
		（这里产生的中断标志位。也是需要在读取后手动清除）
		寄存器溢出频率:   CK_CNT_OV=CK_CNT/(ARR +1)<br>计数器溢出频率 = 与分频器的频率/自动重装栽值 +1。<br>带入上个时序图的公式 得到 <br>计数器溢出频率=CK_PSC/( PSC+1)/(ARR + 1 )<br>也就是 =     时钟频率/预分频器的值+1/自动重装寄存器的值+1
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5618a19e-9945-4f01-b087-1af296ed4baa.webp)
		这里可以再理解一下**预**计数器
		有预装计数器时序和无预装计时器时序
		- 无预装时序计数器
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5e3e395a-b96f-4a0c-84cc-ed521c11c696.webp)
		可以看到，在更新自动加载寄存器为36周。立马就改变了。如果此时计数器寄存器的值为37.那么他并不会开始下个周期。而是一直++++++++。+到FFFF之后才会重新清零。
		- 有预装时序计数器
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3ce9b3b9-3440-4a25-9a84-17845144f335.webp)
		这里的和之前说过的与分频器是一样的。
		如果我们突然改变了自动重装寄存器ARR的值。他不会立刻改。而是等待这个周期结束才改变。
		<empty-block/>
		**这其实也是同步和异步的区别使用影子寄存器就是同步**
	## tim.h库定时器中断和外部计数的常用函数介绍 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5d8814f3-76a1-4eb9-a885-1bfd8e1efeb6.webp)
		> **时基单元部分**
		`TIM_DeInit` 回复缺省配置（恢复初始化前的状态。理解成停止吧）
		`TIM_TimeBaseInit` **时基单元初始化**
		`TIM_TimeBaseStructInit` 把结构体变量赋一个默认值
		> **运行控制部分**
		`TIM_Cmd` 选择寄存器使能还是失能
		> **中断控制部分**
		`TIM_ITConfig` 使能终端输出信号（中断输出控制） 选择定时器、选择配置那个中断输出，使能还是失能
		> **时基单元的始终选择部分**
		`void TIM_InternalClockConfig(TIM_TypeDef* TIMx);` 选择内部时钟
		`void TIM_ITRxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource);` 选择ITRx其他定时器的时钟
		`void TIM_TIxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_TIxExternalCLKSource,<br>uint16_t TIM_ICPolarity, uint16_t ICFilter);` 选择TIx捕获通道的时钟（选择TIx的具体某个引脚、输入 极性和滤波器）
		`void TIM_ETRClockMode1Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity,<br>uint16_t ExtTRGFilter);` 选择ETR通过外部时钟模式1输入的时钟。（预分频器，极性和滤波器）
		`void TIM_ETRClockMode2Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler,<br>uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter);` 选择ETR通过外部时钟模式2输入的时钟（预分频器，极性和滤波器）
		`void TIM_ETRConfig(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity,<br>uint16_t ExtTRGFilter);` 用来配置ETR引脚的预分频器、极性和滤波器….
		> NVIC见外部中断哪一章
		> 单独改写部分参数部分
		`TIM_PrescalerConfig` 单独写预分频值(TIMx、要分频的值、写入的模式 （两种）
		`TIM_CounterModeConfig` 改变计数器模式
		`TIM_ARRPreloadConfig` 是否开启计数器的预装模式（是否使用影子寄存器）
		`TIM_SetCounter` 手动给计数器写个值
		`TIM_SetAutoreload` 手动给重装寄存器写个值
		`TIM_GetCounter` 获取当前计数器的值
		`TIM_GetFlagStatus` 获取当前预分频器的值
		> 获取和清除中断标志位
		`FlagStatus TIM_GetFlagStatus(TIM_TypeDef* TIMx, uint16_t TIM_FLAG);`
		`void TIM_ClearFlag(TIM_TypeDef* TIMx, uint16_t TIM_FLAG);`
		`ITStatus TIM_GetITStatus(TIM_TypeDef* TIMx, uint16_t TIM_IT);`
		`void TIM_ClearITPendingBit(TIM_TypeDef* TIMx, uint16_t TIM_IT);`
		<empty-block/>
	## 编写：定时器定时中断计数 {toggle="true"}
		定时器中断 （这里是向上计数）就是是 计数器CNT的数字到达重装寄存器ARR之后。发生的中断（除此之外还有一个时间响应）。
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4726d093-6fe8-4228-8604-1fe428b96ccc.webp)
		1. 要使用定时器TIM2，首先要把通往TIM2的时钟打通。TIM2在时钟树上可以看到，是挂载在APB1总线上的。`RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);`
		2. 开启时钟之后，就可以去选择时钟源了，时钟源有很多的选择。一般用的是内部时钟或者是外部时钟。外部时钟可以走外部时钟模式1或者外部时钟模式2. **这里选择内部时钟**模.这点需要在tim.h找到相应的函数：`TIM_InternalClockConfig(TIM2);` 
		3. 配置通时钟输入之后，就可以开始配置中间的时基单元了，配置时基单元需要用到一个结构体，然后使用初始化函数，这点跟GPIO的一样。作用可以 看下边的。
		```c
TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;//初始化时基单元要定义的结构体
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;//指定时钟分频、1分频
    //这里的分频是为了给滤波器采样频率f  采样点数N f越低、N越多。采样点数越好。
    //但信号延迟越大。f可以有内部时钟直接而来，也可以由分频器分频的到。
    //这里的分频器就是对送往滤波器的f的分频器进行控制。在这里直接用1就好，和时基单元关系不大
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;//选择计数模式，分别是向上、向下、三种中央对其模式
    TIM_TimeBaseStructure.TIM_Period = 10000 - 1;//ARR重装值  =  值 +1   所以我们再用的时候要-1.才能正确匹配。这里是ARR和PSC定时1s
    TIM_TimeBaseStructure.TIM_Prescaler = 7200 - 1;//PSC分频器分频系数= 值+1
    TIM_TimeBaseStructure.TIM_RepetitionCounter = 0;//重复计数器，这个是高级计数器才有的，我们不需要。所以给0
    TIM_TimeBaseInit(TIM2,&TIM_TimeBaseStructure);//这个函数是初始化时基单元的。
    TIM_ClearFlag(TIM2,TIM_IT_Update);          //清除更新标志位
		```
		   `TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1` 这个结构体参数去看注释。
		大概的意思是他是给滤波器的分频器。填写的值域F  N 有关 。 F是采样频率。越高越好。N是采样次数。 意思是只要我采样到N次 才算他有效。填写啥东西 对应的F和N在手册中可查
		<empty-block/>
		程序在运行`TIM_TimeBaseInit` 函数时，会在最后一步手动更新一下更新事件。因为<br>计时器和ARR重装寄存器。他们都有一个 影子寄存器  因为影子寄存器的存在。我们写入的值（重装寄存器ARR和计数器的值CNT）不会立刻生效，而是等待**此次周期结束**，**也就是在更新事件的时候才会生效。**所以这个函数在最后的时候，**手动生成了一下更新事件。<br>**但副作用是，**更新事件和更新中断是同时发生作用**的。所以此时的中断标志位已经置一了。（中断标志位 不手动清除会一直为1）。所以当程序起来之后，会立马进入中断一次。<br>所以在上电复位之后，在查看计数器时已经是1了）<br>所以在这里我们需要手动的先清除一下此次的更新中断标志位<br>`TIM_ClearFlag(TIM2,TIM_IT_Update); ` 
		1. 再往下走。需要配置TIm的中断输出控制<br>要使能我们的 CNT计数到ARR时产生的中断，才能被NVIC接受到。<br>`TIM_ITConfig(TIM2,TIM_IT_Update,ENABLE);//使能TIM2的更新中断到NVIC`
		2. 下一步就比较熟悉了。 使能NVIC
			```c
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//选择分组
    NVIC_InitTypeDef NVIC_InitStructuer;//定义结构体
    NVIC_InitStructuer.NVIC_IRQChannel = TIM2_IRQn;//选择定时器2在NVIC里的通道
    NVIC_InitStructuer.NVIC_IRQChannelCmd = ENABLE;//是否使能
    NVIC_InitStructuer.NVIC_IRQChannelPreemptionPriority = 2;//抢占优先级
    NVIC_InitStructuer.NVIC_IRQChannelSubPriority = 2;//响应优先级。
    NVIC_Init(&NVIC_InitStructuer);//初始化NVIC
			```
		3. 最后一步！
			使能寄存器！！`TIM_Cmd(TIM2,ENABLE);`
		4. 中断函数的配置。是进入中断是 num++
			这里我们声明了 main.c里的函数里的Num （声明的时候不能同时赋值，否则会出错T.T）
			```c
//配置中断函数
void TIM2_IRQHandler(void)
{
    if(TIM_GetITStatus(TIM2,TIM_IT_Update) == SET)//检查中断标志位
    {
        Num++;
        TIM_ClearITPendingBit(TIM2,TIM_IT_Update);//清除标志位
    }
}

			```
			> **总代码如下！！**
		```c
#include "stm32f10x.h"                  // Device header

extern uint16_t Num;//记录时间的变量、这里是引用了main.c里的Num来控制。

void Timer_Init(void)
{
    //开启RCC时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);//TIM2是挂载在APB1总线上的
    //选择时基单元钟源为内部时钟。在tim.h头文件
    TIM_InternalClockConfig(TIM2);
    //配置时基单元
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;//初始化时基单元要定义的结构体
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;//指定时钟分频、1分频
    //这里的分频是为了给滤波器采样频率f  采样点数N f越低、N越多。采样点数越好。
    //但信号延迟越大。f可以有内部时钟直接而来，也可以由分频器分频的到。
    //这里的分频器就是对送往滤波器的f的分频器进行控制。在这里直接用1就好，和时基单元关系不大
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;//选择计数模式，分别是向上、向下、三种中央对其模式
    TIM_TimeBaseStructure.TIM_Period = 10000 - 1;//ARR重装值  =  值 +1   所以我们再用的时候要-1.才能正确匹配。这里是ARR和PSC定时1s
    TIM_TimeBaseStructure.TIM_Prescaler = 7200 - 1;//PSC分频器分频系数= 值+1
    TIM_TimeBaseStructure.TIM_RepetitionCounter = 0;//重复计数器，这个是高级计数器才有的，我们不需要。所以给0
    TIM_TimeBaseInit(TIM2,&TIM_TimeBaseStructure);//这个函数是初始化时基单元的。
    //他会把我们定义的ARR重装值和PSC分频器值都放到对应的寄存器中
    //但是要知道。计时器和ARR重装寄存器。他们都有一个 影子寄存器  因为影子寄存器的存在
    //我们写入的值不会立刻生效，而是等待此次周期结束，也就是在更新事件的时候才会生效。
    //所以这个函数在最后的时候，手动生成了一下更新事件。
    //但副作用是，更新事件和更新中断是同时发生作用的。所以在我们真正使用他之前，他已经申请了一个中断（所以在上电复位之后计数器已经是1了）
    //所以在这里我们需要手动的先清除一下此次的更新中断标志位（不然他会一直是1，直到在完成启动之后被我们的中断函数捕捉到）
    //手动清除中断标志位
    TIM_ClearFlag(TIM2,TIM_IT_Update);          //清除更新标志位
    //配置输出中断控制
    TIM_ITConfig(TIM2,TIM_IT_Update,ENABLE);//使能TIM2的更新中断到NVIC
    //配置NVIC
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//选择分组
    NVIC_InitTypeDef NVIC_InitStructuer;//定义结构体
    NVIC_InitStructuer.NVIC_IRQChannel = TIM2_IRQn;//选择定时器2在NVIC里的通道
    NVIC_InitStructuer.NVIC_IRQChannelCmd = ENABLE;//是否使能
    NVIC_InitStructuer.NVIC_IRQChannelPreemptionPriority = 2;//抢占优先级
    NVIC_InitStructuer.NVIC_IRQChannelSubPriority = 2;//响应优先级。
    NVIC_Init(&NVIC_InitStructuer);
    //运行控制，使能计数器
    TIM_Cmd(TIM2,ENABLE);
}




//配置中断函数
void TIM2_IRQHandler(void)
{
    if(TIM_GetITStatus(TIM2,TIM_IT_Update) == SET)//检查中断标志位
    {
        Num++;
        TIM_ClearITPendingBit(TIM2,TIM_IT_Update);
    }
}



		```
	## 编写：定时器外部时钟计数 {toggle="true"}
		抄一遍上边的。
		只需要改一下输入的时钟，（以及对定时器的配置。因为我是手动输入..得把他改小点）
		选择的时钟源改为了外部时钟模式2
		```c
    TIM_ETRClockMode2Config(TIM2,TIM_ExtTRGPSC_OFF,TIM_ExtTRGPolarity_NonInverted,0x00);
    //配置时基单元的输入信号为外部时钟。使用的是外部时钟模式2配置↑
    //↑第一个参数是TIm2，第二个参数是选择分频、第三个参数是上升沿还是下降沿有效、第四个参数是配置滤波器的F和N 具体对应在手册中有
		```
		其他的没啥了。不细说了
	## TIM输出比较是什么 {toggle="true"}
		### OC（输出比较）与PWM介绍 {toggle="true"}
			- OC(Output Compare)输出比较
			- 输出比较可以通过比较CNT与CCR寄存器值的关系，来对输出电平进行置1、置0或翻转的操作，用于输出一定频率和占空比的PWM波形
			- 每个**高级定时器和通用定时器都拥有4个输出比较通道**
			- **高级定时器的前3个通道额外拥有死区生成和互补输出的功能（用于驱动三相无刷电机）**
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ca0b6259-e3cf-4526-9cb0-8ac4363421f3.webp)
			看下半部分。其中的捕获/比较寄存器是输入和输出共用的。
			当输入时，他就是输入捕获寄存器
			当输出是，他就是输出比较寄存器
			在输出比较时。  这个电路会比较CNT和我们设置的CCR的值。对应的输出1 或 0
			他非常适合用来输出PWM波
			PWM不细讲。是调节占空比的。
			脉冲宽度调制PWM(Pulse Width Modulation)
			**在具有惯性的系统中，**可以通过对一系列脉冲的宽度进行调制，来等效地获得所需要的模拟参量，常应用于电机控速等领域
			**PWM参数:**
			频率       分辨率          分辨率（占空比变化的细腻程度）
			<empty-block/>
		### 输出比较框图介绍 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/0ab7d90b-6e80-4d65-b3cf-b9342f0611bb.webp)
			左边是CNT和CCR的输入。比较之后根据输出模式输出信号到右边，
			右边有一个控制电路，可以把这个信号取反或者原信号不变。
			再往右就是使能，然后输出到OC1  
			OC1其实就是 在引脚表上的 复用了
		### 输出比较模式介绍 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/14141568-d3be-4d01-b43a-265b2ee261ed.webp)
			输出比较模式有这几种
			冻结意思就是 当前状态维持不变
			其他俩的有效 其实可以理解为高电平。 或低电平
			**最后俩是最重要的。**
		### 输出比较 根据计数器输出 **PWM**的基本结构（通用定时器） {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/52cab000-54a6-428c-a54f-a0f2b6eb6ef8.webp)
			当我们配置好时基单元之后，CNT就开始了**自增**运行(自己设置，一般都是自增)
			这里的输出比较单元 模式是 PWM1模式，  **CNT\<CCR时，为有效电平。**≥则输出无效电平。**输出的信号用REF表示**
			可以看右上角的图， **CNT\<CCR时，为有效电平。**≥则输出无效电平。并且占空比收到CCR的控制
			再经过极性选择。就输出到GPIO了
			可以看图。  
			**PWM的频率  就是  计数器的频率**
			**占空比的大小 是  CCR /  ARR+1  **（CNT和CCR在相等的瞬间就条变为低电平了）
			分辨率大小  是 1/(ARR+1)     也就是自动重装值+1 这个值是越小越好的。意思就是PWM从0到100中间的值就更多。
		### 高级定时器的比较通道大概介绍 {toggle="true"}
			可以看到，大概是一样的。不过多了一个死区生成电路和另一对输出使能电路
			OC1和OC1N是两个互补的端口，为了方便理解，可以假设为他们现在接了一个这样的电路（红线），这是一个基础的控制mos管的电路。
			如果上下都导通，是不允许的，因为电源直接接地
			如果上导通下不导通，则输出高电平
			如果下导通上不导通，则输出低电平
			如果上下都不导通，则为高阻态。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1b18ff84-40fe-4840-91ba-2a33443b4eb3.webp)
			我们可以这么理解。在驱动Mos管的时候。
			上管关断，下管立刻就打开的时候，由于器件的不理想。  可能上管没完全关断。下管就打开了。出现了短暂的上下管导通的现象。这就会导致功率损耗，并且引起发热。
			为了避免这个问题，就引入了死区生成电路。**他会**在上管关闭 的时**候延迟一小段时间**再打开下管。
	## tim.h库定时器OC比较电路常用的函数介绍 {toggle="true"}
		> `void TIM_SetCompare1(TIM_TypeDef* TIMx, uint16_t Compare1);<br>void TIM_SetCompare2(TIM_TypeDef* TIMx, uint16_t Compare2);<br>void TIM_SetCompare3(TIM_TypeDef* TIMx, uint16_t Compare3);<br>void TIM_SetCompare4(TIM_TypeDef* TIMx, uint16_t Compare4);`
		> 单独修改CCR寄存器值的函数
		<empty-block/>
		> `void TIM_OC1Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);<br>void TIM_OC2Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);<br>void TIM_OC3Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);<br>void TIM_OC4Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);`
		> 这些就是了。因为输出比较单元有四个，所以**一个OC就是配置一个比较单元啦。**
		<empty-block/>
		> `void TIM_OCStructInit(TIM_OCInitTypeDef* TIM_OCInitStruct);` 
		> 这个是用来给输出比较结构体赋一个默认值的
		<empty-block/>
		一些小功能和运行时更改参数的函数
		`void TIM_ForcedOC1Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);<br>void TIM_ForcedOC2Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);<br>void TIM_ForcedOC3Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);<br>void TIM_ForcedOC4Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);`
		上面这些是强制让某个输出比较单元输出高或低电平。<br>（仅了解就好，因为他们和对0%占空比100%占空比一样的）
		<empty-block/>
		`void TIM_OC1PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);<br>void TIM_OC2PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);<br>void TIM_OC3PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);<br>void TIM_OC4PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);`
		这四个寄存器是用来配置预装功能的（之前的影子寄存器）<br>让你新写入的CCR不会立刻生效，而是在时间更新后生效（一般不用。知道就好）
		<empty-block/>
		<empty-block/>
		`void TIM_OC1FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);<br>void TIM_OC2FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);<br>void TIM_OC3FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);<br>void TIM_OC4FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);`
		配置快速使能的，用得不多，不用掌握
		<empty-block/>
		`void TIM_ClearOC1Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);<br>void TIM_ClearOC2Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);<br>void TIM_ClearOC3Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);<br>void TIM_ClearOC4Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);`
		清除外部事件的REF信号的。不用掌握
		<empty-block/>
		`void TIM_OC1PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);<br>void TIM_OC1NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);<br>void TIM_OC2PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);<br>void TIM_OC2NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);<br>void TIM_OC3PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);<br>void TIM_OC3NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);<br>void TIM_OC4PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);`
		这些是单独设置输出比较的极性的。其中带N的是高级定时器的互补通道的配置（因为OC4没有互补通道，所有没有OC4N）
		<empty-block/>
		<empty-block/>
		`void TIM_CCxCmd(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_CCx);<br>void TIM_CCxNCmd(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_CCxN);`
		单独修改输出使能参数
		<empty-block/>
		`void TIM_SelectOCxM(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_OCMode);`
		单独更改输出比较模式的函数
		<empty-block/>
		<empty-block/>
	## 小知识：GPIO与GPIO端口映射 {toggle="true"}
		在引脚图上，部分的GPIO是可以映射到别的GPIO口的。不过是固定的。不可以随意的去配置（在AFIO映射部分的手册上有）。使用前要先打开AFIO的时钟。然后使用  `  GPIO_PinRemapConfig（xxxx,ENABLE)` 去重映射。
		不过要注意别把固定复用的JTAG SWD给清除掉。不然就只能用串口去下载程序恢复了。
	## 编写：输出PWM 驱动LED呼吸灯 {toggle="true"}
		结果：
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4eb8d432-d4ac-455d-956e-c8c4f6497bfb.gif)
		配置定时器输出PWM波。
		1. 使能输出PWM的PA0端口，配置PA0为复用推挽输出模式<br>因为在代码中，我们用的是TIM2的OC1输出比较器。在框图中：TIMx的OC1的输出，是固定输出在PA0引脚上。所以需要对PA0引脚复用为输出。
		2. 使能TIM2的时钟，TIM2是挂载在APB1的通道上。
		3. 选择输入时钟为内部时钟。
		4. 配置时基单元<br>在配置时基单元时，我们要根据预期输出的PWM的频率、占空比以及分辨率去设置重装寄存器ARR的值和与分频器PSC的值公式和举例如下：
			- 目的：产生频率为1KHZ  占空比50%  分辨率为1%
			- 公式
				- 分辨率  =  1 / (重装寄存器ARR+1)             
				- 占空比  = 比较控制值CCR / (重装寄存器ARR+1）         
				- 频率    =  时钟频率CK_PSC(72MHZ) / (预分频值PSC+1)（重装寄存器ARR+1)
			- 结果
				- ARR =  100 - 1   自动重装载寄存器
				- CCR = 50           输出比较寄存器
				- PSC = 720          预分频寄存器
		5. 配置OC输出比较电路
		6. 使能TIM2<br>OC和时基单元啥的都在TIm2里。所以最后打开
		7. 使用`TIM_SetCompare1` 更改输出比较寄存器的CCR值。来更改PWM波<br>在这里如果要更改PWM只需要修改CCR就OK。这是因为我们的ARR自动重装载寄存器的值为100-1  所以 输入的CCR就相当于百分比了。实际情况需要考虑一下CCR与ARR的值
		8. 最后是在main.c函数中使用for 让CCR的值从 0  -  100  100 - 0循环。实现PWM驱动LED的呼吸灯效果
		> 程序如下：<br>PWM.c
		```c
#include "stm32f10x.h"                  // Device header

void PWM_Init(void)
{   
    //使能PWM的输出引脚PA0  在引脚复用图可查到
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStructuer;
    GPIO_InitStructuer.GPIO_Mode = GPIO_Mode_AF_PP;//选择复用推挽输出，把GPIO引脚的控制权交给偏上外设
    GPIO_InitStructuer.GPIO_Pin = GPIO_Pin_0;
    GPIO_InitStructuer.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructuer);
    
    //使能TIM2时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
    
    //选择时钟源为内部时钟
    TIM_InternalClockConfig(TIM2);
    
    //配置时基单元
    TIM_TimeBaseInitTypeDef TIM_TImeBaseInitStructure;
    TIM_TImeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;     //指定时钟分频
    TIM_TImeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up; //选择计数模式
    TIM_TImeBaseInitStructure.TIM_Period = 100 - 1; //配置自动重装寄存器ARR
    TIM_TImeBaseInitStructure.TIM_Prescaler = 720 - 1; //配置到计数器信号的分频系数PSC
    TIM_TImeBaseInitStructure.TIM_RepetitionCounter = 0; //重复计数，高级定时器才用到
    TIM_TimeBaseInit(TIM2,&TIM_TImeBaseInitStructure);
    
    /*
        产生频率为1KHZ  占空比50%  分辨率为1%
        分辨率  =  1 / (重装寄存器ARR+1)               ARR =  100 - 1 
        占空比  = 比较控制值CCR / (重装寄存器ARR+1）   CCR = 50
        频率    =  时钟频率CK_PSC(72MHZ) / (预分频值PSC+1)（重装寄存器ARR+1）
           PSC = 720
    
    
    */
    
    //使能OC比较通道 1
    TIM_OCInitTypeDef TIM_OCInitStructure;  //不用引出高级定时器的成员。
    TIM_OCStructInit(&TIM_OCInitStructure); //但是要先把每个成员一个初始值，避免不必要的问题
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;               //设置输出比较模式 - PWM1模式
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;       //是否反转 -  否
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;   //输出使能 - 使能
    TIM_OCInitStructure.TIM_Pulse = 0;       //配置CCR寄存器的值。16位
    
    TIM_OC1Init(TIM2,&TIM_OCInitStructure);
    
    
    //使能定时器
    TIM_Cmd(TIM2,ENABLE);
    
    
}

void PWM_SetCompare1(uint16_t Comparel)
{
    TIM_SetCompare1(TIM2,Comparel);
}

		```
		> main.c
		```c
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "pwm.h"
int main()
{
    OLED_Init();//初始化OLED;
    Delay_Init();//初始化延时
    PWM_Init();//初始化PWM

    
   
    uint16_t i = 0;
    
    while(1)
    {
        for(i = 0; i <= 100; i++)
        {
            PWM_SetCompare1(i);
            Delay_ms(10);//延时，否则占空比变的太快10*100也就是1秒变亮
        }
        for(i = 0; i <= 100; i++)
        {
            PWM_SetCompare1(100 - i);//从100慢慢下降
            Delay_ms(10);
        }
        
        
    }
}



		```
		<empty-block/>
	## 编写：输出PWM 控制舵机 {toggle="true"}
		结果：
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/da1dc0f4-37eb-45e3-96d0-c748a35490a4.gif)
		舵机是由PWM控制的。他会根据固定周期的占空比来调整角度
		我这里的舵机是-90 \~ +90°的
		要求如下：
		- 周期20ms，<br>高电平宽度要求0.5ms\~2.5ms<br>←-  -90°    0.5ms<br>↖   -45°    1ms<br>↑   0°            1.5ms<br>↗   45°       2ms<br>→   90°        2.5ms
		PWM产生同上边的PWM控制LED一样。唯一不同的是需要是把频率设置为50HZ的要求。 以及能产生0.5\~2.5ms的高电平。 
		我建议先把PSC设置为一个比较好算的值。这里是72 -1
		然后ARR可以设置的大一点。
		到时候根据ARR和预期的占空比。 推算出ARR和RCC的比例就OK
		我这里 ARR = 20000 -1    PSC = 72 -1    RCC = 500时 高电平为0.5ms  RCC =  1000时为1ms
		整体代码如下：
		```c
#include "stm32f10x.h"                  // Device header


/*
    ARR = 20000
    占空比0.5时
    RCC = 20000 * 0.5/20 = 500
*/

void PWM_Init(void)
{   
    //使能PWM的输出引脚PA1
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    
    GPIO_InitTypeDef GPIO_InitStructuer;
    GPIO_InitStructuer.GPIO_Mode = GPIO_Mode_AF_PP;//选择复用推挽输出，把GPIO引脚的控制权交给偏上外设
    GPIO_InitStructuer.GPIO_Pin = GPIO_Pin_1;//OC通道输出
    GPIO_InitStructuer.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructuer);
    
    //使能TIM2时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
    
    //选择时钟源为内部时钟
    TIM_InternalClockConfig(TIM2);
    
    //配置时基单元
    TIM_TimeBaseInitTypeDef TIM_TImeBaseInitStructure;
    TIM_TImeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;     //指定时钟分频
    TIM_TImeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up; //选择计数模式
    TIM_TImeBaseInitStructure.TIM_Period = 20000 - 1; //配置自动重装寄存器ARR
    TIM_TImeBaseInitStructure.TIM_Prescaler = 72 - 1; //配置到计数器信号的分频系数PSC
    TIM_TImeBaseInitStructure.TIM_RepetitionCounter = 0; //重复计数，高级定时器才用到
    TIM_TimeBaseInit(TIM2,&TIM_TImeBaseInitStructure);
    
    /*
        产生频率为50HZ  占空比0%  分辨率为0.05
        分辨率  =  1 / (重装寄存器ARR+1)               
        占空比  = 比较控制值CCR / (重装寄存器ARR+1）
        频率    =  时钟频率CK_PSC(72MHZ) / (预分频值PSC+1)（重装寄存器ARR+1）
        0.05 = 1 / ARR + 1  ----    ARR = 20000 - 1
        50 = 72M / （psc+1）（20000）      
        PSC+1 = 72   ---   PSC = 72 -1
    
    
    */
    
    //使能OC比较通道 1
    TIM_OCInitTypeDef TIM_OCInitStructure;  //不用引出高级定时器的成员。
    TIM_OCStructInit(&TIM_OCInitStructure); //但是要先把每个成员一个初始值，避免不必要的问题
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;               //设置输出比较模式 - PWM1模式
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;       //是否反转 -  否
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;   //输出使能 - 使能
    TIM_OCInitStructure.TIM_Pulse = 0;       //配置CCR寄存器的值。16位
    
    TIM_OC2Init(TIM2,&TIM_OCInitStructure);//配置在OC2通道。输出在PA1引脚上
    
    
    //使能定时器
    TIM_Cmd(TIM2,ENABLE);
    
    
}

void PWM_SetCompare2(uint16_t Compare2)//更改为通道2
{
    TIM_SetCompare2(TIM2,Compare2);
}



		```
		舵机控制的main.c代码也很简单
		只是判断然后++
		```c

/*
    PA1引脚输出PWM波控制舵机
    按键调整PWM占空比
    显示在OLED屏幕上
    舵机要求输入PWM要求：周期20ms，高电平宽度要求0.5ms~2.5ms
    ←-  -90°    0.5ms
    ↖   -45°    1ms
    ↑   0°      1.5ms
    ↗   45°     2ms
    →   90°     2.5ms
*/
int main()
{
    
    OLED_Init();//初始化OLED;
    Delay_Init();//初始化延时
    PWM_Init();//初始化PWM
    KEY_Init();//初始化KEY
    
    
    
    OLED_ShowString(1,1,"CCR:");
    OLED_ShowString(2,1,"ARR:20000");
    
    OLED_ShowNum(1,5,0,4);
    uint16_t Num = 0;
    while(1)
    {
        if(!KEY_Get())//低电平有效
        {
            Delay_ms(10);
            if(!KEY_Get())//确定
            {
                while(!KEY_Get());//等待松开
                Num += 500;
                if(Num > 2500)
                {
                    Num = 500;
                }
                OLED_ShowNum(1,5,Num,4);
                PWM_SetCompare2(Num);
            }
        
        }  
    }
}


		```
	## TIM输入捕获是什么 {toggle="true"}
		### 输入捕获 IC 介绍 {toggle="true"}
			- **IC**(Input Capture)输入捕获
			- 输入捕获模式下，**当通道输入引脚出现指定电平跳变时**，**当前CNT的值将被锁存到CCR中**，**可用于测量PWM波形的频率、占空比、脉冲间隔、电平持续时间等参数**
			- **每个高级定时器和通用定时器都拥有4个输入捕获通道**
			- 可配置为**PWMI**模式，同时测量须率和占空比<br>这是专门为了测量PWM的占空比和频率设计的
			- 可配合主从触发模式，实现硬件全自动测量
			看图。右边的部分是输出比较。我们已经介绍过了。
			左边部分则是输入捕获部分，正如介绍所说的。左边的引脚CH1 2  3  4 是输入引脚
			在输入捕获的模式下。当输入滤波和边沿检测电路检测到电平发生跳变的时候。输入补货寄存器就会把当前CNT的值所存到自己的CCR寄存器中。
			对比输出捕获的话。这里的引脚是输入。这里的寄存器是存CNT得值到CCR 而不是 对比CNT和CCR
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5afbea4d-6479-4113-9bb8-fcfc913e52a2.webp)
		### 输入捕获模式框图介绍 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/09bb3f31-9a63-444f-94cb-f9a22784e763.webp)
			1. 输入捕获的前三个输入口有一个异或门。<br>异或门的工作逻辑是  三个引脚的电平有任何一个有电平翻转就立刻把他的输出电平反转<br>这里的还是为了三相无刷电机而设计的。无刷电机有三个霍尔传感器可以检测电机的位置。根据电机位置来换相。<br>输出的电平会经过一个数据选择器。数据选择器如果选择上边的。通道1就会检测这三个通道的电平变化了。如果选择下边的。那么通道就各用各的。
			2. 边缘检测 和滤波电路则是滤除毛刺然后检测电平。这里的检测和之前的一样，也是可以检测高电平、低电平、双边沿 。然后触发后续电路执行动作<br>在边沿检测电路。设计了两套的滤波和边沿检测电路.第一套电路经过滤波和选择输出TI1FP1(Tl1 Filter Polarity 1)给通道1的后续电路、第二套电路是输出TI2FP2给通道2的后续电路。、<br>通道二的也是如此。TI2FP1输入给通道1、TI2FP2输入给通道2.<br>这样做是为了可以灵活切换捕获电路的输入。并且可以把一个通道的输入同时映射到两个捕获单元（是PWMI的经典结构）。<br>除了TIxFP1 和TIxFP2 ，还有一个输入是TRC。这个就是上次讲的输入信号部分的内容。是其他定时器映射发来的信号。<br>经过一个信号选择器之后进入预分频器 ，预分频器分频后的值就可以进入捕获通道了。每来一个信号，CNT的值就被CCR转运一次。转运同时会发生一次捕获事件。这个时间会在状态寄存器置标志位，同时也会可以中断。
		### 输入捕获模式从模式介绍 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/d86c19d7-1cef-4df6-87d7-847f0c6ffd0a.webp)
			这个是对上边捕获框图的再次细分，
			可以看到，信号是先进入滤波器。滤波器可以由ICF的四位寄存器控制采样频率F和采样点数N。<br>具体需要看手册。F越低，N越高。滤波效果越好。
			然后进入边沿检测电路。他有两个输出，一个在上升沿输出1，另一个在下降沿时输出1，他们可以由一个CCP1来控制到底哪一个通道能通网后续的选择器。<br>可以看到，下边也有一个“来自通道2”的输出，也是经过CCIP寄存器选择极性之后输出到后续的选择器。 
			TRC是来自其他的时钟（的从模式）所发来的信号。
			然后经过选择一个信号之后，进入分频器。CCIS的两位寄存器就是选择信号用的。ICPS是用来配置分频器的。他可以配置不分频、2分频、4分频、8分频。再往后的CC1E则是分频器的使能位。
			> 还有两个没说。是在边沿检测器的输出部分，两根线连接了一个或门。那么他是边沿输出，上升沿和下降沿都输出1 也就是边沿信号。信号为TI1FP_ED  他通往的就是从模式控制器<br>在经过选择器选择后的输出信号TI1FP1也是通往从模式控制器的<br>从模式控制器里就可以对CNT进行清零。 不需要软件来操作了。这都是硬件一次执行的。<br>所以可以看出，从模式就是完成自动化操作的利器 啊！
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/0870ee01-ff57-4520-bef3-6abe3c8df228.webp)
			主模式可以把定时器内部的信号映射到TRGO引脚，用于触发别的外设
			从模式可以把其他外设发来的信号，或者是自身外设的一些信号，用于控制自身定时器的运行，也就是被别的信号控制。
			触发源选择就是选择用那个源来连接到TRGI去 触发 从模式  从模式 则可以在右边的列表中，选择一项来自动执行
			比如我们要清除CNT计数，就可以选择TI1FP1上升沿连接到TRGI 从模式选择Reset、  
			这些意义都可以在控制寄存器里手册查看
		### 测周法测量频率思路 {toggle="true"}
			有了上面的框图介绍。 大概就可以设计出测周法测频率的方法了。
			每来一个上升沿用于触发输入捕获，CNT来计数得到N（得到N后要清零，可以使用主从触发模式自动完成），fc则是时钟频率。
			fc/N就是待测信号的频率
		### 输入捕获测频率的框图 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/de473443-1b5b-4caf-9ad6-d67c9e51be3e.webp)
			GPIO的信号经过滤波器滤波、
			边沿检测检测信号之后输出到分频器分频。
			然后分频器触发信号给捕获器。
			捕获器读取CNT的值到CCR中。
			另一路，在把UT1FP1的信号映射到从模式触发器上。从模式控制时基单元的CNT清除操作。
		### 输入捕获测频率和占空比（PWMI）的框图 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/bed2e57e-7714-4685-acc5-29a55bae4ae7.webp)
			和上边一样，不过多了一路
			这里可以自己去选择。 这里只是用到了通道2 的分频器部分。 因为每个边沿检测和极性选择都有两个输出。 TxFP1和TxFP2。
			下边一路选择为下降沿触发，这样 下降沿触发所记录的值 比上 下一个上升沿所记录的值就是占空比了！
			<empty-block/>
			在TIMx功能描述有 手册里
		### 测频和测周法的应用场景 {toggle="true"}
			1. 测频法：在T秒内对上升沿次数计次得到**N**。**频率就是N/T  **（T为1则直接就是频率）<br>测周法是计算的闸门时间内的平均频率、T秒跳变一次.<br>在测周法的时候，可能最后或者第一个信号才过了一半就到了时间，所以测周法有一个**+-1误差**（取决于我们算他为有效还是无效）<br>**测频法适合测量高频率的信号。这样精度更好**
			2. 测周法：测出一个周期的时间，然后取倒数<br>实际上是用一个比已知频率还要高的频率fc。在已知频率的一个周期内，计算fc的高电平的**次数N**。记一个数的时间是1/fc  那么N个数的时间就是N/fc   那么取到数就是测量的信号的的频率啦**fc/N**<br>测频法跳变比较快、已知频率来一个上升沿他就跳一次。波动比较大<br>测周法也是一样，有可能计次记到一半，来上升沿了。**也会出现- +1个误差。**（取决于我们算他为有效还是无效）<br>**测周法适合测量低频率的信号，精度更好**
				> 那多高算高频率。多低算低频呢？<br>这就引出了一个新的概念：**中界频率<br>**中介频率是测频法和测周法误差相等的频率点
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/64d93f8e-cb44-40ea-acf9-105bbbb88f6c.webp)
				可以看到这个图。分别是测频法和测周法。
				他们都有计次N。并且计次N的值是越多越好。也就是越准确。
				我们把测周法和测频法的公式列出来
				测频法：fx = N/T
				测周法：fx = fc/N
				当N = N时，解出来
				所以可以得到下面的公式
				fm = ( fc/T ) \^ (1/2)   （fc/T后开方）
				所以，当测量频率大于fm 用测频法。 
				当测量频率小于fm  用测周法
		### 测周法测频率和占空比的思路 {toggle="true"}
			> 测周法测频率 是 
			由我们已知的频率**fc **来测量待测信号的两个上升沿之间 计数的值。
			在每个上升沿，由输入捕获记录CNT的计数值。<br>由从模式自动清除CNT的计数值** N（待测信号一个周期内，已知信号的上升沿个数）**。
			然后使用公式fc / N 得到频率。单位为秒
			> 测周法测频率和占空比
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/09bb3f31-9a63-444f-94cb-f9a22784e763.webp)
			刚才使用测频法，只使用了一个IC通道。但若要测量占空比，则需要使用两个IC通道。
			由输入捕获框图可以了解到。IC1和IC2 的通道，实际上是**有两个输入滤波和边沿检测器**。他们俩一路输出TI1FP1，一路输出TI1FP2.  一个是直接连接到后续电路。另一个则是交叉连接到了IC2通道。（1 和 2 通道交叉连接  3 和4 通道交叉连接）。
			我们可以通过配置Ic2的通道，使得一个TI1FP2的滤波极性检测器输出与TI1FP1完全相反的检测（1检测上升沿，那么2检测下降沿），在这里也就是让2检测下降沿时CNT所记的数，存到CCR2。  此时 通道2的 CCR2 与通道1 的  CCR1 所记录的是 高电平的时间与整个周期的时间， 所以他们的比值就是占空比。（可以\*100更直观） 
			<empty-block/>
	## tim.h库定时器IC比较电路常用的函数介绍 {toggle="true"}
		1. `void TIM_ICInit(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ICInitStruct);`<br>用结构体配置输入捕获单元的函数<br>因为有交叉通道，所以不想OC一样每个都单独拎出来。
		2. `void TIM_PWMIConfig(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ICInitStruct);`<br>和上个函数类似，但是快速配置两通道的。配置成PWMI模式
		3. `void TIM_ICStructInit(TIM_ICInitTypeDef* TIM_ICInitStruct);` <br>给输入捕获结构体赋一个初始值
		4. `void TIM_SelectInputTrigger(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource);`<br>选择**从模式的**触发源
		5. `void TIM_SelectOutputTrigger(TIM_TypeDef* TIMx, uint16_t TIM_TRGOSource);` <br>选择**主模式**输出**的**触发源。
		6. `TIM_SelectOnePulseMode` <br>选择从模式（要从模式干啥）
		7. `void TIM_SetIC1Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);<br>void TIM_SetIC2Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);<br>void TIM_SetIC3Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);<br>void TIM_SetIC4Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);`<br>配置通道1 、 2、 3、 4、的分频器
		8. `uint16_t TIM_GetCapture1(TIM_TypeDef* TIMx);<br>uint16_t TIM_GetCapture2(TIM_TypeDef* TIMx);<br>uint16_t TIM_GetCapture3(TIM_TypeDef* TIMx);<br>uint16_t TIM_GetCapture4(TIM_TypeDef* TIMx);<br>uint16_t TIM_GetCounter(TIM_TypeDef* TIMx);`<br>**读取**通道1、 2、 3、 4、的CCR
	## 编写：测周法测量频率和占空比 {toggle="true"}
		### 写一个PA0输出可调占空比和频率的的PWM {toggle="true"}
			PA0是TIM2的复用输出引脚。<br>首先要对他进行初始化，
			初始化的步骤是<br>1,、使能GPIOA的时钟， 配置PA0的引脚为复用推挽输出
			```c
//使能PA0输出引脚
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);
			```
			1. 使能定时器外设，并选择输入时钟为内部时钟，然后配置时基单元<br>这里的72 - 1 是随便设置的。他到时候是要在主函数进行修改的。
			```c
    //使能时钟外设
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
    
    //选择TIM2为内部时钟，若不调用此函数，TIM默认也为内部时钟
    TIM_InternalClockConfig(TIM2);
    
    //配置时基单元
    TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;\
    TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;//指定滤波器采样频率F和采样点数N为 模式1
    TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;//向上计数
    TIM_TimeBaseInitStructure.TIM_Period = 100 - 1;//ARR  精度为1 。这里定义为100 - 1
    TIM_TimeBaseInitStructure.TIM_Prescaler = 72 - 1;//PSC 
    TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;//高级定时器的复用定时器
    TIM_TimeBaseInit(TIM2,&TIM_TimeBaseInitStructure);
    
    //清除时基单元配置好后产生的标志位
    TIM_ClearFlag(TIM2,TIM_IT_Update);  
			```
			1. 使用OC比较输出 输出可调的PWM波
			```c
    //配置OC输出比较
    TIM_OCInitTypeDef TIM_OCInitStructure;//创建结构体
    TIM_OCStructInit(&TIM_OCInitStructure);//初始化结构体,为了避免没赋值造成麻烦
    
    TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;//选择输出模式为PWM1
    TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;//是否翻转电平（高电平有效还是低电平有效）
    TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;//是否使能输出
    TIM_OCInitStructure.TIM_Pulse = 0;//CCR寄存器
    TIM_OC1Init(TIM2,&TIM_OCInitStructure);
    
			```
			1. 使能定时器`TIM_Cmd(TIM2,ENABLE);`
			2. 封装修改占空比和频率的函数<br>需要注意的是，修改频率有两种方式，一种是修改ARR重装寄存器的值。 另一种是修改PSC。 这里选择修改PSC。因为ARR不变可以更好的配置占空比。<br>使用的函数是：`TIM_SetCompare1`和`TIM_PrescalerConfig` 修改PSC。为什么这里的修改预分频器不是set_Prescaler  这是因为 在修改psc的时候需要配置参数。不太方便，所以就一并参进去了。`这里的第三个参数是 选择立刻生效还是在更新后生效（影子寄存器预装载）`
			```c

//封装更改CCR寄存器的函数
void PWM_SetCompare1(uint16_t Compare1)
{
    TIM_SetCompare1(TIM2,Compare1);
}


//封装更改PSC寄存器的函数
void PWM_SetPrescaler(uint16_t Prescaler)
{
    TIM_PrescalerConfig(TIM2,Prescaler,TIM_PSCReloadMode_Immediate);
    //这里的第三个参数是 选择立刻生效还是在更新后生效（影子寄存器预装载）
}
			```
			到这里，PA0就可以输出可调的PAM波了。
			<empty-block/>
		### 写一个PA6输入PWM波然后检测占空比和频率 {toggle="true"}
			为什么是PA6 因为我们在这里用的是TIM3  也就是第二个通用定时器去测量。
			而TIM3的 的输入是复用在PA6的引脚上的。
			所以我们在写程序的时候第一步也是使能GPIOA 和 配置PA6为输入  上拉。
			```c
//使能PA0输出引脚
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;//上拉输入
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;//PA6
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);
			```
			配置好GPIO之后就要开始配置时钟了。
			首先要使能TIM3的外设。以及配置TIM3的输入信号
			```c
    //使能时钟外设
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3,ENABLE);
    
    //选择TIM2为内部时钟，若不调用此函数，TIM默认也为内部时钟
    TIM_InternalClockConfig(TIM3);
			```
			下一步就是配置时基单元了。<br>时基单元的配置决定了我们的fc的信号.<br>**这里要注意的是<br>**我们设置的**ARR为65536**.因为CNT的最大计数值决定了在计数的时候所能识别的最低频率。过小的话计数器就溢出了。导致频率不准。<br>PSC 是72 。也就是fc 的频率为72M/72 = 1MHZ。也就是说我们测量的**最低频率为1M/65536 大约等于15HZ。如果频率低于15HZ，计数器就会发生溢出<br>**如果想要**下限**再低一些。那么可以使PSC再加**大**一点。这样标准频率1M就变小了。所支持的频率就也变小了。<br>下一步就是测量频率的**上限**了。这个最大频率，并没有一个明显的界限。 因为随着待测频率的增大， 测量频率的误差也会增大，~~如果非要找一个频率上线，那应该就是标准频率的1MHZ，如果超过标准频率了。信号频率比标准频率还高  那么肯定是测不了的。但是这个上限是没有意义的，因为当信号到1MHZ的时候**误差**已经非常大了。<br>~~所以对于最大误差的要求就是我们对 误差的要求：<br>也就是说，如果你要求误差为千分之一，那么频率上限就是1M/1000 = 1000HZ<br>如果你要求误差为百分之一，那么频率上限就是1M/100 = 10KHZ
			想要提高频率的上限就要把PSC给**降低**一些。提高了标准频率 上限就会提高。<br>（要是频率还想高  就考虑测频法把。 测周法适合低频）
			```c
    
    //配置时基单元
    TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;\
    TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;//指定滤波器采样频率F和采样点数N为 模式1
    TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;//向上计数
    TIM_TimeBaseInitStructure.TIM_Period = 65536 - 1;//决定了CNT计数的最大值，测频率最好要设置大一些，防止计数溢出
    TIM_TimeBaseInitStructure.TIM_Prescaler = 72 - 1;//决定了测周法的标准频率fc，暂时先定义为1MHZ
    TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;//高级定时器的复用定时器
    TIM_TimeBaseInit(TIM3,&TIM_TimeBaseInitStructure);
    
    //清除时基单元配置好后产生的标志位
    TIM_ClearFlag(TIM3,TIM_IT_Update);
			```
			下一步就是配置输入捕获单元IC了。（不是OC了！）<br>他也是一个结构体
			```c
    //配置输出捕获单元IC
    TIM_ICInitTypeDef TIM_ICInitStructure;
    TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;//选择通道几？
    TIM_ICInitStructure.TIM_ICFilter = 0xF;//配置滤波器参数（对应参数在参考手册有0x0~0xF）
    TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;//触发方式（上升下降或者都）--这里是上升沿读取
    TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;//触发信号分频器
    TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;//选择触发信号从哪个引脚输入（配置数据选择器）
    TIM_ICInit(TIM3,&TIM_ICInitStructure);
			```
			现在，我们只是配置了IC1 一个通道， 他是用来在每次上升沿记录当前CNT的值，还需要配置一下IC2通道。让他记录下降沿的值，我们可以复制一下上面的参数然后进行修改：
			```c

    TIM_ICInitStructure.TIM_Channel = TIM_Channel_2;//选择通道2
    TIM_ICInitStructure.TIM_ICFilter = 0xF;
    TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Falling;//下降沿有效
    TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;//触发信号分频器
    TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_IndirectTI;//交叉输入
    TIM_ICInit(TIM3,&TIM_ICInitStructure);
			```
			但是STM还是我我们提供了一个函数，是专门用来配置这个的。<br>`TIM_PWMIConfig(TIM3,&TIM_ICInitStructure);`<br>他等同于上方的这一坨<br>不过他只支持通道1 2 的配置， 通道 3  4 还是得自己配置。<br>
			```plain text
// 会进行判断然后帮你弄成相反的（可以看F2）
//（只支持通道1 和通道2 的配置，不能传入通道3 4）
TIM_PWMIConfig(TIM3,&TIM_ICInitStructure);

			```
			到这里，我们的输入捕获就完成了。
			**但是，**为了使硬件能够自动清除每次上升沿来临时的CNT计数。我们需要配置从触发器来自动完成，而不是通过软件来手动清除。我们可以这样配置。
			```plain text
//配置主从模式的TRGI的触发源为TI1FP1
TIM_SelectInputTrigger(TIM3,TIM_TS_TI1FP1);

//配置从模式为reset
TIM_SelectSlaveMode(TIM3,TIM_SlaveMode_Reset);//读取后自动清除
			```
			第一句函数实际上就是把TRGI的线路，选择为T1FP1. <br>第二句函数则是选择了从模式要执行的命令是  清除CNT
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/0870ee01-ff57-4520-bef3-6abe3c8df228.webp)
			下面，为了获取占空比和频率。我们需要封装两个函数来利用公式计算占空比和频率
			```c

//获取频率
uint32_t IC_GetFreq(void)
{
    //执行公式，fx = fc / N  fc = 1M
    return 1000000 / (TIM_GetCapture1(TIM3)+1);//读取TIM3的通道1的CCR的值
    // +1 为面向对象编程...因为1000HZ显示的是1001
    //+1 的原因是因为每次都会少记录一个。+1补成999+1 这样就能整除了
}



//获取占空比
uint32_t IC_GetButy(void)
{
    //下降沿的计数 比上 上升沿计数就是占空比
    //下降沿记得数存在通道二的CCR2中，上升沿记得数存在CCR1中。
    return (TIM_GetCapture2(TIM3) + 1 ) * 100 / (TIM_GetCapture1(TIM3) + 1);//*100是现实整数 1%~100%
    //CCR总会少一个数，所以面向对象编程

}
			```
			最后main.c里的代码为：
			```c
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
#include "pwm.h"
#include "ic.h"
/*
    PA0输出频率和占空比可调的PWM波
    PA6作为输入 来检测PA0的频率和占空比
    采用测周法
*/

int main()
{
    OLED_Init();//初始化OLED;
    PWM_Init();//初始化PWM
    IC_Init();//初始化IC
    KEY_Init();//初始化KEY
    
    //设置频率和占空比                         //(ARR+1)
    PWM_SetPrescaler(720 - 1);  //1000HZ = 72M / 100 / (PSC+1)
    PWM_SetCompare1(50);        //占空比 = CCR/ARR  = CCR/100
    
    
    OLED_ShowString(1,1,"Freq:00000HZ");
    OLED_ShowString(2,1,"Buty:00%");
    while(1)
    {
        //循环显示当前周期
        OLED_ShowNum(1,6,IC_GetFreq(),5);//显示频率
        OLED_ShowNum(2,6,IC_GetButy(),2);//显示占空比
    }
}


			```
	## TIM编码器接口 {toggle="true"}
		### 编码器介绍 {toggle="true"}
			- Encoder Interface 编码器接口
			- 编码器接口可接收增量(正交)编码器的信号，根据编码器旋转产生的正交信号脉冲，**自动控制**CNT**自增或自减**，从而指示编码器的位置、旋转方向和旋转速度
			- 每个高级定时器和通用定时器都**拥有1个编码器接口**
			- 两个输入引脚**借用了输入捕获的通道1和通道2**
			他是把编码器直接连接到编码器接口上，32会自己根据旋转方向给 CNT ++ 或--。
			如果我们每隔一段时间把CNT的值取出来，然后再把CNT清零，其实就很容易得到它的频率了。
			这其实和我们之前说过的 测频法测频率一样。，
			这样是硬件实现的计次。不会占用软件资源。不用进中断进行简单的num++
			一般简单的程序都会去弄一个硬件的电路来实现。
		### 编码器框图 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/08284b66-b5f7-4af9-ac2e-cce8a06e019e.webp)
			可以看到，编码器是借用了输入捕获的CH1 和CH2引脚。
			这时候计数时钟和计数方向都处于编码器托管的状态，收到编码器控制
		### 编码器接口基本结构 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4c0dd938-85a8-46b0-a137-e1217a50f1d3.webp)
			输入捕获的前俩接口，通过前俩通道的滤波器。极性选择 产生TI1FP1 和TI2FP2 。就到达了编码器接口。 编码器接口直接控制时基单元。
		### 正交编码器简介 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4772be2b-3d47-4314-8744-fb2a31aa2765.webp)
			这也不一定谁提前谁滞后。反正正反 某项一定是超前某项的、
			<empty-block/>
			<empty-block/>
			<empty-block/>
		### 编码器接口的工作模式 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8518578a-e046-40d9-922c-11e6fd12d02c.webp)
		### 正交编码器抗噪声原理 {toggle="true"}
			在一个变，另一个不变的情况下，计数会自己补救和挽回。
			这个是都没反相的。如果有其中一个反相, CNT的自增自减逻辑会相反
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/72f3aeed-2ace-4783-87b8-50d346abe3d9.webp)
	## 编写：计时器正交编码器使用 {toggle="true"}
		计时器的编码器函数就这一个
		`void TIM_EncoderInterfaceConfig(TIM_TypeDef* TIMx, uint16_t TIM_EncoderMode,<br>uint16_t TIM_IC1Polarity, uint16_t TIM_IC2Polarity);`
		第一个参数 TIMx   
		第二个参数 选择编码器模式 
		 第三个参数和第四个参数   是选择通道1 和通道2 的电平极性
		前面的步骤我就略过了
		代码放这里<br>需要注意的是ARR的值给的大一些。别让他溢出了。
		```c

void Encoder_Init(void)
{
    //开启GPIOA和定时器外设的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3,ENABLE);
    
    //配置PA6、PA7
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);
    
    //编码器会托管时钟，这里不需要配置输入时钟
    
    //配置时基单元 
    TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
    TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;//滤波器分频设置F N
    TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;//计数模式没用，会被编码器托管
    TIM_TimeBaseInitStructure.TIM_Period = 65536 - 1; //一般选择满量程计数，这样在从0-1就会变成65535.后续强制把65536转换为有符号数，就可以把他转换为负数。
    TIM_TimeBaseInitStructure.TIM_Prescaler = 1 - 1;//不分频
    TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
    TIM_TimeBaseInit(TIM3,&TIM_TimeBaseInitStructure);
		```
		要注意的是 这里的输入部分不需要配置了。因为编码器接口会托管
		同样的。计数器的计数模式也不需要修改。编码器同样会托管
		下一步就是配置输入捕获了。在输入捕获部分，编码器只用到了滤波和极性选择部分，<br>并且编码器的配置函数中有配置两个通道的极性。所以我们只需要配置一下滤波就好。
		其他的全权交给IC初始化函数来。`TIM_ICStructInit(&TIM_ICInitStructure);//初始化结构体`
		```c
//配置输入捕获单元的滤波器和边缘极性选择
    //其余的交给初始化函数
    TIM_ICInitTypeDef TIM_ICInitStructure;
    TIM_ICStructInit(&TIM_ICInitStructure);//初始化结构体
    TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;//通道为1
    TIM_ICInitStructure.TIM_ICFilter = 0XF;//滤波器为0xF
    TIM_ICInit(TIM3,&TIM_ICInitStructure);
    
    TIM_ICInitStructure.TIM_Channel = TIM_Channel_2;//通道为2
    TIM_ICInitStructure.TIM_ICFilter = 0XF;//滤波器为0xF
    TIM_ICInit(TIM3,&TIM_ICInitStructure);
		```
		最后配置编码器函数， 配置的时候会顺带把极性选择。然后使能TIm就OK
		```c
//配置编码器接口  选择TI12 都计数       是否反向？
    TIM_EncoderInterfaceConfig(TIM3,TIM_EncoderMode_TI12,TIM_ICPolarity_Rising,TIM_ICPolarity_Rising);
    
    TIM_Cmd(TIM3,ENABLE);
		```
		最后就是读取编码器的CNT值了。这里读取完之后要清除掉。这样就可以算出他的速度了，
		并且我们要注意，这里的返回值是有符号整形int型。不是unsigned int 。 <br>因为无符号在向左 也就是 0-1的时候是 -1  而 unsigned  的 0-1 为65535（16位下）
		所以获取的函数是这个：
		```c
//读取编码器的值
//读完后清零CNT
int16_t Encoder_Get(void)
{
    int16_t Temp = TIM_GetCounter(TIM3);
    TIM_SetCounter(TIM3,0);//清零
    return Temp;
}

		```
		在住函数中，我们可以简单的这样调用。来实现一秒计算一次值
		```c
int main(){
    OLED_Init();//初始化OLED;
    Encoder_Init();//初始化Encoder
    Delay_Init();
    
    OLED_ShowString(1,1,"Num:");
    while(1)
    {
        OLED_ShowSignedNum(1,5,Encoder_Get(),5);
        Delay_ms(1000);
    }

}
   
		```
		但是，对于简单的程序可以，而复杂的程序，使用Delay函数就显得不太好了。
		因为Delay函数执行的时候，程序会一直处在Delay的函数内，这时候我们就可以用到之前学的定时中断，让定时器定时1s进入中断。
		<empty-block/>
		这里我直接拿过来我当时的的1s定时的中断
		大概说一下，<br>首先是使能定时器2.
		然后时钟源为内部时钟
		在对时基单元配置为1秒一次中断之后
		清除一下时基单元配置时的更新事件标志位
		然后要能放过TIM的中断能到NVIC
		使用TIM_ITConfig把TIM2的Update 放过去
		下面就开始配置NVIC了。<br>选择分组..配置NVIC …
		```c
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//选择分组
    NVIC_InitTypeDef NVIC_InitStructuer;//定义结构体
    NVIC_InitStructuer.NVIC_IRQChannel = TIM2_IRQn;//选择定时器2在NVIC里的通道
    NVIC_InitStructuer.NVIC_IRQChannelCmd = ENABLE;//是否使能
    NVIC_InitStructuer.NVIC_IRQChannelPreemptionPriority = 2;//抢占优先级
    NVIC_InitStructuer.NVIC_IRQChannelSubPriority = 2;//响应优先级。
    NVIC_Init(&NVIC_InitStructuer);
		```
		然后就可以使能定时器啦\~\~
		在函数的开头，我们声明了位于主函数的一个Speed变量。并且加载了"Encoder.h"的头文件
		在中断中。执行了把当前的编码器所控制的CNT的值读出来。`Encoder_Get();`（调用这个函数会自动清除CNT）然后一秒读一次。
		```c
//配置中断函数
void TIM2_IRQHandler(void)
{
    if(TIM_GetITStatus(TIM2,TIM_IT_Update) == SET)//检查中断标志位
    {
        Speed = Encoder_Get();
        TIM_ClearITPendingBit(TIM2,TIM_IT_Update);
    }
}

		```
		这就实现了定时器中断读取并清除CNT的值。这就不会堵塞程序了！！
		代码如下：
		```c
#include "stm32f10x.h"                  // Device header
#include "Encoder.h"
extern int16_t Speed;//记录时间的变量、这里是引用了main.c里的Num来控制。

void Timer_Init(void)
{
    //开启RCC时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);//TIM2是挂载在APB1总线上的
    //选择时基单元钟源为内部时钟。在tim.h头文件
    TIM_InternalClockConfig(TIM2);
    //配置时基单元
    TIM_TimeBaseInitTypeDef TIM_TimeBaseStructure;//初始化时基单元要定义的结构体
    TIM_TimeBaseStructure.TIM_ClockDivision = TIM_CKD_DIV1;//指定时钟分频、1分频
   
    TIM_TimeBaseStructure.TIM_CounterMode = TIM_CounterMode_Up;//选择计数模式，分别是向上、向下、三种中央对其模式
    TIM_TimeBaseStructure.TIM_Period = 10000 - 1;//ARR重装值  =  值 +1   所以我们再用的时候要-1.才能正确匹配。这里是ARR和PSC定时1s
    TIM_TimeBaseStructure.TIM_Prescaler = 7200 - 1;//PSC分频器分频系数= 值+1
    TIM_TimeBaseStructure.TIM_RepetitionCounter = 0;//重复计数器，这个是高级计数器才有的，我们不需要。所以给0
    TIM_TimeBaseInit(TIM2,&TIM_TimeBaseStructure);//这个函数是初始化时基单元的。

    TIM_ClearFlag(TIM2,TIM_IT_Update);          //清除更新标志位
    //配置输出中断控制
    TIM_ITConfig(TIM2,TIM_IT_Update,ENABLE);//使能TIM2的更新中断到NVIC
    //配置NVIC
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//选择分组
    NVIC_InitTypeDef NVIC_InitStructuer;//定义结构体
    NVIC_InitStructuer.NVIC_IRQChannel = TIM2_IRQn;//选择定时器2在NVIC里的通道
    NVIC_InitStructuer.NVIC_IRQChannelCmd = ENABLE;//是否使能
    NVIC_InitStructuer.NVIC_IRQChannelPreemptionPriority = 2;//抢占优先级
    NVIC_InitStructuer.NVIC_IRQChannelSubPriority = 2;//响应优先级。
    NVIC_Init(&NVIC_InitStructuer);
    //运行控制，使能计数器
    TIM_Cmd(TIM2,ENABLE);
}




//配置中断函数
void TIM2_IRQHandler(void)
{
    if(TIM_GetITStatus(TIM2,TIM_IT_Update) == SET)//检查中断标志位
    {
        Speed = Encoder_Get();
        TIM_ClearITPendingBit(TIM2,TIM_IT_Update);
    }
}



		```
		在住函数中，只是新建了一个speed变量。其余都没有变
		如下：
		```c
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
#include "Encoder.h"
#include "Timer.h"

/*
    使用PA6 和 PA7 引脚的复用模式
    也就是TIM3 的CH1通道和CH2通道
    使用编码器接口
*/
int16_t Speed = 0;//对Timer.c中声明过的变量 

int main(){
    OLED_Init();//初始化OLED;
    Encoder_Init();//初始化Encoder
//    Delay_Init();//初始化延时函数
    
    Timer_Init();//初始化定时1s中断函数
    

    
    
    OLED_ShowString(1,1,"Num:");
    while(1)
    {
        OLED_ShowSignedNum(1,5,Speed,5);
//        Delay_ms(1000);
        
    }

}
    

		```
		<br>结果
		![](https://i-blog.csdnimg.cn/direct/ca2c7e2602564857ac64d39b991f3948.gif)
# **17.   看门狗WDG** {toggle="true"}
	## 什么是独立看门狗 {toggle="true"}
		STM32内置两个看门狗：独立看门狗IWDG、窗口看门狗WWDG
		**独立看门狗**IWDG由专用的低速时钟(LSI)驱动，即使主时钟发生故障它仍有效。<br>独立看门狗适合应用于需要看门狗作为一个在主程序之外能够完全独立工作，并且对时间精度要求低的场合
		**窗口看门狗WWDG**由从APB1时钟分频后得到时钟驱动。通过可配置的时间窗口来检测应用程序非正常的过迟或过早操作
		<empty-block/>
		看门狗可以监控程序的运行状态，当程序因为设计漏洞、硬件故障电磁干扰等原因，出现卡死或跑飞现象时，看门狗能及时复位程序避免程序陷入长时间的罢工状态，保证系统的可靠性和安全性看门狗本质上是一个定时器，当指定时间范围内，程序没有执行喂狗(重置计数器)操作时，看门狗硬件电路就自动产生复位信号
	## 工作原理 {toggle="true"}
		在**键值寄存器(IWDG_KR)**中写入0xCCCC，就可以开始**启用独立看门狗。**
		此时看门狗计数器开始从其复位值0xFFF递减，当计数器值计数到尾值0x000时会产生一个复位信号，复位单片机(IWDG_RESET)
		无论何时，**只要在键值寄存器IWDG_KR中写入0xAAAA(**通常说的喂狗).自动重装载寄存器IWDG_RLR的值就会**重新加载到计数器**，从而**避免看门狗复位**。如果程序异常，就无法正常喂狗，从而系统复位
	## 独立看门狗的框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4dd8ee8c-5610-4086-b33a-1fff7adf50d5.webp)
		键值寄存器IWDG KR  :0-15位有效
		预分频寄存器IWDG PR  :0\~2位有效(具有写保护功能，要操作先取消写保护)
		重装载寄存器IWDG RLR  :0\~11位有效(具有写保护功能，要操作先取消写保护)
		状态寄存器IWDG SR  :0\~1位有效
		输入时钟是内部低速时钟LSI
		然后进入预分频器进行分频。这个分频器是8位。最大256分频
		然后进入递减计数器。 这个计数器是12位的。所以最大值为2\^12 - 1 = 4095
		我们可以通过键寄存器写入特定数据，进行喂狗
		可以看到。上面的寄存器在1.8V工作区。下面的主要部分在VDD供电区。**所以在待机和停机都能正常工作。**
	## IWDG键寄存器 {toggle="true"}
		键寄存器本质上是控制寄存器，用于控制硬件电路的工作
		在可能存在干扰的情况下，一般通过在整个键寄存器写入特定值来代替控制寄存器写入一位的功能，**以降低硬件电路受到干扰的概率**
			<table header-row="true">
			<colgroup>
			<col width="154">
			<col width="450">
			</colgroup>
<tr>
<td>写入键寄存器的值</td>
<td></td>
</tr>
<tr>
<td>0xCCCC</td>
<td>启用独立看门狗</td>
</tr>
<tr>
<td>0xAAAA</td>
<td>IWDG_RLR中的值重新加载到计数器（喂狗）</td>
</tr>
<tr>
<td>0x5555</td>
<td>解除  IWDG_PR和IWDG_PLR的写保护</td>
</tr>
<tr>
<td>其他值</td>
<td>启用 IWDG_PR和IWDG_PLR的写保护</td>
</tr>
			</table>
	## 独立看门狗时间计算公式 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/875dd466-3bbd-468e-ad74-2944bbcef5fb.webp)
		<empty-block/>
	## 窗口看门狗框图 {toggle="true"}
		时钟在经过预分频器分频之后，进入 6位计数器（实际上是一个七位的）。这六位计数器在自减到0之后，下一次减，就会把T6 的1 也减为0 ，**<br>**也就是计数器从1 000000 减为 0 111111 时，相当于**计数器小于0x40就会产生复位**<br>这半部分是**规定了最晚的喂狗时间，**（不喂就自减到0）
		T6左边的WDGA则是窗口看门狗的使能位。他直接控制最后的与门
		上边的一部分是窗口时间的设置。  他连接的是一个比较器。 我们在写入WWDG_CR时，比较器比较并输出。比较的内容是T6:0  \> W6:0  也就是计数值还没减小到我们设置的值时就喂狗了。** 提前喂了，他就会输出1。执行复位**
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ae20fc2d-e24d-4631-bffb-7a6429a11bd0.webp)
	## 窗口看门狗的工作特性 {toggle="true"}
		- 递减计数器T\[6:0\]的值**小于**0x40时，WWDG产生复位
		- 递减计数器T\[6:0\]在窗口W\[6:0\]外被重新装载时，WWDG产生复位
		- 递减计数器T\[6:0\]**等于**0x40时可以产生**早期**唤醒中断(EWI)，用于重装载计数器以避免WWDG复位（溢出前的中断）
		- 定期写入WWDG CR寄存器(喂狗)以避免WWDG复位
	## 窗口看门狗时间计算公式 {toggle="true"}
		4096是固定的分频系数，不然太快了
		分频系数也是指定的。
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3c2d0768-e23c-4e0b-a71b-d86f572bd925.webp)
	## IWDG和WWDG对比 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c9b491d6-584a-4d24-9647-86845afef32c.webp)
	## 独立看门狗函数介绍 {toggle="true"}
		`void IWDG_WriteAccessCmd(uint16_t IWDG_WriteAccess);`<br>使能写入权限（实际上就是往键寄存器里写0x5555解除对PR和RLR的写保护）
		`void IWDG_SetPrescaler(uint8_t IWDG_Prescaler);`<br>设置预分频器
		`void IWDG_SetReload(uint16_t Reload);`<br>设置重装载寄存器
		`void IWDG_ReloadCounter(void);`
		重置计时器（喂狗）  （实际上就是往键寄存器里写0x0xAAAA 喂狗）
		`void IWDG_Enable(void);<br>`开启看门狗  （实际上就是往键寄存器里写0xCCCC  启用独立看门狗） <br>并且他会顺便写一个0x5555之外的值来启动写保护。
		`FlagStatus IWDG_GetFlagStatus(uint16_t IWDG_FLAG)`  <br>标志位查看
		<empty-block/>
		另外RCC里有一个可以查看在复位的时候 检测是看门狗复位还是上电或复位键复位。
		`FlagStatus RCC_GetFlagStatus(uint8_t RCC_FLAG);`
		这个查看标志位函数可以查看一些时钟是否准备好。以及复位的原因标志位<br>比如独立看门狗复位标志位：`RCC_FLAG_WWDGRST`    、 <br>窗口看门狗复位：`RCC_FLAG_WWDGRST`
		`<br>void RCC_ClearFlag(void);` <br>清除标志位（因为复位的这个标志位即使复位也不会自动清零）
	## 编写：独立看门狗 {toggle="true"}
		独立看门狗的开启步骤
		1. 开启LSI的时钟<br>这里在开启独立看门狗的时候会强制开启，不用写
		2. 解除写保护
		3. 自己计算一些喂狗时间。根据LSI的40KHZ频率。（0.025ms）<br>以及重装值和预分频值<br>超时时间 = 时钟周期 \* 预分频系数 \* （重装值 + 1）（重装住在4096之间。因为他是12位的）
		4. 配置预分频
		5. 配置重装值
		6. 启动看门狗
		```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
/*
    独立看门狗测试程序
    喂狗时间为1s内

*/

int main()
{
    OLED_Init();//初始化OLED;
    Delay_Init();
    KEY_Init();
    
    //这是一个IWDG测试程序
    OLED_ShowString(1,1,"Test IWDG");
    
    //复位后判断复位原因
    if(RCC_GetFlagStatus(RCC_FLAG_IWDGRST) == SET )//是独立看门狗复位
    {
        OLED_ShowString(2,1,"IWDG RST");
        //清除独立看门狗复位标志位，这个他复位也不会自动清除，需要手动清
        RCC_ClearFlag();
    }
    else
    {
        OLED_ShowString(2,1,"     RST");
    }
    
    
    //配置独立看门狗
    IWDG_WriteAccessCmd(IWDG_WriteAccess_Enable);//写入0x5555关闭保护
    IWDG_SetPrescaler(IWDG_Prescaler_16);//设置16分频（这样最大时间才为0.025*16*4096 = 1638ms）
    IWDG_SetReload(2500 - 1);//设置重装值为2500 这样喂狗时间就是1s内了。
    IWDG_ReloadCounter();//启动前喂狗一次，重置下计数器
    IWDG_Enable(); //启动小狗
    
    while(1)
    {
        KEY_Get();
        
        
        //0.9s喂狗一次
        Delay_ms(200);
        IWDG_ReloadCounter();
        OLED_ShowString(3,1,"WangWang");//显示喂狗了
        Delay_ms(700);
        OLED_ShowString(3,1,"        ");//闪烁
    }
}
//按键按下会进入while阻塞程序，不让喂狗
//uint8_t KEY_Get(void)
//{
//    if(KEY1)
//    {
//        Delay_ms(10);
//        while(KEY1);//等待松开
//            return 1;
//    }
//    return 0;
//}



		```
		<empty-block/>
	## 窗口看门狗函数介绍 {toggle="true"}
		`void WWDG_DeInit(void);<br>`恢复缺省配置`<br>void WWDG_SetPrescaler(uint32_t WWDG_Prescaler);<br>`写入预分频器`<br>void WWDG_SetWindowValue(uint8_t WindowValue);<br>`写入窗口值`<br>void WWDG_EnableIT(void);<br>`使能中断`<br>void WWDG_SetCounter(uint8_t Counter);<br>`写入计数器（喂狗）`<br>void WWDG_Enable(uint8_t Counter);<br>`使能看门狗 这里在喂狗的时候需要填入<br>使能定时器后要喂狗的值。防止刚使能就复位`<br>FlagStatus WWDG_GetFlagStatus(void);`
		获取标志位和清除标志位↕
		`void WWDG_ClearFlag(void);`
	## 编写：窗口看门狗 {toggle="true"}
		1. 开启APB1总线上的窗口看门狗时钟
			（窗口看门狗没有写保护）
		2. 配置预分频
		3. 配置窗口值
		4. 使能看门狗<br>看门狗使能、计数器溢出标志位、计数器有效位
		```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
/*
    窗口看门狗测试程序
    超时时间50ms
    窗口时间30ms

*/

int main()
{
    OLED_Init();//初始化OLED;
    KEY_Init();
    
    //这是一个WWDG测试程序
    OLED_ShowString(1,1,"Test WWDG");
    
    //复位后判断复位原因
    if(RCC_GetFlagStatus(RCC_FLAG_WWDGRST) == SET )//是独立看门狗复位
    {
        OLED_ShowString(2,1,"WWDG RST");
        //清除窗口看门狗复位标志位，这个他复位也不会自动清除，需要手动清
        RCC_ClearFlag();
    }
    else
    {
        OLED_ShowString(2,1,"     RST");
    }
    /*
        首先得到时钟时间。32M * 1/4096 = 7812.5HZ
        那么他一个周期的时间就是0.000128s
        那么选择分频为3，也就是2^3 = 8分频。那么现在一个周期的时间就是 0.000128 * 8= 0.001024s
        这时候如果重装载寄存器选为最大数字（6位）63 + 1
        那么他的最大超时时间就是0.58s
        也就是最大值为58ms
        我们要设置的50ms在范围内
        那么我们要设置的分频系数就是 3 所求得的重装值应是（约等于）55。实际给54  （因为 - 1）
    
        计算窗口时间，设置为30ms  那么设置窗口的6位的值为
        30 = 1/32m * 4096 * 8 *(54 - W[5:0])
        求得W的值为 21
      
    */
    //配置窗口看门狗
    //开启窗口开门狗时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_WWDG,ENABLE);
    WWDG_SetPrescaler(WWDG_Prescaler_8);
    WWDG_SetWindowValue(0x40 | 21);//30ms 
    WWDG_Enable(0x40 | 54);
    // 50ms 这里的0x40是为了把计数器的第七位置1 
    // 因为当他减为0是生成复位信号。（这也是重装载需要 -1 的原因）
    uint32_t i = 0;
    while(1)
    {
        KEY_Get();
        
        OLED_ShowString(3,1,"Wang");//很奇怪，如果是显示WangWang就不行了....多显示几个字符难道这么浪费时间吗
        Delay_ms(20);
        OLED_ShowString(3,1,"    ");
        Delay_ms(20);
        OLED_ShowNum(4,1,i,4);
        i++;
        WWDG_SetCounter(0x40 | 54);//喂狗，这里注意不要离使能时喂狗太近。

       
        
        
    }
}


		```
# **18.   ADC模数转换器** {toggle="true"}
	## AD模拟简介 {toggle="true"}
		ADC(Analog-Digital Converter)**模拟-数字转换器**
		ADC可以将引脚上连续变化的模拟电压转换为内存中存储的数字变量，建立模拟电路到数字电路的桥梁
		12位逐次逼近型ADC，1us转换时间
		输入电压范围:0\~3.3V，转换结果范围:0\~4095
		18个输入通道，可测量16个外部（GPIO）和2个内部信号源（内部温度传感器和内部参考电压也叫基准电压）
		**规则组**和**注入组**两个转换单元（可以列个组，一次性启动一个组，连续转换多个值）
		**模拟看门狗**自动监测输入电压范围
		<empty-block/>
		STM32F103C8T6   **ADC资源:ADC1、ADC2**，**10个外部输入通道**
	## ADC框图介绍 {toggle="true"}
		### 以ADC0809模块举例介绍ADC {toggle="true"}
			这个是因为以前单片机垃圾，需要外接才能实现AD转换
			通道选择开关，选择一路到比较器进行转换<br>地址锁存和译码就是用来控制通道选择开关的，
			比较器一段接的是待测电压，另一端接的是DAC。DAC是数模转换器（DAC内部是通过加权内阻网络来实现的转换）
			然后通过比较器的输出，不断调整DAC的输出的电压。来得到近似相等的值。
			这就是逐次逼近型的ADC的原理。是通过逐次逼近寄存器SAR来完成的。
			为了最快找到未知电压的编码，通常使用二分查找。
			查找到的最终值会通过8位三态锁存缓冲器输出。
			最后右上角的EOC为转换结束信号。
			START是开始转换信号
			CLOCK是ADC时钟
			下面的VREF+ VREF- 是DAC的参考电压。这个参考电压决定了可以测的待测电压的范围
			最后左下角是整个芯片的供电。
			（通常VCC和参考电压的正 是连接在一起的。）
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f2519742-ee67-4737-9318-103b2754b47e.webp)
		### STM32的ADC框图 {toggle="true"}
			AD的输入通道，由次框图可以看出。由16个GPIO输入通道和两个内部信号源（温度传感器和基准电压）
			他们可以由选择器选择前往注入通道或者规则通道
			其中注入通道有四条。他们使用了四个注入通道数据寄存器。可以一下子全部在总线上输出
			另一个是规则通道。他最多可以开启16条。但是这十六个是共用一个规则通道数据寄存器。一次只能发送一个。如果我们要接收就需要用到DMA转换了。
			> 再往左看
			左边的VREF+VREF- VDDA和VSSA   
			前俩（上边俩）是ADC的参考电压。决定了ADC输入电压的范围。
			后俩（下边俩）是ADC的供电引脚。
			一般情况下。VREF+和VDDA是接在一起的。另外俩也链接在一起
			在F1C8T6的芯片上没有VREF+VREF- 俩引脚。因为内部已经连接
			（VDDA和VSSA是内部模拟部分的电源。比如ADC RC震荡器 锁相环等）
			所以ADC的电压输入范围是0\~3.3V
			> 再往右下角看
			可以看到一个ADCCLK  这个是用于驱动内部逐次比较的时钟。
			他来自于ADC预分频器。他是挂载在APB2的总线上。不过给ADC的预分频器很尴尬。他可以分频为2 4 6 8.  而 APB2的频率为72M  ADC的频率最大为14MHZ<br>所以我们只能使用6分频和8分频。得到12MHZ或者9MHZ的信号给ADC
			> 而他旁边的
			就是DMA请求，这个就是用于触发DMA来极性数据转运
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/841820e5-d54a-4753-8ba2-e97059baa845.webp)
			触发ADC开始转换的信号有两种。
			一种是硬件触发，一种是软件触发。
			硬件出发可以是下面这些
			上面圈圈是注入组，下边是规则组
			可以看到，这些触发源主要是来自定时器 
			用定时器去控制ADC的触发。这样就不会占用软件资源频繁地进入中断了。
			并且也可以通过EXTI 外部中断引脚来控制触发转换。这些都是可以的。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5f8c039f-8adc-4ac0-b643-2633e4931594.webp)
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/55b7f3a4-5ab8-4f6e-a5a7-89701d5c27e4.webp)
			在往上看，可以看到模拟看门狗。
			这个可以在模拟看门狗里指定阈值高限和阈值底限。如果启动了这个模拟看门狗，并且制定了看门的通道。那么 一旦这个通道超出或者小于范围  狗就会叫 也就是在上面申请一个 **模拟看门狗中断**
			模拟看门狗中断旁边则是转换结束和注入转换结束的标志位。他们同时 也会触发中断到NVIC
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c72e6754-3c2e-48b9-8a0c-4246f90e2f32.webp)
			<empty-block/>
		### STM32的ADC基本结构图↑简版 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/2a7073fd-dd95-422f-8f05-bded90971d1c.webp)
			<empty-block/>
	## ADC与GPIO引脚 {toggle="true"}
		这些GPIO就是stm32f103C8T6的所有ADC了。但ADC1 2 都在同一个引脚上。
		这样做 可以实现交叉模式。对一个信号同时采样，也可以分开采样。
		还有别的模式就不细说了
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8bf6d5fb-55ae-40e3-9c80-deb7a80a200d.webp)
		> 下面是通道对应的ADC引脚。（这里F1C8T6没有ADC3）
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/dec51a5d-90ca-4761-8c4c-81e5c445672d.webp)
	## 规则组的四种转换模式 {toggle="true"}
		### 单次转换，非扫描模式 {toggle="true"}
			总结起来是
			触发—处理—EOC标志位置一
			如果要换别的只需要把通道换一下就OK
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/143cdcec-04de-441d-82f8-2192262b3fcd.webp)
		### 连续转换，非扫描模式 {toggle="true"}
			这个模式会自己读取自己的标志位。
			然后一直运行。我们想要的时候直接读取就OK
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/b6c78d25-4b1d-4ad8-a2c8-272674c822da.webp)
		### 单次转换，扫描模式 {toggle="true"}
			这里我们一下子写了很多个通道进去。这里的位置 和 次数是任意的
			这里我们可以指定通道数目。来指定ADC来判断前几个。
			这里要注意在EOC置一的时候，需要用MDA把这些数字挪走。然后再触发下一次的AD转换
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/90764e42-2627-4475-a82a-1887bfdf1c04.webp)
		### 连续转换，扫描模式 {toggle="true"}
			一次转换结束后，立刻开始下一次转换
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/55753964-8d07-462c-b6bf-6afd4fa43bbc.webp)
	## 一些知识补充 {toggle="true"}
		### 规则组的触发信号 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3824e1fa-4af2-4c0d-8d72-19b9b884ddaa.webp)
		### 数据对齐 {toggle="true"}
			**一个是数据往右方，高位补零（一般使用这个）**
			一个是数据往左放，低位补零（他的作用是舍弃一些精度。）
			<embed src=""></embed>
		### 转换时间 {toggle="true"}
			AD转换的步骤  ： 采样，保持，量化，编码
			采样 ：因为电平不断跳变，所以要暂时用电容把他存起来。 开关的打开和关闭需要时间<br>所以**采样需要时间**<br>STM32 ADC的总转换时间为：T_CONV =**采样时间**+12.5个ADC周期<br>例如:当ADCCLK=14MHz，采样时间为1.5个ADC周期TcoNv=1.5+12.5=14个ADC周期= 1us<br>这就是ADC 最快为1us的时间来源
		### ADC校准 {toggle="true"}
			ADC有一个内置自校准模式。校准可大幅减小因内部电容器组的变化而造成的准精度误差。校准期间，在每个电容器上都会计算出一个误差修正码(数字值)，这个码用于消除在随后的转换中每个电容器上产生的误差
			建议在每次上电后执行一次校准
			启动校准前，ADC必须处于关电状态超过至少两个ADC时钟周期
	## ADC的库函数介绍 {toggle="true"}
		`void RCC_ADCCLKConfig(uint32_t RCC_PCLK2);`  RCC.h中<br>用来配置RCC的时钟
		`void ADC_DeInit(ADC_TypeDef* ADCx);<br>void ADC_Init(ADC_TypeDef* ADCx, ADC_InitTypeDef* ADC_InitStruct);<br>void ADC_StructInit(ADC_InitTypeDef* ADC_InitStruct);` <br>`void ADC_Cmd(ADC_TypeDef* ADCx, FunctionalState NewState);`<br>恢复缺省配置、 配置初始化  初始化、使能ADC
		`void ADC_DMACmd(ADC_TypeDef* ADCx, FunctionalState NewState);` <br>开启DMA搬运
		`void ADC_ITConfig(ADC_TypeDef* ADCx, uint16_t ADC_IT, FunctionalState NewState);<br>`控制某个中断能不能通往NVIC
		`void ADC_ResetCalibration(ADC_TypeDef* ADCx);<br>FlagStatus ADC_GetResetCalibrationStatus(ADC_TypeDef* ADCx);<br>void ADC_StartCalibration(ADC_TypeDef* ADCx);<br>FlagStatus ADC_GetCalibrationStatus(ADC_TypeDef* ADCx);`<br>复位校准、获取复位校准状态、开始校准、获取开始校准状态 控制校准状态用的
		`void ADC_SoftwareStartConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);` <br>ADC软件转换控制、 软件触发转换
		`void ADC_SoftwareStartConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);` <br>给SWStart位置1，以开始转换
		`void ADC_ClearFlag(ADC_TypeDef* ADCx, uint8_t ADC_FLAG);` <br>获取标志位状态。比如EOC 转换结束标志位
		`void ADC_DiscModeChannelCountConfig(ADC_TypeDef* ADCx, uint8_t Number);<br>void ADC_DiscModeCmd(ADC_TypeDef* ADCx, FunctionalState NewState);` <br>间断模式配置和启用间断模式
		`void ADC_RegularChannelConfig(ADC_TypeDef* ADCx, uint8_t ADC_Channel, uint8_t Rank, uint8_t ADC_SampleTime);` <br>作用是给序列的每个位置填写指定的位置<br>第一个参数ADCx、  指定的通道、  序列几的位置  、  通道的采样时间
		`void ADC_ExternalTrigConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);` <br>是否允许外部触发转换
		`uint16_t ADC_GetConversionValue(ADC_TypeDef* ADCx);` <br>获取AD转换的数据寄存器，读取转换需要用到
		`uint32_t ADC_GetDualModeConversionValue(void);` <br>双ADC读取转换数据寄存器
		<empty-block/>
		`void ADC_AnalogWatchdogCmd(ADC_TypeDef* ADCx, uint32_t ADC_AnalogWatchdog);<br>void ADC_AnalogWatchdogThresholdsConfig(ADC_TypeDef* ADCx, uint16_t HighThreshold, uint16_t LowThreshold);<br>void ADC_AnalogWatchdogSingleChannelConfig(ADC_TypeDef* ADCx, uint8_t ADC_Channel);` <br>模拟看门狗配置： 配置是否启动看门狗、 配置高低阈值、配置看门通道
		<empty-block/>
		`void ADC_TempSensorVrefintCmd(FunctionalState NewState);` <br>ADC温度传感器，内部参考电压设置 这个是内部的俩通道的
		<empty-block/>
		`FlagStatus ADC_GetFlagStatus(ADC_TypeDef* ADCx, uint8_t ADC_FLAG);<br>void ADC_ClearFlag(ADC_TypeDef* ADCx, uint8_t ADC_FLAG);<br>ITStatus ADC_GetITStatus(ADC_TypeDef* ADCx, uint16_t ADC_IT);<br>void ADC_ClearITPendingBit(ADC_TypeDef* ADCx, uint16_t ADC_IT);` <br>获取标志位状态、清除标志位、获取中断状态、清除中断挂起位
		<empty-block/>
		注入组的一些配置
		```javascript
void ADC_AutoInjectedConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);
void ADC_InjectedDiscModeCmd(ADC_TypeDef* ADCx, FunctionalState NewState);
void ADC_ExternalTrigInjectedConvConfig(ADC_TypeDef* ADCx, uint32_t ADC_ExternalTrigInjecConv);
void ADC_ExternalTrigInjectedConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);
void ADC_SoftwareStartInjectedConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);
FlagStatus ADC_GetSoftwareStartInjectedConvCmdStatus(ADC_TypeDef* ADCx);
void ADC_InjectedChannelConfig(ADC_TypeDef* ADCx, uint8_t ADC_Channel, uint8_t Rank, uint8_t ADC_SampleTime);
void ADC_InjectedSequencerLengthConfig(ADC_TypeDef* ADCx, uint8_t Length);
void ADC_SetInjectedOffset(ADC_TypeDef* ADCx, uint8_t ADC_InjectedChannel, uint16_t Offset);
uint16_t ADC_GetInjectedConversionValue(ADC_TypeDef* ADCx, uint8_t ADC_InjectedChannel);
		```
	## 编写：单通道 单次转换 非扫描模式测电压 {toggle="true"}
		### AD.c {toggle="true"}
			```javascript
void AD_Init(void)
{
    //开启GPIOA时钟和ADC1外设时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1,ENABLE);
    //配置ADC预分频器
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);//12M
    //配置PA0
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;//模拟输入模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);
    
    //配置规则通道组
    ADC_RegularChannelConfig(ADC1,ADC_Channel_0,1,ADC_SampleTime_55Cycles5);//规则组序列1的位置，配置为通道0
    
    //初始化ADC
    ADC_InitTypeDef ADC_InitStructure;
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;     //是否连续转换：否
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;  //右对齐
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None; //外部触发：无
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;      //模式:独立ADC模式
    ADC_InitStructure.ADC_NbrOfChannel = 1;         //指定扫描模式下用几个通道
    ADC_InitStructure.ADC_ScanConvMode = DISABLE;   //是否扫描模式：否
    ADC_Init(ADC1,&ADC_InitStructure);
    //开启ADC1
    ADC_Cmd(ADC1,ENABLE);
    
    //校准
    //复位校准、等待复位校准完成、开始校准、等待校准完成
    ADC_ResetCalibration(ADC1); //复位校准
    while(ADC_GetResetCalibrationStatus(ADC1) == SET) //等待复位校准完成
    ADC_StartCalibration(ADC1); //开始校准
    while(ADC_GetCalibrationStatus(ADC1) == SET); //等待校准完成
}


uint16_t AD_Getvalue(void)
{
    //软件触发转换
    ADC_SoftwareStartConvCmd(ADC1,ENABLE);//软件 开始 转换控制
    while(ADC_GetFlagStatus(ADC1,ADC_FLAG_EOC) == RESET);//规则组转换完成标志位
    //55.5 + 12.5 = 68   72/6 = 12     1/12M * 68 = 5.6us
    //获取转换值
    return ADC_GetConversionValue(ADC1);
}
			```
		### main.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"

#include "AD.h"
/*
    使用ADC1
    检测ADC通道1  PA0引脚的模拟电压
    在OLED显示
*/


uint16_t ADValue = 0;
float Voltage = 0;
    
int main()
{
    OLED_Init();//初始化OLED;
    Delay_Init();
    AD_Init();//初始化AD

    
    OLED_ShowString(1,1,"Value:0000");
    OLED_ShowString(2,1,"Voltage:0.00V");

    while(1)
    {
        ADValue = AD_Getvalue();
        Voltage = (float)ADValue / 4095 * 3.3;
        
        //显示数值
        OLED_ShowNum(1,7,ADValue,4);
        //显示转换的电压
        OLED_ShowNum(2,9,Voltage,1);
        OLED_ShowNum(2,11,(uint16_t)(Voltage * 100 ) % 100, 2);
        Delay_ms(10);
    }
}




			```
	## 编写：单通道  单次转换  实现检测四个电压 {toggle="true"}
		### AD.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header

void AD_Init(void)
{
    //开启GPIOA时钟和ADC1外设时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1,ENABLE);
    //配置ADC预分频器
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);//12M
    //配置PA0
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;//模拟输入模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3 ;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);
    
    //手动在主函数中配置通道。达到切换的效果
    
    //初始化ADC
    ADC_InitTypeDef ADC_InitStructure;
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;     //是否连续转换：否
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;  //右对齐
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None; //外部触发：无
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;      //模式:独立ADC模式
    ADC_InitStructure.ADC_NbrOfChannel = 1;         //指定扫描模式下用几个通道
    ADC_InitStructure.ADC_ScanConvMode = DISABLE;   //是否扫描模式：否
    ADC_Init(ADC1,&ADC_InitStructure);
    //开启ADC1
    ADC_Cmd(ADC1,ENABLE);
    
    //校准
    //复位校准、等待复位校准完成、开始校准、等待校准完成
    ADC_ResetCalibration(ADC1); //复位校准
    while(ADC_GetResetCalibrationStatus(ADC1) == SET) //等待复位校准完成
    ADC_StartCalibration(ADC1); //开始校准
    while(ADC_GetCalibrationStatus(ADC1) == SET); //等待校准完成
}


uint16_t AD_Getvalue(uint8_t ADC_Channel)
{
    //配置规则通道组
    ADC_RegularChannelConfig(ADC1,ADC_Channel,1,ADC_SampleTime_55Cycles5);//规则组序列1的位置，配置为通道ADC_Channel
    //软件触发转换
    ADC_SoftwareStartConvCmd(ADC1,ENABLE);
    while(ADC_GetFlagStatus(ADC1,ADC_FLAG_EOC) == RESET);//等待转换完成
    //获取转换值
    return ADC_GetConversionValue(ADC1);
}























			```
		### main.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"

#include "AD.h"
/*
    使用ADC1
    检测ADC通道1  PA0引脚的模拟电压
    在OLED显示
*/


uint8_t mode = 2;

uint16_t ADValue1 = 0;
uint16_t ADValue2 = 0;
uint16_t ADValue3 = 0;
uint16_t ADValue4 = 0;

float Voltage1 = 0;
float Voltage2 = 0;
float Voltage3 = 0;
float Voltage4 = 0; 

int main()
{
    OLED_Init();//初始化OLED;
    Delay_Init();
    AD_Init();//初始化AD

    if(mode == 1)
    {
        OLED_ShowString(1,1,"Value1:0000");
        OLED_ShowString(2,1,"Value2:0000");
        OLED_ShowString(3,1,"Value3:0000");
        OLED_ShowString(4,1,"Value4:0000");
    }
    else if(mode == 2)
    {
        OLED_ShowString(1,1,"Voltage1:0.00V");
        OLED_ShowString(2,1,"Voltage2:0.00V");
        OLED_ShowString(3,1,"Voltage3:0.00V");
        OLED_ShowString(4,1,"Voltage4:0.00V");
    }
    
    
    while(1)
    {
        
        
        if(mode == 1)
        {
            //显示数值
            ADValue1 = AD_Getvalue(ADC_Channel_0);
            ADValue2 = AD_Getvalue(ADC_Channel_1);
            ADValue3 = AD_Getvalue(ADC_Channel_2);
            ADValue4 = AD_Getvalue(ADC_Channel_3);
            OLED_ShowNum(1,8,ADValue1,4);
            OLED_ShowNum(2,8,ADValue2,4);
            OLED_ShowNum(3,8,ADValue3,4);
            OLED_ShowNum(4 ,8,ADValue4,4);
        }
        else if(mode == 2)
        {
            ADValue1 = AD_Getvalue(ADC_Channel_0);
            ADValue2 = AD_Getvalue(ADC_Channel_1);
            ADValue3 = AD_Getvalue(ADC_Channel_2);
            ADValue4 = AD_Getvalue(ADC_Channel_3);
            //显示转换的电压
            Voltage1 = (float)ADValue1 / 4095 * 3.3;
            Voltage2 = (float)ADValue2 / 4095 * 3.3;
            Voltage3 = (float)ADValue3 / 4095 * 3.3;
            Voltage4 = (float)ADValue4 / 4095 * 3.3;
            
            OLED_ShowNum(1,10,Voltage1,1);
            OLED_ShowNum(1,12,(uint16_t)(Voltage1 * 100 ) % 100, 2);
            OLED_ShowNum(2,10,Voltage2,1);
            OLED_ShowNum(2,12,(uint16_t)(Voltage2 * 100 ) % 100, 2);
            OLED_ShowNum(3,10,Voltage3,1);
            OLED_ShowNum(3,12,(uint16_t)(Voltage3 * 100 ) % 100, 2);
            OLED_ShowNum(4,10,Voltage4,1);
            OLED_ShowNum(4,12,(uint16_t)(Voltage4 * 100 ) % 100, 2);
        }
        Delay_ms(10);
    }
}




			```
# **19.   DMA 直接存储器存取** {toggle="true"}
	## DMA简介 {toggle="true"}
		- DMA(Direct Memory Access)直接存储器存取
		- DMA是AHB总线的设备
		- DMA可以提供外设和存储器（运行内存SRAM和程序存储器Flash）或者存储器和存储器之间的高速数据传输，把一个或者一批数据把一个地址空间复制到另一个地址空间。**无须CPU干预，节省了CPU的资源**
		- 12个独立可配置的通道:  DMA1(7个通道)  DMA2(5个通道)  每个通道都支持软件触发和**特定的硬件**触发（特定 是因为  你要是用那个外设的硬件触发源，就要使用它连接的那个通道。不能任意选择通道）
		- STM32F103C8T6 DMA资源:  DMA1 (7个通道)
		- **存储器 到 存储器一般使用软件触发。** **存储 器 和外设 一般用硬件触发**。因为有一定的时机性
	## 存储器映像 {toggle="true"}
		计算器系统的额五大组成部分是
		- 运算器
		- 控制器
		- 存储器
		- 输入设备
		- 输出设备
		其中运算器和控制器一般合在一起 被称为 CPU。
		所以计算器的核心   就是 CPU  和  存储器
		存储器有两个重要的知识点：**存储器的内容** 和 **存储器的地址**
		ROM是只读存储器，非易失性，掉电不丢失的存储器
		RAM是随机存储器，易失性，掉电丢失的存储器
		他们各分为三块，他们各有不同的起始地址。 他们的终止地址取决于它的容量。编到哪里 哪里就是最终地址。
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ce35fbfc-e60a-435b-8fc8-84d5ddcb2f4e.webp)
		（选项字节 中存的是Flash的读保护 写保护  还有看门狗等等的配置）
		比如代码这样写。那么显示的就是
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/d457e884-04a0-40fd-a7c5-307146d572ed.webp)
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/273101aa-1132-4dc0-9261-940655f4255a.webp)
		一些小知识：
		**SRAM（静态随机存储器）、RAM（随机存储器）和 ROM（只读存储器）的区别主要体现在以下几个方面**：
		1. 数据可修改性
			- SRAM 和 RAM 中的数据可以随时读取和写入，是可读可写的存储器。
			- ROM 中的数据在正常工作状态下通常只能读取，不能写入，其存储的内容在制造时就已确定。
		2. 存储原理
			- SRAM 是利用触发器来存储数据，只要不掉电，数据就不会丢失。
			- RAM 通常指的是 DRAM（动态随机存储器），它利用电容存储电荷来保存数据，需要定时刷新以维持数据。
			- ROM 则是通过掩膜工艺或在特定条件下一次性写入数据。
	## DMA框图 {toggle="true"}
		- DMA是AHB总线的设备
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f59f93b6-c662-4608-9dab-1fa014c35a68.webp)
		寄存器是一种特殊的存储器。一方面CPU可以对寄存器进行读写。 就像对运行内存的读写一样。另一方面，寄存器的每一位背后都连接了导线。这些导线可以用于控制外设电路的状态（比如置引脚的高低电平，导通、断开开关、切换数据选择器、或者多个寄存器结合起来成为计数器、数据寄存器等等）<br>所以，**寄存器 是连接软件和硬件的桥梁，软件对寄存器的读写就间接的控制了硬件的执行**
		> 右边看
		右边的是被动单元，他们的存储器只能被左边的主动单元读写
		- **Flash**  是   主闪存  是ROM只读存储器，不呢被CPU和DMA直接写入，只能读取
		- **SRAM** 是 运行内存
		- **各个外设**都可以看成是 SRAM的寄存器。
		> 左上角看
		主动单元这里，内核有Dcode和系统总线。可以访问右边的存储器
		- **Dcode总线**是专门用来访问Flash的
		- **系统总线**是访问其他东西的
		DMA也有访问的主动权
		- **这里DAM1 和DMA2 **各有一条DMA总线连接到总线矩阵。（下面还有一条DMA总线是以太网外设私有的DMA）
			- DMA1 有7 个通道
			- DMA2 有5 个通道
			- 各个通道可以分别设置它们转运数据的源地址和目的地址
		- **仲裁器：**虽然多个通道可以独立转运数据，但是最终DMA总线只有一条。所有的通道都只能分时复用一条DMA总线，如果产生了冲突， 就会由仲裁器根据通道的优先级来决定谁先用谁后用
		(总线矩阵哪里也会有一个仲裁器。 当DMA和CPU同时访问同一个目标。 那么DMA就会暂停CPU的访问，以防止冲突，不过总线仲裁器会保证分给CPU一半的总线带宽，使CPU也能正常工作)   
		（**总线带宽是指**在单位时间内总线能够传输的数据量。
		总线就像是信息传输的通道，而带宽则表示这个通道在一定时间内能够传输多少信息。它通常用每秒传输的数据位数（如比特/秒，bps）或字节数（如字节/秒，Bps）来衡量。）
		- 每个DMA中都还有一个**  DMA从设备**，  也就是DMA的自身的寄存器。因为DMA作为一个外设。他自己也会有一个相应的配置寄存器。（可以在框图上看到，他也可以被总线矩阵连接控制来读写。）
		- **DMA请求 ：**可以看到，外设的APB1 和APB2 总线 对DMA1 和DMA2 各有一条DMA请求。这个就是用来  硬件触发DMA的数据搬运
	## DMA基本结构框图与各部分作用 {toggle="true"}
		- DMA是AHB总线的设备
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c5336fed-4288-452f-bd26-2350ac032dd7.webp)
		红色圈圈  是DMA搬运的两大站点了
		- 左边是外设寄存器
		- 右边是存储器站点，包括 Flash 和 SRAM<br>当然。Flash 是只读存储器，所以DMA不可以进行Flash到Flash  、 SRAM到Flash的转运操作。
		- 外设和存储器站点 都有三个参数
			- 起始地址
			- 数据宽度<br>指定一次转运要按多大的数据宽度来进行<br>可以是 字节byte  （8bit）、  半字HalfWord   （16bit）  、 和字 Word   （32bit）
			- 地址是否自增
		- 下边的传输计数器，是用来指定 总共转运几次的。是一个自减计数器.最大值为65535个
		- 旁边的自动重装器，决定了传输计数器在回归0之后，是否要重新填充，如果填充了，那么就继续循环。如果不填充 就是 单次模式。
		- **M2M**（Memory to Memory）：用来选择软件触发还是硬件触发， 1 为软件， 0为硬件
			软件触发的逻辑是，以最快的速度，连续不断的触发DMA，争取早日能把传输计数器清零。完成这一轮的转换。（**软件触发和启动重装寄存器后的循环模式不能同时启用！！**）  软件触发一般 用于  存储器到存储器之间的转运。
			硬件触发，是外设在到达某一时机的时候，传一个信号给DMA，触发DMA进行转运
		- 最后就是右下角的开关控制， 使能位了。
		> **需要注意的是 ，**在不循环的模式下，自动重装器不会给传输计数器重新装载。也就是说，在下次搬运的时候。我们需要手动的给计数器重新写入一个新的值，让他进行搬运。<br>**但是！！在DMA开着的时候，不能进行传输计数器的写入，  正确的流程是 关闭DMA、写入计数器、打开DMA <br><br>**并且在使用硬件触发时**，不同的通道对应的硬件触发链接不一样**， 对于F103C8t6来说。DMA1 通道与硬件外设的连接的图如下：
	## 通道与外设对应图（STM32F103C8t6  DMA1） {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/51652de2-5f9c-4168-86a3-e7e1fac21225.webp)
	## 搬运时源端与目标宽度不一致时解决方案\[表\] {toggle="true"}
		**目标比源端的宽度大**。那么  **在目标高位补 0 **
		**目标比源端的宽度小。**那么   **舍弃源端高 位**
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/6ddc6b2a-e068-4eb6-ad76-e30d749d0dde.webp)
	## 数据转运 + DMA 讲解 {toggle="true"}
		以这个例子来说。应该填
		- 起始位置为DataA     目标位置为 Data B
		- 数据宽度是8bit， 也就是1字节
		- 并且两个站点的地址都应该自增。
		- 传输次数为 7  不需要自动重装
		- 触发选择为 软件触发。  这是存储器到存储器的转运
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a94dbb38-7f49-444d-8fbe-bb14e2eb2296.webp)
	## ADC扫描模式  +  DMA 讲解 {toggle="true"}
		左边为ADC的7 个通道，每次触发  7个通道会 依次进行转换。<br>但是在存入的时候只能存入一个到ADC_DR这一个寄存器里。
		所以在每个单独的通道转换之后，就要用DMA移动到SRAM数组。并且自增存储器的 地址
		所以选择外设触发，外设地址选择有ADC_DR的通道
		因为ADC_DR和SRAM都是16bit的宽度， 所以在转运的过程中 选择的是16 bit 的半字输出
		通道有7条，所以 计数为7次。
		对于重装寄存器是否要工作，则要根据ADC的配置来选择<br>如果ADC是单次扫描，那么DMA的计数器可以不自动扫描<br>如果ADC是连续扫描，那么DMA的计数器可以使用自动重装。DMA和ADC同步工作，
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/90760743-15e1-4e4e-965d-25f17624c655.webp)
	## DMA相关函数介绍 {toggle="true"}
		- `void DMA_DeInit(DMA_Channel_TypeDef* DMAy_Channelx);<br>`恢复缺省配置
		- `void DMA_Init(DMA_Channel_TypeDef* DMAy_Channelx, DMA_InitTypeDef* DMA_InitStruct);<br>`初始化
		- `void DMA_StructInit(DMA_InitTypeDef* DMA_InitStruct);<br>`结构体初始化
		- `void DMA_Cmd(DMA_Channel_TypeDef* DMAy_Channelx, FunctionalState NewState);<br>`使能
		- `void DMA_ITConfig(DMA_Channel_TypeDef* DMAy_Channelx, uint32_t DMA_IT, FunctionalState NewState);<br>`中断输出使能
		- `void DMA_SetCurrDataCounter(DMA_Channel_TypeDef* DMAy_Channelx, uint16_t DataNumber);<br>`DMA设置当前数据寄存器   给 传输计数器写数据用的
		- `uint16_t DMA_GetCurrDataCounter(DMA_Channel_TypeDef* DMAy_Channelx);<br>`返回传输计数器的值
		- `FlagStatus DMA_GetFlagStatus(uint32_t DMAy_FLAG);<br>`获取标志位状态
		- `void DMA_ClearFlag(uint32_t DMAy_FLAG);<br>`清楚标志位
		- `ITStatus DMA_GetITStatus(uint32_t DMAy_IT);<br>`获取中断状态
		- `void DMA_ClearITPendingBit(uint32_t DMAy_IT);`<br>清除中断挂起位 
		<empty-block/>
		<empty-block/>
	## 编写：DMA存储器与存储器之间的搬运 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a94dbb38-7f49-444d-8fbe-bb14e2eb2296.webp)
		DMA存储器与存储器搬运。
		const 修饰的内容会被保存在Flash区域，这是只读的，并且上电不丢失
		其他的变量会被保存在地址为0x2000 0000 的静态随机存储器（SRAM）上
		数组与数组之间的搬运在DMA初始化时函数时，要接受两个数组的 源地址、目标地址、以及两个数组的 宽度 。 
		DMA是AHB总线上的 外设。所以在启动时钟时使用的是AHB总线。
		存储器与存储器之间的搬运，一般用软件触发。<br>DMA在被软件出发后，会尽可能快的清零传输计数器，来完成此次搬运。<br>所以在使用软件触发的时候，不能开启自动重装器，否则DMA就会一直搬运。无法停下
		DMA在启动后会立刻开始工作。在干完这一轮之后 ，计数器清零。也就停下了。
		若想要再次触发，则要再次填充传输计数器里的值。<br>要先失能 DMA。否则无法填充。填充的函数是 `DMA_SetCurrDataCounter(DMA1_Channel1,MyDMA_size);` 
		并且在填充完成后，要在函数内判断是否传输完成。完成后才能开启下一轮的转换。<br>这是通过判断`DMA1_FLAG_TC1`标志位来实现的（标志位置1为实现）。这个完成标志位不会自动清除，所以要手动在函数内清除，这是一个很好的习惯。 `DMA_ClearFlag(DMA1_FLAG_TC1);`<br>其他的还有这些标志位<br>*“GL” 是 “Global”（全局）的缩写。<br>“TC” 是 “Transfer Complete”（转运完成）的缩写。<br>“HT” 是 “Half Transfer”（转运过半）的缩写。<br>“TE” 是 “Transfer Error”（转运错误）的缩写。*
		### MyDMA.c代码如下（添加My是为了不与自带的库的函数重复名字） {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header

uint16_t MyDMA_size = 0;

void MyDMA_Init(uint32_t AddrA, uint32_t AddrB, uint16_t size)
{
    MyDMA_size = size;
    //开启DMA外设的时钟
    RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
    //初始化DMA参数
    DMA_InitTypeDef DMA_InitStructure;
    DMA_InitStructure.DMA_PeripheralBaseAddr = AddrA;//源基地址
    DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Byte;//源宽度
    DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Enable;//源地址是否自增
    DMA_InitStructure.DMA_MemoryBaseAddr = AddrB;//目标基地址
    DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_Byte;//目标宽度 字节
    DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;//目标是否自增
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;//传输方向 1-存储器到外设。2-外设到 存储器
                    //因为我们后面要设置那个是外设那个是存储器，选哪个问题不大。
    DMA_InitStructure.DMA_BufferSize = size;//传输计数器  次数 0 - 65535
    DMA_InitStructure.DMA_Mode = DMA_Mode_Normal;//指定重装器是否要重装（注意不能应用在存储器到存储器的 软件重装模式
    DMA_InitStructure.DMA_M2M = DMA_M2M_Enable; //是否使用软件触发  是
    DMA_InitStructure.DMA_Priority = DMA_Priority_VeryHigh;//选择优先级
    DMA_Init(DMA1_Channel1,&DMA_InitStructure);//软件触发，通道随便选。
    //使能DMA
    DMA_Cmd(DMA1_Channel1,DISABLE);//不让他立刻工作，而是调用函数后工作
}

//调用 后 转换一次
void MyDMA_Transfer(void)
{
    DMA_Cmd(DMA1_Channel1,DISABLE);//失能
    DMA_SetCurrDataCounter(DMA1_Channel1,MyDMA_size);
    DMA_Cmd(DMA1_Channel1,ENABLE);//使能
    while(DMA_GetFlagStatus(DMA1_FLAG_TC1) == RESET)//等待转换完成标志位
    DMA_ClearFlag(DMA1_FLAG_TC1);//手动清除标志位，这个不会自动清除
    
    //GL全局标志位、  TC转运完成标志位、
    //HT转运过半标志位、  TE转运错误标志位
    
    //“GL” 是 “Global”（全局）的缩写。
    //“TC” 是 “Transfer Complete”（转运完成）的缩写。
    //“HT” 是 “Half Transfer”（转运过半）的缩写。
    //“TE” 是 “Transfer Error”（转运错误）的缩写。
}


			```
		### main.c代码如下： {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
#include "MyDMA.h"
/*
	DMA搬运数组
	数组一秒搬运一次，
	源数组一秒自增一次
*/

uint8_t DataA[] = { 0x01, 0x02, 0x03, 0x04 };
uint8_t DataB[4] = { 0 };


int main()
{
    OLED_Init();//初始化OLED;
    Delay_Init();//初始化延时
    
    MyDMA_Init((uint32_t)DataA,(uint32_t)DataB,4);//初始化DMA
    
    
    //显示地址
    OLED_ShowString(1,1,"DataA");
    OLED_ShowHexNum(1, 1, (uint32_t)&DataA, 8);
    
    OLED_ShowString(3,3,"DataB");
    OLED_ShowHexNum(3, 1, (uint32_t)&DataB, 8);

    while(1)
    {
        //显示DataA数据
        OLED_ShowHexNum(2, 1, DataA[0], 2);
        OLED_ShowHexNum(2, 4, DataA[1], 2);
        OLED_ShowHexNum(2, 7, DataA[2], 2);
        OLED_ShowHexNum(2, 10, DataA[3], 2);
        //延时
        Delay_ms(1000);
        
        //传输
        MyDMA_Transfer();
        
        //显示DataB数据
        OLED_ShowHexNum(4, 1, DataB[0], 2);
        OLED_ShowHexNum(4, 4, DataB[1], 2);
        OLED_ShowHexNum(4, 7, DataB[2], 2);
        OLED_ShowHexNum(4, 10, DataB[3], 2);
        //延时
        Delay_ms(1000);
        //源数组自增
        DataA[0]++;
        DataA[1]++;
        DataA[2]++;
        DataA[3]++;        
    }
}

//uint8_t SRAM = 0x66;
//const uint8_t Flash = 0x77;
//int main()
//{
//    OLED_Init();//初始化OLED;
//    OLED_ShowHexNum(1,1,SRAM,2);
//    OLED_ShowHexNum(1,4,(uint32_t)&SRAM,8);//地址是0x2000 0000 开头的。位于RAM类型的运行内存SRAM中
//    OLED_ShowHexNum(2,1,Flash,2); 
//    OLED_ShowHexNum(2,4,(uint32_t)&Flash,8);//地址是0x0800 0000 开头的。位于Flash中，是只读的。放到flash可以节省空间
//    OLED_ShowHexNum(3,4,(uint32_t)&ADC1->DR, 8); //ADC_DR寄存器是外设。所以在RAM类型的 0x4000 0000上。这个地址是固定的。
//    while(1)
//    {
//    }
//}




			```
		<empty-block/>
	## 编写：ADC单次连续扫描+DMA单次连续搬运 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/90760743-15e1-4e4e-965d-25f17624c655.webp)
		ADC单次连续扫描，是 没有开启连续转换的<br>要软件触发一次，ADC才能启动连续扫描一哦。<br>ADC在扫描指定的前几个通道后，就自动停下。
		但是对于规则组。所有（16个，对于C8T6只有7个）的通道是共用一个ADC_DR寄存器的。<br>也就是说，同时只能有一个通道在寄存器中等待读取。
		那么！我们就要在每个通道转换完成，然后输出到ADC_DR寄存器的同时，抓紧把已经上好的菜端走，否则就要被下一个菜挤走了。
		初始ADC，需要注意的是，配置通道为4条，并把它放入对应的规则组序列位置。<br>并且要把扫描通道的个数改为 4  。<br>并且要把扫描模式打开
		在初始化ADC后，就准备DMA的初始化了。<br>首先要设置源地址为ADC的DR寄存器。记得把他转为uint32_t的格式，否则无法被正确识别。并且宽度为半字。 16位。  并且是不自增的！<br>其次要设置目标地址为我们定义的四个元素大小的uint16_t的数组AD_Value\[4\]中。<br>。这事宽度为半字，这里要自增。每次搬运到一个元素后自动前往下一个位置。<br>DMA同样的， 要和ADC协同，ADC是4个通道。所以DMA要搬运四次。<br>并且，我们这里是ADC单次连续扫描，DMA单次连续搬运。所以DMA的重装寄存器是关闭的状态，不要让他重装。<br>并且，为了让DMA知道自己该搬运了。就要启动DMA的硬件触发。也就是M2M位是否选择软件触发，填DISABLE。<br>到这里，DMA的配置就基本完成了
		但是。DMA要如何知道自己该干活了呢？  这里就要开启DMA和AD之间的通道了。<br>也就是开启ADC到DMA的输出。`ADC_DMACmd(ADC1, ENABLE);` 这样，ADC1在每次某个通道完成之后，就会告诉DMA，你快来搬！！！！如下是ADC的框图，有一个DMA请求
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/60b4b65b-ae2a-4681-b1e0-e6a6fcb53da1.webp)
		在经过四次循环之后，DMA和ADC就同时结束了。
		如果我们要再次进行搬运，就要先关闭DMA.然后填入新的数字。再使能DMA<br>然后软件触发一次ADC1  。让他再次扫描四个通道。并且完成转换，同时DMA也能在ADC的告知下完成搬运。
		### AD.c代码如下 {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header

uint16_t AD_Value[4];//定义用于存放AD转换结果的全局数组


void AD_Init(void)
{
    //开启GPIOA时钟、ADC1外设时钟、DMA外设的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1,ENABLE);
    RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
    
    
    //配置ADC预分频器
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);//12M
    //配置PA0
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;//模拟输入模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3 ;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);
    
    //配置规则通道
    ADC_RegularChannelConfig(ADC1,ADC_Channel_0,1,ADC_SampleTime_55Cycles5);//规则组序列1的位置，配置为通道0
    ADC_RegularChannelConfig(ADC1,ADC_Channel_1,2,ADC_SampleTime_55Cycles5);//规则组序列2的位置，配置为通道1
    ADC_RegularChannelConfig(ADC1,ADC_Channel_2,3,ADC_SampleTime_55Cycles5);//规则组序列3的位置，配置为通道2
    ADC_RegularChannelConfig(ADC1,ADC_Channel_3,4,ADC_SampleTime_55Cycles5);//规则组序列4的位置，配置为通道3
    
    //初始化ADC
    ADC_InitTypeDef ADC_InitStructure;
    ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;     //是否连续转换：否
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;  //右对齐
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None; //外部触发：无
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;      //模式:独立ADC模式
    ADC_InitStructure.ADC_NbrOfChannel = 4;         //用4个通道
    ADC_InitStructure.ADC_ScanConvMode = ENABLE;   //是否扫描模式：是
    ADC_Init(ADC1,&ADC_InitStructure);
    
    
    /*ADC使能之前 开始配置DMA*/
    //初始化DMA参数
    DMA_InitTypeDef DMA_InitStructure;
    DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;//源基地址
    DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;//半字，16位
    DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;//源地址是否自增：否
    DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)AD_Value;//目标基地址
    DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;//目标宽度 字节
    DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;//目的地否自增:是
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;//传输方向 1-存储器到外设。2-外设到 存储器
    DMA_InitStructure.DMA_BufferSize = 4;//传输计数器   ：4次
    DMA_InitStructure.DMA_Mode = DMA_Mode_Normal;//指定重装器：单次
    DMA_InitStructure.DMA_M2M = DMA_M2M_Disable; //是否使用软件触发  否：
    DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;//选择优先级
    DMA_Init(DMA1_Channel1,&DMA_InitStructure);//硬件触发，ADC1 的硬件触发在通道1 所以不能变
    //使能DMA
    DMA_Cmd(DMA1_Channel1, DISABLE);
    
    //开启ADC到DMA的输出
    ADC_DMACmd(ADC1, ENABLE);

    //开启ADC1
    ADC_Cmd(ADC1, ENABLE);
    
    //校准  
    ADC_ResetCalibration(ADC1); //复位校准
    while(ADC_GetResetCalibrationStatus(ADC1) == SET) //等待复位校准完成
    ADC_StartCalibration(ADC1); //开始校准
    while(ADC_GetCalibrationStatus(ADC1) == SET); //等待校准完成
}

//单次DMA，单次ADC模式
void AD_Getvalue(void)
{ 
    //因为是ADC是单次，所以DMA没有开启重装循环模式
    DMA_Cmd(DMA1_Channel1,DISABLE);//失能
    DMA_SetCurrDataCounter(DMA1_Channel1,4);//装填四次
    DMA_Cmd(DMA1_Channel1,ENABLE);//使能
    
    //软件触发ADC转换
    ADC_SoftwareStartConvCmd(ADC1,ENABLE);
    //其他的等待转换完成，不需要了
    
    //等待DMA搬运完成标志位，手动清除标志位
    while (DMA_GetFlagStatus(DMA1_FLAG_TC1) == RESET);//T.T 注意加分号
    DMA_ClearFlag(DMA1_FLAG_TC1);
    
}

			```
		### AD.h代码如下： {toggle="true"}
			```javascript
#ifndef __AD_H
#define __AD_H

//对外声明数组
extern uint16_t AD_Value[4];

//初始化
void AD_Init(void);

//开启ADC一次，然后搬运出去一次
void AD_Getvalue(void);

#endif

			```
		### main.c代码如下 {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"

#include "AD.h"
/*
	DMA单次  连续扫描，
	DMA单次   连续搬运
显示4个通道的 转换值
*/


int main()
{
    OLED_Init();//初始化OLED;
    Delay_Init();//初始化延时
    AD_Init();
    
    while(1)
    {
            //显示通道
    OLED_ShowString(1,1,"AD1:");
    OLED_ShowString(2,1,"AD2:");
    OLED_ShowString(3,1,"AD3:");
    OLED_ShowString(4,1,"AD4:");
    
    
        
   //开启ADC 转换四条通道 1 次
    //然后DMA搬运到AD_Value数组中
    AD_Getvalue();


    //显示搬运后的数据
    OLED_ShowNum(1, 5, AD_Value[0], 4);
    OLED_ShowNum(2, 5, AD_Value[1], 4);
    OLED_ShowNum(3, 5, AD_Value[2], 4);
    OLED_ShowNum(4, 5, AD_Value[3], 4);
    Delay_ms(100);
        
 
    }
}





			```
	## 编写：ADC重复连续扫描+DMA重复连续搬运 {toggle="true"}
		对于ADC重复连续扫描+ DMA重复连续搬运
		对ADC来说，只需要打开连续转换即可。
		对DMA来说，只需要打开重装载寄存器对传输计数器的重新装载即可。 
		这时就不需要我们手动的一次次触发ADC来完成一次的ADC连续扫描+DMA连续搬运了。
		只需要在初始化函数的时候，让ADC和DMA不断地工作就OK 了。<br>所以需要在初始化函数的时候用软件触发一次ADC。<br>然后ADC就会不断地进行扫描四次通道，然后再次循环。  DMA也会在ADC的 通知下不断搬运。
		因为ADC会一直  循环的对指定的前几条通道扫描、转换。<br>DMA也会同时 不断循环的覆盖数组中的数值。 我们只需要在主函数中不断读取数组的值就可以完成ADC转换结果的显示。<br>这是硬件自动化完成的，不需要软件的干预，大大节省了CPU的资源。
		### AD.c代码如下 {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header

uint16_t AD_Value[4];//定义用于存放AD转换结果的全局数组


void AD_Init(void)
{
    //开启GPIOA时钟、ADC1外设时钟、DMA外设的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1,ENABLE);
    RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
    
    //配置ADC预分频器
    RCC_ADCCLKConfig(RCC_PCLK2_Div6);//12M
    
    //配置PA0
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;//模拟输入模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3 ;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);
    
    //配置规则通道
    ADC_RegularChannelConfig(ADC1,ADC_Channel_0,1,ADC_SampleTime_55Cycles5);//规则组序列1的位置，配置为通道0
    ADC_RegularChannelConfig(ADC1,ADC_Channel_1,2,ADC_SampleTime_55Cycles5);//规则组序列2的位置，配置为通道1
    ADC_RegularChannelConfig(ADC1,ADC_Channel_2,3,ADC_SampleTime_55Cycles5);//规则组序列3的位置，配置为通道2
    ADC_RegularChannelConfig(ADC1,ADC_Channel_3,4,ADC_SampleTime_55Cycles5);//规则组序列4的位置，配置为通道3
    
    //初始化ADC
    ADC_InitTypeDef ADC_InitStructure;
    ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;     //是否连续转换：是
    ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;  //右对齐
    ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None; //外部触发：无
    ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;      //模式:独立ADC模式
    ADC_InitStructure.ADC_NbrOfChannel = 4;         //用4个通道
    ADC_InitStructure.ADC_ScanConvMode = ENABLE;   //是否扫描模式：是
    ADC_Init(ADC1,&ADC_InitStructure);
    
    
    /*ADC使能之前 开始配置DMA*/
    //初始化DMA参数
    DMA_InitTypeDef DMA_InitStructure;
    DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;//源基地址
    DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;//半字，16位
    DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;//源地址是否自增：否
    DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)AD_Value;//目标基地址
    DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;//目标宽度 字节
    DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;//目的地否自增:是
    DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;//传输方向 1-存储器到外设。2-外设到 存储器
    DMA_InitStructure.DMA_BufferSize = 4;//传输计数器   ：4次
    DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;//指定重装器：循环
    DMA_InitStructure.DMA_M2M = DMA_M2M_Disable; //是否使用软件触发  否：
    DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;//选择优先级
    DMA_Init(DMA1_Channel1,&DMA_InitStructure);//硬件触发，ADC1 的硬件触发在通道1 所以不能变
    //使能DMA
    DMA_Cmd(DMA1_Channel1, ENABLE);
    
    //开启ADC到DMA的输出
    ADC_DMACmd(ADC1, ENABLE);

    //开启ADC1
    ADC_Cmd(ADC1, ENABLE);
    
    //校准  
    ADC_ResetCalibration(ADC1); //复位校准
    while(ADC_GetResetCalibrationStatus(ADC1) == SET) //等待复位校准完成
    ADC_StartCalibration(ADC1); //开始校准
    while(ADC_GetCalibrationStatus(ADC1) == SET); //等待校准完成
    
    //软件触发ADC
     ADC_SoftwareStartConvCmd(ADC1,ENABLE);
}
			```
		### AD.h代码如下 {toggle="true"}
			```javascript
#ifndef __AD_H
#define __AD_H

extern uint16_t AD_Value[4];

//ʼ
void AD_Init(void);




#endif

			```
		### main.c代码如下 {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"

#include "AD.h"
/*
	重复ADC扫描，  DMA连续搬运
	显示ADC转换值
	显示ADC转换值转换的电压
*/


int main()
{
    OLED_Init();//初始化OLED;
    Delay_Init();//初始化延时
    
    AD_Init();//开启AD连续转换，DMA连续搬运
    

    
    //显示通道
    OLED_ShowString(1,1,"AD1:");
    OLED_ShowString(2,1,"AD2:");
    OLED_ShowString(3,1,"AD3:");
    OLED_ShowString(4,1,"AD4:");
    OLED_ShowString(1,12,"0.00V");
    OLED_ShowString(2,12,"0.00V");
    OLED_ShowString(3,12,"0.00V");
    OLED_ShowString(4,12,"0.00V");
    
    //求电压
    float AD_Voltage1 = 0;
    float AD_Voltage2 = 0;
    float AD_Voltage3 = 0;
    float AD_Voltage4 = 0;
    
    while(1)
    {




    //显示搬运后的数据
    OLED_ShowNum(1, 5, AD_Value[0], 4);
    OLED_ShowNum(2, 5, AD_Value[1], 4);
    OLED_ShowNum(3, 5, AD_Value[2], 4);
    OLED_ShowNum(4, 5, AD_Value[3], 4);
    
    //显示转换的电压
    AD_Voltage1 = (float) AD_Value[0] / 4095 * 3.3;
    AD_Voltage2 = (float) AD_Value[1] / 4095 * 3.3;
    AD_Voltage3 = (float) AD_Value[2] / 4095 * 3.3;
    AD_Voltage4 = (float) AD_Value[3] / 4095 * 3.3;
    OLED_ShowNum(1,12,AD_Voltage1,1);
    OLED_ShowNum(1,14,(uint16_t)(AD_Voltage1 * 100 ) % 100, 2);
    OLED_ShowNum(2,12,AD_Voltage2,1);
    OLED_ShowNum(2,14,(uint16_t)(AD_Voltage2 * 100 ) % 100, 2);
    OLED_ShowNum(3,12,AD_Voltage3,1);
    OLED_ShowNum(3,14,(uint16_t)(AD_Voltage3 * 100 ) % 100, 2);
    OLED_ShowNum(4,12,AD_Voltage4,1);
    OLED_ShowNum(4,14,(uint16_t)(AD_Voltage4 * 100 ) % 100, 2);
    
    Delay_ms(20);
 
    }
}





			```
	<empty-block/>
# **20.   I2C通信详解** {toggle="true"}
	## I2C通信介绍 {toggle="true"}
		I2C总线 的英文全称是：Inter IC BUS
		他有两根通信线：SCL （Serial Clock）、SDA（Serial Data）<br>串行时钟线和串行数据线<br>支持总线挂载多设备。可以是一主多从和多主多从
		1. 一主多从的意思是单片机作为主机主导I2C总线的运行，其他挂载在I2C总线上的都是从机。从机只有被主机点名之后，才能控制I2C总线。就像老师点了你的名字之后你才能说话。
		2. 而多主多从模式是：每个模块都可以突然站出来，说接下来我是主机，你们听我的。 这时候如果有其他人说我也要当主机，  I2C也会有一个仲裁器来仲裁， 优先级高的胜利。失败的自动变为从机。
		**I2C通信属于 半双工  同步  多设备  带数据应答的 通信**
		而USART是   全双工   异步  点对点的通信
		- **半双工** 是可以双向传输，但每次只能有一个传输方向
		- **全双工** 是既可以双向传输，它又可以同时有两个传输方向。
		- 异步的好处是省下一根时钟线，节省资源。缺点是对时间要求严格，对硬件电路依赖严格。
		- 同步时钟的好处是。对时间要求不严格，对硬件电路不依赖。稳定性也比异步的好，容易使用软件来模拟时序，缺点是多一根时钟线。
		- 多设备则是在一根数据线上挂载很多的设备。在访问其中某个设备时。其他的设备对通信没有影响。
		> I2C通信可以理解外，通过一根时钟线和一根数据线。<br>来链接单片机外的外设。 并且可以**读写其中指定的寄存器**。实现这个外挂模块的完全控制
		比如MPU6050模块，可以进行姿态测量。 它就使用了I2C通信协议<br>或者OLED模块，可以显示字符、图片等信息。也使用了I2C通信协议<br>以及AT24C02存储器模块。也是I2C的通信协议
		或者DS3231 实时时钟模块。也是使用I2C通信。
		因为使用通用的协议。对于开发者来说是很方便的
	## **软件I2C和硬件I2C的区别** {toggle="true"}
		- 硬件电路件I2C的波形更加规整，时钟周期和占空比非常一致。
			每个时钟周期后都有严格的延时，保证每个周期的时间相同。
			硬件电路需要专门的某些GPIO口才能实现<br>
		- 软件I2C的波形较为不规整，每个时钟周期和空闲时间都不一致。
			软件I2C时的引脚操作会有一定的延时，因此各个时钟周期的间隔和占空比都不均匀。
			但是软件实现I2C更加的方便，随便那个GPIO口都可以实现。
			<empty-block/>
	## I2C硬件电路规定 {toggle="true"}
		1. 所有的I2C设备的SCL 均要连接在一起，SDA连接在一起
		2. 所有设备的SCL和SDA均要配置成开漏输出模式
		3. SCL和SDA各添加一个上拉电阻，阻值一般为4.7KΩ左右
		为什么呢？
		因为只有一根数据线，所以采用的是半双工的协议。
		在一主多从的模式下，主机的SCL线可以是推挽输出状态。来拥有对SCL的绝对控制权。<br>（但是仍然采用的是开漏输出+上拉电阻的模式。因为在多主机模式下会用到这个特性）<br>所有从机的SCL线都配置为 浮空输入 或者 上拉输入<br>数据的流向为 主机发送，所有的从机只能被动接收，<br>这样是没问题的
		**但SDA就不能这样了。**因为数据线是只有一条。<br>主机在接收的时候是输入，在发送的时候是输出。<br>同样，从机的SDA也会在输入和输出之间反复切换。<br>如果这时使用的是推挽输出模式下。 在总线没协调好两个设备的时序的情况下。就会出现两个设备都是输出的状态。如果此时一个输出高电平。另一个输出低电平。**就会出现电源对地短路的情况。**如图：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/889fd807-913a-4531-aa22-8b211b99c737.webp)
		这是应该极力避免的。
		所以，在I2C这里的解决办法是，**禁止所有设备为强制上拉的高电平<br>**采用了外置弱上拉电阻  加 开漏输出的电路结构。如图：
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/218972eb-5eb2-4115-820b-06cdee6ad93d.webp)
		在开漏输出模式下，每个引脚的内部结构如图：
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/474aa462-ed9f-4c7e-9f6b-7e2d3b3a4770.webp)
		可以看到。不管是SCL 还是SDA  <br>**在输入时** 都可以随便输入。对电路不会有影响
		**在输出高电平时 **只需要打开自己所控制的mos管，外部的上拉电阻会拉为高电平。
		**在输出低电平时 **只需要闭合自己所控制的mos管，外部的高电平会被强制拉低接地。
		所以，开漏输出不同于推挽输出的强上拉强下拉。
		开漏输出 配合 上拉电阻 是弱上拉、强下拉
		<empty-block/>
		所以，<span color="purple" underline="true">这个模式会有一个  “线与”  的现象。即只要有任意一个或多个设备输出了低电平，总线就处于低电平。只有所有的设备都输出高电平。总线才输出高电平</span>
		**所以，挂载在I2C总线上的 所有设备。 所有的GPIO 引脚 都要设置为 开漏输出的模式。<br>并且在SCL和 SDA两条线 添加上拉电阻**
	## I2C软件设计（时序基本单元） {toggle="true"}
		### 起始条件与终止条件 {toggle="true"}
			> 起始和终止，都是由主机产生的，从机不允许产生起始和终止
			在I2C总线处于空闲状态时。SCL和SDA都被上拉为高电平。
			当主机需要进行数据收发时，就会产生一个起始条件。<br>起始条件就是，时钟线不动，拉低SDA数据线。<br>当从机捕获到SCL时钟线为高电平。SDA数据线产生了下降沿信号时，就会进行自身的复位，等待主机点名。<br>然后主机会拉下SCL时钟线的电平。准备开始下一步动作。
			终止条件是，主机先对SCL时钟线放手，让上拉电阻把SCL拉高为高电平，再对SDA数据线放手，产生一个上升沿。这个上升沿就会触发终止条件
			此时就会恢复到最初的两个高电平的平静状态
			- 起始条件：SCL**高电平期间**，**SDA从高电平切换到低电平**
			- 终止条件：SCL**高电平期间**，**SDA从低电平切换到高电平**
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/44efd343-2122-4f7d-9a3c-937a633463c6.webp)
		### 主机发送一个字节 的时序单元 {toggle="true"}
			> 时钟线SCL低电平期间，主机将数据位依次放到SDA数据线线上**(高位先行**)，然后释放SCL，<br>从机将在SCL高电平期间读取数据位<br>**所以SCL高电平期间SDA不允许有数据变化，**<br>依次循环上述过程8次即可发送一个字节
			因为SCL同步时钟是主机在控制。从机并不知道什么时候时钟会消失。所以从机要尽快在SCL高电平期间读取主机所发送的电平信号（一般在主机松手的一刹那，上升沿来临时就已经读取了）。
			主机也应该在SCL时钟线下降沿上尽快把数据放到SDA数据线上。<br>但是主机有对时钟线的主导权。即使中途有什么事情打断了主机放数据的操作。主机也可以一直拉低时钟SCL。等到把数据放到SDA上之后再松开SCL 使他恢复高电平（这也是同步时序的好处）。
			在主机的主导的SCL时钟下。<br>SCL低电平，主机发送； 高电平，从机接收；<br>如此循环8次 就发送了8位数据，也就是一个字节
			在发送一个字节的时序中，主机完全掌控SCL和SDA，从机只能被动的读取。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1039b380-f0d8-4bbc-9ab7-8098d91e3ff9.webp)
			<empty-block/>
		### 主机接收一个字节 的时序单元 {toggle="true"}
			> 与发送一个字节同理。这里实线是主机控制，虚线是从机控制。<br>时钟线SCL低电平期间，<span underline="true">从机</span>将数据位依次放到SDA数据线线上**(高位先行**)，然后释放SCL，<br><span underline="true">主机</span>将在SCL高电平期间读取数据位<br>**所以SCL高电平期间SDA不允许有数据变化，**<br>依次循环上述过程8次即可读取一个字节<br><br>主机在接收之前，必须释放SDA（因为总线是线与的特征）
			如图：
			> 可以看到。主机在接收前，释放了SDA数据线。<br>这时数据线的主导权到了某个从机的手中。<br>但是SCL时钟线的主导权还是在主机这里。从机只能等待时钟为下降沿时被动发送。所以也是要尽快的发送，否则错过低电平的时期就不好啦（一般在下降沿来临后就已经发送了。贴着下降沿）。<br>主机则是在高电平任意时刻读取就行，反正主机能控制SCL时钟线
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/97e7a338-594c-41cb-9c7a-f2b2406afc0c.webp)
		### 主机/从机  应答 基本单元 {toggle="true"}
			- 发送应答：主机在接收完一个字节之后，在下一个时钟发送一位数据，数据0表示应答，数据1表示非应答在下一个时钟接收一位数
			- 接收应答：主机在发送完一个字节之后，判断从机是否应答，数据0表示应答，数据1表示非应答(主机在接收之前，需要释放SDA)
			> 可以看出，接收方必须在接收到一个字节之后，**在下一个时钟拉低SDA数据线**，如果没有拉低，那么发送方就会认为 这次发送失败，没有人接收到。
			如图：接收应答时，当主机释放SDA时，从机就应该立刻把SDA拉下来。在SCL高电平期间，主机读取应答位，如果应答位为0，就代表接收到。为1 代表没有接收到
			同理，主机在接受到一个字节的时候，也要发送应答。拉低电平，然后使SCL为高电平，等待从机读取。在高电平期间，从机如果收到了主机的应答，他就可以继续执行操作。如果主机没有应答，那么 从机就会乖乖释放SDA的主导权，防止干扰主机之后的工作
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/86639a1f-5917-4557-b6bb-0c22bb6b5726.webp)
		<empty-block/>
	## I2C完整时序（拼接基本单元） {toggle="true"}
		I2C的完整时序主要有3种
		- 指定地址写
		- 当前地址读
		- 指定地址读
		### 从机设备的地址 {toggle="true"}
			主机可以访问总线上的任何一个设备，要把每个从机都确定一个唯一的设备地址<br>从机设备地址，在I2C协议里分为：7位地址和 10位地址 （这里主要是讲7位地址）<br>在每一个I2C设备出厂时，厂商就会给他分配一个I2C 7 位地址（可以在芯片手册中查看）
			比如MPU6050的7位地址是1101 000<br>AT24C02的7位地址是1010 000<br>一般不同芯片的地址是不一样的。<br>相同芯片的地址是一样的。
			为了应对总线上挂载相同芯片的情况：一般器件地址的最后几位，是可以在电路中通过地址改变引脚的高低电平来改变的。（比如MPU6050的AD0引脚就可以决定地址。 低电平为1101 000 接高电平为 1101 001  ；AT24C02地址的最后三位，都可以由板子的A0 A1 A2来确定）
		### 指定地址写 {toggle="true"}
			- 对于**指定设备**(Slave Address)在**指定地址**(Reg Address)**下写入指定数据(**Data)
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f3110903-f452-417d-a543-a93b6f12aafb.webp)
			对于指定设备（从机地址），在指定地址（寄存器地址）下写入数据。
			**空闲状态**
			- 两条总线（SCL和SDA）都是高电平。
			**产生起始条件**
			1. 主机在SCL高电平期间拉低SDA，产生起始条件。
			2. 在起始条件之后，主机发送一个字节，该字节内容为从机地址和读写位（从机地址是7位，读写位是1位，总共8位）。
			**发送从机地址和读写位**
			1. 发送从机地址：确定通信的对象。
			2. 发送读写位：确认接下来是写入还是读出（0表示写入，1表示读出）。
			3. 接收应答位
			4. 主机发送从机地址和读写位后，释放SDA。
			5. 从机响应后会拉低SDA，产生应答位。
			6. 应答位产生后，从机释放SDA的控制权。
			**发送数据**
			1. 主机再次发送第二个字节，通常为寄存器地址或控制字。
			2. 接着发送第三个字节，表示要写入寄存器地址中的数据值。
			**产生停止条件**
			1. 主机拉低SDA，为后续的上升沿做准备。
			2. 然后依次释放SCL和SDA，产生SC**L高电平期间的SDA上升沿，**表示停止条件。
			**示例**
			对于从机地址为1101000的设备，在其内部0x19地址的寄存器中写入0xAA这个数据，数据帧的过程如下：
			1. 起始条件：SCL高电平期间，SDA从高电平拉低。
			2. 发送从机地址和读写位（1101000+0）。
			3. 接收从机应答位。
			4. 发送寄存器地址（0x19）。
			5. 接收从机应答位。
			6. 发送数据值（0xAA）。
			7. 接收从机应答位。
			8. 停止条件：SCL高电平期间，SDA从低电平释放至高电平。
			通过这些步骤，主机可以可靠地在从机的指定寄存器地址中写入数据
		### 当前地址读 {toggle="true"}
			对于指定设备（Slave Address），在**当前地址指针指示的地址下，**读取从机数据（Data）
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/bcd05d5e-988d-4d9e-afd6-4395cf889915.webp)
			**产生起始条件**
			- 主机在SCL高电平期间拉低SDA，产生起始条件。
			**寻址和读写操作**
			1. 主机首先发送一个字节，用于从机的寻址和设置读写标志位。
			2. 在本次通信中，主机发送的目标地址是1101000，读写标志位为1，表示主机接下来要读取数据。
			**接收从机应答位**
			1. 主机发送完寻址字节后，接收从机的应答位。
			2. 从机收到第一个字节后，拉低SDA，表示应答，并将SDA的控制权交回主机。
			**接收数据**
			1. 主机调用接收数据的时序，准备接收从机发送的数据。
			2. 从机得到允许后，在SCL低电平期间将数据写入SDA。
			3. 主机在SCL高电平期间读取SDA上的数据。
			4. 主机在SCL高电平期间依次读取8位数据，从而接收到从机发送的一个字节数据（例如：0000 1111，即0x0F）。
			**指针自动递增**
			1. I2C通信中没有指定寄存器地址的环节，默认情况下，从机中的所有寄存器被分配到一个线性区域中。
			2. 从机内部有一个指针变量，指向某个寄存器。上电时，这个指针通常默认指向**0**地址。
			3. 每次 写入 或 读出 一个字节后，这个指针会自动递增，指向下一个寄存器位置。
			4. 主机读取到的数据是当前指针指向的寄存器的值。
			> 所以一般这个时序用的不是很多
		### 指定地址读 {toggle="true"}
			对于指定设备（Slave Address），在指定地址（Reg Address）下，读取从机数据（Data）
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5f8c232b-2dd5-44ca-8a14-38b839fdadc2.webp)
			对弈指定地址读，其实是先告诉设备我要写，当寄存器指向这个地址之后，我又不写了。我要读。
			**写入操作**
			1. 指定从机地址和读写标志位：<br>从机地址为1101000，读写标志位为0，表示要进行写操作。
			2. 从机应答：<br>从机应答后，主机发送第二个字节（0001 1001），用于指定寄存器地址。<br>这个数据写入到从机的地址指针中，从机的寄存器指针指向0x19位置。
			**重复起始 开始读操作**
			1. 产生一个新的起始条件：<br>主机再次产生一个起始条件。
			2. 重新寻址并指定读写标志位：<br>主机重新发送从机地址（1101000）和读写标志位（此时读写标志位为1，表示读操作）。
			3. 主机接收数据：<br>从机应答后，主机开始接收一个字节的数据。<br>这个字节的数据是从机0x19地址下的内容。
		> 读写操作也不一定只能一次一个字节。<br>在主机读或写的时候，只要主机应答，从机就不会停。直到主机给非应答，从机才会停止，然后把SDA主导权交给主机。
# **21.   STM32通过I2C软件读写MPU6050** {toggle="true"}
	MPU6050是一个6轴姿态传感器，可以测量芯片自身X、Y、Z轴的加速度、角速度参数，通过数据融合，可进一步得到姿态角，常应用于平衡车、飞行器等需要检测自身姿态的场景
	**3轴加速度计**(Accelerometer)：测量X、Y、Z轴的加速度
	**3轴陀螺仪传感器**(Gyroscope)：测量X、Y、Z轴的角速度
	## 1.运动学概念 {toggle="true"}
		- 欧拉角：
			- 欧拉角是用来描述三维空间中刚体旋转的三个角度：俯仰角（Pitch）、滚转角（Roll）和偏航角（Yaw）。
			- **俯仰角（Pitch）**：飞机机头上下倾斜的角度。
			- **滚转角（Roll）**：飞机左右倾斜的角度。
			- **偏航角（Yaw）**：飞机左右转向的角度。
			> 任何一种传感器，都不能获得精确且稳定的欧拉角。要想获得，就需要进行数据融合，综合几种传感器的数据，取长补短。通常有互补滤波、卡尔曼滤波等
	## 2.MPU6050工作原理 {toggle="true"}
		1. 加速度计：<br>加速度计的内部可以想象为这个样子。是一个滑变电阻+两个弹簧
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a9c4f13f-9a5a-4580-b792-acb9a1ed1cfe.webp)
			1. 测量在X、Y、Z轴方向上测量加速度。通过检测重力加速度，可以推断出设备的倾斜角度。例如，在一个静止状态下，水平放置。 X和Y就是0。Z轴会受到一个向下重力所引的加速度；再例如，现在在自由落体。则X，Y，Z都为0。 
			2. <span underline="true">在使用加速度计求角度的时候</span>，只能是静态时才行。否则不行，<br>例如一个水平的小车，水平加速。如图，小人（芯片）收到的是向后和向下的力。<br>这时用三角函数求小车姿态，得到的就是小车停在一个斜坡上受到的力。
				> **总结就是：加速度具有静态稳定性，不具有动态稳定性**
			<columns>
				<column ratio="100">
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ffbe6e57-6b02-429c-a175-2b03db738bf0.webp)
				</column>
				<column ratio="100">
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f2278327-fdb6-490d-8108-aa27f4ca7f4e.webp)
				</column>
			</columns>
		2. 陀螺仪：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e2eb3ff6-a229-4cd6-be0d-c30c988139c1.webp)
			1. 测量设备在X、Y、Z轴上的角速度。陀螺仪可以用于检测设备的旋转运动，比如快速转动或者缓慢旋转等。
			2. <span underline="true">想要通过角速度的到角度</span>，只需要对角速度进行积分即可。但是陀螺仪测角速度也有局限性。 当物体静止时，角速度的值会因为噪声无法完全归零。经过积分的不断累积，角速度的值就会产生缓慢的漂移。
			> 总结就是：**陀螺仪具有动态稳定性，但不具备静态稳定性。**
		> 所以，在使用时，进行互补滤波，就能得到静态和动态都稳定的都稳定的姿态角了
	## 3.MPU6050参数 {toggle="true"}
		**ADC和数据存储**
		- 16位ADC：MPU6050集成了16位的ADC，用于将模拟信号转换为数字信号，量化范围为-32768到32767。
		- 量化过程：ADC将模拟信号转换为数字信号，并以**两个字节**进行存储。
		**加速度计满量程选择：**±2、±4、±8、±16（g）
			如果测量的物体的运动非常剧烈，就可以选择量程大一些。分辨率会更加粗糙
			如果测量的物体的运动比较平缓，就可以选择量程小一些。分辨率会更加细腻
		**仪满量程选择：** ±250、±500、±1000、±2000（°/sec）
			陀螺仪的选择也是同上，满量程越大，测量范围越广、满量程越小，测量分辨率就越高
		**可配置的数字低通滤波器**
		- 允许用户根据应用需求通报配置寄存器，来配置滤波器，滤除噪声和干扰。
		**可配置的时钟源、采样分频**
		- 支持多种时钟源，包括内部振荡器、外部参考时钟。经过采样分频之后为AD和其他内部电路提供时钟。控制了分频系数，就控制了AD转换的快慢了。
		**I2C从机地址：**
		当AD0引脚接低电平（AD0=0）：地址为1101 000 （0x68）。<br>当AD0引脚接高电平（AD0=1）： 地址为1101 001  （0x69）。<br>具体地址配置决定了在I2C总线上的唯一性，避免地址冲突。
		<empty-block/>
	## 4.MPU6050量程选择 {toggle="true"}
		**满量程选择**
		- **剧烈运动：**选择较大的满量程，确保测量范围足够大。
		- **平缓运动：**选择较小的满量程，提升测量分辨率。
		**加速度计满量程示例：**
		- **±16g：**
			- 读取的ADC值为最大值32768时，对应实际加速度为16g。
			- ADC值为32768的一半（16384）时，对应加速度为8g。
		- **±2g：**
			- 读取的ADC值为最大值32768时，对应实际加速度为2g。
			- ADC值为32768的一半（16384）时，对应加速度为1g。
		**测量分辨率：**
		- 满量程越小，测量分辨率越高，测量越精细。
		- 满量程越大，测量范围越广。
		- ADC值与加速度值呈**线性**关系，可以通过乘一个系数从ADC值计算出实际加速度。
	## **5.MPU6050从机地址配置** {toggle="true"}
		**二进制地址转换为十六进制：**
		- 以从机地址110 1000为例。
		- 把7位二进制数1101000转换为十六进制，即分割低4位和高3位：0110 1000，转换后为0x68。
		**I2C时序中的地址格式：**
		- 在I2C通信时，需要发送7位从机地址110 1000加上1位读写位。共八位
		- 使用0x68作为从机地址，需要将0x68左移1位，再加上读写位（0或1）。
		- 转换步骤：
			- 将0x68左移1位：1101 0000（即0xD0）。
			- 再与读写位（0或1）进行或操作：(0x68 \<\< 1) \| 0 或 (0x68 \<\< 1) \| 1
			- 再转换成二进制其实就是0xD0或0xD1
			- 这样其实就是**融入了读写位**的从机地址
		**实际应用：**
		写操作时 直接发送一个字节。从机地址为0xD0。<br>读操作时 直接发送一个字节。从机地址为0xD1。
	## 6.硬件电路 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9ce3469f-3ccd-4e2a-94b6-2b0195e188e9.webp)
		<table>
<tr>
<td>引脚</td>
<td>功能</td>
</tr>
<tr>
<td>VCC、GND</td>
<td>电源3.3v</td>
</tr>
<tr>
<td>SCL、SDA</td>
<td>I2C通信引脚</td>
</tr>
<tr>
<td>XCL、XDA</td>
<td>主机I2C通信引脚</td>
</tr>
<tr>
<td>AD0</td>
<td>从机地址最低位</td>
</tr>
<tr>
<td>INT</td>
<td>中断信号输出</td>
</tr>
		</table>
		<empty-block/>
	## 7.MPU6050框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/eb5ddcc3-1b79-417d-ae1d-f4ea400dca15.webp)
		左边一列是自测响应。用来验证芯片是否正常
		左下角的Charge Pump 是电荷泵，也叫做充电泵。是外接电容的。是用来升压的
		有各种传感器。甚至还有个温度传感器。
		这些东西测出来的值经过ADC进行模数转换。
		这些数据统一放到数据寄存器中。
		我们读取就OK了。其他的都是全自动执行的
		右边部分
		- Interrupt Status Register 中断控制寄存器，可以控制内部的事件到中断引脚的输出
		- FIFO  先入先出寄存器，可以对数据流进行缓存
		- Config Registors  配置寄存器，对内部的各个电路进行配置
		- Sensor Registers  传感器寄存器（数据寄存器），存储了各个传感器的数据
		- Factory Calibratilor   工厂校准，内部的传感器都进行了校准。
		- Digital Motion  Processor(DMP)  数字运动处理器，芯片内部自带的姿态解算的硬件算法。需要配合官方的DMP库，进行姿态解算。
		- FSYNC   帧同步，
		- 剩下的这些这些（上边仨）就是通信接口的部分了。<br>上边是MPU6050与STM32进行通信的接口。<br>下边是MPU6050作为主机的扩展接口
			- 其中SerialInterface  Bypass  Mux  是用来进行旁路控制的，<br>他可以控制 下边的接口是直接接到STM32的总线上（黄线所连接），作为STM32的直接大哥<br>或者是MPU6050自己去控制这些外设（蓝色线所连接）。使STM32是MPU6050的大哥。MPU6050是这些外设的大哥
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9f43a563-c977-4cd5-ac63-7ec5319379ff.webp)
	## 8.MPU60X0寄存器映射（常用） {toggle="true"}
		### 寄存器映射框图各部分作用介绍 {toggle="true"}
			每个寄存器都是八位的，他们有唯一的地址。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/15a07f8c-a46c-4f8b-8163-c78ec0682936.webp)
		### 常用寄存器简介 {toggle="true"}
			按照标黄部分从上往下介绍
			> 第一部分
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5401d6de-ac91-4ce1-937c-caed8e9a4fc6.webp)
			`SMPLRT_DIV ` 采样频率分频器
				- 陀螺仪刷新率（采样频率） =  时钟频率（内部晶振、陀螺仪晶振输出和或者外部时钟引脚方波）  /   （1+分频值）
				- 使用了滤波器，陀螺仪时钟就是1KHZ。 <br>不使用滤波器，陀螺仪时钟是8KHZ
			`CONFIG`     配置寄存器
				- （外部同步这里不用）
				- 低通滤波器可以让输出数据更加平滑，参数越大，输出都懂就越小<br>配置的参数可以是如下的0-7   （最后一位是保留位没用到）
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/b4fbccfa-1962-43ed-9cdc-ebb54be5326d.webp)
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/14066a78-d7a7-47bf-9a60-084d1ad4a2db.webp)
			`GYRO_CONFIG`   陀螺仪配置寄存器
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/0757cdf0-1106-4cef-aa85-4f976725ab02.webp)
				- 高三位是XYZ轴的自测使能位<br>自测响应  =  自测使能时数据  -   自测失能时数据<br>得到的值在范围内就算通过自测。
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3fbeab6d-d00e-401f-b8b5-d85c4fa28d30.webp)
				- 中间两位是满量程选择位<br>量程选择，则需要根据实际情况来。量程越大，量程越广。量程越小，分辨率越高
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4c4161d2-575a-4171-a290-00770ccae111.webp)
			`ACCEL_CONFIG `   加速度计配置寄存器
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/0cb1d19f-ca49-4c70-8881-c0706e7e902e.webp)
				- 同上，高三位为自测使能位
				- 中间两位是满量程选择位
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/66920408-8c83-40a9-8018-a43f8bf08ca7.webp)
				- 不过后面还多了三位，这三位是配置 高通滤波器的<br>在这里暂时用不到。（是内置小功能需要用到的。比如自由落体检测，运动检测、零运动检测等）对我们的输入输出没有影响。
			> 第二部分
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/babef8d9-594f-4e4a-9717-e5c62b7204b0.webp)
			这一大块，是数据寄存器
			其中_H 表示高八位     _L表示第八位
			**加速度计XYZ轴 **
				`ACCEL_XOUT_H`
				`ACCEL_XOUT_L`
				`ACCEL_YOUT_H`
				`ACCEL_YOUT_L`
				`ACCEL_ZOUT_H`
				`ACCEL_ZOUT_L`
				- 这是加速度计的数据寄存器<br>在读取数据时直接读取数据寄存器就OK了。<br>是一个16位的有符号数<br>以二进制补码的方式存储
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/40f49fbb-a5d2-43ca-8e49-deaf2c738c25.webp)
			---
			**温度传感器**
				`TEMP_OUT_H`
				`TEMP_OUT_L`
				- 同上
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/14103d76-d6ca-4885-83ab-88c3d99552e3.webp)
			---
			**陀螺仪XYZ轴**
				`GYRO_XOUT_H`
				`GYRO_XOUT_L`
				`GYRO_YOUT_H`
				`GYRO_YOUT_L`
				`GYRO_ZOUT_H`
				`GYRO_ZOUT_L`
				- 同上
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/582d1077-60e2-43af-821c-38434e175143.webp)
			---
			---
			> 第三部分
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/6c4bc6fe-446e-498f-85b4-351b641aab04.webp)
			`PWR_MGMT_1 `  电源管理寄存器 1
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5ce70025-e3a2-4a35-96a0-ca7b289c68e4.webp)
				- 第一位，设备复位 ：写1 时 所有设备恢复默认值
				- 第二位，睡眠模式：写1 时  芯片睡眠，进入低功耗。芯片不工作。
				- 第三位，循环模式：写1时  设备进入低功耗，过一段时间启动一次。<br>并且唤醒的频率由<span underline="true">电源管理寄存器2</span>决定。
				- 第四位，温度传感器失能：写1时  禁用内部温度传感器
				- 最后三位，用来选择系统时钟来源
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5c41e67c-20c7-4af8-b3c5-7ef47783c1b3.webp)
			---
			`PWR_MGMT_2`    电源管理寄存器 2 
				- 这两位决定了唤醒的频率
				- 后面6位可以分别控制6个轴进入待机模式<br>可以省电用。如果只需要部分数据的话
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/b05358ac-9133-4088-b093-dcced37403eb.webp)
			---
			`WHO_AM_I `     我是谁**（器件ID号） **
				- 这个寄存器是只读的。
				- 中间六位固定为 110100<br>所以读出这个寄存器的值就为0x68
				- 实际上ID号就是这个芯片的I2C地址。
				- 并且通过外部引脚AD0 修改设备号最低位的时候<br>是不反映在寄存器中的。寄存器并不会根据AD0的引脚的电平来变化。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/36de9045-34d8-4bc2-bf44-d48c4652f807.webp)
			---
			> 第四部分
			芯片上电之后，默认的是所有寄存器为0x00。
			除了两个：
			- Register 107:  0x40<br>也就是电源管理寄存器1。0x40即0100 0000 对应寄存器也就是睡眠模式为1<br>也就是**上电默认随眠**
			- Register 117: 0x68<br>这个就是ID号的寄存器了。
	## **9.软件I2C和硬件I2C的区别** {toggle="true"}
		- 硬件电路件I2C的波形更加规整，时钟周期和占空比非常一致。
			每个时钟周期后都有严格的延时，保证每个周期的时间相同。
			硬件电路需要专门的某些GPIO口才能实现<br>
		- 软件I2C的波形较为不规整，每个时钟周期和空闲时间都不一致。
			软件I2C时的引脚操作会有一定的延时，因此各个时钟周期的间隔和占空比都不均匀。
			但是软件实现I2C更加的方便，随便那个GPIO口都可以实现。
		> 这里我们使用的是软件I2C
	## 10.软件I2C读写MPU6050编写步骤 {toggle="true"}
		### 10.1程序整体构架 {toggle="true"}
			首先建立I2C通信层的.c和.h模块，再建立MPU6050.c, 最后是main.c
			- Main.c
				1. 调用MPU6050初始化函数。
				2. 循环读取数据并进行显示。
			- MPU6050.c
				1. 基于I2C通信协议，实现指定地址读，指定地址写
				2. 实现写寄存器来配置芯片，读寄存器得到传感器数据。
			- I2C.c
				1. 初始化GPIO。
				2. 编写基本的I2C操作函数，包括起始条件、终止条件、发送/接收一个字节、发送/接收应答等。
		### 10.2需要注意的点 {toggle="true"}
			1. **开漏输出模式**
				- 开漏输出模式下GPIO有强下拉，弱上拉的特性，所以需要借助外部上拉电阻来输出高电平。
				- 当Out输出为1时，晶体管截止。此时可以直接读取输入数据寄存器，就可以得到开漏模式下输出的值。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/474aa462-ed9f-4c7e-9f6b-7e2d3b3a4770.webp)
			2. **写起始条件时**
				在SCL高电平期间，SDA由高电平变为低电平，产生起始条件（Start Condition）。这表示一次I2C通信的开始。<br>（先拉低SDA再拉低SDA）
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/0b45f84c-922c-45c3-8cab-d804f211dba6.webp)
				- 起始条件不仅要能在SDA和SCL**都为高电平时**按照先SDA再SCL顺序拉低。他们谁先拉高都是无所谓的。
				- 也要兼容在重复起始条件。**在重复起始条件时**。 刚刚完成了应答的操作，<br>SCL将要到拉低为低电平，而SDA的电平不敢确定。所以为了保险起见，需要趁SCL为低电平时 抓紧上拉SDA，再 上拉SCL才能**保证不触发终止**。举个栗子如下：
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/2c9d1636-1116-42ec-90c8-feeb52dbc9ac.webp)
				> **所以综上来说，在写起始条件时。顺序应该为SDA 1、SCL 1、SDA 0、 SCL 0**
			3. **写终止条件时**
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/036c024f-f013-4e4f-9f6e-67e7fb17c057.webp)
				- 同上， 在上次应答完成之后。**SCL下一个周期一定为下降沿。**而SDA的状态不确定。<br>所以为了确保在终止条件时，SDA一定能在SCL为高时产生上升沿，<br>**需要先趁着SCL为低电平把SDA拉低，再把SCL拉高。   再拉高SDA。 **<br>这样就能产生在SCl为高电平时的上升沿了 
			4. **写发送字节时**
				- 发送字节是**高位先行**
				- 起始条件之后，SCL和SDA一定为低电平
				- 除了起始条件和终止条件，其他时候都是SDA再SCL低电平时变化
				- 取出一个字节（8bit）的任一位。可以使用** &** 操作，比如<br>1011 0000 & 0x40 = 1011 0000 & 0100 0000 = 0000 0000（0）<br>1111 0000 & 0x40 = 1011 0000 & 0100 0000 = 0100 0000（非0即1）<br>所以发送的电平一定是0 或 1
				- 循环8次 发送一个字节
			5. **写接收字节时**
				- 首先主机要释放SDA的控制权。也就是输出1，使mos管截止
				- 从机在SCL低电平期间 控制SDA 拉高或拉低
				- 主机控制SCL回到高电平。主机读取SDA
				- 拉低SCL，从机控制SDA。
				- 循环八次..接收一个字节
		### 10.3软件I2C编写步骤 {toggle="true"}
			1. 将打算用作SCL和SDA的GPIO引脚都初始化为开漏输出模式
			2. 将SCL和SDA都置高电平
			3. 写 写SCL函数（可选择加延时）；
			4. 写 写SDA函数（可选择加延时）；
			5. 写 读SDA函数（可选择加延时）；
			6. 写起始条件函数：<br>拉高SDA、拉高SCL、拉低SDA、拉低SCL
			7. 写终止条件函数：<br>拉低SDA、拉高SCL、拉高SDA、
			8. 发送的一个字节函数<br>发送一个字节的**最高位**；<br>拉高SCL（从机读取）<br>拉低SCL（准备下次写入）<br>发送一个字节的**次高位**；<br>拉高SCL（从机读取）<br>拉低SCL（准备下次写入）
				…循环八次。可使用for循环
			9. 接收一个字节的函数<br>主机释放SDA（SDA输出1，SDA对 地 相当于断开）（这时候SCL为低，从机已经在SDA存放了数据）<br>主机拉高SCL（准备读取）<br>主机输入并存入数据（可以用或来存入）<br>主机拉低SCL（从机SDA存放数据）<br>主机拉高SCL（准备读取）<br>主机输入并存入数据（可以用或来存入）<br>…..循环<br>返回循环后接收到的一个字节
			10. 发送应答的函数<br>（发送一个字节的简化版）<br>发送应答<br>SCL高电平（从机读取应答）<br>SCL低电平（进入下一个时序单元）
			11. 接收应答的函数<br>主机释放SDA（SDA输出1，SDA对 地 相当于断开）（这时候SCL为低，从机已经在SDA存放了数据）<br>主机拉高SCL（准备读取）<br>主机输入并存入数据（可以用或来存入）<br>主机拉低SCL（从机SDA存放数据）<br>返回读出的值
		### 10.4软件I2C读写MPU6050编写步骤 {toggle="true"}
			有了软件I2C的初始化，写MPU就很方便了。
			只需要调用已经写好的起始、结束、发送字节、接受字节的时序就可以。
			1. 初始化MPU6050<br>包括调用I2C初始化函数。解除睡眠模式、设置采样频率、配置陀螺仪和加速度计等寄存器
			2. 编写“指定地址**写**一个字节“的函数
				- 起始
				- 发送字节（设备地址+0）（我要写入）
				- 接收从机应答
				- 发送字节（指定要写入的地址）
				- 接收从机应答
				- 发送字节（发送要写入的数据）
				- 接收从机应答
				- 停止
				其中接受从机应答可以去判断，并输出对应的报错信息。这里没有添加。
			3. 编写“指定地址**读**一个字节“的函数
				- 起始
				- 发送字节（设备地址+0）（我要写入）
				- 接收从机应答
				- 发送字节（指定要写入的地址）
				- 接收应答位
				- 重复起始
				- 发送字节（设备地址+1）我要读取
				- 接收应答位
				- 接收并保存从机发送的字节
				- 主机不应答（如果应答，从机会继续发送）
				- 停止
				- 返回读取到的值
			4. 编写“读取六轴传感器各个寄存器的值”
				可以直接传地址，也可以用结构体或者数组。方法很多。
	## 11.代码编写： {toggle="true"}
		### **程序文件简要说明：** {toggle="true"}
			- MyI2C.c：初始化I2C所需要的引脚、编写时序的基本组成（起始、终止、发送/接收一个字节、应答、发送应答）
			- MyI2C.h：函数声明
			- MPU6050.c：初始化MPU6050的寄存器。在MyI2C的基础上，编写对指定地址发送指定值、读取指定地址的函数
			- MPU6050.h：函数声明、数据结构体声明
			- MPU6050_Reg.h：寄存器的地址与名称声明
			- main.c：测试I2C通信结果。
		### MyI2C.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "Delay.h"

#define SCL_Clock   RCC_APB2Periph_GPIOA
#define SCL_PORT    GPIOA
#define SCL_Pin     GPIO_Pin_11

#define SDA_Clock   RCC_APB2Periph_GPIOA
#define SDA_PORT    GPIOA
#define SDA_Pin     GPIO_Pin_12


/**
  * 函    数：初始化I2C
  * 参    数：无
  * 返 回 值：无
  */
void MyI2C_Init(void)
{
    Delay_Init();//初始化延时函数
    
    /*开启SCL和SDA时钟*/
    RCC_APB2PeriphClockCmd(SCL_Clock,ENABLE);
    RCC_APB2PeriphClockCmd(SDA_Clock,ENABLE);
    
    /*初始化SCL和SDA引脚*/
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_OD;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_InitStructure.GPIO_Pin = SCL_Pin;
    GPIO_Init(SCL_PORT,&GPIO_InitStructure);   //SCL配置为开漏输出
    
    GPIO_InitStructure.GPIO_Pin = SDA_Pin;
    GPIO_Init(SDA_PORT,&GPIO_InitStructure);   //SDA配置为开漏输出
    
    GPIO_SetBits(SCL_PORT,SCL_Pin);//拉高SCL
    GPIO_SetBits(SDA_PORT,SDA_Pin);//拉高SDA
}

/**
  * 函    数：写一位SCL
  * 参    数：0或1
  * 返 回 值：无
  */
void MyI2C_W_SCL (uint8_t BitValue)
{
    GPIO_WriteBit(SCL_PORT,SCL_Pin,(BitAction)BitValue);//BitAction是一个typedef的枚举类型值。
    Delay_us(10);   //延时，给从机读取时间
}

/**
  * 函    数：写一位SDA
  * 参    数：0或1
  * 返 回 值：无
  */
void MyI2C_W_SDA (uint8_t BitValue)
{
    GPIO_WriteBit(SDA_PORT,SDA_Pin,(BitAction)BitValue);
    Delay_us(10);   //延时，给从机读取时间
}

/**
  * 函    数：读一位SDA
  * 参    数：无
  * 返 回 值：BitValue
  */
uint8_t MyI2C_R_SDA (void)
{
    uint8_t BitValue = 0x00;   
    BitValue = GPIO_ReadInputDataBit(SDA_PORT,SDA_Pin);
    Delay_us(10);
    return BitValue;
}

/**
  * 函    数：起始时序单元
  * 参    数：无
  * 返 回 值：无
  */
void MyI2C_Start (void)
{
    MyI2C_W_SDA(1);
    MyI2C_W_SCL(1);//按照先SDA拉高再SCL拉高。防止在重复起始时触发终止条件
    MyI2C_W_SDA(0);
    MyI2C_W_SCL(0);//先SDA拉低，再SCL拉低。触发起始条件
}

/**
  * 函    数：结束时序单元
  * 参    数：无
  * 返 回 值：无
  */
void MyI2C_Stop (void)
{
    MyI2C_W_SDA(0);//在SCL为低电平时，先把SDA拉高，为后续做准备
    MyI2C_W_SCL(1);
    MyI2C_W_SDA(1);//先SCL拉高，再SDA拉高。触发结束条件
}

/**
  * 函    数：发送一个字节
  * 参    数：Byte
  * 返 回 值：无
  */
void MyI2C_SendByte (uint8_t Byte)
{
    uint8_t i = 0;
    for(i = 0; i < 8; i++)
    {
        MyI2C_W_SDA( Byte & (0x80 >> i) );//按照最高位依次往右取出bit写入
        MyI2C_W_SCL(1);//从机读取
        MyI2C_W_SCL(0);//准备下次写入
    }
}

/**
  * 函    数：接收一个字节
  * 参    数：无
  * 返 回 值：Byte
  */
uint8_t MyI2C_ReceiveByte (void)
{
    MyI2C_W_SDA(1);//释放SDA控制
    uint8_t Byte = 0x00;//存放得到的bit
    
    uint8_t i = 0;
    for(i = 0; i < 8; i++)
    {
        MyI2C_W_SCL(1);//主机准备读取
        if (MyI2C_R_SDA() == 1)//如果读到的这一位为1
        {
            Byte |= (0x80 >> i);//就把这一位置1，否则默认为0
        }
        MyI2C_W_SCL(0);//从机放下一位bit的数据
    }
    return Byte;
}

/**
  * 函    数：发送应答
  * 参    数：AckBit
  * 返 回 值：无
  */
void MyI2C_SenndAck (uint8_t AckBit)
{
    
        MyI2C_W_SDA(AckBit);//应答
        MyI2C_W_SCL(1);//从机读取
        MyI2C_W_SCL(0);//准备下次动作
}

/**
  * 函    数：接收应答
  * 参    数：无
  * 返 回 值：reply
  */
uint8_t MyI2C_ReceiveAck (void)
{
    MyI2C_W_SDA(1);//释放SDA控制
    uint8_t AckBit = 0x00;//存放得到的bit

    MyI2C_W_SCL(1);//从机填写
    AckBit = MyI2C_R_SDA();//主机读取
    MyI2C_W_SCL(0);//准备下次动作
    return AckBit;
}













			```
		### MyI2C.h {toggle="true"}
			```javascript
#ifndef __MYI2C_H
#define __MYI2C_H

//函    数：初始化I2C
void MyI2C_Init(void);

//函    数：起始时序单元
void MyI2C_Start (void);
    
//函    数：结束时序单元
void MyI2C_Stop (void);
    
//函    数：发送一个字节
void MyI2C_SendByte (uint8_t Byte);
    
//函    数：接收一个字节
uint8_t MyI2C_ReceiveByte (void);
    
//函    数：发送应答
void MyI2C_SenndAck (uint8_t AckBit);
    
//函    数：接收应答
uint8_t MyI2C_ReceiveAck (void);


#endif

			```
		### MPU6050.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "MPU6050.h"
#include "MyI2C.h"

#include "MPU6050_Reg.h"

#define MPU6050ADDRESS      0xD0




/**
  * 函    数：指定地址写一个字节
  * 参    数：RegAddress   指定的寄存器地址
              Data         指定写入的数据
  * 返 回 值：无
  */
void MPU6050_WriteReg(uint8_t RegAddress, uint8_t Data)
{
    MyI2C_Start();//起始
    MyI2C_SendByte(MPU6050ADDRESS);//指定设备地址加写操作
    MyI2C_ReceiveAck();//接收应答位
    MyI2C_SendByte(RegAddress);//指定寄存器地址
    MyI2C_ReceiveAck();//接收应答位
    MyI2C_SendByte(Data);//指定写入指定的数据
    MyI2C_ReceiveAck();//接收应答位
    MyI2C_Stop();//停止
}

/**
  * 函    数：指定地址读一个字节
  * 参    数：RegAddress   指定要读的寄存器地址
  * 返 回 值：无
  */
uint8_t MPU6050_ReadeReg(uint8_t RegAddress)
{
    uint8_t Data = 0x00;
    
    MyI2C_Start();//起始
    MyI2C_SendByte(MPU6050ADDRESS);//指定设备地址加写操作
    MyI2C_ReceiveAck();//接收应答位
    MyI2C_SendByte(RegAddress);//指定寄存器地址
    MyI2C_ReceiveAck();//接收应答位
    
    MyI2C_Start();//重复起始
    MyI2C_SendByte(MPU6050ADDRESS | 0x01);//指定设备地址 加 读操作
    MyI2C_ReceiveAck();//接收应答位
    Data = MyI2C_ReceiveByte();//接收从机发送的字节
    MyI2C_SenndAck(1);//主机发送应答位（应答会继续读）
    MyI2C_Stop();//停止
    return Data;//返回读取到的值
}

/**
  * 函    数：初始化MPU6050
  * 参    数：无
  * 返 回 值：无
  */
void MPU6050_Init(void)
{
    /*初始化I2C*/
    MyI2C_Init();
    /*初始化MPU6050*/
    MPU6050_WriteReg(MPU6050_PWR_MGMT_1,0x01);  //电源管理1：不复位、解除睡眠、不循环、温度传感器不失能、选择X轴陀螺仪时钟
    MPU6050_WriteReg(MPU6050_PWR_MGMT_2,0x00);  //电源管理2：不需要循环模式唤醒频率、每个轴都不需要待机
    
    MPU6050_WriteReg(MPU6050_SMPLRT_DIV,0x09);  //采样率分频：数据输出的快慢，越小输出越快.这里给10分频
    MPU6050_WriteReg(MPU6050_CONFIG,0x06);  //配置寄存器：外部同步不需要、数字低通滤波器设置为110
    MPU6050_WriteReg(MPU6050_GYRO_CONFIG,0x18);   //陀螺仪配置寄存器：不自测、满量程选择：11最大量程
    MPU6050_WriteReg(MPU6050_ACCEL_CONFIG,0x18);    //加速度计配置寄存器：不自测、满量程选择：11最大量程、不用高通滤波器
}


/**
  * 函    数：得到六轴传感器中的数据
  * 参    数：Str   MPU6050_Data的地址
  * 返 回 值：无
  */
SensorData MPU6050_Data;
void MPU6050_GetData(SensorData *Str)//存放这种结构体类型的地址
{
    Str->AccX = ( MPU6050_ReadeReg(MPU6050_ACCEL_XOUT_H) <<8|
                  MPU6050_ReadeReg(MPU6050_ACCEL_XOUT_L));
    Str->AccY = ( MPU6050_ReadeReg(MPU6050_ACCEL_YOUT_H) <<8|
                  MPU6050_ReadeReg(MPU6050_ACCEL_YOUT_L));
    Str->AccZ = ( MPU6050_ReadeReg(MPU6050_ACCEL_ZOUT_H) <<8|
                  MPU6050_ReadeReg(MPU6050_ACCEL_ZOUT_L));
    Str->Temp = ( MPU6050_ReadeReg(MPU6050_TEMP_OUT_H)   <<8|
                  MPU6050_ReadeReg(MPU6050_TEMP_OUT_L));
    Str->GyroX = ( MPU6050_ReadeReg(MPU6050_GYRO_XOUT_H) <<8 |
                  MPU6050_ReadeReg(MPU6050_GYRO_XOUT_L));
    Str->GyroY = ( MPU6050_ReadeReg(MPU6050_GYRO_YOUT_H) <<8 |
                  MPU6050_ReadeReg(MPU6050_GYRO_YOUT_L));
    Str->GyroZ = ( MPU6050_ReadeReg(MPU6050_GYRO_ZOUT_H) <<8 |
                   MPU6050_ReadeReg(MPU6050_GYRO_ZOUT_L));
}
//MPU6050_ACCEL_XOUT_H    0x3B
//MPU6050_ACCEL_XOUT_L    0x3C
//MPU6050_ACCEL_YOUT_H    0x3D
//MPU6050_ACCEL_YOUT_L    0x3E
//MPU6050_ACCEL_ZOUT_H    0x3F
//MPU6050_ACCEL_ZOUT_L    0x40
//MPU6050_TEMP_OUT_H      0x41
//MPU6050_TEMP_OUT_L      0x42
//MPU6050_GYRO_XOUT_H     0x43
//MPU6050_GYRO_XOUT_L     0x44
//MPU6050_GYRO_YOUT_H     0x45
//MPU6050_GYRO_YOUT_L     0x46
//MPU6050_GYRO_ZOUT_H     0x47
//MPU6050_GYRO_ZOUT_L     0x48

			```
		### MPU6050.h {toggle="true"}
			```javascript
#ifndef __MPU6050_H
#define __MPU6050_H

/**
  * 函    数：初始化MPU6050
  * 参    数：无
  * 返 回 值：无
  */
void MPU6050_Init(void);

/**
  * 函    数：指定地址写一个字节
  * 参    数：RegAddress   指定的寄存器地址
              Data         指定写入的数据
  * 返 回 值：无
  */
void MPU6050_WriteReg(uint8_t RegAddress, uint8_t Data);
/**
  * 函    数：指定地址读一个字节
  * 参    数：RegAddress   指定要读的寄存器地址
  * 返 回 值：无
  */
uint8_t MPU6050_ReadeReg(uint8_t RegAddress);


//传感器数据
typedef struct Data 
{
    int16_t AccX;
    int16_t AccY;
    int16_t AccZ;
    int16_t Temp;
    int16_t GyroX;
    int16_t GyroY;
    int16_t GyroZ;
}SensorData;
extern SensorData MPU6050_Data;

/**
  * 函    数：得到六轴传感器中的数据
  * 参    数：Str   MPU6050_Data的地址
  * 返 回 值：无
  */
void MPU6050_GetData(SensorData *Str);

#endif

			```
		### MPU6050_Reg.h {toggle="true"}
			```javascript
#ifndef __MPU6050_REG_H
#define __MPU6050_REG_H

#define MPU6050_SMPLRT_DIV      0x19    //采样率分频
#define MPU6050_CONFIG          0x1A    //配置寄存器
#define MPU6050_GYRO_CONFIG     0x1B    //陀螺仪配置寄存器
#define MPU6050_ACCEL_CONFIG    0x1C    //加速度计配置寄存器
        
#define MPU6050_ACCEL_XOUT_H    0x3B
#define MPU6050_ACCEL_XOUT_L    0x3C
#define MPU6050_ACCEL_YOUT_H    0x3D
#define MPU6050_ACCEL_YOUT_L    0x3E
#define MPU6050_ACCEL_ZOUT_H    0x3F
#define MPU6050_ACCEL_ZOUT_L    0x40
#define MPU6050_TEMP_OUT_H      0x41
#define MPU6050_TEMP_OUT_L      0x42
#define MPU6050_GYRO_XOUT_H	    0x43
#define MPU6050_GYRO_XOUT_L     0x44
#define MPU6050_GYRO_YOUT_H     0x45
#define MPU6050_GYRO_YOUT_L     0x46
#define MPU6050_GYRO_ZOUT_H     0x47
#define MPU6050_GYRO_ZOUT_L     0x48
    
#define MPU6050_PWR_MGMT_1      0x6B    //电源管理1
#define MPU6050_PWR_MGMT_2      0x6C    //电源管理2
#define MPU6050_WHO_AM_I        0x75    //设备号

#endif

			```
		### main.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
#include "MPU6050.h"
/*
    指定6050设备，读取117位寄存器（设备号）
    在OLED显示出来
*/



int main()
{
    OLED_Init();//初始化OLED;
    MPU6050_Init();//初始化MPU6050
    
    
    while(1)    
    {
         MPU6050_GetData(&MPU6050_Data);
        //显示加速度
        OLED_ShowSignedNum(1,1,(int16_t)MPU6050_Data.AccX,5);
        OLED_ShowSignedNum(2,1,(int16_t)MPU6050_Data.AccY,5);
        OLED_ShowSignedNum(3,1,(int16_t)MPU6050_Data.AccZ,5);
        //显示陀螺仪
        OLED_ShowSignedNum(1,8,MPU6050_Data.GyroX,5);
        OLED_ShowSignedNum(2,8,MPU6050_Data.GyroY,5);
        OLED_ShowSignedNum(3,8,MPU6050_Data.GyroZ,5);
        //显示温度
        OLED_ShowSignedNum(4,4,MPU6050_Data.Temp,5);
        
        
    }
    
}





/*测试读写寄存器*/
//#include "stm32f10x.h"                  // Device header
//#include "oled.h"
//#include "Delay.h"
//#include "key.h"
//#include "MPU6050.h"
///*
//    指定6050设备，读取117位寄存器（设备号）
//    写入寄存器，并读取出来。检查是否正确写入
//    在OLED显示出来
//*/

//int main()
//{
//    OLED_Init();//初始化OLED;
//    MPU6050_Init();//初始化MPU6050
//    /*读取0x75寄存器设备号并显示*/
//    uint8_t ID = MPU6050_ReadeReg(0x75);
//    OLED_ShowHexNum(1, 1, ID, 2);
//    /*解除芯片睡眠模式，并测试写入*/
//    MPU6050_WriteReg(0x6B,0x00);//解除
//    MPU6050_WriteReg(0x19,0xAA);//写入
//    uint8_t Data = MPU6050_ReadeReg(0x19);//读出
//    OLED_ShowHexNum(2, 1, Data, 2);//显示
//    MPU6050_WriteReg(0x19,0x66);//写入
//    Data = MPU6050_ReadeReg(0x19);//读出
//    OLED_ShowHexNum(3, 1, Data, 2);//显示
//}

/*测试I2C时序*/
//#include "stm32f10x.h"                  // Device header
//#include "oled.h"
//#include "Delay.h"
//#include "key.h"
//#include "MyI2C.h"
///*
//    查询所有7位I2C地址
//    找到总线上挂载的所有设备
//    在OLED显示出来
//*/

//int main()
//{
//    OLED_Init();//初始化OLED;
//    MyI2C_Init();//初始化I2C
//    
//    /*
//        从机地址为 1101 000 
//        左移后补0 = 1101 0001 = 0xD0（表示指定这个设备写）
//    */
//    uint8_t Row = 1;
//    uint8_t i = 0x00;
//    for (i = 0x00; i < 0x7F; i++)
//    {
//        MyI2C_Start();//起始时序
//        MyI2C_SendByte((i << 1) | 0);//发送字节。写模式
//        uint8_t Ack = MyI2C_ReceiveAck();//接收应答
//        MyI2C_Stop();//结束时序
//        if(Ack == 0)
//        {
//            OLED_ShowHexNum(Row,1,i,2);
//            OLED_ShowBinNum(Row,4,i,7);
//            Row++;
//        }
//    }
//}




			```
	## 12.结果 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/67f79b8b-9761-46b8-b56f-a811ba820033.webp)
	## 13.数据验证 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9bcd1fa9-bd90-43a0-bdea-c2a95b9ef8c5.webp)
		- 陀螺仪旋转检测
			- 陀螺仪绕Z轴旋转，陀螺仪Z轴会输出对应的角速度。
			- 图示中，三维空间的坐标轴X、Y、Z对应陀螺仪的三个方向。
			- 通过陀螺仪的测量，可以获得绕某一轴的旋转角速度信息，帮助理解物体的旋转状态。
		- 加速度计检测
			- 在正方体中放置一个小球，小球压在哪个面上就产生对应轴的输出。
			- 当前芯片水平放置，对应正方体的X轴、Y轴数据基本为0。
			- 小球在底面上，产生1个g的重力加速度，这里显示的数据是1943。
			- 1943代表Z轴方向的支持力，所以Z轴加速度为正。
		- 数据计算
			- 根据测量值1943和满量程32768（16位ADC），计算得出加速度的实际值。
			- 根据测量值1943和满量程32768（16位ADC），计算得出加速度的实际值。
			- 公式： 1943/32768 = Z/16g
			- 所以Z轴的加速度为0.95g。
		- 测量值比例公式
			- 读到的ADC值与满量程值之间的比例关系。
			- 公式： 读到的数据/32768 = X/满量程 （其中，满量程在16位系统中为-32768到32767)
# **22.   STM32通过I2C硬件读写MPU6050** {toggle="true"}
	## 1. STM32的I2C外设简介 {toggle="true"}
		> 硬件I2C具有高速传输、低占用率和稳定性高的优点，适用于对传输速度和稳定性要求较高的场景；<br>而软件I2C具有灵活性高和可移植性强的特点，适用于没有硬件I2C支持或需要扩展硬件I2C功能的场景
		- STM32内部集成了硬件12C收发电路，可以由**硬件自动**执行时钟生成、起始终止条件生成、应答位收发、数据收发等功能，减轻CPU的负担
		- 支持多主机模型
		- 支持7位/10位地址模式
		- 支持不同的通讯速度，标准速度(高达100 kHz)，快速(高达400 kHz)
		- 支持DMA
		- 兼容SMBus协议（System Management Bus 系统管理总线）<br>基于I2C改进而来的，主要用于电源管理系统
		- STM32F103C8T6 硬件I2C资源：**12C1、12C2**
	## 2. STM32的I2C基本框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8bf9832e-ae38-4721-a97d-ff5b519b9d73.webp)
	## 3. STIM32硬件I2C主机发送流程 {toggle="true"}
		### 10位地址与7位地址的区别 {toggle="true"}
			- 7位地址
				- 起始条件后的第一个字节是寻址（地址+读写位）
			- 10位地址
				- 起始条件后的两个字节都是寻址（11110+2位地址+读写位+8位地址）
			<empty-block/>
		### 7位主机发送的时序流程 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a3ad3bfc-c3d6-46d6-9822-ec0824a459f8.webp)
			- 起始、从机地址+读写位、应答、数据1、应答、数据2、应答、….数据N、应答、停止 
			1. 起始<br>STM32默认为从模式，所以需要下在START寄存器下写1就产生起始条件，由硬件自动清除。之后STM32由从模式转为主模式
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8d8b0d5c-fd2a-412e-84b6-b6a1a18eb73e.webp)
			2. 发生EV5事件<br>可以把他当做大标志位。因为有的状态会产生多个标志位。所以EVx事件就是组合了很多标志位的一个大标志位<br>**EV5事件**，就是SB标志为1 。SB是状态寄存器的一个位。表示了硬件的状态(在状态寄存器SR1中可以找到这一位，置1代表起始条件已发送。由硬件自动清除)
			3. 发送一个字节<br>需要写入DR寄存器。硬件电路会自动把DR寄存器的值转入移位寄存器。再把这一个字节发送到I2C总线上。
			4. 应答<br>电路会自动判断。如果没有应答，则会置应答失败标志位。这个标志位可以申请中断。在寻址完成之后
			5. 会发生EV6事件，也就是ADDR为1。手册中查看到的意思是：<br>在主模式下，代表地址发送结束
			6. 发生EV8_1事件<br>此时 TXE = 1，移位寄存器为空、数据寄存器为空<br>这时需要我们写入数据寄存器DR进行数据发送了。
			7. 发生EV8事件<br>DR在填写数据之后，会立刻把数据转移到移位寄存器进行发送。<br>（也就是填写了数据1）<br>这时就是EV8事件：**移位寄存器非空、数据寄存器空。**
			8. 此时再写入DR<br>把数据放到数据寄存器中。同时因写入DR，导致EV8事件清除。<br>（也就是说已经写入了下一个数据：数据2）（在没有应答之前写入）
			9. 接收应答位<br>当从机接收到应答位之后，数据2就转入移位寄存器进行发送。
			10. 发送后又产生了EV8事件<br>**移位寄存器非空、数据寄存器空。**
			11. 此时再写入DR……等待应答….转入移位寄存器发送…产生EV8事件….写入….<br>（在没有应答之前写入）
			12. 直到数据发完，触发EV8_2<br>当**移位寄存器空、数据寄存器也空**时。就会触发EV8_2<br>EV8_2  是 EXT = 1  （数据寄存器为空 ）、<br> BTF = 1（字节发送结束标志位 ）（也就是移位寄存器移位完成后找数据寄存器要下一个数据时发现数据寄存器没数据时的标志位）
			13. 停止<br>（控制寄存器CR1中）<br>STOP、这一位写1的时候，就会在当前字节或在起始条件产生停止条件
			<empty-block/>
		### 7位主机接收的时序流程 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a4a643cf-aa25-4bef-b5a3-d0d028b62c3f.webp)
			7位地址主机接收流程：
				- 起始、从机地址+读、接收、接收数据、发送应答、接收数据、发送应答…接收数据、非应答、停止
			10位地址主机接收流程
				- 起始、发送帧头（11110 + 两位地址 + 0（写））、发送第二个字节的8位地址、重复起始条件、发送枕头（11110 + 两位地址 + 1（写））、**直接接收数据**、发送应答、接收…发送…接收、非应答、停止（没有发送第二次的字节）
			7位详解：
			1. 写入控制位的Start位、产生起始条件
			2. 等待EV5事件<br>表示已发送
			3. 发送地址（地址+读）
			4. 接收应答（ACK，写1  在接受一个字节后就返回一个应答）
			5. 产生EV6事件，<br>代表寻址已完成
			6. 产生EV6_1时间，<br>没有对应的标志事件。
			7. 接收数据1<br>代表数据正在通过移位寄存器输入
			8. 发送应答<br>硬件自动执行，需要配置是否应答。CR1寄存器中，ACK写1  应答， 否则不应答
			9. 产生EV7事件<br>因为此时就代表已经接收到数据了。<br>数据会通过移位寄存器转移到数据寄存器。<br>产生RxNE标志位，表示数据寄存器非空<br>（读DR寄存器来清除这个Ev7事件）
			10. 接收数据2<br>在我们正在读DR寄存器是 <br>数据2就已经移位到移位寄存器中。等待移位到DR数据寄存器中。
			11. 发送应答
			12. 这事数据又被传入DR数据寄存器。EV7事件又来了。需要我们读取、
			13. 如此循环……直到停止<br>也就是需要设置ACK = 0 不应答， 和STOP请求。
			<empty-block/>
			其实同上，这里是在发送一个字节之后，在对面没有应答前，就填充好了下一次要发送的数据。产生的标志位也是一样的。
	## 4. STM32硬件与软件的波形对比 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5ebbba9c-d0a2-4023-9bec-d60dc4af1612.webp)
	## 5. STM32配置硬件I2C外设流程 {toggle="true"}
		1. 开启外设和对应GPIO口的时钟
		2. 把I2C外设对应的GPIO口配置为复用开漏模式
		3. 使用结构体，对I2C进行配置
		4. 使能I2C
	## 6. STM32的I2C.h标准库函数介绍 {toggle="true"}
		### 1.I2C配置和使用函数 {toggle="true"}
			> `void I2C_DeInit(I2C_TypeDef* I2Cx);`
				- 恢复缺省配置
			> `void I2C_Init(I2C_TypeDef* I2Cx, I2C_InitTypeDef* I2C_InitStruct);`
				- 初始化I2C
			> `void I2C_StructInit(I2C_InitTypeDef* I2C_InitStruct);`
				- 初始化结构体
			> `void I2C_Cmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 使能I2C
			> `void I2C_DMACmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- I2C MDA使能
			- `void I2C_DMALastTransferCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 用于控制 I2C 传输中的最后一次传输的相关操作。
			> `void I2C_GenerateSTART(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 调用，生成起始条件
			> `void I2C_GenerateSTOP(I2C_TypeDef* I2Cx, FunctionalState NewState);` 
				- 调用，生成终止条件
			> `void I2C_AcknowledgeConfig(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 配置 I2C 的应答机制。1为应答
			- `void I2C_OwnAddress2Config(I2C_TypeDef* I2Cx, uint8_t Address);`
				- 设置 I2C 设备自身的地址。
			- `void I2C_DualAddressCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 启用或禁用双地址模式，以适应特定的通信需求。
			- `void I2C_GeneralCallCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 控制是否响应通用呼叫。
			- `void I2C_ITConfig(I2C_TypeDef* I2Cx, uint16_t I2C_IT, FunctionalState NewState);`
				- 配置 I2C 的中断功能，根据不同的中断类型进行使能或禁用。
			> `void I2C_SendData(I2C_TypeDef* I2Cx, uint8_t Data);`
				- 发送一个字节的数据。
			> `uint8_t I2C_ReceiveData(I2C_TypeDef* I2Cx);`
				- 接收一个字节的数据。
			> `void I2C_Send7bitAddress(I2C_TypeDef* I2Cx, uint8_t Address, uint8_t I2C_Direction);`
				- 发送 7 位地址并指定通信方向（读或写）。
			- `uint16_t I2C_ReadRegister(I2C_TypeDef* I2Cx, uint8_t I2C_Register);`
				- 读取 I2C 设备的特定寄存器的值。
			- `void I2C_SoftwareResetCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 通过软件方式对 I2C 进行复位操作。
			- `void I2C_NACKPositionConfig(I2C_TypeDef* I2Cx, uint16_t I2C_NACKPosition);`
				- 配置非应答信号（NACK）的位置。
			- `void I2C_SMBusAlertConfig(I2C_TypeDef* I2Cx, uint16_t I2C_SMBusAlert);`
				- 针对 SMBus 警报进行相关配置。
			- `void I2C_TransmitPEC(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 控制是否传输 PEC（Packet Error Checking，数据包错误检查）信息。
			- `void I2C_PECPositionConfig(I2C_TypeDef* I2Cx, uint16_t I2C_PECPosition);`
				- 配置 PEC 的位置。
			- `void I2C_CalculatePEC(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 决定是否计算 PEC 值。
			- `uint8_t I2C_GetPEC(I2C_TypeDef* I2Cx);`
				- 获取计算得到的 PEC 值。
			- `void I2C_ARPCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 控制自动重传功能的启用或禁用。
			- `void I2C_StretchClockCmd(I2C_TypeDef* I2Cx, FunctionalState NewState);`
				- 配置时钟拉伸功能。
			- `void I2C_FastModeDutyCycleConfig(I2C_TypeDef* I2Cx, uint16_t I2C_DutyCycle);`
				- 设置快速模式下的占空比，以优化通信性能。
			- `FlagStatus I2C_GetFlagStatus(I2C_TypeDef* I2Cx, uint32_t I2C_FLAG);`
				- 读取标志位
			- `void I2C_ClearFlag(I2C_TypeDef* I2Cx, uint32_t I2C_FLAG);`
				- 清除标志位
			- `ITStatus I2C_GetITStatus(I2C_TypeDef* I2Cx, uint32_t I2C_IT);`
				- 读取中断标志位
			- `void I2C_ClearITPendingBit(I2C_TypeDef* I2Cx, uint32_t I2C_IT);`
				- 清除中断标志位
		### 2.I2C状态监控功能 {toggle="true"}
			I2C的标志位，往往是由多个标志位组合起来的一个大标志位。
			一个个去判断往往比较麻烦。
			所以I2C的库在.h头文件的最后，
			为我们提供了两种检测方案
			1. 基础状态监测
				- 使用` I2C_CheckEvent()`函数<br>可以同时检测多个标志位来进行比较。<br>一般使用这个。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/17d524f4-49ff-4e7d-84b6-41503d5ca224.webp)
			1. 高级状态监测
				- 使用`I2C_GetLastEvent()`函数<br>(实际上是把SR1和SR2两个状态寄存器拼接成一个16位的数据扔给你，随便你处理。所以一般不用)
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ccdd1fda-8a21-49db-b2aa-335d08138cda.webp)
			1. 基于标志位的状态监控
				- 使用`I2C_GetFlagStatus()`函数<br>可以判断某一个标志位是否置一
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/87a847df-af1b-481d-98c8-a708e33e79fe.webp)
	## 7. STM32编写代码实现**I2C硬件读写MPU6050** {toggle="true"}
		### 7.1编写时需要注意的点 {toggle="true"}
			- 发送字节时
				- 对于发送一个字节，发送地址和读写的一个字节之后，在移位寄存器还未发送完成时，就要填充下一次要发送的字节到DR数据寄存器中。否则将发生EV8_2标志（移位寄存器空，数据寄存器空）。导致读写停止。<br>正常应为EV8事件发生（字节正在发送，移位寄存器非空，数据寄存器空）
				- 对于发送多个字节，只需要重复发送、等待EV8事件即可。不发送时给予不应答、停止即可。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a3ad3bfc-c3d6-46d6-9822-ec0824a459f8.webp)
			- 接收字节时，
				- 对于接收一个字节，在指定完地址，重复起始之后，接收第第一个字节中时，就要提前配置Ack非应答和停止条件。之后等数据全部移位到数据寄存器中，才能保证只接收了一个字节。读取DR寄存器即可。
				- 对于接收多个字节，可以重复应答、然后等待EV7事件到来，读取寄存器。直到不想接收时，在数据进行移位时（还未到达DR寄存器时）提前设置非应答与停止操作。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a4a643cf-aa25-4bef-b5a3-d0d028b62c3f.webp)
		### **7.2程序文件简要说明：** {toggle="true"}
			- MPU6050.c：初始化MPU6050的寄存器。使用STM32 自带的I2C外设，编写对指定地址发送指定值、读取指定地址的函数。以及等待标志位且超时退出的函数
			- MPU6050.h：函数声明、数据结构体声明
			- MPU6050_Reg.h：寄存器的地址与名称声明
			- main.c：测试I2C通信结果。
		### MPU650.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "MPU6050.h"
#include "MPU6050_Reg.h"

#define MPU6050ADDRESS      0xD0


/**
  * 函    数：超时退出、检测标志位
  * 参    数：I2C_TypeDef* I2Cx,    定时器x
              uint32_t I2C_EVENT    标志位名称
  * 返 回 值：无
  * 注意事项：无
  */
void MPU6050_WaitEvent(I2C_TypeDef* I2Cx, uint32_t I2C_EVENT)//超时退出、检测标志位
{
    uint32_t Timeout;
    Timeout = 10000;
    while (I2C_CheckEvent(I2Cx, I2C_EVENT) != SUCCESS)
    {
        Timeout --;
        if (Timeout == 0)
        {
            break;  
        }
    }
}


/**
  * 函    数：指定地址写一个字节
  * 参    数：RegAddress   指定的寄存器地址
              Data         指定写入的数据
  * 返 回 值：无
  */
void MPU6050_WriteReg(uint8_t RegAddress, uint8_t Data)
{

    I2C_GenerateSTART(I2C2,ENABLE);//非阻塞。所以要等待事件发送完成。
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_MODE_SELECT);//等待EV5事件发生

    I2C_Send7bitAddress(I2C2,MPU6050ADDRESS,I2C_Direction_Transmitter);//发送地址和读写操作（也能用发送一个字节来完成）(自动应答)
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);//等待发送EV6事件发生

    I2C_SendData(I2C2,RegAddress);

    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_BYTE_TRANSMITTING);//等待发送EV8事件发生(字节正在发送)
    
    I2C_SendData(I2C2,Data);//直接写入下一个要发的数据
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_BYTE_TRANSMITTED );//等待EV8_2事件发生(发送完成，并且数据寄存器无)
    
    I2C_GenerateSTOP(I2C2,ENABLE);
}

/**
  * 函    数：指定地址读一个字节
  * 参    数：RegAddress   指定要读的寄存器地址
  * 返 回 值：无
  */
uint8_t MPU6050_ReadeReg(uint8_t RegAddress)
{
    uint8_t Data = 0x00;

    I2C_GenerateSTART(I2C2,ENABLE);//起始位
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_MODE_SELECT);//等待EV5事件发生

    I2C_Send7bitAddress(I2C2,MPU6050ADDRESS,I2C_Direction_Transmitter);//发送地址和读写操作（也能用发送一个字节来完成）(自动应答)
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_TRANSMITTER_MODE_SELECTED);//等待发送EV6事件发生

    I2C_SendData(I2C2,RegAddress);
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_BYTE_TRANSMITTED);//等待发送EVEV8_2事件发生(字节发送完毕)

    I2C_GenerateSTART(I2C2,ENABLE);//重复起始
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_MODE_SELECT);//等待EV5事件发生
   
    I2C_Send7bitAddress(I2C2,MPU6050ADDRESS,I2C_Direction_Receiver);//发送地址和读写操作（也能用发送一个字节来完成）(自动应答)
    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_RECEIVER_MODE_SELECTED);//等待发送EV6事件发生
  
    I2C_AcknowledgeConfig(I2C2,DISABLE);//在字节来之前，设置非应答。
    I2C_GenerateSTOP(I2C2,ENABLE);//直接停止。但是会接收字节完毕之后才停

    MPU6050_WaitEvent(I2C2,I2C_EVENT_MASTER_BYTE_RECEIVED);//等待EV7事件到达。代表一个数据的字节已经在DR里了
    Data = I2C_ReceiveData(I2C2);
    
    I2C_AcknowledgeConfig(I2C2,ENABLE);//恢复默认状态，给从机应答
    
    return Data;//返回读取到的值
}

/**
  * 函    数：初始化MPU6050
  * 参    数：无
  * 返 回 值：无
  */
void MPU6050_Init(void)
{
    /*初始化I2C*/
//    MyI2C_Init();
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_I2C2,ENABLE);
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);//开启GPIO和I2C外设时钟
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_OD;
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10 | GPIO_Pin_11;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOB,&GPIO_InitStructure);   //初始化PB10、11为复用开漏
    /*初始化I2C外设*/
    I2C_InitTypeDef I2C_InitStructure;
    I2C_InitStructure.I2C_Ack = I2C_Ack_Enable;//确定是否应答
    I2C_InitStructure.I2C_AcknowledgedAddress = I2C_AcknowledgedAddress_7bit;//指定STM32作为从机。可以响应几位的地址
    I2C_InitStructure.I2C_ClockSpeed = 50000;//最大400KHZ（快速）、标准为（100KHZ）(MPU6050最快也是400KHZ)
    I2C_InitStructure.I2C_DutyCycle = I2C_DutyCycle_2;//配置占空比。进入快速模式后才有用（>100KHZ后） （默认为1:1）
    I2C_InitStructure.I2C_Mode = I2C_Mode_I2C;//I2C模式
    I2C_InitStructure.I2C_OwnAddress1 = 0x00;//指定STM32作为从机。STM本身的地址
    I2C_Init(I2C2,&I2C_InitStructure);
    /*使能I2C外设*/
    I2C_Cmd(I2C2,ENABLE);
    /*初始化MPU6050*/
    MPU6050_WriteReg(MPU6050_PWR_MGMT_1,0x01);  //电源管理1：不复位、解除睡眠、不循环、温度传感器不失能、选择X轴陀螺仪时钟
    MPU6050_WriteReg(MPU6050_PWR_MGMT_2,0x00);  //电源管理2：不需要循环模式唤醒频率、每个轴都不需要待机
    
    MPU6050_WriteReg(MPU6050_SMPLRT_DIV,0x09);  //采样率分频：数据输出的快慢，越小输出越快.这里给10分频
    MPU6050_WriteReg(MPU6050_CONFIG,0x06);  //配置寄存器：外部同步不需要、数字低通滤波器设置为110
    MPU6050_WriteReg(MPU6050_GYRO_CONFIG,0x18);   //陀螺仪配置寄存器：不自测、满量程选择：11最大量程
    MPU6050_WriteReg(MPU6050_ACCEL_CONFIG,0x18);    //加速度计配置寄存器：不自测、满量程选择：11最大量程、不用高通滤波器
}


/**
  * 函    数：得到六轴传感器中的数据
  * 参    数：Str   MPU6050_Data的地址
  * 返 回 值：无
  */
SensorData MPU6050_Data;
void MPU6050_GetData(SensorData *Str)//存放这种结构体类型的地址
{
    Str->AccX = ( MPU6050_ReadeReg(MPU6050_ACCEL_XOUT_H) <<8|
                  MPU6050_ReadeReg(MPU6050_ACCEL_XOUT_L));
    Str->AccY = ( MPU6050_ReadeReg(MPU6050_ACCEL_YOUT_H) <<8|
                  MPU6050_ReadeReg(MPU6050_ACCEL_YOUT_L));
    Str->AccZ = ( MPU6050_ReadeReg(MPU6050_ACCEL_ZOUT_H) <<8|
                  MPU6050_ReadeReg(MPU6050_ACCEL_ZOUT_L));
    Str->Temp = ( MPU6050_ReadeReg(MPU6050_TEMP_OUT_H)   <<8|
                  MPU6050_ReadeReg(MPU6050_TEMP_OUT_L));
    Str->GyroX = ( MPU6050_ReadeReg(MPU6050_GYRO_XOUT_H) <<8 |
                  MPU6050_ReadeReg(MPU6050_GYRO_XOUT_L));
    Str->GyroY = ( MPU6050_ReadeReg(MPU6050_GYRO_YOUT_H) <<8 |
                  MPU6050_ReadeReg(MPU6050_GYRO_YOUT_L));
    Str->GyroZ = ( MPU6050_ReadeReg(MPU6050_GYRO_ZOUT_H) <<8 |
                   MPU6050_ReadeReg(MPU6050_GYRO_ZOUT_L));
}
//MPU6050_ACCEL_XOUT_H    0x3B
//MPU6050_ACCEL_XOUT_L    0x3C
//MPU6050_ACCEL_YOUT_H    0x3D
//MPU6050_ACCEL_YOUT_L    0x3E
//MPU6050_ACCEL_ZOUT_H    0x3F
//MPU6050_ACCEL_ZOUT_L    0x40
//MPU6050_TEMP_OUT_H      0x41
//MPU6050_TEMP_OUT_L      0x42
//MPU6050_GYRO_XOUT_H     0x43
//MPU6050_GYRO_XOUT_L     0x44
//MPU6050_GYRO_YOUT_H     0x45
//MPU6050_GYRO_YOUT_L     0x46
//MPU6050_GYRO_ZOUT_H     0x47
//MPU6050_GYRO_ZOUT_L     0x48

			```
		### MPU650.h {toggle="true"}
			```javascript
#ifndef __MPU6050_H
#define __MPU6050_H

/**
  * 函    数：初始化MPU6050
  * 参    数：无
  * 返 回 值：无
  */
void MPU6050_Init(void);

/**
  * 函    数：指定地址写一个字节
  * 参    数：RegAddress   指定的寄存器地址
              Data         指定写入的数据
  * 返 回 值：无
  */
void MPU6050_WriteReg(uint8_t RegAddress, uint8_t Data);
/**
  * 函    数：指定地址读一个字节
  * 参    数：RegAddress   指定要读的寄存器地址
  * 返 回 值：无
  */
uint8_t MPU6050_ReadeReg(uint8_t RegAddress);


//传感器数据
typedef struct Data 
{
    int16_t AccX;
    int16_t AccY;
    int16_t AccZ;
    int16_t Temp;
    int16_t GyroX;
    int16_t GyroY;
    int16_t GyroZ;
}SensorData;

extern SensorData MPU6050_Data;

/**
  * 函    数：得到六轴传感器中的数据
  * 参    数：Str   MPU6050_Data的地址
  * 返 回 值：无
  */
void MPU6050_GetData(SensorData *Str);

#endif

			```
		### MPU6050_Reg.h {toggle="true"}
			```javascript
#ifndef __MPU6050_REG_H
#define __MPU6050_REG_H

#define MPU6050_SMPLRT_DIV      0x19    //采样率分频
#define MPU6050_CONFIG          0x1A    //配置寄存器
#define MPU6050_GYRO_CONFIG     0x1B    //陀螺仪配置寄存器
#define MPU6050_ACCEL_CONFIG    0x1C    //加速度计配置寄存器
        
#define MPU6050_ACCEL_XOUT_H    0x3B
#define MPU6050_ACCEL_XOUT_L    0x3C
#define MPU6050_ACCEL_YOUT_H    0x3D
#define MPU6050_ACCEL_YOUT_L    0x3E
#define MPU6050_ACCEL_ZOUT_H    0x3F
#define MPU6050_ACCEL_ZOUT_L    0x40
#define MPU6050_TEMP_OUT_H      0x41
#define MPU6050_TEMP_OUT_L      0x42
#define MPU6050_GYRO_XOUT_H	    0x43
#define MPU6050_GYRO_XOUT_L     0x44
#define MPU6050_GYRO_YOUT_H     0x45
#define MPU6050_GYRO_YOUT_L     0x46
#define MPU6050_GYRO_ZOUT_H     0x47
#define MPU6050_GYRO_ZOUT_L     0x48
    
#define MPU6050_PWR_MGMT_1      0x6B    //电源管理1
#define MPU6050_PWR_MGMT_2      0x6C    //电源管理2
#define MPU6050_WHO_AM_I        0x75    //设备号

#endif

			```
		### main.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
#include "MPU6050.h"
/*
    硬件读写I2C
*/



int main()
{
    OLED_Init();//初始化OLED;
    MPU6050_Init();//初始化MPU6050
    
    
    while(1)    
    {
         MPU6050_GetData(&MPU6050_Data);
        //显示加速度
        OLED_ShowSignedNum(1,1,(int16_t)MPU6050_Data.AccX,5);
        OLED_ShowSignedNum(2,1,(int16_t)MPU6050_Data.AccY,5);
        OLED_ShowSignedNum(3,1,(int16_t)MPU6050_Data.AccZ,5);
        //显示陀螺仪
        OLED_ShowSignedNum(1,8,MPU6050_Data.GyroX,5);
        OLED_ShowSignedNum(2,8,MPU6050_Data.GyroY,5);
        OLED_ShowSignedNum(3,8,MPU6050_Data.GyroZ,5);
        //显示温度
        OLED_ShowSignedNum(4,4,MPU6050_Data.Temp,5);
        
        
    }
    
}

			```
		<empty-block/>
	## 8.\[扩展\]为什么STM32硬件I2C为什么在速度快时会调节占空比；为什么有最大速度限制 {toggle="true"}
		由之前的知识我们可以了解到。
		对于I2C而言，在硬件电路上采用了开漏输出+上拉电阻的形式
		这就导致了下拉时为强下拉，上拉时为弱上拉。
		这就导致了信号在由高电平往低电平时是十分迅速的，而从低电平回到高电平时是需要一定时间的。
		在波形上的体现就是：通信速率越快，上升沿呈现出圆弧形越强。而下降沿几乎没有太大影响。
		所以，在速度最快时，需要使得低电平的时间更长一些。
		在图片中的反馈就是这样子的
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/05dd09cf-96ce-42a2-8348-2f6e40c95f0f.webp)
# **23.   SPI通信详解** {toggle="true"}
	## 1. SPI通信介绍 {toggle="true"}
		SPI（Serial Peripheral Interface）和I2C一样，都是实现主控芯片和外挂芯片之间的数据交流。
		**I2C 与 SPI 的比较**
		**I2C的缺点：**
		- **驱动能力差：** I2C接口由于线路上拉电阻的电路结构，使得通信总线的驱动能力较弱。当通信线路电压较高时，电阻上的损耗就越大，限制了I2C的最大通信速度。在标准模式下，I2C的最高时钟频率仅为100kHz，相比于SPI要慢得多。
		**SPI的优点：**
		- **传输速度高：** SPI在设计上更注重速度，其设计简单且灵活，能够支持更高的传输速率。
		SPI是一种串行同步通信协议。它通过四根通信线（SCK、MOSI、MISO、SS）进行数据传输。
		- **全双工：** SPI支持全双工通信，可以同时进行数据发送和接收。
		- **多设备支持：** SPI支持多设备挂载（一主多从），但不支持多主多从。
		- **时钟线：** SPI使用一根时钟线（SCK）来提供时钟信号。数据的输出和输入都在SCK的上升沿或下降沿进行，由主机提供时钟信号，数据接收由从机来完成。
		**SPI通信线**
		- **SCK（Serial Clock）**： 提供时钟信号，由主机控制。数据传输依赖于SCK的时钟信号。
		- **MOSI（Master Out Slave In）**： 主机输出，从机输入。用于主设备向从设备发送数据。
		- **MISO（Master In Slave Out）**： 从机输出，主机输入。用于从设备向主设备发送数据。
		- **SS（Slave Select）**： 片选信号，用于选择从设备。每个从设备都有一个独立的SS线，通过拉低SS信号来选择相应的从设备进行通信。
		**SPI通信线别称**
		- **SCK：** SCLK, CLK, CK
		- **MOSI： **MI, SIO, DO（Data Output）
		- **MISO：** DI（Data Input）
		- **SS： **NSS（Not Slave Select）, CS（Chip Select）
		**时序和数据传输**
		- **无应答机制：** SPI没有应答机制，这意味着主机发送的数据没有确认信号，数据传输的完整性和正确性需要其他手段保障。
		- **时钟信号同步：** SPI通信依赖时钟信号（SCK）进行数据同步，确保数据位在SCK的时钟沿（上升沿或下降沿）传输。
	## 2. SPI和I2C的区别 {toggle="true"}
		**I2C**
		- I2C的硬件电路和软件时序更加复杂
		- I2C通信的性价比高：在消耗最低硬件资源的情况下，实现最多的功能
		- I2C的弱上拉，会限制最大通信速度（标准100KHZ以下，快速400KHZ以下）
		**SPI**
		- SPI传输更快。这一般取决于芯片厂商设计时的需求（比如W25Q64存储器芯片最大速度为80MHZ）
		- SPI设计的功能少，学习简单
		- SPI的硬件开销大，通信线的个数比较多。并且通信时经常会出现资源浪费的情况
		- SPI不会有I2C的应答机制
	## 3. SPI的硬件电路规定 {toggle="true"}
		- 所有SPI设备的SCK、MOSI、MISO分别连在一起
		- 主机另外引出**多条**SS控制线，分别接到各从机的SS引脚
		- 所有设备的输出引脚配置为推挽输出<br>！但是，SPI协议规定，为了防止从机全都是推挽输出导致冲突。<br>！规定**从机在未被点名时**，应该将输出设置为高阻态（引脚断开）
		- 所有设备的输入引脚配置为浮空或上拉输入
		<empty-block/>
		主机通过MOSI输出、从机通过MOSI输入
		主机已通过MISO输入，从机通过MISO输出
		主机（输出）想和谁通信，就把他的控制线输出为低电平。然后被对应的从机接收（输入）（同一时间只能选一个从机）
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/61034075-39bb-4b01-8110-fac1c7eeb571.webp)
	## 4. SPI的移位示意图 {toggle="true"}
		SPI在通信时有三种情况
			- 边发送边接收
			- 只接收，不发送
			- 只发送，不接收
		### 1.边接收边发送 {toggle="true"}
			SPI的主机和从机可以理解为这个框图。
			- 波特率发生器
				波特率发生器就是主机的时钟。波特率发生器会控制主机和从机的移位寄存器的移位操作（左移）
			- 在进行边接收边发送时，比如我们写入了1 0 1 0 1 0 1 0这个数据希望发送到从机、而从机西方发送0 1 0 1 0 1 0 1这个数据，希望发送到主机。
				主机和从机的移位寄存器就会根据时钟的上升沿和下降沿（上升沿发送，下降沿接收）一位一位的发送和接收数据。
				在下降沿没来到之前，暂存在发送寄存器中。
				在下降沿来到之后，接收器就会接收主机/从机发送来的数据，并保存在移位寄存器的最低位。
				如此循环八次，就可以得到一个字节的数据。
				如下图所示
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/93cfe2df-93b4-4aa5-b5ea-6765d67eff7f.webp)
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a845432f-de5d-4bcf-897f-a0bb48632924.webp)
			<empty-block/>
		### 2.只接收，不发送 {toggle="true"}
			在只接收时，接收方还是会发送的，只是发送由我们指定发送0x00或者0xFF。<br>并且接收方发来的数据，发送方是不会去读取的。
		### 3.只发送，不接收 {toggle="true"}
			在发送时，接收方还是会发送的，只是接收由我们指定发送0x00或者0xFF。<br>并且接收方发来的数据，发送方是不会去读取的。
	## 5. SPI软件设计（时序基本单元） {toggle="true"}
		### 5.1 起始条件与终止条件的时序 {toggle="true"}
			- 起始条件：SS从高电平到低电平
			- 终止条件：SS从低电平到高电平
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e48dff7e-de70-48c1-97fe-b830c719aa17.webp)
		### 5.2 主机与从机交换一个字节的时序 {toggle="true"}
			**SPI有两个配置位：****`CPOL`**** 和 ****`CPHA`**
			他们的不同值，对应了SPI交换一个字节的不同方式
			- CPOL = 0时：空闲状态时，SCK为低电平
			- CPOL = 1时：空闲状态时，SCK为高电平
			- CPHA =0时：SCK第一个边沿移入数据，第二个边沿移出数据
			- CPHA =1时：SCK第一个边沿移出数据，第二个边沿移入数据
			- **其中CPHA表示的是时钟相位，决定第一个时钟采样移入还是第二个时钟采样移入**
			**一、交换一个字节（模式1）CPOL = 0   CPHA = 1**
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/91511914-3813-44f8-b6f0-27b278f25bd0.webp)
			1. 点名：SS信号线被置为低电平，表示从机已被选中并准备接收或发送数据。
			2. SCK空闲状态：SCK（Serial Clock）信号线保持为低电平，这是CPOL=0模式下的空闲状态。
			3. 坐下：当数据交换完成后，SS信号线被置回高电平，从机不再被选中。
			4. 第一个边沿来临：SCK的第一个上升沿到来，主机和从机开始发送高位数据。
			5. 第二个边沿来临：SCK的第二个下降沿到来，主机和从机接收到数据到最低位。
			6. 不断循环：这个过程不断重复，直到完成一个字节的数据交换。<br>如果想接收多个数据，就让SS一直为0就OK
			7. 主机输出后的电平没有硬性要求：在数据交换结束后，SCK可以继续保持低电平也可以恢复到空闲状态。
			8. 但是从机恢复推挽输出：从机在未被点名前应是高阻态。
			**二、交换一个字节（模式0）CPOL = 0   CPHA = 0**
			在实际应用中。模式0的应用是最多的。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/37ebf681-5e93-4eb3-82c2-ee4a9edaf46e.webp)
			1. 点名：SS信号线被置为低电平，表示从机已被选中并准备接收或发送数据。
			2. 在第一个边沿来临的时候就要读取数据了，所以在此之前要提前发出最高位。
			3. 主机和从机在第一个边沿到达时开始接收信号了，<br>此时开始捕获数据到自己的移位寄存器的最低位。
			4. 主机和从机在第二个边沿开始发送信号，<br>送出自己的次高位
			5. 如此循环……，在接收到一个字节之后。<br>还会有一个低电平回到SCK静态。此时如果主机不打算交流了（SS返回高电平）<br>此时MOSI，主机输出端 可以上拉也可以保持，然后停止点名。<br>而MISO，从机的输出端 要回到高阻态
			6. 如果想要继续发送字节，就保持SS为0，主机从机的端口也不变。再来一遍这个波形。
			三、**交换一个字节（模式2）CPOL = 1   CPHA = 0**
			> 只不过是把CLK的波形取反一下就可以了。**其他的和模式0一样**
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e5fff027-5bcc-4c2e-8829-d0a24d214dad.webp)
			**四、交换一个字节（模式0）CPOL = 0   CPHA = 0**
			> 只不过是把CLK的波形取反一下就可以了。**其他的和模式1一样**
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/0ca9c7de-bdab-4928-8684-cf93bb8cabfe.webp)
	## 6. SPI通信举例（软硬件波形、控制W25Q64方法） {toggle="true"}
		### 6.1. 举例：SPI通信控制**W25Q64使能** {toggle="true"}
			SPI不像I2C那样，有效数据流第一个字节是寄存器地址、之后依次是读写的数据。<br>使用的是读写寄存器的模型
			SPI使用的是指令码+读写数据的模型：在SPI起始之后，第一个交换给从机的数据就是指令码。在从机中，会有一个对应的指令集。指令集会对应从机的不同功能。不同的功能对应不同的字节个数，比如控制W25Q64 存储器 使能，就需要一个字节。而控制写数据时，就需要多个字节。
			W25Q64存储器的使能的指令为   0x06
			在这里可以参考W25Q64使能的波形图
			这实际上就是主机和从机交换了0x06和0xFF的一个字节。
			在从机经过对比之后。发现0x06为自己的指令集中的使能，那么他就会开启。
			- 并且SPI是没有I2C的应答机制的
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f53c2a8c-0838-48ec-9af8-fde813f58d51.webp)
		### 6.2. 举例：SPI通信控制W25Q64指定地址写 {toggle="true"}
			指定地址写同样的也需要写指令集。不过这次的就不同于使能只有一位字节就可以了。
			- 需要先发送写指令（0x02）、
			- 随后在指定地址（W25Q64有8M的存储空间，所以他的地址长度为24位  \[23::0\]，也就是我们要发送三个字节长的地址）
			- 最后再发送指定数据。（Data）
			波形如下：
			这也是一次次的交换数据
			这段波形的意思是：我要在地址为0x12 34 56的寄存器下写入0x55
			从机的输出一直为1  也就是主机一直跟从机交换0xFF
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3df541f3-669b-4676-9d4a-2a875e0baa62.webp)
			这里的和I2C的一样，每次读写都会使地址指针++、如果在一次发送之后不终止，后续发送或者读取的字节就会一次指到后边的字节中
			<empty-block/>
		### 6.3. 举例：SPI通信控制W25Q64指定地址读 {toggle="true"}
			指定地址读
			- 向SS指定的设备
			- 发送读指令(0x03) 
			- 随后在指定地址(Address\[23:0\])下，读取从机数据（Data）
			这个波形的意思是：
			从前4个字节一直都是0xFF.第五个字节开始，从机开始发送字节。此时主机的输出为0xFF。从机开始发送这个寄存器中的值，从机的输出就不再是0xFF。而主机交换给从机的值是0XFF
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/828c2663-3bfd-4ab4-8740-c14b7a4f83a7.webp)
			这里的和I2C的一样，每次读写都会使地址指针++、如果在一次发送之后不终止，后续发送或者读取的字节就会一次指到后边的字节中
			<empty-block/>
			- 并且还可以看出，由于从机回复 是硬件执行。所以从机回复的高低电平是紧贴着下降沿的。
			- 而软件，在下降沿后 主机的输出并没有紧贴着，稍有延迟。<br>这就是软件模拟的一些延迟。不过因为有同步时钟，这些都问题不大
			- 并且。在W25Q64发送数据时，他的数据是在上一个波形的下降沿提前发送（移出）的。<br>因为这是SPI的  模式0
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/49ea18f5-9d6a-483f-ab64-39ca95ed4adc.webp)
	## 7.SPI基本时序编写 {toggle="true"}
		### 7.1编写时需要注意的点 {toggle="true"}
			**SPI的基本时序有三个**
				- 起始
				- 结束
				- 交换字节
			起始和结束只需要控制SS片选信号的高低电平即可
			**交换字节分为两种方法**
			- 掩码依次提取法
				掩码提取其实就是通过 & 操作来提取出一个字节中的某一位
			- 移位模型法
				移位模型法正是SPI通信时用到的方法。这个方法更加高效
		### **7.2程序文件简要说明：** {toggle="true"}
			- MySPI：为了防止与stm32函数重名，所以添加了My前缀。<br>主要是为了完成SPI的三个基本时序
			- MySPI.h：函数声明
		### MySPI.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header

/*所用引脚列表*/
#define RCCPeriph       RCC_APB2Periph_GPIOA
#define SCK_Port        GPIOA
#define SCK_Pin         GPIO_Pin_6
#define SS_Port         GPIOA
#define SS_Pin          GPIO_Pin_5
#define MOSI_Port       GPIOA
#define MOSI_Pin        GPIO_Pin_4
#define MISO_Port       GPIOA
#define MISO_Pin        GPIO_Pin_3

/**
  * 函    数：写片选信号SS
  * 参    数：BitValue：输入1片选信号SS为高电平
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_W_SS (uint8_t BitValue)
{
    GPIO_WriteBit(SS_Port,SS_Pin,(BitAction)BitValue);
}

/**
  * 函    数：写时钟信号SCK
  * 参    数：BitValue：输入1时钟信号SCK为高电平
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_W_SCK (uint8_t BitValue)
{
    GPIO_WriteBit(SCK_Port,SCK_Pin,(BitAction)BitValue);
}

/**
  * 函    数：写 主机输出，从机输入信号MOSI
  * 参    数：BitValue：输入1时主机输出高电平
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_W_MOSI (uint8_t BitValue)
{
    GPIO_WriteBit(MOSI_Port,MOSI_Pin,(BitAction)BitValue);
}

/**
  * 函    数：读 主机输入，从机输出信号MISO
  * 参    数：无
  * 返 回 值：BitValue
  * 注意事项：无
  */
uint8_t MySPI_R_MISO (void)
{
    return GPIO_ReadInputDataBit(MISO_Port,MISO_Pin);
}

/**
  * 函    数：MySPI初始化
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Init(void)
{   
    /*配置时钟与引脚*/
    RCC_APB2PeriphClockCmd(RCCPeriph,ENABLE);
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = SCK_Pin | SS_Pin | MOSI_Pin;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(MOSI_Port,&GPIO_InitStructure);   //时钟、片选、MOSI都是推挽输出
    
     GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = MISO_Pin;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(MISO_Port,&GPIO_InitStructure);   //MISO 为上拉输入
    
    MySPI_W_SS(1);  //片选默认为高
    MySPI_W_SCK(0); //时钟默认为低
}







/*******************/
/*SPI的三个时序单元*/
/*******************/





/**
  * 函    数：起始信号
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Start(void)
{   
    MySPI_W_SS(0);
}

/**
  * 函    数：终止条件
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Stop(void)
{   
    MySPI_W_SS(1);
}


/**
  * 函    数：交换一个字节（模式0）（方法1）
  * 参    数：SendByte      待发送的字节
  * 返 回 值：ReceiveByte   接收到的字节
  * 注意事项：这是使用掩码依次提取数据中的每一位保存或发送
              好处是不会改变传入参数本身，但是效率不高
              如果要改为模式1，则先上升沿再发送。先下降沿再接收（2、3则直接改时钟极性就ok了）
  */
//uint8_t MySPI_WarpByte(uint8_t SendByte)
//{
//    uint8_t ReceiveByte = 0x00;
//    
//    uint8_t i = 0;
//    for(i = 0; i < 8; i++)
//    {
//        MySPI_W_MOSI(SendByte & (0x80 >> i));  //发送第一个bit
//        MySPI_W_SCK(1);//第上升沿来临
//        if (MySPI_R_MISO() == 1)
//        {
//            ReceiveByte |= (0x80 >> i);    //按照从高往低接收数据
//        }
//        MySPI_W_SCK(0); //下降沿来临
//    }
//    return ReceiveByte;
//} 

/**
  * 函    数：交换一个字节（模式0）（方法2）
  * 参    数：SendByte      待发送的字节
  * 返 回 值：ReceiveByte   接收到的字节
  * 注意事项：这是使用了移位模型的方式。效率更快
              如果要改为模式1，则先上升沿再发送。先下降沿再接收（2、3则直接改时钟极性就ok了）
  */
uint8_t MySPI_WarpByte(uint8_t SendByte)
{
    uint8_t i = 0;
    for(i = 0; i < 8; i++)
    {
        MySPI_W_MOSI(SendByte & 0x80);  //发送第一个bit
        SendByte <<= 1; //发送数据左移一位
        MySPI_W_SCK(1); //第上升沿来临
        if (MySPI_R_MISO() == 1)
        {
            SendByte |= 0x01;    //保存收到的数据到发送寄存器的最低位
        }
        MySPI_W_SCK(0); //下降沿来临
    }
    return SendByte;
} 





			```
		### MySPI.h {toggle="true"}
			```javascript
#ifndef __MYSPI_H
#define __MYSPI_H

//初始化
void MySPI_Init(void);
//起始
void MySPI_Start(void);
//终止
void MySPI_Stop(void);
//交换
uint8_t MySPI_WarpByte(uint8_t SendByte);

#endif

			```
# **24.   STM32通过SPI软件读写W25Q64** {toggle="true"}
	## 1. W25Q64简介 {toggle="true"}
		- W25Qxx系列是一种低成本、小型化、使用简单的**非易失性存储器**，常应用于数据存储、字库存储、固件程序存储等场景
		- 存储介质：Nor Flash（闪存）
		- 时钟频率：80MHz / 160MHz (Dual SPI) / 320MHz (Quad SPI)<br>可以在只发或者只收的时候，把不用的引脚（比如写保护、或者不用的发送、接收引脚）暂时当做数据传输用的。（这里只做了解）
		- 支持SPI模式0和模式3
		- 存储容量（24位地址）：
			<columns>
				<column>
					- W25Q40：	  4Mbit / 512KByte
					- W25Q80：	  8Mbit / 1MByte
					- W25Q16：	  16Mbit / 2MByte
					- W25Q32：	  32Mbit / 4MByte
					- W25Q64：	  64Mbit / 8MByte
					- W25Q128：  128Mbit / 16MByte
					- W25Q256：  256Mbit / 32MByte
					<empty-block/>
				</column>
				<column>
					<empty-block/>
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ced7f136-5320-459b-b46d-54eac04a2aef.webp)
				</column>
			</columns>
		<empty-block/>
	## 2. W25Q64硬件电路图 {toggle="true"}
		<columns>
			<column ratio="100">
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a1394129-7b61-4df0-888c-73e46a040674.webp)
			</column>
			<column ratio="100">
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/23e156e3-ae24-4bbe-b2f0-afb142b37c03.webp)
				这里的小括号IO1、2、3、4就是介绍时所说的双重SPI和四重SPI
				这个电路图没啥好说的
			</column>
		</columns>
		<table header-row="true">
		<colgroup>
		<col>
		<col width="258.78125">
		</colgroup>
<tr>
<td>**引脚**</td>
<td>**功能**</td>
</tr>
<tr>
<td>VCC、GND</td>
<td>电源（2.7\~3.6V）</td>
</tr>
<tr>
<td>CS（SS）</td>
<td>SPI片选</td>
</tr>
<tr>
<td>CLK（SCK）</td>
<td>SPI时钟</td>
</tr>
<tr>
<td>DI（MOSI）</td>
<td>SPI主机输出从机输入</td>
</tr>
<tr>
<td>DO（MISO）</td>
<td>SPI主机输入从机输出</td>
</tr>
<tr>
<td>WP</td>
<td>写保护</td>
</tr>
<tr>
<td>HOLD</td>
<td>数据保持</td>
</tr>
		</table>
		<empty-block/>
	## 3. W25Q64芯片框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/6e2c4ac1-4ba5-4d76-a2a5-bd213a0957cd.webp)
		### 先看右边：存储部分 {toggle="true"}
			- W25Q64的地址宽度是24位。也就是三个字节。
				- 右边这一整块是所有的存储器。存储器以字节为单位。每个字节都有一个对应的地址，左下角为起始地址，右上角为结束地址
			- 右上角的最大地址为7FFFFFh（因为W25Q64的芯片最大容量为8M，只用了一半的地址空间）
			- 整个存储器分为128块（块0\~块7F），以64KB为一块。
				- 两个例子↓
				- 块0  起始地址：**00**0000    结束地址：**00**FFFF
				- 块15 起始地址：**0F**0000    结束地址：**0F**FFFF
			- 一个块又分为16个扇区（扇区0\~扇区F），以4KB为一扇区。
				- 两个例子↓
				- 块0  ，扇区2  起始地址：00**2**000    结束地址：00**2**FFF
				- 块15 ,   扇区14 起始地址：0F**E**000    结束地址：0F**E**FFF
			- 一个扇区又分为16页（页0\~页F），以256字节为一页
				- 两个例子↓
				- 块0  ，扇区2  ，页12  起始地址：002**C**00    结束地址：002**C**FF
				- 块15 ,   扇区14 ，页3   起始地址：0FE**3**00    结束地址：0FE**3**FF
		### SPI控制逻辑部分：SPICommand &Control Logic {toggle="true"}
			芯片内部可以进行地址锁存、数据读写等操作。引出的引脚是我们用来操作的。
		### SPI状态寄存器 StatusRegister {toggle="true"}
			检测芯片是否处于忙状态、是否写使能、是否写保护等等。都是在状态寄存器中体现
		### SPI写控制逻辑 Write Control Logic {toggle="true"}
			与外部WP引脚相连。可以用来进行写保护
		### **SPI高电压生成器High Voltage Generators** {toggle="true"}
			配合Flash进行编程用的。为了使芯片掉电不丢失
		### SPI页地址锁存/计数器、字节地址锁存/计数器 {toggle="true"}
			这两个寄存器就是用来指定地址的。
			我们发的前两个字节会进入页地址锁存计数器中。最后一个字节会进入字节锁存计数器中。
			- 页地址通过写保护和行解码决定操作那一页。
			- 字节地址通过列解码和256页缓存决定指定地址的读写操作
				- 页缓存区是一个256字节的RAM存储器
				- 数据写入就是通过这个RAM缓存区来进行的。
				- 因为Flash存储信息较慢，而RAM读写信息很快。
				- 所以需要RAM先临时保存，Flash根据RAM存储器来进行存储操作。
				- 所以也有一个规定：一次写的字节数不能超过256字节
				- 在在写入时序之后，芯片会进入一段忙的状态（忙着把RAM中的信息存储到Flash中）这个忙的状态会通过一条线，连接SPI状态寄存器。给状态寄存器的BUSY位置1。
				- 因为读取只是看一眼状态，不需要改变，所以读取时很快的。基本不会受到限制
			- 并且因为这两个寄存器都有计数器，所以在读写之后会指针+1
	## 4. Flash操作注意事项 {toggle="true"}
		**写入操作时：**
		- 写入操作前，必须先进行写使能
		- **每个数据位只能由1改写为0，不能由0改写为1**
			所以**写入数据前必须先擦除**，擦除后，所有数据位变为1
		- 擦除必须按**最小擦除单元进行**（整个芯片、按块、按扇区）
		- 连续写入多字节时，**最多写入一页的数据，超过页尾位置的数据**，会回到页首覆盖写入<br>写入操作结束后，芯片进入忙状态，不响应新的读写操作
		**读取操作时：**
		- 直接调用读取时序，无需使能，无需额外操作，没有页的限制，读取操作结束后不会进入忙状态，但不能在忙状态时读取<br>
	## 5. W25Q64手册精简 {toggle="true"}
		这里只精简出一般需要的。其他位请详看手册。括号内是对应手册的章节
		### （11.1 ）STATUS REGISTER 状态寄存器 {toggle="true"}
			1. BUSY位
				- 忙碌（BUSY）是状态寄存器（S0）中的一个只读位，
				- 当设备正在执行页编程、扇区擦除、块擦除、芯片擦除或写状态寄存器指令时，该位被设置为 1 状态。
				- 在此期间，设备将忽略其他指令，除了读状态寄存器和擦除暂停指令（请参阅交流特性中的 tW、tPP、tSE、tBE 和 tCE）。
				- 当编程、擦除或写状态寄存器指令完成时，BUSY位将被清除为 0 状态，表示设备已准备好接受进一步的指令。
			2. 写使能锁存器（WEL）
				- 写使能锁存器（WEL）是状态寄存器（S1）中的一个只读位，
				- 在执行写使能指令后被设置为 1。
				- 当设备被写禁止时，WEL 状态位被清除为 0。
				- 在上电或执行以下任何指令后**会出现写禁止状态**：写禁止、页编程、扇区擦除、块擦除、芯片擦除和写状态寄存器。<br>这表明我们在每次写入操作时都要写使能一下
			3. 这两位在寄存器中是这样的。我们要掌握的在最低的两位
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f310126b-8c02-483c-9d90-9f0de27662ca.webp)
		### （11.2.1）Manufacturer and Device Identification  制造商和设备标识 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/bc200dd1-8420-46f1-95a1-485a8d167ca4.webp)
		### （11.2.2） Instruction Set Table  SPI  指令集 （常用标记） {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e0a669c7-9f99-49eb-bef9-27e36c58595a.webp)
	## **6. W25Q64读写存储器**编写 {toggle="true"}
		### 7.1编写时需要注意的点 {toggle="true"}
			一定要先看懂时序图再去编写存储器 的 读写时序
			**需要注意Flash的操作注意事项**
			**写入操作时：**
			- 写入操作前，必须先进行写使能
			- **每个数据位只能由1改写为0，不能由0改写为1**
				所以**写入数据前必须先擦除**，擦除后，所有数据位变为1
			- 擦除必须按**最小擦除单元进行**（整个芯片、按块、按扇区）
			- 连续写入多字节时，**最多写入一页的数据，超过页尾位置的数据**，会回到页首覆盖写入<br>写入操作结束后，芯片进入忙状态，不响应新的读写操作
			**读取操作时：**
			- 直接调用读取时序，无需使能，无需额外操作，没有页的限制，读取操作结束后不会进入忙状态，但不能在忙状态时读取
		### **7.2程序文件简要说明：** {toggle="true"}
			- MySPI：为了防止与stm32函数重名，所以添加了My前缀。<br>主要是为了完成SPI的三个基本时序
			- MySPI.h：函数声明
			- W25Q64.c：初始化W25Q64存储寄存器。完成读写、擦除存储器的时序
			- W25Q64.h：函数声明、数据结构体声明
			- W25Q64_Ins.h：控制存储器时需要用到的指令集
			- main.c：测试SPI读取存储器结果。
		### MySPI.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header

/*所用引脚列表*/
#define RCCPeriph       RCC_APB2Periph_GPIOA
#define SCK_Port        GPIOA
#define SCK_Pin         GPIO_Pin_6
#define SS_Port         GPIOA
#define SS_Pin          GPIO_Pin_5
#define MOSI_Port       GPIOA
#define MOSI_Pin        GPIO_Pin_4
#define MISO_Port       GPIOA
#define MISO_Pin        GPIO_Pin_3

/**
  * 函    数：写片选信号SS
  * 参    数：BitValue：输入1片选信号SS为高电平
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_W_SS (uint8_t BitValue)
{
    GPIO_WriteBit(SS_Port,SS_Pin,(BitAction)BitValue);
}

/**
  * 函    数：写时钟信号SCK
  * 参    数：BitValue：输入1时钟信号SCK为高电平
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_W_SCK (uint8_t BitValue)
{
    GPIO_WriteBit(SCK_Port,SCK_Pin,(BitAction)BitValue);
}

/**
  * 函    数：写 主机输出，从机输入信号MOSI
  * 参    数：BitValue：输入1时主机输出高电平
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_W_MOSI (uint8_t BitValue)
{
    GPIO_WriteBit(MOSI_Port,MOSI_Pin,(BitAction)BitValue);
}

/**
  * 函    数：读 主机输入，从机输出信号MISO
  * 参    数：无
  * 返 回 值：BitValue
  * 注意事项：无
  */
uint8_t MySPI_R_MISO (void)
{
    return GPIO_ReadInputDataBit(MISO_Port,MISO_Pin);
}

/**
  * 函    数：MySPI初始化
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Init(void)
{   
    /*配置时钟与引脚*/
    RCC_APB2PeriphClockCmd(RCCPeriph,ENABLE);
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = SCK_Pin | SS_Pin | MOSI_Pin;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(MOSI_Port,&GPIO_InitStructure);   //时钟、片选、MOSI都是推挽输出
    
     GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = MISO_Pin;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(MISO_Port,&GPIO_InitStructure);   //MISO 为上拉输入
    
    MySPI_W_SS(1);  //片选默认为高
    MySPI_W_SCK(0); //时钟默认为低
}







/*******************/
/*SPI的三个时序单元*/
/*******************/





/**
  * 函    数：起始信号
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Start(void)
{   
    MySPI_W_SS(0);
}

/**
  * 函    数：终止条件
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Stop(void)
{   
    MySPI_W_SS(1);
}


/**
  * 函    数：交换一个字节（模式0）（方法1）
  * 参    数：SendByte      待发送的字节
  * 返 回 值：ReceiveByte   接收到的字节
  * 注意事项：这是使用掩码依次提取数据中的每一位保存或发送
              好处是不会改变传入参数本身，但是效率不高
              如果要改为模式1，则先上升沿再发送。先下降沿再接收（2、3则直接改时钟极性就ok了）
  */
//uint8_t MySPI_WarpByte(uint8_t SendByte)
//{
//    uint8_t ReceiveByte = 0x00;
//    
//    uint8_t i = 0;
//    for(i = 0; i < 8; i++)
//    {
//        MySPI_W_MOSI(SendByte & (0x80 >> i));  //发送第一个bit
//        MySPI_W_SCK(1);//第上升沿来临
//        if (MySPI_R_MISO() == 1)
//        {
//            ReceiveByte |= (0x80 >> i);    //按照从高往低接收数据
//        }
//        MySPI_W_SCK(0); //下降沿来临
//    }
//    return ReceiveByte;
//} 

/**
  * 函    数：交换一个字节（模式0）（方法2）
  * 参    数：SendByte      待发送的字节
  * 返 回 值：ReceiveByte   接收到的字节
  * 注意事项：这是使用了移位模型的方式。效率更快
              如果要改为模式1，则先上升沿再发送。先下降沿再接收（2、3则直接改时钟极性就ok了）
  */
uint8_t MySPI_WarpByte(uint8_t SendByte)
{
    uint8_t i = 0;
    for(i = 0; i < 8; i++)
    {
        MySPI_W_MOSI(SendByte & 0x80);  //发送第一个bit
        SendByte <<= 1; //发送数据左移一位
        MySPI_W_SCK(1); //第上升沿来临
        if (MySPI_R_MISO() == 1)
        {
            SendByte |= 0x01;    //保存收到的数据到发送寄存器的最低位
        }
        MySPI_W_SCK(0); //下降沿来临
    }
    return SendByte;
} 





			```
		### MySPI.h {toggle="true"}
			```javascript
#ifndef __MYSPI_H
#define __MYSPI_H

//初始化
void MySPI_Init(void);
//起始
void MySPI_Start(void);
//终止
void MySPI_Stop(void);
//交换
uint8_t MySPI_WarpByte(uint8_t SendByte);

#endif

			```
		### W25Q64.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "W25Q64.h"
#include "W25Q64_Ins.h"
#include "MySPI.h"


/**
  * 函    数：初始化W25Q64
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_Init(void)
{
    MySPI_Init();
}


/********************/
/*拼接完整的通信时序*/
/********************/

/**
  * 函    数：查看W25Q64的厂商号和设备号
  * 参    数：ID* Str   存放了ID结构体的指针
  * 返 回 值：无
  * 注意事项：接收第八位时是|=
  */
ID W25Q64_ID;//存放设备ID号的结构体
void W25Q64_ReadID(ID* Str)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_JEDEC_ID);//发送读取设备号指令。返回值不要
    Str->MID = MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收厂商ID 从机发来的设备号。发送值随便
    Str->DID = MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收设备ID高八位
    Str->DID <<= 8;//把接收到的数据放到高八位
    Str->DID |= MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收设备ID低八位
    MySPI_Stop();//停止
}

/**
  * 函    数：写使能
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_WriteEnable(void)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_WRITE_ENABLE);//发送
    MySPI_Stop();//停止
}

/**
  * 函    数：写失能
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_WriteDisable(void)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_WRITE_DISABLE);//发送
    MySPI_Stop();//停止
}

/**
  * 函    数：等待忙函数
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_WaitBusy(void)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_READ_STATUS_REGISTER_1);  //发送
    
    uint32_t TimeOut = 100000;
    while((MySPI_WarpByte(W25Q64_DUMMY_BYTE)&0x01) == 0x01)    //读取状态寄存器1的Busy位是否为1,为1则等待
    {
        TimeOut--;
        if(TimeOut == 0)
        {
            break;  //超时退出
        }
    }
    MySPI_Stop();   //停止
}

/**
  * 函    数：页编程
  * 参    数：Address       要写入那个页地址
              *DataArray    存储字节所用的数组
              Count         一次写入多少字节
  * 返 回 值：无
  * 注意事项：一次只能写入最多0-256个字节
  */
void W25Q64_PageProgram(uint32_t Address, uint8_t* DataArray,uint16_t Count)//(0-256,所以要16位)
{
    W25Q64_WaitBusy();
    //事前等待。（事后等待是先等待再退出，比较保险。 事前等待可以先做别的事，再进去。效率高）
    
    W25Q64_WriteEnable();//写使能（每次写时都要先写使能）
    
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_PAGE_PROGRAM);//发送页编程指令
    MySPI_WarpByte(Address >> 16 );
    MySPI_WarpByte(Address >> 8 );//（接收只能接收8位。会自动舍弃）
    MySPI_WarpByte(Address >> 0);//发送页地址
    uint16_t i = 0;
    for(i = 0; i < Count; i++)
    {
        MySPI_WarpByte(DataArray[i]);//发送Count个数组的第i位
    }
    MySPI_Stop();//停止
}

/**
  * 函    数：页擦除
  * 参    数：Address       要擦除那一页
  * 返 回 值：无
  * 注意事项：最小的擦除单位。4kb 1扇区
  */
void W25Q64_PageErase(uint32_t Address)
{
    W25Q64_WaitBusy();
    //事前等待。（事后等待是先等待再退出，比较保险。 事前等待可以先做别的事，再进去。效率高）
    
    W25Q64_WriteEnable();//写使能（每次写时都要先写使能）
    
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_SECTOR_ERASE_4KB);//发送页编程指令
    MySPI_WarpByte(Address >> 16 );
    MySPI_WarpByte(Address >> 8 );//发送地址
    MySPI_WarpByte(Address >> 0);
    MySPI_Stop();//停止
}

/**
  * 函    数：读取数据
  * 参    数：Address       要读取个地址
              *DataArray    存储字节所用的数组
              Count         一次读取多少字节
  * 返 回 值：无
  * 注意事项：读取可以无限制读取
  */
void W25Q64_ReadData(uint32_t Address, uint8_t* DataArray,uint32_t Count)//读取没有限制
{
    W25Q64_WaitBusy();
    //事前等待。（事后等待是先等待再退出，比较保险。 事前等待可以先做别的事，再进去。效率高）
    
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_READ_DATA);//发送页编程指令
    MySPI_WarpByte(Address >> 16 );
    MySPI_WarpByte(Address >> 8 );//（接收只能接收8位。会自动舍弃）
    MySPI_WarpByte(Address >> 0);//发送页地址
    uint32_t i = 0;
    for(i = 0; i < Count; i++)
    {
        DataArray[i] = MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收Count个字节，放到数组的第i位
    }
    MySPI_Stop();//停止
}




			```
		### W25Q64.h {toggle="true"}
			```javascript
#ifndef __W25Q64_H
#define __W25Q64_H

//初始化W25Q64
void W25Q64_Init(void);


/*厂商和设备ID号*/
typedef struct ID
{
    uint8_t MID;//8位厂商ID
    uint16_t DID;//16位设备ID
}ID;
extern ID W25Q64_ID;

//获取厂商和设备号ID
void W25Q64_ReadID(ID* Str);
//页编程
void W25Q64_PageProgram(uint32_t Address, uint8_t* DataArray,uint16_t Count);
//页擦除
void W25Q64_PageErase(uint32_t Address);
//读取
void W25Q64_ReadData(uint32_t Address, uint8_t* DataArray,uint32_t Count);
#endif

			```
		### W25Q64_Ins.h {toggle="true"}
			```javascript
#ifndef __W25Q64_INS_H
#define __W25Q64_INS_H

#define W25Q64_WRITE_ENABLE	                        0x06
#define W25Q64_WRITE_DISABLE                        0x04
#define W25Q64_READ_STATUS_REGISTER_1               0x05
#define W25Q64_READ_STATUS_REGISTER_2               0x35
#define W25Q64_WRITE_STATUS_REGISTER                0x01
#define W25Q64_PAGE_PROGRAM                         0x02
#define W25Q64_QUAD_PAGE_PROGRAM                    0x32
#define W25Q64_BLOCK_ERASE_64KB	                    0xD8
#define W25Q64_BLOCK_ERASE_32KB	                    0x52
#define W25Q64_SECTOR_ERASE_4KB	                    0x20
#define W25Q64_CHIP_ERASE                           0xC7
#define W25Q64_ERASE_SUSPEND                        0x75
#define W25Q64_ERASE_RESUME                         0x7A
#define W25Q64_POWER_DOWN                           0xB9
#define W25Q64_HIGH_PERFORMANCE_MODE                0xA3
#define W25Q64_CONTINUOUS_READ_MODE_RESET           0xFF
#define W25Q64_RELEASE_POWER_DOWN_HPM_DEVICE_ID	    0xAB
#define W25Q64_MANUFACTURER_DEVICE_ID               0x90
#define W25Q64_READ_UNIQUE_ID                       0x4B
#define W25Q64_JEDEC_ID                             0x9F
#define W25Q64_READ_DATA                            0x03
#define W25Q64_FAST_READ                            0x0B
#define W25Q64_FAST_READ_DUAL_OUTPUT                0x3B
#define W25Q64_FAST_READ_DUAL_IO                    0xBB
#define W25Q64_FAST_READ_QUAD_OUTPUT                0x6B
#define W25Q64_FAST_READ_QUAD_IO                    0xEB
#define W25Q64_OCTAL_WORD_READ_QUAD_IO              0xE3
            
#define W25Q64_DUMMY_BYTE                           0xFF

#endif

			```
		### main.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
#include "W25Q64.h"

/**
  * 函    数：验证SPI控制W25Q64存储器
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */

uint8_t ArrWrite[] = {0x01,0x02,0x03,0x04};
uint8_t ArrRead[4];

int main()
{
    Delay_Init();//初始化演示
    OLED_Init();//初始化OLED;
    W25Q64_Init();//初始化W25Q64存储器

    OLED_ShowString(1,1,"MID:  ,DID:   ");
    
    W25Q64_ReadID(&W25Q64_ID);//读ID放到这个结构体中
    OLED_ShowHexNum(1,5,W25Q64_ID.MID,2);
    OLED_ShowHexNum(1,12,W25Q64_ID.DID,4);//显示MID DID
    
    OLED_ShowString(2,1,"W:");
    OLED_ShowString(3,1,"R:");
    
    W25Q64_PageErase(0x000000);               //擦除地址。写入前需要（最好定位到扇区的起始地址（后三位为0））
    W25Q64_PageProgram(0x000000,ArrWrite,4);  //写入数组中数据到存储器
    W25Q64_ReadData(0x000000,ArrRead,4);      //读取存储器中数据到数组
    
    OLED_ShowHexNum(2, 3, ArrWrite[0], 2);
    OLED_ShowHexNum(2 ,6, ArrWrite[1], 2);
    OLED_ShowHexNum(2, 9, ArrWrite[2], 2);
    OLED_ShowHexNum(2, 12, ArrWrite[3], 2);
    
    OLED_ShowHexNum(3, 3, ArrRead[0], 2);
    OLED_ShowHexNum(3 ,6, ArrRead[1], 2);
    OLED_ShowHexNum(3, 9, ArrRead[2], 2);
    OLED_ShowHexNum(3, 12, ArrRead[3], 2);




    while(1)
    {

    }
}




			```
	## 7. 验证Flash注意事项 {toggle="true"}
		### 1. Flash操作注意事项 {toggle="true"}
			**写入操作时：**
			- 写入操作前，必须先进行写使能
			- **每个数据位只能由1改写为0，不能由0改写为1**
				所以**写入数据前必须先擦除**，擦除后，所有数据位变为1
			- 擦除必须按**最小擦除单元进行**（整个芯片、按块、按扇区）
			- 连续写入多字节时，**最多写入一页的数据，超过页尾位置的数据**，**会回到页首覆盖写入**<br>写入操作结束后，芯片进入忙状态，不响应新的读写操作
			**读取操作时：**
			- 直接调用读取时序，无需使能，无需额外操作，没有页的限制，读取操作结束后不会进入忙状态，但不能在忙状态时读取
		### 2. 验证Flash擦除之后变为全1 {toggle="true"}
			只擦除，不写入，直接读取
			```javascript
    W25Q64_PageErase(0x000000);               //擦除地址。写入前需要（最好定位到扇区的起始地址（后三位为0））
//    W25Q64_PageProgram(0x000000,ArrWrite,4);  //写入数组中数据到存储器
    W25Q64_ReadData(0x000000,ArrRead,4);      //读取存储器中数据到数组
			```
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ad50a54e-97dc-4758-8946-ad5f7eced3e0.webp)
		### 3. 验证**每个数据位只能由1改写为0，不能由0改写为1** {toggle="true"}
			**当前**`0x000000`地址存储为 `0xAA 0xBB 0xCC 0xDD`
			不擦除直接写入
			```javascript
    //W25Q64_PageErase(0x000000);               //擦除地址。写入前需要（最好定位到扇区的起始地址（后三位为0））
    W25Q64_PageProgram(0x000000,ArrWrite,4);  //写入数组中数据到存储器
    W25Q64_ReadData(0x000000,ArrRead,4);      //读取存储器中数据到数组
			```
			1. 直接写入`0xFF 0xFF 0xFF 0xFF `
				结果。字节中0并未改变
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ea4831ad-1541-43f7-8ab4-85c59e704a20.webp)
			2. 直接写入`0x00 0x00 0x00 0x00 `
				结果。字节中1全部改变为0
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/7bab816a-fb14-42cc-86a3-c1ab8eb21d26.webp)
		### 4.**最**多写入一页的数据，超过页尾位置的数据会回到页首覆盖写入 {toggle="true"}
			一页的范围是xxxx00到xxxxFF。因此他能存放（0-255）256个字节
			现在指定写入的地址为`0x0000FF`。写入`0xAA 0xBB 0xCC 0xDD`
			那么按照规则来说，就会在页尾写入0xAA、页首写入 0xBB 0xCC 0xDD
			而另一页并不会写入数据。而读取可以跨页
			**从页尾开始读取**结果如下：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/7596b57f-3863-4656-90d6-beefb787762d.webp)
			从页首开始读取结果如下：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/539458c4-b4b2-4401-8640-03005b0d7b24.webp)
# **25.   STM32通过SPI硬件读写W25Q64** {toggle="true"}
	## **1. STM32的SPI外设简介** {toggle="true"}
		- STM32内部集成了硬件SPI收发电路，可以由硬件自动执行时钟生成、数据收发等功能，减轻CPU的负担
		- 可配置8位/16位数据帧、高位先行/低位先行
		- 时钟频率:fpclk/(2,4,8,16,32,64,128,256)<br>PCLK指的是总线时钟，比如APB1 APB2<br>所以最快的速度就是分频2了。
		- 支持多主机模型、主或从操作
		- 可精简为半双工/单工通信（很少用）
		- 支持DMA
		- 兼容12S协议（音频专用）
		<empty-block/>
		- STM32F103C8T6 硬件SPI资源:SPI1、SPI2（SPI1为APB2外设）（其他位APB1）
	## **2. STM32的SPI框图** {toggle="true"}
		**这个框图是在做主机时的框图、移位寄存器右移**
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/b660ee06-4ca7-4d90-8933-8e071400b0db.webp)
		### **2.1 数据寄存器和移位寄存器（左上角部分）** {toggle="true"}
			**这里和I2C、串口的设计是异曲同工的。**
			**他们的目的都是实现连续的数据流**
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/463126ff-541d-467b-a772-18b870759aa8.webp)
			1. **移位寄存器**
				- 移位寄存器可以由LSBFIRST控制位  控制是左移还是右移（0高位先、1低位先）
				- 此时电路为右移状态。也就是低位先行
			2. **MOSI和MISO**
				- 数据通过移位寄存器 一位一位 的从MOSI移出去。
				- MISO的数据，一位一位的 移入到移位寄存器
				- 输出部分的MOSI和MISO进行了交叉，这个是为了主从模式变换的
			3. **接收缓冲区 和发送缓冲区**
				- 接收缓冲区叫做RDR
				- 发送缓冲区叫做TDR
				- TDR和RDR占用同一个地址，统一叫做DR
				- **在TDR转入移位寄存器时会产生TXE标志位为1。表示发送寄存器为空。**这时候就可以填装下一个字节了。实现不间断的连续发送
				- 在移位寄存器移出完成时，数据移入也会同步完成。这时移入的数据会整体的转入到接收缓冲区RDR。**接收到后，会置RXNE标志位为1。表示接收寄存器非空。**这时就可以在下一个数据来之前，读出RDR。实现连续的接收
		### 控制逻辑（其余右下角的部分） {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3e8088d9-6e6f-4c89-9344-cb4d433293a1.webp)
			1. **波特率发生器**
				- 输入时钟为PCLK （36M或者72M）
				- 经过分频之后输出到SCK引脚
				- 与移位寄存器同步。每产生一个时钟，移位寄存器移入移出一个bit
				- CR1寄存器的BR0、BR1、BR2用来控制分频系数。写入不同的值，可以对PCLK时钟执行2\~256分频。得到SCK时钟
					<table>
<tr>
<td>000      /2 </td>
<td>001      /4</td>
<td>010      /8</td>
<td>011      /16</td>
</tr>
<tr>
<td>100      /32</td>
<td>101      /64</td>
<td>110      /128</td>
<td>111      /256</td>
</tr>
					</table>
			2. 其他的通信电
				这些电路都是黑盒子电路。这里挑选几个重点的讲
				- LSBFIRST — 决定高位先行还是低位先行
				- SPE —（SPI ENABLE）决定SPI使能
				- BR —（Baud Rate）配置波特率，也就是SCK
				- MSTR —（Master）配置主从模式
				- CPOL和CPHA — 选择SPI的四种模式
				CR1寄存器
				- TXE — 发送寄存器空
				- RXNE — 接收寄存器非空
				CR2寄存器
				- 中断使能..DMA使能等
			3. 最后电路的左下角还有一个NSS位也就是SS位（不过我们一般直接用GPIO模拟，这个位置更偏向用于使用到多主机时采用的）
				简要了解一下
				- 多主机时，所有的主机连接到一条NSS线上。当别的NSS置0时。代表现在别人已经是主机了。会沿着线到一个数据选择器。告诉这个机器，已经有设备是主机了。自己不能跟他抢。
	## 3.STM32的SPI基本框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/20b137d9-d608-42b1-bff2-57cacefe328b.webp)
	## 4. STM32的SPI主模式全双工连续传输 时序图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/68935ace-5419-41fc-95ef-9ff94d907cd5.webp)
	## 5. STM32的SPI主模式全双工非连续传输 时序图 {toggle="true"}
		这里可以理解为，一个一个发，发完一个收一个。收完之后再发下一个
		并不会像连续一样 提前写入一个字节候着。会造成一定的资源浪费，拖慢传输速度
		- 适用于对传输速率要求不高的场合。
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f33bdab7-48ad-4be8-a264-eba7932dc9f3.webp)
	## 6. STM32非连续传输时，不同时钟速率的区别 {toggle="true"}
		因为在非连续传输时，数据不是连续的流。所以在软件读取后才发送会浪费很长时间。
		对于CLK时钟频率越快的情况下，造成资源浪费的情况就显得越严重
		### 256分频 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a2c74ef3-fd81-44d7-b157-cad85b311cb8.webp)
		### 128分频 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e0cf54ec-73d9-442b-9f16-31a0b0d9af19.webp)
		### 64分频 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/102fa48e-9013-47e9-b7a3-9301a02b74c0.webp)
		### 2分频（32MHZ） {toggle="true"}
			这里是因为他的频率已经超过我的示波器的采样频率了。但是也是可以看个大概的
			可以看到 ，时间浪费就很严重。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/d766dda2-4bdb-4bde-adc8-8b5b2fd3a86c.webp)
	## 7.  编写SPI硬件读写W25Q64注意事项 {toggle="true"}
		1. 硬件SPI外设不能随意指定，只能看复用到的引脚列表。
			其中NSS片选引脚可以自己指定。用他指定的也可以
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c5771d62-7058-430e-9788-a33696717fb0.webp)
			<empty-block/>
	## 8. STM32的SPI外设相关库函数（只看SPI的） {toggle="true"}
		`void SPI_I2S_DeInit(SPI_TypeDef* SPIx);`
			- I2S  恢复缺省配置
		> `void SPI_Init(SPI_TypeDef* SPIx, SPI_InitTypeDef* SPI_InitStruct);`
			- SPI  初始化
		`void I2S_Init(SPI_TypeDef* SPIx, I2S_InitTypeDef* I2S_InitStruct);`
			- I2S  初始化
		> `void SPI_StructInit(SPI_InitTypeDef* SPI_InitStruct);`
			- SPI  初始化
		`void I2S_StructInit(I2S_InitTypeDef* I2S_InitStruct);`
		> `void SPI_Cmd(SPI_TypeDef* SPIx, FunctionalState NewState);`
			- SPI  外设使能
		`void I2S_Cmd(SPI_TypeDef* SPIx, FunctionalState NewState);`
			- I2S  使能
		`void SPI_I2S_ITConfig(SPI_TypeDef* SPIx, uint8_t SPI_I2S_IT, FunctionalState NewState);`
			- I2S  中断配置
		`void SPI_I2S_DMACmd(SPI_TypeDef* SPIx, uint16_t SPI_I2S_DMAReq, FunctionalState NewState);`
			- I2S  DMA配置
		> `void SPI_I2S_SendData(SPI_TypeDef* SPIx, uint16_t Data);`
			- I2S  写数据到TDR寄存器（发送）
		> `uint16_t SPI_I2S_ReceiveData(SPI_TypeDef* SPIx);`
			- I2S  读DR数据寄存器（接收）
		`void SPI_NSSInternalSoftwareConfig(SPI_TypeDef* SPIx, uint16_t SPI_NSSInternalSoft);`
			- NSS引脚配置
		`void SPI_SSOutputCmd(SPI_TypeDef* SPIx, FunctionalState NewState);`
			- NSS引脚配置
		`void SPI_DataSizeConfig(SPI_TypeDef* SPIx, uint16_t SPI_DataSize);`
			- 8位或16位数据帧配置
		`void SPI_TransmitCRC(SPI_TypeDef* SPIx);`
			- CRC校验配置
		`void SPI_CalculateCRC(SPI_TypeDef* SPIx, FunctionalState NewState);`
			- CRC校验配置
		`uint16_t SPI_GetCRC(SPI_TypeDef* SPIx, uint8_t SPI_CRC);`
			- CRC校验配置
		`uint16_t SPI_GetCRCPolynomial(SPI_TypeDef* SPIx);`
			- CRC校验配置
		`void SPI_BiDirectionalLineConfig(SPI_TypeDef* SPIx, uint16_t SPI_Direction);`
			- 半双时，双向线方向配置
		> `FlagStatus SPI_I2S_GetFlagStatus(SPI_TypeDef* SPIx, uint16_t SPI_I2S_FLAG);`
			- 获取标志位
		> `void SPI_I2S_ClearFlag(SPI_TypeDef* SPIx, uint16_t SPI_I2S_FLAG);`
			- 清除标志位
		`ITStatus SPI_I2S_GetITStatus(SPI_TypeDef* SPIx, uint8_t SPI_I2S_IT);`
			- 获取中断标志位
		`void SPI_I2S_ClearITPendingBit(SPI_TypeDef* SPIx, uint8_t SPI_I2S_IT);`
			- 清除中断标志位
	## 7.编写SPI硬件读写W25Q64步骤 {toggle="true"}
		1. 开启GPIO和SPI外设时钟
		2. 初始化GPIO口
			- MISO为上拉输入，SCK、MOSI复用推挽输出
			- SS 通用推挽输出
		3. 配置SPI外设
			- 用结构体即可
		4. 开关控制
			- 使能
		5. 参考时序图编写代码
	## 8. 编写STM32SPI主模式全双工非连续传输 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f33bdab7-48ad-4be8-a264-eba7932dc9f3.webp)
		### 8.1 步骤 {toggle="true"}
			1. 等待发送寄存器为空标志位TXE = 1
			2. 软件写入数据到发送寄存器
			3. 等待接收完成（这时发送也一定完成）
			4. 读取、返回RDR
			- ！！注意其中的软件清除 是不需要我们手动清除的。<br>因为在我们写入TDR时会顺便清除TXE标志位、读取RDR时会清除RXNE标志位
			**所以这个只需要把底层的MySPI.c代码改一下就可以了**
		### **8.2 程序文件简要说明：** {toggle="true"}
			- MySPI：完成SPI的三个基本时序
			- MySPI.h：函数声明
			- W25Q64.c：初始化W25Q64存储寄存器。完成读写、擦除存储器的时序
			- W25Q64.h：函数声明、数据结构体声明
			- W25Q64_Ins.h：控制存储器时需要用到的指令集
			- main.c：测试硬件SPI读取存储器结果。
		### MySPI.c（相对软件，硬件仅需修改此文件） {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header

/*所用引脚列表*/
#define RCC_GPIO        RCC_APB2Periph_GPIOA
#define RCC_SPI1        RCC_APB2Periph_SPI1
#define SCK_Port        GPIOA
#define SCK_Pin         GPIO_Pin_5
#define SS_Port         GPIOA
#define SS_Pin          GPIO_Pin_4
#define MOSI_Port       GPIOA
#define MOSI_Pin        GPIO_Pin_7
#define MISO_Port       GPIOA
#define MISO_Pin        GPIO_Pin_6

/**
  * 函    数：写片选信号SS
  * 参    数：BitValue：输入1片选信号SS为高电平
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_W_SS (uint8_t BitValue)
{
    GPIO_WriteBit(SS_Port,SS_Pin,(BitAction)BitValue);
}

/**
  * 函    数：MySPI初始化
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Init(void)
{   
    /*配置SPI、GPIO外设时钟*/
    RCC_APB2PeriphClockCmd(RCC_SPI1,ENABLE);
    RCC_APB2PeriphClockCmd(RCC_GPIO,ENABLE);
    /*配置引脚*/
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Pin = SCK_Pin;
    GPIO_Init(SCK_Port,&GPIO_InitStructure);    //时钟配置为 复用 推挽输出
    
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
    GPIO_InitStructure.GPIO_Pin = MOSI_Pin;
    GPIO_Init(MOSI_Port,&GPIO_InitStructure);   //MOSI配置为 复用 推挽输出
    
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Pin = SS_Pin;
    GPIO_Init(SS_Port,&GPIO_InitStructure);     //片选配置为 通用 推挽输出
    
    
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
    GPIO_InitStructure.GPIO_Pin = MISO_Pin;
    GPIO_Init(MISO_Port,&GPIO_InitStructure);   //MISO配置为 上拉输入
    
    /*配置SPI*/
    SPI_InitTypeDef SPI_InitSturcture;
    SPI_InitSturcture.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_128;//分频系数
    SPI_InitSturcture.SPI_CPHA = SPI_CPHA_1Edge;    //第1个边沿采样（读入）
    SPI_InitSturcture.SPI_CPOL = SPI_CPOL_Low;      //CSK默认低电平    == 模式0
    SPI_InitSturcture.SPI_CRCPolynomial = 7;         //指定CRC计算的多项式（默认为7）
    SPI_InitSturcture.SPI_DataSize = SPI_DataSize_8b;//8位数据帧
    SPI_InitSturcture.SPI_Direction = SPI_Direction_2Lines_FullDuplex;  //两根线全双工
    SPI_InitSturcture.SPI_FirstBit = SPI_FirstBit_MSB;//高位先行
    SPI_InitSturcture.SPI_Mode = SPI_Mode_Master;   //主机模式
    SPI_InitSturcture.SPI_NSS = SPI_NSS_Soft;   //软件NSS
    SPI_Init(SPI1, &SPI_InitSturcture);
    
    /*使能SPI*/
    SPI_Cmd(SPI1,ENABLE);
    
    MySPI_W_SS(1);//初始化不选中从机
}







/*******************/
/*SPI的三个时序单元*/
/*******************/





/**
  * 函    数：起始信号
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Start(void)
{   
    MySPI_W_SS(0);
}

/**
  * 函    数：终止条件
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void MySPI_Stop(void)
{   
    MySPI_W_SS(1);
}

/**
  * 函    数：交换一个字节（模式0）（非连续）
  * 参    数：SendByte      待发送的字节
  * 返 回 值：ReceiveByte   接收到的字节
  * 注意事项：这是使用了移位模型的方式。效率更快
              如果要改为模式1，则先上升沿再发送。先下降沿再接收（2、3则直接改时钟极性就ok了）
  */
uint8_t MySPI_WarpByte(uint8_t SendByte)
{
    //等待TXE标志位为1 ，发送寄存器空
    while(SPI_I2S_GetFlagStatus(SPI1,SPI_I2S_FLAG_TXE) != SET);
    //软件写入数据到发送寄存器
    SPI_I2S_SendData(SPI1,SendByte);
    //等待接收完成（这时发送也一定完成）
    while(SPI_I2S_GetFlagStatus(SPI1,SPI_I2S_FLAG_RXNE) != SET);
    //读取RDR
    return SPI_I2S_ReceiveData(SPI1);
} 





			```
		### MySPI.h {toggle="true"}
			```javascript
#ifndef __MYSPI_H
#define __MYSPI_H

//初始化
void MySPI_Init(void);
//起始
void MySPI_Start(void);
//终止
void MySPI_Stop(void);
//交换
uint8_t MySPI_WarpByte(uint8_t SendByte);

#endif

			```
		### W25Q64.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "W25Q64.h"
#include "W25Q64_Ins.h"
#include "MySPI.h"


/**
  * 函    数：初始化W25Q64
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_Init(void)
{
    MySPI_Init();
}


/********************/
/*拼接完整的通信时序*/
/********************/

/**
  * 函    数：查看W25Q64的厂商号和设备号
  * 参    数：ID* Str   存放了ID结构体的指针
  * 返 回 值：无
  * 注意事项：接收第八位时是|=
  */
ID W25Q64_ID;//存放设备ID号的结构体
void W25Q64_ReadID(ID* Str)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_JEDEC_ID);//发送读取设备号指令。返回值不要
    Str->MID = MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收厂商ID 从机发来的设备号。发送值随便
    Str->DID = MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收设备ID高八位
    Str->DID <<= 8;//把接收到的数据放到高八位
    Str->DID |= MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收设备ID低八位
    MySPI_Stop();//停止
}

/**
  * 函    数：写使能
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_WriteEnable(void)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_WRITE_ENABLE);//发送
    MySPI_Stop();//停止
}

/**
  * 函    数：写失能
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_WriteDisable(void)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_WRITE_DISABLE);//发送
    MySPI_Stop();//停止
}

/**
  * 函    数：等待忙函数
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
void W25Q64_WaitBusy(void)
{
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_READ_STATUS_REGISTER_1);  //发送
    
    uint32_t TimeOut = 100000;
    while((MySPI_WarpByte(W25Q64_DUMMY_BYTE)&0x01) == 0x01)    //读取状态寄存器1的Busy位是否为1,为1则等待
    {
        TimeOut--;
        if(TimeOut == 0)
        {
            break;  //超时退出
        }
    }
    MySPI_Stop();   //停止
}

/**
  * 函    数：页编程
  * 参    数：Address       要写入那个页地址
              *DataArray    存储字节所用的数组
              Count         一次写入多少字节
  * 返 回 值：无
  * 注意事项：一次只能写入最多0-256个字节
  */
void W25Q64_PageProgram(uint32_t Address, uint8_t* DataArray,uint16_t Count)//(0-256,所以要16位)
{
    W25Q64_WaitBusy();
    //事前等待。（事后等待是先等待再退出，比较保险。 事前等待可以先做别的事，再进去。效率高）
    
    W25Q64_WriteEnable();//写使能（每次写时都要先写使能）
    
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_PAGE_PROGRAM);//发送页编程指令
    MySPI_WarpByte(Address >> 16 );
    MySPI_WarpByte(Address >> 8 );//（接收只能接收8位。会自动舍弃）
    MySPI_WarpByte(Address >> 0);//发送页地址
    uint16_t i = 0;
    for(i = 0; i < Count; i++)
    {
        MySPI_WarpByte(DataArray[i]);//发送Count个数组的第i位
    }
    MySPI_Stop();//停止
}

/**
  * 函    数：页擦除
  * 参    数：Address       要擦除那一页
  * 返 回 值：无
  * 注意事项：最小的擦除单位。4kb 1扇区
  */
void W25Q64_PageErase(uint32_t Address)
{
    W25Q64_WaitBusy();
    //事前等待。（事后等待是先等待再退出，比较保险。 事前等待可以先做别的事，再进去。效率高）
    
    W25Q64_WriteEnable();//写使能（每次写时都要先写使能）
    
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_SECTOR_ERASE_4KB);//发送页编程指令
    MySPI_WarpByte(Address >> 16 );
    MySPI_WarpByte(Address >> 8 );//发送地址
    MySPI_WarpByte(Address >> 0);
    MySPI_Stop();//停止
}

/**
  * 函    数：读取数据
  * 参    数：Address       要读取个地址
              *DataArray    存储字节所用的数组
              Count         一次读取多少字节
  * 返 回 值：无
  * 注意事项：读取可以无限制读取
  */
void W25Q64_ReadData(uint32_t Address, uint8_t* DataArray,uint32_t Count)//读取没有限制
{
    W25Q64_WaitBusy();
    //事前等待。（事后等待是先等待再退出，比较保险。 事前等待可以先做别的事，再进去。效率高）
    
    MySPI_Start();//起始
    MySPI_WarpByte(W25Q64_READ_DATA);//发送页编程指令
    MySPI_WarpByte(Address >> 16 );
    MySPI_WarpByte(Address >> 8 );//（接收只能接收8位。会自动舍弃）
    MySPI_WarpByte(Address >> 0);//发送页地址
    uint32_t i = 0;
    for(i = 0; i < Count; i++)
    {
        DataArray[i] = MySPI_WarpByte(W25Q64_DUMMY_BYTE);//接收Count个字节，放到数组的第i位
    }
    MySPI_Stop();//停止
}




			```
		### W25Q64.h {toggle="true"}
			```javascript
#ifndef __W25Q64_H
#define __W25Q64_H

//初始化W25Q64
void W25Q64_Init(void);


/*厂商和设备ID号*/
typedef struct ID
{
    uint8_t MID;//8位厂商ID
    uint16_t DID;//16位设备ID
}ID;
extern ID W25Q64_ID;

//获取厂商和设备号ID
void W25Q64_ReadID(ID* Str);
//页编程
void W25Q64_PageProgram(uint32_t Address, uint8_t* DataArray,uint16_t Count);
//页擦除
void W25Q64_PageErase(uint32_t Address);
//读取
void W25Q64_ReadData(uint32_t Address, uint8_t* DataArray,uint32_t Count);
#endif

			```
		### W25Q64_Ins.h {toggle="true"}
			```javascript
#ifndef __W25Q64_INS_H
#define __W25Q64_INS_H

#define W25Q64_WRITE_ENABLE	                        0x06
#define W25Q64_WRITE_DISABLE                        0x04
#define W25Q64_READ_STATUS_REGISTER_1               0x05
#define W25Q64_READ_STATUS_REGISTER_2               0x35
#define W25Q64_WRITE_STATUS_REGISTER                0x01
#define W25Q64_PAGE_PROGRAM                         0x02
#define W25Q64_QUAD_PAGE_PROGRAM                    0x32
#define W25Q64_BLOCK_ERASE_64KB	                    0xD8
#define W25Q64_BLOCK_ERASE_32KB	                    0x52
#define W25Q64_SECTOR_ERASE_4KB	                    0x20
#define W25Q64_CHIP_ERASE                           0xC7
#define W25Q64_ERASE_SUSPEND                        0x75
#define W25Q64_ERASE_RESUME                         0x7A
#define W25Q64_POWER_DOWN                           0xB9
#define W25Q64_HIGH_PERFORMANCE_MODE                0xA3
#define W25Q64_CONTINUOUS_READ_MODE_RESET           0xFF
#define W25Q64_RELEASE_POWER_DOWN_HPM_DEVICE_ID	    0xAB
#define W25Q64_MANUFACTURER_DEVICE_ID               0x90
#define W25Q64_READ_UNIQUE_ID                       0x4B
#define W25Q64_JEDEC_ID                             0x9F
#define W25Q64_READ_DATA                            0x03
#define W25Q64_FAST_READ                            0x0B
#define W25Q64_FAST_READ_DUAL_OUTPUT                0x3B
#define W25Q64_FAST_READ_DUAL_IO                    0xBB
#define W25Q64_FAST_READ_QUAD_OUTPUT                0x6B
#define W25Q64_FAST_READ_QUAD_IO                    0xEB
#define W25Q64_OCTAL_WORD_READ_QUAD_IO              0xE3
            
#define W25Q64_DUMMY_BYTE                           0xFF

#endif

			```
		### main.c {toggle="true"}
			```javascript
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "key.h"
#include "W25Q64.h"

/**
  * 函    数：验证SPI控制W25Q64存储器
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */

uint8_t ArrWrite[] = {0x01,0x02,0x03,0x04};
uint8_t ArrRead[4];

int main()
{
    Delay_Init();//初始化演示
    OLED_Init();//初始化OLED;
    W25Q64_Init();//初始化W25Q64存储器

    OLED_ShowString(1,1,"MID:  ,DID:   ");
    
    W25Q64_ReadID(&W25Q64_ID);//读ID放到这个结构体中
    OLED_ShowHexNum(1,5,W25Q64_ID.MID,2);
    OLED_ShowHexNum(1,12,W25Q64_ID.DID,4);//显示MID DID
    
    OLED_ShowString(2,1,"W:");
    OLED_ShowString(3,1,"R:");
    
    W25Q64_PageErase(0x000000);               //擦除地址。写入前需要（最好定位到扇区的起始地址（后三位为0））
    W25Q64_PageProgram(0x000000,ArrWrite,4);  //写入数组中数据到存储器
    W25Q64_ReadData(0x000000,ArrRead,4);      //读取存储器中数据到数组
    
    OLED_ShowHexNum(2, 3, ArrWrite[0], 2);
    OLED_ShowHexNum(2 ,6, ArrWrite[1], 2);
    OLED_ShowHexNum(2, 9, ArrWrite[2], 2);
    OLED_ShowHexNum(2, 12, ArrWrite[3], 2);
    
    OLED_ShowHexNum(3, 3, ArrRead[0], 2);
    OLED_ShowHexNum(3 ,6, ArrRead[1], 2);
    OLED_ShowHexNum(3, 9, ArrRead[2], 2);
    OLED_ShowHexNum(3, 12, ArrRead[3], 2);




    while(1)
    {

    }
}




			```
# **26.  STM32后备区域：读写BKP备份寄存器与使用RTC实时时钟详解** {toggle="true"}
	## 1   什么是STM32的后备区域 {toggle="true"}
		STM32 的后备区域是芯片内部的一个特殊区域。
		**特点和作用**：
		- 后备区域可以在主电源VCC断电之后由备用电源VBA提供供电。
		- 后备区域存储的数据不会因为复位而重置
		- 但是如果包括主电源和VBAT都断电了。那么备用区域也会清除，因为他们的存储器本质是RAM存储器，掉电丢失。
		**后备区域有什么：**
		- BKP备份寄存器
		- RTC实时时钟
	## \*\*\*\*\*\*\*\*\*分割线\*\*\*\*\*\*\*\*\*\* {toggle="true"}
		下面开始BKP部分
	## 2.1  BKP备份寄存器简介 {toggle="true"}
		**BKP（Backup Registers）备份寄存器（后备寄存器）**
		它位于后备区域。
		- BKP可用于存储数据。
		- **存储特性：**
			1. 当VDD（2.0\~3.6V）电源（主电源）被切断，后备区域仍然由VBAT备用电源（1.8\~3.6V）维持供电。
			2. 并且系统在待机模式下被唤醒，或系统复位或电源复位时，他们也不会被复位
			3. 但是如果备用电源VBAT和主电源VCC都断电了，就会清除数据，因为BKP本质是RAM存储器。掉电丢失数据
		- **STM32的TAMPER引脚**
			 他可以产生的侵入事件可以将所有备份寄存器内容清除
			1. TAMPER是用于引入检测信号（可以是或上升沿/下降沿）的，当发生入侵时，将清除BKP所有内容，并申请中断。
			2. 并且他是由备用电源供电，主电源断电后侵入检测仍然有效，以保证数据安全
		- **BKP的RTC引脚可以输出RTC校准时钟、RTC闹钟脉冲或者秒脉冲**
		- **BKP存储RTC时钟校准寄存器**
		- **STM32后备区域的供电特性：**
			1. 当VDD主电源掉电时，后备区域仍然可以由VBAT的备用电池供电。
			2. 当VDD主电源上电时，后备区域供电会自动从VBAT切换到VDD。
		- **BKP数据存储容量：**
			1. 20字节（中容量和小容量）/ 84字节（大容量和互联型）
		- **手册建议**：
			1. 如果没有外部电池，建议VBAT引脚接到VDD，就是VBAT和主电源接到一起，并且再连接一个100nf的滤波电容
	## 2.2  BKP备份寄存器基本结构 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/b51e1421-aec6-46f3-9f71-e0317b583dc9.webp)
		其中橙色部分就是后备区域，BKP处于后备区域。但后备区域不止有BKP。
		**STM32后备区域的特性：**
		- 当VDD主电源掉电时，后备区域仍然可以由VBAT的备用电池供电。
		- 当VDD主电源上电时，后备区域供电会自动从VBAT切换到VDD。
		**数据寄存器：**
		- 每个寄存器有16位（可存储2个字节）。
		- 中小容量设备有DR1\~DR10，总共10个数据寄存器，每个寄存器存储2个字节，总容量为20字节。
		- 大容量和互联型设备有42个数据寄存器（DR）。
		**TAMPER引脚：**
		- 用于引入检测信号（上升沿/下降沿），清除BKP所有内容以保证数据安全。
		**时钟输出：**
		- 可以将RTC相关时钟从PC13位置的RTC引脚输出出去，供外部使用。
		- 输出校准时钟时，可以配合校准寄存器对RTC的误差进行校准。
	## 2.3  BKP外设头文件 bkp.h介绍 {toggle="true"}
		`void BKP_DeInit(void);`
		- 恢复缺省配置（用于清空BKP所有BKP寄存器）
		`void BKP_TamperPinLevelConfig(uint16_t BKP_TamperPinLevel);`
		- 配置TAMPER引脚的有效电平
		`void BKP_TamperPinCmd(FunctionalState NewState);`
		- 是否开启侵入检测功能
		`void BKP_ITConfig(FunctionalState NewState);`
		- 是否开启中断
		`void BKP_RTCOutputConfig(uint16_t BKP_RTCOutputSource);`
		- 时钟输出功能配置（在RTC引脚上输出时钟信号、RTC校准时钟、RTC闹钟脉冲、秒脉冲）
		`void BKP_SetRTCCalibrationValue(uint8_t CalibrationValue);`
		- 设置RTC校准值
		> `void BKP_WriteBackupRegister(uint16_t BKP_DR, uint16_t Data);`
			- 写BKP备份寄存器
		> `uint16_t BKP_ReadBackupRegister(uint16_t BKP_DR);`
			- 读BKP寄存器
		`FlagStatus BKP_GetFlagStatus(void);`
		查看标志位
		`void BKP_ClearFlag(void);`
		清除标志位
		`ITStatus BKP_GetITStatus(void);`
		查看中断标志位
		`void BKP_ClearITPendingBit(void);`
		清除中断标志位
		> pwr.h中也有一个函数是需要用到的<br>`void PWR_BackupAccessCmd(FunctionalState NewState);`
			- 备份寄存器访问使能（其实是设置BDP位）
	## 2.4  读写 BKP备份寄存器 操作步骤 {toggle="true"}
		1. 开**启PWR和BKP的时钟**
			- PWR的开启函数为`RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);<br>`它的目的是开启后备电源的时钟（可以理解为开启后备电源VBAT）。
			- BKP的开启函数`RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP, ENABLE);`<br>他的目的是开启BKP外设的时钟，可以看到他们都是APB1总线下的。
		2. 使能备份区域的访问
			- 因为RTC实时时钟和BKP备份寄存器，都处于备份区域中。在STM32中，想访问RTC和BKP，就要先开启备份区域的访问权限。它在PwR.h中<br>函数：`PWR_BackupAccessCmd(ENABLE);`
		3. **读写操作**
			- 写入：`BKP_WriteBackupRegister(BKP_DR1,Data);`
			- 读出：`Data = BKP_ReadBackupRegister(BKP_DR1);`
	## 2.5  编写 读写备份寄存器 {toggle="true"}
		### 5.1 文件介绍 {toggle="true"}
			- main.c  ： 读写测试BKP备份寄存器是否在单主电源掉电时丢失数据。是否受到复位影响…
		### main.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "oled.h"
#include "Delay.h"
#include "Key.h"


/**
  * 函    数：STM32 BKP备份寄存器的读写测试
  * 参    数：无
  * 返 回 值：无
  * 注意事项：无
  */
  
uint16_t WriteArr[] = {0x0000, 0x0001};
uint16_t ReadArr[2] = { 0 };
  
int main()
{
    OLED_Init();//初始化OLED;
    KEY_Init();//初始化按键
    
    //使能时钟电源和后备接口时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR,ENABLE);
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP,ENABLE);
    
    //使能备份访问控制
    PWR_BackupAccessCmd(ENABLE);
    
    OLED_ShowString(1,1,"W:");
    OLED_ShowString(2,1,"R:");
    while(1)
    {
        if(KEY_Get() == 1)
        {
            //写入BKP
            BKP_WriteBackupRegister(BKP_DR1,WriteArr[0]);
            BKP_WriteBackupRegister(BKP_DR2,WriteArr[1]);
            
            //显示写入的值
            OLED_ShowHexNum(1,3,WriteArr[0],4);
            OLED_ShowHexNum(1,8,WriteArr[1],4);
            
            WriteArr[0]++;
            WriteArr[1]++;
        }
        //读取BKP
        ReadArr[0] = BKP_ReadBackupRegister(BKP_DR1);
        ReadArr[1] = BKP_ReadBackupRegister(BKP_DR2);
        //显示读取的值
        OLED_ShowHexNum(2,3,ReadArr[0],4);
        OLED_ShowHexNum(2,8,ReadArr[1],4);
        
    }
}




			```
	## \*\*\*\*\*\*\*\*\*分割线\*\*\*\*\*\*\*\*\*\* {toggle="true"}
		下面开始RTC部分
	## 3.1 什么是Unix时间戳 {toggle="true"}
		- Unix时间戳，最早是Unix系统使用的。所以叫Unix时间戳。
		- 目前Windows、linux、安卓等系统都是实用的Unix时间戳。
		- Unix 时间戳（Unix Timestamp）定义为从UTC/GMT的1970年1月1日0时0分0秒开始所经过的秒数，不考虑闰秒。
			- GMT（Greenwich Mean Time）**格林尼治**标准时间是一种以地球自转为基础的时间计量系统。它将地球自转一周的时间间隔等分为24小时，以此确定计时标准<br>（不过他不精准，因为地球自转是越来越慢的。所以又有了如下的规定）
				（格林尼治是伦敦的一个区）
			- UTC（Universal Time Coordinated）协调世界时是一种以原子钟为基础的时间计量系统。它规定铯133原子基态的两个超精细能级间在零磁场下跃迁辐射9,192,631,770周所持续的时间为1秒。当原子钟计时一天的时间与地球自转一周的时间相差超过0.9秒时，UTC会执行闰秒来保证其计时与地球自转的协调一致
			- 我们时间戳所说的1970年1月1日0时0分0秒。指的是伦敦伦敦时间的0时0秒。<br>其他的位置可以分为24个时区。每偏差一个时区，相应的时间就要加或减一个小时<br>
		- 时间戳存储在一个秒计数器中，秒计数器为32位/64位的整型变量
		- 世界上所有时区的秒计数器**相同**，不同时区通过**添加偏移**来得到当地时间<br>比如Unix时间戳为0，代表伦敦的0时0分，那么北京就是+8得到8时0分
		- C语言官方为我们提供了函数，可以直接把Unix时间戳转换为时间。
		- 可以在百度直接搜索unix在线时间戳。
	## 3.2  C语言中时间戳转换函数 {toggle="true"}
		### 2.1 C语言提供的time.h函数介绍 {toggle="true"}
			- C语言的time.h模块提供了时间获取和时间戳转换的相关函数，可以方便地进行秒计数器、日期时间和字符串之间的转换
			- 一些常用的函数如下：（粗细为更重要）
				<table>
				<colgroup>
				<col width="384">
				<col width="292.3333435058594">
				</colgroup>
<tr>
<td>`time_t time(time_t*);`</td>
<td>获取系统时钟</td>
</tr>
<tr>
<td>**`struct tm* gmtime(const time_t*);`**</td>
<td>秒计数器转换为日期时间（格林尼治时间）</td>
</tr>
<tr>
<td>**`struct tm* localtime(const time_t*);`**</td>
<td>秒计数器转换为日期时间（当地时间）</td>
</tr>
<tr>
<td>**`time_t mktime(struct tm*);`**</td>
<td>日期时间转换为秒计数器（当地时间）</td>
</tr>
<tr>
<td>`char* ctime(const time_t*);`</td>
<td>秒计数器转换为字符串（默认格式）</td>
</tr>
<tr>
<td>`char* asctime(const struct tm*);`</td>
<td>日期时间转换为字符串（默认格式）</td>
</tr>
<tr>
<td>`size_t strftime(char*, size_t, const char*, const struct tm*);`</td>
<td>日期时间转换为字符串（自定义格式）</td>
</tr>
				</table>
		### 2.2 时间戳转换图 {toggle="true"}
			这张图就清晰了显示了各个函数的作用：其实就是在各种数据类型之间进行转换。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/927ca6c5-b24a-4919-8138-12e33f095cac.webp)
			- **首先先了解一下各个数据类型是什么**
				1. 秒计数器数据类型：time_t，其实就是一个32或64位的有符号的整形数据。也就是64位的秒计数器（如果不特别声明，默认为64。）
				2. 日期时间数据类型：struct tm，这时一个结构体类型。成员如下：
					![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/4e494c0b-df4b-4a08-8dc3-9879802195f8.webp)
					- 秒、
					- 分、
					- 时、
					- 月的几日、
					- 月份、（需要+1偏移量）
					- 年份（需要+1900偏移量）、
					- 周某开始的星期几、从1月1日开始的第几天、
					- 是否使用夏令时<br>（是为了鼓励夏天时早睡早起节约用电设计的，目前个别国家在使用）
				3. 字符串型数据类型：char\*，用来指向一个表示时间的字符串
			**所以，在转换时，要根据函数的返回值，来进行相应赋值。比如****`struct tm* gmtime(const time_t*);`****的返回值为Struct tm的指针类型。那么在赋值给自己定义的Struct tm xxx 结构体时，就要先用\*取值，才能正确赋值**
			等等以此类推，需要根据不同的类型进行转换， <br><br>（time_t  整形、Struct tm 结构体  Char\* 字符指针..）
	## 3.3  RTC简介 {toggle="true"}
		**RTC (Real Time Clock)：实时时钟。**
		- RTC也位于STM32的后备区域中。可以由VBAT备用电源供电
		- RTC是一个独立的定时器，可为系统提供时钟和日历的功能。
		- RTC和时钟配置系统处于后备区域，系统复位时数据不清零，因为在VDD（2.0 - 3.6V）断电后可借助VBAT（1.8 - 3.6V）供电继续走时。
		- RTCCLK可选择三种RTC时钟源：
			1. HSE时钟除以128（通常为8MHz/128）
			2. LSE振荡器时钟（通常为32.768KHz）
			3. LSI振荡器时钟（40KHz）
	## 3.4  RTC时钟选择 {toggle="true"}
		**时钟信号解释**
		- HSE = 高速外部时钟信号
		- HSI = 高速内部时钟信号
		- LSI = 低速内部时钟信号
		- LSE = 低速外部时钟信号
		- H (High)：高速，L (Low)：低速，E (External)：外部，I (Internal)：内部
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8888c4b8-4201-4b57-b576-cf7313bd072a.webp)
		**时钟选择**
		- 外部高速时钟：一般作为系统主时钟。供内部程序运行和主要外设使用。
		- 内部低速时钟：主要用做看门狗等的使用。
		- 外部低速时钟：这个时钟一般都是为了给RTC提供时钟的。
			1. 因为**LSE外部低速时钟，才可以通过VBAT备用电池供电**，
			2. 所以在主电源断电情况下LSE可以继续震荡，实现RTC主电源掉电继续走时的功能。
			3. 并且他的频率为32.768KHZ。经过2\^15分频之后每次自然溢出时刚好为1s，也就是1HZ。 
	## 3.5   RTC框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ed7cac70-5ca8-459b-9a07-a79a73bafc47.webp)
		**灰色填充区域均是后备区域。**
		**可编程预分频器：**
		- RTC_CNT每秒自增，因此驱动计数器的时钟 TR_CLK 需要是1Hz的信号。<br>实际提供RTC模块的时钟（RTCCLK）频率较高，因此RTCCLK经过20位RTC预分频器（1\~2\^20分频），保证输出给计数器的频率为1Hz。
		**分频和计数：**
		- 输入时钟RTCCLK，经过RTC预分频器（由重装载寄存器RTC_PRL和余数寄存器RTC_DIV控制），计数器重装值ARR和CNT进行分频。
		**RTC_CNT：**
		- 可以作为Unix时间戳的秒计数器，再借用time.h的函数可以方便地得到年月日时分秒。
		**闹钟寄存器RTC_ALR：**
		- 32位寄存器，用来设置闹钟。设置闹钟时，将ALR写入一个秒数，当CNT的值等于ALR设定的闹钟值时，就会产生RTC_Alarm闹钟信号。<br>通过中断系统，在闹钟中断里执行相应操作。
		- 同时，闹钟信号可以让STM32退出待机模式。<br>此外，这个闹钟值是一个定值，只能响一次。若想实现周期闹钟，在每次闹钟响过后，都需要重新设置下一次闹钟时间。
		**中断信号：**
		- RTC_Second（秒中断）：来自于CNT的输入时钟。开启此中断后，程序会每秒进入一次RTC中断。
		- RTC_Overflow（溢出中断）：来自CNT右边，表示CNT的32位计数器计满溢出时触发一次中断。
		- RTC_Alarm（闹钟中断）：当计数器的值和闹钟值相等时触发中断，同时可以唤醒设备退出待机模式。
		**中断标志位和中断输出控制**
		- F（Flag） 结尾的是对应的中断标志位。
		- IE（Interrupt Enable） 结尾的是中断使能。
		- 最后三个信号通过一个或汇聚到NVIC中断控制器。
		**APB1总线和APB1接口：**
		- 程序读写寄存器的地方可以通过APB1总线完成，RTC位于APB1总线上的设备。<br>退出待机模式：唤醒机制
		- 闹钟信号和WKUP引脚都可以唤醒设备，退出待机模式。
	## 3.6  RTC基本框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/10600c2d-d0e3-4265-857c-1376cb20d0e2.webp)
		**时钟来源配置：**
		- 最左边的RTCCLK时钟来源在RCC中配置，可以从三个时钟中选择一个作为RTCCLK。
		**时钟预分频：**
		- 选择的RTCCLK经过预分频器对时钟进行分频。
		- 余数计数器是一个自减计数器，存储当前的计数值。
		- 重装寄存器决定计数目标和分频值。
		- 分频后得到1Hz的秒计数信号，传递给32位计数器，每秒自增一次。
		**闹钟设定：**
		- 32位计数器下有一个32位的闹钟值，可以设定闹钟时间。
		**中断信号触发：**
		- 右侧有三个信号可以触发中断：秒信号、计数器溢出信号和闹钟信号。
		- 这三个信号通过中断输出控制，进行中断使能。
		- 启用的中断信号才能传递到NVIC，然后向CPU申请中断。
		**程序配置步骤：**
		**配置数据选择器：**选择RTCCLK时钟来源。
		**配置重装寄存器：**选择分频系数。
		**配置32位计数器：**
		- 进行日期时间的读写。
		- 如果需要闹钟，配置32位闹钟值。
		**配置中断：**
		- 启用中断，再配置NVIC。
		- 最后，编写对应的中断函数。
	## 3.7  RTC硬件电路 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/8c7e79ee-98ae-4a71-a678-7645d1647b4a.webp)
		1. 备用电池供电
			- **简单连接（左侧）：**使用一个3V的电池B1直接连接到VBAT和GND。这样设计简单，但是电源冗余不高。
			- **推荐连接（中间）：**使用两个3V的电池B2和B3通过两个二极管D1和D2连接到VBAT和GND。这样设计增加了电源的可靠性，因为如果一个电池失效，另一个电池还能提供电源。电容C3（0.1uF）用于滤波，稳定电压。
		2. **外部低速晶振**
			- **晶振部分（中间）：**使用一个32.768kHz的晶振（X1）连接到两个10pF的电容（C1和C2），并接地。这部分电路提供了一个稳定的时钟信号，通常用于RTC（实时时钟）功能。
			- **连接到STM32单片机（右侧）：**OSC32_IN和OSC32_OUT分别连接到STM32单片机的PC14和PC15引脚。
		3. STM32单片机连接
			- **供电和地（右侧）：**VDD和VSS分别是电源和地，VDD连接到电源正极，VSS连接到地。VBAT连接到备用电池供电部分的输出。
			- **时钟信号（右侧）：**PC14和PC15分别连接到外部低速晶振的OSC32_IN和OSC32_OUT。
			- **其他引脚（右侧）：**图中列出了STM32F103C8T6单片机的引脚配置，包括PA0到PA15，PB0到PB15等。这些引脚可以根据具体应用进行配置。
	## 3.8 RTC头文件介绍 {toggle="true"}
		**`void RTC_ITConfig(uint16_t RTC_IT, FunctionalState NewState);`**
		- RTC 中断使能：通过传入指定的中断类型 `RTC_IT` 和状态 `NewState`，实现对 RTC 中断的使能或失能控制。
		**`void RTC_EnterConfigMode(void);`**
		- 进入 RTC 配置模式：用于进入 RTC 的配置状态，以便进行相关参数的修改。
		**`void RTC_ExitConfigMode(void);`**
		- 退出 RTC 配置模式：在完成 RTC 配置操作后，使用此函数退出配置模式。
		**`uint32_t RTC_GetCounter(void);`**
		- 获取 RTC 计数器的值：返回 RTC 计数器的当前数值。
		**`void RTC_SetCounter(uint32_t CounterValue);`**
		- 设置 RTC 计数器的值：将 RTC 计数器设置为指定的数值 `CounterValue`。
		**`void RTC_SetPrescaler(uint32_t PrescalerValue);`**
		- 设置 RTC 预分频的值：为 RTC 预分频设置特定的值 `PrescalerValue`。
		**`void RTC_SetAlarm(uint32_t AlarmValue);`**
		- 设置 RTC 闹钟的值：设定 RTC 闹钟的触发值为 `AlarmValue`。
		**`uint32_t RTC_GetDivider(void);`**
		- 获取 RTC 预分频分频因子的值：获取当前 RTC 预分频分频因子的数值。
		**`void RTC_WaitForLastTask(void);`**
		- 等待最近一次对 RTC 寄存器的写操作完成：确保之前对 RTC 寄存器的写入操作已经完成。
		**`void RTC_WaitForSynchro(void);`**
		- 等待 RTC 寄存器与 RTC 的 APB 时钟同步：等待 RTC 相关寄存器（如 `RTC_CNT`、`RTC_ALR` 和 `RTC_PRL`）与 APB 时钟完成同步。
		**`FlagStatus RTC_GetFlagStatus(uint16_t RTC_FLAG);`**
		- 检查指定的 RTC 标志位设置与否：通过传入标志位 `RTC_FLAG`，返回其状态。
		**`void RTC_ClearFlag(uint16_t RTC_FLAG);`**
		- 清除 RTC 的待处理标志位：清除指定的 RTC 标志位。
		**`ITStatus RTC_GetITStatus(uint16_t RTC_IT);`**
		- 检查指定的 RTC 中断发生与否：根据传入的中断类型 `RTC_IT`，判断中断是否发生。
		**`void RTC_ClearITPendingBit(uint16_t RTC_IT);`**
		- 清除 RTC 的中断待处理位：清除指定 RTC 中断的待处理位。
	## 3.9  使用 RTC实时时钟 操作步骤 {toggle="true"}
		**RTC使用时，一般都是和BKP一起使用的：**
		因为RTC在初始化时，需要配置其32位的时间戳计数器。
		但每次复位或者主电源上电后，在执行主程序时，会重新初始化RTC的时间戳计数器。这就导致了RTC实时时钟本来能在备份电源的供给下计时，但是上电后又给我重新初始化覆盖了。导致不实时了！
		**所以需要在初始化程序中判断是否需要设置32位的时间戳计数器。**
		一般的方法是，在第一次初始化RTC时，顺便在BKP备份寄存器中写入特定值。在下一次初始化RTC时，对BKP备份寄存器的值进行判断。如果之前没有写入，那么就初始化。否则不初始化。
		所以使用RTC实时时钟操作步骤如下：
		- **执行以下操作将使能对BKP和RTC的访问：**
			1. 开**启PWR和BKP的时钟**
				- PWR的开启函数为`RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);<br>`开启后备电源的时钟
				- BKP的开启函数`RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP, ENABLE);`<br>开启BKP外设的时钟
			2. 使能备份区域的访问
				- 在STM32中，想访问RTC和BKP，就要先开启备份区域的访问权限。
				函数：`PWR_BackupAccessCmd(ENABLE);`
		- **判断是否需要初始化RTC见代码部分**
		- **开启LSE时钟**
			1. 开启LSE时钟                    `RCC_LSEConfig(RCC_LSE_ON);` 
			2. 等待LSE时钟开启完毕   `while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) != SET);`
			3. 选择RTCCLK时钟为LSE`RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);`
			4. 使能RTCCLK时钟          `RCC_RTCCLKCmd(ENABLE);`
		- **等待时钟同步**
			1. 因为可能在恢复主电源之后，APB1总线刚刚恢复震荡频率，但是RTCCLK需要经过外部震荡源分频后才能有一次输出。如果直接读取，会导致读取不准确。<br>所以需要等待RTCCLK产生上升沿来激活更新一下时间戳计数器，这时APB1直接读取。才是准确的。<br>**所以软件读取时必须等待**RTCCLK来一个上升沿，使RTC_CRL寄存器中的**RSF位（寄存器同步标志）被硬件置1。**这时再读取RTC_CRL寄存器。才能正确读取。
				函数：`RTC_WaitForSynchro(); `  (等待时钟同步)
		- **等待写入完成**
			1. 对RTC任何寄存器的写操作，都必须在前一次写操作结束后进行。
			2. 可以通过查询RTC_CR寄存器中的RTOFF状态位，判断RTC寄存器是否处于更新中。<br>当RTOFF状态位是1时，才可以写入RTC寄存器
				函数：`RTC_WaitForLastTask();`
		- **设置RTC预分频器**
			1. 设置预分频器`RTC_SetPrescaler(32768 - 1);`
			2. 等待写入完成`RTC_WaitForLastTask();`
		- **写入前其实是需要设置RTC_CRL寄存器中的CNF位<br>（函数会帮我们自动完成，所以不需要）**
			1. 在写入时，必须设置RTC_CRL寄存器中的CNF位，<br>使RTC进入**配置模式**后，才能写入RTC_PRL（预分频器）、RTC_CNT（时间戳计数器）、RTC_ALR（闹钟寄存器）寄存器
		- **设置CNT时间戳计数器时间**
			（利用C语言中time.h来转换时间戳并写入，详见代码）
		- 在BKP备份寄存器中写入特定数据。为下次复位或仅主电源断电后上电时判断是否初始化打下基础
	## 3.10 编写RTC实时时钟显示年月日时分秒 {toggle="true"}
		### 3.10.1 文件介绍 {toggle="true"}
			- MyRTC.c 初始化RTC、通过C语言time.h函数来编写 时间转换时间戳、时间戳转换时间的函数
			- MyRTC.h 函数声明
			- main.c  测试MyRTC显示
		### MyRTC.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include <time.h>

uint16_t MyRTC_Time[] = {2024, 8, 18, 23, 46, 0};   //定义全局的时间数组，数组内容分别为年、月、日、时、分、秒

void MyRTC_SetTime(void);           //函数声明

/**
  * 函    数：RTC初始化
  * 参    数：无
  * 返 回 值：无
  */
void MyRTC_Init(void)
{
    /*开启时钟*/
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);     //开启PWR的时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP, ENABLE);     //开启BKP的时钟
    
    /*备份寄存器、RTC访问使能*/
    PWR_BackupAccessCmd(ENABLE);                            //使用PWR开启对备份寄存器和RTC的访问
    
    if (BKP_ReadBackupRegister(BKP_DR1) != 0xA5A5)          //通过写入备份寄存器的标志位，判断RTC是否是第一次配置
                                                            //if成立则执行第一次的RTC初始化
    {
        RCC_LSEConfig(RCC_LSE_ON);                          //开启LSE时钟
        while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) != SET);  //等待LSE准备就绪
        
        RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);             //选择RTCCLK来源为LSE
        RCC_RTCCLKCmd(ENABLE);                              //RTCCLK使能
                                        
        RTC_WaitForSynchro();                               //等待同步
        RTC_WaitForLastTask();                              //等待上一次操作完成
        
        RTC_SetPrescaler(32768 - 1);                        //设置RTC预分频器，预分频后的计数频率为1Hz
        RTC_WaitForLastTask();                              //等待上一次操作完成
        
        MyRTC_SetTime();                                    //设置时间，调用此函数，全局数组里时间值刷新到RTC硬件电路
        
        BKP_WriteBackupRegister(BKP_DR1, 0xA5A5);           //在备份寄存器写入自己规定的标志位，用于判断RTC是不是第一次执行配置
    }
    else                                                    //RTC不是第一次配置
    {
        RTC_WaitForSynchro();                               //等待同步
        RTC_WaitForLastTask();                              //等待上一次操作完成
    }
}
/**
  * 函    数：RTC设置时间
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，全局数组里时间值将刷新到RTC硬件电路
  */
void MyRTC_SetTime(void)
{
    time_t time_cnt = 0;    //定义秒计数器数据类型
    struct tm time_date;    //定义日期时间数据类型
    
    time_date.tm_year = MyRTC_Time[0] - 1900;       //将数组的时间赋值给日期时间结构体
    time_date.tm_mon = MyRTC_Time[1] - 1;
    time_date.tm_mday = MyRTC_Time[2];
    time_date.tm_hour = MyRTC_Time[3];
    time_date.tm_min = MyRTC_Time[4];
    time_date.tm_sec = MyRTC_Time[5];
    
    time_cnt = mktime(&time_date) - 8 * 60 * 60;    //调用mktime函数，将日期时间转换为秒计数器格式
                                                    //- 8 * 60 * 60为东八区的时区调整
    
    RTC_SetCounter(time_cnt);                       //将秒计数器写入到RTC的CNT中
    RTC_WaitForLastTask();                          //等待上一次操作完成
}

/**
  * 函    数：RTC读取时间
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，RTC硬件电路里时间值将刷新到全局数组
  */
void MyRTC_ReadTime(void)
{
    time_t time_cnt;        //定义秒计数器数据类型
    struct tm time_date;    //定义日期时间数据类型
    
    time_cnt = RTC_GetCounter() + 8 * 60 * 60;      //读取RTC的CNT，获取当前的秒计数器
                                                    //+ 8 * 60 * 60为东八区的时区调整
    
    time_date = *localtime(&time_cnt);              //使用localtime函数，将秒计数器转换为日期时间格式
    
    MyRTC_Time[0] = time_date.tm_year + 1900;       //将日期时间结构体赋值给数组的时间
    MyRTC_Time[1] = time_date.tm_mon + 1;
    MyRTC_Time[2] = time_date.tm_mday;
    MyRTC_Time[3] = time_date.tm_hour;
    MyRTC_Time[4] = time_date.tm_min;
    MyRTC_Time[5] = time_date.tm_sec;
}

			```
		### MyRTC.h {toggle="true"}
			```c
#ifndef __MYRTC_H
#define __MYRTC_H

extern uint16_t MyRTC_Time[];

void MyRTC_Init(void);

void MyRTC_SetTime(void);

void MyRTC_ReadTime(void);

#endif

			```
		### main.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "MyRTC.h"

int main(void)
{
    /*模块初始化*/
    OLED_Init();        //OLED初始化
    MyRTC_Init();       //RTC初始化
    
    /*显示静态字符串*/
    OLED_ShowString(1, 1, "Date:XXXX-XX-XX");
    OLED_ShowString(2, 1, "Time:XX:XX:XX");
    OLED_ShowString(3, 1, "CNT :");
    OLED_ShowString(4, 1, "DIV :");
    
    while (1)
    {
        MyRTC_ReadTime();                           //RTC读取时间，最新的时间存储到MyRTC_Time数组中
        
        OLED_ShowNum(1, 6, MyRTC_Time[0], 4);       //显示MyRTC_Time数组中的时间值，年
        OLED_ShowNum(1, 11, MyRTC_Time[1], 2);      //月
        OLED_ShowNum(1, 14, MyRTC_Time[2], 2);      //日
        OLED_ShowNum(2, 6, MyRTC_Time[3], 2);       //时
        OLED_ShowNum(2, 9, MyRTC_Time[4], 2);       //分
        OLED_ShowNum(2, 12, MyRTC_Time[5], 2);      //秒
        
        OLED_ShowNum(3, 6, RTC_GetCounter(), 10);   //显示32位的秒计数器
        OLED_ShowNum(4, 6, RTC_GetDivider(), 10);   //显示余数寄存器
    }
}

			```
# 27.  STM32 PWR电源控制 与 低功耗模式 详解 {toggle="true"}
	## 1. PWR 电源控制 简介 {toggle="true"}
		**PWR（Power Control）电源控制**
		- PWR负责管理STM32内部的电源供电部分，可以实现可编程电压监测器和低功耗模式的功能
		- 可编程电压监测器（PVD）可以监控VDD电源电压，当VDD下降到PVD阀值以下或上升到PVD阀值之上时，PVD会触发中断，用于执行紧急关闭任务
		- 低功耗模式包括**睡眠模式（Sleep）、停机模式（Stop）和待机模式（Standby）**，可在系统空闲时，降低STM32的功耗，延长设备使用时间
	## 2. PWR 电源控制 框图 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1cb1b6d4-bd45-4db9-a1fb-358cbc2580b1.webp)
		1. 最上面是模拟供电VDDA
			- 主要为AD转换器、温度传感器、复位模块、PLL锁相环供电
			- 其中AD转换器，还有两根参考电压的供电引脚：VREF+、VREF-（在C8T6中直接接到了VDDA和VSSA了。其他引脚比较多的情况下会直接引出去） 
		2. 中间为数字部分供电，分为两部分：VDD供电、1.8V供电
			- 左边 部分为 VDD供电，其中包括IO供电、待机电路、唤醒逻辑、独立看门狗
			- 右边 部分为 VDD通过电压调节器降压到1.8V，提供给1.8V供电区域。<br>1.8V区域包括CPU核心、存储器、内置的 数字外设。
		3. 下面为后备供电区域。VBAT
			- 其中包括LSE 32K晶体振荡器、后备寄存器BKP、RCC_BOCR寄存器（是RCC的寄存器，是备份域控制寄存器）、RTC
			- 低电压检测器是控制供电的。当VDD有电时，这块区域由VDD供电，没电才用VBAT供电。
	## 3. 上电复位和掉电复位 与 可编程电压检测器（PVD） {toggle="true"}
		### 3.1 内嵌复位与电源控制模块特性图 {toggle="true"}
			这个是内嵌的上电掉电复位和可编程电压检测器的框图。填写寄存器对应的值，会有对应的上下限电压阈值。（以典型值为准）
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/476d3004-c8fb-40b1-8f3c-069d67c2d082.webp)
		### 3.2 上电复位和掉电复位 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/f133f7a1-ba68-4457-b0ab-e9202f9276f6.webp)
			1. **当VDD或VDDA电压过低时，内部电路直接产生复位，让stm32复位住，不要乱操作**
			2. 图中的40mV的迟滞。是用来定义上下限的。用来复位和解除复位。
				这是一个典型的迟滞比较器。迟滞比较器设置两个阈值比设置单一的阈值要好。可以防止电压在某个阈值附近波动时，造成输出也来抖动
				- 当电压小于下线PDR时，复位。
				- 当电压大于上限POR时，解除复位
			3. 由上表可知，在STM32中 复位的下限为1.88V、上限为1.92V、每次复位持续时间为2.5ms）
			4. **可以看到：在低于下限电压时，复位信号一直为0，代表复位（低电平有效）。<br>而高于上限时，复位信号为1，代表不复位。**
		### 3.3 可编程电压检测器（PVD） {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/122716dc-2b04-436a-82b1-d666fa9b0cff.webp)
			电压检测器和上电复位和掉电复位电路。都是用来监测VDD和VDDA的供电电压的。
			**而PVD的区别是：**
			1. 阈值电压可以用程序指定
			2. 范围在数据手册中可以看到，可选范围（大概）是2.2V \~ 2.9V
			3. PVD的上限和下限的迟滞电压为100mv，是比上电掉电复位的电压要高的。
			**PVD的工作逻辑：**
			1. 电压超过设置的阈值时，代表正常。PVD输出为0，<br>而电压低于设置的阈值时，代表电压过低。此时 PVD输出为1。开始警告
			2. 产生的这个上升沿和下降沿时可以在外部中断申请中断。在中断中可以进行相关操作。
				![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3cfb44b4-a0a3-4671-841d-30043f7d675f.webp)
		### **3.4 总结来说可以看这个图来理解。** {toggle="true"}
			PVD是来检测2.9V\~2.2V左右的电压，可以在这个范围内设置一个警告线，如果低于则触发警告。可以进入中断做一些事情。
			上电掉电复位则是提供一个最低的阈值，如果低于这个阈值则复位。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ac0b9028-87b1-43b3-8043-d907f124d558.webp)
	## 4. 低功耗模式介绍 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/1cf4e55c-1c5e-4f70-ab6f-efe1f67351e1.webp)
		> <span color="purple">进入停机或者待机模式之后，需要按住复位再下载程序，否则下载失败<br>（我这里不行，我会断电重上电，在未进入待机模式前开始烧录代码）</span>
		> <span color="purple">在理解睡眠、停机、待机模式时，参考PWR电源控制框图 效果更佳</span>
		- **低功耗模式有 睡眠、 停机、 待机、 **
			1. 从上到下，关闭的电路越来越多。
			2. 从上到下，越来越省电
			3. 从上到下，越来越难唤醒
		### 睡眠模式简介 {toggle="true"}
			- **睡眠模式有两种进入方式：**
				1. 直接调用WFI进入：WFI（Wait For Interrupt）等待中断，**有中断才能被唤醒**
					这个一般是为了进入中断处理中断函数。
				2. 直接调用WFE进入：WFE（Wait For Event）等待事件，**有事件才能被唤醒**
					（这个事件可以是外部中断配置为事件模式，也可以是使能了中断，但没有配置NVIC）
					这个一般是不需要进入中断的，直接从睡的地方继续运行。
			- **睡眠模式对电路的影响：**
				1. 对 1.8V区域的时钟 影响是 CPU时钟关、对其他时钟和ADC时钟无影响、<br>对 VDD区域的时钟 无影响<br>对 电压调节器 操作为开<br>（电压调节器是VDD通过调节后的1.8V，给后备区域用的）
				> **总结就是，睡眠模式只把CPU时钟关闭，对其他电路没有任何操作<br>CPU时钟关闭之后，程序就会暂停。但是寄存器的数据都还在<br>注意：**睡眠**进入不了的可能原因：**系统定时器SysTick一直产生中断
		### 停机模式简介 {toggle="true"}
			- **停机模式要操控的标志位：**
				1. PDDS位  ：设置停机还是待机模式**。0为停机模式**、设置为1为待机模式
				2. LPDS位   ： 设置电压调节器的开关，0为开启。1为进入低功耗<br>电压调节器是否关闭，1.8V区域的寄存器仍然能保持寄存器的数据。不过开启后更省电，但唤醒更慢罢了。
				3. SLEEPDEEP位  ： 设置是否进入深度睡眠模式。** 1 为进入。**
				4. 最后调用WFI或者WFE，芯片就可以进入停机模式了。
			- **对电路有何影响：**
				1. 关闭1.8V区域的时钟。<br>这就代表CPU时钟和内置数字外设的时钟都会停止。比如定时器、串口…<br>不过没关闭电源。寄存器的数据都还在。（原因见PWR电源框图）
				2. HSI和HSE的振荡器关闭，<br>因为外设都关了。所以要高速时钟也没用了。所以会关闭HSI内部高速时钟、HSE外部高速时钟。两个低速时钟不会主动关闭，因为这两个时钟要维持RTC和独立看门狗的运行。
			- **如何唤醒**
				1. 在停机模式下。任一**外部中断**（在外设中断寄存器中设置）就可以唤醒
		### 待机模式简介 {toggle="true"}
			- **停机模式要操控的标志位：**
				1. PDDS位  ：设置停机还是待机模式。0为停机模式、**设置为1为待机模式**
				2. SLEEPDEEP位  ： 设置是否进入深度睡眠模式**。 1 为进入。**
				3. 最后调用WFI或者WFE，芯片就可以进入待机模式了。
			- **对电路有何影响：**
				1. 关闭1.8V区域的时钟。<br>这就代表CPU时钟和内置数字外设的时钟都会停止。比如定时器、串口…<br>不过没关闭电源。寄存器的数据都还在。（原因见PWR电源框图）
				2. HSI和HSE的振荡器关闭，<br>因为外设都关了。所以要高速时钟也没用了。所以会关闭HSI内部高速时钟、HSE外部高速时钟。两个低速时钟不会主动关闭，因为这两个时钟要维持RTC和独立看门狗的运行。
				3. **强制关闭电压调节器，1.8V电源直接关断，意味着内部的寄存器数据全部丢失**
			- **如何唤醒**
				1. WKUP的引脚产生上升沿、
				2. RTC闹钟事件
				3. NRST引脚上的外部复位
				4. IWDG复位
	## 5. 低功耗模式选择框图 {toggle="true"}
		执行WFI（Wait For Interrupt）或者 WFE（Wait For Event）指令后，STM32进入低功耗模式。（这两条指令为最终开启低功耗模式的触发条件）
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/5e9e2f82-67e5-4fd3-ba0e-bfe45850dede.webp)
		可以看到。
		一旦WFI 或者 WFE 执行了。 芯片是通过判断这些标志位来进入低功耗的三种模式的。
		- SLEEPDEEP  位  决定是否进入深度睡眠。0为浅睡眠模式（睡眠模式）、1为深度睡眠模式（停机或待机）
			- **在睡眠模式时**
				SLEEPONEXIT位的0、1标志可以决定立刻睡眠，还是等待中断事件处理结束后才睡眠。一般不用，只有在中断中调用睡眠模式，才会考虑这个标志位。
			- **在深度睡眠模式时**
				- 判断PDDS位，为0则进入停机。为1则进入待机模式
					- 停机模式下，会继续判断LPDS位，为0则开启电压调节器。为1则电压调节器开启低功耗（更省电，但唤醒时间更长）
		<span color="pink">*使用睡眠模式时，一般是直接使用使用__WFI来配置为睡眠模式，对于刚才所说的SLEEPDEEP = 0和 SLEEPONEXIT 的位 使用默认的就可以了。所以是不需要配置的。直接输入__WFI<br>而停止模式和待机模式，都有对应的函数可以直接一键配置*</span>
		要是真想去配置的话，输入`SCB->SCR = 0x….;` 就可以配置这些位了
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a57af555-fa5f-4f79-bf64-ff817097714b.webp)
	## 6. 低功耗三种模式总结 {toggle="true"}
		### 6.1 睡眠模式 {toggle="true"}
			- 执行完WFI/WFE指令后，STM32进入睡眠模式，程序暂停运行，唤醒后程序从暂停的地方继续运行
			- SLEEPONEXIT位决定STM32执行完WFI或WFE后，是立刻进入睡眠，还是等STM32从最低优先级的中断处理程序中退出时进入睡眠
			- 在睡眠模式下，所有的I/O引脚都保持它们在运行模式时的状态
			- WFI指令进入睡眠模式，可被任意一个NVIC响应的中断唤醒
			- WFE指令进入睡眠模式，可被唤醒事件唤醒<br>
		### 6.2 停机模式 {toggle="true"}
			- 执行完WFI/WFE指令后，STM32进入停止模式，程序暂停运行，唤醒后程序从暂停的地方继续运行
			- 1.8V供电区域的所有时钟都被停止，PLL、HSI和HSE被禁止，SRAM和寄存器内容被保留下来
			- 在停止模式下，所有的I/O引脚都保持它们在运行模式时的状态
			- 当一个中断或唤醒事件导致退出停止模式时，**HSI被选为系统时钟<br>所以在停止模式唤醒之后，第一时间就是重启HSE，配置主频为72MHZ**
			- 当电压调节器处于低功耗模式下，系统从停止模式退出时，会有一段额外的启动延时
			- WFI指令进入停止模式，可被任意一个EXTI中断唤醒
			- WFE指令进入停止模式，可被任意一个EXTI事件唤醒<br>
		### 6.3 待机模式 {toggle="true"}
			- 执行完WFI/WFE指令后，STM32进入待机模式，唤醒后程序**从头开始运行**
			- **整个1.8V供电区域被断电**，PLL、HSI和HSE也被断电，**SRAM和寄存器内容丢失，**只有备份的寄存器和待机电路维持供电
			- 在待机模式下，**所有的I/O引脚变为高阻态**（浮空输入）
			- WKUP引脚的上升沿、RTC闹钟事件的上升沿、NRST引脚上外部复位、IWDG复位退出待机模式
			- 在进入待机模式之前，一般都需要把所有的外设 都关闭，比如屏幕、电机等等。这样才能达到 最最最省电的效果<br>
	## 7. STM32各个状态下的电量消耗 {toggle="true"}
		### 7.1 睡眠模式 {toggle="true"}
			睡眠模式下，各个频率的电流  是mA级别的电流。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/e2975a55-fbed-43a3-a158-8e73d9e5a346.webp)
		### 7.2 停机模式和待机模式 {toggle="true"}
			电流变为了μA级别。这是非常省电的。
			对于一个300mAh的电池：睡眠（10h），停机（1w h），待机（10w h），<br>的使用时间能相差很大！
			备份区域的供电 消耗电量也很小，仅仅1μA
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/81dae6aa-4ffd-4be3-97a8-bbe1a363ff0d.webp)
	## 8. 修改系统主频的方法 {toggle="true"}
		修改主频可以有效的降低功耗。并且在停机模式下唤醒，也需要修改主频到72MHZ
		这个需要去`system_stm32f10x.c`和`system_stm32f10x.h` 中查看学习
		### 7.1 `system_stm32f10x.c`中可调用的函数 {toggle="true"}
			- 在`system_stm32f10x.c` 文件中有两个外部可调用的函数和一个外部可调用的变量.（可以在头文件中看到。）
				两个函数是`SystemInit()`和`SystemCoreClockUpdate()`
				一个变量是`SystemCoreClock` 
			- `SystemInit()` 是用来配置时钟树的，他是在执行main函数之前，在启动文件中调用的。
			- `SystemCoreClock` 表示主频频率的值。如果我们要显示当前频率就可以直接调用显示
			- `SystemCoreClockUpdate()` 更新上边的`SystemCoreClock` 的值。
		### 7.2 `system_stm32f10x.c`中的宏定义 {toggle="true"}
			解除对应的注释，来选择想要的系统主频（带有钥匙的文件无法修改，需要取消文件的只读属性）
			这个if和 lese的意思是：只要你不是VL（超值系列）。就可以配置那么多的主频（lese下边的）
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/9d11e9fc-861b-4d23-adf3-e1a5786a41eb.webp)
		### \[扩展\]STM32配置系统时钟时都做了什么 {toggle="true"}
			系统在进入main函数之前，会由system_stm32f10x.c函数来配置系统时钟。
			首先system会启动HSI内部高速时钟
			然后会进行各种恢复缺省配置（比如）
			最后调用SetSysClock函数（这是一个分配函数，根据我们之前解除的宏定义，来选择执行不同的配置函数。比如SetSysClockTo72、To56、To48…..）
			在执行的配置函数中。才会真正的对stm32的寄存器开始配置。
			比如To72的配置为：选择HSE外部告诉始终作为锁相环输入，锁相环进行9倍频<br>再选择锁相环输出作为主频。
	## 9. 编写：睡眠模式+串口收发数据 {toggle="true"}
		### **9.1 为什么一定是睡眠模式呢？ ** {toggle="true"}
			- 因为在停机模式和待机模式下。会关闭1.8V区域的时钟。这就导致了USART外设不能接收数据而产生中断。<br>并且停机模式和待机模式只能是由外部中断唤醒的。
			所以只能使用睡眠模式，因为睡眠模式仅仅是关闭了CPU的时钟。使程序不再往下运行。1.8V供电区域和电压调节器都是开着的。可以使用USART的中断进行唤醒。
		### 9.2 代码框架介绍 {toggle="true"}
			- main.c  仅仅添加了一行代码：__WFI 中断睡眠。 其余的不用配置，使用默认为0的就行（睡眠模式0 + 立刻睡眠0） 另外还加入了Running来验证程序是否进入睡眠模式
			- 执行流程：__WFI()后，睡眠，串口发送数据，产生中断，唤醒cpu，执行中断，执行主函数，遇到__WFI() ，睡眠，
			- Serial.c 串口收发数据
			- Serial.h 串口收发数据头文件
			- 注意事项：如果Delay.c中的延时实现是靠SysTick滴答定时器中断实现的。这个中断也会触发唤醒操作。需要关闭SysTick的中断
		### main.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "led.h"
#include "delay.h"
#include "Key.h"
#include "Serial.h"
#include "OLED.h"

uint8_t RxData;

int main()
{
    OLED_Init();//初始化OLED
    Delay_Init();//初始化延时函数
    
    Serial_Init();//初始化串口
    
    OLED_ShowString(1,1,"RxData:");
    while(1)
    {
        if(Serial_GetRxfalg() == 1)
        {
            RxData = Serial_GetRxData();
            Serial_SendByte(RxData);
            OLED_ShowHexNum(1,8,RxData,2);
        }
        OLED_ShowString(2,1,"Running");
        Delay_ms(100);
        OLED_ShowString(2,1,"       ");
        Delay_ms(100);
        
        __WFI();
    }
}

			```
		### Serial.c {toggle="true"}
			```c
#include "Serial.h"

//初始化
void Serial_Init(void)
{
    //使能A9 A10所在的GPIO
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    
    //使能A9 A10的复用外设时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1,ENABLE);
    
    //配置PA9端口为复用推挽输出
    GPIO_InitTypeDef GPIO_InitStructure;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;//复用推挽模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;       //9
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);//初始化
    //这里对于F4芯片可以单独 分开的设置复用以及上下拉。不用分开配置，
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;//复用推挽模式
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;       //9
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOA,&GPIO_InitStructure);//初始化
    
    USART_InitTypeDef USART_InitStructure;
    USART_InitStructure.USART_BaudRate = 9600;//波特率。会自动算好填入BRR寄存器
    USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;//硬件流控制、不使用.
    USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;//用| 使用两个功能
    USART_InitStructure.USART_Parity = USART_Parity_No; //校验位。
    USART_InitStructure.USART_StopBits = USART_StopBits_1;//停止位1位。
    USART_InitStructure.USART_WordLength = USART_WordLength_8b; //不需要校验，所以字长选择8位字长。
    USART_Init(USART1,&USART_InitStructure);
    
    USART_ITConfig(USART1,USART_IT_RXNE,ENABLE);
    
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
    NVIC_InitTypeDef NCIC_InitStructure;
    NCIC_InitStructure.NVIC_IRQChannel = USART1_IRQn;
    NCIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
    NCIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
    NCIC_InitStructure.NVIC_IRQChannelSubPriority = 1;//优先级
    NVIC_Init(&NCIC_InitStructure);
    
    
    USART_Cmd(USART1,ENABLE);
    
    
}

//发送字节
void Serial_SendByte(uint16_t Byte)
{
    USART_SendData(USART1,Byte);
    while(USART_GetFlagStatus(USART1,USART_FLAG_TXE) == RESET);//如果写标志位没有置1（没有发送完），那么就等
}

//发送数组
void Serial_SendArray(uint8_t* Array,uint16_t len)
{
    uint16_t i = 0;
    for(i = 0; i < len; i++)
    {
        Serial_SendByte(Array[i]);
    }
}


//发送字符串
void Serial_SendString(char* String)//有结束的\0 所以不用再传长度了，
{
    uint16_t i = 0;
    for(i = 0; (String[i] != '\0'); i++)
    {
        Serial_SendByte(String[i]);
    }
}


//求次方函数
uint32_t Serial_Pow(uint32_t X,uint8_t Y)
{
    int Num = 1;
    while(Y--)
    {
        Num *= X;
    }
    return Num;
}


//发送字符形式的数字
void Serial_SendNumber(uint32_t Number,uint8_t len)
{
    //从高位像低位取数字然后输出
    uint8_t i = 0;
    for(i = 0; i < len; i++)
    {
        Serial_SendByte(Number / Serial_Pow(10,len - i - 1) %10 + '0');
    
    }
}

//下边没学过，直接搬过来的。
/**
  * 函    数：使用printf需要重定向的底层函数
  * 参    数：保持原始格式即可，无需变动
  * 返 回 值：保持原始格式即可，无需变动
  */
int fputc(int ch, FILE *f)
{
	Serial_SendByte(ch);			//将printf的底层重定向到自己的发送字节函数
	return ch;
}

/**
  * 函    数：自己封装的prinf函数
  * 参    数：format 格式化字符串
  * 参    数：... 可变的参数列表
  * 返 回 值：无
  */
void Serial_Printf(char *format, ...)
{
	char String[100];				//定义字符数组
	va_list arg;					//定义可变参数列表数据类型的变量arg
	va_start(arg, format);			//从format开始，接收参数列表到arg变量
	vsprintf(String, format, arg);	//使用vsprintf打印格式化字符串和参数列表到字符数组中
	va_end(arg);					//结束变量arg
	Serial_SendString(String);		//串口发送字符数组（字符串）
}




uint8_t Serial_RxData;
uint8_t Serial_RxFlag;

uint8_t Serial_GetRxfalg(void)
{
    if(Serial_RxFlag == 1)
    {
        Serial_RxFlag = 0;
        return 1;
    }
    return 0;
}

uint8_t Serial_GetRxData(void)
{
    return Serial_RxData;
}


void USART1_IRQHandler(void)//中断
{
    if(USART_GetFlagStatus(USART1,USART_FLAG_RXNE) == SET)
    {
        Serial_RxData = USART_ReceiveData(USART1);//读取
        Serial_RxFlag = 1;
        USART_ClearITPendingBit(USART1,USART_FLAG_RXNE);//手动清除标志位
    }

}


			```
		### Serial.h {toggle="true"}
			```c
#ifndef __SERIAL_H 

#include "stm32f10x.h"
#include "stdio.h"
#include "stdarg.h"

#define __SERIAL_H 


//初始化串口
void Serial_Init(void);

//发送字节
void Serial_SendByte(uint16_t Byte);

//发送数组
void Serial_SendArray(uint8_t* Array,uint16_t len);

//发送字符串
void Serial_SendString(char* String);

//发送字符形式的数字
void Serial_SendNumber(uint32_t Number,uint8_t len);

//移植printf
void Serial_Printf(char *format, ...);

//获取输入的RXNE标志位
uint8_t Serial_GetRxfalg(void);

//获取输入的字符
uint8_t Serial_GetRxData(void);

#endif

			```
	## 10. PWR.h 电源控制函数介绍 {toggle="true"}
		`void PWR_DeInit(void);` 
			- 恢复缺省配置
		`void PWR_BackupAccessCmd(FunctionalState NewState);`
			- 使能后备区域的访问
		`void PWR_PVDCmd(FunctionalState NewState);`
			- 配置PVD阈值电压
		`void PWR_PVDLevelConfig(uint32_t PWR_PVDLevel);`
			- 使能PVD功能
		`void PWR_WakeUpPinCmd(FunctionalState NewState);`
			- 使能位于PA0位置的WKUP引脚。（配合待机模式使用）
		> `void PWR_EnterSTOPMode(uint32_t PWR_Regulator, uint8_t PWR_STOPEntry);`
			- 进入停止模式
			- 指定电压调节器在停止模式中的状态、使用WFI或WFI指令进入
		> `void PWR_EnterSTANDBYMode(void);`
			- 进入待机模式
			- 使用WFI或WFI指令进入
		`FlagStatus PWR_GetFlagStatus(uint32_t PWR_FLAG);`
			- 获取标志位
		`void PWR_ClearFlag(uint32_t PWR_FLAG);`
			- 清除标志位
	## 11. 编写：停止模式+外部中断计次 {toggle="true"}
		### **11.1 为什么是**停止模式**呢？ ** {toggle="true"}
			- 刚才我们已经验证过睡眠模式了。而停止模式只能由外部中断来触发。所以只能选择外部中断类型的代码来验证
			- 在停止模式下， 1.8V的区域的时钟关闭，CPU和外设都没有时钟了。
			- 但是外部中断的工作是不需要时钟过的
		### 11.2 注意事项 {toggle="true"}
			- 我们需要使用PWR外设，来操作进入停止模式、待机模式。它在APB1总线上
			- **在停止模式被唤醒之后，会默认使用内部高速时钟HSI。**
				需要手动调整时钟为外部高速HSE。如果不设置，在退出停止模式之后程序运行速度会变慢。重新启动主频函数为：`SystemInit();`
		### 11.3 代码框架介绍 {toggle="true"}
			- main.c  测试停止模式工作，使用running闪烁来判断是否停止
			- 执行流程：复位后初始化、进入主循环、进入停止模式、外部中断发生时，唤醒停止模式（此时主频被设置为HSI内部高速时钟）、进入中断执行中断内容、继续从停止位置继续执行。设置主频为72MHZ、继续执行后续函数、然后进入停止
			- CountSensor.c 外部中断计次
			- CountSensor.h 外部中断计次头文件
		### main.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "oled.h"
#include "Delay.h"
#include "CountSensor.h"

int main()
{
    OLED_Init();//初始化OLED;
    CountSensor_Init();//初始化
    
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR,ENABLE);//开启PWR外设
    
    OLED_ShowString(1,1,"Count:");
    while(1)
    {
        
        OLED_ShowSignedNum(1, 7,CountSenSor_Get(),5);
        
        
        OLED_ShowString(2,1,"Running");
        Delay_ms(100);
        OLED_ShowString(2,1,"       ");
        Delay_ms(100);
        
        PWR_EnterSTOPMode(PWR_Regulator_ON,PWR_STOPEntry_WFI);//开启电压调节器， WFI模式进入
        SystemInit();//重新启动主频
    }
}


			```
		### CountSensor.c  {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header

//引脚为A2  

//计次变量
uint16_t CountSensor_Count = 0;

//初始化
void CountSensor_Init(void)
{
    //开启A2的时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
    //开启挂载在APB2外设的AFIO外设时钟
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO,ENABLE);
    //EXTI时钟和NVIC的时钟不用手动打开。
    
    //配置GPIO
    GPIO_InitTypeDef GPIO_InitStructrue;
    GPIO_InitStructrue.GPIO_Mode = GPIO_Mode_IPU;        //上拉输入模式
    GPIO_InitStructrue.GPIO_Pin = GPIO_Pin_2;
    GPIO_InitStructrue.GPIO_Speed = GPIO_Speed_50MHz;    
    GPIO_Init(GPIOA,&GPIO_InitStructrue);
     
    //配置AFIO （F1中在GPIO.h文件中）（目的是为了把GPIOA的PIN2引脚映射到AFIO中。）
    GPIO_EXTILineConfig(GPIO_PortSourceGPIOA,GPIO_PinSource2);//这里需要根据PIn和GPIOx来选择

    
    //配置EXTI
    EXTI_InitTypeDef EXTI_InitStructure;
    EXTI_InitStructure.EXTI_Line = EXTI_Line2;          //选择中断线路，这里是PIN2 所以为2   
    EXTI_InitStructure.EXTI_LineCmd = ENABLE;           //是否使能指定的中断线路
    EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt; //中断或响应模式   
    EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;//上升或下降或边沿触发
    EXTI_Init(&EXTI_InitStructure);
    
    //配置NVIC（因为NVIC属于内核，所以被分配到内核的杂项中去了，在misc.c）
    NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//配置抢占和响应优先级
     
    NVIC_InitTypeDef NVIC_InitStructure;
    NVIC_InitStructure.NVIC_IRQChannel = EXTI2_IRQn;       //在stm32f10x.h文件里。让你找IRQn_Type里的一个中断通道。这里使用的是md的芯片（如果引脚是15-10或者9-5则需要去找对应的那个）这里我是PIN2.所以找2
    NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;         //是否使能指定的中断通道
    NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;//抢占优先级（这里可以看表。看范围，）
    NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;       //指定响应优先级
    NVIC_Init(&NVIC_InitStructure);    
     
}

//获取当前的计数
uint16_t CountSenSor_Get(void)
{
    return CountSensor_Count;
}


//在STM32中，中断的函数都是固定的。他们在启动文件中存放xxx.s 
//以IRQHandler结尾的就是中断函数的名字。
//在这里需要找到对应的中断函数，我这里是2
void EXTI2_IRQHandler(void)//中断函数是无参无返回值的。中断函数必须写对，写错就进不去
{
    //在进入中断后，一般要判断一下这个是不是我们想要的那个中断源触发的中断。
    //但是在这里。我是GPIOA的PIN2引脚，所以不用写。
    //如果是5-9 10-15的引脚。他们EXTI到NVIC是几个共用的。
    //所以需要根据EXTI输入时的16根引脚。来判断是16根引脚的那一根发送的中断请求。
    //这里规范写的话需要加上去
    //查找标志位函数在exit.h中。
    if(EXTI_GetITStatus(EXTI_Line2) == SET)//第一个参数是行数.判断这个线的标志位是不是== SET。是则是我们想要的
    {
    
        CountSensor_Count++;
        EXTI_ClearITPendingBit(EXTI_Line2);//中断结束后，要调用清除标志位的函数。如果你不清除，程序会一直进入中断
    }
}

			```
		### CountSensor.h {toggle="true"}
			```c
#ifndef __COUNTSENSOR_H
#define __COUNTSENSOR_H


//初始化旋转编码计次
void CountSensor_Init(void);

//获取当前计数值
uint16_t CountSenSor_Get(void);

#endif

			```
		<empty-block/>
	## 12. 编写：待机模式+实时时钟 {toggle="true"}
		### **12.1 为什么是待机模式呢？ ** {toggle="true"}
			- 刚才我们已经验证过睡眠模式、停止模式了。<br>待机模式只能由WKUP引脚的上升沿、RTC闹钟事件的上升沿、NRST引脚上外部复位、IWDG复位来退出待机模式。
			- 在待机模式恢复后，会重头开始执行命令。所以不需要配置时钟了。
			- 在待机模式下， 1.8V的区域的时钟关闭，电压调节器也关闭。这会使除了备份区域外的所有寄存器清除数据，并且IO口对外呈现为高阻态。
		### 12.2 注意事项 {toggle="true"}
			- **在进入待机模式之前，一般都需要把所有的外设 都关闭，比如屏幕、电机等等。<br>否则就无法极度省电**
			- 待机模式会重头开始执行命令，所以不需要再配置时钟了。
			- 待机模式烧录代码可以按住复位再烧录<br>（我这里不行，我会断电重上电，在未进入待机模式前开始烧录代码）
			- **测试WKUP引脚时**，只需要函数：`PWR_WakeUpPinCmd(ENABLE);` 就可以了。程序会自动帮我们把WKUP（PA0）引脚设置为下拉输入的配置，不需要GPIO初始化。
		### 12.3 代码框架介绍 {toggle="true"}
			- main.c  测试停止模式工作，使用running闪烁来判断是否进入待机模式
			- 执行流程：复位后初始化、进入主循环、进入待机模式（所有IO口浮空、寄存器重置、CPU核心停止。但是后备区域不会受到影响）、唤醒待机模式、程序重头开始运行。然后再次进入待机模式
			- MyRTC.c RTC实时时钟
			- MyRTC.h RTC实时时钟头文件
		### main.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "MyRTC.h"

int main(void)
{
    /*模块初始化*/
    OLED_Init();        //OLED初始化
    MyRTC_Init();       //RTC初始化
    
    /*显示静态字符串*/
    OLED_ShowString(1, 1, "CNT:");  //时间戳计数器
    OLED_ShowString(2, 1, "ALR:");  //闹钟值
    OLED_ShowString(3, 1, "ALRF:"); //闹钟标志位
    
    PWR_WakeUpPinCmd(ENABLE);
    
    //存下要显示的闹钟值（闹钟寄存器只可写不可读，所以要存下，方便显示）
    uint32_t Alarm = RTC_GetCounter() + 10;
    //设定闹钟为十秒后
    RTC_SetAlarm(Alarm);
    //显示闹钟
    OLED_ShowNum(2, 5, Alarm, 10);
    
    //开启PWR时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);
    
    while (1)
    {
        OLED_ShowNum(1, 5, RTC_GetCounter(), 10);                   //显示32位的秒计数器
        OLED_ShowNum(3, 6, RTC_GetFlagStatus(RTC_FLAG_ALR), 1);     //显示32位的秒计数器
        
        OLED_ShowString(4,1,"Running");
        Delay_ms(100);
        OLED_ShowString(4,1,"       ");
        Delay_ms(100);
        
        //在进入待机模式之前，一般都需要把所有的外设 都关闭，比如屏幕、电机等等。
        //这里清个屏表示一下
        OLED_Clear();

        
        PWR_EnterSTANDBYMode();//进入待机模式
    }
}

			```
		### MyRTC.c  {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include <time.h>

uint16_t MyRTC_Time[] = {2024, 8, 18, 23, 46, 0};   //定义全局的时间数组，数组内容分别为年、月、日、时、分、秒

void MyRTC_SetTime(void);           //函数声明

/**
  * 函    数：RTC初始化
  * 参    数：无
  * 返 回 值：无
  */
void MyRTC_Init(void)
{
    /*开启时钟*/
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);     //开启PWR的时钟
    RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP, ENABLE);     //开启BKP的时钟
    
    /*备份寄存器、RTC访问使能*/
    PWR_BackupAccessCmd(ENABLE);                            //使用PWR开启对备份寄存器和RTC的访问
    
    if (BKP_ReadBackupRegister(BKP_DR1) != 0xA5A5)          //通过写入备份寄存器的标志位，判断RTC是否是第一次配置
                                                            //if成立则执行第一次的RTC初始化
    {
        RCC_LSEConfig(RCC_LSE_ON);                          //开启LSE时钟
        while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) != SET);  //等待LSE准备就绪
        
        RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);             //选择RTCCLK来源为LSE
        RCC_RTCCLKCmd(ENABLE);                              //RTCCLK使能
                                        
        RTC_WaitForSynchro();                               //等待同步
        RTC_WaitForLastTask();                              //等待上一次操作完成
        
        RTC_SetPrescaler(32768 - 1);                        //设置RTC预分频器，预分频后的计数频率为1Hz
        RTC_WaitForLastTask();                              //等待上一次操作完成
        
        MyRTC_SetTime();                                    //设置时间，调用此函数，全局数组里时间值刷新到RTC硬件电路
        
        BKP_WriteBackupRegister(BKP_DR1, 0xA5A5);           //在备份寄存器写入自己规定的标志位，用于判断RTC是不是第一次执行配置
    }
    else                                                    //RTC不是第一次配置
    {
        RTC_WaitForSynchro();                               //等待同步
        RTC_WaitForLastTask();                              //等待上一次操作完成
    }
}
/**
  * 函    数：RTC设置时间
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，全局数组里时间值将刷新到RTC硬件电路
  */
void MyRTC_SetTime(void)
{
    time_t time_cnt = 0;    //定义秒计数器数据类型
    struct tm time_date;    //定义日期时间数据类型
    
    time_date.tm_year = MyRTC_Time[0] - 1900;       //将数组的时间赋值给日期时间结构体
    time_date.tm_mon = MyRTC_Time[1] - 1;
    time_date.tm_mday = MyRTC_Time[2];
    time_date.tm_hour = MyRTC_Time[3];
    time_date.tm_min = MyRTC_Time[4];
    time_date.tm_sec = MyRTC_Time[5];
    
    time_cnt = mktime(&time_date) - 8 * 60 * 60;    //调用mktime函数，将日期时间转换为秒计数器格式
                                                    //- 8 * 60 * 60为东八区的时区调整
    
    RTC_SetCounter(time_cnt);                       //将秒计数器写入到RTC的CNT中
    RTC_WaitForLastTask();                          //等待上一次操作完成
}

/**
  * 函    数：RTC读取时间
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，RTC硬件电路里时间值将刷新到全局数组
  */
void MyRTC_ReadTime(void)
{
    time_t time_cnt;        //定义秒计数器数据类型
    struct tm time_date;    //定义日期时间数据类型
    
    time_cnt = RTC_GetCounter() + 8 * 60 * 60;      //读取RTC的CNT，获取当前的秒计数器
                                                    //+ 8 * 60 * 60为东八区的时区调整
    
    time_date = *localtime(&time_cnt);              //使用localtime函数，将秒计数器转换为日期时间格式
    
    MyRTC_Time[0] = time_date.tm_year + 1900;       //将日期时间结构体赋值给数组的时间
    MyRTC_Time[1] = time_date.tm_mon + 1;
    MyRTC_Time[2] = time_date.tm_mday;
    MyRTC_Time[3] = time_date.tm_hour;
    MyRTC_Time[4] = time_date.tm_min;
    MyRTC_Time[5] = time_date.tm_sec;
}

			```
		### MyRTC.h {toggle="true"}
			```c
#ifndef __MYRTC_H
#define __MYRTC_H

extern uint16_t MyRTC_Time[];

void MyRTC_Init(void);

void MyRTC_SetTime(void);

void MyRTC_ReadTime(void);

#endif

			```
# 28.   STM32 内部FLASH详解 {toggle="true"}
	## 1. STM32 FLASH简介 {toggle="true"}
		- STM32F1系列的FLASH包含程序存储器、系统存储器和选项字节三个部分，**通过闪存存储器接口（外设）可以对程序存储器和选项字节进行擦除和编程**<br>（系统存储器是用来存放原厂写入的用于串口下载的BootLoader的。不允许我们进行修改）
		- 读写FLASH的用途：
			1. 利用程序存储器的剩余空间来保存掉电不丢失的用户数据
			2. 通过在程序中编程（IAP），实现程序的自我更新<br>（直接修改程序本身，不避开程序，与OTA类似）
		- 在线编程（In-Circuit Programming – ICP）用于更新程序存储器的全部内容，它通过JTAG、SWD协议或系统加载程序（Bootloader）下载程序<br>（也就是我们平时用的下载程序的方式 ）
		- 在程序中编程（In-Application Programming – IAP）可以使用微控制器支持的任一种通信接口下载程序<br>（也就是自己写一个BootLoader程序，放到程序不会覆盖到的地方。在需要升级时，让程序跳转到自己写的BootLoader中。根据自己的 协议，比如蓝牙，串口，WIFI等  控制FLASH读写。覆盖原有的程序。这其实也是系统的BootLoader一样）
	## 2. STM32 FLASH与SRAM {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/ce35fbfc-e60a-435b-8fc8-84d5ddcb2f4e.webp)
		1. ROM的存储介质是FLASH、RAM的存储介质是SRAM
		2. FLASH  闪存掉电不丢失、SRAM掉电丢失
		3. 闪存主要有程序存储器、系统存储器、选项字节三个部分。
			- 其中程序存储器是**空间最大**，**最主要的部分**。所以也乘坐主存储器。起始地址为0x0800 0000  ，是用来存储编译后的代码<br>所以我们平时所说的FLASH容量，往往指的是FLASH中程序存储器的容量
			- 系统存储器起始地址为0x 1FFF F000 ，用于存储BootLoader ，用于串口下载
			- 选项字节其实地址为0x1FFF F800 ，用于存储一些独立的配置参数
		4. SRAM区域分为运行内存SRAM、外设寄存器、内核外设寄存器
			- SRAM运行内存的起始地址是0x2000 0000 ， 用于存储运行过程的临时变量
			- 外设寄存器的起始地址是0x4000 0000 ， 用于存储各个外设的配置参数
			- 内核外设寄存器 的起始地址是0xE000 0000 ， 用于存储内核各个外设的配置参数
	## 3. STM32 FLASH 容量、内容介绍 {toggle="true"}
		1. **FLASH在 STM32中根据不同的型号，容量也不同。**
			- 以stm32F10x系列为例
			- 小容量产品：32页 ，每页1K
			- 中容量产品：128页，每页1K
			- 大容量产品：256页，每页2K
		2. 以中容量为例讲解：
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/a6653025-a1ec-4074-8cfc-667ca2ae68c7.webp)
			**这个图中，把FLASH分为了三个块：**
			- 主存储器（程序存储器）：用来存放编译后的程序
			- 信息快： 又可以分为两个
				1. 启动程序代码 （系统存储器）： 存放原厂写入的BootLoader，用于串口下载
				2. 用户选择字节（选项字节） ：用于存放一些独立的参数
			- <span color="blue">闪存存储器接口寄存器：</span>这个的地址是40开头的 ，根据SRAM的地址分配可以看到。闪存存储器接口寄存器是一个外设，与GPIO、定时器、串口等是一个性质的东西。<br>闪存存储器接口可以理解为上述FLASH闪存的管理员。是用来控制闪存的擦除和编程的
			**对于F103 C8T6主存储器有0-63页，共64页，也就是64K**
				- 地址的规律：只要是000、400、800、C00 结尾的。就是页的起始地址。
				1. 启动程序代码（系统存储器）占用了1K空间，地址为0x1FFF F000
				2. 用户选择字节（选项字节）：只有16个字节的配置参数
			闪存存储器接口寄存器，每个寄存器占4个字节。
	## 4. STM32 FLASH 读写注意事项 {toggle="true"}
		1. 通过闪存存储器接口（外设）我们可以对程序存储器和选项字节进行擦除和编程。<br>**但系统存储器是不可以修改的。**它是用来存放原厂写入的用于串口下载的BootLoader的。
		2. 在选取FLASH存储区域时，一定不要覆盖原有的程序。不然就运行不了了。
		3. 所有的FLASH闪存 的写入和擦除规定了：
			- 写入前必须擦除
			- 擦除必须以最小单位进行（这里为页 1K）
			- 擦除后数据位全变为1
			- 数据只能1写0 ，不能0写1
			- 擦除和写入需要等待忙
		4. 在写入数据时，如果指定地址没有被擦除，那么就不会执行编程。同时提出警告（例外是写入0000，这样不会出问题。）
		5. 在写入数据时，如果指定地址为写保护状态，那么也不会执行编程。同时提出警告。
		6. 在整片擦除时，信息块不受影响（系统存储器和选项字节）
		7. 在编程过程中（BSY为1）时，任何读写闪存的操作都会使CPU暂停。直到此次读写结束。
			这时读写内部闪存存储数据的一个弊端。在闪存忙的时候，代码执行会暂停。
			会导致 在读写内部闪存的时候，中断响应不及时等对时间要求比较严格的。
		8. 对选项字节来编程的时候：WRP0位都是反逻辑位：0为实施写保护，1为取消写保护（因为闪存擦除之后都是1，1 是默认的）
		9. 善用STM32 ST-LINK Utility 调试工具
	## 5. STM32 FLASH 基本结构 {toggle="true"}
		以C8T6为例
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3bf0f355-0d63-4908-804a-e45de120c904.webp)
	## 6. STM32 FLASH 读写步骤 {toggle="true"}
		### 6.1 FLASH 解除或添加 读、写保护的方法 {toggle="true"}
			FLASH需要再写入之前解除写保护。这里的操作方式和独立看门狗一样。是通过键寄存器写入特定的键值来实现。可以防止误操作
			- FPEC共有三个键值：
				1. RDPRT键 （解除**读**保护）= 0x000000A5
				2. KEY1 （解除**写**保护1）= 0x45670123
				3. KEY2 （解除**写**保护2）= 0xCDEF89AB
			- 解锁：
				1. 复位后，FPEC被保护，不能写入FLASH_CR（默认是锁的）
				2. 在FLASH_KEYR先写入KEY1，再写入KEY2，解锁
				3. 错误的操作序列会在下次复位前锁死FPEC和FLASH_CR
			- 加锁：
				1. 设置FLASH_CR中的LOCK位（写1）锁住FPEC和FLASH_CR<br>
		### 6.2 FLASH 如何使用指针 读写存储器的方法 {toggle="true"}
			**使用指针读指定地址下的存储器：**
			- uint16_t Data = \*((__IO uint16_t \*)(0x08000000));
			**使用指针写指定地址下的存储器：**
			- \*((__IO uint16_t \*)(0x08000000)) = 0x1234;
			**其中：<br>**#define    __IO    volatile<br>__IO就是 volatile
			是C语言中易变的数据，这是一个安全保障措施。
			1. 一能**防止编译器优化**，（连续对某个变量赋值，或者执行空循环 会在开启优化时被优化）
			2. 二能告诉编译器，这个变量是个易变的数据。**每次读取都要到位**，直接从内存中找，不要去缓存中读取。<br>
		### 6.3 FLASH 闪存 全 擦除 时 过程 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/22e634df-ad1f-473c-b0eb-f8ae0233e2c3.webp)
			1. 读取LOCK位，看看芯片是否被锁，
				- 如果锁住，就执行解锁过程（在KEYR寄存器先写入KEY1，再写入KEY2）
				- 如果解锁，就置控制寄存器的<span color="green">**MER**</span>（Mass Erase 大规模擦除）再置STRT（Start 开始）为1
			2. 判断状态控制器BSY（Busy 忙）是否为1 ， 如果为1 就继续判断。等待他忙完（BSY = 0）跳出循环
			3. 最后一步验证一般不管。
		### 6.4 FLASH 闪存 页 擦除 时 过程 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/6a719c6d-4bb8-4135-9298-4dc2cd6c9cda.webp)
			1. 读取LOCK位，看看芯片是否被锁，
				- 如果锁住，就执行解锁过程（在KEYR寄存器先写入KEY1，再写入KEY2）
				- 如果解锁，就置控制寄存器的<span color="green">**PER**</span>**（**Page Erase 大规模擦除）、**<br>**然后在**AR**（Address Register 地址寄存器）选择要擦除的页。<br>最后置STRT（Start 开始）为1
			2. 判断状态控制器BSY（Busy 忙）是否为1 ， 如果为1 就继续判断。等待他忙完（BSY = 0）跳出循环
			3. 最后一步验证一般不管。
		### 6.5 FLASH 闪存 写入 时 过程 {toggle="true"}
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/fc0c514e-d9e4-424c-9af5-1c4d6eba95e6.webp)
			STM32的闪存在写入之前会检查指定地址有没有擦除。如果没有擦除就写入，<br>STM32则不执行写入操作（除非写入的数据全是0）
			1. 读取LOCK位，看看芯片是否被锁，
				- 如果锁住，就执行解锁过程（在KEYR寄存器先写入KEY1，再写入KEY2）
				- 如果解锁，就置控制寄存器的PG**（Programming** 程序编制）表示我们即将写入数据、
			2. 在指定的地址写入半字（16位）：`*((__IO uint16_t *)(0x08000000)) = 0x1234;` (不需要置STRT了)
			3. 判断状态控制器BSY（Busy 忙）是否为1 ， 如果为1 就继续判断。等待他忙完（BSY = 0）跳出循环
			4. 最后一步验证一般不管。
	## 7. STM32 选项字节 的组织和用途 {toggle="true"}
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/32d165e7-051e-4243-9605-9570dd798249.webp)
		选项字节存储的区域只有16个字节。起始地址为0x1FFF F800
		其中有一半的字节前边都带了个n（比如USER和nUSER……）
		这个的意思是，在写入USER时，要同时写入nUSER的反码..其他都是一样
		只有芯片检测到这两个芯片是反码的关系才会执行相对应的功能。
		这是一个安全保障措施（硬件会自动计算反码并填入）
		**每个存储器的功能**
		- RDP：写入RDPRT键（0x000000A5）后解除读保护
		- USER：配置硬件看门狗和进入停机/待机模式是否产生复位
		- Data0/1：用户可自定义使用
		- WRP0/1/2/3：配置写保护，每一个位对应保护4个存储页（中容量）
			- WRP0位都是反逻辑位：0为实施写保护，1为取消写保护（因为闪存擦除之后都是1，1 是默认的）
			- （ （小容量产品32K）也是每位保护4页，所以只需要WRP0一个字节就够了）
				（ （大容量产品512K）每位保护2页，但是WRP3的位7直接剩下把所有页（62\~255页）全部保护）<br>
	## 8. STM32 选项字节 的擦除和编程 {toggle="true"}
		选项字节本身也是闪存，所以在写入前也要擦除。流程和程序存储器类似，但细节有些出入
		### 擦除 选项字节时 过程 {toggle="true"}
			1. 解锁FLASH闪存   （手册中没写，但一定要打开 这个相当于 门口大锁）
			2. 检查FLASH_SR的BSY位，以确认没有其他正在进行的闪存操作  （事前等待）
			3. 解锁FLASH_CR的OPTWRE （Option Write Enable 选项写入使能）位  （相当于卧室小锁）
			4. 设置FLASH_CR的OPTER（Option Erase 选项清除）位为1 （即将擦除选项字节）
			5. 设置FLASH_CR的STRT（Start 开始）位为1   （触发芯片开始干活）   
			6. 等待BSY位变为0   （等待忙完）
			7. 读出被擦除的选择字节并做验证<br><br>
			<empty-block/>
		### 编程 选项字节时 过程 {toggle="true"}
			1. 解锁FLASH闪存   （手册中没写，但一定要打开 这个相当于 门口大锁）
			2. 检查FLASH_SR的BSY位，以确认没有其他正在进行的闪存操作  （事前等待）
			3. 解锁FLASH_CR的OPTWRE （Option Write Enable 选项写入使能）位  （相当于卧室小锁）
			4. 设置FLASH_CR的OPTPG（Option Programming 选项编程）位为1 （即将开始编程写入）
			5. 写入要编程的半字（16位）到指定的地址
			6. 等待BSY位变为0   （等待忙完）
			7. 读出被擦除的选择字节并做验证
	## 9 . STM32 FLASH 器件的电子签名 {toggle="true"}
		电子签名存放在闪存存储器模块的系统存储区域（就是BootLoader哪里），包含的芯片识别信息在出厂时编写，不可更改，使用指针读指定地址下的存储器可获取电子签名
		闪存容量寄存器：（显示闪存容量）<br>基地址：0x1FFF F7E0<br>大小：16位
		产品唯一身份标识寄存器：（ 可以用来作为序列号、防被盗、激活秘钥）<br>基地址： 0x1FFF F7E8<br>大小：96位<br>可以以字节、半字、字的方式读取<br>
	## 10. STM32 ST-LINK Utility调试工具的使用 {toggle="true"}
		1. **连接**
			- 插入好ST link之后点击连接（不能在stm32休眠模式下连接）
		2. 退出
			- 在使用完ST-LINK Utility调试工具后要及时断开，不然KEIl下载不了程序
		3. **查看程序数据**
			- 连接后，下面窗口中就是闪存中的数据了
			- 可以指定地址 查看数据、指定查看范围、指定以什么类型去看（字节、半字、字、）
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/c6c0f63c-316a-44e9-bc1f-55574d7b9034.webp)
		1. **查看选项字节配置**
			- 进入
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/74e723ef-cb50-4b69-8137-a0453d08ad27.webp)
			- 功能介绍
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/3f50a643-9e07-4145-b057-437e8971f4a8.webp)
	## 11. KEIL FLASH 下载程序 需要注意的设置 {toggle="true"}
		### 11.1 下载程序起始位置。划定范围 {toggle="true"}
			在下载程序时，我们可以选定FLASH和RAM的起始位置。这样下载程序就可以指定位置下载。并且可以限制最大到那个位置，
			比如要下载一个自己写的BootLoader。就可以指定到页尾去下载。
			或者程序的最后几页，打算存储数据用，那么就可以通过划定程序的下载范围，不让下载时覆盖掉自己想保存的数据。 当然，如果程序过大，你划的范围幼小，那么会下载失败。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/d365ec6a-f79b-4346-bf7b-f6968031772a.webp)
		### 11.2 下载程序 时 对FLASH擦除选项 {toggle="true"}
			一般使用用多少擦除多少。下载速度快且不会动后面的程序
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/327a8aa9-9bf3-4ba8-9e5e-28c233ab1396.webp)
		### 11.3 查看程序占用大小 {toggle="true"}
			在程序编译之后，会有一段：Program Size ：……
			前三个数为占用FLASH程序存储的大小， 后俩数 是占用SRAM的大小
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/93d3fbb5-ea13-4a42-b561-b4c4d9178e40.webp)
			<empty-block/>
			**也可以在Target处直接双击，在.map文件拖到最后边。也有大小显示。**
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/246c37d2-85cd-4c46-bda8-6c7fdcede8d6.webp)
	## 12. STM32 flash.h介绍 {toggle="true"}
		### flash.h中三个部分的介绍 {toggle="true"}
			如下是flash.h中的函数声明
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E5%B5%8C%E5%85%A5%E5%BC%8F%E8%BD%AF%E4%BB%B6%E5%BC%80%E5%8F%91/Post-stm32-stdlib/af3bf0cc-ee45-4a1c-bf0a-e18b146b233e.webp)
			**可以看到在flash.h中可以看到有三块区域。分别对应了**
			- 这些是所有F10x设备机都可以使用的函数
			- 所有设备机都可以使用的、新的函数
			- 只有XL加大容量的设备才可以使用的、新的函数（有个预编译，定义了宏才有效）
			**他们的产生原因是：**
			- 期初在stm32中，最初 只有小容量LD、中容量MD、大容量HD
			- 之后，加大容量XL才推出。XL是添加了一块新的、独立的闪存。即有两块
				所以设计者命名新加的一块交Bank2 。与之对应，原来的小中大容量的那一块叫做Bank1
			- 在加大容量系列出来之后，对flash.c .h中的函数 进行了适配和更新（具体更改可以在.c中查看）
			<empty-block/>
		### <span color="blue">适用于所有stm32F10x设备的函数（第一部分）函数介绍</span> {toggle="true"}
			和内核运行代码有关，不需要过多了解
				`void FLASH_SetLatency(uint32_t FLASH_Latency);`
				`void FLASH_HalfCycleAccessCmd(uint32_t FLASH_HalfCycleAccess);`
				`void FLASH_PrefetchBufferCmd(uint32_t FLASH_PrefetchBuffer);`
			> `void FLASH_Unlock(void);`
				- 解锁
			> `void FLASH_Lock(void);`
				- 加锁
			> `FLASH_Status FLASH_ErasePage(uint32_t Page_Address);`
				- 闪存擦除某一页（返回值为完成状态）
			> `FLASH_Status FLASH_EraseAllPages(void);`
				- 闪存全擦除（返回值为完成状态）
			> `FLASH_Status FLASH_EraseOptionBytes(void);`
				- 擦除选项字节（返回值为完成状态）
			> `FLASH_Status FLASH_ProgramWord(uint32_t Address, uint32_t Data);`
				- 在指定地址写入字
			> `FLASH_Status FLASH_ProgramHalfWord(uint32_t Address, uint16_t Data);`
				- 在指定地址写入半字
			`FLASH_Status FLASH_ProgramOptionByteData(uint32_t Address, uint8_t Data);`
				- 选项字节：自定义Data 0 、 1
			`FLASH_Status FLASH_EnableWriteProtection(uint32_t FLASH_Pages);`
				- 写保护
			`FLASH_Status FLASH_ReadOutProtection(FunctionalState NewState);`
				- 读保护
			`FLASH_Status FLASH_UserOptionByteConfig(uint16_t OB_IWDG, uint16_t OB_STOP, uint16_t OB_STDBY);`
				- 用户选项的三个配置位
			`uint32_t FLASH_GetUserOptionByte(void);`
				- 获取用户选项的三个配置位
			`uint32_t FLASH_GetWriteProtectionOptionByte(void);`
				- 获取写保护状态
			`FlagStatus FLASH_GetReadOutProtectionStatus(void);`
				- 获取读保护状态
			`FlagStatus FLASH_GetPrefetchBufferStatus(void);`
				- 获取预取缓冲区状态
			`void FLASH_ITConfig(uint32_t FLASH_IT, FunctionalState NewState);FlagStatus`
				- 中断使能
			`FLASH_GetFlagStatus(uint32_t FLASH_FLAG);`
				- 获取标志位
			`void FLASH_ClearFlag(uint32_t FLASH_FLAG);`
				- 清除标志位
			`FLASH_Status FLASH_GetStatus(void);`
				- 获取状态
			`FLASH_Status FLASH_WaitForLastOperation(uint32_t Timeout);`
				- 等待上一次操作（等待BSY）<br>函数内部会自动调用，并不需要我们单独调用
	## 13. 编写：读写内部FLASH {toggle="true"}
		### 11.1 工程目标 {toggle="true"}
			**在闪存最后一页进行读写。**
			- 为了方便读写，提高效率。在SRAM中定义数组来对数组操作，通过函数间接控制FLASH。
			- SRAM在每次更改时，都把自己以整体备份到闪存中
			- 在每次上电时，把闪存中的 数据初始化加载到SRAM数组中
			- 另外为了判断此闪存是否之前保存过数据，使用页的第一个半字来存放标志位。如果标志位有，那么上电就直接加载闪存数据到SRAM中就可以了。如果不是就把标志位放进去，然后初始化闪存，再把闪存的数据搬运到SRAM中
		### 10.1 工程结构 {toggle="true"}
			- MyFLASH.c ：实现闪存最基本的三个功能：读取、擦除、编程
			- Store.c ：实现对数据的读写和存储管理：定义SRAM数组，把SRAM数组自动备份到FLASH里。复位、上电后，闪存数据会自动读回道SRAM。
			- main.c ： 测试读写内部FLASH
		### MyFLASH.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header

/**
  * 函    数：FLASH读取一个32位的字
  * 参    数：Address 要读取数据的字地址
  * 返 回 值：指定地址下的数据
  */
uint32_t MyFLASH_ReadWord(uint32_t Address)
{
    return *((__IO uint32_t *)(Address));	//使用指针访问指定地址下的数据并返回
}

/**
  * 函    数：FLASH读取一个16位的半字
  * 参    数：Address 要读取数据的半字地址
  * 返 回 值：指定地址下的数据
  */
uint16_t MyFLASH_ReadHalfWord(uint32_t Address)
{
    return *((__IO uint16_t *)(Address));	//使用指针访问指定地址下的数据并返回
}

/**
  * 函    数：FLASH读取一个8位的字节
  * 参    数：Address 要读取数据的字节地址
  * 返 回 值：指定地址下的数据
  */
uint8_t MyFLASH_ReadByte(uint32_t Address)
{
    return *((__IO uint8_t *)(Address));	//使用指针访问指定地址下的数据并返回
}

/**
  * 函    数：FLASH全擦除
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，FLASH的所有页都会被擦除，包括程序文件本身，擦除后，程序将不复存在
  */
void MyFLASH_EraseAllPages(void)
{
    FLASH_Unlock();                 //解锁
    FLASH_EraseAllPages();          //全擦除
    FLASH_Lock();                   //加锁
}

/**
  * 函    数：FLASH页擦除
  * 参    数：PageAddress 要擦除页的页地址
  * 返 回 值：无
  */
void MyFLASH_ErasePage(uint32_t PageAddress)
{
    FLASH_Unlock();                 //解锁
    FLASH_ErasePage(PageAddress);   //页擦除
    FLASH_Lock();                   //加锁
}

/**
  * 函    数：FLASH编程字
  * 参    数：Address 要写入数据的字地址
  * 参    数：Data 要写入的32位数据
  * 返 回 值：无
  */
void MyFLASH_ProgramWord(uint32_t Address, uint32_t Data)
{
    FLASH_Unlock();                         //解锁
    FLASH_ProgramWord(Address, Data);       //编程字
    FLASH_Lock();                           //加锁
}

/**
  * 函    数：FLASH编程半字
  * 参    数：Address 要写入数据的半字地址
  * 参    数：Data 要写入的16位数据
  * 返 回 值：无
  */
void MyFLASH_ProgramHalfWord(uint32_t Address, uint16_t Data)
{
    FLASH_Unlock();                         //解锁
    FLASH_ProgramHalfWord(Address, Data);   //编程半字
    FLASH_Lock();                           //加锁
}

			```
		### MyFLASH.h {toggle="true"}
			```c
#ifndef __MYFLASH_H
#define __MYFLASH_H

//读出字
uint32_t MyFLASH_ReadWord(uint32_t Address);
//读出半字
uint16_t MyFLASH_ReadHalfWord(uint32_t Address);
//读出字节
uint8_t MyFLASH_ReadByte(uint32_t Address);

//清除所有页
void MyFLASH_EraseAllPages(void);
//清除某页
void MyFLASH_ErasePage(uint32_t PageAddress);

//编写字
void MyFLASH_ProgramWord(uint32_t Address, uint32_t Data);
//编写半字
void MyFLASH_ProgramHalfWord(uint32_t Address, uint16_t Data);

#endif

			```
		### Store.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "MyFLASH.h"

#define STORE_START_ADDRESS     0x0800FC00      //存储的起始地址
#define STORE_COUNT             512             //存储数据的个数

uint16_t Store_Data[STORE_COUNT];               //定义SRAM数组

/**
  * 函    数：参数存储模块初始化
  * 参    数：无
  * 返 回 值：无
  */
void Store_Init(void)
{
    /*判断是不是第一次使用*/
    if (MyFLASH_ReadHalfWord(STORE_START_ADDRESS) != 0xA5A5)    //读取第一个半字的标志位，if成立，则执行第一次使用的初始化
    {
        MyFLASH_ErasePage(STORE_START_ADDRESS);                 //擦除指定页
        MyFLASH_ProgramHalfWord(STORE_START_ADDRESS, 0xA5A5);   //在第一个半字写入自己规定的标志位，用于判断是不是第一次使用
        
        for (uint16_t i = 1; i < STORE_COUNT; i ++)             //循环STORE_COUNT次，除了第一个标志位
        {
            MyFLASH_ProgramHalfWord(STORE_START_ADDRESS + i * 2, 0x0000);       //除了标志位的有效数据全部清0
        }
    }
    
    /*上电时，将闪存数据加载回SRAM数组，实现SRAM数组的掉电不丢失*/
    for (uint16_t i = 0; i < STORE_COUNT; i++)             //循环STORE_COUNT次，包括第一个标志位
    {
        Store_Data[i] = MyFLASH_ReadHalfWord(STORE_START_ADDRESS + i * 2);      //将闪存的数据加载回SRAM数组
    }
}

/**
  * 函    数：SRAM数组保存数据到FLASH闪存
  * 参    数：无
  * 返 回 值：无
  */
void Store_Save(void)
{
    MyFLASH_ErasePage(STORE_START_ADDRESS);             //擦除指定页
    
    for (uint16_t i = 0; i < STORE_COUNT; i ++)         //循环STORE_COUNT次，包括第一个标志位
    {
        MyFLASH_ProgramHalfWord(STORE_START_ADDRESS + i * 2, Store_Data[i]);    //将SRAM数组的数据备份保存到闪存
    }
}

/**
  * 函    数：SRAM数组清零，再同步到FLASH闪存
  * 参    数：无
  * 返 回 值：无
  */
void Store_Clear(void)
{
    for (uint16_t i = 1; i < STORE_COUNT; i ++)         //循环STORE_COUNT次，除了第一个标志位
    {       
        Store_Data[i] = 0x0000;                         //SRAM数组有效数据清0
    }                       
    
    Store_Save();                                       //保存数据到闪存
}

			```
		### Store.h {toggle="true"}
			```c
#ifndef __STORE_H
#define __STORE_H

//对外声明保存最后一页的SRAM数组
extern uint16_t Store_Data[];

//初始化
void Store_Init(void);
//把SRAM数组写入FLASH
void Store_Save(void);
//清除FLASH（除了标志位）
void Store_Clear(void);

#endif

			```
		### main.c  {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Store.h"
#include "Key.h"

int main(void)
{
    /*模块初始化*/
    OLED_Init();                //OLED初始化
    KEY_Init();                //按键初始化
    Store_Init();               //参数存储模块初始化，在上电的时候将闪存的数据加载回Store_Data，实现掉电不丢失
    
    /*显示静态字符串*/
    OLED_ShowString(1, 1, "Flag:");
    OLED_ShowString(2, 1, "Data:");
    
    while (1)
    {
        uint8_t Key_Num = KEY_Get();    //记录按下的键   
        if (Key_Num == 1)               //按键1按下
        {
            Store_Data[1] ++;           //变换测试数据
            Store_Data[2] += 2;
            Store_Data[3] += 3;
            Store_Data[4] += 4;
            Store_Save();               //将Store_Data的数据备份保存到闪存，实现掉电不丢失
        }
        
        else if (Key_Num == 2)          //按键2按下
        {
            Store_Clear();              //将Store_Data的数据全部清0
        }
        
        OLED_ShowHexNum(1, 6, Store_Data[0], 4);    //显示Store_Data的第一位标志位
        
        OLED_ShowHexNum(3, 1, Store_Data[1], 4);    //显示Store_Data的有效存储数据
        OLED_ShowHexNum(3, 6, Store_Data[2], 4);
        OLED_ShowHexNum(4, 1, Store_Data[3], 4);
        OLED_ShowHexNum(4, 6, Store_Data[4], 4);
    }
}

			```
	## 14. 编写：读写内部ID {toggle="true"}
		在13的基础上  只修改了main.c
		### main.c {toggle="true"}
			```c
#include "stm32f10x.h"                  // Device header
#include "Delay.h"
#include "OLED.h"
#include "Store.h"

int main(void)
{   
    //0x1FFFF7E0为FLASH ID 寄存器位置，。是一个96位的
    
    OLED_Init();                        //OLED初始化
    
    OLED_ShowString(1, 1, "F_SIZE:");   //显示容量：
    
    OLED_ShowHexNum(1, 8, *((__IO uint16_t *)(0x1FFFF7E0)), 4);     //使用指针读取指定地址下的 闪存容量寄存器
    
    OLED_ShowString(2, 1, "U_ID:");     //显示芯片ID：
    
    //使用指针读取指定地址下的产品唯一身份标识寄存器
    //可以由字节、半字、字的方式读取
    OLED_ShowHexNum(2, 6, *((__IO uint8_t *)(0x1FFFF7E8)), 2);              //字节读出：第一个字节        （0-8位）
    OLED_ShowHexNum(2, 9, *((__IO uint8_t *)(0x1FFFF7E8) + 1), 2);          //字节读出：第二个字节        （9-16位）
    OLED_ShowHexNum(2, 12, *((__IO uint16_t *)(0x1FFFF7E8) + 1), 4);        //半字读出：第3 、4 个字节    （16-32位）
    OLED_ShowHexNum(3, 1, *((__IO uint32_t *)(0x1FFFF7E8 + 1)), 8);      //字  读出：5 6 7 8个字节     （33-64位）
    OLED_ShowHexNum(4, 1, *((__IO uint32_t *)(0x1FFFF7E8 + 2)), 8);      //字  读出：9 10 11 12个字节  （64-96位）
    
    while (1)
    {

    }
}

			```
<empty-block/>
<empty-block/>