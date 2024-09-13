（文章来自2022.01.26的博客）  

&emsp;&emsp;在Python里面，由于有全局解释器锁的存在，在同一时刻只能有一个线程在执行，因此如果是计算密集型的任务，要想实现并行，需要采用多进程编程。
# 1、基本概念
## 1.1 进程的概念
&emsp;&emsp;进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。
## 1.2 进程的状态
&emsp;&emsp;进程执行时的间断性，决定了进程可能具有多种状态。事实上，运行中的进程可能具有以下三种基本状态。
&emsp;&emsp; 1）就绪状态（Ready）：
&emsp;&emsp;进程已获得除处理器外的所需资源，等待分配处理器资源；只要分配了处理器进程就可执行。就绪进程可以按多个优先级来划分队列。例如，当一个进程由于时间片用完而进入就绪状态时，排入低优先级队列；当进程由I/O操作完成而进入就绪状态时，排入高优先级队列。
&emsp;&emsp; 2）运行状态(Running)：
&emsp;&emsp;进程占用处理器资源；处于此状态的进程的数目小于等于处理器的数目。在没有其他进程可以执行时(如所有进程都在阻塞状态)，通常会自动执行系统的空闲进程。
&emsp;&emsp; 3）阻塞状态(Blocked)：
&emsp;&emsp;由于进程等待某种条件（如I/O操作或进程同步），在条件满足之前无法继续执行。该事件发生前即使把处理器资源分配给该进程，也无法运行。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/43f64add5f4af081f35f13c63baa33ae.png)
## 1.3 什么情况下使用多进程
&emsp;&emsp;当一个计算任务可以对输入数据分别进行处理的时候，可以使用多进程编程提高任务执行的效率。开启的进程数可以是小于等于cpu核心数的一个值。

# 2、Python实现多进程
## 2.1 简单示例
&emsp;&emsp;在Python里面，可以通过 multiprocessing 模块在程序中创建、管理进程。
&emsp;&emsp;第一种方式可以使用 Process 类，下面代码创建了3个进程对输入数据进行处理。首先定义一个对数据进行处理的函数，在下面代码中这个函数是 t1 ，然后把要处理的数据序列中的数值分别作为3个进程调用函数 t1 的参数：

```python
import os
import time
import random
from multiprocessing import Process
from multiprocessing import Pool


def t1(tmp):
    print(tmp, 'starting；', '线程ID是', os.getpid())           # 打印当前开启的进程的 ID
    time.sleep(random.randrange(1,5))                          # 让进程持续随机的几秒
    print(tmp, 'end')                                          # 进程结束

if __name__ == '__main__':
    for i in range(3):
        p = Process(target=t1, args=(i,))
        p.start()                                              # 启动进程

    print('主线程', '线程ID是', os.getpid())                      # 打印主进程的ID

```

&emsp;&emsp;输出的结果如下：

```python
# 在这里开启子进程之后没有用join方法启动守护进程，所以主进程先执行完
主线程 线程ID是 19665	
0 starting； 线程ID是 19667
1 starting； 线程ID是 19668
2 starting； 线程ID是 19669
1 end
0 end
2 end

```

&emsp;&emsp;第二种创建和使用进程的方式是用 Pool 类，创建一个进程池管理多个进程。进程池可以提供指定数量的进程给用户使用，即当有新的请求提交到进程池中时，如果池未满，则会创建一个新的进程用来执行该请求；反之，如果池中的进程数已经达到规定最大值，那么该请求就会等待，只要池中有进程空闲下来，该请求就能得到执行。

```python
import os
import time
import random
from multiprocessing import Pool

def t1(tmp):
    print(tmp, 'starting；', '线程ID是', os.getpid())       # 打印当前开启的进程的ID
    time.sleep(random.randrange(1,3))                     # 让进程持续随机的几秒
    tmp = tmp * 10
    print(tmp, 'end')
    return tmp

if __name__ == '__main__':
    p = Pool(3)                                           # 进程池中从无到有创建三个进程,以后一直是这三个进程在执行任务
    res = []
    for i in range(6):
        r = p.apply_async(t1, args=(i,))                 # 这是 apply() 方法的异步版本，该方法不会被阻塞。
        res.append(r.get())								 # 使用get来获取apply_aync的结果

    # 主进程需要使用jion，否则，主进程结束，进程池可能还没来得及执行，也就跟着一起结束了
    p.close()											# 关闭进程池（可以使用 with 语句来管理进程池，这意味着我们无需手动调用 close() 方法关闭进程池）
    p.join()											# 守护进程
    print('################################')

    for j in res:
        print(j)
```

&emsp;&emsp;代码执行结果如下：

