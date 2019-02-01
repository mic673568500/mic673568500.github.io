Interop marshaling管理了数据在托管和非托管内存之间方法的参数或返回值如何传递。
platform invoke（特指微软的CLR提供的跨平台调用方式），是指托管代码调用非托管代码库里导出的函数。

blittable types在托管内存和非托管内存中有相同的表示，代码上相互传递时不需要转换。
platform invoke返回的结构必须是blittable类型。

下述类型是blittable类型：
- System.Byte
- System.SByte
- System.Int16
- System.UInt16
- System.Int32
- System.UInt32
- System.Int64
- System.UInt64
- System.IntPtr
- System.UIntPtr
- System.Single
- System.Double
- blittable类型的一维数组
- 只包含blittable类型的formatted value（格式化类型），以及被marshaled为格式化类型的class。
  
formatted value（格式化类型）是一个显示控制其成员在内存中布局的复杂类型。

对象的引用，不是blittable类型。
bool，不是blittable类型。