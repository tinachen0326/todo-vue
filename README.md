# Todo List (任務清單)

## 專案簡介
這是一個使用 Vue 3 + Vite + Tailwind CSS 開發的 Todo List 任務清單網站，使用者可以新增、編輯、完成、刪除任務，方便管理日常工作或學習計畫。
此專案主要作為前端開發練習，展示 Vue 3 Composition API、Tailwind CSS 樣式設計、以及現代化前端開發流程的實作能力。

## 功能
### 基本功能
- 新增任務
- 標記任務為完成 / 未完成
- 刪除任務
- 編輯任務（雙擊或點擊編輯按鈕）
- 任務狀態切換動態更新

### 進階功能(未來增加)
- 任務優先級設定（高/中/低）
- 即時統計（總任務/進行中/已完成）
- 任務篩選（全部/進行中/已完成）
- 本地儲存（localStorage）
- 一鍵清除已完成任務
- 鍵盤快捷鍵支援（Enter 新增、Esc 取消編輯）
- 流暢的動畫效果
- 響應式設計（RWD）

## 使用技術
- 前端框架：Vue 3 (Composition API)
- 建構工具：Vite
- CSS 框架：Tailwind CSS
- 資料儲存：localStorage
- 版本控制：Git

## 安裝與使用
### 環境需求
- Node.js 16.0 或以上版本
- npm 7.0 或以上版本

## 專案架構
```
todo-list/
├── src/
│   ├── components/         # Vue 組件
│   │   ├── TodoHeader.vue  # 標題與統計顯示
│   │   ├── TodoInput.vue   # 任務輸入區域
│   │   ├── TodoFilter.vue  # 篩選器組件
│   │   ├── TodoList.vue    # 任務列表容器
│   │   └── TodoItem.vue    # 單一任務項目
│   ├── App.vue             # 主應用組件
│   ├── main.js             # 應用程式入口
│   └── style.css           # 全域樣式設定
├── public/                 # 靜態資源
├── index.html              # HTML 入口文件
├── vite.config.js          # Vite 設定檔
├── tailwind.config.js      # Tailwind CSS 設定
├── postcss.config.js       # PostCSS 設定
├── package.json            # 專案配置檔
└── README.md               # 專案說明文件
```

## 學習收穫
- 深入理解 Vue 3 響應式原理
- 原本使用Tailwind CSS v4，但為了確保與其他套件相容，降級到 Tailwind CSS v3
- 掌握組件化開發思維
- 熟悉現代前端開發工具鏈
- 實踐良好的程式碼組織架構
