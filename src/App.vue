<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// 定義資料狀態
const newTodo = ref('')
const todos = ref([])
const API_URL = 'http://localhost:8080'
const loadingId = ref(null)
// 1. 取得所有代辦事項 (對應 Go 的 GET /todos)
const fetchTodos = async () => {
  const response = await axios.get(`${API_URL}/todos`)
  todos.value = response.data
}
const toggleStatus = async (id) => {
  loadingId.value = id; // 👈 1. 開始讀取：記錄是哪個 ID 在轉
  try {
    // 呼叫 PUT API
    await axios.put(`${API_URL}/todos/${id}`)
    // 成功後重新整理清單
    await fetchTodos()
  } catch (error) {
    console.error("更新狀態失敗:", error)
  } finally {
    loadingId.value = null; // 👈 2. 結束讀取：清空，讓轉圈圈變回勾勾
  }
}
// 2. 新增代辦事項 (對應 Go 的 POST /todos)
const addTodo = async () => {
  if (!newTodo.value) return
  await axios.post(`${API_URL}/todos`, {
    title: newTodo.value,
    status: false
  })
  newTodo.value = '' // 清空輸入框
  fetchTodos() // 重新刷新清單
}

// 3. 刪除事項 (對應 Go 的 DELETE /todos/:id)
const deleteTodo = async (id) => {
  await axios.delete(`${API_URL}/todos/${id}`)
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
          <div class="flex items-center cursor-pointer flex-1" @click="toggleStatus(todo.ID)">
            
            <span class="inline-flex w-24 items-center justify-start shrink-0">
              <span v-if="loadingId === todo.ID" class="animate-spin text-blue-500 text-lg mr-2">
                🔄
              </span>
              <span v-else :class="todo.status ? 'text-green-500' : 'text-red-500'" class="flex items-center">
                <span class="mr-1 text-lg">{{ todo.status ? '✓' : '✕' }}</span>
                <span class="text-sm font-medium">{{ todo.status ? '已完成' : '未完成' }}</span>
              </span>
            </span>

            <span :class="{ 'line-through text-gray-400': todo.status }" class="text-slate-700 ml-2">
              {{ todo.title }}
            </span>
          </div>
          
          <button 
            @click.stop="deleteTodo(todo.ID)"
            class="text-red-400 hover:text-red-600 opacity-0 group-hover:opacity-100 transition px-2"
          >
            🗑️
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>