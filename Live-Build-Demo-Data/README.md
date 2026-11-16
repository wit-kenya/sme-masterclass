# Live build demo data

Ten CSV files, one for each Airtable table on the data map. The field names match the map. All names, phone numbers and emails are made up.

| File | Rows |
|---|---|
| 01_suppliers.csv | 10 |
| 02_customers.csv | 10 |
| 03_products.csv | 20 |
| 04_promotions.csv | 3 |
| 05_sales_orders.csv | 11 |
| 06_sales_order_lines.csv | 42 |
| 07_purchase_orders.csv | 3 |
| 08_purchase_order_lines.csv | 6 |
| 09_stock_movements.csv | 62 |
| 10_cash_book.csv | 16 |

## 1. Import in number order

A link field can only match rows in a table that already exists. Import `01` first and `10` last.

For each file: in the base, click **+ Add or import → CSV file**. Then change the field types below. Fields that are not listed stay as text.

| Table | Field → type |
|---|---|
| Suppliers | Phone → Phone number · Email → Email · Lead Time (days) → Number |
| Customers | Phone → Phone number · Email → Email · Type → Single select |
| Products | Category → Single select · Cost Price, Selling Price → Currency (KES) · Reorder Level → Number · Supplier → Link to Suppliers · Active → Checkbox |
| Promotions | Products → Link to Products (allow many) · Discount % → Percent · Start Date, End Date → Date · Message → Long text |
| Sales Orders | Customer → Link to Customers · Order Date → Date · Channel, Status → Single select · Notes → Long text |
| Sales Order Lines | Sales Order → Link to Sales Orders · Product → Link to Products · Quantity → Number · Unit Price → Currency · Discount % → Percent |
| Purchase Orders | Supplier → Link to Suppliers · Order Date, Expected Date → Date · Status → Single select |
| Purchase Order Lines | Purchase Order → Link to Purchase Orders · Product → Link to Products · Quantity → Number · Unit Cost → Currency |
| Stock Movements | Product → Link to Products · Type → Single select · Quantity → Number (allow negative numbers) · Date → Date |
| Cash Book | Date → Date · Direction, Category, Method → Single select · Amount → Currency · Sales Order → Link to Sales Orders · Purchase Order → Link to Purchase Orders |

When you change a text field to a link, Airtable matches each value to the primary field of the other table. The names in the CSVs match exactly.

## 2. Add the calculated fields

Airtable adds the reverse link columns for you (for example, Products gets a "Stock Movements" column). The rollups below use those columns.

| Table | Field | Type | Setting |
|---|---|---|---|
| Products | Stock On Hand | Rollup | Stock Movements → Quantity → `SUM(values)` |
| Products | Stock Status | Formula | `IF({Stock On Hand} <= 0, "Out", IF({Stock On Hand} <= {Reorder Level}, "Low", "OK"))` |
| Sales Order Lines | Line Total | Formula | `{Quantity} * {Unit Price} * (1 - {Discount %})` |
| Sales Orders | Order Total | Rollup | Sales Order Lines → Line Total → `SUM(values)` |
| Sales Orders | Amount Paid | Rollup | Cash Book → Amount → `SUM(values)` |
| Sales Orders | Payment Status | Formula | `IF({Amount Paid} = 0, "Unpaid", IF({Amount Paid} >= {Order Total}, "Paid", "Part-paid"))` |
| Purchase Order Lines | Line Total | Formula | `{Quantity} * {Unit Cost}` |
| Purchase Orders | PO Total | Rollup | Purchase Order Lines → Line Total → `SUM(values)` |
| Purchase Orders | Amount Paid | Rollup | Cash Book → Amount → `SUM(values)` |
| Promotions | Live Discount | Formula | `IF(AND(IS_BEFORE({Start Date}, DATEADD(TODAY(), 1, 'days')), IS_AFTER({End Date}, DATEADD(TODAY(), -1, 'days'))), {Discount %}, 0)` |
| Products | Live Discount % | Rollup | Promotions → Live Discount → `MAX(values)` |
| Products | Price Today | Formula | `{Selling Price} * (1 - {Live Discount %})` |

