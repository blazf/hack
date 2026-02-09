# MRP Webapp Implementation Plan

## Overview
Build a self-contained MRP (Material Requirements Planning) webapp with:
- Master data and transaction data management (stored in localStorage)
- MRP engine to generate work orders and purchase orders
- Gantt chart for production scheduling visualization
- Material level charts over time
- Initialize with sample data from `./data/` on first load

## Project Structure
```
/src
├── /components
│   ├── /ui              # shadcn/ui components
│   ├── /layout          # AppShell, Sidebar, Header
│   ├── /master-data     # Items, BOM, WorkCenters, etc. list/form components
│   ├── /transactions    # CustomerOrders, WorkOrders, Inventory, PurchaseOrders
│   ├── /mrp             # MRPRunner, MRPResults, PlannedOrdersList
│   └── /visualization   # GanttChart, MaterialLevelChart, PurchaseTimeline
├── /store               # Zustand stores (masterData, transactions, mrp)
├── /mrp                 # MRP engine (bom-explosion, netting, lot-sizing, scheduling)
├── /types               # TypeScript interfaces matching JSON structure
├── /lib                 # Utilities
└── /pages               # Dashboard, MasterData, Transactions, MRP, Visualization
/public/data             # Copy of sample JSON files for initialization
```

## Implementation Steps

### Phase 1: Project Setup
1. Create Vite + React + TypeScript project
2. Install dependencies: `tailwindcss`, `@radix-ui/*`, `zustand`, `echarts`, `recharts`, `date-fns`, `zod`, `react-router-dom`
3. Configure Tailwind and shadcn/ui
4. Copy `./data/` to `public/data/` for initialization source

### Phase 2: Data Layer
1. Define TypeScript types matching existing JSON schemas:
   - Master: `Item`, `BOMEntry`, `WorkCenter`, `Routing`, `Warehouse`, `Supplier`, `Customer`, `SupplierCatalogEntry`
   - Transactions: `CustomerOrder`, `WorkOrder`, `InventoryRecord`, `PurchaseOrder`
2. Create Zustand stores with localStorage persistence middleware
3. Implement initialization logic: check localStorage, if empty fetch from `/public/data/` and populate
4. CRUD operations for all entities

### Phase 3: UI - Layout & Master Data
1. Build AppShell with collapsible sidebar navigation
2. Create reusable DataTable component (sorting, filtering, pagination)
3. Implement list views for
 - all master data entities: items, BOW, work centers, routings.
 - transactions: customer orders, work orders, inventory, purchase orders.
4. Implement create/edit forms with Zod validation
5. Add delete with referential integrity checks

### Phase 4: UI - Transactions
1. Customer Orders list with status badges, line items display
2. Work Orders list linked to source orders
3. Inventory view with available quantity (on_hand - allocated)
4. Purchase Orders with receipt tracking
5. Status transition actions on orders

### Phase 5: MRP Engine
1. **BOM Explosion**: Build BOM tree, calculate low-level codes, recursive explosion with scrap factors
2. **Netting**: Gross requirements from demand, net against inventory and scheduled receipts
3. **Lot Sizing**: Apply lot_size from items, min_order_quantity from supplier catalog
4. **Scheduling**: Backward schedule using lead_time_days, calculate operation times from routings
5. **Capacity Planning**: Load work centers, identify overloads
6. **Output**: Generate planned work orders (manufacturable items) and planned purchase orders (purchasable items)

### Phase 6: MRP UI
1. MRP configuration panel (planning horizon date picker)
2. Run MRP button with results display
3. Planned orders list with "Confirm" action to convert to actual orders
4. Exceptions list (capacity overload, material shortages, late orders)

### Phase 7: Visualizations
1. **Gantt Chart**: Y-axis = work centers, X-axis = timeline, bars = work orders colored by status
2. **Purchase Timeline**: Timeline showing PO expected delivery dates
3. **Material Level Chart**: Line chart of projected inventory over time per item, with safety stock line

### Phase 8: Polish
1. Data export/import (JSON download/upload for backup)
2. Reset to sample data option
3. Error handling and loading states
4. Responsive design adjustments

## Critical Files to Reference
- `items.json` - Item types, lead times, lot sizes, purchasable/manufacturable flags
- `bom.json` - Parent-child relationships, quantities, scrap factors
- `routings.json` - Operations per item with work center and time data
- `work_centers.json` - Capacity and efficiency factors
- `customer_orders.json` - Demand source structure
- `inventory.json` - Stock levels and allocation structure

## MRP Algorithm Summary
```
1. Collect demand from customer orders (pending/confirmed/in_production)
2. Process items level-by-level (low-level code order)
3. For each item/date:
   - Gross requirement = customer demand + dependent demand from parent items
   - Scheduled receipts = open WOs + open POs
   - Projected available = prior balance + receipts - gross requirement
   - If projected < safety_stock: create net requirement
   - Apply lot sizing, offset by lead time
   - If manufacturable: create planned WO, explode BOM for child demand
   - If purchasable: create planned PO
4. Load work centers with operation times from routings
5. Report exceptions for capacity/material issues
```
