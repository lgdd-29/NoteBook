# 介绍

OLED的驱动芯片为SSD1306，驱动SSD1306意味着驱动OLED，因为OLED裸屏就是一个根据128+64引脚的拉高拉低来点灯。

•SSD1306是一款OLED/PLED点阵显示屏的控制器，可以嵌入在屏幕中，用于执行接收数据、显示存储、扫描刷新等任务

•驱动接口：128个SEG引脚和64个COM引脚，对应128*64像素点阵显示屏

•内置显示存储器（GDDRAM）：128*64 bit （128*8 Byte）SRAM

•供电：VDD=1.65~3.3V（IC 逻辑），VCC=7~15V（面板驱动）

•通信接口：8位6800/8080并行接口，3/4线SPI接口，I2C接口

# 学习流程

学习OLED的流程，首先是知道OLED的显示原理，然后了解OLED的驱动芯片，可以通过芯片的数据手册、原理图、商家的代码资料。

在看驱动芯片时，要知道是使用什么通信协议，怎么配置，怎么选择发送数据还是命令。

在了解通信协议之后，要了解芯片怎么驱动OLED，怎样进行初始化配置，这个一般芯片手册上有写，也可以通过解析商家代码来看。

芯片的数据手册上的命令部分最为重要，里面是芯片所有的驱动命令，所以想要用代码完成芯片的初始化和驱动必须看命令部分。



# 初始化

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521163652.png)

> 这个是屏幕的初始化要发送的步骤
>
> 初始化I2C，释放SDL和SDA->关闭屏幕->定义时钟，设置时钟大小->设置多路复用率->设置垂直偏移->设置显示RAM起始行寄存器位置->设置左右方向->设置上下方向->设置COM引脚硬件配置->设置对比度->设置预充电期->调整Vcomh电压->打开显示器->设置正常显示->打开电压->打开显示
>
> 时钟的值不明白为什么是0x80，但是可以是其他值，值过大会闪烁，值过小会太暗)
>
> 多用复用率的值因为一般是128*64则填（64-1）的值，即是3F
>
> 设置COM引脚硬件配置的值为0x12是因为`0x12` 是顺序COM配置的值，而 `0x02` 是交错COM配置的值。
>
> 对比度的值为0xCF的原因应该是取中间值，0xFF也是可以的，但是对比有点太明显了。
>
> 预充电期的值也是一个折中的值，既保证了显示效果，也保证了一定的刷新率
>
> Vcomh电压的值最好偏小一点，因为高了会烧坏显示屏

如果没有初始化框图，我们怎么知道该配置哪些地址呢。

SSD1306命令分为：基本命令、滚动命令、寻址设置命令、硬件配置命令、定时驾驶方案设置

基本命令包含：设置对比度、打开显示器、正常/反向显示、设置显示器模式

滚动命令包含：滚动模式以及各种参数（普通显示用不到）

寻址设置命令包含：设置页面寻址起始地址、设置寻址模式、设置起始地址和结束地址、页面地址设置、GDDRAM页起始地址

硬件配置命令：设置RAM显示起始行、设置分段重映射、设置多路复用率、设置COM输出扫描方向、设置显示偏移、设置COM引脚硬件配置

定时驾驶方案命令包含：设置时钟、设置预充电、设置Vcomh

OLED的初始化中将基本命令、硬件配置命令和定时命令都配置了，滚动没用到所以没配置滚动命令配置，而寻址设置命令是用来写入数据的，通过寻址设置命令，将数据写在固定的地方，所以寻址命令用在写数据而不是初始化

重点在于该怎么配置屏幕的初始化，以及通信方式

虽然使用的是I2C通信方式，但是这是最底层的，与芯片的通信是建立在I2C上的，如何用I2C表达出命令是关键，前提是知道发之前大概要先发什么地址

```c
/**
  * 函    数：OLED写命令
  * 参    数：Command 要写入的命令值，范围：0x00~0xFF
  * 返 回 值：无
  */
void OLED_WriteCommand(uint8_t Command)
{
	OLED_I2C_Start();				//I2C起始
	OLED_I2C_SendByte(0x78);		//发送OLED的I2C从机地址
	OLED_I2C_SendByte(0x00);		//控制字节，给0x00，表示即将写命令，可以从I2C通信相关内容找到这部分内容
	OLED_I2C_SendByte(Command);		//写入指定的命令
	OLED_I2C_Stop();				//I2C终止
}

/**
  * 函    数：OLED写数据
  * 参    数：Data 要写入数据的起始地址
  * 参    数：Count 要写入数据的数量
  * 返 回 值：无
  */
void OLED_WriteData(uint8_t *Data, uint8_t Count)
{
	uint8_t i;
	
	OLED_I2C_Start();				//I2C起始
	OLED_I2C_SendByte(0x78);		//发送OLED的I2C从机地址
	OLED_I2C_SendByte(0x40);		//控制字节，给0x40，表示即将写数据
	/*循环Count次，进行连续的数据写入*/
	for (i = 0; i < Count; i ++)
	{
		OLED_I2C_SendByte(Data[i]);	//依次发送Data的每一个数据,一个一个字节的发送
	}
	OLED_I2C_Stop();				//I2C终止
}
```

思路：

> 刚开始先发送一个起始信号，如果是发送命令的话，根据数据手册，要先发送从机地址，根据原理图可以看出来从机地址为0x78，然后如果要发送命令，则Co为0，D/C#为0，后面数据默认为0，所以发送0x78后再发送个0x00表示我要发送命令，然后接着发送想要的命令即可。数据的话就是D/C#为1，后面数据默认为0，所以发送0x78后再发送个0x40表示我要发送数据，后面紧跟数据就可以了。（命令是用来配置OLED，数据是用来显示的）

虽然用的是I2C，但是实际使用要根据芯片的数据手册去写通信。上面是SSD1306的底层通信，是发送数据还是发送命令。从机地址以及是写数据还是写命令可以从数据手册的相关内容中看出。

还有屏幕的初始化，也是建立在通信之上的，一般的初始化就是发送各种命令来配置屏幕，比如打开时钟，设置对比度之类的，前提是能发送命令。所以使用OLED的顺序是，I2C底层协议，用I2C写通信底层，用通信底层配置初始化以及写各种功能函数，最后在主函数中调用功能函数调用屏幕。

```c
/**
  * 函    数：OLED初始化
  * 参    数：无
  * 返 回 值：无
  * 说    明：使用前，需要调用此初始化函数
  */
void OLED_Init(void)
{
	OLED_GPIO_Init();			//先调用底层的端口初始化
	
	/*写入一系列的命令，对OLED进行初始化配置*/
	OLED_WriteCommand(0xAE);	//设置显示开启/关闭，0xAE关闭，0xAF开启
	
	OLED_WriteCommand(0xD5);	//设置显示时钟分频比/振荡器频率
	OLED_WriteCommand(0x80);	//0x00~0xFF
	
	OLED_WriteCommand(0xA8);	//设置多路复用率
	OLED_WriteCommand(0x3F);	//0x0E~0x3F
	
	OLED_WriteCommand(0xD3);	//设置显示偏移
	OLED_WriteCommand(0x00);	//0x00~0x7F
	
	OLED_WriteCommand(0x40);	//设置显示开始行，0x40~0x7F
	
	OLED_WriteCommand(0xA1);	//设置左右方向，0xA1正常，0xA0左右反置
	
	OLED_WriteCommand(0xC8);	//设置上下方向，0xC8正常，0xC0上下反置

	OLED_WriteCommand(0xDA);	//设置COM引脚硬件配置
	OLED_WriteCommand(0x12);
	
	OLED_WriteCommand(0x81);	//设置对比度
	OLED_WriteCommand(0xCF);	//0x00~0xFF

	OLED_WriteCommand(0xD9);	//设置预充电周期
	OLED_WriteCommand(0xF1);

	OLED_WriteCommand(0xDB);	//设置VCOMH取消选择级别
	OLED_WriteCommand(0x30);

	OLED_WriteCommand(0xA4);	//设置整个显示打开/关闭

	OLED_WriteCommand(0xA6);	//设置正常/反色显示，0xA6正常，0xA7反色

	OLED_WriteCommand(0x8D);	//设置充电泵
	OLED_WriteCommand(0x14);

	OLED_WriteCommand(0xAF);	//开启显示
	
	OLED_Clear();				//清空显存数组
	OLED_Update();				//更新显示，清屏，防止初始化后未显示内容时花屏
}
```



# 库文件

## OLED.C

