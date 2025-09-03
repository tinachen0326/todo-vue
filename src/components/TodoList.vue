<template>
  <div class="bg-white rounded-lg shadow-md p-6">
    <!-- 不同情況的空狀態 -->
    <div v-if="todos.length === 0" class="text-center py-12">
      <div v-if="isFiltered">
        <!-- 篩選後無結果 -->
        <div class="text-6xl mb-4 animate-bounce">🔍</div>
        <p class="text-gray-500 text-lg mb-2">沒有找到符合條件的任務</p>
        <p class="text-gray-400 text-sm mb-4">嘗試修改搜尋條件或清除篩選器</p>
        <button
          @click="$emit('clear-filters')"
          class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors duration-200 text-sm font-medium"
        >
          清除篩選
        </button>
      </div>
      <div v-else>
        <!-- 完全沒有任務 -->
        <div class="text-6xl mb-4 animate-pulse">🎯</div>
        <p class="text-gray-500 text-lg mb-4">開始你的第一個任務吧！</p>
        <p class="text-gray-400 text-sm mb-6">點擊上方輸入框新增任務</p>
        <div class="flex flex-col items-center space-y-3">
          <div class="flex items-center text-xs text-gray-400">
            <span class="mr-2">💡</span>
            <span>小提示：你可以設定任務的優先級</span>
          </div>
          <div class="flex items-center text-xs text-gray-400">
            <span class="mr-2">⌨️</span>
            <span>快捷鍵 Ctrl+N 可快速新增任務</span>
          </div>
        </div>
      </div>
    </div>

    <!-- 任務列表 -->
    <TransitionGroup v-else name="fade" tag="div" class="space-y-3">
      <div
        v-for="todo in todos"
        :key="todo.id"
        :class="[
          'todo-item group bg-gray-50 rounded-lg p-4 border-l-4 transition-all duration-200 hover:shadow-md hover:bg-gray-100',
          getPriorityClass(todo.priority),
          todo.completed ? 'opacity-60 bg-gray-100' : '',
        ]"
      >
        <div class="flex items-start gap-3">
          <!-- 完成 checkbox -->
          <div class="flex-shrink-0 mt-1">
            <input
              :id="`todo-${todo.id}`"
              :checked="todo.completed"
              @change="$emit('toggle-todo', todo.id)"
              type="checkbox"
              class="w-5 h-5 text-blue-600 bg-white rounded border-gray-300 focus:ring-blue-500 focus:ring-2 transition-all duration-200 cursor-pointer"
            />
          </div>

          <!-- 任務內容 -->
          <div class="flex-1 min-w-0" v-if="editingId !== todo.id">
            <label
              :for="`todo-${todo.id}`"
              :class="[
                'cursor-pointer text-lg break-words leading-relaxed',
                todo.completed
                  ? 'line-through text-gray-500'
                  : 'text-gray-800 hover:text-gray-900',
              ]"
            >
              {{ todo.text }}
            </label>
            <div class="flex items-center gap-2 mt-2 text-sm">
              <!-- 優先級標籤 -->
              <span
                :class="[
                  'px-2 py-1 rounded-full text-xs font-medium',
                  getPriorityBadgeClass(todo.priority),
                ]"
              >
                {{ getPriorityText(todo.priority) }}
              </span>

              <!-- 分隔符 -->
              <span class="text-gray-300">•</span>

              <!-- 創建時間 -->
              <span class="text-gray-500 flex items-center">
                <svg
                  class="w-4 h-4 mr-1"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                  ></path>
                </svg>
                {{ formatDate(todo.createdAt) }}
              </span>

              <!-- 任務狀態指示器 -->
              <span
                v-if="todo.completed"
                class="text-green-600 flex items-center text-xs"
              >
                <svg
                  class="w-4 h-4 mr-1"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
                  ></path>
                </svg>
                已完成
              </span>
            </div>
          </div>

          <!-- 編輯狀態 -->
          <div v-else class="flex-1">
            <input
              ref="editInput"
              v-model="editText"
              @blur="saveEdit(todo)"
              @keyup.enter="saveEdit(todo)"
              @keyup.escape="cancelEdit"
              type="text"
              class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200"
              maxlength="100"
              placeholder="輸入任務內容..."
            />
            <div class="text-xs text-gray-500 mt-1">
              按 Enter 保存，按 Esc 取消
            </div>
          </div>

          <!-- 操作按鈕 -->
          <div
            class="flex-shrink-0 flex items-center gap-1 opacity-0 group-hover:opacity-100 transition-opacity duration-200"
          >
            <!-- 編輯按鈕 -->
            <button
              v-if="editingId !== todo.id && !todo.completed"
              @click="startEdit(todo)"
              class="p-2 text-gray-400 hover:text-blue-600 hover:bg-blue-50 rounded-lg transition-all duration-200"
              title="編輯任務"
            >
              <svg
                class="w-4 h-4"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"
                ></path>
              </svg>
            </button>

            <!-- 複製按鈕 -->
            <button
              @click="duplicateTodo(todo)"
              class="p-2 text-gray-400 hover:text-green-600 hover:bg-green-50 rounded-lg transition-all duration-200"
              title="複製任務"
            >
              <svg
                class="w-4 h-4"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"
                ></path>
              </svg>
            </button>

            <!-- 刪除按鈕 -->
            <button
              @click="deleteTodo(todo.id)"
              class="p-2 text-gray-400 hover:text-red-600 hover:bg-red-50 rounded-lg transition-all duration-200"
              title="刪除任務"
            >
              <svg
                class="w-4 h-4"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                ></path>
              </svg>
            </button>
          </div>
        </div>

        <!-- 任務進度條 (僅高優先級任務顯示) -->
        <div v-if="todo.priority === 'high' && !todo.completed" class="mt-3">
          <div
            class="flex items-center justify-between text-xs text-gray-600 mb-1"
          >
            <span>高優先級任務</span>
            <span>{{ getDaysFromCreated(todo.createdAt) }} 天前創建</span>
          </div>
          <div class="w-full bg-gray-200 rounded-full h-2">
            <div
              class="bg-red-500 h-2 rounded-full transition-all duration-300"
              :style="{
                width:
                  Math.min(getDaysFromCreated(todo.createdAt) * 10, 100) + '%',
              }"
            ></div>
          </div>
        </div>
      </div>
    </TransitionGroup>

    <!-- 列表底部統計 -->
    <div v-if="todos.length > 0" class="mt-6 pt-4 border-t border-gray-200">
      <div
        class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3"
      >
        <!-- 統計信息 -->
        <div class="flex flex-wrap gap-4 text-sm text-gray-600">
          <div class="flex items-center gap-1">
            <div class="w-2 h-2 bg-blue-500 rounded-full"></div>
            <span>總計 {{ totalCount }}</span>
          </div>
          <div class="flex items-center gap-1">
            <div class="w-2 h-2 bg-yellow-500 rounded-full"></div>
            <span>進行中 {{ activeCount }}</span>
          </div>
          <div class="flex items-center gap-1">
            <div class="w-2 h-2 bg-green-500 rounded-full"></div>
            <span>已完成 {{ completedCount }}</span>
          </div>
        </div>

        <!-- 批量操作按鈕 -->
        <div class="flex gap-2">
          <button
            v-if="activeCount > 0"
            @click="$emit('toggle-all')"
            class="px-3 py-1 bg-green-600 text-white text-sm rounded-lg hover:bg-green-700 transition-colors duration-200"
          >
            全部完成
          </button>
          <button
            v-if="completedCount > 0"
            @click="$emit('clear-completed')"
            class="px-3 py-1 bg-red-600 text-white text-sm rounded-lg hover:bg-red-700 transition-colors duration-200"
          >
            清除已完成 ({{ completedCount }})
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from "vue";

