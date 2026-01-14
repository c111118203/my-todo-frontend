<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// 定義資料狀態
const newTodo = ref('')
const todos = ref([])
const API_URL = 'https://my-todo-backend-p5s7.onrender.com'
// 1. 取得所有代辦事項 (對應 Go 的 GET /todos)
const fetchTodos = async () => {
  const response = await axios.get(`${API_URL}/todos`)
  todos.value = response.data
}
const toggleStatus = async (id) => {
  try {
    // 呼叫剛剛寫好的 PUT API
    await axios.put(`${API_URL}/todos/${id}`)
    // 成功後，重新抓取清單來更新畫面
    fetchTodos()
  } catch (error) {
    console.error("更新狀態失敗:", error)
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
      <div class="flex items-center cursor-pointer" @click="toggleStatus(todo.ID)">
        
        <span class="inline-block w-8 text-lg flex items-center justify-center">
          
          <span v-if="loadingId === todo.ID" class="animate-spin text-blue-500">
            🔄
          </span>
          
          <span v-else :class="todo.status ? 'text-green-500' : 'text-red-500'">
            {{ todo.status ? '✓' : '✕' }}
          
          </span>
            

        </span>
          <span :class="{ 'line-through text-gray-400': todo.status }" class="ml-2">
              {{ todo.status ? '已完成' : '未完成' }}
              </span>
       
      </div>
          
    
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