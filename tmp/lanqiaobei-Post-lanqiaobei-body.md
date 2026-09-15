<callout icon="😀" color="gray_bg">
	此笔记是我在备考蓝桥杯嵌入式时所记录的。标\*\*的意为最终方案，可直接跳转学习
</callout>
> 现在（6月15日）已经考完了。考到了我笔记中的时间转换时间戳以及修改rtc时间和字符串处理的函数。不算难，但是量太多了！！最后十分钟写完了没来得及全部验证，最后五分钟看到好多bug,改改又交了三次。大心脏差点承受不住。希望有个好成绩！—\>老三
# 新建工程 
当前单片机芯片：STMG431RBT6
<empty-block/>
**比赛开始时，解压提供的库文件。**
在工具-Cube-中可以找到不同版本的
<empty-block/>
**安装提供的芯片包**
<empty-block/>
Cube打开后，选择芯片
打开RCC的外部时钟 选择Debug调试方式为串口
外部晶振选择为24MHZ（具体看手册），系统时钟选择80MHZ(HSE、PLL）
设置MDK，设置项目名称，勾选分离.c.h文件
取消勾选使用默认的库文件位置，选择赛事方提供的解压好的库文件。
设置，下载器为DAP；设置下载后自动运行。
<empty-block/>
# 点个灯 
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/ebf1c0bc-a37d-461b-a325-662fdcc57316.webp)
使用寄存器的原因是：因为LCD也用了这几个引脚，为了防止在使用LCD的时候改变LED
<empty-block/>
**初始化时，将所有置为1，PD和PC都是，让LED默认为熄灭**
<empty-block/>
核心代码：
```c
#include "sys.h"

uint8_t led_state = 0x00;

void led_disp(void)
{
	HAL_GPIO_WritePin(GPIOD, GPIO_PIN_2, GPIO_PIN_SET);
	
	HAL_GPIO_WritePin(GPIOC, 0xff00, GPIO_PIN_SET);//熄灭所有LED
	HAL_GPIO_WritePin(GPIOC, led_state <<8, GPIO_PIN_RESET);//根据bit位点亮led
	
	HAL_GPIO_WritePin(GPIOD, GPIO_PIN_2, GPIO_PIN_RESET);
}

```
# 使用LCD模块 
**初始化**
复制资料的lcd.c 、 lcd.h、fonts.h
到Code文件中
在main.c中初始化LCD
```c
LCD_Init();								//LCD初始化
LCD_Clear(Black);					//清除屏幕为黑色
LCD_SetBackColor(Black);	//设置背景色为黑色
LCD_SetTextColor(White);	//设置文字色为白色
```
核心代码
```c
char text[20];				//一行刚好显示20个字符
void lcd_show(void){
	sprintf(text, "Hello world");	//将字符串复制到字符数组
	LCD_DisplayStringLine(Line0, (uint8_t*)text);
	
	sprintf(text, "count:%d", count);	
	LCD_DisplayStringLine(Line3, (uint8_t*)text);
}
```
<empty-block/>
高亮部分字符函数
```c
void LCD_displayHight(uint8_t Line, uint8_t *str, uint8_t pos, uint8_t len) {
    uint8_t i;
    for (i = 0; str[i] != '\0'; i++) {
        if (i >= pos && i < pos + len)	// 自己选高亮的位置
            LCD_SetBackColor(Yellow);	// 高亮部分
        else
            LCD_SetBackColor(Black);	// 非高亮部分
		
        LCD_DisplayChar(Line, 320 - (16 * i), str[i]);
    }
    // 恢复背景颜色为黑色
    LCD_SetBackColor(Black);
}
```
闪烁部分字符函数
```c
#define SHANSHUO_TIME 500   // 闪烁时间
void LCD_displayShanshuo(uint8_t Line, uint8_t *str, uint8_t pos, uint8_t len) {
    static uint32_t lastBlinkTime = 0;
    static uint8_t blinkState = 0;  // 0 表示显示字符，1 表示显示空白

    // 判断是否到了切换闪烁状态的时间
    if (uwTick - lastBlinkTime >= SHANSHUO_TIME) {
        blinkState = !blinkState;  // 切换闪烁状态
        lastBlinkTime = uwTick;    // 更新上次闪烁时间
    }

    uint8_t i;
    for (i = 0; str[i] != '\0'; i++) {
        if (i >= pos && i < pos + len) {
            if (blinkState) {
                // 显示空白，用空格字符 ' ' 表示
                LCD_DisplayChar(Line, 320 - (16 * i), ' ');
            } else {
                // 显示字符
                LCD_DisplayChar(Line, 320 - (16 * i), str[i]);
            }
        }else{
            // 显示非闪烁部分
            LCD_DisplayChar(Line, 320 - (16 * i), str[i]);
		}
	}
}
```
# 解决LCD和LED引脚冲突的问题
1. 在GPIO
修改`LCD_DisplayStringLine`的函数，在第一行和最后一行加上如下代码
同理，例如`LCD_Init`、` LCD_Clear`  都需要加上这两行代码。 注意是uint16_t
```c
void LCD_DisplayStringLine(u8 Line, u8 *ptr)
{
		uint16_t temp = GPIOC->ODR;	//存储GPIOC的输出状态
	
    u32 i = 0;
    u16 refcolumn = 319;//319;

    while ((*ptr != 0) && (i < 20))	 //	20
    {
        LCD_DisplayChar(Line, refcolumn, *ptr);
        refcolumn -= 16;
        ptr++;
        i++;
    }
		GPIOC->ODR = temp;			//恢复GPIOC的输出状态
}
```
<empty-block/>
<empty-block/>
# 闪个灯 
定时器中断实现
<empty-block/>
开启定时器， 计算中断时间， psc  arr
开启NVIC中断
<empty-block/>
在tim.c中找到tim.h， 找到中断回调函数（2531 行），
<empty-block/>
核心代码：
```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM2) 	//定时器2中断函数
	{
		led_mode = !led_mode;	//0 1 循环
	}
	
}
```
注意不要在中断中写控制LED的函数，否则可能会在LCD函数中，打开PD2的使能，导致在复原之前修改了GPIOC的输出。
所以一般把`show_led(1, led_mode);`	写在LCD后
# 按键长按短按双击检测 
下面的太麻烦了， 简单的如下：
1. **仅支持抬起和按下、长按检测（极简版）**
```c
uint8_t key_value, key_down, key_up, key_old = 0;
void KEY_read(void)
{
	if(HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_0) == 0)
		key_value = 1;
	else if(HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_1) == 0)
		key_value = 2;
	else if(HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_2) == 0)
		key_value = 3;
	else if(HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == 0)
		key_value = 4;
	else
		key_value = 0;
	
	key_down = key_value & (key_value ^ key_old);
	key_up = ~key_value & (key_value ^ key_old);
	key_old = key_value;
	
	
	if(key_down == 4) //按下
	{
		key_long_tick = uwTick;//开始计时
	}
	else if(key_vlaue == 4 && uwTick - key_long_tick > 800) //按下，长按生效
	{
				Naozhong_Time.AlarmTime.Hours++;
	}
}
```
<empty-block/>
一个简单的不需要任何外设的单击双击长按代码。基于上述代码改造
```c

#define DOUBLE_TIME  300
#define SHORT_TIME  500
#define LONG_TIME  1000

uint8_t key_value, key_old , key_up, key_down;
uint32_t key_down_time = 0;
uint32_t key_up_time = 0;
uint8_t key_count = 0;
uint8_t key_last = 0;

uint8_t key_short, key_double, key_long;


uint32_t key_tick = 0;
void key_proc(void)
{
	if(uwTick - key_tick < 20)
		return;
	key_tick = uwTick;
	
	uint8_t key_mask = 0;
	key_mask |= (HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_0) == GPIO_PIN_RESET) ? 0x01 : 0;
	key_mask |= (HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_1) == GPIO_PIN_RESET) ? 0x02 : 0;
	key_mask |= (HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_2) == GPIO_PIN_RESET) ? 0x04 : 0;
	key_mask |= (HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0) == GPIO_PIN_RESET) ? 0x08 : 0;

	uint8_t key_new_vlaue = 0;
	if(key_mask == (0x01 + 0x02))		//key 1 + 2
		key_new_vlaue = 5;
	else if(key_mask == (0x01 + 0x04))  //key 1 + 3
		key_new_vlaue = 6;
	else{
		if(key_mask == 0x01) 	key_new_vlaue = 1;
		else if(key_mask == 0x02) 	key_new_vlaue = 2;
		else if(key_mask == 0x04) 	key_new_vlaue = 3;
		else if(key_mask == 0x08) 	key_new_vlaue = 4;
	}
	
	key_down = key_new_vlaue & (key_new_vlaue ^ key_old);
	key_up = ~key_new_vlaue & (key_new_vlaue ^ key_old);
	key_old = key_value = key_new_vlaue;
	
	if(key_down)
	{
		key_down_time = uwTick;
		key_last = key_value;
		if(uwTick - key_up_time < DOUBLE_TIME && key_last == key_value)
		{
			key_double = key_value;	//触发双击
			key_count = 0;
		}
	}
	
	if(key_up)
	{
		if(uwTick - key_down_time < SHORT_TIME && key_double == 0)
		{
			key_up_time = uwTick;
			key_count++;
		}
	}
	
	if(key_value && (uwTick - key_down_time > LONG_TIME))
	{
		key_long = key_value;	//触发长按
		key_count = 0;
	}
	
	if(!key_value && (uwTick - key_up_time > DOUBLE_TIME))  //将此处DOUBLE_TIME 替换为0则双击无效，短按触发极快
	{
		if(key_count == 1)
			key_short = key_last;	//触发短按
		else
			key_short = 0;
		
		key_count = 0;
		key_double = 0;
		key_long = 0;
	}
	
	// if(key_double || key_long)
		// key_short = 0;
	
}
```
<empty-block/>
<empty-block/>
<empty-block/>
<empty-block/>
（Deepseek版， 需要定时器↓）
注意在main.c中开启定时器IT方式哦
<empty-block/>
```c
/*		按键处理			*/

// 按键状态结构体
typedef struct {
    GPIO_TypeDef* port;   // 端口
    uint16_t pin;         // 引脚
    int time;             // 周期计时(2s清零)
    uint8_t timeEN;       // 计时使能
    uint8_t count;        // 按下次数
    uint8_t press_flag;   // 按下标志
    int press_time;       // 按下时长
} Key;

// 初始化四个按键（B1-B4）
Key keys[] = {
    {GPIOB, GPIO_PIN_0, 0, 0, 0, 0, 0}, // B1
    {GPIOB, GPIO_PIN_1, 0, 0, 0, 0, 0}, // B2
    {GPIOB, GPIO_PIN_2, 0, 0, 0, 0, 0}, // B3
    {GPIOA, GPIO_PIN_0, 0, 0, 0, 0, 0}  // B4
};

//定义按键个数
#define KEY_COUNT (sizeof(keys)/sizeof(keys[0]))	
	
// 按键扫描逻辑
void key_scan(void) {
    for(uint8_t i=0; i<KEY_COUNT; i++) {	//逐个扫描按键
        Key* k = &keys[i];
        
        if(k->time > 1800) { // 检测2s周期末端
            if(k->count == 1) {	//如果只按了一下
                if(k->press_time < 1000) {	
                    sprintf(text, "Btn%d: Click", i+1);	//并且时间小于1s，那么是短按
                } else {
                    sprintf(text, "Btn%d: Long ", i+1);	//并且时间大于1s，那么是长按
                }
                LCD_DisplayStringLine(Line2, (uint8_t*)text);	//显示出来
            }
            else if(k->count == 2 && k->press_time < 1000) {	//如果按了两下，并且时间小于1s
                sprintf(text, "Btn%d: Double", i+1);
                LCD_DisplayStringLine(Line2, (uint8_t*)text);	//那么显示双击
            }
            
            // 重置状态
            k->time = k->count = k->press_time = k->timeEN = 0;
        }
    }
}

//定时器终端扫描处理
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
	//定时器3中断函数 1ms一次
	else if(htim->Instance == TIM3)			
	{
		for(uint8_t i=0; i<KEY_COUNT; i++) {	//逐个扫描按键
            Key* k = &keys[i];
            
            // 检测按键动作
            uint8_t pinState = HAL_GPIO_ReadPin(k->port, k->pin);
            if(pinState == GPIO_PIN_RESET && !k->press_flag) { // 按下
                k->press_flag = 1;	//标记按下
                k->timeEN = 1;		//开始计时
                k->count++;			//记录按下次数
            } 
            else if(pinState == GPIO_PIN_SET && k->press_flag) { // 释放
                k->press_flag = 0;	//标记松开
            }
            
            // 更新按下时长
            if(k->press_flag) k->press_time++;
            
            // 周期计时
            if(k->timeEN) {
                if((k->time++) > 2000) { // 满2s自动重置
                    k->time = k->count = k->press_time = k->timeEN = 0;
                }
            }
        }
	}
	
}
```
<empty-block/>
# LCD高亮显示 
通过`LCD_SetBackColor(Black);`
`LCD_SetBackColor(Blue);`
等两种不同颜色的背景，来实现修改高亮
核心代码
```c
if(lcd_highshow == 4) lcd_highshow = 1;
	if(lcd_highshow == 0) lcd_highshow = 3;

	if(lcd_highshow == 1)
	{
		LCD_SetBackColor(Blue);
		sprintf(text, "Hello world");	//将字符串复制到字符数组
		LCD_DisplayStringLine(Line0, (uint8_t*)text);
		LCD_SetBackColor(Black);
		sprintf(text, "count:%d        ", count);	
		LCD_DisplayStringLine(Line1, (uint8_t*)text);
		sprintf(text, "Teeeeest        ");	
		LCD_DisplayStringLine(Line2, (uint8_t*)text);
	}
	else if(lcd_highshow == 2)
	{
		
		sprintf(text, "Hello world");	//将字符串复制到字符数组
		LCD_DisplayStringLine(Line0, (uint8_t*)text);
		LCD_SetBackColor(Blue);
		sprintf(text, "count:%d        ", count);	
		LCD_DisplayStringLine(Line1, (uint8_t*)text);
		LCD_SetBackColor(Black);
		sprintf(text, "Teeeeest        ");	
		LCD_DisplayStringLine(Line2, (uint8_t*)text);
	}
	else if(lcd_highshow == 3)
	{
		
		sprintf(text, "Hello world");	//将字符串复制到字符数组
		LCD_DisplayStringLine(Line0, (uint8_t*)text);
		sprintf(text, "count:%d        ", count);	
		LCD_DisplayStringLine(Line1, (uint8_t*)text);
		LCD_SetBackColor(Blue);
		sprintf(text, "Teeeeest        ");	
		LCD_DisplayStringLine(Line2, (uint8_t*)text);
		LCD_SetBackColor(Black);
	}
```
# PWM输出 
<callout icon="💡" color="gray_bg">
	注意 PWM输出的定时器不要和 其他的定时器冲突，否则会影响频率和占空比的设置
