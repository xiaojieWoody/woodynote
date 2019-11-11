# 流程

* 基础内容-基础语法-MVVM模式-组件化
* 生命周期-动画特效
* 实战项目-环境搭建-使用git-数据模拟-本地开发
* 联调-真机测试-上线

# 内容

* Axios：Ajax数据的获取
* Vue Router：多页面之间的路由
* Vuex：各个组件之间的数据共享
* 异步组件：代码上线，性能更优
* Stylus：编写前端样式
* 递归组件：实现组件自己调用自己
* 插件
* 公用组件

# 前提

* js
* ES6
* webpack：项目打包
* npm：包管理工具

# Vue 初探

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Hello World</title>
	<script src='./vue.js'></script>
</head>
<body>
<div id="app">{{content}}</div>
<script>
	var app = new Vue({
		el:'#app',
		data:{
			content: 'Hello World'
		}
	})
	setTimeout(function(){
		app.$data.content='bye world'
	}, 2000)
</script>
</body>
</html>
```

##v-model 双向数据绑定

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>ToDoList</title>
	<script src='./vue.js'></script>
</head>
<body>
<div id="app">
	<input type="text" v-model='inputValue'></input>
	<button v-on:click="handleBtnClick">提交</button>
	<ul>
		<li v-for="item in list">{{item}}</li>
	</ul>
</div>
<script>
	var app = new Vue({
		el:'#app',
		data:{
			list: [],
			inputValue: ''
		},
		methods: {
			handleBtnClick: function() {
				this.list.push(this.inputValue)
				this.inputValue = ''
			}
		}
	})
</script>
</body>
</html>
```

## jquery-todoList

```html
<!DOCTYPE html>
<html>
<head>
	<title>JQuery_todoList</title>
	<script src="../../jquery.js"></script>
</head>
<body>
	<div id="app">
		<input id="input" type="text">
		<button id="btn">提交</button>
		<ul id="list"></ul>
	</div>

	<script>
		function Page() {
		}

		$.extend(Page.prototype, {
			init: function() {
				this.bindEvents();
			},
			bindEvents: function() {
				var btn = $('#btn');
				btn.on('click', $.proxy(this.handleBtnClick, this));
			},
			handleBtnClick: function() {
				var inputElem = $('#input');
				var inputVal = inputElem.val();
				var ulEle = $('#list');
				ulEle.append('<li>' + inputVal + '</li>');
				inputElem.val('');
			}
		})

		var page = new Page();
		page.init();
	</script>
</body>
</html>
```

## mvp设计模式

* M 模型
* V 视图
* P 控制器

## mvvm设计模式

* 开发只需关注M和V层，面向数据编程

![image-20191103111445797](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103111445797.png)

## 前端组件化

```html
<!DOCTYPE html>
<html lang="en">
<head> 
	<meta charset="UTF-8">
	<title>ToDoList</title>
	<script src='./vue.js'></script>
</head>
<body>
<div id="app">
	<input type="text" v-model='inputValue'></input>
	<button @click="handleBtnClick">提交</button>
	<ul>
		<todo-item 
			v-bind:content="item"
			v-for="item in list">
		</todo-item>
	</ul>

</div>
<script>
	// 全局组件
	// Vue.component("TodoItem", {
	// 	props: ['content'],
	// 	template: "<li>{{content}}</li>"
	// })
	// 局部组件
	var TodoItem = {
		props: ['content'],
		template: "<li>{{content}}</li>"
	}

	var app = new Vue({
		el:'#app',
		// 局部组件
		components: {
			TodoItem: TodoItem
		},
		data:{
			list: [],
			inputValue: ''
		},
		methods: {
			handleBtnClick: function() {
				this.list.push(this.inputValue)
				this.inputValue = ''
			}
		}
	})
</script>
</body>
</html>
```

## 子组件向父组件传递数据

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>ToDoList</title>
	<script src='./vue.js'></script>
