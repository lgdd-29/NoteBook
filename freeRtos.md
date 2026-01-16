# 介绍

实时操作系统（RTOS，Real Time Operating System）是一种专门为嵌入式系统设计的操作系统，其主要特点是能够快速响应外界事件或数据，并在规定的时间内完成处理，以控制生产过程或对处理系统做出快速响应。RTOS的主要目标是提供可预测性和实时性，确保系统能够在严格的时间限制下运行。
RTOS与通用操作系统（如Windows或Linux）不同，它专注于处理实时任务，如控制、通信和数据采集。以下是RTOS的一些关键特点和应用：

1. **多任务处理**：RTOS能够同时执行多个任务，这些任务可以是硬实时（必须在特定时间内完成）或软实时（最好在特定时间内完成）。RTOS管理任务的调度和优先级，确保高优先级任务获得足够的处理时间。
2. **实时响应**：RTOS设计的目标之一是实时响应。嵌入式系统通常需要在特定时间内响应外部事件，如传感器数据或用户输入。RTOS可以确保任务按照其优先级处理，满足实时性要求。
3. **任务同步和通信**：RTOS提供了各种机制来实现任务之间的同步和通信，如信号量、消息队列和互斥锁。这些机制有助于避免竞态条件和确保数据的一致性。
4. **节省资源**：RTOS通常被设计成轻量级，以减小内存占用和处理器负载。这对于资源受限的嵌入式系统非常重要。
5. **可扩展性**：RTOS通常具有可扩展性，允许开发者根据项目需求添加新的任务或功能模块。
RTOS广泛应用于工业自动化、军事、电力、新能源等领域，常见的RTOS系统包括ucosii、rtthread、MsgOS等。

## 任务调度

## 简介

FreeRTOS一共支持三种任务调度方式

### 抢占式调度

主要是针对优先级不同的任务，每个任务都有一个优先级，优先级高的任务可以抢占优先级低的任务

### 时间片调度

主要针对优先级相同的任务，当多个任务的优先级相同时，任务调度器会在每一次系统时钟节拍到的时候切换任务

### 协程式调度

当前执行任务将会一直运行，高优先级的任务不会抢占低优先级的任务。实时性较差且官方不再跟新

## 抢占式调度

抢占式调度（Preemptive Scheduling）是一种在操作系统中使用的调度策略，允许操作系统内核强制暂停当前正在运行的任务（也称为进程或线程），并将CPU的控制权转移给另一个更高优先级或更紧急的任务。以下是抢占式调度的一些关键特点：
1. **优先级**：在抢占式调度中，每个任务都被分配一个优先级。当一个新的任务进入就绪状态，且其优先级高于当前运行任务的优先级时，调度器可以抢占当前任务的执行，让更高优先级的任务运行。
2. **实时性**：抢占式调度特别适用于实时操作系统（RTOS），因为它可以保证高优先级的任务能够及时得到处理，满足硬实时系统的要求。
3. **响应时间**：由于抢占式调度可以快速切换到更高优先级的任务，因此系统的响应时间通常较短。
4. **上下文切换**：在抢占式调度中，当任务被抢占时，操作系统需要进行上下文切换，即保存当前任务的状态，并加载新任务的状态。
5. **动态性**：抢占式调度是动态的，任务调度可以在任何时刻发生，不仅限于任务阻塞或完成时。
抢占式调度算法的一些例子包括：
- **基于优先级的抢占式调度**（Priority-Based Preemptive Scheduling）：任务根据其优先级进行调度，高优先级任务可以抢占低优先级任务的执行。
- **最早截止时间优先（EDF）**：适用于实时系统，任务根据其截止时间的早晚进行调度，更早截止时间的任务可以抢占当前运行的任务。
- **速率单调调度（Rate-Monotonic Scheduling, RMS）**：在实时系统中，任务的优先级通常与其周期成反比，周期越短的任务优先级越高，可以抢占周期更长的任务。
抢占式调度与协作式调度（Cooperative Scheduling）相对，后者依赖于任务主动释放CPU，没有抢占机制，因此可能导致低优先级任务长时间占用CPU，影响系统性能和响应时间。

## 时间片调度

时间片调度（Round Robin Scheduling）是一种基于时间片的抢占式调度算法，广泛用于分时系统和某些实时操作系统中。在时间片调度算法中，每个任务（或进程）被分配一个固定的时间段，称为时间片（time quantum 或 time slice），以下是其主要特点和操作方式：
1. **时间片**：每个任务轮流使用CPU，但每次只能使用一个时间片。时间片的长度通常是固定的，但也可以根据系统的需求或任务的特性动态调整。
2. **轮流执行**：系统中的所有就绪任务按某种顺序（如就绪队列）轮流执行。当一个任务的时间片用完时，它会被置于就绪队列的末尾，而下一个任务开始执行。
3. **抢占式**：如果时间片未用完而更高优先级的任务变为就绪状态，当前任务可能会被抢占，让更高优先级的任务运行。
4. **公平性**：时间片调度算法试图为所有任务提供公平的CPU时间，因为它确保每个任务都有机会在一定时间内运行。
5. **上下文切换**：当任务的时间片用完或被更高优先级的任务抢占时，操作系统执行上下文切换，保存当前任务的状态，并加载下一个任务的状态。
时间片调度的步骤通常如下：
- 任务被添加到就绪队列。
- 调度器选择队列中的第一个任务开始执行。
- 任务执行直到其时间片用完或被阻塞。
- 如果时间片用完，任务被放回就绪队列的末尾。
- 如果任务被阻塞（例如等待I/O），调度器将选择队列中的下一个任务。
- 过程重复，直到所有任务完成。
时间片调度算法适用于以下情况：
- 当多个任务需要共享CPU时。
- 当任务响应时间很重要，但不需要严格的实时保证时。
- 在多用户环境中，每个用户都需要感觉到自己独占系统资源。
时间片调度算法的一个主要缺点是它可能增加上下文切换的开销，特别是当时间片非常短时。此外，它可能不适合所有类型的实时任务，因为实时任务可能需要更严格的响应时间保证。

# STM32移植

在代码文件中创建一个文件夹“FreeRTOS"用于存放FreeRTOS的代码

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-1.png)

将下图选中的代码复制到上面创建的文件夹中（共有9个选项，不要漏了）

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-2.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-3.png)

在自己的工程的 > FreeRTOS > portable 中删减剩以下三个文件即可

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-4.png)

将源码中 > FreeRTOS > Demo > CORTEX_STM32F103_Keil > FreeRTOSConfig.h 复制到自己工程的 > FreeRTOS > include 中

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-5.png)

打开自己的工程，新建分组 FreeRTOS_CODE 和 FreeRTOS_PORTTABLE，并将以下文件添加进去。

**FreeRTOS_CODE**分组里添加 FreeRTOS 文件夹中除了stream_buffer.c的其余六个文件。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-6.png)

**FreeRTOS_PORTTABLE**分组需要添加 FreeRTOS > portable > MemMang > heap_4.c 文件和 FreeRTOS > portable > RVDS > ARM_CM3 > port.c 文件

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-7.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-8.png)

记得将文件路径添加到工程中

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-9.png)

至此，移植所需的文件已经全部就位，开始下一步操作

将以下代码段注释掉（ void SVC_Handler(void)，void PendSV_Handler(void) ，void SysTick_Handler(void)  ）

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-10.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-11.png)

将以下代码粘贴到FreeRTOSConfig.h中

```c
#define xPortPendSVHandler  PendSV_Handler
#define vPortSVCHandler     SVC_Handler
#define xPortSysTickHandler SysTick_Handler
```

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-7-21-12.png)

## 例子

将以下代码复制到main文件中， 现象为A0端口和C13端口的灯各自闪烁不干扰。

```c
#include "stm32f10x.h"                  // Device header
#include "FreeRTOS.h"
#include "task.h"
 
void LED1_Task(void *pvParameters)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOC, &GPIO_InitStructure);
	while(1)
	{
		GPIO_ResetBits(GPIOC, GPIO_Pin_13);
		vTaskDelay(pdMS_TO_TICKS(1000));
		GPIO_SetBits(GPIOC, GPIO_Pin_13);
		vTaskDelay(pdMS_TO_TICKS(1000));
	}
}
 
void LED2_Task(void *pvParameters)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	while(1)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_0);
		vTaskDelay(pdMS_TO_TICKS(500));
		GPIO_ResetBits(GPIOA, GPIO_Pin_0);
		vTaskDelay(pdMS_TO_TICKS(500));
	}
}
 
int main(void)
{
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
	xTaskCreate(LED1_Task, "LED1_Task", configMINIMAL_STACK_SIZE, NULL, 1, NULL);
	xTaskCreate(LED2_Task, "LED2_Task", configMINIMAL_STACK_SIZE, NULL, 1, NULL);
	vTaskStartScheduler();
	while(1)
	{
	}
}
 
 
```

# 系统配置文件

作用：对FreeRTOS进行功能配置和裁剪，以及API函数的使能

> 系统配置文件中基本都是宏定义

## 分类

### INCLUDE

配置FreeRTOS中可选的API函数

### config

完成FreeRTOS的功能配置和裁剪

### 其他配置项

PendSV宏定义，SVC宏定义

## 详细

各种函数的例子会在task.h文件中

各种宏定义会在FreeRTOS.h和FreeRTOSConfig.h文件中

# 函数

## 临界区

```c
taskENTER_CRITICAL();  //进入临界区（关闭中断）
taskEXIT_CRITICAL();   //退出临界区（打开zhong
taskENTER_CRITICAL_FROM_ISR();	//中断级进入临界段
taskEXIT_CRITICAL_FROM_ISR();	//中断级退出临界段
```

临界区：不被打断执行

临界段代码叫做临界区，是指那些必须完整运行，不能打断的代码块

1. 外设（如严格按照时序初始化的外设：IIC、SPI）
2. 系统（系统自身需求）
3. 用户需求

> FreeRTOS在进入临界段代码的时候需要关闭中断，当处理完临界段代码以后再打开中断

## 任务

### 任务调度

```c
vTaskStartScheduler();  //任务调度开始
```

### 任务切换

```c
portYIELD_FROM_ISR(xYieldRequired)
```

### 动态创建

```c
xTaskCreate();                 //动态方式创建任务

/*函数定义*/
BaseType_t xTaskCreate
( 
    TaskFunction_t pxTaskCode,  //指向任务函数的指针
    const char * const pcName,  //任务名字
    const configSTACK_DEPTH_TYPE usStackDepth, //任务堆栈大小，字为单位
    void * const pvParameters,   //传递给任务函数的参数
    UBaseType_t uxPriority,      //任务优先级
    TaskHandle_t * const pxCreatedTask    //任务句柄，任务的任务控制块
)
//返回值
pdPASS            //任务创建成功
errCOULD_NOT_ALLOCATE_REQUIREN_MENORY   //任务创建失败
 /*例子*/
xTaskCreate(LED1_Task, "LED1_Task", configMINIMAL_STACK_SIZE, NULL, 1, NULL);
```

> 任务的任务控制块以及任务的栈空间所需的内存，均由FreeRTOS从FreeRTOS管理的堆中分配

#### 任务流程

1. 将宏configSUPPORT_DYNAMIC_ALLOCATION配置为1
    1. 宏configSUPPORT_DYNAMIC_ALLOCATION在FreeRTOS.h文件中

2. 定义函数入口参数
    1. 任务函数的指针
    2. 任务名字 
    3. 定义任务堆栈大小
    4. 传递给任务函数的参数
    5. 任务优先级
    6. 任务句柄（定义一个任务句柄，然后在任务创建中输入任务句柄，代表着这个任务句柄指向这个任务）

3. 编写任务函数

> 此函数创建的任务会立刻进入就绪态，由任务调度器调度运用

#### 内部实现

1. 申请堆栈内存&任务控制块内存
2. TCB结构体成员赋值
3. 添加新任务到就绪列表中

#### 例子

