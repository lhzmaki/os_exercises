# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

> 
优点：
	最先匹配：实现简单，高地址空间有大块空分区，释放分区很快
	最差匹配：适合中等大小分配较多的情况，不会出现太多的小碎片
	最佳匹配：可避免过多的空闲大分区被拆分，外部碎片很小
	buddy system：不会产生太多的小碎片，对大、中、小空间分配都很均衡，效率也较高（O(logN)），空间释放也比较快（O(logN)）

>
缺点：
	最先匹配：会产生大量外部碎片，分配大块空间时效率低（分区的list太长）
	最差匹配：释放分区较慢，容易破坏大的空闲分区，同样会有外部碎片
	最佳匹配：容易产生许多无用的小碎片，分配或释放后重新维护顺序需要一定的开销，并且释放分区的合并与最差匹配一样慢
	buddy system：有时候会产生较大的内部碎片，对于恰好比2的幂次方大一点点的空间需求，空间浪费比较严重 (比如65KB的需求会浪费63KB的内存空间)

>
	应用局部性原理，Buddy System的合并不会一路执行到底（只往上合并若干次），这一方法应该会提升分配速度，但对空间的利用率没有提升。

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)
>参考网上的buddy systemm，实现了一个简化版的
>
```
#include <iostream>
using namespace std;
const int c_size = 1024;
struct buddy
{
	int node;
	int size;
	bool done;	
};
buddy* tree;
int power_of_2(int size)
{
	int i = 1;
	while (i < size){
		i = i*2;
	}
	return i;
}
void init()
{
	tree = new int [2*c_size - 1];
	int s = c_size;
	for (int i = 0; i < 2*c_size; i++){
		tree[i].node = i;
		if (s % (i+1) == 0)
			s = s/(i+1);
		tree[i].size = s;
		tree[i].done = true;
	}
}
struct ID
{
	int size;
	int offset;
};
ID buddy_alloc(int lg)
{
	if (lg == 0)
		return;
	int size = power_of_2(lg);
	if (tree[0].size < size)
		return;
	int i = 0;
	while(1){
		if (i >= 1023)
			break;
		if (tree[2*i + 1].size >= size)
			i = 2*i + 1;
		else if (tree[2*(i+1)].size >= size)
			i = 2*(i+1);
		else
			break;
	}
	int offset = (tree[i].node+1)*tree[i].size - c_size;
	ID id;
	id.size = size;
	id.offset = offset;
	int dsize = tree[i].size;
	tree[i].size = 0;
	tree[i].done = false;
	for (int j = i/2; j >= 0; j = j/2){
		tree[j].size -= dsize;
		dsize *= 2;
		if (tree[j].size == 0)
			tree[j].done = false;
	}
	return id;
}	
void buddy_free(ID id)
{
	int index = (id.offset + c_size) / id.size - 1;
	tree[index].done = true;
	tree[index].size = id.size;
	while (index > 0)
	{
		int brother;
		if (index %2 == 0)
			brother = index - 1;
		else
			brother = index + 1;
		if (tree[brother].done == true){
			tree[index/2].size = tree[index].size * 2;
			tree[index/2].done = true;
			index /= 2;
		}
		else
			break;
	}
}
int main()
{
	init();
	return 0;
}
```
--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



