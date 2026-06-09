# 🗄️ GameHub - Database Deep Dive & SQL Queries

---

## 📊 Entity Relationship Diagram (ERD)

```
┌──────────────┐
│  PELANGGAN   │
│──────────────│
│ id_pelanggan │◄──┐
│ nama         │   │
│ email        │   │ 1:N
│ username     │   │
│ password     │   │
│ saldo        │   │
│ tanggal_df   │   │
└──────────────┘   │
                   │
                   ├─────────────────────────────────────┐
                   │                                     │
                   ▼                                     │
          ┌─────────────────┐                          │
          │   TRANSAKSI     │                          │
          │─────────────────│                          │
          │ id_transaksi    │                          │
          │ id_pelanggan*   │◄─────────────────────────┘
          │ tanggal_tx      │
          │ total_bayar     │
          └────────┬────────┘
                   │
                   │ 1:N
                   ▼
        ┌──────────────────────┐
        │ DETAIL_TRANSAKSI     │
        │──────────────────────│
        │ id_detail            │
        │ id_transaksi*        │◄────────────────┐
        │ id_game*             │◄─────────────┐  │
        │ id_dlc*              │◄──────────┐  │  │
        │ id_bundle*           │◄────┐     │  │  │
        │ harga_jual           │     │     │  │  │
        └──────────────────────┘     │     │  │  │
                                     │     │  │  │
          ┌──────────────────────┐   │     │  │  │
          │       BUNDLE         │   │     │  │  │
          │──────────────────────│   │     │  │  │
          │ id_bundle            │◄──┘     │  │  │
          │ nama_bundle          │         │  │  │
          │ harga_bundle         │         │  │  │
          │ deskripsi            │         │  │  │
          │ gambar               │         │  │  │
          └──────────┬───────────┘         │  │  │
                     │                     │  │  │
                     │ N:N                 │  │  │
                     ▼                     │  │  │
        ┌──────────────────────┐           │  │  │
        │  BUNDLE_ITEMS        │           │  │  │
        │──────────────────────│           │  │  │
        │ id_bundle*           │───────────┼──┘  │
        │ id_game*             │───────────┼─────┘
        │ id_dlc*              │───────────┘
        └──────────────────────┘
                   │
            ┌──────┴──────┐
            ▼             ▼
     ┌──────────────┐  ┌─────────────┐
     │    GAME      │  │    DLC      │
     │──────────────│  │─────────────│
     │ id_game      │  │ id_dlc      │
     │ judul        │  │ id_game*    │
     │ harga_utama  │  │ nama_dlc    │
     │ genre        │  │ harga_dlc   │
     │ gambar       │  │ tipe (enum) │
     └──────┬───────┘  │ gambar      │
            │          └─────────────┘
            │
            │ N:N
            ▼
     ┌──────────────────┐
     │  GAME_PLATFORM   │
     │──────────────────│
     │ id_game_platform │
     │ id_game*         │
     │ id_platform*     │
     └──────────────────┘
            │
            ▼
     ┌──────────────┐
     │  PLATFORM    │
     │──────────────│
     │ id_platform  │
     │ nama_platform│
     └──────────────┘
```

---

## 🔑 Primary & Foreign Keys

### **Primary Keys (PK):**
```
pelanggan          → id_pelanggan
game               → id_game
dlc                → Id_dlc (catatan: capital D!)
bundle             → id_bundle
bundle_items       → (composite) id_bundle + id_game + id_dlc
platform           → id_platform
game_platform      → id_game_platform
transaksi          → id_transaksi
detail_transaksi   → id_detail
```

### **Foreign Keys (FK):**
```
bundle_items    → id_bundle     → bundle.id_bundle
                → id_game       → game.id_game
                → id_dlc        → dlc.Id_dlc

dlc             → id_game       → game.id_game

game_platform   → id_game       → game.id_game
                → id_platform   → platform.id_platform

transaksi       → id_pelanggan  → pelanggan.id_pelanggan

detail_transaksi→ id_transaksi  → transaksi.id_transaksi
                → id_game       → game.id_game
                → id_dlc        → dlc.Id_dlc
                → id_bundle     → bundle.id_bundle
```

