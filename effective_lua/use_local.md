# Use local： 使用 local修饰的局部变量
在运行任何代码之前，Lua将源代码转换（预编译）为内部格式。此格式是一系列虚拟机指令，类似于真实CPU的机器代码，然后通过C代码解释此内部格式。c代码本质上是一个带着很大的switch的while循环，其中每个case处理每条指令。
也许你已经了解过，Lua自5.0版以来使用了基于寄存器的虚拟机。这个虚拟机的“寄存器”不对应CPU中的实际寄存器（对应实际寄存器就会不可移植，而且实际可用的寄存器数量非常有限），而是使用堆栈（实现为数组加上一些索引）来实现它的寄存器。每个活动的函数都有个激活记录（activation record），里面有自己的寄存器。激活记录其实就是一段堆栈。每个函数最多可使用250个寄存器，因为每条指令只有8位用于寄存器。
鉴于寄存器数量之多，Lua预编译器能够将所有的local变量存储于寄存器中，访问局部变量非常之快。例如，如果a和b是局部变量，那么Lua语句`a = a + b`将生成一条指令：`ADD 0 0 1`（假设a和b分别在寄存器0和1中）。作为比较，如果a和b都是全局的，加法的代码将是这样的：
```
GETGLOBAL 0 0 ; a
GETGLOBAL 1 1 ; b
ADD 0 0 1
SETGLOBAL 0 0 ; a
```
因此，提高Lua程序性能的最重要规则之一：使用 local修饰的局部变量
如果需要从你的程序中压榨性能，有几个显而易见的地方可以使用local。
* 在长循环中调用函数，可以将函数分配给局部变量。例如，代码
  ```
    for i = 1, 1000000 do
        local x = math.sin(i)
    end
  ```
  比下面的代码慢30%：
  ```
    local sin = math.sin
    for i = 1, 1000000 do
        local x = sin(i)
    end
  ```
* 访问外面的local变量（即函数体外面的local变量）虽然没有访问函数内的local变量快，但是仍然比访问全局变量快。下面的代码：
    ```
    function foo (x)
        for i = 1, 1000000 do
            x = x + math.sin(i)
        end
        return x
    end
    print(foo(10))
    ```
可以优化为在函数体外声明一次sin：
    ```
    local sin = math.sin
        function foo (x)
            for i = 1, 1000000 do
                x = x + sin(i)
            end
            return x
        end
    print(foo(10))
    ```
