<template>
  <div
    :class="[
      'p-4 rounded-lg transition-all duration-200 hover:shadow-md',
      todo.completed ? 'bg-gray-50' : 'bg-white border',
      priorityClass,
    ]"
  >
    <div class="flex items-center justify-between">
      <!-- 左側：勾選框和文字 -->
      <div class="flex items-center flex-1">
        <!-- 勾選框 -->
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleComplete"
          class="w-5 h-5 text-indigo-600 rounded mr-3 cursor-pointer"
        />

        <!-- 編輯模式 -->
        <input
          v-if="isEditing"
          v-model="editText"
          @keyup.enter="saveEdit"
          @keyup.esc="cancelEdit"
          @blur="saveEdit"
          type="text"
          class="flex-1 px-2 py-1 border border-gray-300 rounded"
          ref="editInput"
        />

        <!-- 顯示模式 -->
        <div v-else class="flex-1">
          <p
            @dblclick="startEdit"
            :class="[
              'cursor-pointer',
              todo.completed && 'line-through opacity-60',
            ]"
          >
            {{ todo.text }}
          </p>
          <p class="text-xs text-gray-500 mt-1">
            {{ formatDate(todo.createdAt) }}
          </p>
        </div>
      </div>

      <!-- 右側：操作按鈕 -->
      <div class="flex gap-2 ml-4">
        <button
          @click="startEdit"
          class="text-blue-600 hover:text-blue-700"
          title="編輯"
        >
          ✏️
        </button>
        <button
          @click="deleteTodo"
          class="text-red-600 hover:text-red-700"
          title="刪除"
        >
          🗑️
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from "vue";

// 定義 props
const props = defineProps({
  todo: Object,
});

// 定義事件
const emit = defineEmits(["update", "delete"]);

// 響應式資料
const isEditing = ref(false);
const editText = ref("");
const editInput = ref(null);

// 計算屬性 - 優先級樣式
const priorityClass = computed(() => {
  const classes = {
    high: "border-l-4 border-red-500",
    medium: "border-l-4 border-yellow-500",
    low: "border-l-4 border-green-500",
  };
  return classes[props.todo.priority];
});

// 方法 - 切換完成狀態
const toggleComplete = () => {
  emit("update", {
    ...props.todo,
    completed: !props.todo.completed,
  });
};

// 方法 - 開始編輯
const startEdit = async () => {
  isEditing.value = true;
  editText.value = props.todo.text;
  await nextTick();
  editInput.value?.focus();
};

// 方法 - 儲存編輯
const saveEdit = () => {
  if (editText.value.trim() && editText.value !== props.todo.text) {
    emit("update", {
      ...props.todo,
      text: editText.value.trim(),
    });
  }
  cancelEdit();
};

// 方法 - 取消編輯
const cancelEdit = () => {
  isEditing.value = false;
  editText.value = "";
};

// 方法 - 刪除任務
const deleteTodo = () => {
  if (confirm("確定要刪除這個任務嗎？")) {
    emit("delete", props.todo.id);
  }
};

// 方法 - 格式化日期
const formatDate = (dateString) => {
  const date = new Date(dateString);
  const today = new Date();
  const diffTime = Math.abs(today - date);
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));

  if (diffDays === 0) return "今天";
  if (diffDays === 1) return "昨天";
  if (diffDays < 7) return `${diffDays} 天前`;
  return date.toLocaleDateString("zh-TW");
};
</script>
