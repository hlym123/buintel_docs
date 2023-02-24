超声波测距传感器
======================================================
超声波测距传感器可发送和接收超声波。发送出去的超声波遇到障碍物时会产生回波，
通过测量发送超声波到接收到回波的时间间隔：t，再根据声速：s，可计算得传感器与障碍物的距离：d=s/2t。
模块内置运算单元，可直接读取计算结果。传感器的量程：2~200cm；误差：±1%。

.. figure:: ul_sensor.png 
   :width: 260
   :align: center
   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 ultrasonic 模块 
    '''
    from openaie import ultrasonic
    
    '''
     类：超声波测距传感器
     参数:
        port: 端口号 -- 1~8 
    '''
    class ultrasonic(port)
    
    '''
     方法：触发测量 
    '''
    ultrasonic.measure()
    
    '''
     方法：读取测量值 
     说明：在触发测量后，需要等待回波信号（不大于100ms），再读取测量值。
    '''
    ultrasonic.read()
    
    
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 超声波测距** 

::

    import time
    from openaie import *


    us_sensor = ultrasonic(4)
            
    while True:
        us_sensor.measure() # 触发测量  
        time.sleep_ms(100)  # 等待测量完成 
        print("distance: %.1fcm\r\n"%us_sensor.read()) # 读取测量结果 
        time.sleep_ms(400) 
    
 
**2. 超声波测距显示** 

::

    # time.ticks_ms() -- 获取运行时间 
    # time.ticks_diff(deadline, time.ticks_ms()) -- 计算差值
    import time
    import lcd 
    from openaie import *


    us_sensor = ultrasonic(4)

    lcd.rotation(0)
     
    trig = False    
    deadline = 0        
    while True:
        if not trig:
            trig = True
            us_sensor.measure() # 触发测量  
            deadline = time.ticks_ms() + 100  
        if time.ticks_diff(deadline, time.ticks_ms()) < 0: # 等待
            trig = False
            dis = us_sensor.read() # 读取测量结果
            lcd.clear(color=0)
            lcd.draw_string(10, 10, "距离: %.1fcm"%dis, fc=(0,0,255), bc=0)
            lcd.display()
            print("distance: %.1fcm\r\n"%dis) 
        time.sleep_ms(200) 

------------------------------------------------------
















        