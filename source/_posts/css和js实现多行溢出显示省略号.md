---
title: css和js实现多行溢出显示省略号
date: 2018-03-26 22:38:20
tags: js
---
背景：最近遇到个小问题，多行溢出显示省略号，查了大部分资料，只使用css竟然没有通用的方案，于是自己css+js写了一个


#### 单行文本溢出显示省略号
#####  常规css方法
```
width:27em;
overflow: hidden;
text-overflow:ellipsis;
white-space: nowrap;
-o-text-overflow:ellipsis;
```
#### 多行文本溢出显示省略号
#####  第一种方案（只适合chrome）firefox不支持
  - `-webkit-line-clamp` 是一个 不规范的属性（unsupported WebKit property），它没有出现在 CSS 规范草案中。

  - 作用:限制在一个块元素显示的文本的行数。 为了实现该效果，它需要组合其他外来的WebKit属性。常见结合属性：

    - display: -webkit-box：必须结合的属性 ，将对象作为弹性伸缩盒子模型显示 。  

    - -webkit-box-orient： 必须结合的属性 ，设置或检索伸缩盒对象的子元素的排列方式 。  
    
```
display: -webkit-box;
-webkit-box-orient: vertical;
-webkit-line-clamp: 2;  //这里是在第二行有省略号
overflow: hidden;
```
我们来看一下兼容性

支持版本| ie| firefox | safari | chrome | opera
---|--- | --- | --- |---| ---
桌面| no | no | yes | yes | yes
 移动端 | no | no | yes | yes | ?
 
##### 第二种方案（使用伪类:after）

```
&::after {
    content: "...";
    position: absolute;
    bottom: 0;
    right: 0;
    padding-left: 40px;
    background: -webkit-linear-gradient(left, transparent, #fff 55%);
    background: -o-linear-gradient(right, transparent, #fff 55%);
    background: -moz-linear-gradient(right, transparent, #fff 55%);
    background: linear-gradient(to right, transparent, #fff 55%);
}
```
这种方法是限定了div的高度，超出直接就不显示了，但是有一个缺点**当文字很短时后面也还跟着个省略号，没有溢出为什么还要显示省略号呢？**
原本想着通过js判断字符长度来动态添加这个伪类 **（注意，伪类无法动态添加，只能添加某个类。因此，设置该类的伪类为以上代码，并动态添加该类）**  
 js判断字符长度这个地方也很坑。因为英文字符和中文字符（或者其他符号）的长度不一，得去遍历+计算，这种办法我认为效率低，写着玩可以，不实用。

##### 第三种方法，我自己的方法

