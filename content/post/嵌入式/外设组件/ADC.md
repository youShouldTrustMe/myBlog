---
title: ADC

date: 2025-03-01
lastmod: 2025-03-01
cover: https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_7_4_202407090907138.png
category: 嵌入式
tags:
- 外设组件
---



# ADC简介

## 介绍

•ADC（Analog-Digital Converter）模拟-数字转换器

•ADC可以将引脚上连续变化的模拟电压转换为内存中存储的数字变量，建立模拟电路到数字电路的桥梁

•12位逐次逼近型ADC，1us转换时间

•输入电压范围：0~3.3V，转换结果范围：0~4095

•18个输入通道，可测量16个外部和2个内部信号源

•规则组和注入组两个转换单元

•模拟看门狗自动监测输入电压范围

•STM32F103C8T6 ADC资源：ADC1、ADC2，10个外部输入通道

![image-20240709090704068](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_7_4_202407090907138.png)

# 原理

## 常见ADC类型

| **ADC电路类型** | **优点**         | **缺点**                 |
| --------------- | ---------------- | ------------------------ |
| **并联比较型**  | 转换速度最快     | 成本高、功耗高，分辨率低 |
| **逐次逼近型**  | 结构简单，功耗低 | 转换速度较慢             |

## 并联比较型

将输入的参考电压和模拟电压输入进行比较，比较器会输出0或1从而获得二进制的数字量

![image-20240709090821671](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_8_21_202407090908750.png)

## 逐次逼近型

![image-20240709090844769](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_8_44_202407090908861.png)

![image-20240709090850698](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_8_50_202407090908781.png)

# 特性

## 特性参数

![image-20240709091013333](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_10_13_202407090910385.png)

## 各系列主要特性

| **主要特性**    | **F1**                                               | **F4**       | **F7**      | **H7**          |
| --------------- | ---------------------------------------------------- | ------------ | ----------- | --------------- |
| **ADC类型**     | 逐次逼近型                                           |              |             |                 |
| **分辨率**      | 12位                                                 | 6/8/10/12位  | 6/8/10/12位 | 8/10/12/14/16位 |
| **ADC时钟频率** | 14MHz（max）                                         | 36MHz（max） |             |                 |
| **采样时间**    | 采样时间越长, 转换结果相对越准确, 但是转换速度就越慢 |              |             |                 |
| **转换时间**    | 与ADC时钟频率、分辨率和采样时间等有关                |              |             |                 |
| **供电电压**    | VSSA ：0V，VDDA ：2.4V~3.6V（全速运行）              |              |             |                 |
| **参考电压**    | VREF– ：0V，VREF+一般为3.3V                          |              |             |                 |
| **输入电压**    | VREF–≤VIN≤VREF+                                      |              |             |                 |

# 整体结构

## 结构框图

![image-20240709091106271](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_11_6_202407090911343.png)

![image-20240709095945579](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_59_45_202407090959699.png)

## 参考电压/模拟部分电压

ADC所能测量的电压范围是Vref- < VIN < Vref+,如果把VSSA和VREF-接地，把VREF+和VDDA接3V3，得到ADC的输入范围为0-3.3v

![image-20240709091152871](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_11_52_202407090911928.png)

## 输入通道

ADC的信号输入通过通道来实现的，信号通过通道输入到单片机中，单片机经过转换后，将模拟信号转换为数字信号。Stm32中的ADC有18个通道，其中外部16个通道已经在图中标出，这16个通道对应着不同的IO口，除此之外ADC1/2/3还有内部通道。

| **ADC1** | **IO**             | **ADC2** | **IO**      | **ADC3** | **IO**      |
| -------- | ------------------ | -------- | ----------- | -------- | ----------- |
| 通道0    | PA0                | 通道0    | PA0         | 通道0    | PA0         |
| 通道1    | PA1                | 通道1    | PA1         | 通道1    | PA1         |
| 通道2    | PA2                | 通道2    | PA2         | 通道2    | PA2         |
| 通道3    | PA3                | 通道3    | PA3         | 通道3    | PA3         |
| 通道4    | PA4                | 通道4    | PA4         | 通道4    | PF6         |
| 通道5    | PA5                | 通道5    | PA5         | 通道5    | PF7         |
| 通道6    | PA6                | 通道6    | PA6         | 通道6    | PF8         |
| 通道7    | PA7                | 通道7    | PA7         | 通道7    | PF9         |
| 通道8    | PB0                | 通道8    | PB0         | 通道8    | PF10        |
| 通道9    | PB1                | 通道9    | PB1         | 通道9    | 连接内部VSS |
| 通道10   | PC0                | 通道10   | PC0         | 通道10   | PC0         |
| 通道11   | PC1                | 通道11   | PC1         | 通道11   | PC1         |
| 通道12   | PC2                | 通道12   | PC2         | 通道12   | PC2         |
| 通道13   | PC3                | 通道13   | PC3         | 通道13   | PC3         |
| 通道14   | PC4                | 通道14   | PC4         | 通道14   | 连接内部VSS |
| 通道15   | PC5                | 通道15   | PC5         | 通道15   | 连接内部VSS |
| 通道16   | 连接内部温度传感器 | 通道16   | 连接内部VSS | 通道16   | 连接内部VSS |
| 通道17   | 连接内部Vrefint    | 通道17   | 连接内部VSS | 通道17   | 连接内部VSS |

