---
title: nextTick
date: 2019-07-06 14:02:55
tags:
---

```javascript
if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
    timerFunc = () => {
      setImmediate(nextTickHandler)
    }
  } else if (typeof MessageChannel !== 'undefined' && (
    isNative(MessageChannel) ||
    // PhantomJS
    MessageChannel.toString() === '[object MessageChannelConstructor]'
  )) {
    const channel = new MessageChannel()
    const port = channel.port2
    channel.port1.onmessage = nextTickHandler
    timerFunc = () => {
      port.postMessage(1)
    }
  } else
  /* istanbul ignore next */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    // use microtask in non-DOM environments, e.g. Weex
    const p = Promise.resolve()
    timerFunc = () => {
      p.then(nextTickHandler)
    }
  } else {
    // fallback to setTimeout
    timerFunc = () => {
      setTimeout(nextTickHandler, 0)
    }
  }
```
优先定义setImmediate、MessageChannel为什么要优先用他们创建macroTask而不是setTimeout？
HTML5中规定setTimeout的最小时间延迟是4ms，也就是说理想环境下异步回调最快也是4ms才能触发。Vue使用这么多函数来模拟异步任务，其目的只有一个，就是让回调异步且尽早调用。而MessageChannel 和 setImmediate 的延迟明显是小于setTimeout的