`clamp.js`做的正是这个事情，这是[源码链接](https://github.com/josephschmitt/Clamp.js/blob/master/clamp.js)，我们看看clamp的使用，看看他到底做了哪些事？
1. 判断是否支持webkit的line-clamp，如果支持则直接使用  
2. 获取当前的`line-height * clampValue`(行高乘以多少行，即你想显示的区块的高度)
3. 将步骤二取出的值（假定为showHeight）与element的clientHeight比较，clientHeight小于showHeight则不做处理，clientHeight大于showHeight的话，使用js将元素的innerText**不断切割**，直到clientHeight小于或等于showHeight  

*发现了一个小bug哈哈哈，在#244行，相信眼尖的你也发现了吧*  

我的想法：
1. 判断是否支持webkit的私有属性，支持就直接使用
2. 计算`line-height * clamp`（showHeight）,与clientHeight比较
3. clientHeight大于showHeight，则设置父div的高度为showHeight，css为overflow:hidden，并添加以上的css代码（省略号的伪类实现）
4. 将子div的overflow设置为默认值  

美滋滋，[上代码，欢迎PR](https://github.com/weimingxinas/clamp)(https://github.com/weimingxinas/clamp)

```js
import Vue from 'vue';
class Clamp {
    constructor (opt) {
        this.supportsNativeClamp = typeof (document.documentElement.style.webkitLineClamp) !== 'undefined';
        this.className = '_line_clamp_custom_directive';
        this.el = opt.el;
        this.clamp = parseInt(opt.clamp) || 2;
        this.init();
    }

    /**
     * Return the current style for an element.
     * @param {HTMLElement} elem The element to compute.
     * @param {string} prop The style property.
     * @returns {number}
     */
    computeStyle (elem, prop) {
        if (!window.getComputedStyle) {
            window.getComputedStyle = function (el, pseudo) {
                this.el = el;
                this.getPropertyValue = function (prop) {
                    const re = /(-([a-z]){1})/g;
                    if (prop === 'float') prop = 'styleFloat';
                    if (re.test(prop)) {
                        prop = prop.replace(re, function () {
                            return arguments[2].toUpperCase();
                        });
                    }
                    return el.currentStyle && el.currentStyle[prop] ? el.currentStyle[prop] : null;
                };
                return this;
            };
        }
        return window.getComputedStyle(elem, null).getPropertyValue(prop);
    }

    /**
     * Return the current style for an element.
     * @param {HTMLElement} elem The element to compute.
     * @returns {number}
     */
    getLineHeight (elem) {
        let lh = this.computeStyle(elem, 'line-height');
        if (lh === 'normal') {
            // Normal line heights vary from browser to browser. The spec recommends
            // a value between 1.0 and 1.2 of the font size. Using 1.1 to split the diff.
            // line height 每个浏览器各有差异。normal的情况为默认值，浏览器会计算出“合适”的行高，
            // 多数浏览器（Georgia字体下）取值为1.14，即为font-size的1.14倍，如果未设定font-size，
            // 那既是基准值16px的1.14倍
            lh = parseInt(this.computeStyle(elem, 'font-size')) * 1.2;
        }
        return parseInt(lh);
    }

    /**
     * 动态添加class
     */
    addStyle () {
        const hasStyle = !!document.getElementById('lineClampCustomDirective');
        if (!hasStyle) {
            const sty = document.createElement('style');
            sty.type = 'text/css';
            sty.id = 'lineClampCustomDirective';
            const str = `.${this.className} {
                position: relative;
                overflow: hidden;
            }
            .${this.className}::after {
                content: "..."; position: absolute; bottom: 0; right: 0; padding-left: 40px;
                background: -webkit-linear-gradient(left, transparent, #fff 55%);
                background: -o-linear-gradient(right, transparent, #fff 55%);
                background: -moz-linear-gradient(right, transparent, #fff 55%);
                background: linear-gradient(to right, transparent, #fff 55%);
            }`;
            // ie use styleSheet.csstext
            sty.styleSheet ? sty.styleSheet.cssText = str : sty.innerHTML = str;
            document.getElementsByTagName('head')[0].appendChild(sty);
        }
    }
    getMaxHeight () {
        const lineHeight = this.getLineHeight(this.el);
        return lineHeight * this.clamp;
    }

    /**
     * init
     */
    init () {
        const sty = this.el.style;
        if (this.supportsNativeClamp) {
            sty.overflow = 'hidden';
            sty.textOverflow = 'ellipsis';
            sty.webkitBoxOrient = 'vertical';
            sty.display = '-webkit-box';
            sty.webkitLineClamp = this.clamp;
        } else {
            const height = this.getMaxHeight(this.clamp);
            if (height <= this.el.clientHeight) {
                sty.overflow = 'visible';
                this.addStyle();
                const newEl = document.createElement('div');
                newEl.classList.add(this.className);
                newEl.style.height = height + 'px';
                this.el.parentNode.replaceChild(newEl, this.el);
                newEl.appendChild(this.el);
            }
        }
    }
}
Vue.directive('clamp', {
    // 当被绑定的元素插入到 DOM 中时……
    inserted: function (el, binding, vnode, oldVnode) {
        if (!binding.value) return false;
        /* eslint-disable no-new */
        new Clamp({
            el,
            clamp: binding.value
        });
    }
});

```