```c
#include "stm32f10x.h"               
#include "FreeRTOS.h"
#include "task.h"
/*START_TASK任务配置*/
void start_task( void * pvParameters );
#define START_TASK_SIZE 128 
#define START_TASK_PRIO 1
TaskHandle_t start_task_handler;

/*LED1_Task任务配置*/
void LED1_Task(void *pvParameters);
#define LED1_TASK_SIZE 128  
#define LED1_TASK_PRIO 2
TaskHandle_t LED1_task_handler;

/*LED2_Task任务配置*/
void LED2_Task(void *pvParameters);
#define LED2_TASK_SIZE 128  
#define LED2_TASK_PRIO 2
TaskHandle_t LED2_task_handler;

/*********************************************************/

//函数入口--->任务创建
void freertos_demo(void)
{
	xTaskCreate( 		
				(TaskFunction_t) start_task,
                  ( char * ) "start_task", 
                  ( configSTACK_DEPTH_TYPE	) START_TASK_SIZE,
                  (void * ) NULL,
                  (UBaseType_t) START_TASK_PRIO,
                  (TaskHandle_t *) &start_task_handler 
				);
	vTaskStartScheduler();
}

//任务创建，创建LED1_Task和LED2_Task任务
void start_task( void * pvParameters )
 {
	taskENTER_CRITICAL();
	xTaskCreate( 		
				(TaskFunction_t) LED1_Task,
                  ( char *) "LED1_Task", 
                  ( configSTACK_DEPTH_TYPE) LED1_TASK_SIZE,
                  (void * ) NULL,
                  (UBaseType_t) LED1_TASK_PRIO,
                  (TaskHandle_t *) &LED1_task_handler 
				);	 
	xTaskCreate( 		
				(TaskFunction_t	) LED2_Task,
                  ( char * ) "LED2_Task",
                  ( configSTACK_DEPTH_TYPE) LED2_TASK_SIZE,
                  ( void * ) NULL,
                  (UBaseType_t) LED2_TASK_PRIO,
                  (TaskHandle_t *) &LED2_task_handler 
				);
	taskEXIT_CRITICAL();
	vTaskDelete(NULL);
 }
 
//LED1_Task任务
void LED1_Task(void *pvParameters)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_14;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOC, &GPIO_InitStructure);
	while(1)
	{
		GPIO_ResetBits(GPIOC, GPIO_Pin_14);
		vTaskDelay(pdMS_TO_TICKS(1000));
		GPIO_SetBits(GPIOC, GPIO_Pin_14);
		vTaskDelay(pdMS_TO_TICKS(1000));
	}
}
//LED2_Task任务
void LED2_Task(void *pvParameters)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	while(1)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_0);
		vTaskDelay(pdMS_TO_TICKS(500));
		GPIO_ResetBits(GPIOA, GPIO_Pin_0);
		vTaskDelay(pdMS_TO_TICKS(500));
	}
}


int main(void)
{
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
	freertos_demo();
}

```

> 先创建入口函数，然后在入口函数中创建开始任务，然后在开始任务中创建所要做的任务（创建任务相当于给任务定义），后面就是编写任务
>
> void freertos_demo(void);是入口函数
>
> void LED2_Task(void *pvParameters);是任务，两者不一样
>
> 一定要开始任务调度，并且将任务创建删掉，因为任务创建好之后为防止一直调用任务创建，所以在任务创建的最后加上删除任务指令

> 每个任务要有独立的任务配置区域，这样才好修改和区分任务的参数
>
> 使用一个函数前要先看看定义和声明，确保这个函数和这个函数的一些参数要使能，还要看看task.h文件中关于函数的声明和例子

### 静态创建任务

```c
xTaskCreateStatic();           //静态方式创建任务

/*函数定义*/
    TaskHandle_t xTaskCreateStatic
( 
        TaskFunction_t pxTaskCode,  //指向任务函数得指针
        const char * const pcName,   //任务函数名
        const uint32_t ulStackDepth, //任务堆栈大小
        void * const pvParameters,   //传递的任务函数参数
        UBaseType_t uxPriority,   //任务优先级
        StackType_t * const puxStackBuffer, //任务堆栈，由用户分配
        StaticTask_t * const pxTaskBuffer   //任务控制块指针，用户分配
)
        
//返回值
NULL			//用户
其他值


```

任务的任务控制块以及任务的栈空间所需的内存，需用户分配提供

#### 使用流程

1. 需将宏configSUPPORT_STATIC_ALLOCATION配置为1 
2. 定义空闲任务&定时器的任务堆栈及TCB  //空闲任务必须有
3. 实现两个接口函数
4. 定义函数入口参数
5. 编写任务函数

#### 内部实现

1. TCB结构体成员赋值
2. 添加新任务到就绪列表

### 删除任务

```c
vTaskDelete();                 //删除任务
/*函数定义*/
 void vTaskDelete( TaskHandle_t xTaskToDelete );
形参：xTaskDelete---->待删除任务的任务句柄
```

用于创建已被创建的任务

注意：

> 当传入的参数为NULL，则代表删除任务本身
>
> 空闲任务会负责释放被删除任务中由系统分配的内存，但是由用户在任务删除前申请的内存，则需要由用户在任务被删除前提前释放，否则将导致内存泄漏

#### 删除任务流程

1. 使用删除任务函数，需将宏INCLUDE_vTaskDelete配置为1
2. 入口参数输入需要删除的任务句柄

#### 内部实现

1. 获取所要删除任务的控制块
2. 将被删除任务，移除所在列表
3. 判断所需要删除的任务
    1. 删除任务自身，需先添加到等待删除列表，内存释放将在空闲中进行
    2. 删除其他任务，释放内存，任务数量--
4. 更新下一个任务的阻塞超时时间，以防被删除的任务就是下一个阻塞超时的任务

### 任务挂起

```c
vTaskSuspend();        //挂起任务
```

> xTaskToSuspend是待挂起任务的任务句柄
>
> 此函数用于挂起任务，使用时需将宏INCLUDE_vTaskSuspend配置为1
>
> 当传入的参数为NULL，则代表挂起任务自身（当前正在运行的任务）

### 任务恢复（任务过程）

```c
void vTaskResume(TaskHandle_t xTaskToResume);
```

> xTaskToResume 是待恢复任务的任务句柄
>
> 使用时需将INCLUDE_vTaskSuspend定义为1

### 任务恢复 （中断恢复）

```c
BaseType_t xTaskResumeFromISR(TaskHandle_t xTaskToResume);
```

> 返回值：
>
> pdTRUE			任务恢复后需要进行任务切换
>
> pdFALSE		   任务恢复后不需要进行任务切换
>
> 当恢复的任务优先级大于目前正在进行的任务，则需要手动进行任务切换

> xTaskToResume 是待回复任务的任务句柄
>
> 使用该函数时要将INCLUDE_vTaskSuspend 和 INCLUDE_xTaskResumeFromISR都置为1
>
> 该函数专用于中断服务函数中，用于解挂被挂起任务
>
> 中断服务函数中要调用freeRTOS的API函数则中断优先级不能高于freeRTOS所管理的最高优先级（freeRTOS管理的中断服务函数在5-15之间）
>
> 中断优先级越小越高

#### 例子

```c
/**************************EXIT文件******************************/
#include "stm32f10x.h"                  // Device header
void My_EXIT_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO,ENABLE);
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_IPU;
	GPIO_InitStruct.GPIO_Pin= GPIO_Pin_11;
	GPIO_InitStruct.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOA,&GPIO_InitStruct);
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOA,GaPIO_PinSource11);
	
	
	EXTI_InitTypeDef EXTI_InitStruct; 
	EXTI_InitStruct.EXTI_Line=EXTI_Line11;
	EXTI_InitStruct.EXTI_LineCmd=ENABLE;
	EXTI_InitStruct.EXTI_Mode=EXTI_Mode_Interrupt;
	EXTI_InitStruct.EXTI_Trigger=EXTI_Trigger_Rising;
	EXTI_Init(&EXTI_InitStruct);
	
	
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel=EXTI15_10_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd=ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority=6;//5-15
	NVIC_InitStruct.NVIC_IRQChannelSubPriority=0; //一定要为0
	NVIC_Init(&NVIC_InitStruct);

}


/************************main文件*******************************/
#include "stm32f10x.h"               
#include "FreeRTOS.h"
#include "task.h"
#include "My_EXIT.h"
/*START_TASK任务配置*/
void start_task( void * pvParameters );
#define START_TASK_SIZE 128 
#define START_TASK_PRIO 1
TaskHandle_t start_task_handler;

/*LED1_Task任务配置*/
void LED1_Task(void *pvParameters);
#define LED1_TASK_SIZE 128  
#define LED1_TASK_PRIO 2
TaskHandle_t LED1_task_handler;

/*Key_Task任务配置*/
void Key_Task(void *pvParameters);
#define Key_Task_SIZE 128  
#define Key_Task_PRIO 2
TaskHandle_t Key_task_handler;


/*********************************************************/

//函数入口
void freertos_demo(void)
{
	xTaskCreate(
        		 (TaskFunction_t	     )	 start_task,
                  ( char *                )   "start_task", 
                  ( configSTACK_DEPTH_TYPE)   START_TASK_SIZE,
                  (void *    			 ) 	 NULL,
                  (UBaseType_t  		 ) 	 START_TASK_PRIO,
                  (TaskHandle_t *         )   &start_task_handler 
			   );
	vTaskStartScheduler();
}


//任务创建
void start_task( void * pvParameters )
 {
	taskENTER_CRITICAL();
	xTaskCreate( 		
				(TaskFunction_t			 )  LED1_Task,
                  ( char * 			   	   ) "LED1_Task", 
                  ( configSTACK_DEPTH_TYPE	)  LED1_TASK_SIZE,
                  (void * 				   )  NULL,
                  (UBaseType_t			    ) LED1_TASK_PRIO,
                  (TaskHandle_t * 			)  &LED1_task_handler 
			   );	 
							
	xTaskCreate( 
				(TaskFunction_t			  ) Key_Task,
                  ( char * 				   ) "Key_Task",
                  ( configSTACK_DEPTH_TYPE	)  Key_Task_SIZE,
                  (void * 				   )  NULL,
                  (UBaseType_t			    )  Key_Task_PRIO,
                  (TaskHandle_t * 		    )  &Key_task_handler 
			   );

	taskEXIT_CRITICAL();
	vTaskDelete(NULL);
 }
 
void LED1_Task(void *pvParameters)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	while(1)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_0);
		vTaskDelay(pdMS_TO_TICKS(500));
		GPIO_ResetBits(GPIOA, GPIO_Pin_0);
		vTaskDelay(pdMS_TO_TICKS(500));
	}
}
 
void Key_Task(void *pvParameters)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	while(1)
	{
		if(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_9)==0)
		{
			vTaskDelay(pdMS_TO_TICKS(20));
			while(GPIO_ReadInputDataBit(GPIOA,GPIO_Pin_9)==0);
			vTaskDelay(pdMS_TO_TICKS(20));
			vTaskSuspend(LED1_task_handler);
		}
	}
}
 
int main(void)
{
	My_EXIT_Init();
	
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);  //将NVIC分组为4，不要有响应优先级
	freertos_demo();  //一旦进入就不会再返回main();
} 

void EXTI15_10_IRQHandler(void)  
{
	if(EXTI_GetFlagStatus(EXTI_Line11)==SET)  
	{
		BaseType_t xYieldRequired;
    xYieldRequired=xTaskResumeFromISR(LED1_task_handler);
		if(xYieldRequired==pdTRUE)
		{
			portYIELD_FROM_ISR(xYieldRequired);
		}
		EXTI_ClearFlag(EXTI_Line11);  
	}
}

```

> 流程:
>
> 中断初始化
>
> 调用中断函数

> ```c
> NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
> freertos_demo();
> ```
>
> 这两个文件一定要放在main()的最后面
>
> NVIC一定要配置为第四组,因为响应优先级会影响到FreeRTOS的运行

> 配置FreeRTOS的中断跟裸机开发很相似,先初始化中断,然后在FreeRTOS运行之前将其初始化,然后在STM32自带的中断函数中调用想用的API函数就行

## 延时函数

