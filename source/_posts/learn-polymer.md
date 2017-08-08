---
title: Polymer 2.0
date: 2017-08-08 23:05:20
categories:
  - 学习笔记
tags:
  - Web framework
  - Polymer
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