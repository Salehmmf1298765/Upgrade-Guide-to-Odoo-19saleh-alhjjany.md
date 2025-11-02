# Upgrade Guide to Odoo 19 - Main Index

## Overview
This comprehensive guide contains all the changes and errors that need to be addressed when upgrading from Odoo 16/17/18 to version 19.

---

Reference Files

### 1. **Changes in Models and Fields**
` Migration_V19_Models_Fields.md `
- Field changes in different models
Deleted and replaced fields
- Changes in res.partner, sale.order.line, account
- Changes in product_uom, tax_id

### 2. **Changes in Views and XML **
` Migration_V19_Views_XML.md`
Changes in the XML Views structure
Kanban Views Updates
Search Views changes
- Form Views Updates

### 3. **Changes in Security/Groups/Privileges **
` Migration_V19_Security_Groups.md`
- The new system of privileges
Changes to res.groups
- Changes to ir.module.category
- Restructuring Users and Groups

### 4. **Changes in UoM (Units of Measurement)**
` Migration_V19_UoM_Changes.md `
- Remove uom.category
- Changes to factor and factor_inv
- The new system relative_uom_id
Practical examples of promotion

### 5. **Changes in Python/Imports/Methods **
` Migration_V19_Python_Code.md`
- Imports changes
Deleted and replaced methods
Changes in ORM Methods
Registry and Context Updates

### 6. **Changes in HTTP Routes and Controllers **
` Migration_V19_Routes_Controllers.md `
- Change type='json ' to type='jsonrpc '
Controllers Updates
- Changes in Authentication

### 7. **Common Mistakes and Solutions**
` Migration_V19_Common_Errors.md`
List of the most common mistakes
Practical solutions for every mistake
- Examples from real-world practice
Tips to avoid mistakes

### 8. **Step-by-Step Upgrade Plan**
` Migration_V19_Step_by_Step.md`
Preparations before promotion
- Steps for upgrading in detail
- Required tests
Post-Migration Plan

---

Important points for programmers

### Basic Requirements
1. Proper use: Perform a full backup of the database.
2. Proper use: Create a separate test environment
3. Proper use: Review all Custom Modules
4. Proper use: Check compatibility with the new version.

### Work Priorities
1. **Phase One**: Updating Manifest Files and Dependencies
2. **Phase Two**: Handling Model and Field Changes
3. **Third Stage**: Updating Views and XML Files
4. **Fourth Stage**: Updating Security and Groups
5. **Fifth Stage**: Processing Python Code and Imports
6. **Sixth Stage**: Comprehensive Test

### Critical Changes ( Breaking Changes )
**It must be dealt with immediately:**
- res.groups.category_id → privilege_id
- type='json' → type='jsonrpc '
- from odoo import registry → from odoo.modules.registry import Registry
- self._context → self.env.context
- uom.category has been deleted
- res.partner.title has been deleted
- _apply_ir_rules () has been deleted

---

Change statistics

### By type:
- ** Models & Fields**: ~35 changes
- ** Views & XML**: ~15 changes
- ** Security & Groups**: ~20 changes
- ** Python/Imports**: ~25 changes
- ** UoM Changes**: ~10 changes
- ** Routes/Controllers**: ~8 changes

### According to difficulty:
- **Difficult** (requires restructuring): 25%
- **Average** (requires adjustments): 45%
- **Easy** (Direct Change): 30%

---

## Suggested Helpful Tools

### 1. Script for finding errors
bash
# Searching for old uses
grep -r "from odoo import registry "./
grep -r "type='json '" ./
grep -r "category_id.*ref =" ./
grep -r "self._context " ./
```

### 2. Checklist for Upgrade
- [ ] Update __ manifest__.py
- [ ] Models Review
- [ ] Review Views
Security Review
Python Code Review
- [ ] Comprehensive Test

---

## the reviewer
- Odoo Official Documentation 19.0
Community Migration Guides
- Practical Migration Experiences

---
# Changes to Models and Fields - Upgrade to Odoo 19

## 1. Changes in res.partner

Deleted fields
``` python
# Misuse This field was deleted in V19
mobile = fields.Char ()
```

**the solution:**
- Use the ` phone` field instead.
- Or create a new custom field if needed

**Example of modification:**
``` python
# V18
partner.mobile = '0123456789 '

# V19
partner.phone = '0123456789 '
# or
partner.x_mobile = '0123456789 ' # Custom field
```

---

## 2. Changes in sale.order.line

### 2.1 Change product_uom to product_uom_id
Python
# MisuseV18
product_uom = fields.Many2one('uom.uom')

# Proper use V19
product_uom_id = fields.Many2one('uom.uom')
```

### 2.2 Change tax_id to tax_ids
``` python
# MisuseV18
tax_id = fields.Many2one('account.tax')

# Proper use V19
tax_ids = fields.Many2many('account.tax')
```

**Practical example:**
``` python
# V18
line.tax_id = tax_record

# V19
line.tax_ids = [(6, 0, [tax_record.id])]
# or
line.tax_ids = [(4, tax_record.id)]
```

---

## 3. Changes in account.* Models

Deleted fields
``` python
# Misuse This field has been deleted
deprecated = fields.Boolean ()
```

**Note:** Review all modules related to Accounting and make sure that the deleted fields are not being used.

---

## 4. Changes in res.users

Groups Restructuring
``` python
# MisuseV18
class ResUsers(models.Model) :
_ inherit = 'res.users '
    
    groups_id = fields.Many2many('res.groups')

# Proper use V19
class ResUsers(models.Model) :
_ inherit = 'res.users '
    
    group_ids = fields.Many2many('res.groups')
```

**Changes:**
- ` groups_id` → ` group_ids`
Groups are now in a separate model .

---

## 5. Changes in res.groups

### New Structure
``` python
# V18
class ResGroups(models.Model) :
_ name = 'res.groups '
    
    category_id = fields.Many2one('ir.module.category')
    users = fields.Many2many('res.users')

# V19
class ResGroups(models.Model) :
_ name = 'res.groups '
    
    privilege_id = fields.Many2one('res.groups.privilege')
    user_ids = fields.Many2many('res.users')
```

**Main changes:**
1. ` category_id` → ` privilege_id.category_id`
2. ` users` → ` user_ids`
3. A new model has been added : ` res.groups.privilege`

---

## 6. Delete Model: res.partner.title

``` python
# Misuse This model has been completely deleted
_ name = 'res.partner.title '
```

**the solution:**
- Transfer data to alternative structures
- Or use the Selection field instead
- Or create a new Custom Model

**Example of an alternative:**
``` python
class ResPartner(models.Model) :
_ inherit = 'res.partner '
    
    title_custom = fields.Selection ([
(' mr', 'Mr .'),
(' mrs', 'Mrs .'),
(' ms', 'Ms .'),
(' dr', 'Dr .'),
], string='Title ')
```

---

## 7. Changes in UoM Models

### 7.1 Delete uom.category
Python
# Misuse This model has been deleted
_ name = 'uom.category '
```

### 7.2 Changes in uom.uom
``` python
# V18
class UoM(models.Model) :
_ name = 'uom.uom '
    
    category_id = fields.Many2one('uom.category')
    factor = fields.Float ()
    factor_inv = fields.Float ()
    uom_type = fields.Selection ([
(' bigger', 'Bigger ')
(' smaller', 'Smaller ')
(' reference', 'Reference ')
])

# V19
class UoM(models.Model) :
_ name = 'uom.uom '
    
    relative_uom_id = fields.Many2one('uom.uom')
    relative_factor = fields.Float ()
# Deleted: category_id, factor, factor_inv, uom_type
```

**Conversion:**
``` python
# V18
uom.category_id = category
uom.factor_inv = 28.3168
uom.uom_type = 'bigger '

# V19
uom.relative_uom_id = reference_uom
uom.relative_factor = 28.3168
```

---

## 8. Changes in ir.ui.menu

Changes in Fields
``` python
# V18
class IrUiMenu(models.Model) :
_ name = 'ir.ui.menu '
    
    action = fields.Reference ()
    group_ids = fields.Many2many('res.groups')

# V19
class IrUiMenu(models.Model) :
_ name = 'ir.ui.menu '
    
    action_id = fields.Reference ()
    action_model = fields.Char ()
    group_ids = fields.Many2many('res.groups') # Same name but different structure
```

Fields are now updated based on ` action_id` and ` action_model` instead of just ` action` .

---

## 9. Deleting _sql_constraints in some models

### What was deleted
``` python
# MisuseV18 - May not work in V19
_ sql_constraints = [
(' name_uniq', 'unique(name)', 'Name must be unique !')
]
```

**Alternative solution:**
``` python
# Proper use V19 - Use Python Constraints
from odoo import api, models
from odoo. exceptions import ValidationError

class YourModel(models.Model) :
_ name = 'your.model '
    
@ api.constrains('name')
    def _check_unique_name(self) :
        For record in self :
            if self.search_count([('name', '=', record.name), ('id', '!=', record.id)]) > 0 :
                raise ValidationError('Name must be unique!')
```

---

## 10. Changes in Context and Environment

### self._context → self.env.context
``` python
# MisuseV18
current_context = self._context
value = self._context.get('key')

# Proper use V19
current_context = self.env.context
value = self.env.context.get('key')
```

### self._uid → self.env.uid
``` python
# MisuseV18
current_user = self._uid

# Proper use V19
current_user = self.env.uid
```

---

## 11. Changes in ORM Methods

### 11.1 _where_calc and _apply_ir_rules
Python
# MisuseV18
query = self._where_calc(domain)
self._apply_ir_rules(query)

# Proper use V19
query = self._search(domain, bypass_access=True)
```

### 11.2 _ search method
``` python
# V19 - New Use
records = self._search (
    domain=[('state', '=', 'draft')] ,
    bypass_access=True , # New
    limit=10
)
```

---

## 12. Changes in Sequence

### Delete sequence attribute
``` python
# MisuseV18
class YourModel(models.Model) :
_ name = 'your.model '
_sequence = 'your_model_seq '

# Proper use V19
# Odoo automatically uses the default sequence from PostgreSQL
class YourModel(models.Model) :
_ name = 'your.model '
# No need for _ sequence
```

---

## 13. Changes in the _ write() Method

### Update in V19
Python
Note: _write ( ) now does not raise an error for non-existent records.
# V19
record._write(vals) # Will not raise an error if the record does not exist
```

---

## 14. Field Attributes Changes

### Delete Deprecated Attributes
Python
# Misuse was deleted in V19
field = fields.Char (
    deprecated=True , # deleted
    column_format='text ' # Deleted
)

# Proper use V19 - Do not use these Attributes
field = fields.Char ()
```

---

## Summary of Critical Changes

### It must be amended immediately:
1. Proper use `mobile` field in res.partner
2. Proper use `product_uom` → ` product_uom_id`
3. Proper use `tax_id` → ` tax_ids`
4. Proper use `groups_id` → ` group_ids`
5. Proper use `category_id` → `privilege_id` in res.groups
6. Proper use `users` → `user_ids` in res.groups
7. Proper use Delete ` res.partner.title` Model
8. Proper use Delete ` uom.category` Model
9. Proper use `factor_inv` → ` relative_factor`
10. Proper use `category_id` → `relative_uom_id` in UoM
11. Proper use `self._context` → ` self.env.context`
12. Proper use `self._uid` → ` self.env.uid`
13. Proper use `_where_calc()` → `_search(bypass_access=True) `
14. Proper use Delete ` _apply_ir_rules ()`
15. Proper use Delete ` deprecated` and ` column_format` attributes

---

## Script for error detection

bash
#!/ bin/bash
# Search for old fields in Custom Modules

echo "=== Search for mobile field ==="
grep -rn "mobile.*=.*fields \." ./

echo "=== Search for product_uom ==="
grep -rn "product_uom\s *=" ./

echo "=== Search for tax_id (non- tax_ids ) ==="
grep -rn "tax_id\s*=.*Many2one " ./

echo "=== Search for groups_id ==="
grep -rn "groups_id .*=" ./

echo "=== Search for category_id in res.groups ==="
grep -rn "category_id.*ir\.module\.category "./

echo "=== Search for res.partner.title ==="
grep -rn "res\.partner\.title "./

echo "=== Search for uom.category ==="
grep -rn "uom\.category " ./

echo "=== Search for self._context ==="
grep -rn "self\._context " ./

echo "=== Search for self._uid ==="
grep -rn "self\._uid " ./

echo "=== Search for _where_calc === "
grep -rn "_where_calc " ./

