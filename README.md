# 100 Odoo Development Interview Questions and Answers
**Focus:** Odoo 17 and Odoo 18  
**Level:** Beginner to Advanced

## Table of Contents
1. Odoo Fundamentals & Architecture (1–10)
2. Python & Odoo ORM (11–25)
3. Models, Fields & Relationships (26–40)
4. XML Views, Actions & Menus (41–50)
5. Security, Access Rights & Record Rules (51–60)
6. Computed Fields, Onchange, Constraints & ORM Methods (61–70)
7. Modules, Inheritance & Advanced Development (71–80)
8. Controllers, APIs, QWeb & Reports (81–90)
9. Debugging, Performance, Deployment & Real-World Scenarios (91–100)

> **Version note:** Odoo APIs and view conventions may differ by version and edition. Verify version-specific details against the official documentation and source code for your target deployment.

---

## 1. Odoo Fundamentals & Architecture

### 1. What is Odoo?
Odoo is a modular suite of business applications such as CRM, Sales, Inventory, Accounting, HR, and eCommerce. Modules share a common framework and database and can be extended with custom Python and XML code.

### 2. What are the main layers of Odoo's architecture?
The main parts are the web client and views, Python business models, the ORM, PostgreSQL, and the HTTP/controller layer. Together they provide the UI, business rules, persistence, and integrations.

### 3. What is an Odoo module?
A module is a package that adds or modifies functionality. It commonly contains `__manifest__.py`, Python models, XML views/data, security files, and optionally controllers, reports, static assets, and tests.

### 4. What is the purpose of `__manifest__.py`?
It declares module metadata and loading configuration. Common keys include `name`, `version`, `depends`, `data`, `demo`, `installable`, `application`, and `license`. Files listed in `data` are loaded in the specified order.

### 5. What is the difference between Community and Enterprise?
Community is the open-source edition. Enterprise adds proprietary applications, features, and services. Custom modules should declare dependencies accurately and avoid accidental reliance on Enterprise-only features when targeting Community.

### 6. What role does PostgreSQL play in Odoo?
PostgreSQL stores business data. Odoo's ORM generates database operations and integrates them with access checks, relations, caching, and computed fields. Prefer the ORM to direct SQL for normal business operations.

### 7. What happens when a module is installed?
Odoo resolves dependencies, loads the module, registers its models, creates or updates database structures, and loads security and data files. Missing dependencies, invalid XML references, Python import errors, or incorrect access definitions can stop installation.

### 8. What is the difference between installing and upgrading a module?
Installing initializes the module in a database. Upgrading applies code and data changes to an installed module. A common development command is `odoo-bin -d mydb -u my_module --stop-after-init`.

### 9. What is an addons path?
An addons path is a directory Odoo searches for modules, configured through `addons_path`. Put custom modules in a dedicated custom addons directory instead of editing core source files.

### 10. What is the difference between a model and a recordset?
A model is a Python class describing a business object. A recordset is a collection of zero or more records of that model. Odoo methods should generally handle multi-record recordsets unless they explicitly require one record.

---

## 2. Python & Odoo ORM

### 11. What is the Odoo ORM?
The Object-Relational Mapping layer lets developers interact with database records through Python models and recordsets. Common methods include `create()`, `search()`, `write()`, `unlink()`, and `browse()`.

### 12. What is `self.env`?
The environment provides the current user, database cursor, context, company information, and model registry. Examples include `self.env.user`, `self.env.company`, `self.env.context`, and `self.env['res.partner']`.

### 13. What is the difference between `search()`, `browse()`, and `search_count()`?
- `search(domain)` queries records matching a domain.
- `browse(ids)` creates a recordset for known database IDs; it does not filter by business conditions.
- `search_count(domain)` returns the number of matching records.

### 14. What is a domain?
A domain is a list of conditions used to filter records. Example: `[('active', '=', True)]`. Conditions can be combined with `&`, `|`, and `!`; adjacent conditions are implicitly ANDed.

### 15. What is the difference between `create()`, `write()`, and `unlink()`?
`create(vals)` creates records, `write(vals)` updates all records in `self`, and `unlink()` deletes records. These operations remain subject to access rights, record rules, constraints, and model-specific logic.

### 16. Why should Odoo methods support multiple records?
Odoo may call methods from list views, imports, RPC, or batch operations with multiple records. Iterate over `self` or use batch-friendly operations. Use `ensure_one()` only when a single record is genuinely required.

