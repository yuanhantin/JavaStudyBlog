## Java 比较（==, equals)

### 一、= =

==：比较两个对象的引用是否是同一个地址

### 二、equals

object中equals方法调用的就是==，可以在其他类中重写该方法。

### 三、为什么要重写equals要重写hashcode方法

散列集合插入对象时会进行判断，即HashMap的putVal方法，先调用hashcode，如果相同，再调用equals，如果都相同则只插入一个。

如果只重写了 equals 方法，那么默认情况下,**散列集合**(HashMap、LinkedHashMap、HashSet、 LinkedHashSet )进行去重操作时，会**先判断**两个对象的 **hashCode** 是否相同，此时因为没有重写 hashCode 方法，**所以会直接执行** Object 中的 hashCode 方法，而 Object 中的 hashCode 方法对比的是两个对象的引用地址，显然是不同的。所以结果是 false，那么 **equals 方法就不执行了**，直接返回的结果就是 false，所以两个对象不是相等的，于是就在散列集合中插入了两个相同的对象。（这显然是不对的）

但是，如果在重写 equals 方法时，也重写了 hashCode 方法，那么在执行判断时会去执行重写的 hashCode 方法，此时对比的是两个对象的所有属性的 hashCode 是否相同，于是调用 hashCode 返回的结果就是 true，再去调用 equals 方法，发现两个对象确实是相等的，于是就返回 true 了，因此散列集合就不会存储两个一模一样的数据了，于是整个程序的执行就正常了。

### 总结

hashCode 和 equals 两个方法是用来协同判断两个对象是否相等的，采用这种方式的原因是可以提高程序插入和查询的速度，如果在重写 equals 时，不重写 hashCode，就会导致在某些场景下，比如将两个相等的自定义对象存储在**散列集合**时，就会出现程序执行的异常，为了保证程序的正常执行，所以我们就需要在重写 equals 时，也一并重写 hashCode 方法才行。