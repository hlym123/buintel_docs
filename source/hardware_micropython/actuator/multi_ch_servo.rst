多路舵机 
======================================================
 
 
   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 servo 模块 
    '''
    from openaie import multi_ch_servo
    
    '''
     类：舵机
     参数:
        port: 端口号  
    '''
    class multi_ch_servo(port)
    
    '''
     方法：设定角度 
     @ch: 通道选择 1~4
     @angle: 角度 0~180
    '''
    multi_ch_servo.write(ch, angle)
  
    
    
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 电位器控制舵机** 

::

    import time
    from openaie import *

    p = potentiometer(3)  # 电位器连接 端口3
    s = multi_ch_servo(1) # 多路舵机 


    while True:
        val = p.read()
        angle = math_map(val, 0, 100, 0, 180) # 数值映射 
        print("angle: %d"%angle)
        s.write(1, angle)        # 第1路舵机
        s.write(2, 180-angle)   # 第2路舵机
        time.sleep_ms(50)



------------------------------------------------------

        