<script>
import { API } from 'aws-amplify'
import { createTodo } from './graphql/mutations'
import { listTodos } from './graphql/queries'
// other imports
import { onCreateTodo } from './graphql/subscriptions'
// eslint-disable-next-line no-unused-vars
import { Authenticator } from '@aws-amplify/ui-vue'
import '@aws-amplify/ui-vue/styles.css'

export default {
  name: 'App',
  async created() {
    this.getTodos()
  },
  data() {
    return {
      name: '',
      description: '',
      todos: []
    }
  },
  methods: {
    async createTodo() {
      const { name, description } = this
      if (!name || !description) return
      const todo = { name, description }
      this.todos = [...this.todos, todo]
      await API.graphql({
        query: createTodo,
        variables: { input: todo }
      })
      this.name = ''
      this.description = ''
    },
    async getTodos() {
      const todos = await API.graphql({
        query: listTodos
      })
      this.todos = todos.data.listTodos.items
    },

    // other methods
    subscribe() {
      API.graphql({ query: onCreateTodo }).subscribe({
        next: (eventData) => {
          let todo = eventData.value.data.onCreateTodo
          if (this.todos.some((item) => item.name === todo.name)) return // remove duplications
          this.todos = [...this.todos, todo]
        }
      })
    }
  }
}
</script>
<template>
  <div id="todo-container">
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name" />
    <input type="text" v-model="description" placeholder="Todo description" />
    <button v-on:click="createTodo">Create Todo</button>
    <div v-for="item in todos" :key="item.id">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
    </div>
  </div>
  <authenticator>
    <template v-slot="{ user, signOut }">
      <h1>Hello {{ user.username }}!</h1>
      <button @click="signOut">Sign Out</button>
    </template>
  </authenticator>
</template>