### 17. What does `ensure_one()` do?
It raises an error unless the recordset contains exactly one record. It is useful for methods that operate on a single document, but should not be used to conceal code that could support batches.

### 18. What is the difference between `@api.model` and `@api.model_create_multi`?
`@api.model` marks a method as model-level. `@api.model_create_multi` is used for `create()` overrides that accept a list of value dictionaries efficiently. Create overrides should preserve batch support.

### 19. What is `sudo()`?
`sudo()` returns a recordset that bypasses normal access restrictions. Use it only when the business requirement justifies elevated access, keep its scope narrow, and avoid using it as a blanket fix for access errors.

### 20. Compare `with_user()`, `with_company()`, and `with_context()`.
- `with_user(user)` changes the user used for access and behavior.
- `with_company(company)` changes the active company context.
- `with_context(...)` adds or overrides context values such as defaults, language, or allowed companies.
They return new recordsets rather than mutating the original.

### 21. What is the Odoo context?
Context is a dictionary of metadata passed through ORM calls and UI actions. It can carry language, timezone, active IDs, defaults, allowed companies, and custom flags. Do not treat context values as trusted authorization because callers may influence them.

### 22. What is the ORM cache?
Odoo caches field values and prefetches records to reduce repeated database reads. Direct SQL or unusual data manipulation can leave cached values stale, so cache invalidation and computed-field recomputation must be handled correctly when bypassing the ORM.

### 23. Compare `mapped()`, `filtered()`, and `sorted()`.
`mapped('partner_id.name')` extracts values or follows a field path. `filtered(lambda r: r.active)` returns records matching a Python condition. `sorted(key=lambda r: r.name)` sorts a recordset. For large datasets, prefer a database domain over loading everything and filtering in Python.

### 24. How should you execute raw SQL safely?
Prefer the ORM. When SQL is necessary, use parameterized queries through `self.env.cr.execute()` and never interpolate user input into SQL. Raw SQL bypasses ORM access rules and business logic; after SQL writes, handle cache invalidation and dependent computed fields.

### 25. What is the difference between `read()` and `read_group()`?
`read()` retrieves field values. `read_group()` performs database aggregations, such as counts or sums grouped by a field. Database-side aggregation is usually more efficient than fetching all records and calculating totals in Python.

---

## 3. Models, Fields & Relationships

### 26. How do you define an Odoo model?
Create a class inheriting from `models.Model`, define `_name`, and declare fields:

```python
from odoo import models, fields

class LibraryBook(models.Model):
    _name = "library.book"
    _description = "Library Book"

    name = fields.Char(required=True)
    published_date = fields.Date()
```

Import the file from `models/__init__.py`.

### 27. Compare `models.Model`, `models.TransientModel`, and `models.AbstractModel`.
`Model` stores persistent business records. `TransientModel` is intended for temporary wizard data that Odoo can clean up. `AbstractModel` provides reusable behavior without a normal standalone persistent table for its own records.

### 28. Compare `_name`, `_inherit`, and `_inherits`.
`_name` defines a model's technical name. `_inherit` extends an existing model or creates a new model based on an existing one when combined with `_name`. `_inherits` uses delegation inheritance, exposing fields from linked parent records through Many2one relationships.

### 29. What are common Odoo field types?
Common types include `Char`, `Text`, `Boolean`, `Integer`, `Float`, `Monetary`, `Date`, `Datetime`, `Selection`, `Binary`, `Html`, `Many2one`, `One2many`, and `Many2many`. Choose a type matching the data semantics.

### 30. What is a `Many2one` field?
It links a record to at most one record in another model. For example, `partner_id = fields.Many2one('res.partner')` can represent an order's customer.

### 31. What is a `One2many` field?
It represents records in another model whose inverse `Many2one` points to the current record. Example: `line_ids = fields.One2many('sale.order.line', 'order_id')`. It is the inverse view of the relationship, not a column storing a list of IDs.

### 32. What is a `Many2many` field?
It links multiple records on both sides, usually through a relation table. It is useful for tags, categories, skills, and groups. Odoo can generate the relation table automatically or accept explicit relation-table names.

