# ThreadLocal

什么是 ThreadLocal 

1. ThreadLocal 的作用：实现**在同一个线程数据共享**, 从而解决多线程数据安全问题. 
2. ThreadLocal 可以给当前线程关联一个数据(普通变量、对象、数组)set 方法 
3.  ThreadLocal 可以像 Map 一样存取数据，key 为当前线程, get 方法 
4. 每一个 ThreadLocal 对象，**只能**为当前线程关联**一个**数据，如果要为当前线程关联多个数据，就需要使用多个 ThreadLocal 对象实例
5.  每个 ThreadLocal 对象实例定义的时候，一般为 public static 类型，这样无需getter方法直接使用类名.ThreadLocal即可
6. ThreadLocal 中保存数据，在线程销毁后，会自动释放

### 源码解读：

![image-20230411212042581](https://s2.loli.net/2023/04/11/O1vdDtjq249HKy6.png)

```java
get方法源码:
         public T get() {
                  //1. 先得到当前的线程对象
                 Thread t = Thread.currentThread();
                 //2.通过当前线程获取到对应的ThrealLocalMap,map里放着许多个Entry
                 ThreadLocalMap map = getMap(t);
                 if (map != null) {
                      //3. 如果map不为空, 根据当前的 threadlocal对象，得到对应的Entry,底层就是通过哈希值获取下标i,在map中找到下标为i的Entry表,再取出Entry表的k值,如果与当前对象相等就返回,不相等返回null
                     ThreadLocalMap.Entry e = map.getEntry(this);
                     //4. 如果e 不为null,说明有这个对象
                     if (e != null) {
                         @SuppressWarnings("unchecked")
                         //entry对象可以直接.value获取v值,返回当前k:threadlocal关联的数据value
                         T result = (T)e.value;
                         return result;
                     }
                 }
                 return setInitialValue();
             }
```

```java
set方法源码:
老韩解读
public void set(T value) {
    //1. 获取当前线程, 关联到当前线程!
    Thread t = Thread.currentThread();
    //2. 通过线程对象, 获取到ThreadLocalMap
    //   ThreadLocalMap 类型 ThreadLocal.ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    //3. 如果map不为null, 将数据(dog..对象) 放入map 
    //  key:threadLocal     value:数据对象
    //从map.set(this, value);这个源码我们已然看出一个threadlocal只能关联一个数据
    //如果你set, this就会替换
    //4. 如果map为null, 就创建一个和当前线程关联的ThreadLocalMap, 并且该数据放入
    if (map != null)
    map.set(this, value);
    else
    createMap(t, value);
}
```

