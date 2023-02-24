舵机 
======================================================
 
 
   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 servo 模块 
    '''
    from openaie import servo
    
    '''
     类：舵机
     参数:
        port: 端口号  
    '''
    class servo(port)
    
    '''
     方法：设定角度 
     @angle: 角度 0~180
    '''
    servo.write(angle)
  
    
    
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 电位器控制舵机** 

::

    import time
    from openaie import *

    p = potentiometer(3) # 电位器连接 端口3
    s = servo(1)     #  


    while True:
        val = p.read()
        val = math_map(val, 0, 100, 0, 180) # 数值映射 
        print("angle: %d"%val)
        s.write(val)
        time.sleep_ms(50)



------------------------------------------------------

        