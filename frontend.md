# Odoo Frontend Documentation

## Overview

This document provides comprehensive documentation of the Odoo frontend architecture, JavaScript framework, UI components, and views across all modules.

**Version:** Odoo 19.0  
**Total Modules:** 598  
**JavaScript Files:** 5,608  
**XML Views:** 3,383  
**UI Framework:** Odoo Web Framework (OWL - Odoo Web Library)

---

## Table of Contents

1. [Frontend Architecture](#frontend-architecture)
2. [Odoo Web Framework (OWL)](#odoo-web-framework-owl)
3. [View System](#view-system)
4. [JavaScript Components](#javascript-components)
5. [Module-by-Module Frontend Features](#module-by-module-frontend-features)
6. [Website & E-commerce](#website--e-commerce)
7. [Styling & Themes](#styling--themes)
8. [Mobile & Responsive Design](#mobile--responsive-design)

---

## Frontend Architecture

### Technology Stack

**Core Technologies:**
- **JavaScript Framework:** OWL (Odoo Web Library) - Custom reactive framework
- **Template Engine:** QWeb (XML-based templates)
- **CSS Framework:** Bootstrap 5 + Custom Odoo styles
- **Build Tools:** Webpack-like asset bundling
- **Module System:** ES6 modules with AMD fallback

### Frontend Directory Structure

```
addons/web/static/src/
├── core/                       # Core framework
│   ├── browser/               # Browser utilities
│   ├── commands/              # Command palette
│   ├── dialog/                # Dialog system
│   ├── dropdown/              # Dropdown components
│   ├── l10n/                  # Localization
│   ├── network/               # HTTP/RPC
│   ├── notifications/         # Toast notifications
│   ├── popover/               # Popover components
│   ├── registry/              # Service registry
│   ├── user/                  # User service
│   └── utils/                 # Utilities
├── views/                      # View types
│   ├── form/                  # Form view
│   ├── list/                  # List/Tree view
│   ├── kanban/                # Kanban view
│   ├── calendar/              # Calendar view
│   ├── graph/                 # Chart view
│   ├── pivot/                 # Pivot table
│   ├── map/                   # Map view
│   ├── gantt/                 # Gantt chart
│   ├── cohort/                # Cohort analysis
│   └── activity/              # Activity view
├── search/                     # Search panel
├── webclient/                  # Web client shell
├── model/                      # Data models
├── legacy/                     # Legacy code
└── scss/                       # Stylesheets

Individual Module Structure:
addons/[module]/static/
├── src/
│   ├── components/            # Vue-like components
│   ├── models/                # JS models
│   ├── services/              # Services
│   ├── xml/                   # QWeb templates
│   └── scss/                  # Module styles
└── tests/
    └── tours/                 # Integration tests
```

---

## Odoo Web Framework (OWL)

### OWL Component System

**OWL (Odoo Web Library)** is a reactive component framework similar to Vue.js/React.

#### Basic Component

```javascript
/** @odoo-module **/
import { Component, useState } from "@odoo/owl";

export class MyComponent extends Component {
    static template = "my_module.MyTemplate";
    static props = {
        title: String,
        items: Array,
    };
    
    setup() {
        this.state = useState({
            count: 0,
        });
    }
    
    increment() {
        this.state.count++;
    }
}
```

#### Component Template (QWeb)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<templates xml:space="preserve">
    <t t-name="my_module.MyTemplate">
        <div class="my-component">
            <h1 t-esc="props.title"/>
            <p>Count: <t t-esc="state.count"/></p>
            <button t-on-click="increment">Increment</button>
            
            <ul>
                <li t-foreach="props.items" t-as="item" t-key="item.id">
                    <t t-esc="item.name"/>
                </li>
            </ul>
        </div>
    </t>
</templates>
```

#### Component Registration

```javascript
import { registry } from "@web/core/registry";

registry.category("actions").add("my_action", MyComponent);
```

### State Management

**useState Hook:**
```javascript
import { useState } from "@odoo/owl";

setup() {
    this.state = useState({
        data: [],
        loading: false,
    });
}
```

**useService Hook:**
```javascript
import { useService } from "@web/core/utils/hooks";

setup() {
    this.orm = useService("orm");
    this.notification = useService("notification");
    this.actionService = useService("action");
}
```

**useRef Hook:**
```javascript
import { useRef } from "@odoo/owl";

setup() {
    this.inputRef = useRef("myInput");
}

mounted() {
    this.inputRef.el.focus();
}
```

### Component Lifecycle

```javascript
class MyComponent extends Component {
    setup() {
        // Called once during component creation
    }
    
    willStart() {
        // Before first render (async allowed)
        return this.loadData();
    }
    
    mounted() {
        // After first render
    }
    
    willPatch() {
        // Before re-render
    }
    
    patched() {
        // After re-render
    }
    
    willUnmount() {
        // Before component destroyed
    }
}
```

---

## View System

### View Types

Odoo provides multiple view types for different data visualization needs:

#### 1. Form View

**Purpose:** Create/edit individual records

**XML Definition:**
```xml
<record id="view_partner_form" model="ir.ui.view">
    <field name="name">res.partner.form</field>
    <field name="model">res.partner</field>
    <field name="arch" type="xml">
        <form>
            <sheet>
                <div class="oe_title">
                    <h1><field name="name" placeholder="Name"/></h1>
                </div>
                <group>
                    <group>
                        <field name="email"/>
                        <field name="phone"/>
                    </group>
                    <group>
                        <field name="street"/>
                        <field name="city"/>
                        <field name="country_id"/>
                    </group>
                </group>
                <notebook>
                    <page string="Contacts" name="contacts">
                        <field name="child_ids"/>
                    </page>
                    <page string="Sales &amp; Purchase">
                        <field name="property_payment_term_id"/>
                    </page>
                </notebook>
            </sheet>
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="message_ids"/>
            </div>
        </form>
    </field>
</record>
```

**JavaScript Component:**
```javascript
/** @odoo-module **/
import { FormController } from "@web/views/form/form_controller";

export class CustomFormController extends FormController {
    async onSave() {
        // Custom save logic
        await super.onSave(...arguments);
    }
}
```

#### 2. List View (Tree)

**Purpose:** Display records in table format

```xml
<tree string="Partners" editable="bottom">
    <field name="name"/>
    <field name="email"/>
    <field name="phone"/>
    <field name="city"/>
    <field name="country_id"/>
</tree>
```

**Features:**
- Inline editing (`editable="top|bottom"`)
- Multi-selection
- Sorting and filtering
- Pagination
- Export to Excel/CSV

#### 3. Kanban View

**Purpose:** Card-based visualization (drag & drop)

```xml
<kanban default_group_by="stage_id">
    <templates>
        <t t-name="kanban-box">
            <div class="oe_kanban_card">
                <div class="oe_kanban_content">
                    <div class="oe_kanban_title">
                        <field name="name"/>
                    </div>
                    <div class="oe_kanban_footer">
                        <field name="priority" widget="priority"/>
                        <field name="user_id" widget="many2one_avatar_user"/>
                    </div>
                </div>
            </div>
        </t>
    </templates>
</kanban>
```

**JavaScript Enhancement:**
```javascript
import { kanbanView } from "@web/views/kanban/kanban_view";

export const customKanbanView = {
    ...kanbanView,
    Controller: CustomKanbanController,
};

registry.category("views").add("custom_kanban", customKanbanView);
```

#### 4. Calendar View

**Purpose:** Display records on calendar

```xml
<calendar date_start="date_start" 
          date_stop="date_end" 
          color="user_id"
          mode="month">
    <field name="name"/>
    <field name="partner_id"/>
</calendar>
```

**Modes:** day, week, month, year

#### 5. Pivot View

**Purpose:** Data analysis with pivot table

```xml
<pivot string="Sales Analysis">
    <field name="product_id" type="row"/>
    <field name="date" interval="month" type="col"/>
    <field name="price_total" type="measure"/>
</pivot>
```

#### 6. Graph View

**Purpose:** Charts and graphs

```xml
<graph type="bar" stacked="True">
    <field name="date" interval="month"/>
    <field name="amount_total" type="measure"/>
</graph>
```

**Chart Types:** bar, line, pie

#### 7. Gantt View

**Purpose:** Project timeline visualization

```xml
<gantt date_start="date_start"
       date_stop="date_stop"
       default_scale="month">
    <field name="name"/>
    <field name="user_id"/>
</gantt>
```

#### 8. Map View

**Purpose:** Geolocation on map

```xml
<map res_partner="partner_id">
    <field name="name"/>
    <field name="partner_id"/>
</map>
```

#### 9. Activity View

**Purpose:** Activity/todo management

```xml
<activity string="Activities">
    <templates>
        <div t-name="activity-box">
            <field name="user_id"/>
        </div>
    </templates>
</activity>
```

### Search View

**Purpose:** Filter, group, and search records

```xml
<search>
    <field name="name" string="Name"/>
    <field name="email"/>
    
    <filter name="my_partners" 
            string="My Partners"
            domain="[('user_id', '=', uid)]"/>
    
    <filter name="companies" 
            string="Companies"
            domain="[('is_company', '=', True)]"/>
    
    <separator/>
    
    <filter name="country" 
            string="Country" 
            context="{'group_by': 'country_id'}"/>
    
    <searchpanel>
        <field name="country_id" icon="fa-globe"/>
        <field name="state_id" icon="fa-map-marker"/>
    </searchpanel>
</search>
```

---

## JavaScript Components

### Core Components

#### 1. Dialog Component

```javascript
import { Dialog } from "@web/core/dialog/dialog";

setup() {
    this.dialog = useService("dialog");
}

openDialog() {
    this.dialog.add(Dialog, {
        title: "Confirmation",
        body: "Are you sure?",
        confirm: () => this.doAction(),
        cancel: () => {},
    });
}
```

#### 2. Notification Service

```javascript
setup() {
    this.notification = useService("notification");
}

showNotification() {
    this.notification.add("Operation successful", {
        type: "success",  // success, info, warning, danger
        title: "Success",
    });
}
```

#### 3. Action Service

```javascript
setup() {
    this.action = useService("action");
}

doAction() {
    this.action.doAction({
        type: 'ir.actions.act_window',
        res_model: 'res.partner',
        views: [[false, 'list'], [false, 'form']],
    });
}
```

#### 4. ORM Service

```javascript
setup() {
    this.orm = useService("orm");
}

async loadData() {
    const records = await this.orm.searchRead(
        "res.partner",
        [["is_company", "=", true]],
        ["name", "email"],
        { limit: 10 }
    );
    
    await this.orm.create("res.partner", [{
        name: "New Partner",
        email: "test@example.com"
    }]);
    
    await this.orm.write("res.partner", [id], {
        name: "Updated Name"
    });
    
    await this.orm.unlink("res.partner", [id]);
}
```

#### 5. RPC Service

```javascript
setup() {
    this.rpc = useService("rpc");
}

async callMethod() {
    const result = await this.rpc("/web/dataset/call_kw", {
        model: "res.partner",
        method: "custom_method",
        args: [arg1, arg2],
        kwargs: {},
    });
}
```

### Field Widgets

Custom field rendering in forms:

#### Built-in Widgets

**Char/Text Fields:**
- `char` - Basic text input
- `email` - Email validation
- `phone` - Phone formatting
- `url` - URL link
- `text` - Textarea
- `html` - Rich text editor

**Numeric Fields:**
- `integer` - Integer input
- `float` - Float input
- `monetary` - Currency formatting
- `percentage` - Percentage display
- `progressbar` - Progress indicator

**Date/Time:**
- `date` - Date picker
- `datetime` - Date + time picker
- `remaining_days` - Days countdown

**Relational:**
- `many2one` - Dropdown/autocomplete
- `many2many` - Tags/chips
- `one2many` - Embedded list
- `many2many_checkboxes` - Checkbox list
- `many2many_tags` - Tag input
- `many2one_avatar` - Avatar display
- `selection` - Dropdown selection

**Special:**
- `boolean` - Checkbox
- `image` - Image upload/display
- `binary` - File upload
- `priority` - Star rating
- `badge` - Badge display
- `label_selection` - Label badges
- `handle` - Drag handle
- `color` - Color picker
- `statinfo` - Stat button
- `statusbar` - Status pipeline

#### Custom Widget Example

```javascript
/** @odoo-module **/
import { registry } from "@web/core/registry";
import { Component } from "@odoo/owl";

class CustomWidget extends Component {
    static template = "my_module.CustomWidget";
    static props = {
        value: { type: String, optional: true },
        update: Function,
    };
    
    onChange(ev) {
        this.props.update(ev.target.value);
    }
}

registry.category("fields").add("custom_widget", {
    component: CustomWidget,
});
```

**Usage in XML:**
```xml
<field name="my_field" widget="custom_widget"/>
```

---

## Module-by-Module Frontend Features

### Web Module (`web`)

**Core Features:**
- Main webclient application
- View rendering engine
- Action management
- Menu system
- User preferences
- Search functionality

**Key Components:**
- `WebClient` - Main application shell
- `ActionContainer` - Action renderer
- `ControlPanel` - Search and filters
- `NavBar` - Top navigation
- `SideBar` - Action sidebar

**JavaScript Files:** 800+ files

### Mail Module (`mail`)

**Features:**
- Chatter (messaging widget)
- Inbox and notifications
- Channel/chat interface
- Email composer
- Activity management
- Document followers

**Components:**
```javascript
// Chatter integration
<div class="oe_chatter">
    <field name="message_follower_ids"/>
    <field name="activity_ids"/>
    <field name="message_ids"/>
</div>
```

**JavaScript Components:**
- `Chatter` - Main messaging component
- `Composer` - Message composer
- `Thread` - Message thread
- `ActivityMenu` - Activity dropdown
- `MessagingMenu` - Chat menu

### Website Module (`website`)

**Features:**
- Drag-and-drop page builder
- Content management
- SEO tools
- Multi-website support
- Visitor tracking

**Frontend Components:**
- Website editor
- Building blocks (snippets)
- Theme customization
- Mega menu builder
- Form builder

**Page Builder:**
```xml
<template id="custom_snippet">
    <section class="s_custom_snippet">
        <div class="container">
            <h2>Custom Section</h2>
            <p>Content here</p>
        </div>
    </section>
</template>
```

**JavaScript:**
```javascript
// Website editor plugin
odoo.define('website.snippet.editor', function (require) {
    const options = require('web_editor.snippets.options');
    
    options.registry.CustomSnippet = options.Class.extend({
        start: function () {
            // Initialize snippet
        },
    });
});
```

### Point of Sale Module (`point_of_sale`)

**Features:**
- Touch-optimized POS interface
- Offline-first architecture
- Product catalog
- Payment processing
- Receipt printing
- Session management

**Key Screens:**
- Product screen
- Payment screen
- Receipt screen
- Customer display

**JavaScript Architecture:**
```javascript
// POS models
class PosOrder extends Model {
    constructor(obj, options) {
        super(obj, options);
        this.orderlines = [];
        this.paymentlines = [];
    }
    
    add_product(product) {
        // Add product to order
    }
    
    add_paymentline(paymentMethod) {
        // Add payment
    }
}
```

**Offline Sync:**
- IndexedDB for local storage
- Background sync when online
- Order queue management

### Accounting Module (`account`)

**Features:**
- Invoice/bill forms
- Payment forms
- Bank reconciliation interface
- Journal entry forms
- Financial reports
- Tax configuration UI

**Components:**
- Invoice form with payment widget
- Bank reconciliation widget
- Chart of accounts tree
- Tax computation preview

### Project Module (`project`)

**Features:**
- Kanban board for tasks
- Gantt chart timeline
- Task forms with subtasks
- Project dashboard
- Time tracking interface

**Views:**
- Kanban (default)
- List view
- Gantt chart
- Calendar view
- Activity view

### E-commerce Module (`website_sale`)

**Features:**
- Product catalog
- Shopping cart
- Checkout process
- Payment integration
- Order tracking
- Product comparison
- Wishlist

**JavaScript Features:**
```javascript
// Add to cart
$(document).on('click', '.a-submit', function (ev) {
    ev.preventDefault();
    const $form = $(this).closest('form');
    const productId = $form.find('input[name="product_id"]').val();
    
    ajax.jsonRpc('/shop/cart/update', 'call', {
        product_id: parseInt(productId),
        add_qty: 1,
    }).then(function (data) {
        // Update cart widget
    });
});
```

### Blog Module (`website_blog`)

**Features:**
- Blog post editor
- Category/tag management
- Comment system
- Social sharing
- SEO optimization

### Events Module (`website_event`)

**Features:**
- Event listing
- Registration form
- Ticket management
- Event calendar
- Badge printing

### HR Modules

#### **hr_attendance** - Attendance Kiosk
- Touch-friendly check-in/out interface
- Employee photo display
- QR code scanning
- Real-time status updates

#### **hr_expense** - Expense Portal
- Expense submission form
- Receipt upload
- Approval workflow UI
- Expense reports

### Live Chat Module (`im_livechat`)

**Features:**
- Chat widget for websites
- Operator interface
- Canned responses
- Chat history
- Visitor info panel

**Website Integration:**
```xml
<script type="text/javascript">
    window.livechatData = {
        channel_uuid: 'xxx-xxx-xxx',
    };
</script>
```

---

## Website & E-commerce

### Website Builder

**Drag & Drop Editor:**
- Visual page builder
- Pre-built content blocks (snippets)
- Responsive design tools
- Animation options
- Custom CSS/JS injection

**Building Blocks (Snippets):**
```
- Headers/Navbars
- Hero sections
- Features/Services
- Team members
- Testimonials
- Pricing tables
- Call-to-action
- Forms
- Galleries
- Blog lists
- Footer blocks
```

**Snippet Definition:**
```xml
<template id="s_custom_block" name="Custom Block">
    <section class="s_custom_block">
        <div class="container">
            <div class="row">
                <div class="col-lg-6">
                    <h2>Title</h2>
                </div>
                <div class="col-lg-6">
                    <p>Content</p>
                </div>
            </div>
        </div>
    </section>
</template>
```

### E-commerce Features

**Product Pages:**
- Image gallery with zoom
- Variant selection (size, color, etc.)
- Add to cart
- Product description tabs
- Related/accessory products
- Customer reviews
- Stock availability
- Delivery time estimation

**Shopping Cart:**
- Line item management
- Quantity adjustment
- Promo code input
- Shipping calculation
- Tax display
- Cart summary

**Checkout:**
- Multi-step process
- Address forms
- Shipping method selection
- Payment method selection
- Order review
- Payment processing
- Order confirmation

**Payment Processing:**
```javascript
// Payment form submission
$('#payment_form').on('submit', function (ev) {
    ev.preventDefault();
    const $form = $(this);
    
    // Tokenize card (if using Stripe/etc)
    paymentProvider.createToken().then(function (token) {
        $form.append(
            $('<input type="hidden" name="payment_token">').val(token)
        );
        $form.submit();
    });
});
```

---

## Styling & Themes

### CSS Architecture

**Structure:**
```
web/static/src/scss/
├── bootstrap/              # Bootstrap framework
├── primary_variables.scss  # Color/font variables
├── secondary_variables.scss
├── utilities.scss          # Utility classes
├── mixins.scss            # Sass mixins
└── components/            # Component styles
    ├── form.scss
    ├── button.scss
    ├── navbar.scss
    └── ...
```

### Theme Customization

**Variables:**
```scss
// Primary colors
$o-color-primary: #875A7B;
$o-color-secondary: #00A09D;

// Brand colors
$o-brand-primary: $o-color-primary;
$o-brand-odoo: #875A7B;
$o-brand-lightsecondary: #DDDDFF;

// Bootstrap overrides
$primary: $o-brand-primary;
$secondary: $o-brand-secondary;
```

**Custom Theme Module:**
```xml
<template id="assets_frontend" inherit_id="web.assets_frontend">
    <xpath expr="." position="inside">
        <link rel="stylesheet" type="text/scss" href="/theme_custom/static/src/scss/primary_variables.scss"/>
        <link rel="stylesheet" type="text/scss" href="/theme_custom/static/src/scss/custom.scss"/>
    </xpath>
</template>
```

### Responsive Design

**Breakpoints:**
```scss
// Extra small (phones)
@media (max-width: 575.98px) { }

// Small (tablets)
@media (min-width: 576px) and (max-width: 767.98px) { }

// Medium (desktop)
@media (min-width: 768px) and (max-width: 991.98px) { }

// Large (large desktop)
@media (min-width: 992px) and (max-width: 1199.98px) { }

// Extra large
@media (min-width: 1200px) { }
```

**Responsive Utilities:**
```html
<!-- Hide on mobile -->
<div class="d-none d-md-block">Desktop only</div>

<!-- Show only on mobile -->
<div class="d-block d-md-none">Mobile only</div>

<!-- Responsive columns -->
<div class="row">
    <div class="col-12 col-md-6 col-lg-4">
        Responsive column
    </div>
</div>
```

---

## Mobile & Responsive Design

### Mobile App

**Odoo Mobile App:**
- Native iOS and Android apps
- Web view wrapper around Odoo web
- Push notifications
- Offline support
- Camera/barcode integration

### Progressive Web App (PWA)

**Service Worker:**
```javascript
// Service worker for offline support
self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request).then(response => {
            return response || fetch(event.request);
        })
    );
});
```

**PWA Manifest:**
```json
{
    "name": "Odoo",
    "short_name": "Odoo",
    "start_url": "/web",
    "display": "standalone",
    "theme_color": "#875A7B",
    "background_color": "#ffffff",
    "icons": [...]
}
```

### Touch Optimization

**POS Touch Interface:**
- Large touch targets
- Swipe gestures
- Pinch to zoom
- Pull to refresh

**Mobile Responsive:**
- Collapsible menus
- Bottom navigation
- Modal sheets
- Touch-friendly forms

---

## Asset Management

### Asset Bundles

**Main Bundles:**
```xml
<!-- Backend assets -->
<template id="assets_backend">
    <link rel="stylesheet" href="/web/static/src/scss/main.scss"/>
    <script src="/web/static/src/main.js"/>
</template>

<!-- Frontend assets -->
<template id="assets_frontend">
    <link rel="stylesheet" href="/web/static/src/scss/website.scss"/>
    <script src="/web/static/src/website.js"/>
</template>

<!-- POS assets -->
<template id="assets_pos">
    <script src="/point_of_sale/static/src/app/app.js"/>
</template>
```

**Custom Bundle:**
```xml
<template id="my_custom_assets" inherit_id="web.assets_backend">
    <xpath expr="." position="inside">
        <script type="text/javascript" src="/my_module/static/src/js/my_script.js"/>
        <link rel="stylesheet" href="/my_module/static/src/scss/my_style.scss"/>
    </xpath>
</template>
```

### Lazy Loading

**Code Splitting:**
```javascript
// Dynamic import
const loadComponent = async () => {
    const module = await import("./heavy_component.js");
    return module.HeavyComponent;
};
```

---

## Testing & QA

### JavaScript Tests

**QUnit Tests:**
```javascript
/** @odoo-module **/
import { describe, test } from "@odoo/hoot";
import { expect } from "@odoo/hoot-dom";

describe("My Component", () => {
    test("should render correctly", async () => {
        const component = await mount(MyComponent);
        expect(".my-component").toHaveCount(1);
    });
});
```

### Tour Tests

**Integration Tests:**
```javascript
/** @odoo-module **/
import { registry } from "@web/core/registry";

registry.category("web_tour.tours").add("my_tour", {
    test: true,
    steps: () => [
        {
            trigger: ".o_app[data-menu-xmlid='sale.sale_menu_root']",
            content: "Open Sales app",
        },
        {
            trigger: ".o_list_button_add",
            content: "Create new quotation",
        },
        {
            trigger: "input[name='partner_id']",
            content: "Select customer",
            run: "text Customer Name",
        },
        {
            trigger: ".o_form_button_save",
            content: "Save",
        },
    ],
});
```

**Run Tour:**
```javascript
// Browser console
odoo.startTour("my_tour");
```

---

## Performance Optimization

### Frontend Performance

**1. Asset Optimization:**
- Minification and compression
- Image optimization
- Lazy loading images
- Code splitting

**2. Caching:**
```javascript
// Cache API responses
const cache = new Map();

async function fetchData(key) {
    if (cache.has(key)) {
        return cache.get(key);
    }
    const data = await this.orm.searchRead(...);
    cache.set(key, data);
    return data;
}
```

**3. Virtual Scrolling:**
- List views with 1000+ records
- Render only visible items
- Reuse DOM elements

**4. Debouncing:**
```javascript
import { debounce } from "@web/core/utils/timing";

setup() {
    this.search = debounce(this._search, 300);
}

_search(term) {
    // Expensive search operation
}
```

### Loading Optimization

**Skeleton Screens:**
```xml
<div class="o_skeleton">
    <div class="o_skeleton_line"></div>
    <div class="o_skeleton_line"></div>
    <div class="o_skeleton_line"></div>
</div>
```

**Progressive Enhancement:**
- Load critical CSS first
- Defer non-critical scripts
- Preload key resources

---

## Internationalization (i18n)

### Translation

**JavaScript:**
```javascript
import { _t } from "@web/core/l10n/translation";

const message = _t("Hello World");
const formatted = _t("Hello %s", name);
```

**QWeb Templates:**
```xml
<t t-translation="off">
    Don't translate this
</t>

<button>
    <t t-esc="_t('Save')"/>
</button>
```

### Localization

**Date/Number Formatting:**
```javascript
import { localization } from "@web/core/l10n/localization";

// Format number
const formatted = localization.formatNumber(1234.56);
// Result: "1,234.56" (en_US) or "1.234,56" (de_DE)

// Format date
const dateStr = localization.formatDate(date);
```

**Currency:**
```xml
<field name="amount_total" widget="monetary" 
       options="{'currency_field': 'currency_id'}"/>
```

---

## Accessibility (a11y)

### ARIA Support

```xml
<div role="dialog" aria-labelledby="dialog-title" aria-modal="true">
    <h2 id="dialog-title">Dialog Title</h2>
    <p>Dialog content</p>
    <button aria-label="Close dialog">×</button>
</div>
```

### Keyboard Navigation

**Focus Management:**
```javascript
setup() {
    this.focusableElements = useRef("focusable");
}

mounted() {
    this.focusableElements.el.focus();
}
```

**Keyboard Shortcuts:**
- `Alt+H` - Home menu
- `Alt+K` - Command palette
- `Ctrl+K` - Search
- `Esc` - Close dialog/dropdown
- `Tab` - Navigate form fields
- `Enter` - Confirm/submit
- Arrow keys - Navigate lists

---

## Browser Support

**Supported Browsers:**
- Chrome/Edge (latest 2 versions)
- Firefox (latest 2 versions)
- Safari (latest 2 versions)

**Polyfills:**
- Modern JavaScript features
- CSS Grid fallbacks
- Fetch API

---

## Developer Tools

### Debugging

**Debug Mode:**
- Add `?debug=1` to URL
- Access to technical features
- View metadata
- Edit views from UI

**Developer Tools:**
- Browser DevTools
- Odoo debug menu
- Performance profiling
- Network inspection

### Component Inspector

**OWL DevTools:**
- Component tree
- Props inspection
- State inspection
- Performance metrics

---

## Best Practices

### 1. Component Design
- Single responsibility
- Reusable components
- Props validation
- Clear naming

### 2. Performance
- Minimize re-renders
- Use computed properties
- Lazy load heavy components
- Optimize images

### 3. Accessibility
- Semantic HTML
- ARIA labels
- Keyboard navigation
- Color contrast

### 4. Code Quality
- ESLint compliance
- Consistent formatting
- Meaningful comments
- Unit tests

### 5. Security
- Sanitize user input
- CSRF protection
- XSS prevention
- Content Security Policy

---

## Conclusion

The Odoo frontend provides:
- ✅ Modern reactive component framework (OWL)
- ✅ Rich set of view types (15+ view types)
- ✅ Comprehensive UI component library
- ✅ Powerful website builder and e-commerce
- ✅ Mobile-responsive design
- ✅ Touch-optimized interfaces (POS)
- ✅ Extensive customization options
- ✅ 5,608 JavaScript files across 598 modules
- ✅ Internationalization and accessibility support
- ✅ Modern web standards and best practices

All frontend modules follow consistent patterns for maintainability and extensibility.
