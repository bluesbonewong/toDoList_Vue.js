<template xmlns:v-for="http://www.w3.org/1999/xhtml">
  <div id="app">
    <h1>{{title}}</h1>
    <section id="logInAndSignUp">
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
        <div class="formRow">用户名<input type="text"></div>
        <div class="formRow">密码<input type="password"></div>
        <div class="formActions"><input type="submit" value="登录"></div>
      </div>
    </section>
    <section id="todo">
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
        }
      }
    },
    created: function () {
      window.onbeforeunload = () => {
        let dataString = JSON.stringify(this.items)
        window.localStorage.setItem('todoList', dataString)

        let newTodoString = JSON.stringify(this.newTodo)
        window.localStorage.setItem('newTodo', newTodoString)
      }

      let oldDataString = window.localStorage.getItem('todoList')
      this.items = JSON.parse(oldDataString) || []

      let oldNewTodoString = window.localStorage.getItem('newTodo')
      this.newTodo = JSON.parse(oldNewTodoString)
    },
    methods: {
      toggleClass(item) {
        item.isFinished = !item.isFinished
      },
      createNewList() {
        this.items.push({
          label: this.newTodo,
          isFinished: false,
          createAt: new Date()
        })
        this.newTodo = ''
      },
      removeItem(item) {
        let index = this.items.indexOf(item)
        this.items.splice(index, 1)
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
