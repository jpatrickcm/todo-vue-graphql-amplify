<template>
  <div id="app">
    <amplify-authenticator>
      <div class="welcome">
        <h1>Hi, {{ user.username }}</h1>
        <h3>You can add, done, delete and view a listof your tasks</h3>
        <amplify-sign-out></amplify-sign-out>
      </div>

      <div class="form-body">
        <form v-on:submit.prevent autocomplete="off">
          <div>
            <input v-model='form.detail' class="input" placeholder="Add detail" autocomplete="off">
          </div>
          <button @click='createTodo' class='button'>New</button>
          <button @click='getData' class='button'>Refresh</button>
        </form> 
      </div>
      <div class="app-body">
        <ul class="todoList">
          <li class="todo" v-for="todo of sortedTodos" :key='todo.id'>{{todo.detail}}</li>
        </ul>
      </div>
    </amplify-authenticator>
  </div>
</template>

<script>
import { AuthState, onAuthUIStateChange } from "@aws-amplify/ui-components";
import { API, graphqlOperation } from 'aws-amplify';
import { listTodos } from "./graphql/queries";
import { createTodo } from './graphql/mutations';
import { onCreateTodo } from "./graphql/subscriptions";

export default {
  name: "App",
  data() {
    return {
      user: {},
      todos:[],
      form:{}
    };
  },
  computed:{
    sortedTodos(){
      let todos= {...this.todos};
      return todos;
    }
  },
  created() {
    onAuthUIStateChange((state, user) => {
      if (state === AuthState.SignedIn) {
        this.user = user;
        this.getData();
      }
    });

    API.graphql(graphqlOperation(onCreateTodo)).subscribe((sourceData) =>{
      const newTodo = sourceData.value.data.onCreateTodo;
      if (newTodo) {
        if (this.todos.some( x => x.id === newTodo.id) ) {
          return;
        }
        this.todos = [newTodo, ...this.todos];
      }
    });
  },
  methods: {
    async getData(){
      try {
        this.loading = true;
        const response = await API.graphql(graphqlOperation(listTodos));
        this.todos = response.data.listTodos.items;
      } catch (error) {
        console.log('Error loading todos...', error);
      }
      finally{
        this.loading = false;
      }
    }
    ,
    async createTodo(){
      const { detail } = this.form;
      if (!detail ) {
        return;
      }    
      const todo = {detail, todoId: this.todoId};
      try {
        const response = await API.graphql(graphqlOperation(createTodo, {input: todo}));
        this.todos = [...this.todos, response.data.createTodo];
        this.form = {detail:''};
      } catch (error) {
        console.log('Error loading todos...', error);
      }
    }
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: grid;
  place-content: center;
}

.app-body{
  display: grid;
  place-content: center;
}

.input{
  width: 100%;
  padding: 0.5rem;
  font-size: 1rem;
  outline: none;
  border-radius: 0.25rem;
  border-style: none;
  border: solid 1px lightgray;
  box-sizing: border-box;
  margin-top: 8px;
}

.button {
  appearance: none;
  background-color: #2ea44f;
  border: 1px solid rgba(27, 31, 35, .15);
  border-radius: 6px;
  box-shadow: rgba(27, 31, 35, .1) 0 1px 0;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
  display: inline-block;
  /* font-family: -apple-system,system-ui,"Segoe UI",Helvetica,Arial,sans-serif,"Apple Color Emoji","Segoe UI Emoji"; */
  font-size: 14px;
  font-weight: 600;
  line-height: 20px;
  padding: 6px 16px;
  position: relative;
  text-align: center;
  text-decoration: none;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
  vertical-align: middle;
  white-space: nowrap;
  margin-right: 8px;
  margin-top: 8px;
}

.button:focus:not(:focus-visible):not(.focus-visible) {
  box-shadow: none;
  outline: none;
}

.button:hover {
  background-color: #2c974b;
}

.button:focus {
  box-shadow: rgba(46, 164, 79, .4) 0 0 0 3px;
  outline: none;
}

.button:disabled {
  background-color: #94d3a2;
  border-color: rgba(27, 31, 35, .1);
  color: rgba(255, 255, 255, .8);
  cursor: default;
}

.button:active {
  background-color: #298e46;
  box-shadow: rgba(20, 70, 32, .2) 0 1px 0 inset;
}

</style>
