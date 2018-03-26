---
title: javascript中防抖和节流函数
date: 2017-11-15 21:08:46
categories:
- javascript
---
##### 这次来讲一下防抖和节流函数
好久没有写博客hhhh，最近有两门考试TAT.

后面坚持一周一篇吧hhhh觉得写博客也是很有趣的事。

讲一下防抖和节流的应用场景先吧。刚入门前端的同学可能对这个概念还不是很理解。

**节流**：就是像节约水流一样需要我们间隔一段时间开水龙头（控制函数调用频率）

**防抖**：就是对于连续的事件响应我们只需要执行一次回调

++注：给 scroll 加了 debounce（防抖）后，只有用户停止滚动后，才会判断是否到了页面底部（连续触发只执行一次回调）；如果是 throttle（节流） 的话，只要页面滚动就会间隔一段时间判断一次++

### 节流（throttle）
###### 函数节流有哪些应用场景？哪些时候我们需要间隔一定时间触发回调来控制函数调用频率？

- DOM 元素的拖拽功能实现（mousemove）
- 射击游戏的 mousedown/keydown 事件（单位时间只能发射一颗子弹）
- 计算鼠标移动的距离（mousemove）
- Canvas 模拟画板功能（mousemove）
- 监听滚动事件判断是否到页面底部自动加载更多



下面我们用js代码来实现一下throttle(下面借鉴了undersocre，我自己加了点小改进嘻嘻)



```javascript
 /* options的默认值
    *默认有头有尾
    *如果 options 参数传入 {leading: false}
    *那么不会马上触发（等待 wait milliseconds 后第一次触发 func）
    *如果 options 参数传入 {trailing: false}
    *那么最后一次回调不会被触发
    *Notice: options 不能同时设置 leading 和 trailing 为 false**
*/
let throttle=(func,wait,options={})=>{
	let timeout,context,args,result;
	let previous = 0;//上一次执行回调的时间戳
	let later=()=>{
	    // 如果 options.leading === false
        // 则每次触发回调后将 previous 置为 0
        // 否则置为当前时间戳
		previous=options.leading===false? 0:new Date();
		timeout=null;
		result=func.apply(context,args);
		if(!timeout) context = args = null
	}
	let throttled=function(){
	// 第一次执行回调（此时 previous 为 0，之后 previous 值为上一次时间戳）
    // 并且如果程序设定第一个回调不是立即执行的（options.leading === false）
    // 则将 previous 值（表示上次执行的时间戳）设为 now 的时间戳（第一次触发时）
    // 表示刚执行过，这次就不用执行了
	    let now=new Date();
	    if (!previous && options.leading === false) previous = now;
	    let remaining = wait-(now-previous);
	    context=this;
	    args=arguments;
	    
	    if(reamaining<=0 || remaining>wait){
	        if(timeout){
	            clearTimeout(timeout);
	            timeout=null;
	        }
	        previous=now;
	        result=func.apply(context,args);
	        if(!timeout) context=args=null;
	    }else if(!timeout && options.trailing !== false){
	     // 如果已经存在一个定时器，则不会进入该 if 分支
      // 如果 {trailing: false}，即最后一次不需要触发了，也不会进入这个分支
      // 间隔 remaining milliseconds 后触发 later 方法
	        timeout = setTimeout(later,remaining);
	    }
	    
	    return result;
	}
	return throttled;
	
}
```
为什么要remaining>wait呢？这是一个有趣的点：

**原来
    remaining > wait，表示客户端系统时间被调整过
     则马上执行 func 函数**

### 防抖（debounce）
###### 函数去抖有哪些应用场景？哪些时候对于连续的事件响应我们只需要执行一次回调？
- 每次 resize/scroll 触发统计事件
- 文本输入的验证（连续输入文字后发送 AJAX 请求进行验证，验证一次就好）

js高程里面有这么一段简单实现
```javascript
function debounce(method, context) {
     clearTimeout(methor.tId);
     method.tId = setTimeout(function(){
         method.call(context);
     }， 100);
 }
```
那么我们可以对他进行一点改进
```javascript
function debounce(fn, delay) {
  var ctx;
  var args;
  var timer = null;

  var later = function () {
    fn.apply(ctx, args);
    // 当事件真正执行后，清空定时器
    timer = null;
  };

  return function () {
    ctx = this;
    args = arguments;
    // 当持续触发事件时，若发现事件触发的定时器已设置时，则清除之前的定时器
    if (timer) {
      clearTimeout(timer);
      timer = null;
    }
    
    // 重新设置事件触发的定时器
    timer = setTimeout(later, delay);
  };
}
```

*注：在这里用闭包的形式来提供timer，防止它污染全局作用域*

总结：根据实际业务场景，合理的利用debounce（防抖）和throttle（节流）可以优化性能和提高用户体验

这里推荐一下一篇好文，对函数节流进行了性能优化方面的测试[AlloyTeam](http://www.alloyteam.com/2012/11/javascript-throttle/)