<script setup>
import { http } from './api/axios.js'
import { computed, onMounted, reactive, ref } from 'vue'

const todos = reactive([])
const isLoading = ref(false)
const newTodoInput = ref('')
const isMissingText = ref(false)
const isHidingCompletedTodos = ref(false)

const confirmDeleteId = ref(null)

function openConfirmModal(id) {
  my_modal_1.showModal()
  confirmDeleteId.value = id
}

function handleConfirmDelete() {
  if (confirmDeleteId.value !== null) {
    handleDeleteTaskByID(confirmDeleteId.value)
  }
}

async function fetchTodos() {
  isLoading.value = true
  const { data } = await http.get('/todos')
  todos.splice(0, todos.length, ...data)
  isLoading.value = false
}

async function handleCreateNewTodo() {
  if (newTodoInput.value.length < 1) {
    alert('Por favor, insira algum texto')
    isMissingText.value = true

    return
  }
  const { data } = await http.post('/todos', {
    name: newTodoInput.value,
  })
  newTodoInput.value = ''
  isMissingText.value = false
  fetchTodos()
}

async function handleDeleteTaskByID(id) {
  const { data } = await http.delete(`/todos/${id}`)
  fetchTodos()
}

async function handleUpdateTaskByID(id, done) {
  const { data } = await http.put(`/todos/${id}`, {
    done,
  })
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

    <div v-if="isLoading" class="flex justify-center mt-40">
      <span class="loading loading-ball loading-xs"></span>
      <span class="loading loading-ball loading-sm"></span>
      <span class="loading loading-ball loading-md"></span>
      <span class="loading loading-ball loading-lg"></span>
    </div>
    <div v-if="isLoading === false" class="mt-10 flex flex-col gap-5">
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
            @click="handleUpdateTaskByID(todo.id, !todo.done)"
          />
          <label :for="todo.id" :class="{ 'line-through': todo.done }">{{
            todo.name
          }}</label>
        </div>
        <img
          @click="openConfirmModal(todo.id)"
          class="h-5 w-5 cursor-pointer"
          src="./assets/trash.svg"
          alt="trash icon"
        />
      </div>
    </div>

    <dialog id="my_modal_1" class="modal">
      <div class="modal-box">
        <h3 class="font-bold text-lg">Alerta!</h3>
        <p class="py-4">Deseja mesmo excluir essa tarefa?</p>
        <div class="modal-action">
          <form method="dialog" class="flex gap-2">
            <button class="btn btn-ghost">Cancelar</button>
            <button @click="handleConfirmDelete" class="btn btn-error">
              Sim
            </button>
          </form>
        </div>
      </div>
    </dialog>

    <footer class="mt-16 fixed bottom-8 inset-x-8 md:inset-x-20 md:bottom-20">
      <form @submit.prevent="handleCreateNewTodo">
        <input
          type="text"
          placeholder="Nova nota"
          class="input input-bordered w-full"
          :class="{ 'input-error': isMissingText === true }"
          v-model="newTodoInput"
        />
      </form>
    </footer>
  </main>
</template>
