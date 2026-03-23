# Phân Tích Cấu Trúc Repository Odoo Core

> Phân tích tổng quan về các modules, models và cấu trúc của repository `odoo-core`.

---

## Mục Lục

1. [Tổng Quan](#1-tổng-quan)
2. [Phân Loại CE vs Enterprise](#2-phân-loại-ce-vs-enterprise)
3. [Danh Sách Modules Theo Danh Mục](#3-danh-sách-modules-theo-danh-mục)
4. [Cấu Trúc Một Module](#4-cấu-trúc-một-module)
5. [Models Chính Theo Module](#5-models-chính-theo-module)
6. [Sơ Đồ Phụ Thuộc](#6-sơ-đồ-phụ-thuộc)

---

## 1. Tổng Quan

| Thông Tin | Giá Trị |
|-----------|---------|
| Tổng số modules | **618** |
| Số danh mục | **68** |
| Modules có Python models | **232** |
| Tổng số model files | **775** |
| Tổng số model classes | **~1,700** |
| Thư mục chứa modules | `/addons/` |

Odoo Core là một hệ thống ERP (Enterprise Resource Planning) hoàn chỉnh, bao gồm các nghiệp vụ: kế toán, bán hàng, kho hàng, sản xuất, nhân sự, marketing, website v.v.

---

## 2. Phân Loại CE vs Enterprise

### Bối Cảnh

Repository `odoo-core` **là bản Community Edition (CE)**. Phần lớn Enterprise modules nằm trong một repository riêng (`enterprise`). Tuy nhiên, trong repo này vẫn tồn tại một số module được đánh dấu license `OEEL-1` (Enterprise).

Cách phân biệt: Kiểm tra field `'license'` trong file `__manifest__.py` của mỗi module.

| License | Phiên Bản | Ý Nghĩa |
|---------|-----------|---------|
| `LGPL-3` | Community Edition (CE) | Mã nguồn mở, miễn phí |
| `OEEL-1` | Enterprise Edition (EE) | Bản thương mại, cần license trả phí |
| *(không có)* | Không xác định | Thường là CE hoặc module mới |

### Thống Kê

| Loại | Số Module |
|------|-----------|
| **Community (LGPL-3)** | **583** |
| **Enterprise (OEEL-1)** | **4** |
| Không có license field | 29 |
| **Tổng** | **618** |

> **Lưu ý:** 29 module không có `license` field gồm chủ yếu là các module localization mới và một số module đặc thù (`pos_self_order`, `spreadsheet_dashboard`, `google_gmail`...). Đây thực chất vẫn là CE modules.

---

### Modules Enterprise (OEEL-1) — Có trong repo này

Chỉ có **4 modules** được cấp phép Enterprise trong repo `odoo-core`:

| Module | Tên Hiển Thị | Danh Mục |
|--------|-------------|----------|
| `certificate` | Certificate | Hidden/Tools |
| `l10n_hr_edi` | Croatia - e-invoicing | Accounting/Localizations/Reporting |
| `l10n_it_edi_website_sale` | Italy eCommerce eInvoicing | Accounting/Localizations/Website |
| `project_hr_skills` | Project - Skills | Services/Project |

**Mô tả chi tiết:**

- **`certificate`** — Module nội bộ quản lý chứng chỉ số (certificates). Dùng làm nền tảng cho các tính năng xác thực điện tử trong EE.
- **`l10n_hr_edi`** — E-invoicing điện tử theo chuẩn Croatia (bắt buộc từ 2026 theo luật Croatia).
- **`l10n_it_edi_website_sale`** — Tích hợp e-invoicing Italy cho kênh bán hàng eCommerce.
- **`project_hr_skills`** — Liên kết kỹ năng nhân sự (`hr.skills`) với dự án để phân công nhân sự theo năng lực.

---

### Modules Không Có License Field (29 modules)

Các module này chưa khai báo `license` trong manifest, nhưng mặc định thuộc CE:

| Nhóm | Modules |
|------|---------|
| **Localization** | `l10n_dk_fik`, `l10n_ie`, `l10n_in_edi`, `l10n_in_edi_ewaybill`, `l10n_in_ewaybill_stock`, `l10n_in_gstin_status`, `l10n_iq`, `l10n_jo_edi_pos`, `l10n_lb_account`, `l10n_mt_pos`, `l10n_mu_account`, `l10n_my_edi_pos`, `l10n_pe_pos`, `l10n_pe_website_sale`, `l10n_tw_edi_ecpay`, `l10n_tw_edi_ecpay_website_sale`, `l10n_ug`, `l10n_uy_pos`, `l10n_vn_edi_viettel`, `l10n_zm_account` |
| **Point of Sale** | `pos_self_order`, `pos_self_order_adyen`, `pos_self_order_razorpay`, `pos_self_order_sale`, `pos_self_order_stripe` |
| **Productivity** | `spreadsheet_dashboard` |
| **Integrations** | `cloud_storage_migration`, `google_gmail`, `microsoft_outlook` |

---

### Tất Cả Modules CE (LGPL-3) — 583 modules

Toàn bộ các modules còn lại đều là **Community Edition** — bao gồm toàn bộ core business modules:

| Danh Mục | Ví Dụ Modules |
|----------|--------------|
| Accounting | `account`, `account_payment`, `account_edi`, `analytic` |
| Sales | `sale`, `sale_crm`, `sale_stock`, `sale_mrp` |
| CRM | `crm`, `crm_sms`, `crm_livechat` |
| Inventory | `stock`, `stock_account`, `stock_picking_batch` |
| Purchase | `purchase`, `purchase_stock`, `purchase_requisition` |
| Manufacturing | `mrp`, `mrp_account`, `mrp_repair`, `mrp_subcontracting` |
| Human Resources | `hr`, `hr_attendance`, `hr_contract`, `hr_holidays`, `hr_payroll` |
| Project | `project`, `project_account`, `project_todo` |
| Website | `website`, `website_sale`, `website_blog`, `website_slides` |
| Marketing | `mass_mailing`, `event`, `event_sale`, `loyalty` |
| Point of Sale | `point_of_sale`, `pos_restaurant`, `pos_loyalty` |
| Productivity | `calendar`, `discuss`, `mail`, `digest` |
| Localizations | `l10n_vn`, `l10n_us`, `l10n_fr`, `l10n_de`, ... *(112+ quốc gia)* |
| Core/Framework | `base`, `web`, `portal`, `iap`, `sms` |

---

## 3. Danh Sách Modules Theo Danh Mục

### Kế Toán (Accounting)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `account` | Invoicing | base_setup, product, analytic, portal |
| `account_payment` | Payment Acquirers | account |
| `account_edi` | Electronic Invoicing | account |
| `account_check_printing` | Check Printing | account |
| `account_debit_note` | Debit Notes | account |
| `account_tax_python` | Tax with Python | account |
| `account_peppol` | Peppol e-Invoicing | account_edi |
| `analytic` | Analytic Accounting | base |

### Bán Hàng (Sales)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `sale` | Sales | account, mail, product |
| `sale_crm` | Sales CRM | sale, crm |
| `sale_stock` | Sales & Inventory | sale, stock |
| `sale_mrp` | Sales & Manufacturing | sale, mrp |
| `sale_expense` | Sales & Expenses | sale, hr_expense |
| `sale_margin` | Sales Margin | sale |
| `sale_loyalty` | Gift Cards & Loyalty | sale |
| `sale_timesheet` | Timesheets on Sales | sale, hr_timesheet |
| `sale_management` | Sales Management | sale |

### Quản Lý Khách Hàng (CRM)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `crm` | CRM | base_setup, mail, phone_validation |
| `crm_iap_enrich` | CRM Lead Enrichment | crm, iap |
| `crm_livechat` | CRM Livechat | crm, im_livechat |
| `crm_sms` | CRM SMS | crm, sms |
| `crm_mail_plugin` | CRM Mail Plugin | crm |

### Kho Hàng (Inventory)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `stock` | Inventory | product, mail |
| `stock_account` | Stock Accounting | stock, account |
| `stock_delivery` | Delivery Orders | stock |
| `stock_dropshipping` | Dropshipping | stock, purchase, sale_stock |
| `stock_landed_costs` | Landed Costs | stock, account |
| `stock_picking_batch` | Batch Transfers | stock |

### Mua Hàng (Purchase)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `purchase` | Purchase | account, mail |
| `purchase_stock` | Purchase & Inventory | purchase, stock |
| `purchase_mrp` | Purchase & Manufacturing | purchase, mrp |
| `purchase_requisition` | Purchase Agreements | purchase |

### Sản Xuất (Manufacturing)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `mrp` | Manufacturing | stock, mail |
| `mrp_account` | MRP Accounting | mrp, account |
| `mrp_repair` | Repairs | mrp, sale |
| `mrp_subcontracting` | Subcontracting | mrp, purchase |

### Nhân Sự (Human Resources)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `hr` | Employees | base, mail |
| `hr_attendance` | Attendances | hr |
| `hr_contract` | Employee Contracts | hr |
| `hr_expense` | Expenses | hr, account |
| `hr_holidays` | Time Off | hr, mail |
| `hr_payroll` | Payroll | hr_contract |
| `hr_recruitment` | Recruitment | hr, mail |
| `hr_timesheet` | Timesheets | hr, analytic |
| `hr_skills` | Skills & Resumé | hr |

### Dự Án (Project)

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `project` | Project | analytic, mail |
| `project_account` | Project Accounting | project, account |
| `project_timesheet_holidays` | Time Off in Project | project, hr_holidays |
| `project_todo` | To-Do | project |

### Website & Thương Mại Điện Tử

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `website` | Website | web, mail |
| `website_sale` | eCommerce | website, sale |
| `website_blog` | Blog | website |
| `website_event` | Online Events | website, event |
| `website_forum` | Forum | website |
| `website_livechat` | Website Live Chat | website, im_livechat |
| `website_slides` | eLearning | website |

### Marketing

| Module | Tên Hiển Thị | Phụ Thuộc Chính |
|--------|-------------|-----------------|
| `mass_mailing` | Email Marketing | mail, utm |
| `event` | Events | mail |
| `event_sale` | Event Ticketing | event, sale |
| `loyalty` | Loyalty Programs | product |
| `gift_card` | Gift Cards | loyalty |

### Kế Toán Địa Phương (Localizations — l10n)

Có **112 modules** địa phương hóa kế toán cho từng quốc gia, đặt tên theo mã ISO:

| Mã Module | Quốc Gia |
|-----------|---------|
| `l10n_vn` | Việt Nam |
| `l10n_us` | Hoa Kỳ |
| `l10n_fr` | Pháp |
| `l10n_de` | Đức |
| `l10n_uk` | Anh |
| `l10n_in` | Ấn Độ |
| `l10n_cn` | Trung Quốc |
| `l10n_jp` | Nhật Bản |
| `l10n_br` | Brazil |
| `l10n_mx` | Mexico |
| ... | *(và 102 quốc gia khác)* |

### Core & Framework

| Module | Mục Đích |
|--------|---------|
| `base` | Nền tảng ORM, records, fields |
| `web` | Framework web UI (OWL components) |
| `mail` | Messaging, chatter, notifications |
| `portal` | Customer portal |
| `base_setup` | Cấu hình hệ thống ban đầu |
| `iap` | In-App Purchase services |
| `sms` | SMS gateway |
| `digest` | Periodic digest emails |

---

## 4. Cấu Trúc Một Module

Lấy module `account` làm ví dụ đại diện — đây là module lớn và đầy đủ nhất.

### Cây Thư Mục Chuẩn

```
account/
│
├── __init__.py                  # Khai báo package Python
├── __manifest__.py              # Metadata của module (bắt buộc)
├── README.md                    # Mô tả ngắn gọn
│
├── models/                      # Business logic — ORM models
│   ├── __init__.py
│   ├── account_move.py          # Ví dụ: model account.move (hóa đơn)
│   ├── account_payment.py
│   ├── account_journal.py
│   └── ...                      # (53 files trong module account)
│
├── views/                       # Giao diện người dùng (XML)
│   ├── account_move_views.xml
│   ├── account_payment_view.xml
│   ├── account_menuitem.xml     # Menu items
│   └── ...                      # (43 files)
│
├── controllers/                 # HTTP routes & API endpoints
│   ├── __init__.py
│   ├── portal.py                # Customer portal routes
│   └── ...                      # (5 files)
│
├── wizard/                      # Transient models (dialog/popup workflows)
│   ├── account_payment_register.py
│   ├── account_move_reversal.py
│   └── ...                      # (15 Python + 13 XML files)
│
├── report/                      # Report templates & engines
│   ├── account_report.py
│   └── report_invoice.xml
│
├── static/                      # Frontend assets (không qua server)
│   ├── src/
│   │   ├── components/          # OWL Web Components (JS)
│   │   │   ├── tax_totals/
│   │   │   ├── account_move_form/
│   │   │   └── ...              # (40+ components)
│   │   ├── views/               # Custom frontend views
│   │   ├── js/                  # JavaScript files
│   │   ├── css/                 # CSS files
│   │   └── scss/                # SCSS files
│   ├── description/
│   │   └── icon.png             # Icon hiển thị trong App Store
│   └── tests/                   # Frontend test helpers
│
├── tests/                       # Backend unit & integration tests
│   ├── test_account_move.py
│   ├── test_account_payment.py
│   └── ...                      # (65+ files)
│
├── data/                        # Dữ liệu khởi tạo (chạy khi cài module)
│   ├── account_data.xml
│   ├── mail_template_data.xml
│   └── ir_sequence.xml
│
├── demo/                        # Dữ liệu demo (chỉ cho môi trường dev)
│   └── demo_data.py
│
├── security/                    # Phân quyền truy cập
│   ├── account_security.xml     # Groups & rules
│   └── ir.model.access.csv      # CRUD permissions
│
├── i18n/                        # Bản dịch đa ngôn ngữ
│   ├── account.pot              # Template dịch thuật
│   ├── vi.po                    # Tiếng Việt
│   ├── fr.po, de.po, ...        # (67+ ngôn ngữ)
│
└── tools/                       # Utility helpers nội bộ
    └── ...
```

### Giải Thích Từng Thư Mục

| Thư Mục | Bắt Buộc | Vai Trò |
|---------|----------|---------|
| `__manifest__.py` | Có | Định nghĩa tên, version, dependencies, data files... |
| `models/` | Thường có | Định nghĩa data models (ORM), business logic |
| `views/` | Thường có | XML định nghĩa form, list, kanban, search view |
| `controllers/` | Tùy chọn | Route HTTP, REST API, portal |
| `wizard/` | Tùy chọn | Dialog/popup workflow (TransientModel) |
| `report/` | Tùy chọn | PDF/Excel reports |
| `static/` | Tùy chọn | JS, CSS, images — serve trực tiếp |
| `tests/` | Tùy chọn | Unit & integration tests |
| `data/` | Tùy chọn | XML/CSV data load khi install module |
| `demo/` | Tùy chọn | Data demo cho môi trường test |
| `security/` | Thường có | Access rights và record rules |
| `i18n/` | Tùy chọn | File dịch `.po`/`.pot` |

### Nội Dung File `__manifest__.py`

```python
{
    'name': 'Invoicing',                        # Tên hiển thị
    'version': '17.0.1.0.0',                    # Version
    'category': 'Accounting/Accounting',        # Danh mục
    'summary': 'Invoices & Payments',           # Mô tả ngắn
    'description': '...',                       # Mô tả dài
    'depends': [                                # Module phụ thuộc
        'base_setup',
        'product',
        'analytic',
        'portal',
    ],
    'data': [                                   # Files load khi install
        'security/ir.model.access.csv',
        'data/account_data.xml',
        'views/account_move_views.xml',
    ],
    'demo': [                                   # Files demo
        'demo/demo_data.py',
    ],
    'assets': {                                 # Frontend assets
        'web.assets_backend': [
            'account/static/src/components/**/*',
        ],
    },
    'installable': True,
    'auto_install': False,
    'license': 'LGPL-3',
}
```

---

## 5. Models Chính Theo Module

### Module `account` — Kế Toán

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `account.account` | account_account | Tài khoản kế toán |
| `account.move` | account_move | Bút toán / Hóa đơn |
| `account.move.line` | account_move_line | Chi tiết dòng bút toán |
| `account.journal` | account_journal | Sổ nhật ký kế toán |
| `account.payment` | account_payment | Thanh toán |
| `account.bank.statement` | account_bank_statement | Sao kê ngân hàng |
| `account.bank.statement.line` | account_bank_statement_line | Dòng sao kê |
| `account.tax` | account_tax | Thuế |
| `account.tax.repartition.line` | account_tax_repartition_line | Phân bổ thuế |
| `account.reconcile.model` | account_reconcile_model | Mẫu đối soát |
| `account.payment.term` | account_payment_term | Điều khoản thanh toán |
| `account.fiscal.position` | account_fiscal_position | Vị trí tài chính |
| `account.chart.template` | account_chart_template | Template hệ thống tài khoản |
| `account.incoterms` | account_incoterms | Điều kiện giao hàng |

### Module `sale` — Bán Hàng

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `sale.order` | sale_order | Đơn hàng bán |
| `sale.order.line` | sale_order_line | Dòng đơn hàng |
| `sale.report` | sale_report | Báo cáo bán hàng |
| `sale.advance.payment.inv` | — | Wizard tạo hóa đơn tạm ứng |

### Module `stock` — Kho Hàng

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `stock.quant` | stock_quant | Số lượng tồn kho |
| `stock.move` | stock_move | Di chuyển hàng tồn |
| `stock.move.line` | stock_move_line | Chi tiết di chuyển |
| `stock.picking` | stock_picking | Phiếu xuất/nhập kho |
| `stock.picking.type` | stock_picking_type | Loại thao tác kho |
| `stock.location` | stock_location | Vị trí kho |
| `stock.warehouse` | stock_warehouse | Kho hàng |
| `stock.rule` | stock_rule | Quy tắc tuyến cung ứng |
| `stock.route` | stock_location_route | Tuyến cung ứng |
| `stock.lot` | stock_lot | Lô / Serial number |
| `stock.valuation.layer` | stock_valuation_layer | Định giá kho |

### Module `crm` — Quản Lý Khách Hàng

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `crm.lead` | crm_lead | Cơ hội / Lead |
| `crm.stage` | crm_stage | Giai đoạn pipeline |
| `crm.team` | crm_team | Đội ngũ bán hàng |
| `crm.activity.report` | crm_activity_report | Báo cáo hoạt động |

### Module `hr` — Nhân Sự

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `hr.employee` | hr_employee | Nhân viên |
| `hr.employee.public` | hr_employee_public | Thông tin nhân viên công khai |
| `hr.department` | hr_department | Phòng ban |
| `hr.job` | hr_job | Vị trí công việc |

### Module `project` — Dự Án

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `project.project` | project_project | Dự án |
| `project.task` | project_task | Nhiệm vụ / Task |
| `project.task.type` | project_task_type | Giai đoạn task |
| `project.tags` | project_tags | Nhãn dự án |
| `project.milestone` | project_milestone | Mốc dự án |

### Module `mrp` — Sản Xuất

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `mrp.production` | mrp_production | Lệnh sản xuất |
| `mrp.bom` | mrp_bom | Định mức nguyên vật liệu (BOM) |
| `mrp.bom.line` | mrp_bom_line | Dòng BOM |
| `mrp.routing.workcenter` | mrp_routing_workcenter | Công đoạn sản xuất |
| `mrp.workcenter` | mrp_workcenter | Trung tâm làm việc |

### Module `purchase` — Mua Hàng

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `purchase.order` | purchase_order | Đơn đặt hàng |
| `purchase.order.line` | purchase_order_line | Dòng đặt hàng |
| `purchase.report` | purchase_report | Báo cáo mua hàng |

### Module `base` — Core

| Model | Tên Bảng DB | Mô Tả |
|-------|-------------|-------|
| `res.partner` | res_partner | Đối tác / Khách hàng |
| `res.company` | res_company | Công ty |
| `res.users` | res_users | Người dùng |
| `res.groups` | res_groups | Nhóm quyền |
| `res.currency` | res_currency | Đơn vị tiền tệ |
| `res.country` | res_country | Quốc gia |
| `product.product` | product_product | Sản phẩm (variant) |
| `product.template` | product_template | Mẫu sản phẩm |
| `ir.model` | ir_model | Metadata model |
| `ir.fields` | ir_model_fields | Metadata fields |

---

## 6. Sơ Đồ Phụ Thuộc

Dưới đây là các module nền tảng quan trọng và số lượng module phụ thuộc vào chúng:

```
base  ←── web  ←── mail  ←── account  ←── sale
                              ↑              ↑
                         analytic         crm
                              ↑
                           product  ←── stock  ←── mrp
                                          ↑
                                       purchase
```

| Module | Số Module Phụ Thuộc Vào | Vai Trò |
|--------|------------------------|---------|
| `base` | ~600 | Lõi ORM, records, fields |
| `mail` | ~300 | Messaging, chatter |
| `web` | ~400 | Web framework |
| `account` | ~142 | Core kế toán |
| `product` | ~200 | Sản phẩm |
| `stock` | ~80 | Kho hàng |
| `sale` | ~60 | Bán hàng |
| `hr` | ~50 | Nhân sự |

---

*File được tạo tự động từ việc phân tích repository `/home/user/odoo-core` — ngày 23/03/2026.*
