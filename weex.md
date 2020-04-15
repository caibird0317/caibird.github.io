
# WEEX在详情的简单使用和一些常见问题。
----

## weex是啥，为什么供应详情会用WEEX？
> Weex 是使用流行的 Web 开发体验来开发高性能原生应用的框架。在集成了 WeexSDK 之后，你可以使用 JavaScript 语言和前端开发经验来开发移动应用 。 考虑详情的迭代速度比较快，有问题能快速响应支持，不依赖APP发版本。weex开发完后会打成JS预下载到本地，所以加载速度优于目前H5的模式。再加上WEEX一亩田也用。所以当时决策层 选型WEEX。

## weex比较常用的的标签和注意事项 
- div
```
1. 不要在 <div> 中直接添加文本，无法在APP内正确显示。
2. 在 Weex 中，<div> 不可滚动。
3. 要控制 <div> 的层级，建议不要超过14层（官方建议，但是实际发现12层就看到JS告警日志）。
4. 子组件无限制，包括<div>自己
```
- text
```
1. 不支持子组件，包括自己
2. 里直接写文本头尾空白会被过滤，除非是空格加动态文本形式 <text>  {{textData}}</text>
```
 - image 

```
1. 必须指定样式中的宽度和高度和图片地址，图片才能正确显示。
2. 不支持内嵌子组件
3. resize属性支持cover / contain / stretch 几种值（类似于object-fix）。
```
 - list （组件是提供垂直列表功能的核心组件，拥有平滑的滚动和高效的内存管理，非常适合用于长列表的展示。）
