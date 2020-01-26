---
date: 2020-01-25
title: 初步学习JDK的Enum类源码
tags: ['Java', Enum', 'JDK']
categories: ['Java']
---

最近有空看了 Enum 类的源码，下面只挑重点部分记录一下。

### 抽象类

所有 enum 类型自动继承自 Enum 类，由于 java 的单继承特性，也就使得 enum
类型不能继承，个人觉得这是缺点之一，平时开发中会有一些公共的代码无法通过继承来共享。

### 核心成员为 name 和 ordinal

```
private final String name;
private final int ordinal;
```

name 是定义 enum 时的名字，ordinal 是位置，从 0 开始

### 实现 Comparable 接口

Enum 可比较，实现的 compareTo 方法如下：

```
   public final int compareTo(E o) {
        Enum<?> other = (Enum<?>)o;
        Enum<E> self = this;
        if (self.getClass() != other.getClass() && // optimization
            self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }
```

通过 ordinal 可以很方便的实现 compareTo, 直接使用 2 个 enum 的 ordinal 相减。

### valueOf 方法

valueOf 方法是传入一个 enum 的 name，返回对应的 enum 对象。猜测的实现方式应该是维护一个单例的 HashMap，里面包含 name 到 enum 对象的映射。

```
public static <T extends Enum<T>> T valueOf(Class<T> enumType, String name) {
T result = enumType.enumConstantDirectory().get(name);
```

这个 HashMap 通过`enumType.enumConstantDirectory()`获得的，
这个方法是`Class`类的方法。

```
Map<String, T> enumConstantDirectory() {
        if (enumConstantDirectory == null) {
            T[] universe = getEnumConstantsShared();
            if (universe == null)
                throw new IllegalArgumentException(
                    getName() + " is not an enum type");
            Map<String, T> m = new HashMap<>(2 * universe.length);
            for (T constant : universe)
                m.put(((Enum<?>)constant).name(), constant);
            enumConstantDirectory = m;
        }
        return enumConstantDirectory;
    }
private volatile transient Map<String, T> enumConstantDirectory = null;
```

`enumConstantDirectory`定义为`volatile`为`transient`。在多线程情景下使用`volatile`能保证`enumConstantDirectory`读取和写入都是主内存而不是线程的工作内存。使用`transient`是在序列化时忽略该字段，因为这个字段其实只用于本地，不需要作为信息进行网络传输等 IO 操作。

这里实现了一个懒加载，只有第一次调用 valueOf 的时候才构造这个 map.

然而这里有一个原子性的问题，如果多个线程同时检测到`enumConstantDirectory`为 null，会创建多次 HashMap，但是最终主存上只会有一份，多创建的 HashMap 最后会被垃圾回收掉，这样也问题不大。

### 实现 Serializable 接口

这里面学到的东西比较多。Enum 实现了 Serializable 接口，说明它是可以被序列化的。它有一段很值得学习的代码:

```
/**
 * prevent default deserialization
 */
private void readObject(ObjectInputStream in) throws IOException,
    ClassNotFoundException {
    throw new InvalidObjectException("can't deserialize enum");
}

private void readObjectNoData() throws ObjectStreamException {
    throw new InvalidObjectException("can't deserialize enum");
}
```

这里写了，它是用于禁用默认反序列化的。为什么要这样做呢？先从 Enum 是怎么做序列化和反序列化开始。

我们知道，序列化是通过`ObjectOutputStream`的`writeObject`方法实现的。

```
public final void writeObject(Object obj) throws IOException {
        if (enableOverride) {
            writeObjectOverride(obj);
            return;
        }
        try {
            writeObject0(obj, false);
        } catch (IOException ex) {
            if (depth == 0) {
                writeFatalException(ex);
            }
            throw ex;
        }
    }
```

发现它调用的是`writeObject0`方法。再看`writeObject0`方法,

```
if (obj instanceof String) {
    writeString((String) obj, unshared);
} else if (cl.isArray()) {
    writeArray(obj, desc, unshared);
} else if (obj instanceof Enum) {
    writeEnum((Enum<?>) obj, desc, unshared);
} else if (obj instanceof Serializable) {
    writeOrdinaryObject(obj, desc, unshared);
} else {
    if (extendedDebugInfo) {
        throw new NotSerializableException(
            cl.getName() + "\n" + debugInfoStack.toString());
    } else {
        throw new NotSerializableException(cl.getName());
    }
}
```

它判断如果是 Enum 对象，就调用`writeEnum`方法。再看进去：

```
    private void writeEnum(Enum<?> en,
                           ObjectStreamClass desc,
                           boolean unshared)
        throws IOException
    {
        bout.writeByte(TC_ENUM);
        ObjectStreamClass sdesc = desc.getSuperDesc();
        writeClassDesc((sdesc.forClass() == Enum.class) ? desc : sdesc, false);
        handles.assign(unshared ? null : en);
        writeString(en.name(), false);
```

核心就在这里，原来 Enum 对象是先把 TC_ENUM 魔法数字，再把它的类描述和 name 写进去就行了。

反序列化是通过`ObjectInputStream`的`readObject`方法实现的。
一路看进去，核心是在`readEnum`方法里面：

```
String name = readString(false);
Enum<?> result = null;
Class<?> cl = desc.forClass();
if (cl != null) {
    try {
        @SuppressWarnings("unchecked")
        Enum<?> en = Enum.valueOf((Class)cl, name);
        result = en;
    } catch (IllegalArgumentException ex) {
        throw (IOException) new InvalidObjectException(
            "enum constant " + name + " does not exist in " +
            cl).initCause(ex);
    }
    if (!unshared) {
        handles.setObject(enumHandle, result);
    }
}
```

先取出来 name 和 class，再通过`Enum.valueOf`获得 Enum 对象。

可以看到，Enum 不会使用默认的对象的反序列化方法进行反序列化，而是有自己的实现，那么，为什么还要禁用对象的默认反序列化方法呢？原因是 Enum 是要确保单例，如果网络传输过来一段被黑客篡改过的字节，把 TC_ENUM 魔法数字改成 TC_OBJECT，这样，反序列化就会调用`readOrdinaryObject`，这里面会判断，如果有重写`readObject`方法，会调用该方法，由于我们重写了这个方法并抛了一个异常，就避免了这种攻击的情况发生。同样的，如果只有类信息而没有数据，会检查有没有重写`readObjectNoData`, 如果有就调用，这样也避免了攻击。
