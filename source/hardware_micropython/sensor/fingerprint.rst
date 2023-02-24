指纹识别模块
======================================================
 
 
   
应用编程接口说明
++++++++++++++++++++++++++++++++++++++++++++++++++++++

::

    '''
     导入 fingerprint_sensor 模块 
    '''
    from openaie import fingerprint_sensor
    
    '''
     类：指纹识别模块 
     参数:
        port: 端口号 -- 1或7 
    '''
    class fingerprint_sensor(port)
    
    '''
     方法：获取指纹图像 （有手指按下时，可以成功获取指纹图像）
     @return fingerprint.FINGERPRINT_OK 获取指纹图像成功
    '''
    fingerprint_sensor.get_image()
    # example:
    fingerprint = fingerprint_sensor(1)
    while True: # 等待手指按下
        r = fingerprint.get_image()
        if r == fingerprint.FINGERPRINT_OK:
            print("获取指纹图像")
            break 
        time.sleep_ms(100)
        
    '''
     方法：生成指纹特征
     @num: 1, 2 生成指纹 特征1 或 特征2
     @return fingerprint.FINGERPRINT_OK 生成指纹特征成功
    '''
    fingerprint_sensor.gen_char(num)     
    
    '''
     方法：生成指纹模板 
     @return 
        fingerprint.FINGERPRINT_OK 两次录入指纹特征匹配，生成指纹模板成功
        fingerprint.FINGERPRINT_ENROLLMISMATCH 两次录入指纹特征不匹配，生成指纹模板失败
    '''
    fingerprint.create_model()
    
    '''
     方法：存储指纹模板 
     @return ingerprint.FINGERPRINT_OK 存储指纹模板成功
     @id: 指纹存储位置 1~60
    '''
    fingerprint.store_model(id)
    
    '''
     方法：搜索指纹（检测到的指纹与已保存的数据搜索匹配） 
     @return：fingerprint.FINGERPRINT_OK 指纹匹配成功 
     @ps:
        若指纹匹配成功，可以获取下列数据: 
        fingerprint.verify_id 匹配到的指纹 ID
        fingerprint.verify_score 匹配到的指纹 相似度得分 
    '''
    fingerprint.search()
    
    '''
     方法：删除指纹
     @id: 要删除的指纹ID
    '''
    fingerprint_sensor.delete(id)
    
    '''
     方法：录入指纹
     @id: 指纹存储位置 1~60 
     @note: 此方法是上述部分API的封装 
        录入指纹基本流程：
            读入指纹图像，生成指纹特征1
            再次读入指纹图像，生成指纹特征2
            若 特征1 与 特征2 匹配，则生成指纹模板并保存
    '''
    fingerprint_sensor.enroll(id)
    
    '''
     方法：验证(识别)指纹  
     @return: 验证成功返回 True
     @note: 此方法是上述部分API的封装 
            验证(识别)指纹指纹基本流程：
                读入指纹图像
                生成指纹特征
                搜索匹配指纹（是否为已保存的指纹）
    '''
    fingerprint_sensor.verify()
    
 
案例
++++++++++++++++++++++++++++++++++++++++++++++++++++++

**1. 指纹录入与识别** 
::

    import time 
    from openaie import button1, button2, fingerprint_sensor


    fingerprint = fingerprint_sensor(1)
     
    id_num = 1

    while (True):
        if button1.is_press(): # 检测到按键按下
            time.sleep_ms(10) # 延时消抖
            if button1.is_press():
                print("\n===============")
                print("  准备录入指纹  ")
                print("===============")
                fingerprint.enroll(id_num)
                id_num+=1
            while (button1.is_press()) : # 等待按键释放
                pass
            
        if button2.is_press(): # 检测到按键按下
            time.sleep_ms(10) # 延时消抖
            if button2.is_press():
                print("\n===============")
                print("  开始指纹识别  ")
                print("===============")
                fingerprint.verify()
            while (button2.is_press()) : # 等待按键释放
                pass
        

**2. 指纹锁** 
::

    # 指纹 录入与验证(识别)
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


 

------------------------------------------------------

        