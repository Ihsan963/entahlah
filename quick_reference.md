# 🎮 GameHub - Demo Quick Reference

---

## 📌 Project Summary
**GameHub** = Game Store E-Commerce Platform (Database project)
- **Tech:** HTML + CSS + JavaScript (Frontend) | PHP (Backend) | MySQL (Database)
- **Users:** 2 test accounts available
- **Games:** 4 games, 4 DLCs, 4 bundles
- **Sample Transactions:** 11 recorded transactions

---

## 🔑 Test Account

**Account 1:**
```
Username: Jabond
Password: admin123
Balance: Rp 4.100.000 (was Rp 4.100.000)
```

**Account 2:**
```
Username: dhafin
Password: dhafin123
Balance: Rp 7.901.000 (after purchases)
```

---

## 🎯 Demo Flow (5 Minutes)

### **Minute 1-2: Show Frontend**
1. Open `http://localhost/gamehub`
2. Show login page → login as `Jabond`
3. Show dashboard with:
   - Games grid
   - DLC carousel (horizontal scroll)
   - Bundle cards
   - Statistics section
   - Transaction history
   - Customer list

### **Minute 2-3: Show Shopping Cart**
1. Click "Beli" on 1-2 items
2. Show cart counter update
3. Click cart icon → show cart.html
4. Demonstrate item removal
5. Show total price calculation

### **Minute 3-4: Explain Backend**
1. **Architecture:**
   - Frontend sends AJAX request
   - Backend PHP validates & processes
   - Database stores permanently
   
2. **API Endpoints:**
   - `/api/games/get_games.php` → Get all games
   - `/api/checkout/process_checkout.php` → Process payment
   - `/api/transactions/get_transactions.php` → Get history

3. **Database:**
   - 9 tables with relationships
   - Foreign keys ensure data integrity
   - Transactions for ACID compliance

### **Minute 4-5: Show Database**
1. Open phpMyAdmin
2. Show `pelanggan` table → customer data
3. Show `transaksi` table → transaction records
4. Show relationship between tables

---

## 📂 File Structure to Show

```
gamehub/
├── index.html              ← Main dashboard
├── login.html              ← Authentication
├── cart.html               ← Shopping cart
├── api/
│   ├── login.php           ← Login endpoint
│   ├── games/get_games.php ← Get all games
│   ├── checkout/           ← Payment processing
│   └── ...
├── js/
│   ├── app.js              ← Business logic
│   └── api.js              ← API calls
├── css/
│   └── style.css
├── config/
│   └── db.php              ← Database connection
└── gamehub db.sql          ← Database schema
```

---

## 💡 Key Concepts to Highlight

### **1. Frontend Architecture**
- **Single Page App:** No page reloads, AJAX calls
- **localStorage:** Cart data persists in browser
- **Session Management:** User data saved on login

**Show:** Open DevTools → Application → localStorage → Show cart data

### **2. REST API Design**
- **HTTP Methods:** GET (retrieve), POST (create)
- **Request/Response:** JSON format
- **Base URL:** `http://localhost/gamehub/api`

**Show:** Use cURL to test API
```bash
curl -X GET http://localhost/gamehub/api/games/get_games.php
```

### **3. Database Relationships**
```
pelanggan (Users)
  ↓ 1:N
transaksi (Transactions)
  ↓ 1:N
detail_transaksi (Transaction Items)
  ├→ game
  ├→ dlc
  └→ bundle
```

**Show:** ERD in database documentation

### **4. Transaction Processing**
```
User clicks Checkout
  ↓
Validate customer & saldo
  ↓
BEGIN TRANSACTION
  ├─ Deduct saldo
  ├─ Create transaksi record
  └─ Create detail_transaksi for each item
  ↓
COMMIT (success) or ROLLBACK (error)
```

---

## 🔍 Live Demo Checklist

- [ ] **1. Frontend Demo**
  - [ ] Login page responsive
  - [ ] Dashboard loads all products
  - [ ] Add item to cart works
  - [ ] Cart counter updates
  - [ ] Cart page shows items
  - [ ] Remove item works

- [ ] **2. Backend Demo**
  - [ ] Explain API endpoints
  - [ ] Show PHP code structure
  - [ ] Explain validation logic
  - [ ] Show database connection

- [ ] **3. Database Demo**
  - [ ] Open phpMyAdmin
  - [ ] Show table relationships
  - [ ] Run sample query
  - [ ] Show transaction records
  - [ ] Explain foreign keys

- [ ] **4. Full Transaction Demo**
  - [ ] Add items to cart
  - [ ] Checkout (if balance sufficient)
  - [ ] Show success message
  - [ ] Query database to verify
  - [ ] Show saldo decreased
  - [ ] Show transaksi created

---

## 🎓 Discussion Points

### **Q: Why use localStorage for cart?**
**A:** Temporary storage in browser. Real e-commerce would use session/database.