```c
#include "stm32f10x.h"
#include "OLED.h"
#include <string.h>
#include <math.h>
#include <stdio.h>
#include <stdarg.h>

/**
  * 数据存储格式：
  * 纵向8点，高位在下，先从左到右，再从上到下
  * 每一个Bit对应一个像素点
  * 
  *      B0 B0                  B0 B0
  *      B1 B1                  B1 B1
  *      B2 B2                  B2 B2
  *      B3 B3  ------------->  B3 B3 --
  *      B4 B4                  B4 B4  |
  *      B5 B5                  B5 B5  |
  *      B6 B6                  B6 B6  |
  *      B7 B7                  B7 B7  |
  *                                    |
  *  -----------------------------------
  *  |   
  *  |   B0 B0                  B0 B0
  *  |   B1 B1                  B1 B1
  *  |   B2 B2                  B2 B2
  *  --> B3 B3  ------------->  B3 B3
  *      B4 B4                  B4 B4
  *      B5 B5                  B5 B5
  *      B6 B6                  B6 B6
  *      B7 B7                  B7 B7
  * 
  * 坐标轴定义：
  * 左上角为(0, 0)点
  * 横向向右为X轴，取值范围：0~127
  * 纵向向下为Y轴，取值范围：0~63
  * 
  *       0             X轴           127 
  *      .------------------------------->
  *    0 |
  *      |
  *      |
  *      |
  *  Y轴 |
  *      |
  *      |
  *      |
  *   63 |
  *      v
  * 
  */


/*全局变量*********************/

/**
  * OLED显存数组
  * 所有的显示函数，都只是对此显存数组进行读写
  * 随后调用OLED_Update函数或OLED_UpdateArea函数
  * 才会将显存数组的数据发送到OLED硬件，进行显示
  */
uint8_t OLED_DisplayBuf[8][128];

/*********************全局变量*/


/*引脚配置*********************/

/**
  * 函    数：OLED写SCL高低电平
  * 参    数：要写入SCL的电平值，范围：0/1
  * 返 回 值：无
  * 说    明：当上层函数需要写SCL时，此函数会被调用
  *           用户需要根据参数传入的值，将SCL置为高电平或者低电平
  *           当参数传入0时，置SCL为低电平，当参数传入1时，置SCL为高电平
  */
void OLED_W_SCL(uint8_t BitValue)
{
	/*根据BitValue的值，将SCL置高电平或者低电平*/
	GPIO_WriteBit(GPIOB, GPIO_Pin_8, (BitAction)BitValue);
	
	/*如果单片机速度过快，可在此添加适量延时，以避免超出I2C通信的最大速度*/
	//...
}

/**
  * 函    数：OLED写SDA高低电平
  * 参    数：要写入SDA的电平值，范围：0/1
  * 返 回 值：无
  * 说    明：当上层函数需要写SDA时，此函数会被调用
  *           用户需要根据参数传入的值，将SDA置为高电平或者低电平
  *           当参数传入0时，置SDA为低电平，当参数传入1时，置SDA为高电平
  */
void OLED_W_SDA(uint8_t BitValue)
{
	/*根据BitValue的值，将SDA置高电平或者低电平*/
	GPIO_WriteBit(GPIOB, GPIO_Pin_9, (BitAction)BitValue);
	
	/*如果单片机速度过快，可在此添加适量延时，以避免超出I2C通信的最大速度*/
	//...
}

/**
  * 函    数：OLED引脚初始化
  * 参    数：无
  * 返 回 值：无
  * 说    明：当上层函数需要初始化时，此函数会被调用
  *           用户需要将SCL和SDA引脚初始化为开漏模式，并释放引脚
  */
void OLED_GPIO_Init(void)
{
	uint32_t i, j;
	
	/*在初始化前，加入适量延时，待OLED供电稳定*/
	for (i = 0; i < 1000; i ++)
	{
		for (j = 0; j < 1000; j ++);
	}
	
	/*将SCL和SDA引脚初始化为开漏模式*/
    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
 	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_OD;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_8;
 	GPIO_Init(GPIOB, &GPIO_InitStructure);
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
 	GPIO_Init(GPIOB, &GPIO_InitStructure);
	
	/*释放SCL和SDA*/
	OLED_W_SCL(1);
	OLED_W_SDA(1);
}

/*********************引脚配置*/


/*通信协议*********************/

/**
  * 函    数：I2C起始
  * 参    数：无
  * 返 回 值：无
  */
void OLED_I2C_Start(void)
{
	OLED_W_SDA(1);		//释放SDA，确保SDA为高电平
	OLED_W_SCL(1);		//释放SCL，确保SCL为高电平
	OLED_W_SDA(0);		//在SCL高电平期间，拉低SDA，产生起始信号
	OLED_W_SCL(0);		//起始后把SCL也拉低，即为了占用总线，也为了方便总线时序的拼接
}

/**
  * 函    数：I2C终止
  * 参    数：无
  * 返 回 值：无
  */
void OLED_I2C_Stop(void)
{
	OLED_W_SDA(0);		//拉低SDA，确保SDA为低电平
	OLED_W_SCL(1);		//释放SCL，使SCL呈现高电平
	OLED_W_SDA(1);		//在SCL高电平期间，释放SDA，产生终止信号
}

/**
  * 函    数：I2C发送一个字节
  * 参    数：Byte 要发送的一个字节数据，范围：0x00~0xFF
  * 返 回 值：无
  */
void OLED_I2C_SendByte(uint8_t Byte)
{
	uint8_t i;
	
	/*循环8次，主机依次发送数据的每一位*/
	for (i = 0; i < 8; i++)
	{
		/*使用掩码的方式取出Byte的指定一位数据并写入到SDA线*/
		/*两个!的作用是，让所有非零的值变为1*/
		OLED_W_SDA(!!(Byte & (0x80 >> i)));
		OLED_W_SCL(1);	//释放SCL，从机在SCL高电平期间读取SDA
		OLED_W_SCL(0);	//拉低SCL，主机开始发送下一位数据
	}
	
	OLED_W_SCL(1);		//额外的一个时钟，不处理应答信号
	OLED_W_SCL(0);
}

/**
  * 函    数：OLED写命令
  * 参    数：Command 要写入的命令值，范围：0x00~0xFF
  * 返 回 值：无
  */
void OLED_WriteCommand(uint8_t Command)
{
	OLED_I2C_Start();				//I2C起始
	OLED_I2C_SendByte(0x78);		//发送OLED的I2C从机地址
	OLED_I2C_SendByte(0x00);		//控制字节，给0x00，表示即将写命令
	OLED_I2C_SendByte(Command);		//写入指定的命令
	OLED_I2C_Stop();				//I2C终止
}

/**
  * 函    数：OLED写数据
  * 参    数：Data 要写入数据的起始地址
  * 参    数：Count 要写入数据的数量
  * 返 回 值：无
  */
void OLED_WriteData(uint8_t *Data, uint8_t Count)
{
	uint8_t i;
	
	OLED_I2C_Start();				//I2C起始
	OLED_I2C_SendByte(0x78);		//发送OLED的I2C从机地址
	OLED_I2C_SendByte(0x40);		//控制字节，给0x40，表示即将写数据
	/*循环Count次，进行连续的数据写入*/
	for (i = 0; i < Count; i ++)
	{
		OLED_I2C_SendByte(Data[i]);	//依次发送Data的每一个数据
	}
	OLED_I2C_Stop();				//I2C终止
}

/*********************通信协议*/


/*硬件配置*********************/

/**
  * 函    数：OLED初始化
  * 参    数：无
  * 返 回 值：无
  * 说    明：使用前，需要调用此初始化函数
  */
void OLED_Init(void)
{
	OLED_GPIO_Init();			//先调用底层的端口初始化
	
	/*写入一系列的命令，对OLED进行初始化配置*/
	OLED_WriteCommand(0xAE);	//设置显示开启/关闭，0xAE关闭，0xAF开启
	
	OLED_WriteCommand(0xD5);	//设置显示时钟分频比/振荡器频率
	OLED_WriteCommand(0x80);	//0x00~0xFF
	
	OLED_WriteCommand(0xA8);	//设置多路复用率
	OLED_WriteCommand(0x3F);	//0x0E~0x3F
	
	OLED_WriteCommand(0xD3);	//设置显示偏移
	OLED_WriteCommand(0x00);	//0x00~0x7F
	
	OLED_WriteCommand(0x40);	//设置显示开始行，0x40~0x7F
	
	OLED_WriteCommand(0xA1);	//设置左右方向，0xA1正常，0xA0左右反置
	
	OLED_WriteCommand(0xC8);	//设置上下方向，0xC8正常，0xC0上下反置

	OLED_WriteCommand(0xDA);	//设置COM引脚硬件配置
	OLED_WriteCommand(0x12);
	
	OLED_WriteCommand(0x81);	//设置对比度
	OLED_WriteCommand(0xCF);	//0x00~0xFF

	OLED_WriteCommand(0xD9);	//设置预充电周期
	OLED_WriteCommand(0xF1);

	OLED_WriteCommand(0xDB);	//设置VCOMH取消选择级别
	OLED_WriteCommand(0x30);

	OLED_WriteCommand(0xA4);	//设置整个显示打开/关闭

	OLED_WriteCommand(0xA6);	//设置正常/反色显示，0xA6正常，0xA7反色

	OLED_WriteCommand(0x8D);	//设置充电泵
	OLED_WriteCommand(0x14);

	OLED_WriteCommand(0xAF);	//开启显示
	
	OLED_Clear();				//清空显存数组
	OLED_Update();				//更新显示，清屏，防止初始化后未显示内容时花屏
}

/**
  * 函    数：OLED设置显示光标位置
  * 参    数：Page 指定光标所在的页，范围：0~7
  * 参    数：X 指定光标所在的X轴坐标，范围：0~127
  * 返 回 值：无
  * 说    明：OLED默认的Y轴，只能8个Bit为一组写入，即1页等于8个Y轴坐标
  */
void OLED_SetCursor(uint8_t Page, uint8_t X)
{
	/*如果使用此程序驱动1.3寸的OLED显示屏，则需要解除此注释*/
	/*因为1.3寸的OLED驱动芯片（SH1106）有132列*/
	/*屏幕的起始列接在了第2列，而不是第0列*/
	/*所以需要将X加2，才能正常显示*/
//	X += 2;
	
	/*通过指令设置页地址和列地址*/
	OLED_WriteCommand(0xB0 | Page);					//设置页位置
	OLED_WriteCommand(0x10 | ((X & 0xF0) >> 4));	//设置X位置高4位
	OLED_WriteCommand(0x00 | (X & 0x0F));			//设置X位置低4位
}

/*********************硬件配置*/


/*工具函数*********************/

/*工具函数仅供内部部分函数使用*/

/**
  * 函    数：次方函数
  * 参    数：X 底数
  * 参    数：Y 指数
  * 返 回 值：等于X的Y次方
  */
uint32_t OLED_Pow(uint32_t X, uint32_t Y)
{
	uint32_t Result = 1;	//结果默认为1
	while (Y --)			//累乘Y次
	{
		Result *= X;		//每次把X累乘到结果上
	}
	return Result;
}

/**
  * 函    数：判断指定点是否在指定多边形内部
  * 参    数：nvert 多边形的顶点数
  * 参    数：vertx verty 包含多边形顶点的x和y坐标的数组
  * 参    数：testx testy 测试点的X和y坐标
  * 返 回 值：指定点是否在指定多边形内部，1：在内部，0：不在内部
  */
uint8_t OLED_pnpoly(uint8_t nvert, int16_t *vertx, int16_t *verty, int16_t testx, int16_t testy)
{
	int16_t i, j, c = 0;
	
	/*此算法由W. Randolph Franklin提出*/
	/*参考链接：https://wrfranklin.org/Research/Short_Notes/pnpoly.html*/
	for (i = 0, j = nvert - 1; i < nvert; j = i++)
	{
		if (((verty[i] > testy) != (verty[j] > testy)) &&
			(testx < (vertx[j] - vertx[i]) * (testy - verty[i]) / (verty[j] - verty[i]) + vertx[i]))
		{
			c = !c;
		}
	}
	return c;
}

/**
  * 函    数：判断指定点是否在指定角度内部
  * 参    数：X Y 指定点的坐标
  * 参    数：StartAngle EndAngle 起始角度和终止角度，范围：-180~180
  *           水平向右为0度，水平向左为180度或-180度，下方为正数，上方为负数，顺时针旋转
  * 返 回 值：指定点是否在指定角度内部，1：在内部，0：不在内部
  */
uint8_t OLED_IsInAngle(int16_t X, int16_t Y, int16_t StartAngle, int16_t EndAngle)
{
	int16_t PointAngle;
	PointAngle = atan2(Y, X) / 3.14 * 180;	//计算指定点的弧度，并转换为角度表示
	if (StartAngle < EndAngle)	//起始角度小于终止角度的情况
	{
		/*如果指定角度在起始终止角度之间，则判定指定点在指定角度*/
		if (PointAngle >= StartAngle && PointAngle <= EndAngle)
		{
			return 1;
		}
	}
	else			//起始角度大于于终止角度的情况
	{
		/*如果指定角度大于起始角度或者小于终止角度，则判定指定点在指定角度*/
		if (PointAngle >= StartAngle || PointAngle <= EndAngle)
		{
			return 1;
		}
	}
	return 0;		//不满足以上条件，则判断判定指定点不在指定角度
}

/*********************工具函数*/


/*功能函数*********************/

/**
  * 函    数：将OLED显存数组更新到OLED屏幕
  * 参    数：无
  * 返 回 值：无
  * 说    明：所有的显示函数，都只是对OLED显存数组进行读写
  *           随后调用OLED_Update函数或OLED_UpdateArea函数
  *           才会将显存数组的数据发送到OLED硬件，进行显示
  *           故调用显示函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_Update(void)
{
	uint8_t j;
	/*遍历每一页*/
	for (j = 0; j < 8; j ++)
	{
		/*设置光标位置为每一页的第一列*/
		OLED_SetCursor(j, 0);
		/*连续写入128个数据，将显存数组的数据写入到OLED硬件*/
		OLED_WriteData(OLED_DisplayBuf[j], 128);
	}
}

/**
  * 函    数：将OLED显存数组部分更新到OLED屏幕
  * 参    数：X 指定区域左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定区域左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Width 指定区域的宽度，范围：0~128
  * 参    数：Height 指定区域的高度，范围：0~64
  * 返 回 值：无
  * 说    明：此函数会至少更新参数指定的区域
  *           如果更新区域Y轴只包含部分页，则同一页的剩余部分会跟随一起更新
  * 说    明：所有的显示函数，都只是对OLED显存数组进行读写
  *           随后调用OLED_Update函数或OLED_UpdateArea函数
  *           才会将显存数组的数据发送到OLED硬件，进行显示
  *           故调用显示函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_UpdateArea(int16_t X, int16_t Y, uint8_t Width, uint8_t Height)
{
	int16_t j;
	int16_t Page, Page1;
	
	/*负数坐标在计算页地址时需要加一个偏移*/
	/*(Y + Height - 1) / 8 + 1的目的是(Y + Height) / 8并向上取整*/
	Page = Y / 8;
	Page1 = (Y + Height - 1) / 8 + 1;
	if (Y < 0)
	{
		Page -= 1;
		Page1 -= 1;
	}
	
	/*遍历指定区域涉及的相关页*/
	for (j = Page; j < Page1; j ++)
	{
		if (X >= 0 && X <= 127 && j >= 0 && j <= 7)		//超出屏幕的内容不显示
		{
			/*设置光标位置为相关页的指定列*/
			OLED_SetCursor(j, X);
			/*连续写入Width个数据，将显存数组的数据写入到OLED硬件*/
			OLED_WriteData(&OLED_DisplayBuf[j][X], Width);
		}
	}
}

/**
  * 函    数：将OLED显存数组全部清零
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_Clear(void)
{
	uint8_t i, j;
	for (j = 0; j < 8; j ++)				//遍历8页
	{
		for (i = 0; i < 128; i ++)			//遍历128列
		{
			OLED_DisplayBuf[j][i] = 0x00;	//将显存数组数据全部清零
		}
	}
}

/**
  * 函    数：将OLED显存数组部分清零
  * 参    数：X 指定区域左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定区域左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Width 指定区域的宽度，范围：0~128
  * 参    数：Height 指定区域的高度，范围：0~64
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ClearArea(int16_t X, int16_t Y, uint8_t Width, uint8_t Height)
{
	int16_t i, j;
	
	for (j = Y; j < Y + Height; j ++)		//遍历指定页
	{
		for (i = X; i < X + Width; i ++)	//遍历指定列
		{
			if (i >= 0 && i <= 127 && j >=0 && j <= 63)				//超出屏幕的内容不显示
			{
				OLED_DisplayBuf[j / 8][i] &= ~(0x01 << (j % 8));	//将显存数组指定数据清零
			}
		}
	}
}

/**
  * 函    数：将OLED显存数组全部取反
  * 参    数：无
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_Reverse(void)
{
	uint8_t i, j;
	for (j = 0; j < 8; j ++)				//遍历8页
	{
		for (i = 0; i < 128; i ++)			//遍历128列
		{
			OLED_DisplayBuf[j][i] ^= 0xFF;	//将显存数组数据全部取反
		}
	}
}
	
/**
  * 函    数：将OLED显存数组部分取反
  * 参    数：X 指定区域左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定区域左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Width 指定区域的宽度，范围：0~128
  * 参    数：Height 指定区域的高度，范围：0~64
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ReverseArea(int16_t X, int16_t Y, uint8_t Width, uint8_t Height)
{
	int16_t i, j;
	
	for (j = Y; j < Y + Height; j ++)		//遍历指定页
	{
		for (i = X; i < X + Width; i ++)	//遍历指定列
		{
			if (i >= 0 && i <= 127 && j >=0 && j <= 63)			//超出屏幕的内容不显示
			{
				OLED_DisplayBuf[j / 8][i] ^= 0x01 << (j % 8);	//将显存数组指定数据取反
			} 
		}
	}
}

/**
  * 函    数：OLED显示一个字符
  * 参    数：X 指定字符左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定字符左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Char 指定要显示的字符，范围：ASCII码可见字符
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowChar(int16_t X, int16_t Y, char Char, uint8_t FontSize)
{
	if (FontSize == OLED_8X16)		//字体为宽8像素，高16像素
	{
		/*将ASCII字模库OLED_F8x16的指定数据以8*16的图像格式显示*/
		OLED_ShowImage(X, Y, 8, 16, OLED_F8x16[Char - ' ']);
	}
	else if(FontSize == OLED_6X8)	//字体为宽6像素，高8像素
	{
		/*将ASCII字模库OLED_F6x8的指定数据以6*8的图像格式显示*/
		OLED_ShowImage(X, Y, 6, 8, OLED_F6x8[Char - ' ']);
	}
}

/**
  * 函    数：OLED显示字符串（支持ASCII码和中文混合写入）
  * 参    数：X 指定字符串左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定字符串左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：String 指定要显示的字符串，范围：ASCII码可见字符或中文字符组成的字符串
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 返 回 值：无
  * 说    明：显示的中文字符需要在OLED_Data.c里的OLED_CF16x16数组定义
  *           未找到指定中文字符时，会显示默认图形（一个方框，内部一个问号）
  *           当字体大小为OLED_8X16时，中文字符以16*16点阵正常显示
  *           当字体大小为OLED_6X8时，中文字符以6*8点阵显示'?'
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowString(int16_t X, int16_t Y, char *String, uint8_t FontSize)
{
	uint16_t i = 0;
	char SingleChar[5];
	uint8_t CharLength = 0;
	uint16_t XOffset = 0;
	uint16_t pIndex;
	
	while (String[i] != '\0')	//遍历字符串
	{
		
#ifdef OLED_CHARSET_UTF8						//定义字符集为UTF8
		/*此段代码的目的是，提取UTF8字符串中的一个字符，转存到SingleChar子字符串中*/
		/*判断UTF8编码第一个字节的标志位*/
		if ((String[i] & 0x80) == 0x00)			//第一个字节为0xxxxxxx
		{
			CharLength = 1;						//字符为1字节
			SingleChar[0] = String[i ++];		//将第一个字节写入SingleChar第0个位置，随后i指向下一个字节
			SingleChar[1] = '\0';				//为SingleChar添加字符串结束标志位
		}
		else if ((String[i] & 0xE0) == 0xC0)	//第一个字节为110xxxxx
		{
			CharLength = 2;						//字符为2字节
			SingleChar[0] = String[i ++];		//将第一个字节写入SingleChar第0个位置，随后i指向下一个字节
			if (String[i] == '\0') {break;}		//意外情况，跳出循环，结束显示
			SingleChar[1] = String[i ++];		//将第二个字节写入SingleChar第1个位置，随后i指向下一个字节
			SingleChar[2] = '\0';				//为SingleChar添加字符串结束标志位
		}
		else if ((String[i] & 0xF0) == 0xE0)	//第一个字节为1110xxxx
		{
			CharLength = 3;						//字符为3字节
			SingleChar[0] = String[i ++];
			if (String[i] == '\0') {break;}
			SingleChar[1] = String[i ++];
			if (String[i] == '\0') {break;}
			SingleChar[2] = String[i ++];
			SingleChar[3] = '\0';
		}
		else if ((String[i] & 0xF8) == 0xF0)	//第一个字节为11110xxx
		{
			CharLength = 4;						//字符为4字节
			SingleChar[0] = String[i ++];
			if (String[i] == '\0') {break;}
			SingleChar[1] = String[i ++];
			if (String[i] == '\0') {break;}
			SingleChar[2] = String[i ++];
			if (String[i] == '\0') {break;}
			SingleChar[3] = String[i ++];
			SingleChar[4] = '\0';
		}
		else
		{
			i ++;			//意外情况，i指向下一个字节，忽略此字节，继续判断下一个字节
			continue;
		}
#endif
		
#ifdef OLED_CHARSET_GB2312						//定义字符集为GB2312
		/*此段代码的目的是，提取GB2312字符串中的一个字符，转存到SingleChar子字符串中*/
		/*判断GB2312字节的最高位标志位*/
		if ((String[i] & 0x80) == 0x00)			//最高位为0
		{
			CharLength = 1;						//字符为1字节
			SingleChar[0] = String[i ++];		//将第一个字节写入SingleChar第0个位置，随后i指向下一个字节
			SingleChar[1] = '\0';				//为SingleChar添加字符串结束标志位
		}
		else									//最高位为1
		{
			CharLength = 2;						//字符为2字节
			SingleChar[0] = String[i ++];		//将第一个字节写入SingleChar第0个位置，随后i指向下一个字节
			if (String[i] == '\0') {break;}		//意外情况，跳出循环，结束显示
			SingleChar[1] = String[i ++];		//将第二个字节写入SingleChar第1个位置，随后i指向下一个字节
			SingleChar[2] = '\0';				//为SingleChar添加字符串结束标志位
		}
#endif
		
		/*显示上述代码提取到的SingleChar*/
		if (CharLength == 1)	//如果是单字节字符
		{
			/*使用OLED_ShowChar显示此字符*/
			OLED_ShowChar(X + XOffset, Y, SingleChar[0], FontSize);
			XOffset += FontSize;
		}
		else					//否则，即多字节字符
		{
			/*遍历整个字模库，从字模库中寻找此字符的数据*/
			/*如果找到最后一个字符（定义为空字符串），则表示字符未在字模库定义，停止寻找*/
			for (pIndex = 0; strcmp(OLED_CF16x16[pIndex].Index, "") != 0; pIndex ++)
			{
				/*找到匹配的字符*/
				if (strcmp(OLED_CF16x16[pIndex].Index, SingleChar) == 0)
				{
					break;		//跳出循环，此时pIndex的值为指定字符的索引
				}
			}
			if (FontSize == OLED_8X16)		//给定字体为8*16点阵
			{
				/*将字模库OLED_CF16x16的指定数据以16*16的图像格式显示*/
				OLED_ShowImage(X + XOffset, Y, 16, 16, OLED_CF16x16[pIndex].Data);
				XOffset += 16;
			}
			else if (FontSize == OLED_6X8)	//给定字体为6*8点阵
			{
				/*空间不足，此位置显示'?'*/
				OLED_ShowChar(X + XOffset, Y, '?', OLED_6X8);
				XOffset += OLED_6X8;
			}
		}
	}
}

/**
  * 函    数：OLED显示数字（十进制，正整数）
  * 参    数：X 指定数字左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定数字左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Number 指定要显示的数字，范围：0~4294967295
  * 参    数：Length 指定数字的长度，范围：0~10
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowNum(int16_t X, int16_t Y, uint32_t Number, uint8_t Length, uint8_t FontSize)
{
	uint8_t i;
	for (i = 0; i < Length; i++)		//遍历数字的每一位							
	{
		/*调用OLED_ShowChar函数，依次显示每个数字*/
		/*Number / OLED_Pow(10, Length - i - 1) % 10 可以十进制提取数字的每一位*/
		/*+ '0' 可将数字转换为字符格式*/
		OLED_ShowChar(X + i * FontSize, Y, Number / OLED_Pow(10, Length - i - 1) % 10 + '0', FontSize);
	}
}

/**
  * 函    数：OLED显示有符号数字（十进制，整数）
  * 参    数：X 指定数字左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定数字左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Number 指定要显示的数字，范围：-2147483648~2147483647
  * 参    数：Length 指定数字的长度，范围：0~10
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowSignedNum(int16_t X, int16_t Y, int32_t Number, uint8_t Length, uint8_t FontSize)
{
	uint8_t i;
	uint32_t Number1;
	
	if (Number >= 0)						//数字大于等于0
	{
		OLED_ShowChar(X, Y, '+', FontSize);	//显示+号
		Number1 = Number;					//Number1直接等于Number
	}
	else									//数字小于0
	{
		OLED_ShowChar(X, Y, '-', FontSize);	//显示-号
		Number1 = -Number;					//Number1等于Number取负
	}
	
	for (i = 0; i < Length; i++)			//遍历数字的每一位								
	{
		/*调用OLED_ShowChar函数，依次显示每个数字*/
		/*Number1 / OLED_Pow(10, Length - i - 1) % 10 可以十进制提取数字的每一位*/
		/*+ '0' 可将数字转换为字符格式*/
		OLED_ShowChar(X + (i + 1) * FontSize, Y, Number1 / OLED_Pow(10, Length - i - 1) % 10 + '0', FontSize);
	}
}

/**
  * 函    数：OLED显示十六进制数字（十六进制，正整数）
  * 参    数：X 指定数字左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定数字左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Number 指定要显示的数字，范围：0x00000000~0xFFFFFFFF
  * 参    数：Length 指定数字的长度，范围：0~8
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowHexNum(int16_t X, int16_t Y, uint32_t Number, uint8_t Length, uint8_t FontSize)
{
	uint8_t i, SingleNumber;
	for (i = 0; i < Length; i++)		//遍历数字的每一位
	{
		/*以十六进制提取数字的每一位*/
		SingleNumber = Number / OLED_Pow(16, Length - i - 1) % 16;
		
		if (SingleNumber < 10)			//单个数字小于10
		{
			/*调用OLED_ShowChar函数，显示此数字*/
			/*+ '0' 可将数字转换为字符格式*/
			OLED_ShowChar(X + i * FontSize, Y, SingleNumber + '0', FontSize);
		}
		else							//单个数字大于10
		{
			/*调用OLED_ShowChar函数，显示此数字*/
			/*+ 'A' 可将数字转换为从A开始的十六进制字符*/
			OLED_ShowChar(X + i * FontSize, Y, SingleNumber - 10 + 'A', FontSize);
		}
	}
}

/**
  * 函    数：OLED显示二进制数字（二进制，正整数）
  * 参    数：X 指定数字左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定数字左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Number 指定要显示的数字，范围：0x00000000~0xFFFFFFFF
  * 参    数：Length 指定数字的长度，范围：0~16
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowBinNum(int16_t X, int16_t Y, uint32_t Number, uint8_t Length, uint8_t FontSize)
{
	uint8_t i;
	for (i = 0; i < Length; i++)		//遍历数字的每一位	
	{
		/*调用OLED_ShowChar函数，依次显示每个数字*/
		/*Number / OLED_Pow(2, Length - i - 1) % 2 可以二进制提取数字的每一位*/
		/*+ '0' 可将数字转换为字符格式*/
		OLED_ShowChar(X + i * FontSize, Y, Number / OLED_Pow(2, Length - i - 1) % 2 + '0', FontSize);
	}
}

/**
  * 函    数：OLED显示浮点数字（十进制，小数）
  * 参    数：X 指定数字左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定数字左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Number 指定要显示的数字，范围：-4294967295.0~4294967295.0
  * 参    数：IntLength 指定数字的整数位长度，范围：0~10
  * 参    数：FraLength 指定数字的小数位长度，范围：0~9，小数进行四舍五入显示
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowFloatNum(int16_t X, int16_t Y, double Number, uint8_t IntLength, uint8_t FraLength, uint8_t FontSize)
{
	uint32_t PowNum, IntNum, FraNum;
	
	if (Number >= 0)						//数字大于等于0
	{
		OLED_ShowChar(X, Y, '+', FontSize);	//显示+号
	}
	else									//数字小于0
	{
		OLED_ShowChar(X, Y, '-', FontSize);	//显示-号
		Number = -Number;					//Number取负
	}
	
	/*提取整数部分和小数部分*/
	IntNum = Number;						//直接赋值给整型变量，提取整数
	Number -= IntNum;						//将Number的整数减掉，防止之后将小数乘到整数时因数过大造成错误
	PowNum = OLED_Pow(10, FraLength);		//根据指定小数的位数，确定乘数
	FraNum = round(Number * PowNum);		//将小数乘到整数，同时四舍五入，避免显示误差
	IntNum += FraNum / PowNum;				//若四舍五入造成了进位，则需要再加给整数
	
	/*显示整数部分*/
	OLED_ShowNum(X + FontSize, Y, IntNum, IntLength, FontSize);
	
	/*显示小数点*/
	OLED_ShowChar(X + (IntLength + 1) * FontSize, Y, '.', FontSize);
	
	/*显示小数部分*/
	OLED_ShowNum(X + (IntLength + 2) * FontSize, Y, FraNum, FraLength, FontSize);
}

/**
  * 函    数：OLED显示图像
  * 参    数：X 指定图像左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定图像左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Width 指定图像的宽度，范围：0~128
  * 参    数：Height 指定图像的高度，范围：0~64
  * 参    数：Image 指定要显示的图像
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_ShowImage(int16_t X, int16_t Y, uint8_t Width, uint8_t Height, const uint8_t *Image)
{
	uint8_t i = 0, j = 0;
	int16_t Page, Shift;
	
	/*将图像所在区域清空*/
	OLED_ClearArea(X, Y, Width, Height);
	
	/*遍历指定图像涉及的相关页*/
	/*(Height - 1) / 8 + 1的目的是Height / 8并向上取整*/
	for (j = 0; j < (Height - 1) / 8 + 1; j ++)
	{
		/*遍历指定图像涉及的相关列*/
		for (i = 0; i < Width; i ++)
		{
			if (X + i >= 0 && X + i <= 127)		//超出屏幕的内容不显示
			{
				/*负数坐标在计算页地址和移位时需要加一个偏移*/
				Page = Y / 8;
				Shift = Y % 8;
				if (Y < 0)
				{
					Page -= 1;
					Shift += 8;
				}
				
				if (Page + j >= 0 && Page + j <= 7)		//超出屏幕的内容不显示
				{
					/*显示图像在当前页的内容*/
					OLED_DisplayBuf[Page + j][X + i] |= Image[j * Width + i] << (Shift);
				}
				
				if (Page + j + 1 >= 0 && Page + j + 1 <= 7)		//超出屏幕的内容不显示
				{					
					/*显示图像在下一页的内容*/
					OLED_DisplayBuf[Page + j + 1][X + i] |= Image[j * Width + i] >> (8 - Shift);
				}
			}
		}
	}
}

/**
  * 函    数：OLED使用printf函数打印格式化字符串（支持ASCII码和中文混合写入）
  * 参    数：X 指定格式化字符串左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定格式化字符串左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：FontSize 指定字体大小
  *           范围：OLED_8X16		宽8像素，高16像素
  *                 OLED_6X8		宽6像素，高8像素
  * 参    数：format 指定要显示的格式化字符串，范围：ASCII码可见字符或中文字符组成的字符串
  * 参    数：... 格式化字符串参数列表
  * 返 回 值：无
  * 说    明：显示的中文字符需要在OLED_Data.c里的OLED_CF16x16数组定义
  *           未找到指定中文字符时，会显示默认图形（一个方框，内部一个问号）
  *           当字体大小为OLED_8X16时，中文字符以16*16点阵正常显示
  *           当字体大小为OLED_6X8时，中文字符以6*8点阵显示'?'
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_Printf(int16_t X, int16_t Y, uint8_t FontSize, char *format, ...)
{
	char String[256];						//定义字符数组
	va_list arg;							//定义可变参数列表数据类型的变量arg
	va_start(arg, format);					//从format开始，接收参数列表到arg变量
	vsprintf(String, format, arg);			//使用vsprintf打印格式化字符串和参数列表到字符数组中
	va_end(arg);							//结束变量arg
	OLED_ShowString(X, Y, String, FontSize);//OLED显示字符数组（字符串）
}

/**
  * 函    数：OLED在指定位置画一个点
  * 参    数：X 指定点的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定点的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_DrawPoint(int16_t X, int16_t Y)
{
	if (X >= 0 && X <= 127 && Y >=0 && Y <= 63)		//超出屏幕的内容不显示
	{
		/*将显存数组指定位置的一个Bit数据置1*/
		OLED_DisplayBuf[Y / 8][X] |= 0x01 << (Y % 8);
	}
}

/**
  * 函    数：OLED获取指定位置点的值
  * 参    数：X 指定点的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定点的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 返 回 值：指定位置点是否处于点亮状态，1：点亮，0：熄灭
  */
uint8_t OLED_GetPoint(int16_t X, int16_t Y)
{
	if (X >= 0 && X <= 127 && Y >=0 && Y <= 63)		//超出屏幕的内容不读取
	{
		/*判断指定位置的数据*/
		if (OLED_DisplayBuf[Y / 8][X] & 0x01 << (Y % 8))
		{
			return 1;	//为1，返回1
		}
	}
	
	return 0;		//否则，返回0
}

/**
  * 函    数：OLED画线
  * 参    数：X0 指定一个端点的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y0 指定一个端点的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：X1 指定另一个端点的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y1 指定另一个端点的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_DrawLine(int16_t X0, int16_t Y0, int16_t X1, int16_t Y1)
{
	int16_t x, y, dx, dy, d, incrE, incrNE, temp;
	int16_t x0 = X0, y0 = Y0, x1 = X1, y1 = Y1;
	uint8_t yflag = 0, xyflag = 0;
	
	if (y0 == y1)		//横线单独处理
	{
		/*0号点X坐标大于1号点X坐标，则交换两点X坐标*/
		if (x0 > x1) {temp = x0; x0 = x1; x1 = temp;}
		
		/*遍历X坐标*/
		for (x = x0; x <= x1; x ++)
		{
			OLED_DrawPoint(x, y0);	//依次画点
		}
	}
	else if (x0 == x1)	//竖线单独处理
	{
		/*0号点Y坐标大于1号点Y坐标，则交换两点Y坐标*/
		if (y0 > y1) {temp = y0; y0 = y1; y1 = temp;}
		
		/*遍历Y坐标*/
		for (y = y0; y <= y1; y ++)
		{
			OLED_DrawPoint(x0, y);	//依次画点
		}
	}
	else				//斜线
	{
		/*使用Bresenham算法画直线，可以避免耗时的浮点运算，效率更高*/
		/*参考文档：https://www.cs.montana.edu/courses/spring2009/425/dslectures/Bresenham.pdf*/
		/*参考教程：https://www.bilibili.com/video/BV1364y1d7Lo*/
		
		if (x0 > x1)	//0号点X坐标大于1号点X坐标
		{
			/*交换两点坐标*/
			/*交换后不影响画线，但是画线方向由第一、二、三、四象限变为第一、四象限*/
			temp = x0; x0 = x1; x1 = temp;
			temp = y0; y0 = y1; y1 = temp;
		}
		
		if (y0 > y1)	//0号点Y坐标大于1号点Y坐标
		{
			/*将Y坐标取负*/
			/*取负后影响画线，但是画线方向由第一、四象限变为第一象限*/
			y0 = -y0;
			y1 = -y1;
			
			/*置标志位yflag，记住当前变换，在后续实际画线时，再将坐标换回来*/
			yflag = 1;
		}
		
		if (y1 - y0 > x1 - x0)	//画线斜率大于1
		{
			/*将X坐标与Y坐标互换*/
			/*互换后影响画线，但是画线方向由第一象限0~90度范围变为第一象限0~45度范围*/
			temp = x0; x0 = y0; y0 = temp;
			temp = x1; x1 = y1; y1 = temp;
			
			/*置标志位xyflag，记住当前变换，在后续实际画线时，再将坐标换回来*/
			xyflag = 1;
		}
		
		/*以下为Bresenham算法画直线*/
		/*算法要求，画线方向必须为第一象限0~45度范围*/
		dx = x1 - x0;
		dy = y1 - y0;
		incrE = 2 * dy;
		incrNE = 2 * (dy - dx);
		d = 2 * dy - dx;
		x = x0;
		y = y0;
		
		/*画起始点，同时判断标志位，将坐标换回来*/
		if (yflag && xyflag){OLED_DrawPoint(y, -x);}
		else if (yflag)		{OLED_DrawPoint(x, -y);}
		else if (xyflag)	{OLED_DrawPoint(y, x);}
		else				{OLED_DrawPoint(x, y);}
		
		while (x < x1)		//遍历X轴的每个点
		{
			x ++;
			if (d < 0)		//下一个点在当前点东方
			{
				d += incrE;
			}
			else			//下一个点在当前点东北方
			{
				y ++;
				d += incrNE;
			}
			
			/*画每一个点，同时判断标志位，将坐标换回来*/
			if (yflag && xyflag){OLED_DrawPoint(y, -x);}
			else if (yflag)		{OLED_DrawPoint(x, -y);}
			else if (xyflag)	{OLED_DrawPoint(y, x);}
			else				{OLED_DrawPoint(x, y);}
		}	
	}
}

/**
  * 函    数：OLED矩形
  * 参    数：X 指定矩形左上角的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定矩形左上角的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Width 指定矩形的宽度，范围：0~128
  * 参    数：Height 指定矩形的高度，范围：0~64
  * 参    数：IsFilled 指定矩形是否填充
  *           范围：OLED_UNFILLED		不填充
  *                 OLED_FILLED			填充
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_DrawRectangle(int16_t X, int16_t Y, uint8_t Width, uint8_t Height, uint8_t IsFilled)
{
	int16_t i, j;
	if (!IsFilled)		//指定矩形不填充
	{
		/*遍历上下X坐标，画矩形上下两条线*/
		for (i = X; i < X + Width; i ++)
		{
			OLED_DrawPoint(i, Y);
			OLED_DrawPoint(i, Y + Height - 1);
		}
		/*遍历左右Y坐标，画矩形左右两条线*/
		for (i = Y; i < Y + Height; i ++)
		{
			OLED_DrawPoint(X, i);
			OLED_DrawPoint(X + Width - 1, i);
		}
	}
	else				//指定矩形填充
	{
		/*遍历X坐标*/
		for (i = X; i < X + Width; i ++)
		{
			/*遍历Y坐标*/
			for (j = Y; j < Y + Height; j ++)
			{
				/*在指定区域画点，填充满矩形*/
				OLED_DrawPoint(i, j);
			}
		}
	}
}

/**
  * 函    数：OLED三角形
  * 参    数：X0 指定第一个端点的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y0 指定第一个端点的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：X1 指定第二个端点的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y1 指定第二个端点的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：X2 指定第三个端点的横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y2 指定第三个端点的纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：IsFilled 指定三角形是否填充
  *           范围：OLED_UNFILLED		不填充
  *                 OLED_FILLED			填充
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_DrawTriangle(int16_t X0, int16_t Y0, int16_t X1, int16_t Y1, int16_t X2, int16_t Y2, uint8_t IsFilled)
{
	int16_t minx = X0, miny = Y0, maxx = X0, maxy = Y0;
	int16_t i, j;
	int16_t vx[] = {X0, X1, X2};
	int16_t vy[] = {Y0, Y1, Y2};
	
	if (!IsFilled)			//指定三角形不填充
	{
		/*调用画线函数，将三个点用直线连接*/
		OLED_DrawLine(X0, Y0, X1, Y1);
		OLED_DrawLine(X0, Y0, X2, Y2);
		OLED_DrawLine(X1, Y1, X2, Y2);
	}
	else					//指定三角形填充
	{
		/*找到三个点最小的X、Y坐标*/
		if (X1 < minx) {minx = X1;}
		if (X2 < minx) {minx = X2;}
		if (Y1 < miny) {miny = Y1;}
		if (Y2 < miny) {miny = Y2;}
		
		/*找到三个点最大的X、Y坐标*/
		if (X1 > maxx) {maxx = X1;}
		if (X2 > maxx) {maxx = X2;}
		if (Y1 > maxy) {maxy = Y1;}
		if (Y2 > maxy) {maxy = Y2;}
		
		/*最小最大坐标之间的矩形为可能需要填充的区域*/
		/*遍历此区域中所有的点*/
		/*遍历X坐标*/		
		for (i = minx; i <= maxx; i ++)
		{
			/*遍历Y坐标*/	
			for (j = miny; j <= maxy; j ++)
			{
				/*调用OLED_pnpoly，判断指定点是否在指定三角形之中*/
				/*如果在，则画点，如果不在，则不做处理*/
				if (OLED_pnpoly(3, vx, vy, i, j)) {OLED_DrawPoint(i, j);}
			}
		}
	}
}

/**
  * 函    数：OLED画圆
  * 参    数：X 指定圆的圆心横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定圆的圆心纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Radius 指定圆的半径，范围：0~255
  * 参    数：IsFilled 指定圆是否填充
  *           范围：OLED_UNFILLED		不填充
  *                 OLED_FILLED			填充
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_DrawCircle(int16_t X, int16_t Y, uint8_t Radius, uint8_t IsFilled)
{
	int16_t x, y, d, j;
	
	/*使用Bresenham算法画圆，可以避免耗时的浮点运算，效率更高*/
	/*参考文档：https://www.cs.montana.edu/courses/spring2009/425/dslectures/Bresenham.pdf*/
	/*参考教程：https://www.bilibili.com/video/BV1VM4y1u7wJ*/
	
	d = 1 - Radius;
	x = 0;
	y = Radius;
	
	/*画每个八分之一圆弧的起始点*/
	OLED_DrawPoint(X + x, Y + y);
	OLED_DrawPoint(X - x, Y - y);
	OLED_DrawPoint(X + y, Y + x);
	OLED_DrawPoint(X - y, Y - x);
	
	if (IsFilled)		//指定圆填充
	{
		/*遍历起始点Y坐标*/
		for (j = -y; j < y; j ++)
		{
			/*在指定区域画点，填充部分圆*/
			OLED_DrawPoint(X, Y + j);
		}
	}
	
	while (x < y)		//遍历X轴的每个点
	{
		x ++;
		if (d < 0)		//下一个点在当前点东方
		{
			d += 2 * x + 1;
		}
		else			//下一个点在当前点东南方
		{
			y --;
			d += 2 * (x - y) + 1;
		}
		
		/*画每个八分之一圆弧的点*/
		OLED_DrawPoint(X + x, Y + y);
		OLED_DrawPoint(X + y, Y + x);
		OLED_DrawPoint(X - x, Y - y);
		OLED_DrawPoint(X - y, Y - x);
		OLED_DrawPoint(X + x, Y - y);
		OLED_DrawPoint(X + y, Y - x);
		OLED_DrawPoint(X - x, Y + y);
		OLED_DrawPoint(X - y, Y + x);
		
		if (IsFilled)	//指定圆填充
		{
			/*遍历中间部分*/
			for (j = -y; j < y; j ++)
			{
				/*在指定区域画点，填充部分圆*/
				OLED_DrawPoint(X + x, Y + j);
				OLED_DrawPoint(X - x, Y + j);
			}
			
			/*遍历两侧部分*/
			for (j = -x; j < x; j ++)
			{
				/*在指定区域画点，填充部分圆*/
				OLED_DrawPoint(X - y, Y + j);
				OLED_DrawPoint(X + y, Y + j);
			}
		}
	}
}

/**
  * 函    数：OLED画椭圆
  * 参    数：X 指定椭圆的圆心横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定椭圆的圆心纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：A 指定椭圆的横向半轴长度，范围：0~255
  * 参    数：B 指定椭圆的纵向半轴长度，范围：0~255
  * 参    数：IsFilled 指定椭圆是否填充
  *           范围：OLED_UNFILLED		不填充
  *                 OLED_FILLED			填充
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_DrawEllipse(int16_t X, int16_t Y, uint8_t A, uint8_t B, uint8_t IsFilled)
{
	int16_t x, y, j;
	int16_t a = A, b = B;
	float d1, d2;
	
	/*使用Bresenham算法画椭圆，可以避免部分耗时的浮点运算，效率更高*/
	/*参考链接：https://blog.csdn.net/myf_666/article/details/128167392*/
	
	x = 0;
	y = b;
	d1 = b * b + a * a * (-b + 0.5);
	
	if (IsFilled)	//指定椭圆填充
	{
		/*遍历起始点Y坐标*/
		for (j = -y; j < y; j ++)
		{
			/*在指定区域画点，填充部分椭圆*/
			OLED_DrawPoint(X, Y + j);
			OLED_DrawPoint(X, Y + j);
		}
	}
	
	/*画椭圆弧的起始点*/
	OLED_DrawPoint(X + x, Y + y);
	OLED_DrawPoint(X - x, Y - y);
	OLED_DrawPoint(X - x, Y + y);
	OLED_DrawPoint(X + x, Y - y);
	
	/*画椭圆中间部分*/
	while (b * b * (x + 1) < a * a * (y - 0.5))
	{
		if (d1 <= 0)		//下一个点在当前点东方
		{
			d1 += b * b * (2 * x + 3);
		}
		else				//下一个点在当前点东南方
		{
			d1 += b * b * (2 * x + 3) + a * a * (-2 * y + 2);
			y --;
		}
		x ++;
		
		if (IsFilled)	//指定椭圆填充
		{
			/*遍历中间部分*/
			for (j = -y; j < y; j ++)
			{
				/*在指定区域画点，填充部分椭圆*/
				OLED_DrawPoint(X + x, Y + j);
				OLED_DrawPoint(X - x, Y + j);
			}
		}
		
		/*画椭圆中间部分圆弧*/
		OLED_DrawPoint(X + x, Y + y);
		OLED_DrawPoint(X - x, Y - y);
		OLED_DrawPoint(X - x, Y + y);
		OLED_DrawPoint(X + x, Y - y);
	}
	
	/*画椭圆两侧部分*/
	d2 = b * b * (x + 0.5) * (x + 0.5) + a * a * (y - 1) * (y - 1) - a * a * b * b;
	
	while (y > 0)
	{
		if (d2 <= 0)		//下一个点在当前点东方
		{
			d2 += b * b * (2 * x + 2) + a * a * (-2 * y + 3);
			x ++;
			
		}
		else				//下一个点在当前点东南方
		{
			d2 += a * a * (-2 * y + 3);
		}
		y --;
		
		if (IsFilled)	//指定椭圆填充
		{
			/*遍历两侧部分*/
			for (j = -y; j < y; j ++)
			{
				/*在指定区域画点，填充部分椭圆*/
				OLED_DrawPoint(X + x, Y + j);
				OLED_DrawPoint(X - x, Y + j);
			}
		}
		
		/*画椭圆两侧部分圆弧*/
		OLED_DrawPoint(X + x, Y + y);
		OLED_DrawPoint(X - x, Y - y);
		OLED_DrawPoint(X - x, Y + y);
		OLED_DrawPoint(X + x, Y - y);
	}
}

/**
  * 函    数：OLED画圆弧
  * 参    数：X 指定圆弧的圆心横坐标，范围：-32768~32767，屏幕区域：0~127
  * 参    数：Y 指定圆弧的圆心纵坐标，范围：-32768~32767，屏幕区域：0~63
  * 参    数：Radius 指定圆弧的半径，范围：0~255
  * 参    数：StartAngle 指定圆弧的起始角度，范围：-180~180
  *           水平向右为0度，水平向左为180度或-180度，下方为正数，上方为负数，顺时针旋转
  * 参    数：EndAngle 指定圆弧的终止角度，范围：-180~180
  *           水平向右为0度，水平向左为180度或-180度，下方为正数，上方为负数，顺时针旋转
  * 参    数：IsFilled 指定圆弧是否填充，填充后为扇形
  *           范围：OLED_UNFILLED		不填充
  *                 OLED_FILLED			填充
  * 返 回 值：无
  * 说    明：调用此函数后，要想真正地呈现在屏幕上，还需调用更新函数
  */
void OLED_DrawArc(int16_t X, int16_t Y, uint8_t Radius, int16_t StartAngle, int16_t EndAngle, uint8_t IsFilled)
{
	int16_t x, y, d, j;
	
	/*此函数借用Bresenham算法画圆的方法*/
	
	d = 1 - Radius;
	x = 0;
	y = Radius;
	
	/*在画圆的每个点时，判断指定点是否在指定角度内，在，则画点，不在，则不做处理*/
	if (OLED_IsInAngle(x, y, StartAngle, EndAngle))	{OLED_DrawPoint(X + x, Y + y);}
	if (OLED_IsInAngle(-x, -y, StartAngle, EndAngle)) {OLED_DrawPoint(X - x, Y - y);}
	if (OLED_IsInAngle(y, x, StartAngle, EndAngle)) {OLED_DrawPoint(X + y, Y + x);}
	if (OLED_IsInAngle(-y, -x, StartAngle, EndAngle)) {OLED_DrawPoint(X - y, Y - x);}
	
	if (IsFilled)	//指定圆弧填充
	{
		/*遍历起始点Y坐标*/
		for (j = -y; j < y; j ++)
		{
			/*在填充圆的每个点时，判断指定点是否在指定角度内，在，则画点，不在，则不做处理*/
			if (OLED_IsInAngle(0, j, StartAngle, EndAngle)) {OLED_DrawPoint(X, Y + j);}
		}
	}
	
	while (x < y)		//遍历X轴的每个点
	{
		x ++;
		if (d < 0)		//下一个点在当前点东方
		{
			d += 2 * x + 1;
		}
		else			//下一个点在当前点东南方
		{
			y --;
			d += 2 * (x - y) + 1;
		}
		
		/*在画圆的每个点时，判断指定点是否在指定角度内，在，则画点，不在，则不做处理*/
		if (OLED_IsInAngle(x, y, StartAngle, EndAngle)) {OLED_DrawPoint(X + x, Y + y);}
		if (OLED_IsInAngle(y, x, StartAngle, EndAngle)) {OLED_DrawPoint(X + y, Y + x);}
		if (OLED_IsInAngle(-x, -y, StartAngle, EndAngle)) {OLED_DrawPoint(X - x, Y - y);}
		if (OLED_IsInAngle(-y, -x, StartAngle, EndAngle)) {OLED_DrawPoint(X - y, Y - x);}
		if (OLED_IsInAngle(x, -y, StartAngle, EndAngle)) {OLED_DrawPoint(X + x, Y - y);}
		if (OLED_IsInAngle(y, -x, StartAngle, EndAngle)) {OLED_DrawPoint(X + y, Y - x);}
		if (OLED_IsInAngle(-x, y, StartAngle, EndAngle)) {OLED_DrawPoint(X - x, Y + y);}
		if (OLED_IsInAngle(-y, x, StartAngle, EndAngle)) {OLED_DrawPoint(X - y, Y + x);}
		
		if (IsFilled)	//指定圆弧填充
		{
			/*遍历中间部分*/
			for (j = -y; j < y; j ++)
			{
				/*在填充圆的每个点时，判断指定点是否在指定角度内，在，则画点，不在，则不做处理*/
				if (OLED_IsInAngle(x, j, StartAngle, EndAngle)) {OLED_DrawPoint(X + x, Y + j);}
				if (OLED_IsInAngle(-x, j, StartAngle, EndAngle)) {OLED_DrawPoint(X - x, Y + j);}
			}
			
			/*遍历两侧部分*/
			for (j = -x; j < x; j ++)
			{
				/*在填充圆的每个点时，判断指定点是否在指定角度内，在，则画点，不在，则不做处理*/
				if (OLED_IsInAngle(-y, j, StartAngle, EndAngle)) {OLED_DrawPoint(X - y, Y + j);}
				if (OLED_IsInAngle(y, j, StartAngle, EndAngle)) {OLED_DrawPoint(X + y, Y + j);}
			}
		}
	}
}

/*********************功能函数*/


/*****************江协科技|版权所有****************/
/*****************jiangxiekeji.com*****************/

```

