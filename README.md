# Garden Cafe ERP System (with BOM)

A custom-built ERP web application for the Midterm Group Practical Exam. Built with
**Node.js + Express** (backend/API) and a **vanilla HTML/CSS/JS** frontend — no build
step required. Data is stored in a local JSON file (`db/data.json`), which acts as a
lightweight database so the whole project runs with zero external DB setup.

## How to Run

```bash
npm install
npm start
```

Then open **http://localhost:3000** in your browser.

That's it — `npm install` pulls in the only two dependencies (`express`, `cors`), and
`npm start` runs `server.js`. On first run, it auto-creates `db/data.json` from
`db/seed.json` (seed data for items/BOM/warehouse — **customers and suppliers start
empty** so your group can input your own).

## What's Already Seeded vs. What You Add

| Data | Status |
|---|---|
| Company Setup | Stubbed with "Garden Cafe" placeholder info — edit in the **Company** tab |
| Warehouse | 1 pre-created (Main Store) |
| Raw Materials (7) | Pre-seeded from your supplier purchase list (beans, syrup, sugar, cups, lids) |
| Finished Product (1) | "Iced Americano" with a working BOM already defined, so the BOM math is demonstrable immediately |
| **Customers** | **Empty — add your own in the Customers tab (need 5+)** |
| **Suppliers** | **Empty — add your own in the Suppliers tab (need 3+)** |

If you ever want to wipe your entered data and start over from the seed, send a
`POST` request to `/api/reset-demo` (e.g. via Postman or `curl -X POST
http://localhost:3000/api/reset-demo`).

## How This Maps to the Exam Requirements

- **Master Data**: Company Setup, Customers, Suppliers, Items (raw + finished),
  Warehouse — all editable via the UI, all with unique auto-generated IDs.
- **BOM**: `routes/bom.js` computes `Extended Cost = Qty × Unit Cost` and
  `Total BOM Cost = Σ Extended Costs` live, per finished product. Add more BOM lines
  or finished products anytime from the **BOM** tab.
- **Purchasing → Goods Receipt**: Create a PO against a supplier, Approve it, then
  Receive Goods — raw material stock increases and is logged in the movement log.
- **Production**: Creating a Production Order auto-calculates required components
  from the BOM × quantity. **Start** checks stock availability first (blocks with a
  shortage list if insufficient) and consumes raw materials. **Complete** adds
  finished-goods stock.
- **Sales → Delivery**: Create a Sales Order, Approve it, then Deliver — this checks
  finished-goods stock and blocks if insufficient, otherwise decreases stock.
- **Reports/Dashboard**: Inventory report, BOM/cost report, sales report, and a KPI
  dashboard (finished-goods stock, low-stock alerts, total sales, top product BOM
  cost). Inventory, BOM cost, and Sales reports are each exportable as CSV.
- **Controls**: unique auto-IDs, required-field validation, stock-availability
  checks before production/delivery, status fields (Draft/Approved/Received/In
  Production/Completed/Delivered), and a full inventory movement log with
  timestamps and reference documents (basic audit trail).

## Project Structure

```
garden-cafe-erp/
  server.js              # Express app entry point
  package.json
  db/
    seed.json            # Initial data (items, BOM, warehouse — no customers/suppliers)
    database.js           # Simple JSON-file data layer (auto-creates data.json)
    inventoryHelper.js    # Shared stock-movement logger
  routes/
    company.js, customers.js, suppliers.js, items.js, bom.js,
    purchaseOrders.js, goodsReceipts.js, productionOrders.js,
    salesOrders.js, deliveries.js, inventory.js, reports.js
  public/
    index.html, style.css, app.js   # Frontend (tabs for each module)
```

## Demonstrating the Full End-to-End Flow (for your live defense)

1. **Suppliers** tab → add a supplier (or use one you've entered)
2. **Purchasing** tab → create a PO for a raw material → Approve → Receive Goods
   (watch stock increase in the **Inventory** tab)
3. **Production** tab → create a Production Order for Iced Americano → Start
   (consumes raw materials) → Complete (adds finished-goods stock)
4. **Customers** tab → add a customer
5. **Sales** tab → create a Sales Order → Approve → Deliver (watch finished-goods
   stock decrease)
6. **Reports** tab → show updated inventory, BOM cost, and sales figures reflecting
   every step above

## Notes for Your Group

- The BOM for "Iced Americano" uses fractional quantities per cup (e.g. 0.02 of a
  1kg bean bag) since the raw materials are purchased in bulk units (bags, packs).
  Feel free to adjust these quantities in the **BOM** tab to match your actual
  recipe, or add more finished products with their own BOMs (the system supports
  multiple 1- or 2-level BOMs).
- Company Setup is currently a placeholder — update it once your business
  registration/setup details are finalized (per your earlier note that this is
  still pending).
