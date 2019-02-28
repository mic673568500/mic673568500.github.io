#加载

音频：
- 建议使用mp3格式音频文件
- 内存压力大时，可考虑Streaming加载方式，或者较小quality的vorbis格式
- 如果是非及时使用的音效，建议开启load in background提升加载效率
- 大量频发使用的音效，选择decompressed on load降低CPU开销

粒子系统
材质剥离，因为加载很耗时

##assetbundle资源加载
- 切换场景时，尽可能使用assetbundle.load来提升加载效率
- 尽可能使用loadall来加载资源
  - 保证同时使用的资源打包在一起
  - 小，细碎的同种资源打包在一起

##active/deactive频率和耗时
降低频率
复杂UI界面，带有动画组件的game object，不要频繁切换


#内存
## 使用是否合适
###纹理
mipmap统计，mipmap经常很高，不如直接降低图片的分辨率
###网格
3dmax引入的colors或者tangents，uv2数据，不用的可以去掉。静态合批剩内存。
###动画
###render texture
反走样可以调低一点
###粒子系统
## 资源冗余
