<script setup>
import { computed, onMounted, reactive, ref } from 'vue'
import { http } from './api/axios.js'

const todos = reactive([])
const newTodoInput = ref('')
const isHidingCompletedTodos = ref(false)

async function fetchTodos() {
  const { data } = await http.get('/todos')
  todos.splice(0, todos.length, ...data)
}

async function handleCreateNewTodo() {
  const { data } = await http.post('/todos', {
    name: newTodoInput.value,
  })
  newTodoInput.value = ''
  fetchTodos()
}

async function handleDeleteTaskByID(id) {
  const { data } = await http.delete(`/todos/${id}`)
  fetchTodos()
}

async function handleUpdateTaskState(id, done) {
  const { data } = await http.put(`/todos/${id}`, {
    done,
  })
  fetchTodos()
}

function handleChangeIsHidingCompletedTodos() {
  isHidingCompletedTodos.value = !isHidingCompletedTodos.value
}

const todosCompleted = computed(() => {
  return todos.filter(todo => todo.done)
})

const filteredTodos = computed(() => {
  return isHidingCompletedTodos.value ? todos.filter(todo => !todo.done) : todos
})

onMounted(() => {
  fetchTodos()
})
</script>

<template>
  <header class="flex justify-between items-center">
    <div>
      <h1 class="font-bold text-2xl text-white">to do list</h1>
      <p class="text-xs pt-1.5">Vamos deixar de procrastinar?</p>
    </div>
    <img
      class="h-12 w-12 rounded-full"
      src="https://www.github.com/jhiorgenes.png"
      alt=""
    />
  </header>

  <div class="divider"></div>

  <main class="sm:mt-20">
    <div class="grid gap-3 grid-cols-2 items-center sm:flex sm:justify-between">
      <span>{{ todosCompleted.length }} Concluídos</span>
      <button @click="handleChangeIsHidingCompletedTodos" class="btn">
        {{
          isHidingCompletedTodos ? 'Mostrar Concluídos' : 'Ocultar Concluídos'
        }}
      </button>
    </div>

    <div class="mt-10 flex flex-col gap-5">
      <div
        class="flex justify-between items-center"
        v-for="todo in filteredTodos"
        :key="todo.key"
      >
        <div class="flex items-center gap-3">
          <input
            type="checkbox"
            :id="todo.id"
            v-model="todo.done"
            class="checkbox"
            @click="handleUpdateTaskState(todo.id, !todo.done)"
          />
          <label :for="todo.id" :class="{ 'line-through': todo.done }">{{
            todo.name
          }}</label>
        </div>
        <img
          @click="handleDeleteTaskByID(todo.id)"
          class="h-5 w-5 cursor-pointer"
          src="./assets/trash.svg"
          alt="trash icon"
        />
      </div>
    </div>

    <footer class="mt-16 fixed bottom-8 inset-x-8 md:inset-x-20 md:bottom-20">
      <form @submit.prevent="handleCreateNewTodo">
        <input
          type="text"
          placeholder="Nova nota"
          class="input input-bordered w-full"
          v-model="newTodoInput"
        />
      </form>
    </footer>
  </main>
</template>
