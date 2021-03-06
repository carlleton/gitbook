#组件

avalon拥有两大利器，强大的组件化功能以应对**复杂墙**问题，顶级的虚拟DOM机制来解决**性能墙**问题。

组件可谓是指令的集合，但`1＋1  >  2`!


##组件容器

组件容器是一个占位用的元素节点. 当avalon扫描到此位置上时将它替换成组件.

在avalon2中有4类标签可以用作组件容器,分别是wbr, xmp, template, 及ms-开头的自定义标签.

其兼容性如下

| 元素        | 类型           | 说明  |
| :------------- |-------------| -----|
| wbr     | 所有浏览器, 自闭合标签    | 需要使用ms-widget或is来指定组件类型 |
| xmp     | 所有浏览器, 闭合标签       |  需要使用ms-widget或is来指定组件类型,里面可以使用slot属性元素 |
| template | IE9+及W3C浏览器,闭合标签  |  需要使用ms-widget或is来指定组件类型,里面可以使用slot属性元素 |
| ms-* | IE9+及W3C浏览器,闭合标签      | 可以省略ms-widget, 里面可以使用slot属性元素|

> 
>闭合标签，比如`<div></div><a></a>`
>自闭合标签，比如`<input><img><br><link>`

根据上表, 如果要兼容IE6-8, 那么只能使用wbr, xmp来做组件容器

如果不打算支持IE, 那么使用template元素性能最好

如果追求语义化, 只支持IE9+及其他现代浏览器,则使用ms-*自定义标签.

>xmp元素里面不能放xmp, template元素里面不能放template,这是html规范,就像script元素里面不能放script, textarea元素里面不能放textarea. 但我们可以在这些元素里面直接放ms-*自定义标签.

```html
<xmp ms-widget="{is:'ms-dialog'}">
<wbr is="ms-star">
<ms-title slot="title">{{@title}}</ms-title>
</xmp>

```

![](component.png)

**在2.1.16后,如果你为组件容器指定了is属性,没有指定ms-widget,它会默认生成ms-widget属性**

组件容器里是{% em color='red' %}不支持ms-widget之外的其他绑定属性{% endem %}

```html
<wbr :widget="{is:'ms-button'}" :attr="{title:@aaa}" />
```

上面的`:attr`属性被忽略 

##声明组件

如果我们想在页面上使用组件,需要用组件容器与ms-widget/is指令声明一下.

```html
<wbr ms-widget="{is:'ms-button'}" />
```
这就是声明使用一个按钮组件.

当然还有其他几种方式

```html
<xmp ms-widget="{is:'ms-button'}" /></xmp>
<template ms-widget="{is:'ms-button'}" ></template>
<xmp is="ms-button" /></xmp>
<template is='ms-button' ></template>
<ms-button></ms-button>
```



自定义标签都是闭合标签,不能写成下面这样
```html
<ms-button />
```
但是如果你的ms-button是放在xmp或template下面,则允许这样写.

```html
<xmp ms-widget="{is:'ms-dialog'}">
<ms-title slot="title" />
<div slot="content">这是弹出层的内容</div>
<div slot="footer">
<ms-button :widget="@ok" /> <ms-button :widget="@ng" />
</div>
</xmp>
```

如果我们根据is 位置，还可以为分为两类

**avalon静态组件**
is是预先规定了组件的类型，不可更改
```html
<wbr is="button" ms-widget="@options" />
```

**avalon动态组件**
is是可改变的
```html
<wbr  ms-widget="[{is: @is},@options]" />
```

##组件命名

由于组件名在高级浏览器中,可以作用自定义标签的标签名.而HTML标签在HTML5中有严格的规定, 只能出现 $,-,数字与英文单词, 并且只能以字母开头, 中间必须有'-'.

此外,为了方便avalon辨识这个标签名是否为一个数组,avalon强制规定以ms-开头, 即

> ms-button, ms-date-picker, ms-router-link

但是如果你不用自定义标签声明组件,使用{% em color="#ffafc9" %}ms-widget配置对象{% endem %}
来声明组件呢, 你就可以突破部分限制, 可以不以`ms-`开头

> logger, date-picker, router-link

```html
<wbr ms-widget="{is:'texer'}" />
```

##配置对象

`ms-widget`可以省略成`:widget`,它应该对应一个对象, 即配置对象.


avalon2 的默认配置项比avalon1.5 少许多。所有组件通用的配置项

* is, 字符串, 指定组件的类型。如果你使用了自定义标签，从标签名就得知组件类型,则可以省略。

