---
title: '讲一下angular脏检测'
date: 2017-10-16 22:48:26
categories:
- angular
---
## 脏检测的简单实现
angular的脏检测应该是众所周知了，看一下下面这个图，社区某个页面scope绑定的值。我们通过以下代码就可以访问到作用域中绑定的值。

```
angular.element('.content').scope()

```
![image](../../../../upload/ngWatchers.jpg)

假设你在一个ng-click指令对应的handler函数中更改了scope中的一条数据，此时AngularJS会自动地通过调用$digest()来触发一轮$digest循环。

ng只有在指定事件触发后，才进入$digest cycle：

- DOM事件，譬如用户输入文本，点击按钮等。(ng-click)
- XHR响应事件 ($http)
- 浏览器Location变更事件 ($location)
- Timer事件($timeout, $interval)
- 执行$digest()或$apply()

在$digest流程中，Angular将遍历每个数据变量的watcher，比较它的新旧值。当新旧值不同时，触发listener函数，执行相关的操作。

**需要注意的是**，$digest循环不会只运行一次。在当前的一次循环结束后，它会再执行一次循环用来检查是否有models发生了变化。它用来处理在listener函数被执行时可能引起的model变化。因此，$digest循环会持续运行直到model不再发生变化，或者$digest循环的次数达到了10次（好像是叫做TTL）因此，尽可能地不要在listener函数中修改model。

下面是简单双向绑定的实现：

（为了方便理解，我把改变视图的函数写在$digest中，实际上应该分开写）

```javascript
class scope{
        constructor(){
            this.$$watcher=[];                          //监听器
        }
        $watch(name,exp,listener){
            this.$$watcher.push({
                name:name,                              //数据变量名
                last:'',                                //数据变量旧值
                newVal:exp,                             //返回数据变量新值的函数
                listener:listener || function(){}       //监听回调函数
            })
        }
        $digest(){
            let bindList=document.querySelectorAll('[ng-bind]');
            let dirty= true;
            while(dirty){
                dirty=false;
                for(let i=0;i<this.$$watcher.length;i++){
                    let newVal=this.$$watcher[i].newVal();
                    let oldVal=this.$$watcher[i].last;
                    if(newVal!==oldVal && !isNaN(newVal) && !isNaN(oldVal)){
                        dirty=true;//遍历两次，确保model不再变化
                        this.$$watcher[i].listener(newVal,oldVal);//执行监听函数
                        this.$$watcher[i].last=newVal;
                        //下面这一段是改变view
                        for (let j = 0; j < bindList.length; j++) {
                            let modelName=bindList[j].getAttribute("ng-bind");
                            if(modelName==this.$$watcher[i].name) {
                                if (bindList[j].tagName == "INPUT") {
                                    bindList[j].value = this.$$watcher[i].newVal();
                                }
                                else {
                                    bindList[j].innerHTML = this.$$watcher[i].newVal()
                                }
                            }
                        }
                    }
                }
            }
        }
    }
```
*以上是对$watch和$digset的简单实现，并没有考虑数组和对象*

下面这一部分做了三件事
1. 将scope绑定的值添加到$$watcher中
2. 监听ng-click和input事件，触发$digest()
3. 默认执行一次$digest
```javascript
    window.addEventListener('onload',function(){
        let $scope=new scope();
        $scope.count=0;
        $scope.wmx=123;
        $scope.increment=function(){this.count++};
        
        //监听ng-click事件，触发$digest循环（使用es6的let）
        let bindList=document.querySelectorAll('[ng-click]');
        for(let i=0;i<bindList.length;i++){
            bindList[i].onclick=function(){
                $scope[bindList[i].getAttribute("ng-click")]();
                $scope.$digest(); 
            }
        }
        
        //监听input事件，触发$digest循环（使用闭包）
        let inputList=document.querySelectorAll("input[ng-bind]");         
        for(var i=0;i<inputList.length;i++){
            inputList[i].addEventListener("input",(function(index){
                return function(){
                    $scope[inputList[index].getAttribute("ng-bind")]=inputList[index].value;
                    $scope.$digest();
                }
            })(i));
        }
        
        //以下这段是将scope绑定的值添加到$$watchers
        //newVal必需设置为函数。如果是数据变量，那么它的新值将一直等于创建监听器时绑定的
        //值，而实际上数据的值是在不断变化的。使用函数便能在每次调用时返回它的最新值。
        for(let key in $scope){
            if(key!="$$watcher" && typeof $scope[key]!="function") { 
                $scope.$watch(key, function () {
                    return $scope[key];
                })
            }
        }
        //默认触发一次digest（）
        $scope.$digest();
    })
```
看一下效果：[链接](../../../../upload/dirty.html)

有一点很有趣，angular是进入$digest cycle，等待所有model都稳定后，才批量一次性更新UI。
这种机制能减少浏览器repaint次数，从而提高性能。
