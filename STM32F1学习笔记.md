# STM32学习笔记

# 前言

stm32是一个很有用的微控制器，在各种本科竞赛或者项目中可以说是最主流的一种设备了，但是现在网上32相关的课程很多，同学们可能都不知道如何学习，所幸江科大的某位大佬自制了51和32的教程，算得上是B站最浅显易懂的教程了，本人笔记是根据另一位同学的开源笔记改编（这个同学也是看的江科大视频自制，其视频[链接](https://www.bilibili.com/video/BV1GY41127w9/?spm_id_from=333.337.search-card.all.click&vd_source=eea47a16439992e41b232bc5d5684e27)），同时融合了一些其他的，作为开源分享给大家

本人笔记的GitHub仓库如下

https://github.com/Michael-Jetson/STM32_Note

## STM32是什么

STM32是ST公司基于ARM Cortex-M内核开发的32位微控制器，如果大家不太理解微控制器是什么，可以联想一下电脑芯片——电脑芯片就是一个超高配版的微控制器，具备强大的逻辑处理功能，芯片搭配主板和显卡等设备，组成一个电脑，可以完成一系列复杂任务；实际上stm32就是一款超低配版的电脑，其CPU就是32芯片，各种外设就相当于电脑的显卡、内存等。

ARM既指ARM公司，也指ARM处理器内核ARM公司是全球领先的半导体知识产权（IP）提供商，全世界超过95%的智能手机和平板电脑都采用ARM架构ARM公司设计ARM内核，半导体厂商完善内核周边电路并生产芯片，类比一下就是ARM是建筑图纸设计方，半导体厂商是建筑承包施工方，共同完成产品开发

![image-20230410105904654](https://github.com/Michael-Jetson/STM32_Note/blob/main/img/image-20230410105904654.png?raw=true)

一块32开发板，类比成大家熟知的台式电脑，那么实际上内核就是CPU，存储器就是内存和硬盘，外设就是显示器、键盘等

## 32的系统架构



## 新建工程方式总结

我们在使用32单片机的时候需要对其进行编程，过程比大家学过的C/C++编程方法复杂一些

# GPIO

- GPIO（General Purpose Input Output）是通用输入输出口，也就是我们速成的IO口，类似于电脑上的USB口，可以双向传输数据
- 可配置为8种输入输出模式
- 引脚电平：0V~3.3V，部分引脚可容忍5V，具体看引脚型号
- 输出模式下可控制端口输出高低电平，用以驱动LED、控制蜂鸣器、模拟通信协议输出时序等，也就是32输出信号，控制其他元件
- 输入模式下可读取端口的高低电平或电压，用于读取按键输入、外接模块电平信号输入、ADC电压采集、模拟通信协议接收数据等，也就是32读取其他元件的数据

## GPIO的基本结构

 ![image-20230124011346061](https://gitee.com/pu-heliang/photos/raw/master/img/202301240113129.png)

其中左边紫色的箭头是APB2总线，上面挂载了很多GPIO外设模块

GPIO外设的命名是GPIO+A/B/C这种，然后一个GPIO外设有16个引脚，从0至15编号

在每个GPIO模块内，包含了一个寄存器和一个驱动器，内核通过APB2总线对寄存器进行读写操作，然后寄存器可以刷新和保存这个数据（读取模式时，寄存器可以不断读取引脚信号并且保存；输出模式时，寄存器可以保存上一次内核输入的数据进行输出，直至下一次内核的数据输入），这样就可以完成输出/读取电平的功能了

**PS**：如果不熟悉寄存器原理的同学，可以在数字电子技术课程上进行学习相关知识

这里寄存器的每一位对应一个引脚，实际上因为32单片机是32位的，所以内部寄存器也是32位的，但是一个GPIO模块只有16个引脚，所以寄存器只有低16位有对应端口，高16位并没有用到

此外GPIO里面还有一个驱动器，顾名思义，驱动器就是用来增加电信号的驱动能力了，实际上寄存器就是用来存储数据的，如果想完成具体的输出，需要驱动器增大驱动能力

### GPIO结构图

![image-20230124011458002](https://gitee.com/pu-heliang/photos/raw/master/img/202301240114039.png)

这是GPIO的结构图

可以分为两个部分，一个是输入驱动器部分（包含相关电路），可以实现读取功能；一个是输出驱动器部分，可以实现信号的输出

### GPIO原理

我们接下来进行解析工作原理

一、IO引脚

- 首先是最右边的IO引脚，接了两个保护二极管，可以对输入信号的电压进行限幅，让输入电压不至于太高也不至于太低

- 如果出现电压过高，那么上方的二极管就会导通，电流就会流过VDD，而不会通过内部电路，电压过低也会出现类似情况，不会从内部电路汲取电流

二、输入驱动器

- 首先，输入驱动器右边是上下拉电阻，这个是可以通过程序进行设置的；如果上面导通、下面断开，就是上拉输入模型，如果下面断开、下面导通就是下拉输入模式，如果都断开就是浮空输入模式，注意不可以同时导通

- 上拉/下拉模式，其实就是配置一个默认输入电平，因为对于数字端口，输入必须是高电平或者低电平，如果引脚什么也不接，则无法判断是什么电平，如果这样，输入就会出现一种浮空状态，引脚的输入电平容易受外界干扰而改变，类似于一个太空中的物体，一点外力就可以很轻松改变其运动状态，所以为了避免引脚悬空导致的输入数据不确定，就要加上上拉/下拉电阻![image-20230124011954329](https://gitee.com/pu-heliang/photos/raw/master/img/202301240119393.png)

- 如果是上拉模式，上图中连接VSS的开关断开，连接VDD的闭合，接入上拉电阻，二极管反向连接，电阻相当于无穷大，当外部没有输入时。引脚默认为高电平；当然，这里的电阻阻值很大，不会影响正常的输入输出，是一种弱上拉和弱下拉

- 施密特触发器（图中的TTL肖特基触发器，视频中说这是一个翻译错误）的作用是对输入电压进行整形，其执行逻辑如下

  - 如果输入电压大于某一个阈值，输出就会瞬间改为高电平

  - 如果输入电压小于某一个阈值，输出就会瞬间改为低电平

  - 实际上，外界引脚输入虽然是数字信号，但是实际上也是会出现一些干扰的，加上这个施密特触发器，会保障读取信号的过程中不会受干扰，如可以避免输入信号抖动导致的数字信号改变

    **PS**：这个部分的原理也在数字电子技术课程中

三、输入部分（指的是外界输入）

- 输入驱动器里面引出三路信号，分别用作：模拟输入，复用功能输入、输入数据寄存器
- 模拟输入：这路信号引出点是施密特触发器前面，因为施密特触发器会对模拟输入进行整形，从而输出数字信号，如果我们想获取原始的模拟输入信号就必须使用这个功能
- 复用功能输入：这个是链接到其他需要读取端口的外设上的，比如说串口的输入引脚等，这根线是接受数字量的，所以在施密特触发器后面

四、输出部分（指的是内部输出）

- 输出驱动器有两路信号来源，一路其中复用功能输出是来自于其他外设的，一路是来自于输出数据寄存器的，两种控制方式由数据选择器进行控制，选择一路到输出控制部分

- 如果选择输出数据寄存器进行控制，就是普通的IO口输出，对输出数据寄存器进行写入就可以对对应端口进行操作，但是注意，输出数据寄存器只能整体写入，如果想对单独操作输出数据寄存器某一位而不影响其他位，就需要借助位设置/清除寄存器，可以实现单独操作输出数据寄存器的某一位

- 如果直接使用读/写操作对输出数据寄存器进行操作，那么会相当麻烦，如果想单独控制其中某一个端口而不影响其他端口，必须这样操作
  1. 首先，读取寄存器，使用按位与或者按位或的方式单独改变其中某一位
  2. 然后将修改后的数据发送给寄存器

- 或者使用位设置/清楚寄存器，使用方法如下
  1. 如果要对某一位进行置1（输出高电平），则在位设置寄存器对应位写1，其他位置0即可（保持不变），这样内部电路就会自动将输出数据寄存器对应位置1而不改变其他位，一步到位
  2. 如果想对某一位清除（输出低电平），则对位清楚寄存器操作即可

  这个是库函数实现的方法

五、输出驱动器

- 我们使用信号来控制开关（也就是两种MOS管）的导通和关闭，开关负责将IO口接到VDD或者VSS，这样IO口就可以输出高低电平

- 有三种输出方式：推挽输出，开漏输出，关闭

  1. 推挽输出![image-20230410143846371](https://github.com/Michael-Jetson/STM32_Note/blob/main/img/image-20230410143846371.png?raw=true)

     这种模式下，P-MOS和N-MOS均有效

     输出1时，上管导通，下管断开，输出直接接到VDD，输出高电平

     输出0时，上管断开，下管断开，输出直接接到VSS，输出低电平

     这种模式下，高低电平均匀有较强的驱动能力（输出时，IO口直接链接内部电路的电源），所以推挽模式也叫强推输出模式，这种模式下，STM32对IO口有绝对控制权，决定IO口的输出

  2. 开漏输出模式

     这种模式下，P-MOS无效，仅有N-MOS工作

     数据寄存器为1时，下管断开，输出相当于断开，也就是高阻模式

     数据寄存器为0时，下管导通，输出接到VSS，输出低电平

     这种模式下，只有低电平有驱动能力，高电平没有驱动能力

     这种模式可以作为通信协议的驱动方式，比如说I2C的通信引脚

     在多机通信的情况下，这种模式可以避免各个设备之间的相互干扰

     另外，开漏模式还可以用来输出5V的电平信号，比如在IO口外接一个上拉电阻到5V电源，当寄存器输出低电平的时候，内部的N-MOS直接接VSS，当输出高电平时，由外部上拉电阻拉高至5V，这样可以输出5V电平信号，兼容5V设备

  3. 关闭

     当引脚配置为输入模式的时候，两个MOS管都无效，输出关闭，端口电平由外部信号来控制

### GPIO工作模式列表

| 模式名称 | 性质 | 特征 |
| :------: | :--: | :--: |
|浮空输入	|数字输入	|可读取引脚电平，若引脚悬空，则电平不确定|
|上拉输入	|数字输入	|可读取引脚电平，内部连接上拉电阻，悬空时默认高电平|
|下拉输入	|数字输入	|可读取引脚电平，内部连接下拉电阻，悬空时默认低电平|
|模拟输入	|模拟输入	|GPIO无效，引脚直接接入内部ADC|
|开漏输出	|数字输出	|可输出引脚电平，高电平为高阻态，低电平接VSS|
|推挽输出	|数字输出	|可输出引脚电平，高电平接VDD，低电平接VSS|
|复用开漏输出|	数字输出	|由片上外设控制，高电平为高阻态，低电平接VSS|
|复用推挽输出	|数字输出	|由片上外设控制，高电平接VDD，低电平接VSS|

注意一下模拟输入，这个可以说是ADC模数转换器的专属配置了，这里输出断开，施密特触发器无效，相当于只剩下一根线，也就是从引脚直接接入片上外设也就是ADC，此外的七个模式，所有的输入都是有效的

### GPIO使用注意事项

首先，在输入模式下，输出都是无效的（这会干扰输入），但是在输出模式下，输入是有效的，这是因为一个端口只能有一个输出，但是可以有多个输出

下面两张图是使用GPIO点亮发光二极管的不同方式，第一张是低电平驱动，第二张是高电平驱动

限流电阻的左右：可以限制电路防止二极管烧毁，同时可以调节亮度

![image-20230410163815402](https://github.com/Michael-Jetson/STM32_Note/blob/main/img/image-20230410163815402.png?raw=true)

![image-20230410163831363](https://github.com/Michael-Jetson/STM32_Note/blob/main/img/image-20230410163831363.png?raw=true)

两种驱动方式取决于GPIO的驱动能力，但是在单片机里面趋向于第一张接法，因为很多单片机采用了高电平弱驱动，低电平强驱动的规则

但是注意一下，GPIO的驱动能力并不是很强，哪怕是推挽输出模式下，所以对于某些功率较大的元件，必须使用单独的驱动模块（如三极管），下图就是一个使用三极管驱动蜂鸣器的电路

![image-20230410164414499](https://github.com/Michael-Jetson/STM32_Note/blob/main/img/image-20230410164414499.png?raw=true)

## GPIO配置

我们清晰了解了GPIO的原理与结构，那么我们应该怎么对其进行配置来让其工作呢？我们首先来完成点亮LED

### 步骤：

1. 第一步，使用RCC开启GPIO的时钟

   我们都知道，在数电里面逻辑电路很多都需要时钟信号的驱动才可以工作，所以我们想让GPIO开始工作就需要给GPIO模块输入时钟信号，也称为时钟使能

2. 第二步，使用GPIO_Init()函数初始化GPIO

   我们需要先设置GPIO的工作模式等，才可以进一步使用

3. 第三步，使用输出或者输入的函数控制GPIO口

   完成数据的输入输出等

### 时钟使能函数

在原视频中，在项目下的Library文件夹中，有一个stm32f10x_rcc.h文件，顾名思义，这个文件就是stm32f10x系列的rcc头文件，其中我们可以看到有三个函数，其作用就是时钟使能

```c
void RCC_AHBPeriphClockCmd(uint32_t RCC_AHBPeriph,FunctionalState NewState);
void RCC_APB2PeriphClockCmd(uint32_t RCC_APB2Periph,FunctionalState NewState);
void RCC_APB1PeriphClockCmd(uint32_t RCC_APB1Periph,FunctionalState NewState);
```

三个函数分别是RCC中AHB、APB1和APB2的外设时钟控制，两个参数的作用是

参数1：选择外设，参数2：使能或者失能

当我们编译项目之后，可以右键查看函数定义和介绍，我们发现，我们想使用一个GPIO外设（假定这里对GPIOA进行使能），那么第一个参数就使用RCC_APB2Periph_GPIOA，第二个参数选择ENABLE或者DISABLE，这些都是预定义好的变量

### 常用的GPIO函数

#### 复位GPIO外设函数

```c++
void GPIO_DeInit(GPIO_TypeDef* GPIOx);
```

调用这个函数之后，所指定的GPIO外设会被复位（也就是初始化或者恢复出厂设置）

#### 复位AFIO外设函数

```c++
void GPIO_AFIODeInit(void);
```

#### 初始化GPIO口函数

用结构体的参数来初始化GPIO口，先定义一个结构体变量，然后把再给结构体赋值（结构体的值代表配置GPIO端口的方式），最后调用此函数并且传入结构体，函数内部会自动读取结构体的值，然后自动把外设的各个参数配置好

```cpp
void GPIO_Init(GPIO_TypeDef* GPIOx,GPIO_InitTypedef* GPIO_InitStruct);
```

#### GPIO初始化结构体

```c++
GPIO_InitTypeDef GPIO_InitStructure;
```

声明一个GPIO初始化结构体对象

```
GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
```

设置GPIO的工作模式（这是一些枚举定义，在stm32f10x_gpio.h中定义）

| 变量名 | 含义 | 英文解释 |
| :-: | :-: | :-: |
| GPIO_Mode_AIN |模拟输入 | Analog IN |
| GPIO_Mode_IN_FLOATING |浮空输入|In Floating|
| GPIO_Mode_IPD |下拉输入|In Pull Down|
| GPIO_Mode_IPU |上拉输入|In Pull Up|
|GPIO_Mode_Out_OD|开漏输出|Out Open Drain|
| GPIO_Mode_Out_PP |推挽输出|Out Push Pull|
| GPIO_Mode_AF_OD |复用开漏|Atl Open Drain|
| GPIO_Mode_AF_PP |复用推挽|Atl Push Pull|

点灯使用推挽输出

```cpp
GPIO_InitStructure.GPIO_Pin = GPIO_Pin_All;
```

同样我们可以在stm32f10x_gpio.h文件中找到引脚定义

```cpp
#define GPIO_Pin_0                 ((uint16_t)0x0001)  /*!< Pin 0 selected */
#define GPIO_Pin_1                 ((uint16_t)0x0002)  /*!< Pin 1 selected */
#define GPIO_Pin_2                 ((uint16_t)0x0004)  /*!< Pin 2 selected */
#define GPIO_Pin_3                 ((uint16_t)0x0008)  /*!< Pin 3 selected */
#define GPIO_Pin_4                 ((uint16_t)0x0010)  /*!< Pin 4 selected */
#define GPIO_Pin_5                 ((uint16_t)0x0020)  /*!< Pin 5 selected */
#define GPIO_Pin_6                 ((uint16_t)0x0040)  /*!< Pin 6 selected */
#define GPIO_Pin_7                 ((uint16_t)0x0080)  /*!< Pin 7 selected */
#define GPIO_Pin_8                 ((uint16_t)0x0100)  /*!< Pin 8 selected */
#define GPIO_Pin_9                 ((uint16_t)0x0200)  /*!< Pin 9 selected */
#define GPIO_Pin_10                ((uint16_t)0x0400)  /*!< Pin 10 selected */
#define GPIO_Pin_11                ((uint16_t)0x0800)  /*!< Pin 11 selected */
#define GPIO_Pin_12                ((uint16_t)0x1000)  /*!< Pin 12 selected */
#define GPIO_Pin_13                ((uint16_t)0x2000)  /*!< Pin 13 selected */
#define GPIO_Pin_14                ((uint16_t)0x4000)  /*!< Pin 14 selected */
#define GPIO_Pin_15                ((uint16_t)0x8000)  /*!< Pin 15 selected */
#define GPIO_Pin_All               ((uint16_t)0xFFFF)  /*!< All pins selected */
```

实际上，这些引脚定义就是设置了高电平的位置来进行引脚的初始化，学过数电的都知道，数字电路中表示数值的方式就是二进制的多位电平，但是在这里，变量使用的是16进制的数值，所以0x0001（也就是GPIO_Pin_0）等于二进制的0000 0000 0000 0001，0x0002（也就是GPIO_Pin_1）等于二进制的0000 0000 0000 0010

在原码表示下，二进制的十六位数就可以表示16个属性的状态，在这里正好对应一个GPIO外设的16个引脚，所以实际上的就是设置对应位置的电平，第$i$位电平是高电平，那么等于设置$i-1$号引脚完成初始化，比如说我们可以使用二进制数字0000 0000 0000 0001来完成0号引脚的初始化

如果想设置多个引脚，那么就可以一次将多个电平置为高电平，这里可以采用C语言中的与或非操作，即使用

```c++
GPIO_pin=GPIO_Pin_0 | GPIO_Pin_1
```

这样，就在底层进行按位或操作，得到的变量的值是0000 0000 0000 0011（二进制）和0x0003（十六进制），这样可以完成多个引脚的初始化

不过注意一下，一个结构体对象想一次性设置一个GPIO外设的多个引脚，那么这些引脚的工作模式和执行速度是一样的，如果想差异化设置则需要使用不同结构体对象

```cpp
GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
GPIO_Init(GPIOA, &GPIO_InitStructure);
```

GPIO_Init函数的作用是读取结构体参数，进行一系列逻辑判断运算，最后写入GPIO配置寄存器

至此，GPIO的配置就完成了，接下来就可以对其进行操作了

#### 给GPIO结构体变量赋一个默认值函数

```c++
void GPIO_StructInit(GPIO_InitTypedef* GPIO_InitTypedef);
```

## GPIO的输出函数

### 把指定的端口设置为高电平

```c
void GPIO_SetBits(GPIO_InitTypedef* GPIOx,uint16_t GPIO_Pin);
GPIO_SetBits(GPIOA,GPIO_Pin_0);
```

这里将GPIOA的0号引脚设为高电平，也可以使用按位与或者按位或的方式去同时设置多个引脚

### 把指定的端口设置为低电平

```c
void GPIO_ResetBits(GPIO_InitTypedef* GPIOx,uint16_t GPIO_Pin);
```

与GPIO_SetBits函数相同的用法

### 对根据第三个参数的值来设置电平

```c
void GPIO_WriteBit(GPIO_InitTypedef* GPIOx,uint16_t GPIO_Pin,BitAction BitVal);
//用法示例
GPIO_WriteBit(GPIOA,GPIO_Pin_0,Bit_SET);
GPIO_WriteBit(GPIOA,GPIO_Pin_0,Bit_RESET);
```

对GPIO的0号引脚分别置高电平和低电平

这里注意一下，使用这种直接输入数字0/1的方式会导致编译警告

```c++
GPIO_WriteBit(GPIOA,GPIO_Pin_0,1);
GPIO_WriteBit(GPIOA,GPIO_Pin_0,0);
```

警告原因是枚举类型中混入其他类型变量，所以我们需要使用一个强制类型转换
```c++
GPIO_WriteBit(GPIOA,GPIO_Pin_0,(BitAction)1);
GPIO_WriteBit(GPIOA,GPIO_Pin_0,(BitAction)0);
```

### 对GPIOx 16个端口同时进行写入操作：

```c
void GPIO_Write(GPIO_InitTypedef* GPIOx,uint16_t PortVal);
```

在推挽输出模式下，高低电平都具有驱动能力，开漏输出模式的高电平是没有驱动能力的，开漏输出模式的低电平具有驱动能力

#define的新名字在左边，并且可以给任何变量换名字，而typedef只能给变量换名字，新名字在右边

## GPIO的输入函数

### 常见输入模块

按键是一个常用的输入模块，但是存在抖动情况，最简单的滤除方法就是检测到信号变化之后，过20ms再次检测

![](./img/28.png)

其他的传感器模块，有着光照越强/温度越高/红外线越强，电阻就越小的特性，但是阻值无法直观体现，所以会串联一个定值分压电阻，同时并联一个电容，起到滤波作用，保证电路稳定，波形平稳

![](./img/29.png)

这个时候，传感器相当于一个上下拉电阻，这样就可以得到模拟输出（AO）了

当然这个模块还支持数字输出，是通过一个电压比较器实现的

### 读取输入数据寄存器某个端口的输入值，返回值是高低电平函数

```c
uint8_t GPIO_ReadInputDataBit(GPIO_InitTypedef* GPIOx,uint16_t GPIO_Pin);
```

### 读取GPIO的每一位的值，返回值是16位的数据,每一位代表一个端口值

```c
uint16_t GPIO_ReadInputData(GPIO_InitTypedef* GPIOx);
```

#### 读取输出数据寄存器的某一位

```c
uint8_t GPIO_ReadOutputDataBit(GPIO_InitTypedef* GPIOx,uint16_t GPIO_Pin);
```

#### 读取整个输出寄存器

```c
uint16_t GPIO_ReadOutputData(GPIO_InitTypedef* GPIOx);
```

### 程序示例

```c
void LED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);//开启GPIO时钟，这里是开启一个GPIO外设的时钟
	
	GPIO_InitTypeDef GPIO_InitStructure;//定义GPIO结构体
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;//推挽输出
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_2;//打开的引脚
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;//配置响应速度
	GPIO_Init(GPIOA, &GPIO_InitStructure);//写入参数
	
	GPIO_SetBits(GPIOA, GPIO_Pin_1 | GPIO_Pin_2);//置高电平
}

void LED1_ON(void)
{
	GPIO_ResetBits(GPIOA, GPIO_Pin_1);
}

void LED1_OFF(void)
{
	GPIO_SetBits(GPIOA, GPIO_Pin_1);
}

void LED1_Turn(void)
{
	if (GPIO_ReadOutputDataBit(GPIOA, GPIO_Pin_1) == 0)//读取输出引脚的电平
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_1);
	}
	else
	{
		GPIO_ResetBits(GPIOA, GPIO_Pin_1);//设置低电平
	}
}

void LED2_ON(void)
{
	GPIO_ResetBits(GPIOA, GPIO_Pin_2);
}

void LED2_OFF(void)
{
	GPIO_SetBits(GPIOA, GPIO_Pin_2);
}

void LED2_Turn(void)
{
	if (GPIO_ReadOutputDataBit(GPIOA, GPIO_Pin_2) == 0)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_2);
	}
	else
	{
		GPIO_ResetBits(GPIOA, GPIO_Pin_2);
	}
}

```

## 硬件电路

![30](.\img\30-1685534466716-1.png)

这是常用的硬件电路接法，左侧两种是直接连接，引脚必须是上拉或者下拉模式，中间两种是外接上下拉电阻接法，可以是浮空输入模式，一般使用上面两种接法

# 外部中断

- 中断：在主程序运行过程中，出现了特定的中断触发条件（中断源），使得CPU暂停当前正在运行的程序，转而去处理中断程序，处理完成后又返回原来被暂停的位置继续运行
- 中断优先级：当有多个中断源同时申请中断时，CPU会根据中断源的轻重缓急进行裁决，优先响应更加紧急的中断源
- 中断嵌套：当一个中断程序正在运行时，又有新的更高优先级的中断源申请中断，CPU再次暂停当前的中断程序，转而去处理新的中断程序，处理完后依次进行返回
- NVIC：NVIC的中断优先级由优先级寄存器的4位（0~15）决定，这4位可以进行切分，分为高n位的抢占优先级和低4-n位的响应优先级
- 抢占优先级高的可以进行中断嵌套，响应优先级高的可以进行优先排队，抢占优先级和响应优先级均相同的按中断号排队

- EXTI：（Extern Interrupt）外部中断
- EXTI可以检测指定GPIO口的电平信号，当其指定的GPIO口产生电平变化时，EXTI将立即向NVIC发出中断申请，经过NVIC裁决后即可中断CPU主程序，使CPU执行EXTI对应的中断程序
- 支持的触发方式：上升沿/下降沿/双边沿/软件触发
- 支持的GPIO口：所有GPIO口，但相同的Pin不能同时触发中断
- 通道数：16个GPIO_Pin，外加PVD输出、RTC闹钟、USB唤醒、以太网唤醒
- 触发响应方式：中断响应/事件响应

![image-20221224115729670](https://gitee.com/pu-heliang/photos/raw/master/img/202301032043708.png)

AFIO选择中断引脚，外部中断的9-5,15-10会触发同一个中断函数，再根据标志位来区分到底是哪个中断进来的

配置数据选择器，只有一个Pin接到EXTI

![image-20230103203938222](https://gitee.com/pu-heliang/photos/raw/master/img/202301032039837.png)

在STM32中AFIO主要完成两个任务：复用功能引脚重映射、中断引脚选择

或、与、非门

![image-20221224120857813](https://gitee.com/pu-heliang/photos/raw/master/img/202301032044678.png)

## EXTI配置步骤

1. 第一步，配置RCC，把设计到的外设时钟都打开
2. 第二步，配置GPIO，选择端口为输入模式
3. 第三步，配置AFIO，选择使用的一路GPIO，连接到后面的EXTI
4. 第四步，配置EXTI，选择边沿触发方式，选择触发响应方式
5. 第五步，配置NVIC，给中断选择一个合适的优先级

EXTI和NVIC时钟默认是打开的，NVIC是内核的外设，内核的外设都不需要开启时钟，RCC管的都是内核外的外设

### 复位AFIO外设

```c
void GPIO_AFIODeInit(void);
```

##### 锁定GPIO配置函数

##### 锁定引脚的配置，防止意外更改

```c
void GPIO_PinLockConfig(GPIO_TypeDef* GPIOx,uint16_t GPIO_Pin);
```

##### 配置AFIO的事件输出功能函数

```c
void GPIO_EventOutputConfig(uint8_t GPIO_PortSource,uint8_t GPIO_PinSource);
void GPIO_EventOutputCmd(FunctionalState NewState);
```

##### 配置引脚重映射函数

```c
void GPIO_PinRemapConfig(uint32_t GPIO_Remap,FunctionalState NewState);
```

##### 配置AFIO的数据选择器

选择想使用的中断引脚函数

```c
void GPIO_EXTILineConfig(uint8_t GPIO_PortSource,uint8_t GPIO_PinSource);
```

##### 恢复上电默认的状态函数

```c
void EXTI_DeInit(void);
```

##### 根据结构体配置EXTI外设函数

```c
void EXTI_Init(EXTI_InitTypedef* EXTI_InitStruct);
```

##### 给传入的结构体参数赋一个默认值函数

```c
void EXTI_StructInit(EXTI_InitTypedef* EXTI_InitStruct);
```

##### 软件触发外部中断函数

参数给一个中断线，就能软件触发一次这个外部中断函数

```c
void EXTI_GenerateSWInterrupt(uint32_t EXTI_Line);
```

在外设运行的时候会产生一些状态标志位，例如：外部中断来了，挂起寄存器会置一个标志位，标志位放在状态寄存器，

当程序想看这些标志位

##### 获取指定的标志位函数

```c
FlagStatus EXTI_GetFlagStatus(uint32_t EXTI_Line);
```

##### 对置1的标志位进行清除函数

```c
void EXTI_ClearFlag(uint32_t EXTI_Line);
```

##### 在中断函数中获取标志位函数

```c
ITStatus EXTI_GetITStatus(uint32_t EXTI_Line);
```

##### 清除中断挂起标志位函数

```c
void EXTI_ClearITPendingBit(uint32_t EXTI_Line);
```

##### 中断分组函数

```c
void NVIC_PriorityGroupConfig(uint32_t NVIC_PriorityGroup);
```

##### 根据结构体里面的参数初始化NVIC函数

```c
void NVIC_Init(NVIC_InitTypedef* NVIC_InitStruct);
```

##### 设置中断向量表函数

NVIC_SetVectorTable函数的功能是设置向量表的位置和偏移。其中输入参数中，对于32位的OFFSET向量表基地址的偏移量对于FLASH，参数值必须高于0x08000100，对于RAM必须高于0X100.

```c
void NVIC_SetVectorTable(uint8_t NVIC_VectTab,uint32_t Offset);
```

##### 系统低功耗配置函数

```c
void NVIC_SystemLPConfig(uint8_t LowPowerMode,FunctionalState NewState)
```

中断函数要简短快速，不要在中断中执行Delay

#### 程序示例

```c
int16_t Encoder_Count;

void Encoder_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);//开启GPIO时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);//开启AFIO时钟
	
	GPIO_InitTypeDef GPIO_InitStructure;//定义初始化结构体
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;//上拉输入
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;//开启引脚
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;//设置响应速度
	GPIO_Init(GPIOB, &GPIO_InitStructure);//配置参数
	
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource0);//选择中断线路
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource1);
	
	EXTI_InitTypeDef EXTI_InitStructure;//定义外部中断结构体
	EXTI_InitStructure.EXTI_Line = EXTI_Line0 | EXTI_Line1;//设置中断线
	EXTI_InitStructure.EXTI_LineCmd = ENABLE;//开启中断线路
	EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;//中断模式
	EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;//下降沿触发
	EXTI_Init(&EXTI_InitStructure);//写入参数
	
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//中断优先级分组
	
	NVIC_InitTypeDef NVIC_InitStructure;//定义NVIC结构体
	NVIC_InitStructure.NVIC_IRQChannel = EXTI0_IRQn;//设置中断通道
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;//通道使能
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;//抢占优先级
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;//响应优先级
	NVIC_Init(&NVIC_InitStructure);//写入参数

	NVIC_InitStructure.NVIC_IRQChannel = EXTI1_IRQn;//设置中断通道
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;//通道使能
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;//抢占优先级
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 2;//响应优先级
	NVIC_Init(&NVIC_InitStructure);//写入参数
}

int16_t Encoder_Get(void)
{
	int16_t Temp;
	Temp = Encoder_Count;
	Encoder_Count = 0;
	return Temp;
}

void EXTI0_IRQHandler(void)//线路0中断函数
{
	if (EXTI_GetITStatus(EXTI_Line0) == SET)//判断中断挂起位
	{
		/*如果出现数据乱跳的现象，可再次判断引脚电平，以避免抖动*/
		if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_0) == 0)//读取输入高低电平
		{
			if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)
			{
				Encoder_Count --;
			}
		}
		EXTI_ClearITPendingBit(EXTI_Line0);//清除中断挂起标志位
	}
}

void EXTI1_IRQHandler(void)//线路1中断函数
{
	if (EXTI_GetITStatus(EXTI_Line1) == SET)//判断标志位
	{
		/*如果出现数据乱跳的现象，可再次判断引脚电平，以避免抖动*/
		if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)//读取输入高低电平
		{
			if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_0) == 0)
			{
				Encoder_Count ++;
			}
		}
		EXTI_ClearITPendingBit(EXTI_Line1);//清除中断挂起标志位
	}
}

```



### 定时器

- TIM（Timer）定时器
- 定时器可以对输入的时钟进行计数，并在计数值达到设定值时触发中断
- 16位计数器、预分频、自动重装寄存器的时基单元，在72M计数时钟下可以实现最大59.65s的定时
- 不仅具备基本的定时器中断功能，而且还包含内外时钟源选择、输入捕获、输出比较、编码器接口、主从触发模式等多种功能
- 根据复杂度和应用场景分为了高级定时器、通用定时器、基本定时器三种类型

- 对72MHz计72个数就是1MHz，也就是1us的时间，计72000个数，那就是1KHz也就是1ms的时间

- 59.65s =65536 X 65536X 1/72M/(中断频率倒数)，

- STM32的定时器支持级联的模式：一个定时器的输出当做另一个定时器的输入最大定时时间就是59.65s X 65536 X 65536

![image-20221224173356252](https://gitee.com/pu-heliang/photos/raw/master/img/202301032044148.png)

- 预分频器（PSC）：对输入的基准频率提前进行一个分频的操作
- 实际分频系数 = 预分频器的值 + 1，最大可以写65535即65536分频 
- 计数器（CNT）：也是16位，值可以从0~65535，当计数器的值自增（自减）到目标值时，产生中断，完成定时
- 自动重装寄存器（）：也是16位当计数值等于自动重装值时，就是计时的时间到了，就会产生一个中断信号，并且清零计数器，计数器自动开始下一次的计数计时，计数值等于自动重装值的中断一般叫做“更新中断”，此更新中断就会通往NVIC，再配置好NVIC的定时器通道，定时器上的更新中断就会得到CPU的响应了，对应的事件叫做“更新事件”，更新事件不会触发中断，但可以触发内部其他电路的工作

![image-20221224174708831](https://gitee.com/pu-heliang/photos/raw/master/img/202301032044571.png)

- 从基准时钟，到预分频器，再到计数器，计数器自增，同时不断地与自动重装寄存器进行比较，计数器和自动重装寄存器的值相等时，即计时时间到，这时会产生一个更新中断和更新事件，CPU响应更新中断，就完成了定时中断的任务了。

#### 主从触发模式

使用定时器的主模式，可以把定时器的更新事件映射到触发输出TRGO（Trigger Out）的位置，TRGO直接接到DAC的触发转换引脚上，这样定时器的更新就不需要再通过中断来触发DAC转换了

![image-20221224181213236](https://gitee.com/pu-heliang/photos/raw/master/img/202301032044975.png)

缓冲寄存器：某个时刻把预分频器由0改成了1，当计数计到一半的时候改变了分频值，这个变化不会立即生效，而是会等到本次计数周期结束时，产生了了更新事件，预分频器的值才会被传递到缓冲寄存器里面去，才会生效。

举个例子来说，如果我们想改变ARR寄存器中的值，但是当前的定时还没有结束，在这时如果未设置影子寄存器，那么设定的值会立即生效。而如果设置了影子寄存器，那么新的值会在当前计数周期结束之后生效。

计数器计数频率：CK_CNT = CK_PSC / (PSC + 1)

计数器溢出频率：CK_CNT_OV = CK_CNT / (ARR + 1)  = CK_PSC / (PSC + 1) / (ARR + 1)  

#### 开启定时器步骤

1.  第一步，RCC开启时钟
2. 第二步，选择时基单元的时钟源
3. 第三步，配置时基单元
4. 第四步，配置输出中断控制，允许更新中断输出到NVIC
5. 第五步，配置NVIC，在NVIC中打开定时器中断的通道，并分配一个优先级
6. 第六步，运行控制
7. 第七步，使能计数器

#### 定时器常用的库函数

##### 恢复缺省配置函数

```c
void TIM_DeInit(TIM_TypeDef* TIMx);
```

##### 时基单元初始化函数

```c
void TIM_TimeBaseInit(TIM_TypeDef* TIMx, TIM_TimeBaseInitTypeDef* TIM_TimeBaseInitStruct);
```

##### 把结构体变量赋一个默认值函数

```c
void TIM_TimeBaseStructInit(TIM_TimeBaseInitTypeDef* TIM_TimeBaseInitStruct);
```

##### 使能计数器函数

```c
void TIM_Cmd(TIM_TypeDef* TIMx, FunctionalState NewState);
```

##### 使能中断输出信号函数

```c
void TIM_ITConfig(TIM_TypeDef* TIMx, uint16_t TIM_IT, FunctionalState NewState);
```

##### 选择内部时钟函数

```c
void TIM_InternalClockConfig(TIM_TypeDef* TIMx);
```

##### 选择ITRx其他定时器的时钟函数

```c
void TIM_ITRxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource);
```

##### 选择TIx捕获通道的时钟函数

```c
void TIM_TIxExternalClockConfig(TIM_TypeDef* TIMx, uint16_t TIM_TIxExternalCLKSource,uint16_t TIM_ICPolarity, uint16_t ICFilter);
```

参数3：输入的极性 参数4：滤波器

##### 选择ETR通过外部时钟模式1输入的时钟函数

```c
void TIM_ETRClockMode1Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter);
```

参数2：预分频器 参数3：输入的极性 参数4：滤波器

##### 选择ETR通过外部时钟模式2输入的时钟函数

```c
void TIM_ETRClockMode2Config(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter);
```

##### 单独配置ETR引脚的预分频器、极性、滤波器这些参数的函数

```c
void TIM_ETRConfig(TIM_TypeDef* TIMx, uint16_t TIM_ExtTRGPrescaler, uint16_t TIM_ExtTRGPolarity, uint16_t ExtTRGFilter);
```

##### 单独写预分频值函数

```c
void TIM_PrescalerConfig(TIM_TypeDef* TIMx, uint16_t Prescaler, uint16_t TIM_PSCReloadMode);
```

参数3：写入的模式，在更新事件生效，或者在写入后，手动产生一个更新事件，让这个值立刻生效

##### 改变计数器的计数模式函数

```c
void TIM_CounterModeConfig(TIM_TypeDef* TIMx, uint16_t TIM_CounterMode);
```

##### 自动重装器预装功能配置函数

TIM_ARRPreloadConfig设置为DISABLE 和ENABLE的问题，他的作用只是允许或禁止在定时器工作时向ARR的缓冲器中写入新值，以便在更新事件发生时载入覆盖以前的值。

```c
void TIM_ARRPreloadConfig(TIM_TypeDef* TIMx, FunctionalState NewState);
```

##### 给计数器写入一个值函数

```c
void TIM_SetCounter(TIM_TypeDef* TIMx, uint16_t Counter);
```

##### 给自动重装器写入一个值函数

```c
void TIM_SetAutoreload(TIM_TypeDef* TIMx, uint16_t Autoreload);
```

##### 获取当前计数器的值函数

```c
uint16_t TIM_GetCounter(TIM_TypeDef* TIMx);
```

##### 获取当前预分频器的值函数

```c
uint16_t TIM_GetPrescaler(TIM_TypeDef* TIMx);
```

使用跨文件的变量： extern声明变量，告诉编译器，有Num这个变量在别的文件中定义了，在此文件中也可以使用 

#### 程序示例：

```c
void Timer_Init(void)
{
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);//开启TIM2时钟
	
	TIM_InternalClockConfig(TIM2);//使用内部时钟
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;//定义时基单元结构体
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;//设置不分频
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;//设置向上计数
	TIM_TimeBaseInitStructure.TIM_Period = 10000 - 1;//ARR自动重装值
	TIM_TimeBaseInitStructure.TIM_Prescaler = 7200 - 1;//PSC不分频
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;//重复计数器的值，高级定时器特有
	TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStructure);//写入参数
	
	TIM_ClearFlag(TIM2, TIM_FLAG_Update);//清除更新标志位
	TIM_ITConfig(TIM2, TIM_IT_Update, ENABLE);//中断输出
	
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);//中断优先级分组
	
	NVIC_InitTypeDef NVIC_InitStructure;//NVIC结构体
	NVIC_InitStructure.NVIC_IRQChannel = TIM2_IRQn;//定时器通道
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;//使能
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 2;//抢占优先级
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;//响应优先级
	NVIC_Init(&NVIC_InitStructure);//写入参数
	
	TIM_Cmd(TIM2, ENABLE);//开启定时器
}

/*
void TIM2_IRQHandler(void)
{
	if (TIM_GetITStatus(TIM2, TIM_IT_Update) == SET)//判断是否中断溢出
	{
		
		TIM_ClearITPendingBit(TIM2, TIM_IT_Update);//清除中断标志位
	}
}
*/

```



### 输出比较

- OC（Output Compare）输出比较
- 输出比较可以通过比较CNT和CCR寄存器值的关系，来对输出电平进行置1、置0或翻转的操作，用于输出一定频率和占空比的PWM波形
- 每个高级定时器和通用定时器都拥有4个输出比较通道
- 高级定时器的前3个通道额外拥有死去生成和互补输出的功能

##### 输出比较常用的函数

##### 配置输出比较函数

```C
void TIM_OC1Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);
void TIM_OC2Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);
void TIM_OC3Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);
void TIM_OC4Init(TIM_TypeDef* TIMx, TIM_OCInitTypeDef* TIM_OCInitStruct);
```

##### 给输出比较结构体赋一个默认值函数

```C
void TIM_OCStructInit(TIM_OCInitTypeDef* TIM_OCInitStruct);
```

##### 配置强制输出模式函数

在运行中想要暂停输出波形并且强制输出高或者低电平，强制输出高电平和设置百分百占空比一样，强制输出低电平和设置百分百低电平是一样的。

```c
void TIM_ForcedOC1Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
void TIM_ForcedOC2Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
void TIM_ForcedOC3Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
void TIM_ForcedOC4Config(TIM_TypeDef* TIMx, uint16_t TIM_ForcedAction);
```

##### 配置CCR寄存器的预装功能函数

预装功能就是影子寄存器：写入的值不会立即生效，而是在更新事件才会生效

```c
void TIM_OC1PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
void TIM_OC2PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
void TIM_OC3PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
void TIM_OC4PreloadConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPreload);
```

##### 配置快速使能函数

```c
void TIM_OC1FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
void TIM_OC2FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
void TIM_OC3FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
void TIM_OC4FastConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCFast);
```

##### 外部事件时清除REF信号函数

```c
void TIM_ClearOC1Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
void TIM_ClearOC2Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
void TIM_ClearOC3Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
void TIM_ClearOC4Ref(TIM_TypeDef* TIMx, uint16_t TIM_OCClear);
```

##### 单独设置输出比较的极性函数

带N的是高级定时器里互补通道的配置函数

```c
void TIM_OC1PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
void TIM_OC1NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);
void TIM_OC2PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
void TIM_OC2NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);
void TIM_OC3PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
void TIM_OC3NPolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCNPolarity);
void TIM_OC4PolarityConfig(TIM_TypeDef* TIMx, uint16_t TIM_OCPolarity);
```

##### 单独修改输出使能参数函数

```c
void TIM_CCxCmd(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_CCx);
void TIM_CCxNCmd(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_CCxN);
```

##### 选择输出比较模式函数

```c
void TIM_SelectOCxM(TIM_TypeDef* TIMx, uint16_t TIM_Channel, uint16_t TIM_OCMode);
```

##### 单独更改CCR寄存器的值的函数

```c
void TIM_SetCompare1(TIM_TypeDef* TIMx, uint16_t Compare1);
void TIM_SetCompare2(TIM_TypeDef* TIMx, uint16_t Compare2);
void TIM_SetCompare3(TIM_TypeDef* TIMx, uint16_t Compare3);
void TIM_SetCompare4(TIM_TypeDef* TIMx, uint16_t Compare4);
```

##### 使用高级定时器输出PWM时调用使能主输出函数

```c
void TIM_CtrlPWMOutputs(TIM_TypeDef* TIMx, FunctionalState NewState);
```

定时器输出需要使用复用推挽输出，开启复用推挽输出引脚的控制权才能交给片上外设，PWM波形才能通过引脚输出

##### 引脚重映射

开启AFIO时钟函数

```c
void GPIO_PinRemapConfig(uint32_t GPIO_Remap, FunctionalState NewState);
```

- 完全重映射：四个引脚全换
- 部分重映射：前面两个引脚变了或者后面两个引脚变了
- 调试端口不能做普通的GPIO口使用，需要解除复用

#### 程序示例

```c
void PWM_Init(void)
{
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);//开启TIM2时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);//开启GPIO时钟
	
	GPIO_InitTypeDef GPIO_InitStructure;//定义GPIO结构体
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;//复用推挽输出
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_2;//开启引脚
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;//配置响应速度
	GPIO_Init(GPIOA, &GPIO_InitStructure);//写入参数
	
	TIM_InternalClockConfig(TIM2);//使用内部时钟
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;//配置时基单元
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;//不分频
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;//向上计数
	TIM_TimeBaseInitStructure.TIM_Period = 100 - 1;		//ARR
	TIM_TimeBaseInitStructure.TIM_Prescaler = 36 - 1;		//PSC
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;//重复计数器
	TIM_TimeBaseInit(TIM2, &TIM_TimeBaseInitStructure);//写入参数
	
	TIM_OCInitTypeDef TIM_OCInitStructure;//定义输出比较结构体
	TIM_OCStructInit(&TIM_OCInitStructure);//给结构体赋默认值
	TIM_OCInitStructure.TIM_OCMode = TIM_OCMode_PWM1;//PWM1模式
	TIM_OCInitStructure.TIM_OCPolarity = TIM_OCPolarity_High;//有效电平为高电平
	TIM_OCInitStructure.TIM_OutputState = TIM_OutputState_Enable;//使能
	TIM_OCInitStructure.TIM_Pulse = 0;		//CCR
	TIM_OC3Init(TIM2, &TIM_OCInitStructure);//写入参数
	
	TIM_Cmd(TIM2, ENABLE);//开启时钟
}

void PWM_SetCompare3(uint16_t Compare)
{
	TIM_SetCompare3(TIM2, Compare);//设置CCR3的值
}

```



### 输入捕获

- IC（Input Capture）输入捕获
- 输入捕获模式下，当通道输入引脚出现指定电平跳变时，当前CNT的值将被锁存到CCR中，可用于测量PWM波形的频率、占空比、脉冲间隔、电平持续时间等参数
- 每个高级定时器和通用定时器都拥有4个输入捕获通道
- 可配置PWMI模式，同时测量频率和占空比
- 可配合主从触发模式，实现硬件全自动测量

##### 频率测量：

##### 测频法：在闸门时间T内，对上升沿计次，得到N，则频率

![image-20221226183437986](https://gitee.com/pu-heliang/photos/raw/master/img/202301032052812.png)

![image-20221226183513365](https://gitee.com/pu-heliang/photos/raw/master/img/202301032044214.png)

测频法：自定一个闸门时间T，通常设置为1s，在1s时间内，对信号上升沿计次，从0开始计，每来一个上升沿，计次+1，每来一个上升沿，其实就是来了一个周期的信号，在1s时间内，来个几个周期，频率就是多少Hz，（频率的定义：1s内出现了多少个重复的周期），这是一种直接按频率定义来测量的方法，闸门时间也可以是2s，计次值除2，就是频率

测频法测量的是一个闸门时间的多个周期自带一个均值滤波，如果在闸门时间内波形频率有变化，得到的其实是这一段时间的平均频率，测频法测量时间慢，测量结果是一段时间的平均值，值比较平滑

##### 测周法：两个上升沿内，以标准频率计次，得到N，则频率

![image-20221226184746596](https://gitee.com/pu-heliang/photos/raw/master/img/202301032052813.png)

![image-20221226184823583](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045325.png)

测周法：捕获信号的两个上升沿，测量之间持续的时间，使用一个已知的标准频率的计次时钟，来驱动计数器，从一个上升沿开始计，计数器从0开始，一直计到下一个上升沿，停止，计一个数的时间是1/fc，计N个数时间就是N/fc，N/fc就是周期，再取个倒数，就得到频率的公式，fx = fc/N

测周法只测量一个周期，就能出一次结果，出结果的速度取决于待测信号的频率，一般来说测周法结果更新更快，但是由于他只测量一个周期，所以结果值会受噪声的影响，波动比较大。

测频法适合测高频信号，测周法适合测量低频信号

例如：定了1s为闸门周期，结果1s内一个上升沿都没有，但不能认为频率是0，计次N很少时，误差会非常大，所以测频法适合测量高频率的信号，测周法适合低频信号，低频信号，周期比较长，计次就会比较多，有助于减少误差。如果待测频率太高，那么一个周期内只能计一两个数，如果待测信号再高一些，甚至一个数也计不到，不能认为频率无穷大

中界频率：测频法与测周法误差相等时的频率点（测频法和测周法的N相同）

![image-20221226193408126](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045027.png)

计数次数越多，+-1误差对结果的影响越小

待测频率<中界频率，测周法合适

待测频率>中界频率，测频法合适

异或门：当输入引脚的任何一个引脚有电平翻转时，输出引脚就产生一次电平翻转

![image-20221226194451569](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045812.png)

输入信号来到输入滤波器（对信号进行滤波，避免高频的毛刺信号误触发）和边沿检测器（可以选择高电平触发，或者低电平触发）

有两套滤波和边沿检测电路，第一套电路：经过滤波和极性选择得到TI1FP1，输入给通道1的后续电路，第二套电路：经过另一个滤波和极性选择得到TI1FP2，输入给下面通道2的后续电路，同理下面TI2的信号进来，也经过两套滤波和极性选择，得到TI2FP1和TI2FP2，其中TI2FP1输入给上面，TI2FP2输入给下面，两个输入信号进来可以选择各走各的，也可以选择进行交叉，让CH2引脚输入给通道1，或者CH1引脚输入给通道2，这样做的目的可以灵活切换后续捕获电路的输入，通过数据选择器进行灵活选择，可以把一个引脚的输入，同时映射到两个捕获单元，这是PWMI的经典结构，

例如，第一个捕获通道，使用上升沿触发，用来捕获周期，第二个通道，使用下降沿触发，用来捕获占空比，两个通道同时对一个引脚进行捕获，就可以同时测量频率和占空比，这就是PWMI模式。

TRC是为了无刷电机的驱动

输入信号进行滤波和极性选择后，来到预分频器，预分频器，每个通道各有一个，可以选择对前面的信号进行分频，分频之后的触发信号就可以触发捕获电路进行工作了，每来一个触发信号，CNT的值就会向CCR转运一次，转运的同时，会发送一个捕获事件，这个事件会在状态寄存器置标志位，同时也可以产生中断，如果需要再捕获期间处理事情就可以开启这个捕获中断

例如：配置上升沿触发捕获，每来一个上升沿，CNT转运到CCR一次，因为CNT计数器是由内部的标准时钟驱动的，所以CNT的数值，可以用来记录两个上升沿之间的时间间隔，这个时间间隔就是周期，再取个倒数就是测周法测量的频率了，

每次捕获后要把CNT清0，下次再上升沿再捕获的时候取出的CNT才是两个上升沿的时间间隔，可以用主从触发模式，自动来完成。

数字滤波器：由一个事件计数器组成，记录到N个事件后会产生一个输出的跳变，简单来说滤波器的工作原理就是，以采样频率对输入信号进行采样，当连续N个值都为高电平，输出才为高电平，连续N个值都为低电平输出才为低电平，如果信号出现高频抖动，导致连续采样N个值不全都一样，那输出就不会变化，这样就可以达到滤波的效果，采样频率越低，采样个数N越大，滤波效果就越好。

##### 主从触发模式：（主模式、从模式和触发源选择三个功能的简称）

![image-20221226201447496](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045959.png)

主模式：将定时器内部的信号映射到TRGO引脚，用于触发别的外设。

从模式：接收其他外设或者自身外设的一些信号，用于控制自身定时器的运行，也就是被别的信号控制。

触发源选择：选择从模式的触发信号源，也可以认为是从模式的一部分，触发源选择，选择一个指定的信号得到TRGI，TRGI去触发从模式，从模式可以在上述列表里，选择一项操作来自动执行。

例如：让TI1FP11信号自动触发CNT清零，触发源选择可以选择TI1FP1，从模式执行的操作，就可以选择执行Reset的操作，这样TI1FP1的信号就可以自动触发从模式，从模式自动清零CNT，实现硬件全自动测量

##### 输入捕获基本结构：

![image-20221226202515990](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045602.png)

只使用了一个通道，目前只能测量频率，配置好时基单元，启动定时器，CNT就会在预分频之后的时钟驱动下，不断自增，这个CNT就是测周法用来计数计时的，经过预分频之后的时钟频率就是，驱动CNT的标准频率fc，(标准频率 = 72M/预分频系数)，下面输入捕获通道1的GPIO口，输入一个上面的方波信号，经过滤波器和边沿检测，选择TI1FP1为上升沿触发，之后输入选择直连的通道分频器选择不分频，当TI1FP1出现上升沿之后，CNT的当前计数值转运到CCR1里，同时触发源选择，选择TI1FP1选择为触发信号，选中TI1FP1为触发信号，从模式选择复位操作，TI1FP1的上升沿也同样会通过上面的触发源选择那一路，取触发CNT清零，注意是先转运CNT的值到CCR里去，再出发从模式给CNT清零或者是非阻塞的同时转移，CNT的值转移到CCR，同时0转移到CNT里面去，不能是先清零CNT，再捕获，否则捕获值都是0了。

例如：左上角图，信号产生一个上升沿，CCR1 = CNT，就是把CNT的值转运到CCR1里面去，这是输入捕获自动执行的让CNT = 0，清零计数器（从模式自动执行的），在一个周期之内，CNT在标准时钟的驱动下，不断自增，并且由于之前清零过了，所以CNT就是从上升沿开始，从0开始计数一直++，指导，下一次上升沿来临，然后执行相同的操作，CCR1 = CNT，CNT = 0，第二次捕获时CNT，继续执行操作

如果信号频率太低，CNT的计数值可能会溢出

想使用从模式自动清除CNT，只能用通道1和通道2，对于通道3和通道4，就只能开启捕获中断，在中断里手动清零了。（这样做程序会处于频繁中断的状态，比较消耗软件资源）

#### 输入捕获程序示例

```c
RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	TIM_InternalClockConfig(TIM3);
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStructure.TIM_Period = 65536 - 1;		//ARR
	TIM_TimeBaseInitStructure.TIM_Prescaler = 72 - 1;		//PSC
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStructure);
	
	TIM_ICInitTypeDef TIM_ICInitStructure;//定义输入捕获结构体
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;//通道1
	TIM_ICInitStructure.TIM_ICFilter = 0xF;//滤波器开最大
	TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;//上升沿触发
	TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;//不分频
	TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;//直接模式
	TIM_ICInit(TIM3, &TIM_ICInitStructure);
	
	TIM_SelectInputTrigger(TIM3, TIM_TS_TI1FP1);//选择触发源
	TIM_SelectSlaveMode(TIM3, TIM_SlaveMode_Reset);//从模式
	
	TIM_Cmd(TIM3, ENABLE);//开启定时器
}

uint32_t IC_GetFreq(void)
{
	return 1000000 / (TIM_GetCapture1(TIM3) + 1);
}
 
```

### PWMI基本结构：

![image-20221226204252689](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045950.png)

PWMI模式，使用了两个通道捕获一个引脚可以同时测量周期和占空比，TI1FP1配置上升沿触发，触发捕获和清零CNT，TI1FP2，配置为下降沿触发，通过交叉通道，去触发通道2的捕获单元，去触发通道2的捕获单元

例如：左上角图，最开始上升沿，CCR1捕获，同时清零CNT，之后CNT一直++，在下降沿这个时刻，触发CCR2捕获，这时CCR的值就是高电平期间的计数值，CCR2捕获不会触发CNT清零，CNT++，直到下一次上升沿，CCR1捕获周期，CNT清零，这样执行，CCR1就一整个周期的计数值，CCR2就是高电平期间的计数值，用CCR2/CCR1，就是占空比。

##### 单独写入PSC的函数

```c
void TIM_PrescalerConfig(TIM_TypeDef* TIMx, uint16_t Prescaler, uint16_t TIM_PSCReloadMode);
```

##### 输入捕获步骤

第一步，RCC开启时钟，把GPIO的TIM的时钟打开

第二步，GPIO初始化，把GPIO配置成输入模式，一般选择上拉输入或者浮空输入模式

第三步，配置时基单元，让CNT计数器在内部时钟的驱动下自增运行

第四步，配置输入捕获单元，包括滤波器、极性、直连通道还是交叉通道、分频器这些参数

第五步，选择从模式的触发源，触发源选择TI1FP1，调用一个库函数即可

第六步，选择触发之后执行的操作，执行Reset操作，调用一个库函数即可

第七步，调用TIM_Cmd函数，开启定时器

##### 输入捕获常用函数

##### 结构体配置输入捕获单元的函数

输出比较每个通道占用一个函数，输入捕获4个通道是共用一个函数的,在结构体中有额外的参数来选择通道

```c
void TIM_ICInit(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ICInitStruct);
```

##### 另一个输入捕获的初始化函数

与上一个函数类似都是用于初始化输入捕获单元的，上一个函数只是单一的配置一个通道，而这个函数可以快速配置两个通道，把外设电路配置成PWMI的电路

```c
void TIM_PWMIConfig(TIM_TypeDef* TIMx, TIM_ICInitTypeDef* TIM_ICInitStruct);
```

##### 给输入捕获结构体赋一个初始值函数

```c
void TIM_ICStructInit(TIM_ICInitTypeDef* TIM_ICInitStruct);
```

##### 选择输入触发源TRGI函数

调用此函数可以选择从模式的触发源

```c
void TIM_SelectInputTrigger(TIM_TypeDef* TIMx, uint16_t TIM_InputTriggerSource);
```

##### 选择输出触发源TRGO函数



```c
void TIM_SelectOutputTrigger(TIM_TypeDef* TIMx, uint16_t TIM_TRGOSource);
```

##### 选择从模式函数

```c
void TIM_SelectSlaveMode(TIM_TypeDef* TIMx, uint16_t TIM_SlaveMode);
```

##### 单独配置通道1、2、3、4的分频器函数

在参数结构体里也可以配置

```c
void TIM_SetIC1Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
void TIM_SetIC2Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
void TIM_SetIC3Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
void TIM_SetIC4Prescaler(TIM_TypeDef* TIMx, uint16_t TIM_ICPSC);
```

##### 读取四个通道的CCR函数

输出比较模式下，CCR是只写的，要用SetCompare写入，输入捕获模式下，CCR是只读的，要用GetCapture读出

```c
uint16_t TIM_GetCapture1(TIM_TypeDef* TIMx);
uint16_t TIM_GetCapture2(TIM_TypeDef* TIMx);
uint16_t TIM_GetCapture3(TIM_TypeDef* TIMx);
uint16_t TIM_GetCapture4(TIM_TypeDef* TIMx);
```

PWMI模式程序示例

```c
void IC_Init(void)
{
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	TIM_InternalClockConfig(TIM3);
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStructure.TIM_Period = 65536 - 1;		//ARR
	TIM_TimeBaseInitStructure.TIM_Prescaler = 72 - 1;		//PSC
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStructure);
	
	TIM_ICInitTypeDef TIM_ICInitStructure;
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;
	TIM_ICInitStructure.TIM_ICFilter = 0xF;
	TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;
	TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
	TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;//直接模式
	TIM_PWMIConfig(TIM3, &TIM_ICInitStructure);//另一个通道选择相反的配置

	TIM_SelectInputTrigger(TIM3, TIM_TS_TI1FP1);//选择触发源
	TIM_SelectSlaveMode(TIM3, TIM_SlaveMode_Reset);//选择从模式
	
	TIM_Cmd(TIM3, ENABLE);
}

uint32_t IC_GetFreq(void)
{
	return 1000000 / (TIM_GetCapture1(TIM3) + 1);
}

uint32_t IC_GetDuty(void)
{
	return (TIM_GetCapture2(TIM3) + 1) * 100 / (TIM_GetCapture1(TIM3) + 1);
}

```



### 编码器接口

Encoder Interface编码器接口

编码器接口可接收增量（正交）编码器的信号，根据编码器旋转产生的正交信号脉冲，自动控制CNT自增或自减，从而指示编码器的位置、旋转方向和旋转速度

每个高级定时器和通用定时器都拥有1个编码器接口

两个输入引脚借用了输入捕获的通道1和通道2

对于需要频繁执行，操作简单的任务，一般会设计一个硬件模块来自动完成

把两个编码器的A相和B相，接入STM32，定时器的编码器接口，编码器接口自动控制时基单元中的CNT计数器，进行自增或者自减，例如CNT初始值为0，编码器右转CNT++，右转产生一个脉冲，CNT++,左转CNT--，编码器接口（相当于带有方向控制的外部时钟）同时控制CNT的计数时钟和计数方向，CNT的值就表示了编码器的位置，每隔一段时间取一次CNT的值再把CNT清零，每次取出来的值就带标 了编码器的速度，编码器的测速实际上就是测频法测正交脉冲的频率，CNT计次，每隔一段时间取一次计次，也可以用外部中断来接编码器（用软件资源来弥补硬件资源）

![image-20221226224551844](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045516.png)

当编码器的旋转轴转起来时，A相和B相就会输出方波信号，转的越快，方波的频率越高，方波的频率就代表了速度，取出任意一相的信号来测量频率，就能知道旋转速度，只有一相的信号无确定旋转方向。

正交信号：当正转时，A相超前B相90度，反转时，A相滞后B相90度。

正转时，第一个时刻，A相上升沿，对应B此时是低电平，第二个时刻，B相上升沿，对应A相高电平，第三个时刻，A相下降沿，对应B相高电平，B相下降沿，对应A相低电平。

反转时，第一个时刻，B相上升沿，对应A相低电平，第二个时刻A相上升沿，对应B相高电平，第三个时刻，B相下降沿，对应A相高电平，第四个时刻，A相下降沿，对应B相低电平。

当A、B相出现这些边沿时，对应另一相的状态，正转和反转正好是相反的

编码器接口的设计逻辑是：首先把A相和B相的所有边沿作为计数器的计数时钟，出现边沿信号时，就计数自增或者自减，

![image-20221226225908098](https://gitee.com/pu-heliang/photos/raw/master/img/202301032045627.png)

编码器接口的两个引脚，借用了输入捕获单元的前两个通道，编码器的输入引脚就是定时器的CH1和CH2两个引脚，信号的通路就是，CH1通过这里，通向编码器接口，CH3和CH4和编码器接口无关，其中CH1和CH2的输入捕获滤波器和边沿检测，编码器接口也有使用，但是后面的是否交叉，预分频器和CCR寄存器，与编码器接口无关，这就是编码器接口的输入部分，编码器接口的输出部分，相当于从模式控制器，控制CNT的计数时钟和计数方向，输出过程就是如果产生边沿信号，并且对应另一相的状态为正转，则控制CNT自增否则控制CNT自减，此时计数时钟和计数方向都处于编码器接口托管的状态，计数器的自增和自减，受编码器的控制。

编码器接口的基本结构：

![image-20221226230733071](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046598.png)

输入捕获的前两个通道，通过GPIO口接入编码器的A、B相然后通过滤波器和边沿检测极性选择，产生TI1TP1和TI2FP2，通向编码器接口，编码器接口通过控制预分频器控制CNT计数器的时钟，同时，编码器接口还根据编码器的旋转方向，控制CNT的计数方向，编码器正转时，CNT自增，编码器反转时，CNT自减，一般设置ARR为65535，最大量程

工作模式：

![image-20221226231231784](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046208.png)

编码器接口的工作逻辑：TI1FP1和TI2FP2接的就是编码器的A、B相，在A相和B相的上升沿或者下降沿触发计数，向上计数还是向下计数取决于边沿信号发生时，另一相的电平状态（相对信号的电平）

配置流程：

第一步，RCC开启时钟，开启GPIO和定时器的时钟

第二步，配置GPIO，配置为输入模式

第三步，配置时基单元，预分频器选择不分频，自动重装，一般给最大65535

第四步，配置输入捕获单元，只需要配置滤波器和极性两个参数

第五步，配置编码器接口模式，调用一个库函数即可

第六步，调用TIM_cmd启动定时器

如果需要测量编码器的速度：每隔一段固定的闸门时间，取出一次CNT，然后把CNT清零

##### 定时器编码器接口配置函数

```c
void TIM_EncoderInterfaceConfig(TIM_TypeDef* TIMx, uint16_t TIM_EncoderMode,
                                uint16_t TIM_IC1Polarity, uint16_t TIM_IC2Polarity);
```

配置上拉输入还是下拉输入：看外部模块输出的默认电平，与外部模块输出的默认电平相同，防止默认电平打架，如果不确定外部模块输出的默认状态，或者外部信号输出功率非常小，尽量选择浮空输入

#### 编码器接口程序示例

```c
void Encoder_Init(void)
{
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
		
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
	TIM_TimeBaseInitStructure.TIM_Period = 65536 - 1;		//ARR
	TIM_TimeBaseInitStructure.TIM_Prescaler = 1 - 1;		//PSC
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStructure);
	
	TIM_ICInitTypeDef TIM_ICInitStructure;
	TIM_ICStructInit(&TIM_ICInitStructure);
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;
	TIM_ICInitStructure.TIM_ICFilter = 0xF;
	TIM_ICInit(TIM3, &TIM_ICInitStructure);
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_2;
	TIM_ICInitStructure.TIM_ICFilter = 0xF;
	TIM_ICInit(TIM3, &TIM_ICInitStructure);
	
	TIM_EncoderInterfaceConfig(TIM3, TIM_EncoderMode_TI12, TIM_ICPolarity_Rising, TIM_ICPolarity_Rising);//编码器电机模式
	
	TIM_Cmd(TIM3, ENABLE);//开启定时器
}

int16_t Encoder_Get(void)
{
	int16_t Temp;
	Temp = TIM_GetCounter(TIM3);//获取CNT的值
	TIM_SetCounter(TIM3, 0);//设置CNT的值
	return Temp;
}

```

### ADC模拟数字转换器

ADC（Analog-Digital Converter）模拟-数字转换器

ADC可以将引脚上连续变换的模拟量转换成内存中储存的数字变量，建立模拟电路到数字电路的桥梁，ADC读取引脚上的模拟电压，转换为一个数据，存在寄存器里，再把这个数据读取到变量里来，就可以进行显示、判断、记录等操作

12位（分辨率，位数越高，量化结果就越精细，对应分辨率就越高）逐次逼近型ADC，1us转换时间（转换频率），

输入电压范围：0-3.3V，转换结果范围：0~4095，ADC的输入电压要求在芯片的负极和正极之间变化，最低电压是负极0V，最高电压是正极3.3V，经过ADC转换之后最小值是0，最大值是4095，0V对应0,3.3V对应4095，中间都是一一对应的线性关系。

18个输入通道，可测量16个外部和2个内部信号源，外部信号源就是16个GPIO口，在引脚上直接模拟信号就行了，不需要任何的额外电路引脚就能直接测电压，2个内部信号源是内部温度传感器和内部参考电压，温度传感器可以测量CPU的温度，内部参考电压是一个1.2V左右的基准电压，这个基准电压不随外部供电电压变化而变化，如果芯片的供电不是标准的3.3V测量外部引脚的电压就可能不对，这时可以读取基准电压进行校准，这样就可以得到正确的电压值了。

规则组和注入组两个转换单元，这个是STM32 ADC的增强功能，普通AD转换流程是，启动一次转换，读一次值，然后再启动，在读值，这样的流程，但是STM32的ADC可以列一个组，连续转换多个值，一次性启动一个组，连续转换多个值，并且有两个组，一个是用于常规使用的规则组，一个是用于突发事件的注入组。

模拟看门狗自动检测输入电压范围，此ADC一般可以用于测量光线强度、温度，经常会要求光线高于某个阈值、低于某个阈值，或者温度高于某个阈值，低于某个阈值时，执行一些操作，高于某个阈值，低于某个阈值的判断，就可以用模拟看门狗来自动执行，模拟看门狗可以检测指定的某些通道，当AD值高于它设定的上阈值或者下阈值时，就会申请中断，就可以在中断函数中执行相应的操作，这样就不用手动读值，再用if判断了

STM32F103C8T6 ADC资源：ADC1、ADC2,10个外部输入通道，最多只能测量10个外部引脚的模拟信号

##### 逐次逼近型ADC的内部结构

![image-20221228125552084](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046412.png)

这个图是ADC0809的内部结构图，它是一个独立的8位逐次 逼近型ADC芯片，左边IN0~到IN7，是8路输入通道，通过通道选择开关，选中一路，输入到比较器上方进行转换，下面部分是地址锁存和译码，就是想选中哪一路，就把通道号放在这三个引脚上，然后给一个锁存信号，上面对应的通路开关就自动拨好了，相当于可以通过模拟信号的数据选择器，因为ADC转换是一个非常快的过程，给个开始信号，过个几个us就转换完成了想转换多路信号，那不必设计多个AD转换器，只需要一个AD转换器，然后加一个多路选择开关，想转换哪一路，选中对应通道，然后再开始转换就行了，这就是输入通道选择的部分，这个ADC0809只有8个输入通道，STM32内部的ADC是有18个输入通道，对应的是18路输入的多路开关，输入信号选好后，到电压比较器，它可以判断两个输入信号电压的大小关系，输出一个高低电平指示谁大谁小，它的两个输入端，一个是待测的电压，另一个是DAC的电压输出端，DAC是数模转换器，给一个数据，就可以输出数据对应的电压，DAC内部是适应加权电阻网络来实现的转换，将外部输入的未知的电压和一个已知输出的电压，两个同时输入到电压比较器，进行大小判断，如果DAC输出的电压比较大，就调小DAC数据，如果DAC输出的电压比较小，就增大DAC数据，直到DAC输出的电压和外部通道输入的电压近似相等，这样DAC输入的数据就是外部电压的编码数据了，这就是DAC的实现原理，电压调节的过程是逐次逼近SAR来完成的，为了最快找到未知电压的编码，通常会使用二分法进行寻找，EOC（End Of Convert）是转换结束信号，START是开始转换，给一个输入脉冲，开始转换，CLOCK是ADC时钟，因为ADC内部是一步一步进行判断的，所以需要时钟来推动这个过程，下面VREF+和VREF-是DAC的参考电压，例如给一个数据255，是对应5V还是3.3V就由参考电压决定，这个DAC的参考电压也决定了，ADC的输入范围，所以他也是ADC参考电压，左边是整个芯片电路的供电，VCC和GND，通常参考电压的VCC是一样的，会接在一起，参考电压的负极和GND也是一样的，也接到一起，一般情况下ADC输入电压的范围就和ADC的供电是一样的。

##### STM32的ADC：

![image-20221228131906698](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046864.png)

左边是ADC的输入通道、包括16个GPIO口，IN0~IN15，和两个内部的通道，一个是内部温度传感器，另一个是VREFINT（V Reference Internal），内部参考电压，总共是18个输入通道，然后到达模拟多路开关，可以指定想要的通道，右边是多路开关的输出，进入到模数转换器，转化结果会放在数据寄存器中，读取寄存器就能知道ADC转换的结果了，对于普通的ADC，多路开关一般都是只选中一个的，就是选中某个通道、开始转换、等待转换完成、取出结果，这是普通的流程，但是STM32就可以同时选中多个，在转换的时候，还分成了两个组，规则通道组和注入通道组，规则组可以一次最多选中16个通道，注入组最多可以选中4个通道，就像是去餐厅点菜，普通的ADC是，你指定一个菜，老板给你做，然后做好了送给你，而这里是，你指定一个菜单，这个菜单最多可以填16个菜，然后直接递个菜单给老板，老板就按照菜单的顺序依次做好，一次性给你端上来，这样的话就可以大大提高效率，当然菜单也可以只写一个菜，这样这个菜单就简化成普通模式了，对于这个菜单也有两种，一种是规则组菜单，可以同时上16个菜，但是规则组只有一个数据寄存器，就是桌子比较小，最多只能放一个菜，如果上16个菜，前15个菜都会被挤掉，只能的到第16个菜，所以对于规则组转换来说，如果使用这个菜单的话，最好配合DMA来实现，DMA是一个数据转运小帮手，它可以在每上一个菜之后，把这个菜挪到其他地方去，防止被覆盖，规则组虽然可以同时转换16个通道，但是数据寄存器只能存一个结果，如果不想之前的结果被覆盖，那在转换完成之后，就要尽快把结果拿走，注入组，相当于是餐厅的VIP座位，在这个座位上一次最多可以点4个菜，并且数据寄存器有4个可以同时上4个菜，对于注入组而言，就不用担心数据覆盖的问题了，这就是规则组和注入组的介绍，一般情况下，使用规则组就足够了，如果要使用规则组的菜单，那就配合DMA转运数据，这样就不用担心数据覆盖的问题了。

对于规则组，左下角是触发的部分，对于STM32的ADC触发开始转换的信号有两种，一种是软件触发，就是在程序中手动调用一条代码，就可以启动转换了，另一种是硬件触发，就是触发源，触发源主要是来自定时器，有定时器的各个通道，还有TRGO定时器主模式的输出，（定时器可以通向ADC、DAC这些外设，用于触发转换），ADC经常需要过一个固定时间段转换一次，比如每隔1ms转换一次，正常的思路就是，用定时器，每隔1ms申请一次中断，在中断里手动开启一次中断，这样也是可以的，但是频繁进中断对程序是有一定影响的，如果有很多中断都需要频繁进入，那将会影响主程序的执行，并且不同中断之间，由于优先级的不同，也会导致某些中断不能及时的到响应，如果触发ADC的中断不能及时响应，那ADC的转换频率就会产生影响，所以对于需要频繁进中断，并且只在中断里只完成了简单的工作的情况，一般都会有硬件的支持，可以给TIM3定一个1ms的时间，把TIM3的更新事件选择为TRGO输出，然后再ADC这里，选择触发信号TIM3的的TRGO，这样TIM3的更新事件就能通过硬件自动触发ADC转换了，整个过程不需要进中断，节省了中断资源，这就是定时器触发的作用，也可以选择外部中断引脚来触发中断，都可以在程序中配置，左上角是VREF+、VREF-、VDDA和VSSA，VREF+、VREF-这两个是ADC的参考电压，决定了ADC输入电压的范围，VDDA和VSSA是ADC的供电引脚，一般情况下VREF+要接VDDA，VREF-要接VSSA，STM32没有VREF+、VREF-的引脚内内部已经和VDDA和VSSA接在一起了。VDDA和VSSA是内部模拟部分的电源，例如ADC、RC震荡器、锁相环等，在STM32中VDDA接3.3V，VSSA接GND，所以输入电压的范围就是0~3.3V，右边的ADCCLK是ADC的时钟，也就是ADC0809中的CLOCK，是用于驱动内部逐次比较的时钟来自ADC预分频器，ADC预分频器来源于RCC，APB2时钟72MHz，然后通过ADC进行分频，得到ADCCLK，ADCCLK最大是14MHz，对于ADC预分频器，只能选择6分频，结果是12MHz和8分频结果是9MHz，上面的是DMA请求，用于触发DMA进行数据转运，再上面是两个数据寄存器，用于存放转换结果，在上面是模拟看门狗，它们可以存一个阈值高限和阈值低限，如果启动了模拟开门狗，并且指定了看门的通道，那么看门狗就会关注它看门的通道，一但超过这个阈值范围，就会乱叫，就会在上面申请一个模拟看门狗的中断，最后通向NVIC，对于规则组和注入组，它们转换完成后，也会有一个EOC转换完成的信号，EOC是规则组完成的信号，JEOC是注入组完成的信号，这两个信号会在状态寄存器置一个标志位，读取这个标志位，就能知道是不是转换结束了，同时这两个标志位也可以去到NVIC，申请中断，如果开启了NVIC对应的通道，它们就会触发中断。

##### ADC基本结构

![image-20221228141007759](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046518.png)

左边是输入通道，16个GPIO口，外加两个内部的通道，然后进入AD转换器，AD转换器里有两个组，一个是规则组，一个是注入组，规则组最多可以选择16个通道，注入组最多可以选择4个通道，转换的结果有放在AD数据寄存器中，其中规则组只有1个数据寄存器，注入组有4个数据寄存器，下面是触发控制，提供开始转换的的START信号，触发控制可以选择软件触发和硬件触发，硬件触发主要是来自于定时器，当然也可以选择外部中断的引脚，右边是来自RCC的ADC时钟CLOCK，ADC逐次比较的过程就是由此时钟推动，上面可以布置一个模拟看门狗用于检测转换的结果的范围，如果超出设定的阈值，就通过中断输出控制，向NVIC申请中断，规则组和注入组在转换完成后会有个EOC信号，会置一个标志位，也可以通向NVIC，右下角是开关控制，在库函数中，就是ADC_Cmd函数，用于ADC上电的。

##### 输入通道：

![image-20221228141929856](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046469.png)

双ADC模式：就是ADC1和ADC2一起工作，可以配合组成同步模式，交叉模式等等模式，交叉模式就是ADC1和ADC2交叉的对一个通道进行采样，这样可以提高采样率。

##### 规则组的四种转换模式

##### 单次转换、非扫描模式

![](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046631.png)

在非扫描模式下，这个菜单只有第一个序列1的位置有效，这时菜单同时选择一组的方式就退化成简单的选中一个的方式了，我们可以在序列1的位置指定我们想转换的通道，比如通道2，然后就可以触发转换，ADC就会对这个通道2进行模数转换，过一小段时间后，转换完成，转换结果放在数据寄存器里，同时给EOC标志位置1，整个转换过程就结束了。判断这个标志位，如果转换完了，就可以在数据寄存器中读取结果了。如果想再启动一次转换，那就需要再触发一次。转换结束，置EOC标志位，读结果。如果想换一个通道转换，那在转换之前，把第一个位置通道2改成其他通道，然后再启动转换。

##### 连续转换、非扫描模式

![image-20221228143332282](https://gitee.com/pu-heliang/photos/raw/master/img/202301032046752.png)

非扫描模式，所以菜单列表就只用第一个，与上次单次转换不同的是，它在一次转换结束后不会停止，而是立刻开始下一轮转换，然后一直持续下去，这样就只需要触发一次，之后就可以一直转换了。这个模式的好处就是，开始转换之后不需要等待一段时间，它一直都在转换，不需要手动开启转换了。也不用判断是否结束，想要读AD值的时候，就直接从数据寄存器取就行。

##### 单次转换、扫描模式

![image-20221228172651369](https://gitee.com/pu-heliang/photos/raw/master/img/202301032048367.png)

这个模式也是单次转换，所以每触发一次，转换结束后，就会停下来，下次转换就得再触发才能开始，他是扫描模式，这就会用到这个菜单列表了，可以在菜单里点菜，比如第一个菜是通道2，第二个菜是通道5，等等，这里每个位置是通道几可以任意指定，并且也是可以重复的，初始化结构体里还有个参数，就是通道数目，因为这16个位置可以不用完，只用前几个，那就需要再给个通道数目的参数，告诉他，我有几个通道，这里指定通道7，那它就只看前7个位置，然后每次触发之后，它就依次对前7个位置进行AD转换，转换结果都放在数据寄存器中，为了防止数据被覆盖，就需要用DMA及时将数据挪走，7个通道转换完成后，产生EOC信号，转换结束，然后再触发下一次，就又开始新一轮的转换，这就是单次转换，扫描模式的工作流程。

##### 连续转换、扫描模式

![image-20221228173637254](https://gitee.com/pu-heliang/photos/raw/master/img/202301032048094.png)

在上一次模式的基础上，可以在一次转换完成后，立刻开始下一次的转换。在扫描模式的情况下，还可以右边一种模式，叫间断模式，它的作用是，在扫描的过程中，每隔几个转换，就暂停一次，需要再次触发，才能继续

##### 触发控制：

![image-20221228173918884](https://gitee.com/pu-heliang/photos/raw/master/img/202301032048244.png)

这个表就是规则组的触发源，在这个表里有来自定时器的信号，还有来自引脚或者定时器的信号，具体是引脚还是定时器，需要AFIO重映射来确定，最后是软件控制位，也就是软件触发，这些触发信号可以配置右边的寄存器来完成，库函数直接给一个参数就行。

##### 数据对齐

![image-20221228174146250](https://gitee.com/pu-heliang/photos/raw/master/img/202301032048436.png)

这个ADC是12位的，它的转换结果就是一个12位的数据，但是这个数据寄存器是16位的，所以就存在一个数据对齐的问题，分为两种对齐方式，数据右对齐，和数据左对齐，数据右对齐就是12位的数据向右靠，高位多出来的几位就补0，第二种是数据左对齐，是12位的数据向左靠，低位多出来的几位就补0，我们一般使用的都是数据右对齐，这样读取这个16位寄存器，直接就是转换结果，如果选择左对齐，直接读的话，得到的数据会比实际的大，因为数据左对齐实际上就是把数据左移了4次，二进制的数据，数据左移一次，就等效于把这个数据乘2，左移4次，就相等于把结果乘16了，所以直接读会比实际值大16倍，左对齐的作用就是，不需要高分辨率时，就可以选择左对齐再把后面的低4位去掉，这个12位的ADC就退化成8位的ADC了。

##### 转换时间

AD转换的步骤：采样，保持，量化，编码

STM32 ADC的总转换时间为：Tconv = 采样时间（采样保持花费的时间，采样时间越大，越能避免一些毛刺信号的干扰）+12.5个ADC周期（量化编码花费的时间）

例如：当ADCCLK = 14MHz，采样时间为1.5个ADC周期，Tconv = 1.5 + 12.5 = 14个ADC周期 = 1us

##### 校准

ADC有一个内置自校准模式。校准可大幅减小因内部电容器组的变化而造成的准精度误差。校准期间，在每个电容器上都会计算出一个误差修正值（数字值），这个码用于消除在随后的转换中每个电容器上产生的误差

建议在每次上电后执行一次校准

启动校准前，ADC必须处于关电状态超过至少两个ADC时钟周期。

只需要在ADC初始化最后加几条代码即可

##### 硬件电路

![image-20221228175818636](https://gitee.com/pu-heliang/photos/raw/master/img/202301032048081.png)

电位器的两个固定端，一端接3.3V，另一端接GND，这样中间的滑动端就可以输出一个0~3.3V可调的电压输出了，这里可以接ADC的输入通道例如PA0口，当滑动端往上滑时，电压增大，往下滑时，电压减小，电阻的阻值不能给太小，因为它是直接接在电源正负极上的，阻值太小，这个电阻就会很费电，再小可能就发热冒烟了，一般要接K欧级的电阻

中间是传感器输出电压的电路，一般来说，光敏电阻、热敏电阻、红外接收管、麦克风都可以等效为一个可变电阻，电阻阻值没法直接测量，可以通过和一个固定电阻串联分压，来得到一个反应电阻值电压的电路，传感器阻值变小，下拉作用变强，输出端电压就下降，传感器阻值变大时，下拉作用变弱，输出端受上拉电阻的作用，电压就会升高。固定电阻一般选择和传感器电阻相近的电容，这样可以得到一个位于中间电压区域比较好的输出

右边的电路是一个简单的电压转换电路，如果我想测量一个0~5V的VIN电压但是ADC只能接收0~3.3V的电压，那就可以搭建一个简易转换电路，使用电阻进行分压，上面阻值17K，下面组织33K，加一起是50K，中间的电压就是3.3V，就可以进入ADC转换了，这就是简单的电压转换电路

##### ADC初始化步骤

![image-20221228184655915](https://gitee.com/pu-heliang/photos/raw/master/img/202301032048997.png)

第一步，开启RCC时钟，包括ADC和GPIO的时钟，ADCCLK的分频器，也需要配置一下

第二步，配置GPIO，把需要用到的GPIO口配置成模拟输入的模式

第三步，配置多路开关，把左边的通道接入到右边的规则组列表中

第四步，配置ADC转换器，在库函数里，用结构体来配置，配置这一大块电路的参数

第五步，调用ADC_Cmd开启ADC，也可以进行一下校准，减小误差

想要软件触发转换，会有函数可以触发，如果想读取结果也会有函数可以读取结果

##### ADCCLK的配置函数

可以对APB2的72MHz时钟选择2、4、6、8分频，输入到ADCCLK

```c
void RCC_ADCCLKConfig(uint32_t RCC_PCLK2)
```

##### 恢复缺省配置函数

```c
void ADC_DeInit(ADC_TypeDef* ADCx);
```

##### Init初始化函数

```c
void ADC_Init(ADC_TypeDef* ADCx, ADC_InitTypeDef* ADC_InitStruct);
```

##### StructInit结构体初始化函数

```c
void ADC_StructInit(ADC_InitTypeDef* ADC_InitStruct);
```

##### 给ADC上电的函数

```c
void ADC_Cmd(ADC_TypeDef* ADCx, FunctionalState NewState);
```

##### 开启DMA输出信号函数

使用DMA转运数据，就得调用这个函数

```c
void ADC_DMACmd(ADC_TypeDef* ADCx, FunctionalState NewState);
```

##### 中断输出控制函数

```c
void ADC_ITConfig(ADC_TypeDef* ADCx, uint16_t ADC_IT, FunctionalState NewState);
```

##### 复位校准函数

```c
void ADC_ResetCalibration(ADC_TypeDef* ADCx);
```

##### 获取复位校准状态函数

```c
FlagStatus ADC_GetResetCalibrationStatus(ADC_TypeDef* ADCx);
```

##### 开始校准函数

```c
void ADC_StartCalibration(ADC_TypeDef* ADCx);
```

##### 获取开始校准状态函数

```c
FlagStatus ADC_GetCalibrationStatus(ADC_TypeDef* ADCx);
```

##### 软件触发ADC的函数

```c
void ADC_SoftwareStartConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);
```

##### ADC获取软件开始转换状态函数（没啥用）

获取CR2的SWSTART这一位，给SWTART置1，以开始转换，这个函数是返回SWSTART的状态，由于SWSTART位在转换开始后立刻就清0了，所以这个函数的返回值跟转换是否结束，毫无关系

```c
FlagStatus ADC_GetSoftwareStartConvStatus(ADC_TypeDef* ADCx);
```

##### 获取转换是否结束函数

获取标志位状态，参数给EOC的标志位，判断EOC标志位是不是置1了，如果转换结束EOC标志位置1，然后调用此函数，判断标志位，来判断转换是否结束

```c
FlagStatus ADC_GetFlagStatus(ADC_TypeDef* ADCx, uint8_t ADC_FLAG);
```

##### 配置间断模式函数

```c
void ADC_DiscModeChannelCountConfig(ADC_TypeDef* ADCx, uint8_t Number);//每隔，几个通道间断一次
void ADC_DiscModeCmd(ADC_TypeDef* ADCx, FunctionalState NewState);//启用间断模式
```

##### ADC规则组通道配置函数

它的作用是给序列每个位置填写的指定的通道，就是填写点菜点菜的过程

```c
void ADC_RegularChannelConfig(ADC_TypeDef* ADCx, uint8_t ADC_Channel, uint8_t Rank, uint8_t ADC_SampleTime);
```

##### 外部触发转换控制函数

##### 是否允许外部触发转换

```c
void ADC_ExternalTrigConvCmd(ADC_TypeDef* ADCx, FunctionalState NewState);
```

##### ADC获取转换值函数

获取AD转换的数据寄存器，读取转换结构就需要使用这个函数

```c
uint16_t ADC_GetConversionValue(ADC_TypeDef* ADCx);
```

##### ADC获取双模式转换值

读取双ADC模式转换结果的函数

```c
uint32_t ADC_GetDualModeConversionValue(void);
```

##### 注入组相关函数

```c
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

##### 配置模拟看门狗相关函数

```c
void ADC_AnalogWatchdogCmd(ADC_TypeDef* ADCx, uint32_t ADC_AnalogWatchdog);//是否启动模拟看门狗
void ADC_AnalogWatchdogThresholdsConfig(ADC_TypeDef* ADCx, uint16_t HighThreshold, uint16_t LowThreshold);//配置高低阈值
void ADC_AnalogWatchdogSingleChannelConfig(ADC_TypeDef* ADCx, uint8_t ADC_Channel);//配置看门的通道
```

##### 用来控制开启内部的两个通道函数

用来控制开启内部的两个通道（ADC温度传感器，内部参考电压控制）,如果要用到这两个通道需要调用这个函数

```c
void ADC_TempSensorVrefintCmd(FunctionalState NewState);
```

##### 获取标志位状态函数

```c
FlagStatus ADC_GetFlagStatus(ADC_TypeDef* ADCx, uint8_t ADC_FLAG);
```

##### 清除标志位函数

```c
void ADC_ClearFlag(ADC_TypeDef* ADCx, uint8_t ADC_FLAG);
```

##### 获取中断状态函数

```c
ITStatus ADC_GetITStatus(ADC_TypeDef* ADCx, uint16_t ADC_IT);
```

##### 清除中断挂起位

```c
void ADC_ClearITPendingBit(ADC_TypeDef* ADCx, uint16_t ADC_IT);
```

ADC程序实例

```c
void AD_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
		
	ADC_InitTypeDef ADC_InitStructure;
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;//独立模式
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;//右对齐
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;//软件触发
	ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;//连续模式
	ADC_InitStructure.ADC_ScanConvMode = DISABLE;//扫描模式
	ADC_InitStructure.ADC_NbrOfChannel = 1;//通道数
	ADC_Init(ADC1, &ADC_InitStructure);
	
	ADC_Cmd(ADC1, ENABLE);//开启ADC
	
	ADC_ResetCalibration(ADC1);//开始复位校准
	while (ADC_GetResetCalibrationStatus(ADC1) == SET);//获取复位校准状态
	ADC_StartCalibration(ADC1);//开始校准
	while (ADC_GetCalibrationStatus(ADC1) == SET);//获取校准状态
}

uint16_t AD_GetValue(uint8_t ADC_Channel)
{
	ADC_RegularChannelConfig(ADC1, ADC_Channel, 1, ADC_SampleTime_55Cycles5);//设置规则组的函数
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);//软件触发
	while (ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC) == RESET);//获取校准完成标志位
	return ADC_GetConversionValue(ADC1);//返回转换完的AD值
}

```



### DMA

-  DMA（Direct Memory Access）直接存储器存取，主要是用来协助CPU，完成数据转运的工作
- DMA可以提供外设（外设寄存器，一般是外设的数据寄存器DR，Data Register，比如ADC的数据寄存器，串口的数据寄存器）和存储器（运行内存（SRAM）和程序存储器（Flash）是存储变量数组和程序代码的地方）或者存储器与存储器之前的高速数据传输，无须CPU干预，节省了CPU的资源

- 12个独立可配置的通道（数据转运的路径）：DMA1（7个通道），DMA2（5个通道）
- 每个通道都支持软件触发和特定的硬件触发，存储器到存储器的数据转运，一般用软件触发，外设到存储器的转运一般用硬件触发
- STM32F103C8T6 DMA资源：DMA1（7个通道）

##### 存储器映像

![image-20221228215059578](https://gitee.com/pu-heliang/photos/raw/master/img/202301032048812.png)

计算机系统的5大组成部分：运算器、控制器、存储器、输入设备和输出设备，运算器和控制器一般合在一起叫做CPU，计算机的核心关键部分就是CPU和存储器，存储器最重要的是存储器的内容和存储器的地址。

存储器分为两大类：ROM和RAM，ROM就是只读存储器，是一种非易失性、掉电不丢失的存储器，RAM就是随机存储器，是一种易失性、掉电丢失的存储器，ROM分为三块，第一块是程序存储器，Flash，也就是主闪存，用途就是存储C语言编译后的程序代码，也就是下载程序的位置，运行程序一般也是从主闪存中开始运行的，系统存储器和选项字节，这两块存储器也是ROM的一种，掉电不丢失，实际上它们的存储介质也是Flash，非主闪存Flash，系统存储器是用来存储BootLoader，BootLoader程序一般是芯片出厂自动写入的，一般不允许修改，选项字节存的主要是Flash的读保护、写保护，还有看门狗等等的配置，运行内存SRAM存储我们程序中定义变量、数组、结构体的地方，外设寄存器存储的是我们初始化各个外设，最终读写的东西，外设寄存器起始也是SRAM，存储内核外设NVIC和SysTick。

##### DMA的框图

![image-20221228220437956](https://gitee.com/pu-heliang/photos/raw/master/img/202301032049778.png)

左上角是Cortex-M3内核，里面包含了CPU和内核外设，剩下的所有东西都可以看成是存储器，Flash是主闪存，SRAM是运行内存，各个外设都可以看成是寄存器，也是一种SRAM存储器，寄存器是一种特殊的存储器，一方面，CPU可以对寄存器进行读写，就像读写运行内存一样，另一方面，寄存器的每一位背后，都连接了一根导线，这些导线可以控制外设电路的状态，比如置引脚的高低电平，导通和断开、切换数据寄存器，或者多位结合起来，当做计数器、数据寄存器，寄存器是连接软件和硬件的桥梁，软件读写寄存器，就相当于在控制硬件的执行，使用DMA进行数据转运，就相当于从某个地址取内容，再放到另一个地址去。

为了高效有条理地访问存储器，设计了一个总线矩阵，总线矩阵的左端，是主动单元，也就是拥有存储器的访问权，右边这些，就是被动单元，它们的存储器只能被左边的主动单元读写，主动单元内核有DCode和系统总线，可以访问右边的存储器，其中DCode总线是专门访问Flash的，系统总线是访问其他东西的，由于DMA要转运数据，所以DMA也必须要有访问的主动权，主动单元除了内核、CPU剩下的就是DMA总线了，DMA1和DMA2都各自有一条总线，下面以太网外设自己也私有一个DMA总线，DMA1有7个通道，DMA2有5个通道，各个通道可以分别设置它们转运数据的源地址和目的地址，下面的仲裁器，由于DMA只有一条总线，仲裁器可以根据通道的优先级决定哪个通道谁先用，在总线矩阵里，也有一个仲裁器，如果DMA和CPU都要访问同一个目标，那么DMA就会暂停CPU的访问，以防止冲突，不过总线仲裁器，仍然会保证CPU得到一半的总线带宽，使CPU也能正常工作，下面的AHB从设备，也就是DMA自身的寄存器，DMA作为一个外设，也会有自己相应的配置寄存器，连接上了总线右边的AHB总线上，所以DMA既是总线矩阵的主动单元，可以读写各种存储器，也是AHB总线上的被动单元，DMA请求就是DMA的硬件触发源，比如说ADC转换完成、串口接收到数据需要触发DMA转运数据的时候，就会通过这条线路，向DMA发出硬件触发信号，之后DMA就可以执行数据转运的工作了，这就是DMA请求的作用。

Flash是ROM只读存储器的一种，如果通过总线直接访问的话，无论是CPU，还是DMA，都是只读的，只能读取数据，而不能写入，如果DMA的目的地址，填写了Flash的区域，那转运时就会出错。也可以配置Flash接口控制器，对Flash进行写入，先对Flash进行擦除，再写入数据。

##### DMA的基本结构图

![image-20221228222645589](https://gitee.com/pu-heliang/photos/raw/master/img/202301032049195.png)

DMA的数据转运可以从外设到存储器，也可以是从存储器到外设，也可以从存储器转运到存储器，外设和存储器两个站点，都有3个参数，第一个是起始地址，有外设端的起始地址，和存储器端的起始地址，这两个参数决定了数据时从哪里来，到哪里去的，第二个参数是数据宽度，这个参数的作用是，指定一次转运要按多大的数据宽度来进行，可以选择字节Byte、半字节HalfWord和字Word每，字节就是8位转运一个uint8_t，半字节是16位uint16_t，字是32位uint32_t,例如ADC的数据，ADC的数据是uint16_t，所以参数就要选择半字节，依次转运一个uint16_t，第三个参数是地址是否自增，这个参数的作用是，指定一次转运完成后，下一次转运，是不是要把地址移动到下一个位置去，相当于是指针p++，比如ADC扫描模式，用DMA转运数据，外设地址是ADC_DR寄存器，寄存器这边，显然地址是不用自增的，如果自增下一次转运就跑到别的寄存器那里了，存储器这边地址就需要自增，每转运一个数据后，就往后挪个坑，要不然下次再转就把上次的覆盖掉了，这就是地址是否自增的作用，就是指定是否转运一次就挪个坑。

传输存储器：用来指定，总共转运几次，这个传输计数器是个自减计数器，比如写个5，那DMA就只能进行5次数据转运，转运过程中，每转运一次计数器的值就会减1，当传输计数器减到0之后，DMA就不会再进行数据转运了，减到0之后之前自增的地址，也会恢复到起始地址的位置，以方便之后DMA新一轮的转运。传输计数器的右边的自动重装器的作用就是，传输计数器减到0之后，是否要自动恢复到最初的值。比如传输计数器给5，如果不使用自动红装器，那转运5次后，DMA就结束了，如果使用自动重装器，那转运5次，计数器减到0后，就会立即重装到初始值5，自动重装器决定了转运的模式，如果不重装，就是正常的单次模式，如果重装就是循环模式，如果你想转运一个数组，那一般是单次模式，转运一轮就结束了，如果是ADC扫描模式+连续转换那为了配合ADC，DMA也需要使用循环模式，这个循环模式和ADC的连续模式差不多。

DMA的触发控制，触发就是决定DMA在什么时机进行转运的，触发源，有硬件触发，和软件触发，具体选择由M2M（Memory to Memory ）这个参数决定，当给M2M位1时，DMA就会选择软件触发，这个软件触发不是调用某个函数一次就触发一次，而是，以最快的速度，连续不断地出发DMA，指一直到传输计数器清0，软件触发和循环模式不能同时用，因为软件触发是想把传输计数器清零，循环模式是清零后自动重装，如果同时用，那DMA就停不下了，软件触发一般适用于存储器到存储器的转运，因为存储器到存储器的转运是软件启动不需要时机，当M2M位给0，那就是使用硬件触发了，硬件触发源可以选择ADC、串口、定时器等等，使用硬件触发的转运一般是与外设有关的转运，这些转运需要一定的时机，比如ADC转换完成、串口收到数据、定时时间到等等，当硬件达到这些时机时，传一个信号过来，来触发DMA进行转运。

当给DMA使能后，DMA就准备就绪，可以进行转运了。

DMA进行转运的条件：第一，开关控制，DMA_Cmd必须使能，第二，传输计数器必须大于0，第三，触发源，必须有触发信号，触发一次，转运一次，传输计数器自减一次，当传输计数器等于0，且没有自动重装时，无论是否触发，DMA都不会再进行转运了，此时需要DMA_Cmd，给DISABLE，关闭DMA，再为传输计数器写入一个大于0的数，再DMA_Cmd，给ENABLE，开启DMA，DMA才能继续工作，写传输计数器时，必须要先关闭DMA，再进行，不能在DMA开启时，写传输计数器。

##### DMA请求

![image-20221228230043966](https://gitee.com/pu-heliang/photos/raw/master/img/202301032049883.png)

此图是DMA1的请求映像，下面是DMA的7个通道，每个通道都有一个数据选择器，可以选择硬件触发和软件触发，左边的硬件触发源，每个通道的硬件触发源都是不同的，如果想选择ADC1来触发必须选择通道1，如果想选择TIM2的更新事件来触发的话，那就必须选择通道2，每个通道的硬件触发源都不同，如果想使用某个硬件触发源的话，就必须使用它所在的通道。如果使用软件触发那通道就可以任意选择。如果要使用ADC1，那就有个库函数ADC_DMACmd，必须使用这个库函数开启ADC1的这一路输出，它才有效，如果想要选择定时器2的通道3那也会有个TIM_DMACmd函数，用来进行DMA输出控制，触发源具体选择哪个，取决于你把哪个外设的DMA输出开启了，如果都开启了，那是一个或门，理论上三个硬件都可以触发，一般情况下，都是开启其中一个，这7个触发源，进入到仲裁器，进行优先级判断，最终产生内部的DMA1请求，默认优先级是通道号越小，优先级越高，也可以在程序中配置优先级

##### 数据宽度与对齐

![image-20221228231138118](https://gitee.com/pu-heliang/photos/raw/master/img/202301032049212.png)

第一列是源端宽度，第二列是目标宽度，第三列是传输数目，当源端宽度和目标宽度都是8位时，转运第一步在源端的0位置，读数据B0，在目标的0位置，写数据B0，之后就是把B1，从左边挪到右边，接着B2、B3，这是源端和目标都是8位的情况，操作也很正常，继续就是源端是8位，目标是16位，它的操作就是，在源端读B0，在目标写00B0，之后读B1写00B1，等等，意思就是如果目标宽度，比源端的数据宽度大那就在目标数据前面多出来的空位补0，之后8位转运到32位，也是一样的处理，前面空出来的都补0，当目标数据宽度，比源端数据宽度小时，比如由16位转到8位现象就是，读B1B0，只写入B0，读B3B2，只写入B2，把多出来的高位舍弃掉，意思就是如果你把小的数据转到大的里面，高位就会补0，如果把大的数据转到小的里面去，高位就会舍弃掉，如果数据宽度一样，那就没事。

##### 数据转运+DMA

![image-20221228232030872](https://gitee.com/pu-heliang/photos/raw/master/img/202301032049790.png)

将SRAM中的数组DataA，转运到另一个数组DataB中，参数配置：外设地址是DataA数组的首地址，存储器地址，给DataB数组的首地址，数据宽度，两个数组的类型都是uint8_t，所以数据宽度都是按8位的字节传输，两个站点的地址都自增，转运完成后DataB数组的所有数据。就会等于DataA数组。如果左边不自增，右边自增，转运完成后，DataB的所有数据都会等于DataA[0]，如果左边自增，右边不自增，DataB[0]等于DataA的最后一个数，DataB其他的数不变，如果左右都不自增，那就是DataA[0]转到DataB[0]，其他的数据不变。方向参数，是外设站点转运到存储器站点。传输计数器给7，不需要自动重装，触发选择部分选择软件触发，最后调用DMA_Cmd，给DMA使能，转运7次后，传输计数器自减到0，DMA停止，转运完成，这里的数据转运是一种复制转运，转运完成后的DataA的数据并不会消失。

##### ADC扫描模式+DMA

![image-20221228233025173](https://gitee.com/pu-heliang/photos/raw/master/img/202301032049962.png)

左边是ADC扫描模式的转运流程，触发一次，7个通道依次进行AD转换，然后把转换结果都放在ADC_DR寄存器里面，在每个单独的通道转换完成后，进行一次DMA数据转运，并且目的地址进行自增，防止数据被覆盖，DMA的配置，外设地址，写入ADC_DR这个寄存器的地址，存储器的地址，可以在SRAM中定义一个数组ADValue然后把ADValue的地址当做存储器的地址，之后数据宽度，因为ADC_DR和SRAM数组需要uint16_t的数据，所以数据宽度都是16位的半字传输，外设地址不自增，存储器地址自增，传输方向，是外设站点到存储器站点，传输计数器和通道数一样，通道有7个，所以计数7次，计数器知否重装，看ADC的配置，ADC如果是单次扫描，那DMA的传输计数器可以不自动重装，转换一轮就停止，如果ADC是连续扫描，那DMA就可以选择使用自动重装，在ADC启动下一轮的转换的时候，DMA也启动下一轮的转运，ADC和DMA同步工作，触发选择ADC的硬件触发，ADC扫描模式在单个通道完成转换后，不会置任何标志位，也不会产生中断，但是会产生DMA请求，去触发DMA转运。一般来说DMA最常用的用途就是配合ADC的扫描模式，来解决ADC固有的缺陷，数据覆盖的问题。

##### 初始化DMA步骤：

第一步，RCC开启DMA的时钟,AHB总线的设别

第二步，直接调用DMA_Init，初始化配置的参数，包括外设和存储器站点的起始地址、数据宽度、地址是否自增、方向、传输计数器、是否需要自动重装、选择触发源、通道优先级

第三步，DMA_Cmd给指定通道使能，如果使用的是硬件触发，要在对应外设调用XXX_DMACmd，开启一下触发信号的输出，需要DMA的中断，就调用DMA_ITConfig，开启中断输出，再在NVIC中配置相应的中断通道，然后写中断函数就行了，如果传输计数器清0，再想给传输计数器赋值，就DMA失能、写传输计数器、DMA使能，就可以了

##### DMA的库函数

##### 恢复缺省配置

```c
void DMA_DeInit(DMA_Channel_TypeDef* DMAy_Channelx);
```

初始化

```c
void DMA_Init(DMA_Channel_TypeDef* DMAy_Channelx, DMA_InitTypeDef* DMA_InitStruct);
```

##### 结构体初始化

```c
void DMA_StructInit(DMA_InitTypeDef* DMA_InitStruct);
```

##### 使能

```c
void DMA_Cmd(DMA_Channel_TypeDef* DMAy_Channelx, FunctionalState NewState);
```

##### 中断输出使能

```c
void DMA_ITConfig(DMA_Channel_TypeDef* DMAy_Channelx, uint32_t DMA_IT, FunctionalState NewState);
```

##### DMA_设置当前数据寄存器

##### 给传输计数器写数据的

```c
void DMA_SetCurrDataCounter(DMA_Channel_TypeDef* DMAy_Channelx, uint16_t DataNumber); 
```

##### DMA获取当前数据寄存器

##### 返回传输计数器的值

```c
uint16_t DMA_GetCurrDataCounter(DMA_Channel_TypeDef* DMAy_Channelx);
```

##### 获取标志位状态

```c
FlagStatus DMA_GetFlagStatus(uint32_t DMAy_FLAG);
```

##### 清除标志位状态

```c
void DMA_ClearFlag(uint32_t DMAy_FLAG);
```



##### 获取中断状态

```c
ITStatus DMA_GetITStatus(uint32_t DMAy_IT);
```



##### 清除中断挂起位

```c
void DMA_ClearITPendingBit(uint32_t DMAy_IT);
```

#### DMA程序示例

```c
uint16_t AD_Value[4];

void AD_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
	
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;//模拟输入
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);//这只规则组
	ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_2, 3, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_3, 4, ADC_SampleTime_55Cycles5);
		
	ADC_InitTypeDef ADC_InitStructure;
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;//连续转换
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;//扫描模式
	ADC_InitStructure.ADC_NbrOfChannel = 4;//四个通道
	ADC_Init(ADC1, &ADC_InitStructure);
	
	DMA_InitTypeDef DMA_InitStructure;
	DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;//源地址
	DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;//半字长
	DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;//不自增
	DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)AD_Value;//目标地址
	DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;//半字长
	DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;//地址自增
	DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;//外设到存储器
	DMA_InitStructure.DMA_BufferSize = 4;//传输计数器为4
	DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;//循环模式
	DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;//硬件触发
	DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;//中等优先级
	DMA_Init(DMA1_Channel1, &DMA_InitStructure);
	
	DMA_Cmd(DMA1_Channel1, ENABLE);
	ADC_DMACmd(ADC1, ENABLE);//打通ADC到DMA的通道
	ADC_Cmd(ADC1, ENABLE);
	
	ADC_ResetCalibration(ADC1);
	while (ADC_GetResetCalibrationStatus(ADC1) == SET);
	ADC_StartCalibration(ADC1);
	while (ADC_GetCalibrationStatus(ADC1) == SET);
	
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
}
```



### USART串口

##### 通信接口

- 通信的目的：将一个设备的数据传送到另一个设备，扩展硬件系统
- 通信协议：指定通信的规则，通信双方按照协议规则进行数据收发

- 全双工：通信双方能够同时进行双向通信，全双工，一般有两根通信线
- 单工：数据只能从一个设备到另一个设备

- TX和RX是单端信号，它们的高低信号都是相对于GND的，严格上来说GND也算是通信线，串口通信的TX，RX，GND是必须要接的。
- 串口通信有两根通信线（发送端TX和接收端RX）
- TX和RX要交叉连接
- 当只需单向的数据传输时，可以只接一根通信线
- 当电平标准不一致时，需要加电平转换芯片
- 复杂一点的串口通信还有时钟引脚、硬件流控制的引脚

![image-20221227204837052](https://gitee.com/pu-heliang/photos/raw/master/img/202301032043640.png)

##### 串口参数及时序

![image-20221227205148050](https://gitee.com/pu-heliang/photos/raw/master/img/202301032049654.png)

串口数据帧的整体结构：串口中，每一个字节都装载在一个数据帧里面，每个数据帧都由起始位、数据位和停止位组成，数据位有8个，代表一个字节的8位，还可以在数据位的最后加一个奇偶校验位，这样数据位总共就是9位，其中有效载荷时前8位，代表1个字节，校验位跟在有效载荷后面，占1位

波特率：规定串口通信的速率（串口一般使用异步通信，需要双方约定一个通信速率），例如每隔1s发送一位，接收方也要每隔1s接收一位，接收快了，就会重复接收某些位，如果接收慢了，就会漏掉某些位，发送和接收必须约定好速率，波特率本义是每秒传输码元的个数，单位是码元/s，或者直接叫波特(Baud)，比特率是每秒传输的比特数，单位是bit/s，或者叫bps，在二进制调制的情况下，一个码元就是一个bit，此时波特率就等于比特率，单片机的串口通信，基本都是二进制调制，也就是高电平表示1，低电平表示0，一位就是1bit，规定波特率为1000bps，表示1s要发1000位，每一位的时间就是1ms，发送方每隔1ms发送一位，接收方每隔1ms接收一位，

起始位：标志一个数据帧的开始，固定为低电平（串口的空闲状态是高电平，没有数据传输的时候引脚必须置高电平，作为空闲状态）需要传输的时候先发送一个起始位，起始位必须是低电平，来打破空闲状态的高电平，产生一个下降沿（告诉接受设备，这一帧数据要开始了），如果没有起始位，当发送8个1的时候，数据线一直都是高电平，没有任何波动，这样接收方就不知道我是否发送数据，所以必须要有一个固定为低电平的起始位，产生下降沿，来告诉接受设备，为要发送数据了-----起始位固定为0，产生下降沿，表示传输开始

停止位；在一个字节数据发送完成后，必须要有一个停止位，这个停止位的作用是，用于数据帧间隔，固定为高电平，同时这个停止位也是为下一个起始位做准备的，如果没有停止位，那当为数据最后一位是0的时候，下次再发送新的一帧，就没法产生下降沿了-----停止位固定为1，把引脚恢复成高电平，方便下一次的下降沿，如果没有数据了，引脚也为高电平，代表空闲状态

数据位：表示数据帧的有效载荷，1为高电平，0位低电平，低位先行

校验位：用于数据验证，是根据数据位计算得来的，串口使用奇偶校验位方法，奇偶校验可以用来判断数据传输是不是出错了，如果数据出错了可以选择丢弃或者要求重传，校验可以选择三种方式，无校验、奇校验和偶校验，无校验就是不需要校验位，波形就是上图左边的，起始位、数据位，停止位一共3个部分，奇校验和偶校验的波形就是上图右边的，起始位、数据位、校验位、停止位，总共4个部分，如果使用了奇校验，那么包括校验位在内的9位数据会出现奇数个1，如果传输 0000 1111，目前总共4个1，是偶数个，那么校验位就需要再补一个1，连同校验位就是0000 1111 1，总共5个1，保证1的个数为奇数，如果数据是0000 1110，此时3个1，是奇数个，那么校验位就补1个0，连同校验位就是0000 1110 0，总共还是3个1,1的个数为奇数，发送方，在发送数据后，会补一个校验位，保证1的个数为奇数，接收方在接收数据后，会验证数据位和校验位，如果1的个数还是奇数，就认为数据没有出错，如果在传输中，因为干扰，有一位由1变成0，或者由0变成1了，那么整个数据的奇偶特性就会变化，接收方一验证，发现1的个数不是奇数，那就认为传输出错，就可以选择丢弃，或者要求重传，这就是奇校验的差错控制方法。如果选择双方约定偶校验，那就是保证1的个数是偶数，校验方法也是同理，但是奇偶校验的检出率不是很高，例如，如果有两位数据同时出错，就特性不变，那就校验不出来了，就能校验只能保证一定程度上的数据校验，如果想要更高的检出率可以选择CRC校验，STM32内部也有CRC外设。

数据位：有两种表示方法，一种是把校验位作为数据位的一部分，另一种就是把校验位和数据位独立开，数据位就是有效载荷，校验位就是独立的1位，在串口助手里就是选择的把数据位和校验位分开描述的方法，总之无论是合在一起，还是分开描述，描述的都是同一个东西

![image-20221227213228380](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050854.png)

第一个波形：这个波形是发送一个数据0x55时，在TX引脚输出的波形，波特率是9600，每一位的时间就是1/9600，大概是104us，没发送数据的时候是空闲状态高电平，数据帧开始，先发送起始位，产生下降沿，代表数据帧开始，数据0x55转为2进制，低位先行，就是依次发送1010 1010，然后参数是，1位停止，无校验，所以数据帧之后就是停止位，把引脚置回高电平，在STM32中，这个根据字节数据翻转高低电平，是由USART外设自动完成的，不用我们管，也可以软件模拟产生这样的波形，定时器定一个104us的时间，时间到之后，按照数据帧要求，调用GPIO_WriteBit置高低电平，产生一个一模一样的波形，也可以完成串口通信，在TX引脚发送就是置高低电平，在RX引脚接收就是读取高低电平，这也可以由USART外设完成，如果想软件模拟的话那就是定时调用GPIO_ReadInputDataBit来读取每一位，接收的时候也需要一个外部中断，在起始位的下降沿触发，进入接收状态，并且对其采样时钟，然后依次采样8次，这就是接受的逻辑

USART ：同步收发器，UART：异步收发器

同步模式一般是为了兼容别的协议或者特殊用途而设计的，并不兼容两个USART之间进行同步同步通信，串口主要还是异步通信

USART (Universal Synchronous/Asynchronous Receiver/Transmitter)

USART是STM32内部集成的硬件外设，可根据数据寄存器的一个字节数据自动生成数据帧时序，从TX引脚发送出去，也可以自动接收RX引脚的数据帧时序，拼接成一个字节数据，存放在数据寄存器里

自带波特率发生器，最高达4.5Mbits/s，起始就是一个分频器，比如APB2总线给个72MHz的频率然后波特率发生器进行一个分频，得到我们想要的波特率时钟，在这个时钟下，进行收发，就是我们指定的通信波特率，

可配置数据位长度(8/9)、停止位长度(0.5/1/1.5/2)

可选校验位（无校验/奇校验/偶校验）

支持同步模式、硬件流控制、DMA、智能卡、IrDA、LIN，硬件流控制：A设备有个TX向B设备的RX发送数据，A设备一直在发，发的太快了，B处理不过来，如果没有硬件流控制，那B就只能抛弃新数据或者覆盖原数据了，如果有硬件流控制，在硬件电路上，就会多出一根线，如果B没准备好接受，就置高电平，如果准备好了，就置低电平，A接收到了B反馈的准备信号，就只会在B准备好的时候，才发数据，如果B没准备好，那数据就不会发送出去，硬件流控制可以防止处理慢而导致数据丢失的问题，硬件流控制STM32也是有的，但是一般不用，串口也支持DMA数据转运，如果有大量的数据进行收发，可以使用DMA转运数据，减轻CPU的负担

STM32F103C8T6 USART资源：USART1、USART2、USART3，USART1是APB2总线的设备，USART2,3是APB1总线的设备

![image-20221227221317657](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050963.png)

SW_RX、IRDA_OUT/IN这些是智能卡和IrDA通信的引脚

发送和接收的字节数据存在串口的数据寄存器，数据寄存器分为发送数据寄存器TDR(Transmit)，另一个是接收数据寄存器RDR(Receive DR)，两个寄存器占用同一个地址，在程序上只表现为一个寄存器，就是数据寄存器DR(Data Register)，但实际硬件中，分成了两个寄存器，一个用于发送的TDR，一个用于接收的RDR，TDR是只写的,RDR是只读的，当进行写操作时，数据就写入到TDR，当进行读操作的时候，数据就是从RDR中读出来的，还有两个移位寄存器，一个用于发送，一个用于接受，发送移位寄存器的作用就是，把一个字节的数据一位一位地移出去，正好对应串口协议的波形数据位，例如为在某时刻给TDR写入了0x55这个数据，在寄存器就是二进制存储,0101 0101,此时硬件检测到我写入数据了，就会检查移位寄存器是否有数据正在移位，如果没有，这个0101 0101就会立刻全部移动到发送移位寄存器，准备发送，当数据从TDR移动到移位寄存器的时候会置一个标志位，叫TXE(TX Empty)，发送寄存器空，检查这个标志位，如果置1了，就可以在TDR继续写入下一个数据了，当TXE标志位置1时，数据其实还没有发送出去，只要数据从TDR转移到发送移位寄存器了TXE就会置1，我们就可以写入新的数据了，然后发送移位寄存器就会在发生器控制的驱动下，向右移位，然后一位一位地，把数据输出到TX引脚，向右移位，正好与串口协议规定的低位先行是一致的，当数据移位完成后，新的数据就会在此自动地从TDR转移到发送移位寄存器里来，如果当前移位寄存器的移位还没有完成，TDR的数据进行等待，一但移位完成，就会立刻转移过来，TDR和移位寄存器的双重缓存，可以保证连续发送数据的时候，数据帧之间不会有空闲，提高了工作效率，简单来说就是数据一但从TDR转移到发送移位寄存器了，就立刻把下一个数据放在TDR等着，一但转移完毕，新的数据就会立刻跟上，这样做效率比较高。

接收端也是类似的，数据从RX引脚通向接收移位寄存器，在接收器控制器的驱动下，一位一位的读取RX电平，先放在最高位，然后向右移，移位8次后，就能接收板一个字节了，因为串口协议规定是低位先行，所以接受移位寄存器是从个高位往低位方向移动的，之后，当一个字节移位完成之后，这一个字节的数据就会整体地一下子转移到接收数据寄存器RDR里来，在转移的过程中也会置一个标志位，叫RXNE(RX Not Empty)，接收数据寄存器非空，当检测到RXNE置1之后，就可以把数据读走了，同样也是两个寄存器进行缓存，当数据从移位寄存器转移到RDR时，就可以直接接受下一帧数据了，这就是USART外设整个的工作流程。发送需要加上帧头帧尾，接收需要剔除帧头帧尾，这些操作内部电路会自动执行。

发送器控制：用来控制发送移位寄存器的工作的

接收器控制：用来控制接受移位寄存器的工作

硬件数据流控制：有两个引脚，一个是nRTS，一个是nCTS，nRTS(Request To Send)是请求发送，是输出脚，也就是告诉别人，我当前能不能接受，nCTS(Clear To Send)是清除发送，是输入脚也就是接受别人的nRTS的信号的，前面的n是低电平有效，使用步骤：找另一个支持流控的串口它的TX接到我的RX，然后我的RTS输出一个能不能接受的反馈信号，接到对方CTS，当我能接收的是吧，RTS就置低电平，请求对方发送，对方的CTS收到后们就可以一直发，当我处理不过来时，比如接收数据寄存器一直没有读，又有新的数据过来了，代表我没有及时处理，那RTS就置高电平，对方CTS接收到之后，就会暂停发送，直到接受数据寄存器被读走，RTS置低电平，新的数据才会继续发送，反过来，TX给对方发送数据时我的CTS就接到对方的RTS，用于判断对方，能不能接收，TX和CTS是一对的，RX和RTS是一对的，CTS和RTS也要交叉连接，这就是流控的工作模式（一般不使用流控）

右边的模块用于产生同步的时钟信号，配合发送移位寄存器输出，发送移位寄存器每移位一次，同步时钟就跳变一个周期，时钟告诉对方，我移出去一位数据了，看是否需要时钟信号来指导接受一下，这个时钟只支持输出不支持输入，两个USART之间，不能实现同步的串口通信，这个时钟信号的用途，第一个就是兼容别的协议，比如串口加上时钟之后，和SPI协议特别像，所以有了时钟输出的串口，就可以兼容SPI，这个时钟也可以做自适应波特率，比如接受设备不确定发送设备给的什么波特率，可以测量一下这个时钟的周期，然后计算得到波特率（需要另外写程序来实现这个功能）

唤醒单元：实现串口挂在多设备，串口一般是点对点的通信，点对点，只支持两个设备互相通信，想发数据直接发就行，而多设备，在一条总线上，可以接多个从设备，每个设备分配一个地址，先跟某个设备通信，就先进行寻址，确定通信对象，在定义数据收发，这个唤醒单元就可以用来实现多设备的功能，可以给串口分配一个地址，当我发送指定地址时，此设备唤醒开始工作，当我发送别的设备地址时，别的设备就唤醒工作，这个设备没收到地址，就会保持沉默，这样实现多设备的串口通信了。

中断输出控制：中断申请位就是状态寄存器的各种标志位，状态寄存器这里有两个标志位比较重要，一个是TXE发送寄存器空，另一个是RXNE接收寄存器空，这两个是判断发送状态和接收状态的必要标志位，中断输出控制就是配置中断是不是能够通向NVIC

波特率发生器：其实就是分频器，APB时钟进行分频，得到发送和接收移位的时钟，时钟输入是发PCLKx(x=1或2)，因为USART1挂载在APB2，所以就是PCLK2的时钟，一般是72M，其他的USART都挂载在APB1，所以是PCLK1的时钟，一般是36M，之后时钟在进行一个分频，除以一个USARTDIV的分频系数，USARTDIV是一个数值，分为整数部分和小数部分，因为有些波特率，用72M除于一个整数的话，可能除不尽，会有误差，所以这里的分频系数是支持小数点后4位的，分频就更加精准，之后分频完还要再除个16，得到发送时钟和接收器时钟，通向控制部分，然后右边，如果TE（TX Enable）为1，就是发送器使能，发送部分的波特率就有效，如果RE（RX Enable）为1，就是接收器使能了，接收部分的波特率就有效。

##### USART的基本结构

![image-20221227231140269](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050891.png)

最左边的是波特率发生器，用于产生约定的通信速率，时钟来源是PCLK2或1，经过波特率发生器分频后，产生的时钟通向发送控制器和接收控制器，发送控制器和接收控制器用来控制发送移位和接收移位，之后由发送数据寄存器和发送移位寄存器这两个寄存器的配合，将数据一位一位的移出去，通过GPIO口的复用输出，输出到TX引脚，产生串口协议规定的波形，这个移位寄存器是向右移的，是低位先行，当数据由数据寄存器转移到移位寄存器时，会置一个TXE的标志位，通过判断这个标志位，就可以知道是不是可以写入下一个数据了，接收部分也是类似的，RX引脚的波形，通过GPIO输入，在接收控制器的控制下，一位一位地移入接收移位寄存器，移完一帧数据后，数据就会统一转运到接收数据寄存器，在转移的同时，置一个RXNE标志位，检查这个标志位，就可以知道是不是收到数据了，同时这个标志位也可以去申请中断，这样就可以在收到数据时，直接进入中断函数，快速的读取和保存数据，虽然有四个寄存器但是在软件层面上，只有一个DR寄存器可以供我们读写，写入DR时，数据走上面这条路，进行发送，读取DR时，数据走下面这条路，进行接收，这就是USART进行串口数据收发的过程，右下角是个开关控制。

![image-20221227234306675](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050078.png)

四种选择：9位字长，有校验或无校验，8位字长有校验或者无校验，最好选择9位字长，有校验或者8位字长无校验，这样每一帧的有效载荷都是1字节，

![image-20221227234437636](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050088.png)

STM32的串口可以配置停止位为0.5、1、1.5、2，这四种参数的区别，就是停止位的时长不一样，1位停止位，这时停止位的时长就和数据位的一位，时长一样，1.5停止位就是数据位一位，时长的1.5倍，2个停止位，那停止位时长就是2倍，0.5个停止位，时长就是0.5倍，一般选择1位停止位。

![image-20221227234843163](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050872.png)

串口的输出TX比输入RX简单很多，输出就定时翻转TX引脚高低电平就可以，输入不仅要保证采样频率和波特率一致，还要保证每次输入采样的位置，要正好处于每一位的正中间，只有在每一位的正中间采样，这样高低电平读进来，才是最可靠的，如果采样点过于靠前或者靠后，那有可能高低电平正在翻转，电平还不稳定，或者稍有误差，数据就采样错了，输入最好还要对噪声有一定的判断能力，如果是噪声，最好能置个标志位提醒一下，STM32设计的输入电路，上图展示的是USART的起始位侦测，当输入电路侦测到一个数据帧的起始位后，就会以波特率的频率，连续采样一帧数据，同时，从起始位开始，采样位置就要对齐到位的正中间，只要第一位对齐了，后面都是对齐的，为了实现这些功能对输入的电路对采样时钟进行了细分，它会以波特率的16倍频率进行采样，也就是在一位地时间里，可以进行16次采样，它的策略是最开始，空闲状态高电平，那采样就一直是1，在某个位置突然采集到一个0，那么就说明在这两次采样之间，出现了下降沿，如果没有任何噪声，那之后就应该是起始位了，在起始位，会进行连续16次采样，没有噪声的话，这16次采样，肯定都是0，实际电路有噪声，即使出现下降沿了，后序也要再采样几次，以防万一，这个接收电路还会再下降沿之后的第3次、5次、7次，进行一批采样，在第8次、9次、10次，再进行一批采样，且这两批采样，都要要求每3位里面至少有2个0，没有噪声就全是0，满足情况，如果有轻微的噪声，导致3位里面，只有两个0，另一个是1，也算是检测到了起始位，但是在状态寄存器里会置一个NE（Noise Error），噪声标志位，提醒一下，数据收到了，但是有噪声，如果3位里面只有一个0，就不算检测到了起始位，这时电路就忽略前面的数据，重新开始捕捉下降沿，这就是STM32的串口，在接收过程中，对噪声的处理，如果通过了这个起始位侦测，那接收状态就由空闲，变为接收起始位，同时，第8、9、10次采样的位置，就正好是起始位的正中间，之后接收数据位时，就都在第8、9、10次，进行采样，这样就能保证采样位置在位的正中间了，这就是起始位侦测和采样位置对齐的策略

串口初始化：

第一步，开启时钟，把需要用的USART和GPIO的时钟打开

第二步，GPIO初始化，把TX配置成复用输出，RX配置成输入

第三步，配置USART，直接用一个结构体，就可以配置好所有参数

第四步，如果只需要发送的功能就直接开启USART，如果需要接收的功能，还需要再配置中断，在开启USART之前，加上ITConfig和NVIC的代码就可以了

##### 回复缺省值函数

```c
void USART_DeInit(USART_TypeDef* USARTx);
```

##### 配置结构体函数

```c
void USART_Init(USART_TypeDef* USARTx, USART_InitTypeDef* USART_InitStruct);
```

##### 给结构体配置默认值函数

```c
void USART_StructInit(USART_InitTypeDef* USART_InitStruct);
```

##### 配置同步时钟输出函数

包括时钟是否输出，时钟的极性相位等参数

```c
void USART_ClockInit(USART_TypeDef* USARTx, USART_ClockInitTypeDef* USART_ClockInitStruct);
void USART_ClockStructInit(USART_ClockInitTypeDef* USART_ClockInitStruct);
```

##### 开启串口函数

```c
void USART_Cmd(USART_TypeDef* USARTx, FunctionalState NewState);
```

##### 开启串口中断函数

```c
void USART_ITConfig(USART_TypeDef* USARTx, uint16_t USART_IT, FunctionalState NewState);
```

##### 开启USART到DMA的触发通道函数

```c
void USART_DMACmd(USART_TypeDef* USARTx, uint16_t USART_DMAReq, FunctionalState NewState);
```

##### 设置地址函数

```c
void USART_SetAddress(USART_TypeDef* USARTx, uint8_t USART_Address);
```

##### 唤醒函数

```c
void USART_WakeUpConfig(USART_TypeDef* USARTx, uint16_t USART_WakeUp);
```

##### LIN函数

```c
void USART_ReceiverWakeUpCmd(USART_TypeDef* USARTx, FunctionalState NewState);
```

##### 发送数据函数

写DR寄存器

```c
void USART_SendData(USART_TypeDef* USARTx, uint16_t Data);
```

##### 接收数据

##### 读DR寄存器函数

```c
uint16_t USART_ReceiveData(USART_TypeDef* USARTx);
```

##### 智能卡、IrDA函数

```c
void USART_SendBreak(USART_TypeDef* USARTx);
void USART_SetGuardTime(USART_TypeDef* USARTx, uint8_t USART_GuardTime);
void USART_SetPrescaler(USART_TypeDef* USARTx, uint8_t USART_Prescaler);
void USART_SmartCardCmd(USART_TypeDef* USARTx, FunctionalState NewState);
void USART_SmartCardNACKCmd(USART_TypeDef* USARTx, FunctionalState NewState);
void USART_HalfDuplexCmd(USART_TypeDef* USARTx, FunctionalState NewState);
void USART_OverSampling8Cmd(USART_TypeDef* USARTx, FunctionalState NewState);
void USART_OneBitMethodCmd(USART_TypeDef* USARTx, FunctionalState NewState);
void USART_IrDAConfig(USART_TypeDef* USARTx, uint16_t USART_IrDAMode);
void USART_IrDACmd(USART_TypeDef* USARTx, FunctionalState NewState);
```

##### 在中断函数外获取标志位函数

```c
FlagStatus USART_GetFlagStatus(USART_TypeDef* USARTx, uint16_t USART_FLAG);
```

##### 在中断函数外清除标志位函数

```c
void USART_ClearFlag(USART_TypeDef* USARTx, uint16_t USART_FLAG);
```

##### 在中断函数内获取标志位函数

```c
ITStatus USART_GetITStatus(USART_TypeDef* USARTx, uint16_t USART_IT);
```

##### 在中断函数内清除标志位函数

```c
void USART_ClearITPendingBit(USART_TypeDef* USARTx, uint16_t USART_IT);
```

#### 串口通信程序示例

```c
#include "stm32f10x.h"                  // Device header
#include <stdio.h>
#include <stdarg.h>

void Serial_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	USART_InitTypeDef USART_InitStructure;
	USART_InitStructure.USART_BaudRate = 9600;//波特率
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;//无流控
	USART_InitStructure.USART_Mode = USART_Mode_Tx;//发送模式
	USART_InitStructure.USART_Parity = USART_Parity_No;//无校验位
	USART_InitStructure.USART_StopBits = USART_StopBits_1;//停止位一位
	USART_InitStructure.USART_WordLength = USART_WordLength_8b;//8位1字节
	USART_Init(USART1, &USART_InitStructure);
	
	USART_Cmd(USART1, ENABLE);//开启串口
}

void Serial_SendByte(uint8_t Byte)//发送字符
{
	USART_SendData(USART1, Byte);
	while (USART_GetFlagStatus(USART1, USART_FLAG_TXE) == RESET);
}

void Serial_SendArray(uint8_t *Array, uint16_t Length)//发送数组
{
	uint16_t i;
	for (i = 0; i < Length; i ++)
	{
		Serial_SendByte(Array[i]);
	}
}

void Serial_SendString(char *String)//发送字符串
{
	uint8_t i;
	for (i = 0; String[i] != '\0'; i ++)
	{
		Serial_SendByte(String[i]);
	}
}

uint32_t Serial_Pow(uint32_t X, uint32_t Y)
{
	uint32_t Result = 1;
	while (Y --)
	{
		Result *= X;
	}
	return Result;
}

void Serial_SendNumber(uint32_t Number, uint8_t Length)//发送数字
{
	uint8_t i;
	for (i = 0; i < Length; i ++)
	{
		Serial_SendByte(Number / Serial_Pow(10, Length - i - 1) % 10 + '0');
	}
}

int fputc(int ch, FILE *f)//重定向printf
{
	Serial_SendByte(ch);
	return ch;
}

void Serial_Printf(char *format, ...)//重定向printf多串口使用
{
	char String[100];
	va_list arg;
	va_start(arg, format);
	vsprintf(String, format, arg);
	va_end(arg);
	Serial_SendString(String);
}

```



### 串口收发数据包

数据包的作用是：把一个个单独的数据给打包起来，方便进行多字节的数据通信，例如，陀螺仪传感器，需要用串口发送数据到STM32，陀螺仪的数据，X轴为一个字节、Y轴为一个字节、Z轴一个字节，一共3个数据需要连续不断的发送，当我像这样XYZXYZXYZ连续发送的时候，接受方不知道这个数据哪个对应Y，哪个对应X，哪个对应Z，因为接收方可能从任意的位置接收，所以可能出现数据错位的现象，我们需要一种方式把数据进行分割，把XYZ这一批数据分割开来，分割成一批批数据包，这样在接收的时候，就知道了，数据包第一个数据就是X，数据包第二个数据就是Y，数据包第三个数据就是Z，这就是数据包的任务，就是把同一批的数据进行打包和分割。

串口数据包，通常使用的是额外添加包头包尾的这种方式

防止数据包包头包尾和数据重复的方法，第一种，限制载荷数据的范围，在发送的时候对数据进行限幅，第二种，尽量使用固定长度的数据包，第三种，增加包头包尾的数量，并且让它尽量呈现出载荷数据出现不了的状态。

##### 串口收发Hex数据包

![image-20221228025831646](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050993.png)



##### 串口收发文本数据包

![image-20221228031427199](https://gitee.com/pu-heliang/photos/raw/master/img/202301032050392.png)

##### 数据包的收发流程

##### 数据包的发送

数据包的接收

接收固定包长的数据包，设计一种能够记住不同状态的机制，在不同状态执行不同的操作，同时还要进行状态的合理转移，这种程序设计思维叫做“状态机”。

![image-20221228032032420](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051259.png)

第一个状态是等待包头，第二个状态是接收数据，第三个状态是等待包尾，每个状态需要一个变量来标志一下，类似于置置标志位，标志位只有0和1，状态机是多标志位的一种方式

执行流程是最开始S = 0，收到一个数据，进中断，根据S = 0，进入第一个状态的程序，判断数据是不是包头FF，如果是FF，则代表收到包头，之后置S = 1，退出中断，结束，这样下次再进中断，根据S = 1，就可以进行接收数据的程序了，在第一个状态，如果收到的不是FF就说明数据包没有对齐，应该等待数据包包头的出现，这时状态仍然是0，下次进中断，就还是判断包头的逻辑，直到出现FF，才能转到下一个状态，之后出现了FF，就可以转移到接收数据的状态了，这时再收到数据，就可以直接把它存在数据中，另外再用一个变量，记录收了多少个数据，如果没收够4个，就一直是接收状态，如果收够了，就置S = 2，下次进中断时，就可以进入下一个状态了，最后一个状态就是等待包尾，判断数据是不是FE，这样就可以置S = 0，回到最初的状态，开始下一个轮回。

状态机使用的基本步骤：先根据项目要求，画几个圈，考虑好各个状态在什么情况下会进行转移，如何转移，画好线和转移条件，最后根据图来进行编程，例如，做个菜单，按什么键，切换什么菜单，执行什么程序。

![image-20221228033422462](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051552.png) 

### I2C

- I2C总线（Inter IC Bus）
- 两根通信线：SCL（Serial Clock）、SDA（Serial Data）
- 同步，半双工
- 带数据应答
- 支持总线挂在多设备（一主多从、多主多从）
- 起始条件：SCL高电平期间，SDA从高电平切换到低电平
- 终止条件：SCL高电平期间，SDA从低电平切换到高电平
- 每个时序单元的SCL都是以低电平开始，低电平结束
- 从机不允许产生起始和终止

![image-20221225003129365](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051634.png)

- 发送一个字节：SCL低电平期间，主机将数据位依次放到SDA线上（高位先行），然后释放SCL，从机将在SCL高电平期间读取数据位，所以SCL高电平期间SDA不允许有数据变化，依次循环上述过程8次即可发生一个字节

![image-20221226205443369](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051462.png)

- 主机拉低SCL，把数据放在SDA上，主机松开SCL，从机读取SDA的数据

- （高位先行）在SCL低电平期间，主机如果想要发送0，就拉低SDA到低电平，如果想要发送1，就放手，SDA回弹到高电平，在SCL低电平期间允许改变SDA的电平，当这一位放好后，主机就松手时钟线，SCL回弹到高电平，在高电平期间是从机读取SDA的时候，SCL高电平期间，SDA不允许变化，SDA处于高电平时从机需要尽快读取SDA，一般是在上升沿的时刻，从机已经读取完成了，主机在放手SCL一段时间后，就可以继续拉低SCL传输下一位，主机需要在SCL下降沿之后尽快把数据放在SDA上，主机有时钟的主导权，不需要着急，只需要在低电平的任意时刻把数据放在SDA上就行了，数据放完之后，主机再松手SCL，SCL高电平从机读取这一位，在SCL的同步下，依次进行主机发送和从机接收，循环8次就发送了8位数据，也就是一个字节。

- 接收一个字节：SCL低电平期间，从机将数据位依次放到SDA线上（高位先行），然后释放SCL，主机将在SCL高电平期间读取数据位，所以SCL高电平期间期间SDA不允许有数据变化，依次循环上述过程8次，即可接收一个字节（主机再接收之前，需要释放SDA，释放SDA就相当于切换为输入模式）。

![image-20221225004611284](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051676.png)

- 也可以理解为：所有设备包括主机始终都属于输入模式，当主机需要发送的时候，就可以去主动拉低SDA，而主机再被动接收的时候，就必须先释放SDA，总线是线与的特征，任何一个设备拉低了，总线就是低电平，如果接收的时候还拽着SDA不放手，无论别人发什么数据，总线都始终属于是低电平。

- 发送应答：主机在接收完一个字节之后，在下一个时钟发送一位数据，数据0表示应答，数据1表示非应答
- 接收应答：主机在发送完一个字节之后，在下一个时钟接收一位数据，判断从机是否应答，数据0表示应答，数据1表示非应答（主机在接收之前，需要释放SDA）

![image-20221225005553172](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051606.png)

- 也可以理解为发送1位和接收1位，这一位用来作为应答，在发送完一个数据之后，就要立即进行接收应答，来判断从机是否接收到主机发送的数据
- 主机在起始条件之后，要先发送一个字节叫一下从机名字，所有从机都会收到第一个字节，与自己的名字（地址）比较，如果一样，相对应的从机就会响应主机的读写操作，在同一条I2C总线里，挂在的每个设别地址必须不一样，防止主机叫一个地址有多个设备都响应。
- 从机设备地址，在I2C协议标准里分为7位地址和10位地址
- 每个I2C设厂时，厂商都会为它分配一个7位的地址、
- MPU6050的地址是：1101 000
- 一般不同型号的设备地址都是不同的，相同型号的设备地址都是相同的
- 如果相同型号的设备挂在在同一条总线上，可以利用设备的地址的可变部分，一般器件地址的最后几位是可以在电路中改变的，例如MPU6050地址的最后一位，由板子上的AD0引脚确定，AD0引脚接低电平，那它的地址就是1101 000，AD0引脚接高电平那它的地址就是1101 001，AT24C02地址的最后三位都可以分别由这个板子上的A0、A1、A2引脚确定

#### 指定地址写

- 对于指定设备（Slave Address），在指定地址（Reg Address）下写入数据（Data）

- 空闲状态下两个总线都是高电平，主机需要给从机写入数据的时候，在SCL高电平期间，拉低SDA，产生起始条件，在起始条件之后紧跟的时序，必须是发送一个字节的时序，字节的内容必须是从机地址+读写位，（从机地址是7位，读写位是1位加起来就是1个字节8位）（发送从机地址：确定通信的对象），（发送读写位：确认接下来是要写入还是读出，0：写入，1：读出），紧跟着的单元是接收从机的应答位(Receive Ack,RA),这个时刻主机需要释放SDA，如果从机应答，从机会立即拉低SDA，应答位产生后，从机释放SDA，从机交出SDA的控制权，同样的时序再来一遍，第二个字节数据就会送入指定数据的内部，一般第二个字节是寄存器地址或者是指令控制字，第三个字节是想要往寄存器地址中写入的值，如果主机不想发送数据了，要产生停止条件，在产生停止条件之前，先拉低SDA，会后续的上升沿做准备，然后释放SCL，再释放SDA，产生SCL高电平期间SDA的上升沿

![image-20221228195109520](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051953.png)

- 此数据帧的作用是：对于从机地址为1101000的设备在其内部0x19地址的寄存器中，写入0xAA这个数据

#### 当前地址读

- 对于不指定设别（Slave Address），在当前地址指针指示的地址下，读取从机数据（Data）

![image-20221228195140964](https://gitee.com/pu-heliang/photos/raw/master/img/202301032051730.png)

- 在SCL高电平期间，拉低SDA，产生起始条件，主机首先发送一个字节，来进行从机的寻址和读写标志位，图示波形代表，本次寻址的目标是1101000的设备，读写标志为1，表示主机接下来想要读取设备，发送一个字节后，接收从机应答位，代表从机收到了第一个字节，把SDA的控制权交给从机，主机调用接收一个字节的时序，进行接收操作，从机接收到了主机的允许，可以在SCL低电平期间写入SDA，主机在哪SCL高电平期间读取SDA，主机再SCL高电平期间依次读取8位，就接收到了从机发送的一个字节数据0000 1111也就是（0x0F），没有指定地址这个环节，0x0F，（在从机中所有寄存器被分配到了一个线性区域中，会有个单独的指针变量，指示着其中一个寄存器，这个指针上电一般默认0地址，每写入一个字节或者读出一个字节后，这个指针就是自动自增一次，移动到下一个位置），从机返回的是当前指针指向的寄存器的值

#### 指定地址读

- 对于指定设备（Slave Address），在指定地址（Reg Address）下读取从机数据（Data）

![image-20221225014731596](https://gitee.com/pu-heliang/photos/raw/master/img/202301032052615.png)

- 指定从机地址是1101000 读写标志位是0，代表要进行写的操作，经过从机应答后，在发送一个字节第二个字节0001 1001，用来指定地址，这个数据就写入到从机的地址指针里了，从机接收到这个地址后，它的寄存器指针就指向了0x19这个位置，不给从机发要写入的数据，而是再来个起始条件，起始条件后，重新寻址并且指定读写标志位，此时读写标志位为1代表开始读，继续主机接收一个字节，这个字节数据就是0x19地址下的数据。

写多个字节：重复三遍，发送一个字节和接收应答，第一个数据就写入0x19的位置（写入一个地址后地址指针会自动+1，编程吧0x1A）第二个数据就会写到0x1A的位置，第三个数据写入的是0x1B的位置

S欧拉角：飞机与XYZ轴的夹角，反应了飞机的姿态，侧仰，上倾，下倾；

获得欧拉角需要多个数据，常用的数据融合算法：互补滤波、卡尔曼滤波等

MPU6050 XCL和SDA是扩展使用，通常是外接磁力计或者气压计

### I2C外设

STM32内部集成了硬件收发电路，可以由硬件自动执行时钟生成、起始终止条件生成、应答位收发、数据收发等功能，软件只需要写入控制寄存器CR和数据寄存器DR就可以实现协议，为了实时监控时序的状态，软件还需要读取状态寄存器SR，来了解外设电路当前处于什么状态，类似于开车，写入控制寄存器CR，就像是踩油门、打方向盘来控制汽车的运行，读取状态寄存器SR，就像是观看仪表盘，来观测汽车的运行状态，有了这些寄存器，就可以完全掌握外设电路的运行了，同时也可以减轻CPU的负担，

- 支持多主机模型
- 支持7位/10位地址模式
- 支持不同的通讯速度，标准速度(高达100kHz)，快速(高达400kHz)
- 支持DMA
- 兼容SMBus协议
- STM32F103C8T6 硬件I2C资源：I2C1、I2C2

I2C通信，分为主机和从机，主机拥有主动控制总线的权利，从机只能在主机允许的情况下，才能控制总线，主机一个人掌控所有，不存在权利冲突

![image-20230113185249502](https://gitee.com/pu-heliang/photos/raw/master/img/202301131852537.png)

进阶版的I2C设计了多主机的模型，多主机模型分为固定多主机和可变多主机，固定多主机就是总线上有两个或多个固定的主机，上面几个始终固定为主机，下面的几个时钟固定为从机，就像是教室里讲台上站了多个老师，下面坐的学生可以被任意一个老师点名，老师可以主动发起对学生的控制，学生不能控制老师，当两个老师同时想说话就是总线冲突状态，这时要进行总线仲裁，仲裁失败的一方让出总线控制权，这种讲台上站多个老师的情况就是固定多主机。

![image-20230113185309643](https://gitee.com/pu-heliang/photos/raw/master/img/202301131853676.png)

可变多主机就是，总线上可以挂载多个设备，总线上没有固定的主机和从机，任何一个设备都可以在总线空闲时跳出来作为主机，然后指定其他任何一个设备进行通信，当这个通信完成后，跳出来的主机要退回从机的位置，就像是在教室里，只有一堆学生，没有老师，默认情况下，所有学生都是从机，都不能说话，当有某个学生想说话 时，站出来变成主机，然后指定其他任何一个学生进行通信，通信完成后坐下变成从机。当有多个学生同时跳出来时，就是总线冲突状态，这时要进行总线仲裁，仲裁失败的一方让出总线控制权，这种所有设备一视同仁，谁做主机谁跳出来的模型，就是可变多主机，对stm32的I2C使用的是可变多主机的模型

![image-20230113185326426](https://gitee.com/pu-heliang/photos/raw/master/img/202301131853476.png)

I2C起始之后的第一个字节，必须是寻址+读写位，一个字节只能有7位地址，只需要在规定，起始位之后的前两个字节，都作为寻址，就可以完成10位地址寻址，这就是10位地址的基本思路，第一个字节有7个空位，第二个字节，有8ge空位，加一起是15位地址，但是I2C只有10位地址模式，还剩下5位当标志位，因为发送第一个字节后，不知道后面这个字节是不是寻址，所以需要再第一个字节写个特定的数据，作为10位寻址位的标志位，标志位就是11110，也就是第二个字节如果也是寻址，那第一个字节的前5位就必须是11110，如果前5位是11010，就说明它是7位寻址，如果前五位是11110，那第一个字节剩下的两位，和第二个字节的8位，都作为寻址，这就组成了10位地址，11110作为10位地址模式的标志位，不会在7位地址模式下出现的，这就是7位地址和10位地址的1区别。

支持DMA可以在多字节传输的时候提高传输效率，比如指定地址读多字节或写多字节的时序，如果想要连续读或者写非常多的字节那用一下DMA自动帮忙转运数据，这个过程的效率就会大大提升，如果只有几个字节就没必要用DMA了。

兼容SMBus（System Management Bus）协议，是系统管理总线，SMBus是基于I2C总线改建而来的，主要用于电源管理系统中。

I2C外设的框图：

![image-20230113191059347](https://gitee.com/pu-heliang/photos/raw/master/img/202301131910399.png)

上图是STM32内部I2C外设的结构图

左边是外设的通信引脚SDA和SCL，SMBALERT是SMBus用的，这种外设模块引出来的引脚，一般都是借助GPIO口的复用模式与外部世界相连的。

上面的那部分是SDA，数据控制部分，数据收发的核心部分是数据寄存器和数据移位寄存器，当我们需要发送数据时，可以把一个字节数据写到数据寄存器DR，当移位寄存器没没有数据移位时这个数据寄存器的值就会进一步转到移位寄存器里，在移位的过程中把下一个数据放到数据寄存器里等着了，一旦前一个数据移位完成，下一个数据就可以无缝衔接，继续发送，当数据从数据寄存器转运到移位寄存器时，就会置状态寄存器的TXE位为1，表示发送寄存器为空，这是发送的流程，在接收时，输入的数据，一位一位地，从引脚移入到移位寄存器里，当一个字节的数据收齐之后，数据就整体从移位寄存器转到数据寄存器，同时置标志位RXNE，表示接受寄存器非空，这时候就可以把数据从数据寄存器读出来了。I2C是半双工，所以数据收发是同一组数据寄存器和移位寄存器比较器和地址寄存器这是从机模式使用的，STM32的I2C是基于可变多主机模型设计的，STM32不进行通信的时候,就是从机，作为从机，就能被别人召唤，它就应该有从机地址，从机地址由自身地址寄存器指定，可以自定一个从机地址，写到这个自身地址寄存器，当STM32作为从机，被寻址时，如果收到的寻址通过比较器判断，和自身地址相同那STM32就作为从机，响应外部主机的召唤，并且STM32支持同时响应两个从机地址，所以就有自身地址寄存器和双地址寄存器，这里需要再多主机模式下理解，STM32作为从机才需要有这部分，STM32一般是一主多从的模型，STM32不会作为从机，所以这一块就不需要用。右边是STM32设计的数据校验模块，当我们发生多字节的数据帧时，在这里硬件可以自动执行CRC校验计算，CRC是一种常见的数据校验算法，它会根据前面的数据，进行各种数据运算，然后会得到一个字节的校验位，附加在数据帧后面，在接收到这一帧数据后，STM32硬件也可以自动执行校验的判定，如果数据在传输的过程中出错了，CRC校验算法就通不过，硬件就会置校验错误标志位，告诉你数据错了，类似于串口的奇偶校验，也是进行数据有效性验证的。

再看SCL部分，时钟控制是用来控制SCL线的，在时钟控制寄存器写对应位，电路就会执行对应的功能，写入控制寄存器，可以对整个控制逻辑电路进行控制，读取状态寄存器，可以得知电路的工作状态。内部有一些标志位置1后，可以申请中断，如果开启了中断，当事件发生后，程序就可以跳到中断函数来处理这个事件了，在很多字节进行收发时，可以配个DMA提高效率，这些就是I2C外设的框图

基本结构图：

![image-20230113195826630](https://gitee.com/pu-heliang/photos/raw/master/img/202301131958678.png)

移位寄存器和数据寄存器DR的配合是通信的核心部分，因为I2C是高位先行所以，这个移位寄存器是向左移位，在发送的时候，最高位先移出去，然后是次高位，等等，一个SCL时钟，移位一次，移位8次，这样就能把一个字节由高位到低位，依次放到SDA线上了，在接收的时候，数据通过GPIO口从右边依次移进来，最终移8次，一个字节就接受完成了，之后GPIO口使用硬件I2C的时候，这两个对应的GPIO口都要配置成复用开漏输出的模式，复用，就是GPIO口的状态是交由片上外设来控制的，开漏输出，这是I2C协议里要求的端口配置，SCL这里时钟控制器通过GPIO去控制时钟线，这里简化成一主多从的模型，时钟这里只画了输出的方向，实际上，如果是多主机的模型，时钟线也是会进行输入的，这个时钟的输入可以先不管，SDA的部分输出数据，输出到端口，输入数据也是通过GPIO，输入到移位寄存器。最后打开开关控制

#### 主机发送

![image-20230113201052661](https://gitee.com/pu-heliang/photos/raw/master/img/202301132010735.png)