```
1. 不允许相同方向的 <list> 或者 <scroller> 互相嵌套
2. 在APP内，不可见的list cell会被回收，不会常驻内存。
```
- cell
```
1. 必须以一级子组件的形式存在于 list（cell的父组件必须是list）。
2. cell的子组件可以添加任意类型的组件，但尽量避免滚动组件。
```
- scroller
```
用法和注意事项 基本和list组件相同
不同在于 scroller常驻内存，不会被销毁，而list不可见部分会被回收。
```
slider
```
轮播组件 没啥好说的。
```
更多标签的>>>>>>[传送门](https://weex.apache.org/zh/docs/components/div.html)


## weex使用（开发 调试  发布）。
````
开发： 见code地址： http://code.cnhnb.com/webapp/waxberry.git。

调试：
  1. 浏览器调试仅仅只作为接口调试，样式啥的，千万别信浏览器,千万别信浏览器,千万别信浏览器。和APP相差太多
  2. app调试，安卓和IOS样式表现个不相同，各个底层SDK也不一样（APP终端继承）每个功能都需要验证安卓和IOS。踩过坑，小心 (*^_^*)。
     所以weex开发存本远大于H5。

发布： weex的发布流程是，前端打包js上传到指定域名服务器中。然后需要修改APP启动配置，定义WEEX版本高于上一次发布版本，加载最新资源。

````

## WEEX怎么样和APP进行交互。
> 原理通H5交互， APP本地启动时实例化一条WEEX与APP交互通道(HnWeexModule),weex通过requireModule来加载这个通道
    
 调用APP登录的例子:
````javascript
//  注册
const hnModule = weex.requireModule("HnWeexModule");
// 调用 
hnModule.gotoNative("login");
````

## weex中CSS和vue语法注意事项
>weex之所以难用在于 [CSS 传送门](https://weex.apache.org/zh/docs/styles/common-styles.html) 裁剪的比较丑陋
````css
1. 所有的CSS语法只支持列出来的文档列出来的. 见上面CSS传送门。
2. 不支持简写 譬如 :
   .page{
       background:#666666; /*APP无作用*/
       background-color: #666666; /* OK */
   }
3. display仅只支持flex属性，不需要手动设置。（你细品，会导致复杂点的布局基本会做不了）。
4. 样式分为通用样式和组件样式,for example:
    div{
        font-size: 16px; /* 无效  属于text的组件样式 */ 
        background-color: red; /* 有效 通用样式 */
    }
    text{
        font-size: 16px; /* 有效  text组件的专属样式*/ 
        background-color: red; /* 有效 通用样式 */
    }

5. 定位position不支持z-index. 层级的显示顺序 后面或覆盖前面的。
6. 单位支持px和wx 区别在于px动态适配屏幕， wx否之。
7. 每个样式属性对平台的处理方式可能会存在兼容问题。用前要看文档.
8. 伪类只支持4个 active, focus, disabled, enabled。所有组件都支持 active, 
   但只有 input 组件和 textarea 组件支持 focus, enabled, disabled。
9. 出于性能考虑，Weex 目前只支持单个类选择器，并且只支持 CSS 规则的子集。
10. 其他若干问题，暂不具体描述。

````

>vue一些指令也做了一些裁剪， 依赖样式的指令基本用不了，这里只列出常见的具体见[官方文档](https://weex.apache.org/zh/guide/use-vue-in-weex.html#%E6%94%AF%E6%8C%81%E7%9A%84%E5%8A%9F%E8%83%BDl)

| Vue 指令        | 是否支持           | 说明  |
| ------------- |:-------------:| -----:|
| v-html      | 不支持 | Weex 中没有 HTML 解析器  |
| v-on      | 支持   |   不支持事件修饰符 |
| v-show | 不支持      |   不支持 display: none; |
| v-cloak| 不支持      |  只支持单类名选择器 |


## weex怎么基于业务场景优化和规避一些常见问题。
  ### 1. 关于weex得降级处理方案。
  ````
  weex上线以来陆陆续续接到客服反馈得信息，少部分用户会一直打不开weex供应详情，经日志排查，打不开基本时安卓实例化WEEX SDK失败，
  这个情况下，当weex 实例化失败时候，直接跳转H5详情，这也是为啥同时存在weex和H5得2个详情得原因。也为后续无缝切换H5方案做了铺垫。
  ````
  
  ### 2.图片兼容问题IOS无法显示。
  ````
    详情图片没有编码会导致在IOS无法显示，必须针对图片进行encodeURI(url)编码，原因是图片里面可能存在特殊字符譬如？，导致src被IOS截掉。
  ````

  ### 3.关于WEEX详情大图显示图片模糊，文字无法看清得问题详细处理过程及解决方案(有点长，不感兴趣可以直接跳过)。
  ````html
  前面说了 <image>这个标签它必须定宽和有效得地址源。之前详情显示货品详情得处理方式是
  开始先设置图片得默认得宽是750px 高为250px,然后通过加载完回调回来得图片信息等比
  缩放得到宽去显示图片，觉得完美规避了weex的坑。事实发现还是too young too naive！
  随着柑橘交易中心上线，运营反馈里面文字看不清，要求开发排查。
  一 开始认为是图片本身的问标，然后下载发现高清文字清晰，排除图片本身问题
  二 开始总结规律，发现有的图片模糊，有的清晰，是不是跟图片本身有关。
     2.1 引入固定的清晰和模糊的2张图片，运行发现依然如初
     2.2 改变模糊的图片的属性宽高，宽的值不能动一屏幕，只能调整高，发现高度确实会引起图片变模糊
     2.3 找到变模糊的阈值，发现阈值敲好是原始尺寸缩放到一屏幕宽得的等比高度。（好像破案了）
     2.4 控制变量法验证这个问题，开始对清晰的图下手了，设置默认值高度，发现变模糊了。（确定了方向）

  得出结论： 当初始值是小于等比的默认高度是，图片原始信息返回回来图片模糊。

  解决办法： 当时分了几步去考虑。
     1. 每个图片都是用户上传得，无法控制到底是正方形或者长方形，意味着一开始我必须得知道缩放比。
     2. 知道缩放比，必须知道图片真实得宽高数据，意味着必须预加载图片信息（接口没有，本着学习得态度自己去拿图片的各种数据）。
     3. 怎么样去设置图片，想到得是改造图片数据把真实信息返回回来插入到图片中这样预设的宽高就是后面要显示的宽高。
     4. 到了这步就剩下怎么拿数据了，一开始觉得很简单 new Image()然后 调用onload方法就完了，事实发现想多了。
        4.1 WEEX根本没有new Image()
        4.2 能否按照视图层load方法循环拿到图片数据 <image v-show="0" src="xxx" @load=""> ,操蛋的是v-show根本不支持。
        不想设置到底下后面删DOM，觉得太丑陋了（其实当时想的是，没有别的方法就用这个了~）
        4.3 冷静了个把小时， 图片是上传到七牛，七牛有没有API能拿到图片属性，imageInfo这个API满足~~~。
        4.4 开始构建_loadImagePromise方法，考虑到很多图片，其中一张图片挂了的情况构建数据.代码地址（utils/loadImageAll.js）

    测试环境验证无问题，上线观测稳定，结案。

    体会： 心怀敬畏（qu ni da ye），特别是对WEEX。
  ````
  ### 4. 优选的标志显示在标题的处理方法。（是的，你没看错）
  ````html
    前面说到了display只有flex.需求是2行能换行且超出省略。text组件不能嵌套，text-indent before伪类根本就不支持。
    问题就来了。本着解决不了问题，就把提问题的人解决掉的原则进行了：
    1. 想说服产品 UI如果是优选，一行分左右左侧显示优选标志，右侧显示标题。UI回复： XXXXXXXXXX省略1000字。 我：好的。
    2. 绕回来了，解决不了提问题的人，就必须解决问题。 能不能将优选的图定位盖在文字上，然后文字首行预留位置？
       2.1 pading-left设置位置会导致第二行也会留白 PASS掉。
       2.2 前面说到TEXT 头尾空白会被过滤，除非是空格加动态文本形式。思路有了
       2.3 代码：
        <!-- 是否打优选的标 -->
        <div>
            <div class="title-container-left" v-if="shop && shop.isYx">
                <text class="iconfont show-yx">&#xe73e;</text>
                <text class="detail-title">{{"        "+detail.title}}</text>
            </div>
            <text class="title-container-left detail-title" v-else>{{detail.title}}</text>
        </div>

    结案。
  ````

  ### 5. APP启动时weex升级流程如下  
![img](https://files.cnhnb.com/tools/markdown/weex.png)

  ### 6. 未完待续ing.期待你们的体验


## 附上一些链接：

>[如何评价阿里无线前端发布的Weex?](https://www.zhihu.com/question/37636296/answer/72881168)

>[weex :issue](https://github.com/alibaba/weex/issues)

后记：如果哪天下掉WEEX，记得庆祝下。

![img](https://img.99danji.com/uploadfile/2020/0413/20200413114859998.gif)