echo "=== Search for _apply_ir_rules === "
grep -rn "_apply_ir_rules "./
```

---# Changes in Views and XML - Upgrade to Odoo 19

## 1. Changes in Kanban Views

### 1.1 Change <kanban-box> to <card>
xml
<!-- MisuseV18 -->
<kanban>
< templates >
< t t-name="kanban-box ">
< div class="oe_kanban_card ">
< field name="name "/>
</ div >
</t>
</templates>
</ kanban >

Proper use V19 -->
<kanban>
< templates >
< t t-name="kanban-box ">
<card>
< field name="name "/>
</card>
</t>
</templates>
</ kanban >
```

**Note:** All `< div class="oe_kanban_card ">` must be changed to `< card >` to be compatible with the new UI .

---

## 2. Changes in Search Views

### 2.1 Delete expand and string from group tag
xml
<!-- MisuseV18 -->
<search>
< group expand="0" string="Group By ">
< filter name="group_by_customer" string="Customer "
                context="{'group_by': 'partner_id'} "/>
</group>
</search>

Proper use V19 -->
<search>
<group>
< filter name="group_by_customer" string="Customer "
                context="{'group_by': 'partner_id'} "/>
</group>
</search>
```

**Changes:**
Delete ` expand="0 "`
- Delete ` string="Group By "`
- Only `<group> ` without attributes

---

## 3. Changes in Form Views

### 3.1 FormView Controller and Debug Mode
xml
<!-- V19 - requires prop 'mode ' in debug mode -->
< form string="Form View ">
<!-- mode must be explicitly specified in debug mode -->
</ form >
```

**In JavaScript/OWL :**
javascript
// V19
< FormView mode="edit " />
```

---

## 4. Changes in Data Files (XML)

### 4.1 UoM Records Update
xml
<!-- MisuseV18 -->
< record id="product_uom_cubic_cm" model="uom.uom ">
< field name="name">cm³</field >
< field name="category_id" ref="uom.product_uom_categ_vol "/>
< field name="factor_inv">28.3168</field >
< field name="uom_type">bigger</field >
</record>

Proper use V19 -->
< record id="product_uom_cubic_cm" model="uom.uom ">
< field name="name">cm³</field >
< field name="relative_factor">28.3168</field >
< field name="relative_uom_id" ref="uom.product_uom_litre "/>
</record>
```

**Changes:**
Delete ` category_id`
Delete ` uom_type`
- ` factor_inv` → ` relative_factor`
- Add ` relative_uom_id`

---

## 5. Changes in Security/Groups XML

### 5.1 The new structure for res.groups
xml
<!-- MisuseV18 -->
< record id="base.module_category_sales_sales" model="ir.module.category ">
< field name="description">Helps you handle your quotations, sale orders and invoices.</field >
< field name="sequence">1</field >
</record>

< record id="group_sale_salesman" model="res.groups ">
< field name="name">User: Own Documents Only</field >
< field name="category_id" ref="base.module_category_sales_sales " />
< field name="implied_ids" eval="[(4, ref('base.group_user')] "/>
< field name="comment">The user will have access to his own data in the sales application.</field >
</record>

Proper use V19 -->
< record model="ir.module.category" id="module_category_sales ">
< field name="name">Sales</field >
< field name="sequence">5</field >
</record>

< record model="res.groups.privilege" id="res_groups_privilege_sales ">
< field name="name">Sales</field >
< field name="sequence">1</field >
< field name="category_id" ref="base.module_category_sales "/>
</record>

< record id="group_sale_salesman" model="res.groups ">
< field name="name">User: Own Documents Only</field >
< field name="sequence">10</field >
< field name="privilege_id" ref="res_groups_privilege_sales "/>
< field name="implied_ids" eval="[(4, ref('base.group_user')] "/>
< field name="comment">The user will have access to his own data in the sales application.</field >
</record>
```

**Transfer Steps:**
1. Create ` ir.module.category` if it does not already exist.
2. Create a new ` res.groups.privilege`
3. Link ` privilege_id` instead of ` category_id` in ` res.groups`
4. Add ` sequence` to the group

---

## 6. Changes to Menu Items

### 6.1 Update ir.ui.menu
xml
<!-- MisuseV18 -->
< record id="menu_sale" model="ir.ui.menu ">
< field name="name">Sales</field >
< field name="action" ref="action_orders "/>
< field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman')] "/>
</record>

Proper use V19 -->
< record id="menu_sale" model="ir.ui.menu ">
< field name="name">Sales</field >
< field name="action_id" ref="action_orders "/>
< field name="action_model">sale.order</field >
< field name="groups_id" eval="[(4, ref('sales_team.group_sale_salesman')] "/>
</record>
```

**Changes:**
- ` action` → ` action_id`
- Adding ` action_model` (optional but recommended)

---

7. Changes in Tree Views

### 7.1 Attributes Update
xml
<!-- V18 & V19 - often the same structure -->
< tree string="Orders" decoration-info="state=='draft '">
< field name="name "/>
< field name="partner_id "/>
< field name="amount_total "/>
</tree>
```

**Note:** Tree views generally haven't changed much, but be sure to:
- Use new field names (e.g., ` product_uom_id` instead of ` product_uom` )
- Do not use deleted fields (such as ` mobile` in res.partner )

---

## 8. Changes in Report Templates

QWeb Reports Update
xml
<!-- Make sure you use the correct field names -->
<!-- MisuseV18 -->
< t t-foreach="doc.order_line" t-as="line ">
< span t-field="line. product_uom "/>
< span t-field="line. tax_id "/>
</t>

Proper use V19 -->
< t t-foreach="doc.order_line" t-as="line ">
< span t-field="line. product_uom_id "/>
< span t-esc="', '.join(line.tax_ids.mapped('name')) "/>
</t>
```

---

## 9. Changes in Action Windows

### 9.1 Update ir.actions.act_window
xml
<!-- V18 & V19 - Generally the same structure -->
< record id="action_sale_order" model="ir.actions.act_window ">
< field name="name">Sales Orders</field >
< field name="res_model">sale.order</field >
< field name="view_mode">tree,form,kanban</field >
< field name="domain">[]</field >
< field name="context">{}</field >
</record>
```

**Note:** Check the context to make sure that outdated fields are not being used.

---

## 10. Changes in Server Actions

### 10.1 New restrictions on ir.actions.server
xml
<!-- MisuseV18 - May not work in V19 -->
< record id="action_server_example" model="ir.actions.server ">
< field name="name">Example Action</field >
< field name="model_id" ref="model_sale_order "/>
< field name="state">code</field >
< field name="code ">
        record.sudo().action_confirm ()
</ field >
< field name="groups_id" eval="[(4, ref('sales_team.group_sale_manager')] "/>
</record>

<!-- Proper use V19 - sudo cannot be used with groups_id -->
< record id="action_server_example" model="ir.actions.server ">
< field name="name">Example Action</field >
< field name="model_id" ref="model_sale_order "/>
< field name="state">code</field >
< field name="code ">
        record.action_confirm ()
</ field >
< field name="groups_id" eval="[(4, ref('sales_team.group_sale_manager')] "/>
</record>
```

**Important rule:** Do not use ` sudo ()` and ` groups_id` at the same time in Server Actions .

---

## 11. Changes in Field Widgets

### 11.1 Review Widgets in use
xml
<!-- Make sure Widgets are compatible with V19 -->
< field name="partner_id" widget="many2one "/>
< field name="date" widget="date "/>
< field name="amount" widget="monetary "/>
```

**Note:** Most widgets have not changed, but some custom widgets may need updating.

---

## 12. Changes in Inherit Views

### 12.1 xpath expressions
xml
<!-- V18 & V19 - Same method -->
< record id="view_order_form_inherit" model="ir.ui.view ">
< field name="name">sale.order.form.inherit</field >
< field name="model">sale.order</field >
< field name="inherit_id" ref="sale.view_order_form "/>
< field name="arch" type="xml ">
< xpath expr="//field[@name='partner_id']" position="after ">
< field name="custom_field "/>
</xpath>
</ field >
</record>
```

**Warning:** If you inherit a view that uses old fields, you must update the xpath .

---

## 13. General Tips for Views

### Before the promotion:
1. Proper use: Review all XML files
2. Proper use: Search for old fields
3. Proper use: Ensure that you do not use deleted models .
4. Proper use: Check groups and categories.
5. Proper use: Test the views in a development environment.

### After the promotion:
1. Proper use: Open all Forms and Tree views
2. Proper use: Test the Search views and Filters
3. Proper use: Refer to Kanban views
4. Proper use: Test the reports
5. Proper use: Ensure the actions are working correctly.

---

## 14. Script for troubleshooting Views

bash
#!/ bin/bash
# Searching for XML problems in Views

echo "=== Searching for the old kanban-box ==="
grep -rn "t-name=\"kanban-box\"" --include="*.xml " ./

echo "=== Search for expand in group ==="
grep -rn "group expand=" --include="*.xml " ./

echo "=== Search for category_id in res.groups ==="
grep -rn "category_id.*ref=" --include="*.xml" ./security/
grep -rn "category_id.*ref=" --include="*.xml" ./data/

echo "=== Searching for factor_inv in UoM ==="
grep -rn "factor_inv" --include="*.xml " ./

echo "=== Search for uom_type ==="
grep -rn "uom_type" --include="*.xml " ./

echo "=== Searching for category_id in UoM ==="
grep -rn "uom\.category" --include="*.xml " ./

echo "=== Search for product_uom (not product_uom_id ) ==="
grep -rn "name=\"product_uom\"" --include="*.xml " ./

echo "=== Search for tax_id (non- tax_ids ) ==="
grep -rn "name=\"tax_id\"" --include="*.xml " ./

echo "=== Search for mobile field ==="
grep -rn "name=\"mobile\"" --include="*.xml " ./

echo "=== Search for res.partner.title ==="
grep -rn "res\.partner\.title" --include="*.xml " ./
```

---

## 15. Checklist for Views

XML Data Files :
- [ ] UoM records update
- [ ] Update Groups and Privileges
Menu items update
Server Actions Review

View Files :
- [ ] Kanban views update
- [ ] Update Search views
- [ ] Review Form views
Tree Views Review
- [ ] Update Report templates

### Field Names :
- [ ] ` product_uom` → ` product_uom_id`
- [ ] ` tax_id` → ` tax_ids`
- [ ] ` groups_id` → ` group_ids`
- [ ] ` category_id` → `privilege_id` (in res.groups )
mobile` in res.partner [ ]

---
# Changes in Security/Groups/Privileges - Upgrade to Odoo 19

## Overview
Odoo 19 introduced a completely new Privileges System that restructures the way groups and permissions are managed.

---

## 1. The new system structure

### 1.1 The relevant models

#### Before V19 :
```
ir.module.category
↓
res.groups (category_id)
↓
res.users (groups_id)
```

#### In V19 :
```
ir.module.category
↓
res.groups.privilege (category_id) ← **NEW**
↓
res.groups (privilege_id)
↓
res.users (group_ids)
```

---

## 2. New Model : res.groups.privilege

### 2.1 Definition
``` python
# V19 - New Model
class ResGroupsPrivilege(models.Model) :
_name = 'res.groups.privilege '
_ description = 'Groups Privilege '
    
    name = fields.Char(required=True)
    category_id = fields.Many2one('ir.module.category', required=True)
    sequence = fields.Integer(default=10)
```

### 2.2 XML Example
xml
< record model="res.groups.privilege" id="privilege_sales ">
< field name="name">Sales</field >
< field name="category_id" ref="base.module_category_sales "/>
< field name="sequence">1</field >
</record>
```

---

## 3. Changes in res.groups

### 3.1 Modifications in Model
``` python
# MisuseV18
class ResGroups(models.Model) :
_ name = 'res.groups '
    
    name = fields.Char(required=True)
    category_id = fields.Many2one('ir.module.category')
    users = fields.Many2many('res.users', 'res_groups_users_rel ',
' gid', 'uid ')
    implied_ids = fields.Many2many('res.groups')
    comment = fields.Text ()

# Proper use V19
class ResGroups(models.Model) :
_ name = 'res.groups '
    
    name = fields.Char(required=True)
    privilege_id = fields.Many2one('res.groups.privilege') # New
    sequence = fields.Integer(default=10) # new
    user_ids = fields.Many2many('res.users', 'res_groups_users_rel ',
' gid', 'uid ') # Change Name
    implied_ids = fields.Many2many('res.groups')
    comment = fields.Text ()