外部的16个通道在转换时又分为规则通道和注入通道，其中规则通道最多有16路，注入通道最多有4路

### 规则通道

规则通道是用于普通的周期性 ADC 数据采集的通道。它们适用于周期性地采集模拟信号，例如传感器测量、输入信号的连续转换等。在规则通道中，可以设置 ADC 转换的采样时间、分辨率等参数。

### 注入通道

注入通道是用于触发式或突发式 ADC 数据采集的通道。它们适用于在特定事件或条件下进行数据采集，例如外部触发或中断事件。注入通道可以用于实现对不同事件的响应并采集模拟信号。在注入通道中，您可以设置触发源、采样时间等参数。

## 转换顺序

知道了ADC的转换通道后，如果ADC只使用一个通道来转换，那就很简单，但如果是使用多个通道进行转换就涉及到一个先后顺序了，毕竟规则转换通道只有一个数据寄存器。所以将转换的顺序分组，每组转换的顺序组成一个序列。

多个通道的使用顺序分为俩种情况：

- 规则通道的转换顺序

- 注入通道的转换顺序

这里的规则组只有一个寄存器，所以说当数据转换完成之后就需要尽快转移数据，防止数据覆盖。注入组有四个寄存器，可以将转换的数据存在各自的寄存器中。

### 规则通道转换顺序

规则通道中的转换顺序由三个寄存器控制：SQR1、SQR2、SQR3，它们都是32位寄存器。SQR寄存器控制着转换通道的数目和转换顺序，只要在对应的寄存器位SQx中写入相应的通道，这个通道就是第x个转换。具体的对应关系如下

![image-20240709091418307](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_14_18_202407090914364.png)

### 优先级比较

注入组优先级高于规则组，注入组的转换可以打断规则组。

![image-20240709091446952](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_14_47_202407090914012.png)

### 规则序列

![image-20240709091508334](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_15_8_202407090915402.png)

| **规则序列寄存器控制关系汇总** |                |                            |          |
| ------------------------------ | -------------- | -------------------------- | -------- |
| 寄存器                         | 寄存器位       | 功能                       | 取值     |
| SQR3                           | SQ1 [ 4 : 0 ]  | 设置第1个转换的通道        | 通道0~17 |
|                                | SQ2 [ 4 : 0 ]  | 设置第2个转换的通道        | 通道0~17 |
|                                | SQ3 [ 4 : 0 ]  | 设置第3个转换的通道        | 通道0~17 |
|                                | SQ4 [ 4 : 0 ]  | 设置第4个转换的通道        | 通道0~17 |
|                                | SQ5 [ 4 : 0 ]  | 设置第5个转换的通道        | 通道0~17 |
|                                | SQ6 [ 4 : 0 ]  | 设置第6个转换的通道        | 通道0~17 |
| SQR2                           | SQ7 [ 4 : 0 ]  | 设置第7个转换的通道        | 通道0~17 |
|                                | SQ8 [ 4 : 0 ]  | 设置第8个转换的通道        | 通道0~17 |
|                                | SQ9 [ 4 : 0 ]  | 设置第9个转换的通道        | 通道0~17 |
|                                | SQ10 [ 4 : 0 ] | 设置第10个转换的通道       | 通道0~17 |
|                                | SQ11 [ 4 : 0 ] | 设置第11个转换的通道       | 通道0~17 |
|                                | SQ12 [ 4 : 0 ] | 设置第12个转换的通道       | 通道0~17 |
| SQR1                           | SQ13 [ 4 : 0 ] | 设置第13个转换的通道       | 通道0~17 |
|                                | SQ14 [ 4 : 0 ] | 设置第14个转换的通道       | 通道0~17 |
|                                | SQ15 [ 4 : 0 ] | 设置第15个转换的通道       | 通道0~17 |
|                                | SQ16 [ 4 : 0 ] | 设置第16个转换的通道       | 通道0~17 |
|                                | SQL [ 3 : 0 ]  | 设置规则序列要转换的通道数 | 0~15     |

### 注入序列

和规则通道转换顺序的控制一样，注入通道的转换也是通过注入寄存器来控制，只不过只有一个JSQR寄存器来控制，控制关系如下

