# Module `project` — Phân Tích Chi Tiết

> **Đường dẫn:** `addons/project/`
> **Phiên bản:** 1.3 | **License:** LGPL-3 (Community Edition)
> **Danh mục:** Services/Project

---

## Mục Lục

1. [Tổng Quan](#1-tổng-quan)
2. [Phụ Thuộc](#2-phụ-thuộc)
3. [Kiến Trúc Models](#3-kiến-trúc-models)
4. [Chi Tiết Từng Model](#4-chi-tiết-từng-model)
5. [Hệ Thống Phân Quyền](#5-hệ-thống-phân-quyền)
6. [Frontend — Views & Components](#6-frontend--views--components)
7. [Wizards](#7-wizards)
8. [Modules Mở Rộng](#8-modules-mở-rộng)
9. [Luồng Hoạt Động Chính](#9-luồng-hoạt-động-chính)

---

## 1. Tổng Quan

Module `project` là ứng dụng quản lý dự án trung tâm của Odoo. Nó cung cấp toàn bộ hạ tầng để:

- Tạo và quản lý **Dự án** (Projects) theo giai đoạn (stage)
- Quản lý **Công việc** (Tasks) trong từng dự án với Kanban, List, Calendar, Graph, Pivot
- Theo dõi tiến độ qua **Milestones** và **Project Updates**
- Chia sẻ dự án với **khách hàng portal** (Project Sharing)
- Lặp lại công việc tự động qua **Recurring Tasks**
- Thu thập phản hồi khách hàng bằng **Customer Ratings**

---

## 2. Phụ Thuộc

### Modules phải có (depends)

| Module | Vai Trò |
|--------|---------|
| `analytic` | Gắn dự án với tài khoản phân tích (analytic account) để theo dõi chi phí/doanh thu |
| `base_setup` | Cài đặt hệ thống cơ bản |
| `mail` | Chatter, email, theo dõi thay đổi (tracking), hoạt động (activities) |
| `portal` | Cho phép khách hàng/portal user truy cập dự án từ bên ngoài |
| `rating` | Tính năng đánh giá sự hài lòng khách hàng |
| `resource` | Lịch làm việc (working time calendar) để tính thời gian thực tế |
| `web` | Framework frontend (OWL, views, actions) |
| `web_tour` | Tour hướng dẫn người dùng mới |
| `digest` | Gửi email tóm tắt định kỳ (KPI digest) |

---

## 3. Kiến Trúc Models

```
project.project          ← Dự Án
    │
    ├── project.project.stage     ← Giai đoạn của Dự Án (Kanban cột dự án)
    ├── project.milestone         ← Cột Mốc của Dự Án
    ├── project.update            ← Báo Cáo Cập Nhật Tiến Độ
    ├── project.collaborator      ← Người Cộng Tác (portal sharing)
    ├── project.tags              ← Nhãn (dùng chung cho project & task)
    │
    └── project.task             ← Công Việc
            │
            ├── project.task.type          ← Giai đoạn của Task (cột Kanban task)
            ├── project.task.recurrence    ← Cấu Hình Lặp Lại Task
            └── project.task.stage.personal ← Giai đoạn cá nhân (mỗi user một view)
```

**Tổng số models:** 8 models chính + 2 mixin/extension.

---

## 4. Chi Tiết Từng Model

### 4.1 `project.project` — Dự Án

**Kế thừa (inherit):**
- `portal.mixin` — Có URL portal riêng
- `mail.alias.mixin` — Nhận email tạo task tự động qua alias
- `rating.parent.mixin` — Quản lý đánh giá từ khách hàng
- `mail.thread` — Chatter + theo dõi thay đổi
- `mail.activity.mixin` — Hoạt động (lịch hẹn, cuộc gọi...)
- `mail.tracking.duration.mixin` — Đo thời gian ở mỗi stage
- `analytic.plan.fields.mixin` — Gắn kế hoạch phân tích

**Các fields quan trọng:**

| Field | Kiểu | Mô Tả |
|-------|------|--------|
| `name` | Char | Tên dự án |
| `user_id` | Many2one → `res.users` | Người quản lý dự án |
| `partner_id` | Many2one → `res.partner` | Khách hàng liên quan |
| `company_id` | Many2one → `res.company` | Công ty sở hữu |
| `stage_id` | Many2one → `project.project.stage` | Giai đoạn hiện tại của dự án |
| `type_ids` | Many2many → `project.task.type` | Các cột task (stage) áp dụng cho dự án này |
| `tasks` / `task_ids` | One2many → `project.task` | Toàn bộ task của dự án |
| `account_id` | Many2one → `account.analytic.account` | Tài khoản phân tích |
| `privacy_visibility` | Selection | Quyền hiển thị: `followers` / `employees` / `portal` |
| `allow_task_dependencies` | Boolean | Bật/tắt phụ thuộc giữa các task |
| `allow_milestones` | Boolean | Bật/tắt cột mốc milestone |
| `rating_active` | Boolean | Bật/tắt đánh giá khách hàng |
| `date_start` / `date` | Date | Ngày bắt đầu / kết thúc dự án |
| `task_count` | Integer (computed) | Tổng số task |
| `open_task_count` | Integer (computed) | Số task đang mở |
| `closed_task_count` | Integer (computed) | Số task đã đóng |
| `task_completion_percentage` | Float (computed) | % hoàn thành |
| `last_update_status` | Selection | Trạng thái mới nhất: On Track / At Risk / Off Track / On Hold / Done |
| `milestone_ids` | One2many → `project.milestone` | Các cột mốc |
| `collaborator_ids` | One2many → `project.collaborator` | Người cộng tác portal |
| `is_favorite` | Boolean (computed) | Dự án yêu thích của user hiện tại |
| `task_properties_definition` | PropertiesDefinition | Định nghĩa thuộc tính tùy chỉnh cho task |

---

### 4.2 `project.task` — Công Việc

**Kế thừa (inherit):**
- `portal.mixin` — Có URL portal
- `mail.thread.cc` — Chatter + CC email
- `mail.activity.mixin` — Hoạt động
- `rating.mixin` — Nhận đánh giá
- `mail.tracking.duration.mixin` — Đo thời gian ở mỗi stage
- `html.field.history.mixin` — Lưu lịch sử chỉnh sửa mô tả (HTML versioning)

**Thứ tự mặc định:** `priority desc, sequence, date_deadline asc, id desc`

**Các fields quan trọng:**

| Field | Kiểu | Mô Tả |
|-------|------|--------|
| `name` | Char | Tiêu đề task |
| `description` | Html | Mô tả chi tiết (có versioning) |
| `project_id` | Many2one → `project.project` | Dự án chứa task |
| `stage_id` | Many2one → `project.task.type` | Giai đoạn (cột kanban) |
| `user_ids` | Many2many → `res.users` | Những người được phân công |
| `partner_id` | Many2one → `res.partner` | Khách hàng liên quan |
| `priority` | Selection | `0` = Low, `1` = High |
| `state` | Selection | Trạng thái chi tiết (xem bảng bên dưới) |
| `is_closed` | Boolean (computed) | Task đã đóng hay chưa |
| `date_deadline` | Datetime | Hạn chót |
| `date_assign` | Datetime | Ngày phân công gần nhất |
| `allocated_hours` | Float | Số giờ phân bổ |
| `milestone_id` | Many2one → `project.milestone` | Cột mốc liên quan |
| `parent_id` | Many2one → `project.task` | Task cha |
| `child_ids` | One2many → `project.task` | Sub-tasks |
| `subtask_count` | Integer (computed) | Số sub-task |
| `depend_on_ids` | Many2many → `project.task` | Task bị chặn bởi (Blocked By) |
| `dependent_ids` | Many2many → `project.task` | Task đang bị task này chặn |
| `recurring_task` | Boolean | Có phải task lặp lại không |
| `recurrence_id` | Many2one → `project.task.recurrence` | Cấu hình lặp lại |
| `personal_stage_type_id` | Many2one → `project.task.type` | Giai đoạn cá nhân của user |
| `tag_ids` | Many2many → `project.tags` | Nhãn |
| `task_properties` | Properties | Thuộc tính tùy chỉnh (theo định nghĩa của project) |
| `working_hours_open` | Float (computed) | Số giờ làm việc để phân công |
| `working_hours_close` | Float (computed) | Số giờ làm việc để hoàn thành |

**Các trạng thái (state) của Task:**

| Giá Trị | Hiển Thị | Ý Nghĩa |
|---------|----------|---------|
| `01_in_progress` | In Progress | Đang thực hiện |
| `02_changes_requested` | Changes Requested | Yêu cầu chỉnh sửa |
| `03_approved` | Approved | Đã được duyệt |
| `1_done` | Done | Hoàn thành (CLOSED) |
| `1_canceled` | Cancelled | Đã hủy (CLOSED) |
| `04_waiting_normal` | Waiting | Đang chờ |

> `1_done` và `1_canceled` được định nghĩa trong `CLOSED_STATES` — đây là hai trạng thái đóng.

---

### 4.3 `project.task.type` — Giai Đoạn Task (Stage)

Dùng làm **cột trong Kanban view** của task. Một stage có thể dùng chung cho nhiều dự án.

| Field | Mô Tả |
|-------|--------|
| `name` | Tên giai đoạn (ví dụ: New, In Progress, Done) |
| `project_ids` | Many2many — các dự án sử dụng stage này |
| `sequence` | Thứ tự hiển thị |
| `fold` | Có thu gọn trên Kanban không |
| `mail_template_id` | Gửi email tự động khi task đến stage này |
| `rating_template_id` | Gửi email đánh giá khi task đến stage này |
| `user_id` | Chủ sở hữu (dùng cho personal stage) |
| `auto_validation_state` | Tự động thay đổi kanban state theo phản hồi khách hàng |

---

### 4.4 `project.project.stage` — Giai Đoạn Dự Án

Khác với `project.task.type`, đây là stage của **bản thân dự án** (hiển thị khi bật nhóm `group_project_stages`).

| Field | Mô Tả |
|-------|--------|
| `name` | Tên giai đoạn dự án |
| `sequence` | Thứ tự |
| `fold` | Dự án trong stage này được coi là đã đóng |
| `mail_template_id` | Email gửi khi dự án đến stage này |
| `company_id` | Giới hạn theo công ty |

---

### 4.5 `project.milestone` — Cột Mốc

Đánh dấu các điểm mốc quan trọng trong dự án. Task có thể được gắn vào milestone.

| Field | Mô Tả |
|-------|--------|
| `name` | Tên cột mốc |
| `project_id` | Dự án sở hữu |
| `deadline` | Hạn chót cột mốc |
| `is_reached` | Đã đạt được chưa |
| `reached_date` | Ngày đạt được (tính tự động) |
| `task_ids` | Các task liên quan |
| `task_count` / `done_task_count` | Số task / số task đã xong |
| `is_deadline_exceeded` | Quá hạn mà chưa đạt |
| `can_be_marked_as_done` | Có thể đánh dấu hoàn thành không (khi tất cả task đã đóng) |

---

### 4.6 `project.update` — Báo Cáo Tiến Độ

Cho phép Project Manager đăng báo cáo trạng thái định kỳ cho dự án.

| Field | Mô Tả |
|-------|--------|
| `name` | Tiêu đề báo cáo |
| `status` | Trạng thái: On Track / At Risk / Off Track / On Hold / Done |
| `progress` | % tiến độ (0–100) |
| `user_id` | Tác giả |
| `description` | Nội dung báo cáo (HTML) |
| `date` | Ngày báo cáo |
| `project_id` | Dự án liên quan |
| `task_count` / `closed_task_count` | Snapshot số task tại thời điểm báo cáo |

**Màu sắc trạng thái (`STATUS_COLOR`):**

| Trạng Thái | Màu |
|-----------|-----|
| On Track | Xanh lá (20) |
| At Risk | Cam (22) |
| Off Track | Đỏ (23) |
| On Hold | Xanh nhạt (21) |
| Done | Tím (24) |

---

### 4.7 `project.task.recurrence` — Cấu Hình Lặp Lại Task

Quản lý logic tự động tạo task lặp lại. Khi một task lặp lại được đánh dấu Done, task mới sẽ tự động được tạo.

| Field | Mô Tả |
|-------|--------|
| `repeat_interval` | Lặp mỗi N đơn vị |
| `repeat_unit` | Đơn vị: day / week / month / year |
| `repeat_type` | `forever` (mãi mãi) hoặc `until` (đến ngày cụ thể) |
| `repeat_until` | Ngày kết thúc lặp |

---

### 4.8 `project.collaborator` — Người Cộng Tác Portal

Quản lý danh sách người dùng portal được chia sẻ truy cập vào dự án.

| Field | Mô Tả |
|-------|--------|
| `project_id` | Dự án được chia sẻ (chỉ dự án `privacy_visibility = portal`) |
| `partner_id` | Người cộng tác |
| `limited_access` | Quyền truy cập hạn chế (chỉ xem task được giao) |

> Khi collaborator đầu tiên được tạo, hệ thống tự động kích hoạt các `ir.rule` và quyền truy cập portal. Khi không còn collaborator nào, tính năng bị tắt.

---

## 5. Hệ Thống Phân Quyền

### 5.1 Nhóm Quyền (Security Groups)

| Group ID | Tên | Mô Tả |
|----------|-----|--------|
| `group_project_user` | User | Người dùng thông thường — truy cập dự án được phép |
| `group_project_manager` | Administrator | Quản trị viên — xem tất cả dự án |
| `group_project_rating` | Use Rating on Project | Bật tính năng đánh giá khách hàng |
| `group_project_stages` | Use Stages on Project | Bật giai đoạn cho dự án (project stage) |
| `group_project_recurring_tasks` | Use Recurring Tasks | Bật task lặp lại |
| `group_project_task_dependencies` | Use Task Dependencies | Bật phụ thuộc giữa task |
| `group_project_milestone` | Use Milestones | Bật milestone |

### 5.2 Quy Tắc Truy Cập (Record Rules)

| Rule | Áp Dụng Cho | Quy Tắc |
|------|-------------|---------|
| `project_comp_rule` | Tất cả user | Chỉ thấy dự án cùng công ty |
| `project_project_manager_rule` | `group_project_manager` | Xem được tất cả dự án |
| `project_public_members_rule` | `group_user` | Dự án `followers` chỉ thấy nếu đang follow |

---

## 6. Frontend — Views & Components

### 6.1 Views của Dự Án (project.project)

| View | Mô Tả |
|------|--------|
| `project_project_kanban` | Kanban — hiển thị dự án dạng thẻ |
| `project_project_list` | Danh sách dự án |
| `project_project_calendar` | Lịch theo deadline |
| `project_update_kanban` | Kanban cho báo cáo tiến độ |
| `project_update_list` | Danh sách báo cáo tiến độ |

### 6.2 Views của Task (project.task)

| View | Mô Tả |
|------|--------|
| `project_task_kanban` | Kanban — cột theo stage, kéo thả |
| `project_task_list` | Danh sách task |
| `project_task_form` | Form chi tiết task |
| `project_task_calendar` | Lịch theo deadline |
| `project_task_graph` | Biểu đồ (lazy loaded) |
| `project_task_pivot` | Pivot table (lazy loaded) |
| `burndown_chart` | Biểu đồ Burndown Chart (lazy loaded) |

### 6.3 Components Tùy Chỉnh

| Component | Chức Năng |
|-----------|-----------|
| `project_task_state_selection` | Widget chọn state (thay thế selection mặc định) |
| `project_task_priority_switch_field` | Nút bật/tắt độ ưu tiên (Low/High) |
| `project_task_name_with_subtask_count_char_field` | Hiển thị tên task kèm số sub-task |
| `project_is_favorite` | Toggle yêu thích dự án |
| `project_many2one_field` | Widget Many2one đặc biệt cho dự án |
| `project_right_side_panel` | Panel bên phải của Kanban dự án |
| `project_state_selection` | Chọn trạng thái dự án (với màu sắc) |
| `project_status_with_color_selection` | Status kèm màu (On Track/At Risk...) |
| `subtask_kanban_list` | Danh sách sub-task trong kanban |

### 6.4 Project Sharing (Webclient riêng)

Module có một **webclient riêng** (`project.webclient`) dành cho giao diện chia sẻ portal. Đây là một ứng dụng frontend độc lập (không phải backend Odoo thông thường) được nhúng vào trang portal.

---

## 7. Wizards

| Wizard | Mô Tả |
|--------|--------|
| `project.share.wizard` | Chia sẻ dự án với người dùng portal — tạo collaborator |
| `project.share.collaborator.wizard` | Quản lý danh sách collaborator trong wizard chia sẻ |
| `project.task.type.delete.wizard` | Xóa stage task — xử lý task đang trong stage đó |
| `project.project.stage.delete.wizard` | Xóa stage dự án — xử lý dự án đang trong stage đó |

---

## 8. Modules Mở Rộng

Các module khác trong repo `odoo-core` phụ thuộc vào `project`:

| Module | Tính Năng Bổ Sung |
|--------|-------------------|
| `hr_timesheet` | Ghi nhận thời gian thực tế (timesheets) vào task |
| `project_account` | Tích hợp kế toán — doanh thu, chi phí theo dự án |
| `project_todo` | Tích hợp To-Do cá nhân với task |
| `project_sms` | Gửi SMS từ task/dự án |
| `project_stock` | Quản lý vật tư/tồn kho cho dự án |
| `project_mrp` | Tích hợp sản xuất (Manufacturing) với dự án |
| `project_mail_plugin` | Plugin email để tạo task từ email client |
| `website_project` | Trang website công khai cho dự án |
| `project_hr_skills` *(EE)* | Gắn kỹ năng nhân sự vào task/dự án |

---

## 9. Luồng Hoạt Động Chính

### 9.1 Vòng Đời Dự Án

```
[Tạo Dự Án] → [Thiết lập Stage, Milestone] → [Thêm Task]
      ↓
[Theo dõi tiến độ qua Updates] → [Cập nhật trạng thái On Track/At Risk...]
      ↓
[Milestone đạt được] → [Dự án hoàn thành / Chuyển stage Done]
```

### 9.2 Vòng Đời Task

```
[Tạo Task] → [Phân công user] → [Kéo qua các Stage trên Kanban]
      ↓
[Cập nhật state: In Progress → Approved → Done / Cancelled]
      ↓
[Task lặp lại: tự động tạo task mới khi Done]
```

### 9.3 Chia Sẻ Với Khách Hàng (Project Sharing)

```
[Bật privacy_visibility = portal] → [Wizard chia sẻ]
      ↓
[Tạo project.collaborator] → [Kích hoạt ir.rule portal]
      ↓
[Khách hàng nhận email] → [Truy cập giao diện portal]
      ↓
[Xem/chỉnh sửa task được phân công trong project.webclient]
```

### 9.4 Đánh Giá Khách Hàng (Customer Ratings)

```
[Bật rating_active trên Project] → [Cấu hình rating_status]
      ↓
[Task chuyển sang stage có rating_template_id]
      → [Email đánh giá tự động gửi cho khách hàng]
      ↓
[Phản hồi tốt → state = Approved]
[Phản hồi trung bình/xấu → state = Changes Requested]
```

---

*Tài liệu được tổng hợp từ mã nguồn `addons/project/` trong repository `odoo-core`.*
