当前，需要给gameObject挂GameObjectEntity脚本，数据才能被组件系统访问。

GameObjectEntity在OnEnable时为gameObject建一个带有所有组件的entity。

组件完全相同的entity是同一类型。这个类型由一个EntityArchetype对象表示。EntityArchetype是一个独特的的数组，元素是组件类型。EntityManager使用EntityArchetype将所有实体在chunk里分组。

```
// Using typeof to create an EntityArchetype from a set of components
EntityArchetype archetype = EntityManager.CreateArchetype(typeof(MyComponentData), typeof(MySharedComponent));

// Same API but slightly more efficient
EntityArchetype archetype = EntityManager.CreateArchetype(ComponentType.Create<MyComponentData>(), ComponentType.Create<MySharedComponent>());

// Create an Entity from an EntityArchetype
var entity = EntityManager.CreateEntity(archetype);

// Implicitly create an EntityArchetype for convenience
var entity = EntityManager.CreateEntity(typeof(MyComponentData), typeof(MySharedComponent));
```

EntityManager把组件数据存在16K大小的固定块中，这些块叫做chunk。

一个EntityArchetype对应一堆（相同类型的）chunk。

一个chunk只能属于一个EntityArchetype。

struck ArchetypeChunk是个指向某个chunk的指针。

chunk里包含什么：
- 一个指向Archetype的指针
- 组件的个数，chunck的剩余容量
- 指向SharedComponentData特定值的索引数组，每个SharedComponentData类型，chunk只能有一个特定值。
- 每个组件有个ChangeVersion记录了上次修改的时间。