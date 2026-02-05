# 多專案甘特圖進度系統（MVP 規劃）

本文件提供一個可落地的 MVP（最小可行產品）規格，目標是讓團隊可以同時管理多個專案、以甘特圖呈現排程、並持續更新目前進度。

## 1. 產品目標

- 同時管理多個專案（Project Portfolio）。
- 每個專案可拆分任務（Task）與里程碑（Milestone）。
- 以甘特圖查看：開始日期、結束日期、依賴關係、目前進度。
- 支援多人協作與權限控管（至少 Owner / Member）。
- 提供狀態快照（例如：落後項目、本週進度、即將到期任務）。

## 2. 角色與權限（MVP）

- **Owner（專案擁有者）**
  - 建立/刪除專案
  - 管理成員
  - 編輯所有任務
- **Member（專案成員）**
  - 可檢視專案
  - 可新增/更新自己被指派的任務進度

## 3. 核心資料模型

> 下列欄位可對應到 SQL 或 NoSQL；若用 SQL，建議先以 PostgreSQL 實作。

### 3.1 Project

- `id` (uuid)
- `name` (string)
- `description` (text)
- `ownerId` (uuid)
- `startDate` (date)
- `endDate` (date)
- `status` (enum: planning | active | done | archived)
- `createdAt`, `updatedAt`

### 3.2 Task

- `id` (uuid)
- `projectId` (uuid)
- `name` (string)
- `assigneeId` (uuid, nullable)
- `startDate` (date)
- `endDate` (date)
- `progress` (integer, 0~100)
- `priority` (enum: low | medium | high)
- `status` (enum: todo | doing | blocked | done)
- `parentTaskId` (uuid, nullable)
- `createdAt`, `updatedAt`

### 3.3 TaskDependency

- `id` (uuid)
- `projectId` (uuid)
- `fromTaskId` (uuid)
- `toTaskId` (uuid)
- `type` (enum: finish_to_start)  
  > MVP 先只做最常見的「前一任務完成後才能開始」。

### 3.4 ProjectMember

- `id` (uuid)
- `projectId` (uuid)
- `userId` (uuid)
- `role` (enum: owner | member)

## 4. MVP 功能清單

### 4.1 專案層級

- 新增專案
- 編輯專案基本資訊（名稱、日期、狀態）
- 專案列表 + 篩選（active / done）

### 4.2 任務層級

- 新增任務
- 調整任務日期（拖曳或表單）
- 更新進度（0~100）
- 設定依賴關係
- 變更任務狀態（todo/doing/blocked/done）

### 4.3 甘特圖視圖

- 依專案顯示時間軸
- 任務條顯示 `名稱 + 進度%`
- 依賴線顯示
- 今日線（Today marker）
- 縮放（日 / 週）

### 4.4 儀表板（簡版）

- 專案整體完成度：`平均 progress`
- 逾期任務數（`endDate < today && status != done`）
- 本週即將到期任務數

## 5. API 設計（範例）

### Project

- `GET /api/projects`
- `POST /api/projects`
- `GET /api/projects/:projectId`
- `PATCH /api/projects/:projectId`
- `DELETE /api/projects/:projectId`

### Task

- `GET /api/projects/:projectId/tasks`
- `POST /api/projects/:projectId/tasks`
- `PATCH /api/tasks/:taskId`
- `DELETE /api/tasks/:taskId`

### Dependency

- `POST /api/projects/:projectId/dependencies`
- `DELETE /api/dependencies/:dependencyId`

### Dashboard

- `GET /api/projects/:projectId/summary`

## 6. 建議技術路線

- 前端：React + 甘特圖元件（例如 dhtmlx-gantt、frappe-gantt 或 Syncfusion）
- 後端：Node.js + Express
- DB：PostgreSQL
- 認證：JWT / OAuth（Google 或企業 SSO）

## 7. 進度計算規則（建議）

- 任務進度由成員手動更新，範圍 0~100。
- 專案進度可先用「任務平均」：

```txt
projectProgress = sum(task.progress) / taskCount
```

- 若後續需要更準確，可改成「工時加權」：

```txt
projectProgress = sum(task.progress * task.estimatedHours) / sum(task.estimatedHours)
```

## 8. 開發排程建議（4 週）

- 第 1 週：資料模型 + CRUD API（Project / Task）
- 第 2 週：甘特圖整合、任務拖曳改期
- 第 3 週：依賴關係、儀表板摘要
- 第 4 週：權限、測試、部署

## 9. 驗收標準（MVP）

- 可建立至少 3 個專案且互不干擾。
- 每個專案至少 20 筆任務，甘特圖可正常載入。
- 可更新任務進度並即時反映在甘特圖與摘要面板。
- 可識別逾期任務與即將到期任務。

## 10. 下一步

若你希望，我可以在下一版直接提供：

1. PostgreSQL migration（`projects`, `tasks`, `dependencies`, `project_members`）
2. Express API 樣板程式碼
3. React 甘特圖頁面雛形（含 mock data）
