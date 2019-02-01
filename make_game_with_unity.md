#准备工具

Unity：游戏引擎，从官网下载
Unity 破解工具：可以使用Unity的高级功能，游戏发布时再买版权
Visual Studio:集成开发工具，安装Unity时会带普通版本，但是最好用专业版。
VS Code：lua，json,md等语言的编辑器
git或svn：版本控制工具
github：免费的仓库


#确定游戏类型，游戏发布的平台，目标机型

PC、主机上的横版过关或平台跳跃，2D或3D未确定
需要考虑向移动平台移植
可本地多人和网络多人，其中一台玩家的机器做server

#技术选取

网络库：C#的Telepathy，服务器客户端对称
输入库：Rewired，可视化配置，不用关心各种输入设备
序列化库：protobuf

#框架
方案1：写个主循环类，所有子系统都在这个类里明确控制启动、更新和结束顺序，做一个永久的game object，挂个脚本，调用主循环类。
方案2：使用Unity2018的ecs，每个子系统继承unity的ComponentSystem, 标记每个子系统的依赖项，将更新顺序交给unity的内部实现。
为了能使用Unity最新的job system，提高游戏性能，这里选择方案2


2019.1.31
测试了子系统的定期更新功能
看了下rewired自带的玩家编辑的控制器重定向功能
决定玩家修改输入控制的功能往后放，放在demo后，游戏的中后期做。

2019.2.1
实测，unity当前的ecs系统十分难用  "com.unity.entities": "0.0.12-preview.23"
