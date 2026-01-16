# 大概流程

## IAP工程搭建

1. 下载官方工程
2. 根据芯片调整工程
3. APP区编写
    1. STM32存储
4. 生成.bin文件
5. 串口IAP升级
    1. ymodem通信协议
    2. 串口通信协议
    3. iap技术
6. 蓝牙OTA
    1. QT上位机
    2. C++ 
    3. 蓝牙浅应用

# isp技术

## 介绍

IAP（In Application Programming，在应用中编程）是一种嵌入式系统中常用的固件更新技术。它允许用户程序在运行过程中，通过预留的通信接口（如串口、USB、网络等）对设备的固件进行更新升级。这种技术极大地提高了设备的可维护性和灵活性，特别是在智能家居、汽车电子、物联网设备等需要频繁更新固件的场景中尤为重要。

无论是ICP技术还是ISP技术，都需要有机械性的操作如连接下载线，设置跳线帽等。若产品的电路板已经层层密封在外壳中，要对其进行程序更新无疑困难重重，若产品安装于狭窄空间等难以触及的地方，更是一场灾难。但若进引入了IAP技术，则完全可以避免上述尴尬情况，而且若使用远距离或无线的数据传输方案，甚至可以实现远程编程和无线编程。这绝对是ICP或ISP技术无法做到的。某种微控制器支持IAP技术的首要前提是其必须是基于可重复编程闪存的微控制器。STM32微控制器带有可编程的内置闪存，同时STM32拥有在数量上和种类上都非常丰富的外设通信接口，因此在STM32上实现IAP技术是完全可行的。

## 工作原理

### 基本原理

IAP技术通过将Flash存储器划分为两个主要区域来实现固件更新：

Bootloader区域：包含引导加载程序，负责初始化硬件、设置内存映射，并在需要时加载和更新用户应用程序（User Application）。Bootloader出厂后通常固定不变，只有在特定条件下（如接收到升级指令）才执行更新操作。

User Application区域：存放用户的应用程序代码，这部分代码在需要时可以通过Bootloader进行更新。

### 启动流程

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/f84c8c7763f642829cea22dd5a0b163e.png)



![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/76ab536479414544be1a295fa969309e.png)

### 工作流程

1. 设备启动：设备上电后，首先执行Bootloader程序。Bootloader检查是否有升级指令或新固件数据待处理。
2. 固件更新检查：如果检测到有升级需求，Bootloader通过预留的通信接口接收新固件数据，并将其写入Flash的User Application区域。
3. 固件验证：写入完成后，Bootloader进行固件验证，确保数据完整性和正确性
4. 跳转执行：验证通过后，Bootloader跳转到User Application区域的新固件执行。

### 关键步骤

1. Flash存储器管理：包括擦除扇区、写入数据和校验数据等操作，这些操作通常以块或扇区为单位进行。
2. 数据传输与接收：通过串口、USB等通信接口接收新固件数据，并存储在RAM中，待验证无误后写入Flash。
3. 安全性与完整性校验：通过加密、签名和校验和等技术确保数据在传输和存储过程中的安全性和完整性。

## Bootloader

Bootloader是一个用户自定义的升级程序，因此Bootloader是一个独立的工程项目。在实际开发过程中，Bootloader是很少会需要进行调整的，因为Bootloader的调整就需要拆机重新烧录，会相当的麻烦。因此一家企业的Bootloader都需要反复的确认无误之后再进行应用程序开发。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/a8840059d349453596521f6b4b84a1ee.png)

# ymodem协议

## 帧格式

一帧的数据格式：帧头、包号、包号反码、数据、校验

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241120150439.png)

### 帧头

帧头表示两种数据帧长度，主要是信息块长度不同。

【1】以SOH(0x01)开始的数据包，信息块是128字节，该类型帧总长度为133字节。2的7次方为128

【2】以STX(0x02)开始的数据包，信息块是1024字节，该类型帧总长度为1029字节。2的10次方等于1024

### 包序号

包序号为数据块的编号，将要传送的数据进行分块编号，包序号只有一个字节，范围0~255;对于数据包大于255的，则序号归零，重复计算。如下：

数据帧0，数据帧1，数据帧2...数据帧255，

数据帧0，数据帧1，数据帧2...数据帧255，

...

包序号和取反包序号，帧序的取反，YModem特地这么做是为了给数据是否正确提供一种判断依据，通过判断这两个字节是否为取反关系，就可以知道数据是否传输出错。

### 校验

Ymodem采用的是CRC16校验算法，校验值为2字节，传输。crc高八位在前面，低八位在后面；CRC计算数据为信息块数据，不包括帧头、包头、包号反码。

## 通讯过程

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/ed681468adb29659d7be3edefba5594c.png)

### 握手信号

通讯：SENDR(发送方)和RECIVER(接收方)