### 33. What are relational field commands?
Commands modify One2many and Many2many relationships. Modern code can use `fields.Command.create(vals)`, `update(id, vals)`, `delete(id)`, `unlink(id)`, `link(id)`, `clear()`, and `set(ids)`. `delete` removes a related record, while `unlink` removes the relation and may preserve the related record.

### 34. How do `required=True` and SQL constraints differ?
`required=True` declares a field as required in Odoo's model/UI behavior. SQL constraints enforce database-level integrity, including for operations outside normal UI flows. Use database constraints for rules that the database can reliably enforce.

### 35. What is a `Selection` field?
It stores one value from a defined set, for example `state = fields.Selection([('draft', 'Draft'), ('done', 'Done')])`. Use stable technical keys and clear user-facing labels.

### 36. What is a `Monetary` field?
It stores an amount associated with a currency field, for example `amount = fields.Monetary(currency_field='currency_id')`. Use Odoo currency conversion and rounding utilities instead of assuming every currency has the same precision.

### 37. What is `_sql_constraints` used for?
It defines database constraints such as uniqueness or checks. Example:

```python
_sql_constraints = [
    ('code_unique', 'unique(code)', 'The code must be unique.'),
]
```

Adding constraints can fail during upgrades if existing data violates them.

### 38. How do you define a Python constraint?
Use `@api.constrains('field_name')` on a method that validates records when the named fields change. Raise `ValidationError` when a business rule is violated. Prefer SQL constraints for simple rules enforceable by the database.

### 39. What is the difference between classical and delegation inheritance?
Classical inheritance with `_inherit` extends model fields and behavior. Delegation with `_inherits` exposes parent fields through linked parent records. They differ in storage and method-resolution behavior.

### 40. What is `_rec_name`?
It specifies the field used as a record's display name in many relational widgets. Odoo commonly defaults to `name` when available. If another field identifies the record, configure `_rec_name` or the version-appropriate display-name behavior.

---

## 4. XML Views, Actions & Menus

### 41. What is an Odoo view?
A view defines how records appear in the web client. Common types include form, list, kanban, search, calendar, graph, pivot, and activity. Views are usually XML records linked to a model.

### 42. Compare form, list, and kanban views.
A form view displays or edits one record. A list view shows many records in rows and columns. A kanban view presents records as cards, often grouped by a status or category.

### 43. How do you create a window action?
A window action opens records for a model:

```xml
<record id="action_library_book" model="ir.actions.act_window">
    <field name="name">Books</field>
    <field name="res_model">library.book</field>
    <field name="view_mode">list,form</field>
</record>
```

Check the view-mode syntax supported by the exact Odoo version you target.

### 44. How do you create a menu item?
Define `ir.ui.menu` records and connect a menu to an action:

```xml
<menuitem id="menu_library_root" name="Library"/>
<menuitem id="menu_library_books" name="Books"
          parent="menu_library_root"
          action="action_library_book"/>
```

Menu visibility does not grant model access by itself.

### 45. What are XML external IDs?
External IDs identify records across module data files and allow other records to reference them. During upgrades, Odoo can update records by external ID instead of creating duplicates. Changing IDs can break references from other modules.

### 46. What does `xpath` do in an inherited view?
It locates an element in a parent view so an extension can insert, replace, or modify content. Use stable selectors and test the combined view because parent-view changes can make brittle XPath expressions fail.

### 47. How do you hide a field or button conditionally?
Use supported view modifiers such as `invisible`, `readonly`, or `required`. In Odoo 17/18, modifiers commonly use direct expressions, e.g. `invisible="state != 'draft'"`. UI visibility is not a security boundary; enforce important rules server-side.

### 48. Compare `domain` and `context` on a relational field.
A `domain` filters records offered in the relational selector. A `context` passes metadata or defaults to the related model. A domain is not a replacement for record rules because it does not prevent access through other routes.

### 49. What does `groups` do on a view element?
It limits display of an element to users in specified groups. This is useful for UI customization but does not replace access rights or record rules.

### 50. Compare manifest `data` and `demo`.
`data` files are loaded during normal installation and updates. `demo` files provide sample records and are generally loaded only when demo data is enabled. Keep file ordering correct when records reference one another.

---

## 5. Security, Access Rights & Record Rules

### 51. What are Odoo access rights?
Access rights grant model-level read, write, create, and delete permissions to groups. They are commonly defined in `ir.model.access.csv`. Effective permissions can be broadened by membership in multiple groups, so grant only what is required.

