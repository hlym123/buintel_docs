网络(network)
======================================== 
 
# ref: https://docs.micropython.org/en/latest/library/network.WLAN.html?highlight=wlan

应用编程接口说明  
****************************************
 
::
    
    '''
     类 network.WLAN (无线局域网)
     @interface_id
        network.STA_IF (station aka client, connects to upstream WiFi access points) 
        network.AP_IF  (access point, allows other WiFi clients to connect)
    '''
    class network.WLAN(interface_id)
    
    '''
     Activate (“up”) or deactivate (“down”) network interface
    '''
    WLAN.active([is_active])
    
    '''
     连接到网络 
     @ssid 网络名 
     @key  密码
    '''
    WLAN.connect(ssid=None, key=None, *, bssid=None) 
    
    '''
     断开网络连接 
    '''
    WLAN.disconnect()
    
    '''
     扫描发现可用的网络
     @return (ssid, bssid, channel, RSSI, security, hidden)
      security
        0 – open
        1 – WEP
        2 – WPA-PSK
        3 – WPA2-PSK
        4 – WPA/WPA2-PSK
     hidden
        0 – visible
        1 – hidden
    '''
    WLAN.scan()
    
    
    '''
     Return the current status of the wireless connection.
    '''
    WLAN.status([param])
    
    '''
     
    '''
    WLAN.isconnected()

    '''
    Get/set IP-level network interface parameters: IP address, subnet mask, gateway and DNS server. 
    '''
    WLAN.ifconfig([(ip, subnet, gateway, dns)])


    WLAN.config('param')


案例   
****************************************
	
**联网**

::

    import network

    wlan = network.WLAN(network.STA_IF) # create station interface
    wlan.active(True)       # activate the interface
    res = wlan.scan()             # scan for access points
    for r in res:
        #print("ssid: %s, bssid: %s, channel: %d, RSSI: %d, security: %s, hidden: %d"%r[:])
        print("网络: %-20s 信号强度: %ddb"%(r[0].decode(), r[3]))
     
 
    import network

    wlan = network.WLAN(network.STA_IF) # create station interface
    wlan.active(True)       # activate the interface
    wlan.scan()             # scan for access points
    wlan.isconnected()      # check if the station is connected to an AP
    wlan.connect('ssid', 'key') # connect to an AP
    wlan.config('mac')      # get the interface's MAC address
    wlan.ifconfig()         # get the interface's IP/netmask/gw/DNS addresses

    ap = network.WLAN(network.AP_IF) # create access-point interface
    ap.config(ssid='ESP-AP') # set the SSID of the access point
    ap.config(max_clients=10) # set how many clients can connect to the network
    ap.active(True)         # activate the interface

**Example**: 联网

::

    def do_connect():
        import network
        wlan = network.WLAN(network.STA_IF)
        wlan.active(True)
        if not wlan.isconnected():
            print('connecting to network...')
            wlan.connect('ssid', 'key')
            while not wlan.isconnected():
                pass
        print('network config:', wlan.ifconfig())
        
    
**Example**: 双机通信
通过 socket 通信 

::

    '''
     服务端 设置热点
    '''
    import network

    esp_ap = network.WLAN(network.AP_IF) # create access-point interface
    esp_ap.active(True)         # activate the interface
    esp_ap.ifconfig(('192.168.1.4', '255.255.255.0', '192.168.1.1', '8.8.8.8'))  
    esp_ap.config(essid='DEMO_AP') # set the ESSID of the access point
    print(esp_ap.ifconfig())
    print(" ===================== ")

    import socket, struct   
    server = socket.socket()         # 创建 socket 对象
    sockaddr = socket.getaddrinfo('0.0.0.0', 9002)[0][-1] #  '0.0.0.0'
    # 完成名字到地址以及服务到端口这两种转换，返回的是一个addrinfo的结构（列表）指针而不是一个地址清单。这些addrinfo结构随后可由套接口函数直接使用。
    # 0.0.0.0并不是一个真实的的IP地址，它表示本机中所有的IPV4地址。监听0.0.0.0的端口，就是监听本机中所有IP的端口。
    server.bind(sockaddr)
    server.listen(5)    # 开始监听
    cnt = 0
    while True:    # conn就是客户端链接过来而在服务端为期生成的一个链接实例
        conn, addr = server.accept()    # 等待链接,多个链接的时候就会出现问题,其实返回了两个值
        print(conn, addr)
        data = conn.recv(1024)    # 接收数据
        
        if len(data) > 10:
            temperature = struct.unpack('>f', data[2:6])[0]
            humidity = struct.unpack('>f', data[6:10])[0]
            count = struct.unpack('>H', data[10:12])[0]
            print("%d -- temp: %.1fC -- humi: %.1f%%"%(count, temperature, humidity))
        
        #print('recive:', data.decode('utf-8'))    # 打印接收到的数据
        msg = str(cnt) + ' - data from server'
        
        cnt += 1
        conn.send(msg.encode('utf-8'))    # 然后再发送数据
        conn.close()


    