</callout>
例：
1000HZ, 占空比50，  PA1输出  方波
参考：psc80     arr1000    自动预加载启用       高电平时间  500
```c
	HAL_TIM_PWM_Start(&htim2, TIM_CHANNEL_2);	//启动TIM的CHANNEL2的PWM
	TIM2->CCR2 = 500;	//设置CCR为500
```
<empty-block/>
<empty-block/>
# 测量PWM的输出频率 
**使用输入捕获测量PWM频率**
测量两个上升沿之间的计数值，计算得到
<empty-block/>
设置psc为80-1，也就是 测量的频率为1MHZ
然后打开NVIC
<empty-block/>
记得使能哦
`HAL_TIM_IC_Start_IT`
输入捕获回调函数在 2534行 tim.h里哦
和定时器中断的回调不是一个
`void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim);`
计算频率：`pwm_f = 80000000/(80*capture_value)`
核心代码
```c
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM17)
	{
		//capture_value = HAL_TIM_ReadCapturedValue(&htim17, TIM_CHANNEL_1);	//捕获CNT值（实际读取CCR，捕获是把CNT赋值给CCR
		capture_value = TIM17->CCR 1;	//也可以这样捕获,更简洁
		TIM17->CNT = 0;	//清零
		
		pwm_f = 80000000/(capture_value) - 1; //(手动修正，哈哈哈)
	}

}
```
# 输入捕获测 输入频率+占空比 
输入捕获测频率
```c

char text[20];					//一行刚好显示20个字符

uint32_t capture1_value = 0;		//捕获值
uint32_t capture2_value = 0;
uint32_t pwm1_f = 0;			
uint32_t pwm2_f = 0;			

void show_lcd(void)
{
	sprintf(text, "Hello world");	//将字符串复制到字符数组
	LCD_DisplayStringLine(Line0, (uint8_t*)text);
	//显示输出频率1
	sprintf(text, "       fre1:%d       ", pwm1_f);
	LCD_DisplayStringLine(Line1, (uint8_t*)text);
	//显示输出频率2
	sprintf(text, "       fre2:%d       ", pwm2_f);
	LCD_DisplayStringLine(Line2, (uint8_t*)text);
}

void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM16)
	{
		capture1_value = TIM16->CCR1;
		TIM16->CNT = 0;	//清零
		pwm1_f = 80000000/(80*capture1_value); 
	}
	else if(htim->Instance == TIM2)
	{
		capture2_value = TIM2->CCR1;
		TIM2->CNT = 0;	//清零
		pwm2_f = 80000000/(80*capture2_value);
	}

}

```
<empty-block/>
输入捕获测量频率和占空比↓
大概是通过检测两次上升沿和上升沿和下降沿的比值计算的。 用 状态机
核心代码：
```c
uint32_t R39_fre, R40_fre, R39_dut, R40_dut,R39_High, R40_High, R39_OK, R40_OK;
uint8_t  R39_state, R39_flag, R40_state, R40_flag;
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM3)
	{
		if(R39_flag == 0)
		{
			switch(R39_state)
			{
				case 0: /* 捕获第一个上升沿 */
					TIM3->CNT = 0;
					R39_High = 0;
					R39_OK = 0;
					__HAL_TIM_SET_CAPTUREPOLARITY(&htim3, TIM_CHANNEL_1, TIM_ICPOLARITY_FALLING);
				
					R39_state = 1;
					break;
				
				case 1: /* 捕获第一个下降沿 */
					
					R39_High = TIM3->CCR1;
					__HAL_TIM_SET_CAPTUREPOLARITY(&htim3, TIM_CHANNEL_1, TIM_ICPOLARITY_RISING);
				
					R39_state = 2;
					break;
				case 2: /* 捕获第二个上升沿 */
					R39_OK = TIM3->CCR1;
					R39_flag = 1;
					R39_state = 0;
					break;
			}
		}
	}
	else if(htim->Instance == TIM8)
	{
		if(R40_flag == 0)
		{
			switch(R40_state)
			{
				case 0: /* 捕获第一个上升沿 */
					TIM8->CNT = 0;
					R40_High = 0;
					R40_OK = 0;
					__HAL_TIM_SET_CAPTUREPOLARITY(&htim8, TIM_CHANNEL_1, TIM_ICPOLARITY_FALLING);
				
					R40_state = 1;
					break;
				
				case 1: /* 捕获第一个下降沿 */
					
					R40_High = TIM8->CCR1;
					__HAL_TIM_SET_CAPTUREPOLARITY(&htim8, TIM_CHANNEL_1, TIM_ICPOLARITY_RISING);
				
					R40_state = 2;
					break;
				case 2: /* 捕获第二个上升沿 */
					R40_OK = TIM8->CCR1;
					R40_flag = 1;
					R40_state = 0;
					break;
			}
		}
	}
}

void Read_fre_dut_proc(void)
{
	if(R39_flag == 1)
	{
		R39_flag = 0;
		R39_fre = (double)1000000 / (R39_OK + 1);
		R39_dut = (double)R39_High / R39_OK *100;
	}
	if(R40_flag == 1)
	{
		R40_flag = 0;
		R40_fre = (double)1000000 / (R40_OK + 1);
		R40_dut = (double)R40_High / R40_OK *100;
	}
}

```
# ADC 软件触发 
PB15   R37
PB12    R38
DAC1 PA4
核心代码
```c
uint32_t R38_value;
double R38_V;
void Get_R38_ADC_Value_proc(void)
{
	HAL_ADC_Start(&hadc1);
	R38_value = HAL_ADC_GetValue(&hadc1);
	R38_V = R38_value * 3.3 / 4095;
}

uint32_t DAC_Data = 0;
void DAC_Out_proc(void)
{
	DAC_Data = R37_value;
	HAL_DAC_Start(&hdac1,DAC_CHANNEL_1);
	HAL_DAC_SetValue(&hdac1, DAC_CHANNEL_1, DAC_ALIGN_12B_R, DAC_Data);
}
```
<empty-block/>
# 串口简单中断接收 proc 
开中断，**开微库**
fputc重定向
IT单字节循环接受 
<empty-block/>
while循环50ms任务：判断并清除buff中的数据
<empty-block/>
找到接收完成回调函数
`__weak void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)`
<empty-block/>
核心代码：（ 注意取址符号）
1. 重定向
	```c
int fputc(int ch, FILE *f)
{
	  HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 50); 
  return ch;
}
	```
2. 接收完成中断
	```c
uint8_t u1_rxbuff[30];
uint8_t u1_rxcount = 0;
extern uint8_t u1_rxdata;
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if(huart->Instance == USART1)
	{
		u1_rx_tick = uwTick;	//防止提前刷新
		u1_rxbuff[u1_rxcount++] = u1_rxdata;
		HAL_UART_Receive_IT(&huart1, &u1_rxdata, 1);
	}
}
	```
