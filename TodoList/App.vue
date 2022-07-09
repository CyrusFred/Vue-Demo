<template>
  <div id="root">
    <div class="todo-container">
      <div class="todo-wrap">
        <MyHeader @addTodo="addTodo" />
        <MyList :todoList="todoList" />
        <MyFooter :todoList="todoList" @checkAllTodo="checkAllTodo" @clearAllTodo="clearAllTodo" />
      </div>
    </div>
  </div>
</template>

<script>
import pubsub from 'pubsub-js'
import MyHeader from './components/MyHeader'
import MyList from './components/MyList'
import MyFooter from './components/MyFooter'

export default {
  name: 'App',
  components: { MyHeader, MyList, MyFooter },
  data () {
    return {
      // 原始数据模板
      // todoList: [ 
      //    { id: '001', title: '吃饭', done: false },
      //    { id: '002', title: '喝酒', done: true },
      //    { id: '003', title: '抽烟', done: false },
      // ],
      todoList: JSON.parse(localStorage.getItem('todoList')) || []
    }
  },
  methods: {
    // 添加一个todo
    addTodo (todoObj) {
      // console.log('我是app组件，我收到了数据：', x)
      this.todoList.unshift(todoObj)
    },
    // 勾选or取消勾选一个todo
    checkTodo (id) {
      this.todoList.forEach((todo) => {
        if (todo.id === id) todo.done = !todo.done
      })
    },
    // 更新一个todo
    updateTodo (id, title) {
      this.todoList.forEach((todo) => {
        if (todo.id === id) todo.title = title
      })
    },
    // 删除一个todo
    deleteTodo (_, id) {
      this.todoList = this.todoList.filter(todo => todo.id !== id)
    },
    // 全选或者取消全选
    checkAllTodo (done) {
      this.todoList.forEach(todo => todo.done = done)
    },
    // 清除所有已经完成的todo
    clearAllTodo () {
      this.todoList = this.todoList.filter((todo) => {
        return !todo.done
      })
    }
  },
  watch: {
    todoList: {
      deep: true,
      handler (value) {
        localStorage.setItem('todoList', JSON.stringify(value))
      }
    }
  },
  mounted () {
    this.$bus.$on('checkTodo', this.checkTodo)
    this.$bus.$on('updateTodo', this.updateTodo)
    // this.$bus.$on('deleteTodo', this.deleteTodo) // 全局事件总线写的
    // Vue开发者工具不支持第三方库 看不出来deleteTodo事件  只支持全局事件总线和最初的子传父 父传子方法
    this.pubId = pubsub.subscribe('deleteTodo', this.deleteTodo)
  },
  beforeDestroy () {
    this.$bus.$off('checkTodo')
    this.$bus.$off('updateTodo')
    // this.$bus.$off('deleteTodo') //事件总线写法
    pubsub.unsubscribe(this.pubId)
  }
}
</script>

<style>
/*base*/
body {
  background: #fff;
}


.btn {
  display: inline-block;
  padding: 4px 12px;
  margin-bottom: 0;
  font-size: 14px;
  line-height: 20px;
  text-align: center;
  vertical-align: middle;
  cursor: pointer;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.2),
    0 1px 2px rgba(0, 0, 0, 0.05);
  border-radius: 4px;
}

.btn-danger {
  color: #fff;
  background-color: #da4f49;
  border: 1px solid #bd362f;
}

.btn-danger:hover {
  color: #fff;
  background-color: #bd362f;
}

.btn-edit {
  color: #fff;
  background-color: rgb(41, 41, 241);
  border: 1px solid rgb(63, 109, 244);
}

.btn:focus {
  outline: none;
}

.todo-container {
  width: 600px;
  margin: 0 auto;
}


.todo-container .todo-wrap {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}
</style>
