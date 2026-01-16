![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/3.4-1.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/354eff86a1dd35e7b174a331f164ea8c.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/099d5f978e62ecb2d3f2b748a40c3eee.jpeg)

# 想法



克拉克变化，将ia,ib,ic转化为Iα，Iβ

帕克变换，将Iα，Iβ转变为Iq,Id

一般情况下Id为0，Iq的物理意义是力矩的大小

电角度，是转一圈实际会给我几圈的信号

因为涉及到速度以及时间，所以会很在意每次运算的时间和运算间隔，因为需要知道运算间隔，所以定时器的时间设置一定要清楚，知道自己多少ms进一次中断，以及pwm的ARR值，这个也很关键，在FOC运算中也是其中一个参数。

偏置电压float center = voltage_power_supply / 2.0f;中一般是固定除以2的，拿实际电压除以2做偏置电压，这也是一个简单的SVPWM，真正的SVPWM是可以自己调整这个偏置电压，使其最小不会低于0，最大不会过voltage_power_supply / 2.0f，所以说它会提高母线电压利用率

力矩和速度必须匹配一个比较好的关系，必然电机力矩过大或者力矩过小都会发烫，抖动严重。

速度开环一般涉及几个函数

电角度的计算

角度归一化

帕克逆变换，拉克逆变换

PWM输出

速度生成器

# 克拉克变换

## 解释

所谓**克拉克变换，实际上就是降维解耦的过程，把难以辨明和控制的三相相位差120°电机波形降维为两维矢量**。、

它的思路其实特别的简单，**第一就是把三相随时间变换的，相位差为120°的电流波形抽象化为三个间隔120°的矢量**。

**第二就是利用三角函数对矢量进行降维，降维到两个坐标轴**，从此复杂的三相变化问题就降解为了α-β坐标轴的坐标上的数值变化问题。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/3.1-3.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%85%8B%E6%8B%89%E5%85%8B%E5%8F%98%E6%8D%A2.png)

通过拉克拉变换，可以通过Ia,Ib,Ic得到Iα，Iβ

通过电流采集得到Ia,Ib,Ic算出Iα，Iβ从而得出Id,Iq

代码的计算如下

```c
Iα=Ia;
Iβ=0.577*(2*Ib+Ia);
```

# 克拉克逆变换

## 介绍

将Iα，Iβ逆变换成Ia，Ib，Ic。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%85%8B%E6%8B%89%E5%85%8B%E9%80%86%E5%8F%98%E6%8D%A2.png)

## 代码

```c
Ia=Iα;
Ib=0.866*Iβ-0.5*Iα;
Ic=-0.5*Iα-0.866*Iβ;
```

# 帕克变换

## 介绍

将Iα，Iβ转换为Id，Iq

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/7a32982cb57942f39b4b4f8f349f13a6.png)

## 代码

```c
//electrical_angle是电角度
Id=cos(electrical_angle)*Iα-sin(electrical_angle)*Iβ;  //一般默认Id等于0
Iq=sin(electrical_angle)*Iβ+cos(electrical_angle)*Iα
```



# 帕克逆变换

## 介绍

将Iq，Id转变为Iα，Iβ（通常Id=0）

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%B8%95%E5%85%8B%E5%8F%98%E6%8D%A2.png)

## 代码

```c
//electrical_angle是电角度
Iα=Id*cos(electrical_angle)-Iq*sin(electrical_angle);
Iβ=Iq*cos(electrical_angle)+Id*sin(electrical_angle);
```

# 电角度计算

## 介绍

电角度=机械角度*极对数

机械角度：实际电机转一圈

极对数：NS对数

电角度的意义在于你实际转一圈会告诉我转了个NS对的信号，打个比方，比如一对NS，实际转一圈，给的信号也是一圈，如果两对NS，实际转一圈，给的信号是转两圈。

## 代码的实现

```c
float electrical_angle(float Mechanical_angle,int pole_pairs)
{
	return (Mechanical_angle*pole_pairs);
}	
```

# 角度归一化

## 介绍

因为极对数的存在，如果一个参数来代表转了几圈，如果高速运转几圈则参数的值会变得很大，所以要限制在2PI中，也就是一圈。相当于取余，我们只知道位置在哪就可以，不用知道具体转了多少圈。

## 代码的实现

```c
float Normalized_angle(float angle)
{
	float a=fmod(angle,2*PI);
	return a>=0?a:(a+2*PI);
}
```



# 偏置电压

## 介绍

因为FOC是三个变化的正弦波，但是单片机很少有负电压，所以一般通过编制电压将电压拉到驱动电压的一半，在此基础上进行升降。

## 代码的实现

```c
float center = voltage_power_supply / 2.0f;     //偏置电压
  Ua = center + Uq * sin_a;    
  Ub = center + Uq * sin_b;
  Uc = center + Uq * sin_c;
```

> 虽然是软件进行偏置，因为输入的电压会转变成PWM的占空比，所以电压越大占空比越大，软件手动偏置与现实关系映射。





# FOC速度开环控制

## 过程

第一步，初始化（具体是PWM,定时器初始化）

第二步：准备前置函数

1. 电角度计算
2. 角度归一化
3. 计算变换
4. 速度生成器
5. 输出PWM函数

第三步：在中断中调用输出函数

速度开环控制主要是，输入一个速度，通过速度和运算间隔算出下一瞬间要转的角度，以及根据速度*kv值获得扭矩Iq的值。

定时器的运算时间不能太慢，建议可以给1ms，因为慢的话卡壳会很严重，但是仍旧有一个问题，就是所有浮点型的运算都放在中断中的话，会以为运算很慢导致其他程序严重受阻，所以这是一个问题

## 代码实现

### PWM

PWM涉及到了设置死区时间以及PWM互补，所以设置上会稍微麻烦一点，然后ARR和PSC的值要根据电机去设置

stm32f103c8t6只有TIM1才有互补功能，所以初始化也是TIM1

```c
/*--------------Definition.h----------------*/
#define TIM_USE TIM1
#define PWM1_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE)
#define PWM1_GPIO_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE)
#define PWM1N_GPIO_CLOCK	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE)
#define ARR 4799  //这里关乎的电机的精度，大部分用的是15kHz，所以72000000/(4799+1)/(0+1)=15kHz
#define PSC 0
#define CCR1 2400
#define CCR2 2400
#define CCR3 240
#define PWM_DeadTime 0x24  //死区时间，经验为0x24
#define TIM_CLOCK2 RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
#define TIM_USE2 TIM2
#define TIM_Channel TIM2_IRQn
typedef struct {
	const uint16_t PWM1_PIN;
	const uint16_t PWM2_PIN;
	const uint16_t PWM3_PIN;
	const uint16_t PWM1N_PIN;
	const uint16_t PWM2N_PIN;
	const uint16_t PWM3N_PIN;
	GPIO_TypeDef* PWM_Prot;
	GPIO_TypeDef* PWMN_Prot;
}MyPWM_Device_t;
extern const MyPWM_Device_t mypwm;
/*--------------Definition.c----------------*/
const MyPWM_Device_t mypwm=
{
  .PWM1_PIN=GPIO_Pin_8,
	.PWM2_PIN=GPIO_Pin_9,
	.PWM3_PIN=GPIO_Pin_10,
	.PWM1N_PIN=GPIO_Pin_13,
	.PWM2N_PIN=GPIO_Pin_14,
	.PWM3N_PIN=GPIO_Pin_15,
	.PWM_Prot=GPIOA,
	.PWMN_Prot=GPIOB
};
/*--------------MyPWM.h---------------------*/
#ifndef __MYPWM_H__
#define __MYPWM_H__


void PWM_Init(void);

#endif
/*--------------MyPWM.c---------------------*/
#include "stm32f10x.h"                  // Device header
#include "Definition.h"
#include "MyPWM.h"
void PWM_Init(void)
{
	PWM1_CLOCK;
	PWM1_GPIO_CLOCK;
	PWM1N_GPIO_CLOCK;
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = mypwm.PWM1_PIN|mypwm.PWM2_PIN|mypwm.PWM3_PIN;	
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(mypwm.PWM_Prot, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Pin = mypwm.PWM1N_PIN|mypwm.PWM2N_PIN|mypwm.PWM3N_PIN;
	GPIO_Init(mypwm.PWMN_Prot, &GPIO_InitStructure);
	
	TIM_InternalClockConfig(TIM1);    //高级定时器必备，只有这个TIM1才能正常启动
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_CenterAligned1;  //这里采用的是中央对齐模式，这个很重要，因为无刷电机生成的波形是以6V为基点上下浮动的，所以要选中央对齐
	TIM_TimeBaseInitStructure.TIM_Period = ARR;
	TIM_TimeBaseInitStructure.TIM_Prescaler = PSC;
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM_USE, &TIM_TimeBaseInitStructure); 
	
	TIM_OCInitTypeDef TIM_OCInitStructure;
	TIM_OCStructInit(&TIM_OCInitStructure);
	TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
	TIM_OCInitStructure.TIM_OutputState=TIM_OutputState_Enable;   //使能位
	TIM_OCInitStructure.TIM_OutputNState=TIM_OutputNState_Enable;  //互补使能位
	TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;  //主通道极性为高
	TIM_OCInitStructure.TIM_OCNPolarity=TIM_OCPolarity_High;  //互补极性为高
	TIM_OCInitStructure.TIM_OCIdleState=TIM_OCIdleState_Set;  //主通道空闲为高
	TIM_OCInitStructure.TIM_OCNIdleState=TIM_OCIdleState_Reset; //互补空闲为底
	
	TIM_BDTRInitTypeDef TIM_BDTRInitStruct;
	TIM_BDTRStructInit(&TIM_BDTRInitStruct);
	TIM_BDTRInitStruct.TIM_DeadTime=PWM_DeadTime;  //这个是死区时间
	TIM_BDTRInitStruct.TIM_AutomaticOutput=TIM_AutomaticOutput_Disable;  //禁用自动输出
	TIM_BDTRInitStruct.TIM_Break=TIM_Break_Disable;  //禁用刹车功能
	TIM_BDTRInitStruct.TIM_LOCKLevel=TIM_LOCKLevel_OFF;  
	TIM_BDTRInitStruct.TIM_OSSRState=TIM_OSSRState_Enable;
	TIM_BDTRInitStruct.TIM_OSSIState=TIM_OSSRState_Enable;
	TIM_BDTRConfig(TIM_USE,&TIM_BDTRInitStruct);
	
	
	TIM_OCInitStructure.TIM_Pulse = CCR1;
	TIM_OC1Init(TIM_USE, &TIM_OCInitStructure);  //通道占空比
	TIM_OCInitStructure.TIM_Pulse = CCR2;
	TIM_OC2Init(TIM_USE, &TIM_OCInitStructure);
	TIM_OCInitStructure.TIM_Pulse = CCR3;
	TIM_OC3Init(TIM_USE, &TIM_OCInitStructure);
	TIM_CtrlPWMOutputs(TIM_USE,ENABLE);
	TIM_Cmd(TIM_USE, ENABLE);
}
```