```c
vTaskDelay(); //相对延时函数
vTaskDelayUntil(pxPreviousWakeTime, xTimeIncrement); //绝对延时函数
pdMS_TO_TICKS(1000); //1000ms
vTaskDelay(pdMS_TO_TICKS(1000));//定时一秒  要两者结合在一起
```



是指整

## 内存

### 空闲任务内存分配

```c
/*函数定义*/
void vApplicationGetIdleTaskMemory( StaticTask_t ** ppxIdleTaskTCBBuffer,   //分配内存
                                    StackType_t ** ppxIdleTaskStackBuffer,  
                                    uint32_t * pulIdleTaskStackSize ); 
```

# 中断

## 中断优先级

STM32的芯片有8位的寄存器来配置中断的优先级，但STM32只是用了中断优先级配置寄存器的高四位[7:4]，所以STM32能提供16级的中断优先等级。

抢占优先级：抢占优先级高的中断可以打断正在执行但抢占优先级低的中断。

子优先级：当同时发生具有相同抢占优先级的两个中断时，子优先级数值小的优先执行。

### 中断优先级分组

```c
NCIC_PriorityGroup_0		0级抢占优先级			 0-15级子优先级
NCIC_PriorityGroup_1		0-1级抢占优先级		 0-7级子优先级
NCIC_PriorityGroup_2		0-3级抢占优先级		 0-3级子优先级
NCIC_PriorityGroup_3		0-7级抢占优先级		 0-1级子优先级
NCIC_PriorityGroup_4		0-15级抢占优先级		 0级子优先级
```

```c
NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);  //分配优先级分组
```

> FreeRTOS主要用的是分组4。

### 特点

1. 低于configMAX_SYSCALL_INTERRUPT_PRIORITY优先级(<5)的中断里才允许调用FreeRTOS的API函数(5-15)
2. 建议将所有优先级位指定为抢占优先级位，方便FreeRTOS管理。
3. **中断优先级数值越小越优先，任务优先级数值越大越优先。**

## 中断相关寄存器

三个系统中断优先级配置寄存器

SHPR1:0xE000ED18

SHPR2:0xE000ED1C

SHPR3:0xE000ED20

三个中断屏蔽寄存器

PRIMASK:这是一个只有1个位的寄存器。置1时关闭所有中断，只剩下NMI和硬fault可以响应。

FAULTMASK:这是个只有1个位的寄存器。当它置1时，只有NMI才能响应，关闭其他的中断。

BASEPRI：这个寄存器最多有9位。定义了被屏蔽优先级的阈值。当它设成某个值后，所有优先级大于等于此值的中断都被关。（小于这个阈值的优先级都被关闭）

FreeRTOS所使用的中断管理就是利用的BASEPRI这个寄存器

## 函数

```c
portENABLE_INTERRUPTS();  //打开所有中断
portENABLE_INTERRUPTS(6);  //关闭6级以下的中断
portDISABLE_INTERRUPTS();  //关闭所有中断
```

## PendSV中断

触发源：

1. 滴答定时器中断调用

			   2. 执行FreeRTOS提供的相关API函数：portYIELD()

本质：通过向中断控制和状态寄存器ICSR的bit28写入1挂起PendSV来启动PendSV中断

## 外部打断例子

```c
/********************main.c文件***************************/
#include "stm32f10x.h"               
#include "FreeRTOS.h"
#include "semphr.h"
#include "task.h"
#include "OLED.h"
#include "serial.h"
#include "key.h"
#include "LED.h"
#include "EXTI1.h"
/*      按键扫描
*通过外部中断释放信号量，将按键任务设置为高优先级
*按键任务执行到获取信号量就会因为没有信号量而进入
*阻塞态，一旦信号量存在，任务就会进入就绪态，所以
*相当于按键按下就会有任务发生,但是容易因为按键抖动
*而进去中断，要在函数中进行消抖
*/

/*START_TASK任务配置*/
void start_task( void * pvParameters );
#define START_TASK_SIZE 128 
#define START_TASK_PRIO 1
TaskHandle_t start_task_handler;

/*TASK1任务配置*/
void task1( void * pvParameters );
#define TASK1_SIZE 128 
#define TASK1_PRIO 3
TaskHandle_t task1_handler;



/********************二值信号量配置*******************************/
SemaphoreHandle_t signal;  //二值信号量句柄


/*****************中断配置**************************************/

static BaseType_t xHigherPriorityTaskWoken;


/*********************************************************/


//函数入口
void freertos_demo(void)
{
	xTaskCreate( 		
									(TaskFunction_t						) start_task,
                  ( char * 									) "start_task", 
                  ( configSTACK_DEPTH_TYPE	) START_TASK_SIZE,
                  (void * 									) NULL,
                  (UBaseType_t							) START_TASK_PRIO,
                  (TaskHandle_t * 					) &start_task_handler 
						); 									
	vTaskStartScheduler();
	vTaskDelete(NULL);   								
}

void start_task( void * pvParameters )
 {
	taskENTER_CRITICAL();
	 xTaskCreate( 		
				(TaskFunction_t						) task1,
                  ( char * 									) "task1", 
                  ( configSTACK_DEPTH_TYPE	) TASK1_SIZE,
                  (void * 									) NULL,
                  (UBaseType_t							) TASK1_PRIO,
                  (TaskHandle_t * 					) &task1_handler 
						);
	Encoder_Init();
	signal = 	xSemaphoreCreateBinary();		//创建二值信号量
	Key_Init();								
	taskEXIT_CRITICAL();
	vTaskDelete(NULL);
 }
 
 void task1( void * pvParameters )
 {
	 while(1)
	 {
		xSemaphoreTake(signal,portMAX_DELAY);  //等待获取二值信号量
		LED_Turn();  
	 }
 }
 
int main(void)
{
	OLED_Init();
	Serial_Init();
	LED_Init();
	Key_Init();
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
	freertos_demo();
}
 

void EXTI15_10_IRQHandler(void)  //硬件配置中断
{
	if(EXTI_GetFlagStatus(EXTI_Line10)==SET)  
	{
		EXTI_ClearFlag(EXTI_Line10);
		xHigherPriorityTaskWoken=pdFAIL;   //判断是否会触发更高优先级任务
		xSemaphoreGiveFromISR(signal,&xHigherPriorityTaskWoken);  //使用带有ISR的信号释放函数
		  if (xHigherPriorityTaskWoken == pdTRUE)
    {
        portYIELD_FROM_ISR(xHigherPriorityTaskWoken);   //安全切换任务
    }
	}
}


/********************Key.c文件***************************/
#include "stm32f10x.h"                  // Device header
#include "FreeRTOS.h"
#include "task.h"
void Key_Init(void)
{
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_IPD;
	GPIO_InitStruct.GPIO_Pin=GPIO_Pin_10;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
	GPIO_Init(GPIOB,&GPIO_InitStruct);
}

/********************EXIT.c文件**********************/
#include "stm32f10x.h"                 

void Encoder_Init(void)
{
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB,ENABLE);
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO,ENABLE);

	
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Mode=GPIO_Mode_IPD;
	GPIO_InitStruct.GPIO_Pin= GPIO_Pin_10;
	GPIO_InitStruct.GPIO_Speed=GPIO_Speed_50MHz;
	GPIO_Init(GPIOB,&GPIO_InitStruct);

	
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB,GPIO_PinSource10);

	
	EXTI_InitTypeDef EXTI_InitStruct; 
	EXTI_InitStruct.EXTI_Line=EXTI_Line10;
	EXTI_InitStruct.EXTI_LineCmd=ENABLE;
	EXTI_InitStruct.EXTI_Mode=EXTI_Mode_Interrupt;
	EXTI_InitStruct.EXTI_Trigger=EXTI_Trigger_Falling;
	EXTI_Init(&EXTI_InitStruct); 

	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_4);
	NVIC_InitTypeDef NVIC_InitStruct;
	NVIC_InitStruct.NVIC_IRQChannel=EXTI15_10_IRQn;
	NVIC_InitStruct.NVIC_IRQChannelCmd=ENABLE;
	NVIC_InitStruct.NVIC_IRQChannelPreemptionPriority=0;
	NVIC_InitStruct.NVIC_IRQChannelSubPriority=6;   //注意：抢占优先级必须要高于5，才能受freeRTOS控制
	NVIC_Init(&NVIC_InitStruct); 
}

/********************LED.c文件*******Y***************/
#include "stm32f10x.h"                  // Device header
#include "FreeRTOS.h"
#include "task.h"
void LED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	GPIO_SetBits(GPIOA,GPIO_Pin_0);
}

void LED_Turn(void)
{
    GPIOA->ODR ^= GPIO_Pin_0;  //可以通过直接控制寄存器实现电平转换
}
```



# 定时器

configTICK_RATE_HZ是滴答定时器的频率  修改该宏的值可以改变定时器的频率



# 列表

## 列表种类

1. 就绪列表
2. 阻塞列表
3. 挂起列表

## 简介

列表是FreeRTOS中的一个数据结构，概念上和链表有点类似，列表被用来跟踪FreeRTOS中的任务

列表相当于链表，列表项相当于节点，FreeRTOS中的列表是一个双向环形链表

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/2024-8-30-20-40.png)

### 列表的特点

列表项间的地址非连续的，是人为的链接再一起的。列表项的数目是由后期添加的个数决定的，随时可以改变

## 列表结构

```c
typedef struct xLIST
{
    listFIRST_LIST_INTEGRITY_CHECK_VALUE     /*校验值*/
    volatile UBaseType_t uxNumberOfltems;	 /*列表中的列表项数量*/
    Listltem_t * configLIST_VOLATILE pxlndex	/*用于遍历列表的指针*/
    MiniListltem_t xListEnd					/*末尾列表项目（迷你列表项且不算在列表项的数量中）*/
    listSECOND_LIST_INTEGRITY_CHECK_VALUE	 /*校验值*/
}List_t;
```

1. 在该结构体中，包含了两个宏，这两个宏是确定的已知常量，FreeRTOS通过检查这两个常量的值，来判断列表的数据在程序运行过程中，是否遭到破坏，该功能一般用于调试，默认是不开启的
2. 成员uxNumberOfltems，用于记录列表中的列表项的个数（不包含xListEnd）
3. 成员pxlndex用于指向列表中的某个列表项，一般用于遍历列表中的所有列表项
4. 成员变量xListEnd是一个迷你列表项，排在最末尾

## 列表项

列表项是列表中用于存放数据的地方，在lith.h文件中，有列表项的相关结构体定义

```c
struct xLIST_ITEM
{
    listFIRST_LIST_ITEM_INTEGRITY_CHECK_VALUE          	/*用于检测列表项的数据完整性*/ 
    configLIST_VOLATILE TickType_t xItemValue;          /*列表项的值*/
    struct xLIST_ITEM * configLIST_VOLATILE pxNext;     /*下一个列表项*/
    struct xLIST_ITEM * configLIST_VOLATILE pxPrevious;	/*上一个列表项*/
    void * pvOwner;                                    	/*列表项的种类*/
    struct xLIST * configLIST_VOLATILE pxContainer;    	/*列表项所在列表*/
    listSECOND_LIST_ITEM_INTEGRITY_CHECK_VALUE         	/*用于检测列表项的数据完整性*/
};
typedef struct xLIST_ITEM ListItem_t; 
```

> 1. 成员变量xltemValue为列表项的值，这个值多用于按升序对列表中的列表项进行排序。
> 2. 成员变量pxNext和pxPrevious分别用于指向列表中列表项的下一个列表项和上一个列表项
> 3. 成员变量pxOwner用于指向包含列表项的对象(通常是任务控制块)
> 4. 成员变量pxContainer用于指向列表项所在列表

## 迷你列表项

迷你列表项也是列表项，但 迷你列表项仅用于标记列表的末尾和挂载其他插入列表的列表项

```C
 struct xMINI_LIST_ITEM
    {
        listFIRST_LIST_ITEM_INTEGRITY_CHECK_VALUE		/*用于检测数据完整性*/
        configLIST_VOLATILE TickType_t xItemValue;		/*列表项的值*/
        struct xLIST_ITEM * configLIST_VOLATILE pxNext;	/*上一个列表项*/
        struct xLIST_ITEM * configLIST_VOLATILE pxPrevious;	/*下一个列表项*/
    };
    typedef struct xMINI_LIST_ITEM MiniListItem_t;
```