3. while循环检测
	```c
uint32_t u1_rx_tick = 0;
void U1_Rx_proc(void)
{
	if(uwTick - u1_rx_tick < 50)
		return;
	u1_rx_tick = uwTick;
	
	if(u1_rxbuff[0] == '#' && u1_rxcount == 1)
	{
		printf("成功反转屏幕\r\n");
	}
	
	
	u1_rxcount = 0;
	memset(u1_rxbuff, 0, sizeof(u1_rxbuff));
}
	```
4. while前使能
	```c
	
	uint8_t u1_rxdata;	  //全局变量哦
	
	main()
	{
		HAL_UART_Receive_IT(&huart1, &u1_rxdata, 1);
		while(1)
		{
			...
		}
	}
	```
<empty-block/>
# 串口利用定时器不定长数据接收 
比赛一般用的串口波特率为9600
换算为每1.15ms一个字节
也就是说，当1.5ms后没有接收到数据，就代表数据接收完毕，可以进行处理
<empty-block/>
定时器psc = 8000- 1  也就是CNT  10000 次每秒， 一秒如果超过15次则代表本次接收完成。
只需要在IT 接收的基础上通过标志位在每次接收时判断 CNT并清零就OK
<empty-block/>
(实际是有串口空闲中断的，不过稍麻烦)
核心代码
```c
#include "sys.h"


char text[20];

uint8_t uart_rxdata;

uint8_t uart_flag = 0;
uint8_t uart_r_count = 0;
uint8_t uart_r_buff[20];

char uart_sendbuff[20];

void show_lcd(void)
{
	sprintf(text, "Hello world");	//将字符串复制到字符数组
	LCD_DisplayStringLine(Line0, (uint8_t*)text);
}


//接收完成回调
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if(huart->Instance == USART1)
	{
		TIM4->CNT = 0;
		uart_flag = 1;
		uart_r_buff[uart_r_count] = uart_rxdata;	//把接收到的数据存入
		uart_r_count++;
		
		HAL_UART_Receive_IT(&huart1, &uart_rxdata, 1);
	}
}


void uart_rx_fun(void)
{
    if(uart_flag)
    {
        if(TIM4->CNT > 15)
        {
		       /*------------逻辑处理部分------------*/
            // 长度判断 && 字符串比较
            if(uart_r_count == 3 && memcmp(uart_r_buff, "lan", 3) == 0)
            {
                sprintf(uart_sendbuff, "lan\r\n");
                HAL_UART_Transmit(&huart1, (uint8_t *)uart_sendbuff, strlen(uart_sendbuff), 50);
            }
						/*------------逻辑处理部分------------*/
						
            // 清除缓冲区（修复循环条件）
            uint8_t temp_count = uart_r_count;
            uart_r_count = 0;
            uart_flag = 0;
            memset(uart_r_buff, 0, temp_count); // 仅清除实际接收的部分
        }
    }
}
```
# 验证日期合法性  以及时间刻计算日期间隔秒数的方法 
```c
int is_leap(int year) {
    return (year % 4 == 0 && year % 100 != 0) || (year % 400 == 0);
}

// 验证日期合法性
int is_valid_date(int year, int month, int day) {
    if (month < 1 || month > 12) return 0;
    int days[] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
    if (is_leap(year)) days[2] = 29; // 闰年2月天数
    return day >= 1 && day <= days[month];
}
// 判断时间（HHmm）是否合法
int is_valid_time(uint8_t hh, uint8_t mm, uint8_t ss) {
    // 拆分小时和分钟
    // 验证范围：小时 0-23，分钟 0-59
    return (hh >= 0 && hh <= 23) && (mm >= 0 && mm <= 59) && (ss >= 0 && ss <= 59);
}

//计算日期
struct tm             //一个结构体
time_t current_time;  //存储时间戳


localtime()
功能：将时间戳转换为本地时间的 struct tm 结构体。
实例：
		time_t current_time;
    struct tm *local = localtime(&current_time);
    printf("当前本地时间: %d-%02d-%02d %02d:%02d:%02d\n",
           local->tm_year + 1900, local->tm_mon + 1, local->tm_mday,
           local->tm_hour, local->tm_min, local->tm_sec);

mktime()
功能：将 struct tm 结构体表示的时间转换为时间戳。
实例：
		time_t timestamp = mktime(&tm_info);
    printf("转换后的时间戳: %ld\n", timestamp);
```
<empty-block/>
# \*\*  输入捕获+复位模式测 输入频率+占空比 
以R39的555定时器举例
1. 配置定时器
	<empty-block/>
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/725c7d33-69a8-45a2-968c-c5ec9bdf74fc.webp)
	**勾选NVIC中断**
	**PCS = 80-1**
	**ARR 最大**
	**直接捕获捕获上升沿， 间接捕获捕获下降沿**
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/a26c09fc-488b-4a52-92c7-4eacdcf01f6e.webp)
<empty-block/>
main函数启动：
```c
HAL_TIM_IC_Start_IT(&htim3, TIM_CHANNEL_1);
	HAL_TIM_IC_Start_IT(&htim3, TIM_CHANNEL_2);
```
**void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef \*htim)**
**HAL_TIM_ACTIVE_CHANNEL_1**
```c
//输入捕获回调函数
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM3)
	{
		if(htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1)    
		{
			R39_t_count = TIM3->CCR1;	//记录一个周期记录的数量
		}
		if(htim->Channel == HAL_TIM_ACTIVE_CHANNEL_2)
		{
			R39_up_count = TIM3->CCR2;	//记录高电平 记录的数量
		}
	}
}
```
频率和周期计算
float F = 1000000.0f/R39_t_count ;<br>float D = R39_up_count /R39_t_count \* 100;
# \*\* ADC + DMA 
芯片内部温度计算：（1.43-V）/0.0043+25.0
1. 激活ADC对应通道
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/786be940-2239-4fa8-94d6-3c8f75eeeb2f.webp)
2. 打开DMA（注意一下，这里需要改成环形缓冲区，内存步长可以不用改为Word，在创建数组时注意一致就OK）
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/3572e2e4-9e2b-4ed5-bec4-5aa08e24ddd3.webp)
3. 如果同ADC 多个通道，则在下方输入转换通道的个数，扫描模式会自动打开，并且在main函数中定义数组，而不是单个变量。
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/00b654c4-bbdb-4d8d-aaf1-fa0e131d0bc6.webp)
4. 采样周期最好** 最高调为247.5 **，再大貌似会出问题
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/48386a4c-f94b-4e00-9484-4a2956940a3d.webp)
启动代码  **注意是65535 **不是4095
<empty-block/>
	```c
	uint32_t ADC_Value;
	
	
	
	HAL_ADCEx_Calibration_Start(&hadc1,ADC_SINGLE_ENDED);
	HAL_ADC_Start_DMA(&hadc1, &ADC_Value, 1);
	
	V = 3.3/65535*ADC_Value;
	```
<empty-block/>
# \*\* DAC + DMA 
DAC输出三种波形
<empty-block/>
使用定时器的溢出事件 控制频率。
使用填充数据的大小，控制幅度 0-3.3 → 0-4095
<empty-block/>
1. 配置定时器
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/a7f76cee-c32a-4098-a95d-d354d228272f.webp)
2. 配置DAC
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/8b2b329c-c111-4198-92e8-ab0f12789a5f.webp)
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/2e081ec1-3131-4ec1-bd36-7a92ad5b8191.webp)
<empty-block/>
波形buff生成函数
```c

uint16_t zhengxian_buff[100];
uint16_t fang_buff[100];
uint16_t sanjiao_buff[100];

#include "math.h"
#define PI 3.1415926

//填充波形
void zhengxian_tianchong(void)
{
	uint8_t i;
	for(i = 0; i < 100; i++)
	{					// 把 -1 ~ 1 映射为 0 ~ 2 然后 再恢复为 0 - 1 然后换算为DAC比值
		zhengxian_buff[i] = (sin(2 * PI / 100 * i) + 1) / 2 * 4095 ;
	}
}
void fang_tianchong(uint8_t duty)
{
	if(duty > 100) duty = 100;
	
	//填充高电平
	uint8_t i;
	for(i = 0; i < duty; i++)
	{
		fang_buff[i] = 0xffff;
	}
	//填充低电平
	for(i = duty; i < 100; i++)
	{
		fang_buff[i] = 0x0000;
	}
}
//对齐度
void sanjiao_tianchong(uint8_t duty)
{
	uint8_t i;
	//填充高电平
	for(i = 0; i < duty; i++)
	{				//将4095 分为duty份， 递增
		sanjiao_buff[i] = (4095 / duty) * i;
	}
	//填充低电平
	for(i = duty; i < 100; i++)
	{				//将4095 分为100 - duty份， 递减
		sanjiao_buff[i] = (100 - i) * (4095 / (100 - duty));
	}
}
```
在main中启用的函数
```c
 //启动定时器
 HAL_TIM_Base_Start(&htim15);
//通道1输出正弦波
	HAL_DAC_Start_DMA(&hdac1, DAC_CHANNEL_1, (uint32_t*)fang_buff, 100, DAC_ALIGN_12B_R);
	//通道2输出三角波
	HAL_DAC_Start_DMA(&hdac1, DAC_CHANNEL_2, (uint32_t*)sanjiao_buff, 100, DAC_ALIGN_12B_R);
```
**修改DAC频率**
TIM15-\>ARR = ( 80000000 / (TIM15-\>PSC + 1) / 目标频率 - 1 ) ;
得到DAC通道1当前电压
DAC_A = 3.3 \* (DAC1-\>DOR1 / 4095.0)
<empty-block/>
# \*\* eeprom 连续 读写 
芯片型号 M24C02
eeprom，非易失性存储器(掉电不丢失）
<empty-block/>
主机和从从机发消息，都需要回应。
<empty-block/>
<empty-block/>
<empty-block/>
设备地址
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/c0e80f83-eb8d-47a2-bcca-3232bdafcb7c.webp)
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/d5174460-c93e-4d19-b919-029815b7d124.webp)
E1对应A0
E2对应A1
E3对应A2
可见，全为0.
R/W位，为1，是R读，
R/W位，为0，是W写；
<empty-block/>
地址：0xa0
<empty-block/>
读 0xa1
<empty-block/>
**底层程序为官方提供**
main.c
```c
struct __eeprom_Test{
	uint8_t a;
	uint16_t b;
	float c;
}test = {222, 33333, 4.44};

			eeprom_w(0, (uint8_t*)&test, sizeof(test));
```
核心读写代码：
**fun.c**
```c
void eeprom_w(uint8_t addr, uint8_t *pData, uint8_t len)
{
	uint8_t page = addr % 8;  // 当前页剩余空间
    uint8_t writeLen;
    
    while(len > 0) {
        writeLen = (page + len > 8) ? (8 - page) : len;  // 计算本次写入长度
        
        I2CStart();
        I2CSendByte(0xA0);  // 写模式
        I2CWaitAck();
        I2CSendByte(addr);
        I2CWaitAck();
        
        for(uint8_t i=0; i<writeLen; i++) {
            I2CSendByte(*pData++);
            I2CWaitAck();
        }
        
        I2CStop();
        HAL_Delay(5);  // 关键延时
        
		//写完当前页，计算剩余页数
        addr += writeLen;
        len -= writeLen;
        page = 0;  // 新页剩余空间重置
    }
}

void eeprom_r(uint8_t addr, uint8_t *data, uint8_t n)
{
	I2CStart();      
	I2CSendByte(0xa0);
	I2CWaitAck();
	I2CSendByte(addr);
	I2CWaitAck();
	I2CStop();			
	
	I2CStart();			
	I2CSendByte(0xa1);
	I2CWaitAck();
	
	while(n--)
	{
		*data++ = I2CReceiveByte();
		if(n == 0)	I2CSendNotAck();
		else		I2CSendAck();
	}
	I2CStop();
}
```
使用方法
`eeprom_w(0, (uint8_t*)&test, sizeof(test));`
<empty-block/>
# \*\* MCP4017 
MCP4017 的IIC地址：0101111   （0x5e）
100K  分为127份  0 为 0Ω ， 127 为10K
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/8c40a22f-9c44-4916-a58c-69df94c31e4d.webp)
最后一位为R/W  读为1 写为0
一般只用写， 也就是0101 1110   =  0x5e
100K  分为127份  0 为 0Ω ， 127 为10K
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/1616c9c3-6eec-4da9-a57d-f38b156771a0.webp)
```c
void mcp4017_w(uint8_t data)
{
	I2CStart();
	I2CSendByte(0x5e);
	I2CWaitAck();
	
	I2CSendByte(data);
	I2CWaitAck();
	I2CStop();
}

uint8_t mcp4017_r()
{
	uint8_t data;
	I2CStart();
	I2CSendByte(0x5f);
	I2CWaitAck();
	data = I2CReceiveByte();
	I2CSendNotAck();
	I2CStop();
	return data;
}
```
<empty-block/>
# \*\* RTC实时时钟 
主要功能实现：
1. 设置时间和日期
2. 读取时间和日期
3. 设置一个闹钟
<empty-block/>
<empty-block/>
cube找到 rtc  勾选两个勾（使能时钟源和日历)
选择闹钟A，  勾选NVIC中断
选择24h计数法
数据格式选择二进制计数法，**Binary dada format**
配置时钟的 时分秒。
配置日历，星期 ，月份等等等
设置闹钟
<empty-block/>
设置掩盖工作日、小时等等。（具体看是每天、每时，响起等需求）
生成代码—-
<empty-block/>
简单的看这个：
```c
RTC_DateTypeDef rtc_date;
RTC_TimeTypeDef rtc_time;
uint32_t rtc_tick = 0;
void RTC_proc(void)
{
	if(uwTick - rtc_tick < 100)
		return;
	rtc_tick = uwTick;
	
	HAL_RTC_GetTime(&hrtc, &rtc_time, RTC_FORMAT_BIN);
	HAL_RTC_GetDate(&hrtc, &rtc_date, RTC_FORMAT_BIN);
}
```
<empty-block/>
__**注意，即使不需要日期也要把获取日期的函数写上去，不然时间不会流动**
<empty-block/>
修改闹钟时间
```c
		RTC_AlarmTypeDef Temp;

		HAL_RTC_GetAlarm(&hrtc, &Temp, RTC_ALARM_A, RTC_FORMAT_BIN);
		Temp.AlarmTime.Seconds += 5;
		
		HAL_RTC_SetAlarm_IT(&hrtc, &Temp, RTC_FORMAT_BIN);
```
<empty-block/>
# \*\*  定时器+状态机 任意按键长按短按双击 组合 
1. 启用定时器，
	设置10ms一次中断，PSC = 800-1  ARR = 1000-1
	开启中断