---

## 📝 SQL Query Examples

### **1. SELECT Queries**

#### **A. Ambil semua games dengan detail platformnya**
```sql
SELECT 
    g.id_game,
    g.judul,
    g.harga_utama,
    g.genre,
    GROUP_CONCAT(p.nama_platform SEPARATOR ', ') as platforms
FROM game g
LEFT JOIN game_platform gp ON g.id_game = gp.id_game
LEFT JOIN platform p ON gp.id_platform = p.id_platform
GROUP BY g.id_game
ORDER BY g.judul ASC;
```

**Output:**
```
id_game | judul                    | harga_utama | genre | platforms
--------|--------------------------|-------------|-------|---------------------------
1       | Persona 5 Royal          | 529000      | JRPG  | PC, Playstation, XBOX
2       | Persona 3 Reload         | 849000      | JRPG  | (NULL)
5       | Cyberpunk 2077           | 699999      | (NULL)| (NULL)
10      | Digimon Story Time Strg  | 899999      | (NULL)| PC, Playstation, XBOX
```

---

#### **B. Ambil semua DLC dengan game asalnya**
```sql
SELECT 
    d.Id_dlc,
    d.nama_dlc,
    d.harga_dlc,
    d.tipe,
    g.judul as game_utama
FROM dlc d
LEFT JOIN game g ON d.id_game = g.id_game
ORDER BY d.id_game, d.nama_dlc;
```

**Output:**
```
Id_dlc | nama_dlc            | harga_dlc | tipe      | game_utama
-------|---------------------|-----------|-----------|--------------------
1      | Joker Outfit        | 159000    | Kosmetik  | Persona 3 Reload
2      | Cyberpunk Expansion | 299000    | Ekspansi  | Cyberpunk 2077
3      | Alternate Dimension | 199000    | Ekspansi  | Digimon Story...
4      | Story Expansion     | 200000    | Ekspansi  | Persona 3 Reload
```

---

#### **C. Ambil detail bundle lengkap**
```sql
SELECT 
    b.id_bundle,
    b.nama_bundle,
    b.harga_bundle,
    b.deskripsi,
    GROUP_CONCAT(
        COALESCE(g.judul, d.nama_dlc) 
        SEPARATOR ' + '
    ) as items_dalam_bundle
FROM bundle b
LEFT JOIN bundle_items bi ON b.id_bundle = bi.id_bundle
LEFT JOIN game g ON bi.id_game = g.id_game
LEFT JOIN dlc d ON bi.id_dlc = d.Id_dlc
GROUP BY b.id_bundle
ORDER BY b.id_bundle;
```

**Output:**
```
id_bundle | nama_bundle              | harga_bundle | deskripsi       | items
----------|--------------------------|--------------|-----------------|-----
1         | Persona Paradise         | 1199000      | Bundle Persona  | Pers5 + Pers3
2         | Persona 3 Joker          | 900000       | Joker in Pers3  | Pers3 + Joker
3         | Cyberpunk Ultimate Bundle| 899000       | CP+Expansion    | CP2077 + Exp
4         | Digimon Complete Bundle  | 999000       | Complete        | Digimon + Exp
```

---

#### **D. Transaksi pelanggan tertentu**
```sql
SELECT 
    t.id_transaksi,
    t.tanggal_transaksi,
    t.total_bayar,
    COUNT(dt.id_detail) as jumlah_item,
    GROUP_CONCAT(
        COALESCE(g.judul, d.nama_dlc, b.nama_bundle)
        SEPARATOR ', '
    ) as items
FROM transaksi t
LEFT JOIN detail_transaksi dt ON t.id_transaksi = dt.id_transaksi
LEFT JOIN game g ON dt.id_game = g.id_game
LEFT JOIN dlc d ON dt.id_dlc = d.Id_dlc
LEFT JOIN bundle b ON dt.id_bundle = b.id_bundle
WHERE t.id_pelanggan = 1
GROUP BY t.id_transaksi
ORDER BY t.tanggal_transaksi DESC;
```

