![image-20191103092029432](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103092029432.png)

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

* v-model 双向数据绑定

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

* jquery

  ![image-20191103111115965](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103111115965.png)

  ![image-20191103111046217](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103111046217.png)

  ![image-20191103111200284](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103111200284.png)

* mvp设计模式

  ![image-20191103111322552](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103111322552.png)

* mvvm设计模式，开发只需关注M和V层，面向数据编程

  ![image-20191103111445797](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191103111445797.png)

* 前端组件化

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

* 子组件向父组件传递数据

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
  				this.list.splice = (index, 1)  // 无效，没删除......
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

  

# Vue项目实战

# 项目上线实战