### 52. What is a record rule?
A record rule filters which records a user can access after model access rights are considered. Rules use domains and can be global or group-specific. Test how all applicable rules interact.

### 53. Compare access rights and record rules.
Access rights answer whether a user may perform an operation on a model at all. Record rules determine which records are available for that operation. Secure modules usually need both.

### 54. How do you define access rights in CSV?
A typical file looks like this:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_library_book_user,library.book user,model_library_book,base.group_user,1,1,1,0
```

Verify the model external ID and choose the minimum required permissions.

### 55. What is a global record rule?
A global rule applies independently of group-specific membership and restricts records for users to whom it applies. Poorly designed global rules can unexpectedly block legitimate access, so test them carefully.

### 56. How do group-specific record rules combine?
Group-specific rules can allow records matching any applicable group rule, while global rules continue to restrict access. The result depends on all applicable rules, so test users who belong to multiple groups.

### 57. Does hiding a menu or button secure an operation?
No. UI hiding improves usability but does not authorize server requests. Protect operations with access rights, record rules, and server-side validation because RPC or another UI path may invoke the same functionality.

### 58. What are `check_access_rights()` and `check_access_rule()`?
They are APIs for checking model-level permissions and record-level rules. Signatures and recommended usage can vary by version; consult the target version. The ORM automatically performs many checks in normal operations.

### 59. How should `sudo()` be used securely?
Use it only for a narrowly scoped operation that genuinely requires elevated access. Validate inputs, avoid returning sensitive data to unauthorized users, and never assume a button caller is trusted.

### 60. How do you troubleshoot an Access Error?
Check the user's groups, model access CSV, record-rule domains, company restrictions, related-model permissions, and custom code using `sudo()` or changing context. Reproduce the issue as the affected user rather than only as an administrator.

---

## 6. Computed Fields, Onchange, Constraints & ORM Methods

### 61. What is a computed field?
It derives a value from other fields using a Python method:

```python
total = fields.Float(compute="_compute_total", store=True)

@api.depends("quantity", "unit_price")
def _compute_total(self):
    for record in self:
        record.total = record.quantity * record.unit_price
```

The compute method should assign a value for every record in `self`.

### 62. What does `@api.depends` do?
It declares fields that affect a computed value. Odoo uses dependencies to trigger recomputation when inputs change. Include relevant relational paths such as `line_ids.price_subtotal` where needed.

### 63. Compare stored and non-stored computed fields.
A stored field uses `store=True`, persists its computed value, and is generally easier to search, group, and sort. A non-stored field is computed when accessed and is not stored as a normal database column. Stored fields require correct dependencies and may trigger significant recomputation.

### 64. What is an inverse method?
An inverse method explains how a manually supplied value for a computed field should update its source fields. Use it when the reverse mapping is meaningful and unambiguous; not every computed field needs an inverse.

### 65. What is `@api.onchange`?
It runs during form interaction when specified fields change, allowing the UI to suggest values or display warnings before saving. It is not a replacement for constraints because imports, RPC calls, and programmatic creates may not trigger the same UI flow.

### 66. Compare `@api.onchange` and `@api.depends`.
`onchange` supports interactive form behavior and can update unsaved values. `depends` declares dependencies for ORM computed-field recomputation. Use computed fields for values that must remain correct regardless of how data is changed.

### 67. How do you enforce business rules?
Use SQL constraints for database-enforceable rules and `@api.constrains` for Python validation. Critical rules must be enforced on the server, not only through view attributes or onchange methods.

### 68. Why override `create()` or `write()`?
Override them when business requirements must run during creation or update, such as validating transitions or recording custom behavior. Preserve batch support, avoid searches inside loops, and prefer existing computed-field or constraint mechanisms when appropriate.

### 69. How do you add a button method?
Create a public model method and call it from a form button with `type="object"`. Validate state and permissions inside the method; do not rely on the button's visibility as protection.

### 70. How should state transitions be implemented?
Define allowed states, expose clear transition actions, and validate the current state before changing it. Consider repeated calls, concurrent operations, and side effects such as creating stock moves or accounting entries.

---

## 7. Modules, Inheritance & Advanced Development

### 71. What is a typical custom module structure?
```text
my_module/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── my_model.py
├── views/
│   └── my_model_views.xml
├── security/
│   ├── ir.model.access.csv
│   └── security.xml
├── data/
├── controllers/
├── report/
└── tests/
```
Include only what is needed and list XML/CSV files in the manifest in dependency order.

### 72. Why are `__init__.py` files important?
They tell Python which subpackages and files to import. If a model file is not imported, Odoo may not register its model even though the file exists.

### 73. How does module dependency management work?
Declare required modules in `depends`. Odoo loads dependencies first. Avoid relying on accidental installation order or creating circular dependencies; move shared functionality into a common module if necessary.

### 74. How do you extend a standard model?
Use `_inherit` with the technical name:

```python
from odoo import models, fields