// Props
const props = defineProps({
  todos: {
    type: Array,
    required: true,
  },
  isFiltered: {
    type: Boolean,
    default: false,
  },
});

// Emits
const emit = defineEmits([
  "toggle-todo",
  "edit-todo",
  "delete-todo",
  "duplicate-todo",
  "toggle-all",
  "clear-completed",
  "clear-filters",
]);

// 響應式狀態
const editingId = ref(null);
const editText = ref("");
const editInput = ref(null);

// 計算屬性
const totalCount = computed(() => props.todos.length);
const activeCount = computed(
  () => props.todos.filter((todo) => !todo.completed).length
);
const completedCount = computed(
  () => props.todos.filter((todo) => todo.completed).length
);

// 方法
const startEdit = (todo) => {
  editingId.value = todo.id;
  editText.value = todo.text;
  nextTick(() => {
    editInput.value?.focus();
    editInput.value?.select();
  });
};

const saveEdit = (todo) => {
  if (editText.value.trim() && editText.value.trim() !== todo.text) {
    emit("edit-todo", {
      ...todo,
      text: editText.value.trim(),
    });
  }
  editingId.value = null;
  editText.value = "";
};

const cancelEdit = () => {
  editingId.value = null;
  editText.value = "";
};

const deleteTodo = (id) => {
  if (confirm("確定要刪除這個任務嗎？")) {
    emit("delete-todo", id);
  }
};

