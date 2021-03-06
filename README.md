# Smart Charger Case of AB1532 by STM8S003</br>

TWS耳机充电盒是一个功能比较简单的产品，如果在一个工厂出现十几款充电盒方案，无论从哪一方
面来衡量(业务/成本/生产)都是太多了。减少SKU的数量尽量让单个SKU的产能最大化，这是当初萌
生设计一款充电盒能最大程度满足当前的需求又具有一定的灵活性的原因。</br>

以洛达TWS耳机充电盒方案来探讨。</br>

Docs/ChargerCase_ref.pdf是常见的洛达充电盒低成本MCU方案参考线路图，如果在此线路上UI要
求LED支持呼吸灯显示/支持按键/支持充电设备检测，GPIO口就不够用，满足不了需求。</br>
低成本MCU通常不支持CCP，PWM有的话也仅有一路输出，如果要支持LED呼吸灯功能，支持4个LED时
硬件线路需要5个GPIO口。</br>

## 设计目标</br>
在满足洛达充电盒基本功能需求之外，支持以下需求:</br>
  1.  flash型MCU</br>
  2.  最多支持4个LED</br>
  3.  4路PWM并可独立控制</br>
  4.  支持Hall Sensor</br>
  5.  至少支持一个按键</br>
  6.  支持充电设备检测</br>
  7.  支持NTC温度检测</br>
  8.  5V常在条件下工作电流不大于100uA\</br>

  * **为什么是STM8S003**？</br>
  ST MCU在市场上应用广范，是很多国产MCU厂商学习的对手。STM8S003的价格RMB1.10左右，相
  比于低成本MCU价格估计贵了RMB0.30左右，成本差异在工厂的可接受的范围内(外协方案商供给
  工厂的充电盒MCU在RMB1.20左右)。查阅规格书，评估下来资源满足规划的充电盒的功能需求。
  用其它家的类似MCU也是可以，比如新唐N76E003AT20，据说是与STM8S003 PIN脚兼容。</br>
  对于规模不大的工厂，用一套充电盒方案解决当前的需求及可能存在的需求变化，替换现有的多
  种充电盒方案，以降低业务/开发/仓储成本，具有一定的可行性。</br>
  低成本MCU基本上都是OTP MCU，只能烧录一次，软件无法升级。每家工厂多少都会有一些尾货或
  一定数量的备品，有些时候更改UI就能改成另外一款产品。对于工厂而言肯定会用到多家方案，
  Qualcomm/Airoha/BES等等，如果每个方案都有十来款充电盒，这个数量会很可观，如何来减少
  这种问题的出现，在产品开发的时候就需要考虑到。

  * **为什么PWM输出端口要可独立控制**？</br>
目前在TWS耳机市场，充电盒LED数量4/3/1都有，超过4个LED的可能有但大多数客户的产品最多是
4个。未来充电盒UI是否有客户会提需要LED都是呼吸灯的情况，这个不太确定，如果MCU支持CCP能
同时输出4路PWM或更多路PWM，基本上能满足可能出现的UI需求变化。

  * **为什么要支持Hall Sensor**？</br>
目前所接触到的洛达TWS耳机方案在电商客户群体的趋势是充电盒开盖开机，有些客户UI要求左右耳
同步关机，如果有一个耳机电量不足时放入充电盒充电在未关盖时另一个耳机还能继续使用。

  * **为什么充电盒要支持按键功能**？</br>
大多数TWS耳机在销售到终端用户手中时没有TWS配对功能(工厂在生产时配对好出货)，用户端存在
的TWS配对需求通过按键功能来实现。

  * **为什么要支持充电设备检测**？</br>
充电盒的充电状态是由充电芯片的充电状态脚来获得，在未充电和充满电的状态下，充电芯片的状态
脚的L或H状态是相同的，如果要得到充电盒充满电的状态，需要知道充电设备是否存在。

  * **为什么要支持NTC温度检测**？</br>
有些充电芯片虽然支持NTC温度监控，但只在充电状态下有效，在非充电状态下不会去监控温度。出
口欧盟产品认证的具体要求到底是充电温度监控还是工作状态温度监控，尚未查到确切的资料，以防
万一先预留一个ADC口做温度监控。

  * **为什么要定义功耗需求**？</br>
