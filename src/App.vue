<template>
  <!-- Loading 畫面 -->
  <div
    v-if="isLoading"
    class="flex flex-col justify-center items-center h-screen bg-gradient-to-br from-blue-400 to-purple-600"
  >
    <div
      class="animate-spin rounded-full h-16 w-16 border-b-2 border-white mb-4"
    ></div>
    <div class="text-2xl text-white font-semibold">載入中...</div>
  </div>

  <!-- 主要內容 -->
  <div v-else class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100">
    <div class="container mx-auto px-4 py-8 max-w-4xl">
      <!-- 應用標題 -->
      <header class="text-center mb-8">
        <h1 class="text-4xl font-bold text-gray-800 mb-2">
          <span class="text-5xl"></span>
          我的任務清單
        </h1>
        <p class="text-gray-600 text-lg">高效管理每日任務</p>
      </header>

      <!-- 錯誤訊息 -->
      <div
        v-if="errorMessage"
        class="mb-6 bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded-lg shadow-md"
      >
        <div class="flex items-center">
          <span>{{ errorMessage }}</span>
        </div>
      </div>

      <!-- 新增任務表單 -->
      <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <form @submit.prevent="addTodo" class="space-y-4">
          <div class="flex flex-col sm:flex-row gap-4">
            <input
              ref="inputRef"
              v-model="newTodo.text"
              type="text"
              placeholder="輸入新任務..."
              class="flex-1 px-4 py-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent transition-all duration-200"
              maxlength="100"
            />
            <select
              v-model="newTodo.priority"
              class="px-4 py-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-200"
            >
              <option value="low">次要</option>
              <option value="medium">一般</option>
              <option value="high">緊急</option>
            </select>
            <button
              type="submit"
              class="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 transition-colors duration-200 font-medium whitespace-nowrap"
            >
              新增任務
            </button>
          </div>
          <div class="text-sm text-gray-500">
            {{ newTodo.text.length }}/100 字元
          </div>
        </form>
      </div>

      <!-- 搜尋和篩選 -->
      <div class="bg-white rounded-lg shadow-lg p-6 mb-6">
        <div
          class="flex flex-col sm:flex-row gap-4 items-start sm:items-center justify-between"
        >
          <!-- 搜尋框 -->
          <div class="flex-1">
            <input
              v-model="searchQuery"
              type="text"
              placeholder="搜尋任務..."
              class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-200"
            />
          </div>

          <!-- 篩選按鈕 -->
          <div class="flex gap-2">
            <button
              v-for="filterOption in filterOptions"
              :key="filterOption.value"
              @click="filter = filterOption.value"
              :class="[
                'px-4 py-2 rounded-lg font-medium transition-colors duration-200',
                filter === filterOption.value
                  ? 'bg-blue-600 text-white shadow-md'
                  : 'bg-gray-100 text-gray-700 hover:bg-gray-200',
              ]"
            >
              {{ filterOption.label }}
            </button>
          </div>
        </div>

        <!-- 統計信息 -->
        <div class="mt-4 flex flex-wrap gap-4 text-sm text-gray-600">
          <span class="flex items-center gap-1">
            <span class="w-2 h-2 bg-blue-500 rounded-full"></span>
            總計: {{ statistics.total }}
          </span>
          <span class="flex items-center gap-1">
            <span class="w-2 h-2 bg-yellow-500 rounded-full"></span>
            進行中: {{ statistics.active }}
          </span>
          <span class="flex items-center gap-1">
            <span class="w-2 h-2 bg-green-500 rounded-full"></span>
            已完成: {{ statistics.completed }}
          </span>
          <span v-if="searchQuery" class="flex items-center gap-1">
            <span class="w-2 h-2 bg-purple-500 rounded-full"></span>
            搜尋結果: {{ filteredTodos.length }}
          </span>
        </div>
      </div>

      <!-- 任務列表 -->
      <div class="bg-white rounded-lg shadow-lg p-6">
        <!-- 空狀態設計 -->
        <div v-if="filteredTodos.length === 0" class="text-center py-12">
          <div v-if="searchQuery || filter !== 'all'">
            <!-- 篩選後無結果 -->
            <p class="text-gray-500 text-lg mb-2">沒有找到符合條件的任務</p>
            <p class="text-gray-400 text-sm mb-4">嘗試修改搜尋條件或清除篩選</p>
            <button
              @click="clearFilters"
              class="px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition-colors duration-200"
            >
              清除篩選
            </button>
          </div>
          <div v-else>
            <!-- 完全沒有任務 -->
            <p class="text-gray-500 text-lg mb-4">開始你的第一個任務吧！</p>
            <p class="text-gray-400 text-sm">點擊上方輸入框新增任務</p>
          </div>
        </div>

        <!-- 任務列表 -->
        <TransitionGroup v-else name="fade" tag="div" class="space-y-3">
          <div
            v-for="todo in filteredTodos"
            :key="todo.id"
            :class="[
              'task-item bg-gray-50 rounded-lg p-4 border-l-4 transition-all duration-200 hover:shadow-md',
              getPriorityClass(todo.priority),
              todo.completed ? 'opacity-60' : '',
            ]"
          >
            <div class="flex items-start gap-3">
              <!-- 完成checkbox -->
              <input
                :id="`todo-${todo.id}`"
                v-model="todo.completed"
                @change="saveTodos"
                type="checkbox"
                class="w-5 h-5 text-blue-600 rounded focus:ring-blue-500 mt-1 transition-all duration-200"
              />

              <!-- 任務內容 -->
              <div class="flex-1 min-w-0" v-if="editingId !== todo.id">
                <label
                  :for="`todo-${todo.id}`"
                  :class="[
                    'cursor-pointer text-lg break-words',
                    todo.completed
                      ? 'line-through text-gray-500'
                      : 'text-gray-800',
                  ]"
                >
                  {{ todo.text }}
                </label>
                <div class="flex items-center gap-2 mt-2 text-sm text-gray-500">
                  <span
                    :class="{
                      'text-red-600': todo.priority === 'high',
                      'text-yellow-600': todo.priority === 'medium',
                      'text-green-600': todo.priority === 'low',
                    }"
                  >
                    {{ getPriorityText(todo.priority) }}
                  </span>
                  <span>•</span>
                  <span>{{ formatDate(todo.createdAt) }}</span>
                </div>
              </div>

              <!-- 編輯狀態 -->
              <div v-else class="flex-1">
                <input
                  ref="editInputRef"
                  v-model="editText"
                  @blur="saveEdit(todo)"
                  @keyup.enter="saveEdit(todo)"
                  @keyup.escape="cancelEdit"
                  type="text"
                  class="w-full px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-blue-500 transition-all duration-200"
                  maxlength="100"
                />
              </div>

              <!-- 操作按鈕 -->
              <div class="flex gap-2 ml-2">
                <button
                  v-if="editingId !== todo.id && !todo.completed"
                  @click="startEdit(todo)"
                  class="p-2 text-blue-600 hover:bg-blue-100 rounded-lg transition-colors duration-200"
                  title="編輯任務"
                >
                  編輯
                </button>
                <button
                  @click="deleteTodo(todo.id)"
                  class="p-2 text-red-600 hover:bg-red-100 rounded-lg transition-colors duration-200"
                  title="刪除任務"
                >
                  刪除
                </button>
              </div>
            </div>
          </div>
        </TransitionGroup>

        <!-- 批量操作 -->
        <div v-if="todos.length > 0" class="mt-6 pt-4 border-t border-gray-200">
          <div
            class="flex flex-col sm:flex-row gap-3 justify-between items-center"
          >
            <div class="text-sm text-gray-600">
              鍵盤快捷鍵: Ctrl+K (聚焦搜尋), Ctrl+D (清除已完成)
            </div>
            <div class="flex gap-2">
              <button
                v-if="statistics.active > 0"
                @click="toggleAllCompleted"
                class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors duration-200 text-sm"
              >
                全部完成
              </button>
              <button
                v-if="statistics.completed > 0"
                @click="clearCompleted"
                class="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 transition-colors duration-200 text-sm"
              >
                清除已完成 ({{ statistics.completed }})
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- 頁腳 -->
      <footer class="text-center mt-8 text-gray-600">
        <p class="text-sm">
          使用 Vue 3 + Tailwind CSS 打造 |
          <a href="#" class="text-blue-600 hover:text-blue-800 underline"
            >查看原始碼</a
          >
        </p>
      </footer>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick, watch } from "vue";

