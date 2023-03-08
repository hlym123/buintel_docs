快速参考
========================================


延时和计时 
****************************************

::

    '''
     导入 time 模块 
    '''
    import time

    '''
     秒级延时
    '''
    time.sleep(seconds)            
    
    '''
     毫秒级延时
    '''
    time.sleep_ms(ms)       
    
    '''
     微秒级延时
    '''
    time.sleep_us(us)        
    
    '''
     获取运行累计时间（系统内计数器，上电后每隔1ms加1）
    '''
    start = time.ticks_ms() 
    
    '''
     计算时间和  
    '''
    deadline = time.ticks_add(ticks, delta)  
        
    '''
     计算时间差
    '''
    delta = time.ticks_diff(ticks1, ticks2)  

    
**Example**: 通过 time.ticks_ms() 实现一定时间的延时   
::
  
    import time

    cnt = 0
    delay_time = 500 # 500ms 
    deadline = time.ticks_add(time.ticks_ms(), delay_time) 
    while True:
        if time.ticks_diff(time.ticks_ms(), deadline) > 0:
            deadline = time.ticks_add(time.ticks_ms(), delay_time)
            print("count: ", cnt)
            cnt += 1
            
    '''
     ...
    '''

         
         
.. note:: 当通过 time.sleep() 实现延时时，程序会完全停下等待；而通过判断 time.ticks_ms() 的值实现延时更高效。
 


定时器   
****************************************    

::

    '''
     从 machine 中导入 Timer 模块
    '''
    from machine import Timer

    '''
     创建定时器
     @period: 定时周期(ms)
     @mode: 定时器模式 
        Timer.ONE_SHOT 单次
        Timer.PERIODIC 周期重复 
     @callback: 回调函数，当定时时间到来时会自动调用
    '''
    tim = Timer(period, mode=Timer.PERIODIC, callback)
    
**Example**: 定时器使用     
::

    import time
    from machine import Timer

    cnt = 0
    def show(n):
        global cnt
        print("count: ", cnt)

    # 创建单次定时器，定时时间为3秒
    tim1 = Timer(period=3000, mode=Timer.ONE_SHOT, callback=lambda t:print("3秒单次定时"))

    # 创建周期定时器，定时间隔为1秒
    tim2 = Timer(period=1000, mode=Timer.PERIODIC, callback=show)
     
    while True:
        cnt += 1
        time.sleep_ms(100)
     

引脚控制     
****************************************    

::

    '''
     从 machine 中导入 Pin
    '''
    from machine import Pin

    '''
     Pin 类
     @id: 引脚号
     @mode: 模式
        Pin.IN  输入 
        Pin.OUT 输出 
     @pull:
        Pin.None 
        Pin.PULL_UP   上拉
        Pin.PULL_DOWN 下拉
    '''
    class machine.Pin(id, mode=- 1, pull=- 1)
    
    '''
     获取或设置引脚电平
     @x: 0--低电平，1--高电平
     @return：引脚电平  
    '''
    Pin.value(x)
    
    '''
     中断
     @handle: 中断回调
     @trigger: 中断触发方式
        Pin.IRQ_FALLING     下降沿（高电平到低电平变化）
        Pin.IRQ_RISING      上升沿（低电平到高电平变化）
        Pin.IRQ_LOW_LEVEL   低电平
        Pin.IRQ_HIGH_LEVEL  高电平
    '''
    Pin.irq(handler=None, trigger=Pin.IRQ_FALLING|Pin.IRQ_RISING)

**Example**: LED控制 

板载 LED 接引脚 25，高电平亮
::

    import time
    from machine import Pin

    led = Pin(25, Pin.OUT)
    while True:
        led.value(1)
        time.sleep_ms(200)
        led.value(0)
        time.sleep_ms(800)
        
**Example**: 按键检测（循环检测）

板载按键1连接引脚21，按键二连接引脚20，按下接低电平。
::         

    import time
    from machine import Pin  

    pin_key1 = Pin(21, Pin.IN, Pin.PULL_UP) # 上拉输入模式 
    pin_key2 = Pin(20, Pin.IN, Pin.PULL_UP)  

    while True:
        if not pin_key1.value(): # 检测到按键按下
            time.sleep_ms(10) # 延时消抖
            if not pin_key1.value():
                print("button 1 press")
            while (not pin_key1.value()) : # 等待按键释放
                pass
        if not pin_key2.value():  
            time.sleep_ms(10)  
            if not pin_key2.value():
                print("button 2 press")
            while (not pin_key2.value()) :  
                pass    

