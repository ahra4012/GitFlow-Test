<template>
  <div class="card mt-2" v-for="(todo, index) in todos" :key="todo.id">
    <div class="card-body p-2 d-flex align-items-center">
      <div class="form-check flex-grow-1">
        <!-- 자식컴포넌트가 된 이상 v-model로 양방향바인딩을 하는 것은 옳지 않다. (props로 받은 속성이니까) -->
        <input class="form-check-input"
               type="checkbox"
               :checked="todo.completed"
               @input="toggleTodo(index)" />
        <!-- 🐯 스타일 바인딩 유심히 살펴보기 -->
        <!-- <label class="form-check-label" :style="todo.completed ? todoStyle : {}">{{ -->
        <label class="form-check-label" :class="{ todo: todo.completed }">{{
            todo.subject
          }}</label>
      </div>
      <div>
        <button class="btn btn-danger btn-sm" @click="deleteTodo(index)">Delete</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  // props: ['todos'],
  props: {
    todos: {
      type: Array,
      required: true,
    },
  },
  emits:[
      'toggle-todo',
      'delete-todo'
  ],
  setup(props, {emit}) {
    const toggleTodo = (index) => {
      emit('toggle-todo', index);
    };

    const deleteTodo = (index) => {
      emit('delete-todo', index)
    }

    return {
      toggleTodo,
      deleteTodo
    };
  },
};
</script>

<style></style>