## OLED.h

```c
#ifndef __OLED_H
#define __OLED_H

#include <stdint.h>
#include "OLED_Data.h"

/*参数宏定义*********************/

/*FontSize参数取值*/
/*此参数值不仅用于判断，而且用于计算横向字符偏移，默认值为字体像素宽度*/
#define OLED_8X16				8
#define OLED_6X8				6

/*IsFilled参数数值*/
#define OLED_UNFILLED			0
#define OLED_FILLED				1

/*********************参数宏定义*/


/*函数声明*********************/

/*初始化函数*/
void OLED_Init(void);

/*更新函数*/
void OLED_Update(void);
void OLED_UpdateArea(int16_t X, int16_t Y, uint8_t Width, uint8_t Height);

/*显存控制函数*/
void OLED_Clear(void);
void OLED_ClearArea(int16_t X, int16_t Y, uint8_t Width, uint8_t Height);
void OLED_Reverse(void);
void OLED_ReverseArea(int16_t X, int16_t Y, uint8_t Width, uint8_t Height);

/*显示函数*/
void OLED_ShowChar(int16_t X, int16_t Y, char Char, uint8_t FontSize);
void OLED_ShowString(int16_t X, int16_t Y, char *String, uint8_t FontSize);
void OLED_ShowNum(int16_t X, int16_t Y, uint32_t Number, uint8_t Length, uint8_t FontSize);
void OLED_ShowSignedNum(int16_t X, int16_t Y, int32_t Number, uint8_t Length, uint8_t FontSize);
void OLED_ShowHexNum(int16_t X, int16_t Y, uint32_t Number, uint8_t Length, uint8_t FontSize);
void OLED_ShowBinNum(int16_t X, int16_t Y, uint32_t Number, uint8_t Length, uint8_t FontSize);
void OLED_ShowFloatNum(int16_t X, int16_t Y, double Number, uint8_t IntLength, uint8_t FraLength, uint8_t FontSize);
void OLED_ShowImage(int16_t X, int16_t Y, uint8_t Width, uint8_t Height, const uint8_t *Image);
void OLED_Printf(int16_t X, int16_t Y, uint8_t FontSize, char *format, ...);

/*绘图函数*/
void OLED_DrawPoint(int16_t X, int16_t Y);
uint8_t OLED_GetPoint(int16_t X, int16_t Y);
void OLED_DrawLine(int16_t X0, int16_t Y0, int16_t X1, int16_t Y1);
void OLED_DrawRectangle(int16_t X, int16_t Y, uint8_t Width, uint8_t Height, uint8_t IsFilled);
void OLED_DrawTriangle(int16_t X0, int16_t Y0, int16_t X1, int16_t Y1, int16_t X2, int16_t Y2, uint8_t IsFilled);
void OLED_DrawCircle(int16_t X, int16_t Y, uint8_t Radius, uint8_t IsFilled);
void OLED_DrawEllipse(int16_t X, int16_t Y, uint8_t A, uint8_t B, uint8_t IsFilled);
void OLED_DrawArc(int16_t X, int16_t Y, uint8_t Radius, int16_t StartAngle, int16_t EndAngle, uint8_t IsFilled);

/*********************函数声明*/

#endif


/*****************江协科技|版权所有****************/
/*****************jiangxiekeji.com*****************/

```