</head>
<body>
<div id="app">
	<input type="text" v-model='inputValue'></input>
	<button @click="handleBtnClick">提交</button>
	<ul>
		<todo-item 
			:content="item"
			v-bind:index="index"
			v-for="(item,index) in list" 
			@delete="handleItemDelete">
		</todo-item>
	</ul>

</div>
<script>
  // v-on:click 可以替换成 @click
	// v-bind:content="item" 可简写成 :content="item"
	// 局部组件
	var TodoItem = {
		// 子组件接受父组件传递的值
		props: ['content', 'index'],
		template: "<li @click='handleItemClick'>{{content}}</li>",
		methods: {
			handleItemClick: function() {
				// 子组件向外触发事件 ，然后父组件监听子组件 @delete
				this.$emit("delete", this.index);
			}
		}
	}


	var app = new Vue({
		el:'#app',
		// 注册局部组件
		components: {
			TodoItem: TodoItem
		},
		data:{
			list: [],
			inputValue: ''
		},
		methods: {
			handleBtnClick: function() {
				this.list.push(this.inputValue)
				this.inputValue = ''
			},
			handleItemDelete: function(index) {
				alert(index)
				this.list.splice(index, 1)  //删除......
			}
		}
	})
</script>
</body>
</html>
```

# 基础知识

![image-20191103120621666](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103120621666.png)

![image-20191103120820877](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103120820877.png)

* vue实例的生命周期钩子

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<meta charset="UTF-8">
  	<title>Vue实例生命周期函数</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  	<div id="app"></div>
  
  	<script>
  		// 生命周期函数就是vue实例在某一个时间点会自动执行的函数
  		// 参考官网声明周期流程图
  		var vm = new Vue({
  			el: "#app",
  			data: {
  				msg: "hello world!"
  			},
  			template: "<div>{{msg}}</div>",
  			beforeCreate: function() {
  				console.log("beforeCreate");
  			},
  			created: function() {
  				console.log("created");
  			},
  			beforeMount: function() {
  				console.log(this.$el);
  				console.log("beforeMount");
  			},
  			mounted: function() {
  				console.log(this.$el);
  				console.log("mounted");
  			},
  			beforeDestory: function() {
  				console.log("beforeDestory");
  			},
  			destoryed: function() {
  				console.log("destoryed");
  			},
  			beforeUpdate: function() {
  				console.log("beforeUpdate");
  			},
  			updated: function() {
  				console.log("updated");
  			}
  		})
  	</script>
  </body>
  </html>
  ```

* 模版语法

  ```html
  <div id="app">
  	<div>{{name + ' Lee'}}</div>
  	<div v-text="name + 'Lee'"></div>
  	<div v-html="name + 'Jack'"></div>
  </div>
  <script>
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			name: "Woody"
  		}
  	})
  </script>
  ```