代码：
Key.h
```c
#ifndef __KEY_H
#define __KEY_H

#include "sys.h"

struct keys{
	uint32_t up_time;		//抬起累计时间
	uint32_t down_time;		//按下累计时间
	uint8_t  key_short;		//短按触发
	uint8_t  key_double;	//双击触发
	uint8_t  key_long;		//长按触发
	uint8_t  key_swt;		//状态机
	uint8_t  key_state;		//当前按键状态
};

extern struct keys key[];

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim);	//回调函数


#endif

```
Key.c
```c
#include "key.h"

struct keys key[4] = {0};

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
	if(htim->Instance == TIM17)
	{
		// 获取当前状态 0 为按下
		key[0].key_state = HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_0);
		key[1].key_state = HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_1);
		key[2].key_state = HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_2);
		key[3].key_state = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_0);
		// 处理四个按键状态机
		uint8_t i = 0;
		for(i = 0; i < 4; i++)
		{
			switch(key[i].key_swt)
			{
				case 0:		//初始状态
				{
					if(key[i].key_state == 0)	//检测到按下
					{
						key[i].down_time = 0;	//清除按下时间
						key[i].up_time = 0;		//清除抬起时间
						key[i].key_swt = 1;		//进入确定个状态
					}
				}
				break;
				case 1:		//确定状态
				{
					if(key[i].key_state == 0)	//(10ms后)仍然为按下
						key[i].key_swt = 2;		//进入逻辑判断状态
					else
						key[i].key_swt = 0;
				}
				break;
				case 2:
				{
					if(key[i].key_state == 0)	//如果当前按下
					{
						key[i].down_time++;		//累计按下时间
						if(key[i].down_time > 100)	//按下超过1s
							key[i].key_long = 1;	//触发长按
						if(key[i].up_time < 20 && key[i].up_time > 0)	//抬起时间在0-200ms按下
							key[i].key_swt = 3;		//进入双击确认状态 
					}
					if(key[i].key_state == 1)	//如果在状态2抬起
					{
						key[i].up_time++;		//记录抬起时间
						if(key[i].up_time > 20 && key[i].up_time < 100){
							key[i].key_short = 1;	//如果大于200且小于1000ms 则为短按
							key[i].key_swt = 0; 	//回到初始状态
						}
						if(key[i].down_time > 100)	//如果按下时间大于1s
							key[i].key_swt = 0;		//回到初始状态（重要，必须在抬起时回到初始状态，否则出错）
					}
				}
				break;
				case 3:	// 双击确认状态
				{
					if(key[i].key_state == 0)	//确定按下
						key[i].key_swt = 4;		//进入双击状态
					else
						key[i].key_swt = 2;
				}
				break;
				case 4:
				{
					if(key[i].key_state == 1)	//如果当前抬起（重要，必须 抬起 才算）
					{
						key[i].key_double = 1;	//触发双击
						key[i].key_swt = 0;		//回到初始状态（重要，必须在抬起时回到初始状态，否则出错）
					}
				}
			}
		}
	}
}



```
<empty-block/>
<empty-block/>
常见使用逻辑
```c
	// 首先处理所有按键的独立短按事件，这些事件优先级最高
    // MCP4017 ++
    if(key[0].key_short && !key[1].key_short) {
        //----执行操作-----
        key[0].key_short = 0;
    }
    // MCP4017 --
    if(key[1].key_short && !key[0].key_short) {
         //----执行操作-----
        key[1].key_short = 0;
    }

    // 处理组合按键事件
    // 同时处于抬起状态
    if(key[0].key_state && key[1].key_state) {
        // 按键1和2同时双击触发
        if(key[0].key_double && key[1].key_double) {
             //----执行操作-----
            // 清除状态
            key[0].key_double = key[1].key_double = 0;
            key[0].key_short = key[1].key_short = 0; // 确保清除短按状态
        }
        // 按键1和2同时短按触发
        else if(key[0].key_short && key[1].key_short) {
             //----执行操作-----
            // 清除状态
            key[0].key_short = key[1].key_short = 0;
        }
    }

    // 两个按键长按 一直触发
    if(key[0].key_long && key[1].key_long) {
        //----执行操作-----
        key[0].key_long = key[1].key_long = 0;
    }

    // 两个按键长按，只触发一次
    static uint8_t key23_Flag = 1;
    if(key[2].key_long && key[3].key_long) {
        if(key23_Flag) {
            key23_Flag = 0;
             //----执行操作-----
             key[2].key_long = key[3].key_long = 0;
        }
    } else if(key[2].key_state && key[3].key_state) {
        // 当两个按键都释放时重置标志
        key23_Flag = 1;
    }
	
	// EEPROM W
	if(key[2].key_short)
	{
		 //----执行操作-----
		key[2].key_short = 0;
	}
	// EEPROM R
	if(key[3].key_short)
	{
		 //----执行操作-----
		key[3].key_short = 0;
	}
	
	if(key[3].key_double)
	{
		 //----执行操作-----
		key[3].key_double = 0;
	}
```
<empty-block/>
<empty-block/>
# \*\* 反转屏幕的方法（可能会用到） 
在lcd.c中找下面的寄存器
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/942d228a-ee90-4111-8418-c95c4c0ddb83.webp)
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/dcc78a58-8197-4a83-9ee0-23f891331ba1.webp)
改成 右侧备注表明的  替换值即可
这里面就这俩，还是很好找的
<empty-block/>
核心代码
```c
uint32_t u1_rx_tick = 0;
void U1_Rx_proc(void)
{
	if(uwTick - u1_rx_tick < 50)
		return;
	u1_rx_tick = uwTick;
	
	if(u1_rxbuff[0] == '#' && u1_rxcount == 1)
	{
		printf("成功反转屏幕\r\n");
		static uint8_t lcd_mode = 0;
		if(lcd_mode != 0){
			lcd_mode = 0;
			LCD_Clear(Black);
			LCD_WriteReg(R1, 0x0000);   // set SS and SM bit		  //0x0100
			LCD_WriteReg(R96, 0x2700);  // Gate Scan Line		  0xA700
		}else{
			lcd_mode = 1;
			LCD_Clear(Black);
			LCD_WriteReg(R96, 0xA700);  //将替换值代入后~
			LCD_WriteReg(R1, 0x0100); 
		}
	}
	else if(u1_rxcount > 0)
	{
		printf("%s\r\n", u1_rxbuff);
		printf("输入#反转屏幕\r\n");
	}
	
	u1_rxcount = 0;
	memset(u1_rxbuff, 0, sizeof(u1_rxbuff));
}
```
<empty-block/>
# \*\* UART+ DMA+Idle 
1. 设置DMA
	uart只需要改个波特率
	**使能UART中断**
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/86c8943c-70a9-4e47-a7ea-a40b7ff3c02d.webp)
main.c启用
`HAL_UARTEx_ReceiveToIdle_DMA(&huart1, rx_buff, 100);`
回调函数 及 数据分析方法
```c

uint8_t Rx_Size = 0;
extern uint8_t rx_buff[100];
void HAL_UARTEx_RxEventCallback(UART_HandleTypeDef *huart, uint16_t Size)
{
	if(huart->Instance == USART1)
	{	
		Rx_Size = Size;
		uart_rx_flag = 1;
		HAL_UARTEx_ReceiveToIdle_DMA(&huart1, rx_buff, 100);
	}
}

void rx_data_chuli(void)
{
	if(uart_rx_flag)
	{
		if(rx_buff[0] == '#')
		{
			static uint8_t lcd_mode = 0;
			if(lcd_mode != 0){
				lcd_mode = 0;
				LCD_Clear(Black);
				LCD_WriteReg(R1, 0x0000);   // set SS and SM bit		  //0x0100
				LCD_WriteReg(R96, 0x2700);  // Gate Scan Line		  0xA700
			}else{
				lcd_mode = 1;
				LCD_Clear(Black);
				LCD_WriteReg(R96, 0xA700);  //将替换值代入后~
				LCD_WriteReg(R1, 0x0100); 
			}
		}
		else
		{
			HAL_UART_Transmit(&huart1, rx_buff, Rx_Size, 20);
		}
		
		Rx_Size = 0;
		uart_rx_flag = 0;
		memset(rx_buff, 0, 100);
	}
}

```
# \*\* 任务调度器的方法 
```c
#include "scheduler.h"

//任务数量
uint8_t task_num = 0;

//任务结构体
typedef struct{
	void (*taskRun)(void);   //函数指针
	uint32_t tick;           //任务运行间隔
	uint32_t last;           //上一次运行的时间（开机后初次运行的时间）
}task_t;

//任务数组
task_t Tasks[]={
	{key_proc, 20, 0},
	{lcd_proc, 50, 0},
	{uart_proc, 10, 0},
	{led_proc, 100, 0},
	{rtc_prco, 50, 0}
};

//调度器初始化
void scheduler_Init(void)
{
	task_num = sizeof(Tasks) / sizeof(task_t);
}

//调度器运行
void scheduler_Run(void)
{
	uint8_t i = 0;
	for(i = 0; i < task_num; i++)
	{
		if(uwTick >= Tasks[i].last + Tasks[i].tick)
		{
			Tasks[i].last = uwTick;
			Tasks[i].taskRun();
		}
	}
}
```
# 客观题训练
1. 线与门电路是：OC门
2. G4内核Cortex-M4
3. 不支持数据类型为双字
4. RS232至少需要三根线（Tx、Rx、GND）
5. 较少线的下载方式：SWJ（SW、JTAG可以调试，JTAG需要10、14、20Pin）
6. 菊花链：SPI
7. SUB外设：HSE时钟
8. DMA可以和MCU并行工作、不需要i经过MCU访问内存
9. 二极管温度升高时，反向饱和电流增大
10. ADC主要技术指标包括：量化误差、转换速率、分辨率。（不包含频率）
11. G431是3级流水线（取指解码和执行）
12. 8421BCD计数器需要至少4个触发器
13. 共集电极、发射机和基极的特点
	<table header-column="true">