1. 成员变量xltemValue为列表项的值，这个值多用于按升序对列表中的列表项进行排序。
2. 成员变量pxNext和pxPrevious分别用于指向列表中列表项的下一个列表项和上一个列表项
3. 迷你列表项只用于标记列表的末尾和挂载其他插入列表中的列表项，因此不需要成员变量pxOwner和pxContainer，以节省内存开销

## 函数

```c
vListInitialise()				/*初始化列表*/
void vListInitialise( List_t * const pxList )  //pxList-->待初始化列表
{			/*初始化时，列表中只有xListEnd，因此pxlndex指向xListEnd*/
    pxList->pxIndex = ( ListItem_t * ) &( pxList->xListEnd );//-->指向末尾列表项
    pxList->xListEnd.xItemValue = portMAX_DELAY;//-->将数值调最大
    pxList->xListEnd.pxNext = ( ListItem_t * ) &( pxList->xListEnd );//-->下一个列表项指向自己
    pxList->xListEnd.pxPrevious = ( ListItem_t * ) &( pxList->xListEnd );//-->上一个列表项指向自己
    pxList->uxNumberOfItems = ( UBaseType_t ) 0U;//-->0个列表项
    listSET_LIST_INTEGRITY_CHECK_1_VALUE( pxList );//数据检验
    listSET_LIST_INTEGRITY_CHECK_2_VALUE( pxList );//数据检验
}

/*-----------------------------------------------------------*/
vListInitialiseItem()			/*初始化列表项*/
void vListInitialiseItem( ListItem_t * const pxItem )  //pxItem-->待初始化列表项
{
    pxItem->pxContainer = NULL;
    listSET_FIRST_LIST_ITEM_INTEGRITY_CHECK_VALUE( pxItem );
    listSET_SECOND_LIST_ITEM_INTEGRITY_CHECK_VALUE( pxItem );
}

/*-----------------------------------------------------------*/
vListInsertEnd()				/*列表末尾插入列表项，是一种无序的插入法*/
void vListInsertEnd( List_t * const pxList,ListItem_t * const pxNewListItem )//pxList-->列表，pxNewListltem-->列表项
{
    ListItem_t * const pxIndex = pxList->pxIndex;		//获取列表pxlndex指向的列表项			  	
    pxNewListItem->pxNext = pxIndex;				    //更新待插入列表项的指针成员变量
    pxNewListItem->pxPrevious = pxIndex->pxPrevious;
    pxIndex->pxPrevious->pxNext = pxNewListItem;		//更新列表中原本列表项的指针成员变量
    pxIndex->pxPrevious = pxNewListItem;
    pxNewListItem->pxContainer = pxList;				//更新待插入列表项的所在列表成员变量
    ( pxList->uxNumberOfItems )++;						//更新列表中列表项的数量
}

/*-----------------------------------------------------------*/
vListInsert()					/*将列表项按照值有序的插入到列表中*/
void vListInsert( List_t * const pxList,ListItem_t * const pxNewListItem )  //pxList是列表，pxNewListltem待插列表项
{
    ListItem_t * pxIterator;
    const TickType_t xValueOfInsertion = pxNewListItem->xItemValue;
    listTEST_LIST_INTEGRITY( pxList );
    listTEST_LIST_ITEM_INTEGRITY( pxNewListItem );
    if( xValueOfInsertion == portMAX_DELAY )  //如果列表项值等于末尾列表项的值
        {
            pxIterator = pxList->xListEnd.pxPrevious;
        }
    else	//如果不等于
        {
            for( pxIterator = ( ListItem_t * ) &( pxList->xListEnd );//遍历列表，找到插入的位置
                pxIterator->pxNext->xItemValue <= xValueOfInsertion; 
                pxIterator = pxIterator->pxNext ){}
        }
    pxNewListItem->pxNext = pxIterator->pxNext;   //将待插入的列表项插入指定位置
    pxNewListItem->pxNext->pxPrevious = pxNewListItem;
    pxNewListItem->pxPrevious = pxIterator;
    pxIterator->pxNext = pxNewListItem;
     pxNewListItem->pxContainer = pxList;
    ( pxList->uxNumberOfItems )++;
}

/*-----------------------------------------------------------*/
uxListRemove()					/*列表移除列表项*/
UBaseType_t uxListRemove( ListItem_t * const pxItemToRemove )   //pxItemToRemove是待删除列表项
{
    List_t * const pxList = pxItemToRemove->pxContainer;
    pxItemToRemove->pxNext->pxPrevious = pxItemToRemove->pxPrevious;
    pxItemToRemove->pxPrevious->pxNext = pxItemToRemove->pxNext;
    mtCOVERAGE_TEST_DELAY();
    if( pxList->pxIndex == pxItemToRemove )
    {
        pxList->pxIndex = pxItemToRemove->pxPrevious;
    }
    else
    {
        mtCOVERAGE_TEST_MARKER();
    }

    pxItemToRemove->pxContainer = NULL;
    ( pxList->uxNumberOfItems )--;
    return pxList->uxNumberOfItems;  //返回整数，值为列表剩余lie'biao'xiang
}
```

# 任务调度器

## 任务启动

```c
vTaskStartScheduler();
```

作用：用于启动任务调度器，任务调度器启动后，FreeRTOS便会开始进行任务调度

> 1. 创建空闲任务
> 2. 如果使能软件定时器，则创建定时器任务
> 3. 关闭中断，防止调度器开始之前或过程中，受中断干扰，会在运行第一个任务时打开中断
> 4. 初始化全局变量，并将任务调度器的运行标志设置为已运行
> 5. 初始化任务运行时间统计功能的时基定时器
> 6. 调用函数xPortStartScheduler()

```c\
prvStartFirstTask();		/*开启第一个任务*/
vPortSVCHandler();			/*SVC中断服务函数*/
```

假设我们要启动的第一个任务是任务A（优先级最高），那么就需要将任务A的寄存器值恢复到CPU寄存器。任务A的寄存值，在一开始创建任务时就保存在任务堆栈里边！

> 中断产生时，硬件自动将xPSR, PC(R15), LR(R14), R12, R3-R0保存和恢复，而R4~R11需要手动保存和恢复。
>
> 进入中断后硬件会强制使用MSP指针，此时LR(R14)的值将会被自动更新为特殊的EXC_RETURN

```c
prvStartFirstTask();		/*开启第一个任务*/
```

用于初始化第一个任务前的环境，主要是重新设置MSP指针，并使能全局中断。

> MSP指针：程序在运行过程中需要一定的栈空间来保存局部变量等一些信息。当有信息保存到栈中时，MCU会自动更新SP指针(R13)，ARM Cortex-M内核提供了两个栈空间

### 任务堆栈指针

程序在运行过程中需要一定的栈空间来保存局部变量等一些信息。当有信息保存到栈中时，MCU会自动更新SP指针。

主堆栈指针（MSP）：它由OS内核，异常服务例程以及所有需要特权访问的应用程序代码来使用。

进程堆栈指针（PSP）：用于常规的应用程序代码（不处于异常服务例程中时）。

在FreeRTOS中，使用的是双堆栈指针，中断使用MSP(主堆栈)，中断以外使用PSP（进程堆栈），两者互不干扰。

## 任务切换                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      

任务切换的本质：就是CPU寄存器的切换。

假设当由任务A切换到任务B，主要分为两步：

第一步：需暂停任务A的执行，并将此时任务A的寄存器保存到任务堆栈，这个过程叫做保存现场

第二步：将任务B的各个寄存器值（被存于任务堆栈中）恢复到CPU寄存器中，这个过程叫做恢复现场

对任务A保存现场，对任务B恢复现场，这个整体的过程称之为：上下文切换

>  任务切换的过程在PendSV中断服务函数里边完成

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87%E7%BC%96%E8%BE%91_20240909211423.jpg)

## 任务相关的API函数

### 总任务函数

```c
uxTaskPriorityGet();						//获取任务优先级
vTaskPrioritySet();						    //设置任务优先级
uxTaskGetNumberOfTasks();					//获取系统中任务的数量
uxTaskGetSystemState();						//获取所有任务状态信息
vTaskGetInfo();							   //获取指定单个的任务信息
xTaskGetCurrentTaskHandle();				//获取当前任务的任务句柄
xTaskGetHandle();						   //根据任务名获取该任务的任务句柄
uxTaskGetStackHighWaterMark();				//获取任务的任务栈历史剩余最小值
eTaskGetState();						   //获取任务状态
vTaskList();							  //以“表格”形式获取所有任务的信息
vTaskGetRunTimeStats();				  	   //获取任务的运行时间
```

### 任务详解

#### 获取任务优先级

```c
UBaseType_t uxTaskPriorityGet( const TaskHandle_t xTask );  //用于获取某个任务的任务优先级
```

形参：xTask-->任务句柄，NULL代表任务自身

返回值：一个UBaseType_t类型的整数--->任务优先级数值

> 该函数用于获取指定任务的任务优先级，使用该函数前需将宏INCLUDE_uxTaskPriorityGet 置1

#### 设置任务优先级

```c
void vTaskPrioritySet( TaskHandle_t xTask,UBaseType_t uxNewPriority )					    //用于改变某个任务的优先级
```

形参：xTask-->任务句柄，NULL代表任务自身

​		   uxNewPriority--->需要设置的任务优先级

> 该函数用于改变指定任务的任务优先级，使用该函数前需将宏INCLUDE_vTaskPrioritySet 置1

#### 获取系统中任务的数量

```c
UBaseType_t uxTaskGetNumberOfTasks( void )     //获取系统中任务的数量
```

返回值：一个UBaseType_t类型的整数 --->任务数量

> 函数入口这个任务是无法删除的，返回值永远包含了函数入口这个任务。

#### 获取所有任务状态信息

```c
UBaseType_t uxTaskGetSystemState( TaskStatus_t * const 				   pxTaskStatusArray,
                                  const UBaseType_t 				   uxArraySize,
                                  configRUN_TIME_COUNTER_TYPE * const    pulTotalRunTime )
```

形参：pxTaskStatusArray  --->指向TaskStatus_t结构体数组首地址

​			uxArraySize--->接收信息的数组大小

​			pulTotalRunTime  --->系统总运行时间，为NULL则省略总运行时间值

返回值：一个UBaseType_t类型的整形--->获取信息的任务数量

> 此函数用于获取系统中所有任务的任务状态信息，使用前需将宏configUSE_TRACE_FACILITY置1

```c
typedef struct xTASK_STATUS
{
    TaskHandle_t		xHandle;  							/*任务句柄*/
    const char*			pcTaskName;							/*任务名*/
    UBaseType_t			xTaskNumber;						/*任务编号*/
    eTaskState			eCurrentState;						/*任务优先级*/
    UBseType_t			uxCurrentPriority;					/*任务原始优先级*/
    UBaseType_t			uxBasePriority;						/*任务运行时间*/
    configRUN_TIME_COUNTER_TYPE		ulRunTimeCounter;		  /*任务栈基地址*/
    configSTACK_DEPTH_TYPE			usStackHighWaterMark;	  /*任务栈历史剩余最小值*/
} TaskStatus_t;
```

##### 例子

```c
 void task1( void * pvParameters )
 {
	 eTaskState task_num=0;  //任务数量
	 eTaskState task_num2=0; //任务数量
	 TaskStatus_t * status_array=0;  //结构体变量
	 uint8_t i=0;   
	 
	 task_num=uxTaskGetNumberOfTasks();  //获取任务数量
	 
	 status_array=pvPortMalloc(sizeof(TaskStatus_t)*task_num);
     //因为结构体存储着多个任务的多个状态，所以需要申请结构体的内存
	 task_num2=uxTaskGetSystemState(status_array,task_num,NULL);
     //这一步已经获得了所有任务的状态
	 Serial_Printf("任务名	任务优先级	任务编号\r\n");
	 for(i=0;i<task_num2;i++)
	 {
							Serial_Printf("%s\t%d\t%d\t\r\n",status_array[i].pcTaskName,status_array[i].eCurrentState,status_array[i].xTaskNumber);
      //输出结构体中的成员，分别有任务名、任务优先级、任务编号
	 }
	while(1)
	{
		vTaskDelay(pdMS_TO_TICKS(1000));
	}
 }
```

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240910224058.png)

