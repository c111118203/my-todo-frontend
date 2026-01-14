<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// 定義資料狀態
const newTodo = ref('')
const todos = ref([])

// 1. 取得所有代辦事項 (對應 Go 的 GET /todos)
const fetchTodos = async () => {
  const response = await axios.get('http://localhost:8080/todos')
  todos.value = response.data
}

// 2. 新增代辦事項 (對應 Go 的 POST /todos)
const addTodo = async () => {
  if (!newTodo.value) return
  await axios.post('http://localhost:8080/todos', {
    title: newTodo.value,
    status: false
  })
  newTodo.value = '' // 清空輸入框
  fetchTodos() // 重新刷新清單
}

// 3. 刪除事項 (對應 Go 的 DELETE /todos/:id)
const deleteTodo = async (id) => {
  await axios.delete(`http://localhost:8080/todos/${id}`)
  fetchTodos()
}

// 元件掛載時立刻執行抓取資料
onMounted(() => {
  fetchTodos()
})
</script>

<template>
  <div class="min-h-screen bg-slate-100 py-10 px-4">
    <div class="max-w-md mx-auto bg-white rounded-xl shadow-lg p-6">
      <h1 class="text-2xl font-bold text-slate-800 mb-6 text-center">Go + Vue Todo List</h1>
      
      <div class="flex gap-2 mb-6">
        <input 
          v-model="newTodo" 
          @keyup.enter="addTodo"
          class="flex-1 px-4 py-2 border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="添加新任務..."
        />
        <button 
          @click="addTodo"
          class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition"
        >
          新增
        </button>
      </div>

      <ul class="space-y-3">
        <li 
          v-for="todo in todos" :key="todo.ID"
          class="flex justify-between items-center p-3 bg-slate-50 rounded-lg group hover:bg-slate-100 transition"
        >
          <span class="text-slate-700">{{ todo.title }}</span>
          <button 
            @click="deleteTodo(todo.ID)"
            class="text-red-400 hover:text-red-600 opacity-0 group-hover:opacity-100 transition"
          >
            🗑️
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>