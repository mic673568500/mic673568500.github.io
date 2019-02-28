











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