::
    
    '''
     客户端
    '''
    import network
    
	def do_connect(essid, password):
		import network
		esp_sta = network.WLAN(network.STA_IF)  
		esp_sta.active(True) 
		if esp_sta.isconnected() == True:
			print('network has connected')
			return
		print('connecting to network...')
		esp_sta.connect(essid, password) 
		while not esp_sta.isconnected() :
			pass
		print('connection successful')
		print('network config:', esp_sta.ifconfig()) 
		
	do_connect('DEMO_AP', '')


	import socket, time, struct

	send_buf = bytearray(20)

	addr = socket.getaddrinfo('192.168.1.4', 9002)[0][-1] # 链接IP地址
	for i in range(5):
		client = socket.socket()
		client.connect(addr)
		
		send_buf[:2] = 'TX'.encode()
		send_buf[2:6] = struct.pack('>f', 35.6+i*0.1)
		send_buf[6:10] = struct.pack('>f', 65.8+i)
		send_buf[10:12] = struct.pack('>H', i)
		client.send(send_buf)  
		
		#msg = str(i) + ' - data from client. 你好，这是来自......' # strip默认取出字符串的头尾空格
		#client.send(msg.encode('utf-8'))      # 发送一条信息 python3 只接收btye流
		
		data = client.recv(1024)             # 接收一个信息，并指定接收的大小 为1024字节
		print('recv:', data.decode('utf-8'))         # 输出我接收的信息
		time.sleep(1)
	client.close() # 关闭这个链接 




**Example**: 网络时间同步 

::

    # m_ntptime.py
    try:
        import usocket as socket
    except:
        import socket
    try:
        import ustruct as struct
    except:
        import struct

    # (date(2000, 1, 1) - date(1900, 1, 1)).days * 24*60*60
    NTP_DELTA = 3155673600-8*60*60 # 时区:东8区

    # The NTP host can be configured at runtime by doing: ntptime.host = 'myhost.org'
    host = "pool.ntp.org"


    def time():
        NTP_QUERY = bytearray(48)
        NTP_QUERY[0] = 0x1B
        addr = socket.getaddrinfo(host, 123)[0][-1]
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        try:
            s.settimeout(5)
            res = s.sendto(NTP_QUERY, addr)
            msg = s.recv(48)
        finally:
            s.close()
        val = struct.unpack("!I", msg[40:44])[0]
        return val - NTP_DELTA


    # There's currently no timezone support in MicroPython, and the RTC is set in UTC time.
    def settime():
        t = time()
        import machine
        import utime

        tm = utime.gmtime(t)
        machine.RTC().datetime((tm[0], tm[1], tm[2], tm[6] + 1, tm[3], tm[4], tm[5], 0))

::

    from machine import RTC
    import time

    rtc = RTC()
    rtc.datetime((2021, 4, 2, 5, 9, 39, 0, 0))

    '''
     网络连接
    '''
    def do_connect(essid, password):
        import network
        esp_sta = network.WLAN(network.STA_IF) # 工作站模式
        esp_sta.active(True) # 激活网络接口
        if esp_sta.isconnected() == True:
            print('network has connected')
            return
        print('connecting to network...')
        esp_sta.connect(essid, password) # 连接到网络
        while not esp_sta.isconnected() :
            pass
        print('connection successful')
        print('network config:', esp_sta.ifconfig()) 

    do_connect('xuchang', 'zzyue1998@')

    '''
     网络时间同步
    '''
    import m_ntptime as ntptime 

    print("同步前本地时间：%s" %str(time.localtime()))
    ntptime.settime()
    print("同步后本地时间：%s" %str(time.localtime()))
    rt = time.localtime()
    week = ("星期一", "星期二", "星期三", "星期四", "星期五", "星期六", "星期日")
    print("%d年%d月%d日 "%rt[0:3], "%s"%week[rt[6]], " %02d:%02d:%02d"%rt[3:6])

    while True:
        rt = time.localtime()
        print("%d年%d月%d日 "%rt[0:3], "%s"%week[rt[6]], " %02d:%02d:%02d"%rt[3:6])
        time.sleep_ms(1000)


    
**Example**: 天气预报获取 

::

    import urequests
    import ure as re

    # https://www.tianqi.com/
    local = 'baoanqu' # 深圳市宝安区
    url = 'http://i.tianqi.com/index.php?c=code&a=getcode&id=55&py=' + local
    r = urequests.get(url)  
    content = r.text

    date = re.search(r'<li class="t3">(.*?)</li>', content)
    date = date.group(1)
    print ("日期：", date)
     
    weather = re.search(r'height: 18px;overflow: hidden;">(.*?)</span>', content)
    weather = weather.group(1)
    print("天气: ", weather)

    temp = re.search(r'<h5><span class="f1">(.*?)</span>~<span class="f2">(.*?)</span></h5>', content)
    temp1 = temp.group(1)
    temp2 = temp.group(2)
    print ("温度： %s ~ %s ℃"%(temp1, temp2))

    index = re.search(r'height:36px"><h4>(.*?)</h4><p>(.*?)</p></a></div>', content)
    print("指数: ", index.group(1), index.group(2))

	
	