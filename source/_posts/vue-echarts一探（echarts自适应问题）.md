---
title: vue+echarts一探（echarts自适应问题）
date: 2018-02-27 22:52:37
categories:
- vue
---
要开学了。。。过了个年，感觉又佛系了不少  

背景：最近在做一个项目，用到vue和echart。  
然后就遇到了个问题，echart自适应的问题，我的项目中是有侧边栏的，当侧边栏收起的时候，echart实例已经生成，不会跟随父容器的宽度改变。  
页面布局如下：（md粗糙的表示一下） 

header | header
---|---
侧边栏 | 内容滚动区域

首先百度google，网上大部分是针对window窗口大小的变化而对echart图进行resize。代码如下：

```javascript
window.addEventListener("resize", function () {
    chart.resize();
});
```
这个是针对window窗口大小变化的。而且也存在问题，咱们下面继续讨论。  
以上并不符合我的需求。 
经过我的一番冥思苦想，想到了两种解决方案。因为是刚学ehcart，所以在vue文件中都是比较简单粗暴的给个div，然后直接搞！！！
```html
<div id="chart" ></div>
<script>
    // 在mouted里面初始化
    let chart = echarts.init(document.getElementById('chart'));
    chart.setOption(this.chartData);
</script>
```
如echarts文档中标准，教科书般的代码！（写完出了图我还沾沾自喜）

下面说一下我想到的解决方案：  （侧边栏的收缩我弄了个变量showNav存在vuex中）
####  在util中弄个函数，在每个Echart的init的下面去调用
   util函数传入当前echart实例和vm实例。传入vm实例的目的是想利用watch去监听vuex中的showNav，从而调用chart.resize()对echart图进行重绘。   
   代码如下：
    
```javascript
const chartResize = (chart, _this) => {
	if(chart && _this) {
		window.addEventListener("resize", function () {
			chart.resize();
		});
		_this.$watch("_this.$store.getters['SHOW_NAV']", function(val){
			chart.resize();
		})
	}
}
```
####   将echart写在chart.vue中，在chart.vue中init并将自适应部分写在该组件中，每处需要echart图的时候使用该组件。
（这个其实我感觉是比较合理比较正确的，在chart.vue中去初始化，不应该在每处需要echart都初始化一次，同时对自适应能做到统一处理）。
那么这部分的做法是在chart.vue中监听vuex中的showNav从而调用chart.resize()   
代码如下：(只截取了一部分)

 
```html
<template>
    <div :id="id" ></div>
</template>

<script>
import { mapGetters } from 'vuex';
import util from '../util/util';
import echarts from 'echarts';

export default {
    name: 'chart',
    props: ['id', 'option', 'heightClass', 'group'],
    computed: {
        ...mapGetters({
            showNav: 'GET_SHOW_NAV'
        })
    },
    watch: {
        showNav: function(val) {
            setTimeout(() => {
                this.chart.resize();
            }, 0);
        },
        option: function(val) {
            this.chart.setOption(this.option, true);
        },
        group: function(group) {
            this.chart.group = group;
        }
    },
    methods: {
        init() {
            let chart = echarts.init(document.getElementById(this.id));
            this.chart = chart;
            if (this.group) {
                this.chart.group = this.group;
            }
            setTimeout(() => {
                chart.setOption(this.option, true);
            }, 0);
            util.chartResResize(chart);
            });
        }
    },
    mounted() {
        if (this.option) {
            this.init();
        }
    }
};
</script>

```

后面翻看了[vue-echarts](https://github.com/ecomfe/vue-echarts/blob/master/src/components/ECharts.vue)中的代码，思路也是一致，封装起来后传入option。同时，它的组件Echarts.vue写得挺优秀的，值得学习。我观察到该组件有个prop为auto-resize，于是看了下对resize的处理，与网络上搜索的大致相同。但是有一点细节它处理的比较好，show the code：

```javascript
if (this.autoResize) {
    this.__resizeHandler = debounce(() => {
      chart.resize()
    }, 100, { leading: true })
    addListener(this.$el, this.__resizeHandler)
}
```
此处加上了节流（用了lodash的debounce）  
读者们是否也注意到了呢？window的resize事件我们一般要加上节流函数，防止频繁触发。


以上大致的echarts自适应的一些小小心得。接下来会勤奋一点写博客。总感觉总结和心得的东西一直都有，也有记录在有道云，但是想拿上来发篇博客就感觉不够格。

补充：我想到了第三种方法，接下来会再写一篇，监听父容器的resize事件。看看最后能不能写成vue的plugin的形式分享出来