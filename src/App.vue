<template xmlns:v-for="http://www.w3.org/1999/xhtml">
	<div id="app">
		<h1>{{title}}<span>{{title2}}</span></h1>
		<section id="logInAndSignUp" v-if="!currentUser">
			<div class="wrapper">
				<div class="tab-title">
					<label class="active"><input type="radio" name="type" value="signUp" v-model="actionType"
					                             @click="toggleTabClass">注册</label>
					<label><input type="radio" name="type" value="login" v-model="actionType" @click="toggleTabClass">登录</label>
				</div>
				<div class="signUp" v-if="actionType === 'signUp'">
					<form v-on:submit.prevent=signUp>
						<div class="formRow">用户名<input type="text" v-model="formData.userName"></div>
						<div class="formRow">密码<input type="password" v-model="formData.password"></div>
						<div class="formActions"><input type="submit" value="注册"></div>
					</form>
				</div>
				<div class="login" v-if="actionType === 'login'">
					<form v-on:submit.prevent=login>
						<div class="formRow">用户名<input type="text" v-model="formData.userName"></div>
						<div class="formRow">密码<input type="password" v-model="formData.password"></div>
						<div class="formActions"><input type="submit" value="登录"></div>
					</form>
				</div>
			</div>
		</section>
		<section id="todo" v-if="currentUser">
			<div>
				<button @click="logout">登出</button>
			</div>
			<div class="newTask">
				<input type="text" v-model="newTodo" @keyup.enter="createNewList">
			</div>
			<ul>
				<li v-for="item in items" :class="{finished:item.isFinished}" v-on:click="toggleClass(item)">
					{{item.label}}<input type="checkbox" id="checkbox" v-model="item.isFinished">
					<p v-if="item.isFinished">{{item.createAt}}</p>
					<p v-else>{{item.createAt}}</p>
					<button v-on:click="removeItem(item)">X</button>
				</li>
			</ul>
		</section>

	</div>
</template>

<script>
	import AV from 'leancloud-storage'

	let APP_ID = 'C8YAmtPqP1udTULP5RfRbRcR-gzGzoHsz'
	let APP_KEY = 'PcW6DladugIs35VrRljo6im5'
	AV.init({
		appId: APP_ID,
		appKey: APP_KEY,
	})

	export default {
		data() {
			return {
				title: 'Welcome,',
				title2: 'ToDoList',
				isActive: true,
				items: [],
				newTodo: '',
				checked: false,
				actionType: 'signUp',
				formData: {
					userName: '',
					password: ''
				},
				currentUser: ''
			}
		},
		created: function () {
			// 定义好currentUser
			this.currentUser = this.getCurrentUser()
			this.fetchTodos()
		},
		methods: {
			toggleTabClass(e) {
				// <label><input>注册</label> 这种形式
				// 如果在在label上放置click事件，只点击label的话
				// 触发第一次click事件，然后label由于和input相关联，相当于input也被点击了一次，input导致了事件冒泡
				// 然后再次触发label上的click事件，所以click事件被执行了两回

				// 通过input的冒泡，去切换tab的class active
				for (let i = 0; i < e.target.parentElement.parentElement.children.length; i++) {
					e.target.parentElement.parentElement.children[i].classList.remove('active')
				}
				e.target.parentElement.classList.add('active')
			},
			toggleClass(item) {
				// 切换class，标记已完成、未完成
				item.isFinished = !item.isFinished
				console.log(1)
			},
			fetchTodos() {
				// 获取 User 的 todos

				// 1.用户刷新页面，但没有退出
				// 2.用户登出，再登录
				if (this.currentUser) {
					let query = new AV.Query('todos')
					query.find().then((todos) => {
						console.log(todos)

						let realTodos = todos[0] // todos是一个数组，取第一项，因为我们只更新第一项
						let id = realTodos.id // 在leancloud，每一个数据都有一个ObjectId
						this.items = JSON.parse(realTodos.attributes.content) // 把todo的内容替换当前items的内容
						this.items.id = id
						console.log(this.items)
					}, function (error) {
						console.log(error)
					})
				}
			},
			updateTodos() {
				let dataString = JSON.stringify(this.items)
				// 更新数据，需要知道数据具体的ObjectId
				let avTodos = AV.Object.createWithoutData('todos', this.items.id)
				avTodos.set('content', dataString)
				avTodos.save().then(() => {
					console.log('更新成功')
				})
			},
			saveTodos() {
				let dataString = JSON.stringify(this.items)
				let AVTodos = AV.Object.extend('todos')
				let avTodos = new AVTodos

				let acl = new AV.ACL()
				acl.setReadAccess(AV.User.current(), true) // 只有当前用户能读
				acl.setWriteAccess(AV.User.current(), true) // 只有当前用户能写

				avTodos.set('content', dataString)

				avTodos.setACL(acl) // 设置访问控制

				avTodos.save().then((todo) => {
					// 保存成功后的代码
					this.items.id = todo.id // 保存成功后，说明数据库有了数据，一定把id加上，否则不会执行update
					console.log('save：', todo)
				}, function (error) {
					console.log('fail')
				})
			},
			saveOrUpdateTodos() {
				if (this.items.id) {
					// 已有数据记录，则更新
					this.updateTodos()
				} else {
					// 没有数据记录，则保存
					this.saveTodos()
				}
			},
			createNewList() {
				// 新建列表项，实现增加功能
				this.items.push({
					label: this.newTodo,
					isFinished: false,
					createAt: new Date()
				})
				this.newTodo = ''
				this.saveOrUpdateTodos()
			},
			removeItem(item) {
				// 删除列表项，实现删除功能
				let index = this.items.indexOf(item)
				this.items.splice(index, 1)
				this.saveOrUpdateTodos()
			},
			signUp() {
				// leancloud 注册

				// 新建 AVUser 对象实例
				let user = new AV.User()
				// 设置用户名
				user.setUsername(this.formData.userName)
				// 设置密码
				user.setPassword(this.formData.password)
				user.signUp().then((loginedUser) => {
					this.currentUser = this.getCurrentUser()
				}, function (error) {
					alert('注册失败')
				})
			},
			login() {
				// leancloud 登录
				AV.User.logIn(this.formData.userName, this.formData.password).then((loginedUser) => {
					this.currentUser = this.getCurrentUser()
					this.fetchTodos()
				}, function (error) {
					console.log(error)
				})
			},
			logout() {
				// leancloud 登出
				AV.User.logOut()
				this.currentUser = null
				window.location.reload()
			},
			getCurrentUser() {
				// leancloud 获取当前用户

				// 先做判断，当前用户是否已经登录，若已登录，直接进入TODO页面
				// 否则，一旦刷新，每次都要重新登录
				// 这个判断主要是为了 created 中的 定义currentUser 服务
				let current = AV.User.current()
				if (current) {
					let {id, createAt, attributes: {username}} = current
					return {id, createAt, username}
				} else {
					return null
				}
			}
		}
	}
