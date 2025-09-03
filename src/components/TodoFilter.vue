<template>
  <div class="bg-white rounded-lg shadow-md p-4 mb-6">
    <div class="flex justify-between items-center">
      <!-- 篩選按鈕 -->
      <div class="flex gap-2">
        <button
          v-for="option in filterOptions"
          :key="option.value"
          @click="$emit('update:modelValue', option.value)"
          :class="[
            'px-4 py-2 rounded-lg transition',
            modelValue === option.value
              ? 'bg-indigo-600 text-white'
              : 'bg-gray-200 text-gray-700 hover:bg-gray-300',
          ]"
        >
          {{ option.label }}
        </button>
      </div>

      <!-- 清除已完成按鈕 -->
      <button
        v-if="stats.completed > 0"
        @click="handleClearCompleted"
        class="text-red-600 hover:text-red-700 transition"
      >
        清除已完成
      </button>
    </div>
  </div>
</template>

<script setup>
// 定義 props
defineProps({
  modelValue: String,
  stats: Object,
});

// 定義事件
const emit = defineEmits(["update:modelValue", "clear-completed"]);

// 篩選選項
const filterOptions = [
  { label: "全部", value: "all" },
  { label: "進行中", value: "active" },
  { label: "已完成", value: "completed" },
];

// 方法 - 清除已完成
const handleClearCompleted = () => {
  if (confirm("確定要清除所有已完成的任務嗎？")) {
    emit("clear-completed");
  }
};
</script>
