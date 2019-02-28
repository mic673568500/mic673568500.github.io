#耗时瓶颈定位
Render.OpaqueGeometry  不透明渲染
 - MeshRenderer.Render 非蒙皮网格渲染
 - Mesh.DrawVBO draw call
 - MeshSkinning.Render 蒙皮网格的渲染
 - Material.SetPassFast 跟材质数目一致，跟批次成正比， 每次提交draw call之前，修改Render State
Render.TransparentGeometry 半透明渲染
Culling 相机裁剪
 - ParticleSystem.ScheduleGeometryJobs与相机窗口内部粒子系统个数成正比，主线程任务调度
 - ParticleSystem.GeometryJobs子线程中做粒子模拟
Camera.ImageEffects 图像后处理

#不透明物体渲染影响因素
- 与物体个数 draw call 数成正比
- 面片数不敏感
- 与场景中的材质数成正比
- 对CPU的影响程度： Draw Call > 材质数目 > 面片数

#粒子系统渲染影响因素
- 粒子系统个数
- 粒子个数
- CPU的影响：粒子系统个数 > 粒子数

#shadow渲染性能试验
 - 高分辨率比中、低分辨率耗时高很多，推荐中、低

#PBR渲染性能试验
- 对CPU端开销并不是很高，主要增加开销在GPU端

#多线程渲染
 - 推荐开启
 - 不能用在渲染模块的负载分析

#像素填充率，Over Draw
- 像素着色器压力大
- unity 5.x渲染管线，先将场景渲染到rt上，再把rt贴到 framebuffer上
- 可通过Screen.SetResolution降低渲染分辨率
## UI界面over draw优化
 - 减少UI层数
 - 屏幕被UI完全遮挡时，隐藏场景内容
 - 将多层UI渲染到一张rt重复利用，只在元素变化时重新渲染
## 粒子系统over draw优化
 - 减少特效层数
 - 用更紧致的mesh替代方形