class ResPartner(models.Model):
    _inherit = "res.partner"

    customer_reference = fields.Char()
```

Keep overrides focused and test interactions with other installed modules.

### 75. How do you override a method safely?
Preserve its signature and return contract, support batch operations, call `super()` at the appropriate point, and avoid unexpected side effects. Check whether the original method accepts one record, a recordset, or a list of dictionaries.

### 76. What is method resolution order in Odoo?
Python's method resolution order determines which implementation is called through the inheritance chain. Final behavior can depend on installed modules and dependencies. Use cooperative `super()` calls rather than calling a specific parent implementation directly without a strong reason.

### 77. What are migrations?
Migrations move a database safely between module or Odoo versions. They may rename fields, transform data, update references, or adapt schema changes. Test migrations on a copy of production data and take verified backups.

### 78. How do you use external IDs in Python?
Use `self.env.ref('module.external_id')` to retrieve a record by external ID. Handle missing references when appropriate, and remember that a record's numeric database ID may differ across environments.

### 79. What are scheduled actions (cron jobs)?
Scheduled actions execute model methods periodically for tasks such as synchronization, reminders, or cleanup. Make jobs idempotent, process data in batches, handle failures, and do not assume a job executes exactly once.

### 80. How do you write automated Odoo tests?
Odoo supports Python tests using classes such as `TransactionCase` and, for suitable browser flows, `HttpCase`. Test business rules, permissions, computed values, edge cases, and regressions. Keep tests deterministic and isolate external services.

---

## 8. Controllers, APIs, QWeb & Reports

### 81. What is an Odoo controller?
A controller defines HTTP routes using Odoo's web framework. Controllers can render pages, receive form submissions, or expose endpoints for integrations. Import the controller package so its routes are registered.

### 82. What does `@http.route` do?
It registers a route and configures settings such as HTTP methods, authentication, request type, and CSRF behavior:

```python
from odoo import http
from odoo.http import request

class LibraryController(http.Controller):
    @http.route("/library/books", type="http", auth="user", methods=["GET"])
    def books(self, **kwargs):
        books = request.env["library.book"].search([])
        return request.render("my_module.library_book_page", {"books": books})
