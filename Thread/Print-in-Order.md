## [1114.Print in Order](https://leetcode-cn.com/problems/print-in-order/)

Suppose we have a class:

```
public class Foo {
  public void first() { print("first"); }
  public void second() { print("second"); }
  public void third() { print("third"); }
}
```


The same instance of Foo will be passed to three different threads. Thread A will call first(), thread B will call second(), and thread C will call third(). Design a mechanism and modify the program to ensure that second() is executed after first(), and third() is executed after second(). 

#### Example 1:

Input: [1,2,3]
Output: "firstsecondthird"
Explanation: There are three threads being fired asynchronously. The input [1,2,3] means thread A calls first(), thread B calls second(), and thread C calls third(). "firstsecondthird" is the correct output.
Example 2:

Input: [1,3,2]
Output: "firstsecondthird"
Explanation: The input [1,3,2] means thread A calls first(), thread B calls third(), and thread C calls second(). "firstsecondthird" is the correct output.

#### Note:

We do not know how the threads will be scheduled in the operating system, even though the numbers in the input seems to imply the ordering. The input format you see is mainly to ensure our tests' comprehensiveness.

#### Solutions：

```Python
from threading import Lock
class Foo(object):
    def __init__(self):
        self.lock1 = Lock()
        self.lock2 = Lock()
        self.lock1.acquire()
        self.lock2.acquire()
    def first(self, printFirst):
        """
        :type printFirst: method
        :rtype: void
        """
        
        # printFirst() outputs "first". Do not change or remove this line.
        
        printFirst()
        self.lock1.release()

    def second(self, printSecond):
        """
        :type printSecond: method
        :rtype: void
        """
        
        # printSecond() outputs "second". Do not change or remove this line.
        self.lock1.acquire() 
        printSecond()
        self.lock2.release()
        # or 
        '''
        with lock1:
            printSecond()
       		self.lock2.release()
        '''
            
    def third(self, printThird):
        """
        :type printThird: method
        :rtype: void
        """
        
        # printThird() outputs "third". Do not change or remove this line.
        self.lock2.acquire()
        printThird()
```

#### refer1：

 https://www.liaoxuefeng.com/wiki/1016959663602400/1017629247922688 

 多线程和多进程最大的不同在于，多进程中，同一个变量，各自有一份拷贝存在于每个进程中，互不影响，而多线程中，所有变量都由所有线程共享，所以，任何一个变量都可以被任何一个线程修改，因此，线程之间共享数据最大的危险在于多个线程同时改一个变量，把内容给改乱了。 

```Python
import time, threading
 
# 假定这是你的银行存款:
balance = 0
 
def change_it(n):
    # 先存后取，结果应该为0:
    global balance
    balance = balance + n
    balance = balance - n
 
def run_thread(n):
    for i in range(100000):
        change_it(n)
 
t1 = threading.Thread(target=run_thread, args=(5,))
t2 = threading.Thread(target=run_thread, args=(8,))
t1.start()
t2.start()
t1.join()
t2.join()
print(balance)
```

 我们定义了一个共享变量`balance`，初始值为`0`，并且启动两个线程，先存后取，理论上结果应该为`0`，但是，由于线程的调度是由操作系统决定的，当t1、t2交替执行时，只要循环次数足够多，`balance`的结果就不一定是`0`了。 

 原因是因为高级语言的一条语句在CPU执行时是若干条语句，即使一个简单的计算： balance = balance + n

也分两步：

1. 计算`balance + n`，存入临时变量中；
2. 将临时变量的值赋给`balance`。

 也就是可以看成： 

x = balance + n
balance = x

 由于x是局部变量，两个线程各自都有自己的x，当代码正常执行时： 

```Python
初始值 balance = 0
 
t1: x1 = balance + 5 # x1 = 0 + 5 = 5
t1: balance = x1     # balance = 5
t1: x1 = balance - 5 # x1 = 5 - 5 = 0
t1: balance = x1     # balance = 0
 
t2: x2 = balance + 8 # x2 = 0 + 8 = 8
t2: balance = x2     # balance = 8
t2: x2 = balance - 8 # x2 = 8 - 8 = 0
t2: balance = x2     # balance = 0
 
结果 balance = 0
```

但是t1和t2是交替运行的，如果操作系统以下面的顺序执行t1、t2：

```Python
初始值 balance = 0
 
t1: x1 = balance + 5  # x1 = 0 + 5 = 5
 
t2: x2 = balance + 8  # x2 = 0 + 8 = 8
t2: balance = x2      # balance = 8
 
t1: balance = x1      # balance = 5
t1: x1 = balance - 5  # x1 = 5 - 5 = 0
t1: balance = x1      # balance = 0
 
t2: x2 = balance - 8  # x2 = 0 - 8 = -8
t2: balance = x2   # balance = -8
 
结果 balance = -8
```

究其原因，是因为修改`balance`需要多条语句，而执行这几条语句时，线程可能中断，从而导致多个线程把同一个对象的内容改乱了。

两个线程同时一存一取，就可能导致余额不对，你肯定不希望你的银行存款莫名其妙地变成了负数，所以，我们必须确保一个线程在修改`balance`的时候，别的线程一定不能改。

如果我们要确保`balance`计算正确，就要给`change_it()`上一把锁，当某个线程开始执行`change_it()`时，我们说，该线程因为获得了锁，因此其他线程不能同时执行`change_it()`，只能等待，直到锁被释放后，获得该锁以后才能改。由于锁只有一个，无论多少线程，同一时刻最多只有一个线程持有该锁，所以，不会造成修改的冲突。创建一个锁就是通过`threading.Lock()`来实现：

```Python
balance = 0
lock = threading.Lock()
 
def run_thread(n):
    for i in range(100000):
        # 先要获取锁:
        lock.acquire()
        try:
            # 放心地改吧:
            change_it(n)
        finally:
            # 改完了一定要释放锁:
            lock.release()
```

当多个线程同时执行`lock.acquire()`时，只有一个线程能成功地获取锁，然后继续执行代码，其他线程就继续等待直到获得锁为止。

获得锁的线程用完后一定要释放锁，否则那些苦苦等待锁的线程将永远等待下去，成为死线程。所以我们用`try...finally`来确保锁一定会被释放。

锁的好处就是确保了某段关键代码只能由一个线程从头到尾完整地执行，坏处当然也很多，首先是阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了。其次，由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁，导致多个线程全部挂起，既不能执行，也无法结束，只能靠操作系统强制终止。

#### refer2:

```Python
# encoding=utf8
 
import threading
import time
 
lock = threading.Lock()
l = []
 
def test1(n):
	lock.acquire()
	l.append(n)
	print l
	lock.release()
 
def test(n):
	l.append(n)
	print l
 
def main():
	for i in xrange(0, 10):
		th = threading.Thread(target=test, args=(i, ))
		th.start()
if __name__ == '__main__':
	main()
```

test

```
[0]
[0, 1]
[0, 1, 2]
[0, 1, 2, 3][
0, 1, 2, 3, 4]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4[, 05, , 16, , 27, ]3
, 4, 5, 6[, 07, , 18, ]2
, 3, 4, [50, , 61, , 72, , 83, , 94], 
5, 6, 7, 8, 9]
```

test1

```
[0]
[0, 1]
[0, 1, 2]
[0, 1, 2, 3]
[0, 1, 2, 3, 4]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6]
[0, 1, 2, 3, 4, 5, 6, 7]
[0, 1, 2, 3, 4, 5, 6, 7, 8]
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

