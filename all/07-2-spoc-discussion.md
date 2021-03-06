# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 基本理解
 - 什么是信号量？它与软件同步方法的区别在什么地方？
 - 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
 - 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。
> 它的一种出错的情况
```
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 信号量使用

 - 什么是条件同步？如何使用信号量来实现条件同步？
 - 什么是生产者-消费者问题？
 - 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？

### 管程

 - 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
 - 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
 - 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 哲学家就餐问题

 - 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 读者-写者问题

 - 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
 - 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？
 
## 小组思考题

1. （spoc） 每人用python threading机制用信号量和条件变量两种手段分别实现[47个同步问题](07-2-spoc-pv-problems.md)中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解[]python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)
 
 >第12题，信号量代码如下：
 
```
import threading
import time
import os
import random

people = [0, 0, 0]
movie = -1
count = 100
mutex = threading.Lock()

semaphore = threading.Semaphore()

class Movie1(threading.Thread):
	def __init__(self):
		threading.Thread.__init__(self)

	def run(self):
		global semaphore, movie, count, people
		while True:
			if semaphore.acquire():
				if (count == 0):
					print "the time is over\n"
					break
				print "now is the movie 1\nthere are %d people is watch\n" %(people[0])
				movie = 1
				while (people[0] != 0):
					continue
				movie = -1
				print "the movie 1 is over\n"
				semaphore.release()
				time.sleep(2)

class Movie2(threading.Thread):
	def __init__(self):
		threading.Thread.__init__(self)

	def run(self):
		global semaphore, movie, count, people
		while True:
			if semaphore.acquire():
				if (count == 0):
					print "the time is over\n"
					break
				print "now is the movie 2\nthere are %d people is watch\n" %(people[1])
				movie = 2
				while (people[1] != 0):
					continue
				movie = -1
				print "the movie 2 is over\n"
				semaphore.release()
				time.sleep(2)

class Movie3(threading.Thread):
	def __init__(self):
		threading.Thread.__init__(self)

	def run(self):
		global semaphore, movie, count, people
		while True:
			if semaphore.acquire():
				if (count == 0):
					print "the time is over\n"
					break
				print "now is the movie 3\nthere are %d people is watch\n" %(people[2])
				movie = 3
				while (people[2] != 0):
					continue
				movie = -1
				print "the movie 3 is over\n"
				semaphore.release()
				time.sleep(2)

class People(threading.Thread):

	num = 0

	def __init__(self):
		threading.Thread.__init__(self)

	def run(self):
		global count, people, movie
		num = random.randint(0, 2)
		if mutex.acquire():
			people[num] += 1
			mutex.release()
		while(movie != (num+1)):
			continue
		print "i will leave\n"
		if mutex.acquire():
			people[num] -= 1
			count -= 1
			mutex.release()

if __name__ == "__main__":		

	p1 = Movie1()
	p1.start()
	p2 = Movie2()
	p2.start()
	p3 = Movie3()
	p3.start()

	for i in range(0, 100):
		c = People()
		c.start()
		time.sleep(0.1)

```