## OLED_Data.c

```c
#include "OLED_Data.h"

/**
  * 数据存储格式：
  * 纵向8点，高位在下，先从左到右，再从上到下
  * 每一个Bit对应一个像素点
  * 
  *      B0 B0                  B0 B0
  *      B1 B1                  B1 B1
  *      B2 B2                  B2 B2
  *      B3 B3  ------------->  B3 B3 --
  *      B4 B4                  B4 B4  |
  *      B5 B5                  B5 B5  |
  *      B6 B6                  B6 B6  |
  *      B7 B7                  B7 B7  |
  *                                    |
  *  -----------------------------------
  *  |   
  *  |   B0 B0                  B0 B0
  *  |   B1 B1                  B1 B1
  *  |   B2 B2                  B2 B2
  *  --> B3 B3  ------------->  B3 B3
  *      B4 B4                  B4 B4
  *      B5 B5                  B5 B5
  *      B6 B6                  B6 B6
  *      B7 B7                  B7 B7
  * 
  */

/*ASCII字模数据*********************/

/*宽8像素，高16像素*/
const uint8_t OLED_F8x16[][16] =
{
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,//   0
	0x00,0x00,0x00,0xF8,0x00,0x00,0x00,0x00,
	0x00,0x00,0x00,0x33,0x30,0x00,0x00,0x00,// ! 1
	0x00,0x16,0x0E,0x00,0x16,0x0E,0x00,0x00,
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,// " 2
	0x40,0xC0,0x78,0x40,0xC0,0x78,0x40,0x00,
	0x04,0x3F,0x04,0x04,0x3F,0x04,0x04,0x00,// # 3
	0x00,0x70,0x88,0xFC,0x08,0x30,0x00,0x00,
	0x00,0x18,0x20,0xFF,0x21,0x1E,0x00,0x00,// $ 4
	0xF0,0x08,0xF0,0x00,0xE0,0x18,0x00,0x00,
	0x00,0x21,0x1C,0x03,0x1E,0x21,0x1E,0x00,// % 5
	0x00,0xF0,0x08,0x88,0x70,0x00,0x00,0x00,
	0x1E,0x21,0x23,0x24,0x19,0x27,0x21,0x10,// & 6
	0x00,0x00,0x00,0x16,0x0E,0x00,0x00,0x00,
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,// ' 7
	0x00,0x00,0x00,0xE0,0x18,0x04,0x02,0x00,
	0x00,0x00,0x00,0x07,0x18,0x20,0x40,0x00,// ( 8
	0x00,0x02,0x04,0x18,0xE0,0x00,0x00,0x00,
	0x00,0x40,0x20,0x18,0x07,0x00,0x00,0x00,// ) 9
	0x40,0x40,0x80,0xF0,0x80,0x40,0x40,0x00,
	0x02,0x02,0x01,0x0F,0x01,0x02,0x02,0x00,// * 10
	0x00,0x00,0x00,0xF0,0x00,0x00,0x00,0x00,
	0x01,0x01,0x01,0x1F,0x01,0x01,0x01,0x00,// + 11
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	0x00,0xB0,0x70,0x00,0x00,0x00,0x00,0x00,// , 12
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	0x00,0x01,0x01,0x01,0x01,0x01,0x01,0x01,// - 13
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	0x00,0x30,0x30,0x00,0x00,0x00,0x00,0x00,// . 14
	0x00,0x00,0x00,0x00,0x80,0x60,0x18,0x04,
	0x00,0x60,0x18,0x06,0x01,0x00,0x00,0x00,// / 15
	0x00,0xE0,0x10,0x08,0x08,0x10,0xE0,0x00,
	0x00,0x0F,0x10,0x20,0x20,0x10,0x0F,0x00,// 0 16
	0x00,0x10,0x10,0xF8,0x00,0x00,0x00,0x00,
	0x00,0x20,0x20,0x3F,0x20,0x20,0x00,0x00,// 1 17
	0x00,0x70,0x08,0x08,0x08,0x88,0x70,0x00,
	0x00,0x30,0x28,0x24,0x22,0x21,0x30,0x00,// 2 18
	0x00,0x30,0x08,0x88,0x88,0x48,0x30,0x00,
	0x00,0x18,0x20,0x20,0x20,0x11,0x0E,0x00,// 3 19
	0x00,0x00,0xC0,0x20,0x10,0xF8,0x00,0x00,
	0x00,0x07,0x04,0x24,0x24,0x3F,0x24,0x00,// 4 20
	0x00,0xF8,0x08,0x88,0x88,0x08,0x08,0x00,
	0x00,0x19,0x21,0x20,0x20,0x11,0x0E,0x00,// 5 21
	0x00,0xE0,0x10,0x88,0x88,0x18,0x00,0x00,
	0x00,0x0F,0x11,0x20,0x20,0x11,0x0E,0x00,// 6 22
	0x00,0x38,0x08,0x08,0xC8,0x38,0x08,0x00,
	0x00,0x00,0x00,0x3F,0x00,0x00,0x00,0x00,// 7 23
	0x00,0x70,0x88,0x08,0x08,0x88,0x70,0x00,
	0x00,0x1C,0x22,0x21,0x21,0x22,0x1C,0x00,// 8 24
	0x00,0xE0,0x10,0x08,0x08,0x10,0xE0,0x00,
	0x00,0x00,0x31,0x22,0x22,0x11,0x0F,0x00,// 9 25
	0x00,0x00,0x00,0xC0,0xC0,0x00,0x00,0x00,
	0x00,0x00,0x00,0x30,0x30,0x00,0x00,0x00,// : 26
	0x00,0x00,0x00,0xC0,0xC0,0x00,0x00,0x00,
	0x00,0x00,0x80,0xB0,0x70,0x00,0x00,0x00,// ; 27
	0x00,0x00,0x80,0x40,0x20,0x10,0x08,0x00,
	0x00,0x01,0x02,0x04,0x08,0x10,0x20,0x00,// < 28
	0x40,0x40,0x40,0x40,0x40,0x40,0x40,0x00,
	0x04,0x04,0x04,0x04,0x04,0x04,0x04,0x00,// = 29
	0x00,0x08,0x10,0x20,0x40,0x80,0x00,0x00,
	0x00,0x20,0x10,0x08,0x04,0x02,0x01,0x00,// > 30
	0x00,0x70,0x48,0x08,0x08,0x08,0xF0,0x00,
	0x00,0x00,0x00,0x30,0x36,0x01,0x00,0x00,// ? 31
	0xC0,0x30,0xC8,0x28,0xE8,0x10,0xE0,0x00,
	0x07,0x18,0x27,0x24,0x23,0x14,0x0B,0x00,// @ 32
	0x00,0x00,0xC0,0x38,0xE0,0x00,0x00,0x00,
	0x20,0x3C,0x23,0x02,0x02,0x27,0x38,0x20,// A 33
	0x08,0xF8,0x88,0x88,0x88,0x70,0x00,0x00,
	0x20,0x3F,0x20,0x20,0x20,0x11,0x0E,0x00,// B 34
	0xC0,0x30,0x08,0x08,0x08,0x08,0x38,0x00,
	0x07,0x18,0x20,0x20,0x20,0x10,0x08,0x00,// C 35
	0x08,0xF8,0x08,0x08,0x08,0x10,0xE0,0x00,
	0x20,0x3F,0x20,0x20,0x20,0x10,0x0F,0x00,// D 36
	0x08,0xF8,0x88,0x88,0xE8,0x08,0x10,0x00,
	0x20,0x3F,0x20,0x20,0x23,0x20,0x18,0x00,// E 37
	0x08,0xF8,0x88,0x88,0xE8,0x08,0x10,0x00,
	0x20,0x3F,0x20,0x00,0x03,0x00,0x00,0x00,// F 38
	0xC0,0x30,0x08,0x08,0x08,0x38,0x00,0x00,
	0x07,0x18,0x20,0x20,0x22,0x1E,0x02,0x00,// G 39
	0x08,0xF8,0x08,0x00,0x00,0x08,0xF8,0x08,
	0x20,0x3F,0x21,0x01,0x01,0x21,0x3F,0x20,// H 40
	0x00,0x08,0x08,0xF8,0x08,0x08,0x00,0x00,
	0x00,0x20,0x20,0x3F,0x20,0x20,0x00,0x00,// I 41
	0x00,0x00,0x08,0x08,0xF8,0x08,0x08,0x00,
	0xC0,0x80,0x80,0x80,0x7F,0x00,0x00,0x00,// J 42
	0x08,0xF8,0x88,0xC0,0x28,0x18,0x08,0x00,
	0x20,0x3F,0x20,0x01,0x26,0x38,0x20,0x00,// K 43
	0x08,0xF8,0x08,0x00,0x00,0x00,0x00,0x00,
	0x20,0x3F,0x20,0x20,0x20,0x20,0x30,0x00,// L 44
	0x08,0xF8,0xF8,0x00,0xF8,0xF8,0x08,0x00,
	0x20,0x3F,0x00,0x3F,0x00,0x3F,0x20,0x00,// M 45
	0x08,0xF8,0x30,0xC0,0x00,0x08,0xF8,0x08,
	0x20,0x3F,0x20,0x00,0x07,0x18,0x3F,0x00,// N 46
	0xE0,0x10,0x08,0x08,0x08,0x10,0xE0,0x00,
	0x0F,0x10,0x20,0x20,0x20,0x10,0x0F,0x00,// O 47
	0x08,0xF8,0x08,0x08,0x08,0x08,0xF0,0x00,
	0x20,0x3F,0x21,0x01,0x01,0x01,0x00,0x00,// P 48
	0xE0,0x10,0x08,0x08,0x08,0x10,0xE0,0x00,
	0x0F,0x18,0x24,0x24,0x38,0x50,0x4F,0x00,// Q 49
	0x08,0xF8,0x88,0x88,0x88,0x88,0x70,0x00,
	0x20,0x3F,0x20,0x00,0x03,0x0C,0x30,0x20,// R 50
	0x00,0x70,0x88,0x08,0x08,0x08,0x38,0x00,
	0x00,0x38,0x20,0x21,0x21,0x22,0x1C,0x00,// S 51
	0x18,0x08,0x08,0xF8,0x08,0x08,0x18,0x00,
	0x00,0x00,0x20,0x3F,0x20,0x00,0x00,0x00,// T 52
	0x08,0xF8,0x08,0x00,0x00,0x08,0xF8,0x08,
	0x00,0x1F,0x20,0x20,0x20,0x20,0x1F,0x00,// U 53
	0x08,0x78,0x88,0x00,0x00,0xC8,0x38,0x08,
	0x00,0x00,0x07,0x38,0x0E,0x01,0x00,0x00,// V 54
	0xF8,0x08,0x00,0xF8,0x00,0x08,0xF8,0x00,
	0x03,0x3C,0x07,0x00,0x07,0x3C,0x03,0x00,// W 55
	0x08,0x18,0x68,0x80,0x80,0x68,0x18,0x08,
	0x20,0x30,0x2C,0x03,0x03,0x2C,0x30,0x20,// X 56
	0x08,0x38,0xC8,0x00,0xC8,0x38,0x08,0x00,
	0x00,0x00,0x20,0x3F,0x20,0x00,0x00,0x00,// Y 57
	0x10,0x08,0x08,0x08,0xC8,0x38,0x08,0x00,
	0x20,0x38,0x26,0x21,0x20,0x20,0x18,0x00,// Z 58
	0x00,0x00,0x00,0xFE,0x02,0x02,0x02,0x00,
	0x00,0x00,0x00,0x7F,0x40,0x40,0x40,0x00,// [ 59
	0x00,0x0C,0x30,0xC0,0x00,0x00,0x00,0x00,
	0x00,0x00,0x00,0x01,0x06,0x38,0xC0,0x00,// \ 60
	0x00,0x02,0x02,0x02,0xFE,0x00,0x00,0x00,
	0x00,0x40,0x40,0x40,0x7F,0x00,0x00,0x00,// ] 61
	0x00,0x20,0x10,0x08,0x04,0x08,0x10,0x20,
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,// ^ 62
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	0x80,0x80,0x80,0x80,0x80,0x80,0x80,0x80,// _ 63
	0x00,0x02,0x04,0x08,0x00,0x00,0x00,0x00,
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,// ` 64
	0x00,0x00,0x80,0x80,0x80,0x80,0x00,0x00,
	0x00,0x19,0x24,0x22,0x22,0x22,0x3F,0x20,// a 65
	0x08,0xF8,0x00,0x80,0x80,0x00,0x00,0x00,
	0x00,0x3F,0x11,0x20,0x20,0x11,0x0E,0x00,// b 66
	0x00,0x00,0x00,0x80,0x80,0x80,0x00,0x00,
	0x00,0x0E,0x11,0x20,0x20,0x20,0x11,0x00,// c 67
	0x00,0x00,0x00,0x80,0x80,0x88,0xF8,0x00,
	0x00,0x0E,0x11,0x20,0x20,0x10,0x3F,0x20,// d 68
	0x00,0x00,0x80,0x80,0x80,0x80,0x00,0x00,
	0x00,0x1F,0x22,0x22,0x22,0x22,0x13,0x00,// e 69
	0x00,0x80,0x80,0xF0,0x88,0x88,0x88,0x18,
	0x00,0x20,0x20,0x3F,0x20,0x20,0x00,0x00,// f 70
	0x00,0x00,0x80,0x80,0x80,0x80,0x80,0x00,
	0x00,0x6B,0x94,0x94,0x94,0x93,0x60,0x00,// g 71
	0x08,0xF8,0x00,0x80,0x80,0x80,0x00,0x00,
	0x20,0x3F,0x21,0x00,0x00,0x20,0x3F,0x20,// h 72
	0x00,0x80,0x98,0x98,0x00,0x00,0x00,0x00,
	0x00,0x20,0x20,0x3F,0x20,0x20,0x00,0x00,// i 73
	0x00,0x00,0x00,0x80,0x98,0x98,0x00,0x00,
	0x00,0xC0,0x80,0x80,0x80,0x7F,0x00,0x00,// j 74
	0x08,0xF8,0x00,0x00,0x80,0x80,0x80,0x00,
	0x20,0x3F,0x24,0x02,0x2D,0x30,0x20,0x00,// k 75
	0x00,0x08,0x08,0xF8,0x00,0x00,0x00,0x00,
	0x00,0x20,0x20,0x3F,0x20,0x20,0x00,0x00,// l 76
	0x80,0x80,0x80,0x80,0x80,0x80,0x80,0x00,
	0x20,0x3F,0x20,0x00,0x3F,0x20,0x00,0x3F,// m 77
	0x00,0x80,0x80,0x00,0x80,0x80,0x00,0x00,
	0x00,0x20,0x3F,0x21,0x00,0x20,0x3F,0x20,// n 78
	0x00,0x00,0x80,0x80,0x80,0x80,0x00,0x00,
	0x00,0x1F,0x20,0x20,0x20,0x20,0x1F,0x00,// o 79
	0x80,0x80,0x00,0x80,0x80,0x00,0x00,0x00,
	0x80,0xFF,0xA1,0x20,0x20,0x11,0x0E,0x00,// p 80
	0x00,0x00,0x00,0x80,0x80,0x80,0x80,0x00,
	0x00,0x0E,0x11,0x20,0x20,0xA0,0xFF,0x80,// q 81
	0x80,0x80,0x80,0x00,0x80,0x80,0x80,0x00,
	0x20,0x20,0x3F,0x21,0x20,0x00,0x01,0x00,// r 82
	0x00,0x00,0x80,0x80,0x80,0x80,0x80,0x00,
	0x00,0x33,0x24,0x24,0x24,0x24,0x19,0x00,// s 83
	0x00,0x80,0x80,0xE0,0x80,0x80,0x00,0x00,
	0x00,0x00,0x00,0x1F,0x20,0x20,0x00,0x00,// t 84
	0x80,0x80,0x00,0x00,0x00,0x80,0x80,0x00,
	0x00,0x1F,0x20,0x20,0x20,0x10,0x3F,0x20,// u 85
	0x80,0x80,0x80,0x00,0x00,0x80,0x80,0x80,
	0x00,0x01,0x0E,0x30,0x08,0x06,0x01,0x00,// v 86
	0x80,0x80,0x00,0x80,0x00,0x80,0x80,0x80,
	0x0F,0x30,0x0C,0x03,0x0C,0x30,0x0F,0x00,// w 87
	0x00,0x80,0x80,0x00,0x80,0x80,0x80,0x00,
	0x00,0x20,0x31,0x2E,0x0E,0x31,0x20,0x00,// x 88
	0x80,0x80,0x80,0x00,0x00,0x80,0x80,0x80,
	0x80,0x81,0x8E,0x70,0x18,0x06,0x01,0x00,// y 89
	0x00,0x80,0x80,0x80,0x80,0x80,0x80,0x00,
	0x00,0x21,0x30,0x2C,0x22,0x21,0x30,0x00,// z 90
	0x00,0x00,0x00,0x00,0x80,0x7C,0x02,0x02,
	0x00,0x00,0x00,0x00,0x00,0x3F,0x40,0x40,// { 91
	0x00,0x00,0x00,0x00,0xFF,0x00,0x00,0x00,
	0x00,0x00,0x00,0x00,0xFF,0x00,0x00,0x00,// | 92
	0x00,0x02,0x02,0x7C,0x80,0x00,0x00,0x00,
	0x00,0x40,0x40,0x3F,0x00,0x00,0x00,0x00,// } 93
	0x00,0x80,0x40,0x40,0x80,0x00,0x00,0x80,
	0x00,0x00,0x00,0x00,0x00,0x01,0x01,0x00,// ~ 94
};

