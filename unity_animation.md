# 原动画系统
使用原动画系统的原因：

不需要 ik， 不需要Humanoid animation retargeting。 需要动画融合。
unity 5.1 - 5.3， animator的内存开销大，无法动态加载动画片段。

# 新动画系统
##影响动画CPU性能的因素
###同屏角色数

影响 Camera.Render（MeshSkinning.Render：PutGeometryJobFence，Mesh.DrawVBO）

Animators.Update（DirtySceneObjects,WriteAnimatedValues,ProcessAnimations）

MeshSkinning.Update(CalsMatrices,更新骨骼矩阵，跟面片无关)

近似线性增长

###单角色的mesh复杂数

Camera.Render （PutGeometryJobFence，蒙皮计算，分发job，等待）

线性增长

###单角色的骨骼数

Animators.Update（DirtySceneObjects）

MeshSkinning.Update(CalsMatrices)

###引擎的选项

multithreadendering选项, 新增render线程，大幅提高Camera.Render的CPU效率，提高帧率

动画系统:optimize game objects选项，很多 transform不再更新，降低骨骼数目对主线程的影响。也能提高帧率。multithreadendering开启时，优化不再明显，因为瓶颈出现在render线程，减少主线程耗时对fps提高不明显（gfx.waitforopresent等待），但是仍然要优化，可以把CPU时间让给游戏逻辑；但是无法使用布娃娃或者换装系统

动画类型选项：

- generic耗时（打开optimize） << generic耗时 < legacy耗时
- generic耗时（打开optimize）< humanoid
- humanoid 更省内存
- legacy 适合位移、缩放等简单UI 特效动画

apply root motion选项：如果不必要，不去勾选； 开启optimize game objects可优化

animation blend：
增大processanimations的开销， 避免频繁blend，替换不必要的blend tree 和 layers

高CPU耗时的函数
Camera.Render （PutGeometryJobFence受mesh复杂度影响，蒙皮时间长）
Animators.Update 不受mesh复杂度影响
MeshSkinning.Update 不受mesh复杂度影响


##动画内存的优化
anim.compression选项 optimal < keyframe reduction < off

误差调大，压缩率高，可以着重调大rotation error, 在看不出来误差的情况下压缩率也高

非constant曲线数目决定内存大小，dense曲线比stream曲线更省

humanoid储存于muscle空间，动画曲线数目更少
- 降低动画精度，尾数截断，将其他曲线变成constant曲线

#多角色场景动画替代方案
### bake mesh
- 预处理：利用skinnedmeshrenderer.bakemesh对同种怪物进行烘焙，将蒙皮网格转化为普通网格
- 运行时：根据动画名和时间，获取对应的网格数据进行渲染。
推荐 MeshAnimator插件

消除Animators.Update和MeshSkinning.Update的调用，避免渲染时skinning运算的等待

增加内存开销，无法进行blend

### GPU skinning
将skinning过程转移到GPU中去，unity无法在降低手机上的动画开销

- 将蒙皮网格进行采样，将骨骼节点的动画矩阵信息以纹理的方式进行存储
- 在GPU中完成定点运算