<tr>
<td>**参数**</td>
<td>**共集电极（CC）**</td>
<td>**共发射极（CE）**</td>
<td>**共基极（CB）**</td>
</tr>
<tr>
<td>**电压增益**</td>
<td>**≈1（无放大）**</td>
<td>高（几十～几百）</td>
<td>高</td>
</tr>
<tr>
<td>**电流增益**</td>
<td>高（β+1）</td>
<td>高（β）</td>
<td>低（≈1）</td>
</tr>
<tr>
<td>**输入阻抗**</td>
<td>**高**</td>
<td>中</td>
<td>低</td>
</tr>
<tr>
<td>**输出阻抗**</td>
<td>**低**</td>
<td>高</td>
<td>高</td>
</tr>
<tr>
<td>**相位关系**</td>
<td>同相</td>
<td>**反相**</td>
<td>同相</td>
</tr>
<tr>
<td>**带宽**</td>
<td>窄</td>
<td>中</td>
<td>宽</td>
</tr>
<tr>
<td>**典型应用**</td>
<td>**缓冲器、阻抗匹配**</td>
<td>通用放大器</td>
<td>高频放大</td>
</tr>
	</table>
14. $`存储空间 = 地址线线数量^2 * (数据线/8)`$
15. stm32G431芯片有111个中断，包括9个内核中断和**102个可屏蔽中断**，具有16级可编程的中断优先级（NVIC管理的优先级分五组，两个层次）
16. ADC转换过程：采样量化编码
17. G431-\>GPIO 速度配置对应关系
	<table>
<tr>
<td>**配置值（二进制）**</td>
<td>**速度模式**</td>
<td>**典型输出速度**</td>
</tr>
<tr>
<td>00</td>
<td>**低速（Low）**</td>
<td>≤ 2 MHz</td>
</tr>
<tr>
<td>01</td>
<td>**中速（Medium）**</td>
<td>≤ 10 MHz</td>
</tr>
<tr>
<td>10</td>
<td>**高速（High）**</td>
<td>≤ 50 MHz</td>
</tr>
<tr>
<td>11</td>
<td>**超高速（Very High）**</td>
<td>≤ 100 MHz</td>
</tr>
	</table>
18. G431 仅支持小端模式即， 数据的低字节存储在低地址，高字节存储在高地址。
19. DMA每个通道有3个事件：传输完成、半完成、错误
20. 三角波转换为矩形波：用施密特触发器
21. PCM = ICM x UCEO
22. 此运放电路直接带入公式算：Uo = -Rf \* (U1/R1 + U2/R2)
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/412b6eab-9ff3-4889-b865-fff8767fabed.webp)
23. 晶体管制造工艺最简单
24. G431RBT6不待LCD控制器与FSMC外设
25. 总线是信号线的集合：数据总线、控制总线、地址总线
26. 全双工、半双工、单工的区别
	<table>
<tr>
<td>**类型**</td>
<td>**数据传输方向**</td>
<td>**能否同时双向通信**</td>
<td>**典型应用场景**</td>
</tr>
<tr>
<td>单工</td>
<td>仅能单向传输</td>
<td>❌</td>
<td>广播、电视信号、遥控器、摄像头输出视频</td>
</tr>
<tr>
<td>半双工</td>
<td>可双向传输，但某一时刻仅能单向</td>
<td>❌</td>
<td>对讲机、默认无流控的 UART 串口、早期以太网</td>
</tr>
<tr>
<td>全双工</td>
<td>可双向且能同时传输</td>
<td>✅</td>
<td>电话、网络通信（TCP/IP ）、USB、SPI（多数实现 ）</td>
</tr>
	</table>
27. 电容器的主要参数：标称容量、允许偏差、额定工作电压、绝缘电阻、温度系数、频率特性等
28. EXTI
	<table>
	<colgroup>
	<col>
	<col width="282.99998474121094">
	<col>
	</colgroup>
<tr>
<td>EXTI16</td>
<td>**PVD**（可编程电压监测器）输出</td>
<td>用于电压异常等监测中断</td>
</tr>
<tr>
<td>EXTI17</td>
<td>**RTC（**实时时钟）闹钟事件</td>
<td>响应 RTC 闹钟触发中断</td>
</tr>
<tr>
<td>EXTI18</td>
<td>**USB **OTG FS 唤醒事件</td>
<td>USB 高速设备唤醒系统</td>
</tr>
<tr>
<td>EXTI19</td>
<td>**以太网**唤醒事件</td>
<td>以太网相关唤醒场景用</td>
</tr>
<tr>
<td>EXTI20</td>
<td>USB OTG HS（FS 配置下）唤醒事件</td>
<td>USB 高速相关唤醒</td>
</tr>
<tr>
<td>EXTI21</td>
<td>RTC 入侵和时间戳事件</td>
<td>RTC 入侵检测、记录事件时间戳</td>
</tr>
<tr>
<td>EXTI22</td>
<td>RTC 唤醒事件</td>
<td>RTC 定时唤醒系统</td>
</tr>
	</table>
29. 积分电路：电阻平，电容竖；微分相反
	- 积分电路**可以用来将矩形波变成三角波输出**
	- 微分电路**可以把矩形脉冲变换为尖脉冲输出**
30. 功能简单但频繁调用：内敛函数
	- 内联函数（Inline Function）是一种编译器优化技术，通过在调用点直接展开函数体代码来减少函数调用开销。
	- `inline` 关键字声明的函数，编译器会尝试将函数体直接嵌入到调用处，而非通过常规的函数调用机制（如栈帧创建、参数压栈、跳转等））
31. 模数转换器的**分辨率**可以通过**输出二进制数字信号的位数**来判断。
32. stm32g431xx.h定义了各类外设的寄存器结构体和相关位定义
33. **时序 **数字电路的输出与电路的**原状态和当前输入有关**
34. 实现A/D的方法有:计数法、双积分法、逐次逼近法
35. stm32的片内FLASH，一次可以写入16bit的数据
36. USB和UART没有同步时钟（CLK）
37. 二极管的伏安特性曲线（正向部分）在**温度下降时右移**
38. 8个触发器能保存2\^8次方状态
39. TTL电路悬空则 = 高电平
40. 贴片电阻，4位表示则精度1%，3位精度百分之5.<br>例如1002 = 10K 精度1%， 103表示，10K精度百分之5%
41. 定时器的分类和编号
	<table header-row="true">
<tr>
<td>**高级定时器**</td>
<td>**基本定时器**</td>
<td>通用定时器</td>
<td>**特殊定时器**</td>
</tr>
<tr>
<td>**TIM1、TIM8**</td>
<td>**TIM6、TIM7**</td>
<td>其他都是通用</td>
<td>低功耗定时器（LPTIM）</td>
</tr>
	</table>
42. I2C起始条件： **SCL 保持高电平期间**，**SDA 从高电平跳变到低电平**
43. I2C停止条件：**SCL 保持高电平期间**，**SDA 从低电平跳变到高电平**
44. 555定时器频率和占空比计算公式
	<columns>
		<column>
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/4c9a8cfb-81f8-49b9-974a-3b7ad0daabb1.webp)
		</column>
		<column>
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/c1802c69-c6ac-4dbe-9294-e8a183e4524d.webp)
			$`D = \frac{t_{\text{充}}}{T} = \frac{R_{A}}{R_{A} + R_{B}}`$
			$`f = \frac{1}{T} \approx \frac{1.43}{(R_{A} + R_{B})C}`$
		</column>
	</columns>
45. 可以作为注释中MCO输出的时钟源是：SYSCLK、HSI、HSE、PLL/2、
46. USB:串行通信、 支持热插拔、 速度比RS232快、 级联星型拓扑结构（分为主机、集线器和设备）
47. STM32内核级外设有 `NVIC `和 `SysTick`
48. 睡眠、停止和待机状态特点
	<table>
	<colgroup>
	<col width="81.65718078613281">
	<col width="226.666015625">
	<col width="369.6571807861328">
	</colgroup>
