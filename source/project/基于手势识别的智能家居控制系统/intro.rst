基于手势识别的智能家居控制系统 
======================================================  



器材:

* JetsonNano
* USB摄像头
* 便捷式HDMI显示屏
* 控制器：开元 
* 模块：可编程全彩LED；电机风扇；舵机云台 


程序(JetsonNano 端)  
++++++++++++++++++++++++++++++++++++++++++++++++++++++
检测识别手势

::
    
    '''
     @20221110 
     
     手势识别智能家居控制
     
     风扇开关控制 
     风扇方向角度控制
     灯开关控制 

     'TX' + 设备控制1 + 设备控制2 
        0x01 获取控制
        0x10 开/关灯 
        0x11 开/关风扇    
        0x12 风扇摇头     
    '''
    import cv2
    import time
    import math
    import mediapipe as mp
    #import pygame 
    import serial


    mp_drawing = mp.solutions.drawing_utils
    mp_hands = mp.solutions.hands


    finger_landmark_point = ((0,1,2,3,4),
                             (0,5,6,7,8),
                             (0,9,10,11,12),
                             (0,13,14,15,16),
                             (0,17,18,19,20))

    '''
     手势定义
    '''
    gesture_def = [
                    {'stat':[0,1,0,0,0], 'label':'one'},
                    {'stat':[0,1,1,0,0], 'label':'two'},
                    {'stat':[0,1,1,1,0], 'label':'three'},
                    {'stat':[0,1,1,1,1], 'label':'four'},
                    {'stat':[1,1,1,1,1], 'label':'five'},
                    {'stat':[0,0,1,1,1], 'label':'ok'},
                    {'stat':[1,1,0,0,1], 'label':'love you'},
                    {'stat':[0,0,0,0,0], 'label':'fist'},
                    ]

    gesture_ctrl_def = [
                        {'stat':[0,1,0,0,0], 'label':'one'},
                        {'stat':[0,1,1,0,0], 'label':'moror_fan_angle'}, # 风扇角度控制 
                        {'stat':[0,1,1,1,0], 'label':'three'},
                        {'stat':[0,1,1,1,1], 'label':'four'},
                        {'stat':[1,1,1,1,1], 'label':'five'},
                        {'stat':[0,0,1,1,1], 'label':'ok'},
                        {'stat':[1,1,0,0,1], 'label':'love you'},
                        {'stat':[0,0,0,0,0], 'label':'fist'},
                        ]

    '''
     计算矢量夹角
     @v1 [x0,y0,x1,y1]
     @v2 [x0,y0,x1,y1]
     @return 矢量夹角(°)
    '''
    def vector_included_angle(v1, v2):
        dx1 = v1[2] - v1[0]
        dy1 = v1[3] - v1[1]
        dx2 = v2[2] - v2[0]
        dy2 = v2[3] - v2[1]
        v1_angle = math.atan2(dy1, dx1)/math.pi*180 # 弧度转度
        v2_angle = math.atan2(dy2, dx2)/math.pi*180
        if v1_angle*v2_angle >= 0:
            angle = abs(v1_angle-v2_angle)
        else:
            angle = abs(v1_angle) + abs(v2_angle)
            if angle > 180:
                angle = 360 - angle
        return angle 

    '''
     创建手检测器
    '''
    hands = mp_hands.Hands(
            #model_complexity=0,
            min_detection_confidence=0.5,
            min_tracking_confidence=0.5)    

    '''
     打开并设置摄像头
    '''
    cap = cv2.VideoCapture(0)
    if not cap.isOpened():
            print("open camera failed!")
            exit(-1)
    # 设置分辨率 720P:1280x720; 1080P:1920x1080
    cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280) 
    cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 720)
    # 设置帧率
    cap.set(cv2.CAP_PROP_FPS, 30)
    # 设置曝光
    cap.set(cv2.CAP_PROP_EXPOSURE, 2)

    last_replay_time = 0
    last_sw_time = 0
    last_res = 0
    #pygame.mixer.init()
    err_cnt = 0

    '''
     打开串口
    '''
    com = serial.Serial('/dev/ttyTHS1', 115200, timeout=10)
    tx_buf = bytearray(5)
    tx_buf[:2] = 'TX'.encode() # 帧头

    light_on = False
    flag_stable = False
    while True:     
        # 读取视频帧
        success, image = cap.read()
        if not success:
            err_cnt += 1
            if err_cnt > 3:
                break
            print("frame read error!")
            continue
        image_h, image_w = image.shape[0:2]
        raw_image = image.copy()

        # 获取检测结果
        results = hands.process(cv2.cvtColor(image, cv2.COLOR_BGR2RGB))

        # 如果检测结果不为空
        if results.multi_hand_landmarks:
            handedness = results.multi_handedness
            for hand_index, hand_landmarks in enumerate(results.multi_hand_landmarks):

                # 绘制 landmarks
                mp_drawing.draw_landmarks(
                    image,
                    hand_landmarks,
                    mp_hands.HAND_CONNECTIONS)

                # 获取手掌属性 左或右
                palm_type = handedness[hand_index].classification[0].label
                palm_type = 'Left' if (palm_type == 'Right') else 'Right' # 镜像反转

                # 获取手掌各关键点坐标
                landmark_x = []
                landmark_y = []
                for p in hand_landmarks.landmark:
                    landmark_x.append(math.ceil(p.x*image_w))
                    landmark_y.append(math.ceil(p.y*image_h))

                # 手掌边界框
                x1 = min(landmark_x) 
                x2 = max(landmark_x) 
                y1 = min(landmark_y) 
                y2 = max(landmark_y) 
                palm_bbox = x1,y1,x2,y2
                c = (0xe7,0xed,0x3e) if palm_type == "Left" else (0x83,0x79,0xf4)
                cv2.rectangle(image, (x1,y1), (x2,y2), c, 2)
                #cv2.putText(image, "{}".format(palm_type), (x1, y1-10), cv2.FONT_HERSHEY_COMPLEX, 1.0, c, 2)

                # 手指状态(弯曲或伸直)计算
                finger_stat = []
                for i in range(5):
                    p = finger_landmark_point[i]
                    angle_sum = 0 
                    for i in range(3):
                        v1 = [landmark_x[p[i]], landmark_y[p[i]], landmark_x[p[i+1]], landmark_y[p[i+1]]]
                        v2 = [landmark_x[p[i+1]], landmark_y[p[i+1]],landmark_x[p[i+2]], landmark_y[p[i+2]]]
                        a = vector_included_angle(v1, v2)
                        angle_sum += a
                    if angle_sum > 45: # 根基累加角度判断手指状态，数值自己测试确定
                        finger_stat.append(0)
                    else:
                        finger_stat.append(1)

                # 搜索匹配手势
                for i in range(len(gesture_def)):
                    if finger_stat == gesture_def[i]['stat']:
                        if i != last_res: # 前后两次识别结果不同
                            last_sw_time = time.time() # 记录当前时间
                            last_replay_time = 0    # 重置语音重播时间
                        # 防止在手势动作变换时播放（保持同一手势超过 1S 判断为手势稳定）；以及防止同一结果短时间内重复播放
                        if time.time() - last_sw_time > 0.5 and time.time() - last_replay_time > 2:
                            last_replay_time = time.time() # 记录语音重播时间
                            #sound = pygame.mixer.Sound("media/%d.mp3"%(i+1)) # 播放语音
                            #sound.play()
                            flag_stable = True
                        else:
                            flag_stable = False
                        last_res = i
                        cv2.putText(image, gesture_def[i]['label'], (x1, y1-10), cv2.FONT_HERSHEY_COMPLEX, 1.0, c, 2)

                try: 
                    # 获取控制权 love you 
                    if finger_stat == gesture_ctrl_def[6]['stat']:     
                        cv2.putText(image, "control", (20, 20), cv2.FONT_HERSHEY_COMPLEX, 1.0, (0xFF,0x00,0x00), 2)
                        tx_buf[2] = 0x01 
                        if flag_stable: 
                            com.write(tx_buf)
                    # 开/关灯 ok 
                    elif finger_stat == gesture_ctrl_def[5]['stat']:
                        tx_buf[2] = 0x10 
                        if flag_stable: 
                            com.write(tx_buf)
                    # 风扇控制 two  
                    elif finger_stat == gesture_ctrl_def[1]['stat']:
                        # 当(无名指，尾指弯曲，食指和中指伸直且并拢)时   控制风扇的摆动角度
                        if abs(landmark_x[8] - landmark_x[12]) < 100 and abs(landmark_y[8] - landmark_y[12]) < 50:  
                            tx_buf[2] = 0x12
                            tx_buf[3] = int(landmark_x[8]/10)
                            cv2.putText(image, str(landmark_x[8]), (x1, y1-50), cv2.FONT_HERSHEY_COMPLEX, 1.0, (0xFF,0x00,0x00), 2)
                            cv2.putText(image, str(landmark_y[8]), (x1 + 100, y1-50), cv2.FONT_HERSHEY_COMPLEX, 1.0, (0xFF,0x00,0x00), 2)                  
                            com.write(tx_buf)
                        else:
                            tx_buf[2] = 0x11 
                            if flag_stable:                   
                                com.write(tx_buf)
                except:
                    pass 
                
        # 显示
        image = cv2.flip(image, 1)
        cv2.imshow('Hands', image)
        if cv2.waitKey(1) & 0xFF == ord('q'): # 按 ESC 键退出
            break

    cap.release()
    cv2.destroyAllWindows()


