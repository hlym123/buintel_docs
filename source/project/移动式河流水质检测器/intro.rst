移动式河流水质检测器 
======================================================  

系统分主机（接收显示）和从机（检测发送）两部分，从机检测记录水相关参数（温度、浊度、TDS等），主机接收显示数据。

器材:

* 控制器：开元x2 
* 模块：温度传感器；浊度传感器；TDS传感器；GPS卫星定位导航模块；无线串口通信模块x2   


程序(检测发送端) 
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    from openaie import get_port_pin, bds, rf
    import time, struct, _thread
    from machine import ADC, Pin
    import onewire, ds18x20
    import lcd
     
     
    lcd.rotation(0)
    lcd.set_backlight(60)


    '''
     温度检测(DS18B20)
    '''
    sig = machine.Pin(get_port_pin(3, 2)) # 开元主控端口3，信号线2
    ds = ds18x20.DS18X20(onewire.OneWire(sig))
    roms = ds.scan()
    print('found devices:', roms[0])

    convert = False
    temp_deadline = 0
    temperature = 0
    def read_temp():
        global convert, temp_deadline, temperature 
        if not convert:
            convert = True
            ds.convert_temp() # 开始转换 
            temp_deadline = time.ticks_add(time.ticks_ms(), 750) 
        if time.ticks_diff(temp_deadline, time.ticks_ms()) < 0: # 等待
            convert = False
            temperature = ds.read_temp(roms[0]) # 读取测量结果
            #print("temperature: %.1f℃\r\n"%temperature)
       
    '''
     TDS检测
    '''
    adc1 = ADC(Pin(get_port_pin(3, 1))) # 开元主控端口3，信号线1 ADC1
    k = 1 # k为对照标准TDS溶液得到的计算值 
    tds = 0
    def read_tds(temp=None):
        global tds
        r = adc1.read_u16()
        #print("ADC value: ", r)
        vol = r*3.3/65535.0 # 转换为电压
        #print("Voltage: V", vol)
        if temp is not None: # 温度补偿 
            temp_correct = 1+0.02*(temp-25) # 温度修正
            vol = vol*temp_correct # 电压修正 
        tds_row = 66.71*pow(vol,3) - 127.93*vol*vol + 428.7*vol
        tds = int(tds_row*k) # 矫正后的值

    '''
     浊度检测 
    '''        
    adc2 = ADC(Pin(get_port_pin(4, 1))) # 开元主控端口4，信号线1 ADC2
    turbidity = 0
    def read_turbidity():
        global turbidity
        r = adc2.read_u16()
        #print("ADC value: ", r)
        vol = r*3.3/65536.0 # 转换为电压
        #print("Voltage: V", vol)
        turbidity = vol
        
    '''
     无线串口通信模块初始化
    '''
    rf = rf(7)
    if rf.set('tx', '11223'):
        print("发送模式初始化成功")
    else:
        print("发送模式初始化失败")

        
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
        
    '''
     GPS数据接收解析
    '''    
    my_gps = bds(1)
    def core1_main():
        global my_gps
        display_deadline = 0
        gps_deadline = 0

        while True:
            if time.ticks_diff(gps_deadline, time.ticks_ms()) < 0:  
                gps_deadline = time.ticks_add(time.ticks_ms(), 200)  
                my_gps.update() # GPS更新数据
     
            if time.ticks_diff(display_deadline, time.ticks_ms()) < 0:  
                display_deadline = time.ticks_add(time.ticks_ms(), 500) # 数据更新周期 500ms 
     
                read_turbidity()
                time.sleep_ms(10)
                read_temp()
                read_tds(temperature)
                
                lcd.clear(color=(0,0,0))
                # 显示卫星信息
                lcd.draw_string(10, 30, '可见卫星: %s 颗'%my_gps.satellites_in_view, fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 50, '使用卫星: %s 颗'%my_gps.satellites_in_use, fc=(0,0,255), bc=(0,0,0))
                # 显示经纬度
                lcd.draw_string(10, 70, '经度: %s'%(my_gps.longitude[0]), fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 90, '纬度: %s'%(my_gps.latitude[0]), fc=(0,0,255), bc=(0,0,0))
                # 显示日期时间 
                day, month, year = my_gps.date[:] # 获取日期（UTC）
                hour, minute, second = my_gps.timestamp[:] # 获取时间（UTC）
                year, month, day, hour, minute, second = datetime([year+2000, month, day, hour, minute, second])[:] # 时区转换
                date_string = "%04d/%02d/%02d"%(year, month, day)
                lcd.draw_string(10, 5, date_string, fc=(0,0,255), bc=(0,0,0))
                time_string = "%02d:%02d:%02d "%(hour, minute, second)
                lcd.draw_string(100, 5, time_string, fc=(0,0,255), bc=(0,0,0))
                # 显示测量信息
                lcd.draw_string(10, 150, "温度: %.1f"%temperature, fc=(0,0,255), bc=0)
                if tds>300:
                    tds_color = (255,0,0)
                elif tds>200:
                    tds_color = (0,0,255)
                else:
                    tds_color = (0,255,0)
                lcd.draw_string(10, 170, "TDS: %d ppm"%tds, fc=tds_color, bc=0)
                if turbidity > 2.96:
                    lcd.draw_string(10, 190, "浑浊度: 1 级", fc=(0,255,0), bc=0)
                elif turbidity > 2.64:
                    lcd.draw_string(10, 190, "浑浊度: 2 级", fc=(0,0,255), bc=0)
                elif turbidity> 1.84:
                    lcd.draw_string(10, 190, "浑浊度: 3 级", fc=(255,0,255), bc=0)
                else:
                    lcd.draw_string(10, 190, "浑浊度: 4 级", fc=(255,0,0), bc=0)
                #lcd.draw_string(10, 190, "浑浊度: %.2f"%turbidity, fc=(0,0,255), bc=0)
                lcd.display()
                
      
    _thread.start_new_thread(core1_main, ())

    send_deadline = 0
    send_buf = bytearray(20)
    while True:
        if time.ticks_diff(send_deadline, time.ticks_ms()) < 0:  
            send_deadline = time.ticks_add(time.ticks_ms(), 500) # 数据发送周期 1000ms 

            send_buf[:2] = 'TX'.encode() # 帧头
            send_buf[2:6] = struct.pack('>f', temperature)
            send_buf[6:10] = struct.pack('>f', turbidity)
            send_buf[10:12] = struct.pack('>H', tds)
            send_buf[12:16] = struct.pack('>f', my_gps.longitude[0])
            send_buf[16:20] = struct.pack('>f', my_gps.latitude[0])
            rf.send(send_buf)

            