</script>

<style>
	* {
		margin: 0;
		padding: 0;
	}

	ul,
	li {
		list-style: none;
	}

	#app {
		font-family: 'Avenir', Helvetica, Arial, sans-serif, onsolas, Monaco, 'Andale Mono', 'Ubuntu Mono', monospace;
		-webkit-font-smoothing: antialiased;
		-moz-osx-font-smoothing: grayscale;
		text-align: center;
		color: #2c3e50;
		width: 520px;
		margin: 60px auto;
		box-shadow: 5px 5px 25px 0 rgba(46, 61, 73, .2);
	}

	#app > h1 {
		padding: 20px 0;
		font-size: 50px;
	}

	#app > h1 > span {
		color: #E23150;
	}

	#logInAndSignUp {
		/*background: #FAFBFC;*/
		border-bottom-left-radius: 10px;
		border-bottom-right-radius: 10px;
	}

	#logInAndSignUp > .wrapper {
		box-sizing: border-box;
		width: 100%;
		margin: 0 auto;
	}

	#logInAndSignUp > .wrapper > .tab-title {
		display: flex;
		justify-content: space-around;
		font-size: 20px;
	}

	#logInAndSignUp > .wrapper > .tab-title > label {
		display: block;
		flex-grow: 1;
		text-align: center;
		padding: 20px;
		/*border-bottom: 1px solid #dbe2e8;*/
		/*background: #FAFBFC;*/
		cursor: pointer;
		line-height: 29px;
		transition: all .5s;
	}

	#logInAndSignUp > .wrapper > .tab-title > label:hover {
		color: #E23150;
	}

	#logInAndSignUp > .wrapper > .tab-title > label.active {
		color: #E23150;
	}

	#logInAndSignUp > .wrapper > .tab-title > label.active:after {
		content: '';
		display: block;
		width: 100%;
		height: 2px;
		background: #E23150;
		margin-top: 6px;
	}

	/*#logInAndSignUp > .wrapper > .tab-title > label:first-child {*/
	/*border-right: 1px solid #dbe2e8;*/
	/*}*/

	#logInAndSignUp > .wrapper > div > label > input {
		display: none;
	}

	.finished {
		text-decoration: line-through;
	}


</style>
