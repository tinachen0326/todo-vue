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
            v-html="highlightedText"
          ></p>
          <div class="flex items-center gap-2 mt-1">
            <!-- 優先級標籤 -->
            <span
              :class="['text-xs px-2 py-0.5 rounded-full', priorityBadgeClass]"
            >
              {{ priorityLabel }}
            </span>
            <!-- 時間 -->
            <span class="text-xs text-gray-500">
              {{ formatDate(todo.createdAt) }}
            </span>
          </div>
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
  searchQuery: {
    type: String,
    default: "",
  },
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

// 優先級標籤樣式
const priorityBadgeClass = computed(() => {
  const classes = {
    high: "bg-red-100 text-red-600",
    medium: "bg-yellow-100 text-yellow-600",
    low: "bg-green-100 text-green-600",
  };
  return classes[props.todo.priority];
});

// 優先級標籤文字
const priorityLabel = computed(() => {
  const labels = {
    high: "高優先",
    medium: "中優先",
    low: "低優先",
  };
  return labels[props.todo.priority];
});

// 搜尋高亮文字
const highlightedText = computed(() => {
  if (!props.searchQuery) {
    return props.todo.text;
  }

  const regex = new RegExp(`(${props.searchQuery})`, "gi");
  return props.todo.text.replace(
    regex,
    '<mark class="bg-yellow-200">$1</mark>'
  );
});

// 其他方法保持不變...
const toggleComplete = () => {
  emit("update", {
    ...props.todo,
    completed: !props.todo.completed,
  });
};

const startEdit = async () => {
  isEditing.value = true;
  editText.value = props.todo.text;
  await nextTick();
  editInput.value?.focus();
};

const saveEdit = () => {
  if (editText.value.trim() && editText.value !== props.todo.text) {
    emit("update", {
      ...props.todo,
      text: editText.value.trim(),
    });
  }
  cancelEdit();
};

const cancelEdit = () => {
  isEditing.value = false;
  editText.value = "";
};

const deleteTodo = () => {
  if (confirm("確定要刪除這個任務嗎？")) {
    emit("delete", props.todo.id);
  }
};

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
