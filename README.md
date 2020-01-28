# FR_RotaryEncoder
一个Arduino库用于旋转编码器 (型号 KY-040/EC11 或类似)

    "FR_RotaryEncoder" 是一个用于内嵌PUSH开关的机械旋转编码器的Arduino库.
    
    Copyright (c) 2019 by Ilias Iliopoulos info@fryktoria.com

    This file is part of "FR_RotaryEncoder".

    Licensed under GNU General Public License, version 3 of the License
      https://www.gnu.org/licenses/  

    The part related to the debouncing of the rotary encoder is based on
    an idea documented in http://www.technoblogy.com/list?28Y4 
    The article and code is copyrighted as:

      David Johnson-Davies - www.technoblogy.com - 7th July 2018
      Arduino/Genuino Uno   
      CC BY 4.0
      Licensed under a Creative Commons Attribution 4.0 International license: 
      http://creativecommons.org/licenses/by/4.0/  

  =============================================================================

代码支持:

   * 通过中断或循环运行
   * 内嵌旋转编码器旋转运动消抖方法不需设置消抖时间，基于David Johnson-Davies前期工作
   * 读取开关状态并识别长按
   * 设置旋转敏感性

  =============================================================================

  This code is implemented with the Arduino IDE and has been tested with 
  IDE 1.8.10, on an Arduino Nano.  

  代码在KY-040旋转编码器上面测试通过，pins A 和 B 有上拉电阻，pin SW没有上拉, 可参考:
  https://www.aliexpress.com/item/32648520888.html?spm=a2g0s.9042311.0.0.27424c4dMoxkCX 
 
  =============================================================================

#  使用:

0. 包含库

      ```c++
      #include "FR_RotaryEncoder.h"
      ```

1. 创建类实例并指定用于A (SCK), B (DT) 和 PUSH开关 (SW)的针脚。
     把 A 脚设置为支持中断功能的 PIN 2 或 PIN 3 是非常重要的。
     另外把 SW 脚设置为 A 脚不用的 PIN 2 或 3也是非常重要的。 
     B脚可以是任何其他的Arduino I/O 脚。
     下面的类名 `rencoder` 是随意起的。

      ```c++
      RotaryEncoder rencoder(pinSCK, pinDT, pinSW);
      ```
      
     常见Arduino板子的中断资源列表：
     板子型号:					int.0	int.1	int.2	int.3	int.4	int.5	Level
     
     Uno, Mini, Pro, ATmega168, ATmega328		2	3					5v
     
     Mega2560................................. 2      3      21      20     19    18               5v
     
     Leonardo, Micro, ATmega32U4.............. 3      2      0       1      7     x                5v
     
     Digistump, Trinket, ATtiny85............. 2/physical pin 7                                    5v
     
     Due, SAM3X8E............................. all digital pins                                    3v
     
     Zero, ATSAMD21G18........................ all digital pins, except pin 4                      3v
     
     Blue Pill, STM32F103xxxx boards.......... all digital pins, maximun 16 pins at the same time  3v
     
     ESP8266.................................. all digital pins, except gpio6 - gpio11 & gpio16    3v/5v
     
     ESP32.................................... all digital pins                                    3v

2. 检查旋转编码器的线路，默认情况下 , Arduino 输入被设置为 **without** 上拉。
     如果模块有上拉电阻在 PIN A 和 B，你不需要做任何事。但是如果模块没有上拉电阻，
     你必须手动设置使用内部 MCU 上拉。

      ```c++
      rencoder.enableInternalRotaryPullups(); 
      ```

    如果模块 PUSH 开关也没有上拉电阻，你也需要激活内部上拉电阻设置：

      ```c++
      rencoder.enableInternalSwitchPullup(); 
      ```

3. 有上拉电阻并且开关连接到 GND ，稳定状态 (switch not pressed)是 1 ，
     激活状态 (switch pressed) 是 0。如果你的硬件有一个下拉电阻从 PIN 到 Vcc 
     你可以反转逻辑如下：

      ```c++
      rencoder.setSwitchLogic(true);
      ```

      你也可以使用 `rencoder.setRotaryLogic()` 来反转旋转的方向。

