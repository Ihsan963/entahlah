# 🧪 GameHub - API Testing Guide & cURL Examples

---

## 📡 API Base URL
```
http://localhost/gamehub/api
```

---

## 🔑 Authentication Endpoints

### **1. Login**

**Endpoint:** `POST /login.php`

**cURL Command:**
```bash
curl -X POST http://localhost/gamehub/api/login.php \
  -H "Content-Type: application/json" \
  -d '{
    "username": "Jabond",
    "password": "admin123"
  }'
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

**Response (Failed - Wrong Password):**
```json
{
  "success": false,
  "message": "Username atau password salah.",
  "data": [],
  "status": 401
}
```

**JavaScript Implementation:**
```javascript
async function login(username, password) {
  const response = await fetch('http://localhost/gamehub/api/login.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username, password })
  });
  
  const data = await response.json();
  
  if (data.success) {
    // Save to localStorage
    localStorage.setItem('pelanggan', JSON.stringify(data.data));
    window.location.href = 'index.html';
  } else {
    alert('Login gagal: ' + data.message);
  }
}
```

---

### **2. Register**

**Endpoint:** `POST /register.php`

**cURL Command:**
```bash
curl -X POST http://localhost/gamehub/api/register.php \
  -H "Content-Type: application/json" \
  -d '{
    "nama": "John Doe",
    "email": "john@example.com",
    "username": "johndoe",
    "password": "password123"
  }'
```

**Response (Success):**
```json
{
  "success": true,
  "message": "Registrasi berhasil.",
  "data": {
    "id_pelanggan": 4,
    "nama": "John Doe",
    "email": "john@example.com",
    "username": "johndoe"
  }
}
```

**Response (Failed - Username exists):**
```json
{
  "success": false,
  "message": "Username sudah terdaftar.",
  "data": [],
  "status": 400
}
```

**JavaScript Implementation:**
```javascript
async function register(nama, email, username, password) {
  const response = await fetch('http://localhost/gamehub/api/register.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ nama, email, username, password })
  });
  
  const data = await response.json();
  
  if (data.success) {
    alert('Registrasi berhasil! Silakan login.');
    window.location.href = 'login.html';
  } else {
    alert('Registrasi gagal: ' + data.message);
  }
}
```

---

## 🎮 Product Catalog Endpoints

### **3. Get All Games**

**Endpoint:** `GET /games/get_games.php`

**cURL Command:**
```bash
curl -X GET http://localhost/gamehub/api/games/get_games.php
```

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
    {
      "id_game": 5,
      "judul": "Cyberpunk 2077",
      "harga_utama": 699999,
      "genre": null,
      "gambar": "cyberpunk2077.jpg"
    },
    {
      "id_game": 10,
      "judul": "Digimon Story Time Stranger",
      "harga_utama": 899999,
      "genre": null,
      "gambar": "digimonstory.jpg"
    }
  ]
}
```

**JavaScript Implementation:**
```javascript
async function loadGames() {
  const response = await fetch('http://localhost/gamehub/api/games/get_games.php');
  const data = await response.json();
  
  if (data.success) {
    const games = data.data;
    const container = document.getElementById('game-container');
    
    container.innerHTML = games.map(game => `
      <div class="game-card">
        <img src="img/games/${game.gambar}" alt="${game.judul}">
        <div class="game-body">
          <h3 class="game-title">${game.judul}</h3>
          <p class="game-price">Rp ${game.harga_utama.toLocaleString('id-ID')}</p>
          <button onclick="beliItem('${game.judul}', ${game.harga_utama}, ${game.id_game}, 'game')">
            Beli
          </button>
        </div>
      </div>
    `).join('');
  }
}
```

---

### **4. Get All DLCs**

**Endpoint:** `GET /dlcs/get_dlcs.php`