程序(开元主控端)  
++++++++++++++++++++++++++++++++++++++++++++++++++++++
根据识别到的手势控制执行

::

    '''
     @20221110 
     
     手势识别智能家居控制
     
     风扇开关控制 
     风扇方向角度控制
     灯开关控制 

     'TX' + 设备控制1 + 设备控制2 
      0x01 获取控制
      0x10 开/关灯 
      0x11 开/关风扇  
      0x12 风扇摇头   
    '''
    import time
    import lcd
    from machine import UART, Pin
    from openaie import math_map, servo, rgb_led, motor_fan, buzzer



    light = rgb_led(5) # 全彩LED -- 端口5
    m = motor_fan(8)   # 电机风扇 -- 端口8
    m.set(0)
    s = servo(6)       # 舵机 -- 端口6 
    s.write(90)


    serial = UART(0, baudrate=115200, tx=Pin(16), rx=Pin(17))  


    flag_ctrl = False  
    flag_light = False  
    flag_fan = False 
    last_time_ctrl_on = 0
    lcd.clear(color=0)
    lcd.rotation(0)
    lcd.draw_string(40, 10, "手势识别智能家居控制", fc=(0,0,255), bc=0)
    lcd.draw_line(0, 36, 219, 36, color=(0,0,255), thickness=5)
    lcd.draw_string(10, 50, "灯: 关", fc=(255,0,0), bc=0)
    lcd.draw_string(130, 50, "风扇: 关", fc=(255,0,0), bc=0)
    lcd.draw_string(88, 182, "停止控制", fc=(255,0,0), bc=0)
    lcd.display()
    while True:
        if serial.any() > 0:
            rx_buf = serial.read() 
            try:
                if rx_buf[:2].decode('utf-8') == 'TX': # 帧头判断
                    if rx_buf[2] == 0x01: # 获取控制  
                        flag_ctrl = True 
                        lcd.draw_circle(119, 190, 80, thickness=8, color=(0,255,0), fill=False)
                        lcd.draw_string(88, 182, "开始控制", fc=(0,255,0), bc=0)
                        buzzer.tone(345)
                        time.sleep_ms(200)
                        buzzer.no_tone()
                        last_time_ctrl_on = time.ticks_ms()
                    if flag_ctrl: # 当获取控制时 
                        if rx_buf[2] == 0x10: # 开/关灯 
                            flag_ctrl = False
                            lcd.draw_circle(119, 190, 80, thickness=8, color=(0,0,0), fill=False)
                            lcd.draw_string(88, 182, "停止控制", fc=(255,0,0), bc=0)
                            flag_light = not flag_light
                            if flag_light:
                                light.set(0, (100,100,100))
                                lcd.draw_string(10, 50, "灯: 开", fc=(0,255,0), bc=0)
                            else:
                                lcd.draw_string(10, 50, "灯: 关", fc=(255,0,0), bc=0)
                                light.set(0, (0,0,0))
                            light.display()
                        elif rx_buf[2] == 0x11: # 开/关风扇
                            flag_ctrl = False
                            lcd.draw_circle(119, 190, 80, thickness=8, color=(0,0,0), fill=False)
                            lcd.draw_string(88, 182, "停止控制", fc=(255,0,0), bc=0)
                            flag_fan = not flag_fan
                            print(flag_fan)
                            if flag_fan:
                                m.set(50)
                                lcd.draw_string(130, 50, "风扇: 开", fc=(0,255,0), bc=0)
                            else:
                                m.set(0)
                                lcd.draw_string(130, 50, "风扇: 关", fc=(255,0,0), bc=0)
                        elif rx_buf[2] == 0x12: # 风扇摇头
                            last_time_ctrl_on = time.ticks_ms()
                            angle = math_map(rx_buf[3], 20, 120, 150, 30)
                            s.write(angle)
                lcd.display()
            except:
                pass 
                
        if flag_ctrl:
            if time.ticks_diff(time.ticks_ms(), last_time_ctrl_on) > 6000: # 超时自动关闭控制
                flag_ctrl = False
                lcd.draw_circle(119, 190, 80, thickness=8, color=(0,0,0), fill=False)
                lcd.draw_string(88, 182, "停止控制", fc=(255,0,0), bc=0)
                lcd.display()
                
        time.sleep_ms(50)
          


------------------------------------------------------