```python
0 starting； 线程ID是 19843
0 end
1 starting； 线程ID是 19844
10 end
2 starting； 线程ID是 19845
20 end
3 starting； 线程ID是 19843
30 end
4 starting； 线程ID是 19844
40 end
5 starting； 线程ID是 19845
50 end
################################
0
10
20
30
40
50
```

## 2.2 生产者、消费者模型
&emsp;&emsp;如果可以把一个数据处理任务分成不同的阶段，比如划分成2个阶段，这两个阶段的执行速度存在显著差异，那么可以采用生产者、消费者模型，生产者是数据处理的第一阶段，消费者是数据处理的第二阶段。
&emsp;&emsp;生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。
&emsp;&emsp;一个实例如下：

```python
from multiprocessing import Pool, Manager
import time, random, os

# 生产者，处理数据的第一阶段
def producer(q, num):
    q.put(num)
    time.sleep(random.randrange(1, 3))		#生产者执行的快一点
    print(os.getpid(), '放入队列', num)

# 消费者，处理数据的第二阶段
def consumer(num):
    res = num * 10
    time.sleep(random.randrange(3,7))		# 消费者执行的慢一点
    print(os.getpid(), '取出处理了', num, '处理后是', res)

if __name__ == '__main__':
    # multiprocess.Queue是跨进程通信队列。但是不能用于multiprocessing.Pool多进程的通信。
    # 进程池multiprocessing.Pool()的多进程之间的通信要用multiprocessing.Manager().Queue()
    q = Manager().Queue()

    p1 = Pool(2)    # 生产者们
    p2 = Pool(2)    # 消费者们

    ori = [1, 2, 3, 4, 5, 6]
    for i in ori:
        p1.apply_async(producer, args=(q, i, ))
    p1.close()
    p1.join()

    while True:
        if not q.empty():			# 队列为空值一般不是真实状态，在这里由于消费者执行的慢，所以可以作为结束状态的判断
            tmp = q.get()
            p2.apply_async(consumer, args=(tmp,))
        else:
            break

    p2.close()
    p2.join()

    print('主')
```

&emsp;&emsp;代码输出如下：

```python
22219 放入队列 1
22220 放入队列 2
22219 放入队列 3
22220 放入队列 4
22219 放入队列 5
22220 放入队列 6
22221 取出处理了 1 处理后是 10
22222 取出处理了 2 处理后是 20
22221 取出处理了 3 处理后是 30
22222 取出处理了 4 处理后是 40
22221 取出处理了 5 处理后是 50
22222 取出处理了 6 处理后是 60
主
```
## 2.3 回调函数
&emsp;&emsp;和生产者消费者模型类似，假如消费者的任务执行很快，那么可以使用回调函数，让主进程作为消费者执行回调函数：

```python
import os
import time
import random
from multiprocessing import Pool

def t2(tmp):
    print(tmp, 'starting；', '线程ID是', os.getpid())            # 打印当前开启的进程的ID
    time.sleep(random.randrange(1,3))                          # 让进程持续随机的几秒
    print(tmp, 'end')
    return tmp

def t3(tmp):
    tmp = tmp * 10
    print(tmp, 'starting；', '线程ID是', os.getpid())            # 打印当前开启的进程的ID
    print(tmp, 'end')

if __name__ == '__main__':
    l = [1,2,3,4,5,6]

    p = Pool(3)
    res = []
    for i in l:
        r = p.apply_async(t2,args=(i,),callback=t3)				# 回调函数是 t3 ，t3 的输入是函数 t2 的输出，t3 由主进程执行
        res.append(r.get())

    p.close()
    p.join()
    print(os.getpid())                                          # 打印主进程id
```

&emsp;&emsp; 执行结果如下：
```python
1 starting； 线程ID是 22366
1 end
10 starting； 线程ID是 22364
10 end
2 starting； 线程ID是 22367
2 end
20 starting； 线程ID是 22364
20 end
3 starting； 线程ID是 22368
3 end
30 starting； 线程ID是 22364
30 end
4 starting； 线程ID是 22366
4 end
40 starting； 线程ID是 22364
40 end
5 starting； 线程ID是 22367
5 end
50 starting； 线程ID是 22364
50 end
6 starting； 线程ID是 22368
6 end
60 starting； 线程ID是 22364
60 end
22364	# 主进程的ID
```

# 3 总结
&emsp;&emsp;在使用Python处理计算密集型任务的时候，多进程可以作为单机并行的最后手段。多进程编程的核心有两点：1 是把计算任务整合为一个函数，2 是函数参数是序列化的。这样就可以把任务拆解，并行执行。

&emsp;&emsp;
参考链接：
https://baike.baidu.com/item/%E8%BF%9B%E7%A8%8B/382503
https://www.cnblogs.com/linhaifeng/articles/7428874.html
https://blog.csdn.net/yuanlulu/article/details/83116565
