---
title: element-input
date: 2019-07-06 14:26:36
tags: vue components
---

##### 1. 布局
修改前后置元素内容可以发现，中间input的宽度是自适应的  
采用table-cell布局，table内表格宽度都是自适应的，某一列很宽的话，另外的列就会变窄
##### 2. 善用v-bind="$attrs"
##### 3. compositionstart 

```javascript
@compositionstart="handleComposition"
@compositionupdate="handleComposition"
@compositionend="handleComposition"
```
compositionstart 事件触发于一段文字的输入之前（类似于 keydown 事件，但是该事件仅在若干可见字符的输入之前，而这些可见字符的输入可能需要一连串的键盘操作、语音识别或者点击输入法的备选词）
简单来说就是切换中文输入法时在打拼音时(此时input内还没有填入真正的内容)，会首先触发compositionstart，然后每打一个拼音字母，触发compositionupdate，最后将输入好的中文填入input中时触发compositionend。触发compositionstart时，文本框会填入 “虚拟文本”（待确认文本），同时触发input事件；在触发compositionend时，就是填入实际内容后（已确认文本）
##### 4. textarea 的自动计算

```javascript
  if (!hiddenTextarea) {
    hiddenTextarea = document.createElement('textarea');
    document.body.appendChild(hiddenTextarea);
  }

  hiddenTextarea.setAttribute('style', `${contextStyle};${HIDDEN_STYLE}`);
  hiddenTextarea.value = targetElement.value || targetElement.placeholder || '';
```
非常的简单明了，思路如下
1. 创建textarea元素，并设置为不可见
设置隐藏的代码如下：

```javascript
const HIDDEN_STYLE = `
  height:0 !important;
  visibility:hidden !important;
  overflow:hidden !important;
  position:absolute !important;
  z-index:-1000 !important;
  top:0 !important;
  right:0 !important
`;
```
2. 将textarea的值设置为输入的值，并获取scrollHeight；并确定是哪一种盒模型，计算出最终高度；
```javascript
  let height = hiddenTextarea.scrollHeight;
  if (boxSizing === 'border-box') {
    height = height + borderSize;
  } else if (boxSizing === 'content-box') {
    height = height - paddingSize;
  }
```
3. 如果有设置minRows或者maxRows则与height比较

```javascript
 hiddenTextarea.value = '';
let singleRowHeight = hiddenTextarea.scrollHeight - paddingSize;
```
这里的代码是取得单行的高度（注意，要减去paddingSize）
4. minheight的设置  
- elemet： calcTextareaHeight(minRows = 1)  
设置了默认值为1，minheight = singleRowHeight * minRows;

```javascript
let minHeight = singleRowHeight * minRows;
if (boxSizing === 'border-box') {
  minHeight = minHeight + paddingSize + borderSize;
}
```
- gs

props中minHeight的传值默认为80
```javascript
 if (this.minHeight) {
  opt.minHeight = typeof this.minHeight === 'number' ? `${this.minHeight}px` : this.minHeight;
}
```