* id, 字符串, 指定组件vm的$id，这是可选项。如果该组件是位于SPA的子页里面，也请务必指定此配置项，能大大提高性能。


* onInit, onReady, onViewChange, onDispose四大生命周期钩子。


> 注意,如果你在配置对象没有写id({% em color="#a9ea00" %}或$id, 兼并2.1.4之前的版本{% endem %}), 会在控制下看一个警告


其他组件需要传入的属性与方法，也可以写配置对象中。


> 此外,  如果你的组件是位于SPA的子页面中，或是由ms-html动态生成。

但组件对应的真实节点被移出DOM树时，该组件会被销毁。为了进一步提高性能，你可以在组件容器中定义一个cached属性，其值为true，它就能常驻内存。

```html
<wbr cached="true" ms-widget="[@allConfig, {id: 'xxx_'+$index}]"/>
```

> 用了cached时，必须指定**id**配置项(2.1.5之前是$id)。


##插槽元素与插卡元素

为了方便传入很长的HTML格式的参数，web components规范发明了slot机制。

avalon使用了一些黑魔法也让旧式IE浏览器支持它。

通俗来说, 我们用组件容器为组件占位, 我们也用**插槽容器**为**插卡元素**占位.

我们看一下`ms-view`组件的定义与声明:
```javascript
avalon.component('ms-view',{
    template:'<div class="view"><slot name="content" /></div>',
    defaults: {
       content: ""
    }
})
```


```html
<div ms-controller='test'>
<ms-view>
<div slot="content">这是子视图的内容</div>
</ms-view>
</div>
```

`<slot name="content" />` 叫做**插槽元素**,用来占位的,实际上它在内部会转换两个注释节点

```html
<div class="view">
<!--slot:content-->
<!--slot-end:-->
</div>
```

组件容器中的带slot属性的元素, `<div slot="content">这是子视图的内容</div>`,就是**插卡元素**. 插卡元素最终会移动到组件对应的注释节点中去.
```html
<div class="view">
<!--slot:content-->
<div slot="content">这是子视图的内容</div>
<!--slot-end:-->
</div>
```


##soleSlot机制

中文叫**单插槽**或**匿名插槽**. 这是插槽机制的一个特例.

比如我们做一个按钮组件:

```javascript
avalon.component('ms-button', {
    template: '<button type="button"><span><slot name="buttonText" /></span></button>',
    defaults: {
        buttonText: "button"
    }
})

```

那么外面要这么使用
```html
<ms-button><b slot="buttonText">xxx</b></ms-button>
```

事实上我们只想传入一个文本,不想传入b元素.这样定义太冗余了.

就像button标签,可以直接
```html
<button>按钮</button>
```
于是就有单插槽机制. 它要求`组件`内部只有一个地方可以插东西, 并且将`组件容器`的所有孩子或文本都作为一个插卡.

我们看一下新的定义与声明方式:
```javascript
avalon.component('ms-button', {
    template: '<button type="button"><span><slot /></span></button>',
    defaults: {
        buttonText: "button"
    },
    soleSlot: 'buttonText'
})

```

```html
<ms-button>xxx</ms-button>
```

avalon扫描后, 生成的组件是这个样子:


```html
<button type="button">
<span>
<!--slot:buttonText-->
xxx
<!--slot-end:-->
</span>
</button>
```

插槽机制可以解决我们传入大片内容的难题, 多个slot元素拥有同一个name值。

![](./dom-insert.png)


##组件定义

avalon定义组件时是使用**avalon.component**方法。

avalon.component方法有两个参数,第一个标签名,必须以ms-开头;第二个是配置对象.

配置对象里也有3个配置项

* template,自定义标签的outerHTML,它必须是用一个普通的HTML元素节点包起来,里面可以使用ms-*等指令, 里面的属性只能是{% em color="red" %}defaults中定义的属性与方法{% endem %}
* defaults,用来定义这个组件的VM有什么属性与方法
* soleSlot,表示组件只有一个插槽,会将组件容器的所有孩子都移到这里来 ,可选。


```javascript
avalon.component('ms-pager', {
      template: '<div><input type="text" ms-duplex-number="@num"/><button type="button" ms-on-click="@onPlus">+++</button></div>',
      defaults: {
          num: 1,
          onPlus: function () {
              this.num++;
          }
      }
  });  
```
##组件的作用域

2.2之前，组件的VM是会继承它上方的所有VM的属性，导致改动了某一个组件的属性后，影响到其他地方。

