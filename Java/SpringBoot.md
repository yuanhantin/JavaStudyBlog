# Springboot

注意，所有需要被扫描的包都需要放在main类的子包下，如果没有这样做，则需要在main类添加注解自行配置扫描路径。

静态资源则放在resource目录的static目录下



## Spring 注入组件的注解

### Spring传统注解

- @Component

- @Controller

- @Service

- @Repository

   这些在Spring 中的传统注解仍然有效，通过这些注解可以给容器注入组件

### @RestController

可以直接使用  @RestController 替换掉 @Controller + @ResponseBody 

需要注意的是

1. 如果使用@Controller 注解，在对应的方法上，视图解析器可以解析return 的jsp,html页面，并且跳转到相应页面；若返回json等内容到页面，则需要加@ResponseBody注解。
2. 如果使用@RestController注解，相当于@Controller+@ResponseBody两个注解的结合，返回json数据不需要在方法前面加@ResponseBody注解了，但使用@RestController这个注解，就不能返回jsp,html页面，视图解析器无法解析jsp,html页面。

### @Configuration 和  @Bean  

[SpringBoot - @Configuration、@Bean注解的使用详解（配置类的实现） (hangge.com)](https://www.hangge.com/blog/cache/detail_2506.html)

[Spring注解之@Configuration和@Bean使用详解 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/old/1997806)

### @Import

### @Conditional

### @ImportResource



## 接收参数相关注解

### @PathVariable 

### @RequestHeader 

### @ModelAttribute 

### @RequestParam 

### @MatrixVariable

### @CookieValue



### @ResponseBody

#### 基本介绍

- @ResponseBody的作用其实是将Java对象转为json格式的数据。

- @responseBody注解的作用是**将controller的方法返回的对象**通过适当的转换器**转换为指定的格式**之后，写入到**response对象**的body区，通常用来返回JSON数据或者是XML数据。
  注意：在使用此注解之后不会再走视图处理器，而是直接将数据写入到输入流中，他的效果等同于通过response对象输出指定格式的数据。
- @ResponseBody是作用在方法上的，@ResponseBody 表示该方法的返回结果直接写入 HTTP response body 中，一般在异步获取数据时使用【也就是AJAX】。
  注意：在使用 @RequestMapping后，返回值通常解析为跳转路径，但是加上 @ResponseBody 后返回结果不会被解析为跳转路径，而是直接写入 HTTP response body 中。 比如异步获取 json 数据，加上 @ResponseBody 后，会直接返回 json 数据。@RequestBody 将 HTTP 请求正文插入方法中，使用适合的 HttpMessageConverter 将请求体写入某个对象。

#### 在Controller注解下未使用ResponseBody会发生的404BUG

- 使用@Controller注解，如果你的方法上**没有**使用 @ResponseBody注解，会导致spring mvc框架认为你这个方法的返回值就是 ModelAndViewer对象，相当于是一个 待跳转的页面，导致跳转的时候找不到这个 viewer ，导致404报错。
- **因此，如果你要在一个Controller下返回一个Java对象的话，记得加上ResponseBody注解。**





## 校验注解

这里主要介绍3个以及配套注解@Valid，剩下的放在后面，需要的对照的自己搜索使用。

- @NotNull：不能为null，但可以为empty
- @NotEmpty：不能为null，而且长度必须大于0
- @NotBlank：只能作用在String上，不能为null，而且调用trim()后，长度必须大于0

这些注解用于加在Java的实体类属性字段上，在进行请求或请求的时候会根据字段的注解进行对应的校验，简化校验相关的代码。比如说这样：

```java
@Data
public class userForm {
    @NotBlank(message = "username校验不通过")
    private String username;
    @NotBlank
    private String password;
    @NotBlank
    private String email;
}
```

当校验不通过时，会返回对应的message描述，如果不加的话，默认就是校验不通过。

注意在使用这些注解时，在controller层一定要加上@Valid一起使用，不然上面这些注解不起作用

除此之外，我们还可以使用一个叫BindingResult 的API，用于对前端传进来的参数进行校验，省去了大量的逻辑判断操作，否则的话我们需要对上面的实体类属性全部都写一个if语句进行判断。使用了该API的话只需要一个 if (bindingResult.hasErrors()) 即可。

需要注意的是@Valid 和 BindingResult 是 一 一 对应的，如果有多个@Valid，那么每个@Valid后面都需要添加BindingResult用于接收bean中的校验信息。

比如说这样：

```java
@RestController
@RequestMapping("/user")
public class userController {
    @PostMapping("/register")
    
    
    //主要看这几行
    public ResponseVo Register(@Valid @RequestBody userForm userForm, BindingResult bindingResult){
        if (bindingResult.hasErrors()){
            //输出出错的字段的message信息，也就是上面填写的username校验不通过
            //当然也可以不使用sout，其实使用日志输出更方便
            System.out.println(bindingResult.getFieldError().getDefaultMessage());
        }
        
        
        return ResponseVo.error(ResponseEnum.NEED_LOGIN);
    }
}
```

#### 常用校验注解

```
常用校验注解

@Null 只能是null
@NotNull 不能为null 注意用在基本类型上无效，基本类型有默认初始值
@AssertFalse 必须为false
@AssertTrue 必须是true

字符串/数组/集合检查：(字符串本身就是个数组)
@Pattern(regexp="reg") 验证字符串满足正则
@Size(max, min) 验证字符串、数组、集合长度范围
@NotEmpty 验证字符串不为空或者null
@NotBlank 验证字符串不为null或者trim()后不为空

数值检查：同时能验证一个字符串是否是满足限制的数字的字符串
@Max 规定值得上限int
@Min 规定值得下限
@DecimalMax("10.8") 以传入字符串构建一个BigDecimal，规定值要小于这个值 
@DecimalMin 可以用来限制浮点数大小
@Digits(int1, int2) 限制一个小数，整数精度小于int1；小数部分精度小于int2
@Digits 无参数，验证字符串是否合法
@Range(min=long1,max=long2) 检查数字是否在范围之间
这些都包括边界值

日期检查：Date/Calendar
@Post 限定一个日期，日期必须是过去的日期
@Future 限定一个日期，日期必须是未来的日期

其他验证：
@Vaild 递归验证，用于对象、数组和集合，会对对象的元素、数组的元素进行一一校验
@Email 用于验证一个字符串是否是一个合法的右键地址，空字符串或null算验证通过
@URL(protocol=,host=,port=,regexp=,flags=) 用于校验一个字符串是否是合法UR

原文链接：https://blog.csdn.net/lihua5419/article/details/83418043
```



此外，如果我们前端需要的异常返回的格式与默认的不同的话，因此我们需要对异常的格式进行处理一下，这里我们使用

#### 全局异常的格式处理

我们使用@ControllerAdvice+@ExceptionHandler进行全局异常处理

```java
@ControllerAdvice
public class RuntimeExceptionHandler {
    @ExceptionHandler(RuntimeException.class)	//填写你要拦截处理的异常
    @ResponseBody							 //这里返回的json格式所以需要加一个ResponseBody注解
    public ResponseVo handler(RuntimeException e){
        //这里就可以进行处理
        return ResponseVo.error(ResponseEnum.ERROR, e.getMessage());
    }
}
```
