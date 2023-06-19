## Map遍历方法

1. 增强for循环使用方便，但性能较差，不适合处理超大量级的数据。

2. 迭代器的遍历速度要比增强for循环快很多，是增强for循环的2倍左右。

3. 使用entrySet遍历的速度要比keySet快很多，是keySet的1.5倍左右。

   ```java
   增强for循环，keySet迭代 -> 31 ms
   增强for循环，entrySet迭代 -> 20 ms
   迭代器，keySet迭代 -> 17 ms
   迭代器，entrySet迭代 -> 10.33 ms
   ```

### entrySet是什么

entryset是一个hashmap的映射，每一个元素都是一个hashmap里面元素的地址。entrySet()的返回值是一个Set集合，此集合的类型为Map.Entry。

### keySet()方法

返回值是Map中key值的集合

```java
public class Main {
    public static void main(String[] args) {
        HashMap<String,String> map = new HashMap<String,String>();
        map.put("123","123");
        map.put("321","123");
        map.put("132","123");

        for (Map.Entry<String,String> entry: map.entrySet()) {      //使用entrySet()遍历，增强for循环
            System.out.println("key=" + entry.getKey()+" "+"value="+entry.getValue());
        }

        Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();   //entryset 迭代器遍历  最快的方法
        while(iterator.hasNext()){
            Map.Entry<String, String> entry = iterator.next();
            System.out.println("key = "+entry.getKey()+"value = "+entry.getValue());
        }

    }

```