2.2之后，组件的VM是完全独立，它的属性只能是defaults里面预设好的。然后我们通过ms-widget传入一个对象，对它进行赋值（defaults中没有的属性是不会赋值）


##生命周期

avalon2组件拥有完善的生命周期钩子，方便大家做各种操作。

|   | avalon2| web component | react |
| -- | -- | -- | -- |
| 初始化 | onInit | createdCallback | getDefaultProps |
| 插入DOM树 | onReady | attachedCallback | componentDidMount |
| 视图变化 | onViewChange | attributeChangedCallback | componentDidUpdate |
| 移出DOM树 | onDispose | detachedCallback | componentWillUnmount |



onInit，这是组件的vm创建完毕就立即调用时，这时它对应的元素节点或虚拟DOM都不存在。只有当这个组件里面不存在子组件或子组件的构造器都加载回来，那么它才开始创建其虚拟DOM。否则原位置上被一个注释节点占着。

onReady，当其虚拟DOM构建完毕，它就生成其真实DOM，并用它插入到DOM树，替换掉那个注释节点。相当于其他框架的attachedCallback， inserted, componentDidMount.

onViewChange，当这个组件或其子孙节点的某些属性值或文本内容发生变化，就会触发它。它是比Web Component的attributeChangedCallback更加给力。

onDispose，当这个组件的元素被移出DOM树，就会执行此回调，它会移除相应的事件，数据与vmodel。

##组件间通信

只要能拿到通信双方的组件实例就行，onInit, onReady的事件对象都有一个vmodel属性，就是它的实例，然后操作对方的实例属性就行了。

```javascript
var c1, c2
avalon.component('ms-a', {
    template: '<button type="button">A</button>',
    defaults: {
       onInit:function(e){
            c1 = e.vmodel
        },
        buttonText: "button"
    }
})

avalon.component('ms-b', {
    template: '<button type="button">B</button>',
    defaults: {
        onInit:function(e){
            c2 = e.vmodel
        },
        buttonText: "button"
    }
})

```

##工作原理

avalon会用之前的VM来扫描组件容器的内部, 生成节点集合nodes1

avalon根据ms-widget配置对象与组件的defaults属性,生成新的VM,然后扫描组件的template,生成节点集合nodes2

avalon根据soleSlot或用户在slot属性,在nodes1中查找所有插卡元素

然后将插卡元素在nodes2中替换同名的插槽元素

将nodes2转换真实DOM,替换组件容器.

以后组件VM的属性发生改变,或ms-widget的传参发生改变,只会对个别位置进行最小化刷新,不再重复上面步骤



##具体例子

请移步到[Github](https://github.com/RubyLouvre/avalon/tree/master/perf/component)


## 官方组件或推荐组件

### Promise 
[mmPromise](https://github.com/RubyLouvre/mmDeferred)
```
npm install avalon-promise
```
[bluebird](https://github.com/petkaantonov/bluebird)

[ypromise](https://github.com/yahoo/ypromise/blob/master/promise.js)

### ajax组件 

重点推荐
[fetch-polyfill](https://github.com/RubyLouvre/fetch-polyfill)
```
npm install fetch-polyfill2

```
[mmRequest](https://github.com/RubyLouvre/mmRequest)

[qwest.js](https://github.com/pyrsmk/qwest)

[reqwest.js](https://github.com/ded/reqwest)

[ForbesLindesay/ajax](https://github.com/ForbesLindesay/ajax)

##ajax提交出问题

请确保你提交的数据是纯JS数据，而不是vm。请见[数据模型](http://avalonjs.coding.me/vm.html#数据模型)

```javascript
function submitData(){
    var data = vm.Item.$model //如果Item是数组，在IE6－8，请改用
    // data = vm.Item.toJSON(), 如果想提交整个vm，请改用
    // data = vm.$model
    $.ajax({
          url: '../sdfwds/dsfds/dsfsd.action',
          data:data,
          type: 'GET',
          success: function(){

          }

    })
}
```

### redux事件派发组件 
[mmDux](https://github.com/RubyLouvre/mmDux)
```
npm install mmDux
```

### 路由组件

[mmRouter](https://github.com/RubyLouvre/mmRouter)

[page.js](https://github.com/visionmedia/page.js)


### 动画组件 
[tween.js](https://github.com/tweenjs/tween.js)
```
npm install tween.js

```

### 弹窗组件 
[ms-modal](https://github.com/RubyLouvre/ms-modal)
```
npm install ms-modal
```

### 分页组件 
[ms-pager](https://github.com/RubyLouvre/ms-pager)
```
npm install ms-pager
```



