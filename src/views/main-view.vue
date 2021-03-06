<template>
  <header class="header">
    <h1>todos</h1>
    <input class="new-todo"
      autofocus autocomplate="off"
      placeholder="What needs to be done?"
      v-model="newTodo"
      v-on="keyup:addTodo | key 'enter'">
  </header>
  <section class="main" v-show="todos.length" v-cloak>
    <input type="checkbox" class="toggle-all" v-model="allDone">
    <ul class="todo-list">
      <li class="todo"
        v-repeat="todo: filteredTodos"
        v-class="completed: todo.completed, editing: todo == editedTodo">
        <div class="view">
          <input type="checkbox" class="toggle" v-model="todo.completed">
          <label v-on="dblclick: editTodo(todo)">{{todo.title}}</label>
          <button class="destroy" v-on="click: removeTodo(todo)"></button>
        </div>
        <input type="text" class="edit"
        v-model="todo.title"
        v-todo-focus="todo == editedTodo"
        v-on="blur: doneEdit(todo),
              keyup: doneEdit(todo) | key 'enter',
              keyup: cancelEdit(todo) | key 'esc'">
      </li>
    </ul>
  </section>
  <footer class="footer" v-show="todos.length" v-cloak>
    <span class="todo-count">
      <strong v-text="remaining"></strong>{{remaining | pluralize 'item'}} left
    </span>
    <ul class="filters">
      <li><a v-link="/all" v-class="selected: visibility == 'all'">All</a></li>
      <li><a v-link="/active" v-class="selected: visibility == 'active'">Active</a></li>
      <li><a v-link="/completed" v-class="selected: visibility == 'completed'">completed</a></li>
    </ul>
    <button class="clear-completed" v-on="click:removeCompleted" v-show="todos.length > remaining">
      Clear completed
    </button>
  </footer>
</template>

<script>
var store = require('../store')

// var filters = require('../todofilter')
var all = require('../filters/all')
var active = require('../filters/active')
var completed = require('../filters/completed')


module.exports = {
  inherit: true,
  data: function () {
    return {
      todos: store.fetch(),
      newTodo: '',
      editedTodo: null
    }
  },

  components: {
    'app-footer': require('../components/footer.vue')
  },

  computed: {
    filteredTodos: function() {
      var func;
      switch(this.visibility){
        case "all": func = all(this.todos);break;
        case "active": func = active(this.todos);break;
        case "completed": func = completed(this.todos);break;
      }
      return func;
    },
    remaining: function() {
      return active(this.todos).length;
    },
    allDone: {
      get: function() {
        return this.remaining === 0;
      },
      set: function(value) {
        this.todos.forEach(function(todo) {
          todo.completed = value;
        })
      } 
    }
  },

  ready: function () {
      this.$watch('todos', function (todos) {
        store.save(todos);
      }, { deep: true });
  },

  methods: {
    addTodo: function() {
      var value = this.newTodo && this.newTodo.trim();
      if(!value){
      return;
      }
      this.todos.push({ title: value, completed: false});
      this.newTodo = '';
    },

    removeTodo: function(todo) {
      this.todos.$remove(todo);
    },

    editTodo: function(todo) {
      this.beforeEditCache = todo.title;
      this.editedTodo = todo;
    },

    doneEdit: function(todo) {
      if(!this.editedTodo) {
        return;
      }
      this.editedTodo = null;
      todo.title = todo.title.trim();
      if(!todo.title) {
        this.removeTodo(todo);
      }
    },

    cancelEdit: function(todo) {
      this.editedTodo = null;
      todo.title = this.beforeEditCache;
    },

    removeCompleted: function() {
      this.todos = active(this.todos);
    }
  },

  directives: {
    'todo-focus': function(value) {
      if(!value) {
        return;
      }
      var el = this.el;
      setTimeout(function() {
        el.focus();
      }, 0);
    }
  }

}
</script>

<style lang="stylus">
</style>