接收方需要发送Ymodem_c(字符C，ASII码为0x43)命令后，发送方收到收才可以开始发送起始帧

发送方和接收方建立连接后，发送方首先发送一个包含文件信息的头文件数据包。
接收方接收到头文件后，进行校验并返回一个ACK（确认）信号。

### 起始帧

Ymodem起始帧并不直接传输文件的内容，而是现将文件名和文件大小置于数据帧中传输；起始帧是以SOH，133字节长度帧传输，格式如下：

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241120151455.png)

其中包号为固定为0x00；Filename为文件名称，文件名称后必须加0x00作为结束；Filesize为文件大小值，文件大小值后必须加0x00作为结束；余下未满128字节数据区域，则以0x00填充

### 数据帧

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241120151554.png)

【1】对于SOH帧，若余下数据小于128字节，则以0x1A填充，该帧长度仍为133字节。

【2】对于STX帧需考虑几种情况：

●余下数据等于1024字节，以1029长度帧发送；

●余下数据小于1024字节，但大于128字节，以1029字节帧长度发送，无效数据以0x1A填充。

●余下数据等于128字节，以133字节帧长度发送。

●余下数据小于128字节，以133字节帧长度发送，无效数据以0x1A填充。

发送方开始按照顺序发送数据包，每个数据包包含数据内容和校验信息。
接收方接收到数据包后，进行校验。如果数据包无误，接收方返回ACK信号；如果有误，返回NAK（否认）信号，请求重传。

### 结束帧

Ymodem的结束帧采用SOH 133字节长度帧传输的，该帧不携带数据（空包），即数据区、校验都以0x00填充。

当所有数据包传输完毕后，发送方发送一个EOT（End of Transmission）信号，表示文件传输结束。
接收方接收到EOT信号后，返回一个ACK信号确认，并等待下一个文件的头文件数据包，或者断开连接。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241120151653.png)

### Ymodem命令

1）EOT、CAN信号由发送端发送

2）ASK、NAK、C信号由接收端发出

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241120152034.png)

### 分类

Ymodem分成YModem-1K与YModem-g

YModem-1K用1024字节信息块传输取代标准的128字节传输，数据的发送回使用CRC校验，保证数据传输的正确性。它每传输一个信息块数据时，就会等待接收端回应ACK信号，接收到回应后，才会继续传输下一个信息块，保证数据已经全部接收。

YModem-g传输形式与YModem-1K差不多，但是它去掉了数据的CRC校验码，同时在发送完一个数据块信息后，它不会等待接收端的ACK信号，而直接传输下一个数据块。正是它没有涉及错误校验，才使得它的传输速度比YModem-1K来得块。

一般都会选择YModem-1K传输，平时所说的YModem也是指的是YModem-1K




# CRC16校验算法

CRC（循环冗余校验）是一种常用的错误检测码，用于检测数据在传输或存储过程中是否发生错误。CRC16是一种使用16位CRC多项式的校验算法。以下是一个常见的CRC16算法的实现，通常被称为CRC-16-IBM或CRC-16-ANSI，其多项式为`0x8005`（即`x^16 + x^15 + x^2 + 1`）。

```c
#include <stdint.h>
#include <stddef.h>

uint16_t crc16(const uint8_t *data, size_t length) {
    uint16_t crc = 0xFFFF;  // 初始值
    for (size_t i = 0; i < length; ++i) {
        crc ^= (uint16_t)data[i];  // 将数据与CRC寄存器异或

        for (uint8_t j = 0; j < 8; ++j) {  // 对每一位数据进行处理
            if (crc & 0x0001) {
                crc = (crc >> 1) ^ 0xA001;  // 如果最低位为1，右移一位后与多项式异或
            } else {
                crc >>= 1;  // 如果最低位为0，直接右移一位
            }
        }
    }
    return crc;
}

// 使用示例
int main() {
    uint8_t data[] = {0x31, 0x32, 0x33, 0x34, 0x35};  // 示例数据 "12345"
    uint16_t result = crc16(data, sizeof(data) / sizeof(data[0]));
    // 输出CRC16结果
    printf("CRC16: 0x%04X\n", result);
    return 0;
}

```

这个函数接收一个字节数组`data`和数据的长度`length`，然后计算并返回16位的CRC校验值。在校验过程中，数据每一位都进行了处理，如果数据位为1，则将CRC寄存器右移一位后与多项式`0xA001`（即`0x8005`的反序）进行异或操作。

请注意，CRC算法有多种变体，其多项式、初始值、结果是否反序、输入数据是否反序等可能有所不同。在实际应用中，需要根据具体协议或标准选择正确的CRC算法实现

# 软件实现

1. **Bootloader编写**
    1. 使用Keil MDK等开发工具编写Bootloader程序，该程序负责初始化硬件、检查升级指令、接收新固件数据并写入Flash
    2. 设置中断向量表偏移量，确保新固件的中断向量表能够被正确识别和执行。
