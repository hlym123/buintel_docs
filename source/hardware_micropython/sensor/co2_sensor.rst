二氧化碳传感器 
====================================================== 
室内CO2浓度正常值在500~700ppm之间；大于900ppm则需要注意通风


   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 pir 模块 
    '''
    from openaie import co2_sensor
    
    '''
     类：co2_sensor
     参数:
        port: 端口号 -- 1~8 
    '''
    class co2_sensor(port)
    
    '''
     方法：检查数据是否准备好 
    '''
    co2_sensor.data_ready()
    
    '''
     方法：读取 二氧化碳 浓度
     量程：400 ~ 2000 ppm  
     精度：±(50 ppm + 5%的测量值)
    '''
    co2_sensor.co2  
    
    '''
     方法：读取 温度
     @note: 此模块集成有温湿度传感器
        量程：-10°C~60°C
        精度：±1.5°C 
    '''
    co2_sensor.temperature
    
    '''
     方法：读取 湿度
     @note: 此模块集成有温湿度传感器
        量程: 0%~100% 相对湿度
        精度: ±6%
    '''    
    co2_sensor.humidity
    
 
 
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 二氧化碳浓度检测** 
::

    '''
     二氧化碳检测
     模拟新风系统，当室内二氧化碳浓度超标时，
     开启新风系统，从室外吸入空气平衡室内二氧化碳
    '''
    from openaie import*
    import struct, time
    import lcd


    co2_sensor = co2_sensor(2) # 二氧化碳传感器 -- 端口2
    m = motor_fan(7)           # 电机风扇 -- 端口7
    m.set(0)

    #  
    co2_index = (400, 700, 1000, 2000)
    co2_color = ((0,255,0), (227,207,0), (255,128,0))
    co2_label = ("空气清新", "空气良好", "空气浑浊")

    # 绘制界面
    lcd.rotation(0)
    lcd.clear(color=0)
    lcd.draw_string(56, 5, "二氧化碳浓度检测", fc=(0,0,255), bc=(0,0,0))
    lcd.draw_string(150, 70, str(co2_index[0]), fc=(0,0,255), bc=(0,0,0))
    lcd.draw_rectangle(190, 70, 40, 60, color=co2_color[0], thickness=1, fill=True)
    lcd.draw_string(150, 130, str(co2_index[1]), fc=(0,0,255), bc=(0,0,0))
    lcd.draw_rectangle(190, 130, 40, 60, color=co2_color[1], thickness=1, fill=True)
    lcd.draw_string(150, 190, str(co2_index[2]), fc=(0,0,255), bc=(0,0,0))
    lcd.draw_rectangle(190, 190, 40, 60, color=co2_color[2], thickness=1, fill=True)
    lcd.display()
            
    while True:
        if co2_sensor.data_ready: # 数据已更新
            co2 = co2_sensor.co2          # 读取 二氧化碳 浓度
            temp = co2_sensor.temperature # 读取温度 
            humi = co2_sensor.humidity    # 读取湿度
            print("CO2: %d ppm"%co2)
            print("Temperature: %0.1f *C"%temp)
            print("Humidity: %0.1f %%\n"%humi)

            for i in range(3):
                if co2 > co2_index[i] and co2 < co2_index[i+1]:
                    lcd.draw_string(10, 70, "二氧化碳: %dPPM "%co2, fc=co2_color[i], bc=(0,0,0))
                    lcd.draw_string(10, 90, co2_label[i], fc=co2_color[i], bc=(0,0,0))
                    
            if co2 > co2_index[1]:
                m.set(80)
                lcd.draw_string(10, 130, "新风系统: 开启", fc=(0,0,255), bc=(0,0,0))
            else:
                m.set(0)
                lcd.draw_string(10, 130, "新风系统: 关闭", fc=(0,0,255), bc=(0,0,0))
                
            lcd.draw_string(10, 170, "温度: %.1fC  "%temp, fc=(0,0,255), bc=(0,0,0))
            lcd.draw_string(10, 190, "湿度: %.1f%%  "%humi, fc=(0,0,255), bc=(0,0,0))

            lcd.display()
            
        time.sleep_ms(500)
    
    
  







            
        
                                  
                                  

    
    
    