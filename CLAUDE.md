# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Start the server
```bash
./odoo-bin -d <database_name>
```

### Run tests
```bash
# All tests in a module
./odoo-bin --test-enable --init <module_name> -d <database_name>

# By tag (module / class / method)
./odoo-bin --test-tags /module_name -d <database_name>
./odoo-bin --test-tags /module_name:ClassName -d <database_name>
./odoo-bin --test-tags /module_name:ClassName.test_method -d <database_name>

# Single test file
./odoo-bin --test-file=addons/<module>/tests/test_foo.py -d <database_name>
```

Tag format: `[-][tag][/module][:class][.method]` — e.g., `--test-tags standard` runs only standard-tagged tests.

### Lint
```bash
flake8 .
```
Config is in `setup.cfg`. No pre-commit hooks are configured.

### Scaffold a new module
```bash
./odoo-bin scaffold <module_name> addons/
```

## Architecture

### Framework core (`odoo/`)

| File / Package | Role |
|---|---|
| `models.py` | ORM — BaseModel, recordsets, CRUD, caching |
| `fields.py` | Field types (Char, Many2one, Computed, …) |
| `api.py` | Environment, decorators (`@api.depends`, `@api.constrains`, …) |
| `http.py` | WSGI app, routing, `Controller` base class |
| `sql_db.py` | PostgreSQL cursor pool and query helpers |
| `modules/` | Module registry, loading, dependency graph |
| `tests/` | Test base classes (`TransactionCase`, `HttpCase`, …) |
| `tools/` | Utilities: `SQL`, translations, config, caching |
| `cli/` | `odoo-bin` sub-commands (server, shell, scaffold, …) |

### Addon modules (`addons/`)

619 modules. Each follows the same layout:

```
addons/<module>/
├── __manifest__.py   # metadata: name, depends, data, assets, license
├── models/           # Model classes
├── views/            # XML: form/list/kanban/search views, menus, actions
├── controllers/      # HTTP routes
├── wizard/           # TransientModel-based dialogs
├── security/         # ir.model.access.csv + record rules XML
├── data/             # Initial/demo data XML
├── static/src/       # JS (OWL components), SCSS, assets
└── tests/            # test_*.py
```

License in `__manifest__.py`: `LGPL-3` = Community Edition, `OEEL-1` = Enterprise.

### ORM model types and inheritance

```python
# Extend an existing model (most common — no new table)
class SaleOrder(models.Model):
    _inherit = 'sale.order'

# Create a new model
class MyModel(models.Model):
    _name = 'my.model'
    _description = 'My Model'

# Mixin / abstract (no table, mixed in via _inherit list)
class MyMixin(models.AbstractModel):
    _name = 'my.mixin'

# Delegation inheritance (wraps another model via Many2one)
class Employee(models.Model):
    _name = 'hr.employee'
    _inherits = {'resource.resource': 'resource_id'}
```

Multiple unrelated modules can all `_inherit` the same model; their fields and methods are merged in the registry at startup.

### Computed fields

```python
name = fields.Char(compute='_compute_name', store=True, readonly=False)

@api.depends('first_name', 'last_name')
def _compute_name(self):
    for rec in self:
        rec.name = f"{rec.first_name} {rec.last_name}"
```

`store=True` persists to the DB and makes the field searchable. Add `inverse='_inverse_name'` to allow writes.

### Controllers

```python
class MyController(http.Controller):
    @http.route('/my/path', type='json', auth='user', methods=['POST'])
    def endpoint(self, **kw):
        ...
```

`type='json'` automatically parses JSON bodies and serializes return values. `type='http'` returns a `Response` object or rendered template. `auth='public'` allows unauthenticated access; `auth='none'` skips all auth.

### Domain (filter) syntax

```python
# Used in search(), field `domain=`, and record rules
[('state', '=', 'draft'), '|', ('user_id', '=', uid), ('team_id.member_ids', 'in', [uid])]
# Prefix operators: '&' (default), '|', '!'
# Comparison operators: =, !=, <, >, <=, >=, in, not in, like, ilike, child_of, parent_of
```

### Test base classes

| Class | Use when |
|---|---|
| `TransactionCase` | Standard unit tests — each test method is rolled back |
| `SingleTransactionCase` | Tests that must share state across methods |
| `HttpCase` | Browser/JS tour tests, XMLRPC/JSON-RPC integration |

Tag tests with `@tagged('post_install', '-at_install')` to run after full install. The default tag is `standard`.

### Environment

```python
self.env            # current Environment (available in all model methods and tests)
self.env.user       # res.users record of the current user
self.env.company    # current company (res.company)
self.env.cr         # raw psycopg2 cursor
self.env['model.name']       # get model proxy
self.env.ref('module.xml_id')  # fetch record by XML id
# SUPERUSER_ID = 1 — pass as uid to bypass access rights
```
