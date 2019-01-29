
c# job system是指 unity的job system的c#部分， unity的job system还有native部分。

为了避免上下文切换，job系统只对一个CPU的核建一个工作线程（还有些核留给了操作系统和其他让出的应用），也就是 线程数 <= CPU核数。

在c#中，job系统把job塞到队列里；在底层，工作线程取出job然后执行。可以认为队列是c# job system和native job system的分界线。

job系统保证job的依赖性，也就是说如果job a 依赖 job b，则b肯定先于a执行。

竞争情形（race conditions）是指输出由不可控的时间决定。（比如两个线程都要修改某个数据）（如果没有安全措施，很容易出bug）

job系统检测所有的竞争情形，保证bug不会发生，job系统有以下机制：
1. 通过复制主线程的数据让数据隔离。
2. job只能访问blittagble types，这样就不需要托管代码和本地代码传递时有转换。
3. job通过memcpy将数据传到本地内存中，然后托管方（C#）在job执行时可以访问。

NativeContainer是个C#对native内存的封装，它的实现是有个指针指向非托管的分配的内存。它允许job访问与主线程共享的数据。
NativeContainer的类型目前有：
1. unity本身带一个NativeArray和可以操作NativeArray的NativeSlice。
ECS包还有：
2. NativeList - a resizable NativeArray.
3. NativeHashMap - key and value pairs.
4. NativeMultiHashMap - multiple values per key.
5. NativeQueue - a first in, first out (FIFO) queue.

NativeContainer有内建的安全系统，只有在Editor和Play模式下，NativeContainer才有安全检查。
NativeContainer使用DisposeSentinel检查内存泄漏，使用AtomicSafetyHandle转移对自己的使用权。
安全系统允许多个jobs同时读取数据。
默认情况，一个job既可以读，又可以写数据（性能会降低）。一个job写访问时，其他线程不能写。
可以使用ReadOnly属性标记一个NativeContainer是只读的。
static数据是没有这个保护的，它会绕过所有的保护系统，崩unity。比如MonoBehaviour。

NativeContainer的分配器类型：
- Allocator.Temp，最快分配，生命期最多一帧，不能把这样的数据传给job，需要在方法返回前调用他的Dispose。
- Allocator.TempJob，慢一点，生命期最多4帧，线程安全。4帧后不Dispose会有warning。
- Allocator.Persistent，最慢的，其实就是对malloc的包装，生命期没限制。性能很重要时不要使用这个。

在unity里，任何继承IJob的struct被称为job。
job的成员变量要么是 blittable类型的，要么是NativeContainer类型的。
Job需要实现Eexcute方法。
在主线程里，向NativeContainer成员写数据，是唯一访问job数据的方式。


IJob.Schedule只能在主线程里调用，表示把job放入job系统的队列，一旦开始，不可打断。
```
// Create a native array of a single float to store the result. This example waits for the job to complete for illustration purposes
NativeArray<float> result = new NativeArray<float>(1, Allocator.TempJob);

// Set up the job data
MyJob jobData = new MyJob();
jobData.a = 10;
jobData.b = 10;
jobData.result = result;

// Schedule the job
JobHandle handle = jobData.Schedule();

// Wait for the job to complete
handle.Complete();

// All copies of the NativeArray point to the same memory, you can access the result in "your" copy of the NativeArray
float aPlusB = result[0];

// Free the memory allocated by the result array
result.Dispose();
```

可以使用JobHandle.CombineDependencies将多个依赖的句柄合为一个句柄。
JobHandle.Complete可以立刻执行这个job，之后主线程可以安全访问NativeContainer了。

如果说一个Job做一个任务，那么一个ParallelFor可以同时做多个任务。
ParallelFor有个NativeArray，将index传给Execute执行一个元素上的操作，从而实现元素之间的并行运算。
ParallelFor把自己的数据分批，然后交给内部的job，由再由job系统分配给每个工作线程。当job提前完成自己的计算后，可以steal其他job的批数据（多线程计算调度策略）。为了保证缓存局部性，它只偷取那个job的一半数据。
调用ParallelFor.Schedule时，你需要指定数组长度和分批的个数。指定数组长度是因为有可能有多个成员是数组，job系统不知道用哪一个决定长度。分批个数帮助你试验出最优的性能。


```
NativeArray<float> a = new NativeArray<float>(2, Allocator.TempJob);

NativeArray<float> b = new NativeArray<float>(2, Allocator.TempJob);

NativeArray<float> result = new NativeArray<float>(2, Allocator.TempJob);

a[0] = 1.1;
b[0] = 2.2;
a[1] = 3.3;
b[1] = 4.4;

MyParallelJob jobData = new MyParallelJob();
jobData.a = a;  
jobData.b = b;
jobData.result = result;

// Schedule the job with one Execute per index in the results array and only 1 item per processing batch
JobHandle handle = jobData.Schedule(result.Length, 1);

// Wait for the job to complete
handle.Complete();

// Free the memory allocated by the arrays
a.Dispose();
b.Dispose();
result.Dispose();
```
ParallelForTransform是针对Transforms操作的一种ParallelFor类型。

不要直接更新NativeContainer的内容，需要拷出来再赋值回去。

只能在主线程里调用Schedule和Complete。Complete立即提升本job的优先级，并在调用线程（只可能是主线程）里执行。可以清理安全系统的状态，可以让主线程获得数据的所有权。
当主线程准备好数据时就可以调用Schedule开始Job，当需要用数据的时候再调用Complete。在你认为较为空闲的时候开始你的Job。

如果不用写入，使用ReadOnly标记NativeContainer提高性能。

unity的profiler里，主线程的WaitForJobGroup标记表明它正在等一个job完成，可以跟踪JobHandle.Complete找到这个依赖。

job有个Run函数可以让你直接在主线程调用执行任务，用于代码调试。

job里不要分配托管内存，非常慢。没法利用burst编译提高性能。