// 響應式狀態
const isLoading = ref(true);
const todos = ref([]);
const newTodo = ref({ text: "", priority: "medium" });
const searchQuery = ref("");
const filter = ref("all");
const editingId = ref(null);
const editText = ref("");
const errorMessage = ref("");

// 模板引用
const inputRef = ref(null);
const editInputRef = ref(null);

// 篩選選項
const filterOptions = [
  { value: "all", label: "全部" },
  { value: "active", label: "進行中" },
  { value: "completed", label: "已完成" },
];

// 計算屬性
const filteredTodos = computed(() => {
  let result = todos.value;

  // 先根據搜尋過濾
  if (searchQuery.value) {
    result = result.filter((todo) =>
      todo.text.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
  }

  // 再根據狀態過濾
  switch (filter.value) {
    case "active":
      return result.filter((todo) => !todo.completed);
    case "completed":
      return result.filter((todo) => todo.completed);
    default:
      return result;
  }
});

const statistics = computed(() => ({
  total: todos.value.length,
  active: todos.value.filter((todo) => !todo.completed).length,
  completed: todos.value.filter((todo) => todo.completed).length,
}));

// 方法
const addTodo = () => {
  try {
    if (!newTodo.value.text || newTodo.value.text.trim().length === 0) {
      throw new Error("任務內容不能為空");
    }

    if (newTodo.value.text.length > 100) {
      throw new Error("任務內容不能超過 100 個字");
    }

    todos.value.unshift({
      id: Date.now(),
      text: newTodo.value.text.trim(),
      completed: false,
      priority: newTodo.value.priority,
      createdAt: new Date().toISOString(),
    });

    // 重置表單
    newTodo.value = { text: "", priority: "medium" };
    errorMessage.value = "";

    // 聚焦輸入框
    nextTick(() => {
      inputRef.value?.focus();
    });
  } catch (error) {
    errorMessage.value = error.message;
    setTimeout(() => {
      errorMessage.value = "";
    }, 3000);
  }
};

const deleteTodo = (id) => {
  if (confirm("確定要刪除這個任務嗎？")) {
    todos.value = todos.value.filter((todo) => todo.id !== id);
  }
};

const startEdit = (todo) => {
  editingId.value = todo.id;
  editText.value = todo.text;
  nextTick(() => {
    editInputRef.value?.focus();
    editInputRef.value?.select();
  });
};

const saveEdit = (todo) => {
  if (editText.value.trim()) {
    todo.text = editText.value.trim();
  }
  editingId.value = null;
  editText.value = "";
};

const cancelEdit = () => {
  editingId.value = null;
  editText.value = "";
};

const toggleAllCompleted = () => {
  const allCompleted = statistics.value.active === 0;
  todos.value.forEach((todo) => {
    todo.completed = !allCompleted;
  });
};

const clearCompleted = () => {
  if (confirm(`確定要清除 ${statistics.value.completed} 個已完成的任務嗎？`)) {
    todos.value = todos.value.filter((todo) => !todo.completed);
  }
};

const clearFilters = () => {
  searchQuery.value = "";
  filter.value = "all";
};

const getPriorityClass = (priority) => {
  const classes = {
    high: "border-red-500 bg-red-50",
    medium: "border-yellow-500 bg-yellow-50",
    low: "border-green-500 bg-green-50",
  };
  return classes[priority] || classes.medium;
};

const getPriorityText = (priority) => {
  const priorityMap = {
    high: "緊急",
    medium: "一般",
    low: "次要",
  };
  return priorityMap[priority] || "一般";
};

const formatDate = (dateString) => {
  const date = new Date(dateString);
  const now = new Date();
  const diffTime = Math.abs(now - date);
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));

  if (diffDays === 0) {
    return "今天";
  } else if (diffDays === 1) {
    return "昨天";
  } else if (diffDays < 7) {
    return `${diffDays} 天前`;
  } else {
    return date.toLocaleDateString("zh-TW");
  }
};

