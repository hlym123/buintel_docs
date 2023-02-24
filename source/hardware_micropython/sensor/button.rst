按键组 
======================================================
按键按下为低电平，弹起为高电平。

.. figure:: button.png 
   :width: 200
   :align: center
   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 button_group 模块 
    '''
    from openaie import button_group

    '''
     类：按键组 
     参数:
        port: 端口号 -- 1~8 
    '''
    class button_group(port)
    
    '''
     方法：检查按键是否按下
     参数：
        index: 1或2，对应按键1或按键2
     返回值：
        按键按下 -- True 
        按键弹起 -- False
    ''' 
    button_group.is_press(index) 
    

案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 按键状态检测**

::

    import time 
    from openaie import button_group # 导入按键模块
    
    bt2 = button_group(2) # 按键模块连接到端口2 
    while True:
        if bt2.is_press(1) :
            print("press")
        else:
            print("release")
        time.sleep_ms(100)


**2. 按键控制开关灯** 

::

    import time
    from openaie import button_group, rgb_led
    
    rgb = rgb_led(1)
    bt2 = button_group(2)   
    while (True):
        if bt2.is_press(1): # 检测到按键1按下
            time.sleep_ms(10)
            if bt2.is_press(1):
                print("button 1 press")
                rgb.set(0, (0,0,50))
                rgb.set(1, (0,0,0))
                rgb.display()
            while (bt2.is_press(1)) : # 等待按键1释放
                pass
        if bt2.is_press(2): # 检测到按键2按下
            time.sleep_ms(10)
            if bt2.is_press(2):
                print("button 2 press")
                rgb.set(0, (0,0,0))
                rgb.set(1, (0,0,50))
                rgb.display()
            while (bt2.is_press(2)) : # 等待按键2释放
                pass  
                
------------------------------------------------------


                
                
                