/*宽6像素，高8像素*/
const uint8_t OLED_F6x8[][6] = 
{
	0x00,0x00,0x00,0x00,0x00,0x00,//   0
	0x00,0x00,0x00,0x2F,0x00,0x00,// ! 1
	0x00,0x00,0x07,0x00,0x07,0x00,// " 2
	0x00,0x14,0x7F,0x14,0x7F,0x14,// # 3
	0x00,0x24,0x2A,0x7F,0x2A,0x12,// $ 4
	0x00,0x23,0x13,0x08,0x64,0x62,// % 5
	0x00,0x36,0x49,0x55,0x22,0x50,// & 6
	0x00,0x00,0x00,0x07,0x00,0x00,// ' 7
	0x00,0x00,0x1C,0x22,0x41,0x00,// ( 8
	0x00,0x00,0x41,0x22,0x1C,0x00,// ) 9
	0x00,0x14,0x08,0x3E,0x08,0x14,// * 10
	0x00,0x08,0x08,0x3E,0x08,0x08,// + 11
	0x00,0x00,0x00,0xA0,0x60,0x00,// , 12
	0x00,0x08,0x08,0x08,0x08,0x08,// - 13
	0x00,0x00,0x60,0x60,0x00,0x00,// . 14
	0x00,0x20,0x10,0x08,0x04,0x02,// / 15
	0x00,0x3E,0x51,0x49,0x45,0x3E,// 0 16
	0x00,0x00,0x42,0x7F,0x40,0x00,// 1 17
	0x00,0x42,0x61,0x51,0x49,0x46,// 2 18
	0x00,0x21,0x41,0x45,0x4B,0x31,// 3 19
	0x00,0x18,0x14,0x12,0x7F,0x10,// 4 20
	0x00,0x27,0x45,0x45,0x45,0x39,// 5 21
	0x00,0x3C,0x4A,0x49,0x49,0x30,// 6 22
	0x00,0x01,0x71,0x09,0x05,0x03,// 7 23
	0x00,0x36,0x49,0x49,0x49,0x36,// 8 24
	0x00,0x06,0x49,0x49,0x29,0x1E,// 9 25
	0x00,0x00,0x36,0x36,0x00,0x00,// : 26
	0x00,0x00,0x56,0x36,0x00,0x00,// ; 27
	0x00,0x08,0x14,0x22,0x41,0x00,// < 28
	0x00,0x14,0x14,0x14,0x14,0x14,// = 29
	0x00,0x00,0x41,0x22,0x14,0x08,// > 30
	0x00,0x02,0x01,0x51,0x09,0x06,// ? 31
	0x00,0x3E,0x49,0x55,0x59,0x2E,// @ 32
	0x00,0x7C,0x12,0x11,0x12,0x7C,// A 33
	0x00,0x7F,0x49,0x49,0x49,0x36,// B 34
	0x00,0x3E,0x41,0x41,0x41,0x22,// C 35
	0x00,0x7F,0x41,0x41,0x22,0x1C,// D 36
	0x00,0x7F,0x49,0x49,0x49,0x41,// E 37
	0x00,0x7F,0x09,0x09,0x09,0x01,// F 38
	0x00,0x3E,0x41,0x49,0x49,0x7A,// G 39
	0x00,0x7F,0x08,0x08,0x08,0x7F,// H 40
	0x00,0x00,0x41,0x7F,0x41,0x00,// I 41
	0x00,0x20,0x40,0x41,0x3F,0x01,// J 42
	0x00,0x7F,0x08,0x14,0x22,0x41,// K 43
	0x00,0x7F,0x40,0x40,0x40,0x40,// L 44
	0x00,0x7F,0x02,0x0C,0x02,0x7F,// M 45
	0x00,0x7F,0x04,0x08,0x10,0x7F,// N 46
	0x00,0x3E,0x41,0x41,0x41,0x3E,// O 47
	0x00,0x7F,0x09,0x09,0x09,0x06,// P 48
	0x00,0x3E,0x41,0x51,0x21,0x5E,// Q 49
	0x00,0x7F,0x09,0x19,0x29,0x46,// R 50
	0x00,0x46,0x49,0x49,0x49,0x31,// S 51
	0x00,0x01,0x01,0x7F,0x01,0x01,// T 52
	0x00,0x3F,0x40,0x40,0x40,0x3F,// U 53
	0x00,0x1F,0x20,0x40,0x20,0x1F,// V 54
	0x00,0x3F,0x40,0x38,0x40,0x3F,// W 55
	0x00,0x63,0x14,0x08,0x14,0x63,// X 56
	0x00,0x07,0x08,0x70,0x08,0x07,// Y 57
	0x00,0x61,0x51,0x49,0x45,0x43,// Z 58
	0x00,0x00,0x7F,0x41,0x41,0x00,// [ 59
	0x00,0x02,0x04,0x08,0x10,0x20,// \ 60
	0x00,0x00,0x41,0x41,0x7F,0x00,// ] 61
	0x00,0x04,0x02,0x01,0x02,0x04,// ^ 62
	0x00,0x40,0x40,0x40,0x40,0x40,// _ 63
	0x00,0x00,0x01,0x02,0x04,0x00,// ` 64
	0x00,0x20,0x54,0x54,0x54,0x78,// a 65
	0x00,0x7F,0x48,0x44,0x44,0x38,// b 66
	0x00,0x38,0x44,0x44,0x44,0x20,// c 67
	0x00,0x38,0x44,0x44,0x48,0x7F,// d 68
	0x00,0x38,0x54,0x54,0x54,0x18,// e 69
	0x00,0x08,0x7E,0x09,0x01,0x02,// f 70
	0x00,0x18,0xA4,0xA4,0xA4,0x7C,// g 71
	0x00,0x7F,0x08,0x04,0x04,0x78,// h 72
	0x00,0x00,0x44,0x7D,0x40,0x00,// i 73
	0x00,0x40,0x80,0x84,0x7D,0x00,// j 74
	0x00,0x7F,0x10,0x28,0x44,0x00,// k 75
	0x00,0x00,0x41,0x7F,0x40,0x00,// l 76
	0x00,0x7C,0x04,0x18,0x04,0x78,// m 77
	0x00,0x7C,0x08,0x04,0x04,0x78,// n 78
	0x00,0x38,0x44,0x44,0x44,0x38,// o 79
	0x00,0xFC,0x24,0x24,0x24,0x18,// p 80
	0x00,0x18,0x24,0x24,0x18,0xFC,// q 81
	0x00,0x7C,0x08,0x04,0x04,0x08,// r 82
	0x00,0x48,0x54,0x54,0x54,0x20,// s 83
	0x00,0x04,0x3F,0x44,0x40,0x20,// t 84
	0x00,0x3C,0x40,0x40,0x20,0x7C,// u 85
	0x00,0x1C,0x20,0x40,0x20,0x1C,// v 86
	0x00,0x3C,0x40,0x30,0x40,0x3C,// w 87
	0x00,0x44,0x28,0x10,0x28,0x44,// x 88
	0x00,0x1C,0xA0,0xA0,0xA0,0x7C,// y 89
	0x00,0x44,0x64,0x54,0x4C,0x44,// z 90
	0x00,0x00,0x08,0x7F,0x41,0x00,// { 91
	0x00,0x00,0x00,0x7F,0x00,0x00,// | 92
	0x00,0x00,0x41,0x7F,0x08,0x00,// } 93
	0x00,0x08,0x04,0x08,0x10,0x08,// ~ 94
};
/*********************ASCII字模数据*/


