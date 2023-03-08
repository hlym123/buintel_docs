卫星定位授时 
======================================================  


材料:
	控制器: 开元
	模块：GPS卫星定位导航模块 
	

程序   
+++++++++++++++++++++++++++++++++++++++++++++++++++++

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