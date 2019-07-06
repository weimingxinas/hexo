---
title: element-radio
date: 2019-07-06 14:26:36
tags: vue components
---
#### 原生单选框的实现：
    
#### vue单选框的实现：
只需要一个v-model即可达到互斥效果，v-model的值是data里面的数据，进行了双向绑定，由此可见并没有通过name属性来达到互斥，那么时怎么实现的呢？首先先来了解下v-model的本质，v-model本质上是语法糖

v-model：对input输入框的input事件进行监听，当键盘敲下时就实时改变searchText的值，同时修改searchText的值，输入框的value也跟着变化

```
function genRadioModel (
  el: ASTElement,
  value: string,
  modifiers: ?ASTModifiers
) {
  const number = modifiers && modifiers.number
  let valueBinding = getBindingAttr(el, 'value') || 'null'
  valueBinding = number ? `_n(${valueBinding})` : valueBinding
  addProp(el, 'checked', `_q(${value},${valueBinding})`)
  addHandler(el, 'change', genAssignmentCode(value, valueBinding), null, true)
}
```
addProp这个方法就会把checked属性的值_q('Jack',name)放入属性列表，这里_q是looseEqual方法的简写，表示宽松比较(如果是对象，则通过JSON.stringify转成字符串比较，否则直接String()转换比较)2个值是否相同，这样这里的逻辑就明确了，如果单选框的value的值和v-model的值相同，那么就加上一个checked属性，表示该单选被选中，自然而然其他单选框value的值和v-model的值不同，所以就不是选中状态，没有checked属性，所以达到了互斥效果

#### radio组件的要点:
##### 1. label放在最外层的作用是扩大鼠标点击范围，无论是点击在文字还是input上都能够触发响应。
##### 2. 隐藏input标签，自己模拟radio  
 真正的input透明度为0，且是绝对定位脱离文档流，因此不占空间且我们看不到，注意不是display:none或者visibility:hidden,如果是none或者hidden的话则无法触发鼠标点击了，只有opacity:0
##### 3. disabled
功能上的禁用是通过设置input的disabled属性来实现,下面源码中的真正的input的:disabled="isDisabled"

```
isDisabled() {
    return this.isGroup
      ? this._radioGroup.disabled || this.disabled || (this.elForm || {}).disabled
      : this.disabled || (this.e lForm || {}).disabled;
  },

```
##### 4.role="radio"

```
:aria-checked="model === label"
:aria-disabled="isDisabled"
```

复制代码这几句都是用来为不方便的人士提供的功能，比如屏幕阅读器，  role的作用是描述一个非标准的tag的实际作用。  
比如用div做button，那么设置div 的 role="button"，辅助工具就可以认出这实际上是个button。  
aria的意思是Accessible Rich Internet  
Application，aria-*的作用就是描述这个tag在可视化的情境中的具体信息。
##### 5.tabindex
规定了按下tab键该元素获取焦点的顺序，同样是个计算属性

```
tabIndex() {
    return !this.isDisabled ? (this.isGroup ? (this.model === this.label ? 0 : -1) : 0) : -1;
}
```
如果为禁用状态，tabindex为-1，则无法使用tab键使该元素获取焦点，如果不是禁用状态下，如果该单选按钮是在单选框组组件内且是选中状态则可以通过tab键获取焦点，否则无法通过tab键获取焦点，
当 tabindex > 0 的元素都切换之后，才会切换到 tabindex = 0 的元素，并且按出现的先后次序进行切换，这里的逻辑就是tab只能访问到选中状态下的单选按钮
##### 6.@keydown.space.stop.prevent="model = isDisabled ? model : label"
用tab切换不同选项时，按空格可以快速选择目标项，方便操作
##### 7.$nextTick
> $nextTick的作用是将回调延迟到下次DOM 更新循环之后执行
##### 8.slot

```
<slot></slot>
<template v-if="!$slots.default">{{label}}</template>
```
可以直接写成

```
<slot>{{label}}</slot>
```
