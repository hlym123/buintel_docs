空气质量监测系统 
======================================================  

检测显示环境二氧化碳浓度、PM2.5浓度

器材:

* 控制器：开元 
* 模块：粉尘传感器；二氧化碳传感器；电机风扇 
 

程序 
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    import time
    import lcd
    from openaie import *


    dust_sensor = dust_sensor(1) # 粉尘传感器 -- 端口1
    co2_sensor = co2_sensor(2)   # 二氧化碳传感器 -- 端口2
    m = motor_fan(7)             # 电机风扇 -- 端口7
    m.set(0)

    # AQI(空气质量指数)等级划分 0~50~100~150~200~300~500
    aqi_index = (0, 50, 100, 150, 200, 300, 500)
    aqi_color = ((0,255,0), (227,207,0), (255,128,0), (255,0,0), (135,38,87), (128,42,42))
    aqi_label = ("优      ", "良      ", "轻度污染", "中度污染", "重度污染", "严重污染")

    # 二氧化碳浓度等级划分
    co2_index = (400, 700, 1000, 2000)
    co2_color = ((0,255,0), (227,207,0), (255,128,0))
    co2_label = ("空气清新", "空气良好", "空气浑浊")

    # 绘制初始界面
    lcd.rotation(0)
    lcd.clear(color=(0,0,0))
    lcd.draw_string(72, 5, "空气质量监测", fc=(0,0,255), bc=(0,0,0))
    for i in range(6):
        lcd.draw_string(160, i*40+50, str(aqi_index[i]), fc=(0,0,255), bc=(0,0,0))
        lcd.draw_rectangle(190, i*40+50, 40, 40, color=aqi_color[i], thickness=1, fill=True)

    # 设置空气净化器开关阈值
    threshold = aqi_index[2]
    while True:
        # 读取显示 粉尘传感器 测量值
        pm2_5 = dust_sensor.read()
        for i in range(6):
            if pm2_5 > aqi_index[i] and pm2_5 < aqi_index[i+1]:
                lcd.draw_string(10, 50, "PM2.5: %d PPM "%pm2_5, fc=aqi_color[i], bc=(0,0,0))
                lcd.draw_string(10, 70, "空气质量:" + aqi_label[i], fc=aqi_color[i], bc=(0,0,0))
        
        # 根据 PM2.5 数值判断是否开启空气净化器(电机风扇)
        if pm2_5 > threshold:
            m.set(80)
            lcd.draw_string(10, 90, "空气净化器: 开启", fc=(0,255,0), bc=(0,0,0))
        else:
            m.set(0)
            lcd.draw_string(10, 90, "空气净化器: 关闭", fc=(0,255,0), bc=(0,0,0))
                
        # 读取显示 二氧化碳传感器 测量值
        if co2_sensor.data_ready: # 数据已更新
            co2 = co2_sensor.co2          # 读取 二氧化碳 浓度
            temp = co2_sensor.temperature # 读取温度
            humi = co2_sensor.humidity    # 读取湿度
            for i in range(3):
                if co2 > co2_index[i] and co2 < co2_index[i+1]:
                    lcd.draw_string(10, 130, "二氧化碳: %d PPM  "%co2, fc=co2_color[i], bc=(0,0,0))
                    lcd.draw_string(10, 150, co2_label[i], fc=co2_color[i], bc=(0,0,0))
                    
            lcd.draw_string(10, 210, "温度: %.1fC  "%temp, fc=(0,255,0), bc=(0,0,0))
            lcd.draw_string(10, 230, "湿度: %.1f%%  "%humi, fc=(0,255,0), bc=(0,0,0))

            # 根据 CO2 数值判断是否开启新风系统(电机风扇)
            if co2 > co2_index[1]:
                m.set(80)
                lcd.draw_string(10, 170, "新风系统: 开启", fc=(0,255,0), bc=(0,0,0))
            else:
                m.set(0)
                lcd.draw_string(10, 170, "新风系统: 关闭", fc=(0,255,0), bc=(0,0,0))
     
        lcd.display()
        time.sleep_ms(1000)
 


------------------------------------------------------