### FOC变换

```c
/*---------------Definition.h------------------*/
#ifndef	__DEFINITION_H__
#define __DEFINITION_H__
#include "stm32f10x.h" 
/*pwm底层索引*/
#define TIM_USE TIM1
#define PWM1_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE)
#define PWM1_GPIO_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE)
#define PWM1N_GPIO_CLOCK	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE)
#define ARR 4799
#define PSC 0
#define CCR1 2400
#define CCR2 2400
#define CCR3 2400
#define PWM_DeadTime 0x24
#define TIM_CLOCK2 RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
#define TIM_USE2 TIM2
#define TIM_Channel TIM2_IRQn
typedef struct {
	const uint16_t PWM1_PIN;
	const uint16_t PWM2_PIN;
	const uint16_t PWM3_PIN;
	const uint16_t PWM1N_PIN;
	const uint16_t PWM2N_PIN;
	const uint16_t PWM3N_PIN;
	GPIO_TypeDef* PWM_Prot;
	GPIO_TypeDef* PWMN_Prot;
}MyPWM_Device_t;
extern const MyPWM_Device_t mypwm;




/*FOC结构体定义*/
#define constrain(x,low,high) ((x)<(low)?(low):((x)>(high)?(high):(x)))
#define PI 3.14159265358979323846f
#define Kv 70.0f
typedef struct{
	float kp;
	float ki;
	float kd;
}PID_Drive_t;

typedef struct{
	const float voltage_power_supply;
	const float voltage_power_limit;
	const float pole_pairs;
	const int8_t motor_sign;
}FOC_Config_t;

typedef struct{
	float angle_rad;
	float angle_deg;
	float Ua,Ub,Uc;
	float Ia,Ib,Ic;
	float Ualpha,Ubeta;
	float Ialpha,Ibeta;
	float Iq,Id;
	float Uq,Ud;
	float deviation_angle;
	float Speed_expert;
}FOC_Status_t;

typedef struct{
	FOC_Config_t	config;
	FOC_Status_t	status;
	PID_Drive_t	Site_PID;
	PID_Drive_t Speed_PID;
	PID_Drive_t	Current_q_PID;
	PID_Drive_t	Current_d_PID;	
}FOC_Drive_t;
extern FOC_Drive_t foc;
	



#endif
/*---------------Definition.c------------------*/
#include "Definition.h"
const MyPWM_Device_t mypwm=
{
  .PWM1_PIN=GPIO_Pin_8,
	.PWM2_PIN=GPIO_Pin_9,
	.PWM3_PIN=GPIO_Pin_10,
	.PWM1N_PIN=GPIO_Pin_13,
	.PWM2N_PIN=GPIO_Pin_14,
	.PWM3N_PIN=GPIO_Pin_15,
	.PWM_Prot=GPIOA,
	.PWMN_Prot=GPIOB
};

/*FOC结构体初始化*/
FOC_Drive_t foc=
{
	 .config={
		 .voltage_power_supply=12,
		 .voltage_power_limit=10,
		 .pole_pairs=6,
		 .motor_sign=1
	 },
	 .status={.Speed_expert=100},
	 .Site_PID={0},
	 .Speed_PID={0},
	 .Current_q_PID={0},
	 .Current_d_PID={0}
};

/*---------------FOC.h-------------------------*/
#ifndef __FOC_H__
#define __FOC_H__

void FOC_Init(void);
void countPWM(float Uq,float Ud,float angle_el);
void setPWM(float Ua,float Ub,float Uc);
float electrical_angle(float Mechanical_angle);
float Normalized_angle(float angle);
float Angle_chance(float angle);
#endif



/*---------------FOC.c-------------------------*/
#include "stm32f10x.h"                  // Device header
#include "math.h"
#include "MyPWM.h"
#include "FOC.h"
#include "Timer2.h"
#include "Definition.h"
#include "MT6816.h"
  

void FOC_Init(void)
{
	PWM_Init();
	Timer2_Init();
}


// 电角度计算
float electrical_angle(float Mechanical_angle)
{
	return foc.config.motor_sign*(Mechanical_angle*foc.config.pole_pairs);
}	

//角度归一化
float Normalized_angle(float angle)
{
	float a=fmod(angle,2*PI);
	return a>=0?a:(a+2*PI);
}
//角度化为弧度
float Angle_chance(float angle)
{ float angle_deg;
	float angle_rad;
  angle_deg=angle-67;
	if (angle_deg > 180.0f) {
        angle_deg -= 360.0f;
    } else if (angle_deg < -180.0f) {
        angle_deg += 360.0f;
    }             
		angle_rad=angle_deg*PI/180.0f;
	return angle_rad;
}

//输出PWM
void setPWM(float Ua,float Ub,float Uc)
{
	float dc_a,dc_b,dc_c;
	
	//进行限压
	Ua=constrain(Ua,0,foc.config.voltage_power_limit);
	Ub=constrain(Ub,0,foc.config.voltage_power_limit);
	Uc=constrain(Uc,0,foc.config.voltage_power_limit);

  //计算占空比
	dc_a=constrain(Ua/foc.config.voltage_power_supply,0.0f,1.0f);       
	dc_b=constrain(Ub/foc.config.voltage_power_supply,0.0f,1.0f);
	dc_c=constrain(Uc/foc.config.voltage_power_supply,0.0f,1.0f);

  //赋值
	TIM_SetCompare1(TIM_USE,dc_a*ARR);       
	TIM_SetCompare2(TIM_USE,dc_b*ARR);
	TIM_SetCompare3(TIM_USE,dc_c*ARR);
}

void countPWM(float Uq,float Ud,float angle_el)   
{
	
	angle_el=Normalized_angle(angle_el);   //角度归一化
	
	float sin_a=sin(angle_el);               
	float sin_b=sin(angle_el-2.0f*PI/3.0f);
	float sin_c=sin(angle_el+2.0f*PI/3.0f);
	//帕克逆变换
	foc.status.Ualpha =  -Uq*sin(angle_el); 
  foc.status.Ubeta =   Uq*cos(angle_el);
	//克拉克逆变换
	foc.status.Ua = foc.status.Ualpha + foc.config.voltage_power_supply/2;
  foc.status.Ub = (sqrt(3)*foc.status.Ubeta-foc.status.Ualpha)/2 + foc.config.voltage_power_supply/2;
  foc.status.Uc = (-foc.status.Ualpha-sqrt(3)*foc.status.Ubeta)/2 + foc.config.voltage_power_supply/2;
	
	setPWM(foc.status.Ua,foc.status.Ub,foc.status.Uc);   //输出Ua,Ub,Uc
}	

/*---------------main.c------------------------*/
#include "stm32f10x.h"                  // Device header
#include "Definition.h"
#include "FOC.h"
#include "Delay.h"
int main()
{
	FOC_Init();
	while(1)
	{
	}
}
void TIM2_IRQHandler(void)  
{
	if(TIM_GetITStatus(TIM_USE2,TIM_IT_Update)==SET)
	{
		foc.status.Uq=foc.status.Speed_expert/Kv;  //根据速度和Kv得到Uq
		foc.status.angle_rad = Normalized_angle(foc.status.angle_rad + foc.status.Speed_expert*0.001f);  //速度生成器
		countPWM(foc.status.Uq,0,electrical_angle(Normalized_angle(foc.status.angle_rad)));  //输出
		TIM_ClearITPendingBit(TIM_USE2,TIM_IT_Update);
	}
}

```

