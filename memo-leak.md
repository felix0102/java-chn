### 内存泄漏

#### **内存泄漏示例**

下面展示的这个示例更具体一些。

在 Java 中，创建一个新对象时，例如 `Integer num = new Integer(5)`，并不需要手动分配内存。因为 JVM 自动封装并处理了内存分配。在程序执行过程中，JVM 会在必要时检查内存中还有哪些对象仍在使用，而不再使用的那些对象则会被丢弃，并将其占用的内存回收和重用。这个过程称为“[垃圾收集](http://blog.csdn.net/renfufei/article/details/53432995)”。JVM 中负责垃圾回收的模块叫做“[垃圾收集器（GC）](http://blog.csdn.net/renfufei/article/details/54407417)”。

Java 的自动内存管理依赖 [GC](http://blog.csdn.net/column/details/14851.html)，GC 会一遍又一遍地扫描内存区域，将不使用的对象删除。简单来说，**Java 中的内存泄漏，就是那些逻辑上不再使用的对象，却没有被 [垃圾收集程序](http://blog.csdn.net/renfufei/article/details/54144385) 给干掉**。从而导致垃圾对象继续占用堆内存中，逐渐堆积，最后产生 `java.lang.OutOfMemoryError: Java heap space` 错误。

很容易写个 Bug 程序，来模拟内存泄漏：

```java
import java.util.*;
public class KeylessEntry {
    static class Key {
        Integer id;
        Key(Integer id) {
        this.id = id;
        }
        @Override
        public int hashCode() {
        return id.hashCode();
        }
     }
    public static void main(String[] args) {
        Map m = new HashMap();
        while (true){
        for (int i = 0; i < 10000; i++){
           if (!m.containsKey(new Key(i))){
               m.put(new Key(i), "Number:" + i);
           }
        }
        System.out.println("m.size()=" + m.size());
        }
    }
}
```

粗略一看，可能觉得没什么问题，因为这最多缓存 10000 个元素嘛！

但仔细审查就会发现，Key 这个类只重写了 hashCode() 方法，却没有重写 equals() 方法，于是就会一直往 HashMap 中添加更多的 Key。

> 请参考：《[Java 中 hashCode 与 equals 方法的约定及重写原则](http://blog.csdn.net/renfufei/article/details/14163329)》。

随着时间推移，“cached”的对象会越来越多。当泄漏的对象占满了所有的堆内存，[GC](http://blog.csdn.net/renfufei/article/details/53432995) 又清理不了，就会抛出 `java.lang.OutOfMemoryError: Java heap space` 错误。

解决办法很简单，在 Key 类中恰当地实现 equals() 方法即可：

```
@Override
public boolean equals(Object o) {
    boolean response = false;
    if (o instanceof Key) {
       response = (((Key)o).id).equals(this.id);
    }
    return response;
}
```

说实话，很多时候内存泄漏，但是可能功能是正常的，达到一定程度才会出问题。所以，在寻找真正的内存泄漏原因时，这种问题的隐蔽性可能会让你死掉很多很多的脑细胞。