**cURL Command:**
```bash
curl -X GET http://localhost/gamehub/api/dlcs/get_dlcs.php
```

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
    {
      "Id_dlc": 2,
      "id_game": 5,
      "nama_dlc": "Cyberpunk Expansion Pack",
      "harga_dlc": 299000,
      "tipe": "Ekspansi",
      "gambar": "cyberpunkdlc.jpg"
    }
  ]
}
```

---

### **5. Get All Bundles**

**Endpoint:** `GET /bundles/get_bundles.php`

**cURL Command:**
```bash
curl -X GET http://localhost/gamehub/api/bundles/get_bundles.php
```

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
    {
      "id_bundle": 2,
      "nama_bundle": "Persona 3 Joker",
      "harga_bundle": 900000,
      "deskripsi": "Joker in Persona 3",
      "gambar": "persona3+jokerbundle.jpg"
    },
    {
      "id_bundle": 3,
      "nama_bundle": "Cyberpunk Ultimate Bundle",
      "harga_bundle": 899000,
      "deskripsi": "Cyberpunk 2077 + Cyberpunk Expansion Pack",
      "gambar": "cyberpunkbundle.jpg"
    },
    {
      "id_bundle": 4,
      "nama_bundle": "Digimon Complete Bundle",
      "harga_bundle": 999000,
      "deskripsi": "Digimon Story Time Stranger + Alternate Dimension",
      "gambar": "digimonbundle.jpg"
    }
  ]
}
```

---

## 💳 Checkout & Transaction Endpoints

### **6. Process Checkout**

**Endpoint:** `POST /checkout/process_checkout.php`

**cURL Command:**
```bash
curl -X POST http://localhost/gamehub/api/checkout/process_checkout.php \
  -H "Content-Type: application/json" \
  -d '{
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
  }'
```

**Response (Success):**
```json
{
  "success": true,
  "id_transaksi": 12,
  "total_bayar": 2099000,
  "saldo_sisa": 7901000
}
```

**Response (Failed - Insufficient Balance):**
```json
{
  "error": "Saldo tidak cukup. Saldo kamu: Rp 500.000"
}
```

**Response (Failed - Empty Cart):**
```json
{
  "error": "Keranjang kosong"
}
```

**JavaScript Implementation:**
```javascript
async function checkout(id_pelanggan, cart) {
  const response = await fetch('http://localhost/gamehub/api/checkout/process_checkout.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ id_pelanggan, cart })
  });
  
  const data = await response.json();
  
  if (data.success) {
    alert(`Pembelian berhasil! Transaksi #${data.id_transaksi}`);
    alert(`Total: Rp ${data.total_bayar.toLocaleString('id-ID')}`);
    alert(`Saldo tersisa: Rp ${data.saldo_sisa.toLocaleString('id-ID')}`);
    
    // Clear cart
    localStorage.removeItem('cart');
    window.location.href = 'index.html';
  } else {
    alert('Checkout gagal: ' + data.error);
  }
}
```

---

### **7. Get Customer Transactions**

**Endpoint:** `GET /transactions/get_transactions.php?id_pelanggan=1`

**cURL Command:**
```bash
curl -X GET "http://localhost/gamehub/api/transactions/get_transactions.php?id_pelanggan=1"
```

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
    {
      "id_transaksi": 2,
      "tanggal_transaksi": "2026-06-04 17:48:30",
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
    }
  ]
}
```

**JavaScript Implementation:**
```javascript
async function loadTransactions(id_pelanggan) {
  const response = await fetch(`http://localhost/gamehub/api/transactions/get_transactions.php?id_pelanggan=${id_pelanggan}`);
  const data = await response.json();
  
  if (data.success) {
    const transactions = data.data;
    const container = document.getElementById('transaksi-container');
    
    container.innerHTML = transactions.map(tx => `
      <div class="tx-card">
        <h3>Transaksi #${tx.id_transaksi}</h3>
        <p class="tx-date">${new Date(tx.tanggal_transaksi).toLocaleString('id-ID')}</p>
        <ul>
          ${tx.items.map(item => `<li>${item.nama}</li>`).join('')}
        </ul>
        <p class="tx-total">Total: Rp ${tx.total_bayar.toLocaleString('id-ID')}</p>
      </div>
    `).join('');
  }
}
```

---

## 📊 Statistics & Reporting Endpoints

### **8. Get Dashboard Statistics**

**Endpoint:** `GET /statistics/get_dashboard_stats.php`

**cURL Command:**
```bash
curl -X GET http://localhost/gamehub/api/statistics/get_dashboard_stats.php
```

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

**JavaScript Implementation:**
```javascript
async function loadStatistics() {
  const response = await fetch('http://localhost/gamehub/api/statistics/get_dashboard_stats.php');
  const data = await response.json();
  
  if (data.success) {
    const stats = data.data;
    
    document.getElementById('total-customers').textContent = stats.total_customers;
    document.getElementById('total-transactions').textContent = stats.total_transactions;
    document.getElementById('total-revenue').textContent = 
      'Rp ' + stats.total_revenue.toLocaleString('id-ID');
    document.getElementById('avg-transaction').textContent = 
      'Rp ' + stats.average_transaction.toLocaleString('id-ID');
  }
}
```

---

### **9. Get Customer Info**

**Endpoint:** `GET /customers/get_customer.php?id=1`

**cURL Command:**
```bash
curl -X GET "http://localhost/gamehub/api/customers/get_customer.php?id=1"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id_pelanggan": 1,
    "nama": "Dhafin Madany",
    "email": "dhafin@example.com",
    "username": "dhafin",
    "saldo": 7901000,
    "tanggal_daftar": "2026-05-10 06:25:59"
  }
}
```

---

## 🧪 Testing Workflow

### **Full User Journey Test**

**1. Register new user:**
```bash
curl -X POST http://localhost/gamehub/api/register.php \
  -H "Content-Type: application/json" \
  -d '{
    "nama": "Test User",
    "email": "test@example.com",
    "username": "testuser",
    "password": "testpass123"
  }'
