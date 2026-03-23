# PHASES.md — Project Gantt View Development Phases

Dự án: Addon `project_gantt` — Gantt/Timeline View cho Odoo Project Module
Thư viện: frappe-gantt (MIT)
Trạng thái: Planning ✓ | Implementation: chờ bắt đầu

---

## Tổng quan các Phase

| Phase | Tên | Deliverable chính |
|---|---|---|
| 1 | Module Skeleton & Model | Addon load được, field `custom_date_start` tồn tại trong DB |
| 2 | Multi-project Gantt cơ bản | Project bars hiển thị đúng trên timeline |
| 3 | Task Gantt cơ bản + Navigation | Task bars hiển thị, điều hướng giữa 2 view |
| 4 | Drag & Drop | Kéo bar → cập nhật dates trong DB |
| 5 | Click-to-Create | Click vùng trống → tạo task mới |
| 6 | Hover/Click Popover | Hiển thị thông tin task khi hover/click |
| 7 | Task Dependencies | Mũi tên phụ thuộc giữa các tasks |
| 8 | Time Scale Switcher | Chuyển đổi Day / Week / Month / Year |
| 9 | Polish & Styling | Giao diện khớp Odoo design system, xử lý edge cases |

---

## Phase 1 — Module Skeleton & Model Layer

**Mục tiêu:** Tạo bộ khung addon, thêm field `custom_date_start` vào `project.task`,
xác nhận module load thành công.

**Files cần tạo:**

```
addons/project_gantt/__init__.py
addons/project_gantt/__manifest__.py
addons/project_gantt/models/__init__.py
addons/project_gantt/models/project_task.py
```

**Chi tiết:**
- `__manifest__.py`: `depends: ['project']`, khai báo assets (chưa có file JS/CSS thật)
- `project_task.py`: thêm field mới vào `project.task`

```python
custom_date_start = fields.Datetime("Planned Start Date", index=True)
```

**Kiểm tra:**
```bash
./odoo-bin -d <db> -i project_gantt
# Vào Settings > Technical > Fields, tìm project.task
# → xác nhận custom_date_start tồn tại
```

---

## Phase 2 — Multi-project Gantt View (Cơ bản)

**Mục tiêu:** Render Gantt chart cho `project.project` —
mỗi project là 1 thanh bar theo `date_start` → `date`.

**Files cần tạo:**

```
addons/project_gantt/static/src/lib/frappe_gantt/frappe-gantt.min.js
addons/project_gantt/static/src/lib/frappe_gantt/frappe-gantt.css
addons/project_gantt/static/src/views/project_project_gantt/project_project_gantt_view.js
addons/project_gantt/static/src/views/project_project_gantt/project_project_gantt_view.xml
addons/project_gantt/static/src/views/project_project_gantt/project_project_gantt_view.scss
addons/project_gantt/views/project_gantt_views.xml
addons/project_gantt/views/project_gantt_menus.xml
```

**Chi tiết JS:**
- Đăng ký view type `project_project_gantt` vào Odoo view registry
- Load projects: `orm.searchRead('project.project', [], ['name', 'date_start', 'date'])`
- Khởi tạo: `new Gantt(el, tasks, { view_mode: 'Month' })`
- Default scale: Month

**Kiểm tra:**
```bash
./odoo-bin -d <db> -u project_gantt
# Vào Project → menu "All Projects (Gantt)"
# → xác nhận bars hiển thị đúng theo date_start/date
```

---

## Phase 3 — Task Gantt View (Cơ bản) + Navigation

**Mục tiêu:** Render Gantt cho `project.task` và kết nối điều hướng giữa 2 view.

**Files cần tạo:**

```
addons/project_gantt/static/src/views/project_task_gantt/project_task_gantt_view.js
addons/project_gantt/static/src/views/project_task_gantt/project_task_gantt_view.xml
addons/project_gantt/static/src/views/project_task_gantt/project_task_gantt_view.scss
```

**Files bổ sung:**
- `views/project_gantt_views.xml`: thêm view record cho `project.task`
- `views/project_gantt_menus.xml`: thêm menu item "Tasks (Gantt)"

**Chi tiết JS:**
- Load tasks: `orm.searchRead('project.task', [], ['name', 'custom_date_start', 'date_deadline', 'date_end', 'project_id', 'depend_on_ids'])`
- Bar end = `date_deadline || date_end`
- Navigation: click project row → `action.doAction(...)` với domain `[['project_id', '=', id]]`
- Default scale: Week

**Kiểm tra:**
- Vào Task Gantt → task bars render đúng
- Click project row trong multi-project Gantt → navigate sang task Gantt đúng project

---

## Phase 4 — Drag & Drop

**Mục tiêu:** Kéo thanh bar để điều chỉnh start/end date, tự động lưu vào DB.