/*汉字字模数据*********************/

/*相同的汉字只需要定义一次，汉字不分先后顺序*/
/*必须全部为汉字或者全角字符，不要加入任何半角字符*/

/*宽16像素，高16像素*/
const ChineseCell_t OLED_CF16x16[] = {
	
	"，",
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	0x00,0x00,0x58,0x38,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	
	"。",
	0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	0x00,0x00,0x18,0x24,0x24,0x18,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
	
	"你",
	0x00,0x80,0x60,0xF8,0x07,0x40,0x20,0x18,0x0F,0x08,0xC8,0x08,0x08,0x28,0x18,0x00,
	0x01,0x00,0x00,0xFF,0x00,0x10,0x0C,0x03,0x40,0x80,0x7F,0x00,0x01,0x06,0x18,0x00,
	
	"好",
	0x10,0x10,0xF0,0x1F,0x10,0xF0,0x00,0x80,0x82,0x82,0xE2,0x92,0x8A,0x86,0x80,0x00,
	0x40,0x22,0x15,0x08,0x16,0x61,0x00,0x00,0x40,0x80,0x7F,0x00,0x00,0x00,0x00,0x00,
	
	"世",
	0x20,0x20,0x20,0xFE,0x20,0x20,0xFF,0x20,0x20,0x20,0xFF,0x20,0x20,0x20,0x20,0x00,
	0x00,0x00,0x00,0x7F,0x40,0x40,0x47,0x44,0x44,0x44,0x47,0x40,0x40,0x40,0x00,0x00,
	
	"界",
	0x00,0x00,0x00,0xFE,0x92,0x92,0x92,0xFE,0x92,0x92,0x92,0xFE,0x00,0x00,0x00,0x00,
	0x08,0x08,0x04,0x84,0x62,0x1E,0x01,0x00,0x01,0xFE,0x02,0x04,0x04,0x08,0x08,0x00,
	
	/*按照上面的格式，在这个位置加入新的汉字数据*/
	//...
	
	
	/*未找到指定汉字时显示的默认图形（一个方框，内部一个问号），请确保其位于数组最末尾*/
	"",		
	0xFF,0x01,0x01,0x01,0x31,0x09,0x09,0x09,0x09,0x89,0x71,0x01,0x01,0x01,0x01,0xFF,
	0xFF,0x80,0x80,0x80,0x80,0x80,0x80,0x96,0x81,0x80,0x80,0x80,0x80,0x80,0x80,0xFF,

};