**Output:**
```
id_tx | tanggal_transaksi    | total  | jml | items
------|----------------------|--------|-----|------
11    | 2026-06-07 20:42:39  | 159000 | 1   | Joker Outfit
10    | 2026-06-06 15:38:13  | 159000 | 1   | Joker Outfit
9     | 2026-06-06 15:36:30  | 159000 | 1   | Joker Outfit
...
```

---

#### **E. Dashboard Statistics**
```sql
SELECT 
    COUNT(DISTINCT p.id_pelanggan) as total_customers,
    COUNT(DISTINCT t.id_transaksi) as total_transactions,
    COALESCE(SUM(t.total_bayar), 0) as total_revenue,
    COALESCE(AVG(t.total_bayar), 0) as avg_transaction,
    MAX(t.tanggal_transaksi) as last_transaction
FROM pelanggan p
FULL OUTER JOIN transaksi t ON p.id_pelanggan = t.id_pelanggan;
```

**Output:**
```
total_customers | total_transactions | total_revenue | avg_transaction | last_transaction
----------------|-------------------|---------------|-----------------|------------------
2               | 11                | 17951000      | 1631909         | 2026-06-07 20:42
```

---

#### **F. Top-selling items**
```sql
SELECT 
    COALESCE(g.judul, d.nama_dlc, b.nama_bundle) as item_name,
    CASE 
        WHEN g.id_game IS NOT NULL THEN 'Game'
        WHEN d.Id_dlc IS NOT NULL THEN 'DLC'
        WHEN b.id_bundle IS NOT NULL THEN 'Bundle'
    END as tipe,
    COUNT(*) as times_sold,
    SUM(dt.harga_jual) as total_revenue
FROM detail_transaksi dt
LEFT JOIN game g ON dt.id_game = g.id_game
LEFT JOIN dlc d ON dt.id_dlc = d.Id_dlc
LEFT JOIN bundle b ON dt.id_bundle = b.id_bundle
GROUP BY dt.id_game, dt.id_dlc, dt.id_bundle
ORDER BY times_sold DESC;
```

**Output:**
```
item_name            | tipe   | times_sold | total_revenue
---------------------|--------|------------|----------------
Persona Paradise     | Bundle | 5          | 5995000
Persona 3 Joker      | Bundle | 4          | 3600000
Joker Outfit         | DLC    | 3          | 477000
```

---

### **2. INSERT Queries**

#### **A. User Registration**
```sql
INSERT INTO pelanggan (id_pelanggan, nama, email, username, password, saldo, tanggal_daftar)
VALUES (
    NULL,  -- auto increment
    'John Doe',
    'john@example.com',
    'johndoe',
    'hashed_password_or_plaintext',
    0,  -- default saldo = 0
    NOW()
);
```

---

#### **B. Create Transaction (Checkout Process)**
```sql
-- Step 1: Deduct balance
UPDATE pelanggan 
SET saldo = saldo - 2099000
WHERE id_pelanggan = 1;

-- Step 2: Create transaction record
INSERT INTO transaksi (id_transaksi, id_pelanggan, tanggal_transaksi, total_bayar)
VALUES (NULL, 1, NOW(), 2099000);

-- Step 3: Get created transaction ID
SET @id_transaksi = LAST_INSERT_ID();

-- Step 4: Insert transaction details (for each item)
INSERT INTO detail_transaksi (id_detail, id_transaksi, id_game, id_dlc, id_bundle, harga_jual)
VALUES 
    (NULL, @id_transaksi, NULL, NULL, 1, 1199000),  -- Bundle 1
    (NULL, @id_transaksi, NULL, NULL, 2, 900000);   -- Bundle 2
```

**Better approach (using Transaction):**
```sql
START TRANSACTION;

UPDATE pelanggan 
SET saldo = saldo - 2099000
WHERE id_pelanggan = 1;

INSERT INTO transaksi (id_transaksi, id_pelanggan, tanggal_transaksi, total_bayar)
VALUES (NULL, 1, NOW(), 2099000);

SET @id_transaksi = LAST_INSERT_ID();

INSERT INTO detail_transaksi (id_detail, id_transaksi, id_game, id_dlc, id_bundle, harga_jual)
VALUES 
    (NULL, @id_transaksi, NULL, NULL, 1, 1199000),
    (NULL, @id_transaksi, NULL, NULL, 2, 900000);

COMMIT;
```

