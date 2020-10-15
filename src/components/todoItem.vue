<template>
  <div>
    <alert></alert>
    <div>
      <div class="input-group mb-3">
        <label for="item-note" class="position-absolute text-secondary" style="top: 6px; left: 15px; z-index: 9;"><i class="fas fa-pencil-alt"></i></label>
        <input id="item-note" type="text" class="form-control pl-5" placeholder="Add Note" v-model="newTodo" @click="addTodos" v-focus>
      </div>
      <div :class="{'input-fadein' : isAdd }" class="fade-setting">
        <div class="input-group mb-3">
          <label for="item-copmment" class="position-absolute text-secondary" style="top: 6px; left: 15px; z-index: 9;"><i class="far fa-comment-dots"></i></label>
          <input id="item-copmment" type="text" class="form-control pl-5" placeholder="Comment" v-model="newTodoComment">
        </div>
        <div v-if="isAdd" class="d-flex justify-content-between">
          <calendar @passPickDate="pickTaskDay" :passDateData="pickDateData">
            <span class="btn btn-primary bg-white border-0 text-secondary" role="button">
              <i class="far fa-calendar-check mr-md-2"></i>
              {{pickDateData.monthNum | monthStr}} {{pickDateData.date | dateNumberFix}}
            </span>
          </calendar>
          <div>
            <button type="button" class="btn btn-outline-secondary" @click="cancelAddTodos">Cancel</button>
            <button type="button" class="btn btn-primary ml-1" @click="updateTodos">Update Task</button>
          </div>
        </div>
      </div>
    </div>

    <hr>
    <ul class="nav nav-tabs" v-if="todos.length > 0">
      <li class="nav-item">
        <a class="nav-link text-primary" href="#"
        :class="{active: visibility == 'all'}"
        @click.prevent="visibility = 'all'">All Task</a>
      </li>
      <li class="nav-item">
        <a class="nav-link text-primary" href="#"
        :class="{active: visibility == 'inProgress'}"
        @click.prevent="visibility = 'inProgress'">In Progress</a>
      </li>
      <li class="nav-item">
        <a class="nav-link text-primary" href="#"
        :class="{active: visibility == 'completed'}"
        @click.prevent="visibility = 'completed'">Completed</a>
      </li>
    </ul>

    <p class="mt-3 text-secondary text-center" v-if="filterTodos.length === 0 && todos.length > 0">{{`No to-do items matching the filter criteria`}}</p>
    <p class="mt-3 h5 text-primary text-center" v-if="todos.length === 0">{{`Your Key To Successful Move, Lets Get Listed !`}}</p>
    <div class="card mb-1" v-for="(item, key) in filterTodos" :key="key"
    :class="{'task-completed':item.completed, 'text-secondary':item.completed}">
      <div class="card-body">
        <h5 class="card-title" v-if="cacheTodo.id !== item.id">
          <span role="button" @click="updateStared(item)"><i class="far fa-star text-warning" :class="{'fas':item.isStared}"></i></span>
          {{item.todo}}
        </h5>
        <p class="card-text" v-if="cacheTodo.id !== item.id">{{item.comment}}</p>
        <div class="d-flex justify-content-between" v-if="cacheTodo.id !== item.id">
          <div>
            <em>
              <span><i class="far fa-calendar-check"></i></span>
              {{item.todoDate.monthNum | monthStr}} {{item.todoDate.date | dateNumberFix}} , {{item.todoDate.year}}
            </em>
          </div>
          <div class="btn-group btn-group-sm" role="group" v-if="cacheTodo.id !== item.id">
            <button type="button" class="btn btn-outline-secondary" @click="removeTodos(item)"><i class="fas fa-trash-alt"></i></button>
            <button type="button" class="btn btn-outline-primary" @click="editTodos(item)"><i class="fas fa-edit"></i></button>
            <button type="button" class="btn" :class="{'btn-primary':item.completed, 'btn-outline-primary': !item.completed}"
            @click="completedTodos(item)"><i class="far fa-check-circle"></i></button>
          </div>
        </div>
        <!-- 編輯欄位 -->
        <div v-if="cacheTodo.id === item.id">
          <div class="input-group mb-3">
            <input type="text" class="form-control" v-model="cacheTitle" @keyup.esc="cancelEdit">
          </div>
          <div class="input-group mb-3">
            <input type="text" class="form-control form-control-sm" v-model="cacheComment"
            placeholder="Comment" @keyup.esc="cancelEdit">
          </div>
        </div>
        <div class="d-flex justify-content-between" v-if="cacheTodo.id === item.id">
          <calendar @passPickDate="pickTaskDay" :passDateData="cacheDateData">
            <span class="d-inline-block text-center p-1 border text-secondary">
              <i class="far fa-calendar-check mr-md-2"></i>
              {{cacheDateData.monthNum | monthStr}} {{cacheDateData.date | dateNumberFix}} , {{cacheDateData.year}}
            </span>
          </calendar>
          <div>
            <button type="button" class="btn btn-outline-secondary" @click="cancelEdit">Cancel</button>
            <button type="button" class="btn btn-primary ml-1" @click="doneEdit(item)">Done</button>
          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import calendar from './calendar.vue'
