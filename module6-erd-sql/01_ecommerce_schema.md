# M6-01: E-commerce Schema Design

## Normalization Recap

| NF | Rule | Test | Pain if violated |
|----|------|------|------------------|
| 1NF | Every cell = one atomic value | No comma-separated lists, no repeating groups | Can't query individual values |
| 2NF | Every non-key column depends on the **whole** PK | Only matters for composite PKs — does column X need ALL parts of the key? | Same data in many rows → update anomalies |
| 3NF | No non-key column depends on another non-key column | Is there a chain PK → A → B? If so, B should be in A's table | Transitive duplication → inconsistent data |

**One sentence:** Each fact lives in exactly ONE place.
**Mnemonic:** "The key, the whole key, and nothing but the key."

---

## ERD

```
users (1) ──→ (N) orders (1) ──→ (N) order_items (N) ←── (1) products
  │                  │                                        │
  │ 1:N              │ 1:1                                    │ 1:N
  ▼                  ▼                                        ▼
cart_items         payments                                 reviews ←── users (1:N)
                                                              │
                                              products (N) ←── (1) categories (self-ref)
```

### Tables

**users** — id (PK), email (UQ), name, phone, role (buyer/seller), created_at

**categories** — id (PK), name, parent_id (FK→self) — tree structure: Electronics > Phones > iPhone

**products** — id (PK), seller_id (FK→users), category_id (FK), name, description, price, stock, created_at

**cart_items** — id (PK), user_id (FK), product_id (FK), quantity, added_at — UQ(user_id, product_id)

**orders** — id (PK), user_id (FK), status (pending/paid/shipped/delivered/cancelled), total_amount, shipping_addr, created_at

**order_items** — id (PK), order_id (FK), product_id (FK), quantity, unit_price (snapshot of price at purchase time)

**payments** — id (PK), order_id (FK, UQ → 1:1), method (card/bank/cod), status (pending/success/failed), amount, paid_at

**reviews** — id (PK), user_id (FK), product_id (FK), rating (1-5), comment, created_at — UQ(user_id, product_id)

---

## Key Design Decisions

### unit_price in order_items (intentional denormalization)
Products change price over time. Order history must show price at time of purchase, not current price. `unit_price` = snapshot = đơn giá.

### categories.parent_id (self-referencing FK)
Creates unlimited-depth tree in one table. Alternative: closure table or materialized path — more complex, needed only for deep hierarchies with frequent subtree queries.

### payments separate from orders (1:1)
Different lifecycle — order exists before payment. Own status flow. Easy to extend to 1:N (multiple payment attempts) by removing UQ constraint.

### cart_items in DB (not just localStorage)
Persists across devices. Enables abandoned cart reminders. Analytics on cart-but-not-purchased. Production pattern: DB for logged-in users + localStorage for guests, merge on login.

### total_amount cached in orders
Avoids JOINing order_items on every read. Calculated once at checkout, never modified. For refunds, create a separate record.

---

## Interview Questions

### Q: 1000 people buy the last item — how to prevent overselling?
Atomic conditional update (optimistic locking):
```sql
UPDATE products SET stock = stock - 1 WHERE id = ? AND stock > 0;
-- Check affected rows: 0 = out of stock, 1 = success
```
Single SQL statement = atomic. Postgres guarantees only one transaction wins on same row.

### Q: Why store total_amount if you can compute it?
Performance. 10M orders × 5 items = 50M rows to scan for a simple order list. Cached total = single table scan, no JOIN.

### Q: How to add product variants (size, color)?
Add `product_variants(id, product_id, sku, size, color, price, stock)`. `order_items` references `variant_id` instead of `product_id`.

### Q: One person is both buyer and seller?
Simple: `role ENUM('buyer', 'seller', 'both')`. Production: separate `sellers` table with extra fields (store_name, bank_account) linked 1:1 to users.