```

### 3.2 Main changes:
1. **Delete:** ` category_id`
2. **Add:** ` privilege_id`
3. **Addition:** ` sequence`
4. **Change name:** ` users` → ` user_ids`

---

## 4. Changes in res.users

### 4.1 Modifications in Model
``` python
# MisuseV18
class ResUsers(models.Model) :
_ name = 'res.users '
    
    groups_id = fields.Many2many('res.groups', 'res_groups_users_rel ',
' uid', 'gid ')

# Proper use V19
class ResUsers(models.Model) :
_ name = 'res.users '
    
    group_ids = fields.Many2many('res.groups', 'res_groups_users_rel ',
' uid', 'gid ')
```

### 4.2 In the code:
``` python
# MisuseV18
user.groups_id = [(4, group_id)]

# Proper use V19
user.group_ids = [(4, group_id)]
```

---

## 5. Delete Class Group Impelled

### What was deleted
``` python
# MisuseV18 - Deleted in V19
class GroupsImplied(models.Model) :
_name = 'res.groups.implied '
# ... This model has been deleted
```

**Solution:** Use ` implied_ids` directly in ` res.groups` .

---

## 6. A complete practical example of promotion

### 6.1 V18 Code
xml
<!-- security/security.xml - V18 -->

<!-- Definition of Category -->
< record id="module_category_custom" model="ir.module.category ">
< field name="name">Custom Module</field >
< field name="description">Custom module category</field >
< field name="sequence">10</field >
</record>

<!-- Definition of Groups -->
< record id="group_custom_user" model="res.groups ">
< field name="name">User</field >
< field name="category_id" ref="module_category_custom "/>
< field name="implied_ids" eval="[(4, ref('base.group_user')] "/>
</record>

< record id="group_custom_manager" model="res.groups ">
< field name="name">Manager</field >
< field name="category_id" ref="module_category_custom "/>
< field name="implied_ids" eval="[(4, ref('group_custom_user')] "/>
</record>
```

### 6.2 V19 Code
xml
<!-- security/security.xml - V19 -->

<!-- Definition of Category (same) -->
< record id="module_category_custom" model="ir.module.category ">
< field name="name">Custom Module</field >
< field name="description">Custom module category</field >
< field name="sequence">10</field >
</record>

<!-- **NEW**: Definition of Privilege -->
< record id="privilege_custom" model="res.groups.privilege ">
< field name="name">Custom Privilege</field >
< field name="category_id" ref="module_category_custom "/>
< field name="sequence">1</field >
</record>

<!-- Definition of Groups (in the new structure) -->
< record id="group_custom_user" model="res.groups ">
< field name="name">User</field >
< field name="privilege_id" ref="privilege_custom "/>
< field name="sequence">10</field >
< field name="implied_ids" eval="[(4, ref('base.group_user')] "/>
</record>

< record id="group_custom_manager" model="res.groups ">
< field name="name">Manager</field >
< field name="privilege_id" ref="privilege_custom "/>
< field name="sequence">20</field >
< field name="implied_ids" eval="[(4, ref('group_custom_user')] "/>
</record>
```

---

## 7. Changes in ir.model.access (CSV)

### 7.1 General Structure
``` csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
```

**Note:** The overall structure hasn't changed in V19 , but be sure to:
- Use the correct group names
- Verify that the groups actually exist

### 7.2 Example
``` csv
# security/ir.model.access.csv - V18 & V19
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_custom_model_user,custom.model.user,model_custom_model,group_custom_user,1,1,1,0
access_custom_model_manager,custom.model.manager,model_custom_model,group_custom_manager,1,1,1,1
```

---

## 8. Changes to Record Rules

### 8.1 General Structure
xml
<!-- V18 & V19 - Same Structure -->
< record id="rule_custom_model_user" model="ir.rule ">
< field name="name">User: Own Records</field >
< field name="model_id" ref="model_custom_model "/>
< field name="domain_force">[('user_id', '=', user.id)]</field >
< field name="groups" eval="[(4, ref('group_custom_user')] "/>
</record>
```

**Note:** Record rules haven't changed, but make sure the Group names are correct .

---

## 9. Working with Groups in Python

### 9.1 Searching for Groups
Python
# MisuseV18
groups = self.env['res.groups'].search ([
(' category_id', '=', category_id )
])

# Proper use V19
privilege = self.env['res.groups.privilege'].search ([
(' category_id', '=', category_id )
], limit=1 )
groups = self.env['res.groups'].search ([
(' privilege_id', '=', privilege.id )
])
```

### 9.2 Adding Users to Groups
``` python
# MisuseV18
group.users = [(4, user.id)]

# Proper use V19
group.user_ids = [(4, user.id)]
```

### 9.3 Checking Group membership
``` python
# MisuseV18
if group in user.groups_id :
    pass

# Proper use V19
if group in user.group_ids :
    pass

# or
if user.has_group('module.group_name') :
    pass # This method hasn't changed
```

---

## 10. Decorator for Groups

### 10.1 in Methods
``` python
from odoo import api, models
from odoo. exceptions import AccessError

# V18 & V19 - Same method
class CustomModel(models.Model) :
_ name = 'custom.model '
    
@ api.model
    def create(self, vals) :
        if not self.env.user.has_group('module.group_manager') :
            raise AccessError('Only managers can create records')
        return super().create(vals)
```

**Note:** ` has_group ()` has not changed, but make sure you are using the correct XML ID for the group .

---

## 11. Changes to Menu Items and Actions

### 11.1 Linking Groups to Menus
xml
<!-- V18 & V19 - Same method -->
< record id="menu_custom" model="ir.ui.menu ">
< field name="name">Custom Menu</field >
< field name="action" ref="action_custom "/>
< field name="groups_id" eval="[(4, ref('group_custom_user')] "/>
</record>
```

### 11.2 Linking Groups to Views
xml
<!-- V18 & V19 - Same method -->
< record id="view_custom_form" model="ir.ui.view ">
< field name="name">custom.form</field >
< field name="model">custom.model</field >
< field name="groups_id" eval="[(4, ref('group_custom_manager')] "/>
< field name="arch" type="xml ">
<form>
<!-- ... -->
</ form >
</ field >
</record>
```

---

## 12. Groups in Field-level Security

### 12.1 in Python Models
``` python
# V18 & V19 - Same method
class CustomModel(models.Model) :
_ name = 'custom.model '
    
    name = fields.Char ()
    secret_field = fields.Char(groups="module.group_manager")
```

### 12.2 in XML Views
xml
<!-- V18 & V19 - Same method -->
<form>
< field name="name "/>
< field name="secret_field" groups="module.group_manager "/>
</ form >
```

---

## 13. Practical Promotion Steps

Step 1: Identify all groups
bash
# Search all groups in module
grep -rn "model=\"res.groups\"" ./security/
```

Step 2: For each group , do the following:
1. Proper use: Specify the user's ` category_id`
2. Proper use : Create a new ` res.groups.privilege`
3. Proper use update ` res.groups` to use ` privilege_id`
4. Proper use: Add ` sequence` to the group

Step 3: Update References
bash
# Search for all uses of groups_id in Users
grep -rn "groups_id " ./
# Replace with group_ids
```

Step 4: Testing
1. Proper use of module installation/upgrade
2. Proper use Check Settings → Users & Companies → Groups
3. Proper use of new Privileges verification
4. Proper use test of validity

---

## 14. Script for automatic upgrade

14.1 Python Script for Generating Privileges
``` python
#!/ usr/bin/env python3
"""
Script to generate res.groups.privilege from old res.groups
"""
import xml.etree.ElementTree as ET

def generate_privilege_from_group(group_xml_id, group_name, category_ref, sequence=1) :
    privilege_id = f"privilege_{group_xml_id.split('_', 1 )[1] } "
    
    return f """
< record model="res.groups.privilege" id="{privilege_id} ">
< field name="name">{group_name}</field >
< field name="category_id" ref="{category_ref} "/>
< field name="sequence">{sequence}</field >
</record>
"""

# Usage:
# privilege_xml = generate_privilege_from_group (
# ' group_custom_user ',
# ' Custom User ',
# ' module.category_custom ',
#1
# )
```

### 14.2 Bash Script for Search and Replace
bash
#!/ bin/bash
# Replace groups_id with group_ids in Python files

find . -name "*.py" -type f -exec sed -i 's/\.groups_id/\.group_ids/g ' {} +

echo " groups_id has been replaced with group_ids in all Python files "
```

---

## 15. Common Mistakes and Solutions

Error 1: Category_id in res.groups
```
Error: Field 'category_id' does not exist in 'res.groups '
```

**the solution:**
xml
<!-- Replace -->
< field name="category_id" ref ="..."/>
<!-- B -->
< field name="privilege_id" ref="privilege_xxx "/>
```

### Error 2: ' users' field not found
```
Error: Field 'users' does not exist
```

**the solution:**
``` python
# Replace
group.users = [(4, user.id)]
# B
group.user_ids = [(4, user.id)]
```

Error 3: groups_id in res.users
```
Error: Field 'groups_id' does not exist in 'res.users '
```

**the solution:**
``` python
# Replace
user.groups_id
# B
user.group_ids
```

---

## 16. Checklist for Upgrade

### Before the promotion:
- [ ] Create a list of all groups in the Module
- [ ] Specify the categories used
- [ ] Backup security files

### During the promotion:
- [ ] Create ` res.groups.privilege` for each category
- [ ] Update all ` res.groups` records
- [ ] Replace ` category_id` with ` privilege_id`
- [ ] Replace ` users` with ` user_ids`
- [ ] Replace ` groups_id` with ` group_ids` in res.users
- [ ] Add ` sequence` to groups

### After the promotion:
Module Installation Test
- [ ] Check Groups in Settings
- [ ] Testing the Authority
- [ ] Check Menu visibility
- [ ] Record rules test

---
# Changes in UoM (Units of Measurement) - Upgrade to Odoo 19

## Overview
Odoo 19 has completely restructured the Unit of Measure system to make it simpler and more straightforward.

---

## 1. Key Changes

### 1.1 What was deleted
``` python
# Misuse was deleted in V19
- uom.category ( Full Model )
- uom_type ( Field in uom.uom )
- factor ( Field in uom.uom )
- factor_inv ( Field in uom.uom )
```

### 1.2 What was added
``` python
# Proper use (new in V19)
- relative_uom_id ( Many2one in uom.uom )
- relative_factor ( Float in uom.uom )
```

---

## 2. Delete Model: uom.category

### 2.1 Old Architecture ( V18 )
``` python
class UoMCategory(models.Model) :
_ name = 'uom.category '
_ description = 'Product UoM Categories '
    
    name = fields.Char(required=True)
    measure_type = fields.Selection ([
(' unit', 'Units '),
(' weight', 'Weight ')
(' volume', 'Volume '),
(' length', 'Length ')
(' working_time', 'Working Time '),
])
```

### 2.2 in V19
```
MisuseModel completely deleted
uom.category no longer exists
```

**Solution:** Use ` relative_uom_id` to directly link the units.

---

## 3. Changes in the uom.uom Model

### 3.1 Full Comparison

#### V18 :
``` python
class UoM(models.Model) :
_ name = 'uom.uom '
    
    name = fields.Char(required=True)
    category_id = fields.Many2one('uom.category', required=True)
    factor = fields.Float(digits=0, required=True, default=1.0)
    factor_inv = fields.Float (
        digits=0 ,
        compute='_compute_factor_inv ',
        readonly=False ,
        required=True ,
        default=1.0
)
    uom_type = fields.Selection ([
(' bigger', 'Bigger than the reference Unit of Measure '),
(' reference', 'Reference Unit of Measure for this category '),
(' smaller', 'Smaller than the reference Unit of Measure ')
], required=True, default='reference ')
    rounding = fields.Float(digits=0, default=0.01, required=True)
    active = fields.Boolean(default=True)
```

#### V19 :
``` python
class UoM(models.Model) :
_ name = 'uom.uom '
    
    name = fields.Char(required=True)
    relative_uom_id = fields.Many2one('uom.uom') # new
    relative_factor = fields.Float (# new
        digits=0 ,
        default=1.0 ,
        required=True
)
    rounding = fields.Float(digits=0, default=0.01, required=True)
    active = fields.Boolean(default=True)
    
# Deleted:
# - category_id
# - factor
# - factor_inv
# - uom_type
```

---

## 4. How to convert: practical examples

### 4.1 Example: Cubic Centimeter (cm³)

#### V18 :
xml
< record id="product_uom_cubic_cm" model="uom.uom ">
< field name="name">cm³</field >
< field name="category_id" ref="uom.product_uom_categ_vol "/>
< field name="factor_inv">28.3168</field >
< field name="uom_type">bigger</field >
< field name="rounding">0.01</field >
</record>
```

#### V19 :
xml
< record id="product_uom_cubic_cm" model="uom.uom ">
< field name="name">cm³</field >
< field name="relative_uom_id" ref="uom.product_uom_litre "/>
< field name="relative_factor">28.3168</field >
< field name="rounding">0.01</field >
</record>
```