需要注意的是，只有当JL=4的时候，注入通道的转换顺序才会按照JSQ1、JSQ2、JSQ3、JSQ4的顺序执行。当JL<4时，注入通道的转换顺序恰恰相反，也就是执行顺序为：JSQ4、JSQ3、JSQ2、JSQ1。

| **注入序列寄存器控制关系汇总** |                |                            |          |
| ------------------------------ | -------------- | -------------------------- | -------- |
| 寄存器                         | 寄存器位       | 功能                       | 取值     |
| JSQR                           | JSQ1 [ 4 : 0 ] | 设置第1个转换的通道        | 通道0~17 |
|                                | JSQ2 [ 4 : 0 ] | 设置第2个转换的通道        | 通道0~17 |
|                                | JSQ3 [ 4 : 0 ] | 设置第3个转换的通道        | 通道0~17 |
|                                | JSQ4 [ 4 : 0 ] | 设置第4个转换的通道        | 通道0~17 |
|                                | JL [ 1 : 0 ]   | 设置注入序列要转换的通道数 | 0~3      |

## 触发源

ADC转换的输入、通道、转换顺序都已经说明了，但ADC转换是怎么触发的呢？就像通信协议一样，都要规定一个起始信号才能传输信息，ADC也需要一个触发信号来实行模/数转换。

1、通过直接配置寄存器触发，通过配置控制寄存器CR2的ADON位，写1时开始转换，写0时停止转换。在程序运行过程中只要调用库函数，将CR2寄存器的ADON位置1就可以进行转换。

![image-20240709091632310](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_16_32_202407090916381.png)

![image-20240709091639763](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_16_39_202407090916824.png)

![image-20240709091646323](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_16_46_202407090916364.png)

2、通过内部定时器或者外部IO触发转换，也就是说可以利用内部时钟让ADC进行周期性的转换，也可以利用外部IO使ADC在需要时转换，具体的触发由控制寄存器CR2决定。

![image-20240709091703278](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_17_3_202407090917331.png)

## 转换时间

ADC的每一次信号转换都要时间，这个时间就是转换时间，转换时间由输入时钟和采样周期来决定。

### **输入时钟**

由于ADC在STM32中是挂载在APB2总线上的，所以ADC得时钟是由PCLK2（72MHz）经过分频得到的，分频因子由 RCC 时钟配置寄存器RCC_CFGR 的位 15:14 ADCPRE[1:0]设置，可以是 2/4/6/8 分频，一般配置分频因子为8，即8分频得到ADC的输入时钟频率为9MHz。

![image-20240709091746501](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_17_46_202407090917566.png)

### 采样周期

采样周期是确立在输入时钟上的，配置采样周期可以确定使用多少个ADC时钟周期来对电压进行采样，采样的周期数可通过 ADC采样时间寄存器 ADC_SMPR1 和 ADC_SMPR2 中的 SMP[2:0]位设置，ADC_SMPR2 控制的是通道 0~9， ADC_SMPR1 控制的是通道 10~17。每个通道可以配置不同的采样周期，但最小的采样周期是1.5个周期，也就是说如果想最快时间采样就设置采样周期为1.5.

### 转换时间

转换时间=采样时间+12.5个周期

12.5个周期是固定的，一般我们设置 PCLK2=72M，经过 ADC 预分频器能分频到最大的时钟只能是 12M，采样周期设置为 1.5 个周期，算出最短的转换时间为 1.17us。

![image-20240709091819636](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_18_19_202407090918693.png)

## 数据寄存器

当使用ADC独立模式（也就是只使用一个ADC，可以使用多个通道）时，数据存放在低16位中，当使用ADC多模式时高16位存放ADC2的数据。需要注意的是ADC转换的精度是12位，而寄存器中有16个位来存放数据，所以要规定数据存放是左对齐还是右对齐。

当使用多个通道转换数据时，会产生多个转换数据，然鹅数据寄存器只有一个，多个数据存放在一个寄存器中会覆盖数据导致ADC转换错误，所以我们经常在一个通道转换完成之后就立刻将数据取出来，方便下一个数据存放。一般开启DMA模式将转换的数据，传输在一个数组中，程序对数组读操作就可以得到转换的结果。

![image-20240709091844265](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_18_44_202407090918344.png)

注入通道转换的数据寄存器有4个，由于注入通道最多有4个，所以注入通道转换的数据都有固定的存放位置，不会跟规则寄存器那样产生数据覆盖的问题。 ADC_JDRx 是 32 位的，低 16 位有效，高 16 位保留，数据同样分为左对齐和右对齐，具体是以哪一种方式存放，由ADC_CR2 的 11 位 ALIGN 设置。

![image-20240709091903918](C:\Users\ryf\AppData\Roaming\Typora\typora-user-images\image-20240709091903918.png)

