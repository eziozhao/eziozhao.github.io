---
title: 2020年还不懂vue生命周期？一文带你理解vue生命周期那些事
tags:
  - vue
abbrlink: a439326f
date: 2020-09-13 23:18:56
---
# 前言
所谓生命周期，就是vue实例从创建到销毁所经历的一系列过程，掌握生命周期能帮我们更好的了解vue的设计思想，也更方便debug。

vue为我们提供了一系列钩子函数，方便我们在各阶段进行操作，本文也先从这些钩子函数说起。
<!--more-->
# Demo
我们创建一对父子组件，并通过观察子组件的创建过程，来了解生命周期的变化。

父组件
```html
<template>
    <div id="app">
        <header>我是父组件</header>
        <button @click="show=true">点我创建子组件</button>
        <br/>
        <child v-if="show" proptest="属性值"></child>
    </div>
</template>

<script>
    import child from "@/components/child";

    export default {
        name: 'App',
        data() {
            return {
                show: false
            }
        },
        components: {
            child
        },

    }
</script>

<style>
    #app{
        text-align: center;
        background-color: aquamarine;
    }
</style>

```
子组件
```html
<template>
    <div style="background-color: #6495ed">
        <header>我是子组件</header>
        <span>{{year}}</span>
        <button @click="changeTime">时光机</button>
        <br/>
        <button @click="destroy">点我销毁子组件</button>
    </div>
</template>

<script>
    export default {
        name: "child",
        data() {
            return {
                year: '现在是2020年'
            }
        },
        props: ['proptest'],
        methods: {
            changeTime() {
                this.year === '大人，时代变了' ? this.year = '现在是2020年' : this.year = '大人，时代变了'
            },
            methodsAvailable() {

            },
            destroy(){
                this.$destroy()
            }
        },
        computed: {
            time() {
                return Date.now()
            }
        },
        beforeCreate() {
            console.log('------------start-------------')
            console.log('当前是beforeCreate')
            console.log('this', this)
            console.log('el', this.$el)
            console.log('data', this.$data)
            console.log('prop', this.$props)
            console.log('watch', this._watchers)
            console.log('methods', this.methodsAvailable)
            console.log('computed', this.time)
            console.log('year', this.year)
            console.log('------------end-------------')
            console.log('')
        },
        created() {
            console.log('------------start-------------')
            console.log('当前是created')
            console.log('el', this.$el)
            console.log('data', this.$data)
            console.log('prop', this.$props)
            console.log('watch', this._watchers)
            console.log('methods', this.methodsAvailable)
            console.log('computed', this.time)
            console.log('year', this.year)
            console.log('------------end-------------')
            console.log('')
        },
        beforeMount() {
            console.log('------------start-------------')
            console.log('当前是beforeMount')
            console.log('el', this.$el)
            console.log('data', this.$data)
            console.log('year', this.year)
            console.log('------------end-------------')
            console.log('')
        },
        mounted() {
            console.log('------------start-------------')
            console.log('当前是mounted')
            console.log('el', this.$el)
            console.log('data', this.$data)
            console.log('year', this.year)
            console.log('------------end-------------')
            console.log('')

        },
        beforeUpdate() {
            console.log('------------start-------------')
            console.log('当前是beforeUpdate')
            console.log('el', this.$el.innerHTML)
            console.log('data', JSON.stringify(this.$data))
            console.log('year', this.year)
            console.log('------------end-------------')
            console.log('')
        },
        updated() {
            console.log('------------start-------------')
            console.log('当前是updated')
            console.log('el', this.$el.innerHTML)
            console.log('data', JSON.stringify(this.$data))
            console.log('year', this.year)
            console.log('------------end-------------')
            console.log('')
        },
        beforeDestroy() {
            console.log('------------start-------------')
            console.log('当前是beforeDestroy')
            console.log('el', this.$el)
            console.log('data', this.$data)
            console.log('year', this.year)
            console.log('------------end-------------')
            console.log('')
        },
        destroyed() {
            console.log('------------start-------------')
            console.log('当前是destroyed')
            console.log('el', this.$el)
            console.log('data', this.$data)
            console.log('year', this.year)
            console.log('watch', this._watchers)
            console.log('methods', this.methodsAvailable)
            console.log('computed', this.time)
            console.log('------------end-------------')
            console.log('')
        }
    }
</script>

<style scoped></style>

```
代码运行后浏览器显示如下

![wBFJnU.png](https://s1.ax1x.com/2020/09/13/wBFJnU.png)

# 实践
 点击创建子组件按钮，观察控制台打印的信息

## beforeCreate

![w0XygH.png](https://s1.ax1x.com/2020/09/13/w0XygH.png)

这一阶段，this绑定到了当前实例，但是组件还没有绑定属性，el,data,prop等都处于undefined状态

## created

![w0j9xJ.png](https://s1.ax1x.com/2020/09/13/w0j9xJ.png)

此时data,prop都已绑定，watch开始侦听，computed计算属性生效，methods中的方法已经可以调用。然而，挂载阶段还没开始，$el目前尚不可用

## beforeMount

![w0v9Ff.png](https://s1.ax1x.com/2020/09/13/w0v9Ff.png)

准备开始挂载，相关的 render 函数首次被调用，此时$el仍是不可用。至于什么是render函数，可以参考官方这个例子 https://cn.vuejs.org/v2/guide/render-function.html

## mounted
![w0xFN6.png](https://s1.ax1x.com/2020/09/13/w0xFN6.png)

挂载完成，此时$el已经可用。页面显示如下

![wBFA6f.png](https://s1.ax1x.com/2020/09/13/wBFA6f.png)


以上几个钩子函数中，created和mounted估计是平时用的最多的，created中可以进行加入数据的初始化逻辑，但是此时还不能操作dom节点。mounted中已经可以操作dom节点，而且eventbus监听一般也放在mounted中。

注意：如果child组件中还有子组件，那么在child中如果想操作子组件的dom,需要将代码写在nextTick中

那么当data中的数据更新时，又会发生什么呢？我们点击```时光机```按钮

## beforeUpdate
![wBp9Bj.png](https://s1.ax1x.com/2020/09/13/wBp9Bj.png)

可以看到此时data中的值已经改变，但是$el仍未改变。

## updated
![wBpECV.png](https://s1.ax1x.com/2020/09/13/wBpECV.png)

此时$el的也已经更新。

从官方文档中也可以看出beforeUpdate发生在虚拟DOM重新渲染和打补丁之前，所以个人理解beforeUpdate也许叫beforeDOMUpdate更形象。

![wBplU1.png](https://s1.ax1x.com/2020/09/13/wBplU1.png)

最后，我们点击```销毁```按钮

## beforeDestroy
![wBiIw4.png](https://s1.ax1x.com/2020/09/13/wBiIw4.png)

此时实例依然完全可用，各属性都可调用

## destroyed

![wBE7DK.png](https://s1.ax1x.com/2020/09/13/wBE7DK.png)

此时实例已经销毁。清理了与其它实例的连接，解绑它的全部指令及事件监听器。

但是vm.$destroyed()并不会清除浏览器上的dom。所以更推荐在父组件中直接通过 v-if 来控制子组件的销毁。

此时虽然有打印值，但是可以看到watch实际上已经是非激活状态，此时再点击界面上的```时光机```按钮,页面也不再变化。

自此，vue生命周期过程已经全部介绍完了，如果发现文中有错误，欢迎交流。