## 总结

速度开环控制是我们自己模拟一个电机使其转动，通过PWM设置使实际电机跟着模拟电机跑，实际电机有没有到位我们是不知道的。通过速度生成器我们知道下一步的电角度，然后这个电角度通过拉克拉逆变换和派克逆变换变成Ua,Ub,Uc。需要的力矩Uq的大小是通过速度Up=Kv*Speed得到的，物理意义就是速度越快，我们输入的Up就越大

其中值得注意的是定时器和PWM的设置，以及死区时间的设置，以及速度与Uq的参数要调，极对数要调

# FOC位置闭环控制

## 大框

目前来说FOC位置闭环就是将FOC速度开环控制中的速度用传感器传回来的位置取代，然后要引入pid，就是当前位置与目标位置的差值越大，力越大。

如果编码器与磁铁同心，那么只需要校准偏差做补偿，如果离心会相对麻烦一点

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251012215208.png)

## 细致

先将定时器2，PWM和mt6816初始化搞定

编写函数

1. 电角度计算
2. 角度归一化
3. 计算变换
4. 输出PWM函数
5. 获取MT6816位置参数
6. 位置pid

逻辑如下

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/5-10.png)

输入一个指定位置，输入pid中，pid内部计算差值然后通过计算得出Uq，然后



# FOC速度闭环控制

## 低通滤波

### 代码

```c
float LowPassFilter(float x)
{
    float alpha=TF/(TF+dt);  //TF是一个常数，dt越大，那么alpha值越小，则当前值的比例越大，dt越小，alpha值越大，那么上一刻的值比例越大
    float y=alpha*y_prev+(1.0f-alpha)*x;
    y_prev=y;
    timestamp_prev=timestamp;
    return y;
}
```

> 低通滤波就是用上一次的值对这一次的值进行修正，如果两次时间间隔太长，那么上一次的值已经不具备修正的作用了。

## 大框

先解决获取速度，然后呢写一个pid来通过速度控制电机转动

通过mt6816获取角度，然后想一下怎么获取速度

然后将速度放进pid里，通过差值来控制力矩

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20251015094003.png)

> 速度环如果只有P，那么当电机的负载变大时，电机转动就达不到目标，只有加上积分I去消除负载变化所导致的稳态误差才行。



## 细致

mt6816获取的是角度，可以通过一个10ms的定时器，然后10ms计算一次，但是怎么区分正反转以及转了多少圈，mt6816能给我的只有0-360度的数值。

## 代码

```c
//这段代码用的都是弧度
float Speed_Get(float angle_current)  
{
    float Speed;
    static float full_rotations,full_rotations_last;
    static float angle_last;
    float angle_Diff=angle_current-angle_last;
    if(abs(angle_Diff)>(0.8f*2PI)) full_rotations+=(angle_Diff>0)?-1:1;  //如果角度的绝对值大于一圈的80%，则圈数加1
    angle_last=angle_current
    Speed=((float)(full_rotations-full_rotations_last)*2PI+(angle_current-angle_last))/Timer2.time; //Timer2.time->定时器中断时间    
    return (float)full_rotations*2PI+angle_current;
}

```



# FOC电流闭环控制

## 硬石驱动板电流采集介绍

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251013204651.png)

> 这里说的是我们用stm32可以直接采集硬石的线电压，并且驱动板的电压有1.65V的直流偏置，意味者1.65V为中心点，在1.65V上下波动

## 三电阻采集

三电极要避免去采样PWM占空比接近100%的那一相电流

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/3c4672caeadb767aa9c2efe9e6d774ef.png)

## 原理介绍

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/8-1.png)

我们如果知道Ia,Ib，就可以知道Iα和Iβ，然后Iα和Iβ就可以求得Iq和Id，所以如果要控制力矩，那么就需要Iq达到我们想要的Iq_ref，所以我们输入一个期望值Iq_ref，然后采集的线电流通过计算得到Iq即可。

```c
Uq=PID(Iq_ref-Iq)
```

这里的pid适合于PI计算，不适合D

## 大框

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2c9aca46c1693daaa6b61e88d1997369.png)

大致分为几步

首先是确定功能

* 能检测线电流
* 电机能转
* 能获取MT6816的值
* 上位机能显示Iq和Iq_ref
* Iq和Iq_ref的闭环PID
* 每隔1ms进入中断执行

其次是根据功能确定步骤

* 初始化ADC（要使用DMA进行转运）
* 初始化PWM
* 初始化SPI
* 初始化UART
* 初始化timer
* 编写函数
    * ADC的值得Iq
    * FOC变换
    * 电流闭环PID函数
    * 获取MT6816值
    * 发送上位机
    * 上电找零点
    * 在中断中执行

> 初始化的函数在前面几个环都有，并且基本不变

## 注意事项

重点在于获取Iq的值，因为单个环的话不好测，所以将重点放在获取Iq以及Iq的PID上，整体效果要等三环控制才行。

让id尽可能地接近0，提高效率
 使用电压开环控制时，由于种种原因，id并不是一直为0，可能存在偏移或是跳动，转速越高这种情况就越厉害，而使用电流闭环可以较好地遏制这种现象。

下面两张图显示了在20V供电下，某台电机空载全速转动下的id和iq波形，其中蓝色的是id。第一张图使用电压开环控制，第二张图使用电流闭环控制。

## 代码

### ADC+DMA初始化

使用ADC1，将三线电压采集，然后通过DMA转运到一个数组

stm32f103c8t6剩余的引脚中有PA0,PA1,PB0，对应这CH0,CH1,CH8

```c
/*-----------------Definition.h--------------*/
typedef struct {
	const uint16_t ADC1_CH1;
	const uint16_t ADC1_CH2;
	const uint16_t ADC1_CH8;
	GPIO_TypeDef* ADC_CH1_CH7_Prot;
	GPIO_TypeDef* ADC_CH8_CH9_Prot;
}MyADC_Drive_t;
extern const MyADC_Drive_t myadc1;
/*-----------------Definition.c----------------*/
const MyADC_Drive_t myadc1=
{
	.ADC1_CH1=GPIO_Pin_0,
	.ADC1_CH2=GPIO_Pin_1,
	.ADC1_CH8=GPIO_Pin_0,
	.ADC_CH1_CH7_Prot=GPIOA,
	.ADC_CH8_CH9_Prot=GPIOB
};
/*-----------------ADC.h-----------------------*/
 #ifndef __MYADC_H__
 #define __MYADC_H__
extern uint16_t I_Value[3];  //实则这里可以读取两个就可以
void AD_Init(void);
 #endif

/*-----------------ADC.c----------------------*/
#include "stm32f10x.h"                  // Device header
#include "Definition.h"
uint16_t I_Value[3];
void AD_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);	
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);	
	
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = myadc1.ADC1_CH1|myadc1.ADC1_CH2;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(myadc1.ADC_CH1_CH7_Prot, &GPIO_InitStructure);	
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = myadc1.ADC1_CH8;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(myadc1.ADC_CH8_CH9_Prot, &GPIO_InitStructure);
	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_55Cycles5);	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_8, 3, ADC_SampleTime_55Cycles5);	
	
	ADC_InitTypeDef ADC_InitStructure;					
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;	
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;	
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;	
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;		
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;	
	ADC_InitStructure.ADC_NbrOfChannel = 3;
	ADC_Init(ADC1, &ADC_InitStructure);

	DMA_InitTypeDef DMA_InitStructure;	
	
	DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;	
	DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;
	DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
	DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)I_Value;
	DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
	DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;	
	DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
	DMA_InitStructure.DMA_BufferSize = 3;
	DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;	
	DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;
	DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;						
	DMA_Init(DMA1_Channel1, &DMA_InitStructure);
	DMA_Cmd(DMA1_Channel1, ENABLE);	
	ADC_DMACmd(ADC1, ENABLE);
	ADC_Cmd(ADC1, ENABLE);
	ADC_ResetCalibration(ADC1);								
	while (ADC_GetResetCalibrationStatus(ADC1) == SET);
	ADC_StartCalibration(ADC1);
	while (ADC_GetCalibrationStatus(ADC1) == SET);
	
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
}
 
 
```

