# Vue生命周期

![image-20230418204207952](https://s2.loli.net/2023/04/18/6rGzFUAvNxaCwnK.png)

1. new Vue() new 了一个 Vue 的实例对象，此时就会进入组件的创建过程。
2. Init Events & Lifecycle 初始化组件的事件和生命周期函数 
3. beforeCreate 组件创建之后遇到的第一个生命周期函数，这个阶段 data 和 methods 以及 dom 结构都未被初始化，也就是获取不到 data 的值，不能调用 methods 中的函数 
4. Init injections & reactivity 这个阶段中, 正在初始化 data 和 methods 中的方法 
5. created - 这个阶段组件的 data 和 methods 中的方法已初始化结束，可以访问，但是 dom 结构未初始化，页面未渲染 
6. 在内存中编译模板结构
7. beforeMount 当模板在内存中编译完成，此时内存中的模板结构还未渲染至页面上，看不到真实的数据
8. Create vm.$el and replace ‘el’ with it 这一步，再把内存中渲染好的模板结构替换至真实的 dom 结构也就是页面上
9. mounted 此时，页面渲染好，用户看到的是真实的页面数据， 生命周期创建阶段完毕，进入到了运 行中的阶段

在本文中我们探讨vue对象里面的2个参数及其对应的dom界面和dom数据可以被使用的时间

- data
- methods
- 用户界面dom
- dom数据

也就是上面4个东西在8大生命周期函数阶段是否可以被使用

在Vue对象中有8大生命周期函数

- beforeCreate 
- created 
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed

本文中重要探讨前6个，最后两个实际中也不常用，以下是案例代码及其运行的页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>

<body>
<div id="app">
    <span id="num">{{num}}</span>
    <button @click="num++">赞！</button>
    <h2>{{name}}，有{{num}}次点赞</h2>
</div>
<script src="vue.js"></script>
<script>
    let vm = new Vue({
        el: "#app",
        data: {//数据池
            name: "Tom",
            num: 0
        },
        methods: {
            show() {
                return this.name;
            },
            add() {
                this.num++;
            }
        },
        beforeCreate() {		//生命周期函数-创建vue实例前
            console.log("=============beforeCreate==========");
            console.log("数据模型/数据池的数据是否加载/使用?[no]", this.name, " ", this.num);
            //console.log("自定义方法是否加载/使用?[no]", this.show());
            console.log("用户页面dom是否加载/使用?[yes]", document.getElementById("num"));
            console.log("用户页面dom是否被渲染?[no]", document.getElementById("num").innerText);
        },
        created() {				//生命周期函数-创建vue实例
            console.log("=============created==========");
            console.log("数据模型/数据池的数据是否加载/使用?[yes]", this.name, " ", this.num);
            console.log("自定义方法是否加载/使用?[yes]", this.show());
            console.log("用户页面dom是否加载/使用?[yes]", document.getElementById("num"));
            console.log("用户页面dom是否被渲染?[no]", document.getElementById("num").innerText);
            //可以发出Ajax
            //接收返回的数据
            //再次去更新data数据池的数据
            //编译内存模板结构
            //.....

        },
        beforeMount() {			//生命周期函数-挂载前
            console.log("=============beforeMount==========");
            console.log("数据模型/数据池的数据是否加载/使用?[yes]", this.name, " ", this.num);
            console.log("自定义方法是否加载/使用?[yes]", this.show());
            console.log("用户页面dom是否加载/使用?[yes]", document.getElementById("num"));
            console.log("用户页面dom是否被渲染?[no]", document.getElementById("num").innerText);

        },
        mounted() {				//生命周期函数-挂载后
            console.log("=============mounted==========");
            console.log("数据模型/数据池的数据是否加载/使用?[yes]", this.name, " ", this.num);
            console.log("自定义方法是否加载/使用?[yes]", this.show());
            console.log("用户页面dom是否加载/使用?[yes]", document.getElementById("num"));
            console.log("用户页面dom是否被渲染?[yes]", document.getElementById("num").innerText);

        },
        beforeUpdate() {		//生命周期函数-数据池数据更新前
            console.log("=============beforeUpdate==========");
            console.log("数据模型/数据池的数据是否加载/使用?[yes]", this.name, " ", this.num);
            console.log("自定义方法是否加载/使用?[yes]", this.show());
            console.log("用户页面dom是否加载/使用?[yes]", document.getElementById("num"));
            console.log("用户页面dom数据是否被更新?[no]", document.getElementById("num").innerText);
            //验证数据==>修正
            // if(this.num < 10 ) {
            //     this.num = 8;
            // }
        },
        updated() {				//生命周期函数-数据池数据更新后
            console.log("=============updated==========");
            console.log("数据模型/数据池的数据是否加载/使用?[yes]", this.name, " ", this.num);
            console.log("自定义方法是否加载/使用?[yes]", this.show());
            console.log("用户页面dom是否加载/使用?[yes]", document.getElementById("num"));
            console.log("用户页面dom数据是否被更新?[yes]", document.getElementById("num").innerText);
        }

    })
</script>
</body>
</html>
```

这是没有点赞时的抓包

![image-20230418211019898](https://s2.loli.net/2023/04/18/JyBvfTq1ZGmWNY3.png)

点赞后的抓包

![image-20230418211022530](https://s2.loli.net/2023/04/18/w216AFNZBfHJRjE.png)

### 总结：

- beforeCreate 时，
  - data是否可以使用？					  no
    自定义方法是否可以使用？		   no
    用户页面dom是否可以使用？		**yes**
    用户页面dom的data是否被渲染？ no
- created 
  - data是否可以使用？					  yes
    自定义方法是否可以使用？		   yes
    用户页面dom是否可以使用？		yes
    用户页面dom的data是否被渲染？ **no**			//因为还没有渲染data，是倒数第二次更改数据的机会
- beforeMount
  - data是否可以使用？					  yes
    自定义方法是否可以使用？		   yes
    用户页面dom是否可以使用？		yes
    用户页面dom的data是否被渲染？ **no**			//倒数第一次更改数据机会
- mounted
  - data是否可以使用？					  yes
    自定义方法是否可以使用？		   yes
    用户页面dom是否可以使用？		yes
    用户页面dom的data是否被渲染？ yes
- beforeUpdate
  - data是否可以使用？					  yes
    自定义方法是否可以使用？		   yes
    用户页面dom是否可以使用？		yes
    用户页面dom数据是否被更新？     **no**
- updated
  - data是否可以使用？					  yes
    自定义方法是否可以使用？		   yes
    用户页面dom是否可以使用？		yes
    用户页面dom数据是否被更新？    yes

### 那么我们知道了生命周期有什么用呢？

beforecreate : 可以在这加个loading事件，在加载实例时触发

created : 初始化完成时的事件写在这里，如在这结束loading事件，Ajax异步请求也适宜在这里调用

mounted : 挂载元素，获取到DOM节点

updated : 如果对数据统一处理，在这里写上相应函数

beforeDestroy : 可以做一个确认停止事件的确认框

