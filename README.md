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

    The tricky part is when this mode is used along with `setRotationalStep()`
              and the step is set to a value different than 1. Suppose you have set the 
              limits to max=10 and min=-9 and the step to 2. In the counter-clockwise direction,
              the position may receive values 0, -2, -4, -6, -8 but the next step 
              would be -10 which is lower than min. If it was allowed to take the value -9,
              the clockwise direction would subsequently take values -7, -5, -3, -1, 1 etc. which
              is probably not desirable. Therefore, the lowest position will be -8,
              regardless of the fact that the min is even lower.

    **true**: After increasing over the maximum value, the position will be set to the minimum 
             value, regardless of the setting of the rotational step. Similarly for the other
             direction.

5. You can set any of the operating parameters using the methods documented in **FR_RotaryEncoder.h**

6. If you wish to use interrupts, you must create your interrupt handling routine(s) or Interrupt Service Routines (ISR), such as:

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

     In `setup()` you must associate each interrupt pin with the 
        relevant interrupt handling routine, such as:


      ```c++
      attachInterrupt(digitalPinToInterrupt(pinSCK), ISRrotary, CHANGE);
      attachInterrupt(digitalPinToInterrupt(pinSW), ISRswitch, CHANGE);
      ```


     If you do not want to use interrupts, but use the polling method instead, please note
        that you are not limited to use pins 2 and 3. Any other Arduino I/O pin will do.

     In such a case, you must place in the `loop()` the routines that handle the encoder. 
        If you only use the rotary part and not the switch, you can use:

      ```c++
      rencoder.rotaryUpdate();
      ```

     If you only use the switch, use:

      ```c++
      rencoder.switchUpdate();
      ```

     If you use both, you can do  both jobs at the same time by simply using:

      ```c++
      rencoder.update();
      ```

7. If you wish to use the **Long Press** functionality, in order to identify if the 
       switch has been pressed continouously, or to retrieve the time that the switch 
       remains pressed with `rencoder.keyPressedTime()`, you need to place `rencoder.switchUpdate()`
       or `rencoder.update()` in the loop, **even if your code uses interrupts**. It
       becomes questionable, of course, why you should continue to use such a 
       mixed interrupt/polling solution, but every design is different and, in any case,
       the library supports it. 

The library is accompanied with lots of examples which demonstrate most of the library
  features.   
  
For any issues, you can contact me at <mailto://info@fryktoria.com>       