### **Q: Why have detail_transaksi table?**
**A:** One transaction can have multiple items (game + DLC + bundle). Normalization.

### **Q: What if payment fails?**
**A:** ROLLBACK — all changes undone. Customer's saldo not affected.

### **Q: How ensure data consistency?**
**A:** Foreign keys + transactions + validation. ACID properties.

### **Q: Difference between game, dlc, bundle?**
**A:** Game = standalone product | DLC = add-on for game | Bundle = package deal

---

## 🛠️ Troubleshooting

**Problem:** Login page doesn't load
- **Fix:** Check if Apache/PHP running: `sudo systemctl status apache2`

**Problem:** API returns error
- **Fix:** Check database connected: Open phpMyAdmin
- **Fix:** Check PHP error logs: `/var/log/apache2/error.log`

**Problem:** Can't add to cart
- **Fix:** Check DevTools Console for JavaScript errors
- **Fix:** Verify localStorage enabled in browser settings

**Problem:** Checkout fails "Saldo tidak cukup"
- **Fix:** Account balance too low (only Rp 100.000 remains)
- **Fix:** Login with different account or add test balance

---

## 📊 Database Queries to Show

### **1. See all games with prices:**
```sql
SELECT id_game, judul, harga_utama, genre FROM game;
```

**Output:**
```
id_game | judul                    | harga_utama | genre
--------|--------------------------|-------------|------
1       | Persona 5 Royal          | 529000      | JRPG
2       | Persona 3 Reload         | 849000      | JRPG
5       | Cyberpunk 2077           | 699999      | NULL
10      | Digimon Story Time ...   | 899999      | NULL
```

### **2. See all bundles:**
```sql
SELECT id_bundle, nama_bundle, harga_bundle FROM bundle;
```

### **3. See transaction history of customer 1:**
```sql
SELECT t.id_transaksi, t.tanggal_transaksi, t.total_bayar
FROM transaksi t
WHERE t.id_pelanggan = 1
ORDER BY t.tanggal_transaksi DESC;
```

### **4. See what items were in transaction 1:**
```sql
SELECT dt.id_detail, dt.harga_jual,
       g.judul as game_name,
       d.nama_dlc as dlc_name,
       b.nama_bundle as bundle_name
FROM detail_transaksi dt
LEFT JOIN game g ON dt.id_game = g.id_game
LEFT JOIN dlc d ON dt.id_dlc = d.Id_dlc
LEFT JOIN bundle b ON dt.id_bundle = b.id_bundle
WHERE dt.id_transaksi = 1;
```

### **5. See customer balance:**
```sql
SELECT id_pelanggan, nama, saldo FROM pelanggan;
```

---

## 🎬 Presentation Slides Outline

**Slide 1: Project Title**
- GameHub: E-Commerce Game Store
- Database Project

**Slide 2: What is GameHub?**
- Online store for buying games, DLCs, bundles
- Features: Browse, Add to Cart, Checkout, History
- Real-world scenario

**Slide 3: Technology Stack**
- Frontend: HTML5, CSS3, JavaScript (Vanilla)
- Backend: PHP 8.2 (REST API)
- Database: MySQL/MariaDB
- Server: Apache (LAMP)

**Slide 4: System Architecture**
- Client-side: index.html, login.html, cart.html
- Server-side: PHP API endpoints
- Database: 9 normalized tables

**Slide 5: Database Schema**
- pelanggan (users)
- game, dlc, bundle (products)
- transaksi, detail_transaksi (orders)
- platform, game_platform (relationships)

**Slide 6: Key Features**
- User authentication
- Product catalog (games, DLCs, bundles)
- Shopping cart
- Checkout with balance validation
- Transaction history
- Dashboard statistics

**Slide 7: Transaction Flow**
- User adds items → Cart
- Clicks checkout → API call
- Backend validates → Check balance
- BEGIN TRANSACTION → Update DB
- COMMIT or ROLLBACK

**Slide 8: Important Concepts**
- Normalization (3NF)
- Foreign keys & relationships
- ACID properties (Transactions)
- API design (REST)
- Session management

**Slide 9: LIVE DEMO**
- Login
- Browse products
- Add to cart
- Checkout
- Check database

**Slide 10: Q&A**

---

## 📚 Documentation Files Included

1. **demo_gamehub.md** - Complete overview (frontend, backend, features, setup)
2. **database_deep_dive.md** - SQL queries, schema details, performance tips
3. **api_testing_guide.md** - cURL examples, API endpoints, testing workflow
4. **This file** - Quick reference & demo checklist

---

## ⏱️ Time Allocation

- **2 min:** Frontend demo (UI walkthrough)
- **2 min:** Backend explanation (Architecture & APIs)
- **1 min:** Database overview (Tables & queries)
- **Total: 5 minutes** (adjustable based on audience interest)

---

**GameHub Demo Guide** | Mata Kuliah: Basis Data | 2026