### PWM初始化

```c
/*-----------------Definition.h--------------*/
/*定时器底层参数索引*/
#define TIM_USE TIM1
#define PWM1_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE)
#define PWM1_GPIO_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE)
#define PWM1N_GPIO_CLOCK	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE)
#define ARR 4799
#define PSC 0
#define CCR1 2400
#define CCR2 2400
#define CCR3 2400
#define PWM_DeadTime 0x24   //经验
#define TIM_CLOCK2 RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
#define TIM_USE2 TIM2
#define TIM_Channel TIM2_IRQn
typedef struct {
	const uint16_t PWM1_PIN;
	const uint16_t PWM2_PIN;
	const uint16_t PWM3_PIN;
	const uint16_t PWM1N_PIN;
	const uint16_t PWM2N_PIN;
	const uint16_t PWM3N_PIN;
	GPIO_TypeDef* PWM_Prot;
	GPIO_TypeDef* PWMN_Prot;
}MyPWM_Device_t;
extern const MyPWM_Device_t mypwm;
/*-----------------Definition.c----------------*/
const MyPWM_Device_t mypwm=
{
  .PWM1_PIN=GPIO_Pin_8,
	.PWM2_PIN=GPIO_Pin_9,
	.PWM3_PIN=GPIO_Pin_10,
	.PWM1N_PIN=GPIO_Pin_13,
	.PWM2N_PIN=GPIO_Pin_14,
	.PWM3N_PIN=GPIO_Pin_15,
	.PWM_Prot=GPIOA,
	.PWMN_Prot=GPIOB
};
/*---------PWM.h----------------------*/
#ifndef __MYPWM_H__
#define __MYPWM_H__

void PWM_Init(void);

#endif

/*----------PWM.c-----------------------*/
#include "stm32f10x.h"                  // Device header
#include "Definition.h"
#include "MyPWM.h"
void PWM_Init(void)
{
	PWM1_CLOCK;
	PWM1_GPIO_CLOCK;
	PWM1N_GPIO_CLOCK;
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = mypwm.PWM1_PIN|mypwm.PWM2_PIN|mypwm.PWM3_PIN;	
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(mypwm.PWM_Prot, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Pin = mypwm.PWM1N_PIN|mypwm.PWM2N_PIN|mypwm.PWM3N_PIN;
	GPIO_Init(mypwm.PWMN_Prot, &GPIO_InitStructure);
	
	TIM_InternalClockConfig(TIM1);
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_CenterAligned1;
	TIM_TimeBaseInitStructure.TIM_Period = ARR;
	TIM_TimeBaseInitStructure.TIM_Prescaler = PSC;
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM_USE, &TIM_TimeBaseInitStructure); 
	
	TIM_OCInitTypeDef TIM_OCInitStructure;
	TIM_OCStructInit(&TIM_OCInitStructure);
	TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;
	TIM_OCInitStructure.TIM_OutputState=TIM_OutputState_Enable;
	TIM_OCInitStructure.TIM_OutputNState=TIM_OutputNState_Enable;
	TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;
	TIM_OCInitStructure.TIM_OCNPolarity=TIM_OCPolarity_High;
	TIM_OCInitStructure.TIM_OCIdleState=TIM_OCIdleState_Set;
	TIM_OCInitStructure.TIM_OCNIdleState=TIM_OCIdleState_Reset;
	
	TIM_BDTRInitTypeDef TIM_BDTRInitStruct;
	TIM_BDTRStructInit(&TIM_BDTRInitStruct);
	TIM_BDTRInitStruct.TIM_DeadTime=PWM_DeadTime;
	TIM_BDTRInitStruct.TIM_AutomaticOutput=TIM_AutomaticOutput_Disable;
	TIM_BDTRInitStruct.TIM_Break=TIM_Break_Disable;
	TIM_BDTRInitStruct.TIM_LOCKLevel=TIM_LOCKLevel_OFF;
	TIM_BDTRInitStruct.TIM_OSSRState=TIM_OSSRState_Enable;
	TIM_BDTRInitStruct.TIM_OSSIState=TIM_OSSRState_Enable;
	TIM_BDTRConfig(TIM_USE,&TIM_BDTRInitStruct);
	
	
	TIM_OCInitStructure.TIM_Pulse = CCR1;
	TIM_OC1Init(TIM_USE, &TIM_OCInitStructure);
	TIM_OCInitStructure.TIM_Pulse = CCR2;
	TIM_OC2Init(TIM_USE, &TIM_OCInitStructure);
	TIM_OCInitStructure.TIM_Pulse = CCR3;
	TIM_OC3Init(TIM_USE, &TIM_OCInitStructure);
	TIM_CtrlPWMOutputs(TIM_USE,ENABLE);
	TIM_Cmd(TIM_USE, ENABLE);
}

```

### 主要代码

思路：

ADC获取到的线电压存在I_Value数组中，然后通过帕克变换和克拉克变换得到Iq，然后通过PID得到Uq。然后Uq再通过帕克逆变换和克拉克逆变换得到Ua,Ub,Uc，驱动无刷电机。

