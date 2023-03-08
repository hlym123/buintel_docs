GPS卫星定位导航模块
======================================================
通信接口：I2C

GPS/北斗双模定位导航模块，可用于导航，定位，授时等。定位精度10m，首次定位时间大于32秒，当板载指示灯闪烁时，表示定位成功。



应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 bds 模块 
    '''
    from openaie import bds
    
    '''
     类：gps
     参数:
        port: 端口号 -- 1~8 
    '''
    class bds(port)
    
    '''
     方法：更新数据  
    '''
    bds.update()

    '''
     变量：可见卫星数 
    '''
    bds.satellites_in_view
    
    '''
     变量：使用卫星数 
    '''
    bds.satellites_in_use

    '''
     变量：经度（浮点数）
    '''
    bds.longitude
    
    '''
     变量：纬度（浮点数）
    '''
    bds.latitude
    
    '''
     变量：海拔（m）（只做参考，不准确）
    '''
    bds.altitude
    
    '''
     变量：速度（km/h）
    '''
    bds.speed
    
    '''
     变量：年，月，日（UTC时间，北京时间+8）
    '''
    bds.year, bds.month, bds.day  
    
    '''
     变量：时，分，秒（UTC时间，北京时间+8）
        
    '''
    bds.hour, bds.minute, bds.second  

案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 信息读取显示** 

::

    from openaie import bds
    import time
    import lcd
 

     
    '''
     时区转换
     @dt: 日期时间 格式[year, month, day, hour, minute, second]
     @timezone: 时区 默认为东8区，即北京时间
    '''
    def datetime(dt, timezone=8):
        year, month, day, hour, minute, second = dt[:]
        month_day = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
        if year%4 == 0: # 闰年判断
            month_day[1] = 29
        hour += timezone
        if hour >= 24:
            hour -= 24
            day += 1
            if day > month_day[month-1]:
                day -=  month_day[month-1]
                month += 1
                if month > 12:
                    month = 1
                    year += 1
        return [year, month, day, hour, minute, second]
     
     
    gps = bds(3) # GPS/北斗卫星导航定位模块 -- 端口3

    # 显示屏设置
    lcd.rotation(0)
    lcd.clear(color=0)
    lcd.set_backlight(50)

    deadline = 0
    cnt = 0
    while True:
        if time.ticks_diff(deadline, time.ticks_ms()) < 0:
            deadline = time.ticks_add(time.ticks_ms(), 1000)  # 显示刷新间隔 1000ms
            try:
                # 更新 gps 数据 
                gps.update()
                
                lcd.clear(color=(0,0,0))
                lcd.draw_string(72, 5, '卫星定位授时', fc=(0,0,255), bc=(0,0,0))
                # 显示日期时间
                year, month, day, hour, minute, second = datetime([gps.year, gps.month, gps.day, gps.hour, gps.minute, gps.second])[:] # 时区转换
                date_string = "%04d/%02d/%02d"%(year, month, day)
                lcd.draw_string(10, 40, date_string, fc=(0,0,255), bc=(0,0,0))
                time_string = "%02d:%02d:%02d "%(hour, minute, second)
                lcd.draw_string(110, 40, time_string, fc=(0,0,255), bc=(0,0,0))
                # 卫星信息
                lcd.draw_string(10, 75, '可见卫星: %s 颗'%gps.satellites_in_view, fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 95, '使用卫星: %s 颗'%gps.satellites_in_use, fc=(0,0,255), bc=(0,0,0))
                # 位置
                lcd.draw_string(10, 115, '经度: %s'%gps.longitude, fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 135, '纬度: %s'%gps.latitude, fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 155, '海拔: %d m'%gps.altitude, fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 175, '速度: %.2f km/h'%gps.speed, fc=(0,0,255), bc=(0,0,0))
                
                lcd.draw_string(10, 255, 'cnd: %d'%cnt, fc=(0,0,255), bc=(0,0,0))
                cnt += 1
                lcd.display()
            except:
                print("read error")
              
        time.sleep_ms(10)


    
   
------------------------------------------------------

以下为 UART 接口 卫星定位导航模块 说明    
    

通信接口UART，默认波特率：9600，数据位8，停止位1。数据格式为 NEMA0183 标准，输出频率为1Hz。板载指示灯在定位成功时闪烁。

定位精度10m，首次定位时间大于32秒。

 
   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 bds 模块 
    '''
    from openaie import bds
    
    '''
     类：gps
     参数:
        port: 端口号 -- 1或7 
    '''
    class bds(port)
    
    '''
     方法：更新数据 
    '''
    bds.update()
    
    '''
     变量：可见卫星数 
    '''
    bds.satellites_in_view
    
    '''
     变量：使用卫星数 
    '''
    bds.satellites_in_use
    
    '''
     变量：经度（字符串）
    '''
    bds.longitude_string
    
    '''
     变量：纬度（字符串）
    '''
    bds.latitude_string
    
    '''
     变量：经度（浮点数）
    '''
    bds.longitude[0]
    
    '''
     变量：纬度（浮点数）
    '''
    bds.latitude[0]
    
    '''
     变量：海拔（m）
    '''
    bds.altitude
    
    '''
     变量：速度（km/h）
    '''
    bds.speed[2]
    
    '''
     变量：日期（UTC时间，北京时间+8）
        日，月，年
    '''
    day, month, year = bds.date[:]
    
    '''
     变量：时间戳（UTC时间，北京时间+8）
        时，分，秒
    '''
    hour, minute, second = bds.timestamp[:]
    
 
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 信息读取显示** 

