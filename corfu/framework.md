# Corfudb 框架介绍

## 1. Corfu Objects

### 1. CorfuCompileProxy

 Java Object通过```CorfuCompileProxy```类封装为Corfu Object。

```java
// CorfuCompileProxy.java
package org.corfudb.runtime.object;

public class CorfuCompileProxy<T> implements ICorfuSMRCompileProxyInternal<T> {
    // 底层对象，存储着对象的实际状态（即version），并提供线程安全的访问方法。
    VersionLockedObject<T> underlyingObject;
    // 与log进行交互。
    final CorfuRuntime rt;
    // The ID of the stream of the log.
    final UUID streamID;
    // 序列化器，提供对应种类的日志序列化方法。
    ISerializer serializer;
    
    /* Java Object的实际访问方法如下:
    ** 只读(Accessor)：调用access方法
    ** 只写(Mutator)：调用logUpdate方法
    ** 读写(AccessorMutator)：调用logUpdate进行写，并调用getUpcallResult获得最新结果。
    */
    public <R> R access(ICorfuSMRAccess<R, T> accessMethod, Object[] conflictObject);
    public long logUpdate(String smrUpdateFunction, final boolean keepUpcallResult,
                          Object[] conflictObject, Object... args);
    public <R> R getUpcallResult(long timestamp, Object[] conflictObject);
}
```

以```SMRMap```类为例，其实际创建时会返回一个```SMRMap$CORFUSMR```类的实例，该实例同时绑定了一个```CorfuCompileProxy```的实例，该proxy实例中的underlyingObject就是实际操作的对象。

对```SMRMap```调用```get```和```put```方法，首先会调用```SMRMap$CORFUSMR```的对应方法（access/logUpdate/getUpcallResult），该调用会被转发到proxy的同名方法。根据是否处于一个事务中，proxy会做出不同应对，但最终都是调用underlyingObject的对应方法进行访问。后者提供了线程安全的访问方法。

- [ ] ```SMRMap```构造时的setArguments选项。
- [ ] ```SMRMap$CORFUSMR```是从哪来的？

### 2. VersionLockedObject

#### 1. access

这个函数接受三个函数作为参数：直接访问函数、更新函数、访问函数。

逻辑如下：先使用直接访问函数进行访问，如果出现版本不一致等情况，则调用更新函数更新本地对象，然后再调用访问函数得到最新的值。

直接访问函数：通过调用smrStream的pos()得到一个Long类型的值，为

#### 2. logUpdate

#### 3. getUpcallResult

## 2. Corfu Views

### 1. AddressSpaceView

### 2. LayoutView

### 3. SequencerView

### 4. StreamView

### 5. ObjectView

### 6. ManagementView

### 7. LayoutManagementView



## 3. Log Data 和 Log Entry

```LogData```类是日志中存储的所有内容的抽象，包括以下几类：

+ 0：DATA					// 有效内容
+ 1：EMPTY
+ 2：HOLE
+ 3：TRIMMED
+ 4：RANK_ONLY

```LogEntry```类负责日志中存储的有效条目，也即上述分类中的 DATA ，分为以下几类：

+ 0：NOP
+ 1：SMR                          // 也即Corfu Object的存储形式```SMREntry```
+ 2：MULTIOBJSMR　　// ```MultiObjectSMREntry```，由```MultiSMREntry```组成的容器。
+ 3：MULTISMR              //  ```MultiSMREntry```，由多个```SMREntry```组成的序列，表示事务产生的多条记录
+ 4：CHECKPOINT　　  // ```CheckpointEntry```，事务校验点记录，分为START、CONTINUATION、END三类。

```LogData```和```LogEntry```都实现了对应的序列化和反序列化方法。

+ T -> WriteT; (T=Byte/Short/Integer/Long/Boolean/Float/Double)

+ Bytes[]/ ByteBuf -> WriteInt(length); WriteBytes;

+ String -> WriteInt(length); WriteBytes;

+ Map -> WriteInt(size); for each entry do: serialize(key), serialize(value);

+ Set/List -> WriteInt(size);  for each entry do: serialize(entry);

+ UUID -> WriteLong; WriteLong;

+ Layout -> b = toJson(layout).getBytes(); WriteInt(b.length); WriteBytes(b);

+ SMREntry -> WriteShort(SMRmethod.len); WriteBytes(SMRmethod.getBytes());

  ​				  -> WriteByte(serializerType); WriteByte(SMRarguments.len);

  ​				  -> for each SMRargument x, do: serializerType.serialize(x);

## 4. Corfu Services

> ```
>  |------------------------------------------------------------|
>  |                                                            |
>  |                                  |----------------------|  |
>  |                   ---------------|  SequencerServer     |  |
>  |                  /               |----------------------|  |
>  |    |---------|  /                                          |
>  |    | Service | /                 |----------------------|  |
> -----|  Router |-------------------|  LogUnitServer       |  |
>  |    |---------| \                 |----------------------|  |
>  |                 \                                          |
>  |                  \               |----------------------|  |
>  |                   ---------------|  LayoutsServer       |  |
>  |                                  |----------------------|  |
>  |                                                            |
>  |------------------------------------------------------------|
> ```