const duplicateTodo = (todo) => {
  emit("duplicate-todo", {
    ...todo,
    text: `${todo.text} (副本)`,
    id: Date.now(),
    completed: false,
    createdAt: new Date().toISOString(),
  });
};

const getPriorityClass = (priority) => {
  const classes = {
    high: "border-red-500 bg-red-50",
    medium: "border-yellow-500 bg-yellow-50",
    low: "border-green-500 bg-green-50",
  };
  return classes[priority] || classes.medium;
};

const getPriorityBadgeClass = (priority) => {
  const classes = {
    high: "bg-red-100 text-red-800 border border-red-200",
    medium: "bg-yellow-100 text-yellow-800 border border-yellow-200",
    low: "bg-green-100 text-green-800 border border-green-200",
  };
  return classes[priority] || classes.medium;
};

const getPriorityText = (priority) => {
  const priorityMap = {
    high: "🔴 高優先級",
    medium: "🟡 中優先級",
    low: "🟢 低優先級",
  };
  return priorityMap[priority] || "🟡 中優先級";
};

const formatDate = (dateString) => {
  const date = new Date(dateString);
  const now = new Date();
  const diffTime = Math.abs(now - date);
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));
  const diffHours = Math.floor(diffTime / (1000 * 60 * 60));
  const diffMinutes = Math.floor(diffTime / (1000 * 60));

  if (diffMinutes < 1) {
    return "剛才";
  } else if (diffMinutes < 60) {
    return `${diffMinutes} 分鐘前`;
  } else if (diffHours < 24) {
    return `${diffHours} 小時前`;
  } else if (diffDays === 0) {
    return "今天";
  } else if (diffDays === 1) {
    return "昨天";
  } else if (diffDays < 7) {
    return `${diffDays} 天前`;
  } else {
    return date.toLocaleDateString("zh-TW");
  }
};

const getDaysFromCreated = (dateString) => {
  const date = new Date(dateString);
  const now = new Date();
  const diffTime = Math.abs(now - date);
  return Math.floor(diffTime / (1000 * 60 * 60 * 24));
};
</script>

<style scoped>
/* 動畫效果 */
.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.fade-move {
  transition: transform 0.3s ease;
}

/* 任務項目動畫 */
.todo-item {
  transform-origin: center;
}

.todo-item:hover {
  transform: translateY(-1px);
}

/* 自定義 checkbox 樣式 */
input[type="checkbox"]:checked {
  background-image: url("data:image/svg+xml,%3csvg viewBox='0 0 16 16' fill='white' xmlns='http://www.w3.org/2000/svg'%3e%3cpath d='m13.854 3.646-7.5 7.5a.5.5 0 0 1-.708 0l-3.5-3.5a.5.5 0 1 1 .708-.708L6 10.293l7.146-7.147a.5.5 0 0 1 .708.708z'/%3e%3c/svg%3e");
}

/* 響應式調整 */
@media (max-width: 640px) {
  .todo-item {
    padding: 0.75rem;
  }

  .group-hover\:opacity-100 {
    opacity: 1; /* 在手機上始終顯示操作按鈕 */
  }
}

/* 滾動條樣式 */
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style>
