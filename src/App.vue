<template xmlns:v-for="http://www.w3.org/1999/xhtml">
  <div id="app">
    <h1>{{title}}</h1>
    <section id="logInAndSignUp" v-if="!currentUser">
      <div>
        <label><input type="radio" name="type" value="signUp" v-model="actionType">注册</label>
        <label><input type="radio" name="type" value="login" v-model="actionType">登录</label>
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
        title: 'Welcome,ToDoList',
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
      // 将数据储存至leanCloud

      window.onbeforeunload = () => {
        // onbeforeunload,window api 在窗口关闭时执行
        let dataString = JSON.stringify(this.items)
        window.localStorage.setItem('todoList', dataString)

        let AVTodos = AV.Object.extend('todos')
        let avTodos = new AVTodos
        avTodos.set('content', dataString)
        avTodos.save().then(function (todo) {
          // 保存成功后的代码
          console.log('success')
        }, function (error) {
          console.log('fail')
        })
      }

      let oldDataString = window.localStorage.getItem('todoList')
      this.items = JSON.parse(oldDataString) || []

      let oldNewTodoString = window.localStorage.getItem('newTodo')
      this.newTodo = JSON.parse(oldNewTodoString)

      // 定义好currentUser
      this.currentUser = this.getCurrentUser()
    },
    methods: {
      toggleClass(item) {
        // 切换class，标记已完成、未完成
        item.isFinished = !item.isFinished
      },
      createNewList() {
        // 新建列表项，实现增加功能
        this.items.push({
          label: this.newTodo,
          isFinished: false,
          createAt: new Date()
        })
        this.newTodo = ''
      },
      removeItem(item) {
        // 删除列表项，实现删除功能
        let index = this.items.indexOf(item)
        this.items.splice(index, 1)
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
        }, function (error) {
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
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }

  .finished {
    text-decoration: line-through;
  }


</style>
