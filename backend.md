# Odoo Backend Documentation

## Overview

This document provides comprehensive documentation of the Odoo backend architecture, business logic, controllers, and API implementations across all modules.

**Version:** Odoo 19.0  
**Total Modules:** 598  
**Controllers:** 468 files  
**Wizards:** 363 files  
**Core Python Files:** 3,000+

---

## Table of Contents

1. [Backend Architecture](#backend-architecture)
2. [Core Backend Components](#core-backend-components)
3. [HTTP Layer & Routing](#http-layer--routing)
4. [Business Logic Layer](#business-logic-layer)
5. [Module-by-Module Backend Features](#module-by-module-backend-features)
6. [API Architecture](#api-architecture)
7. [Security & Authentication](#security--authentication)
8. [Background Jobs & Automation](#background-jobs--automation)

---

## Backend Architecture

### Technology Stack

**Core Technologies:**
- **Python:** 3.10 - 3.13
- **Framework:** Custom Odoo Framework
- **Web Server:** Werkzeug WSGI
- **Async Processing:** Gevent
- **Database:** PostgreSQL 13+

### Key Backend Directories

```
odoo/
├── __main__.py              # Application entry point
├── http.py                  # HTTP request handling
├── service/                 # Core services
│   ├── server.py           # Server management
│   ├── model.py            # Model service (RPC)
│   ├── db.py               # Database service
│   └── security.py         # Security checks
├── api/                     # API decorators
├── orm/                     # ORM implementation
├── models/                  # Model base classes
├── tools/                   # Utility functions
├── cli/                     # Command-line interface
└── addons/                  # Business modules
    └── [module]/
        ├── models/          # Business models
        ├── controllers/     # HTTP endpoints
        ├── wizard/          # Transient wizards
        └── report/          # Report generators
```

---

## Core Backend Components

### 1. Server Management (`odoo/service/`)

**Main Services:**

#### Database Service (`db.py`)
- Database creation/deletion
- Database listing
- Backup/restore operations
- Database duplication

#### Model Service (`model.py`)
- RPC endpoint for model operations
- Execute model methods remotely
- Transaction management

#### Server Service (`server.py`)
```python
# Server lifecycle
- start()           # Start HTTP/RPC servers
- stop()            # Graceful shutdown
- reload()          # Reload registry
- cron_spawn()      # Start scheduled jobs
```

### 2. HTTP Layer (`odoo/http.py`)

**Request Processing Pipeline:**
```
HTTP Request
    ↓
Werkzeug Router
    ↓
Endpoint Resolution
    ↓
Authentication
    ↓
Session Management
    ↓
Controller Method
    ↓
Response Generation
    ↓
HTTP Response
```

**Key Classes:**

```python
# Request object
class HttpRequest:
    session             # User session
    env                 # Environment (ORM access)
    params              # Request parameters
    httprequest         # Werkzeug request
    
# Response types
class Response:
    - render()          # Template rendering
    - redirect()        # HTTP redirect
    - json()            # JSON response
    - download()        # File download
```

### 3. Environment & Context (`odoo/orm/environments.py`)

**Environment Object:**
```python
env = self.env
env.user            # Current user (res.users)
env.company         # Current company (res.company)
env.companies       # Allowed companies
env.lang            # Current language
env.context         # Execution context dict
env.uid             # Current user ID
env.su              # Sudo environment (bypass security)

# Switching context
env.with_context(lang='fr_FR')
env.with_user(user)
env.with_company(company)
```

### 4. API Decorators (`odoo/api/`)

**Common Decorators:**

```python
@api.model              # Class method (no recordset)
@api.depends('field')   # Computed field dependencies
@api.constrains('field') # Validation constraints
@api.onchange('field')  # UI field change handlers
@api.autovacuum        # Scheduled vacuum method
@api.model_create_multi # Optimized batch creation
```

### 5. Tools & Utilities (`odoo/tools/`)

**Key Utilities:**

```python
# Mail utilities
mail.py                 # Email sending
mail_template.py        # Template rendering

# File handling
misc.py                 # File operations
image.py                # Image processing
pdf.py                  # PDF generation

# Data processing
date_utils.py           # Date/time utilities
float_utils.py          # Float comparison
safe_eval.py            # Safe Python evaluation
convert.py              # Data conversion
xml_utils.py            # XML processing

# Performance
cache.py                # Caching decorators
profiler.py             # Performance profiling
```

---

## HTTP Layer & Routing

### Controller Definition

**Basic Controller:**
```python
from odoo import http
from odoo.http import request

class MyController(http.Controller):
    
    @http.route('/my/endpoint', type='http', auth='user')
    def my_endpoint(self, **kwargs):
        return request.render('module.template', {
            'data': 'value'
        })
    
    @http.route('/api/data', type='json', auth='user')
    def api_endpoint(self, **kwargs):
        return {'status': 'success', 'data': []}
```

**Route Parameters:**
- `type`: 'http' (web pages) or 'json' (API)
- `auth`: 'user', 'public', 'none'
- `methods`: ['GET', 'POST', 'PUT', 'DELETE']
- `csrf`: CSRF protection (default: True)
- `cors`: CORS headers
- `website`: Website context required

### Session Management

**Session Object:**
```python
request.session.uid             # User ID
request.session.login           # User login
request.session.get('key')      # Session data
request.session.update({})      # Update session
request.session.logout()        # Clear session
```

### Response Types

**1. Template Rendering:**
```python
return request.render('module.template_name', {
    'variable': value
})
```

**2. JSON Response:**
```python
return {'success': True, 'data': data}
```

**3. Redirect:**
```python
return request.redirect('/target/url')
```

**4. File Download:**
```python
return request.make_response(
    file_content,
    headers=[
        ('Content-Type', 'application/pdf'),
        ('Content-Disposition', 'attachment; filename="file.pdf"')
    ]
)
```

---

## Business Logic Layer

### Model Methods

**Standard CRUD:**
```python
# Create
record = model.create({'field': value})
records = model.create([{'field': value1}, {'field': value2}])

# Read
records = model.search([('field', '=', value)])
records = model.browse([1, 2, 3])
data = records.read(['field1', 'field2'])

# Update
records.write({'field': new_value})

# Delete
records.unlink()
```

**Advanced Operations:**
```python
# Search with limit/offset
records = model.search(domain, limit=10, offset=20, order='name')

# Count
count = model.search_count(domain)

# Exists check
if records.exists():
    # records still valid

# Copy
new_record = record.copy()

# Toggle active
records.toggle_active()
```

### Computed Methods

```python
class SaleOrder(models.Model):
    _name = 'sale.order'
    
    amount_total = fields.Monetary(compute='_compute_amount_total')
    
    @api.depends('order_line.price_total')
    def _compute_amount_total(self):
        for order in self:
            order.amount_total = sum(order.order_line.mapped('price_total'))
```

### Constraints

```python
# SQL Constraint
_sql_constraints = [
    ('code_uniq', 'UNIQUE(code)', 'Code must be unique'),
]

# Python Constraint
@api.constrains('date_start', 'date_end')
def _check_dates(self):
    for record in self:
        if record.date_start > record.date_end:
            raise ValidationError("Start date must be before end date")
```

### Onchange Methods

```python
@api.onchange('partner_id')
def _onchange_partner_id(self):
    if self.partner_id:
        self.email = self.partner_id.email
        self.phone = self.partner_id.phone
```

---

## Module-by-Module Backend Features

### Accounting Module (`account`)

**Controllers:**
- `/account/invoice/download` - Invoice PDF download
- `/account/payment/process` - Payment processing
- `/account/bank/sync` - Bank synchronization

**Business Logic:**
```python
# Invoice posting
def action_post(self):
    # Validate invoice
    # Create journal entries
    # Update partner balance
    # Send notifications
    
# Payment reconciliation
def reconcile(self):
    # Match payments with invoices
    # Create reconciliation records
    # Update payment status
```

**Background Jobs:**
- Auto-posting invoices
- Payment reminder emails
- Currency rate updates
- Bank statement imports

---

### Sales Module (`sale`)

**Controllers:**
- `/shop` - E-commerce frontend
- `/shop/cart` - Shopping cart
- `/shop/checkout` - Checkout process
- `/shop/confirmation` - Order confirmation

**Business Logic:**

```python
class SaleOrder(models.Model):
    _name = 'sale.order'
    
    def action_confirm(self):
        """Confirm quotation to sales order"""
        # Validate order
        # Reserve inventory
        # Create invoice
        # Generate delivery order
        
    def _prepare_invoice(self):
        """Prepare invoice values from order"""
        return {
            'partner_id': self.partner_id.id,
            'invoice_line_ids': self._prepare_invoice_lines()
        }
    
    def _create_delivery(self):
        """Create delivery/picking from order"""
        # Create stock.picking
        # Create stock.move for each line
```

**Wizards:**
- Payment link wizard
- Make invoice wizard
- Cancel orders wizard

---

### Inventory Module (`stock`)

**Controllers:**
- `/stock/barcode` - Barcode scanning interface
- `/stock/report` - Stock reports

**Business Logic:**

```python
class StockPicking(models.Model):
    _name = 'stock.picking'
    
    def button_validate(self):
        """Validate transfer/delivery"""
        # Check availability
        # Update stock quantities
        # Create accounting entries
        # Update sale/purchase order status
        
    def do_unreserve(self):
        """Release reserved quantities"""
        
    def action_assign(self):
        """Reserve quantities"""
```

**Scheduled Actions:**
- Inventory replenishment
- Expired product alerts
- Stock level warnings
- Automatic reordering

---

### HR Module (`hr`)

**Controllers:**
- `/hr/employee/profile` - Employee portal
- `/hr/attendance/kiosk` - Check-in kiosk

**Business Logic:**

```python
class HrEmployee(models.Model):
    _name = 'hr.employee'
    
    def action_create_user(self):
        """Create portal/internal user for employee"""
        
    def _compute_related_users(self):
        """Link employee to user account"""

class HrAttendance(models.Model):
    _name = 'hr.attendance'
    
    def check_in(self):
        """Employee check-in"""
        
    def check_out(self):
        """Employee check-out"""
```

---

### Project Module (`project`)

**Controllers:**
- `/project/task` - Task view
- `/project/timeline` - Gantt view

**Business Logic:**

```python
class ProjectTask(models.Model):
    _name = 'project.task'
    
    def action_assign_to_me(self):
        """Assign task to current user"""
        
    def action_open_parent_task(self):
        """Navigate to parent task"""
        
    def write(self, vals):
        """Override to track changes"""
        # Track stage changes
        # Send notifications
        # Update project metrics
        return super().write(vals)
```

**Automation:**
- Task reminder emails
- SLA tracking
- Milestone notifications
- Resource allocation

---

### Mail Module (`mail`)

**Controllers:**
- `/mail/inbox` - Inbox view
- `/mail/read` - Mark as read
- `/mail/tracking/open` - Email open tracking

**Business Logic:**

```python
class MailThread(models.AbstractModel):
    """Mixin for models with chatter"""
    _name = 'mail.thread'
    
    def message_post(self, body='', subject=None, **kwargs):
        """Post message to chatter"""
        
    def message_subscribe(self, partner_ids):
        """Subscribe partners to document"""
        
    def activity_schedule(self, act_type_id, user_id, date_deadline):
        """Schedule activity/to-do"""
```

**Email Processing:**
- Incoming email gateway
- Bounce handling
- Spam filtering
- Attachment processing

---

### Website Module (`website`)

**Controllers:**
- `/` - Homepage
- `/page/*` - CMS pages
- `/sitemap.xml` - SEO sitemap
- `/website/info` - Website info

**Business Logic:**

```python
class Website(models.Model):
    _name = 'website'
    
    def get_current_website(self):
        """Get website for current request"""
        
    def is_publisher(self):
        """Check if user can edit website"""
        
class WebsiteVisitor(models.Model):
    """Track website visitors"""
    _name = 'website.visitor'
    
    def _update_visitor_last_visit(self):
        """Update last visit timestamp"""
```

**SEO Features:**
- Meta tag management
- Sitemap generation
- Robot.txt handling
- URL routing

---

### Point of Sale Module (`point_of_sale`)

**Controllers:**
- `/pos/web` - POS interface
- `/pos/ticket` - Print ticket
- `/pos/create_from_ui` - Create order from UI

**Business Logic:**

```python
class PosOrder(models.Model):
    _name = 'pos.order'
    
    def _process_payment_lines(self):
        """Process payments for order"""
        
    def action_pos_order_paid(self):
        """Mark order as paid"""
        # Update payment status
        # Create invoice if needed
        # Update inventory
        
    def refund(self):
        """Create refund order"""
```

**Offline Sync:**
- Local storage sync
- Order queue management
- Product data caching
- Session management

---

## API Architecture

### XML-RPC / JSON-RPC APIs

**Endpoint:** `/xmlrpc/2/*` or `/jsonrpc`

**Authentication:**
```python
# Login
uid = common.authenticate(db, username, password, {})

# Execute methods
models.execute_kw(db, uid, password, 
    'res.partner', 'search', 
    [[['is_company', '=', True]]])

models.execute_kw(db, uid, password,
    'res.partner', 'read', 
    [partner_ids], {'fields': ['name', 'email']})
```

### REST API Pattern

**Custom REST Controllers:**
```python
@http.route('/api/v1/partners', type='json', auth='user', methods=['GET'])
def get_partners(self, **kwargs):
    partners = request.env['res.partner'].search([])
    return partners.read(['name', 'email', 'phone'])

@http.route('/api/v1/partners', type='json', auth='user', methods=['POST'])
def create_partner(self, **kwargs):
    partner = request.env['res.partner'].create(kwargs)
    return {'id': partner.id}
```

### Webhook Support

**Outgoing Webhooks:**
```python
def _notify_webhook(self, event_type):
    """Send webhook notification"""
    webhook_url = self.env['ir.config_parameter'].get_param('webhook.url')
    data = {
        'event': event_type,
        'data': self.read()[0]
    }
    requests.post(webhook_url, json=data)
```

---

## Security & Authentication

### Authentication Methods

**1. Session-based (Web):**
```python
request.session.authenticate(db, login, password)
```

**2. API Keys:**
```python
@http.route('/api/endpoint', auth='api_key')
def endpoint(self):
    # Authenticate via X-API-Key header
    pass
```

**3. OAuth 2.0:**
- Provider integrations (Google, Microsoft, etc.)
- `auth_oauth` module

**4. LDAP:**
- `auth_ldap` module
- Active Directory integration

**5. Two-Factor Authentication:**
- TOTP support (`auth_totp`)
- Recovery codes
- Authenticator apps

### Access Control

**1. Group-based Access:**
```python
# Check group membership
if self.env.user.has_group('base.group_system'):
    # Admin action
    pass
```

**2. Record Rules:**
```xml
<record id="rule_my_records" model="ir.rule">
    <field name="name">My Records</field>
    <field name="model_id" ref="model_res_partner"/>
    <field name="domain_force">
        [('user_id', '=', user.id)]
    </field>
</record>
```

**3. Field-level Security:**
```python
field_name = fields.Char(groups='base.group_system')
```

### Secure Coding Practices

**1. SQL Injection Prevention:**
```python
# Good - parameterized query
self.env.cr.execute("SELECT * FROM table WHERE id = %s", (record_id,))

# Bad - string formatting
self.env.cr.execute(f"SELECT * FROM table WHERE id = {record_id}")
```

**2. XSS Prevention:**
```python
from markupsafe import Markup
safe_html = Markup(untrusted_html).striptags()
```

**3. CSRF Protection:**
- Automatic CSRF tokens for forms
- `csrf=False` only for public APIs

---

## Background Jobs & Automation

### Scheduled Actions (`ir.cron`)

**Define Cron Job:**
```xml
<record id="ir_cron_job" model="ir.cron">
    <field name="name">Daily Cleanup</field>
    <field name="model_id" ref="model_my_model"/>
    <field name="state">code</field>
    <field name="code">model._cron_cleanup()</field>
    <field name="interval_number">1</field>
    <field name="interval_type">days</field>
    <field name="numbercall">-1</field>
</record>
```

**Cron Method:**
```python
@api.model
def _cron_cleanup(self):
    """Cleanup old records"""
    old_date = fields.Date.today() - timedelta(days=90)
    old_records = self.search([
        ('create_date', '<', old_date),
        ('state', '=', 'draft')
    ])
    old_records.unlink()
```

### Base Automation (`base_automation`)

**Automated Actions:**
- Trigger: On Create, Update, Delete, Time condition
- Actions: Update fields, send email, execute code, create activity

**Example:**
```xml
<record id="automation_welcome_email" model="base.automation">
    <field name="name">Send Welcome Email</field>
    <field name="model_id" ref="base.model_res_partner"/>
    <field name="trigger">on_create</field>
    <field name="action_server_ids" eval="[(6, 0, [ref('action_send_email')])]"/>
</record>
```

### Queue Jobs (with `queue_job` module)

**Async Job:**
```python
from odoo.addons.queue_job.job import job

@job
def process_large_import(self, data):
    """Process import in background"""
    for line in data:
        # Process each line
        pass

# Enqueue job
self.with_delay().process_large_import(data)
```

---

## Error Handling

### Exception Types

```python
from odoo.exceptions import (
    AccessError,       # Access denied
    UserError,         # User-friendly error
    ValidationError,   # Validation failed
    RedirectWarning,   # Error with action button
    MissingError,      # Record doesn't exist
)

# Raise user error
if not condition:
    raise UserError(_("Invalid operation"))

# Validation error
if value < 0:
    raise ValidationError(_("Value must be positive"))
```

### Logging

```python
import logging
_logger = logging.getLogger(__name__)

_logger.debug("Debug message")
_logger.info("Info message")
_logger.warning("Warning message")
_logger.error("Error message")
_logger.exception("Exception occurred")  # Include traceback
```

---

## Performance Optimization

### 1. Database Query Optimization

```python
# Bad - N+1 queries
for partner in partners:
    invoices = partner.invoice_ids  # Query per partner

# Good - Prefetch
partners = self.env['res.partner'].search([])
partners.mapped('invoice_ids')  # Single query

# Use read_group for aggregations
data = self.env['sale.order'].read_group(
    [('state', '=', 'sale')],
    ['amount_total:sum'],
    ['partner_id']
)
```

### 2. Batch Operations

```python
# Bad - Multiple writes
for record in records:
    record.write({'state': 'done'})

# Good - Batch write
records.write({'state': 'done'})
```

### 3. Caching

```python
from odoo.tools import ormcache

@ormcache('arg1', 'arg2')
def expensive_computation(self, arg1, arg2):
    # Cached result per (arg1, arg2)
    return result
```

---

## Testing Backend Code

### Unit Tests

```python
from odoo.tests import TransactionCase

class TestSaleOrder(TransactionCase):
    
    def setUp(self):
        super().setUp()
        self.partner = self.env['res.partner'].create({
            'name': 'Test Partner'
        })
        
    def test_order_confirmation(self):
        order = self.env['sale.order'].create({
            'partner_id': self.partner.id,
        })
        order.action_confirm()
        self.assertEqual(order.state, 'sale')
```

### HTTP Tests

```python
from odoo.tests import HttpCase

class TestWebsite(HttpCase):
    
    def test_homepage(self):
        response = self.url_open('/')
        self.assertEqual(response.status_code, 200)
```

---

## Integration Patterns

### External API Integration

```python
import requests

class APIIntegration(models.Model):
    _name = 'api.integration'
    
    def _call_external_api(self):
        """Call external REST API"""
        url = "https://api.example.com/endpoint"
        headers = {'Authorization': f'Bearer {self.api_key}'}
        
        response = requests.get(url, headers=headers, timeout=30)
        response.raise_for_status()
        
        return response.json()
```

### Webhook Receivers

```python
@http.route('/webhook/stripe', type='json', auth='none', csrf=False)
def stripe_webhook(self, **kwargs):
    """Receive Stripe webhook"""
    payload = request.httprequest.data
    sig_header = request.httprequest.headers.get('Stripe-Signature')
    
    # Verify signature
    # Process webhook data
    
    return {'status': 'success'}
```

---

## Deployment & Configuration

### Configuration File (`odoo.conf`)

```ini
[options]
db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoo
addons_path = /path/to/addons
http_port = 8069
workers = 4
max_cron_threads = 2
limit_time_cpu = 600
limit_time_real = 1200
log_level = info
```

### Multi-Processing

- Workers for HTTP requests
- Separate cron threads
- Load balancing with proxy

### Server Commands

```bash
# Start server
./odoo-bin -c odoo.conf

# Update module
./odoo-bin -u module_name -d database

# Install module
./odoo-bin -i module_name -d database

# Create database
./odoo-bin -d newdb --without-demo=all
```

---

## Best Practices

### 1. Code Organization
- One class per file
- Logical grouping of methods
- Clear naming conventions

### 2. Documentation
- Docstrings for public methods
- Inline comments for complex logic
- README.md for modules

### 3. Error Handling
- Use appropriate exception types
- Provide helpful error messages
- Log errors appropriately

### 4. Security
- Never trust user input
- Use parameterized queries
- Validate all data

### 5. Performance
- Minimize database queries
- Use batch operations
- Cache expensive computations

---

## Conclusion

The Odoo backend provides:
- ✅ Robust Python-based framework
- ✅ Comprehensive ORM for database operations
- ✅ Flexible HTTP routing and API support
- ✅ Rich business logic across 598 modules
- ✅ Advanced security and authentication
- ✅ Scalable architecture with multi-processing
- ✅ Extensive automation and scheduling capabilities
- ✅ Integration-ready with webhooks and APIs

All modules follow consistent patterns for maintainability and extensibility.
