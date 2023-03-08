空气质量监测 
======================================================  


材料:
	控制器: 开元
	模块：粉尘传感器
	

程序   
+++++++++++++++++++++++++++++++++++++++++++++++++++++

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
		