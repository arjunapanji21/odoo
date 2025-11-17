# Odoo Database Documentation

## Overview

This document provides comprehensive documentation of the Odoo database architecture, models, and data structures across all modules in the workspace.

**Version:** Odoo 19.0  
**Total Modules:** 598  
**Total Model Files:** 3,101

---

## Table of Contents

1. [Database Architecture](#database-architecture)
2. [Core Database Concepts](#core-database-concepts)
3. [ORM Framework](#orm-framework)
4. [Module-by-Module Database Documentation](#module-by-module-database-documentation)
5. [Database Schema Patterns](#database-schema-patterns)
6. [Data Relationships](#data-relationships)

---

## Database Architecture

### PostgreSQL Backend

Odoo uses PostgreSQL as its primary database management system:
- **Minimum Version:** PostgreSQL 13+
- **Connection Management:** Managed through `odoo/sql_db.py`
- **Connection Pooling:** Automatic connection pooling for performance
- **Multi-tenancy:** Support for multiple databases on single installation

### Key Database Files

```
odoo/
├── sql_db.py              # Database connection management
├── orm/
│   ├── models.py          # Base model classes
│   ├── fields.py          # Field type definitions
│   ├── registry.py        # Model registry management
│   └── environments.py    # Environment and cursor management
└── models/
    └── __init__.py        # Model exports
```

---

## Core Database Concepts

### 1. Models

Odoo uses an Object-Relational Mapping (ORM) system where Python classes represent database tables:

**Base Model Types:**
- **Model:** Persistent models stored in database (`odoo.models.Model`)
- **TransientModel:** Temporary models for wizards (`odoo.models.TransientModel`)
- **AbstractModel:** Inherited models without database tables (`odoo.models.AbstractModel`)

### 2. Field Types

**Basic Fields:**
- `Char` - Character strings (VARCHAR)
- `Text` - Long text content (TEXT)
- `Html` - HTML content with sanitization
- `Integer` - Integer numbers
- `Float` - Floating point numbers
- `Monetary` - Currency amounts with precision
- `Boolean` - True/False values
- `Date` - Date without time
- `Datetime` - Date with time

**Relational Fields:**
- `Many2one` - Foreign key relationship
- `One2many` - Reverse of Many2one
- `Many2many` - Many-to-many relationship
- `Reference` - Polymorphic foreign key

**Special Fields:**
- `Binary` - Binary data/files
- `Selection` - Fixed list of values
- `Properties` - Dynamic field definitions
- `Json` - JSON data storage

### 3. Magic Columns

Every model automatically includes these system fields:
- `id` - Primary key (serial)
- `create_date` - Record creation timestamp
- `create_uid` - User who created record
- `write_date` - Last modification timestamp
- `write_uid` - User who last modified record

### 4. Computed Fields & Stored Fields

- **Computed Fields:** Calculated on-the-fly using Python methods
- **Stored Fields:** Computed but persisted in database for performance
- **Related Fields:** Shortcuts to access related model fields

---

## ORM Framework

### Model Definition Example

```python
from odoo import models, fields, api

class Partner(models.Model):
    _name = 'res.partner'
    _description = 'Contact'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    
    name = fields.Char(string='Name', required=True, index=True)
    email = fields.Char(string='Email')
    phone = fields.Char(string='Phone')
    company_type = fields.Selection([
        ('person', 'Individual'),
        ('company', 'Company')
    ], default='person')
    
    invoice_ids = fields.One2many('account.move', 'partner_id', 
                                   string='Invoices')
    
    @api.depends('name', 'email')
    def _compute_display_name(self):
        for record in self:
            if record.email:
                record.display_name = f"{record.name} <{record.email}>"
            else:
                record.display_name = record.name
```

### Key ORM Features

1. **Model Inheritance:**
   - `_inherit` - Extend existing model
   - `_inherits` - Delegate inheritance (composition)

2. **Constraints:**
   - SQL Constraints - Database-level constraints
   - Python Constraints - Application-level validation

3. **Recordsets:**
   - All model operations return recordsets
   - Support iteration, slicing, filtering
   - Batch operations for performance

4. **CRUD Operations:**
   - `create()` - Create new records
   - `read()` - Read record data
   - `write()` - Update records
   - `unlink()` - Delete records

5. **Search Domains:**
   - Domain DSL for filtering: `[('field', 'operator', 'value')]`
   - Operators: `=`, `!=`, `>`, `<`, `in`, `like`, `ilike`, etc.

---

## Module-by-Module Database Documentation

### Core Modules

#### 1. **Base Module** (odoo/addons)
Built-in models in the Odoo core:

**Key Models:**
- `res.partner` - Contacts and companies
- `res.users` - System users
- `res.company` - Multi-company support
- `res.groups` - Access control groups
- `res.currency` - Currency definitions
- `res.country` - Countries and states
- `ir.model` - Model metadata
- `ir.model.fields` - Field metadata
- `ir.attachment` - File attachments
- `ir.sequence` - Auto-incrementing sequences
- `ir.cron` - Scheduled actions
- `ir.ui.view` - View definitions
- `ir.ui.menu` - Menu structure
- `ir.actions.*` - Action definitions

---

### Accounting Modules (24 modules)

#### **account** - Invoicing & Accounting
Main accounting functionality with 25+ models.

**Core Models:**
- `account.move` - Journal entries (invoices, bills, payments)
- `account.move.line` - Journal entry lines
- `account.account` - Chart of accounts
- `account.journal` - Accounting journals
- `account.tax` - Tax definitions
- `account.payment` - Payment records
- `account.payment.term` - Payment terms
- `account.bank.statement` - Bank statements
- `account.reconcile.model` - Reconciliation rules
- `account.fiscal.position` - Fiscal positions

**Key Features:**
- Multi-currency support
- Automated reconciliation
- Tax calculation engine
- Financial reporting
- Analytic accounting integration

**Database Tables:**
```sql
-- Main tables
account_move                    -- Invoices, bills, payments
account_move_line               -- Journal entry lines
account_account                 -- Chart of accounts
account_journal                 -- Journals (sales, purchase, bank)
account_tax                     -- Tax rates and rules
account_payment                 -- Payment tracking
account_bank_statement          -- Bank reconciliation
account_reconcile_model         -- Auto-reconciliation rules
```

#### **account_payment** - Payment Processing
Payment management and provider integration.

**Models:**
- `account.payment.method` - Payment methods
- `account.payment.method.line` - Payment method lines
- `payment.transaction` - Payment transactions
- `payment.token` - Saved payment methods

#### **analytic** - Analytic Accounting
Cost and revenue tracking across dimensions.

**Models:**
- `account.analytic.account` - Analytic accounts
- `account.analytic.line` - Analytic entries
- `account.analytic.plan` - Analytic plans
- `account.analytic.distribution` - Cost distribution

---

### Sales Modules (35+ modules)

#### **sale** - Sales Management
Complete sales order management system.

**Core Models:**
- `sale.order` - Sales orders
- `sale.order.line` - Order lines
- `sale.order.template` - Quotation templates
- `crm.team` - Sales teams

**Database Structure:**
```sql
sale_order                      -- Sales orders
sale_order_line                 -- Order line items
sale_order_template             -- Quote templates
sale_order_template_line        -- Template lines
sale_order_template_option      -- Optional products
```

**Features:**
- Quotation/order workflow
- Product configurator
- Pricing rules
- Delivery management
- Invoice generation

#### **crm** - Customer Relationship Management
Lead and opportunity tracking.

**Models:**
- `crm.lead` - Leads and opportunities
- `crm.stage` - Pipeline stages
- `crm.team` - Sales teams
- `crm.tag` - Lead tags
- `crm.lost.reason` - Lost reasons

#### **sales_team** - Sales Team Management
Multi-team sales organization.

**Models:**
- `crm.team` - Sales team definitions
- `crm.team.member` - Team membership

---

### Inventory & Supply Chain Modules (45+ modules)

#### **stock** - Inventory Management
Complete warehouse and inventory system.

**Core Models:**
- `stock.picking` - Transfers/deliveries
- `stock.move` - Product movements
- `stock.move.line` - Detailed move lines
- `stock.warehouse` - Warehouse configuration
- `stock.location` - Storage locations
- `stock.quant` - Inventory quantities
- `product.product` - Products
- `product.template` - Product templates

**Database Tables:**
```sql
stock_picking                   -- Delivery orders, receipts
stock_move                      -- Product movements
stock_move_line                 -- Detailed tracking
stock_warehouse                 -- Warehouse config
stock_location                  -- Locations/bins
stock_quant                     -- On-hand inventory
stock_inventory                 -- Inventory adjustments
stock_production_lot            -- Serial/lot numbers
```

**Features:**
- Multi-warehouse support
- Lot and serial number tracking
- Barcode scanning
- Inventory valuation
- Delivery routes and rules

#### **purchase** - Purchase Management
Purchase order and vendor management.

**Models:**
- `purchase.order` - Purchase orders
- `purchase.order.line` - PO lines
- `purchase.requisition` - Purchase agreements
- `res.partner` - Vendors

#### **mrp** - Manufacturing
Manufacturing orders and bill of materials.

**Models:**
- `mrp.production` - Manufacturing orders
- `mrp.bom` - Bills of material
- `mrp.bom.line` - BOM components
- `mrp.workcenter` - Work centers
- `mrp.routing` - Manufacturing routings

---

### HR & Employee Management Modules (35+ modules)

#### **hr** - Human Resources
Employee information and organization.

**Models:**
- `hr.employee` - Employee records
- `hr.department` - Departments
- `hr.job` - Job positions
- `hr.contract` - Employment contracts

**Database Tables:**
```sql
hr_employee                     -- Employee master data
hr_department                   -- Organizational structure
hr_job                          -- Job positions
hr_contract                     -- Employment contracts
hr_employee_category            -- Employee categories
```

#### **hr_attendance** - Time & Attendance
Employee check-in/check-out tracking.

**Models:**
- `hr.attendance` - Attendance records
- `hr.employee` - Extended with attendance fields

#### **hr_holidays** - Time Off Management
Leave request and approval workflow.

**Models:**
- `hr.leave` - Leave requests
- `hr.leave.type` - Leave types
- `hr.leave.allocation` - Leave allocations

#### **hr_expense** - Expense Management
Employee expense tracking and reimbursement.

**Models:**
- `hr.expense` - Expense records
- `hr.expense.sheet` - Expense reports

#### **hr_recruitment** - Recruitment
Job application and candidate tracking.

**Models:**
- `hr.applicant` - Job applications
- `hr.job` - Job positions
- `hr.recruitment.stage` - Application stages

---

### Project Management Modules (20+ modules)

#### **project** - Project Management
Task and project tracking.

**Models:**
- `project.project` - Projects
- `project.task` - Tasks
- `project.milestone` - Milestones
- `project.tags` - Task tags

**Database Structure:**
```sql
project_project                 -- Projects
project_task                    -- Tasks
project_task_type               -- Task stages
project_milestone               -- Milestones
project_tags                    -- Tags
project_update                  -- Project updates
```

#### **hr_timesheet** - Timesheet
Time tracking on tasks.

**Models:**
- `account.analytic.line` - Timesheet entries
- `project.task` - Extended with timesheet

---

### E-commerce & Website Modules (60+ modules)

#### **website** - Website Builder
Content management system.

**Models:**
- `website` - Website configuration
- `website.page` - Web pages
- `website.menu` - Website menu
- `website.visitor` - Visitor tracking

#### **website_sale** - E-commerce
Online store functionality.

**Models:**
- `sale.order` - Shopping cart orders
- `website.pricelist` - E-commerce pricelists
- `product.ribbon` - Product badges

#### **website_blog** - Blog
Blog and content publishing.

**Models:**
- `blog.blog` - Blog configuration
- `blog.post` - Blog posts
- `blog.tag` - Post tags

---

### Point of Sale Modules (40+ modules)

#### **point_of_sale** - POS
Retail point of sale system.

**Models:**
- `pos.order` - POS orders
- `pos.order.line` - Order lines
- `pos.session` - Cashier sessions
- `pos.config` - POS configuration
- `pos.payment` - POS payments
- `pos.payment.method` - Payment methods

**Database Tables:**
```sql
pos_order                       -- POS orders
pos_order_line                  -- Order lines
pos_session                     -- Cash sessions
pos_config                      -- POS terminals
pos_payment                     -- Payments
pos_category                    -- Product categories
```

---

### Communication Modules (15+ modules)

#### **mail** - Messaging
Core messaging and notification system.

**Models:**
- `mail.message` - Messages and emails
- `mail.channel` - Discussion channels
- `mail.followers` - Document followers
- `mail.activity` - Scheduled activities
- `mail.template` - Email templates
- `mail.tracking.value` - Field change tracking

**Database Structure:**
```sql
mail_message                    -- Messages/emails
mail_channel                    -- Chat channels
mail_channel_member             -- Channel membership
mail_followers                  -- Document followers
mail_activity                   -- To-do activities
mail_template                   -- Email templates
mail_tracking_value             -- Field tracking
```

#### **im_livechat** - Live Chat
Customer support chat.

**Models:**
- `im_livechat.channel` - Chat channels
- `mail.channel` - Extended for live chat

---

### Marketing Modules (35+ modules)

#### **mass_mailing** - Email Marketing
Mass email campaigns.

**Models:**
- `mailing.mailing` - Email campaigns
- `mailing.list` - Mailing lists
- `mailing.contact` - Contacts
- `mailing.trace` - Delivery tracking

#### **survey** - Surveys
Survey creation and response collection.

**Models:**
- `survey.survey` - Surveys
- `survey.question` - Questions
- `survey.user_input` - Survey responses
- `survey.user_input.line` - Answer details

---

### Localization Modules (217 modules)

#### Country-Specific Accounting
Tax rules, chart of accounts, and reporting for 100+ countries.

**Pattern - l10n_XX modules:**
Each localization module extends:
- `account.account` - Country-specific accounts
- `account.tax` - Local tax rules
- `account.fiscal.position` - Tax mappings

**Major Localizations:**
- `l10n_us` - United States
- `l10n_uk` - United Kingdom
- `l10n_fr` - France
- `l10n_de` - Germany
- `l10n_cn` - China
- `l10n_in` - India
- `l10n_mx` - Mexico
- `l10n_br` - Brazil
- And 100+ more countries

#### EDI Integrations (30 modules)
Electronic invoicing for various countries.

**Models:**
- `account.edi.document` - EDI documents
- `account.edi.format` - EDI formats

---

### Payment Provider Modules (19 modules)

Payment gateway integrations for online payments.

**Supported Providers:**
- Stripe (`payment_stripe`)
- PayPal (`payment_paypal`)
- Adyen (`payment_adyen`)
- Authorize.net (`payment_authorize`)
- Razorpay (`payment_razorpay`)
- Mollie (`payment_mollie`)
- And 13+ more providers

**Common Models:**
- `payment.provider` - Payment provider config
- `payment.transaction` - Payment transactions
- `payment.token` - Saved payment methods

---

## Database Schema Patterns

### 1. Multi-Company Architecture

```sql
-- Most business models include:
company_id                      -- Many2one to res.company
-- Automatic filtering by user's companies
```

### 2. Chatter Integration

Models with messaging inherit `mail.thread`:
```sql
message_ids                     -- One2many to mail.message
activity_ids                    -- One2many to mail.activity
message_follower_ids            -- One2many to mail.followers
```

### 3. State Management

Common state field pattern:
```python
state = fields.Selection([
    ('draft', 'Draft'),
    ('confirmed', 'Confirmed'),
    ('done', 'Done'),
    ('cancel', 'Cancelled')
], default='draft')
```

### 4. Sequence Numbering

```python
name = fields.Char(string='Number', 
                   required=True, 
                   copy=False, 
                   readonly=True,
                   default='New')
```

### 5. Access Control

```sql
-- Security via groups
read_uid                        -- Users who can read
write_uid                       -- Users who can write
-- Additional: ir.rule for record-level security
```

---

## Data Relationships

### Common Relationship Patterns

1. **Master-Detail:** One2many relationships
   - `sale.order` → `sale.order.line`
   - `account.move` → `account.move.line`
   - `purchase.order` → `purchase.order.line`

2. **Associations:** Many2many relationships
   - `res.users` ↔ `res.groups`
   - `product.template` ↔ `product.tag`
   - `project.task` ↔ `project.tags`

3. **Polymorphic:** Reference fields
   - `mail.message.res_model` + `res_id`
   - `mail.activity.res_model` + `res_id`

### Cross-Module Dependencies

**Core Dependencies Flow:**
```
base
├── mail (messaging)
├── web (UI framework)
├── product (products)
│   └── uom (units)
├── account (accounting)
│   ├── payment (payments)
│   └── analytic (analytics)
├── sale
│   └── stock
│       ├── purchase
│       └── mrp
└── hr
    ├── hr_attendance
    ├── hr_holidays
    └── hr_expense
```

---

## Database Performance Considerations

### Indexing Strategy

1. **Automatic Indexes:**
   - Primary keys (`id`)
   - Foreign keys (Many2one fields)
   - Fields with `index=True`

2. **Custom Indexes:**
   ```python
   _sql_constraints = [
       ('name_uniq', 'UNIQUE(name)', 'Name must be unique!'),
   ]
   ```

### Query Optimization

1. **Prefetching:** Automatic batching of reads
2. **Lazy Loading:** One2many and Many2many loaded on access
3. **Read Groups:** Aggregation at database level
4. **Domain Optimization:** Push filters to SQL WHERE clauses

### Data Archiving

Most models support soft delete:
```python
active = fields.Boolean(default=True)
```
Records are filtered out when `active=False` instead of being deleted.

---

## Database Maintenance

### Common Operations

1. **Backup:**
   - Web interface: Database Manager
   - Command line: `pg_dump`

2. **Migrations:**
   - Automatic schema updates on module upgrade
   - Custom migration scripts in `migrations/` folder

3. **Data Cleaning:**
   - Scheduled actions for cleanup
   - Archive old records
   - Vacuum analyze for performance

### Database Size Management

Typical database size factors:
- Attachments (largest - stored in `ir_attachment`)
- Message history (`mail_message`)
- Tracking values (`mail_tracking_value`)
- Analytic lines (`account_analytic_line`)

---

## Security Model

### Record Rules (`ir.rule`)

Domain-based record filtering:
```python
[('company_id', 'in', company_ids)]  # Multi-company
[('user_id', '=', user.id)]          # Own records
[('team_id.member_ids', 'in', [user.id])]  # Team records
```

### Access Control Lists (`ir.model.access`)

Model-level permissions (Create, Read, Update, Delete) per group.

### Field-Level Security

```python
groups = fields.Many2many('res.groups', 
                          groups='base.group_system')
```

---

## Conclusion

The Odoo database architecture provides:
- ✅ Comprehensive business data model (598 modules)
- ✅ Flexible ORM with powerful abstractions
- ✅ Multi-company and multi-currency support
- ✅ Scalable PostgreSQL backend
- ✅ Extensive localization support
- ✅ Rich ecosystem of integrated modules

All modules follow consistent patterns for maintainability and extensibility.