<tr>
<td>**模式**</td>
<td>**区别（功耗、状态、特性 ）**</td>
<td>**唤醒方法**</td>
</tr>
<tr>
<td>睡眠模式</td>
<td>内核停，<br>外设（时钟、定时器等）运行；寄存器、RAM 数据保留；功耗较低，唤醒最快</td>
<td>任意中断（如外部中断、定时器中断等 ）</td>
</tr>
<tr>
<td>停止模式</td>
<td>内核、外设时钟关，<br>保留 1.8V 供电；寄存器、RAM 数据保留；功耗比睡眠低，唤醒稍慢</td>
<td>外部中断（如按键触发 EXTI ）、RTC 闹钟中断等</td>
</tr>
<tr>
<td>待机模式</td>
<td>关闭所有时钟、内核 1.8V 供电；<br>仅备份寄存器数据保留，其他丢失；功耗最低，唤醒最慢</td>
<td>唤醒引脚（如 PA0 ）上升沿、RTC 闹钟中断、复位（外部复位、独立看门狗复位等 ）</td>
</tr>
	</table>
49. 一个DMA请求至少占用2个周期的CPU访问系统总线时间。
50. STM32 APB2 IO的最大反转速度为18MHZ
51. 两个相同的电压放大器，对同一个信号源进行放大，A电路输出电压小，则A电路输入电阻小
52. DMA外设是逻辑或请求， 同一时刻只有一个有效。
53. DMA的传输最大数目是65535
54. 要提升电压比较器ide抗干扰能力：**滞回比较器**
55. Cortex-M3系列处理器 **支持Thumb-2 指令集 不支持Thumb指令集，<br>Cortex-M4 在 Thumb-2 基础上增加了DSP 指令集、FPU 指令集。**
56. 启动模式说明
	<table>
	<colgroup>
	<col width="144.9905242919922">
	<col width="68.99052429199219">
	<col width="169.99951171875">
	<col width="218.98672485351562">
	</colgroup>
<tr>
<td>**启动模式选择引脚**</td>
<td></td>
<td>**启动模式**</td>
<td></td>
</tr>
<tr>
<td>BOOT1</td>
<td>BOOT0</td>
<td></td>
<td></td>
</tr>
<tr>
<td>X</td>
<td>0</td>
<td>主闪存存储器</td>
<td>主闪存存储器被选为启动区域</td>
</tr>
<tr>
<td>0</td>
<td>1</td>
<td>系统存储器</td>
<td>系统存储器被选为启动区域</td>
</tr>
<tr>
<td>1</td>
<td>1</td>
<td>内置 SRAM</td>
<td>内置 SRAM 被选为启动区域</td>
</tr>
	</table>
57. RLC串联电路，谐振频率F0=1000HZ，当频率为800HZ时，正弦电压源激励时该电路呈容性
	<columns>
		<column ratio="68.75">
			**RLC 并联电路**
			- 当$`F > f_0`$时，电路容性；
			- 当$`F = f_0`$时，电路阻性；
			- 当$`F < f_0`$时，电路感性。
		</column>
		<column>
			**RLC 串联电路**
			- 当$`F > f_0`$时，电路感性；
			- 当 $`F = f_0`$时，电路阻性；
			- 当$`F < f_0`$时，电路容性。
		</column>
	</columns>
58. 5个D触发器构成的<span underline="true">**环形计数器**</span>，计数长度为<span underline="true">5</span>
59. CPU寄存器
	<table>
	<colgroup>
	<col>
	<col>
	<col width="315.99998474121094">
	</colgroup>
<tr>
<td>**名称**</td>
<td>**寄存器**</td>
<td>**介绍**</td>
</tr>
<tr>
<td>**通用寄存器（R0-R12）**</td>
<td>R0-R12（32 位）</td>
<td>临时存储数据、参与运算及传递参数，R0-R7 适配 16 位指令，R8-R12 适配 32 位指令。</td>
</tr>
<tr>
<td>**堆栈指针（SP）**</td>
<td><span color="blue">**R13**</span></td>
<td>**含主栈指针 MSP（内核 / 中断）和进程栈指针 PSP（用户任务），确保栈隔离与对齐。**</td>
</tr>
<tr>
<td>**链接寄存器（LR）**</td>
<td><span color="blue">**R14**</span></td>
<td>**存储函数 / 子程序返回地址**，中断时赋值为 EXC_RETURN，嵌套调用需手动压栈。</td>
</tr>
<tr>
<td>**程序计数器（PC）**</td>
<td><span color="blue">**R15**</span></td>
<td>**指向当前 / 下一条指令地址**，读操作返回 “当前地址 + 4”，写操作实现跳转。</td>
</tr>
<tr>
<td>**应用程序状态寄存器**</td>
<td>APSR</td>
<td>记录 ALU 运算标志（Z/C/N/V），用于条件判断指令。</td>
</tr>
<tr>
<td>**执行程序状态寄存器**</td>
<td>EPSR</td>
<td>记录指令执行状态（如 Thumb 状态位 T），确保指令按模式执行。</td>
</tr>
<tr>
<td>**中断程序状态寄存器**</td>
<td>IPSR</td>
<td>只读，记录当前服务的中断号，用于识别中断源。</td>
</tr>
<tr>
<td>**PRIMASK**</td>
<td>PRIMASK</td>
<td>置 1 时屏蔽除 NMI 和 HardFault 外的所有中断，适用于高实时性任务。</td>
</tr>
<tr>
<td>**FAULTMASK**</td>
<td>FAULTMASK</td>
<td>置 1 时屏蔽所有异常 / 中断（含 HardFault），中断返回后自动清零。</td>
</tr>
<tr>
<td>**BASEPRI**</td>
<td>BASEPRI</td>
<td>按优先级屏蔽等于 / 低于设定值的中断，灵活控制中断响应范围。</td>
</tr>
<tr>
<td>**控制寄存器（CONTROL）**</td>
<td>CONTROL</td>
<td>模式控制（SPSEL 选 MSP/PSP、权限位区分特权级 / 用户级）；M4 中关联 FPU 功能。</td>
</tr>
	</table>
60. **两个逻辑函数恒等**，他们必然有唯一的**真值表**
61. 差分信号：要求等间距等长、差分信号。（RS232不是差分信号，差分信号不用共地）
62. NAND Flash 和 NOR Flash的区别（总的来说是Nor更贵更好，但更小）
	<table>
	<colgroup>
	<col width="137.3314208984375">
	<col width="235.33709716796875">
	<col width="290.3314208984375">
	</colgroup>
<tr>
<td>**对比维度**</td>
<td>**NAND Flash**</td>
<td>**NOR Flash**</td>
</tr>
<tr>
<td>**存储结构**</td>
<td>串行页 / 块结构，高密度存储</td>
<td>并行单元结构，低密度存储</td>
</tr>
<tr>
<td>**读写特性**</td>
<td>顺序读快、随机读慢，擦除速度快，块级擦写</td>
<td>随机读写快，支持字节级操作，擦除速度较慢，擦除单元小</td>
</tr>
<tr>
<td>**容量与成本**</td>
<td>容量大（1Gb+），单位成本低</td>
<td>容量小（2Gb 内），单位成本高</td>
</tr>
<tr>
<td>**应用场景**</td>
<td>数据存储（SSD/U 盘 / SD 卡）、大容量存储</td>
<td>程序存储（BIOS / 固件）、XIP 直接执行</td>
</tr>
<tr>
<td>**管理复杂度**</td>
<td>需要垃圾回收、磨损均衡等复杂机制</td>
<td>接口简单，无需额外管理机制</td>
</tr>
<tr>
<td>**执行方式**</td>
<td>需加载到 RAM 运行</td>
<td>支持片上执行（XIP）</td>
</tr>
<tr>
<td>**典型功耗**</td>
<td>擦写功耗低，待机功耗高</td>
<td>随机读功耗低，待机功耗低</td>
</tr>
	</table>
63. 具有压电效应的滤波器有：石英晶体滤波器， 声表面波滤波器
64. Cortex-M3、M4处理器均支持**2个堆栈**,别为**主堆栈（Main Stack, MSP）** 和**进程堆栈（Process Stack, PSP）**。
	<table>
	<colgroup>
	<col>
	<col>
	<col width="301.99998474121094">
	</colgroup>
<tr>
<td>**堆栈类型**</td>
<td>**主堆栈（MSP）**</td>
<td>**进程堆栈（PSP）**</td>
</tr>
<tr>
<td>**默认使用场景**</td>
<td>- 系统启动时的默认堆栈。- 处理异常（中断、Fault 等）时自动切换使用。- 操作系统内核、关键任务或特权模式下的代码使用。</td>
<td>- 应用程序（用户任务）在非特权模式下的默认堆栈。- 由操作系统调度任务时切换使用，实现任务隔离。</td>
</tr>
<tr>
<td>**寄存器关联**</td>
<td>由`MSP`寄存器直接控制，异常处理时硬件自动加载 / 保存上下文到 MSP。</td>
<td>由`PSP`寄存器控制，需软件（如 OS 调度器）显式切换，用于任务级上下文切换。</td>
</tr>
<tr>
<td>**访问权限**</td>
<td>特权模式下可访问，用户模式下不可直接访问（需通过异常处理切换）。</td>
<td>特权模式和用户模式均可访问，但用户模式下仅能操作自己的 PSP 区域。</td>
</tr>
<tr>
<td>**典型应用**</td>
<td>系统初始化、中断处理、内核服务例程等。</td>
<td>应用任务的局部变量、函数调用栈、中断嵌套时的临时存储（若 OS 支持）。</td>
</tr>
	</table>