**Transfer Notes:**
Delete ` category_id` ← No more categories
- Delete ` uom_type` ← The system will automatically calculate it from relative_factor
- ` factor_inv` → `relative_factor` ← Same value
- Add ` relative_uom_id` ← reference unit

---

### 4.2 Example: Kilometer

#### V18 :
xml
< record id="product_uom_km" model="uom.uom ">
< field name="name">km</field >
< field name="category_id" ref="uom.product_uom_categ_length "/>
< field name="factor_inv">1000</field >
< field name="uom_type">bigger</field >
</record>
```

#### V19 :
xml
< record id="product_uom_km" model="uom.uom ">
< field name="name">km</field >
< field name="relative_uom_id" ref="uom.product_uom_meter "/>
< field name="relative_factor">1000</field >
</record>
```

---

### 4.3 Example: Milliliter (مليتر)

#### V18 :
xml
< record id="product_uom_ml" model="uom.uom ">
< field name="name">ml</field >
< field name="category_id" ref="uom.product_uom_categ_vol "/>
< field name="factor">1000</field >
< field name="uom_type">smaller</field >
</record>
```

#### V19 :
xml
< record id="product_uom_ml" model="uom.uom ">
< field name="name">ml</field >
< field name="relative_uom_id" ref="uom.product_uom_litre "/>
< field name="relative_factor">0.001</field >
</record>
```

**note:**
- In V18: `factor=1000` means 1000 ml = 1 liter
- In V19: `relative_factor=0.001` means 1 ml = 0.001 liter

---

## 5. Understanding the relative_factor

### 5.1 Equation
```
1 [this unit] = relative_factor × [reference unit]
```

### 5.2 Examples:

Example 1: Kilometer
```
1 km = 1000 × meter
relative_uom_id = meter
relative_factor = 1000
```

Example 2: Centimeter
```
1 cm = 0.01 × meter
relative_uom_id = meter
relative_factor = 0.01
```

Example 3: Ton
```
1 ton = 1000 × kg
relative_uom_id = kg
relative_factor = 1000
```

Example 4: Gram
```
1 gram = 0.001 × kg
relative_uom_id = kg
relative_factor = 0.001
```

---

## 6. Conversion from factor/factor_inv

### 6.1 Rules

#### If uom_type = 'bigger ':
Python
# V18
factor_inv = X
# V19
relative_factor = X # same value
```

#### If uom_type = 'smaller ':
Python
# V18
factor = X
# V19
relative_factor = 1 / X # Inverse of value
```

#### If uom_type = 'reference ':
Python
# V19
relative_uom_id = False # or do not specify it
relative_factor = 1.0
```

---

## 7. Creating a new UoM in V19

### 7.1 Example: Creating a " Dozen "

xml
< record id="product_uom_dozen" model="uom.uom ">
< field name="name">Dozen</field >
< field name="relative_uom_id" ref="uom.product_uom_unit "/>
< field name="relative_factor">12</field >
< field name="rounding">1</field >
</record>
```

**Explanation:** 1 Dozen = 12 Units

---

### 7.2 Example: Creating a " Pallet "

xml
< record id="product_uom_pallet" model="uom.uom ">
< field name="name">Pallet</field >
< field name="relative_uom_id" ref="uom.product_uom_unit "/>
< field name="relative_factor">100</field >
< field name="rounding">1</field >
</record>
```

**Explanation:** 1 Pallet = 100 Units

---

## 8. Working with UoM in Python

### 8.1 Obtaining UoM
``` python
# V18 & V19 - Same method
kg_uom = self.env.ref('uom.product_uom_kgm')
```

8.2 Unit Conversion
``` python
# V18 & V19 - Same method
product_uom = self.env['uom.uom']

# Conversion from one unit to another
qty_in_kg = product_uom._compute_quantity (
    Qty=10 , # Quantity
    from_unit=gram_uom , # from Gram
    to_unit=kg_uom , # to kilo
    round=True
)
# Result: 0.01 kg
```

### 8.3 Searching for UoM
``` python
# MisuseV18
uoms = self.env['uom.uom'].search ([
(' category_id', '=', category.id )
])

# Proper use V19
# No category_id found , search using relative_uom_id
reference_uom = self.env.ref('uom.product_uom_kgm')
uoms = self.env['uom.uom'].search ([
'|',
(' id', '=', reference_uom.id ),
(' relative_uom_id', '=', reference_uom.id )
])
```

---

## 9. Compatibility Check

Python 9.1
``` python
# V18
def _check_uom_compatibility(uom1, uom2) :
    return uom1.category_id == uom2.category_id

# V19
def _check_uom_compatibility(uom1, uom2) :
# We need to check the reference uom
    ref1 = uom1.relative_uom_id or uom1
    ref2 = uom2.relative_uom_id or uom2
    return ref1 == ref2
```

---

## 10. UoM data upgrade available

### 10.1 Migration Script Example
``` python
# in migration script
def migrate(cr, version) :
" UoM upgrade from V18 to V19 "
    
#1. Reading old data
    cr.execute ("")
        SELECT id, name, category_id, factor, factor_inv, uom_type
        FROM uom_uom
        WHERE category_id IS NOT NULL
""")
    
    old_uoms = cr.fetchall ()
    
    for uom_id, name, category_id, factor, factor_inv, uom_type in old_uoms :
# 2. Finding the reference UoM
        cr.execute ("")
            SELECT id FROM uom_uom
            WHERE category_id = %s AND uom_type = 'reference '
            LIMIT 1
""", (category_id,) )
        
        ref_uom = cr.fetchone ()
        if not ref_uom :
            continue
            
# 3. Calculating relative_factor
        if uom_type == 'bigger ':
            relative_factor = factor_inv
        elif uom_type == 'smaller ':
            relative_factor = 1.0 / factor if factor else 1.0
        else: # reference
            relative_factor = 1.0
            ref_uom = None
        
# 4. Update the record
        cr.execute ("")
            UPDATE uom_uom
            SET relative_uom_id = %s ,
                relative_factor = %s
            WHERE id = %s
"", (ref_uom[0] if ref_uom else None, relative_factor, uom_id) )
```

---

## 11. Common Mistakes and Solutions

### Error 1: category_id doesn't exist
```
Error: field 'category_id' does not exist in model 'uom.uom '
```

**the solution:**
xml
Delete -->
< field name="category_id" ref ="..."/>

<!-- and replace with -->
< field name="relative_uom_id" ref ="..."/>
```

---

Error 2: factor_inv doesn't exist
```
Error: field 'factor_inv' does not exist
```

**the solution:**
xml
<!-- Replace -->
< field name="factor_inv">1000</field >

<!-- B -->
< field name="relative_factor">1000</field >
```

---

Error 3: uom_type doesn't exist
```
Error: field 'uom_type' does not exist
```

**the solution:**
xml
Delete this line -->
< field name="uom_type">bigger</field >

<!-- The system automatically calculates the type from the relative_factor -->
```

---

### Error 4: model 'uom.category' doesn't exist
```
Error: The model uom. category does not exist
```

**the solution:**
Delete all references to ` uom.category` in:
XML files
Python code
- CSV files

---

## 12. Checklist for Upgrade

### For XML Data Files :
Delete all `< record model="uom.category ">`
- [ ] in `< record model="uom.uom ">`:
Delete ` category_id`
` uom_type` - [ ]
- [ ] ` factor_inv` → ` relative_factor`
- [ ] ` factor` → `1/ relative_factor`
] Add ` relative_uom_id`

### For Python Code :
Delete ` uom.category` imports
UoMs search update
- [ ] Compatibility Check Update
- [ ] Computed fields review

### For Views :
- [ ] Delete category_id from Views
- [ ] Add relative_uom_id
- [ ] Add relative_factor
- [ ] Delete uom_type, factor, factor_inv

---

13. Script for troubleshooting

bash
#!/ bin/bash
# Research on old UoM uses

echo "=== Search for uom.category ==="
grep -rn "uom\.category " ./

echo "=== Searching for category_id in UoM ==="
grep -rn "category_id.*uom "./

echo "=== Searching for factor_inv ==="
grep -rn "factor_inv " ./

echo "=== Search for uom_type ==="
grep -rn "uom_type " ./

echo "=== Searching for factor field ==="
grep -rn "\"factor\"" ./data/ ./models/

echo "=== XML Verification ==="
grep -rn "model=\"uom\.category\"" --include="*.xml " ./
```

---

## 14. Quick Conversion Table

| V18 Field | V19 Field | Notes |
|-----------|-----------|----------|
| ` category_id` | `relative_uom_id` | From category to UoM direct |
| ` factor_inv` | `relative_factor` | Same value (for bigger ) |
| ` factor` | `1/relative_factor` | Inverse value (of smaller ) |
| ` uom_type ` | *Deleted* | Calculated automatically |

---

15. Examples from Odoo Standard

### Standard units in V19 :

Length :
```
- meter (reference) - relative_uom_id = False
- km - relative_uom_id = meter, relative_factor = 1000
- cm - relative_uom_id = meter, relative_factor = 0.01
```

Weight :
```
- kg (reference) - relative_uom_id = False
- ton - relative_uom_id = kg, relative_factor = 1000
- gram - relative_uom_id = kg, relative_factor = 0.001
```

Volume :
```
- liter (reference) - relative_uom_id = False
- cubic meter - relative_uom_id = liter, relative_factor = 1000
- ml - relative_uom_id = liter, relative_factor = 0.001
```

---
# Changes in Python Code/Imports/Methods - Upgrade to Odoo 19

## 1. Changes in Imports

### 1.1 Registry Import
``` python
# MisuseV18
from odoo import registry

# Proper use V19
from odoo.modules.registry import Registry
```

**Usage:**
``` python
# MisuseV18
reg = registry(db_name)

# Proper use V19
reg = Registry(db_name)
```

---

### 1.2 XlsxWriter Import
``` python
# MisuseV18
from odoo.tools.misc import xlsxwriter

# Proper use V19
import xlsxwriter
```

**Note:** Now the library is imported directly instead of odoo.tools.misc

---

### 1.3 Module Resources Import
``` python
# MisuseV18
from odoo.modules.module import get_module_resource

# Proper use V19
from odoo. modules import get_resource_from_path
```

**Usage:**
``` python
# MisuseV18
path = get_module_resource('module_name', 'static', 'img', 'logo.png')

# Proper use V19
path = get_resource_from_path('module_name/static/img/logo.png')
```

---

### 1.4 Domain/Expressions Import
``` python
# MisuseV18 (Deprecated)
from odoo.osv import expression
# or
from odoo.osv.expression import AND, OR

# Proper use V19
from odoo. fields import Domain
# Or for operators
from odoo import fields
```

**Usage:**
``` python
# V19
domain = Domain([('state', '=', 'draft')])
combined_domain = domain & [('partner_id', '!=', False)]
```

---

## 2. Changes in Context and Environment

### 2.1 self._context → self.env.context
``` python
# MisuseV18
class MyModel(models.Model) :
_ name = 'my.model '
    
    def my_method(self) :
        ctx = self._context
        value = self._context.get('key', False)
        
        return self.with_context(new_key='value')

# Proper use V19
class MyModel(models.Model) :
_ name = 'my.model '
    
    def my_method(self) :
        ctx = self.env.context
        value = self.env.context.get('key', False)
        
        return self.with_context(new_key='value')
```

---

### 2.2 self._uid → self.env.uid
``` python
# MisuseV18
class MyModel(models.Model) :
_ name = 'my.model '
    
    def get_current_user(self) :
        user_id = self._uid
        return self.env['res.users'].browse(user_id)

# Proper use V19
class MyModel(models.Model) :
_ name = 'my.model '
    
    def get_current_user(self) :
        user_id = self.env.uid
        return self.env['res.users'].browse(user_id)
```

---

## 3. Changes in ORM Methods

### 3.1 _where_calc and _apply_ir_rules
``` python
# MisuseV18
class MyModel(models.Model) :
_ name = 'my.model '
    
    def custom_search(self, domain) :
        query = self._where_calc(domain)
        self._apply_ir_rules(query, 'read')
        return query

# Proper use V19
class MyModel(models.Model) :
_ name = 'my.model '
    
    def custom_search(self, domain) :
# Use _search with bypass_access
        return self._search(domain, bypass_access=True)