4. 设置旋转编码器正负极限值和 wrap-around 模式：

      ```c++
      rencoder.setRotaryLimits(rotaryMinimum, rotaryMaximum, rotaryWrapMode);
      ```

    **rotaryWrap** 模式就是循环。

    **false**: 增加方向永远小于最大值，减少方向永远大于最小值

    需要关注的是这种模式后跟 `setRotationalStep()`并且步进值不为 1. 假设你在
    逆时针方向设置 max=10 并且 min=-9 步进值为 2，位置值应该是 0, -2, -4, -6
    , -8 但下一个步进值应该是比 min小的 -10。 如果被允许取值 -9，顺时针方向随
    后被取值为 -7, -5, -3, -1, 1 等。 这好像不是很令人满意。最小值应该是 -8，
    无需理会小于最小值的事实。

    **true**: 当超过最大值时，位置将被设置为最小值，而不管旋转步进值。另一个方向类似。

5. 你可以任意设置运行参数，详见文档 **FR_RotaryEncoder.h**

    // 设置方向
    // CW = Clockwise 
    // CCW = Counter-clockwise 
    // NOT_MOVED = Not moved since initialization or setPosition()
	int getDirection();

	// 返回编码器当前方向
	int getPosition();

	// 设置起始位置
	void setPosition(int newPosition);

	// 设置编码器最大值
	void setMaxValue(int newMaxValue);

	// 设置编码器最小值
	void setMinValue(int newMinValue);

	// 设置循环模式，如果在同一方向超出最大最小值
    //   true goes from maxValue to minValue
    //   false stays at maxValue or minValue, if continued in the same direction 
	void setWrapMode(bool newWrapMode);

    // 设置旋转敏感度
    // false (default): Requires two clicks per transition
    // true: Requires one click per transition (except the first after setup 
    // which depends on the initial switch position) 
    void setSensitive(bool fast);

    // 设置步进值
    void setRotationalStep(int step);

    // 更新编码器状态
    void rotaryUpdate();

    // PUSH开关

    // 使能内部开关上拉电阻
    void enableInternalSwitchPullup();

    // 设置开关逻辑
    //  true means:  switch ON if pin is 1, OFF if pin is 0
    //  false means: switch OFF if pin is 1, ON if pin is 0
    void setSwitchLogic(bool logic);

    // 设置开关消抖时间，单位ms
    void setSwitchDebounceDelay(unsigned long dd);

    // 返回开关状态
    // See enum SwitchState
    //   SW_OFF OFF
    //   SW_ON ON
    //   SW_LONG Long press
    int getSwitchState(); 

    // 设置长按的时间闸门
    void setLongPressTime(unsigned long longPress);

    // 开关被按压返回TRUE
    bool keyPressed();

    // 返回开关被按压的时间，单位ms
    unsigned long keyPressedTime();

6. 如果你想使用中断，你需要创建自己的中断处理函数，比如：interrupt handling routine(s) or Interrupt Service Routines (ISR)

      ```c++
      // Interrupt handling routine for the rotary
      void ISRrotary() {
         rencoder.rotaryUpdate();
      }
      // Interrupt handling routine for the switch
      void ISRswitch() {
        rencoder.switchUpdate();
      }
      ```

     在 `setup()` 中你必须与中断引脚建立关联，例如：


      ```c++
      attachInterrupt(digitalPinToInterrupt(pinSCK), ISRrotary, CHANGE);
      attachInterrupt(digitalPinToInterrupt(pinSW), ISRswitch, CHANGE);
      ```


     如果你不想使用中断而使用轮询，你可以使用任何 Arduino I/O 引脚而非必须使用 PIN 2 或 3。

     这种情况下你必须把处理编码器的语句置入 `loop()` ，如：

      ```c++
      rencoder.rotaryUpdate();
      ```

     如果你仅仅使用PUSH开关，可以：

      ```c++
      rencoder.switchUpdate();
      ```

     如果你两个都用，可以：

      ```c++
      rencoder.update();
      ```

7. 如果你希望使用长按 **Long Press** 功能，为了识别开关是否被连续按压，或者想通过
       函数`rencoder.keyPressedTime()` 返回保持按压的时间，即便是你使用中断模式，
       也要在 Loop 循环里置入`rencoder.switchUpdate()` 或 `rencoder.update()`。 

The library is accompanied with lots of examples which demonstrate most of the library
  features.   
  
For any issues, you can contact me at <mailto://info@fryktoria.com>       

