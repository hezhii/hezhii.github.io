---
title: Polymer 2.0
date: 2017-08-08 23:05:20
categories:
  - 技术
tags:
  - Polymer
  - 前端
---
{% blockquote %}
**Unlock the Power of Web Components**. Polymer is a JavaScript library that helps you create custom reusable HTML elements, and use them to build performant, maintainable apps.
{% endblockquote %}

<!-- more -->

这篇博客是我在学习[Polymer 2.0](https://www.polymer-project.org/)的过程中记录的一些笔记，并不是一个Polymer的教程。博客记录了我在学习Polymer时认为重要的一些地方以及一些自己的理解，便于以后回过头来复习和总结。

## Polymer元素

1. 使用`<dom-module>`标签来定义组件的样式和文档内容，给`<dom-module>`赋予一个匹配组件的is属性的id标记，并在`<dom-module>`中添加一个插入一个`<template>`标签，Polymer就会自动克隆模板内容到组件的local DOM中。
  {% codeblock lang:html %}
    <dom-module id="x-foo">
      <template>
        <style></style>
        <div>I am x-foo!</div>
        
        <!--类似于vue中的slot-->
        <content></content>
        
     </template>
    </dom-module>
    
    <script>
      Polymer({
        is: 'x-foo'
      });
    </script>
  {% endcodeblock %}
2. 自定义元素名称必须是小写字母，且至少包含一个`-`。
3. Polymer现在只支持扩展原生HTML组件（例如input或button，将来会支持扩展其它自定义组件），这些原生组件扩展被称为**自定义类型扩展组件**。
4. 为了在组件之间共享样式，可以在一个`<dom-module>`组件内打包一个样式集合。
  {% codeblock lang:html %}
    <dom-module id="shared-styles">
      <template>
        <style>
          .red { color: red; }
        </style>
      </template>
    </dom-module>
    
    <!--如何使用-->
    <link rel="import" href="../shared-styles/shared-styles.html">
    <dom-module id="x-foo">
      <template>
        <!-- include the style module by name -->
        <style include="shared-styles"></style>
        <style>
          :host { display: block; }
        </style>
        
        <div>I am x-foo!</div>
      </template>
    </dom-module>
    <script>
      Polymer({is: 'x-foo'});
    </script>
  {% endcodeblock %}

## 组件

### 属性

1. **type**：属性类型。支持Boolean、Date、Number、String、Array、Object，通过覆盖*_deserializeValue*方法可以增加对其他属性的支持。
2. **value**：属性的默认值。Boolean、Number、String、Function。如果默认值是数组或者实例对象，通过函数返回。
3. **computed**：计算属性。
4. **notify**：Boolean，当属性改变时，是否触发事件。触发propertyName-change
5. **readOnly**：私有属性，通过方法调用。
6. **reflectToAttribute**：当组件的属性发生变化时，更新dom上的相应属性。
7. 在template中使用属性时，驼峰命名需要使用-的方式，如果是元素原生的属性，后面加上$符号。

### 行为

1. Polymer支持使用称为**behaviors**的共享代码模块来扩展自定义组件原型。行为是一个看起来像一个典型的Polymer原型的对象；行为可以定义生命周期回调、声明的属性、默认标记、观察器和监听器；行为是从组件中提取出来的公共方法，类似于切面编程。
  {% codeblock lang:javascript %}
    Polymer({
      is: 'super-element',
      behaviors: [SuperBehavior]
    });
  {% endcodeblock %}

### 事件

1. **listeners**对象中定义了事件及其相应的处理方法。事件监听器可以监听`this.$`集合中的任意元素上的事件，事件类型需要定义为`elementId.eventName`的方式，这也就意味着，这种方式只能为拥有id的节点添加事件监听。
  {% codeblock lang:html %}
    <dom-module id="x-custom”> 
      <template> 
        <div>I will respond</div>
          <div>to a tap on</div>
          <div>any of my children!</div>
          <div id="special">I am special!</div>
      </template> 
    </dom-module>
    
    <script> 
      Polymer({
        is: 'x-custom’,
        listeners: {
           'tap': 'regularTap’,
           'special.tap': ‘specialTap'
           },
        regularTap: function(e) {
          alert("Thank you for tapping"); 
         }, 
        specialTap: function(e) {
          alert("It was special tapping"); 
         } 
      }); 
    </script>
  {% endcodeblock %}
2. 基于上面个的方法，如果不想仅仅为了添加事件而指定元素id，可以通过on-event的方法指定事件。
3. 通过下面的方法可以绑定或解绑事件。
  {% codeblock lang:js %}
    this.listen(this.$.myButton, 'tap', 'onTap');
    this.unlisten(this.$.myButton, 'tap', 'onTap');
  {% endcodeblock %}
4. *fire*方法可以出发一个自定义事件。

## 数据系统

1. **notify**属性控制属性的修改是否会**向上流动**，默认值为`false`，即该属性修改后，不会触发任何东西。不会影响到其他的，**阻止数据出去**。
2. **readOnly**属性控制属性是否可以**向下流动**，默认值为`false`，即相应数据的修改不会影响到自己，**阻止数据进来**。
3. `[[]]`阻止数据**向上流动**，即使指定**notify**为`true`。
4. **注意**：这里的向上、向下流动，不是data和界面之间的流动，而是父组件和子组件。notify和readOnly是对自己的属性而言的，而[[ ]]和{% raw %}{{ }}{% endraw %}相当于是在使用该属性的时候，对使用的那个属性而言。

## 全局配置

1. **dom**:
    * `shady`，所有的Local DOM都使用shady DOM进行渲染，即使只是shadow DOM(当前默认使用此选项).
    * `shadow`，Local DOM在支持shadow DOM时使用shadow DOM做渲染(未来会默认使用此选项)。
2. **lazyRegister**:
    * `true`，将一些注册时的活动延迟到第一个组件实例被创建时，默认是`false`(默认值将来可能改变)。
    * `max`，延迟所有行为执行一直到第一个组件被创建。当设置**lazyRegister**为`max`时，不能改变一个组件的`is`属性或通过定义*factoryImpl*方法来创建一个自定义构造函数。Polymer会调用组件的*beforeRegister*用以保留使用ES6定义组件的能力。组件的*beforeRegister*会在特性的*beforeRegister*之前调用.
3. **useNativeCSSProperties**：为`true`时, Polymer在浏览器支持时使用本地自定义CSS属性。默认是`false`，由于Safari 9还不支持。
4. **noUrlSettings**：为`true`时, Polymer的设置只能通过页面上脚本来设置，也就是说通过URL查询字符串设置的`?dom=shadow`会被忽略，默认为`false`。

## API

1. **Polymer.CaseMap**：提供两个静态方法，用来完成`-`和驼峰命名之间的互转。`polymer-element <==> polymerElement`

## 特性

1. 支持父子组件双向的数据流，对数据的流动有完全的控制权。
2. 组件间的样式共享，利用CSS变量，进行组件的样式定制。
3. 修改了对象的getter、setter，当对象的属性发生变化的时候，调用set方法设置属性并调用监听该属性变化的方法。
4. behavior的作用和vue中的mixin类似，提供一些通用的处理。
