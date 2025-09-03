# Todo List (任務清單)

## 專案簡介
這是一個使用 Vue 3 + Vite + Tailwind CSS 開發的 Todo List 任務清單網站，使用者可以新增、編輯、完成、刪除任務，方便管理日常工作或學習計畫。
此專案主要作為前端開發練習，展示 Vue 3 Composition API、Tailwind CSS 樣式設計、以及現代化前端開發流程的實作能力。

## 核心功能
- 任務管理 - 新增、編輯、刪除、標記完成
- 優先級設定 - 高、中、低三種優先級
- 搜尋功能 - 即時搜尋任務內容
- 篩選檢視 - 全部、進行中、已完成
- 資料持久化 - localStorage 本地儲存
- 即時統計 - 顯示任務完成狀態

## 使用技術
- 前端框架：Vue 3 (Composition API)
- 建構工具：Vite
- CSS 框架：Tailwind CSS
- 程式語言: JavaScript ES6+
- 資料儲存：localStorage
- 版本控制：Git
- 部署平台: Vercel

## 專案架構
```
todo-vue-practice/
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
- 原本使用Tailwind CSS v4，但為了確保與其他套件相容，降級到 Tailwind CSS v3
- 深入理解 Vue 3 響應式原理
- 掌握組件化開發思維
- 熟悉現代前端開發工具鏈
- 實踐良好的程式碼組織架構

## 未來優化
- 加入拖曳排序功能
- 任務到期提醒
- 深色模式切換
- 資料匯出/匯入
- PWA 離線使用
- 後端 API 整合
- 使用者認證系統

## 版權聲明
此專案僅供學習參考使用。
