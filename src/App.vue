<template>
  <div
    class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 py-8 px-4"
  >
    <div class="max-w-3xl mx-auto">
      <!-- 標題與統計 -->
      <TodoHeader :stats="statistics" />

      <!-- 輸入區域 -->
      <TodoInput @add-todo="addTodo" />

      <!-- 篩選器 -->
      <TodoFilter
        v-model="filter"
        :stats="statistics"
        @clear-completed="clearCompleted"
      />

      <!-- 任務列表 -->
      <TodoList
        :todos="filteredTodos"
        @update-todo="updateTodo"
        @delete-todo="deleteTodo"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from "vue";
import TodoHeader from "./components/TodoHeader.vue";
import TodoInput from "./components/TodoInput.vue";
import TodoFilter from "./components/TodoFilter.vue";
import TodoList from "./components/TodoList.vue";

// 響應式資料
const todos = ref([]);
const filter = ref("all");

// 計算屬性 - 根據篩選條件顯示任務
const filteredTodos = computed(() => {
  switch (filter.value) {
    case "active":
      return todos.value.filter((todo) => !todo.completed);
    case "completed":
      return todos.value.filter((todo) => todo.completed);
    default:
      return todos.value;
  }
});

// 計算屬性 - 統計資料
const statistics = computed(() => ({
  total: todos.value.length,
  active: todos.value.filter((t) => !t.completed).length,
  completed: todos.value.filter((t) => t.completed).length,
}));

// 生命週期 - 載入儲存的資料
onMounted(() => {
  const saved = localStorage.getItem("todos");
  if (saved) {
    todos.value = JSON.parse(saved);
  }
});

// 監聽器 - 自動儲存到 localStorage
watch(
  todos,
  (newTodos) => {
    localStorage.setItem("todos", JSON.stringify(newTodos));
  },
  { deep: true }
);

// 方法 - 新增任務
const addTodo = (todoData) => {
  todos.value.unshift({
    id: Date.now(),
    text: todoData.text,
    completed: false,
    priority: todoData.priority,
    createdAt: new Date().toISOString(),
  });
};

// 方法 - 更新任務
const updateTodo = (updatedTodo) => {
  const index = todos.value.findIndex((t) => t.id === updatedTodo.id);
  if (index !== -1) {
    todos.value[index] = updatedTodo;
  }
};

// 方法 - 刪除任務
const deleteTodo = (id) => {
  todos.value = todos.value.filter((t) => t.id !== id);
};

// 方法 - 清除已完成
const clearCompleted = () => {
  todos.value = todos.value.filter((t) => !t.completed);
};
</script>
