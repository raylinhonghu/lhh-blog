---

title: Learn Vue By Example

category: Vue

date: 2020-08-02

author: 林鸿鹄

---

## VUE instance 


```
<div id="vue-app">
	<h1> {{name}} </h1>
</div>
})
```

vue 会生成一个模版然后绑定数据，然后再创建渲染到页面。

```
new Vue({
	el: '#vue-app',
	data: {
		name: 'Shaun'
	}
})
```


## Data && Methods

```
new Vue({
	el: '#vue-app',
	data: {
		name: 'Shaun'
	}，
	methods: {
		greeting: function(time) {
			return `good ${time} ${this.name}`
		}
	}
})
```
可以直接在模版里使用方法, data 里的数据会被 prop 到 instance 上所以从 this 关键字里就可以拿到。

```
<div id="vue-app">
	<h1> {{greeting('afternoon')}} </h1>
</div>
})
```

## Data-binding

```
new Vue({
	el: '#vue-app',
	data: {
		name: 'Shaun',
		website: 'http://www.baidu.com',
		link: '<a :href="http://www.baidu.com"> baidu </a>'
		
	}
})
```

```
<div id="vue-app">
	<a v-bind:href="website"> baidu </a>
	<a :href="website"> baidu </a>
	<input type="text" :value="name"/>
	<p v-html="link"></p>
</div>
})
```

## Event

```
new Vue({
	el: '#vue-app',
	data: {
		age:23,
		x:0,
		y:0,
	},
	methods: {
		add: function(inc) {
			this.age += inc
		},
		updateXY: function(event) {
			this.x = event.offsetX;
			this.y = event.offsetY;
		}
	}
})
```

```
<div id="vue-app">
	<button v-on:click="add(1)"> Add </button>
	<button @click="age--"> Add </button>
	<button v-on:dblclick="add(10)"> Add </button>
	<p> My age is {{age}} </p>
	
	<div id='canvas' v-on:mousemover="updateXY">{{x}},{{y}}</div>
</div>
})
```

## Event Modifier 

```
new Vue({
	el: '#vue-app',
	data: {
		age:23,
		x:0,
		y:0,
	},
	methods: {
		click: function() {
			alert('you clicked');
		}
	}
})
```


```
<div id="vue-app">
	<button v-on:click.once="add(1)"> Add </button>
	
	<a v-on:click.prevent href="http://www.baidu.com"/>
</div>
})
```

## Keyboard Events

```
<div id="vue-app">
	<label> Name: </label>
	<input type="text" v-on:keyup.alt.enter="logName"/>
</div>
})
```

## Two-way Data Binding

```
new Vue({
	el: '#vue-app',
	data: {
		name: '1112'
	}
})

```

```
<div id="vue-app">
	<label> Name: </label>
	<input type="text" v-model="name"/>
</div>
```

## Computed Property
监听 variables， variables 变了才会触发。

```
new Vue({
	el: '#vue-app',
	data: {
		age: 20,
		a: 1,
		b: 2
	}，
	computed: {
		addToA: function() {
			return this.age + this.a;
		}
	}
})

```

```
<div id="vue-app">
	<div>{{a}}</div>
	<div>{{addToA}}</div>
	
	<button v-on:click="a++"> add A</button>
</div>
```

## Dynamic CSS Classes


```
new Vue({
	el: '#vue-app',
	data: {
		available: false,
	}，
	computed: {
		addClass: function() {
			return {
				available: this.available
			};
		}
	}
})

```

```
<div id="vue-app">
	<button v-on:click="available = !available"> Change </button> 
	<div v-bind:class="addClass"></div>
</div>
```

## Conditionals 

```
<div id="vue-app">
	<div v-if="error"> error </div>
	<div v-else-if="success"> success </div>
	<div v-show="error"> error </div>
	<div v-show="success"> success </div>
</div>
```

## Looping


```
new Vue({
	el: '#vue-app',
	data: {
		ninjas: [
			{name:'ray',age: 23},
			{name:'james',age: 33},
			{name:'roy',age: 25},
		]
	}
})

```

```
<div id="vue-app">
	<template v-for="ninja in ninjas">
		<div v-for="(val,key) in ninja">
			<h1> {{key}} - {{val}} </h1> 
		<div> 
	<template>
	
</div>
```

## Multiple Vue instances


```
const one = new Vue({
	el: '#vue-app1',
	data: {
		title:'titleone'
	}
})

const two = new Vue({
	el: '#vue-app2',
	data: {
		title: '555'
	}.
	methods: {
		changeTitleOne: function() {
			one.title = 'title one changed'
		}
	}
})

two.title = "666"

```

## Components 

```
Vue.component('greeting',{
	template: '<div on-click="changeName"> {{name}} </div>',
	data: function() {
		return {
			name: 'yoshi'
		}
	},
	methods: {
		changeName: function() {
			this.name = 'Mario'
		}
	}
})

```

```
<greeting/>
```

## Referencing 

```
const a = new Vue({
 el: '...',
 data: {
  output: 'apple'
 },
 methods:{
 	readRefs: function(){
 		this.output = this.$refs.input.value
 	}
 }
```

 
```
<input type="text" ref="input"/>
<button v-on:click="readRefs"> Submit </button> 
<div> your fav food {{output}} </div>
```

## Css Scoping

```
<style scoped>
</style>
```

## Event Bus

```bus.$emit```

## Props

```this.$emit```

## Life Cycle

- beforeCreate
- created // http
- beforeMount （rare use）
- mounted

- 这里之后页面才展示出来

- beforeUpdate
- updated


- beforeDestroy
- destroyed

## Slots
就像 react 的children 但是可以命名

## Dynamic Component

```
<keep-alive>
	<component v-bind:is="component"></component>
</keep-alive>
```

## Input Binding

## Custom Directive 

```
Vue.directive('rainbow', {
	bind(el, binding, vnode){
		if(bind.value == true){
			el.style.color = "#" + Math.random().toString().slice(2,8)
		}
	}
})
```

## Filters 

```
Vue.filter('to-uppercase', function(value){
	return value.toUpperCase();
})
```

## Mixins
mixins: []



