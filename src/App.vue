<script setup>
import { computed, onMounted, ref, watch } from 'vue'

const newTodo = ref('')
const filter = ref('all')
const todos = ref([])

const STORAGE_KEY = 'todoli.todos'

const addTodo = () => {
  const title = newTodo.value.trim()
  if (!title) return

  todos.value.unshift({
    id: Date.now(),
    title,
    completed: false,
    createdAt: new Date().toISOString(),
  })
  newTodo.value = ''
}

const toggleTodo = (todo) => {
  todo.completed = !todo.completed
}

const removeTodo = (todoId) => {
  todos.value = todos.value.filter((todo) => todo.id !== todoId)
}

const clearCompleted = () => {
  todos.value = todos.value.filter((todo) => !todo.completed)
}

const filteredTodos = computed(() => {
  if (filter.value === 'active') {
    return todos.value.filter((todo) => !todo.completed)
  }
  if (filter.value === 'completed') {
    return todos.value.filter((todo) => todo.completed)
  }
  return todos.value
})

const remainingCount = computed(() =>
  todos.value.reduce((total, todo) => total + (todo.completed ? 0 : 1), 0)
)

const completedCount = computed(() =>
  todos.value.reduce((total, todo) => total + (todo.completed ? 1 : 0), 0)
)

onMounted(() => {
  try {
    const raw = localStorage.getItem(STORAGE_KEY)
    if (raw) {
      const parsed = JSON.parse(raw)
      if (Array.isArray(parsed)) {
        todos.value = parsed
      }
    }
  } catch (error) {
    console.error('Erro ao carregar tarefas', error)
  }
})

watch(
  todos,
  (value) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(value))
  },
  { deep: true }
)
</script>

<template>
  <main class="page">
    <section class="card">
      <header class="header">
        <div>
          <p class="eyebrow">To-do list</p>
          <h1>Organize sua rotina com o Todoli</h1>
          <p class="subtitle">
            Cadastre tarefas, acompanhe o progresso e finalize tudo com
            facilidade.
          </p>
        </div>
        <div class="stats">
          <div>
            <span class="stat-value">{{ remainingCount }}</span>
            <span class="stat-label">pendentes</span>
          </div>
          <div>
            <span class="stat-value">{{ completedCount }}</span>
            <span class="stat-label">concluídas</span>
          </div>
        </div>
      </header>

      <form class="input-row" @submit.prevent="addTodo">
        <input
          v-model="newTodo"
          type="text"
          placeholder="Adicione uma nova tarefa"
          aria-label="Adicionar tarefa"
        />
        <button type="submit">Adicionar</button>
      </form>

      <div class="filters">
        <button
          :class="{ active: filter === 'all' }"
          type="button"
          @click="filter = 'all'"
        >
          Todas
        </button>
        <button
          :class="{ active: filter === 'active' }"
          type="button"
          @click="filter = 'active'"
        >
          Ativas
        </button>
        <button
          :class="{ active: filter === 'completed' }"
          type="button"
          @click="filter = 'completed'"
        >
          Concluídas
        </button>
        <button
          class="ghost"
          type="button"
          @click="clearCompleted"
          :disabled="completedCount === 0"
        >
          Limpar concluídas
        </button>
      </div>

      <ul class="todo-list" v-if="filteredTodos.length">
        <li v-for="todo in filteredTodos" :key="todo.id" class="todo-item">
          <label class="todo-content">
            <input
              type="checkbox"
              :checked="todo.completed"
              @change="toggleTodo(todo)"
            />
            <span :class="{ done: todo.completed }">{{ todo.title }}</span>
          </label>
          <button class="icon-button" type="button" @click="removeTodo(todo.id)">
            Remover
          </button>
        </li>
      </ul>

      <div v-else class="empty">
        <p>Nenhuma tarefa por aqui ainda.</p>
        <span>Adicione uma nova tarefa para começar.</span>
      </div>
    </section>
  </main>
</template>
