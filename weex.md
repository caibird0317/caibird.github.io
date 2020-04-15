
# WEEX在详情的简单使用和一些常见问题。
----

## weex是啥，为什么供应详情会用WEEX？
> Weex 是使用流行的 Web 开发体验来开发高性能原生应用的框架。在集成了 WeexSDK 之后，你可以使用 JavaScript 语言和前端开发经验来开发移动应用 。 考虑详情的迭代速度比较快，有问题能快速响应支持，不依赖APP发版本。weex开发完后会打成JS下载到本地，所以加载速度优于目前H5的模式。再加上WEEX一亩田也用。所以当时决策层 选型WEEX。

![img](http://tiebapic.baidu.com/forum/w%3D580/sign=83229efc9526cffc692abfba89004a7d/16d03687e950352af23bcd6d4443fbf2b2118b24.jpg)


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
  1. 浏览器调试仅仅只作为接口调试，样式啥的，千万别信浏览器。和APP相差太多
  2. app调试，安卓和IOS样式表现个不相同，各个底层SDK也不一样（APP终端继承）每个功能都需要验证安卓和IOS（踩过坑，小心 @_@! ） 。
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
```
 vue
```

| Vue 指令        | 是否支持           | 说明  |
| ------------- |:-------------:| -----:|
| v-html      | 不支持 | Weex 中没有 HTML 解析器  |
| v-on      | 支持   |   不支持事件修饰符 |
| v-show | 不支持      |   不支持 display: none; |
| v-cloak| 不支持      |  只支持单类名选择器 |


## weex怎么基于业务场景优化和规避一些常见问题。
  1. 


附上一些链接：

[如何评价阿里无线前端发布的Weex?](https://www.zhihu.com/question/37636296/answer/72881168)