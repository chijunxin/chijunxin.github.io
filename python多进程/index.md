# python多进程



<!--more-->


## multiprocessing 模块
***multiprocessing*** 模块是`Python`标准库中的一部分，提供了创建和管理进程的功能。

* Process类：用于创建和控制进程。
* Queue类：用于进程间通信的队列。
* Pipe类：用于进程间通信的管道。
* Lock类：用于进程同步的锁机制。
* Pool类：用于管理进程池。

### Process类 : 创建和启动进程
**使用Process类创建进程**
可以通过继承`Process`类或者直接实例化`Process`对象来创建进程。

{{< admonition example "继承Process类" >}}
``` python
import multiprocessing

class MyProcess(multiprocessing.Process):
    def run(self):
        print("Hello from a process!")

# 创建并启动进程
process = MyProcess()
process.start()
process.join()
```
{{< /admonition >}}

{{< admonition example "直接实例化Process对象" >}}
``` python
import multiprocessing

def process_function():
    print("Hello from a process!")

# 创建并启动进程
process = multiprocessing.Process(target=process_function)
process.start()
process.join()
```

**传递参数给进程**
可以通过`args`参数向进程函数传递参数。

``` python
import multiprocessing

def process_function(name):
    print(f"Hello from {name}!")

# 创建并启动进程
process = multiprocessing.Process(target=process_function, args=("Process-1",))
process.start()
process.join()
```
在这个示例中，我们向进程函数传递了一个参数`name`。
{{< /admonition >}}

### Queue类和Pipe类 : 进程间通信
在多进程编程中，进程间通信（IPC）是非常重要的。`multiprocessing`模块提供了多种方式来实现进程间通信，包括队列（`Queue`）和管道（`Pipe`）。

{{< admonition example "使用Queue进行进程间通信" >}}
`Queue`类提供了进程安全的队列，用于在进程之间传递数据。
``` python
import multiprocessing

def producer(queue):
    for i in range(5):
        queue.put(i)
        print(f"Produced {i}")

def consumer(queue):
    while True:
        item = queue.get()
        if item is None:
            break
        print(f"Consumed {item}")

# 创建队列
queue = multiprocessing.Queue()

# 创建生产者和消费者进程
producer_process = multiprocessing.Process(target=producer, args=(queue,))
consumer_process = multiprocessing.Process(target=consumer, args=(queue,))

# 启动进程
producer_process.start()
consumer_process.start()

# 等待生产者进程完成
producer_process.join()

# 向队列中添加结束信号
queue.put(None)

# 等待消费者进程完成
consumer_process.join()
```
在这个示例中，使用`Queue`类实现了生产者和消费者模式的进程间通信。
{{< /admonition >}}

{{< admonition example "使用Pipe进行进程间通信" >}}
`Pipe`类提供了双向通信的管道，用于在两个进程之间传递数据。
``` python
import multiprocessing

def sender(pipe):
    for i in range(5):
        pipe.send(i)
        print(f"Sent {i}")

def receiver(pipe):
    while True:
        item = pipe.recv()
        if item is None:
            break
        print(f"Received {item}")

# 创建管道
parent_conn, child_conn = multiprocessing.Pipe()

# 创建发送者和接收者进程
sender_process = multiprocessing.Process(target=sender, args=(parent_conn,))
receiver_process = multiprocessing.Process(target=receiver, args=(child_conn,))

# 启动进程
sender_process.start()
receiver_process.start()

# 等待发送者进程完成
sender_process.join()

# 向管道中添加结束信号
parent_conn.send(None)

# 等待接收者进程完成
receiver_process.join()
```
在这个示例中，使用`Pipe`类实现了简单的进程间通信。
{{< /admonition >}}

### Lock类 : 进程同步
在多进程编程中，多个进程可能会访问共享资源，导致数据竞争。为了防止这种情况，可以使用同步机制，如锁（`Lock`）。

{{< admonition example "使用Lock进行同步" >}}
``` python
import multiprocessing

# 创建一个锁对象
lock = multiprocessing.Lock()

# 创建共享内存
m = multiprocessing.Value('i', 0)
n = multiprocessing.Value('i', 0)

def process_function(m, n):
    for _ in range(100000):
        m.value += 1
    with lock:
        for _ in range(100000):
            n.value += 1

# 创建并启动多个进程
processes = []
for _ in range(10):
    process = multiprocessing.Process(target=process_function, args=(m, n,))
    process.start()
    processes.append(process)

# 等待所有进程完成
for process in processes:
    process.join()

print(f"Final m.value:{m.value} n.value:{n.value}")
```
在这个示例中，使用锁（`Lock`）对象来保护对共享变量`counter`的访问，确保进程安全。
{{< /admonition >}}

### Pool类 : 进程池
进程池是一种管理和重用进程的机制，可以提高多进程编程的效率。在Python中，可以使用`Pool`类来实现进程池。
{{< admonition example "使用进程池" >}}
``` python
import multiprocessing

def worker(num):
    """该函数将在子进程中执行"""
    print('Worker %d' % num)

# 创建进程池
pool = multiprocessing.Pool(4)
# 启动进程池中的进程
pool.map(worker, range(10))
# 关闭进程池
pool.close()
# 等待进程池中的进程结束
pool.join()
```
在这个示例中，创建了一个进程池，并向进程池提交了多个任务。
{{< /admonition >}}

{{< admonition example "多进程计算密集型任务" >}}
``` python
import multiprocessing
import math

def factorial(n):
    return math.factorial(n)

numbers = [100000, 200000, 300000, 400000, 500000]

# 创建进程池
with multiprocessing.Pool(processes=5) as pool:
    results = pool.map(factorial, numbers)

for number, result in zip(numbers, results):
    print(f"Factorial of {number} is calculated.")
```
在这个示例中，使用多进程计算了一组大数字的阶乘，并打印了计算结果。
{{< /admonition >}}

### 处理进程异常
在多进程编程中，处理进程异常也是非常重要的。可以通过捕获进程函数中的异常并将其记录或处理。

{{< admonition example "异常处理" >}}
``` python
import multiprocessing

def process_function():
    try:
        raise ValueError("An error occurred!")
    except Exception as e:
        print(f"Exception in process: {e}")

# 创建并启动进程
process = multiprocessing.Process(target=process_function)
process.start()
process.join()
```
在这个示例中，在进程函数中捕获了异常并进行了处理。
{{< /admonition >}}

### multiprocessing的一些注意事项

#### 全局变量的共享问题
每个子进程都有自己的内存空间，因此全局变量在多进程之间不能直接共享。如果需要共享数据，可以使用 multiprocessing.Value 或 multiprocessing.Array 来创建共享内存。

#### 进程间通信问题
多个进程之间需要相互通信，可以使用 multiprocessing.Queue 或 multiprocessing.Pipe 来进行进程间通信。

#### 进程池的使用
如果需要同时启动多个进程，可以使用进程池来管理进程。进程池可以避免频繁地启动和关闭进程所带来的开销，提高程序的效率。

#### 子进程的异常处理
每个子进程都是一个独立的进程，当子进程出现异常时，主进程并不会收到通知。因此需要在子进程中进行异常处理，并将异常信息通过进程间通信的方式传递给主进程。

#### 进程的启动方式
可以使用 multiprocessing.Process 来创建进程，也可以使用 multiprocessing.Pool 来创建进程池。进程池可以方便地管理多个进程，避免手动启动和关闭进程所带来的麻烦。