65. RTC时钟源：HSE/128、LSE、LSI
66. I2C协议中设备地址有7和10位模式。
67. 电压增益为-20dB， 因为：$`G = 20*\lg A`$， 所以A = 0.1。
68. 多级放大电路的通频带会**变窄**
69. 直流电源滤波用低通滤波， 有用信号为固定频率用带通滤波
70. 运放+若干电阻，无法构成乘法器和振荡器（但可以构成跟随器和比较器）
71. 超声波传感器基于**压电效应**
72. 二段网络的等效电动势，等于端口开路时测的电压。
73. 知道就好
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/16810644-8a23-48be-8ed8-0e8440cba50f.webp)
## DMA专项
### DMA简介
1. DMA 全称DirectMemoryAccess，即直接存储器访问。
2. DMA传输将数据从一个地址空间复制到另一个地址空间。当CPU初始化这个传输动作，传输动作本身是由DMA控制器来实现和完成的。
3. DMA传输方式<span color="blue">无需CPU直接控制传输</span>，也没有中断处理方式那样保留现场和恢复现场过程，通过硬件为RAM和IO设备开辟一条直接传输数据的通道，使得CPU的效率大大提高。
4. DMA作用：为CPU减负。
5. STM32 最多有 2 个 DMA 控制器（DMA2 仅在大容量、互联型产品中存在 ）。DMA1 有 <span color="blue">7</span> 个通道，DMA2 有 <span color="blue">5</span> 个通道 。每个通道专门管理一个或多个外设对存储器访问的请求，搭配仲裁器协调请求优先级。
### STM32 的DMA有以下一些特性：
1. 每个通道都直接连接专用的硬件DMA请求，都支持软件触发，这些通过软件来配置。
2. 在七个请求间的**优先权可以通过软件编程设置(共有**<span color="blue">**四级**</span>**：很高、高、中等和低)，**假如在<br>相等优先权时由硬件决定**(**<span color="blue">**请求0优先于请求1，依此类推**</span>**) **。
3. 独立的源和目标数据区的传输宽度**(字节、半字、全字)**，模拟打包和拆包的过程。**源和<br>目标地址必须按数据传输宽度对齐。**
4. 支持循环的缓冲器管理
5. **每个通道都有3个事件标志**(DMA 半传输，DMA传输完成和DMA传输出错)，这3个<br>事件标志逻辑 <span color="blue">**或**</span> 成为一个单独的中断请求。
6. 外设和存储器，存储器和外设的传输，存储器和存储器间的传输。
7. **闪存、SRAM、外设的SRAM、APB1APB2和AHB外设均可作为访问的源和目标。**
8. **可编程的数据传输数目：最大为65536**
9. 从外设（TIMx、ADC、SPIx、I2Cx 和 USARTx）产生的 DMA 请求，通过逻辑或输入到<br>DMA 控制器，**这就意味着同时只能有一个请求有效。**外设的 DMA 请求，可以通过设置<br>相应的外设寄存器中的控制位，被独立地开启或关闭。
## 总线分类专项
1. 总线按功能和规范可分为五大类型：数据总线、地址总线、控制总线、扩展总线及局部<br>总线。
	- 数据总线、地址总线和控制总线也统称为系统总线，即通常意义上所说的总线。<br>常见的数据总线为ISA、EISA、VESA、PCI等。
	- 地址总线：是专门用来传送地址的，由于地址只能从CPU传向外部存储器或I/O端口，所<br>以地址总线总是单向三态的，这与数据总线不同，地址总线的位数决定了CPU可直接寻址<br>的内存空间大小。
	- 控制总线：用来传送控制信号和时序信号。控制信号中，有的是微处理器送往存储器和I/O<br>接口电路的；也有是其它部件反馈给CPU的，比如：中断申请信号、复位信号、总线请求<br>信号、设备就绪信号等。
2. 按照传输数据的方式划分，可以分为串行总线和并行总线。
	- 串行总线中，二进制数据逐位通过一根数据线发送到目的器件；
	- ++并行总线的数据线通常超过2根。常见的串行总线有SPI、I2C、USB及RS232等。
3. 按照时钟信号是否独立，可以分为同步总线和异步总线。
	- 同步总线的时钟信号独立于数据，而异步总线的时钟信号是从数据中提取出来的。SPI、I2C<br>是同步串行总线，
	- RS232采用异步串行总线。
4. 微机中总线一般有内部总线、系统总线和外部总线。
	- **内部总线**是微机内部各外围芯片与处理器之间的总线，用于芯片一级的互连，有I2C，SCI，<br>IIS，SPI，UART，JTAG，CAN，SDIO，GPIO；
	- **系统总线**是微机中各插件板与系统板之间的总线，用于插件板一级的互连，有ISA，EISA，<br>VESA，PCI；
	- **外部总线**则是微机和外部设备之间的总线，微机作为一种设备，通过该总线和其他设备进行<br>信息与数据交换，它用于设备一级的互连，有RS-232-C，RS-485，IEEE-488，USB。
## 信号转换专项
1. RC桥式正弦波振荡电路，产生正弦波，利用自激震荡
	![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/44414b2c-5a89-4b56-9b97-0576891da82c.webp)
2. 单限比较器可产生矩形波（大于Ut为负，小于Ut为正）
<columns>
	<column ratio="50">
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/9c5a36d3-eebe-4471-b140-be4d1acff8f7.webp)
	</column>
	<column ratio="50">
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/c54afdc7-12af-48db-be0d-0770725d5c84.webp)
		<empty-block/>
	</column>
</columns>
1. 滞回比较器
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/984d14a0-8905-4b2f-838b-2a6f10b9ca94.webp)
1. 积分电路
![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/a4ff6fc2-899c-464b-9fde-b2c19dc7e011.webp)
1. 加减运算电路
<columns>
	<column ratio="50">
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/c5719fd4-8da5-474d-8c73-c618bb507162.webp)
	</column>
	<column ratio="50">
		![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/posts/Post-lanqiaobei/2f983890-621f-4688-b45f-e81a3a1ca773.webp)
	</column>
</columns>
## 一些函数专项
1. **重载函数**
	同一个函数完成不同的功能，常用来实现功能类似而所处理的数据类型<br>不同的问题。允许在同一范围中声明几个功能类似的同名函数，但是这些同名函数的形式参<br>数（指参数的个数、类型或者顺序）必须不同。
2. 内联函数
	其目的是为了提高函数的执行效率，用关键字 `inline` 放在函数定义(注意是定义而非声明)的前面即可将函数指定为内联函数，内联函数一般都是1-5行的小函数。
	```c
inline int max(int a, int b){
		return a > b ? a : b;
}
	```
	调用： `cout<<max(a, b)<<endl;`<br>则在编译时展开为：`cout<<(a>b?a:b)<<endl;` 
	因为调用函数比求解等价表达式要慢得多，所以用内联函数，从而消除了把 max写成函数的额外执行开销
3. 递归函数
	一种计算过程，其中每一步都要用到前一步或前几步的结果，例如连加、连乘及阶乘等。<br>**递归函数必须要有明确的功能和边界条件。**
## 通信外设专项
### **一、串行通信接口与总线协议分类**
### **1. 异步串行通信接口（UART 及其衍生标准）**
- **UART（通用异步收发器）**
	- 性质：异步串行通信的总称，属于物理层接口。
	- 包含标准：RS232、RS499、RS423、RS422、RS485 等。
- **RS 系列接口标准（物理层规范）**
	<table>
<tr>
<td>**标准**</td>
<td>**传输方式**</td>
<td>**电平特性**</td>
<td>**传输线数**</td>
<td>**应用场景**</td>
</tr>
<tr>
<td>RS-232</td>
<td>全双工</td>
<td>逻辑 1：-15V\~-5V，逻辑 0：+3V\~+15V</td>
<td>最少 3 根</td>
<td>MCU 与 PC 机通信（短距离）</td>
</tr>
<tr>
<td>RS-485</td>
<td>半双工</td>
<td>逻辑 1：+2V\~+6V，逻辑 0：-6V\~-2V</td>
<td>2 根</td>
<td>多设备总线（长距离、抗干扰）</td>
</tr>
<tr>
<td>RS-422</td>
<td>全双工</td>
<td>差分信号（类似 RS-485）</td>
<td>4 根</td>
<td>工业控制（较少见）</td>
</tr>
<tr>
<td>RS-423</td>
<td>单端传输</td>
<td>类似 RS-232 但电平范围更宽</td>
<td>-</td>
<td>过渡标准，已较少使用</td>
</tr>
<tr>
<td>RS-499</td>
<td>高速版 RS-422</td>
<td>更高传输速率</td>
<td>-</td>
<td>高速串行通信</td>
</tr>
	</table>
- **TTL 电平（逻辑电平信号）**
	- 传输方式：全双工
	- 电平特性：逻辑 1：2.4V\~5V，逻辑 0：0V\~0.5V
	- 应用：MCU 间直接通信（短距离，需电平转换芯片与 RS-232 互联）。
### **2. 同步串行总线协议**
- **I2C（Inter-Integrated Circuit）**
	- 性质：同步、半双工、二线制总线协议（数据链路层）。
	- 特点：
		- 仅需 SDA（数据线）和 SCL（时钟线），支持多主机仲裁。
		- 传输速率较低（标准模式 100kbps，高速模式 3.4Mbps）。
	- 应用：MCU 与传感器、存储器等低速外设通信。
- **SPI（Serial Peripheral Interface）**
	- 性质：同步、全双工、高速总线协议。
	- 特点：
		- 4 根线：SCK（时钟）、MOSI（主发从收）、MISO（主收从发）、CS（片选）。
		- 传输速率可达几十 Mbps，支持单主机多从机。
	- 应用：高速外设通信（如 Flash、ADC、显示屏驱动）。
### **3. 差分信号总线**
- **CAN（Controller Area Network）**
	- 性质：独立差分总线，物理层与数据链路层集成。
	- 特点：
		- 电平定义：CAN_High 与 CAN_Low 的电压差（逻辑 1：-1.5V\~0V，逻辑 0：+1.5V\~+3V）。
		- 抗干扰能力强，支持多节点实时通信（汽车电子、工业控制）。
	- 硬件：独立引脚，可与 USB 复用。
- **RS-485（属于差分信号接口）**
	- 见 “RS 系列接口标准” 部分。
### **4. 通用外部总线（USB）**
- **USB（Universal Serial Bus）**
	- 性质：高速串行总线标准，独立于串口体系。
	- 特点：
		- 4 根线：GND、5V 电源线、D + 和 D-（差分信号线，3.3V 电平）。
		- 支持热插拔、高速传输（USB 2.0 最高 480Mbps，USB 3.0 达 5Gbps）。
		- 单主机架构，最多连接 127 个设备（通过 Hub 扩展）。
	- 应用：电脑与外设（键盘、鼠标、存储设备等）。
### **二、核心对比维度总结**
<table>
<tr>
<td>**分类维度**</td>
<td>**异步串行（UART/RS）**</td>
<td>**同步总线（I2C/SPI）**</td>
<td>**差分总线（CAN/RS-485）**</td>
<td>**USB**</td>
</tr>
<tr>
<td>**时钟同步**</td>
<td>无需（异步）</td>
<td>需要（同步）</td>
<td>部分需要（如 CAN）</td>
<td>内置时钟同步机制</td>
</tr>
<tr>
<td>**传输方式**</td>
<td>全双工 / 半双工</td>
<td>全双工（SPI）/ 半双工（I2C）</td>
<td>半双工（RS-485）/ 全双工（CAN）</td>
<td>全双工</td>
</tr>
<tr>
<td>**抗干扰性**</td>
<td>单端信号（RS-232 差）</td>
<td>单端信号（抗干扰性一般）</td>
<td>差分信号（抗干扰性强）</td>
<td>差分信号 + 屏蔽线（抗干扰性强）</td>
</tr>
<tr>
<td>**典型应用**</td>
<td>低速串口通信</td>
<td>芯片间低速 / 高速通信</td>
<td>工业控制、汽车电子</td>
<td>外设通用接口</td>
</tr>
<tr>
<td>**电平标准**</td>
<td>TTL/RS-232 等</td>
<td>TTL 兼容</td>
<td>差分电平</td>
<td>专用 3.3V 差分电平</td>
</tr>
</table>
### **三、关键术语补充**
- **物理层与数据链路层**：RS 系列、TTL、CAN 等属于物理层规范（定义电气特性）；I2C、SPI 等属于数据链路层协议（定义通信时序与逻辑）。
- **全双工与半双工**：
	- 全双工：可同时收发数据（如 RS-232、SPI）。
	- 半双工：同一时刻只能收或发（如 RS-485、I2C）。