// 本地存儲方法
const saveTodos = () => {
  try {
    localStorage.setItem("todos", JSON.stringify(todos.value));
  } catch (error) {
    console.error("儲存失敗:", error);
    errorMessage.value = "儲存失敗，請重試";
    setTimeout(() => {
      errorMessage.value = "";
    }, 3000);
  }
};

const loadTodos = () => {
  try {
    const saved = localStorage.getItem("todos");
    if (saved) {
      todos.value = JSON.parse(saved);
    }
  } catch (error) {
    console.error("載入失敗:", error);
    todos.value = [];
    errorMessage.value = "載入資料失敗";
    setTimeout(() => {
      errorMessage.value = "";
    }, 3000);
  }
};

// 鍵盤快捷鍵
const setupKeyboardShortcuts = () => {
  document.addEventListener("keydown", (e) => {
    // Ctrl/Cmd + K：聚焦搜尋
    if ((e.ctrlKey || e.metaKey) && e.key === "k") {
      e.preventDefault();
      const searchInput = document.querySelector('input[placeholder*="搜尋"]');
      searchInput?.focus();
    }

    // Ctrl/Cmd + D：清除已完成
    if ((e.ctrlKey || e.metaKey) && e.key === "d") {
      e.preventDefault();
      if (statistics.value.completed > 0) {
        clearCompleted();
      }
    }

    // Ctrl/Cmd + N：聚焦新增任務
    if ((e.ctrlKey || e.metaKey) && e.key === "n") {
      e.preventDefault();
      inputRef.value?.focus();
    }
  });
};

// 生命週期
onMounted(() => {
  isLoading.value = true;

  // 模擬載入時間
  setTimeout(() => {
    loadTodos();
    setupKeyboardShortcuts();
    isLoading.value = false;

    // 初始聚焦
    nextTick(() => {
      inputRef.value?.focus();
    });
  }, 500);
});

// 監聽 todos 變化自動保存
watch(todos, saveTodos, { deep: true });
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

/* 任務項目 hover 效果 */
.task-item:hover {
  transform: translateY(-1px);
}

/* 自定義滾動條 */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 10px;
}

::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style>
