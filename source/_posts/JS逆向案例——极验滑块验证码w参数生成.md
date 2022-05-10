---
title: JS逆向案例——极验滑块验证码w参数生成
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-21 15:13:39
password:
summary:
tags: [JS, 逆向, 验证码]
categories: [JS逆向]
---



>  免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



#### 前言

本文是机器过极验滑块验证码系列文章的第二篇，提交验证请求的w参数逆向分析。后边还会陆陆续续发文，讲解如何补环境，如何利用像素点RGB差值获取缺口位置以及通过机器学习获取缺口位置，最后会通过几个采用极验验证码的网站去完整的展示整个自动化过程。而极验滑块系列只是验证码系列的第一个系列，后边会罗列市面上常用的验证码，然后发文一一解决。

上一篇文章见：[JS逆向案例——极验滑块验证码底图还原](https://blog.heshipeng.com/JS%E9%80%86%E5%90%91%E6%A1%88%E4%BE%8B%E2%80%94%E2%80%94%E6%9E%81%E9%AA%8C%E6%BB%91%E5%9D%97%E9%AA%8C%E8%AF%81%E7%A0%81%E5%BA%95%E5%9B%BE%E8%BF%98%E5%8E%9F/)



#### 逆向分析过程

网址：aHR0cHM6Ly93d3cudGlhbnlhbmNoYS5jb20v

以天眼查的登录为例，提交滑块验证请求时，w参数跟值。



##### 抓包分析

在底图还原的那篇文章中就提到过一共有四个重要的包，其中前面三个包都没有用到w参数，只有在向服务器提交验证码验证的请求时才需要w参数，如下：

![image-20220421230146225](https://img.heshipeng.com/202204212301379.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们点进去ajax.php这个请求的方法调用栈：

![image-20220421230321601](https://img.heshipeng.com/202204212303683.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们可以看到gt, challenge以及w参数生成位置，如下图：

![image-20220421230550163](https://img.heshipeng.com/202204212305223.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 确定逆向目标

通过上边的抓包分析，可以确定的是w参数有2个变量r7z和H7z拼接而成，然后r7z是调用一个函数生成，这个函数接受一个参数q7z；H7z也是调用一个函数生成。这样逆向的目标也就确定了，就是抠出这2个函数以及参数q7z。



##### H7z生成规则

H7z是由一个无参的函数V7z[M9r.C8z(92)]生成，所以先看这个函数。可以看到每次调用的结果都不同。

![image-20220421231717931](https://img.heshipeng.com/202204212317007.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



可以看到这个函数采用了流程平坦化，**对于流程平坦化，在调试时看下return语句返回的值，如果有多个return语句，则每个return语句都打上断点，如果只有一个return语句，看下return的那个变量，每个给这个变量赋值的语句都要打上断点**，显然这里函数返回的是Y0B，则给这个变量赋值的几个地方都断上：

![image-20220421231912531](https://img.heshipeng.com/202204212319591.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们断进来发现，只走了M9r.k9r()\[18\]\[36\]\[24\]这个分支，实例化一个v0B对象，然后调用这个对象的某一个方法，返回一串加密后的字符串。

![image-20220421232420203](https://img.heshipeng.com/202204212324263.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



利用console把`Y0B = new v0B()[M9r.C8z(699)](g0B[M9r.C8z(818)](D0B))`这行代码简化为`Y0B = new v0B().encrypt(g0B.wb())`，最后D0B为undefine所以省略。



所以这里要得到Y0B就先得抠出g0b的wb方法，然后抠出v0B的encrypt方法，我们一步步来吧。



1. g0b对象的wb方法

我们点进去wb方法，可以看到虽然是流程平坦化，但是只有一个case，最后返回的是一个逗号表达式，我们只看最后一个，也就是函数真正返回的是J0B变量，而J0B变量是调用C7B函数生成的。如下图：

![image-20220421233831380](https://img.heshipeng.com/202204212338521.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



没办法，我们只有接着追进去调试C7B函数，可以看到花里胡哨的其实只是调用了四次H1W函数，然后拼接成一个字符串返回。如图：

![image-20220421234115235](https://img.heshipeng.com/202204212341304.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们再扒一扒H1W函数的代码看下，将返回的那条语句反混淆之后结果为`(65536 * (1 + Math.random()) | 0).toString(16).substring(1)` 。

![image-20220421234837015](https://img.heshipeng.com/202204212348080.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



到这里都是调用内置的函数了，而且算法比较简单，所以不需要进一步往下挖了。整理一下：

```js
var G0b = function() {}
G0b.prototype.wb = function() {
    return H1W() + H1W() + H1W() + H1W();
}

function H1W() {
    return (65536 * (1 + Math.random()) | 0).toString(16).substring(1);
}

g0b = new G0b();
```



2. v0B对象的encrypt方法

接着看下v0B对象的encrypt方法。点进去看这个方法体，发现这个方法还依赖了很多其它的方法，而且依赖的这些方法都不是内置方法，如果单抠这样的一个个方法就很麻烦了，更不用想去一个个理清楚这些方法的逻辑，然后用Python去自己实现了。遇到这样的情况一般比较简单的方法是全抠整个JS，然后想办法导出所需要的方法即可。

![image-20220422002133796](https://img.heshipeng.com/202204220021962.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



想要导出V9B方法，就需要导出v0B对象，因为V9B方法属于v0B对象，而如何导出v0B对象呢？就要看这个对象的上一层是什么。如果遇到代码行数很多，代码层次比较多的时候推荐使用Notepad++方便去管理这种层次结构。下面介绍这个小技巧：

 

使用Notepad++查看JS代码层次结构小技巧：先拷贝整个文件至Notepad++，然后选择试图->折叠所有层次。

![image-20220422003418919](https://img.heshipeng.com/202204220034063.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



接着CTRL+F搜索我们要的代码，比如这里是v0B = function()

![image-20220422003646204](https://img.heshipeng.com/202204220036294.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



可以看到搜索结果只展开了包含我们搜索代码的那些分支，其余的依旧没有展开，这样非常方便我们观察代码的层次结构。

![image-20220422003746128](https://img.heshipeng.com/202204220037208.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



通过上图可以看到，只要加载这整个JS文件，就会执行流程平坦化中的代码，也就会得到v0B对象就会被定义，我们只需要全局定义一个变量接收v0B即可。如下图：

![image-20220422004359174](https://img.heshipeng.com/202204220043297.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



代码改好之后，我们放到console上试验一下，成功拿到encrypt方法。

![image-20220422004733474](https://img.heshipeng.com/202204220047547.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



既然wb方法和encrypt方法都拿到了，那我们也就算是H7z的值。我们同样试验一下，也没问题。

![image-20220422005005272](https://img.heshipeng.com/202204220050357.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



H7z的值拿到了，接下来就是r7z，而r7z依赖于q7z，所以我们先看q7z。



##### q7z生成规则

抠出q7z生成的代码：`q7z = n0B[M9r.R8z(699)](h7B[M9r.C8z(105)](Y7z), V7z[M9r.R8z(818)]())`，反混淆之后为：`q7z = n0B.encrypt(h7B.stringify(Y7z), V7z.wb())`

encrypt，stringify，wb 🤔这几个怎么看起来那么眼熟？



V7z.wb实际上跟我们前面抠的g0b.wb一毛一样，然后h7B.stringify实际上就等同于JSON.stringify

![image-20220422122917004](https://img.heshipeng.com/202204221229448.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



n0B.encrypt则显然与前面抠的v0B.encrypt不同，这两个方法参数个数不同，返回值类型也不同。



所以要解决q7z，则需要抠出Y7z是如何生成的以及抠出n0B.encrypt。看下我们前面抠v0B的方式，是不是可以如法炮制🤨

![image-20220422124107011](https://img.heshipeng.com/202204221241305.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



运行结果如下，可以看到成功拿到了n0B.encrypt，只不过n0B.encrypt和之前的v0B.encrypt使用不一样。

![image-20220422124505892](https://img.heshipeng.com/202204221245985.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



接下来就是Y7z了。我们先看下Y7z是个啥？🤓

![image-20220422125036518](https://img.heshipeng.com/202204221250611.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



随机滑动滑块几次，观察这几个参数哪些是固定的，哪些是变化的。目测除了版本号v之外，其余几个参数都不是固定的，心累😅

![image-20220422154448445](https://img.heshipeng.com/202204221544594.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



找到Y7z定义的地方，如下图，我们挨个看吧。

![image-20220422155807811](https://img.heshipeng.com/202204221558976.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



1. 先看看aa是如何生成的。

可以看到aa是由F7z变量赋值，我们在当前函数中找给F7z赋值的语句，并断上，如下图：

![image-20220422160401488](https://img.heshipeng.com/202204221604649.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



`F7z = e7B[M9r.C8z(779)](V7z[M9r.R8z(602)])`反混淆为`F7z = e7B.t(V7z.b)`，V7z.b为一个13位的时间戳。我们跟进去这个方法：

![image-20220422172757864](https://img.heshipeng.com/202204221727047.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们先看返回值：`f1z[M9r.R8z(592)](M9r.C8z(346)) + M9r.C8z(370) + B1z[M9r.R8z(592)](M9r.R8z(346)) + M9r.R8z(370) + o1z[M9r.R8z(592)](M9r.C8z(346))`反混淆为`f1z.join("") + "!!" B1z.join("") + "!!" +  o1z.join("")`，从这里可知，想要得到aa的值，就必须知道flz，Blz，olz的值。



我们再看看X1z：X1z是一个轨迹数组，每个元素都是一个包含三个向量的数组，分别是x坐标，y坐标，时间。经过分析知这个轨迹数组是浏览器监听鼠标事件得出来的，我们用机器去自动过验证码的时候是没办法通过这种方式得到的这个轨迹数组的，唯一的方式可能是写一种模拟拖动滑块的算法，生成这种轨迹，然后传给这个函数去计算aa的值。

![image-20220422173201445](https://img.heshipeng.com/202204221732544.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



分析到这里，很显然我们要魔改这个生成aa的函数，传入一个轨迹数组，返回aa的值。由于这个函数也很复杂，所以考虑直接抠出这个函数，而不必去纠结具体的flz，Blz，olz是怎么得到的。



说干就干。操作如下，解释说明下几个标红的地方。开头定义一个全局变量\_e7B用于导出生成aa的函数'\x74'，然后我们定位到'\x74'函数属于e7B对象，把这个对象赋值给\_e7B，然后把X1z变量作为参数提上来，把里面的这个变量删掉。

![image-20220422174419168](https://img.heshipeng.com/202204221744356.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



代码改好之后，我们测试一下，可以看到生成的aa与我们网站得到的一致。

![image-20220422175211230](https://img.heshipeng.com/202204221752482.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



虽然我们拿到了aa的值，但是跟生成F7z的里面那个值有出入，那就是说定义aa的地方虽然调用了e7B.t方法生成aa，但是外面有其它地方对这个值进行了修改。

![image-20220422182356210](https://img.heshipeng.com/202204221823454.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们在这个函数中搜索F7z，然后给所有包含F7z的赋值语句下断，一步步调试后发现修改F7z的地方如下：

![image-20220422185131813](https://img.heshipeng.com/202204221851258.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们抠出来这条语句，然后反混淆为：`e7B.u(F7z, V7z.d.c, V7z.d.s)`，我们看下V7z.d.c和V7z.d.s：

![image-20220422185546434](https://img.heshipeng.com/202204221855613.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



看着又有点似曾相识，emm，没错分别对应get.php返回的c和s：

![image-20220422185646048](https://img.heshipeng.com/202204221856167.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



然后我们得抠下e7B.u这个方法，意外的发现，其实这个方法包含的对象前面抠过来，既然对象已经抠了，这个方法自然就有了。

![image-20220422185947808](https://img.heshipeng.com/202204221859968.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



验证一下，结果没毛病：

![image-20220422190458150](https://img.heshipeng.com/202204221904380.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



2. 再看看userresponse如何生成的

抠出生成userresponse的代码，然后反混淆为：`i7B.C(g7z, V7z.d.challenge)`，g7z是拖拽鼠标滑动滑块的距离，可以通过轨迹数组计算出这个滑动的距离。

代码如下：

```js
for (let index = 0; index < X1z.length; index++) {
	passtime += X1z[index][2];
	g7z += X1z[index][0];
}
g7z -= X1z[0][0];
```



V7z.d.challenge是get.php返回的：

![image-20220422210127144](https://img.heshipeng.com/202204222101497.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

i7B.C这个函数的话，按照上边介绍的方法，先抠出i7B对象，自然就可以拿到C方法了。



3. passtime的计算

目测passtime是轨迹数组的每个向量的时间累积：

![image-20220422211356975](https://img.heshipeng.com/202204222113188.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



计算代码如下：

```js
let passtime = 0;
for (let index = 0; index < X1z.length; index++) {
    passtime += X1z[index][2];
}
```



4. imgload生成

imgload表示当前页面加载的图片数，这里我们用random随机一个值。



5. rp的计算

rp计算的代码为：`I0B(V7z[M9r.R8z(190)][M9r.R8z(189)] + V7z[M9r.C8z(190)][M9r.C8z(425)][M9r.R8z(504)](0, 32) + Y7z[M9r.C8z(193)])`反混淆之后为：`I0B(gt + challenge.slice(0, 32) +  passtime)`，gt，challenge，passtime都已经算出，抠出I0B方法即可，如何抠？参照上面的方式。



##### r7z生成规则

有了q7z，r7z自然就很简单了。因为前面说过`r7z = p7B.Ha(q7z)`，只需要抠出p7B即可，过程同理。



##### 编写代码

按照上面的抠出相应的方法后，然后编写生成w参数的代码，代码如下：

```js
var G0b = function() {}

function H1W() {
    return (65536 * (1 + Math.random()) | 0).toString(16).substring(1);
}

var wb = H1W() + H1W() + H1W() + H1W();

G0b.prototype.wb = function() {
    return wb;
}

var _v0B;
var _n0B;
var _e7B;
var _i7B;
var _p7B;
var _I0B;

// 抠出对应的object 

function get_H7z() {
    let g0b = new G0b();
    let aaa = new _v0B();
    return aaa.encrypt(g0b.wb());
}

function get_q7z(X1z, c, s, gt, challenge) {
    let g0b = new G0b();
    let passtime = 0;
    for (let index = 0; index < X1z.length; index++) {
        passtime += X1z[index][2];
    }
    console.log(_e7B.u(_e7B.t(new Date().getTime(), X1z), c, s));
    let Y7z = {
        "aa": _e7B.u(_e7B.t(new Date().getTime(), X1z), c, s),
        "userresponse": _i7B.C(Math.floor(Math.random() * 200), challenge),
        "passtime": passtime,
        "imgload": Math.floor(Math.random() * 200),
        "ep": {"v": "6.0.9"},               
        "rp": _I0B(gt + challenge.slice(0, 32) +  passtime)         
    };
    return _n0B.encrypt(JSON.stringify(Y7z), g0b.wb());
}

function get_r7z(q7z) {
    return _p7B.Ha(q7z);
}


var X1z = [[21,30,0],[1,0,22],[2,0,8],[2,0,17],[3,0,17],[3,0,16],[2,0,17],[3,0,16],[2,0,17],[3,0,17],[2,0,17],[1,0,16],[1,0,17],[1,0,33],[1,0,17],[2,0,16],[2,0,17],[1,0,17],[1,0,16],[1,0,17],[1,1,50],[1,0,67],[0,0,18383]];
var c = [12, 58, 98, 36, 43, 95, 62, 15, 12];
var s = "424f4e78";
var challenge = "bb56791bfda35fac04bd7f7b14a5c8654r";
var gt = "f5c10f395211c77e386566112c6abf21";


var h7z = get_H7z();
var q7z = get_q7z(X1z, c, s, gt, challenge);
var r7z = get_r7z(q7z);
var w = r7z + h7z;

console.log(w);
```



##### 查找bug

从开始逆向极验滑块，到完整的抠出w的算法只花了一天时间，本来一切顺风顺水，本来以为so easy，但是去用抠出来的w参数去发起ajax.php请求时，一直不成功。然后就是各种查找bug，查找bug花了我四天时间...期间各种猜想都尝试过了，感觉当时想着要不算了。但是想着自己前后花了快一个星期的时间，不能轻易言弃。这里列举主要的几个问题。



1. w参数的h7z和r7z两部分的关联性

前面说过，w参数是由2部分组成h7z和r7z。两部分看起来没有关联，其实这里有一个坑，这俩是有关联的。我们再理一下思路：

```js
h7z = V0B.encrypt(g0b.wb())
q7z = n0B.encypt(JSON.stringify(Y7z), g0b.wb())
r7z = p7B.Ha(q7z)
```

h7z和r7z的生成这俩都用到了一个随机字符串wb，但是这两个随机字符串必须一致!!!，就是说后端会解密h7z和r7z然后比较这两个随机字符串是否一致，如果不一致就会不通过。我前期就是在生成h7z和r7z的地方调用了2次wb，导致验证一直不通过。正确的做法调用一次wb，并用一个全局变量保存，然后生成h7z和r7z的地方直接去拿这个全局变量即可。



2. 同名的对象

另外导致一直通过的另外一部分原因是同名的对象有一些，再抠对象的时候一定要仔细，千万不能抠错。



3. aa轨迹的确定

跟某一个参数的值的时候，除了要在变量声明的地方分析变量值是如何变化的，还要注意在其它地方，尤其是逗号表达式的地方也隐藏着值的变化。比如：`j1r = (F7z = e7B[M9r.R8z(544)](F7z, V7z[M9r.R8z(190)][M9r.C8z(540)], V7z[M9r.R8z(190)][M9r.R8z(6)])`表面上是变量j1r的赋值，隐藏着变量F7z的值的变化。



这里介绍一些查找bug的技巧：

这里的w跟值包含2部分，r7z和h7z。可以先定位是h7z还是r7z的问题，定位到是哪半区的问题后，然后根据实际网站运行的结果和你自己编写的代码运行的结果一步步调试，2者结果为什么不一致，具体分析原因。

也可以利用浏览器的override功能或者hook尽可能的多输出一些变量的值，对比自己程序运行的结果和网站输出的值，看是哪一步出现问题。我们在调bug的时候就利用了override替换js文件输出了很多日志，如下图：

![image-20220428145928176](https://img.heshipeng.com/202204281459340.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