| Sales Order Lines | Product Name | Formula | `ARRAYJOIN({Product})` |
| Sales Order Lines | Stock Now | Rollup | Product → Stock On Hand → `MAX(values)` |
| Sales Order Lines | Short Now | Formula | `IF({Quantity} > {Stock Now}, 1, 0)` |
| Sales Orders | Short Lines | Rollup | Sales Order Lines → Short Now → `SUM(values)` |
| Sales Orders | Balance | Formula | `{Order Total} - {Amount Paid}` |
| Sales Orders | Customer Name | Formula | `ARRAYJOIN({Customer})` |
| Purchase Order Lines | Product Name | Formula | `ARRAYJOIN({Product})` |
| Purchase Order Lines | PO Status | Rollup | Purchase Order → Status → `ARRAYJOIN(values)` |
| Purchase Order Lines | Open Qty | Formula | `IF(OR({PO Status} = 'Draft', {PO Status} = 'Sent'), {Quantity}, 0)` |
| Products | On Order | Rollup | Purchase Order Lines → Open Qty → `SUM(values)` |
| Products | Reorder Qty | Formula | `MAX(0, 2 * {Reorder Level} - {Stock On Hand})` |

These helper fields let the n8n workflows read answers (today's price, what is short, what to reorder, what is owed) without any code. Live Discount, Live Discount % and Price Today handle today's price without any date logic. Set the formatting of **Live Discount** and **Live Discount %** to Percent, and **Price Today** to Currency.

## 3. Check the numbers

If Airtable shows these values, the import is correct.

- **Products:** Sugar 1kg = 46 (Low). Cooking Oil 3L = 7 (Low). Butter 250g = 0 (Out). All other products are OK.
- **Price Today:** while Sugar Week runs (25 October to 7 November), Sugar 1kg = 157.5 and Sugar 2kg = 306. Every other product shows its Selling Price. After 7 November the sugars go back to full price by themselves.
- **Sales Orders:** SO-1005 is Part-paid (23,150 total, 15,000 paid). SO-1007 is Part-paid (12,635 total, 9,000 paid). SO-1008 and SO-1011 are Unpaid. All others are Paid, except SO-1009, which is Cancelled.
- **Purchase Orders:** PO-2002 has 6,800 still to pay. PO-2003 is Sent and not paid.
- **Cash Book, October:** money in 77,745. Money out 76,600.

If a number is wrong, check the field type first (Percent, Currency, negative numbers allowed).

## Stories in the data

Use these during the session.

- **A promotion drives demand.** Sugar Week started on 25 October. SO-1008 says "Nimeona offer ya sukari!" Sugar 1kg is now Low, so the low-stock workflow raises a purchase order to Greenbelt Sugar Supplies.
- **A price change.** Cooking Oil 1L went from 300 to 320 on 18 October. The SO-1003 line keeps 300. The SO-1008 line shows 320. This is why the order line copies the price.
- **Out of stock.** Butter 250g is 0 after 2 packets spoiled (an Adjustment row). SO-1009 was cancelled because of it. PO-2003 (Sent) brings 30 more on 1 November.
- **Part-payments.** Mama Akinyi Eatery pays "kidogo kidogo": two Cash Book rows on SO-1007.
- **A repeat customer.** Mama Wanjiku Kiosk ordered twice (SO-1001 and SO-1010).
- **An order that is not confirmed yet.** SO-1011 came from the chatbot and is New, so it has no stock movement yet. The rule: stock leaves when an order is Confirmed.
- **An inactive product.** Drinking Chocolate 200g is unticked. The agent must not offer it.
- **Every channel appears:** WhatsApp, Email, Chatbot and Walk-in.

## Before the session

- The phone numbers are made up, but they can belong to real people. Do not send WhatsApp messages or broadcasts to them. For live tests, replace one customer's phone and email with your own.
- The emails use `example.com`, which cannot receive mail.
