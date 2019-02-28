AssetBundle包含了平台相关的资产。
一个bundle里的资源可能依赖另一个bundle里的资源。
bundle可选压缩，内建算法LZMA或LZ4。
bundle可以做dlc，可以减少初始安装大小，降低运行时内存压力
AssetBundle可以指磁盘上的bundle文档
AssetBundle可以指代码中的对象

打AssetBundles可以有以下策略：
逻辑实体分组，比如：
把UI的纹理和布局数据打在一起；
把角色的模型和动画打在一起；
把多场景使用的模型和纹理打在一起；

逻辑实体分组适合dlc的情形了，问题就是负责分配的人员需要准确知道每个资产何时使用；

类型分组策略，比如：
将音频打在一起，将shader打在一起
类型分组策略适合多平台的情形，比如，如果音频的压缩格式在windows上和mac上是一致的，那么只需要打同一个即可；shader可能不行；而texture因为在不同版本的压缩格式几乎不变，打出的包可以跨版本使用。

小贴士：
把经常需要更新的物体和不经常需要改变的物体分开打
更有可能同时加载的物体打在一起：比如模型，它的纹理和它的动画；
如果多个包中的资产依赖另一个不同的包中的资产，把依赖项放到单独的包中
如果两个物体不可能同时加载，例如标准资产和高清资产，不要打在一起
如果少于50%的内容是频繁同时加载的，考虑拆分
如果小包（5-10个资产）经常同时加载，考虑合并它们
如果一堆物体只是同一个物体的不同版本，考虑使用 variant


BuildAssetBundleOptions.None使用LZMA格式压缩，产生最小的尺寸和最长的载入时间，并且使用时要解压整个包，解压后，使用LZ4重新压缩到硬盘上，并且使用时不需要全部解压了。所以这个最好用在所有资产都同时使用的情况。LZMA格式压缩最好只用在最初下载包的时候。通过 UnityWebRequestAssetBundle加载包会自动重新用LZ4压缩到硬盘上。如果你用其他方式下载包体，可以使用AssetBundle.RecompressAssetBundleAsync重新压缩。
BuildAssetBundleOptions.UncompressedAssetBundle就是不压缩。
BuildAssetBundleOptions.ChunkBasedCompression使用的LZ4，压缩后比LZMA大，但是可以部分解压。












BuildTarget，选择打包平台
打包后会有多出很多文件，2*(n+1)个
每个bundle会有一个对应的manifest文件
此外，还有一个manifest bundle
scene的assetbundle跟普通的assetbundle结构有些不同
每个bundle的manifest文件里包含 crc数据，依赖数据，资产等信息


一个对象的依赖如果没有在任何包里，则直接复制跟这个对象打入相同的包。如果多个包依赖他，则他被复制打入多个包里。


AssetBundle.LoadFromMemoryAsync
传入字节数据，或者传入crc值，如果是LZMA则会解压，LZ4则会在压缩状态载入。

````
using UnityEngine;
using System.Collections;
using System.IO;

public class Example : MonoBehaviour
{
    IEnumerator LoadFromMemoryAsync(string path)
    {
        AssetBundleCreateRequest createRequest = AssetBundle.LoadFromMemoryAsync(File.ReadAllBytes(path));
        yield return createRequest;
        AssetBundle bundle = createRequest.assetBundle;
        var prefab = bundle.LoadAsset<GameObject>("MyObject");
        Instantiate(prefab);
    }
}
```

AssetBundle.LoadFromFile
加载非压缩文件非常高效，加载LZMA时要全部解压后再读入内存。
```
public class LoadFromFileExample extends MonoBehaviour {
    function Start() {
        var myLoadedAssetBundle = AssetBundle.LoadFromFile(Path.Combine(Application.streamingAssetsPath, "myassetBundle"));
        if (myLoadedAssetBundle == null) {
            Debug.Log("Failed to load AssetBundle!");
            return;
        }
        var prefab = myLoadedAssetBundle.LoadAsset.<GameObject>("MyObject");
        Instantiate(prefab);
    }
}
5.3之前，安卓的Application.streamingAssetsPath会失败，因为在jar包里。
```

WWW.LoadFromCacheOrDownload
会被UnityWebRequest替代
远程的资产会被自动缓存，有个工作线程来解压和缓存，然后类似AssetBundle.LoadFromFile读取。
www对象因为缓存压力大，bundle最好小一点，几MB以内；在小内存的平台上，比如移动平台，确保每次只下载一个bundle，防止内存飙升。
如果缓存文件夹空间不足，会逐步删掉最近最少使用的buldle，如果空间还是不足，会缓存在内存中。
改变第二个参数：version，可以强制更新缓存。

UnityWebRequest
```
IEnumerator InstantiateObject()

    {
        string uri = "file:///" + Application.dataPath + "/AssetBundles/" + assetBundleName;        UnityEngine.Networking.UnityWebRequest request = UnityEngine.Networking.UnityWebRequest.GetAssetBundle(uri, 0);
        yield return request.Send();
        AssetBundle bundle = DownloadHandlerAssetBundle.GetContent(request);
        GameObject cube = bundle.LoadAsset<GameObject>("Cube");
        GameObject sprite = bundle.LoadAsset<GameObject>("Sprite");
        Instantiate(cube);
        Instantiate(sprite);
    }
```

Loading AssetBundle Manifests
加载那个额外的assetbundle，加载AssetBundleManifest类型的对象，即可。

```
AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
AssetBundleManifest manifest = assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
```

加载一个叫assetBundle的bundle的所有依赖的bundle
```
AssetBundle assetBundle = AssetBundle.LoadFromFile(manifestFilePath);
AssetBundleManifest manifest = assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
string[] dependencies = manifest.GetAllDependencies("assetBundle"); //Pass the name of the bundle you want the dependencies for.
foreach(string dependency in dependencies)
{
    AssetBundle.LoadFromFile(Path.Combine(assetBundlePath, dependency));
}
```

AssetBundle.Unload(true)卸载所有的GameObjects及其依赖项，不包括复制出来的GameObjects。
Unload(false)卸载bundle，断开与实例的链接，如果再LoadAsset()的话会出现2份。
因此，大多数项目需要用 AssetBundle.Unload(true)
如果使用了AssetBundle.Unload(false)，需要手动调用 Resources.UnloadUnusedAssets或者载入一个新的scene让Resources.UnloadUnusedAssets自动调用。