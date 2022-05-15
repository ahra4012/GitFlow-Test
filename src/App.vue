<template>
  <div v-if="toggle">true</div>
  <div v-else>false</div>
  <button
    @click="onToggle"
  >Toggle</button>
  <div class="container">
    <h2>Todo List</h2>

    <input type="text"
           class="form-control"
           placeholder="search"
           v-model="searchText"
    />

    <hr />

    <todo-simple-form @add-todo="addTodo" />

    <div v-if="!filteredTodos.length">
      There is nothing to display
    </div>

    <div>{{error}}</div>
    <todo-list
        :todos="filteredTodos"
        @toggle-todo="toggleTodo"
        @delete-todo="deleteTodo"
    />
    <hr/>
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li v-if="currentPage !==1"
            class="page-item"><a class="page-link" href="#"
                                 @click="getTodos(currentPage - 1)"
        >Previous</a></li>
        <li
            v-for="item in numberOfPages"
            :key="item"
            class="page-item"
            :class="currentPage === item ? 'active':''"
        >
          <a class="page-link" href="#" @click="getTodos(item)">{{item}}</a>
        </li>
        <li v-if="numberOfPages !== currentPage"
            class="page-item">
          <a class="page-link" href="#"
             @click="getTodos(currentPage + 1)"
          >Next</a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import {ref,computed} from 'vue'
import TodoSimpleForm from "@/components/TodoSimpleForm";
import TodoList from "@/components/TodoList";
import axios from 'axios'
export default {
  name: 'App',
  components:{
    TodoList,
    TodoSimpleForm
  },
  setup(){
    const toggle = ref(false)
    const todos = ref([])
    const error = ref('')
    const numberOfTodos = ref(0)
    const limit = 5;
    const currentPage = ref(1);

    const numberOfPages = computed(()=>{
      return Math.ceil(numberOfTodos.value/limit)
    })

    const getTodos = async (page = currentPage.value) =>{
      currentPage.value = page;
      try{
        const res = await axios.get(
            `http://localhost:3000/todos?_page=${page}&_limit=${limit}`
        );
        todos.value = res.data
        numberOfTodos.value = res.headers['x-total-count']
      } catch (err){
        error.value = 'something went wrong'
      }
    }
    getTodos()
    const todoStyle = {
      textDecoration:'line-through',
      color:'gray',
    }

    const toggleTodo = async (index) => {
      error.value = ''
      const id = todos.value[index].id
      try{
        await axios.patch('http://localhost:3000/todos/'+ id, {
          completed : !todos.value[index].completed
        })

        todos.value[index].completed = !todos.value[index].completed
      } catch (err){
        error.value = 'something went wrong'
      }
    }

    const deleteTodo = async (index) => {
      error.value = ''
      const id = todos.value[index].id
      try{
        await axios.delete('http://localhost:3000/todos/'+ id)
        todos.value.splice(index ,1)
      } catch (err){
        error.value = 'something went wrong'
      }

    }

    const onToggle = () =>{
      toggle.value = !toggle.value
    }

    const addTodo = async (todo) => {
      error.value=''
      try{
        const res = await axios.post('http://localhost:3000/todos',{
          subject:todo.subject,
          completed:todo.completed
        })
        todos.value.push(res.data)
      } catch (err){
        error.value = 'something went wrong'
      }
    }

    const searchText = ref('')
    const filteredTodos = computed(() => {
      if(searchText.value) {
        return todos.value.filter(todo => {
          return todo.subject.includes(searchText.value)
        })
      }
      return todos.value;
    })

    return{
      toggle,
      todos,
      onToggle,
      deleteTodo,
      addTodo,
      todoStyle,
      toggleTodo,
      searchText,
      filteredTodos,
      error,
      getTodos,
      numberOfPages,
      currentPage
    }
  }
}
</script>

<style>
.name{
  color:red
}
.blue{
  color:blue
}
.todo{
  color:gray;
  text-decoration: line-through;
}
</style>
