#枚举和迭代器

不用中文写一遍不踏实系列

本文准确描述C#5.0的枚举和迭代器。

enumerator是个对象，是在一连串值上的一个"光标"，它是只读的，并且只能向前移动。它实现下述接口之一：
'System.Collections.IEnumerator'
'System.Collections.Generic.IEnumerator<T>'
其实具备MoveNext方法和Current属性的对象在技术上来说就可以被当作enumerator了。

enumerable对象逻辑上表示能够提供那个"光标"（也就是enumerator对象）的一连串值。它或者实现IEnumerable/IEnumerable<T>，或者有一个返回enumerator对象的GetEnumerator方法就行。


枚举的两种写法

'''
// 高层级写法
foreach (char c in "beer")
    Console.WriteLine (c);

// 低层级写法
using (var enumerator = "beer".GetEnumerator())
while (enumerator.MoveNext())
{
    var element = enumerator.Current;
    Console.WriteLine (element);
}
'''

特别的，只要你实现了'System.Collections.IEnumerable'，因为它有Add方法，你就可以一步完成enumerable对象的初始化和赋值：
'''
List<int> list = new List<int> {1, 2, 3};
// 编译器会翻译成
// List<int> list = new List<int>();
// list.Add (1);
// list.Add (2);
// list.Add (3);
'''

如果说foreach是enumerator的消费者，迭代器iterator就是enumerator的生产者。
包含了yield语句的方法、属性或者索引器就是iterator。iterator必须返回下述接口之一：
'''
// Enumerable interfaces
System.Collections.IEnumerable
System.Collections.Generic.IEnumerable<T>
// Enumerator interfaces
System.Collections.IEnumerator
System.Collections.Generic.IEnumerator<T>
'''
iterator返回的类型不同，语义就不同。

比如这就是一个iterator:
'''
static IEnumerable<string> Foo()
{
yield return "One";
yield return "Two";
yield return "Three";
}
'''

每次yield return返回一个值,离开生产者，等消费者调用MoveNext时再次进入执行后续代码。

yield break用来退出iterator，不能用return。

yield return不能用在有catch分句的try语句里，也不能用在catch或者finally代码块里。因为编译器会把iterator翻译成一个有MoveNext、Current和Dispose成员的原始类，这个在有异常处理时特别复杂。
iterator（生产者）的yield return可以在try...finally代码里使用。但是外面的调用者（消费者）写的如果是MoveNext这样的暴露代码，并且没有枚举到结尾，也没有dispose，就会绕过finally代码，这是个坑。你需要用using语句离开时强制调用dispose，例如：
'''
string firstElement = null;
var sequence = Foo();
using (var enumerator = sequence.GetEnumerator())
{
    if (enumerator.MoveNext())
    {
        firstElement = enumerator.Current;
    }      
}
'''

iterator可以高度组合，作为生产者的iterator可以作为消费者来驱动另一个生产者，例如：
'''
static void Main()
{
    foreach (int fib in EvenNumbersOnly (Fibs(6)))
    Console.WriteLine (fib);
}
static IEnumerable<int> Fibs (int fibCount)
{
    for (int i = 0, prevFib = 1, curFib = 1; i < fibCount; i++)
    {
        yield return prevFib;
        int newFib = prevFib+curFib;
        prevFib = curFib;
        curFib = newFib;
    }
}
static IEnumerable<int> EvenNumbersOnly (IEnumerable<int> sequence)
{
    foreach (int x in sequence)
        if ((x % 2) == 0)
            yield return x;
}
'''