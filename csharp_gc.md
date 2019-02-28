# C#和GC

## 基本原理
从root开始，通过引用关系遍历堆上的所有对象，找不到的就回收了。

##常用算法
Reference Counting, Mark Sweep, Copy Collection

## Mark Sweep算法
挂起线程，确定root，创建可达图，标记不可回收的，回收，移动整理（压缩，指针修复）
root：全局对象，静态变量，局部对象，函数调用参数，当前寄存器中的对象指针， finalization queue.

指针修复时需要修复的指针：stack，CPU寄存器中的指针，堆中的引用指针。

可达：
debug模式，局部对象在离开函数调用之后才会被当做不可达。
传给com+的对象，还要维持一个引用计数，以兼容那边的机制，引用计数为0时才不可达。

Pinned objects
是指不能移动位置的对象，包括fixed修饰的，以及传给非托管代码的对象。

## 分代算法
假设：大量新创建的对象生命周期较短，部分内存回收比全部内存回收要快，新创建对象关联性较强，有更好的cache命中率；
Gen0，Gen1，Gen2，0代GC后幸存的进入1代，1代GC后幸存的进入2代，2代gc成为full gc
0代和1代总共大概16M，GC在几毫秒至几十毫秒之间；
2代可以很大，看应用程序，GC可能需要花费几秒
三代的GC频率大致为1：10:100

## Finalization Queue和Freachable Queue
含有Finalize方法的对象在new时，会在Finalization Queue里添加一个对它的指针
GC时，如果对象被回收，指针会从Finalization Queue 放到Freachable Queue里，称为复生；
放到Freachable Queue在之后的时间调用Finalize并被剔除。
ReRegisterForFinalize将对象的指针加入Finalization Queue，SuppressFinalize将对象从Finalization Queue和Freachable Queue里剔除。