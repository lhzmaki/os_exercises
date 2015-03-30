#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？
```
快速的实现外存中的页与内存中的页的替换
```
2. 全局和局部置换算法的不同？
```    
局部置换算法：针对同一个进程，每个进程的页数相同
全局置换算法：可以置换不同进程中的页
```
3. 最优算法、先进先出算法和LRU算法的思路？
```
最优算法：将之后最后出现的页置换
先进先出：将之前最先出现的页置换
LRU：将之前出现的页的最近出现位置进行排序，将最小的那个置换
```
4. 时钟置换算法的思路？
```
 增加一个标记位，标记有没出现，将指针依次下移，遇到标记为1的，则将其置0，标记为0的则替换
```
5. LFU算法的思路？
```
将内存中的页计数，置换时替换计数最小的页
```
6. 什么是Belady现象？
```
增加内存中的页数，但缺页率反而增加的现象
```
7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？
```
页面不断的换入换出导致花费很多的时间
```
13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象
```
	证明：
		原理:因为小的物理页帧的栈包含于大数目的物理页帧的栈
		证明根据课堂老师给出的基础：
			来证明s(t) 始终包含于 s'(t)
			利用归纳法，假设 1<=i<=t-1 时 s(i)包含于s'(i)，现在要证s(t)依然包含于s'(t)
			(1) b(t)同时属于s(t)和s'(t)：此时s(t)和s'(t)都不发生变化，满足包含关系；
			(2) b(t)不属于s(t),属于s'(t)：s(t) 替换后，由于b(t)∈s(t)，所以s(t)包含于s'(t)
			(3)  (1)和(2)很容易证明，
				对于b(t)同时不属于s(t-1)和s'(t-1)的情况，我们依然按照视频里栈的方式对s(t-1)和s'(t-1)排序
				由于s(t-1)包含于s'(t-1),所以s(t-1)内每一个元素都存在于s'(t-1)中。
				现在两个栈都是按最后一次访问的时间的顺序来排列的，由于s(t)在进行替换时会替换s(t-1)里面最长时间没被访问的元素(栈底)，设为a,那么a显然也存在于s'(t-1)里面，并且它不一定是s'(t-1)的栈底。
					A. 当a是s'(t-1)的栈底时，s(t)和s'(t)替换的都是a, s(t) = s(t-1) - {a} + {b(t)} , s'(t) = s'(t-1) - {a} + {b(t)}
					B. 当a不是s'(t-1)的栈底是，则s'(t-1)的栈底c必然不属于s(t-1)，否则就会与a是s(t-1)的栈底矛盾（即c比a有更长的时间未被访问），此种情况下s(t)和s'(t)依然满足包含关系
			(4) 由归纳假设可以得知此种情况不存在
```

(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考附件代码或独自实现。
```
#include <iostream>
using namespace std;

const int SIZE = 5;
int PT = 0;

struct PAGE
{
	bool visit;
	bool write;
	int num;
	/* data */
};


int main()
{
	PAGE memory[SIZE];
	
	for (int i = 0; i < SIZE; i++){
		memory[i].visit = false;
		memory[i].write = false;
		memory[i].num = 0;
	}

	int vm[10];
	bool wm[10];
	for (int i = 0; i < 10; i++){
		cin >> vm[i] >> wm[i];
		bool ans = false;
		for (int j = 0; j < SIZE; j++){
			if (memory[j].num == vm[i]){
				ans = true;
				memory[j].write = (wm[i] || memory[j].write);
				break;
			}
		}
		
		while(!ans){
			cout << "hello" << endl;
			if (memory[PT].visit == false && memory[PT].write == false)
			{
				memory[PT].visit = true;
				memory[PT].write = wm[i];
				memory[PT].num = vm[i];
				ans = true;
			}
			else if (memory[PT].visit == false && memory[PT].write == true){
				memory[PT].write = false;
				PT++;
				if (PT == SIZE)
					PT = 0;
			}
			else if (memory[PT].visit == true && memory[PT].write == false){
				memory[PT].visit = false;
				PT++;
				if (PT == SIZE)
					PT = 0;
			}
			else if (memory[PT].visit == true && memory[PT].write == true){
				memory[PT].visit = false;
				PT++;
				if (PT == SIZE)
					PT = 0;
			}
		}

		for (int j = 0; j < SIZE; j++)
			cout << memory[j].visit << memory[j].write << memory[j].num << endl;
	}

	return 0;
}

```
## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
