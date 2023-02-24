人体热释电传感器 
====================================================== 



   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 pir_sensor 模块 
    '''
    from openaie import pir_sensor
    
    '''
     类：pir_sensor
     参数:
        port: 端口号 -- 1或7 
    '''
    class pir_sensor(port)
    
    '''
     方法：检测
     @return：检测到有人移动返回 True 
    '''
    pir_sensor.detect()
 
    
 
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 人体感应灯** 

::

    from openaie import rgb_led, pir_sensor
    import time
    import lcd


    pir_sensor = pir_sensor(2) # 人体热释电传感器 -- 端口2           
    rgb = rgb_led(7)           # 全彩LED -- 端口6          

    deadline = 0
    delay_time = 3000 # 单次触发延时时间 3000ms
    light_on = False

    # 绘制初始界面
    lcd.rotation(0)
    lcd.clear(color=0)
    lcd.draw_string(76, 5, "自动感应灯", fc=(0,0,255), bc=0)
    lcd.display()
     
    while True:
        if pir_sensor.detect(): # 检测到有人
            deadline = time.ticks_add(time.ticks_ms(), delay_time) # 重新计算截止时间
            if not light_on:
                for i in range(3):
                    rgb.set(i, (50,50,50))
                rgb.display()
                light_on = True
                lcd.draw_string(10, 50, "检测到有人", fc=(0,255,0), bc=0)
                lcd.draw_string(10, 90, "照明设备:开启", fc=(0,0,255), bc=0)
                lcd.display()
                print("turn on the light")
            
        if light_on:
            print("now: ", time.ticks_ms())
            if time.ticks_diff(time.ticks_ms(), deadline) > 0: # 判断是否超时，超时自动关闭照明
                for i in range(3):
                    rgb.set(i, (0,0,0))
                rgb.display()
                light_on = False
                lcd.draw_string(10, 50, "没有检测到", fc=(255,0,0), bc=0)
                lcd.draw_string(10, 90, "照明设备:关闭", fc=(0,0,255), bc=0)
                lcd.display()
                print("turn off the light")
                
        time.sleep_ms(100)


            
        
                                  
                                  

    
    
    