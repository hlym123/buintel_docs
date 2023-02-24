光照强度传感器
======================================================
 
\ `光照强度 <https://www.cnblogs.com/zlbg/p/4049962.html>`_ 
传感器可检测光照强度，测量范围：1~65535lx(勒克斯)。

住宅建筑照明标准值

.. figure:: 住宅建筑照明标准值.png
    :height: 300 px
    :width: 500 px
    :scale: 100 %
    :alt: alternate text
    :align: center
   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 light_sensor 模块 
    '''
    from openaie import light_sensor
    
    '''
     类：光照强度传感器 
     参数:
        port: 端口号  
    '''
    class light_sensor(port)
    
    '''
     方法：读取光照强度
    '''
    light_sensor.read()
  
    
    
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 读取显示光照强度** 

::

    import time 
    import lcd 
    from openaie import light_sensor

    light = light_sensor(2)

    while True:
        val = light.read()
        info = ('brightness: %d lux'%val)
        lcd.clear(color=0)
        lcd.draw_string(10, 10, info, fc=(0,0,255), bc=0)
        lcd.display()
        time.sleep_ms(500)
        
        
        


------------------------------------------------------

        