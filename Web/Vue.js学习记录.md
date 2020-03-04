# Vue.js学习记录

最近在学习Vue.js，特此写一篇博客记录，避免看过只是看过，好记性不如烂笔头~

## 模板语法

## 插值

插值表达式，可以将Vue对象的data数据中的message数据，插值到字符串中。

- 变量插值

```
<div id="app">
  <p>{{ message }}</p>
</div>

<script>
new Vue({
  el: '#app',
  data: {
    message: '菜鸟教程'
  }
})
```

- 表达式插值

```
<div id="app">
    {{5 + 5}}<br>
    {{ ok ? 'YES' : 'NO' }}<br>
    {{ message.split('').reverse().join('') }}
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    ok: true,
    message: 'RUNOOB'
  }
})
</script>
```

## 指令

Vue指令，指的是v-前缀的属性。下面介绍Vue常用的指令：

#### v-html（Html内容渲染）

如果想在插值表达式中插入html代码进行渲染，使用插值是不可以的，会将html代表作为字符串显示，如果想进行html渲染，则需要使用v-html指令，将message变量交给v-html指令处理。

```
<div id="app">
    <div v-html="message"></div>
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    message: '<h1>菜鸟教程</h1>'
  }
})
</script>
```

#### v-if、v-else 条件渲染

v-if、v-else 条件指令，可以让我们使用if-else逻辑动态显示内容。下面是一个是否登录，动态显示已登录、未登录的例子。

```
<div id="app">
    <p v-if="isLogin">您好，管理员</p>
    <p v-else>您好，您还没有登录喔</p>
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    isLogin: true
  }
})
</script>
```

#### v-else-if、v-else指令

只有v-if、v-else指令还是不能满足我们的多种判断场景，所以后面Vue添加了v-else-if、v-else指令，补充了多种判断的条件渲染。例如以下例子，不同的用户类型，登录成功后，显示不同的提示语。

```
<div id="app">
    <p v-if="userType ==='admin'">您好，管理员</p>
    <p v-else-if="userType === 'vip'">您好，vip用户，今天也要元气满满喔</p>
    <p v-else-if="userType === 'host'">您好，xxx主播，马上开播吧</p>
    <p v-else>您好，xxx用户</p>
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    //用户类型
    userType : "admin"
  }
})
</script>
```

#### v-show指令

v-show和v-if、v-else指令类似，都可以做到条件渲染，但是v-show更加适合频繁切换的场景，v-show指令为更改元素的display属性来达到显示、隐藏，而v-if、v-else指令为DOM操作，并不适合频繁地切换，适合比较少的场景。例如我们使用v-show改造已登录、未登录提示语（毕竟用户不会频繁切换登录！）

```
<div id="app">
    <p v-show="isLogin">您好，管理员</p>
    <p v-else>您好，您还没有登录喔</p>
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    isLogin: true
  }
})
</script>
```

#### v-for 列表遍历

列表标签可以说是最常用的元素了，当列表的数量不定，由后端数据决定时，怎样都是不可能写死列表的个数的，所以v-for指令就来解决这个问题了。下面用2个例子演示v-for指令。

## 遍历数组内的元素

- 直接遍历

```
<div id="app">
    <div v-for="item in list">{{ item }}</div>
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            list: [1, 2, 3, 4, 5]
        }
    })
</script>
```

- 如果需要获取数据的位置角标，则需要在v-for中进行改动，将原有的v-for="item in list"，改为v-for="(index, item) in list"。index为角标索引。

```
<div id="app">
    <div v-for="(index, item) in list">{{ item }}</div>
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            list: [1, 2, 3, 4, 5]
        }
    })
</script>
```

#### 遍历数组对象

一般开发数据可不止1个，所以一般遍历的都是数组中的对象。如果列表的数据量非常大，性能消耗会比较大。而Vue提供对列表进行复用优化，而需要使用这个优化则需要给列表添加一个key，必须让key的属性唯一，例如v-bind:key="item.id"，由于id是唯一、不会重复，推荐都加上key。

```
<div id="app">
    <div v-for="item in list" v-bind:key="item.id">{{ item.name }}</div>
</div>
<script>
    new Vue({
        el: '#app',
        data: {
            list: [
                   { id : '1', name: '北京'},
                   { id : '2', name: '上海'},
                   { id : '3', name: '广州'}
            ]
        }
    })
</script>
```

#### v-bind（属性绑定）

- 绑定属性


- 绑定样式

例如消息已读操作，当勾选了已读的CheckBox，文字颜色从黑色变为灰色。使用v-bind指令，绑定.read样式，当isRead为true时使用，false则恢复原本样式。

```
<head>
	<style>
		.read {
		  color: #CCC
		}
	</style>
</head>

<body>
	<div id="app">
		已读<input type="checkbox" v-model="isRead">
		<br>
		<div v-bind:class="{'read': isRead}">
		    您有一条新通知...
		</div>
	</div>
<body/>
    
<script>
	new Vue({
	    el: '#app',
	    data:{
	      //是否已读状态
	      isRead: false
	    }
	});
</script>
```

- 绑定表达式

遍历城市列表，每个div的id为城市id。

```
<div id="app">
    <div v-for="item in list" v-bind:id="item.id">{{ item }}</div>
</div>

<script>
new Vue({
  el: '#app',
  data: {
            list: [
                   {id : '1', name: '北京'},
                   {id : '2', name: '上海'},
                   {id : '3', name: '广州'}
            ]
   }
})
</script>
```