* 计算属性，方法和侦听器

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>计算属性</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="app">
  	{{fullName}}
  	{{age}}
  </div>
  <script>
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			firstName: "Dell",
  			lastName: "Lee",
  			// 监听器
  			// fullName: "Dell Lee",
  			age: 18
  		},
  		// 浏览器控制台 vm.$data.firstName='Jack' 调试
  
  		// 监听器
  		// watch: {
  		// 	firstName: function() {
  		// 		console.log("计算了一次");
  		// 		this.fullName = this.firstName + " " + this.lastName;
  		// 	},
  		// 	lastName: function() {
  		// 		console.log("计算了一次");
  		// 		this.fullName = this.firstName + " " + this.lastName;
  		// 	}
  		// }
  
  		// 方法，没有缓存机制
  		// methods: {
  		// 	fullName: function() {
  		// 		console.log("计算了一次");
  		// 		return this.firstName + " " + this.lastName;
  		// 	}
  		// }
  
  		// 推荐
  		// 计算属性（有缓存机制，当依赖的属性值发生变化时才会重新计算一次）
  		// 可在控制台中改变属性值来测试，如从在Console中，vm.firstName="Mike" 就会重新计算一次
  		computed: {
  			// fullName: function() {
  			// 	console.log("计算了一次");
  			// 	return this.firstName + " " + this.lastName
  			// }
  			// getter和setter
  			fullName: {
  				get: function() {
  					return this.firstName + " " + this.lastName;
  				},
  				// 控制台 vm.fullName='Woody fine'
  				set: function(value) {
  					var arr = value.split(" ");
  					this.firstName = arr[0];
  					this.lastName = arr[1];
  				}
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```

* vue中样式绑定

  ```html
  <!DOCTYPE html>
  <html>
<head>
  	<title>Vue中样式绑定</title>
  	<script src="./vue.js"></script>
  	<style>
  		.activated {
  			color: red;
  		}
  	</style>
  </head>
  <body>
  
  <div id="app">
  	<div 
  		@click="handleDivClick"
  		:class='{activated:isActivated}'
  	>
  		Hello World
  	</div>
  </div>
  <script>
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			isActivated: false
  		},
  		methods: {
  			handleDivClick: function() {
  				this.isActivated = !this.isActivated;
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```
  
  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>Vue中样式绑定</title>
  	<script src="./vue.js"></script>
  	<style>
  		.activated {
  			color: red;
  		}
  	</style>
  </head>
  <body>
  
  <div id="app">
  	<div 
  		@click="handleDivClick"
  		:class="[activated, activatedOne]"
  	>
  		Hello World
  	</div>
  </div>
  <script>
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			activated: "",
  			activatedOne: "activated-one"
  		},
  		methods: {
  			handleDivClick: function() {
  				this.activated = this.activated === "activated" ? "" : "activated";
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```
  
  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>Vue中样式绑定</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  
  <div id="app">
  	<div :style="[styleObj, {fontSize: '50px'}]" @click="handleDivClick">
  		Hello World
  	</div>
  </div>
  <script>
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			styleObj: {
  				color: "black"
  			}
  		},
  		methods: {
  			handleDivClick: function() {
  				this.styleObj.color = this.styleObj.color === "black" ? "red" : "black";
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```

* vue中条件渲染

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>Vue中的条件渲染</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="app">
  	<div v-if="show">{{message}}</div>
  	<div v-else>Bye World</div>
  	<div v-show="show">{{message}}</div>
  
  	<div v-if="te ==='a'">This is A</div>
  	<div v-else-if="te === 'b'">This is B </div>
  	<div v-else>This is Others</div>
  </div>
  <script>
  	// v-if为false时，元素不存在DOM上
  	// v-show为false时，元素存在于DOM上
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			show: false,
  			message: "Hello World",
  			te: "a",
  		}
  	})
  </script>
  </body>
  </html>
  ```

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>Vue中的条件渲染</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="app">
  	<div v-if="show">
  		用户名：<input key="username" />
  	</div>
  	<div v-else>
  		邮箱：<input key="email"/>
  	</div>
  </div>
  <script>
  	// 指定key，避免复用bug
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			show: false,
  		}
  	})
  </script>
  </body>
  </html>
  ```

* vue中的列表渲染

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>Vue中的列表渲染</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="app">
  	<div v-for="(item, index) of list"
  		:key="item.id">
  		{{item.text}} --- {{index}}
  	</div>
  
  	<template v-for="(item, index) of list"
  			  :key="item.id">
  			<div>
  				{{item.text}} --- {{index}}
  			</div>
  			<span>
  				{{item.text}}
  			</span>
  	</template>
  </div>
  <script>
  	// 带key值可以提高效率（key值唯一且不是index的值）
  	// 操作数组 push pop shift unshift splice sort reverse
  	// 通过遍历方法操作数组，才能实时显示改变后的数组值，直接通过数组下标去操作不会实时显示改变后的值
  	// 例如控制台操作，替换数组第二个，
  		//vm.list[1]={id:"333", text:"Dell1"}，数组是改变了，但是页面显示没变
  		//vm.list.splice(1,1,{id:"333333", text:"Woody1"})，遍历数组可以改变
  		//vm.list=[{...}],通过改变数组引用，也可以改变
  	// template标签不会真正被渲染到页面上		
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			list: [
  				{
  					id: "01020301",
  					text: "hello"
  				}, {
  					id: "3212313",
  					text: "Woody"
  				}, {
  					id: "343211",
  					text: "Fine"
  				}
  			]
  		}
  	})
  </script>
  </body>
  </html>
  ```

* 对象渲染

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>对象的渲染</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="app">
  	<div v-for="(item, key, index) of userInfo">
  		{{item}} ---- {{key}} ---- {{index}}
  	</div>
  </div>
  <script>
  	// 数据改变，页面实时变化
  	// 增加值，页面实时变化：
  		// 方法1: 改变对象的引用，如 vm.userInfo = {......}
  		// 方法2: Vue.set(vm.userInfo, "address", "beijing") 
  		// 方法3: vm.$set(vm.userInfo, "address", "beijing") 
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			userInfo: {
  				name: "Dell",
  				age: 12,
  				gender: "male",
  				salary: "secret"
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```

  ```html
  <div id="app">
  	<div v-for="(item, index) of userInfo">
  		{{item}}
  	</div>
  </div>
  <script>
  	// 实时显示改变后的值：
  	// Vue.set(vm.userInfo, 1, 10)
  	// vm.$set(vm.userInfo, 3, 1111)
  	var vm = new Vue({
  		el: "#app",
  		data: {
  			userInfo: [1,2,3,4,5]
  		}
  	})
  </script>
  ```

* 组件使用中的细节点

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>组件使用中的细节</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="root">
  	<table>
  		<tbody>
  			<tr is="row"></tr>
  			<tr is="row"></tr>
  			<tr is="row"></tr>
  		</tbody>
  		<ul>
  			<li>
  				<option is="row"></option>
  				<option is="row"></option>
  				<option is="row"></option>
  			</li>
  		</ul>
  	</table>
  
  	<div ref="hello"
  		 @click="handleClick"
  		>
  		hello world
  	</div>
  </div>
  <script>
  	// 直接写 <row>会有问题，用 is=“row“ 就没得问题
  	// 在子组件中，data必须是个函数
  	// 通过ref可以获取DOM节点
  	Vue.component('row', {
  		// template: '<tr><td>this is a row</td></tr>'
  		data: function() {
  			return {
  				content: 'this is a content'
  			}
  		},
  		template: '<tr><td>{{content}}</td></tr>'
  		
  	})
  	var vm = new Vue({
  		el: '#root',
  		methods: {
  			handleClick: function() {
  				alert(this.$refs.hello.innerHTML)
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>组件使用中的细节</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="root">
  	<counter @change="handleChange" ref="one"></counter>
  	<counter @change="handleChange" ref="two"></counter>
  	<div>{{total}}</div>
  </div>
  <script>
  	Vue.component('counter', {
  		template: '<div @click="handleClick">{{number}}</div>',
  		data: function() {
  			return {
  				number : 0
  			}
  		},
  		methods: {
  			handleClick: function() {
  				this.number ++
  				this.$emit('change')
  			}
  		}
  	})
  	var vm = new Vue({
  		el: "#root", 
  		data: {
  			total : 0
  		},
  		methods: {
  			handleChange: function() {
  				this.total = this.$refs.one.number + this.$refs.two.number
  				// console.log(this.$refs.one.number)
  				// console.log(this.$refs.two.number)
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```

* 父子组件的数据传递

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>父子组件数据传递</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="root">
  	<counter :count="3" @incr="handleIncrease"></counter>
  	<counter :count="2" @incr="handleIncrease"></counter>
  	<div>{{total}}</div>
  </div>
  <script>
  	// :count="3" 表示是数字3 ， count=“3”，表示是字符串3
  	// 子组件不能改变父组件传递的数据，单向数据流规定
  	// 子组件通过事件向父组件传递值
  	var counter = {
  		// 接收父组件传递的内容
  		props: ['count'],
  		// 子组件的data一定是个函数
  		data: function() {
  			return {
  				// 父组件的值赋值给子组件的data
  				number: this.count
  			}
  		},
  		template: '<div @click="handleClick">{{number}}</div>',
  		methods: {
  			handleClick: function() {
  				this.number = this.number + 2;
  				// 每次改变2
  				this.$emit('incr', 2)
  			}
  		}
  	}
  
  	var vm = new Vue({
  		el: '#root',
  		data: {
  			total: 5
  		},
  		// 注册子组件
  		components: {
  			counter: counter
  		},
  		methods: {
  			handleIncrease: function(step) {
  				this.total += step
  			}
  		}
  	})
  </script>
  </body>
  </html>
  ```

* 组件参数校验与非props特性

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>组件参数校验与非Props特性</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="root">
  	<child content="hello "></child>
  </div>
  <script>
  	// 传递数字 :content="123"
  	// 传递字符串： content="123"
  	// 子组件有相同的变量接收父组件传递的值——props：不会在DOM标签中显示属性值；子组件可以使用父组件传递的值
  	// 子组件没有相同的变量接收父组件传递的值——非props：会在DOM标签中显示属性值；且子组件无法使用该值
  	Vue.component('child', {
  		// props: ['content'],
  		props: {
  			// 子组件接收的父组件的值必须是字符串类型
  			// content: String
  			// content: Number
  			// content: [Number, String]
  			content: {
  				type: String,
  				// required: true,
  				// default: 'default value'
  				validator: function(value) {
  					return (value.length > 5)
  				}
  			}
  		},
  		template: '<div>{{content}}</div>'
  	})
  
  	var vm = new Vue({
  		el: '#root'
  	})
  </script>
  </body>
  </html>
  ```

* 给组件绑定原生事件

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>给组件绑定原生事件</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="root">
  	<child @click="handleChildClick"></child>
  </div>
  <script>
  	// 3. <child>中监听自定义事件
  	Vue.component('child', {
  		// 监听原生事件
  		// 1. 子组件被点击触发click事件
  		template: '<div @click="handleChildClick">Child</div>',
  		methods: {
  			handleChildClick: function() {
  				//  2. 触发自定义事件
  				this.$emit('click')
  			}
  		}
  	})
  
  	var vm = new Vue({
  		el: "#root",
  		methods: {
  			handleChildClick: function() {
  				// 4. 被执行
  				alert('click')
  			}
  		}
  	})	
  </script>
  </body>
  </html>
  ```

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  	<title>给组件绑定原生事件</title>
  	<script src="./vue.js"></script>
  </head>
  <body>
  <div id="root">
  	<child @click.native="handleChildClick"></child>
  </div>
  <script>
  	// @click.native 监听原生事件
  	Vue.component('child', {
  		template: '<div @click="handleChildClick">Child</div>'
  	})
  
  	var vm = new Vue({
  		el: "#root",
  		methods: {
  			handleChildClick: function() {
  				alert('click')
  			}
  		}
  	})	
  </script>
  </body>
  </html>
  ```

* 非父子组件间的传值

  ![image-20191106075258546](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191106075258546.png)

  * 使用总线机制

    ```html
    <!DOCTYPE html>
    <html>
    <head>
    	<title>非父子组件间传值(Bus/总线/发布订阅模式/观察者模式)</title>
    	<script src="./vue.js"></script>
    </head>
    <body>
    <div id="root">
    	<child content="Dell"></child>
    	<child content="Lee"></child>
    </div>
    <script>
    	
    	// 总线
    	Vue.prototype.bus = new Vue()
    
    	Vue.component('child', {
    		data: function() {
    			// 避免修改父组件的值
    			return {
    				selcContent: this.content
    			}
    		},
    		props: {
    			content: String
    		},
    		template: '<div @click="handleClick">{{selcContent}}</div>',
    		methods: {
    			handleClick: function() {
    				// alert(this.content)
    				this.bus.$emit('change', this.selcContent)
    			}
    		},
    		mounted: function() {
    			var this_ = this
    			this.bus.$on('change', function(msg) {
    				// alert(msg)  // 弹出两次相同的内容
    				this_.selcContent = msg
    			})
    		}
    	})
    
    	var vm = new Vue({
    		el: "#root"
    	})	
    </script>
    </body>
    </html>
    ```

* 在Vue中使用插槽

  

# Vue项目实战

# 项目上线实战