# 🎮 GameHub - Demo & Penjelasan Lengkap
**Aplikasi E-Commerce Game Store** | Mata Kuliah: Basis Data

---

## 📋 Daftar Isi
1. [Ringkasan Project](#ringkasan-project)
2. [Arsitektur Sistem](#arsitektur-sistem)
3. [Database Schema](#database-schema)
4. [Frontend Structure](#frontend-structure)
5. [Backend API](#backend-api)
6. [Fitur Utama](#fitur-utama)
7. [Alur Transaksi](#alur-transaksi)
8. [Cara Setup](#cara-setup)

---

## 📌 Ringkasan Project

**GameHub** adalah sebuah platform e-commerce untuk menjual game, DLC (Downloadable Content), dan Bundle. Pengguna dapat:
- 🔐 Login/Register
- 🎮 Melihat katalog game
- 🛒 Menambah item ke keranjang
- 💳 Melakukan checkout
- 📊 Melihat statistik transaksi

**Tech Stack:**
- **Frontend:** HTML5, CSS3, Vanilla JavaScript
- **Backend:** PHP 8.2 (REST API)
- **Database:** MySQL/MariaDB
- **Server:** Apache (LAMP Stack)

---

## 🏗️ Arsitektur Sistem

```
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (Client-side)                  │
│  ┌──────────────┐  ┌────────────────┐  ┌──────────────┐    │
│  │ index.html   │  │  login.html    │  │  cart.html   │    │
│  │ (Dashboard)  │  │ (Auth Page)    │  │(Cart Page)   │    │
│  └──────┬───────┘  └────────┬───────┘  └──────┬───────┘    │
│         │                   │                  │             │
│         └───────────────────┴──────────────────┘             │
│                       ↓                                      │
│         ┌────────────────────────────────┐                  │
│         │  app.js & api.js               │                  │
│         │ (Business Logic & API Calls)   │                  │
│         └────────────────┬───────────────┘                  │
│                          │                                  │
│         ┌────────────────────────────────┐                  │
│         │  localStorage (Session Data)   │                  │
│         │  - pelanggan (user data)       │                  │
│         │  - cart (items)                │                  │
│         └────────────────┬───────────────┘                  │
└────────────────────────────┼────────────────────────────────┘
                             │ AJAX/Fetch Requests
                             ↓
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND (Server-side)                    │
│              REST API (PHP) - http://localhost/gamehub/api   │
│  ┌─────────────┬──────────────┬────────────────┐            │
│  │ auth/       │ catalog/     │ checkout/      │            │
│  │ ├─ login    │ ├─ games     │ ├─ process     │            │
│  │ ├─ register │ ├─ dlcs      │ └─ ...         │            │
│  │ └─ ...      │ ├─ bundles   │                │            │
│  │             │ └─ ...       │ transactions/  │            │
│  │             │              │ ├─ list        │            │
│  │  customers/ │  statistics/ │ └─ ...         │            │
│  │  ├─ info    │  ├─ dashboard│                │            │
│  │  └─ ...     │  └─ ...      │                │            │
│  └─────────────┴──────────────┴────────────────┘            │
│                          ↓                                  │
│         ┌──────────────────────────────────┐                │
│         │  config/db.php & helper.php      │                │
│         │  (Database Connection & Utils)   │                │
│         └──────────────────┬───────────────┘                │
│                            ↓                                │
└─────────────────────────────────────────────────────────────┘
                             │
                    Database Connection
                             ↓
                   ┌──────────────────┐
                   │    MySQL/MariaDB │
                   │  gamehub (DB)    │
                   └──────────────────┘
```

---

## 🗄️ Database Schema

### **Tabel 1: `pelanggan`** (User/Customer)
```sql
CREATE TABLE pelanggan (
  id_pelanggan INT PRIMARY KEY AUTO_INCREMENT,
  nama VARCHAR(100),
  email VARCHAR(100),
  username VARCHAR(100) NOT NULL,
  password VARCHAR(50) NOT NULL,
  tanggal_daftar DATETIME,
  saldo INT DEFAULT 0        -- Saldo digital user
);
```

**Penjelasan:**
- Menyimpan data user yang terdaftar
- Saldo adalah jumlah uang digital yang dimiliki user
- Username & password untuk login

---

### **Tabel 2: `game`** (Produk Game)
```sql
CREATE TABLE game (
  id_game INT PRIMARY KEY AUTO_INCREMENT,
  judul VARCHAR(255),
  harga_utama INT,          -- Harga game utama
  genre VARCHAR(50),
  gambar VARCHAR(255)       -- Nama file gambar
);
```

**Data Contoh:**
- Persona 5 Royal (Rp 529.000)
- Persona 3 Reload (Rp 849.000)
- Cyberpunk 2077 (Rp 699.999)
- Digimon Story Time Stranger (Rp 899.999)

---

### **Tabel 3: `dlc`** (Downloadable Content)
```sql
CREATE TABLE dlc (
  Id_dlc INT PRIMARY KEY AUTO_INCREMENT,
  id_game INT NOT NULL,     -- Referensi ke game
  nama_dlc VARCHAR(255),
  harga_dlc INT,
  tipe ENUM('Ekspansi', 'Karakter', 'Kosmetik'),
  gambar VARCHAR(255),
  FOREIGN KEY (id_game) REFERENCES game(id_game)
);
```

**Penjelasan:**
- DLC adalah konten tambahan untuk game tertentu
- Tipe: Ekspansi (cerita baru), Karakter (skin), Kosmetik (visual)
- Contoh: Joker Outfit untuk Persona 3 Reload (Rp 159.000)

---

### **Tabel 4: `bundle`** (Paket Bundle)
```sql
CREATE TABLE bundle (
  id_bundle INT PRIMARY KEY AUTO_INCREMENT,
  nama_bundle VARCHAR(255),
  harga_bundle INT,         -- Harga total bundle
  deskripsi TEXT,
  gambar VARCHAR(255)
);
```

**Penjelasan:**
- Bundel adalah paket berisi beberapa game/DLC dengan harga spesial
- Contoh: "Persona Paradise" (Rp 1.199.000) = Persona 5 Royal + Persona 3 Reload

---

### **Tabel 5: `bundle_items`** (Detail Bundle)
```sql
CREATE TABLE bundle_items (
  id_bundle INT NOT NULL,   -- Referensi ke bundle
  id_game INT,              -- Bisa berisi game
  id_dlc INT,               -- Atau DLC
  FOREIGN KEY (id_bundle) REFERENCES bundle(id_bundle),
  FOREIGN KEY (id_game) REFERENCES game(id_game),
  FOREIGN KEY (id_dlc) REFERENCES dlc(Id_dlc)
);
```

**Penjelasan:**
- Junction table yang menghubungkan bundle dengan games/DLCs
- Satu bundle bisa punya multiple games/DLCs

**Contoh Data:**
```
Bundle ID 1 (Persona Paradise) → Game 1 (Persona 5 Royal)
                              → Game 2 (Persona 3 Reload)
```

---

### **Tabel 6: `platform`** (Platform Permainan)
```sql
CREATE TABLE platform (
  id_platform INT PRIMARY KEY AUTO_INCREMENT,
  nama_platform VARCHAR(50)
);
```

**Data:**
- PC
- Playstation
- XBOX
- Mobile

---

### **Tabel 7: `game_platform`** (Game di Platform Apa)
```sql
CREATE TABLE game_platform (
  id_game_platform INT PRIMARY KEY AUTO_INCREMENT,
  id_game INT NOT NULL,
  id_platform INT NOT NULL,
  FOREIGN KEY (id_game) REFERENCES game(id_game),
  FOREIGN KEY (id_platform) REFERENCES platform(id_platform)
);
```

**Penjelasan:**
- Menunjukkan game mana yang tersedia di platform mana
- Contoh: Persona 5 Royal tersedia di PC, Playstation, XBOX

---

### **Tabel 8: `transaksi`** (Transaksi Penjualan)
```sql
CREATE TABLE transaksi (
  id_transaksi INT PRIMARY KEY AUTO_INCREMENT,
  id_pelanggan INT NOT NULL,
  tanggal_transaksi DATETIME,
  total_bayar INT,
  FOREIGN KEY (id_pelanggan) REFERENCES pelanggan(id_pelanggan)
);
```

**Penjelasan:**
- Mencatat setiap transaksi yang dilakukan customer
- Menyimpan tanggal transaksi dan total pembayaran

---

### **Tabel 9: `detail_transaksi`** (Detail Item per Transaksi)
```sql
CREATE TABLE detail_transaksi (
  id_detail INT PRIMARY KEY AUTO_INCREMENT,
  id_transaksi INT NOT NULL,
  id_game INT,
  id_dlc INT,
  id_bundle INT,
  harga_jual INT,           -- Harga saat dibeli
  FOREIGN KEY (id_transaksi) REFERENCES transaksi(id_transaksi),
  FOREIGN KEY (id_game) REFERENCES game(id_game),
  FOREIGN KEY (id_dlc) REFERENCES dlc(Id_dlc),
  FOREIGN KEY (id_bundle) REFERENCES bundle(id_bundle)
);
```

**Penjelasan:**
- Satu transaksi bisa punya multiple items
- Menyimpan harga saat pembelian (untuk history)
- Salah satu dari id_game/id_dlc/id_bundle diisi (not null)

**Contoh:**
```
Transaksi ID 1:
  └─ Item 1: Bundle 1 (Rp 1.199.000)
  └─ Item 2: Bundle 2 (Rp 900.000)
  └─ Total: Rp 2.099.000
```

---

## 🎨 Frontend Structure

### **File Structure:**
```
gamehub/
├── index.html          # Main Dashboard (setelah login)
├── login.html          # Login/Register Page
├── cart.html           # Shopping Cart Page
├── js/
│   ├── app.js          # Main logic & cart management
│   └── api.js          # API communication functions
├── css/
│   ├── style.css       # Main stylesheet
│   ├── globals.css     # Global styles
│   └── styleguide.css  # UI guide
└── img/
    ├── games/          # Game thumbnails
    ├── bundles/        # Bundle images
    ├── dlcs/           # DLC images
    └── ...
```

---

### **index.html - Main Dashboard**

**Layout:**
```
┌─────────────────────────────────────────────────────────┐
│  GameHub    [Nav Menu]           [Search]  [Username]  │
│                                           [Cart Icon]   │ ← Topbar
├─────────────────────────────────────────────────────────┤
│                   Hero Section                          │ ← Welcome Banner
│  GameHub adalah platform jual beli game favorit...      │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ GAMES                                                   │ ← Section
│  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐             │
│  │ Card  │ │ Card  │ │ Card  │ │ Card  │             │
│  │ Game  │ │ Game  │ │ Game  │ │ Game  │             │
│  │Price  │ │Price  │ │Price  │ │Price  │             │
│  │Beli   │ │Beli   │ │Beli   │ │Beli   │             │
│  └───────┘ └───────┘ └───────┘ └───────┘             │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ DLC (SCROLL HORIZONTAL)                                 │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐  →                   │
│  │ DLC │ │ DLC │ │ DLC │ │ DLC │                      │
│  │Card │ │Card │ │Card │ │Card │                      │
│  └─────┘ └─────┘ └─────┘ └─────┘                      │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ BUNDLES                                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐   │
│  │Bundle Card   │ │Bundle Card   │ │Bundle Card   │   │
│  │dengan detail │ │dengan detail │ │dengan detail │   │
│  └──────────────┘ └──────────────┘ └──────────────┘   │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ STATISTIK                                               │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐              │
│  │Stat Card │ │Stat Card │ │Stat Card │              │
│  └──────────┘ └──────────┘ └──────────┘              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ TRANSAKSI TERBARU (Transaction History)                │
│  ┌─────────────────────────────────────┐              │
│  │Transaksi #1 - 04 Jun 2026           │              │
│  │Persona Paradise + Persona 3 Joker   │              │
│  │Total: Rp 2.099.000                  │              │
│  └─────────────────────────────────────┘              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│ PELANGGAN ONLINE                                        │
│  [Avatar] Dhafin Madany (Rp 10.000.000)               │
│  [Avatar] James Bond (Rp 4.100.000)                   │
└─────────────────────────────────────────────────────────┘
```

---

### **login.html - Authentication**

**Features:**
- Login form (username + password)
- Register form (nama, email, username, password)
- Session management via localStorage
- Validation & error handling

---

### **cart.html - Shopping Cart**

**Features:**
- Display cart items
- Edit quantity
- Remove items
- Checkout button
- Total calculation

---

## 🔌 Backend API

### **Base URL:** `http://localhost/gamehub/api`

---

### **1. Authentication Endpoints**

#### **POST /login.php**
**Request:**
```json
{
  "username": "Jabond",
  "password": "admin123"
}
```

**Response (Success):**
```json
{
  "success": true,
  "message": "Login berhasil.",
  "data": {
    "id_pelanggan": 3,
    "nama": "James Bond",
    "username": "Jabond",
    "email": "lalala@gmail.com"
  }
}
```

**Logic:**
- Check username di database (case-insensitive)
- Verify password (plain text atau bcrypt)
- Return user data (tanpa password)

---

#### **POST /register.php**
**Request:**
```json
{
  "nama": "John Doe",
  "email": "john@example.com",
  "username": "johndoe",
  "password": "password123"
}
```

**Response (Success):**
```json
{
  "success": true,
  "message": "Registrasi berhasil.",
  "data": {
    "id_pelanggan": 4,
    "nama": "John Doe",
    "email": "john@example.com"
  }
}
```

**Logic:**
- Validate input (required fields)
- Check if username/email already exists
- Insert new user to `pelanggan` table
- Default saldo = 0 (Rp 0)

---

### **2. Game Catalog Endpoints**

#### **GET /games/get_games.php**
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id_game": 1,
      "judul": "Persona 5 Royal",
      "harga_utama": 529000,
      "genre": "JRPG",
      "gambar": "persona5royal.jpg"
    },
    {
      "id_game": 2,
      "judul": "Persona 3 Reload",
      "harga_utama": 849000,
      "genre": "JRPG",
      "gambar": "persona3reload.jpg"
    },
    ...
  ]
}
```

---

#### **GET /dlcs/get_dlcs.php**
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "Id_dlc": 1,
      "id_game": 2,
      "nama_dlc": "Joker Outfit",
      "harga_dlc": 159000,
      "tipe": "Kosmetik",
      "gambar": "jokerp5rdlc.jpg"
    },
    ...
  ]
}
```

---

#### **GET /bundles/get_bundles.php**
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id_bundle": 1,
      "nama_bundle": "Persona Paradise",
      "harga_bundle": 1199000,
      "deskripsi": "Bundle Persona",
      "gambar": "p5r-p3rbundle.jpg"
    },
    ...
  ]
}
```

---

### **3. Checkout Endpoint**

#### **POST /checkout/process_checkout.php**

**Request:**
```json
{
  "id_pelanggan": 1,
  "cart": [
    {
      "nama": "Persona Paradise",
      "harga": 1199000,
      "id": 1,
      "tipe": "bundle",
      "id_bundle": 1
    },
    {
      "nama": "Persona 3 Joker",
      "harga": 900000,
      "id": 2,
      "tipe": "bundle",
      "id_bundle": 2
    }
  ]
}
```

**Logic:**
1. ✅ Validasi pelanggan ada
2. ✅ Hitung total = Rp 2.099.000
3. ✅ Check saldo pelanggan (harus ≥ total)
4. ✅ Jika saldo tidak cukup → return error
5. ✅ **Transaction (BEGIN):**
   - Kurangi saldo pelanggan
   - Insert ke tabel `transaksi`
   - Insert setiap item ke tabel `detail_transaksi`
   - Select saldo baru
6. ✅ **Commit** atau **Rollback** jika error

**Response (Success):**
```json
{
  "success": true,
  "id_transaksi": 1,
  "total_bayar": 2099000,
  "saldo_sisa": 7901000
}
```

**Response (Gagal - Saldo Tidak Cukup):**
```json
{
  "error": "Saldo tidak cukup. Saldo kamu: Rp 1.000.000"
}
```

---

### **4. Customer Endpoints**

#### **GET /customers/get_customer.php?id=1**
**Response:**
```json
{
  "success": true,
  "data": {
    "id_pelanggan": 1,
    "nama": "Dhafin Madany",
    "email": "dhafin@example.com",
    "username": "dhafin",
    "saldo": 10000000,
    "tanggal_daftar": "2026-05-10 06:25:59"
  }
}
```

---

### **5. Transaction & Statistics Endpoints**

#### **GET /transactions/get_transactions.php?id_pelanggan=1**
**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id_transaksi": 1,
      "tanggal_transaksi": "2026-06-04 17:47:14",
      "total_bayar": 2099000,
      "items": [
        {
          "nama": "Persona Paradise",
          "tipe": "bundle",
          "harga": 1199000
        },
        {
          "nama": "Persona 3 Joker",
          "tipe": "bundle",
          "harga": 900000
        }
      ]
    },
    ...
  ]
}
```

---

#### **GET /statistics/get_dashboard_stats.php**
**Response:**
```json
{
  "success": true,
  "data": {
    "total_customers": 2,
    "total_transactions": 11,
    "total_revenue": 17951000,
    "average_transaction": 1631909
  }
}
```

---

## ✨ Fitur Utama

### **1. 🔐 Authentication (Login/Register)**
```
User → Click Login Page → Input username/password → 
API Check credentials → Valid? → 
YES → Save ke localStorage → Redirect ke Dashboard
NO  → Show error message
```

---

### **2. 🛍️ Browse & Add to Cart**
```
User View Games/DLC/Bundles → Click "Beli" → 
Add to localStorage cart → Update cart counter → Show toast
```

**Cart Storage (localStorage):**
```javascript
{
  "cart": [
    {
      "nama": "Persona Paradise",
      "harga": 1199000,
      "id": 1,
      "tipe": "bundle",
      "id_bundle": 1,
      "gambar": "..."
    },
    ...
  ]
}
```

---

### **3. 💳 Checkout & Payment**
```
User → View Cart → Click Checkout → 
Send cart data to API → 
API validate saldo → 
SUCCESS → Deduct saldo → Create transaction → Save details → Return success
FAIL → Return error message
```

**Payment Deduction:**
```
Saldo sebelum: Rp 10.000.000
Total pembelian: Rp 2.099.000
Saldo sesudah: Rp 7.901.000
```

---

### **4. 📊 Dashboard Statistics**
- Total pelanggan aktif
- Total transaksi
- Total revenue
- Rata-rata transaksi
- Riwayat transaksi pelanggan
- Daftar pelanggan dengan saldo

---

### **5. 🔄 Cart Management**
- Add item (cek duplikat)
- Remove item
- View total price
- Clear cart
- Persist in localStorage

---

## 🔄 Alur Transaksi Lengkap

### **Scenario: User Dhafin Membeli Bundle**

**Step 1: Authentication**
```
Input: username="dhafin", password="dhafin123"
API /login.php:
  SELECT * FROM pelanggan WHERE username='dhafin'
  → Found, password match
  → Return id_pelanggan=1, nama='Dhafin Madany'
  → Save ke localStorage
```

**Step 2: Browse Products**
```
API /games/get_games.php:
  SELECT * FROM game
  → Return 4 games
  → Display di dashboard

API /bundles/get_bundles.php:
  SELECT * FROM bundle
  → Return 4 bundles
  → Display di dashboard
```

**Step 3: Add to Cart**
```
User clicks "Beli" on "Persona Paradise"
JavaScript:
  cart.push({
    "id_bundle": 1,
    "nama": "Persona Paradise",
    "harga": 1199000,
    "tipe": "bundle"
  })
  localStorage.cart = JSON.stringify(cart)
  Update cart counter → "1 Item | Rp 1.199.000"
```

**Step 4: Add More Items**
```
User clicks "Beli" on "Persona 3 Joker"
cart.push({
  "id_bundle": 2,
  "nama": "Persona 3 Joker",
  "harga": 900000,
  "tipe": "bundle"
})
Update cart counter → "2 Items | Rp 2.099.000"
```

**Step 5: Checkout**
```
POST /checkout/process_checkout.php
Body:
{
  "id_pelanggan": 1,
  "cart": [
    {"id_bundle": 1, "harga": 1199000, ...},
    {"id_bundle": 2, "harga": 900000, ...}
  ]
}

Backend:
1. SELECT saldo FROM pelanggan WHERE id_pelanggan=1
   → saldo = 10.000.000
2. total = 2.099.000
3. 10.000.000 >= 2.099.000? YES ✅

4. BEGIN TRANSACTION
   
   a) UPDATE pelanggan SET saldo = 10000000 - 2099000
      WHERE id_pelanggan = 1
      → saldo baru = 7.901.000
   
   b) INSERT INTO transaksi (id_pelanggan, tanggal_transaksi, total_bayar)
      VALUES (1, NOW(), 2099000)
      → id_transaksi = 1
   
   c) INSERT INTO detail_transaksi (id_transaksi, id_bundle, harga_jual)
      VALUES (1, 1, 1199000)
      VALUES (1, 2, 900000)
   
   d) SELECT saldo FROM pelanggan WHERE id_pelanggan=1
      → saldo = 7.901.000

5. COMMIT ✅

Response:
{
  "success": true,
  "id_transaksi": 1,
  "total_bayar": 2099000,
  "saldo_sisa": 7901000
}
```

**Step 6: Verify in Database**

```sql
-- Check transaction
SELECT * FROM transaksi WHERE id_transaksi=1;
→ id_transaksi=1, id_pelanggan=1, tanggal_transaksi='2026-06-04 17:47:14', total_bayar=2099000

-- Check transaction details
SELECT * FROM detail_transaksi WHERE id_transaksi=1;
→ id_detail=1, id_transaksi=1, id_bundle=1, harga_jual=1199000
→ id_detail=2, id_transaksi=1, id_bundle=2, harga_jual=900000

-- Check customer saldo
SELECT saldo FROM pelanggan WHERE id_pelanggan=1;
→ saldo = 7901000
```

---

## 🚀 Cara Setup

### **Requirement:**
- PHP 8.2+
- MySQL 8.0+ atau MariaDB 10.4+
- Apache Web Server
- Composer (optional, untuk dependency management)

### **Step 1: Install Database**
```bash
# Akses MySQL
mysql -u root -p

# Create database
CREATE DATABASE gamehub;
USE gamehub;

# Import SQL file
source /path/to/gamehub\ db.sql;
```

### **Step 2: Configure Database Connection**
Edit `config/db.php`:
```php
<?php
$host     = 'localhost';
$db       = 'gamehub';
$username = 'root';
$password = ''; // Sesuaikan dengan password MySQL kamu
$charset  = 'utf8mb4';

$dsn = "mysql:host=$host;dbname=$db;charset=$charset";

try {
    $pdo = new PDO($dsn, $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die('Connection error: ' . $e->getMessage());
}
?>
```

### **Step 3: Setup Apache DocumentRoot**
```bash
# Copy project ke DocumentRoot (biasanya /var/www/html)
cp -r GameHub-main /var/www/html/gamehub

# Set permissions
chmod -R 755 /var/www/html/gamehub
chmod -R 777 /var/www/html/gamehub/img
```

### **Step 4: Configure Apache VirtualHost** (Optional)
Edit `/etc/apache2/sites-available/gamehub.conf`:
```apache
<VirtualHost *:80>
    ServerName gamehub.local
    DocumentRoot /var/www/html/gamehub
    
    <Directory /var/www/html/gamehub>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Enable:
```bash
a2ensite gamehub
a2enmod rewrite
systemctl restart apache2
```

### **Step 5: Test Application**
```
URL: http://localhost/gamehub
atau
URL: http://gamehub.local
```

---

## 🎯 Demo Flow

1. **Login Page**
   - Demo account: username=`Jabond`, password=`admin123`
   - Atau register akun baru

2. **Home Dashboard**
   - Lihat semua games, DLCs, bundles
   - Lihat statistik dashboard
   - Lihat riwayat transaksi

3. **Add to Cart**
   - Klik "Beli" pada game/DLC/bundle
   - Lihat cart counter update
   - Klik cart icon untuk lihat detail

4. **Checkout**
   - Go to cart page
   - Review items
   - Click checkout
   - Transaction processed

5. **Verify Results**
   - Check dashboard stats update
   - Check customer saldo berkurang
   - Check transaction history muncul

---

## 📝 Kesimpulan

**GameHub** adalah aplikasi full-stack yang mendemonstrasikan:
- ✅ Database design dengan relationships
- ✅ API RESTful dengan PHP
- ✅ Frontend dengan vanilla JavaScript & localStorage
- ✅ Authentication & session management
- ✅ Transaction management dengan database consistency
- ✅ CRUD operations
- ✅ Error handling & validation

Aplikasi ini cocok untuk pembelajaran mata kuliah **Basis Data** karena:
- Schema yang relational & normalized
- Complex queries dengan JOINs & aggregations
- Transaction handling (ACID properties)
- Foreign key constraints
- Data integrity management

---

**Project Mata Kuliah: Basis Data** | GameHub E-Commerce Platform | 2026