- **差分信号**：通过两根线的电压差传输数据，抗共模干扰能力强（如 CAN、RS-485），适合长距离通信。
<empty-block/>
# **C 语言字符串函数详解**
在 C 语言中，字符串处理是非常基础且重要的操作，下面对代码中演示的字符串函数进行系统总结：
### **1. 字符串查找函数 - strstr**
- **功能**：在主字符串中查找子字符串首次出现的位置
- **原型**：`char *strstr(const char *haystack, const char *needle);`
- **参数**：
	- `haystack`：主字符串（被查找的字符串）
	- `needle`：子字符串（要查找的字符串）
- **返回值**：子字符串首次出现的地址，若未找到返回 NULL
- **示例**：`strstr("nihao beijing", "jing")` 会返回 "jing" 的起始地址
- **注意**：查找时区分大小写，且子字符串不能是空字符串
### **2. 字符串转数值函数 - atoi**
- **功能**：将字符串转换为整数
- **原型**：`int atoi(const char *nptr);`
- **参数**：`nptr` 为要转换的字符串
- **返回值**：转换后的整数值
- **示例**：`atoi("100 100 100")` 会返回 100（遇到非数字字符停止转换）
- **注意**：
	- 跳过字符串前的空白字符
	- 遇到第一个非数字字符时停止转换
	- 结果范围受 int 类型限制
### **3. 字符串分割函数 - strtok**
- **功能**：按指定分隔符分割字符串
- **原型**：`char *strtok(char *str, const char *delim);`
- **参数**：
	- `str`：要分割的字符串（首次调用时传入，后续调用传 NULL）
	- `delim`：分隔符字符串
- **返回值**：分割后的子字符串，若无更多子串则返回 NULL
- **示例**：`strtok("(101,102,103,104)", "(),")` 可解析出整数列表
- **注意**：
	- 函数会修改原字符串（在分隔符处替换为 '\\0'）
	- 首次调用后，后续调用需传入 NULL 以继续分割
### **4. 格式化字符串读取 - sscanf**
- **功能**：从字符串中按指定格式读取数据
- **原型**：`int sscanf(const char *str, const char *format, ...);`
- **参数**：
	- `str`：要读取的字符串
	- `format`：格式控制字符串
	- `...`：接收数据的变量指针
- **返回值**：成功匹配的参数个数
- **示例**：`sscanf("(1,2,3,4)", "(%d,%d,%d,%d)", &a,&b,&c,&d)`
- **注意**：与 scanf 功能类似，但数据源是字符串而非标准输入
### **5. 字符串比较函数 - strcmp 与 strncmp**
<table>
<tr>
<td>**函数**</td>
<td>**功能**</td>
<td>**原型**</td>
<td>**返回值解释**</td>
</tr>
<tr>
<td>`strcmp`</td>
<td>比较两个字符串</td>
<td>`int strcmp(const char *s1, const char *s2);`</td>
<td>若 s1\<s2 返回负数，若 s1==s2 返回 0，若 s1\>s2 返回正数</td>
</tr>
<tr>
<td>`strncmp`</td>
<td>比较两个字符串的前 n 个字符</td>
<td>`int strncmp(const char *s1, const char *s2, size_t n);`</td>
<td>前 n 个字符的比较结果</td>
</tr>
</table>
- **示例**：`strcmp("abc", "abd")` 返回负数，因为 'c'\<'d'
- **注意**：
	- 按字符 ASCII 值逐个比较
	- 遇到 '\\0' 或比较完 n 个字符时停止（strncmp）
### **6. 字符串连接函数 - strcat 与 strncat**
<table>
<tr>
<td>**函数**</td>
<td>**功能**</td>
<td>**原型**</td>
<td>**注意事项**</td>
</tr>
<tr>
<td>`strcat`</td>
<td>连接两个字符串</td>
<td>`char *strcat(char *dest, const char *src);`</td>
<td>dest 必须有足够空间容纳连接后的字符串</td>
</tr>
<tr>
<td>`strncat`</td>
<td>连接两个字符串的前 n 个字符</td>
<td>`char *strncat(char *dest, const char *src, size_t n);`</td>
<td>最多添加 n 个字符，自动添加 '\\0'</td>
</tr>
</table>
- **示例**：`strcat("nihao", " beijing")` 结果为 "nihao beijing"
- **注意**：
	- 源字符串 (src) 的 '\\0' 会被复制到目标字符串 (dest)
	- 目标字符串必须有足够的空间（至少为 dest 长度 + src 长度 + 1）
### **7. 字符串复制函数 - strcpy 与 strncpy**
<table>
<tr>
<td>**函数**</td>
<td>**功能**</td>
<td>**原型**</td>
<td>**注意事项**</td>
</tr>
<tr>
<td>`strcpy`</td>
<td>复制字符串</td>
<td>`char *strcpy(char *dest, const char *src);`</td>
<td>dest 必须有足够空间</td>
</tr>
<tr>
<td>`strncpy`</td>
<td>复制字符串的前 n 个字符</td>
<td>`char *strncpy(char *dest, const char *src, size_t n);`</td>
<td>若 src 长度小于 n，dest 不会自动补 '\\0'</td>
</tr>
</table>
- **示例**：`strcpy(dest, "Hello")` 将 "Hello" 复制到 dest
- **注意**：
	- strcpy 会复制 src 的 '\\0' 到 dest
	- strncpy 若 src 长度 \>=n，dest 不会自动添加 '\\0'，需手动处理
# **math 库常用函数**
在蓝桥杯嵌入式比赛中，math.h 库的函数也是重点考察内容，以下是需要熟练掌握的常用函数：
### **1. 数值计算函数**
<table>
<tr>
<td>**函数**</td>
<td>**功能**</td>
<td>**原型**</td>
<td>**示例**</td>
</tr>
<tr>
<td>`fabs`</td>
<td>计算浮点数的绝对值</td>
<td>`double fabs(double x);`</td>
<td>`fabs(-3.14)` 返回 3.14</td>
</tr>
<tr>
<td>`floor`</td>
<td>向下取整</td>
<td>`double floor(double x);`</td>
<td>`floor(3.8)` 返回 3.0</td>
</tr>
<tr>
<td>`ceil`</td>
<td>向上取整</td>
<td>`double ceil(double x);`</td>
<td>`ceil(3.2)` 返回 4.0</td>
</tr>
<tr>
<td>`round`</td>
<td>四舍五入取整</td>
<td>`double round(double x);`</td>
<td>`round(3.6)` 返回 4.0</td>
</tr>
<tr>
<td>`fmod`</td>
<td>计算浮点数取模</td>
<td>`double fmod(double x, double y);`</td>
<td>`fmod(7.0, 3.0)` 返回 1.0</td>
</tr>
</table>
### **2. 幂指对数函数**
<table>
<tr>
<td>**函数**</td>
<td>**功能**</td>
<td>**原型**</td>
<td>**示例**</td>
</tr>
<tr>
<td>`pow`</td>
<td>计算 x 的 y 次方</td>
<td>`double pow(double x, double y);`</td>
<td>`pow(2, 3)` 返回 8.0</td>
</tr>
<tr>
<td>`sqrt`</td>
<td>计算平方根</td>
<td>`double sqrt(double x);`</td>
<td>`sqrt(25.0)` 返回 5.0</td>
</tr>
<tr>
<td>`exp`</td>
<td>计算自然指数 (e\^x)</td>
<td>`double exp(double x);`</td>
<td>`exp(1)` 返回 e≈2.71828</td>
</tr>
<tr>
<td>`log`</td>
<td>计算自然对数 (lnx)</td>
<td>`double log(double x);`</td>
<td>`log(e)` 返回 1.0</td>
</tr>
<tr>
<td>`log10`</td>
<td>计算常用对数 (log₁₀x)</td>
<td>`double log10(double x);`</td>
<td>`log10(100)` 返回 2.0</td>
</tr>
</table>
### **3. 三角函数**
<table>
<tr>
<td>**函数**</td>
<td>**功能**</td>
<td>**原型**</td>
<td>**注意事项**</td>
</tr>
<tr>
<td>`sin`</td>
<td>计算正弦值</td>
<td>`double sin(double x);`</td>
<td>x 为弧度值</td>
</tr>
<tr>
<td>`cos`</td>
<td>计算余弦值</td>
<td>`double cos(double x);`</td>
<td>x 为弧度值</td>
</tr>
<tr>
<td>`tan`</td>
<td>计算正切值</td>
<td>`double tan(double x);`</td>
<td>x 为弧度值</td>
</tr>
<tr>
<td>`asin`</td>
<td>计算反正弦值</td>
<td>`double asin(double x);`</td>
<td>返回值为弧度，范围 \[-π/2,π/2\]</td>
</tr>
<tr>
<td>`acos`</td>
<td>计算反余弦值</td>
<td>`double acos(double x);`</td>
<td>返回值为弧度，范围 \[0,π\]</td>
</tr>
<tr>
<td>`atan`</td>
<td>计算反正切值</td>
<td>`double atan(double x);`</td>
<td>返回值为弧度，范围 \[-π/2,π/2\]</td>
</tr>
<tr>
<td>`atan2`</td>
<td>计算二维反正切</td>
<td>`double atan2(double y, double x);`</td>
<td>返回值为弧度，范围 \[-π,π\]</td>
</tr>
</table>
### **4. 类型转换函数**
<table>
<tr>
<td>**函数**</td>
<td>**功能**</td>
<td>**原型**</td>
<td>**示例**</td>
</tr>
<tr>
<td>`fabsf`</td>
<td>单精度浮点数绝对值</td>
<td>`float fabsf(float x);`</td>
<td>与 fabs 类似，但处理 float 类型</td>
</tr>
<tr>
<td>`floorf`</td>
<td>单精度浮点数向下取整</td>
<td>`float floorf(float x);`</td>
<td>与 floor 类似，但处理 float 类型</td>
</tr>
<tr>
<td>`roundf`</td>
<td>单精度浮点数四舍五入</td>
<td>`float roundf(float x);`</td>
<td>与 round 类似，但处理 float 类型</td>
</tr>
<tr>
<td>`trunc`</td>
<td>截断取整（直接舍去小数部分）</td>
<td>`double trunc(double x);`</td>
<td>`trunc(3.8)` 返回 3.0</td>
</tr>
</table>
# 冒泡排序
```c
// 基础冒泡排序 - 升序排列
void bubbleSort(int arr[], int n) {
    int i, j, temp;
    // 外层循环：控制排序轮次，共需n-1轮
    for (i = 0; i < n - 1; i++) {
        // 内层循环：每轮将最大元素"冒泡"到末尾
        for (j = 0; j < n - i - 1; j++) {
            // 比较相邻元素，若前一个大于后一个则交换
            if (arr[j] > arr[j + 1]) {
                temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```