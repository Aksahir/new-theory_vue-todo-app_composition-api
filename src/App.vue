<script setup>
  import { ref, computed, watch, onBeforeMount, onMounted } from 'vue';
  import StatusFilter from './components/StatusFilter.vue';
  import TodoItem from './components/TodoItem.vue';
  import * as todoApi from './api/todos';
  import Message from './components/Message.vue';

  const title = ref("");
  const status = ref('all');
  const todos = ref([]);
  const errorMessage = ref(null);
  
  // let initialTodos = [];

  // try {
  //   initialTodos = JSON.parse(localStorage.getItem('todos'));
  // } catch (error) {}

  // if (!Array.isArray(initialTodos)) {
  //   initialTodos = [];
  // }

  

  // onBeforeMount(() => {
  //   try {
  //     todos.value = JSON.parse(localStorage.getItem('todos'));
  //   } catch (error) {}

  //   if (!Array.isArray(todos.value)) {
  //     todos.value = [];
  //   }
  // });

  onMounted(async () => {
    try {
      todos.value = await todoApi.getTodos();
    } catch (error) {
      errorMessage.value.show('Unable to load todos');
    }
  });

  const activeTodos = computed(() =>
    todos.value.filter((todo) => !todo.completed)
  );

  const visibleTodos = computed(() => {
    if (status.value === 'active') {
      return activeTodos.value;
    }

    if (status.value === 'completed') {
      return todos.value.filter(todo => todo.completed);
    }

    return todos.value;
  });

  // watch(
  //   todos,
  //   newTodos => {
  //     localStorage.setItem('todos', JSON.stringify(newTodos));
  //   },
  //   { deep: true },
  // );

  const addTodo = async () => {
    if (!title.value) {
      errorMessage.value.show('Title should not be empty');
    return;
  }

    try {
      const newTodo = await todoApi.createTodo(title.value);

      todos.value.push(newTodo);
      title.value = '';
    } catch (error) {
      errorMessage.value.show('Unable to add a todo');
    }
  }

  const deleteTodo = async (todoId) => {
    try {
      await todoApi.deleteTodo(todoId);
      todos.value = todos.value.filter(todo => todoId !== todo.id);
    } catch (error) {
      errorMessage.value.show('Unable to delete a todo');
    }
  };

  const updateTodo = async ({ id, title, completed }) => {
    try {
      const updatedTodo = await todoApi.updateTodo({ id, title, completed });
      const currentTodo = todos.value.find(todo => todo.id === id);

      Object.assign(currentTodo, updatedTodo);
    } catch (error) {
      errorMessage.value.show('Unable to update a todo');
    }
  };

</script>

<template>
  <div class="todoapp">
      <h1 class="todoapp__title">todos {{ todos.length }}</h1>

      <div class="todoapp__content">
        <header class="todoapp__header">
          <button
            v-if="todos.length > 0"
            class="todoapp__toggle-all"
            :class="{ active: activeTodos.length === 0 }"
          ></button>

          <form @submit.prevent="addTodo">
            <input
              class="todoapp__new-todo"
              v-model="title"
              placeholder="What needs to be done?"
            ></input>
          </form>
        </header>

        <TransitionGroup
          tag="section"
          name="todolist"
          class="todoapp__main"
          v-if="todos.length > 0"
        >
          <TodoItem
            v-for="todo of visibleTodos"
            :key="todo.id"
            :todo="todo"
            @delete="deleteTodo(todo.id)" 
            @update="updateTodo($event)"
          />    
        </TransitionGroup>

        <footer class="todoapp__footer" v-if="todos.length > 0">
          <span class="todo-count">
            {{ activeTodos.length }} items left
          </span>

          <StatusFilter v-model="status" />

          <button
            class="todoapp__clear-completed"
            :disabled="todos.length === activeTodos.length"
          >
            Clear completed
          </button>
        </footer>
      </div>

      <Message class="is-warning" ref="errorMessage">
        <template #header>
          <p>Server Error</p>
        </template>

        <template #default="{ text }">
          <p>{{ text }}</p>
        </template>
      </Message>
    </div>
</template>

<style scoped>
  .todolist-enter-active,
  .todolist-leave-active {
    max-height: 60px;
    transition: all 0.5s ease;
  }
  .todolist-enter-from,
  .todolist-leave-to {
    opacity: 0;
    max-height: 0;
    transform: scaleY(0);
  }
</style>