#### 获取指定单个的任务信息

```c
void vTaskGetInfo( 
    			TaskHandle_t  xTask, //要获取信息的任务句柄
    			TaskStatus_t  *pxTaskStatus, //接收任务信息的结构体变量
    			BaseType_t    xGetFreeStackSpace, //任务栈历史剩余最小值，当为"pdFALSE"时跳过这步，当为"pdTRUE"则检查历史剩余最小堆栈
    			eTaskState    eState //任务状态，可直接赋值，如想获取带入"elnvalid"
				);

typedef enum
{
    eRunning=0,  /*运行态*/
    eReady,		/*就绪态*/
    eBlocked,	/*阻塞态*/
    eSuspended,	/*挂起态*/
    eDeleted,	/*任务被删除*/
    eInvalid	/*无效*/
}eTaskState
```

> 此函数用于获取指定的单个任务的状态信息，使用该函数需将宏configUSE_TRACE_FACILITY置1

```c
typedef struct xTASK_STATUS
{
    TaskHandle_t		xHandle;  							/*任务句柄*/
    const char*			pcTaskName;							/*任务名*/
    UBaseType_t			xTaskNumber;						/*任务编号*/
    eTaskState			eCurrentState;						/*任务优先级*/
    UBseType_t			uxCurrentPriority;					/*任务原始优先级*/
    UBaseType_t			uxBasePriority;						/*任务运行时间*/
    configRUN_TIME_COUNTER_TYPE		ulRunTimeCounter;		  /*任务栈基地址*/
    configSTACK_DEPTH_TYPE			usStackHighWaterMark;	  /*任务栈历史剩余最小值*/
} TaskStatus_t;
```

##### 例子

```c
TaskStatus_t *array2=0;  //定义结构体变量
 array2=pvPortMalloc(sizeof(TaskStatus_t));  //只有1个任务，只分配一个结构体大小的内存
vTaskGetInfo(task2_handler,array2,pdTRUE,eInvalid);//函数
Serial_Printf("任务名	任务优先级	任务编号\r\n");
Serial_Printf("%s\t%d\t%d\t\r\n",array2->pcTaskName,array2->eCurrentState,array2->xTaskNumber);//串口打印
```

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240910223615.png)

#### 获取当前任务句柄

```c
TaskHandle_t xTaskGetCurrentTaskHandle( void )
```

返回值：当前任务的任务句柄

> 用该函数前需将宏INCLUDE_xTaskGetCurrentTaskHandle置1
>
> TaskHandle_t类型的句柄本质上是一个十六进制的地址，所以串口输出要用16进制的输出

##### 例子

```c
TaskHandle_t Handle=0;  //定义变量存储任务句柄
Handle=xTaskGetCurrentTaskHandle();
Serial_Printf("任务句柄：%#x\r\n",Handle);
```



#### 获取任务状态

```c
* eTaskState eTaskGetState( TaskHandle_t xTask )
```

形参：xTask--->待获取状态任务的句柄

返回值：eTaskState--->任务状态

```c
typedef enum
{
    eRunning=0,  /*运行态*/
    eReady,		/*就绪态*/
    eBlocked,	/*阻塞态*/
    eSuspended,	/*挂起态*/
    eDeleted,	/*任务被删除*/
    eInvalid	/*无效*/
}eTaskState
```

> 使用该函数前要将宏INLCUDE_eTaskGetState置1

#### 表格形式获取任务状态

```c
void vTaskList(char *pcWriteBuffer);
```

形参：pcWriteBuffer--->接收任务信息的缓存指针

信息包括：

Name: 创建任务给任务分配的名字

State:任务的状态信息，B是阻塞态，R是就绪态，S是挂起态，D是删除态

Priority:任务优先级

Stack:堆栈历史最小剩余大小

Num:任务编号

> 使用此函数前要将宏configUSE_TRACE_FACILITY和configUSE_STATS_FORMATTING_FUNCTIONS置1

##### 例子

```c
char task_Buff[300];  //用于存放信息状态(建议放全局变量)
vTaskList(task_Buff);  //此时信息已经存进了task_Buff中
Serial_Printf("%s",task_Buff);  //通过串口打印
```

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240910223443.png)

#### 获取任务运行时间

```c
void vTaskGetRunTimeStats( char *pcWriteBuffer );
```

形参：pcWriteBuffer--->接收任务运行时间信息的缓存指针

返回值:	Task--->任务名称

​				Abs Time--->任务实际运行的总时间（绝对时间）

​				%Time--->占总处理时间的百分比

> 该函数多用于产品调试
>
> 使用此函数时需将configGENERATE_RUN_TIME_STAT、configUSE_STATS_FORMATTING_FUNCTIONS置1
>
> 将宏configGENERATE_RUN_TIME_STAT置1之后，还需要实现2个宏定义
>
> portCONFIGURE_TIMER_FOR_RUN_TIME_STATS(); -->用于初始化配置运行时间统计的时基定时器（这个时基定时器的计时精度需高于系统时钟节拍精度的10至100倍！)
>
> portGET_RUN_TIME_COUNTER_VALUE();  -->用于获取该功能时基硬件定时器计数的计数值

# 时间片调度

## 简介

同等优先级任务轮流地享有相同地CPU时间（可设置），叫时间片，在FreeRTOS中，一个时间片就等于Systick中断周期（1ms触发一次）

使用时间片调度需把宏configUSE_TIME_SLICING和configUSE_PREEMPTION置1

> 只有相同优先等级的任务才能平均分配任务，如果优先级较高并且没有进入阻塞态，则会一直执行优先级较高的任务并不执行优先级较低的任务

## 延时函数

```c
vTaskDelay();			//相对延时
xTaskDelayUntil();		//绝对延时
```

相对延时：指每次延时都是从执行函数vTaskDelay()开始，直到延时指定的时间结束

绝对延时：指将整个任务的运行周期看成一个整体，适用于需要按照一定频率运行的任务。（绝对延时是指你任务执行时间的总时间，任务主体时间不够，靠延时来凑）

```c
void vTaskDelay( const TickType_t xTicksToDelay );
```

形参：xTicksToDelay--->延时时间ms

> 延时时任务会进入阻塞态

```c
vTaskDelayUntil( pxPreviousWakeTime, xTimeIncrement );//绝对函数
```

形参：pxPreviousWakeTime --->上一次运行的时间指针

​			xTimeIncrement--->延时时间ms

> 使用之前要将宏INCLUDE_vTaskDelayUntil置1
>
> pxPreviousWakeTime 用xTaskGetTickCount()获取

例子：

```c
 void task2( void * pvParameters )
 {
	 TickType_t xLastWakeTime;  //定义变量
	 xLastWakeTime=xTaskGetTickCount();  //获取时间运行指针
	 while(1)
	 {
		 LED2_Turn();
		vTaskDelayUntil(&xLastWakeTime,500);
	 }
 }
```

### 总结

vTaskDelay是将任务进入阻塞态，vTaskDelayUntil是严格一个任务确保有一段时间执行

# 队列

## 简介

队列是任务到任务、任务到中断、中断到任务数据交流的一种机制（消息传递）

队列在读写队列操作的时候会将中断关闭，这样可以防止多任务同时访问导致冲突。

我们只需要调用API函数即可。全局变量是没有任何保护。

FreeRTOS基于队列，实现了多种功能，其中包括队列集、互斥信号量、计数型信号量、二值信号量、递归互斥信号量，因此很有必要深入了解FreeRTOS的队列。

在队列中可以存储数量有限、大小固定的数据。队列中的每一个数据叫做队列项目，队列能够存储队列项目的最大量称为队列长度。

所以在创建队列时，就要指定队列长度以及队列项目的大小。

### 特点

1. 数据入队出队的方式

队列通常采用“先进先出“(FIFO)的数据存储缓存机制，即先入队的数据会先从队列中被读取，FreeRTOS中也可以配置为”后进后出“LIFO方式

2. 数据传递方式

FreeRTOS中队列采用实际值传递，即将数据拷贝到队列中进行传递，FreeRTOS采用拷贝数据传递，也可以传递指针，所以在传递较大的数据的时候采用指针传递。

3. 多任务访问

队列不属于某个任务，任何任务和中断都可以向队列发送/读取信息

4. 出队、入队堵塞

当任务向一个队列发送信息时，可以指定一个阻塞时间，假设此时当队列已满无法入队。

a.若阻塞时间为0：直接返回不会等待

b. 若阻塞时间为0~port_MAX_DELAY：等待设定的阻塞时间，若在该时间内还无法入队，超时后直接返回不再等待

c. 若阻塞时间为port_MAX_DELAY：死等，一直等到可以入队为止。出队阻塞与入队阻塞类似

> 如果队列满了，此时写不进去数据。
>
> 1. 将该任务的状态列表挂载到阻塞列表
> 2. 将该任务的事件列表挂载等待发送列表中
>
> 如果队列为空，此时读取不了数据
>
> 1. 将该任务的状态列表挂载到阻塞列表
> 2. 将该任务的事件列表挂载等待接收列表中
>
> 如果多个任务同时写入信息给一个满队列，这些任务都进入阻塞状态。当队列有空间时，哪个任务先写入队列：
>
> 1. 优先级最高的任务
> 2. 如果大家优先级相同，那等待时间最久的任务会进入就绪态。

## 队列操作基本过程

1. 创建队列
2. 往队列写入第一个信息
3. 往队列写入第二个信息
4. 从队列中读取第一个信息（第二个信息位置会移至第一个位置）

## 队列结构介绍

```c
typedef struct QueueDefinition 
{
    int8_t * pcHead;           //存储区域的起始地址
    int8_t * pcWriteTo;        //下一个写入的位置
    union
    {
        QueuePointers_t xQueue;    
        SemaphoreData_t xSemaphore; 
    } u;

    List_t xTasksWaitingToSend;     //等待发送列表       
    List_t xTasksWaitingToReceive;   //等待接收列表
    volatile UBaseType_t uxMessagesWaiting;  //非空闲队列队项目数量
    UBaseType_t uxLength;         //队列长度
    UBaseType_t uxItemSize;       //队列项目的大小

    volatile int8_t cRxLock;      //读取上锁计数器
    volatile int8_t cTxLock;      //写入上锁计数器                                                                                   
    //其他的一些条件编译
} xQUEUE;
```

当用于队列使用时

```c
typedef struct QueuePointers
{
    int8_t * pcTail;   //互斥信号量持有者
    int8_t * pcReadFrom;  //递归互斥信号量的获取计数器
} QueuePointers_t;
```

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87%E7%BC%96%E8%BE%91_20240913204356.jpg)

## 相关API函数

使用队列的主要流程：创建队列->写队列->读队列

```c
xQueueCreate();			//动态方式创建队列
xQueueCreateStatic();	//静态方式创建队列
xQueueSend(); 			//往队列的尾部写入信息
xQueueSendToBack();		//同xQueueSend()
xQueueSendToFront();	//往队列的头部写入信息
xQueueOverwrite();		//覆写队列信息（只用于队列长度为1的情况下）
```

动态和静态之间的区别在于：队列所需的内存空间由FreeRTOS从FreeRTOS管理的堆中分配，而静态创建需要用户自行分配内存。

### 动态创建

```c
QueueHandle_t xQueueCreate(
                              UBaseType_t uxQueueLength,
                              UBaseType_t uxItemSize
                          );
```

形参：uxQueueLength			队列长度

​			uxItemSize					队列项目的大小

返回值：NULL			队列创建失败

​				其他值			队列创建成功，返回队列句柄

> 使用之前要将宏configSUPPORT_DYNAMIC_ALLOCATION置1

