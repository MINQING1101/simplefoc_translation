---
layout: default
title: Stepper Driver 4PWM
nav_order: 1
permalink: /stepper_driver_4pwm
parent: StepperDriver
grand_parent: Driver code
grand_grand_parent: Writing the Code
grand_grand_grand_parent: Arduino <span class="simple">Simple<span class="foc">FOC</span>library</span>
---

# Stepper Driver - `StepperDriver4PWM`（步进驱动器）

这个类提供了一个大多数常见的 4PWM 步进驱动器的抽象层。基本上，任何可以使用 4PWM 信号运行的步进驱动器都可以用这个类来表示。
实例：

- L298n
- MX1508
- Shield R3 DC Motor Driver Module
- 等等


<img src="extras/Images/stepper4pwm.png" class="width60">

## Step 1. Hardware setup（步骤1. 硬件设置）
要创建接口到步进驱动器，需要为电机的每个阶段指定 4 `pwm`  引脚编号和为每个相 `en1` 和 `en2` 添加可选的 `enable` 引脚。

```cpp
//  StepperDriver4PWM( int ph1A,int ph1B,int ph2A,int ph2B, int en1 (optional), int en2 (optional))
//  - ph1A, ph1B - phase 1 pwm pins
//  - ph2A, ph2B - phase 2 pwm pins
//  - en1, en2  - enable pins (optional input)
StepperDriver4PWM driver = StepperDriver4PWM(5, 6, 9, 10, 7,  8);
```

## Step 2.1 PWM Configuration（步骤2.1 PWM 配置）
```cpp
// pwm frequency to be used [Hz]
// for atmega328 fixed to 32kHz
// esp32/stm32/teensy configurable
driver.pwm_frequency = 50000;
```
<blockquote class="warning">
⚠️ 基于 ATMega328 芯片的 Arduino  设备固定的 pwm 频率为 32kHz。
</blockquote>

下面是  Arduino <span class="simple">Simple<span class="foc">FOC</span>library</span> 中使用的不同微控制器及其PWM频率和分辨率的列表。

MCU | default frequency（默认频率） | MAX frequency（最大频率） | PWM resolution（分辨率） | Center-aligned（中心对齐） | Configurable freq（可配置的频率） 
--- | --- | --- | --- | --- | --- 
Arduino UNO(Atmega328) | 32 kHz | 32 kHz | 8bit | yes | no
STM32 | 50kHz | 100kHz | 14bit | yes | yes
ESP32 | 40kHz | 100kHz | 10bit | yes | yes
Teensy | 50kHz | 100kHz | 8bit | yes | yes

这些设置都在 library 库的源文件的 `drivers/hardware_specific/x_mcu.cpp/h` 中定义。


## Step 2.2 Voltages（步骤2.2 电压）
驱动器类（Driver class）可以为驱动器输出引脚设置 pwm 占空比，需要知道它插接的直流电源电压。此外，用户能通过驱动器类（Driver class）设置驱动器输出引脚的绝对直流限压 。
```cpp
// power supply voltage [V]
driver.voltage_power_supply = 12;
// Max DC voltage allowed - default voltage_power_supply
driver.voltage_limit = 12;
```

<img src="extras/Images/stepper_limits.png" class="width60">

 `StepperMotor` 类也使用这个参数。 如图所示当限压 `driver.voltage_limit` 设置了，它会和 FOC 算法中的 `StepperMotor` 类通信，相位电压大约是  `driver.voltage_limit/2`。

因此，如果担心电机产生的电流过大，这个参数是非常重要的。在这种情况下，该参数可以当作安全特性来使用。

## Step 2.3 Initialisation（步骤2.3 初始化）
当必要的配置参数都设置好了，就会调用驱动器函数 `init()` 。该函数使用配置参数，并配置驱动器代码执行所需的所有硬件和软件。
```cpp
// driver init
driver.init();
```

## Step 3. Using encoder in real-time（步骤3. 实时使用编码器）

BLDC driver class was developed to be used with the <span class="simple">Simple<span class="foc">FOC</span>library</span> and to provide the abstraction layer for FOC algorithm implemented in the `StepperMotor` class. But the `StepperDriver4PWM` class can used as a standalone class as well and once can choose to implement any other type of control algorithm using the bldc driver.  

## FOC algorithm support
在 FOC 控制下，驱动器的使用是由运动控制算法内部完成的，只需将驱动器连接到  `StepperMotor` 类。
```cpp
// linking the driver to the motor
motor.linkDriver(&driver)
```

## Standalone driver （独立的驱动器）
如果使用 bldc 驱动器作为一个独立的设备，实现操作是很容易的。下面是一个非常简单的应用程序的实例代码。
```cpp
// Stepper driver standalone example
#include <SimpleFOC.h>

// Stepper driver instance
StepperDriver4PWM driver = StepperDriver4PWM(5, 6, 9,10, 7, 8);

void setup() {
  
  // pwm frequency to be used [Hz]
  driver.pwm_frequency = 50000;
  // power supply voltage [V]
  driver.voltage_power_supply = 12;
  // Max DC voltage allowed - default voltage_power_supply
  driver.voltage_limit = 12;
  
  // driver init
  driver.init();

  // enable driver
  driver.enable();

  _delay(1000);
}

void loop() {
    // setting pwm
    // phase 1: 3V, phase 2: 6V
    driver.setPwm(3,6);
}
```