2. **User Application编写**
    1. 编写用户应用程序，实现具体功能。
    2. 在需要时，通过某种机制（如特定命令）触发Bootloader进行固件升级。
3. **Flash分区**
    1. 在链接脚本中明确划分Bootloader和User Application的Flash区域大小。
    2. 确保两个区域不重叠，并预留足够的空间用于固件更新。
4. **固件升级流程**
    1. 设备上电后，Bootloader首先检查是否有升级指令。
    2. 如果有，则通过串口等通信接口接收新固件数据。
    3. 接收完成后，进行固件验证和写入操作。
    4. 验证无误后，跳转到新固件执行

## 注意事项

* Flash擦除和写入的性能：Flash存储器的擦除和写入操作相对较慢，需要仔细设计IAP流程以确保系统稳定性和可靠性。
* 中断和异常处理：在IAP执行过程中，需要妥善处理中断和异常以防止系统崩溃。
* 电源管理：确保在IAP执行过程中设备有足够的电源供应，特别是在写入大量数据时。
* 安全性考虑：实现强大的安全性措施以保护固件更新过程和数据传输的安全性。

# 官方例程

## bootload

使用STM32自带的ISP官方例程

选择芯片，编译检查

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/fdca311928469a6b35edfefc1771d668.png)

由于STM32F103C8T6的内存是中等容量大小，我们还需要在下面进行修改，将阴影处的“xxxHD_VL”内容更换为“xxxMD”：

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/5f378a945036429dacda747d5e377b1e.png)

选择自己的烧录器，比如我使用的是PWLINK，就选择CMSIS-DAP，大家根据自己的LINK填写：

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/184a1ff21e94917d3061eceb33711357.png)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/25b583ccf0969d46c1bd2532a859f8ac.png)

烧录方式选择为SW模式

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241120130051.png)

然后将bootload改小一点：(这个例程用2k即可)

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/6135bfe16b02f88dc3535f61fc367e24.png)

应用地址也调整一下，注意：这个地址在后面APP工程中会用到，需要对应一致，记住这个0x8005000：

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/96923836e9f3d75b55e5b7a58a7b23f9.png)

后面将bootload程序烧录进去就可以，串口1波特率是115200

> 一次调好的工程可以直接copy直接用

## APP

APP区就是我们用户自己自定义的具体功能区，起始地址从Bootload之后，利用bootload将跳转到APP区，这里不过多介绍，只要保证bootload的跳转地址是APP的起始地址即可。

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/13fb867335cda10384241694eb013705.png)

还有一个重点是，IAP升级时，我们用SecureCRT软件通过串口利用Ymodem协议传输.bin固件文件，所以首先我们需要在编译后生成bin文件：

```c
$K\ARM\ARMCC\bin\fromelf.exe --bin --output=@L.bin !L
```

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/62531b276fc09e616eec5abc924ff31e.png)

编译后生成.bin文件

使用野火调试助手

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241120130610.png)

> 出现那个目录即表明bootload启动成功，发送数据1即可开始接收数据

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20241120130703.png)

通过YModem协议发送程序的.bin文件

![](https://cdn.jsdelivr.net/gh/lgdd-29/NoteBook/Notephoto/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20241120130750.png)

发送3之后单片机即开始自己烧录传送过去的固件升级包

**注意：**使用FreeRTOS的时候要在main()添加一下代码

```c
SCB->VTOR = FLASH_BASE | 0x8005000;  //中断向量偏移表
```

是为了告诉FreeRTOS我中断向量偏移了，这样FreeRTOS才能运行。

## 代码流程

UART串口初始化，复位键初始化

如果自定义键没按下

{

则UART串口重新初始化并且想对方发送介绍

并进入主菜单功能：

计算用户程序将要加载的Flash块编号

根据型号创建掩码，也就是地址，然后根据这个地址检查Flash是否有给写保护

如果给写保护则告诉对方不能发送，

如果没有给写保护，则获取串口给的按键值(1,2,3,4)

根据1，2，3，4的ACILL码进行if判断是否进入哪个程序

1-->在Flash中下载用户应用程序

2-->从Flash上传用户应用程序

3-->跳转用户应用程序

获取用户程序地址，执行用户程序应用

4-->接触Flash写保护

}

如果自定义键按下

{

​	读取用户应用程序的入口点地址，并存在JumAddress变量中

​	跳转到用户应用程序的入口点

}

> 自定义按键可以在stm32100e_eval.h文件中找到Key push-button的宏定义，在里面即可改动main中的自定义按键，可以根据这个IO口的高低电平来选择bootload中刚启动是否要烧录新的程序或者用以前的程序，如果接高电平则用的是以前的程序，如果接低电平进入串口接收界面。不过只有复位或者刚上电之后才能选择，意味着每次复位或者上电才能进行固件升级。不过可以加个



### 代码解析

