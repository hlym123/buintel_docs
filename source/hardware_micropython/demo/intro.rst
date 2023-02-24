案例 
======================================================  

1. 人体热释电传感器    
++++++++++++++++++++++++++++++++++++++++++++++++++++++

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

		
2. 二氧化碳检测    
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

	'''
	 @date: 20220719
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
		
    
  
3. 空气质量检测    
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

	'''
	 @date:20220719
	 PM2.5检测 
	 
	 以PM2.5浓度作为空气质量检测标准 
	 
	'''
	import time
	import lcd 
	from openaie import *


	sensor = dust_sensor(1) # 粉尘传感器 -- 端口1
	m = motor_fan(7) 		# 电机风扇 -- 端口7
	m.set(0)

	# AQI(空气质量指数)等级划分 0~50~100~150~200~300~500
	aqi_index = (0, 50, 100, 150, 200, 300, 500)
	aqi_color = ((0,255,0), (227,207,0), (255,128,0), (255,0,0), (135,38,87), (128,42,42))
	aqi_label = ("优", "良", "轻度污染", "中度污染", "重度污染", "严重污染")

	# 绘制初始界面 
	lcd.rotation(0)
	lcd.clear(color=(0,0,0))
	lcd.draw_string(72, 5, "空气质量检测", fc=(0,0,255), bc=(0,0,0))
	for i in range(6):
		lcd.draw_string(160, i*40+50, str(aqi_index[i]), fc=(0,0,255), bc=(0,0,0))
		lcd.draw_rectangle(190, i*40+50, 40, 40, color=aqi_color[i], thickness=1, fill=True)

	# 设置空气净化器开关阈值
	threshold = aqi_index[2]
	while True:
		# 读取 PM2.5 测量值 
		pm2_5 = sensor.read()
		
		# 根据 PM2.5 数值判断是否开启空气净化器
		if pm2_5 > threshold:
			m.set(80)
			lcd.draw_string(10, 110, "空气净化器:开启", fc=(0,0,255), bc=(0,0,0))
		else:
			m.set(0)
			lcd.draw_string(10, 110, "空气净化器:关闭", fc=(0,0,255), bc=(0,0,0))
			
		# 显示 PM2.5 测量值    
		for i in range(6):
			if pm2_5 > aqi_index[i] and pm2_5 < aqi_index[i+1]:
				lcd.draw_string(10, 50, "PM2.5: %d PPM "%pm2_5, fc=aqi_color[i], bc=(0,0,0))
				lcd.draw_string(10, 70, "空气质量:" + aqi_label[i], fc=aqi_color[i], bc=(0,0,0))
				
		lcd.display()
			
		time.sleep_ms(1000)
		
		
4. 指纹识别    
++++++++++++++++++++++++++++++++++++++++++++++++++++++ 

::
  
	'''
	 @date: 20220719
	 
	 指纹 录入与验证(识别)
	 
	'''
	import time
	import lcd
	from openaie import button_group, fingerprint_sensor, servo


	'''
	 录入指纹
	 流程：
		 读入指纹图像，生成指纹特征1
		 再次读入指纹图像，生成指纹特征2
		 若 特征1 与 特征2 匹配，则生成指纹模板并保存
	'''
	def enroll(id):
		lcd.draw_string(10, 100, "录入指纹...", fc=(0,0,255), bc=(0,0,0))
		for i in range(2):
			if i == 0:
				print("请按手指")
				lcd.draw_string(10, 120, "请按手指    ", fc=(0,0,255), bc=(0,0,0))
			else:
				print("请重按手指")
				lcd.draw_string(10, 120, "请重按手指  ", fc=(0,0,255), bc=(0,0,0))
			lcd.display()
			# 1. 获取指纹图像
			while True: # 等待手指按下
				r = fingerprint.get_image()
				if r == fingerprint.FINGERPRINT_OK:
					print("获取指纹图像")
					break 
			# 2. 根据录入图像生成指纹特征 
			r = fingerprint.gen_char(i+1) 
			if r == fingerprint.FINGERPRINT_OK:
				print("生成指纹特征", i+1)
			if i < 1:
				print("请移开手指")
				lcd.draw_string(10, 120, "请移开手指  ", fc=(0,0,255), bc=(0,0,0))
				lcd.display()
				time.sleep_ms(800) 
		# 3. 合并指纹特征，生成指纹模板
		print("生成指纹模板")
		r = fingerprint.create_model()
		if r == fingerprint.FINGERPRINT_OK:
			print("两次录入指纹特征匹配，生成指纹模板成功")
		elif r == fingerprint.FINGERPRINT_ENROLLMISMATCH:
			print("两次录入指纹特征不匹配，生成指纹模板失败")      
		# 4. 存储指纹模板 
		print("存储指纹模板到位置: %d"%id)
		r = fingerprint.store_model(id)
		if r == fingerprint.FINGERPRINT_OK:
			print("录入指纹成功")
			lcd.draw_string(10, 120, "录入指纹成功", fc=(0,255,0), bc=(0,0,0))
			lcd.display()

	'''
	 验证指纹
		读入指纹图像
		生成指纹特征
		搜索匹配指纹（是否为已保存的指纹）
	'''
	def verify():
		lcd.draw_string(10, 100, "验证指纹...  ", fc=(0,0,255), bc=(0,0,0))
		 # 1. 等待手指放置
		print("请按手指")
		lcd.draw_string(10, 120, "请按手指      ", fc=(0,0,255), bc=(0,0,0))
		lcd.display()
		while True: # 等待手指按下
			r = fingerprint.get_image()
			if r == fingerprint.FINGERPRINT_OK:
				print("获取指纹图像")
				break 
			time.sleep_ms(100)
		 
		# 2. 根据录入图像生成指纹特征 
		r = fingerprint.gen_char(1)
		if r == fingerprint.FINGERPRINT_OK:
			print("生成指纹特征1")
		elif r == fingerprint.FINGERPRINT_IMAGEMESS:
			print("指纹不清晰")

		# 3. 搜索指纹 
		r = fingerprint.search()
		if r == fingerprint.FINGERPRINT_OK:
			print("匹配成功")
			lcd.draw_string(10, 120, "匹配成功 ID:%d"%fingerprint.verify_id, fc=(0,255,0), bc=(0,0,0))
			print("score: %d"%fingerprint.verify_score)
			lcd.display()
			return 0 
		elif r == 0x09:
			print("没有搜索到")
			lcd.draw_string(10, 120, "未识别指纹  ", fc=(255,0,0), bc=(0,0,0))
			lcd.display()
			return -1

	 

	# 初始显示界面
	lcd.rotation(0)    
	lcd.clear(color=0)    
	lcd.draw_string(72, 10, "指纹识别测试", fc=(0,0,255), bc=(0,0,0))
	lcd.draw_string(10, 50, "按 按键1 录入指纹", fc=(0,0,255), bc=(0,0,0))
	lcd.draw_string(10, 70, "按 按键2 识别指纹", fc=(0,0,255), bc=(0,0,0)) 
	lcd.display()

	bt2 = button_group(2)               # 按键模块连接到 -- 端口1
	fingerprint = fingerprint_sensor(7) # 指纹识别传感器 -- 端口7
	s = servo(5)                        # 舵机 -- 端口5
	s.write(0)

	id_num = 1
	deadline = 0  
	while True:
		if bt2.is_press(1): # 检测到按键按下
			time.sleep_ms(10) # 延时消抖
			if bt2.is_press(1):
				print("\n===============")
				print("  准备录入指纹  ")
				print("===============")  
				enroll(id_num)
				id_num += 1
			while (bt2.is_press(1)) : # 等待按键释放
				pass

		if bt2.is_press(2): # 检测到按键按下
			time.sleep_ms(10) # 延时消抖
			if bt2.is_press(2):
				print("\n===============")
				print("  开始指纹识别  ")
				print("===============")
				if verify() == 0:
					deadline = time.ticks_add(time.ticks_ms(), 3000)
					print("success")
					s.write(90) # 打开
				else:
					pass
			while (bt2.is_press(2)) : # 等待按键释放
				pass
			
		if time.ticks_diff(deadline, time.ticks_ms()) < 0: # 超时自动关闭 
			s.write(0)
			
		time.sleep_ms(10)


5. 卫星定位授时    
++++++++++++++++++++++++++++++++++++++++++++++++++++++ 

::

	'''
	 @date: 20220719
	 
	 卫星定位授时
	 
	'''
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


	my_gps = bds(1) # 卫星定位模块 -- 端口1


	# 绘制初始显示界面
	lcd.rotation(0)

	  
	deadline = 0
	gps_deadline = 0
	cnt = 0
	while True:  
		if time.ticks_diff(gps_deadline, time.ticks_ms()) < 0:  
			gps_deadline = time.ticks_add(time.ticks_ms(), 100)  
			my_gps.update() # GPS更新数据
	  
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
			lcd.draw_string(10, 175, '速度: %.2f km/h'%my_gps.speed[2], fc=(0,0,255), bc=(0,0,0))
			lcd.draw_string(10, 215, 'count: %d'%cnt, fc=(0,0,255), bc=(0,0,0))
			cnt += 1
			
			lcd.display()

	 




            











 

------------------------------------------------------