/*********************汉字字模数据*/


/*图像数据*********************/

/*测试图像（一个方框，内部一个二极管符号），宽16像素，高16像素*/
const uint8_t Diode[] = {
	0xFF,0x01,0x81,0x81,0x81,0xFD,0x89,0x91,0xA1,0xC1,0xFD,0x81,0x81,0x81,0x01,0xFF,
	0xFF,0x80,0x80,0x80,0x80,0x9F,0x88,0x84,0x82,0x81,0x9F,0x80,0x80,0x80,0x80,0xFF,
};

/*按照上面的格式，在这个位置加入新的图像数据*/
//...

/*********************图像数据*/


/*****************江协科技|版权所有****************/
/*****************jiangxiekeji.com*****************/

```

## OLED_Data.h

```c
#ifndef __OLED_DATA_H
#define __OLED_DATA_H

#include <stdint.h>

/*字符集定义*/
/*以下两个宏定义只可解除其中一个的注释*/
#define OLED_CHARSET_UTF8			//定义字符集为UTF8
//#define OLED_CHARSET_GB2312		//定义字符集为GB2312

/*字模基本单元*/
typedef struct 
{
	
#ifdef OLED_CHARSET_UTF8			//定义字符集为UTF8
	char Index[5];					//汉字索引，空间为5字节
#endif
	
#ifdef OLED_CHARSET_GB2312			//定义字符集为GB2312
	char Index[3];					//汉字索引，空间为3字节
#endif
	
	uint8_t Data[32];				//字模数据
} ChineseCell_t;

