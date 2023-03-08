板载功能  
======================================================  


按键  
******************************************************
板载两个用户按键 button1 和 button2，按键按下为低电平，弹起为高电平。 

应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::
    
    '''
     导入  button1, button2 模块 
    '''
    from openaie import button1, button2
    
    '''
     方法：检查按键是否按下
     返回值：
        按键按下 -- True 
        按键弹起 -- False
    ''' 
    button1.is_press() 
    button2.is_press() 
    

案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 按键检测** 

::

    import time 
    from openaie import button1, button2

    while (True):
        if button1.is_press(): # 检测到按键按下
            time.sleep_ms(10) # 延时消抖
            if button1.is_press():
                print("button 1 press")
            while (button1.is_press()) : # 等待按键释放
                pass
        if button2.is_press(): # 检测到按键按下
            time.sleep_ms(10) # 延时消抖
            if button2.is_press():
                print("button 2 press")
            while (button2.is_press()) : # 等待按键释放
                pass
				
.. Note:: 机械按键在按下瞬间会存在一定的抖动，此时按键连接引脚读取的电平状态不稳定，可通过一定的延时略过此时的状态。
    
------------------------------------------------------
      

 
蜂鸣器 
******************************************************
板载无源蜂鸣器，可通过不同频率的信号驱动，发出不同音调的声音。

应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++
    
::

    '''
     导入 buzzer 模块 
    '''
    from openaie import buzzer
    
    ''' 
     方法：蜂鸣器鸣响
     参数：
        freq -- 频率
    ''' 
    buzzer.tone(freq) 

    ''' 
     方法：关停蜂鸣器
    ''' 
    buzzer.no_tone()    

         
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 鸣响测试**

::

    import time
    from openaie import buzzer
    
    tone_list = (289, 661, 700, 786, 882, 990, 1112) # Do、Re、Mi、Fa、Sol、La、Si
    for i in range(7):    
        buzzer.tone(tone_list[i])
        time.sleep_ms(500)
    buzzer.no_tone()
    
------------------------------------------------------

 
液晶显示屏
****************************************************** 
板载2.4英寸液晶显示屏，分辨率：240*320(QVGA)。


应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 lcd 模块 
    '''
    import lcd
    
    '''
     方法：初始化
    ''' 
    lcd.init()
    
    ''' 
     方法：清空显示
     参数：
        color: 颜色 -- 元组类型(r,g,b)，值:0~255；或整数：0~0xFFFFFF。
    
     Example:
        lcd.clear(color=(0,0,255)) # 元组表示蓝色
        # or
        lcd.clear(color=0x0000FF)  # 16进制表示蓝色
    ''' 
    lcd.clear(color)
    
    '''
     方法：设置显示方向
     参数：
        r: 0, 1, 2, 3 
           0,2 为竖屏显示，此时显示屏的宽为240，高为320； 
           1,3 为横屏显示，此时显示屏的宽为320，高为240。
    '''
    lcd.rotation(r)
    
    '''
     方法：设置背光亮度
     参数：
        val: 0~100
    '''
    lcd.set_backlight(val)
    
    '''
     方法：画字符 
     参数：
        x, y: 起始位置
        string: 字符串（支持中文，暂不支持中文状态下的标点符号。）
        fc: 字体颜色 
        bc: 背景色
    '''
    lcd.draw_string(x, y, string, fc=(r,g,b), bc=(r,g,b))
    
    '''
     方法：画线 
     参数：
        x0, y0: 起点位置
        x1, y1: 终点位置
        color: 线颜色，元组类型 r,g,b：0~255
        thickness: 线宽
    '''
    lcd.draw_line(x0, y0, x1, y1, color=(r,g,b), thickness=1)

    '''
     方法：画矩形 
     参数：
        x, y: 起始位置
        w, h：宽和高
        color: 线颜色，元组类型 r,g,b：0~255
        thickness: 线宽
        fill: 是否填充
    '''
    lcd.draw_rectangle(x, y, w, h, color=(r,g,b), thickness=1, fill=0)

    '''
     方法：画圆 
     参数：
        x, y: 圆心位置
        radius：半径
        color: 线颜色，元组类型 r,g,b：0~255
        thickness: 线宽
        fill: 是否填充
    '''
    lcd.draw_circle(x, y, radius, color=(r,g,b), thickness=1, fill=0)
    
    '''
     方法：显示，将显示缓存数据输出显示 
    '''
    lcd.display()
   

案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 绘图测试**
::

    # 导入显示模块
    import lcd

    # 设置背光亮度
    lcd.set_backlight(80)
    # 清空显示
    lcd.clear(color=(0,0,0))
    # 设置显示方向
    lcd.rotation(0)
    # 画线
    lcd.draw_line(10, 10, 80, 80, color=(0,255,0), thickness=5)
    # 画矩形
    lcd.draw_rectangle(100, 20, 80, 60, color=(255,0,0), thickness=8, fill=0)
    # 画圆形，实心
    lcd.draw_circle(120, 160, 30, color=(0,0,255), thickness=1, fill=1)
    # 显示字符
    lcd.draw_string(100, 200, "lcd draw test", fc=(0,255,0), bc=(0,0,0))
    # 输出显示 
    lcd.display()
 
------------------------------------------------------