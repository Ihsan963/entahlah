# 🎮 GameHub - Game Store E-Commerce

> Platform jual-beli game, DLC, dan bundle online dengan database relasional

[![Status](https://img.shields.io/badge/Status-Working-brightgreen)]()
[![Tech](https://img.shields.io/badge/Tech-PHP%20%7C%20MySQL%20%7C%20JavaScript-blue)]()
[![Course](https://img.shields.io/badge/Course-Database-orange)]()

---

## 📌 Penjelasan Singkat

**GameHub** adalah aplikasi e-commerce sederhana untuk membeli game dan konten tambahan (DLC). Pengguna bisa login, lihat produk, tambah ke keranjang, dan checkout. Saldo berkurang otomatis saat membeli.

**Mengapa penting:** Mendemonstrasikan konsep database yang sebenarnya - bagaimana data tersimpan, terelasi, dan diproses saat ada transaksi nyata.

---

## 🎯 Fitur Utama

- ✅ **User Authentication** - Login & Register
- ✅ **Product Catalog** - Games, DLCs, Bundles
- ✅ **Shopping Cart** - Simpan di browser
- ✅ **Checkout** - Proses pembayaran dengan validasi saldo
- ✅ **Transaction History** - Riwayat pembelian
- ✅ **Dashboard Stats** - Statistik penjualan

---

## 💻 Tech Stack

```
Frontend:     HTML5 + CSS3 + Vanilla JavaScript
Backend:      PHP 8.2 (REST API)
Database:     MySQL / MariaDB
Server:       Apache (LAMP Stack)
```

---

## 📂 Struktur Folder

```
gamehub/
├── index.html              ← Dashboard utama (setelah login)
├── login.html              ← Halaman login/register
├── cart.html               ← Halaman keranjang belanja
│
├── api/                    ← Backend PHP APIs
│   ├── login.php           ├─ Login endpoint
│   ├── register.php        ├─ Register endpoint
│   ├── games/get_games.php ├─ Ambil semua games
│   ├── dlcs/get_dlcs.php   ├─ Ambil semua DLC
│   ├── bundles/            ├─ Ambil semua bundles
│   ├── checkout/           ├─ Proses pembayaran
│   ├── transactions/       ├─ Riwayat transaksi
│   ├── statistics/         └─ Statistik dashboard
│
├── js/
│   ├── app.js              ← Logika utama (cart, events)
│   └── api.js              ← Fungsi API calls
│
├── css/
│   ├── style.css           ← Styling utama
│   ├── globals.css         ← Global styles
│   └── styleguide.css      ← UI guidelines
│
├── config/
│   ├── db.php              ← Koneksi database
│   └── helper.php          ← Fungsi utility
│
├── img/                    ← Gambar produk
│   ├── games/              ├─ Gambar game
│   ├── dlcs/               ├─ Gambar DLC
│   └── bundles/            └─ Gambar bundle
│
└── gamehub db.sql          ← Database schema
```

---

## 🗄️ Database Schema

### **9 Tabel Utama:**

#### 1. `pelanggan` (Users)
```
id_pelanggan (PK)
├─ nama
├─ email
├─ username
├─ password
├─ saldo (jumlah uang digital)
└─ tanggal_daftar
```
💡 Menyimpan data user yang terdaftar

---

#### 2. `game` (Produk Game)
```
id_game (PK)
├─ judul
├─ harga_utama
├─ genre
└─ gambar
```
💡 Katalog game yang dijual

---

#### 3. `dlc` (Konten Tambahan)
```
Id_dlc (PK)
├─ id_game (FK → game)
├─ nama_dlc
├─ harga_dlc
├─ tipe (Ekspansi/Karakter/Kosmetik)
└─ gambar
```
💡 Konten tambahan untuk game tertentu

---

#### 4. `bundle` (Paket Bundle)
```
id_bundle (PK)
├─ nama_bundle
├─ harga_bundle
├─ deskripsi
└─ gambar
```
💡 Paket berisi beberapa game/DLC dengan harga special

---

#### 5. `bundle_items` (Detail Bundle)
```
id_bundle (FK)
├─ id_game (FK → game)
└─ id_dlc (FK → dlc)
```
💡 Menghubungkan bundle dengan games/DLCs yang dikandung

---

#### 6. `platform` (Platform Game)
```
id_platform (PK)
└─ nama_platform (PC/Playstation/XBOX/Mobile)
```
💡 Platform apa saja yang tersedia

---

#### 7. `game_platform` (Game di Platform Apa)
```
id_game_platform (PK)
├─ id_game (FK → game)
└─ id_platform (FK → platform)
```
💡 Game mana tersedia di platform mana

---

#### 8. `transaksi` (Transaksi Penjualan)
```
id_transaksi (PK)
├─ id_pelanggan (FK → pelanggan)
├─ tanggal_transaksi
└─ total_bayar
```
💡 Catat setiap transaksi/pembelian

---

#### 9. `detail_transaksi` (Item per Transaksi)
```
id_detail (PK)
├─ id_transaksi (FK → transaksi)
├─ id_game (FK → game)
├─ id_dlc (FK → dlc)
├─ id_bundle (FK → bundle)
└─ harga_jual
```
💡 Detail: game/DLC/bundle apa saja yang dibeli dalam satu transaksi

---

## 🔄 Bagaimana Aplikasi Bekerja?

### **Flow Singkat:**

```
User                Backend (PHP)           Database (MySQL)
  │                    │                         │
  ├──→ Input login ──→ Cek username/password ──→ Query user
  │                    ↑                         │
  │                    └─────────── Return id ──┘
  │
  ├──→ Lihat games ──→ SELECT * FROM game ──→ Return games list
  │
  ├──→ Klik "Beli" ──→ Simpan di localStorage (BROWSER)
  │    Barang 1,2,3
  │
  └──→ Click Checkout ──→ POST /checkout
                            │
                            ├─ Cek saldo cukup?
                            ├─ Mulai transaksi (BEGIN)
                            ├─ Kurangi saldo
                            ├─ Create transaksi record
                            ├─ Create detail untuk setiap item
                            └─ Simpan semua (COMMIT)
                                    ↓
                            UPDATE pelanggan → saldo berkurang
                            INSERT transaksi → buat record baru
                            INSERT detail → simpan barang yang dibeli
```

---

## 🔌 API Endpoints

### **Authentication**
```
POST /api/login.php
  Input:  { username, password }
  Output: { id_pelanggan, nama, email }

POST /api/register.php
  Input:  { nama, email, username, password }
  Output: { id_pelanggan, nama, email }
```

### **Products**
```
GET /api/games/get_games.php
  Output: [ { id_game, judul, harga_utama, genre, gambar } ]

GET /api/dlcs/get_dlcs.php
  Output: [ { Id_dlc, nama_dlc, harga_dlc, tipe, gambar } ]

GET /api/bundles/get_bundles.php
  Output: [ { id_bundle, nama_bundle, harga_bundle, deskripsi } ]
```

### **Checkout & Orders**
```
POST /api/checkout/process_checkout.php
  Input:  { id_pelanggan, cart: [...] }
  Output: { success, id_transaksi, total_bayar, saldo_sisa }

GET /api/transactions/get_transactions.php?id_pelanggan=1
  Output: [ { id_transaksi, tanggal_transaksi, total_bayar, items[] } ]
```

### **Statistics**
```
GET /api/statistics/get_dashboard_stats.php
  Output: { total_customers, total_transactions, total_revenue, avg_transaction }

GET /api/customers/get_customer.php?id=1
  Output: { id_pelanggan, nama, email, saldo, tanggal_daftar }
```

---

## 👤 Test Account

```
Login 1:
  Username: Jabond
  Password: admin123
  Balance:  Rp 4.100.000

Login 2:
  Username: dhafin
  Password: dhafin123
  Balance:  Rp 7.901.000
```

---

## 🚀 Cara Setup

### **1. Download Database**
```bash
# Buka MySQL
mysql -u root -p

# Create database
CREATE DATABASE gamehub;
USE gamehub;

# Import SQL file
source /path/to/gamehub\ db.sql;
```

### **2. Setup Database Connection**
Edit `config/db.php`:
```php
$host     = 'localhost';
$db       = 'gamehub';
$username = 'root';
$password = '';  // Sesuaikan dengan password MySQL Anda
```

### **3. Copy ke Web Server**
```bash
# Copy folder ke Apache DocumentRoot
cp -r GameHub-main /var/www/html/gamehub

# Set permissions
chmod -R 755 /var/www/html/gamehub
```

### **4. Akses Aplikasi**
```
URL: http://localhost/gamehub
```

---

## 📖 Mengerti Alur Transaksi

### **Contoh: User Dhafin membeli 2 bundle**

**Step 1: Login**
```
Input: username="dhafin", password="dhafin123"
Output: id_pelanggan=1, nama="Dhafin Madany"
Status: ✅ Login berhasil, simpan ke localStorage
```

**Step 2: Browse Products**
```
GET /api/bundles/get_bundles.php
Output: 4 bundles ditampilkan
```

**Step 3: Add to Cart**
```
Klik "Beli" Persona Paradise (Rp 1.199.000)
→ localStorage.cart = [
    { id_bundle: 1, harga: 1199000, tipe: "bundle" }
  ]

Klik "Beli" Persona 3 Joker (Rp 900.000)
→ localStorage.cart = [
    { id_bundle: 1, harga: 1199000, tipe: "bundle" },
    { id_bundle: 2, harga: 900000, tipe: "bundle" }
  ]

Total: Rp 2.099.000
```

**Step 4: Checkout - Backend Processing**
```
POST /api/checkout/process_checkout.php
{
  "id_pelanggan": 1,
  "cart": [...]
}

Backend Logic:
├─ Check: Pelanggan ada? ✅
├─ Check: Saldo 10.000.000 >= 2.099.000? ✅
├─ BEGIN TRANSACTION
│  ├─ UPDATE pelanggan: saldo = 10000000 - 2099000 = 7.901.000
│  ├─ INSERT transaksi: (id=1, total=2099000)
│  ├─ INSERT detail_transaksi: 
│  │  ├─ Item 1: bundle 1, harga 1199000
│  │  └─ Item 2: bundle 2, harga 900000
│  └─ SELECT saldo_baru = 7.901.000
└─ COMMIT ✅
```

**Step 5: Verify di Database**
```sql
-- Check saldo berkurang
SELECT saldo FROM pelanggan WHERE id_pelanggan=1;
→ 7.901.000 ✅

-- Check transaksi terbuat
SELECT * FROM transaksi WHERE id_pelanggan=1;
→ id_transaksi=1, total=2.099.000 ✅

-- Check items di transaksi
SELECT * FROM detail_transaksi WHERE id_transaksi=1;
→ Item 1: bundle 1 (Rp 1.199.000) ✅
→ Item 2: bundle 2 (Rp 900.000) ✅
```

---

## 🎓 Konsep Penting yang Ditunjukkan

### **1. Relational Database**
- Tabel saling terhubung dengan Foreign Keys
- Data terorganisir & tidak redundan
- Contoh: bundle → bundle_items → game/dlc

### **2. Transaction (ACID)**
- **Atomicity:** Semua operasi berhasil atau semua gagal
- **Consistency:** Data selalu valid setelah transaksi
- **Isolation:** Transaksi tidak saling gangggu
- **Durability:** Data tersimpan permanent

Contoh: Jika ada error saat UPDATE saldo, INSERT transaksi batal total.

### **3. Normalization (3NF)**
- Data tidak redundan
- Setiap tabel punya satu tanggung jawab
- Mudah di-update tanpa anomali

### **4. API Design (REST)**
- Setiap endpoint punya fungsi jelas
- Request/response format standard (JSON)
- HTTP status codes yang tepat

### **5. Session Management**
- User login disimpan di localStorage
- Setiap request membawa id_pelanggan
- Logout = hapus localStorage

---

## 🧪 Testing dengan cURL

### **Test Login:**
```bash
curl -X POST http://localhost/gamehub/api/login.php \
  -H "Content-Type: application/json" \
  -d '{"username":"Jabond","password":"admin123"}'
```

### **Test Get Games:**
```bash
curl -X GET http://localhost/gamehub/api/games/get_games.php
```

### **Test Checkout:**
```bash
curl -X POST http://localhost/gamehub/api/checkout/process_checkout.php \
  -H "Content-Type: application/json" \
  -d '{
    "id_pelanggan":1,
    "cart":[
      {"id_bundle":1,"harga":1199000,"tipe":"bundle"}
    ]
  }'
```

---

## ⚠️ Penting untuk Demo

### **Checklist Sebelum Demo:**
- [ ] MySQL running (check with phpMyAdmin)
- [ ] gamehub database imported
- [ ] config/db.php sudah sesuai dengan password MySQL Anda
- [ ] Folder gamehub ada di /var/www/html/
- [ ] Buka browser: http://localhost/gamehub
- [ ] Try login dengan test account

### **Selama Demo:**
1. **Show Frontend:**
   - Login page
   - Dashboard (games, dlcs, bundles)
   - Add to cart
   - Shopping cart page
   - Checkout

2. **Show Backend:**
   - Buka file api/ untuk show endpoint
   - Buka config/db.php untuk show koneksi
   - Explain validation logic

3. **Show Database:**
   - Open phpMyAdmin
   - Click database gamehub
   - Show tables
   - Run query: `SELECT * FROM transaksi;`
   - Run query: `SELECT * FROM detail_transaksi WHERE id_transaksi=1;`

---

## 🔍 Troubleshooting

| Problem | Solution |
|---------|----------|
| Page blank / 404 | Check if Apache running. URL harus `http://localhost/gamehub` |
| Can't login | Check database imported. Test account ada di pelanggan table |
| Checkout failed | Check saldo cukup. Use account dengan balance lebih besar |
| Cart empty | Clear browser localStorage. Try again |
| API error | Check db.php password sesuai dengan MySQL password Anda |

---

## 📊 Demo Statistics

```
Databases:     1 (gamehub)
Tables:        9 (with relationships)
Sample Users:  2 (Jabond, dhafin)
Sample Games:  4 (Persona, Cyberpunk, Digimon)
Sample DLC:    4 (Joker, Expansion, etc)
Sample Bundle: 4 (combinations)
Transactions:  11 (recorded history)
Total API:     9 endpoints
```

---

## 📚 Dokumentasi Lengkap

Untuk penjelasan lebih detail, lihat file-file ini:

| File | Isi |
|------|-----|
| **quick_reference.md** | Demo checklist & tips |
| **demo_gamehub.md** | Penjelasan lengkap semua aspek |
| **database_deep_dive.md** | SQL queries & optimization |
| **api_testing_guide.md** | API testing dengan cURL |
| **INDEX.md** | Panduan cara pakai semua file |

---

## ✅ Success Criteria

Demo dianggap berhasil jika:

✔️ Bisa login dengan test account  
✔️ Dashboard tampil dengan games/dlcs/bundles  
✔️ Bisa add item ke cart  
✔️ Cart counter update  
✔️ Bisa checkout  
✔️ Saldo berkurang (verify di database)  
✔️ Transaksi muncul di database  
✔️ Bisa jelaskan arsitektur 3-layer  
✔️ Bisa show database relationships  

---

## 💡 Tips

- **Jangan bikin transaksi banyak** dengan akun yang sama (akan habis saldonya)
- **Open DevTools** saat demo untuk show localStorage & network requests
- **Prepare phpMyAdmin** sebelum demo untuk show database
- **Siapkan cURL command** jika ada yang minta test API
- **Backup test accounts** dengan saldo besar sebelum demo

---

## 📞 Contact

Jika ada pertanyaan, cek dokumentasi:
- Architecture? → demo_gamehub.md
- Database? → database_deep_dive.md
- API? → api_testing_guide.md
- Demo tips? → quick_reference.md
- Confused? → INDEX.md

---

## 📄 License

Project untuk mata kuliah Basis Data

---

**Siap untuk demo? Good luck! 🎉**

*Last updated: June 2026*