```c
#define queueQUEUE_TYPE_BASE                  ( ( uint8_t ) 0U )
//队列
#define queueQUEUE_TYPE_SET                   ( ( uint8_t ) 0U )
//队列集
#define queueQUEUE_TYPE_MUTEX                 ( ( uint8_t ) 1U )
//互斥信号量
#define queueQUEUE_TYPE_COUNTING_SEMAPHORE    ( ( uint8_t ) 2U )
//计数型信号量
#define queueQUEUE_TYPE_BINARY_SEMAPHORE      ( ( uint8_t ) 3U )
//二值信号量
#define queueQUEUE_TYPE_RECURSIVE_MUTEX       ( ( uint8_t ) 4U )
//递归互斥信号量
```

### 队列写入信息

```c
BaseType_t xQueueGenericSend( QueueHandle_t xQueue,  //待写入的队列
                              const void * const pvItemToQueue,	 //待写入信息
                              TickType_t xTicksToWait,	//阻塞超时时间
                              const BaseType_t xCopyPosition )	//写入的位置
```

返回值：pdTURE	队列写入成功

​			  errQUEUE_FULL  队列写入失败

```c
BaseType_t xQueueSend(
                              QueueHandle_t xQueue, //写入队列的句柄
                              const void * pvItemToQueue,//待写入信息地址
                              TickType_t xTicksToWait
                         );
```

### 队列读取信息

```c
xQueueReceive();		//从队列头部读取信息，并删除信息
xQueuePeek();			//从队列头部读取信息
xQueueReceiveFromISR();	 //在中断中从队列头部读取消息，并删除消息
xQueuePeekFromISR();   	 //在中断中从队列头部读取消息
```

```c
BaseType_t xQueueReceive( QueueHandle_t xQueue,  		//待读取的队列
                          void * const pvBuffer,		//信息读取缓冲区
                          TickType_t xTicksToWait )		//阻塞超时时间
```

返回值： pdTRUE		读取成功

​				pdFALSE	 读取失败

>  此函数用于在任务中，从队列中读取信息，并且信息读取成功后，会将信息从队列中删除

```c
BaseType_t xQueuePeek( QueueHandle_t xQueue,//待读取的队列
                       void * const pvBuffer,//信息读取缓冲区
                       TickType_t xTicksToWait )//阻塞超时时间
```

返回值： pdTRUE		读取成功

​				pdFALSE	 读取失败

>  此函数用于在任务中，从队列中读取信息，并且信息读取成功后，不会将信息从队列中删除

# 队列集

**使用情况**：一个队列只允许任务间传递的消息为同一种数据类型，如果需要在任务间传递不同数据类型的消息时，那么就可以使用队列集 ！

作用：用于对多个队列或信号量进行“监听”，其中不管哪一个消息到来，都可让任务退出阻塞状态

假设：有个接收任务，使用到队列接收和信号量的获取，如下：(信号量本质就是一个队列，所以此时有两个队列)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241114204117.png)

## API函数

```c
xQueueCreateSet(); //创建队列集
xQueueAddToSet(); //队列添加到队列集中
xQueueRemoveFromSet(); //从队列集中移除队列
xQueueSelectFromSet(); //获取队列集中有有效消息的队列
xQueueSelectFromSetFromISR(); //在中断中获取队列集中有有效消息的队列
```

### 创建队列集

```c
QueueSetHandle_t xQueueCreateSet( const UBaseType_t uxEventQueueLength )
```

> 形参：uxEventQueueLength---->队列集可容纳的队列数量
>
> 返回值：
>
> > NULL -->队列创建失败
> >
> > 其他值-->队列集创建成功，返回队列集句柄

### 添加队列到队列集

```c
BaseType_t xQueueAddToSet( QueueSetMemberHandle_t xQueueOrSemaphore,QueueSetHandle_t xQueueSet )
```

> 形参：xQueueOrSemaphore-->待添加的队列句柄        xQueueSet-->队列集
>
> 返回值：pdPASS-->队列集添加队列成功      pdFAIL-->队列集添加队列失败
>
> **此函数用于往队列集中添加队列，要注意的时，队列在被添加到队列集之前，队列中不能有有效的消息**

### 移除队列集

```c
BaseType_t xQueueRemoveFromSet( QueueSetMemberHandle_t xQueueOrSemaphore,QueueSetHandle_t xQueueSet )
```

> 形参：xQueueOrSemaphore-->待移除的队列句柄       xQueueSet-->队列集
>
> 返回值：pdPASS-->队列集移除队列成功      pdFAIL-->队列集移除队列失败
>
> **此函数用于从队列集中移除队列， 要注意的是，队列在从队列集移除之前，必须没有有效的消息**

### 获取队列集有效信息

```c
QueueSetMemberHandle_t xQueueSelectFromSet( QueueSetHandle_t xQueueSet,const TickType_t xTicksToWait)
```

> 形参：xQueueSet-->待移除的队列句柄       xTicksToWait-->阻塞超时时间
>
> 返回值：NULL-->获取消息失败     其他值 -->获取到消息的队列句柄
>
> **此函数用于从队列集中移除队列， 要注意的是，队列在从队列集移除之前，必须没有有效的消息**

### 使用流程

1. 启用队列集功能需要将宏configUSE_QUEUE_SETS 配置为 1
2. 创建队列集
3. 创建队列或信号量
4. 往队列集中添加队列或信号量
5. 往队列发送信息或释放信号量
6. 获取队列集的消息

# 信号量

## 简介

信号量是一种解决同步问题的机制，可以实现对共享资源的有序访问。（传递状态）

当计数值大于0，代表有信号量资源

当释放信号量，信号量计数值（资源数）加一

当获取信号量，信号量计数值（资源数）减一

1. 信号量是否有资源
    1. 有资源，获取信号量成功
    2. 无资源，获取信号量失败，任务堵塞，释放信号量资源

信号量的计数值都有限制的：限定最大值。

如果最大值被限定为1，那么它就是二值信号量。

如果最大值不是1，它就是计数型信号量。

## 队列于信号量的对比

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87%E7%BC%96%E8%BE%91_20240921161502.jpg)

## 二值信号量

使用API函数之前要包含"semphr.h"头文件

二值信号量的本质是一个队列长度为1的队列，该队列就只有空和满两种情况，这就是二值。

二值信号量通常用于互斥访问或任务同步，于互斥信号量比较类似，但是二值信号量有可能导致优先级翻转的问题，所以**二值信号量更适合同步。**

> 任务或中断都可以释放和获取二值信号量

### API函数

使用二值信号量的过程：创建二值信号量->释放二值信号量->获取二值信号量

```c
xSemaphoreCreateBinary( void );	//动态创建二值信号量
xSemaphoreCreateBinaryStatic()	//静态创建二值信号量
xSemaphoreGive();			   //释放信号量
xSemaphoreGiveFromISR();	   //在中断中释放信号量
xSemaphoreTake();			  //获取信号量
xSemaphoreTakeFromISR();	   //在中断中获取信号量
```

#### 创建函数

```c
SemaphoreHandle_t xSemaphoreCreateBinary( void );
/*创建队列长度为1，大小为0，类型为二值信号量的队列*/
```

返回值:

NULL		创建失败

其他值		创建成功返回二值信号量的句柄

```C
#define queueQUEUE_TYPE_BASE                  ( ( uint8_t ) 0U )  //队列
#define queueQUEUE_TYPE_SET                   ( ( uint8_t ) 0U )  //队列集
#define queueQUEUE_TYPE_MUTEX                 ( ( uint8_t ) 1U )  //互斥信号量
#define queueQUEUE_TYPE_COUNTING_SEMAPHORE    ( ( uint8_t ) 2U )  //计数型信号量
#define queueQUEUE_TYPE_BINARY_SEMAPHORE      ( ( uint8_t ) 3U )  //二值信号量
#define queueQUEUE_TYPE_RECURSIVE_MUTEX       ( ( uint8_t ) 4U )  //递归互斥信号量
```

#### 释放

```c
/**
  * 函    数：释放信号量
  * 参    数：xSemaphore-->要释放的信号量句柄
  * 返 回 值：pdPASS-->释放信号量成功、errQUEUE_FULL-->释放信号量失败
  * 注意事项：#define configUSE_MUTEXES 1
			#define configUSE_RECURSIVE_MUTEXES 1
			#define configUSE_COUNTING_SEMAPHORES 1
			#include "semphr.h"
  */
xSemaphoreGive( xSemaphore );
```

pdPASS-->释放信号量成功

errQUEUE_FULL-->释放信号量失败

#### 获取二值信号量

```c
xSemaphoreTake( xSemaphore, xBlockTime )
```

形参：

xSemaphore-->要获取的信号量句柄

xBlockTime-->阻塞时间  //portMAX_DELAY-->阻塞时间最大值

返回值：

pdTRUE-->获取信号量成功

pdFALSE-->超时，获取信号量失败

### 总结

二值信号量更偏向于单对象对单对象，但是于通知的单对单不一样，二至信号的单对单中的单对象可以根据情况改变，释放的可以对应着获取的，而通知更像是一个任务对应一个任务，不能改变了。

二值信号量为空的时候，高优先级获取二值信号量的时候会因为二值信号量为空进入阻塞态，一旦二值信号量为满就会进入就绪状态，而本身作为高优先级，就会立马打断进行任务。



## 计数型信号量

计数型信号量相当于队列长度大于1的队列，因此计数型信号量能够容纳多个资源，这在计数型信号量被创建的时候确定的（队列长度在创建的时候已经给确定）

计数型信号量适用场合：

**事件计数**：当每次事件发生后，在事件处理函数中释放计数型信号量（计数值+1），其他任务会获取计数型信号量（计数值-1），这种场合一般在创建时初始计数值设置为0.

**资源管理**：信号量表示有效的资源数目。任务必须先获取信号量（信号量计数值-1）才能获取资源控制权。当计数值减为零时表示没有的资源。当任务使用完资源后，必须释放信号量（信号量计数值+1）。信号量创建时计数值应等于最大资源数目。

### 使用流程

创建计数型信号量->释放信号量->获取信号量

### API函数

```c
xQueueCreateCountingSemaphore();    //使用动态方法创建计数型信号量
xQueueCreateCountingSemaphoreStatic();  //使用静态方法创建计数型信号量
uxSemaphoreGetCount();  //获取信号量的计数值
```

> 计数型信号量的释放和获取与二值信号量相同

#### 动态创建

```c
/**
  * 函    数：动态创建计数型函数
  * 参    数：uxMaxCount->最大值限定，uxInitialCount->计数值的初始值
  * 返 回 值：QueueHandle_t->计数型信号量的句柄,NULL->创建失败
  * 注意事项：使用之前要包含"queue.h"头文件
  */
QueueHandle_t xQueueCreateCountingSemaphore( const UBaseType_t uxMaxCount,const UBaseType_t uxInitialCount )
```

#### 获取计数值

```c
/**
  * 函    数：获取计数型信号量的计数值
  * 参    数：xQueue  -->信号量句柄
  * 返 回 值：UBaseType_t--> 当前信号量的计数值大小（整数）
  */
UBaseType_t uxQueueMessagesWaiting( const QueueHandle_t xQueue )
```



## 优先级翻转

优先级翻转：高优先级的任务反而慢执行，低优先级的任务反而优先执行

优先级翻转在抢占式内核中是非常常见的，但是在实时操作系统中是不允许出现优先级翻转的，因为优先级翻转会破坏任务的预期顺序，可能会导致未知的严重后果。

> 在使用二值信号量的时候，经常会遇到优先级翻转的问题。

### 例子

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241114200003.png)

> 如果高优先级还在准备状态，而低优先级已经是就绪状态，低优先级会读取信号量，导致二值信号量为0，此时高优先级读取二值信号量会因为二值信号量为0而一直在阻塞状态直到低优先级任务释放二值信号，所以导致优先级翻转，从现象上看，就像是中优先级的任务比高优先级任务具有更高的优先权，因为中优先级会影响低优先级，而低优先级使高优先级阻塞。

## 互斥 信号量

