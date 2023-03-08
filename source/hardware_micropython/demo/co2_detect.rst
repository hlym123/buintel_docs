二氧化碳检测 
======================================================  


材料:
	控制器: 开元
	模块：二氧化碳传感器
	

程序   
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
		