---

### **3. UPDATE Queries**

#### **A. Update customer balance**
```sql
UPDATE pelanggan
SET saldo = saldo - 1000000
WHERE id_pelanggan = 1;
```

---

#### **B. Update game price**
```sql
UPDATE game
SET harga_utama = 599000
WHERE id_game = 1;
```

---

### **4. DELETE Queries**

#### **A. Delete transaction (Cascade)**
```sql
-- Hapus detail transaksi dulu (FK constraint)
DELETE FROM detail_transaksi
WHERE id_transaksi = 1;

-- Kemudian hapus transaksi
DELETE FROM transaksi
WHERE id_transaksi = 1;
```

---

## 📊 Index Analysis

### **Current Indexes:**
```sql
-- Pelanggan
ALTER TABLE pelanggan ADD PRIMARY KEY (id_pelanggan);

-- Game
ALTER TABLE game ADD PRIMARY KEY (id_game);

-- DLC
ALTER TABLE dlc ADD PRIMARY KEY (Id_dlc);
ALTER TABLE dlc ADD KEY (id_game);

-- Bundle
ALTER TABLE bundle ADD PRIMARY KEY (id_bundle);

-- Bundle Items
ALTER TABLE bundle_items 
    ADD KEY (id_bundle),
    ADD KEY (id_game),
    ADD KEY (id_dlc);

-- Platform
ALTER TABLE platform ADD PRIMARY KEY (id_platform);

-- Game Platform
ALTER TABLE game_platform
    ADD PRIMARY KEY (id_game_platform),
    ADD KEY (id_game),
    ADD KEY (id_platform);

-- Transaksi
ALTER TABLE transaksi
    ADD PRIMARY KEY (id_transaksi),
    ADD KEY (id_pelanggan);

-- Detail Transaksi
ALTER TABLE detail_transaksi
    ADD PRIMARY KEY (id_detail),
    ADD KEY (id_transaksi),
    ADD KEY (id_game),
    ADD KEY (id_dlc),
    ADD KEY (id_bundle);
```

### **Recommended Additional Indexes (for performance):**
```sql
-- For faster lookups by username
ALTER TABLE pelanggan ADD UNIQUE KEY (username);
ALTER TABLE pelanggan ADD INDEX (email);

-- For date range queries
ALTER TABLE transaksi ADD INDEX idx_tanggal (tanggal_transaksi);

-- For customer transaction queries
ALTER TABLE transaksi ADD INDEX idx_pelanggan_tanggal (id_pelanggan, tanggal_transaksi);

-- For DLC lookups by game
ALTER TABLE dlc ADD INDEX idx_game_dlc (id_game, Id_dlc);
```

---

## 🔒 Referential Integrity & Constraints

### **Foreign Key Constraints:**
```sql
-- Bundle Items → Bundle, Game, DLC
ALTER TABLE bundle_items
ADD CONSTRAINT bundle_items_ibfk_1 
    FOREIGN KEY (id_bundle) REFERENCES bundle(id_bundle);
ADD CONSTRAINT bundle_items_ibfk_2 
    FOREIGN KEY (id_game) REFERENCES game(id_game);
ADD CONSTRAINT bundle_items_ibfk_3 
    FOREIGN KEY (id_dlc) REFERENCES dlc(Id_dlc);

-- DLC → Game
ALTER TABLE dlc
ADD CONSTRAINT dlc_ibfk_1 
    FOREIGN KEY (id_game) REFERENCES game(id_game);

-- Game Platform → Game, Platform
ALTER TABLE game_platform
ADD CONSTRAINT game_platform_ibfk_1 
    FOREIGN KEY (id_game) REFERENCES game(id_game);
ADD CONSTRAINT game_platform_ibfk_2 
    FOREIGN KEY (id_platform) REFERENCES platform(id_platform);

-- Transaksi → Pelanggan
ALTER TABLE transaksi
ADD CONSTRAINT transaksi_ibfk_1 
    FOREIGN KEY (id_pelanggan) REFERENCES pelanggan(id_pelanggan);

-- Detail Transaksi → Transaksi, Game, DLC, Bundle
ALTER TABLE detail_transaksi
ADD CONSTRAINT detail_transaksi_ibfk_1 
    FOREIGN KEY (id_transaksi) REFERENCES transaksi(id_transaksi);
ADD CONSTRAINT detail_transaksi_ibfk_2 
    FOREIGN KEY (id_game) REFERENCES game(id_game);
ADD CONSTRAINT detail_transaksi_ibfk_3 
    FOREIGN KEY (id_dlc) REFERENCES dlc(Id_dlc);
ADD CONSTRAINT detail_transaksi_ibfk_4 
    FOREIGN KEY (id_bundle) REFERENCES bundle(id_bundle);
```

