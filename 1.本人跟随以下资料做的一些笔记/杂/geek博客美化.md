#本文参考至[geek语雀](https://www.yuque.com/awescnb/user/rycpvv#r8Fbr  "geek")
###傻瓜式教学第一步
打开博客园点击右上角 我的博客-设置-开通js权限！！！
![](https://img2023.cnblogs.com/blog/3043157/202302/3043157-20230209002959606-175067239.png)

本处已经开启,未申请小伙伴点击申请,原因填美化博客就好，大概率10分钟之内通过审核，趁这10分钟里我们配置一下下面的操作。

### 第二步
将以下代码复制至页面定制css代码
```
:root{--sk-size:60px;--sk-color:#ffb3cc}
#loading{position:fixed;top:0;left:0;right:0;height:100vh;display:flex;justify-content:center;align-items:center;background-image:url(https://guangzan.gitee.io/awescnb/assets/images/background/cell.gif);z-index:99999}
.sk-fold{width:var(--sk-size);height:var(--sk-size);position:relative;transform:rotateZ(45deg)}
.sk-fold-cube{float:left;width:50%;height:50%;position:relative;transform:scale(1.1)}
.sk-fold-cube:before{content:"";position:absolute;top:0;left:0;width:100%;height:100%;background-color:var(--sk-color);animation:sk-fold 2.4s infinite linear both;transform-origin:100% 100%}
.sk-fold-cube:nth-child(2){transform:scale(1.1) rotateZ(90deg)}
.sk-fold-cube:nth-child(4){transform:scale(1.1) rotateZ(180deg)}
.sk-fold-cube:nth-child(3){transform:scale(1.1) rotateZ(270deg)}
.sk-fold-cube:nth-child(2):before{animation-delay:.3s}
.sk-fold-cube:nth-child(4):before{animation-delay:.6s}
.sk-fold-cube:nth-child(3):before{animation-delay:.9s}
@keyframes sk-fold{
0%,10%{transform:perspective(140px) rotateX(-180deg);opacity:0}
25%,75%{transform:perspective(140px) rotateX(0);opacity:1}
100%,90%{transform:perspective(140px) rotateY(180deg);opacity:0}}
```
###<font color=Hotpink>注意禁用默认模板css</font>
![](https://img2023.cnblogs.com/blog/3043157/202302/3043157-20230209002932267-310988576.png)

###第三步
将以下代码复制至页首定制html代码
```
<section id="loading">
  <div class="sk-fold">
    <div class="sk-fold-cube"></div>
    <div class="sk-fold-cube"></div>
    <div class="sk-fold-cube"></div>
    <div class="sk-fold-cube"></div>
  </div>
</section>
```
###第四步
将以下代码复制至页脚定制html代码
```
<script src="https://guangzan.gitee.io/awescnb/index.js"></script>
<script>
  const opts = {
    theme:{
      name:'geek',					///此处是主题可以访问此网站查看所有皮肤主题 https://www.yuque.com/awescnb/user/kyi19z
      avatar:'https://pic.cnblogs.com/avatar/3043157/20221122212334.png',	///avatar为头像，头像链接如何获取请看下文
      headerBackground: 'https://img.zcool.cn/community/01f79e5d6cd2cea80120526dccb751.jpg@3000w_1l_2o_100sh.jpg'
    },
    signature: {
    enable: true,
    contents: [
    "<b>欢迎来到翰林猿的博客</b>",		///中间可以修改为自己的签名，<b>是字体加粗的意思
    "<b>Hello world!</b>",
    ],
}
  }
   $.awesCnb(opts)
</script>
```
###如何获取头像或者背景图片链接如下
1.头像可以从博客园网站右上角-我的园子-将鼠标放在图片上，点击复制图像链接，将其替换第四步的avatar的代码
![](https://img2023.cnblogs.com/blog/3043157/202302/3043157-20230209005633381-196271003.png)
![](https://img2023.cnblogs.com/blog/3043157/202302/3043157-20230209005733641-1495184808.png)
2.背景图片同理，在网上选择一个图片复制图像链接替换第四步的headerBackground的代码
	如果是想用本地图片，需要将图片上传至图床或者百度，qq空间之类的地方再进行上一步。
这里推荐一个免费图床，缺点是图片加载较慢	[路过图床](https://imgse.com/ "路过图床")


###最终效果博客[翰林猿的博客](https://www.cnblogs.com/hanlinyuan "翰林猿的博客")