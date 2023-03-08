多线程(_thread) 
======================================== 
 
# ref: https://docs.python.org/zh-cn/3/library/_thread.html


应用编程接口说明  
****************************************

::

    '''
    开启一个新线程并返回其标识。 
    线程执行函数 function 并附带参数列表 args (必须是元组)。 可选的 kwargs 参数指定一个关键字参数字典。
    '''
    _thread.start_new_thread(function, args[, kwargs])

    '''
    返回一个新的锁对象。锁中的方法在后面描述。初始情况下锁处于解锁状态。
    '''
    lock = _thread.allocate_lock()

    '''
     申请获得锁，有必要的话会等待其他线程释放锁
    '''
    lock.acquire()

    '''
     释放锁。锁必须已经被获取过，但不一定是同一个线程获取的。
    '''
    lock.release()

    '''
     返回锁的状态：如果已被某个线程获取，返回 True，否则返回 False。
    '''
    lock.locked()

    '''
    锁对象的类型。
    '''
    _thread.LockType


案例   
****************************************
    
::

    import time, _thread

    cnt = 0

    # create lock
    lock = _thread.allocate_lock()

    lock.acquire()  
    def increase(d):
        global cnt
        while True:
            cnt += 1
            time.sleep_ms(d)
            lock.release()
            
    def print_count(d):
        global cnt
        while True:
            lock.acquire()  
            print("count: ", cnt)
     
    _thread.start_new_thread(increase, (1000, ))
    _thread.start_new_thread(print_count, (2000, ))