**Files sửa:**
- `project_task_gantt_view.js`: bắt callback `on_date_change`
- `project_project_gantt_view.js`: tương tự cho project bars

**Chi tiết:**

```javascript
on_date_change: async (task, start, end) => {
    await this.orm.write('project.task', [parseInt(task.id)], {
        custom_date_start: serializeDateTime(start),
        date_deadline: serializeDateTime(end),
    });
}
```

- Import `serializeDateTime` từ `@web/core/l10n/dates`
- Validate: start < end trước khi gọi ORM write
- Hiển thị notification sau khi lưu thành công

**Kiểm tra:**
- Kéo bar → mở form task → xác nhận `custom_date_start` và `date_deadline` đã cập nhật

---

## Phase 5 — Click-to-Create Task

**Mục tiêu:** Click vào vùng trống trên timeline → mở dialog tạo task mới với date pre-filled.

**Files sửa:**
- `project_task_gantt_view.js`: bắt sự kiện `click` trên SVG `.gantt-grid`

**Chi tiết:**
- Tính date từ tọa độ click dựa trên column width và `gantt.gantt_start`
- Mở form task với context `{ default_custom_date_start: calculatedDate }`
- Sau khi tạo xong: refresh gantt

**Kiểm tra:**
- Click vào ngày bất kỳ trên grid → dialog mở với `custom_date_start` pre-filled đúng ngày

---

## Phase 6 — Hover/Click Popover

**Mục tiêu:** Hover hoặc click vào task bar → hiển thị popover thông tin task.

**Files cần tạo:**

```
addons/project_gantt/static/src/components/gantt_task_popover/gantt_task_popover.js
addons/project_gantt/static/src/components/gantt_task_popover/gantt_task_popover.xml
```

**Files sửa:**
- `project_task_gantt_view.js`: dùng `custom_popup_html` callback của frappe-gantt

**Chi tiết:**
- `custom_popup_html(task)` → trả về HTML string cho tooltip
- Nội dung: tên task, project, deadline, stage, người phụ trách
- Click task bar → mở form task

**Kiểm tra:**
- Hover vào task bar → popover xuất hiện với đúng thông tin
- Click task bar → form task mở ra

---

## Phase 7 — Task Dependencies (Mũi tên phụ thuộc)

**Mục tiêu:** Hiển thị mũi tên SVG kết nối các task có quan hệ phụ thuộc.

**Files sửa:**
- `project_task_gantt_view.js`: bổ sung logic map `depend_on_ids`

**Chi tiết:**

```javascript
// Khi map tasks sang frappe-gantt format
dependencies: task.project_id.allow_task_dependencies
    ? task.depend_on_ids.join(',')
    : '',
```

- frappe-gantt tự vẽ mũi tên SVG dựa trên field `dependencies`
- Chỉ bật khi `project.allow_task_dependencies = True`

**Kiểm tra:**
- Tạo 2 tasks có dependency → mũi tên hiển thị
- Tắt `allow_task_dependencies` → mũi tên biến mất

---

## Phase 8 — Time Scale Switcher

**Mục tiêu:** Cho phép người dùng chuyển đổi giữa Day / Week / Month / Year.

**Chi tiết:**
- frappe-gantt có sẵn `view_mode_select: true` → tự render dropdown chọn scale
- Persist lựa chọn trong `localStorage`
- Cập nhật URL params để giữ scale khi reload

**Kiểm tra:**
- Chọn Day → timeline render theo ngày
- Reload trang → scale được giữ nguyên

---

## Phase 9 — Polish & Styling

**Mục tiêu:** Giao diện sạch, khớp Odoo design system, xử lý edge cases.

**Checklist:**
- [ ] SCSS: màu bar theo trạng thái task (done / in progress / blocked)
- [ ] Xử lý task không có `custom_date_start` → cảnh báo hoặc ẩn khỏi gantt
- [ ] Xử lý project không có `date_start` / `date` → hiển thị thông báo
- [ ] Empty state khi không có task / project nào
- [ ] Kiểm tra performance với 100+ tasks
- [ ] Responsive: dùng được ở độ rộng tối thiểu 1280px

**Kiểm tra cuối:**
```bash
./odoo-bin --test-tags /project_gantt -d <db>
flake8 addons/project_gantt/
```

---

## Dependency giữa các Phase

```
Phase 1 (skeleton)
    └── Phase 2 (multi-project gantt)
            └── Phase 3 (task gantt + navigation)
                    ├── Phase 4 (drag & drop)
                    ├── Phase 5 (click-to-create)
                    ├── Phase 6 (popover)
                    ├── Phase 7 (dependencies)
                    └── Phase 8 (scale switcher)
                            └── Phase 9 (polish)
```

> Phase 4–8 có thể phát triển song song sau khi Phase 3 hoàn tất.