```

**Note:** `_ apply_ir_rules ` was completely removed in V19 .

---

3.2 search () with args parameter
``` python
# MisuseV18
records = self.search(args=[('state', '=', 'draft')])

# Proper use V19
records = self.search(domain=[('state', '=', 'draft')])
# Or simply
records = self.search([('state', '=', 'draft')])
```

**Change:** Parameter `args ` has been renamed to ` domain `.

---

### 3.3 filtered_domain ()
``` python
# V19 - New: Maintains the original order
records = self.env['sale.order'].search ([])
draft_records = records.filtered_domain([('state', '=', 'draft')])
# Now maintains the original records order
```

---

### 3.4 _ write() method
``` python
# V19 - Change: No longer raises an error for non-existent records
class MyModel(models.Model) :
_ name = 'my.model '
    
    def update_record(self, vals) :
# In V19 , _write will not raise an error if the record does not exist
        self._write(vals)
```

---

## 4. Changes in SQL Execution

### 4.1 New Methods
``` python
# V19 - New methods for SQL execution
from odoo import models, api

class MyModel(models.Model) :
_ name = 'my.model '
    
@ api.model
    def execute_query(self) :
# The old method (still works)
        self.env.cr.execute("SELECT * FROM my_table")
        
# New routes in V19
#1. For reading only
        self.env.execute_query("SELECT * FROM my_table")
        
# 2. For writing
        self.env.execute_update("UPDATE my_table SET field = %s", [value])
```

---

## 5. Changes in Model Attributes

### 5.1 Delete sequence attribute
``` python
# MisuseV18
class MyModel(models.Model) :
_ name = 'my.model '
_ sequence = 'my_model_seq '

# Proper use V19
class MyModel(models.Model) :
_ name = 'my.model '
# PostgreSQL uses automatic sequence
```

---

### 5.2 Delete _sql_constraints ( in some cases)
Python
# MisuseV18 (May not work in some cases )
class MyModel(models.Model) :
_ name = 'my.model '
    
_ sql_constraints = [
(' name_uniq', 'unique(name)', 'Name must be unique !')
]

# Proper use V19 - Use Python constraints
from odoo. exceptions import ValidationError

class MyModel(models.Model) :
_ name = 'my.model '
    
@ api.constrains('name')
    def _check_name_unique(self) :
        For record in self :
            existing = self.search ([
(' name', '=', record.name )
(' id', '!=', record.id )
], limit=1 )
            if existing :
                raise ValidationError('Name must be unique!')
```

---

## 6. Changes in Field Attributes

### 6.1 Delete deprecated and column_format
``` python
# MisuseV18
class MyModel(models.Model) :
_ name = 'my.model '
    
    old_field = fields.Char (
        deprecated=True ,
        column_format='text '
)

# Proper use V19
class MyModel(models.Model) :
_ name = 'my.model '
    
    old_field = fields.Char ()
#Delete deprecated and column_format
```

---

### 6.2 index property
``` python
# V19 - Use index property for Fields
class MyModel(models.Model) :
_ name = 'my.model '
    
    name = fields.Char(index=True) # Create index
    code = fields. Char(index='unique') # unique index
```

---

## 7. Changes in Authentication

### 7.1 authenticate() method
``` python
# MisuseV18
from odoo.http import request

def login(creds) :
    uid = request.session.authenticate (
        request.session.db
        creds['login'] ,
        creds['password']
)

# Proper use V19
from odoo.http import request

def login(creds) :
    uid = request.session.authenticate (
        request.env ,
        creds['login'] ,
        creds['password']
)
```

**Change:** The first Parameter changed from ` db` to ` env` .

---

## 8. Changes in Readonly Checks

### 8.1 _web_client_readonly ()
``` python
# MisuseV18
class MyModel(models.Model) :
_ name = 'my.model '
    
    def _web_client_readonly(self) :
        return super()._web_client_readonly ()

# Proper use V19
class MyModel(models.Model) :
_ name = 'my.model '
    
    def _web_client_readonly(self, rule, arg) :
        return super()._web_client_readonly(rule, arg)
```

**Change:** Now accepts additional parameters ` rule` and ` arg` .

---

## 9. Practical examples of promotion

### 9.1 Example: Custom Search Method
Python
# MisuseV18
class SaleOrder(models.Model) :
_ inherit = 'sale.order '
    
    def find_draft_orders(self) :
        domain = [('state', '=', 'draft')]
        query = self._where_calc(domain)
        self._apply_ir_rules(query, 'read')
        
        self.env.cr.execute(query)
        ids = [row[0] for row in self.env.cr.fetchall()]
        return self.browse(ids)

# Proper use V19
class SaleOrder(models.Model) :
_ inherit = 'sale.order '
    
    def find_draft_orders(self) :
        domain = [('state', '=', 'draft')]
# Use search directly
        ids = self._search(domain, bypass_access=False)
        return self.browse(ids)
        
# Or simply:
# return self.search(domain)
```

---

### 9.2 Example: Excel Report Generation
Python
# MisuseV18
from odoo import models
from odoo.tools.misc import xlsxwriter
import io

class ReportExcel(models.AbstractModel) :
_name = 'report.module.report_excel '
    
    def generate_xlsx_report(self, workbook, data, objects) :
        sheet = workbook.add_worksheet ()
# ...

# Proper use V19
from odoo import models
import xlsxwriter # direct import
import io

class ReportExcel(models.AbstractModel) :
_name = 'report.module.report_excel '
    
    def generate_xlsx_report(self, workbook, data, objects) :
        sheet = workbook.add_worksheet ()
# ...
```

---

### 9.3 Example: Getting Module Resources
Python
# MisuseV18
from odoo import models
from odoo.modules.module import get_module_resource

class MyModel(models.Model) :
_ name = 'my.model '
    
    def get_logo_path(self) :
        return get_module_resource (
' my_module ',
' static ',
' img ',
' logo.png '
)

# Proper use V19
from odoo import models
from odoo. modules import get_resource_from_path

class MyModel(models.Model) :
_ name = 'my.model '
    
    def get_logo_path(self) :
        return get_resource_from_path (
' my_module/static/img/logo.png '
)
```

---

### 9.4 Example: Domain Operations
Python
# MisuseV18
from odoo import models
from odoo.osv.expression import AND, OR

class MyModel(models.Model) :
_ name = 'my.model '
    
    def get_combined_domain(self) :
        domain1 = [('state', '=', 'draft')]
        domain2 = [('partner_id', '!=', False)]
        return AND([domain1, domain2])

# Proper use V19
from odoo import models, fields

class MyModel(models.Model) :
_ name = 'my.model '
    
    def get_combined_domain(self) :
        domain1 = fields.Domain([('state', '=', 'draft')])
        domain2 = fields.Domain([('partner_id', '!=', False)])
        return domain1 & domain2
        
# Or simply:
# return [('state', '=', 'draft'), ('partner_id', '!=', False)]
```

---

## 10. Common Mistakes and Solutions

### Error 1: ImportError: cannot import name 'registry '
Python
# Error:
from odoo import registry # Misuse

# the solution:
from odoo.modules.registry import Registry # Proper use
```

---

### Error 2: AttributeError: '_context' attribute
Python
# Error:
ctx = self._context # Misuse

# the solution:
ctx = self. env. context # Proper use
```

---

### Error 3: AttributeError: '_apply_ir_rules' method
Python
# Error:
self._apply_ir_rules(query, 'read') # Misuse

# the solution:
# Use _search with bypass_access
ids = self._search(domain, bypass_access=True) # Proper use
```

---

### Error 4: ImportError: xlsxwriter from odoo.tools.misc
Python
# Error:
from odoo.tools.misc import xlsxwriter # Misuse

# the solution:
import xlsxwriter # Proper use
```

---

### Error 5: ImportError: get_module_resource
Python
# Error:
from odoo.modules.module import get_module_resource # Misuse

# the solution:
from odoo. modules import get_resource_from_path # Proper use
```

---

## 11. Decorators and API Changes

### 11.1 @ api decorators (unchanged)
Python
# V18 & V19 - Same use
from odoo import api, models

class MyModel(models.Model) :
_ name = 'my.model '
    
@ api.model
    def model_method(self) :
        pass
    
@ api.depends('field1', 'field2')
    def _compute_field(self) :
        pass
    
@ api.onchange('field')
    def _onchange_field(self) :
        pass
    
@ api.constrains('field')
    def _check_field(self) :
        pass
```

---

12. Working with Registry

### 12.1 Correct Use
``` python
# Proper use V19
from odoo.modules.registry import Registry

def some_function(db_name) :
# Obtaining Registry
    registry = Registry(db_name)
    
# Using the Registry
    with registry.cursor() as cr :
        env = api.Environment(cr, SUPERUSER_ID, {})
# ... your code
```

---

## 13. Multi-threading and Concurrent Access

### 13.1 Using the Registry in Threads
``` python
# V19
from odoo.modules.registry import Registry
from odoo import api, SUPERUSER_ID
import threading

def background_task(db_name) :
    registry = Registry(db_name)
    
    with registry.cursor() as cr :
        env = api.Environment(cr, SUPERUSER_ID, {})
# your background work
        cr.commit ()

# Launching Thread
thread = threading.Thread (
    target=background_task ,
    args=('database_name',)
)
thread.start ()
```

---

## 14. Checklist for Python Code

### Imports :
- [ ] ` from odoo import registry` → `from odoo.modules.registry import registry`
- [ ] ` from odoo.tools.misc import xlsxwriter` → `import xlsxwriter`
- [ ] ` from odoo.modules.module import get_module_resource` → `from odoo.modules import get_resource_from_path `
- [ ] ` from odoo.osv.expression` → `from odoo.fields import Domain`

Context & Environment :
- [ ] ` self._context` → ` self.env.context`
- [ ] ` self._uid` → ` self.env.uid`

### ORM Methods :
- [ ] ` self._where_calc()` → `self._search(bypass_access=True) `
- [ ] Delete ` self._apply_ir_rules ()`
- [ ] ` search(args=...)` → `search(domain=...) `

### Model Attributes :
- [ ] Delete `_sequence` attribute
] Review ` _sql_constraints`
- [ ] Delete ` deprecated` and ` column_format` from Fields

Methods :
- [ ] ` authenticate(db, ...)` → `authenticate(env, ...) `
- [ ] `_web_client_readonly ()` → `_web_client_readonly(rule, arg) `

---

## 15. Script for troubleshooting

bash
#!/ bin/bash
# Researching Python code problems

echo "=== Import Issues ==="
grep -rn "from odoo import registry" --include="*.py " ./
grep -rn "from odoo.tools.misc import xlsxwriter" --include="*.py " ./
grep -rn "from odoo.modules.module import get_module_resource" --include="*.py "./
grep -rn "from odoo.osv" --include="*.py "./

echo "=== Context Issues ==="
grep -rn "self\._context" --include="*.py " ./
grep -rn "self\._uid" --include="*.py " ./

echo "=== ORM Issues ==="
grep -rn "_where_calc" --include="*.py "./
grep -rn "_apply_ir_rules" --include="*.py " ./
grep -rn "search(args=" --include="*.py " ./

echo "==== Model Attributes ==="
grep -rn "_sequence\s*=" --include="*.py " ./
grep -rn "deprecated\s*=" --include="*.py " ./
grep -rn "column_format\s*=" --include="*.py " ./
```

---
# Changes to HTTP Routes and Controllers - Upgrade to Odoo 19

## 1. Main change: type='json' → type='jsonrpc '

### 1.1 The Problem
Python
# MisuseV18
from odoo import http

class MyController(http.Controller) :
    
@ http.route('/my/endpoint', type='json', auth='user')
    def my_json_endpoint(self, **kw) :
        return {'result': 'success'}
```

**Error in V19 :**
```
Warning: type='json' is deprecated, use type='jsonrpc '
```

---

### 1.2 Solution
``` python
# Proper use V19
from odoo import http

class MyController(http.Controller) :
    
@ http.route('/my/endpoint', type='jsonrpc', auth='user')
    def my_json_endpoint(self, **kw) :
        return {'result': 'success'}
```

**Change:** ` type='json'` → ` type='jsonrpc '`

---

Route Types in V19

### 2.1 Available Types
``` python
from odoo import http

class MyController(http.Controller) :
    
# 1. HTTP Route (for regular pages)
@ http.route('/my/page', type='http', auth='public', website=True)
    def my_page(self, **kw) :
        return http.request.render('module.template', {})
    
#2. JSON-RPC Route (for API calls )
@ http.route('/my/api', type='jsonrpc', auth='user', methods=['POST'])
    def my_api(self, param1, param2) :
        return {'status': 'ok'}
    
# 3. JSON Route (Old - deprecated but still working)
@ http.route('/my/old_api', type='json', auth='user')
    def my_old_api(self) :
# It will work but a warning will appear
        return {}
```