import alert from './AlertMessage'

export default {
  components: {
    calendar,
    alert
  },
  data () {
    return {
      visibility: 'all',
      isAdd: false,
      pickDateData: {
        date: '',
        monthNum: '',
        year: ''
      },
      newTodo: '',
      newTodoComment: '',
      todos: [],
      cacheTodo: {},
      cacheTitle: '',
      cacheComment: '',
      cacheDateData: {}
    }
  },
  methods: {
    addTodos () {
      if (!this.isAdd) {
        const date = new Date()
        this.pickDateData.date = date.getDate()
        this.pickDateData.monthNum = date.getMonth() + 1
        this.pickDateData.year = date.getFullYear()
      }
      this.isAdd = true
      this.cancelEdit()
    },
    pickTaskDay (value) {
      // 存取 $emit 回傳的參數
      this.pickDateData = Object.assign({}, value)
      this.cacheDateData = Object.assign({}, value)
    },
    updateTodos () {
      const value = this.newTodo.trim()
      if (!value) {
        this.$bus.$emit('pushMsg', 'Please enter the title of the to-do item !', 'danger')
        return
      }
      const timeId = Math.floor(Date.now())
      const todoDate = this.pickDateData
      const todoData = {
        id: timeId,
        todo: value,
        comment: this.newTodoComment.trim(),
        completed: false,
        isStared: false,
        todoDate
      }
      this.todos.push(todoData)
      this.newTodo = ''
      this.newTodoComment = ''
      this.isAdd = false
      this.updateTodosData()
      this.getTodosData()
      this.$bus.$emit('pushMsg', 'Update completed !', 'warning')
    },
    cancelAddTodos () {
      this.newTodo = ''
      this.newTodoComment = ''
      this.isAdd = false
    },
    updateTodosData () {
      localStorage.setItem('data', JSON.stringify(this.todos))
    },
    getTodosData () {
      this.todos = JSON.parse(localStorage.getItem('data')) || []
    },
    completedTodos (keyItem) {
      let keyIndex = ''
      this.todos.forEach(function (item, index) {
        if (keyItem.id === item.id) {
          keyIndex = index
        }
      })
      this.todos[keyIndex].completed = !this.todos[keyIndex].completed
      this.updateTodosData()
      this.getTodosData()
      if (this.todos[keyIndex].completed) {
        this.$bus.$emit('pushMsg', 'Congratulations for completing a task !', 'warning')
      }
    },
    editTodos (edditItem) {
      // cacheTodo 物件與 {}空物件賦值，進行編輯狀態切換
      this.cacheTodo = edditItem
      this.cacheTitle = edditItem.todo
      this.cacheComment = edditItem.comment
      this.cacheDateData = Object.assign({}, edditItem.todoDate)
      this.cancelAddTodos()
    },
    cancelEdit () {
      this.cacheTodo = {}
    },
    doneEdit (item) {
      item.todo = this.cacheTitle
      item.comment = this.cacheComment
      item.todoDate = this.pickDateData
      this.cacheTodo = {}
      this.updateTodosData()
      this.getTodosData()
      this.$bus.$emit('pushMsg', 'Update completed !', 'warning')
    },
    removeTodos (keyItem) {
      let keyIndex = ''
      this.todos.forEach(function (item, index) {
        if (keyItem.id === item.id) {
          keyIndex = index
        }
      })
      this.todos.splice(keyIndex, 1)
      this.updateTodosData()
      this.getTodosData()
      this.$bus.$emit('pushMsg', 'To-do item removed !', 'secondary')
    },
    updateStared (item) {
      item.isStared = !item.isStared
      this.updateTodosData()
    }
  },
  computed: {
    filterTodos () {
      let filterTodos = []
      if (this.visibility === 'all') {
        filterTodos = this.todos
      } else if (this.visibility === 'inProgress') {
        this.todos.forEach(function (item) {
          if (item.completed === false) {
            filterTodos.push(item)
          }
        })
      } else if (this.visibility === 'completed') {
        this.todos.forEach(function (item) {
          if (item.completed === true) {
            filterTodos.push(item)
          }
        })
      }
      return filterTodos
    }
  },
  directives: {
    focus: {
      inserted: function (el) {
        el.focus()
      }
    }
  },
  filters: {
    monthStr (n) {
      const allMonthArr = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
      return allMonthArr[n - 1]
    },
    dateNumberFix (n) {
      let pickDateStr = ''
      if (n.toString().length === 1) {
        pickDateStr = `0${n}`
      } else {
        pickDateStr = n.toString()
      }
      return pickDateStr
    }
  },
  mounted () {
    this.getTodosData()
  }
}
</script>

<style leng="scss">
  .fade-setting{
    height: 0px;
    overflow: hidden;
    transition: all 0.5s ;
  }
  .fade-setting.input-fadein{
    height: 100px;
    overflow: visible;
  }
  .card.task-completed {
    background-color: #ede8df;
  }
  .card.task-completed .card-title, .card.task-completed .card-text{
    text-decoration : line-through;
  }
</style>