/*ASCII字模数据声明*/
extern const uint8_t OLED_F8x16[][16];
extern const uint8_t OLED_F6x8[][6];

/*汉字字模数据声明*/
extern const ChineseCell_t OLED_CF16x16[];

/*图像数据声明*/
extern const uint8_t Diode[];
/*按照上面的格式，在这个位置加入新的图像数据声明*/
//...

#endif


/*****************江协科技|版权所有****************/
/*****************jiangxiekeji.com*****************/

```





# SSD1306数据手册

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225255.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225337.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225404.png)

> 通过芯片的框图可以知道很多东西
>
> 通过数据手册了解每个输入引脚的作用是什么，因为配置引脚一定与这些输入引脚有关，将输入引脚分类并整理
>
> 通过翻译可以得知，

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225425.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225449.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225507.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225546.png)

> 配合原资料看

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225725.png)

> 这是关于SSD1306的引脚说明
>
> 通过引脚分布可以知道，如果使用裸屏，外围电路大概怎么画，以及在通信的过程中要注意哪些事项
>
> 详细了解每个引脚的作用与功能后，会清楚裸屏的外围电路是用来干嘛的。



![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225748.png)

> 通过数据手册，可以知道BS脚是用来选择使用何种通信协议，而且怎么去配置通信协议。
>
> 通过这个可以知道数据线在哪种通信协议代表的是什么

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225814.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225840.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225906.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521225936.png)

> 这是数据手册关于I2C通信部分的内容，里面说到了需要将SDAIN和SDAOUT连在一起才能充当SDA，所以在使用I2C协议的时候不能错过这种小细节
>
> 用I2C发送的数据是数据还是命令是通过D/C#引脚来决定的
>
> 根据原理图可以看出来，D/C#引脚为0，所以从机地址为0x78

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230008.png)

发送命令的话，Co为0，D/C#为0，后面几个默认为0，所以发送完从机地址后，再发送个0x00表示自己要发送命令了。



![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230042.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230130.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230243.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230306.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230332.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230358.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250521230422.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081541.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081613.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081644.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081713.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081743.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081808.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081833.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250522102538.jpg)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20250522102600.jpg)

图一是设置COM引脚的值为0x02，也就是A4和A5都给0

图二是设置COM引脚的值为0x22，也就是A4给1，A5给0，显示效果远好于图一，采用的是顺序COM引脚配置

> 设置COM输出扫描方向，本质是上下的扫描方向，C8是正常上下，C0是颠倒，这个要根据实际情况取值
>
> 设置分段重新映射的本质是上下左右颠倒的情况，如果上下左右颠倒可以使用设置分段重新映射

> SSD1306 OLED驱动芯片的“设置分段重新映射”（Set COM Output Scan Direction）功能允许你改变共通电极（COM）的扫描方向。这实际上会影响OLED显示屏上内容的垂直显示方向。
> 在SSD1306中，这个功能通过命令0xC0和0xC8来实现：
> 0xC0: 正常模式 (Normal Mode)。共通电极从上到下扫描，这是默认的扫描方向。
> 0xC8: 重映射模式 (Remapped Mode)。共通电极从下到上扫描，这会导致显示内容上下翻转。
> 为什么需要这个功能？
> 灵活的显示方向： 根据你的应用需求，你可能需要OLED屏幕以不同的方向安装或显示内容。这个功能允许你在软件层面轻松地改变显示方向，而无需物理上旋转屏幕。
> 与硬件配合： 在某些硬件设计中，由于PCB板布局或其他限制，OLED屏幕可能需要以特定的方向安装。通过设置COM输出扫描方向，你可以确保显示内容以正确的方向呈现。
> 兼容性： 有些OLED模块可能在制造时就已经设置了特定的扫描方向。通过软件调整，可以确保驱动程序与硬件模块的正确配合。
> 如何使用？
> 使用非常简单，只需要通过I2C或SPI接口发送相应的命令即可。
> 示例：
> 假设你正在使用I2C接口与SSD1306通信，以下是一个简单的示例代码，展示如何使用0xC0和0xC8命令来设置COM输出扫描方向：
> #include <Wire.h>
> // SSD1306的I2C地址
> #define SSD1306ADDRESS 0x3C
> void setup() {
>   Wire.begin(); // 初始化I2C
>   // 其他初始化代码...
>   // 发送0xC0命令以设置正常的扫描方向
>   Wire.beginTransmission(SSD1306ADDRESS);
>   Wire.write(0xC0); // 发送0xC0命令
>   Wire.endTransmission();
>   // 或者发送0xC8命令以设置翻转的扫描方向
>   Wire.beginTransmission(SSD1306ADDRESS);
>   Wire.write(0xC8); // 发送0xC8命令
>   Wire.endTransmission();
> }
> void loop() {
>   // 主循环代码...
> }
> 在这个示例中，我们首先初始化I2C接口，然后通过I2C向SSD1306发送0xC0或0xC8命令来设置COM输出的扫描方向。
> 总结：
> “设置分段重新映射” (Set COM Output Scan Direction) 功能允许你改变OLED屏幕的垂直显示方向。
> 0xC0 命令设置正常的扫描方向（从上到下）。
> 0xC8 命令设置翻转的扫描方向（从下到上）。
> 这个功能在灵活的显示方向、与硬件配合以及兼容性方面非常有用。
> 通过理解和使用这个功能，你可以更好地控制你的OLED显示屏，以满足各种应用需求。

> SSD1306是一款常用的单色OLED显示屏驱动芯片，支持I2C或SPI通信接口，可以驱动128x64或128x32像素的OLED显示屏。多路复用率（Multiplex Ratio）是OLED驱动芯片中的一个重要参数，它决定了显示屏上实际使用的扫描行数。
>
> 在OLED显示屏中，像素点是由多个有机发光二极管组成的，每个像素点可以独立控制亮度和颜色。为了控制这些像素点，驱动芯片需要逐行扫描显示屏上的像素点，这个过程称为多路复用。多路复用率就是指在一次完整的扫描过程中，驱动芯片实际扫描的行数。
>
> 例如，如果一个128x64像素的OLED显示屏的多路复用率设置为64，那么驱动芯片在一次完整的扫描过程中会扫描64行像素点。如果多路复用率设置为32，那么驱动芯片只会扫描32行像素点，这样显示屏的刷新率会提高，但显示的像素点数量会减少，导致显示效果变差。
>
> 在SSD1306中，多路复用率可以通过发送特定的控制命令来设置。例如，对于128x64像素的OLED显示屏，可以将多路复用率设置为64，以充分利用显示屏的所有像素点。对于128x32像素的OLED显示屏，可以将多路复用率设置为32。
>
> 总之，多路复用率是OLED驱动芯片中的一个重要参数，它决定了显示屏上实际使用的扫描行数，从而影响显示效果和刷新率。

> 在SSD1306 OLED驱动芯片中，COM引脚硬件配置（Set COM Pins Hardware Configuration）的作用和区别主要体现在以下几个方面：
>
> ### 作用
>
> 1. **左右重映射（COM Pin Configuration）**:
>     - **作用**：允许你改变COM引脚的左右映射方式。这可以影响显示内容的水平显示方向。
>     - **应用场景**：当你需要改变显示内容的水平方向时，例如从左到右或从右到左显示，这个配置非常有用。
> 2. **交替配置（Alternative Configuration）**:
>     - **作用**：启用或禁用COM引脚的交替配置。交替配置可以减少显示器的闪烁和提高显示质量。
>     - **应用场景**：在需要高质量显示的应用中，例如视频播放或动态图像显示，启用交替配置可以提供更好的视觉效果。
>
> ### 区别
>
> 1. **左右重映射**:
>     - **有左右重映射**：COM引脚的左右顺序可以根据需要进行调整，显示内容可以从左到右或从右到左显示。
>     - **无左右重映射**：COM引脚的左右顺序固定，显示内容只能按照默认的方向显示。
> 2. **交替配置**:
>     - **启用交替配置**：COM引脚的配置会交替进行，这有助于减少显示器的闪烁和提高显示质量。
>     - **禁用交替配置**：COM引脚的配置不会交替进行，显示效果可能会受到闪烁的影响。
>
> ### 示例

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081855.png)

> 时钟的值可以根据数据手册推荐的，也就是1000b，只采用1分频的话，怎么也就是10000000=0x80

> SSD1306的预充电期（Pre-Charge Period）是指在OLED显示屏中，驱动芯片对显示屏的行和列进行预充电的时间。预充电期是OLED驱动芯片中的一个重要参数，它影响着显示屏的显示效果和刷新率。
> 在OLED显示屏中，每个像素点都是由一个有机发光二极管组成的，这些二极管需要一定的电压才能点亮。在显示新的一帧图像之前，驱动芯片需要先对显示屏的行和列进行预充电，以确保所有的像素点都能获得足够的电压。
> 预充电期通常由两个参数控制：预充电时间（Pre-Charge Time）和预充电电压（Pre-Charge Voltage）。预充电时间是指驱动芯片对行和列进行预充电的时间长度，而预充电电压是指预充电过程中使用的电压值。
> 在SSD1306中，预充电期可以通过发送特定的控制命令来设置。例如，可以使用以下命令来设置预充电期：
> // 设置预充电期
> SSD1306Command(0xD9); // Set Pre-Charge Period
> SSD1306Command(0xF1); // 例如，设置预充电期为0xF1
> 在这个例子中，0xD9是设置预充电期的命令，而0xF1是预充电期的具体值。具体的预充电期值需要根据显示屏的特性和需求来确定。通常，预充电期值越大，预充电时间越长，显示效果越好，但刷新率会降低。
> 总之，预充电期是OLED驱动芯片中的一个重要参数，它影响着显示屏的显示效果和刷新率。通过发送特定的控制命令，可以设置SSD1306的预充电期，以优化显示屏的性能。

> 在SSD1306 OLED驱动芯片中，Vcomh取消选择级别（Vcomh Deselect Level）是指当像素未被选中时，提供给共模电极（Common Electrode）的电压水平。这个电压水平对于OLED显示屏的对比度和显示效果至关重要。
> Vcomh是Vcommon high的缩写，它代表了OLED显示屏中未选中像素的公共电极上的电压。通过调整Vcomh取消选择级别，可以改变OLED显示屏的对比度。较高的Vcomh电压通常会导致更高的对比度，而较低的Vcomh电压则可能导致对比度降低。
> 在SSD1306中，Vcomh取消选择级别可以通过发送特定的控制命令来设置。例如，可以使用以下命令来设置Vcomh取消选择级别：
> // 设置Vcomh取消选择级别
> SSD1306Command(0xDB); // Set Vcomh Deselect Level
> SSD1306Command(0x40); // 例如，设置Vcomh取消选择级别为0x40
> 在这个例子中，0xDB是设置Vcomh取消选择级别的命令，而0x40是Vcomh取消选择级别的具体值。具体的Vcomh取消选择级别值需要根据显示屏的特性和需求来确定。通常，这个值在0x00到0x7F之间，其中0x00表示最低的Vcomh电压，而0x7F表示最高的Vcomh电压。
> 通过调整Vcomh取消选择级别，可以优化OLED显示屏的对比度和显示效果。然而，需要注意的是，过高的Vcomh电压可能会导致显示异常或烧毁像素，因此需要谨慎设置。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081925.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522081946.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082013.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082039.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082102.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082130.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082209.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082231.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082251.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082323.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082350.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082417.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082439.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082459.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082518.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082539.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082559.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082622.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082645.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082706.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20250522082734.png)