![image-20240709091910136](C:\Users\ryf\AppData\Roaming\Typora\typora-user-images\image-20240709091910136.png)

## 中断

![image-20240709091927948](C:\Users\ryf\AppData\Roaming\Typora\typora-user-images\image-20240709091927948.png)

![image-20240709091933514](C:\Users\ryf\AppData\Roaming\Typora\typora-user-images\image-20240709091933514.png)

从框图中可以知道数据转换完成之后可以产生中断，有三种情况：

**规则通道转换完成中断**

规则通道数据转换完成之后，可以产生一个中断，可以在中断函数中读取规则数据寄存器的值。这也是单通道时读取数据的一种方法。

**注入通道转换完成中断**

注入通道数据转换完成之后，可以产生一个中断，并且也可以在中断中读取注入数据寄存器的值，达到读取数据的作用。

**模拟看门狗事件**

当输入的模拟量（电压）不再阈值范围内就会产生看门狗事件，就是用来监视输入的模拟量是否正常。

以上中断的配置都由ADC_SR寄存器决定：

![image-20240709092612778](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_26_12_202407090926838.png)

## 电压转换



要知道，转换后的数据是一个12位的二进制数，我们需要把这个二进制数代表的模拟量（电压）用数字表示出来。比如测量的电压范围是0~3.3V，转换后的二进制数是`x`，因为12位ADC在转换时将电压的范围大小（也就是3.3）分为4096（2^12）份，所以转换后的二进制数x代表的真实电压的计算方法就是：
$$
y=\frac{3.3 \times x}{4096}
$$



## 单次转换和连续转换

![image-20240709092032414](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_20_32_202407090920470.png)

一组只转换一个通道

![image-20240709092046905](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_20_46_202407090920976.png)

连续转换，在一轮转换结束后，继续下一轮的转换

![image-20240709092102096](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_21_2_202407090921174.png)

每次转换后都会停下来，但是和非扫描模式不同的是，扫描模式会扫描序列，然后对列表中的序列转换出来

![image-20240709092120818](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_21_20_202407090921890.png)

连续转换，扫描模式

![image-20240709092134632](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_21_34_202407090921711.png)

扫描模式

![image-20240709092149036](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_21_49_202407090921081.png)

## 不同模式的组合

![image-20240709092208802](https://gitlab.com/18355291538/picture/-/raw/main/pictures/2024/07/9_9_22_8_202407090922859.png)

# 实验

使用ADC的步骤

电位器作为ADC输入

1. 确定ADC挂载在哪个总线上，并开启ADC的时钟
2. 配置与ADC通道相对应的GPIO引脚为模拟输入模式
3. 配置规则组通道，包括通道号、顺序和采样时间
4. 设置ADC的工作模式、数据对齐方式 、外部触发源、通道数量等参数
5. 使能ADC
6. 校准ADC

```c
void adc_init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC3,ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOF,ENABLE);
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	GPIO_InitTypeDef GPIO_InitStruct;
	GPIO_InitStruct.GPIO_Pin = ADC_PIN;
	GPIO_InitStruct.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStruct.GPIO_Speed = GPIO_Speed_50MHz;
	
	GPIO_Init(ADC_PORT,&GPIO_InitStruct);

	ADC_RegularChannelConfig(ADC3,ADC_Channel_4,1,ADC_SampleTime_55Cycles5);	//配置规则通道组，填充菜单列表
	
	ADC_InitTypeDef ADC_InitStruct;
	ADC_InitStruct.ADC_ContinuousConvMode = DISABLE;
	ADC_InitStruct.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStruct.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;	//软件触发
	ADC_InitStruct.ADC_Mode = ADC_Mode_Independent;
	ADC_InitStruct.ADC_NbrOfChannel = 1;
	ADC_InitStruct.ADC_ScanConvMode = DISABLE;
	ADC_Init(ADC3,&ADC_InitStruct);
	
	ADC_Cmd(ADC3,ENABLE);
	
	//开始校准
	ADC_ResetCalibration(ADC3);	//复位校准
	while(ADC_GetResetCalibrationStatus(ADC3) != RESET);	//等待复位校准完成
	ADC_StartCalibration(ADC3);	//开始校准
	while(ADC_GetCalibrationStatus(ADC3) == SET);//等待校准完成

}

u16 ad_getValue(void)
{
	ADC_SoftwareStartConvCmd(ADC3,ENABLE);
	while(ADC_GetFlagStatus(ADC3,ADC_FLAG_EOC) == RESET);	//等待转换完成，也就是EOC信号
	return ADC_GetConversionValue(ADC3);	//读取完数据将会自动清除数据位
}


int main(void){
	
	char str[50];
    
	serial_init(); 
	adc_init();	
	
	while(1){
		sprintf(str,"read: %u\n ",ad_getValue());
		send_string(str);
		Delay_ms(1000);
	}
}

```

