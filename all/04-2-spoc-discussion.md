#lec9 虚存置换算法spoc练习

## 个人思考题
1. 置换算法的功能？

2. 全局和局部置换算法的不同？

3. 最优算法、先进先出算法和LRU算法的思路？

4. 时钟置换算法的思路？

5. LFU算法的思路？

6. 什么是Belady现象？

7. 几种局部置换算法的相关性：什么地方是相似的？什么地方是不同的？为什么有这种相似或不同？

8. 什么是工作集？

9. 什么是常驻集？

10. 工作集算法的思路？

11. 缺页率算法的思路？

12. 什么是虚拟内存管理的抖动现象？
		
13. 操作系统负载控制的最佳状态是什么状态？

## 小组思考题目

----
(1)（spoc）请证明为何LRU算法不会出现belady现象

```
	只需证明S(t)包含于S'(t)，采用递归法：
	S(1) < S'(1)
	假设S(k-1) < S'(k-1)
	则对于b(k)
	当b(k)属于S(k-1)时，S(k) < S'(k)
	当b(k)不属于S(k-1)但属于S'(k-1)时，S(k) = S(k-1)-v+b < S'(k)
	当b(k)都不属于S(k-1)和S'(k-1)时，设S(k-1) = n1, n2 .... np, S'(k-1) = n1, n2, ... np, np+1,....nq
	S(k-1)中剔除nv,当S'(k-1)的np+1, ... nq若小于nv，则仍有S(k) < S'(k)
					当S'(k-1)的np+1,...nq若都大于nv，则都剔除nv，仍有S(k) < S'(k)
	综上都有S(k) < S'(k),所以LRU不会出现belady现象
```
(2)（spoc）根据你的`学号 mod 4`的结果值，确定选择四种替换算法（0：LRU置换算法，1:改进的clock 页置换算法，2：工作集页置换算法，3：缺页率置换算法）中的一种来设计一个应用程序（可基于python, ruby, C, C++，LISP等）模拟实现，并给出测试。请参考附件代码或独自实现。

## 扩展思考题
（1）了解LIRS页置换算法的设计思路，尝试用高级语言实现其基本思路。此算法是江松博士（导师：张晓东博士）设计完成的，非常不错！

参考信息：

 - [LIRS conf paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang02_LIRS.pdf)
 - [LIRS journal paper](http://www.ece.eng.wayne.edu/~sjiang/pubs/papers/jiang05_LIRS.pdf)
 - [LIRS-replacement ppt1](http://dragonstar.ict.ac.cn/course_09/XD_Zhang/(6)-LIRS-replacement.pdf)
 - [LIRS-replacement ppt2](http://www.ece.eng.wayne.edu/~sjiang/Projects/LIRS/sig02.ppt)