#### v-model（双向绑定）

v-model指令，能让数据双向绑定。下面是input输入框和文字双向绑定的例子，当输入框内容改变时，文字会相应同步文字。

```
<div id="app">
    <p>{{ message }}</p>
    <input v-model="message">
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    message: 'Hi!'
  }
})
</script>
```

#### v-on 事件处理

- 点击事件

```
//完整写法
<a v-on:click="onHandleClick">百度一下</a>
//缩写
<a @click="onHandleClick">百度一下</a>
```

#### 组件

组件为Vue强大的特性之一，可以将HTML元素进行重复利用，封装可重用的代码，让我们代码模块化。

- 全局注册

全局注册，项目中全局可用，但是组件名不能重复，否则会覆盖。而且注册全局组件，如果你使用webpack来构建，即使这个组件你不使用，也会包含在最终的构建中，无疑增加用户无味的下载流量。

全局注册语法：Vue.component(targetName，options);

例如下面的内容组件，内容组件名为content，在options对象的template属性中添加组件内容。

```
<div id="app">
	<content></content>
</div>

<script>
//注册全局组件
Vue.component('content', {
  template: '<h1>自定义内容组件!</h1>'
})

new Vue({
  el: '#app'
})
</script>
```

- 局部注册

给指定的Vue实例注册组件，多个Vue实例之间是独立的。需要在Vue实例中，添加components属性，进行注册组件，格式为：组件名：模板对象。

```
<div id="app">
	<my_component></my_component>
</div>

<script>
//声明组件内容模板
var MyComponent = {
  template: '<h1>自定义内容组件!</h1>'
}

new Vue({
  el: '#app',
  components: {
    //暂时不能在子组件中使用，需要在子组件中引入
    'my_component': MyComponent
  }
})
</script>
```

- 让局部注册的子组件使用自定义组件

上面的自定义组件，是不能在子组件中使用的，如果需要在子组件使用，则需要2步：

1）引入组件
2）注册组件

```
//子组件

//引入组件
import MyComponent from './MyComponent.vue'

export default {
  components: {
    //局部注册组件
    MyComponent
  },
  // ...
}
```

- 给组件传递参数

一般封装组件，都会将变化的参数提取出来，让使用方传入。组件申明时，options对象中，添加一个props属性，属性值为一个数组，填入参数名，template模板中使用参数名。最后在使用处添加属性值即可。

```
<div id="app">
	<child message="Hi!"></child>
</div>

<script>
//局部注册
Vue.component('child', {
  //声明props，标识模板需要使用的变量
  props: ['message'],
  //模板中使用传入的参数
  template: '<span>{{ message }}</span>'
})

new Vue({
  el: '#app'
})
```

- 动态设置参数

上面我们给组件提供了自定义属性，让使用方设置数据，但是一般数据为后台数据动态设置，而不会直接写死在标签属性上，而是动态设置数据。我们可以通过v-bind指令，让父组件上的数据，和自定义组件参数绑定。如下例，v-bind:message="parentMsg"，指定为父组件data中的parentMsg。

添加一个输入框，使用v-model指令，双向绑定parentMsg到输入框的内容中，实现输入框和自定义组件数据同步。

注意：props为单向绑定，当父组件的数据更新时，子组件的自定义属性会自动更新。但是反过来不行，就是子组件更新数据，父组件不会更新。

```
<div id="app">
	<div>
	  <input v-model="parentMsg">
	  <br>
	  <child v-bind:message="parentMsg"></child>
	</div>
</div>

<script>
//局部注册
Vue.component('child', {
  props: ['message'],
  template: '<span>{{ message }}</span>'
})
// 创建根实例
new Vue({
  el: '#app',
  data: {
	parentMsg: '父组件内容'
  }
})
</script>
```

- 自定义事件，子组件发送数据给父组件

上面讲到，通过props可以从父组件传递数据到子组件，而props设计为单向流，子组件无法修改数据更新父组件的数据，如果需要从子组件发送数据给父组件更新时，则需要自定义事件，让父组件进行更新。

如下面的例子，自定义一个数字自增组件button-counter，点击一下按钮，更新父组件中的数字文字。

**注意：** 子组件的data必须传入一个function方法，才能每个实例都独立一份数据拷贝，否则会共享数据，导致实例数据互相影响。

```
<div id="app">
	<div id="counter-event-example">
	  <p>{{ total }}</p>
	  <!-- 6、使用自定义组件，通过v-on:观察自定义组件，发出的自定义事件 -->
	  <button-counter v-on:increment="incrementTotal"></button-counter>
	  <button-counter v-on:increment="incrementTotal"></button-counter>
	</div>
</div>

<script>
//1、局部注册组件
Vue.component('button-counter', {
  //模板
  template: '<button v-on:click="incrementHandler">{{ counter }}</button>',
  //3、定义传入子组件的数据，data必须是一个函数，否则多个组件实例会共享数据，传入方法则不会
  data: function () {
    return {
      count: 0
    }
  },
  methods: {
    //4、自定义组件内部，定义点击回调
    incrementHandler: function () {
      //自增
      this.counter += 1
      //5、发送自定义事件，父组件接收到后进行处理（事件在组件使用时使用v-on观察事件）
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    //2、定义父组件数据，计数总数
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
```

# 未完待续...