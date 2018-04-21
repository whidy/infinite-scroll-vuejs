# Vue下滚动到页面底部无限加载数据Demo

> 看到一篇[Implementing an Infinite Scroll with Vue.js](https://alligator.io/vuejs/implementing-infinite-scroll/), 觉得挺实用的就看了下, 顺便简单翻译了一下给需要的人参考.
>
> **从这个项目中可以加深对Vue的生命周期的理解, 何时开始axios请求, 如何结合Vue使用原生js来写scroll事件等等**, 我这里主要是对原文的重点提取和补充

## 本文技术要点

* `Vue`生命周期
* `axios`简单用法
* `moment.js`格式化日期
* 图片懒加载
* 结合原生js来写`scroll`事件
* 请求节流

## 创建项目

首先创建一个简单的vue项目

```bash
# vue init webpack-simple infinite-scroll-vuejs
```

然后安装项目所需要用的一些插件

```bash
# npm install axios moment
```

## 初始化用户数据

项目搭建完后, 跑起来看看

```bash
# npm run dev
```

打开`http://localhost:8080`无误后, 我们开始修改代码, 先看看获取用户数据这块,

```html
<script>
import axios from 'axios'
import moment from 'moment'
export default {
  name: 'app',
  // 创建一个存放用户数据的数组
  data() {
    return {
      persons: []
    }
  },
  methods: {
    // axios请求接口获取数据
    getInitialUsers() {
      for (var i = 0; i < 5; i++) {
        axios.get(`https://randomuser.me/api/`).then(response => {
          this.persons.push(response.data.results[0])
        })
      }
    },
    formatDate(date) {
      if (date) {
        return moment(String(date)).format('MM/DD/YYYY')
      }
    },
  beforeMount() {
    // 在页面挂载前就发起请求
    this.getInitialUsers()
  }
}
</script>
```

> 这里原作者也专门提醒, **完全没有必要也不建议在加载页面的时候请求5次, 只是这个接口一次只能返回1条数据, 仅用于测试才这样做的.** 当然我这里也可以通过Mock来模拟数据, 就不详细说了~

接下来来写模板结构和样式, 如下:

```html
<template>
  <div id="app">
    <h1>Random User</h1>
    <div class="person" v-for="(person, index) in persons" :key="index">
      <div class="left">
        <img :src="person.picture.large" alt="">
      </div>
      <div class="right">
        <p>{{ person.name.first}} {{ person.name.last }}</p>
        <ul>
          <li>
            <strong>Birthday:</strong> {{ formatDate(person.dob)}}
          </li>
          <div class="text-capitalize">
            <strong>Location:</strong> {{ person.location.city}}, {{ person.location.state }}
          </div>
        </ul>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.person {
  background: #ccc;
  border-radius: 2px;
  width: 20%;
  margin: 0 auto 15px auto;
  padding: 15px;

  img {
    width: 100%;
    height: auto;
    border-radius: 2px;
  }

  p:first-child {
    text-transform: capitalize;
    font-size: 2rem;
    font-weight: 900;
  }

  .text-capitalize {
    text-transform: capitalize;
  }
}
</style>
```

这样页面就能显示5个人的个人信息了.

## 处理无限滚动加载逻辑

我们接下来需要在`methods`里面添加`scroll()`来监听滚动, 并且这个事件是应该在`mounted()`这个生命周期内的. 代码如下:

```html
<script>
  // ...
  methods: {
    // ...
    scroll(person) {
      let isLoading = false
      window.onscroll = () => {
        // 距离底部200px时加载一次
        let bottomOfWindow = document.documentElement.offsetHeight - document.documentElement.scrollTop - window.innerHeight <= 200
        if (bottomOfWindow && isLoading == false) {
          isLoading = true
          axios.get(`https://randomuser.me/api/`).then(response => {
            person.push(response.data.results[0])
            isLoading = false
          })
        }
      }
    }
  },
  mounted() {
    this.scroll(this.persons)
  }
}
</script>
```

> 这段代码原文是有一点拼写错误的. 我这里修正了, 另外增加了距离底部即开始加载数据, 并进行截流.

保存好, 回到浏览器, 查看效果, 已经没有问题了~

## 总结

滚动到页面底部无限加载的功能在Vue上实现其实和普通的页面开发差不多, 每次滚动加载未完成的情况下不会触发请求下一次, 每次请求`push`到数组内, 通过`<img :src="">`的数据绑定实现了懒加载(其实0 0我不太认可...), 看完是不是感觉挺简单的.

最后, 我把这个也弄了一份在GitHub上面, 有需要的可以看看~

---

## 项目运行

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```