```c
/***********Definition.h*********************/
#ifndef	__DEFINITION_H__
#define __DEFINITION_H__
#include "stm32f10x.h" 
#include "Definition.h"
/*SPI底层索引*/
typedef struct {
    const uint16_t CS;
    const uint16_t CLK;
    const uint16_t MOSI;
    const uint16_t MISO;
    GPIO_TypeDef*  Port; 
} SPI_Pins;
/*MT6816寄存器索引*/
typedef struct {
    const SPI_Pins pins;   

    const uint8_t READ_OP;
    const uint8_t WRITE_OP;
    const uint8_t DATA_REG_1;
    const uint8_t DATA_REG_2;

} MT6816_Device_t;
extern const MT6816_Device_t mt6816;

/*PWM底层索引*/
#define TIM_USE TIM1
#define PWM1_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_TIM1, ENABLE)
#define PWM1_GPIO_CLOCK RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE)
#define PWM1N_GPIO_CLOCK	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE)
#define ARR 4799
#define PSC 0
#define CCR1 2400
#define CCR2 2400
#define CCR3 2400
#define PWM_DeadTime 0x24
#define TIM_CLOCK2 RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2,ENABLE);
#define TIM_USE2 TIM2
#define TIM_Channel TIM2_IRQn
typedef struct {
	const uint16_t PWM1_PIN;
	const uint16_t PWM2_PIN;
	const uint16_t PWM3_PIN;
	const uint16_t PWM1N_PIN;
	const uint16_t PWM2N_PIN;
	const uint16_t PWM3N_PIN;
	GPIO_TypeDef* PWM_Prot;
	GPIO_TypeDef* PWMN_Prot;
}MyPWM_Device_t;
extern const MyPWM_Device_t mypwm;

/*ADC-DMA底层索引*/
typedef struct {
	const uint16_t ADC1_CH1;
	const uint16_t ADC1_CH2;
	const uint16_t ADC1_CH8;
	GPIO_TypeDef* ADC_CH1_CH7_Prot;
	GPIO_TypeDef* ADC_CH8_CH9_Prot;
}MyADC_Drive_t;
extern const MyADC_Drive_t myadc1;




/*FOC参数索引*/
#define constrain(x,low,high) ((x)<(low)?(low):((x)>(high)?(high):(x)))
#define PI 3.14159265358979323846f

typedef struct{
 float Site_Kp;     
 float Site_Kd;
 uint16_t expert_angle;    
 uint16_t deviation_angle;
 float Ele_Kp;   
 float Ele_Ki;
}FOC_PID_drive_t;

typedef struct{
	FOC_PID_drive_t pid;      
  float pole_pairs;   //电机极对数
  int8_t motor_sign;  //电机方向
}FOC_drive_t;
extern FOC_drive_t foc;



#endif
/***********Definition.c*********************/
#include "Definition.h"
const MT6816_Device_t mt6816 = 
{
    .pins = {
        .CS   = GPIO_Pin_4,
        .CLK  = GPIO_Pin_5,
        .MOSI = GPIO_Pin_7,
        .MISO = GPIO_Pin_6,
        .Port = GPIOA
    },
    .READ_OP   = 0x80,
    .WRITE_OP  = 0x00,
    .DATA_REG_1 = 0x03,
    .DATA_REG_2 = 0x04
};

const MyPWM_Device_t mypwm=
{
  .PWM1_PIN=GPIO_Pin_8,
	.PWM2_PIN=GPIO_Pin_9,
	.PWM3_PIN=GPIO_Pin_10,
	.PWM1N_PIN=GPIO_Pin_13,
	.PWM2N_PIN=GPIO_Pin_14,
	.PWM3N_PIN=GPIO_Pin_15,
	.PWM_Prot=GPIOA,
	.PWMN_Prot=GPIOB
};
/*FOC中的pid参数*/
FOC_drive_t foc=
{
	.pid={
		.Site_Kp=0.001,
		.Site_Kd=0.0001,
		.expert_angle=0,
		.deviation_angle=157,
		.Ele_Kp=0.001,
		.Ele_Ki=0
	},
	.pole_pairs=6,
    .motor_sign=-1   //电机方向，+1代表正方向，-1代表负方向
};

const MyADC_Drive_t myadc1=
{
	.ADC1_CH1=GPIO_Pin_0,
	.ADC1_CH2=GPIO_Pin_1,
	.ADC1_CH8=GPIO_Pin_0,
	.ADC_CH1_CH7_Prot=GPIOA,
	.ADC_CH8_CH9_Prot=GPIOB
};
/***********Main.c***************************/
#include "stm32f10x.h"                  // Device header
#include "Definition.h"
#include "MyADC.h"
#include "MT6816.h"
#include "FOC.h"
#include "Serial.h"
#include "Delay.h"
float angle_deg;
int main()
{
	
	AD_Init();
	MT6816_Init();
	FOC_Init();
    Serial_Init();
	countPWM(0.5,0,PI/2.0f);   //定标准
	Delay_ms(100);
	foc.pid.deviation_angle=mt6816_read_angle();  //获取偏离标准
	TIM_Cmd(TIM_USE2,ENABLE);
	while(1)
	{
		Serial_SendString("Site:");Serial_SendNumber(angle_deg,4);
		Serial_SendString("\r\n");
		Serial_SendString("Ia:");Serial_SendNumber((float)I_Value[0]*(3.3/4095)/(4.02*0.02f),4);
		Serial_SendString("\r\n");
		Serial_SendString("Ib:");Serial_SendNumber((float)I_Value[1]*(3.3/4095)/(4.02*0.02f),4);
		Serial_SendString("\r\n");
		Delay_ms(10);
	}
}
void TIM2_IRQHandler(void)  
{
	if(TIM_GetITStatus(TIM_USE2,TIM_IT_Update)==SET)
	{
		angle_deg=mt6816_read_angle();  
		float angle_rad = angle_deg * PI / 180.0f;    //将角度转换为弧度
		float Uq;
		Uq=FOC_ELECPID(0.1,angle_rad);  //电流的pid
		countPWM(Uq,0,electrical_angle(Normalized_angle(angle_deg),foc.pole_pairs));
		TIM_ClearITPendingBit(TIM_USE2,TIM_IT_Update);
	}
}
/***********FOC.h****************************/
#ifndef __FOC_H__
#define __FOC_H__

void FOC_Init(void);
void countPWM(float Uq,float Ud,float angle_el);
void setPWM(float Ua,float Ub,float Uc);
float electrical_angle(float Mechanical_angle,int pole_pair);
float Normalized_angle(float angle);
float FOC_SITEPID(uint16_t angle,uint16_t expert_angle);
float FOC_ELECPID(uint16_t iq_ref,float angle_rad);
#endif



/***********FOC.c***************************/
#include "stm32f10x.h"                 
#include "math.h"
#include "MyPWM.h"
#include "FOC.h"
#include "Timer2.h"
#include "Definition.h"
#include "MT6816.h"
#include "MyADC.h"
float Ua=0,Ub=0,Uc=0;
float voltage_power_supply=12;
float shaft_angle=0;
float voltage_limit=10;     
float Ualpha,Ubeta; 
  

void FOC_Init(void)
{
	PWM_Init();
	Timer2_Init();
}

/*电角度计算*/
float electrical_angle(float Mechanical_angle,int pole_pair)  
{
	return foc.motor_sign*(Mechanical_angle*pole_pair); 
}	

/*角度归一化*/
float Normalized_angle(float angle)
{
	float a=fmod(angle,2*PI);
	return a>=0?a:(a+2*PI);
}

void setPWM(float Ua,float Ub,float Uc)
{
	float dc_a,dc_b,dc_c;
	
	Ua=constrain(Ua,0,voltage_limit);
	Ub=constrain(Ub,0,voltage_limit);
	Uc=constrain(Uc,0,voltage_limit);

	dc_a=constrain(Ua/voltage_power_supply,0.0f,1.0f);       
	dc_b=constrain(Ub/voltage_power_supply,0.0f,1.0f);
	dc_c=constrain(Uc/voltage_power_supply,0.0f,1.0f);

	TIM_SetCompare1(TIM_USE,dc_a*ARR);       
	TIM_SetCompare2(TIM_USE,dc_b*ARR);
	TIM_SetCompare3(TIM_USE,dc_c*ARR);
}

void countPWM(float Uq,float Ud,float angle_el)  
{
	
	angle_el=Normalized_angle(angle_el);  
	
   
	float sin_a=sin(angle_el);               
	float sin_b=sin(angle_el-2.0f*PI/3.0f);
	float sin_c=sin(angle_el+2.0f*PI/3.0f);
    /*帕克逆变换*/
	Ualpha =  -Uq*sin(angle_el); 
    Ubeta =   Uq*cos(angle_el);
	/*克拉克逆变换*/
	Ua = Ualpha + voltage_power_supply/2;
    Ub = (sqrt(3)*Ubeta-Ualpha)/2 + voltage_power_supply/2;
    Uc = (-Ualpha-sqrt(3)*Ubeta)/2 + voltage_power_supply/2;
	
	setPWM(Ua,Ub,Uc);   
}	

/*位置PID*/
float FOC_SITEPID(uint16_t angle,uint16_t expert_angle)
{
	 static float deviationP,deviationD,deviationD_Last;
	 deviationP=expert_angle-angle-foc.pid.deviation_angle;
		if (deviationP > 180.0f) {
        deviationP -= 360.0f;
    } else if (deviationP < -180.0f) {
        deviationP += 360.0f;
    }
	 deviationD_Last=deviationD;
	 deviationD=deviationP;
	 return foc.pid.Site_Kp*deviationP+foc.pid.Site_Kd*(deviationD-deviationD_Last);
}
/*电流闭环*/
void FOC_ELECPID(float *Iq,float *Id)
{
	float Iq1,Iq2,Iq3;
	float Ialpha,Ibeta;
	float Iq;                                                       
	static float deviationP;
	Iq1=(float)I_Value[0]*(3.3/4095)/(4.02*0.02f);  //这里要进行强转
	Iq2=(float)I_Value[1]*(3.3/4095)/(4.02*0.02f);
	Iq3=(float)I_Value[2]*(3.3/4095)/(4.02*0.02f);
	Ialpha=Iq1;
	Ibeta=0.577*(2*Iq2+Iq1);
	*Iq=sin(angle_rad)*Ibeta+cos(angle_rad)*Ialpha;
    *Id=cos(angle_rad)*Ialpha-sin(angle_rad)*Ialpha; 
}



```



# MT6816

## 介绍

**MT6816CT** 是麦歌恩（MagEnc/Magconn）推出的一款 **磁编码器芯片**，集成了磁场感应、信号处理和角度解码模块，适用于旋转位置测量。MT6816CT 通过检测永磁体（一般安装在轴中心）周围的磁场变化来判断角度，适合高精度、无接触式的旋转测量应用。

MT6816属于磁编码器使用的是多轴霍尔传感器阵列，能够精确检测 X、Y 平面的磁通密度：永磁体磁场在平面上随旋转而变化，芯片感应到磁通分量 BxB_xBx 和 ByB_yBy，利用三角函数计算角度：θ=arctan⁡2(By,Bx)，这个角度就是当前磁体相对于初始位置的旋转角度。

## 特点

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/d6c22fe2275941129d296c1e21060108.png)

## 引脚定义

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010202026.png)

## 外围电路设计

![image-20251010201903762](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/image-20251010201903762.png)

## 功能框图

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010202136.png)



## 输出模式

MT6816可以输出ABC,UVW和PWM信号，另外还可以通过4线或3线SPI接口读取14位的绝对角度寄存器。其中ABZ,UVW和SPI接口是互相复用IO引脚的。SPI接口和ABZ/UVW之间是通过HVPP引脚进行配置的，当HVPP接高电平VDD时，相关IO管脚切换至SPI模式；当HVPP接地时，芯片相关IO切换至ABZ或UVW模式。ABZ和UVW模式的切换，由芯片内部相关寄存器控制。4线SPI和3线SPI也是通过芯片内部寄存器进行切换控制的，MT6816出厂默认配置喂4线SPI。

## IO引脚功能配置

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010202714.png)