::

    import lcd, time, math 
    from openaie import bds
     

    '''
     时区转换 
     @dt: 日期时间 格式[year, month, day, hour, minute, second]
     @timezone: 时区 默认为东8区，即北京时间  
    '''
    def datetime(dt, timezone=8):
        year, month, day, hour, minute, second = dt[:]
        month_day = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]
        if year%4 == 0: # 闰年判断
            month_day[1] = 29
        hour += timezone
        if hour >= 24:
            hour -= 24 
            day += 1
            if day > month_day[month-1]:
                day -=  month_day[month-1]
                month += 1 
                if month > 12: 
                    month = 1
                    year += 1
        date_string = "%04d/%02d/%02d"%(year, month, day)
        time_string = "%02d:%02d:%02d "%(hour, minute, second)
        #print(date_string, ' ', time_string)
        return [year, month, day, hour, minute, second]



    # 显示屏设置
    lcd.set_backlight(50)
    lcd.rotation(0)

    my_gps = bds(1)

    deadline = 0
    while True:  
        my_gps.update()
        if time.ticks_diff(deadline, time.ticks_ms()) < 0:
            deadline = time.ticks_add(time.ticks_ms(), 500)  # 显示刷新间隔 500ms
            
            lcd.clear(color=(0,0,0))
            lcd.draw_string(72, 5, '卫星定位授时', fc=(0,0,255), bc=(0,0,0))
            # 显示日期时间 
            day, month, year = my_gps.date[:] # 获取日期（UTC）
            hour, minute, second = my_gps.timestamp[:] # 获取时间（UTC）
            year, month, day, hour, minute, second = datetime([year+2000, month, day, hour, minute, second])[:] # 时区转换
            date_string = "%04d/%02d/%02d"%(year, month, day)
            lcd.draw_string(10, 40, date_string, fc=(0,0,255), bc=(0,0,0))
            time_string = "%02d:%02d:%02d "%(hour, minute, second)
            lcd.draw_string(110, 40, time_string, fc=(0,0,255), bc=(0,0,0))
            # 卫星信息
            lcd.draw_string(10, 75, '可见卫星: %s 颗'%my_gps.satellites_in_view, fc=(0,0,255), bc=(0,0,0))
            lcd.draw_string(10, 95, '使用卫星: %s 颗'%my_gps.satellites_in_use, fc=(0,0,255), bc=(0,0,0))
            # 位置 
            longitude = my_gps.longitude[0]
            latitude = my_gps.latitude[0] 
            lcd.draw_string(10, 115, '经度: %s'%longitude, fc=(0,0,255), bc=(0,0,0))
            lcd.draw_string(10, 135, '纬度: %s'%latitude, fc=(0,0,255), bc=(0,0,0))
            lcd.draw_string(10, 155, '海拔: %d m'%my_gps.altitude, fc=(0,0,255), bc=(0,0,0))
            lcd.draw_string(10, 175  , '速度: %.2f km/h'%my_gps.speed[2], fc=(0,0,255), bc=(0,0,0))

            lcd.display()

 
------------------------------------------------------
















        