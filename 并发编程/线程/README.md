# 线程

GIL全局锁实际上把所有线程的执行代码都给上了锁，所以，多线程在Python中只能交替执行，即使100个线程跑在100核CPU上，也只能用到1个核。

## 创建多线程

### 方法

- 函数
- 类

#### 函数

```
import  threading
threading.Thread( target=调用方法, args=对方法传参 ) 
```

#### 类

- 必须继承 `threading.Thread` 这个父类；
- 必须复写 `run` 方法

这里的 `run` 方法，和我们上面`线程函数`的性质是一样的，可以写我们的业务逻辑程序。在 `start()` 后将会调用

## 线程对象的方法

```
# 启动子线程
t.start()

# 阻塞子线程，待子线程结束后，再往下执行
# 子线程完成运行之前，这个子线程的父线程将一直被阻塞。
# 注意:join()方法的位置是在for循环外的，也就是说必须等待for循环里的线程都结束后，才去执行主线程。
t.join()

# 判断线程是否在执行状态，在执行返回True，否则返回False
t.is_alive()
t.isAlive()

# 设置线程是否随主线程退出而退出，默认为False
t.daemon = True
t.daemon = False

# 设置线程名
t.name = "My-Thread"

setDaemon(True)将线程声明为守护线程 , 必须在start() 方法调用之前设置，如果不设置为守护线程程序会被无限挂起。 
```

## 线程锁

### 互斥锁

#### 加锁

```
# 生成锁对象，全局唯一
lock = threading.Lock()
```

#### 获取钥匙🔑

```
# 获取锁。未获取到会阻塞程序，直到获取到锁才会往下执行
lock.acquire()
```

#### 释放锁

```
# 释放锁，归还锁，其他人可以拿去用了
lock.release()
```

lock.acquire() 和 lock.release()必须成对出现。否则就有可能造成死锁。

为了规避这个问题。我推荐使用使用上下文管理器来加锁

```
lock = threading.Lock()
# with 语句会在这个代码块执行前自动获取锁，在执行结束后自动释放锁
with lock:
    # 这里写自己的代码
    pass
```

### 可重入锁（RLock）

有时候在同一个线程中，我们可能会多次请求同一资源，俗称锁嵌套。按照常规的做法，会造成死锁的

可重入锁（RLock），只在同一线程里放松对锁(通行证)的获取，意思是，只要在同一线程里，程序就当你是同一个人，这个锁就可以复用，其他的话与`Lock`并无区别

RLock内部维护着一个Lock和一个counter变量，counter记录了acquire的次数，从而使得资源可以被多次require。直到一个线程所有的acquire都被release，其他的线程才能获得资源。 

```
信号量
Semaphore管理一个内置的计数器，
每当调用acquire()时内置计数器-1；
调用release() 时内置计数器+1；
计数器不能小于0；当计数器为0时，acquire()将阻塞线程直到其他线程调用release()。
进程池从头到尾只有规定的进程数量,而信号量是产生一堆线程/进程
```

### 防止死锁的加锁机制

死锁通常以下几种

- 同一线程，嵌套获取同把互斥锁，造成死锁

RLock

- 多个线程，不按顺序同时获取多个锁。造成死锁

```
线程1，嵌套获取A,B两个锁，线程2，嵌套获取B,A两个锁。 由于两个线程是交替执行的，是有机会遇到线程1获取到锁A，而未获取到锁B，在同一时刻，线程2获取到锁B，而未获取到锁A。由于锁B已经被线程2获取了，所以线程1就卡在了获取锁B处，由于是嵌套锁，线程1未获取并释放B，是不能释放锁A的，这是导致线程2也获取不到锁A，也卡住了。两个线程，各执一锁，各不让步。造成死锁。
```

**join 和 互斥锁 都可以实现 串行运行的效果**

线程抢 GIL锁(相当于 执行权限),  -->  拿到执行权限后 才能拿到 互斥锁(lock), 如果lock没有被释放则堵塞,权限会立刻交出来.

GIL锁是解释器级别的,保护的是解释器级别的数据,比如垃圾回收的数据

lock是保护自己开发的应用程序的数据

**join 和 互斥锁 的区别**

join 任务里的所有代码 都是串行执行的;

加锁 只是加锁的部分即修改共享数据的部分 是串行的

## 创建线程池

多线程处理任务时也不是线程越多越好，由于在切换线程的时候，需要切换上下文环境，依然会造成cpu的大量开销

### 内置模块

创建线程池是通过`concurrent.futures`函数库中的`ThreadPoolExecutor`类来实现

```
import time
import threading
from concurrent.futures import ThreadPoolExecutor

def target():
    for i in range(5):
        print('running thread-{}:{}'.format(threading.get_ident(), i))
        time.sleep(1)

# 创建一个最大容纳数量为5的线程池
pool = ThreadPoolExecutor(5)
for i in range(10):
    # 往线程池上塞任务
    pool.submit(target)

####创建线程池还可以使用更优雅的方式，就是使用上下文管理器####
with ThreadPoolExecutor(5) as pool:
    for i in range(100):
        pool.submit(target)
```

1. 使用 with 语句 ，通过 ThreadPoolExecutor 构造实例，同时传入 max_workers 参数来设置线程池中最多能同时运行的线程数目。
2. 使用 submit 函数来提交线程需要执行的任务到线程池中，并返回该任务的句柄（类似于文件、画图），注意 submit() 不是阻塞的，而是立即返回。
3. 通过使用 done() 方法判断该任务是否结束。上面的例子可以看出，提交任务后立即判断任务状态，显示四个任务都未完成。在延时2.5后，task1 和 task2 执行完毕，task3 仍在执行中
4. 使用 result() 方法可以获取任务的返回值。

### 自定义线程池

使用消息队列来自定义线程池

```
import time
import threading
from queue import Queue

def target(queue):
    while True:
        task = queue.get()
        if task == "stop":
            queue.task_done()
            break

        task()
        queue.task_done()

def do_task():
    for i in range(5):
        print('running thread-{}:{}'.format(threading.get_ident(), i))
        time.sleep(1)

class MyQueue(Queue):
    def close(self):
        for i in range(self.maxsize):
            self.put("stop")

def custome_pool(task_func, max_workers):
    queue = MyQueue(max_workers)
    for n in range(max_workers):
        t = threading.Thread(target=task_func, args=(queue,))
        t.daemon = True
        t.start()
    return queue

pool = custome_pool(task_func=target, max_workers=5)

for i in range(10):
    pool.put(do_task)

pool.close()
pool.join()
```