## SPI通讯

### 接口介绍

MT6816提供了4线或者3线SPI输出

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010202714.png)

### 时序

MT6816的SPI使用模式3（CPOL=1,CPHA=1)传输数据。数据传输开始于CSN的下降沿，结束于CSN的上升沿，MT6816在时钟上升沿采样数据。

> 在主控中初始化SPI模式时记得是模式3

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010203412.png)

### 时序参数

![](C:\学习\笔记\照片\微信截图_20251010203453.png)

> 时间也很重要，要确保通讯有足够的时间。

### 通讯协议

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010203653.png)

MT6816的CSN下降沿激活SPI通信，CSN的上升沿结束SPI啊通信。SCK时钟信号由上位机发送给MT6816，在非通信状态下，清保持SCK为高电平；MOIS（上位机输出，MT6816输入）和MISO（上位机输入，MT6816输出）是SPI接口的两路数据信号，数据都是在时钟信号SCK的下降沿发生改变，所以推荐使用SCK时钟信号的上升沿对数据进行采样。

比特0：读写标志位。低电平为写操作，此时数据DI7~DI0写入芯片；高电平为读操作，此时从芯片读书数据DO7-DO0.

比特1-7：地址A6-A0。寄存器操作地址。

比特8-15：数据DI7-DI0(写模式)。会被写入芯片的数据

比特89-15：数据DO7-DO0（读模式）。从芯片读出的数据。

## 读取角度数据

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010204618.png)

## 角度数据寄存器

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010204643.png)

> 值得注意的是，角度的数据是由两个寄存器共同存储，获取到两个寄存器的数据要进行合并得到14bit的数据，然后代入公式。
>
> Angle=(((uint16_t)data[0x03] << 8) | data[0x04])>>2;

## 角度换算公式

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251010204717.png)



## 获取程序

### 大框

MT6816采用SPI通信，所以大致分为原理层和底层，采用江科大的SPI程序，因为MT6816要使用SPI模式3，根据MT6816寄存器0x03和0x04两部分以及读写位，所以发送的数据是0x08|0x03，也就是读0x03寄存器，和0x08|0x04，读0x04寄存器，然后将读的值根据手册内容合并一下。

```c
/*----------------------------------main.h-------------------------------------------*/
#include "stm32f10x.h"                  // Device header
#include "MT6816.h"
int main()
{
	MT6816_Init();
	uint16_t Angle;
	while(1)
	{
		Angle=mt6816_read_angle();  //这里获取的是原始数据，没有计算
	}
}
/*-------------------------------Definiton.h--------------------------------*/
#ifndef	__DEFINITION_H__
#define __DEFINITION_H__
#include "stm32f10x.h" 
//SPI引脚结构体定义
typedef struct {
    const uint16_t CS;
    const uint16_t CLK;
    const uint16_t MOSI;
    const uint16_t MISO;
    GPIO_TypeDef*  Port; 
} SPI_Pins;
//mt6816寄存器结构体定义
typedef struct {
    const SPI_Pins pins;   

    const uint8_t READ_OP;
    const uint8_t WRITE_OP;
    const uint8_t DATA_REG_1;
    const uint8_t DATA_REG_2;

} MT6816_Device_t;
//声明结构体
extern const MT6816_Device_t mt6816;

#endif


/*-------------------------------Definiton.c--------------------------------*/
#include "Definition.h"
//mt6816底层驱动引脚及寄存器驱动，从应用层到底层
const MT6816_Device_t mt6816 = 
{
    .pins = {
        .CS   = GPIO_Pin_4,
        .CLK  = GPIO_Pin_5,
        .MOSI = GPIO_Pin_7,
        .MISO = GPIO_Pin_6,
        .Port = GPIOA
    },
    .READ_OP   = 0x80,
    .WRITE_OP  = 0x00,
    .DATA_REG_1 = 0x03,
    .DATA_REG_2 = 0x04
};



/*-------------------------------MySPI.h--------------------------------*/
#ifndef __MYSPI_H__
#define __MTSPI_H__
void MySPI_Init(void);
void MySPI_Start(void);
void MySPI_Stop(void);
uint8_t MySPI_SwapByte(uint8_t ByteSend);
#endif


/*-------------------------------MySPI.c-------------------------------------*/
#include "stm32f10x.h"                  // Device header
#include "Definition.h"
 void MySPI_W_SS(uint8_t BitValue)
{
	GPIO_WriteBit(mt6816.pins.Port, mt6816.pins.CS, (BitAction)BitValue);		
}

void MySPI_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_SPI1, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = mt6816.pins.CS;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_2MHz;
	GPIO_Init(mt6816.pins.Port, &GPIO_InitStructure);				
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = mt6816.pins.CLK| mt6816.pins.MOSI;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(mt6816.pins.Port, &GPIO_InitStructure);
	
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin =mt6816.pins.MISO;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(mt6816.pins.Port, &GPIO_InitStructure);				
	
	SPI_InitTypeDef SPI_InitStructure;					
	SPI_InitStructure.SPI_Mode = SPI_Mode_Master;		
	SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex;	
	SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b;		
	SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB;	
	SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_256;  //SPI通信速度	
	SPI_InitStructure.SPI_CPOL = SPI_CPOL_High;	   //这里要根据mt6816的SPI模式来进行选择			
	SPI_InitStructure.SPI_CPHA = SPI_CPHA_2Edge;   //这里要根据mt6816的SPI模式来进行选择		
	SPI_InitStructure.SPI_NSS = SPI_NSS_Soft;				
	SPI_InitStructure.SPI_CRCPolynomial = 7;			
	SPI_Init(SPI1, &SPI_InitStructure);						
	

	SPI_Cmd(SPI1, ENABLE);								
	

	GPIO_WriteBit(mt6816.pins.Port, mt6816.pins.CS, 1);
	
}
uint8_t MySPI_SwapByte(uint8_t ByteSend)
{
	while (SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_TXE) != SET);	
	
	SPI_I2S_SendData(SPI1, ByteSend);								
	
	while (SPI_I2S_GetFlagStatus(SPI1, SPI_I2S_FLAG_RXNE) != SET);	
	
	return SPI_I2S_ReceiveData(SPI1);							
}

void MySPI_Start(void)
{
	GPIO_WriteBit(mt6816.pins.Port, mt6816.pins.CS,0);				
}

void MySPI_Stop(void)
{
	GPIO_WriteBit(mt6816.pins.Port, mt6816.pins.CS, 1);				
}
	
/*------------------------------MT6816.h--------------------------------------*/
#ifndef __MT6816_H__
#define __MT6816_H__
 void MT6816_Init(void);
 uint16_t mt6816_read_angle(void);

#endif
/*-----------------------------MT6816.c--------------------------------------*/
#include "stm32f10x.h"                  // Device header
#include "MySPI.h"
#include "Definition.h"

void MT6816_Init(void)
{
	MySPI_Init();
}

uint16_t mt6816_read_angle(void)
{
  uint8_t Data1 = 0x00;
	uint8_t Data2 = 0x00;
	
  MySPI_Start();
  MySPI_SwapByte(READ_OP|MT6816_Data1);   //读指令+寄存器地址         
  Data1 =MySPI_SwapByte(0x00);    
  MySPI_Stop();
  MySPI_Start();
  MySPI_SwapByte(READ_OP|MT6816_Data2);   //读指令+寄存器地址  
	Data2 =MySPI_SwapByte(0x00);
  MySPI_Stop();
	uint16_t MyAngle = (Data1<<8)|Data2;   //合并数据 
 
	return MyAngle>>2;  //得到0-360度的角度值
}

```

## 使用

编码器测的角度是0-4095循环变化的，所以核心在于怎么计算圈数。

### 代码

```c
//这段代码用的都是弧度
float Get_angle(float angle_current)  
{
    float full_rotations;
    static float angle_last;
    float angle_Diff=angle_current-angle_last;
    if(abs(angle_Diff)>(0.8f*2PI)) full_rotations+=(angle_Diff>0)?-1:1;  //如果角度的绝对值大于一圈的80%，则圈数加1
    angle_last=angle_current
    return (float)full_rotations*2PI+angle_current;
}

```



## 校准

### 零位校准



# 驱动板

## 零散知识

### GND

- **PGND (Power Ground)**：功率地，给大电流、高功率器件走的“地”。
- **AGND (Analog Ground)**：模拟地，给微弱、敏感的模拟信号走的“地”。
- **DGND (Digital Ground)**：数字地，给开关频繁、噪声大的数字信号走的“地”。

#### 1. PGND (Power Ground) - 功率地

##### **作用**

PGND 是为大功率、大电流的电路部分提供的电流回流路径。这些电路通常包括：

- 电源输入/输出
- 电机驱动器
- 继电器、开关
- 高功率LED驱动电路
- DC-DC转换器（开关电源）

