##  SpirngBoot的三种前端传参方式与对应的后端三种接收参数方式

- 前端请求传**Json对象**，则后端使用@RequestParam
- 前端请求传**Json对象的字符串**，则后端使用@RequestBody
- 前端请求传**未转成Json对象的字符串**，则后端使用@RequestParam



我们平常在项目中使用PostMan进行测试的时候，有个小坑会导致后端接收不到参数。

@RequsetBody

一般处理 **不是**Content-Type: application/x-www-form-urlencoded编码格式的数据，而是Json之类的

@RequestParam

可以处理**是**Content-Type: application/x-www-form-urlencoded编码格式的数据

用图说话就是：

当使用x-www-form-urlencoded的时候，后端采用@RequestParam注解接收

![image-20230603224017642](https://s2.loli.net/2023/06/03/AUTtLojYp6VS4wu.png)

![image-20230603224600702](https://s2.loli.net/2023/06/03/Ca43RnTEQ85swf2.png)

当使用Json格式传参的时候，后端采用@RequestBody注解接收

![image-20230603224617218](https://s2.loli.net/2023/06/03/QeO71TUBZAzLcDG.png)

![image-20230603224424477](https://s2.loli.net/2023/06/03/ZEJ7qfoIcVybjue.png)