---

## 3. Practical examples of conversion

### 3.1 Example: A Simple API Endpoint
``` python
# MisuseV18
from odoo import http
from odoo.http import request

class ProductController(http.Controller) :
    
@ http.route('/api/products', type='json', auth='user')
    def get_products(self, limit=10) :
        products = request.env['product.product'].search([], limit=limit)
        return [{
' id': p.id ,
' name': p.name ,
' price': p.list_price
[ for p in products ]

# Proper use V19
from odoo import http
from odoo.http import request

class ProductController(http.Controller) :
    
@ http.route('/api/products', type='jsonrpc', auth='user')
    def get_products(self, limit=10) :
        products = request.env['product.product'].search([], limit=limit)
        return [{
' id': p.id ,
' name': p.name ,
' price': p.list_price
[ for p in products ]
```

---

### 3.2 Example: Create/Update via API
``` python
# MisuseV18
class SaleController(http.Controller) :
    
@ http.route('/api/sale/create', type='json', auth='user', methods=['POST'])
    def create_sale_order(self, partner_id, line_ids) :
        order = request.env['sale.order'].create ({
' partner_id': partner_id ,
' order_line': [(0, 0, line) for line in line_ids]
})
        return {'order_id': order.id}

# Proper use V19
class SaleController(http.Controller) :
    
@ http.route('/api/sale/create', type='jsonrpc', auth='user', methods=['POST'])
    def create_sale_order(self, partner_id, line_ids) :
        order = request.env['sale.order'].create ({
' partner_id': partner_id ,
' order_line': [(0, 0, line) for line in line_ids]
})
        return {'order_id': order.id}
```

---

### 3.3 Example: File Upload
``` python
# V18 & V19 - Same method ( type='http ')
from odoo import http
from odoo.http import request
import base64

class FileController(http.Controller) :
    
@ http.route('/upload/file', type='http', auth='user', methods=['POST'], csrf=False)
    def upload_file(self, **post) :
        file = post.get('file')
        if file :
            attachment = request.env['ir.attachment'].create ({
' name': file.filename ,
' datas': base64.b64encode(file.read()) ,
' res_model': 'res.partner ',
' res_id': int(post.get('partner_id'))
})
            return request. make_response (
                json.dumps({'attachment_id': attachment.id}) ,
                headers=[('Content-Type', 'application/json')]
)
```

**Note:** File uploads use ` type='http'` and not ` jsonrpc` .

---

4. Auth Options

### 4.1 Available options (unchanged in V19 )
``` python
from odoo import http

class MyController(http.Controller) :
    
# 1. public - No login required
@ http.route('/public/page', type='http', auth='public')
    def public_page(self) :
        return "Public content "
    
# 2. user - Requires a logged-in user
@ http.route('/user/page', type='http', auth='user')
    def user_page(self) :
        return f"Hello {http.request.env.user.name} "
    
# 3. none - does not load session
@ http.route('/webhook', type='http', auth='none', csrf=False)
    def webhook(self) :
        return "OK "
```

---

## 5. CORS and Headers

### 5.1 Adding CORS Headers
Python
# V18 & V19 - Same method
from odoo import http
import JSON

class APIController(http.Controller) :
    
@ http.route('/api/data', type='jsonrpc', auth='public', cors='*')
    def get_data(self) :
        return {'data': 'value'}
    
# Or add headers manually
@ http.route('/api/custom', type='http', auth='public')
    def custom_endpoint(self) :
        data = {'result': 'ok'}
        return http.request.make_response (
            json.dumps(data) ,
            headers =[
(' Content-Type', 'application/json '),
(' Access-Control-Allow-Origin ', '*'),
(' Access-Control-Allow-Methods', 'GET, POST, OPTIONS '),
]
)
```

---

6. CSRF Protection

### 6.1 Disabling CSRF (for external APIs )
Python
# V18 & V19 - Same method
from odoo import http

class WebhookController(http.Controller) :
    
@ http.route('/webhook/receive', type='http', auth='none', csrf=False, methods=['POST'])
    def receive_webhook(self, **post) :
Webhook handling from external service
        return 'OK '
```

**Warning:** Use ` csrf=False` with caution and only for safe endpoints .

---

7. Methods Restriction

### 7.1 Identifying HTTP Methods
Python
# V18 & V19 - Same method
from odoo import http

class RestrictedController(http.Controller) :
    
# GET only
@ http.route('/api/read', type='jsonrpc', auth='user', methods=['GET'])
    def read_data(self) :
        return {}
    
# POST only
@ http.route('/api/write', type='jsonrpc', auth='user', methods=['POST'])
    def write_data(self, data) :
        return {}
    
# GET and POST
@ http.route('/api/flexible', type='http', auth='user', methods=['GET', 'POST'])
    def flexible(self, **kw) :
        if http.request.httprequest.method == 'GET ':
            return "GET request "
        else :
            return "POST request "
```

---

8. Request and Response

### 8.1 Accessing the Request
``` python
# V18 & V19 - Same method
from odoo import http

class RequestController(http.Controller) :
    
@ http.route('/my/endpoint', type='jsonrpc', auth='user')
    def my_endpoint(self) :
# Accessing the request
        request = http.request
        
# User Information
        user = request.env.user
        
# HTTP request
        http_request = request. httprequest
        
# Parameters
        params = request.params
        
# Environment
        env = request.env
        
        return {'user': user.name}
```

---

8.2 Returning Different Responses
``` python
from odoo import http
import JSON

class ResponseController(http.Controller) :
    
# JSON Response
@ http.route('/api/json', type='jsonrpc', auth='user')
    def json_response(self) :
        return {'status': 'ok'}
    
# HTML Response
@ http.route('/page/html', type='http', auth='public')
    def html_response(self) :
        return http.request.render('module.template', {})
    
# Custom Response
@ http.route('/api/custom', type='http', auth='public')
    def custom_response(self) :
        return http.request.make_response (
            json.dumps({'custom': 'response'}) ,
            headers=[('Content-Type', 'application/json')]
)
    
# Redirect
@ http.route('/redirect', type='http', auth='public')
    def redirect_response(self) :
        return http.request.redirect('/target/page')
    
# Download File
@ http.route('/download/file', type='http', auth='user')
    def download_file(self) :
        content = b"File content "
        return http.request.make_response (
            content
            headers =[
(' Content-Type', 'application/octet-stream '),
(' Content-Disposition', 'attachment; filename="file.txt "')
]
)
```

---

## 9. Error Handling

### 9.1 Error Handling
``` python
# V18 & V19 - Same method
from odoo import http
from odoo. exceptions import UserError, ValidationError
import logging

_logger = logging.getLogger(__name__)

class SafeController(http.Controller) :
    
@ http.route('/api/safe', type='jsonrpc', auth='user')
    def safe_endpoint(self, record_id) :
        try :
            record = http.request.env['my.model'].browse(record_id)
            if not record.exists ():
                return {'error': 'Record not found'}
            
            return {'data': record.name}
            
        except ValidationError as e :
_logger.error (f"Validation error: {e}")
            return {'error': str(e)}
            
        except Exception as e :
_logger.exception ("Unexpected error")
            return {'error': 'Internal server error'}
```

---

10. Website Routes

### 10.1 Routes for Website
``` python
# V18 & V19 - Same method
from odoo import http

class WebsiteController(http.Controller) :
    
@ http.route('/shop/custom', type='http', auth='public', website=True)
    def custom_shop_page(self, **kw) :
# website=True makes the route available in the website context.
        return http.request.render('module.shop_template ', {
' products': http.request.env['product.product'].search ([])
})
    
@ http.route('/shop/product/<int:product_id>', type='http', auth='public', website=True)
    def product_detail(self, product_id, **kw) :
        product = http.request.env['product.product'].browse(product_id)
        return http.request.render('module.product_detail ', {
' product': product
})
```

---

11. Sitemap and SEO

### 11.1 Adding routes to the sitemap
``` python
# V18 & V19 - Same method
from odoo import http

class SitemapController(http.Controller) :
    
@ http.route('/sitemap.xml', type='http', auth='public', website=True, sitemap=True)
    def sitemap(self) :
        return http.request.render('website.sitemap_xml', {})
```

---

## 12. Common Mistakes and Solutions

Error 1: type='json' deprecated warning
``` python
# Error:
@ http.route('/api/endpoint', type='json', auth='user') # Misuse

# the solution:
@ http.route('/api/endpoint', type='jsonrpc', auth='user') # Proper use
```

---

Error 2: CORS errors
``` python
Error: CORS blocked

# the solution:
@ http.route('/api/endpoint', type='jsonrpc', auth='public', cors='*')
def endpoint(self) :
    return {}
```

---

Error 3: CSRF token missing
``` python
Error: CSRF validation failed

# Solution (for external webhooks ):
@ http.route('/webhook', type='http', auth='none', csrf=False)
def webhook(self) :
    return 'OK '
```

---

## 13. Testing Controllers

### 13.1 Unit Tests for Controllers
``` python
# V18 & V19 - Same method
from odoo.tests import HttpCase, tagged

@tagged ('post_install', '-at_install')
class TestMyController(HttpCase) :
    
    def test_json_endpoint(self) :
JSON-RPC endpoint test
        response = self.url_open (
'/ api/products ',
            data=json.dumps ({
' jsonrpc': '2.0 ',
' method': 'call ',
' params': {'limit': 5}
}),
            headers={'Content-Type': 'application/json'}
)
        
        result = response.json ()
        self.assertEqual(result.get('result'), expected_result)
    
    def test_http_endpoint(self) :
HTTP endpoint test
        response = self.url_open('/public/page')
        self.assertEqual(response.status_code, 200)
```

---

## 14. Checklist for Controllers

### Before the promotion:
- [ ] Search for all ` type='json'` in Controllers
- [ ] Review CORS settings
- [ ] Review CSRF settings

### During the promotion:
- [ ] Replace ` type='json'` with ` type=' jsonrpc'`
- [ ] Testing all API endpoints
- [ ] Checking error handling

### After the promotion:
- [ ] Testing all routes
- [ ] Check for warnings
CORS and CSRF testing
- [ ] Review logs for errors

---

## 15. Script for troubleshooting

bash
#!/ bin/bash
Controllers Problems

echo "=== Search for type='json ' ==="
grep -rn "type='json'" --include="*.py" ./controllers/
grep -rn 'type="json"' --include="*.py" ./controllers/

echo "=== Searching for Controllers without methods ==="
grep -rn "@http.route" --include="*.py" ./controllers/ | grep -v "methods ="

echo "=== Search for csrf=False ==="
grep -rn "csrf=False" --include="*.py" ./controllers/

echo "=== Number of Routes in Module ==="
grep -c "@http.route"./controllers/*.py
```

---

16. Best Practices

### 16.1 General Tips
1. Proper use: Use ` type='jsonrpc'` for APIs
2. Proper use: Use ` type='http'` for pages and file uploads.
3. Proper use: Explicitly define ` methods` .
4. Proper use: Only use ` csrf=False` when necessary.
5. Proper use: Add appropriate error handling .
6. Proper use: Use logging for debugging.
7. Proper use: Test CORS for external APIs
8. Proper use and documentation of API endpoints

### 16.2 Security
1. Only use ` auth='none'` when absolutely necessary.
2. Do not use ` csrf=False` for user-facing endpoints
3. Check the permissions within the method.
4. Validate all inputs
5. Do not display sensitive information in errors

---
Common Mistakes and Solutions - Upgrading to Odoo 19

## 1. Models and Fields Errors

### Error 1.1: Field 'category_id' does not exist in 'res.groups '
```
Error: Field 'category_id' does not exist
Model: res.groups
```

**Reason:** ` category_id` was replaced by ` privilege_id` in res.groups .

**the solution:**
xml
<!-- Misuse -->
< field name="category_id" ref="base.module_category_sales "/>

<!-- Proper use -->
< record model="res.groups.privilege" id="privilege_sales ">
< field name="name">Sales</field >
< field name="category_id" ref="base.module_category_sales "/>
</record>

< field name="privilege_id" ref="privilege_sales "/>
```

---

### Error 1.2: Field 'mobile' does not exist in 'res.partner '
```
Error: Field 'mobile' does not exist
Model: res.partner
```

**Reason:** The ` mobile` field was deleted from res.partner .

**the solution:**
``` python
# Misuse
partner.mobile = '0123456789 '

# Proper use: Use your phone instead.
partner.phone = '0123456789 '