互斥信号量其实就是一个拥有优先级继承的二值信号量，在同步的应用中二值信号量最适合。互斥信号量适合用于那些需要互斥访问的应用中！

> 优先级继承：当一个互斥信号量正在被一个低优先级的任务持有时， 如果此时有个高优先级的任务也尝试获取这个互斥信号量，那么这个高优先级的任务就会被阻塞。**不过这个高优先级的任务会将低优先级任务的优先级提升到与自己相同的优先级。**

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241114201116.png)

> 优先级继承并不能完全的消除优先级翻转的问题，它只是尽可能的降低优先级翻转带来的影响
>
> 注意：互斥信号量不能用于中断服务函数中，原因如下：
>
> (1) 互斥信号量有任务优先级继承的机制， 但是中断不是任务，没有任务优先级， 所以互斥信号量只能用与任务中，不能用于中断服务函数。
>
> (2) 中断服务函数中不能因为要等待互斥信号量而设置阻塞时间进入阻塞态。

### API函数

**使用互斥信号量：首先将宏configUSE_MUTEXES置一**

```c
xQueueCreateMutex();   //使用动态方法创建互斥信号量。
xQueueCreateMutexStatic() ; //使用静态方法创建互斥信号量 			
```

> 使用流程：创建互斥信号量 ->（task）获取信号量 ->（give）释放信号量
>
> 互斥信号量的释放和获取函数与二值信号量相同 ！只不过互斥信号量不支持中断中调用
>
> 注意：**创建互斥信号量时，会主动释放一次信号量**

#### 创建函数

```c
QueueHandle_t xQueueCreateMutex( const uint8_t ucQueueType );
```

返回值：NULL---->创建失败

​			  其他值---->创建成功返回互斥信号量的句柄

# 事件标志组

事件标志位：用一个位，来表示事件是否发生

事件标志组是一组事件标志位的集合， 可以简单的理解事件标志组，就是一个整数。

## 特点

它的每一个位表示一个事件（高8位不算）

每一位事件的含义，由用户自己决定，如：bit0表示按键是否按下，bit1表示是否接受到消息 … …

> 这些位的值为1：表示事件发生了；值为0：表示事件未发生

任意任务或中断都可以读写这些位

可以等待某一位成立，或者等待多位同时成立

## 介绍

一个事件组就包含了一个 EventBits_t 数据类型的变量，变量类型 EventBits_t 的定义如下所示： 

```c
typedef TickType_t EventBits_t;
#if ( configUSE_16_BIT_TICKS  = =  1 )
	typedef   uint16_t   TickType_t;
#else
	typedef   uint32_t   TickType_t;
#endif
#define  configUSE_16_BIT_TICKS    0 

```

> EventBits_t 实际上是一个 16 位或 32 位无符号的数据类型 

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%9B%BE%E7%89%871.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241115164003.png)

## API 函数

```c
xEventGroupCreate();   //使用动态方式创建事件标志组
xEventGroupCreateStatic();  //使用静态方式创建事件标志组
xEventGroupClearBits();  //清零事件标志位
xEventGroupClearBitsFromISR();  //在中断中清零事件标志位
xEventGroupSetBits();  //设置事件标志位
xEventGroupSetBitsFromISR();  //在中断中设置事件标志位
xEventGroupWaitBits();  //等待事件标志位
xEventGroupSync();  //设置事件标志位，并等待事件标志位
```

### 创建函数

```c
EventGroupHandle_t xEventGroupCreate( void ) 
```

**返回值	**:	NULL -->事件标志组创建失败			其他值--->事件标志组创建成功，返回其句柄



### 清除事件标志位

```c
EventBits_t xEventGroupClearBits( EventGroupHandle_t xEventGroup, const EventBits_t uxBitsToClear )
```

形参：xEventGroup -->待操作的事件标志组句柄				uxBitsToClear--->待清零的事件标志位

返回值：整数 --->清零事件标志位之前事件组中事件标志位的值



### 设置事件标志位

```c
EventBits_t xEventGroupSetBits( EventGroupHandle_t xEventGroup, const EventBits_t uxBitsToSet )
```

形参：xEventGroup -->待操作的事件标志组句柄 				uxBitsToSet--->待设置的事件标志位

返回值：整数 --->函数返回时，事件组中的事件标志位值



### 等待事件标志位

```c
EventBits_t xEventGroupWaitBits( EventGroupHandle_t xEventGroup,		//等待的事件标志组句柄
                                 const EventBits_t uxBitsToWaitFor,			//	等待的事件标志位，可以用逻辑或等待多个事件标志位	
                                 const BaseType_t xClearOnExit,		/*功等待到事件标志位后，清除事件组中对应的事件标志位，
																pdTRUE  ：清除uxBitsToWaitFor指定位；pdFALSE：不清除*/
                                 const BaseType_t xWaitForAllBits,		/*等待 uxBitsToWaitFor 中的所有事件标志位（逻辑与）  																	            pdTRUE：等待的位，全部为1	pdFALSE：等待的位，某个为1*/
                                 TickType_t xTicksToWait ) 		//等待的阻塞时间
```

返回值：等待的事件标志位值 --->等待事件标志位成功，返回等待到的事件标志位		其他值--->等待事件标志位失败，返回事件组中的事件标志位

> 特点：可以等待某一位、也可以等待多位、等到期望的事件后，还可以清除某些位

### 同步函数

```c
EventBits_t xEventGroupSync(    EventGroupHandle_t xEventGroup,		//等待事件标志所在事件组
                          	    const EventBits_t uxBitsToSet, 		//达到同步点后，要设置的事件标志
                            	const EventBits_t uxBitsToWaitFor,		//等待的事件标志
                                 TickType_t xTicksToWait );		//等待的阻塞时间
```

返回值：等待的事件标志位值 --->等待事件标志位成功，返回等待到的事件标志位		其他值--->等待事件标志位失败，返回事件组中的事件标志位

> 例子：
>
> Task1：做饭、Task2：做菜
>
> Task1做好自己的事之后，需要等待菜也做好，大家在一起吃饭。





# 任务通知

### 介绍

任务通知：用来通知任务的，任务控制块中的结构体成员变量 ulNotifiedValue就是这个通知值。

**使用队列、信号量、事件标志组时都需另外创建一个结构体，通过中间的结构体进行间接通信！**

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/1231.png)

**使用任务通知时，任务结构体TCB中就包含了内部对象，可以直接接收别人发过来的"通知"**

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%9B%BE%E7%89%87123.png)

> 如果不用任务通知，那么改变一个数据就先得创建一个队列，然后在任务一中将数据发送给队列，任务二再从队列中读出数据
>
> 而任务通知是直接改变任务二的成员变量，任务二在调用的时候变量已经给改变了，所以没有中间的通讯对象。

#### 更新方式

* 不覆盖接受任务的通知值 -->如果任务中的变量没有值，则进行覆写，如果有值则不覆写
* 覆盖接受任务的通知值 --> 不管任务中的通知值有没有值，都进行覆写
* 更新接受任务通知值的一个或多个bit
* 增加接受任务的通知值	-->通知值加1

> 只要合理，灵活的利用任务通知的特点，可以在一些场合中替代队列、信号量、事件标志组！

#### 优势劣势

**优势：**

* 效率更高-->使用任务通知向任务发送事件或数据比使用队列、事件标志组或信号量快得多
* 使用内存更小-->使用其他方法时都要先创建对应的结构体，使用任务通知时无需额外创建结构体

**劣势：**

* 无法发送数据给ISR-->ISR没有任务结构体，所以无法给ISR发送数据。但是ISR可以使用任务通知的功能，发数据给任务。(ISR:中断)
* 无法广播给多个任务-->任务通知只能是被指定的一个任务接收并处理，对象是确定的。
* 无法缓存多个数据-->任务通知是通过更新任务通知值来发送数据的，任务结构体中只有一个任务通知值，只能保持一个数据。
* 发送受阻不支持阻塞-->发送方无法进入阻塞状态等待

## 任务通知值

任务都有一个结构体：任务控制块TCB，它里边有两个结构体成员变量：

```c
typedef  struct  tskTaskControlBlock 
{
	… …
    	#if ( configUSE_TASK_NOTIFICATIONS  ==  1 )
        	volatile  uint32_t    ulNotifiedValue [ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
        	volatile  uint8_t      ucNotifyState [ configTASK_NOTIFICATION_ARRAY_ENTRIES ];
    	endif
	… …
} tskTCB;
#define  configTASK_NOTIFICATION_ARRAY_ENTRIES	1  	/* 定义任务通知数组的大小, 默认: 1 */

```

> 使用前必须将configUSE_TASK_NOTIFICATIONS置1

* 一个是 uint32_t 类型，用来表示通知值
* 一个是 uint8_t 类型，用来表示通知状态

### 更新方式

* 计数值（数值累加，类似信号量）
* 相应位置一（类似事件标志组）
* 任意数值（支持覆写和不覆写，类似队列）

## 通知状态

```c
#define	taskNOT_WAITING_NOTIFICATION  	( ( uint8_t ) 0 )		 /* 任务未等待通知 */
#define taskWAITING_NOTIFICATION		( ( uint8_t ) 1 )		 /* 任务在等待通知 */
#define taskNOTIFICATION_RECEIVED      	( ( uint8_t ) 2 )		 /* 任务在等待接收 */

```

* 任务未等待通知 ：任务通知默认的初始化状态（直接发送）
* 等待通知：接收方已经准备好了（已经调用了接收任务通知函数），等待发送方给个通知
* 等待接收：发送方已经发送出去（已经调用了发送任务通知函数），等待接收方接收

## API函数

任务通知API函数主要有两类：①发送通知 ，②接收通知。

**注意**：发送通知API函数可以用于**任务**和中断服务函数中；**接收通知**API函数只能用在任务中。

### 发送通知

```c
xTaskNotify()		        //发送通知，带有通知值
xTaskNotifyAndQuery() 		//发送通知，带有通知值并且保留接收任务的原通知值
xTaskNotifyGive()		   //发送通知，不带通知值,通信值++
xTaskNotifyFromISR()		    /*			            	*/
xTaskNotifyAndQueryFromISR() 	/*在中断中发送任务通知   		*/
vTaskNotifyGiveFromISR()		/*						*/
```



```c
BaseType_t xTaskNotify( TaskHandle_t xTaskToNotify, uint32_t ulValue, eNotifyAction eAction );
BaseType_t xTaskNotifyAndQuery( TaskHandle_t xTaskToNotify, uint32_t ulValue, eNotifyAction eAction, uint32_t *pulPreviousNotifyValue );
BaseType_t xTaskNotifyGive( TaskHandle_t xTaskToNotify );
```

```c
BaseType_t xTaskGenericNotify( TaskHandle_t xTaskToNotify,  //接收任务通知的任务句柄
                               UBaseType_t uxIndexToNotify,  //任务的指定通知（任务通知相关数组成员)
                               uint32_t ulValue,		//任务通知值
                               eNotifyAction eAction,  //通知方式（通知值更新方式）
                               uint32_t * pulPreviousNotificationValue )  //用于保存更新前的任务通知值（为NULL则不保存）
```

> 发送通知函数本质上就是调用了这一个函数，唯一的区别在于入口参数不一样

#### 任务通知方式

```c
typedef  enum
{    
	eNoAction = 0, 			/* 无操作 */
	eSetBits				/* 更新指定bit */
	eIncrement				/* 通知值加一 */
 	eSetValueWithOverwrite		/* 覆写的方式更新通知值 */
	eSetValueWithoutOverwrite	/* 不覆写通知值 */
} eNotifyAction;

```



### 接收通知

```c
ulTaskNotifyTake()  //获取任务通知，可以设置在退出此函数的时候将任务通知值清零或者减一。
				  //当任务通知用作二值信号量或者计数信号量的时候，使用此函数来获取信号量。
xTaskNotifyWait()  //获取任务通知，比 ulTaskNotifyTak()更为复杂，可获取通知值和清除通知值的指定位
```