**Example**: 按键检测（中断方式）
::

    import time
    from machine import Pin  

    pin_key1 = Pin(21, Pin.IN, Pin.PULL_UP) # 上拉输入模式 
    pin_key2 = Pin(20, Pin.IN, Pin.PULL_UP) # 上拉输入模式 

    def key1(n):
        print("key1 press")
         
    pin_key1.irq(handler=key1, trigger=Pin.IRQ_FALLING) # 下降沿触发

    pin_key2.irq(handler=lambda key2: print("检测到按键2按下"), trigger=Pin.IRQ_FALLING) # 下降沿触发

    while True:
        pass
     
            
串口控制  
**************************************** 
(UART, Universal Asynchronous Receiver/Transmitter)

**应用编程接口说明**
::
    '''
     从 machine 中导入 UART 
    '''
    from machine import UART
    
    '''
     类 UART
     @id: 0、1
     @baudrate: 波特率  
        9600 
        57600 
        115200
        ... 
     @tx: 发送引脚 
     @rx：接收引脚 
    '''
    class machine.UART(id, baudrate, tx, rx)
    
    '''
     读取接收缓冲区数据大小
     @return 缓冲区接收数据字节数（大于1为接收到数据） 
    '''
    UART.any()
    
    '''
     读取数据 
     @nbytes 读取数据大小（可选参数，为0表示读取所有数据）
    '''
    UART.read([nbytes])
    
    '''
     读取数据到buf中
     @buf 读取的数据
     @nbytes 读取数据大小（可选参数，为0表示读取所有数据）
     @return 实际读取数据的字计数
    '''
    UART.readinto(buf[, nbytes])
    
    '''
     读取一行数据（遇到换行符结束）
     @return 读取到的数据 
    '''
    UART.readline()
    
    '''
     写数据 
     @buf 要发送的数据 
     @return 实际发送的字节数 
    '''
    UART.write(buf)
    
**Example:** 接收数据 
::

    from machine import UART, Pin
    
    # 开元主控端口1 
    uart0 = UART(0, baudrate=115200, tx=Pin(16), rx=Pin(17)) 
    
    # 开元主控端口7
    #uart1 = UART(1, baudrate=115200, tx=Pin(4), rx=Pin(5)) 
    
    uart0.write('hello')   
    while True:
        if uart0.any() > 0 # 若接收到数据 
            rx = uart0.read() # 读取数据  
            print(rx)
    
集成电路总线 
****************************************   
(I2C, Inter-Integrated Circuit)

**应用编程接口说明** 
::

    '''
     从 machine 中导入 SoftI2C 
    '''
    from machine import SoftI2C
    
    '''
     类 SoftI2C
     @scl 串行时钟 
     @sda 串行数据
     @freq 传输速率（最大400000）
     @timeout 可选参数，应答超时时间（ms）
    '''
    class machine.SoftI2C(scl=Pin(scl), sda=Pin(sda), freq=100000, [timeout])
    
    '''
     扫描设备
     @return 扫描结果 
    '''
    SoftI2C.scan()
    
    '''
     写数据 
     @addr 设备地址 
     @buf 要写入的数据        
    '''
    SoftI2C.writeto(addr, buf)
    
    '''
     写数据到寄存器中 
     @addr 设备地址
     @memaddr 寄存器地址 
     @buf 要写入的数据 
    '''
    SoftI2C.writeto_mem(addr, memaddr, buf)
    
    ''' 
     读数据到 buf 中 
     @addr 设备地址 
     @buf 读取的数据（读取数据字节数由buf长度决定） 
        buf 为 bytearray() 对象 
        
     Example:
        nbytes = 4
        buf = bytearray(nbytes)
        i2c.readfrom_into(0x68, buf) # 读取4字节数据 
    '''
    SoftI2C.readfrom_into(addr, buf)
    
    '''
     读取寄存器数据到 buf 中 
     @addr 设备地址 
     @memaddr 寄存器地址 
     @buf 读取的数据（读取数据字节数由buf长度决定）

     Example:
        nbytes = 4
        buf = bytearray(nbytes)
        i2c.readfrom_mem_into(0x68, 0x01, buf) # 读取4字节数据(寄存器地址自动递增)      
    '''
    SoftI2C.readfrom_mem_into(addr, memaddr, buf)