# Or create a custom field
class ResPartner(models.Model) :
_ inherit = 'res.partner '
    
    x_mobile = fields.Char(string='Mobile')
```

---

### Error 1.3: Field 'product_uom' does not exist
```
Error: Field 'product_uom' does not exist
Model: sale.order.line
```

**Reason:** ` product_uom` has been changed to ` product_uom_id` .

**the solution:**
``` python
# Misuse
line.product_uom = uom_id

# Proper use
line.product_uom_id = uom_id
```

xml
<!-- Misuse -->
< field name="product_uom "/>

<!-- Proper use -->
< field name="product_uom_id "/>
```

---

### Error 1.4: Field 'tax_id' does not exist
```
Error: Field 'tax_id' does not exist
Model: sale.order.line
```

**Reason:** ` tax_id` (Many2one) has been changed to ` tax_ids` (Many2many) .

**the solution:**
``` python
# Misuse
line.tax_id = tax

# Proper use
line.tax_ids = [(6, 0, [tax.id])]
# or
line.tax_ids = [(4, tax.id)]
```

---

### Error 1.5: Model 'uom.category' does not exist
```
Error: The model uom. category does not exist
```

**Reason:** Model `uom.category ` has been completely deleted.

**the solution:**
xml
<!-- Misuse Delete this -->
< record id="custom_category" model="uom.category ">
< field name="name">Custom Category</field >
</record>

<!-- Proper use: Use relative_uom_id directly -->
< record id="custom_uom" model="uom.uom ">
< field name="name">Custom UoM</field >
< field name="relative_uom_id" ref="uom.product_uom_unit "/>
< field name="relative_factor">12</field >
</record>
```

---

### Error 1.6: Field 'factor_inv' does not exist
```
Error: Field 'factor_inv' does not exist
Model: uom.uom
```

**Reason:** ` factor_inv` has been replaced by ` relative_factor` .

**the solution:**
xml
<!-- Misuse -->
< field name="factor_inv">1000</field >

<!-- Proper use -->
< field name="relative_factor">1000</field >
```

---

### Error 1.7: Model 'res.partner.title' does not exist
```
Error: The model res.partner.title does not exist
```

**Reason:** Model deleted.

**the solution:**
``` python
Create a Selection field instead .
class ResPartner(models.Model) :
_ inherit = 'res.partner '
    
    custom_title = fields.Selection ([
(' mr', 'Mr .'),
(' mrs', 'Mrs .'),
(' ms', 'Ms .'),
], string='Title ')
```

---

## 2. Python Code Errors

### Error 2.1: ImportError: cannot import name 'registry '
```
ImportError: cannot import name 'registry' from 'odoo '
```

**Reason:** Registry import path changed.

**the solution:**
``` python
# Misuse
from odoo import registry

# Proper use
from odoo.modules.registry import Registry
```

---

### Error 2.2: AttributeError: '_context' attribute
```
AttributeError: 'sale.order' object has no attribute '_context '
```

**Reason:** `_ context ` has been replaced by ` env.context `.

**the solution:**
``` python
# Misuse
ctx = self._context

# Proper use
ctx = self.env.context
```

---

### Error 2.3: AttributeError: '_uid' attribute
```
AttributeError: object has no attribute '_uid '
```

**Reason:** `_uid` has been replaced by ` env.uid` .

**the solution:**
``` python
# Misuse
user_id = self._uid

# Proper use
user_id = self.env.uid
```

---

### Error 2.4: AttributeError: '_apply_ir_rules' method
```
AttributeError: '_apply_ir_rules' does not exist
```

**Reason:** Method deleted.

**the solution:**
``` python
# Misuse
query = self._where_calc(domain)
self._apply_ir_rules(query)

# Proper use
ids = self._search(domain, bypass_access=True)
```

---

### Error 2.5: ImportError: xlsxwriter from odoo.tools.misc
```
ImportError: cannot import name 'xlsxwriter '
```

**Reason:** Import path changed.

**the solution:**
``` python
# Misuse
from odoo.tools.misc import xlsxwriter

# Proper use
import xlsxwriter
```

---

### Error 2.6: ImportError: get_module_resource
```
ImportError: cannot import name 'get_module_resource '
```

**Reason:** Function moved.

**the solution:**
``` python
# Misuse
from odoo.modules.module import get_module_resource

# Proper use
from odoo. modules import get_resource_from_path
```

---

### Error 2.7: TypeError: authenticate() argument
```
TypeError: authenticate() takes 3 positional arguments but 4 were given
```

**Reason:** authenticate() signature changed.

**the solution:**
``` python
# Misuse
request.session.authenticate(request.session.db, login, password)

# Proper use
request.session.authenticate(request.env, login, password)
```

---

3. XML and Views Errors

Error 3.1: Element 'kanban-box' not recognized
```
Warning: Template 'kanban-box' is deprecated
```

**Reason:** You should use `< card >` instead of `< div class="oe_kanban_card ">`.

**the solution:**
xml
<!-- Misuse -->
< t t-name="kanban-box ">
< div class="oe_kanban_card ">
< field name="name "/>
</ div >
</t>

<!-- Proper use -->
< t t-name="kanban-box ">
<card>
< field name="name "/>
</card>
</t>
```

---

### Error 3.2: Invalid attribute 'expand' in group
```
Warning: Attribute 'expand' is not supported
```

**Reason:** ` expand` and ` string` were deleted from the group tag in search views .

**the solution:**
xml
<!-- Misuse -->
< group expand="0" string="Group By ">
< filter name="group_customer "/>
</group>

<!-- Proper use -->
<group>
< filter name="group_customer "/>
</group>
```

---

### Error 3.3: Field reference error in inherited view
```
Error: Field 'product_uom' does not exist in xpath expression
```

**Reason:** Field name changed in parent view .

**the solution:**
xml
<!-- Misuse -->
< xpath expr="//field[@name='product_uom']" position="after ">

<!-- Proper use -->
< xpath expr="//field[@name='product_uom_id']" position="after ">
```

---

### Error 3.4: No update flag preventing updates
```
Warning: View not updated due to noupdate flag
```

**Reason:** Data file has ` noupdate="1 "`.

**the solution:**
xml
<!-- Misuse -->
< data noupdate="1 ">
< record id="view_form" model="ir.ui.view ">
<!-- ... -->
</record>
</ data >

<!-- Proper use for upgrade, temporarily uninstall noupdate -->
< data noupdate="0 ">
< record id="view_form" model="ir.ui.view ">
<!-- ... -->
</record>
</ data >
```

**or:**
sql
-- directly in the database
UPDATE ir_model_data 
SET noupdate = false 
WHERE name = 'view_form' AND module = 'your_module ';
```

---

## 4. Security and Groups Errors

### Error 4.1: Field 'users' does not exist in res.groups
```
Error: Field 'users' does not exist
Model: res.groups
```

**Reason:** ` users` has been changed to ` user_ids` .

**the solution:**
``` python
# Misuse
group.users = [(4, user.id)]

# Proper use
group.user_ids = [(4, user.id)]
```

---

### Error 4.2: Field 'groups_id' does not exist in res.users
```
Error: Field 'groups_id' does not exist
Model: res.users
```

**Reason:** ` groups_id` has been changed to ` group_ids` .

**the solution:**
``` python
# Misuse
user.groups_id = [(4, group.id)]

# Proper use
user.group_ids = [(4, group.id)]
```

---

### Error 4.3: Cannot use sudo with groups_id in server action
```
Error: Cannot combine sudo() with groups_id in ir.actions.server
```

**Reason:** A new restriction in V19 .

**the solution:**
xml
<!-- Misuse -->
< record id="action_server" model="ir.actions.server ">
< field name="code ">
        record.sudo().action_confirm ()
</ field >
< field name="groups_id" eval="[(4, ref('group_manager')] "/>
</record>

<!-- Proper use: Delete sudo () or delete groups_id -->
< record id="action_server" model="ir.actions.server ">
< field name="code ">
        record.action_confirm ()
</ field >
< field name="groups_id" eval="[(4, ref('group_manager')] "/>
</record>
```

---

5. HTTP Route Errors

Error 5.1: type='json' is deprecated
```
Warning: type='json' is deprecated, use type='jsonrpc '
```

**Reason:** ` type='json'` is outdated.

**the solution:**
``` python
# Misuse
@ http.route('/api/endpoint', type='json', auth='user')

# Proper use
@ http.route('/api/endpoint', type='jsonrpc', auth='user')
```

---

### Error 5.2: CORS policy blocking request
```
Error: Access to XMLHttpRequest has been blocked by CORS policy
```

**Reason:** CORS is not activated.

**the solution:**
``` python
# Proper use
@ http.route('/api/endpoint', type='jsonrpc', auth='public', cors='*')
def endpoint(self) :
    return {}
```

---

Error 5.3: CSRF verification failed
```
Error: CSRF token missing or incorrect
```

**Reason:** CSRF protection is enabled.

**Solution (for webhooks only):**
``` python
# Proper use
@ http.route('/webhook', type='http', auth='none', csrf=False)
def webhook(self) :
    return 'OK '
```

---

## 6. Manifest Errors

Error 6.1: Module version not updated
```
Warning: Module version is still 16.0.1.0.0
```

**Reason:** The version in `__ manifest__.py ` has not been updated.

**the solution:**
``` python
# Misuse
' version': '16.0.1.0.0 '

# Proper use
' version': '19.0.1.0.0 '
```

---

Error 6.2: Missing dependencies
```
Error: Module 'xyz' depends on non-existing module 'abc '
```

**Reason:** Dependency has not been upgraded or deleted.

**the solution:**
``` python
# in __ manifest__.py
' depends ': [
' base ',
' sale ',
# Misuse Delete non-existent modules
# ' old_module ',
]
```

---

## 7. Database Migration Errors

### Error 7.1: Column does not exist
```
psycopg2.errors.UndefinedColumn: column "category_id" does not exist
```

**Reason:** Migration was not implemented correctly.

**the solution:**
``` python
Create a migration script in migrations/19.0.1.0 /
def migrate(cr, version) :
# Add a new column
    if not column_exists(cr, 'uom_uom', 'relative_uom_id') :
        cr.execute ("")
            ALTER TABLE uom_uom 
            ADD COLUMN relative_uom_id INTEGER
""")
    
# Transferring data from old to new
    cr.execute ("")
        UPDATE uom_uom 
        SET relative_uom_id = ref_uom.id
        FROM (
            SELECT id, category_id 
            FROM uom_uom 
            WHERE uom_type = 'reference '
ref_uom
        WHERE uom_uom.category_id = ref_uom.category_id
        AND uom_uom.uom_type != 'reference '
""")

def column_exists(cr, table, column) :
    cr.execute ("")
        SELECT 1 FROM information_schema.columns 
        WHERE table_name=%s AND column_name=%s
""", (table, column) )
    return bool(cr.fetchone())
```

---

Error 7.2: Foreign key constraint violation
```
psycopg2.errors.ForeignKeyViolation :
insert or update on table violates foreign key constraint
```

**Reason:** Incorrect data migration .

**the solution:**
``` python
def migrate(cr, version) :
# Delete the old FK constraints first
    cr.execute ("")
        ALTER TABLE res_groups 
        DROP CONSTRAINT IF EXISTS res_groups_category_id_fkey
""")
    
# Implement migration
# ...
    
# Add new FK constraints
    cr.execute ("")
        ALTER TABLE res_groups 
        ADD CONSTRAINT res_groups_privilege_id_fkey
        FOREIGN KEY (privilege_id) REFERENCES res_groups_privilege(id)
""")
```

---

8. Performance Errors

Error 8.1: Slow query after migration
```
Warning: Query takes too long to execute
```

**Reason:** Missing indexes on new fields .

**the solution:**
``` python
# in Model
class MyModel(models.Model) :
_ name = 'my.model '
    
# Add index
    name = fields.Char(index=True)
    partner_id = fields.Many2one('res.partner', index=True)
```

**Or in migration :**
``` python
def migrate(cr, version) :
# Add index
    cr.execute ("")
        CREATE INDEX IF NOT EXISTS idx_my_model_partner_id
        ON my_model(partner_id)
""")
```

---

## 9. Testing Errors

### Error 9.1: Tests failing after migration
```
AssertionError: Expected... but got ...
```

**Reason:** Tests uses outdated field names.

**the solution:**
``` python
# Misuse
def test_create_order(self) :
    order = self.env['sale.order'].create ({
' partner_id': self.partner.id ,
})
    line = order.order_line [0]
    self.assertEqual(line.product_uom, self.uom)

# Proper use
def test_create_order(self) :
    order = self.env['sale.order'].create ({
' partner_id': self.partner.id ,
})
    line = order.order_line [0]
    self.assertEqual(line.product_uom_id, self.uom)
```

