# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **product prototype documentation repository** for a ski resort ticketing operations backend system. This is NOT a code repository - it contains only Markdown documentation files describing the product design.

**Purpose**: Design and document the雪哒APP (Xueda APP) ski resort ticketing platform that:
- Aggregates 200+ ski resort ticketing systems through自我游(Ziwoyou) API
- Manages ski tickets, ski+hotel packages, and coaching courses
- Serves both C-end users (雪哒APP) and B-end distributors

## Architecture

### Core System Flow
```
自我游票务系统 (3rd party supplier)
    ↓ Daily sync
票务库 (Ticket inventory with calendar-based stock)
    ↓ Apply templates
产品库 (Product catalog)
    ↓ Publish
雪哒APP (C-end) + 分销商 (B-end distributors)
    ↓ Purchase
订单管理 (Order management with sub-order splitting)
```

### Four-Template System
1. **雪场模板** (Ski Resort Template): Resort info, facilities, transportation
2. **酒店模板** (Hotel Template): Hotel info, room types, check-in/out times
3. **分类模板** (Category Template): Product structure and default content for ticket types
4. **商家模板** (Merchant Template): Global雪哒APP information

## Critical Design Principles

### 1. Extreme Simplicity
- **Product names**: Directly use ticket inventory names (no complex generation/parsing)
- **Template configuration**: Only 5-8 essential fields per template
- **No complex logic**: No variable systems, no auto-extraction from ticket names

### 2. Template Reusability
- Templates configured once, applied to multiple products
- Template updates automatically sync to associated products
- Products can "detach" from templates for full customization

### 3. Calendar-Based Inventory (NOT product-level)
- Inventory stored at ticket level per date (daily calendar)
- Products reference ticket inventory in real-time
- Synced from自我游every 10 minutes

### 4. Order Splitting for Consecutive Purchases
- User buys 5 consecutive days → 1 main order + 5 sub-orders
- Each day has independent verification code
- Supports partial refunds (unused days only)

## Key Technical Concepts

### Inventory Management
```sql
-- Inventory is stored per ticket SKU per date
ticket_inventory (
    ticket_sku,
    date,           -- Each date has separate inventory
    total_stock,
    sold_count,
    available       -- Calculated: total - sold
)

-- Products do NOT store inventory
-- They reference ticket_inventory in real-time
```

### Order Structure
```
Main Order #2025010700001 (¥1,600 for 5 days)
├─ Sub-order 1: 2025-01-15, ¥320, verify_code: 8756 4321
├─ Sub-order 2: 2025-01-16, ¥320, verify_code: 8756 4322
├─ Sub-order 3: 2025-01-17, ¥320, verify_code: 8756 4323
├─ Sub-order 4: 2025-01-18, ¥320, verify_code: 8756 4324
└─ Sub-order 5: 2025-01-19, ¥320, verify_code: 8756 4325
```

## Documentation Files

### Essential Reading Order (for understanding the system)
1. `README.md` - Start here for document index
2. `库存与拆单逻辑.md` - Core inventory and order splitting logic
3. `模板中心-精简版.md` - Template system architecture
4. `产品完整流程-极简版.md` - Complete product lifecycle
5. `产品详情页设计.md` - C-end UI structure

### File Purposes
- `雪场票务运营后台低保真原型_1.md`: Original comprehensive design (85KB)
- `模板中心-精简版.md`: Simplified template center design
- `产品完整流程-极简版.md`: Product creation, maintenance, C-end display
- `库存与拆单逻辑.md`: Calendar inventory + order splitting + database schema
- `产品详情页设计.md`: C-end product detail page for 3 product types

## Product Types

1. **单次雪票** (Single Ski Ticket)
   - Templates: Ski Resort + Category + Merchant

2. **住滑套餐** (Ski+Hotel Package)
   - Templates: Ski Resort + Hotel + Category + Merchant
   - Inventory = min(hotel_inventory, ticket_inventory)

3. **教练培训** (Coaching Course)
   - Templates: Ski Resort + Category + Merchant

## Important Business Rules

### Product Creation (2 minutes per product)
```
Step 1: Select ticket from票务库
Step 2: Select templates (auto-matched by resort/hotel)
Step 3: Product name = ticket name (can be edited later)
Step 4: Set price, channels
Step 5: Publish
```

### Batch Creation (3 minutes for 10 products)
- Select multiple tickets
- Apply same template
- Unified pricing rule (e.g., cost + ¥40)
- Batch publish

### C-end Display Rules
- **Must show**: Ski resort info (facilities, transportation) for decision-making
- **Must show**: Hotel info for ski+hotel packages
- **Never show**: Supplier (自我游) - only show雪哒APP
- **Never show**: Internal ticket SKU

### Inventory Check Flow
1. Frontend: Real-time AJAX query when user selects date
2. Backend: Double-check on order creation
3. Payment: Lock inventory after successful payment
4. Sync: Release inventory on refund/cancellation

## Working with This Repository

### Editing Documentation
- All files are Markdown (.md)
- Use Chinese for content (this is a Chinese product)
- Follow the极简原则(extreme simplicity principle)
- When adding new features, update relevant docs:
  - Business flow → `产品完整流程-极简版.md`
  - Inventory/orders → `库存与拆单逻辑.md`
  - Templates → `模板中心-精简版.md`
  - C-end UI → `产品详情页设计.md`

### Design Philosophy
- **Avoid complexity**: If it requires parsing, auto-generation, or complex mapping → it's too complex
- **Operator control**: Give operations team full control, don't over-automate
- **Template inheritance**: Use templates for consistency, allow detachment for flexibility
- **Calendar-first**: Always think "inventory per date" not "total inventory"

### Common Misconceptions to Avoid
1. ❌ Product has its own inventory → ✅ Product references ticket's calendar inventory
2. ❌ Auto-generate product titles with variables → ✅ Use ticket name directly, edit manually
3. ❌ One order for 5 consecutive days → ✅ 1 main order + 5 sub-orders
4. ❌ Show supplier to users → ✅ Only show雪哒APP merchant info

## Version History
- Initial design: 2025-01-07
- Simplified optimization: 2025-01-08
- Inventory/splitting refinement: 2025-01-08