**Example:** 扫描发现总线中的设备    
:: 

    from machine import SoftI2C, Pin
    from openaie import get_port_pin

    # 端口6 
    scl_pin = get_port_pin(6, 1)
    sda_pin = get_port_pin(6, 2)

    # 创建并初始化I2C总线
    i2c = SoftI2C(scl=Pin(scl_pin), sda=Pin(sda_pin), freq=100000)

    res = i2c.scan() # 扫描发现总线中的设备
    cnt = 1
    for r in res: # 逐个结果打印 
        print("发现设备%d，地址：%#x"%(cnt, r))
        cnt += 1
   
模数转换   
****************************************    
(ADC, Analog-to-Digital Converter)
ADC检测电压不能超过3.3V

**应用编程接口说明** 
::

    '''
     从 machine 中导入 ADC 
    '''
    from machine import ADC
    
    '''
     创建 ADC 对象 
     @pin_num 引脚号 
        26(ADC0) -- 开元主控端口3，信号线2
        27(ADC1) -- 开元主控端口3，信号线1 
        28(ADC2) -- 开元主控端口4，信号线1  
    '''
    adc = ADC(Pin(pin_num))     
    
    '''
     读取ADC转换结果
     @return 0-65535 对应 0.0~3.3V
    '''
    adc.read_u16()     

**Example:** 读取打印ADC结果     
::

    from machine import ADC, Pin 
    import time  

    adc1 = ADC(Pin(27)) # 开元主控端口3，信号线1 
    while True:
        r = adc1.read_u16()
        print("ADC value: ", r)
        time.sleep_ms(100)


实时时钟
****************************************    
(RTC, Real Time Clock)  
 
::
    
    '''
     从 machine 导入 RTC 模块
    '''
    from machine import RTC

    '''
     RTC 类
    '''
    class RTC()
    
    '''
     设置或获取日期和时间 
     @week: 0~6 -- 星期一~星期日
    '''
    RTC.datetime((year, month, day, week, hour, minute, second, 0))  
                                         
**Example**: 时间显示 
::

    import lcd, time
    from machine import RTC
    import _thread as thread

    rtc = RTC()
    rtc.datetime((2022, 4, 1, 4, 8, 47, 0, 0)) # 2022/4/1 Friday 08:09:00

    lcd.rotation(0) # 设为竖屏显示

    week = ("星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期日")

    delay_time = 1000 # 1000ms 
    deadline = time.ticks_add(time.ticks_ms(), delay_time)
    while True:
        if time.ticks_diff(time.ticks_ms(), deadline) > 0:
            deadline = time.ticks_add(time.ticks_ms(), delay_time)
            now = rtc.datetime()
            date_info = "%d-%02d-%02d"%(now[:3])
            time_info = " %02d:%02d:%02d "%(now[4:7])
            lcd.clear(color=0)
            lcd.draw_string(10, 10, date_info+time_info+week[now[3]], fc=(0,0,255), bc=0)
            lcd.display()
  
                                     
看门狗定时器   
****************************************        
(WDT, Watch Dog Timer)

看门狗定时器是主控内一个外设功能，主要功能是当系统出现故障时，使系统复位重新运行。
当使能看门狗定时器时，定时器的计数值不断递增，计数值大于设定值时将使系统复位。
当程序正常运行时在一定时间间隔内喂狗（清零计数值），定时器将重新开始计数。

::

    '''
     导入 WDT 模块 
    '''
    from machine import WDT

    '''
     初始化看门狗，超时时间为5000ms 
    '''
    wdt = WDT(timeout=5000)
    
    '''
     喂狗（清零计数值）
    '''
    wdt.feed()
    
**Example**: 按键按下执行喂狗操作，当按键按下间隔大于看门狗超时时间时，系统复位，程序重新运行。
::

    from machine import WDT
    import time, lcd
    from openaie import button1

    lcd.rotation(0)

    # 初始化看门狗，超时时间为3000ms 
    wdt = WDT(timeout=3000)

    cnt = 0
    delay_time = 200 
    deadline = time.ticks_add(time.ticks_ms(), delay_time)
    while True:
        if time.ticks_diff(time.ticks_ms(), deadline) > 0:
            deadline = time.ticks_add(time.ticks_ms(), delay_time)
            print("cnt: ", cnt)
            lcd.clear(color=(0,0,0))
            lcd.draw_string(10, 10, "count: %d"%cnt, fc=(0,255,0), bc=0)
            lcd.display()
            cnt += 1
        
        if button1.is_press(): # 检测到按键按下
            time.sleep_ms(10) # 延时消抖
            if button1.is_press():
                wdt.feed()
                print("watch dog feed")
            while (button1.is_press()) : # 等待按键释放
                pass
 
  
多线程 
****************************************    

::

    import _thread 
    
    
 








    
  


    
    