```

**2. Login:**
```bash
curl -X POST http://localhost/gamehub/api/login.php \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "testpass123"
  }'
```

**Note:** Save the `id_pelanggan` from response

**3. Get available games:**
```bash
curl -X GET http://localhost/gamehub/api/games/get_games.php
```

**4. Add to cart (localStorage in browser - simulated):**
```
Cart data:
[
  { id_game: 1, nama: "Persona 5 Royal", harga: 529000, tipe: "game", id_game: 1 }
]
```

**5. Checkout:**
```bash
curl -X POST http://localhost/gamehub/api/checkout/process_checkout.php \
  -H "Content-Type: application/json" \
  -d '{
    "id_pelanggan": 4,
    "cart": [
      {
        "nama": "Persona 5 Royal",
        "harga": 529000,
        "id": 1,
        "tipe": "game",
        "id_game": 1
      }
    ]
  }'
```

**6. Check transaction history:**
```bash
curl -X GET "http://localhost/gamehub/api/transactions/get_transactions.php?id_pelanggan=4"
```

**7. Check updated customer info:**
```bash
curl -X GET "http://localhost/gamehub/api/customers/get_customer.php?id=4"
```

---

## 🔍 Debugging Tips

### **Check if API is working:**
```bash
# Simple request
curl -X GET http://localhost/gamehub/api/games/get_games.php -v
```

**Expected output:**
```
< HTTP/1.1 200 OK
< Content-Type: application/json
...json response...
```

### **Check PHP errors:**
```bash
# Enable error reporting in PHP
# Edit config/db.php and add:
ini_set('display_errors', 1);
error_reporting(E_ALL);
```

### **Test database connection:**
```bash
# Check if MySQL is running
mysql -u root -p -e "SELECT 1;"

# Check gamehub database exists
mysql -u root -p -e "USE gamehub; SHOW TABLES;"
```

### **Check API POST data:**
```bash
# Use -v flag to see full request/response
curl -X POST http://localhost/gamehub/api/login.php \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"test"}' \
  -v
```

---

## 📋 API Response Format

All API responses follow this format:

**Success Response:**
```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... }
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "Error description",
  "data": [],
  "status": 400
}
```

**Status Codes:**
- `200` — OK (request successful)
- `201` — Created (resource created)
- `400` — Bad Request (validation error)
- `401` — Unauthorized (login failed)
- `404` — Not Found
- `500` — Server Error

---

## 🔐 CORS & Security Notes

**Current CORS Policy:**
```php
header("Access-Control-Allow-Origin: *");
```

**⚠️ WARNING:** This allows requests from ANY domain. For production:

```php
header("Access-Control-Allow-Origin: https://yourdomain.com");
header("Access-Control-Allow-Credentials: true");
header("Access-Control-Allow-Methods: GET, POST, PUT, DELETE");
header("Access-Control-Allow-Headers: Content-Type");
```

**Password Security:**
Currently storing plaintext passwords. Should use `password_hash()`:

```php
// Hash password on registration
$hashed = password_hash($password, PASSWORD_BCRYPT);

// Verify on login
password_verify($input_password, $hashed_from_db);
```

---

**API Testing Guide** | GameHub Project | 2026
