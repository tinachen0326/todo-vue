<template>
  <div class="bg-white rounded-lg shadow-md p-6">
    <!-- 空狀態 -->
    <div v-if="todos.length === 0" class="text-center py-12">
      <div class="text-6xl mb-4">📋</div>
      <p class="text-gray-500 text-lg">還沒有任何任務，開始新增吧！</p>
    </div>

    <!-- 任務列表 -->
    <TransitionGroup name="fade" tag="div" class="space-y-2">
      <TodoItem
        v-for="todo in todos"
        :key="todo.id"
        :todo="todo"
        @update="$emit('update-todo', $event)"
        @delete="$emit('delete-todo', $event)"
      />
    </TransitionGroup>
  </div>
</template>

<script setup>
import TodoItem from "./TodoItem.vue";

// 定義 props
defineProps({
  todos: Array,
});

// 定義事件
defineEmits(["update-todo", "delete-todo"]);
</script>

<style scoped>
/* 動畫效果 */
.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from {
  opacity: 0;
  transform: translateY(-10px);
}

.fade-leave-to {
  opacity: 0;
  transform: translateX(20px);
}
</style>
