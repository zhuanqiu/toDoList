<template>
  <div class="todo-app">
    <div class="header">
      <input
        type="text"
        placeholder="新增事项"
        v-model="newTodo"
        :class="inputClass"
        @keyup.enter="addTodo"
      />
      <button @click="addTodo">新增</button>
    </div>
    <div class="actions">
      <button @click="toggleAll(true)">全部标记为已完成</button>
      <button @click="toggleAll(false)">全部标记为未完成</button>
      <button @click="showAll">显示全部</button>
      <button @click="showCompleted">显示已完成</button>
      <button @click="showActive">显示未完成</button>
      <button @click="clearCompleted">清除已完成</button>
      <button @click="clearAll">清除全部</button>
    </div>
    <div class="info">
      <span>当前显示: {{ filterName }}，共 {{ filteredTodos.length }} 个</span>
    </div>
    <ul class="todo-list">
      <li
        v-for="(todo, index) in filteredTodos"
        :key="index"
        :class="{ completed: todo.completed, editing: todo === editingTodo }"
      >
        <div class="view">
          <input type="checkbox" v-model="todo.completed" />
          <label @dblclick="editTodo(todo)">{{ todo.label }}</label>
          <button class="destroy" @click="removeTodo(index)">X</button>
        </div>
        <input
          v-if="todo === editingTodo"
          type="text"
          v-model="todo.label"
          @keyup.enter="saveTodo(todo)"
          @keyup.esc="cancelEdit(todo)"
          class="edit"
        />
      </li>
    </ul>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from 'vue'

const newTodo = ref('')
const todos = ref<{ label: string; completed: boolean }[]>([])
const editingTodo = ref<{ label: string; completed: boolean } | null>(null)
const filter = ref('all')

const inputClass = computed(() => {
  if (newTodo.value.trim().length === 0) return 'normal'
  if (newTodo.value.trim().length < 2) return 'wrong'
  return 'permission'
})

const filterName = computed(() => {
  switch (filter.value) {
    case 'active':
      return '未完成'
    case 'completed':
      return '已完成'
    default:
      return '全部'
  }
})

const filteredTodos = computed(() => {
  switch (filter.value) {
    case 'active':
      return todos.value.filter((todo) => !todo.completed)
    case 'completed':
      return todos.value.filter((todo) => todo.completed)
    default:
      return todos.value
  }
})

const addTodo = () => {
  const value = newTodo.value.trim()
  if (value.length >= 2) {
    todos.value.push({ label: value, completed: false })
    newTodo.value = ''
    saveTodosToLocalStorage()
  } else if (value.length > 0) {
    alert('字數不小於2')
  }
}

const removeTodo = (index: number) => {
  todos.value.splice(index, 1)
  saveTodosToLocalStorage()
}

const toggleAll = (checked: boolean) => {
  todos.value.forEach((todo) => {
    todo.completed = checked
  })
  saveTodosToLocalStorage()
}

const clearCompleted = () => {
  todos.value = todos.value.filter((todo) => !todo.completed)
  saveTodosToLocalStorage()
}

const clearAll = () => {
  todos.value = []
  saveTodosToLocalStorage()
}

const showAll = () => {
  filter.value = 'all'
}

const showCompleted = () => {
  filter.value = 'completed'
}

const showActive = () => {
  filter.value = 'active'
}

const editTodo = (todo: { label: string; completed: boolean }) => {
  editingTodo.value = todo
}

const saveTodo = (todo: { label: string; completed: boolean }) => {
  if (todo.label.trim().length > 0) {
    editingTodo.value = null
    saveTodosToLocalStorage()
  } else {
    removeTodo(todos.value.indexOf(todo))
  }
}

const cancelEdit = (todo: { label: string; completed: boolean }) => {
  editingTodo.value = null
  todo.label = todos.value[todos.value.indexOf(todo)].label
}

const saveTodosToLocalStorage = () => {
  localStorage.setItem('todos', JSON.stringify(todos.value))
}

const loadTodosFromLocalStorage = () => {
  const storedTodos = JSON.parse(localStorage.getItem('todos') || '[]')
  todos.value = storedTodos
}

onMounted(() => {
  loadTodosFromLocalStorage()
  window.addEventListener('beforeunload', saveTodosToLocalStorage)
})

onUnmounted(() => {
  window.removeEventListener('beforeunload', saveTodosToLocalStorage)
})
</script>
<style scoped>

</style>