```

Use route settings appropriate to the endpoint and Odoo version.

### 83. Compare `type="http"` and `type="json"`.
`http` handles conventional HTTP requests and responses. `json` uses Odoo's JSON-RPC-style request/response handling in versions supporting that route type. Do not assume it behaves exactly like a generic REST JSON endpoint; verify the target version's protocol.

### 84. How do you secure an API endpoint?
Choose authentication deliberately, validate inputs, restrict methods, enforce permissions, and return only authorized data. Consider CSRF for browser-session routes, rate limiting, audit logs, and safe errors. Never expose privileged operations publicly without robust authorization.

### 85. How can an external system integrate with Odoo?
Depending on version, edition, and requirements, use supported external APIs, custom controllers, scheduled imports/exports, webhooks, or message queues. Define authentication, schemas, retries, idempotency, pagination, and error handling.

### 86. What is QWeb?
QWeb is Odoo's XML-based templating engine for web pages and reports. Templates render values, loop through records, and conditionally display content. Keep complex business calculations in Python when practical and escape user-provided content appropriately.

### 87. How do you create a PDF report?
Define a report action and a QWeb template, then configure the report metadata and paper format as needed. Odoo renders the template and converts it to PDF using the configured report engine. Test pagination, language, currency formatting, and multi-record output.

### 88. Compare a QWeb report and a form view.
A form view is an interactive interface for viewing and editing records. A QWeb report is formatted output intended for printing or export, such as an invoice or quotation. Reports need deliberate layout and pagination.

### 89. How do you return a file from a controller?
Return an HTTP response with the correct body and headers, such as `Content-Type` and a safe `Content-Disposition`. Check authorization before serving the file, avoid exposing filesystem paths, and consider memory use for large exports. Helper APIs can vary by version.

### 90. How should API errors be designed?
Use a consistent response schema, appropriate status codes where supported, and machine-readable error codes. Do not leak stack traces, credentials, or database details. Log useful diagnostics server-side while excluding secrets and unnecessary personal data.

---

## 9. Debugging, Performance, Deployment & Real-World Scenarios

### 91. How do you debug an Odoo module?
Start with the full server traceback and reproduce the issue in a development database. Identify whether it happens during module loading, a request, or a scheduled job. Inspect the relevant Python, XML, data, or security file and add a regression test after fixing it.

### 92. How do you update a module from the command line?
A common command is:

```bash
./odoo-bin -d mydb -u my_module --stop-after-init
```

On Windows, use the appropriate Python executable and `odoo-bin` path. Back up important data before risky upgrades and review logs for failed updates.

### 93. How do you improve slow ORM code?
- Avoid searches inside loops.
- Filter with domains in PostgreSQL.
- Use `read_group()` for aggregations.
- Fetch only necessary fields.
- Process large datasets in batches.
- Add indexes only when justified by query patterns.
- Profile with realistic data before optimizing.

### 94. Why is searching inside a loop inefficient?
It can create an N+1 query pattern: one query retrieves records and another query runs for each record. Batch the lookup, gather relevant IDs, and organize results in memory or use grouped queries.

### 95. What is an N+1 query problem?
It occurs when code fetches a set of records and then issues another query for each record. ORM prefetching helps some cases, but custom loops can still cause it. Use profiling or SQL logs to find repeated queries and batch reads/writes.

### 96. How do you handle multi-company behavior?
Respect active and allowed companies, use correct company fields, and test with users who have different company combinations. Where relevant, use `_check_company_auto` and `check_company=True` following the target version's conventions. Do not bypass company boundaries with broad `sudo()` calls.

### 97. Why should you avoid manual commits in model methods?
Odoo normally manages transactions at request or job boundaries. Manual commits can break atomicity and leave partial updates if later operations fail. Avoid `commit()` in ordinary business logic unless there is a carefully justified framework-aware requirement.

### 98. How do you deploy a custom module safely?
1. Put version-controlled code in the configured addons path.
2. Install required Python and system dependencies.
3. Back up the database and filestore.
4. Test installation or upgrade on staging.
5. Run the module upgrade during a planned deployment.
6. Review logs and validate workflows and permissions.
7. Monitor production and keep a rollback plan.

### 99. Scenario: A computed field is incorrect after another field changes. What do you check?
Verify that the compute method assigns a value for every record and that `@api.depends` includes every relevant input. For relational data, include the appropriate dependency path. Also investigate raw SQL, cache invalidation, and whether a migration requires recomputation.

### 100. Scenario: A user cannot edit a record or can access another company's record. How do you investigate?
Reproduce the issue as that user. For editing failures, inspect model write access, record rules, field group restrictions, readonly modifiers, and custom checks. For cross-company access, review company fields, allowed companies, rule domains, relational company checks, and use of `sudo()`. Fix authorization at the source instead of merely changing the UI.

---

## Final Study Checklist

- [ ] Create a module and declare dependencies.
- [ ] Define models, fields, and relational commands.
- [ ] Use `search`, `create`, `write`, `unlink`, domains, and recordsets.
- [ ] Implement computed fields, onchange methods, and constraints.
- [ ] Extend standard models and views safely.
- [ ] Create actions, menus, and XML data records.
- [ ] Configure access rights and record rules.
- [ ] Build a controller or integrate an external service.
- [ ] Create a QWeb PDF report.
- [ ] Debug module upgrades and optimize ORM queries.
- [ ] Test multi-company behavior and deployment procedures.

## Official References

- Odoo 17 Developer Documentation: https://www.odoo.com/documentation/17.0/developer.html
- Odoo 18 Developer Documentation: https://www.odoo.com/documentation/18.0/developer.html
- Odoo Source Code: https://github.com/odoo/odoo

**Interview tip:** Practice explaining not only what each feature does, but when to use it, its common failure modes, security implications, and how you would test it in a real business workflow.