##### **特点**

- **电流大**：承载着安培级别的电流。
- **噪声大**：由于电流大且变化快（如开关电源），PGND 上会产生显著的电压波动和电磁干扰（EMI）。
- **阻抗低是关键**：为了减少压降和发热，PGND 的走线或铺铜必须尽可能短、粗。

**可以把 PGND 想象成城市里的“主干道”或“高速公路”，上面跑的都是重型卡车，虽然路很宽，但震动和噪音也很大。**

#### 2. AGND (Analog Ground) - 模拟地

##### **作用**

AGND 是为微弱、高精度的模拟信号电路提供的稳定参考电位。这些电路对噪声极其敏感，包括：

- 传感器接口（如温度、压力传感器）
- 音频电路（麦克风、放大器、ADC前端）
- 精密测量仪器（如万用表、示波器前端）
- 射频电路

##### **特点**

- **电流小**：通常处理的是微安（μA）或毫安级别的信号电流。
- **对噪声极度敏感**：微小的噪声（毫伏甚至微伏级别）都可能淹没真实信号，导致测量错误或音频出现杂音。
- **参考点是关键**：AGND 的核心作用是提供一个“纯净”的0V参考基准。它自身的电位必须非常稳定。

**可以把 AGND 想象成医院的“手术室”或录音棚，要求绝对安静、干净，任何一点噪音都可能造成严重后果。**



#### 3. DGND (Digital Ground) - 数字地

##### **作用**

DGND 是为数字逻辑电路提供的回流路径。这些电路的特点是信号在高低电平之间快速切换。包括：

- 微控制器（MCU）、CPU
- 内存、逻辑门电路
- 数字通信接口（如SPI, I2C, UART）

##### **特点**

- **噪声大**：数字信号的快速开关（上升/下降沿）会产生高频噪声和瞬态大电流。这是电路板上的主要噪声源之一。
- **电流不连续**：电流是脉冲式的，而不是连续的。
- **需要低阻抗回路**：为了保证信号完整性，DGND 也需要低阻抗，但其主要挑战是抑制高频噪声。

**可以把 DGND 想象成城市里的“商业区”，人来人往，活动频繁，虽然单个人（电流）不大，但整体环境非常嘈杂。**

#### 核心区别与处理原则：为什么要分开？

**核心矛盾：** 噪声。

- PGND 和 DGND 是**噪声源**。
- AGND 是**噪声受害者**。

如果将它们简单地连接在一起，PGND 和 DGND 上的噪声会通过公共的“地”路径直接耦合到 AGND 上，污染了模拟电路的参考基准，导致系统工作不正常。

#### 作用

**处理原则：分而治之，适当连接。**

1. **物理隔离**：在PCB布局时，将模拟电路、数字电路和功率电路分区布局。AGND、DGND、PGND 也分别进行铺铜，形成独立的“岛屿”。
2. **单点接地 / 星型接地**：这是最关键的一步。虽然物理上分开，但它们最终需要一个共同的参考点，否则整个系统电位会漂移。正确的做法是：
    - 将所有地（AGND, DGND, PGND）通过**一个点**连接在一起。
    - 这个点通常选择在电源入口处，或者ADC/DAC芯片的参考地引脚附近。
    - 这样，大电流（PGND）和高频噪声（DGND）就不会流经敏感的AGND区域，而是直接“回家”（回到电源源头）。
3. **使用磁珠或0欧姆电阻桥接**：在某些设计中，特别是DGND和AGND之间，可以使用一个**磁珠**或**0欧姆电阻**进行连接。
    - **磁珠**：对高频噪声呈现高阻抗，能有效阻隔DGND的高频噪声窜入AGND，同时对直流（DC）呈现低阻抗，保证了直流电位的统一。
    - **0欧姆电阻**：相当于一根导线，但它在布局上提供了一个灵活的“断点”，方便调试时断开或测量。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/97e51e05985048d397dc3efff298a5f3.png)

1、为什么要进行数字地和模拟地隔离
①因为地线阻抗不是0，当电流流经它时就会有电压降，使得各处的参考地电压不再相同，这样就相当于产生了波动
②由于数字电路一般是PWM波或者方波，存在大量谐波，通过地线传播，模拟信号又对噪声很敏感，很容易干扰到信号的质量。有一句话形容得很形象，数字电路的地像波涛汹涌的大海，模拟电路的地像平静的湖面，如果直接连在一起，不就是变成一滩浑水了。
③反过来也是，当模拟信号为高频或强电信号时，比如220V或者PA信号，也会影响到数字电路的正常工作。
④将数字地和模拟地分开，就是为了将数字地和模拟地的共地电阻降到最小。
![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251015123719.png)



## 布局建议

 首先关于散热问题，推荐采用打散热过孔，一般在焊盘上打若干个阵列的过孔对散热的效果提升还是蛮明显的，同时孔的外径和内径也有讲究，一般外径20mil对应内径8mil。关于这个盘中孔的操作，嘉立创提供了相当多的不同工艺，如果想要讲究极致的性能，选择最贵的那款过孔塞铜浆绝对没错。具体的工艺介绍：[嘉立创推出IC散热器件的完美解决方案：“铜浆塞孔+电镀盖帽”](http://mp.weixin.qq.com/s?__biz=MzA4MTExNjgyNQ==&mid=2452120018&idx=1&sn=44be60c466c57f8437c5bbdb10922a8c&chksm=88452a24bf32a332fa36d0c6183634bea027c2879ad3473d0ec796b9c5ab099a4d4064b2e517&mpshare=1&scene=23&srcid=1022B7Zf7TZyxAwyXPhjKRWb&sharer_sharetime=1666451254484&sharer_shareid=eeb99ebe496cfb5387956bc9a03ae97d#rd)

 与此同时，大面积的铺铜也可以加速热量的散发，但是铺铜也有技巧，最好铺放连续不断的铜，连续的铜热量散发的路径会更加宽广一些，具体如下图所示。同时不同的铜厚也会对散热产生不同的效果，按道理上来说铜厚一般越厚散热能力越强。大家可以下单PCB的时候尝试一下2盎司的铜厚甚至更高，看看会不会有什么不同。

总结散热设计。主要是以下的六点：

 i.散热过孔是最高效率的散热路径。

 ii.尽量使用连续不断的铺铜，有利于散热。

 iii.尽可能使用1.5或者2盎司的铜。

 iv.过孔与铜的连接采用direct-connect。

 v.使用8mil内径20mil外径的过孔用来防止多余的渗锡。（excessive sold wicking）

 vi.不同的层之间采用阵列过孔连接是最小[阻抗](https://so.csdn.net/so/search?q=阻抗&spm=1001.2101.3001.7020)的连接办法。

**2.摆放与连接**

 过孔尽可能不要去截断完整的地（ground plane），如下图所示

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/76aa7c70c29432986d45bcd03183957e.png)

驱动线要尽可能粗并且尽可能的短，同时尽可能使用泪滴，门极驱动线的宽度在1盎司铜的情况下要至少20mil粗，如果要走更大的电流就要更粗。走线要尽可能走直线，如果遇到要转弯的也要尽量走钝角线，直角的走线会反射信号造成干扰，

 走线也要尽可能平行，特别是信号线，尽可能长并且尽可能相互平行。元器件摆放也是，要尽可能相互平行摆放，可能有人会疑惑了，这两个建议不是有点相互冲突的吗？是的，电路板设计其实说到底就是对设计规则的取舍，选择最适合自己设计的规则。满足所有所谓的布板建议，那是不现实的。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/deb8a5357e42daafb19ab75101557719.png)

 **3.母体电容、旁路电容以及各种电容**

 母体电容的作用是什么呢？首先我们要知道大电源供电并不是理想的恒定的，就像例如24V，里面会含有很多次的高次[谐波](https://so.csdn.net/so/search?q=谐波&spm=1001.2101.3001.7020)，导致这个电压会有很多毛刺，而大容量的母体电容就可以滤除一些低频的毛刺。同时它还可以存储能量，在驱动器有大电流需求的时候供电给驱动器用于输出。

 关于母体电容的摆放是有讲究的，一般推荐摆放在供电处附近。并且通过多个过孔与power层相连接，除此之外最好选择低等效电阻（[ESR](https://so.csdn.net/so/search?q=ESR&spm=1001.2101.3001.7020)）的母体电容。同时，自举电容最好尽可能的靠近电机驱动装置，例如MOS管与驱动芯片。在芯片的Vin引脚附近最好也可以放一个电容，既可以防止电磁干扰（EMI），也可以让芯片的供电更加稳定。

 先连接容值小的电容后连接容值大的电容，因为随着电容容值的增加，PCB布板、线路等因素带来的电感会显得微不足道。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/517ac579a6275f90d468251923adce1f.png)

 而旁路电容的布局同样也有很多讲究，首先在旁路电容与活跃元器件的连接之间不能使用过孔，具体如下图。同时保证旁路电容与对应的工作元器件要摆在同一层，减小过孔带来的电感影响。旁路电容与IC距离越近越好，一般小于0.2cm为宜。同时走线的长度与宽度之比不能大于3：1

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/ae3e60be6762dd16f62bbd4d3e6caa77.png)