```c
uint32_t ulTaskNotifyTake( BaseType_t xClearCountOnExit, TickType_t xTicksToWait );
//实际调用的是下面那个函数
uint32_t ulTaskGenericNotifyTake( UBaseType_t uxIndexToWaitOn,		//任务的指定通知（任务通知相关数组成员）
                                  BaseType_t xClearCountOnExit,		//指定在成功接收通知后，将通知值清零或减 1，
                                 								 //pdTRUE：把通知值清零；pdFALSE：把通知值减一
                                  TickType_t xTicksToWait )		//阻塞等待任务通知值的最大时间
```

返回值：0-->接收失败		非0-->接收成功，返回任务通知的通知值

```c
BaseType_t xTaskNotifyWait( uint32_t ulBitsToClearOnEntry, 
                           uint32_t ulBitsToClearOnExit, 
                           uint32_t *pulNotificationValue, 
                           TickType_t xTicksToWait );
//实际调用的是下面那个函数
BaseType_t xTaskGenericNotifyWait( UBaseType_t uxIndexToWaitOn,  //任务的指定通知（任务通知相关数组成员）
                                   uint32_t ulBitsToClearOnEntry,  //等待前清零指定任务通知值的比特位（旧值对应bit清0）
                                   uint32_t ulBitsToClearOnExit,//成功等待后清零指定的任务通知值比特位（新值对应bit清0）
                                   uint32_t * pulNotificationValue, //用来取出通知值（如果不需要取出，可设为NULL）
                                   TickType_t xTicksToWait ) //阻塞等待任务通知值的最大时间
```

> 此函数用于获取通知值和清除通知值的指定位值，适用于模拟队列和事件标志组，使用该函数来获取任务通知 。 

返回值：pdTRUE-->等待任务通知成功		pdFALSE-->等待任务通知失败

#### 总结

* 当任务通知用作于信号量时，使用函数获取信号量：ulTaskNotifyTake()
* 当任务通知用作于事件标志组或队列时，使用此函数来获取： xTaskNotifyWait()

# 软件定时器

## 简介

**定时器：** 从指定的时刻开始，经过一个指定时间，然后触发一个超时事件，用户可自定义定时器的周期

**硬件定时器：** 芯片本身自带的定时器模块，硬件定时器的精度一般很高，每次在定时时间到达之后就会自动触发一个中断，用户在**中断服务函数**中处理信息。

**软件定时器**： 是指具有定时功能的软件，可设置定时周期，当指定时间到达后要调用**回调函数**（也称超时函数），用户在**回调函数**中处理信息

### **软件定时器优缺点**

**优点**： 硬件定时器数量有限，而软件定时器理论上只需有足够内存，就可以创建多个；

**缺点**： 软件定时器相对硬件定时器来说，精度没有那么高（因为它以系统时钟为基准，系统时钟中断优先级又是最低，容易被打断）。 对于需要高精度要求的场合，不建议使用软件定时器。

### 特点

* **可裁剪**：软件定时器是可裁剪可配置的功能， 如果要使能软件定时器，需将configUSE_TIMERS 配置项配置成 1 
* **单次和周期**：软件定时器支持设置成：单次定时器或周期定时器 

**注意** ：软件定时器的超时回调函数是由软件定时器服务任务调用的，软件定时器的超时回调函数本身不是任务，因此不能在该回调函数中使用可能会导致任务阻塞的 API 函数。

**软件定时器服务任务**：在调用函数 vTaskStartScheduler()开启任务调度器的时候，会创建一个用于管理软件定时器的任务，这个任务就叫做**软件定时器服务任务**。

**软件定时器服务任务作用：** 

1. 负责软件定时器超时的逻辑判断
2. 调用超时软件定时器的超时回调函数
3. 处理软件定时器命令队列

### 命令队列

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%9B%BE%E7%89%871231.png)

FreeRTOS 提供了许多软件定时器相关的 API 函数，这些 API 函数大多都是往定时器的队列中写入消息（发送命令），这个队列叫做软件定时器命令队列，是提供给 FreeRTOS 中的软件定时器使用的，用户是不能直接访问的。 

## 相关配置

* 当FreeRTOS 的配置项 configUSE_TIMERS 设置为1，在启动任务调度器时，会自动创建软件定时器的服务/守护任务prvTimerTask( ) ；
* 软件定时器服务任务的优先级为 configTIMER_TASK_PRIORITY = 31；
* 定时器的命令队列长度为 configTIMER_QUEUE_LENGTH = 5 ；  -->这个值是自己配置的， 一般是5

**注意**：软件定时器的超时回调函数是在软件定时器服务任务中被调用的，服务任务不是专为某个定时器服务的，它还要处理其他定时器。

> 所以，定时器的回调函数不要影响其他“人”：
>
> 1、回调函数要尽快实行，不能进入阻塞状态，即不能调用那些会阻塞任务的 API 函数，如：vTaskDelay() ；
>
> 2、访问队列或者信号量的非零阻塞时间的 API 函数也不能调用。 



## 状态

软件定时器共有两种状态：

休眠态：软件定时器可以通过其句柄被引用，但因为没有运行，所以其定时超时回调函数不会被执行

运行态：运行态的定时器，当指定时间到达之后，它的超时回调函数会被调用

**注意**：新创建的软件定时器处于休眠状态 ，也就是未运行的！ 



## 单次定时器和周期定时器

FreeRTOS 提供了两种软件定时器：

* **单次定时器**： 单次定时器的一旦定时超时，只会执行一次其软件定时器超时回调函数，不会自动重新开启定时，不过可以被手动重新开启。
* **周期定时器**：周期定时器的一旦启动以后就会在执行完回调函数以后自动的重新启动 ，从而周期地执行其软件定时器回调函数。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%9B%BE%E7%89%871232.png)

> Timer1：周期定时器，定时超时时间为 2 个单位时间，开启后，一直以2个时间单位间隔重复执行； 
>
> Timer2：单次定时器，定时超时时间为 1 个单位时间，开启后，则在第一个超时后就不在执行了。

###    **软件定时器的状态转换图**

单次定时器状态转换图：

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%9B%BE342%E7%89%873.png)

周期定时器状态转换图：

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/123.png)

## API函数

### 结构体成员介绍

```c
typedef struct tmrTimerControl                  
    {
        const char * pcTimerName;                 /* 软件定时器名字(没有多大作用) */
        ListItem_t xTimerListItem;                  /* 软件定时器列表项 */
        TickType_t xTimerPeriodInTicks;           /* 软件定时器的周期 */     
        void * pvTimerID;                         /* 软件定时器的ID */
        TimerCallbackFunction_t pxCallbackFunction;  /* 软件定时器的回调函数(两个定时器调用同一个回调函数时用ID判断是哪个定时器调用) */
        #if ( configUSE_TRACE_FACILITY == 1 )
            UBaseType_t uxTimerNumber;             /*  软件定时器的编号，调试用  */
        #endif
        uint8_t ucStatus;                          /*  软件定时器的状态  */
    } xTIMER;
```

### 函数

```c 
xTimerCreate()  		//动态方式创建软件定时器
xTimerCreateStatic() 	//静态方式创建软件定时器
xTimerStart() 		   //开启软件定时器定时
xTimerStartFromISR()   //在中断中开启软件定时器定时
xTimerStop()    	   //停止软件定时器定时
xTimerStopFromISR()   //在中断中停止软件定时器定时
xTimerReset()         //复位软件定时器定时
xTimerResetFromISR()  //在中断中复位软件定时器定时
xTimerChangePeriod()  //更改软件定时器的定时超时时间
xTimerChangePeriodFromIS ()  //在中断中更改定时超时时间
```

* 创建软件定时器API函数
* 开启软件定时器API函数
* 停止软件定时器API函数
* 复位软件定时器API函数
* 更改软件定时器超时时间API函数

#### 创建定时器

```c
TimerHandle_t xTimerCreate(  const char * const pcTimerName,  //软件定时器名
                                TickType_t xTimerPeriodInTicks,  //定时超时时间，单位：系统时钟节拍
                                BaseType_t xAutoReload,  //定时器模式， pdTRUE：周期定时器， pdFALSE：单次定时器
                                void * pvTimerID,  //软件定时器 ID，用于多个软件定时器公用一个超时回调函数
                                TimerCallbackFunction_t pxCallbackFunction   //软件定时器超时回调函数
                          )
```

返回值：NULL-->软件定时器创建失败    其他值-->软件定时器创建成功，返回其句柄



#### 开启定时器

```c
BaseType_t xTimerStart( TimerHandle_t xTimer, TickType_t xTicksToWait );
```

形参：xTimer-->待开启的软件定时器的句柄	xTicksToWait-->发送命令到软件定时器命令队列的最大等待时间

返回值：pdPASS-->软件定时器开启成功		pdFAIL-->软件定时器开启失败

#### 停止软件定时器

```c
BaseType_t   xTimerStop(  TimerHandle_t 	xTimer,	const TickType_t 	xTicksToWait); 
```

形参：xTimer-->待停止的软件定时器的句柄			xTicksToWait-->发送命令到软件定时器命令队列的最大等待时间

返回值：pdPASS-->软件定时器停止成功		pdFAIL-->软件定时器停止失败

#### 复位软件定时器

```c
BaseType_t xTimerReset( TimerHandle_t xTimer, TickType_t xTicksToWait );
```

形参：xTimer-->待复位的软件定时器的句柄		xTicksToWait-->发送命令到软件定时器命令队列的最大等待时间

返回值：pdPASS-->软件定时器复位成功		pdFAIL-->软件定时器复位失败

> 该功能将使软件定时器的重新开启定时，复位后的软件定时器以复位时的时刻作为开启时刻重新定时

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%9B%BE432%E7%89%875.png)

#### 更改软件定时器超时事件

```c
BaseType_t xTimerChangePeriodFromISR( TimerHandle_t xTimer,//待更新的软件定时器的句柄
                                         TickType_t xNewPeriod,//新的定时超时时间，单位：系统时钟节拍
                                         BaseType_t *pxHigherPriorityTaskWoken //发送命令到软件定时器命令队列的最大等待时间
                                    );
```

返回值：pdPASS-->软件定时器定时超时时间更改成功		pdFAIL-->软件定时器定时超时时间更改失败

# 低功耗模式

## 介绍

很多应用场合对于功耗的要求很严格，比如可穿戴低功耗产品、物联网低功耗产品等

一般MCU都有相应的低功耗模式，裸机开发时可以使用MCU的低功耗模式。

[FreeRTOS](https://so.csdn.net/so/search?q=FreeRTOS&spm=1001.2101.3001.7020)也提供了一个叫Tickless的低功耗模式，方便带FreeRTOS操作系统的应用开发

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241115211225.png)

## 低功耗详解

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241115211327.png)

为了可以降低功耗，又不影响系统运行，该如何做？

> 可以在本该空闲任务执行的期间，让MCU 进入相应的低功耗模式；当其他任务准备运行的时候，唤醒MCU退出低功耗模式

难点：

* 进入低功耗之后，多久唤醒？也就是下一个要运行的任务如何被准确唤醒
* 任何中断均可唤醒MCU，若滴答定时器频繁中断则会影响低功耗的效果？

> 将滴答定时器的中断周期修改为低功耗运行时间
>
> 退出低功耗后，需补上系统时钟节拍数

## 配置

1. lconfigUSE_TICKLESS_IDLE      (此宏用于使能低功耗 Tickless 模式 )
2. lconfigEXPECTED_IDLE_TIME_BEFORE_SLEEP    (此宏用于定义系统进入相应低功耗模式的最短时长)
3. lconfigPRE_SLEEP_PROCESSING(x)   (此宏用于定义需要在系统进入低功耗模式前执行的事务，如：进入低功耗前关闭外设时钟，以达到降低功耗的目的)
4. l configPOSR_SLEEP_PROCESSING(x)  (此宏用于定义需要在系统退出低功耗模式后执行的事务，如：退出低功耗后开启之前关闭的外设时钟，以使系统能够正常运行)

> **注意**： 第三步和第四步的函数是需要自己写的，需要自己找到#define lconfigPRE_SLEEP_PROCESSING后面定义一个函数，然后在这个函数中将自己打开的时钟关掉，然后在第四步的函数中打开