洛达方案充电盒UI要求通常是在关盖时耳机充满电5V才关闭，此时充电盒进入低功耗状态(MCU进入
低电状态)，充电盒功耗在此条件下电流大多数都能做到20uA以下。在5V常在条件下未放入耳机时
工作电流普遍都在1mA左右，有的甚至能达到2mA以上，对于充电盒本身电池只有600mAH左右的容量
而言，无论是MCU本身资源的限制还是其它原因，功耗都太大，可能放一晚上电池盒就处于低电状态
了。</br>
查阅STM8S003规格书，在HALT模式下5V供电电流小于10uA；在16MHz主频，5V供电条件下，ADC模
块工作电流1mA, 软件可通过定时唤醒的方式降低工作电流。16MHz主频下STM8S003工作电流3--4
mA; 8MHz主频下工作电流1--2mA, 实际验证后8MHz主频已经可以满足功能需求。在8MHz主频下实
际测试ADC模块(扫瞄4个AD口)在间隔1S唤醒条件下工作电流20--30uA, 将间隔唤醒的时间加长可以
获得更低的功耗，对于设计目标小于100uA的要求MCU端可以满足。</br>
ETA9697实际测试功耗时电池电压在3.30V--4.10V时输出5V，工作电流均可维持在10uA以内。再加
上洛达耳机方案需要的充电盒外围线路功耗在20uA左右，元器件质量对此功耗有很大影响，曾经硬件
工程师在充电盒的POGO PIN端加了几个TVS管，功耗直接奔mA级。</br>
整个线路评估下来工作电流可控制在70uA左右。实际在飞线搭原型机上测试在5V常在未放入耳机时充
电盒工作电流可控制在60--70uA。</br>

  &ensp; ![chgcase_if](https://i.loli.net/2020/08/21/P3Xr5TRiD6EHqom.png)</br>

上面这部分线路在设计时是想用XC8108来替换的(如下图所示)。

  &ensp; &ensp; &ensp; &ensp;&ensp;   ![opt_xc8108](https://i.loli.net/2020/08/21/pYotXVFPIfyCMhU.png)</br>

因为洛达耳机方案充电盒在单线通讯发送命令时会有轻微的"吱吱"噪音 (这个噪音的大小取决于控制线路的匹配，无法根除)。因为XC8108具有逆流降止功能，控制XC8108的CE脚关断输出时不会对充电芯片造成影响，想利用它的这个特性来实现充电盒的单线通讯功能。

XC8108的开关时延如下表所示：</br>

&ensp; &ensp; ![turn_onoff](https://i.loli.net/2020/08/21/SOW5dyrhoYHU4m6.png)

CE控制脚的电压升高对开关时延是否会有改善在规格书上未有明确说明，如果充电盒通讯命令协议是以1ms单位来计时，用XC8108来替代控制线路可能需要修改协议，后面如果有机会再来完善这部分功能的评估。

具体线路图可查看Docs/ChargerCase_stm8.pdf，STM8S003洛达耳机充电盒线路图，线路图上分配的GPIO功能定义是目前认为最佳的选择(STM8S003 GPIO的复用功能还是有坑要填的)，各个GPIO相应的功能在STM8S003/STM8S103F3P均可实现。


## LED Gammar Correction</br>

呼吸灯通常是将PWM占空比从0加到100，再从100减到0，往复变化。这样做的缺点是LED显示在视觉上是突然间变亮或变暗，因此在LED显示PWM调光中引入Gamma Correction，来获得比较满意的灰度变化(线性渐变)。</br>

&ensp; Gamma曲线</br>
&ensp; ![gamma_curve](https://i.loli.net/2020/08/21/2vhMTdngiIUDtRf.png)</br>

&ensp;&ensp;  ( 关于Gamma的意义，可以参阅以下文章：</br>
&ensp;&ensp;&ensp;  **色彩校正中的gamma值是什么** </br>
&ensp;&ensp;&ensp;  https://www.zhihu.com/question/27467127</br>
&ensp;&ensp;&ensp;)

&ensp; 灰度等级</br>
&ensp; ![gray_lvl](https://i.loli.net/2020/08/21/1pYxZtKVITnbJMi.png)

取： Output = Input ^ (1/Gamma)</br>
&ensp;&ensp;&ensp;&ensp;     (Gamma = 2.2)\</br>

如果直接将数学公式引入代码中去计算，STM8S003主频最高才16MHz，运算量太大估计跑不动。

换种思路：Output是我们希望的线性渐变的灰度的百分比值 (从0到100%，简化计算，只取整数</br>
&ensp; &ensp; &ensp; &ensp; &ensp; &ensp;&ensp; 值0--100)，Input是输入的PWM占空比，因为PWM周期未定，无法直接计算得到</br>
&ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp;PWM占空比数值。偿试以0--100的数值代入Input求得Output, 再观察Input与Output</br>
&ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp;数值的变化关系,然后再将Input数值逐渐加大，当Output数值为1时的Input值就是我们</br>
&ensp; &ensp; &ensp; &ensp; &ensp; &ensp; &ensp;想要得到的PWM周期值，由此就得到一个0--100%的table表。</br>

```c
/**
  * gamma = 2.2
  * Output = Input^(1/gamma)
  */
CONST uint16_t gamma_table[100] = {
	    0,     1,     5,    11,    21,    35,    51,    72,    97,   126,
	  158,   195,   237,   282,   332,   387,   446,   509,   577,   650,
	  728,   811,   899,   991,  1088,  1189,  1297,  1409,  1526,  1649,
	 1777,  1910,  2048,  2191,  2340,  2494,  2654,  2819,  2989,  3165,
	 3346,  3533,  3725,  3923,  4127,  4336,  4551,  4771,  4997,  5229,
	 5467,  5710,  5960,  6215,  6475,  6742,  7015,  7293,  7578,  7867,
	 8165,  8467,  8775,  9090,  9410,  9737, 10069, 10408, 10753, 11104,
	11461, 11824, 12194, 12569, 12951, 13339, 13734, 14135, 14542, 14955,
	15375, 15801, 16233, 16672, 17117, 17568, 18026, 18490, 18961, 19438,
	19922, 20412, 20909, 21412, 21922, 22439, 22962, 23491, 24027, 24570,
  /*	25119,*/
};
```

这个table表是否能为STM8S003所用？</br>
STM8S003输出4路PWM只能选用TIM1, 16位自动重载定时器，PWM最大脉冲宽度25119 < 2^16, 只要将TIM1自动重载值设为大于或等于PWM最大脉冲宽度，就能实现PWM占空比从0--100%变化。在范例代码中只取了table表中的0--99。</br>

## Code
### stm8s_tim1_pwm.rar

@par Example Description</br>

This example shows how to configure the TIM1 peripheral to generate 4 PWM signals with different duty cycles by gamma_table value.

This example provides a short description of how to use the TIM1 PWM peripheral:  
Change the macro define in chgcase_cfg.h listed as below before compile,
TIM1_CH [1:4] will output corresponding PWM pulse.</br>

```c
#define GUI_ALL_ON                1//fixed pwm duty
//#define GUI_MARQUEE             1//marquee control
//#define GUI_LUMOS               1//breathing light
```

@par How to use it?</br>
In order to make the program work, you must do the following:
+ Copy all source files from this example folders INC and SRC to your target disk
+ Open your toolchain EWSTM8 and create a new project add all files you copied
+ Configure your project options:
  - General Options -> Target -> Device: STM8S003F3 or STM8S103F3P
  - C/C++ Compiler -> Preprocessor -> Additional include dirctories: $PROJ_DIR$\INC
  - C/C++ Compiler -> Preprocessor -> Defined symbol: STM8S003 or STM8S103
  - Debugger -> Setup -> Driver: ST-Link
+ Rebuild all files and load your image into target memory
+ Run the example

@par HINT</br>
- Before using TIM1_CH3 and TIM1_CH4 you have to configure option bytes in order
  to enable alternate function.</br>
- To do so</br>
  - With EWSTM8 (menu: ST-Link -> Option bytes -> AFR0 = Alternate Active)

@notes
- STM8S Standard Peripheral Library: STM8S_StdPeriph_Driver V2.3.0
- IAR Embedded Workbench for STM8 IDE (EWSTM8)  
 - Version 3.10.4

PS :</br>
 &ensp;&ensp;It is too expensive of ST'mcu at now 2021 that there are no results for COVID-19.