程序(接收显示端) 
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     注意：无线模块接线更改!!! 
    '''
    import lcd, time, struct
    from openaie import rf 

     

    # 无线串口通信模块初始化
    rf = rf(1)
    if rf.set('rx', '11223'):
        print("发送模式初始化成功")
    else:
        print("发送模式初始化失败")
        

    # 显示屏设置 
    lcd.rotation(1)
    lcd.set_backlight(60)
     
    rx_cnt = 0
    update_deadline = 0

    # 
     
    while True:
        if time.ticks_diff(update_deadline, time.ticks_ms()) < 0:  
            update_deadline = time.ticks_add(time.ticks_ms(), 500)
            
            # 读取接收数据
            rx_buf = rf.read(32)
            if rx_buf is not None:
                #print('rx len: ',len(rx_buf))
                if len(rx_buf) < 20:
                    continue
                      
                temperature = struct.unpack('>f', rx_buf[2:6])[0]
                turbidity = struct.unpack('>f', rx_buf[6:10])[0]
                tds = struct.unpack('>H', rx_buf[10:12])[0]
                longitude = struct.unpack('>f', rx_buf[12:16])[0]
                latitude = struct.unpack('>f', rx_buf[16:20])[0]

                lcd.clear(color=(0,0,0))
                #
                lcd.draw_string(112, 5, '河流水质检测', fc=(0,0,255), bc=(0,0,0))
                lcd.draw_line(0, 30, 319, 30, color=(0,0,255), thickness=2)
                
                # 显示经纬度
                lcd.draw_string(10, 50, '设备: %d'%1, fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 70, '经度: %s'%longitude, fc=(0,0,255), bc=(0,0,0))
                lcd.draw_string(10, 90, '纬度: %s'%latitude, fc=(0,0,255), bc=(0,0,0))
                # 显示测量信息
                lcd.draw_string(10, 150, "温度: %.1f"%temperature, fc=(0,0,255), bc=0)
                if tds>300:
                    tds_color = (255,0,0)
                elif tds>200:
                    tds_color = (0,0,255)
                else:
                    tds_color = (0,255,0)
                lcd.draw_string(10, 170, "TDS: %d ppm"%tds, fc=tds_color, bc=0)
                if turbidity > 2.96:
                    lcd.draw_string(10, 190, "浑浊度: 1 级", fc=(0,255,0), bc=0)
                elif turbidity > 2.64:
                    lcd.draw_string(10, 190, "浑浊度: 2 级", fc=(0,0,255), bc=0)
                elif turbidity> 1.84:
                    lcd.draw_string(10, 190, "浑浊度: 3 级", fc=(255,0,255), bc=0)
                else:
                    lcd.draw_string(10, 190, "浑浊度: 4 级", fc=(255,0,0), bc=0)
                    #lcd.draw_string(10, 190, "浑浊度: %.2f"%turbidity, fc=(0,255,0), bc=0)
                lcd.display()
                
                print("receive count: ", rx_cnt)
                rx_cnt += 1
                
        time.sleep_ms(50)
        
 


------------------------------------------------------
