---
title: vuex-初体验
date: 2018-01-13 16:56:02
tags:
categories:
---

一个vue项目的开发刚刚结束，其中有许多地方用到了组件之间的通信，虽有props，eventbus可以解决，但是趁这段时间了解了一下vuex的实现，感觉还是很方便的，这里我就向大家介绍一下vuex的使用。

<!-- more -->

vuex按照官方的建议是用在大型的单页面项目中，这种项目公共的数据会多一些，管理起来很方便，小型项目就没必要了，因为虽然 Vuex 可以帮助我们管理共享状态，但也附带了更多的概念和框架。如果项目比较小，使用props，$on与$emit完全可以解决。好了，步入正题！

使用vue-cli初始化一个vue项目。
```
vue init webpack demo
```
这样一个vue就初始化完成了，但是这个项目是不包含vuex的，那我们怎么将vuex集成到项目中呢？

首先需要将vuex安装的项目中
```
npm install vuex --save
```
然后在src目录下创建store文件夹，里面创建index.js

vuex的使用必须显式地通过 Vue.use() 来安装

在index.js中添加如下代码
```javascript
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)
```
然后将store注册到根实例下（main.js） 该store实例会注入到根组件下的所有子组件中，且子组件能通过 this.$store 访问到。

```javascript
new Vue({
    el: '#app',
    router,
    store,
    components: { App },
    template: '<App/>'
})
```
到这里，vuex就安装到项目中了。接下来就开始介绍vuex的使用。

vuex一共5个重要属性，分别是state，getter，mutation，action，module。

### State

State可以理解为一个全局的data属性，所有页面都可以访问。

在store/index.js中注册store实例
```javascript
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)

const store = new Vuex.Store({
    state: {
        storeMsg: 'from store'
    }
});
export default store
```

在组件中可以通过 `this.$store.state` 来获取到state对象，我们在helloWorld.vue文件中获取一下
```javascript
export default {
  data() {
      return {
          msg: this.$store.state.storeMsg
      }
  }
};
```
渲染就如之前那样用就好
```html
 <div>
    {{msg}}
 </div>
```

当然，我们最好使用官方推荐的写法，这里用到一个辅助函数 `mapState`
```javascript
import { mapState } from "vuex";
export default {
  data() {
    return {
      msg: this.$store.state.storeMsg
    };
  },
  computed: {
    ...mapState({
      message: "storeMsg"
      // message: state => state.storeMsg
    })
  }
};
```

### getter

getter是用来过滤数据的，比如我们多个页面都需要用到同一个过滤规则，这时getter就派上用场了，下面的这个例子是将一群人中的男性筛选出来，并渲染到列表中


```javascript
//  store/index.js
const store = new Vuex.Store({
  state: {
    storeMsg: 'from store',
    gender: [
      { sex: 0, name: '张三'},
      { sex: 1, name: '小花'},
      { sex: 1, name: '小丽'},
      { sex: 0, name: '李四'}
    ]
  },
  getters: {
    filterGender: (state) => (sex) => {
            return state.gender.filter(gender => gender.sex == sex)
        }
  }
})
```

```html
<!-- helloWorld.vue -->
<template>
    <div>
        <p>{{msg}}</p>
        <p>{{message}}</p>
        <p v-for="item in filterGender(0)">
            <span v-text="item.name"></span>
        </p>
    </div>
</template>

<script>
import { mapState, mapGetters } from "vuex";
export default {
  data() {
    return {
      msg: this.$store.state.storeMsg,
    };
  },
  computed: {
    ...mapState({
      message: "storeMsg"
      //message: state => state.storeMsg
    }),
    ...mapGetters(["filterGender"])
  }
};
</script>
```

### mutation

可以将mutation理解事件，用来改变store中的状态，当你想要提交一个mutation时需要以store.commit的方式来调用或者使用辅助函数 `mapMutations`
我们先来注册一个mutation

```javascript
// store/index.js

const store = new Vuex.Store({
    state: {
        count: 0,
    },
    mutations: {
        addCount(state, obj) {
            state.count += obj.num
        }
    }
});
```

以this.$store.commit提交mutation的方式有两种。
一种是以载荷（官方说法）的方式提交

```html
<!-- helloWorld.vue -->

<template>
    <div>
       <span>{{count}}</span>
       <button @click="add">+2</button>
    </div>    
</template>

<script>
import { mapState, mapGetters, mapMutations } from "vuex";
export default {
  methods: {
    add() {
      this.$store.commit("addCount", { num: 2 });
    }
  }
};
</script>
```

另一种是以对象的方式提交

```html
<script>
import { mapState, mapGetters, mapMutations } from "vuex";
export default {
  methods: {
    add() {
      this.$store.commit({ type: "addCount", num: 2 });
    }
  }
};
</script>
```

使用辅助函数 `mapMutations`
```html
<!-- helloWorld.vue -->

<template>
    <div>
       <span>{{count}}</span>
       <button @click="addCount({num:1})">+1</button>
    </div>    
</template>

<script>
import { mapState, mapGetters, mapMutations } from "vuex";
export default {
  methods: {
    ...mapMutations(["addCount"])
  }
};
</script>
```

### action

action 的存在是为了处理异步操作，因为在mutation中必须是同步函数，具体的介绍官方有详细的解释，这里就不多说了。

Action 类似于 mutation，不同的在于：
  * Action 提交的是 mutation，而不是直接变更状态。
  * Action 可以包含任意异步操作。

我们来注册一个action
```javascript
// store/index.js

    mutations: {
        addCount(state, obj) {
            state.count += obj.num
        },
        act(state) {
            state.count += 1
            alert(state.count)
        }
    },
    actions: {
        carryCount({ commit }) {
            setTimeout(function() {
                commit('act')
            }, 1000)
        }
    }
```

```html
<span @click="carryCount">action</span>

<script>
import { mapState, mapGetters, mapMutations, mapActions} from "vuex";
export default {
  methods: {
    ...mapActions(['carryCount'])
  }
};
</script>

```

### module

如果项目过大，store中的state，mutations等可能会有很多，导致store显得相当臃肿，module就是解决这个问题的。

在store下新建moduleA.js
```javascript
const moduleA = {
    namespaced: true,
    state: {},
    mutations: {},
    getters: {},
    actions: {}
}
export default moduleA;

```

namespaced-命名空间，为true会解决不同module命名冲突的问题。

在store/index.js中
```javascript
import moduleA from './moduleA'

const store = new Vuex.Store({
    modules: {
        moduleA
    }
})
```

在组件中使用的时候和之前基本一样，注意的是在辅助函数中，需要将module的属性名加上，区分module，如下：

```javascript
<script>
export default {
    computed: {
        ...mapState('moduleA',{
            count: "count"
        }),
    },
    methods: {
        ...mapMutations('moduleA',["addCount"]),
    }
};
</script>
```

