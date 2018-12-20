# 避免运行时编译动态代码
虽然Lua编译器比很多其他语言的编译器要高效很多，但是除非必须需要真正的动态代码（例如，你想让最终的用户输入代码），不然不要在程序里编译代码。
举个例子，下述代码用100000个函数返回1-100000个常量，来创建一个table：
```
local lim = 10000
local a = {}
for i = 1, lim do
    a[i] = loadstring(string.format("return %d", i))
end
print(a[10]()) --> 10
```
这个代码运行了1.4秒。
换做使用闭包代替动态编译，下述代码只用1/10的时间就创建了这100000个函数。
```
function fk (k)
    return function () return k end
end
local lim = 100000
local a = {}
for i = 1, lim do a[i] = fk(i) end
print(a[10]()) --> 10

```