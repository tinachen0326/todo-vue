<template>
  <div class="text-center mb-8">
    <h1 class="text-4xl font-bold text-gray-800 mb-4">📝 我的任務清單</h1>
    <p class="text-gray-600 mb-4">{{ currentDate }}</p>

    <!-- 統計資訊 -->
    <div class="flex justify-center gap-8">
      <div class="text-center">
        <div class="text-2xl font-bold text-blue-600">{{ stats.total }}</div>
        <div class="text-sm text-gray-500">總任務</div>
      </div>
      <div class="text-center">
        <div class="text-2xl font-bold text-orange-600">{{ stats.active }}</div>
        <div class="text-sm text-gray-500">進行中</div>
      </div>
      <div class="text-center">
        <div class="text-2xl font-bold text-green-600">
          {{ stats.completed }}
        </div>
        <div class="text-sm text-gray-500">已完成</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";

// 接收父組件傳遞的 props
defineProps({
  stats: {
    type: Object,
    required: true,
  },
});

// 計算屬性 - 顯示當前日期
const currentDate = computed(() => {
  const options = {
    year: "numeric",
    month: "long",
    day: "numeric",
    weekday: "long",
  };
  return new Date().toLocaleDateString("zh-TW", options);
});
</script>
