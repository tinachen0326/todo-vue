<template>
  <div class="bg-white rounded-lg shadow-md p-6 mb-6">
    <form @submit.prevent="handleSubmit" class="flex gap-2">
      <!-- 輸入框 -->
      <input
        v-model="newTodo.text"
        type="text"
        placeholder="輸入新任務..."
        class="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
      />

      <!-- 優先級選擇 -->
      <select
        v-model="newTodo.priority"
        class="px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
      >
        <option value="low">低</option>
        <option value="medium">中</option>
        <option value="high">高</option>
      </select>

      <!-- 新增按鈕 -->
      <button
        type="submit"
        class="bg-indigo-600 text-white px-6 py-2 rounded-lg hover:bg-indigo-700 transition duration-200"
      >
        新增
      </button>
    </form>
  </div>
</template>

<script setup>
import { ref } from "vue";

// 定義要發送的事件
const emit = defineEmits(["add-todo"]);

// 響應式資料 - 新任務
const newTodo = ref({
  text: "",
  priority: "medium",
});

// 方法 - 處理表單提交
const handleSubmit = () => {
  if (newTodo.value.text.trim()) {
    // 發送事件給父組件
    emit("add-todo", { ...newTodo.value });
    // 清空輸入
    newTodo.value.text = "";
    newTodo.value.priority = "medium";
  }
};
</script>