---

## 💾 Data Integrity Examples

### **A. Cascade Delete Prevention**
```sql
-- Jika kita hapus bundle, apa yang terjadi?
DELETE FROM bundle WHERE id_bundle = 1;

-- Error (jika ada FK constraint):
-- Cannot delete or update a parent row: 
-- a foreign key constraint fails 
-- (`gamehub`.`bundle_items`, CONSTRAINT `bundle_items_ibfk_1`)

-- Solution: Soft delete (mark as inactive)
ALTER TABLE bundle ADD COLUMN is_active BOOLEAN DEFAULT 1;
UPDATE bundle SET is_active = 0 WHERE id_bundle = 1;
```

---

### **B. Transaction Consistency**
```sql
-- Simulasi: User membeli Rp 15.000.000 tapi saldo hanya Rp 10.000.000

START TRANSACTION;

SELECT saldo FROM pelanggan WHERE id_pelanggan = 1;
-- Result: 10000000

-- Check if balance sufficient
IF 10000000 >= 15000000:
    ROLLBACK;
    -- Output: Saldo tidak cukup
ELSE:
    UPDATE pelanggan SET saldo = 10000000 - 15000000;
    INSERT INTO transaksi ...
    INSERT INTO detail_transaksi ...
    COMMIT;
    -- Output: Success
```

---

## 📈 Query Performance Tips

### **1. Use EXPLAIN to analyze queries:**
```sql
EXPLAIN SELECT * FROM game 
INNER JOIN game_platform ON game.id_game = game_platform.id_game
WHERE game_platform.id_platform = 1;

-- Output analysis:
-- - Check if it uses index
-- - Check rows scanned
-- - Optimize if needed
```

---

### **2. Avoid N+1 Query Problem:**

❌ **Bad (N+1 queries):**
```php
// Query 1: Get all games
$games = $pdo->query("SELECT * FROM game");

// Query N: For each game, get platforms (N queries!)
foreach ($games as $game) {
    $stmt = $pdo->prepare("SELECT * FROM platform 
        WHERE id_platform IN (
            SELECT id_platform FROM game_platform 
            WHERE id_game = ?
        )");
    $stmt->execute([$game['id_game']]);
}
```

✅ **Good (1 query with JOIN):**
```php
$query = "
    SELECT g.*, p.nama_platform
    FROM game g
    LEFT JOIN game_platform gp ON g.id_game = gp.id_game
    LEFT JOIN platform p ON gp.id_platform = p.id_platform
";
$stmt = $pdo->query($query);
```

---

## 🎓 Learning Points

### **Concepts Demonstrated:**
1. **Normalization**: Database normalized to 3NF
2. **Relationships**: 1:1, 1:N, N:N relationships
3. **Foreign Keys**: Referential integrity
4. **Transactions**: ACID properties
5. **Aggregation**: GROUP BY, SUM, COUNT, AVG
6. **JOINs**: INNER JOIN, LEFT JOIN, GROUP_CONCAT
7. **Subqueries**: Nested queries for complex logic
8. **Indexing**: Performance optimization
9. **Data Integrity**: Constraints & validation
10. **Real-world scenario**: E-commerce transaction flow

---

**Database Design for E-Commerce Platform** | GameHub Project | 2026