当然！FOC（Field-Oriented Control，磁场定向控制）无刷电机驱动板的PCB布局是一个极具挑战性的任务，因为它完美地融合了**大功率、高噪声的功率部分**和**微弱、敏感的模拟/数字控制部分**。布局的好坏直接决定了驱动板的性能、稳定性和可靠性。

### 核心思想：分区与隔离

这是FOC驱动板布局的**第一黄金法则**。你必须将电路板清晰地划分为几个功能区，并严格管理它们之间的连接。
1.  **功率区**：
    *   **包含**：MOSFET（或IGBT）、三相桥驱动电路、母线电容、电流采样电阻、电源输入接口。
    *   **特点**：高电压（如12V/24V/48V）、大电流（数安培到数十安培）、高开关频率噪声（PWM频率通常在10kHz-100kHz）。
2.  **控制区**：
    *   **包含**：MCU（微控制器）、晶振、调试接口（SWD/JTAG）、通信接口（UART/CAN/USB）、控制逻辑电源（3.3V/5V LDO）。
    *   **特点**：低电压、低功耗、对噪声极其敏感。
3.  **接口区**：
    *   **包含**：电机接线端子、电源输入端子、编码器接口、信号输入输出等。
    *   **特点**：连接外部世界，容易引入干扰。
---
### 布局关键点详解
#### 1. 功率区的布局：电流路径为王
功率区的核心是**最小化高频大电流回路的面积**。这直接关系到EMC和开关损耗。
*   **三相桥的布局**：
    *   将6个MOSFET（通常是3个半桥）紧密地排列在一起。
    *   上管和下管应尽可能靠近，以缩短它们之间的连接路径。
    *   **母线电容的放置至关重要**：它必须**紧贴**MOSFET的输入端（ Drain for high-side, Source for low-side）。目标是让从母线电容到MOSFET，再从MOSFET返回母线电容的**高频电流环路面积最小**。
    
*   **电流采样电阻**：
    *   采样电阻必须放置在**下管的源极到地**的路径上。
    *   采样电阻到MOSFET源极的连线要**短而粗**。
    *   **采样信号的走线是重中之重**：必须使用**差分走线（Kelvin连接）**。即从采样电阻的两端直接引线到运放或MCU的ADC引脚，**绝对不能**与功率地共用一段走线，否则功率电流的压降会严重污染采样信号。
*   **散热考虑**：
    *   MOSFET是主要发热源，其下方的PCB覆铜要尽可能大，并多打散热过孔，连接到背面的散热铜层。
#### 2. 接地策略：隔离与汇流
这是FOC布局的**第二黄金法则**，也是最容易出错的地方。
*   **星型接地**：
    *   **功率地**：所有功率元件的地（母线电容地、电流采样电阻地、MOSFET源极）连接在一起，形成一个**PGND网络**。
    *   **控制地**：所有控制元件的地（MCU地、LDO地、通信地）连接在一起，形成一个**DGND/AGND网络**。
    *   **单点连接**：PGND和DGND/AGND**只在一个点**连接。这个最佳连接点是**母线电容的负极**。这样，所有大电流的噪声都会直接流回母线电容，而不会污染敏感的控制地。
    
#### 3. 控制区的布局：安静与稳定
*   **MCU放置**：将MCU放在控制区的中心，远离功率区和接口区。
*   **晶振**：必须**紧贴MCU的晶振引脚**，晶振下方不要走任何信号线，最好铺地屏蔽。晶振的负载电容也要靠近放置。
*   **去耦电容**：
    *   **MCU电源引脚**：每个VDD引脚旁边（距离<5mm）放置一个100nF的陶瓷电容，然后稍远一点放置一个较大的10uF电容。
    *   **驱动芯片电源**：同样需要紧靠芯片放置去耦电容。
*   **信号线**：
    *   PWM信号线从MCU到驱动芯片，要远离功率走线，最好有地线屏蔽。
    *   编码器信号线（特别是ABZ差分信号）是敏感信号，要走差分线，并远离电机线和PWM线。
#### 4. 电源布局
*   **主电源路径**：从电源输入端子到母线电容，再到MOSFET的路径要**短、粗、直**。使用覆铜或宽走线。
*   **隔离电源**：如果使用隔离电源（如隔离的DC-DC或光耦）来为控制区供电，可以简化接地问题，但成本更高。若不隔离，则必须严格遵守星型接地。
---
### 布局检查清单
在完成布局后，用这个清单检查一遍：
1.  [ ] **分区明确**：功率、控制、接口区是否清晰分开？
2.  [ ] **功率回路最小**：母线电容到MOSFET的环路面积是否尽可能小？
3.  [ ] **电流采样**：是否使用了差分走线？
4.  [ ] **接地策略**：是否实现了星型接地？PGND和DGND是否只在母线电容处单点连接？
5.  [ ] **去耦电容**：所有IC的电源引脚旁是否都紧贴着放置了去耦电容？
6.  [ ] **晶振**：是否紧贴MCU，且下方干净？
7.  [ ] **敏感信号**：PWM、编码器、采样信号是否远离了噪声源？
8.  [ ] **散热**：MOSFET下方是否有足够的覆铜和过孔？
### 总结
FOC驱动板的PCB布局是一个**权衡的艺术**，但核心原则不变：**让大电流在它自己的小圈子里折腾，别让它去干扰安静的小信号区**。只要严格遵守分区和星型接地的原则，你的驱动板就已经成功了一大半。记住，一个好的布局可以省去后续大量的调试和EMC整改时间

## 功能与元器件

第一次先只画驱动板，驱动板内部不包含主控，这样可以方便排查问题，以及防止有问题烧坏主控芯片。
一共分为3部分

电源类：开关降压芯片，滤波电容

接口类：电流采集接口、主控接口、电源接口

芯片类：电机驱动芯片(DRV8303)，电流采集芯片（TI INA240）

### 元器件选型

MOS管：IRF3205

电机驱动芯片：DRV8303

电流采集芯片（INA240)



## DRV8303

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251015200658.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/3918f6c4362c4208be2c999634f3b01c.png)

DRV8303是一个驱动芯片，主要是将32的信号放大驱动MOS管，所以仍旧需要6个MOS管组成六桥臂逆变器。

在这里不适用BUCK降压引脚

### 引脚定义

> 1.SDI：过流过温引脚
>
> 2.SDO：错误引脚
>
> 3.DTC:死区时间的设置
>
> 4.nSCS：SPI的CS引脚
>
> 5.SDI:SPI
>
> 6.SDO:SPI
>
> 7.SCL:SPI
>
> 8.DC_CAL:内部分流放大器做偏移校正的引脚
>
> 9：GVDD：电荷泵
>
> 10，11.CP1,CP2：充电泵
>
> 12.EN_GATE:启用门驱动器和电流放大器，接个GPIO
>
> 13、14、15、16、17、18、19：PWM引脚
>
> 20.REF:接AVCC并且通过2.2UF/>6.3V接到AGND，这是芯片的参考电压，偏置电压也是REF的
>
> 21.22.SO1、SO2：电流放大器的输出，要接56欧的电阻
>
> 23.AVDD:连接电容1UF/>30V接地
>
> 24.AGND:直接接模拟地
>
> 23.DVDD：要接DVDD，并通过1UF/>6.3V接到到AGND
>
> 45.VDD_SPI:数字供电

> 如果想要在错误或者过问过流的时候让LCD发光，那么5，6引脚应该要接PMOS管。
>
> DTC接的电阻代表死区时间的多少，但是我们自己在32的定时器中设置了死区时间，所以DTC给了1欧的电阻就可以，也可以大一点没关系。
>
> DC_CAL高的时候就进行校正，底电平的时候就不校正，随便接一个GPIO就可以
>
> GVDD接个电容就可以，值从数据手册可以得出，要接2.2uf的电容
>
> CP1和CP2之间要接陶瓷电容
>
> 

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/1d35efa2ea3b49f6b520f772d1b89aca.png)

其中G为电压放大倍数，可通过SPI设置，可设置有10，20，40，80V/V。

### MP6536

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251016164418.png)

vsp工作电流最大3.5mA

欠压锁定电压2.2V

最小PWM脉冲速度30ns，Vpwm=0V至5V（目前还不清楚3.3V是否可以）

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20251016164940.png)

SW引脚接电机，是电机的输出引脚

VSP是工作电压
