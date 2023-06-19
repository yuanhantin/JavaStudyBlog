# Git使用教程（理论实体结合体系版）

## 下载安装：

按照这个博客来就好

[Windows系统Git安装教程（详解Git安装过程） - 学为所用 - 博客园 (cnblogs.com)](https://www.cnblogs.com/xueweisuoyong/p/11914045.html)

## Git命令大全：

[Git 大全 - Gitee.com](https://gitee.com/all-about-git#仓库)

## 最小配置：

在桌面右键点击Git Bash Here进入命令行，GUI我们不常用。

![image-20230609132310642](https://s2.loli.net/2023/06/09/unmpsEJbG4A9zFH.png)

首先要设置你的用户名称和 e-mail 地址, 因为每次 Git 提交都会使用该信息

开始配置你的用户名和邮箱，单引号中间随便填你的信息即可，最后用 git config --system --list 查看当前用户配置，该指令显示的就是 C:\Users\windows\.gitconfig 内容（后面再来说这是什么文件）

![image-20230609132221802](https://s2.loli.net/2023/06/09/7LRXm48ryDOt1ZB.png)

‪指令成功执行后，在C:\Users\windows\.gitconfig会生成一个文件 ，打开之后就是你刚刚配置的玩意

![image-20230609134428438](https://s2.loli.net/2023/06/09/PiHx8jN1tIKvWBF.png)

OK配置完基本信息，来讲一下理论知识，然后再开始教大家具体的操作。

## Git 工作原理 [ 重要 ] ：

### 四大工作区域：

Git 本地有三个工作区域：

- 工作区（Working Directory）
- 暂存区(Stage/Index)
- 仓库区 (Repository)
- 再加上远程的 git 仓库(Remote Directory)  就可以分为四个工作区域

具体说明一下：

- 工作区：就是你平时存放代码的地方 
- 暂存区：用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
- 仓库区（本地仓库）：就是安全存放数据的位置，这里面有你提交到所有版本的数据
- 远程仓库：就是托管代码的服务器(比如 Github/Gitee)，也可以简单的认为是一台用于远程数据交换的电脑

### 一图胜千言：4个区域之间要使用到的指令

![image-20230609144725222](https://s2.loli.net/2023/06/09/23JRrlap6SwujXF.png)

需要注意的是，clone的作用类似pull，但是还是有点区别的，具体文章看这里就好。（我们一般从GitHub上获取别人的代码用的都是clone）

[Git教程 git pull 和 git clone的区别 - 腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/old/2108853)

## 实践：

### 建 Git 仓库

- 在已经有项⽬代码的文件夹里建Git仓库
  - 在已有的项目代码文件夹处右键，点击Git Bash Here进入命令行输入 git init ，然后就会出现一个隐藏的文件夹  .git
  - ![image-20230609145425742](https://s2.loli.net/2023/06/09/4wMJHnGaXTLZf5m.png)
- 在不存在的代码的文件夹中建Git仓库
  - 前面操作相同，指令改成输入 git init [自定义项目名] 即可，然后在idea新建项目时使用这个文件夹就好了。
  - ![image-20230609145754843](https://s2.loli.net/2023/06/09/O6k3AdE7i4FoHbU.png)

###  克隆远程仓库

就是将远程服务器上的仓库完全镜像一份至本地，这里我们拿JavaGuide的作为示范，如果我们想要获取到他的开源文件，我们只需要在你想要保存clone下来的文件的地方右键点击Git Bash Here进入命令行输入 git clone [URL]

![image-20230609150430098](https://s2.loli.net/2023/06/09/vkBwEif9c6dFICP.png)

随便找个文件夹，我们输入git clone 加上复制下来他的URL，等进度条到100%即可（如果没有魔法的话可能会很慢甚至失败）再细我就不好明说了，如果实在不知道魔法是什么的小白，具体可以找置顶文章联系我。

![image-20230609150818903](https://s2.loli.net/2023/06/09/OfcQIlS8yC4DxgL.png)

### 文件操作指令

**先来介绍文件的4大状态：**

- Untracked: 未跟踪，在文件夹中,，但并没有加入到 git 库,，不参与版本控制，需要通过 git add 状态变为 Staged
- Unmodify: 未修改，文件已经入库，而且与工作区的内容相同。
  1. 如果想要把他移出版本库，可以使用 git rm 移出去不再同步
  2. 如果被修改了，就会变成 Modified 已修改状态
- Modified: 已修改，这种文件有两个去处
  1. 可以使用 git add 将其加入暂存区
  2. 也可以使用 git checkout 丢弃修改过的内容，返回到刚刚 未修改 的状态（即从库中取出文件，覆盖当前已修改的文件）
- Staged: 暂存状态，同样，也有两个去处
  1. 执行 git commit 则将其同步到库中，此时版本库与工作区一致，变成未修改状态
  2. 执行 git reset HEAD filename 取消暂存, 文件状态变为已修改状态

**实践一下吧**

找到刚刚建好git仓库的文件夹，还是打开git bash here 输入git status [文件名] ，可以先写个几个字母然后按tab键一键补全文件名。比如说这里就是先写个 s 然后按 tab 就会自动出来 src/ 了

![image-20230609153548884](https://s2.loli.net/2023/06/09/UugDGh9dqFzkvKA.png)

用 git  add 加入暂存区

![image-20230609154021336](https://s2.loli.net/2023/06/09/Gr5aFIhtX4Zo6gR.png)

然后就可以开始提交至本地仓库了，使用 git commit -m "自定义一些需要携带的信息"  

![image-20230609154453914](https://s2.loli.net/2023/06/09/Z5WdMp6UNHo2rk9.png)

可以看到如果我们再用 git status 来查看状态的话，会提示没有什么需要提交的了。

### 忽略文件

实际需求：我们不想把某些文件纳入版本控制中， 如何处理?

这块内容都是一些配置，可以直接看这篇文章就好了

[.gitignore文件语法和常见写法（就看这篇就行了）_gitinore 以 * 结尾_石头wang的博客-CSDN博客](https://blog.csdn.net/w8y56f/article/details/103263924)

具体这个文件在哪呢，一般在idea文件夹下

![image-20230609155337629](https://s2.loli.net/2023/06/09/RGn325JMTFjdkwL.png)

![image-20230609155342682](https://s2.loli.net/2023/06/09/dAsMWZ91IRFYELB.png)

打开之后可以打开上面的网址对照的文件学一下每一行是什么意思，这里我就不再细说了。

![image-20230609155408230](https://s2.loli.net/2023/06/09/AdeXQz7aLSYGvg2.png)

### 创建自己的远程仓库

这里我们使用GitHub，如果没有魔法也可以使用Gitee，但是Gitee还要实名认证什么的也挺麻烦。实际两个都差不多操作，这里就用GitHub进行讲解。

首先是打开GitHub官网注册一个自己的账号好吧，这里就是正常注册账号的流程，自己跟着官方指导鼓捣鼓捣就好了。当然，这里为了照顾一些看到英文就头大的同学，还是贴个教程给你们吧。

[注册Github账号详细教程【超详细篇 适合新手入门】-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1173409)

之后就可以在右上角开始创建我们自己的仓库了，这里借用官方文档的配图

![包含创建新存储库选项的下拉菜单](https://s2.loli.net/2023/06/09/Bn3UEj4vO5gJlP1.png)

为仓库写一个简短好记的名称

![创建 GitHub 存储库的第一步的屏幕截图。 “存储库名称”字段包含文本“hello-world”，并用深橙色框出。](https://s2.loli.net/2023/06/09/LclkD9yjEWwrz2T.png)

设置为开源还是私有

![用于选择存储库可见性的单选按钮](https://s2.loli.net/2023/06/09/emM2fKn9WFLr5Zt.png)

勾选自动创建readme文件，.gitignore我们一般使用的是Java就用Java类型即可

![image-20230609170435390](https://s2.loli.net/2023/06/09/gET2qUjcVhfI5Ru.png)

那么到这里，一个基本的仓库就建好了。

### 上传文件至远程仓库

这里我就顺便把我目前做的所有笔记上传至GitHub玩一玩好了

首先是把刚刚新建好的仓库clone到本地某个文件夹先

![image-20230609174058129](https://s2.loli.net/2023/06/09/Q5yqWKtj8GI4TrX.png)

把当前的文件夹和远程仓库绑定  git remote add origin [远程仓库URL地址]

![image-20230609174210625](https://s2.loli.net/2023/06/09/rG3d7eWYfzgoLkR.png)

把咱们的笔记放进去之后，输入 git add .  将笔记加入到暂存区

![image-20230609174216112](https://s2.loli.net/2023/06/09/wp2brmPfV4QY68T.png)

检查一下我们的仓库分支是什么，这个是一开始新建仓库时可以指定的，比如main或者master，刚刚在GitHub建仓库的时候默认是main了

![image-20230609174222161](https://s2.loli.net/2023/06/09/8osZQ41SKdGDei6.png)

调用push指令将其更新到远程仓库，指令是 git push -u origin [仓库分支]

![image-20230609174228339](https://s2.loli.net/2023/06/09/34vTSw9dbgFKita.png)

大功告成哈哈哈 ，成品地址 [yuanhantin/JavaStudyBlog (github.com)](https://github.com/yuanhantin/JavaStudyBlog)

![image-20230609174854145](https://s2.loli.net/2023/06/09/zs1XJ5EkFfW8j4n.png)

### 总结：

```
第一次创建仓库时
1.clone到本地
git clone [远程仓库URL地址]

2.把当前的文件夹和远程仓库绑定
git remote add origin [远程仓库URL地址]

3.更改一系列文件后
git add .  将所有更改后的文件加入到暂存区

4.调用push指令将其更新到远程仓库
git push origin [仓库分支]

以后再需要更新修改文件只要使用后2个指令即可

有时候会显示这样一句话导致push失败，原因是本地仓库相对于远程仓库不是最新，先pull更新本地，再把自己的push上去
Updates were rejected because the remote contains work that you do
这个时候有两个解决方法，一是强推，二就是pull一下再更新
如果确保本地没问题的话，可直接用强推 git push -f origin [仓库分支]即可
```

最后，实际情况可能还会有很多原因导致你不能将本地代码推送到仓库中，不过git这方面也没啥太多bug，只需要把报错复制到谷歌上搜一下就有结果了，这里不再赘述太多。

## 分支与合并:

实际开发过程中，有可能一个项目会有多个版本的需求，比如说我们QQ有什么国际版，HD版，精简版之类的。所以这里就需要分支的概念了，分支出来的版本有可能是与主版本平行的永远不会相交，也有可能某一天需要将他与主版本合并。画一张图大概长这样：

![image-20230612133925283](C:\Users\windows\AppData\Roaming\Typora\typora-user-images\image-20230612133925283.png)

当然，合并不一定是跟主分支，也可以跟其他分支合并。

列一下指令先，简单看看就好，等会我们再示范一下怎么使用

```apl
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个本地分支，但依然停留在当前分支
$ git branch [分支名]

# 切换到该分支
$ git checkout [分支名]

#新建一个远程分支,将冒号左右两边的分支绑定，并将本地分支提交至新建的远程分支
$ git push origin 本地分支名:新建的远程分支名
```

实战一下：

![image-20230612164630672](https://s2.loli.net/2023/06/12/mlparyh9HOTiF2d.png)

![image-20230612164903622](https://s2.loli.net/2023/06/12/WxuOUK7JDjT518a.png)

看看我们的仓库有没有变化，这里为什么我又使用了Gitee了，因为由于魔法出了点问题搞得GitHub提交老是超时，懒得弄了，换成了国内的Gitee，不过也不影响讲解，问题不大好吧。

master分支我在创建的时候忘记截图了，就是把readme文件内容改成了test1而已。

![image-20230612165459893](https://s2.loli.net/2023/06/12/pQtjS5qmFdZUNav.png)

![image-20230612170023518](https://s2.loli.net/2023/06/12/Nl9eKxRiZHyUw5a.png)

那么分支讲解完毕，我们来看看合并吧。

在 Git 中，进行合并是非常简单方便的。它只需要两个步骤：

- （1） 切换到那个需要接收改动的分支上。

- （2） 执行 “git merge” 命令，并且在后面加上那个将要合并进来的分支的名称。

  

  来让我们把 “master2” 分支的改动合并到 “master” 中去：

```
$ git checkout master
$ git merge master2
```

当然，为了以示区别，我们新建一个Merge文件，并将它同步到mater2分支上去，过程我就不演示了，上面已经使用过很多次了。

![image-20230612171911237](https://s2.loli.net/2023/06/12/TcB3uXQSEbKtIjP.png)

![image-20230612172010770](https://s2.loli.net/2023/06/12/NEPGzbpRrOwXTf4.png)

然后我们开始执行合并分支操作，诶，出问题了？？？合并冲突，想一想为什么。

![image-20230612173522762](https://s2.loli.net/2023/06/12/y8LsFdIw3KRBjTO.png)

因为我们master的readme文件中第一行的内容是 test1，而master2中的文件内容为 master2 ，在合并的时候计算机不知道你要保留master还是master2的内容，所以引发了合并冲突。如何解决呢？就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

打开我们的Readme文件，可以看到Git自动加些东西进去，差不多就长这样的一个提示，什么意思不重要，反正一下就让我们看到问题出在第一行了，我们修改一下就好。（不过正常的提示不长这样，可能因为是md文件格式出了点小问题，不过问题不大，等会我们讲讲正常的提示）

![image-20230612174158883](https://s2.loli.net/2023/06/12/2xHEZKyofi4DVjn.png)

那么我们把master分支的文件中的 test1修改成 和master2分支的文件中一样的 master2 ，解决好冲突之后，我们再进行一次 add、commit、merge操作，发现这次不再报错了，然后我们push到远程仓库看看效果。

![image-20230612175833101](https://s2.loli.net/2023/06/12/oEJVZtSI2a7fRyC.png)

可以看到将 master2 合并到 master 过后确实多了一个Merge.txt文件了

![image-20230612175906597](https://s2.loli.net/2023/06/12/olEcHtnadgNW97K.png)

再来讲讲刚刚挖的坑，所谓的正常提示是什么，有什么用？

### 正常的提示：

当 Git 无法执行自动合并时，因为更改在同一区域，它会用特殊字符来表示冲突的区域。这些字符序列是这样的：

- `<<<<<<<`
- `=======`
- `>>>>>>>`

`<<<<<<<`和`=======`之间的所有内容都是你的本地修改。 这些修改还没有在远程版本库中。`=======`和`>>>>>>>`之间的所有行都是来自远程版本库或另一个分支的修改。现在你需要研究这两个部分并做出决定。

下面的图片显示了一个文件的内容，表明自动合并没有发生，有一个冲突。冲突发生在我们在本地修改了文件，增加了一行`- Sleep`。但与此同时，其他人在同一区域添加了 `- Gym` 行，从而推送了一个修改。

因此，`- Sleep`行被标记为本地修改，`- Gym`行被标记为从远程仓库或其他分支传入的修改。

![“由于同一区域的变化而产生的合并冲突”](https://s2.loli.net/2023/06/12/35kT6r18pWuwHjQ.jpg)

所以这个时候就得去找你的同事扯皮了，到底是用你的sleep还是用他的gym，亦或是新建一行而不是在同一行添加。





