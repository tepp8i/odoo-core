# PLAN.md — Gantt/Timeline View cho Odoo Project Module

## Mục tiêu

Xây dựng Gantt / Timeline view cho Odoo Project module dưới dạng addon riêng `project_gantt`,
cho phép quản lý và theo dõi tiến độ dự án và tasks theo trục thời gian.

---

## Scope

### In Scope

| # | Tính năng |
|---|---|
| 1 | **Multi-project Gantt** — mỗi project là 1 row, bar dựa trên `date_start` → `date` |
| 2 | **Task Gantt** — mỗi task là 1 row, bar dựa trên `custom_date_start` → `date_deadline` (fallback `date_end`) |
| 3 | **Drag & drop** — kéo bar để điều chỉnh start/end date, lưu tự động vào DB |
| 4 | **Click-to-create** — click vùng trống trên timeline → tạo task mới với date pre-filled |
| 5 | **Hover/click popover** — hiển thị thông tin task (tên, deadline, stage, assignee, project) |
| 6 | **Time scale** — Day / Week / Month / Year |
| 7 | **Task dependency arrows** — mũi tên phụ thuộc giữa tasks (bật/tắt theo `allow_task_dependencies`) |
| 8 | **Navigation** — menu riêng cho cả 2 view + click project row → task Gantt của project đó |
| 9 | Field mới `custom_date_start` (Planned Start Date) trên `project.task` |

### Out of Scope

| # | Không làm |
|---|---|
| 1 | Hour-level zoom |
| 2 | Resource allocation / critical path |
| 3 | Real-time collaborative editing |
| 4 | Hiển thị sub-tasks trong Gantt |
| 5 | Export Gantt ra PDF / image |
| 6 | Mobile responsiveness |

---

## Quyết định kỹ thuật

### Library: frappe-gantt

- **License:** MIT — dùng tự do, không phí thương mại
- **Bundle size:** ~50KB
- **Tích hợp:** Vanilla JS, không cần build step riêng
- **Hỗ trợ:** Drag & drop, time scales (Day/Week/Month/Year), dependency arrows
- **Không hỗ trợ natively:** Hour view, click-to-create → sẽ custom thêm

### Addon mới: `project_gantt`

Không sửa module `project` gốc. Toàn bộ code mới nằm trong `addons/project_gantt/`.

---

## Kiến trúc module

```
addons/project_gantt/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── project_task.py             # thêm custom_date_start vào project.task
├── views/
│   ├── project_gantt_views.xml     # ir.ui.view definitions
│   └── project_gantt_menus.xml     # menu items + window actions
└── static/src/
    ├── lib/
    │   └── frappe_gantt/
    │       ├── frappe-gantt.min.js
    │       └── frappe-gantt.css
    ├── views/
    │   ├── project_task_gantt/     # Task Gantt OWL view
    │   └── project_project_gantt/ # Multi-project Gantt OWL view
    └── components/
        └── gantt_task_popover/     # Popover component
```

---

## Data model

### Field mới trên `project.task`

```python
custom_date_start = fields.Datetime("Planned Start Date", index=True)
```

### Fields dùng cho Gantt bars

| View | Model | Start field | End field |
|---|---|---|---|
| Multi-project Gantt | `project.project` | `date_start` (có sẵn) | `date` (có sẵn) |
| Task Gantt | `project.task` | `custom_date_start` (mới) | `date_deadline` → fallback `date_end` |

### Task dependencies

Dùng `depend_on_ids` / `dependent_ids` đã có sẵn trên `project.task`.
Chỉ hiển thị mũi tên khi `project.allow_task_dependencies = True`.

---

## JS Architecture (OWL + frappe-gantt)

Mỗi view theo pattern chuẩn của Odoo:

```
View (registry entry)
  └── Controller (data loading, action handling)
        └── Renderer (OWL component wrapping frappe-gantt)
```

### Frappe-gantt task format

```javascript
{
    id: String(task.id),
    name: task.name,
    start: task.custom_date_start,     // "YYYY-MM-DD HH:mm"
    end: task.date_deadline || task.date_end,
    progress: 0,
    dependencies: "id1,id2",          // chỉ khi allow_task_dependencies = true
}
```

### Key callbacks

| Callback | Hành động |
|---|---|
| `on_date_change(task, start, end)` | `orm.write(...)` cập nhật dates vào DB |
| `on_click(task)` | Mở task form view |
| click trên `.gantt-grid` | Tính date từ tọa độ → mở dialog tạo task |
| `custom_popup_html(task)` | Render thông tin task vào popover |

---

## Files tham khảo (không sửa)

| File | Lý do tham khảo |
|---|---|
| `addons/project/models/project_task.py:259–270` | `depend_on_ids`, `allow_task_dependencies` |
| `addons/project/models/project_project.py:135–138` | `date_start`, `date` fields |
| `addons/project/static/src/views/project_task_calendar/` | Pattern custom view trong project module |
| `addons/web/static/src/views/view.js` | ViewType registry |
| `addons/project/views/project_task_views.xml` | Existing actions để extend |

---

## Verification checklist

- [ ] `./odoo-bin -d <db> -i project_gantt` — module load không lỗi
- [ ] Menu "All Projects (Gantt)" hiển thị project bars đúng
- [ ] Click project row → navigate sang Task Gantt filter đúng project
- [ ] Drag task bar → `custom_date_start` / `date_deadline` cập nhật trong DB
- [ ] Click vùng trống → dialog tạo task mở với date pre-filled
- [ ] Hover/click task → popover hiển thị thông tin đúng
- [ ] Dependency arrows hiển thị khi `allow_task_dependencies = True`
- [ ] Time scale switcher: Day / Week / Month / Year hoạt động đúng
