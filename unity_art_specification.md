# 大前提
程序工程和美术工程的各项设置一致，是游戏从生产到上线各个阶段表现正常的前提。需要有人负责相关设置的一致性，设置变更必须经过美术和技术的协商，在经过充分评估后，由负责人统一提交。负责人通常是程序或者TA。
这些一般在前期固定，后期有少量修改。
目前确认的设置包括：
- 游戏或工程设置：
 - build setting（如果美术工程里有资源打包操作）
 - project setting - playersetting（color space 非常重要）
 - project setting - graphics
 - project setting - quality
- scene设置：游戏运行的scene，和美术检查效果的scene，要同步。包括（prefab，meta，light setting）等。


## 光照相关规划
1. 每房间摆放光源
2. 最多一个实时像素光源，可尝试加最多4个实时顶点光源作为辅光，设置环境光
- *原因：base pass：一个像素光，4个顶点光，其他sh光，光照贴图，环境光，自发光；前向渲染默认只有base pass 的方向光有影子，除非开 multi_compile_fwdadd_fullshadows；其他pass：其他每个像素光一个pass*
3. 不开雾
4. 房间烘焙，家具实时

## 场景光照设置（window-rendering-light setting）
realtime lighting: false
     保证没有实时的全局光照
mix lighting : true
     使用混合光（实时+烘焙）
baked GI: true
     拥有全局光照的效果只能用烘焙保证效率
lighting mode: shadowmask
     直接光对静态物体用烘焙，对动态物体用实时，有动态投影。
compress lightmaps: true
     光照贴图可以自动生成不同平台默认的压缩格式，如果不勾选，之后需要在贴图上设置，保证移动平台上的贴图是压缩的
directional mode: directional
     生成了颜色贴图和方向贴图，我们的脚本支持使用这两个贴图

烘焙间接光，直接光用实时光和实时阴影，房间尝试用（directional light +  spotlight cookie）做阴影


## 贴图设置：
不带alpha通道的贴图（含烘焙后的颜色贴图）
format：
安卓：RGB Compressed ETC2 4 bits
ios：RGB Compressed PVRTC 4 bits
带alpha通道的贴图（含烘焙后的方向贴图）
format
安卓：RGBA Compressed ETC2 8 bits
ios：RGBA Compressed PVRTC 4 bits
ios: UI贴图，使用PVRTC会模糊，尝试使用ASTC，压缩率高且不会模糊。

## shader工作流程
1. 模型的材质使用custom shader
为了减少游戏中的shader变体，以及后续的shader优化，项目中的3d资源禁止使用unity build-in standard相关的shader，而是使用自己的shader：
Custom/Standard-2Sided， Custom/Standard和Custom/Standard (Specular setup)
美术在倒入模型到unity工程时，模型的材质往往是默认为standard的，需要替换为custom的对应版本。
（可使用工具Windows-ShaderTool，批量替换build-in下的shader为custom下的shader）
2. 每个shader的变体需要有占位材质
对于shader因为keywords不同而产生的变体，需要在Assets/Plugin/CommonTools/Shader/Custom 下建立一个使用相同变体的占位材质。
（可使用工具Windows-ShaderTool，批量扫描出所有的变体，并用每个变体出现时的第一个材质生成占位材质。另外，通过检查占位材质，你也能发现是否有build-in shader没有清理干净）

占位材质会在构建时跟custom shader打入同一个ab文件内，所有的模型材质将依赖这个ab文件，在游戏中，他们的shader变体也从这个文件加载。没有对应占位材质的资源在游戏中将会无法找到shader而显示异常。（紫红状态）
3. 合理设计和控制shader变体数
应当合理设计和控制shader变体的数量，减少占位材质ab文件的大小和游戏运行时的内存。



场景总体规划
低配机要求：每场景（模型+投影）6w面
5W面模型 + 1W面的减面模型投影，1:5的减面是否可行

模型制作要求
1 移动平台 每个mesh首先要求：300 - 1500 面
2 UV贴图，接缝和硬边（双重顶点）尽可能地少


测试数据及评价：
总体：1 无法接受多个像素光；2 顶点光源数，模型是否接受投影对性能几乎没有影响；3 在华为和联想上，开软影，影响较小

测试基于以下配置：一个像素方向光，一个顶点光，low shadow， 所有模型接收投影，std pbr，1024贴图大小

三星A3000
模型个数|	顶点数/面数|	帧率|	其他参数|	 评价
----|--|--|--|-- 
29模型， 22软影|	75.8k/81.9k|	27-28|	dc57，tr81.9，sp28|	 刚刚低于标准	 
25模|	39.9k/42k|	限帧， 30+|	dc35，tr42，sp22|	 相当好	 
39模|	58.4k/61.4k|	29-30|	dc45，tr61.4，sp24|	 可行	 
50模|	75.8k/81.9k|	27-28|	dc56，tr81.9，sp30|	 刚刚低于标准	 
 	 	 	 	 	 

华为P6，development build包无法运行，看不到具体数据，只有模型数和帧率
模型个数|	顶点数/面数|	帧率|	其他参数|	 评价	
----|--|--|--|--
22+ 模型 ， 22+ 软影|	 约75-80k|	限帧，30+|	 -|	 可行	 
33+ 模， 33+ 软影|	 -|	22-24|	 -|	 -	 
33+ 模|	 约58-60k|	26-27|	 -|	 刚刚低于标准	 
 	 	 	 	 	 
 	 

联想4.2
模型个数 |顶点数/面数|	帧率|	其他参数|	 评价	
----|--|--|--|-- 
29模型|	39.9k/42.0k|	19-25|	SetPass : 24    Draw Calls: 35 Total Batches: 35 Tris: 42.0k Verts: 39.9k|	 低于标准	 
29模  21软影|	73.0k/81.9k|	19-25|	SetPass: 27    Draw Calls: 56 Total Batches: 56 Tris: 81.9k Verts: 74.8k|	 低于标准	 
40模|	58.4k/61.4k|	16-23|	SetPass : 24    Draw Calls: 46 Total Batches: 46 Tris: 61.4k Verts: 58.4k|	 低于标准	 
 	 	 	 	 	 
 	 	

 

其他问题：
备选方案：
shadowmask
烘焙间接光，直接光用实时光，静态物体（房间）烘焙阴影，阴影贴图

反射：
尝试reflection probe 做不规则物体的反射

性能瓶颈在GPU，Render thread？
合批，室内光？