---

## 10. Debugging Tips

### 10.1 Enable Debug Mode
bash
# in command line
./ odoo-bin -d database -u module_name --log-level=debug

# in config file
[ options ]
log_level = debug
log_handler = :DEBUG
```

### 10.2 Check Module Status
``` python
# in Odoo shell
./ odoo-bin shell -d database

>>> env = Environment(cr, SUPERUSER_ID, {})
>>> module = env['ir.module.module'].search([('name', '=', 'your_module')])
>>> print(module.state)
>>> print(module.latest_version)
```

### 10.3 SQL Debugging
``` python
# in Python code
import logging
_logger = logging.getLogger(__name__)

# Log SQL queries
self.env.cr.execute("SELECT * FROM my_table")
_ logger.info(f"Query: {self.env.cr.query}")
_logger.info (f"Results: {self.env.cr.fetchall () }")
```

---

## 11. Checklist for Troubleshooting

### When an error occurs:
- [ ] Read the entire error ( stack trace )
- [ ] Specify the relevant Model or Field
- [ ] Search this file for a similar error
- [ ] Review the appropriate Migration files
- [ ] Test the solution in a development environment first
- [ ] Review logs after application

For persistent errors:
- [ ] Delete __pycache __
- [ ] Restart Odoo
- [ ] Update Module with -u flag
- [ ] Refer to database schema
- [ ] Check migration scripts

---

## 12. Additional Resources

### Get help:
1. Odoo Documentation (v19)
2. Odoo Community Forum
3. GitHub Issues
4. This guide

Useful tools :
- pgAdmin (for checking the database )
PyCharm Debugger
Odoo Debug Mode
Browser DevTools

---# Step-by-step upgrade plan - from Odoo 16/17/18 to 19

## Phase 1: Preparation ( Pre-Migration )

### 1.1 Backup
bash
# Proper use Backup Database
pg_dump -U odoo -d database_name -F c -f backup_before_v19_$(date +%Y%m%d).backup

# Proper use Backup Filestore
tar -czf filestore_backup_$(date +%Y%m%d).tar.gz ~/.local/share/Odoo/filestore/database_name

# Proper use Backup Custom Addons
tar -czf custom_addons_backup_$(date +%Y%m%d).tar.gz /path/to/custom_addons
```

**Very important:** Do not continue without a full backup !

---

### 1.2 Creating a Test Environment
bash
# Create a database test
createdb -U odoo -T database_name database_name_test

# Copy filestore
cp -r ~/.local/share/Odoo/filestore/database_name \
~/. local/share/Odoo/filestore/database_name_test
```

---

### 1.3 Inventory of Current Modules
``` python
# Contact the database
./ odoo-bin shell -d database_name

Get a list of installed modules
>>> modules = env['ir.module.module'].search ([
... (' state', 'in', ['installed', 'to upgrade'] )
... ])
>>> for m in modules :
... print(f"{m.name}: {m.latest_version}")
```

**Save the list to a file:**
bash
# in Odoo shell
>>> with open('installed_modules.txt', 'w') as f :
... for m in modules :
... f.write(f"{m.name},{m.latest_version},{m.state}\n")
```

---

### 1.4 Checking Custom Modules
bash
# Custom Modules List
ls -la /path/to/custom_addons /

For each module , check:
# 1. __ manifest__.py
# 2. depends
# 3. version
```

** Checklist for each Module :**
Does it rely on external modules ?
- [ ] Does it have modifications ( inherits ) to standard modules ?
- [ ] Does it use features that might be deprecated ?

---

### 1.5 Document Review
- [ ] Reading Odoo 19 Release Notes
- [ ] Breaking Changes Review
- [ ] Review the Migration files in this guide

---

## Phase 2: Preparing the new environment

### 2.1 Installing Odoo 19
bash
# Clone Odoo 19
git clone https://github.com/odoo/odoo.git -b 19.0 --depth 1 odoo19

# Creating a Virtual Environment
cd odoo19
python3 -m venv venv
source venv/bin/activate # Linux/Mac
# venv\Scripts\activate # Windows

# Fix Dependencies
pip install -r requirements.txt
```

---

### 2.2 Config File Setup
``` ini
# odoo19.conf
[ options ]
addons_path = /path/to/odoo19/addons, /path/to/custom_addons
admin_passwd = admin_password
db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoo_password
logfile = /var/log/odoo/odoo19.log
log_level = debug
```

---

### 2.3 Operational Test
bash
# Running Odoo 19 (without database )
./ odoo-bin -c odoo19.conf --stop-after-init

# It must work without errors
```

---

## Phase 3: Updating Custom Modules

### 3.1 Update __ manifest__.py
``` python
# In each custom module
{
' name': 'Module Name ',
' version': '19.0.1.0.0 ', # ← Change this
' category': 'Category ',
' depends': ['base', 'sale'] , # ← See dependencies
' data ': [
' security/ir.model.access.csv ',
' views/views.xml ',
],
' installable': True ,
' application': False ,
}
```

** Checklist :**
- [ ] Update version to 19.0.xxx
- [ ] Review depends
- [ ] Ensure all files are present

---

### 3.2 Update Models (Python Files)
bash
# Use the search and replace script
cd /path/to/custom_addons/your_module

# 1. Registry import
find . -name "*.py" -exec sed -i 's/from odoo import registry/from odoo.modules.registry import Registry/g ' {} +

# 2. Context
find . -name "*.py" -exec sed -i 's/self\._context/self.env.context/g ' {} +

# 3. UID
find . -name "*.py" -exec sed -i 's/self\._uid/self.env.uid/g ' {} +

#4. xlsxwriter
find . -name "*.py" -exec sed -i 's/from odoo.tools.misc import xlsxwriter/import xlsxwriter/g ' {} +
```

**Then check manually:**
- [ ] changes in field names
Methods deleted
- [ ] ORM changes

See : ` Migration_V19_Models_Fields.md` and ` Migration_V19_Python_Code.md`

---

### 3.3 Update Views (XML Files)
bash
# in module directory
cd /path/to/custom_addons/your_module

# Search for XML problems
grep -rn "type='json '" ./
grep -rn "category_id.*ref=" ./security/
grep -rn "factor_inv"./data /
grep -rn 'expand="0"' ./views/
```

**Manual updates required:**
- [ ] Kanban views: `<kanban-box>` → `<card >`
- [ ] Search views : Delete ` expand ` and ` string ` from `< group >`
- [ ] UoM data : Structure update
- [ ] Groups : Updating privilege system

See : ` Migration_V19_Views_XML.md`

---

Security Files Update

#### security.xml
xml
<!-- Before -->
< record id="group_user" model="res.groups ">
< field name="name">User</field >
< field name="category_id" ref="module_category_custom "/>
</record>

<!-- After -->
< record id="privilege_custom" model="res.groups.privilege ">
< field name="name">Custom</field >
< field name="category_id" ref="module_category_custom "/>
</record>

< record id="group_user" model="res.groups ">
< field name="name">User</field >
< field name="privilege_id" ref="privilege_custom "/>
< field name="sequence">10</field >
</record>
```

#### ir.model.access.csv
``` csv
# Check that group_id refers to existing groups
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_model_user,model.user,model_custom_model,group_user,1,1,1,0
```

: ` Migration_V19_Security_Groups.md`

---

### 3.5 Controllers Update
``` python
# in controllers /
# Search for type='json '
grep -rn "type='json'"./controllers /

# Replace
# Misusetype='json '
# Proper use type='jsonrpc '
```

: ` Migration_V19_Routes_Controllers.md`

---

### 3.6 Creating Migration Scripts

If the module requires data migration :

bash
Create a migrations folder
mkdir -p migrations/19.0.1.0

Create a script
touch migrations/19.0.1.0/pre-migrate.py
touch migrations/19.0.1.0/post-migrate.py
```

**Example : pre-migrate.py **
``` python
def migrate(cr, version) :
"""
    Migration script for V19
It is executed before the module is loaded.
"""
# Adding new columns
    cr.execute ("")
        ALTER TABLE custom_model 
        ADD COLUMN IF NOT EXISTS new_field VARCHAR
""")
    
# Data Transfer
    cr.execute ("")
        UPDATE sale_order_line 
        SET tax_ids = ARRAY[tax_id] 
        WHERE tax_id IS NOT NULL
""")
```

** Example post-migrate.py:**
``` python
def migrate(cr, version) :
"""
It is executed after the module is loaded.
"""
    from odoo import api, SUPERUSER_ID
    
    env = api.Environment(cr, SUPERUSER_ID, {})
    
# Updating Records
    records = env['custom.model'].search ([])
    For record in records :
        record.compute_something ()
```

---

## Phase 4: Initial Test

### 4.1 Running Odoo with Database Test
bash
# Run with database test
./ odoo-bin -c odoo19.conf -d database_name_test --stop-after-init

# Check the logs
tail -f /var/log/odoo/odoo19.log
```

**Search for:**
MisuseErrors
Warnings
Deprecation notices

---

### 4.2 Module Update
bash
# Update custom module specific
./ odoo-bin -c odoo19.conf -d database_name_test -u module_name --stop-after-init

# Or update all modules
./ odoo-bin -c odoo19.conf -d database_name_test -u all --stop-after-init
```

---

### 4.3 Error Review

**If errors occur:**
1. Check stack trace
2. Search in ` Migration_V19_Common_Errors.md`
3. Correct the error in the module
4. Try again

**Common mistakes:**
- Old field names
- Model deleted
Incorrect import paths
- Outdated XML structure

---

### 4.4 Functional Test
bash
# Running Odoo (without stop-after-init )
./ odoo-bin -c odoo19.conf -d database_name_test
```

**Open your browser and test it:**
- [ ] Login
- [ ] Open Main Menus
Open Forms
- [ ] Search and Filter
- [ ] Creating new records
- [ ] Update records
- [ ] Reports
Custom features

---

## Stage 5: Advanced Tests

### 5.1 Unit Tests
bash
# Run tests for a specific module
./ odoo-bin -c odoo19.conf -d database_name_test -u module_name --test-enable --stop-after-init

# Run tests for each Module
./ odoo-bin -c odoo19.conf -d database_name_test -u all --test-enable --stop-after-init
```

---

### 5.2 Performance Testing
``` python
# In Python code , add logging
import time
import logging
_logger = logging.getLogger(__name__)

def slow_method(self) :
    start = time.time ()
# ... code
    duration = time. time() - start
_ logger.info(f"Method took {duration:.2f} seconds")
```

**watch:**
- Slow queries
Memory usage
CPU usage

---

### 5.3 Data Integrity
sql
-- Check data integrity
-- 1. Orphaned records
SELECT * FROM sale_order WHERE partner_id NOT IN (SELECT id FROM res_partner) ;

2. Unexpected null values
SELECT * FROM product_product WHERE uom_id IS NULL ;

3. Invalid foreign keys
SELECT * FROM res_groups WHERE privilege_id IS NOT NULL 
  AND privilege_id NOT IN (SELECT id FROM res_groups_privilege) ;
```

---

## Phase 6: Optimization

### 6.1 Adding Indexes
``` python
# in models
class MyModel(models.Model) :
_ name = 'my.model '
    
    partner_id = fields.Many2one('res.partner', index=True)
    date = fields.Date(index=True)
    state = fields.Selection([...], index=True)
```

---

### 6.2 SQL Constraints Review
``` python
class MyModel(models.Model) :
_ name = 'my.model '
    
_ sql_constraints = [
(' code_uniq', 'unique(code)', 'Code must be unique !')
]
```

---

### 6.3 Cleanup
bash
# Delete old data
./ odoo-bin -c odoo19.conf -d database_name_test shell

>>> # Delete old logs
>>> env['ir.logging'].search([]).unlink ()
>>> # Delete old mail messages
>>> old_messages = env['mail.message'].search ([
...(' date', '<', '2020-01-01 ')
... ])
>>> old_messages.unlink ()
>>> cr.commit ()
```

---

## Phase 7: Documentation

7.1 Documenting Changes
markdown
# CHANGELOG.md

## Version 19.0.1.0.0 (2025-10-30)

### Changed
Updated to Odoo 19
- Changed `product_uom` to `product_uom_id` in all forms
- Updated security groups to use privilege system

Added
- New field `x_custom_field` in sale.order
- Migration scripts for data conversion

### Removed
- Removed dependency on deprecated_module

Fixed
- Fixed issue with UoM conversion
- Fixed group visibility in menus
```

---
