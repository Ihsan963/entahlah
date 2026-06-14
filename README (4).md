# 🌐 Enterprise Network HQ-Branch Simulation
### Tugas Modul 5 — Computer Engineering Lab M-IoT

> Simulasi jaringan enterprise yang menghubungkan kantor pusat **Jakarta (HQ)** dan kantor cabang **Surabaya (Branch)** menggunakan GRE Tunnel, OSPF, VRRP, dan berbagai teknologi jaringan lainnya di atas platform **PNETLab**.

---

## 📋 Daftar Isi

- [Gambaran Umum](#gambaran-umum)
- [Topologi Jaringan](#topologi-jaringan)
- [Perangkat yang Digunakan](#perangkat-yang-digunakan)
- [Tabel Addressing](#tabel-addressing)
- [Modul 1 — Cisco Switch Jakarta](#modul-1--cisco-switch-jakarta)
- [Modul 2 — Cisco Router Jakarta](#modul-2--cisco-router-jakarta)
- [Modul 3 — MikroTik Router Jakarta](#modul-3--mikrotik-router-jakarta)
- [Modul 4 — Ubuntu Server Jakarta](#modul-4--ubuntu-server-jakarta)
- [Modul 5 — FortiGate Jakarta](#modul-5--fortigate-jakarta)
- [Modul 6 — MikroTik ISP](#modul-6--mikrotik-isp)
- [Modul 7 — Switch & MikroTik Surabaya](#modul-7--switch--mikrotik-surabaya)
- [Modul 8 — FortiGate Surabaya](#modul-8--fortigate-surabaya)
- [Modul 9 — Verifikasi GRE Tunnel & OSPF](#modul-9--verifikasi-gre-tunnel--ospf)
- [Modul 10 — Pengujian Akhir](#modul-10--pengujian-akhir)

---

## 📖 Gambaran Umum

Proyek ini merupakan simulasi jaringan enterprise skala menengah yang terdiri dari dua site:

| Site | Lokasi | Fungsi |
|------|--------|--------|
| **HQ** | Jakarta | Kantor Pusat — Cisco Router, MikroTik Router, Ubuntu Server, FortiGate |
| **Branch** | Surabaya | Kantor Cabang — MikroTik Router, Cisco Switch, FortiGate |
| **ISP** | — | Simulasi Provider — MikroTik RouterOS |

Kedua site dihubungkan menggunakan **GRE Tunnel** antar FortiGate, dengan **OSPF** berjalan di atas tunnel untuk pertukaran routing dinamis antar site.

---

## 🗺️ Topologi Jaringan

<!-- Tempel screenshot topologi PNETLab di sini -->
![Topologi Jaringan](screenshots/topology.png)

---

## 🖥️ Perangkat yang Digunakan

| Perangkat | OS / Tipe | Fungsi |
|-----------|-----------|--------|
| **vIOS-Jakarta** | Cisco IOS (vIOS) | Sub-interface VLAN, VRRP Master VLAN 10 & 60, DHCP Relay |
| **Switch-Jakarta** | Cisco IOS Switch | VLAN 10/20/60, Trunk ke Cisco & MikroTik |
| **Mikrotik-Jakarta** | MikroTik RouterOS | VLAN interface, VRRP Master VLAN 20, DHCP Relay |
| **Ubuntu VLAN60** | Ubuntu Server | ISC-DHCP Server (VLAN 10 & 20) + Nginx Web Server |
| **FortiGate Jakarta** | FortiOS (VM64-KVM) | Edge Firewall, NAT, GRE Tunnel endpoint, OSPF |
| **Mikrotik-ISP** | MikroTik RouterOS | Simulasi ISP, NAT Masquerade ke Cloud |
| **FortiGate Surabaya** | FortiOS (VM64-KVM) | Edge Firewall, NAT, GRE Tunnel endpoint, OSPF |
| **Mikrotik Surabaya** | MikroTik RouterOS | Gateway VLAN 30 (DHCP) & VLAN 40 |
| **Switch Surabaya** | Cisco IOS Switch | VLAN 30/40, Trunk ke MikroTik Surabaya |
| **VLAN10 / VLAN20** | VPCS | Client Jakarta (Finance & IT) |
| **VLAN30** | VPCS | Client Surabaya Sales (DHCP) |
| **VLAN40** | VPCS | Client Surabaya Operations (Static IP) |
| **Tinycore VLAN40** | Tinycore Linux | Client Surabaya Operations (browser test) |

---

## 📊 Tabel Addressing

### Jakarta / HQ

#### VLAN Jakarta
| VLAN | Nama | Network | Gateway Virtual (VRRP) | DHCP Server |
|------|------|---------|------------------------|-------------|
| 10 | FINANCE | 192.168.10.0/24 | 192.168.10.1 | Ubuntu Server Jakarta |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | Ubuntu Server Jakarta |
| 60 | SERVER-HQ | 192.168.60.0/24 | 192.168.60.1 | — (Static) |

#### Cisco Router Jakarta (vIOS-Jakarta)
| Interface | IP Address | Keterangan |
|-----------|------------|------------|
| Gi0/0 | 10.10.100.1/30 | Link ke FortiGate Jakarta port1 |
| Gi0/1.10 | 192.168.10.2/24 | Sub-interface VLAN 10 |
| Gi0/1.20 | 192.168.20.2/24 | Sub-interface VLAN 20 |
| Gi0/1.60 | 192.168.60.2/24 | Sub-interface VLAN 60 |

#### MikroTik Router Jakarta
| Interface | IP Address | Keterangan |
|-----------|------------|------------|
| ether1 | 10.10.101.2/30 | Link ke FortiGate Jakarta port2 |
| vlan10-finance | 192.168.10.3/24 | IP fisik VLAN 10 |
| vlan20-it | 192.168.20.3/24 | IP fisik VLAN 20 |
| vlan60-server | 192.168.60.3/24 | IP fisik VLAN 60 |

#### VRRP Jakarta
| VLAN | Virtual IP | Master | Backup |
|------|------------|--------|--------|
| 10 | 192.168.10.1 | Cisco Router (priority 110) | MikroTik (priority 90) |
| 20 | 192.168.20.1 | MikroTik (priority 110) | Cisco Router (priority 90) |
| 60 | 192.168.60.1 | Cisco Router (priority 110) | MikroTik (priority 90) |

#### FortiGate Jakarta
| Interface | IP | Terhubung ke |
|-----------|-----|-------------|
| port1 | 10.10.100.2/30 | Cisco Router Jakarta |
| port2 | 10.10.101.1/30 | MikroTik Jakarta |
| port3 | 10.0.12.2/30 | MikroTik ISP |
| GRE-JKT-SBY | 172.16.0.1/32 | Tunnel ke FortiGate Surabaya |

#### Ubuntu Server Jakarta
| IP Address | Gateway | Service |
|------------|---------|---------|
| 192.168.60.10/24 | 192.168.60.1 | ISC-DHCP Server + Nginx Web Server |

#### DHCP Pool Jakarta
| VLAN | Range | Gateway |
|------|-------|---------|
| 10 | 192.168.10.100 – 192.168.10.200 | 192.168.10.1 |
| 20 | 192.168.20.100 – 192.168.20.200 | 192.168.20.1 |

---

### ISP

| Interface | IP | Keterangan |
|-----------|-----|-----------|
| ether1 | DHCP dinamis (Cloud PNETLab) | Akses internet |
| ether2 | 10.0.12.1/30 | Link ke FortiGate Jakarta |
| ether3 | 10.0.13.1/30 | Link ke FortiGate Surabaya |

---

### Surabaya / Branch

#### VLAN Surabaya
| VLAN | Nama | Network | Gateway | Keterangan |
|------|------|---------|---------|------------|
| 30 | SALES | 192.168.30.0/24 | 192.168.30.1 | DHCP dari MikroTik Surabaya |
| 40 | OPERATIONS | 192.168.40.0/24 | 192.168.40.1 | IP Static manual |

#### MikroTik Router Surabaya
| Interface | IP | Keterangan |
|-----------|-----|-----------|
| ether1 | 10.10.200.2/30 | Link ke FortiGate Surabaya |
| vlan30-sales | 192.168.30.1/24 | Gateway VLAN 30 |
| vlan40-operations | 192.168.40.1/24 | Gateway VLAN 40 |

#### FortiGate Surabaya
| Interface | IP | Terhubung ke |
|-----------|-----|-------------|
| port1 | 10.0.13.2/30 | MikroTik ISP |
| port2 | 10.10.200.1/30 | MikroTik Surabaya |
| GRE-SBY-JKT | 172.16.0.2/32 | Tunnel ke FortiGate Jakarta |

#### IP Client Surabaya
| Client | VLAN | IP | Gateway |
|--------|------|----|---------|
| PC Sales (VPCS) | 30 | DHCP (range 192.168.30.100–200) | 192.168.30.1 |
| PC Operations (VPCS) | 40 | 192.168.40.10/24 (static) | 192.168.40.1 |
| Tinycore Linux | 40 | 192.168.40.20/24 (static) | 192.168.40.1 |

---

### GRE Tunnel

| Tunnel | Perangkat | Local WAN | Remote WAN | Tunnel IP |
|--------|-----------|-----------|------------|-----------|
| GRE-JKT-SBY | FortiGate Jakarta | 10.0.12.2 | 10.0.13.2 | 172.16.0.1/32 |
| GRE-SBY-JKT | FortiGate Surabaya | 10.0.13.2 | 10.0.12.2 | 172.16.0.2/32 |

### Network via OSPF

| Site | Network | Keterangan |
|------|---------|------------|
| Jakarta | 192.168.10.0/24 | VLAN 10 Finance |
| Jakarta | 192.168.20.0/24 | VLAN 20 IT |
| Jakarta | 192.168.60.0/24 | VLAN 60 Server |
| Jakarta | 172.16.0.1/32 | GRE Tunnel Jakarta |
| Surabaya | 192.168.30.0/24 | VLAN 30 Sales |
| Surabaya | 192.168.40.0/24 | VLAN 40 Operations |
| Surabaya | 172.16.0.2/32 | GRE Tunnel Surabaya |

---

## Modul 1 — Cisco Switch Jakarta

**Hal yang dikonfigurasi:** VLAN 10 (FINANCE), VLAN 20 (IT), VLAN 60 (SERVER-HQ), access port client, trunk ke Cisco Router dan MikroTik Router.

### Bukti

**1. `show vlan brief`**

<!-- Tempel screenshot show vlan brief di sini -->
![show vlan brief](screenshots/m1-show-vlan-brief.png)

**2. `show interfaces trunk`**

<!-- Tempel screenshot show interfaces trunk di sini -->
![show interfaces trunk](screenshots/m1-show-interfaces-trunk.png)

---

## Modul 2 — Cisco Router Jakarta

**Hal yang dikonfigurasi:** Sub-interface VLAN 10/20/60, VRRP (Master VLAN 10 & 60), DHCP Relay ke Ubuntu Server, link ke FortiGate Jakarta.

### Bukti

**1. `show ip interface brief`**

<!-- Tempel screenshot show ip interface brief di sini -->
![show ip interface brief](screenshots/m2-show-ip-int-brief.png)

**2. `show vrrp brief`**

<!-- Tempel screenshot show vrrp brief di sini -->
![show vrrp brief](screenshots/m2-show-vrrp-brief.png)

**3. Screenshot konfigurasi subinterface**

<!-- Tempel screenshot konfigurasi subinterface di sini -->
![konfigurasi subinterface](screenshots/m2-subinterface-config.png)

**4. Ping dari Cisco Router ke FortiGate Jakarta (`ping 10.10.100.1`)**

<!-- Tempel screenshot ping di sini -->
![ping ke FortiGate Jakarta](screenshots/m2-ping-fortigate.png)

---

## Modul 3 — MikroTik Router Jakarta

**Hal yang dikonfigurasi:** VLAN interface 10/20/60, VRRP (Master VLAN 20), DHCP Relay, default route ke FortiGate Jakarta.

### Bukti

**1. `/ip address print`**

<!-- Tempel screenshot /ip address print di sini -->
![ip address print](screenshots/m3-ip-address-print.png)

**2. `/interface vrrp print`**

<!-- Tempel screenshot /interface vrrp print di sini -->
![interface vrrp print](screenshots/m3-vrrp-print.png)

**3. `/ip dhcp-relay print`**

<!-- Tempel screenshot /ip dhcp-relay print di sini -->
![dhcp relay print](screenshots/m3-dhcp-relay-print.png)

**4. `/ip route print`**

<!-- Tempel screenshot /ip route print di sini -->
![ip route print](screenshots/m3-ip-route-print.png)

**5. Ping dari MikroTik ke FortiGate Jakarta (`ping 10.10.101.1`)**

<!-- Tempel screenshot ping di sini -->
![ping ke FortiGate Jakarta](screenshots/m3-ping-fortigate.png)

---

## Modul 4 — Ubuntu Server Jakarta

**Hal yang dikonfigurasi:** IP static 192.168.60.10/24, ISC-DHCP Server (VLAN 10 & 20), Nginx Web Server.

> ⚠️ **Troubleshooting:** Hubungkan Ubuntu ke network management dulu, install paket, baru pindah ke Switch VLAN 60 dan set IP static.

### Bukti

**1. `ip a`**

<!-- Tempel screenshot ip a di sini -->
![ip a](screenshots/m4-ip-a.png)

**2. `ip route`**

<!-- Tempel screenshot ip route di sini -->
![ip route](screenshots/m4-ip-route.png)

**3. Isi file `/etc/dhcp/dhcpd.conf`**

<!-- Tempel screenshot isi dhcpd.conf di sini -->
![dhcpd.conf](screenshots/m4-dhcpd-conf.png)

**4. `ping 8.8.8.8`**

<!-- Tempel screenshot ping 8.8.8.8 di sini -->
![ping 8.8.8.8](screenshots/m4-ping-internet.png)

---

## Modul 5 — FortiGate Jakarta

**Hal yang dikonfigurasi:** Interface port1/port2/port3, default route, static route internal Jakarta, firewall policy + NAT, GRE Tunnel ke Surabaya, OSPF over GRE, redistribute static.

### Bukti

**1. `get system interface physical`**

<!-- Tempel screenshot get system interface physical di sini -->
![get system interface physical](screenshots/m5-interface-physical.png)

**2. `get router info routing-table all`**

<!-- Tempel screenshot routing table di sini -->
![routing table all](screenshots/m5-routing-table-all.png)

**3. Screenshot firewall policy**

<!-- Tempel screenshot firewall policy di sini -->
![firewall policy](screenshots/m5-firewall-policy.png)

**4. `execute ping 8.8.8.8`**

<!-- Tempel screenshot ping 8.8.8.8 di sini -->
![ping 8.8.8.8](screenshots/m5-ping-internet.png)

**5. `execute ping 172.16.0.2` (IP tunnel Surabaya)**

<!-- Tempel screenshot ping tunnel di sini -->
![ping tunnel Surabaya](screenshots/m5-ping-tunnel.png)

**6. `get router info ospf neighbor`**

<!-- Tempel screenshot ospf neighbor di sini -->
![ospf neighbor](screenshots/m5-ospf-neighbor.png)

**7. `get router info routing-table ospf`**

<!-- Tempel screenshot routing table ospf di sini -->
![routing table ospf](screenshots/m5-routing-table-ospf.png)

---

## Modul 6 — MikroTik ISP

**Hal yang dikonfigurasi:** IP ether2 & ether3, DHCP client ether1 (Cloud NAT), default route, NAT masquerade.

> ℹ️ MikroTik ISP **tidak** menjalankan OSPF. IP ether1 bersifat dinamis (bisa berbeda setiap sesi).

### Bukti

**1. `/ip address print`**

<!-- Tempel screenshot /ip address print di sini -->
![ip address print](screenshots/m6-ip-address-print.png)

**2. `/ip route print`**

<!-- Tempel screenshot /ip route print di sini -->
![ip route print](screenshots/m6-ip-route-print.png)

**3. `/ip firewall nat print`**

<!-- Tempel screenshot firewall nat di sini -->
![firewall nat print](screenshots/m6-firewall-nat.png)

**4. `ping 8.8.8.8`**

<!-- Tempel screenshot ping 8.8.8.8 di sini -->
![ping 8.8.8.8](screenshots/m6-ping-internet.png)

**5. Ping antar WAN FortiGate (ping 10.0.12.2 & ping 10.0.13.2)**

<!-- Tempel screenshot ping antar WAN di sini -->
![ping antar WAN FortiGate](screenshots/m6-ping-wan.png)

---

## Modul 7 — Switch & MikroTik Surabaya

**Hal yang dikonfigurasi:** VLAN 30 (SALES) & VLAN 40 (OPERATIONS) di Switch, trunk ke MikroTik Surabaya, VLAN interface di MikroTik, DHCP Server VLAN 30, IP static VLAN 40.

### Bukti Switch Surabaya

**1. `show vlan brief`**

<!-- Tempel screenshot show vlan brief di sini -->
![show vlan brief Surabaya](screenshots/m7-switch-vlan-brief.png)

**2. `show interfaces trunk`**

<!-- Tempel screenshot show interfaces trunk di sini -->
![show interfaces trunk Surabaya](screenshots/m7-switch-trunk.png)

### Bukti MikroTik Surabaya

**3. `/ip address print`**

<!-- Tempel screenshot /ip address print di sini -->
![ip address print](screenshots/m7-mt-ip-address.png)

**4. `/ip dhcp-server print`**

<!-- Tempel screenshot /ip dhcp-server print di sini -->
![dhcp server print](screenshots/m7-mt-dhcp-server.png)

**5. `/ip pool print`**

<!-- Tempel screenshot /ip pool print di sini -->
![ip pool print](screenshots/m7-mt-ip-pool.png)

**6. `/ip route print`**

<!-- Tempel screenshot /ip route print di sini -->
![ip route print](screenshots/m7-mt-ip-route.png)

**7. Client VLAN 30 mendapat IP DHCP**

<!-- Tempel screenshot ip dhcp di VPCS di sini -->
![DHCP client VLAN 30](screenshots/m7-vlan30-dhcp.png)

**8. Ping client Surabaya ke 8.8.8.8**

<!-- Tempel screenshot ping 8.8.8.8 dari VLAN30 di sini -->
![ping internet dari Surabaya](screenshots/m7-vlan30-ping-internet.png)

---

## Modul 8 — FortiGate Surabaya

**Hal yang dikonfigurasi:** Interface port1/port2, default route ke ISP, static route VLAN Surabaya, firewall policy + NAT, GRE Tunnel ke Jakarta, OSPF over GRE, redistribute static.

### Bukti

**1. `get system interface physical`**

<!-- Tempel screenshot get system interface physical di sini -->
![get system interface physical](screenshots/m8-interface-physical.png)

**2. `get router info routing-table all`**

<!-- Tempel screenshot routing table all di sini -->
![routing table all](screenshots/m8-routing-table-all.png)

**3. Screenshot firewall policy (part 1 & 2)**

<!-- Tempel screenshot firewall policy part 1 di sini -->
![firewall policy part 1](screenshots/m8-firewall-policy-1.png)

<!-- Tempel screenshot firewall policy part 2 di sini -->
![firewall policy part 2](screenshots/m8-firewall-policy-2.png)

**4. `execute ping 8.8.8.8`**

<!-- Tempel screenshot ping 8.8.8.8 di sini -->
![ping 8.8.8.8](screenshots/m8-ping-internet.png)

**5. `execute ping 172.16.0.1` (IP tunnel Jakarta)**

<!-- Tempel screenshot ping tunnel Jakarta di sini -->
![ping tunnel Jakarta](screenshots/m8-ping-tunnel.png)

**6. `get router info ospf neighbor`**

<!-- Tempel screenshot ospf neighbor di sini -->
![ospf neighbor](screenshots/m8-ospf-neighbor.png)

**7. `get router info routing-table ospf`**

<!-- Tempel screenshot routing table ospf di sini -->
![routing table ospf](screenshots/m8-routing-table-ospf.png)

---

## Modul 9 — Verifikasi GRE Tunnel & OSPF

**Verifikasi:** Ping WAN antar FortiGate, ping IP tunnel, OSPF neighbor Full, routing table OSPF, ping antar client Jakarta dan Surabaya.

### Bukti FortiGate Jakarta

**1. Ping WAN ke Surabaya & Ping IP Tunnel + OSPF neighbor + routing table OSPF**

<!-- Tempel screenshot ping WAN JKT→SBY, ping tunnel, ospf neighbor, routing table di sini -->
![verifikasi FortiGate Jakarta](screenshots/m9-jkt-verify.png)

### Bukti FortiGate Surabaya

**2. Ping WAN ke Jakarta & Ping IP Tunnel + OSPF neighbor + routing table OSPF**

<!-- Tempel screenshot ping WAN SBY→JKT, ping tunnel, ospf neighbor, routing table di sini -->
![verifikasi FortiGate Surabaya](screenshots/m9-sby-verify.png)

### Ping Antar Client

**3. Ping dari VLAN10 (Client Jakarta) ke VLAN30 (Client Surabaya)**

<!-- Tempel screenshot ping VLAN10 → VLAN30 di sini -->
![ping Jakarta ke Surabaya](screenshots/m9-ping-jkt-to-sby.png)

**4. Ping dari VLAN30 (Client Surabaya) ke VLAN10 (Client Jakarta)**

<!-- Tempel screenshot ping VLAN30 → VLAN10 di sini -->
![ping Surabaya ke Jakarta](screenshots/m9-ping-sby-to-jkt.png)

---

## Modul 10 — Pengujian Akhir

### Bukti

**1. Client VLAN10 Jakarta — IP DHCP dari Ubuntu Server**

<!-- Tempel screenshot ip dhcp VLAN10 di sini -->
![DHCP VLAN10](screenshots/m10-vlan10-dhcp.png)

**2. Client VLAN20 Jakarta — IP DHCP dari Ubuntu Server**

<!-- Tempel screenshot ip dhcp VLAN20 di sini -->
![DHCP VLAN20](screenshots/m10-vlan20-dhcp.png)

**3. Client Jakarta ping internet (8.8.8.8)**

<!-- Tempel screenshot ping 8.8.8.8 dari Jakarta di sini -->
![ping internet Jakarta](screenshots/m10-jkt-ping-internet.png)

**4. Client Surabaya ping internet (8.8.8.8)**

<!-- Tempel screenshot ping 8.8.8.8 dari Surabaya di sini -->
![ping internet Surabaya](screenshots/m10-sby-ping-internet.png)

**5. Ping antar site**

<!-- Tempel screenshot ping antar site di sini -->
![ping antar site](screenshots/m10-ping-antar-site.png)

**6. Akses Web Server Jakarta dari Tinycore Surabaya (`http://192.168.60.10`)**

<!-- Tempel screenshot akses web server di sini -->
![akses web server Jakarta](screenshots/m10-webserver-access.png)

**7. `get router info routing-table ospf` — FortiGate Jakarta**

<!-- Tempel screenshot routing table ospf FortiGate Jakarta di sini -->
![routing table ospf Jakarta](screenshots/m10-ospf-table-jkt.png)

**8. `get router info routing-table ospf` — FortiGate Surabaya**

<!-- Tempel screenshot routing table ospf FortiGate Surabaya di sini -->
![routing table ospf Surabaya](screenshots/m10-ospf-table-sby.png)

**9. Analisis Singkat Jalur Traffic Jakarta ke Surabaya**

> "Traffic dari client Jakarta (VLAN10/20/60) diteruskan melalui Cisco Router atau MikroTik Router Jakarta menuju FortiGate Jakarta via link transit (10.10.100.0/30 dan 10.10.101.0/30). FortiGate Jakarta kemudian meneruskan traffic melalui GRE Tunnel (GRE-JKT-SBY) dengan IP tunnel 172.16.0.1 menuju FortiGate Surabaya (172.16.0.2) melewati jaringan WAN ISP (10.0.12.0/30 → 10.0.13.0/30). Route antar-site didistribusikan secara dinamis menggunakan OSPF over GRE. Setelah sampai di FortiGate Surabaya, traffic diteruskan ke MikroTik Surabaya dan akhirnya ke client VLAN30/40."

---

## ✅ Hasil Pengujian Akhir

| Pengujian | Hasil |
|-----------|-------|
| Client VLAN 10 Jakarta mendapat IP DHCP | ✅ |
| Client VLAN 20 Jakarta mendapat IP DHCP | ✅ |
| Client VLAN 30 Surabaya mendapat IP DHCP | ✅ |
| Client VLAN 40 Surabaya menggunakan IP static | ✅ |
| Client Jakarta ping 8.8.8.8 | ✅ |
| Client Surabaya ping 8.8.8.8 | ✅ |
| Client Jakarta ping Client Surabaya (antar site) | ✅ |
| Client Surabaya ping Client Jakarta | ✅ |
| OSPF neighbor antar FortiGate berstatus Full | ✅ |
| Route antar site tersebar via OSPF (O E2) | ✅ |
| Tinycore Surabaya akses `http://192.168.60.10` | ✅ |

---

## 🚦 Jalur Traffic

```
Client Jakarta → Internet
  VLAN 10/20 Client
    → VRRP Gateway (Cisco/MikroTik Jakarta)
    → FortiGate Jakarta (NAT + Firewall)
    → MikroTik ISP
    → Cloud NAT / Internet

Client Surabaya → Web Server Jakarta
  VLAN 30/40 Client
    → MikroTik Surabaya
    → FortiGate Surabaya
    → GRE Tunnel (172.16.0.2 → 172.16.0.1)
    → FortiGate Jakarta
    → Cisco/MikroTik Jakarta
    → Ubuntu Server (192.168.60.10)
```

---

## ⚙️ Urutan Pengerjaan

```
1. Modul 1  — Cisco Switch Jakarta      (VLAN + Trunk)
2. Modul 2  — Cisco Router Jakarta      (Sub-if + VRRP + DHCP Relay)
3. Modul 3  — MikroTik Jakarta          (VLAN + VRRP + DHCP Relay)
4. Modul 4  — Ubuntu Server Jakarta     (ISC-DHCP + Nginx)
              ⚠️ Install paket dulu di network management, lalu pindah ke Switch
5. Modul 5  — FortiGate Jakarta         (Interface + NAT + GRE + OSPF)
6. Modul 6  — MikroTik ISP             (IP + NAT Masquerade)
7. Modul 7  — Switch & MikroTik SBY    (VLAN 30/40 + DHCP)
8. Modul 8  — FortiGate Surabaya        (Interface + NAT + GRE + OSPF)
9. Modul 9  — Verifikasi GRE + OSPF    (Ping tunnel + OSPF neighbor Full)
10. Modul 10 — Pengujian Akhir          (End-to-end connectivity)
```

---

## 📁 Struktur Repositori

```
📦 enterprise-network-hq-branch
 ┣ 📜 README.md
 ┣ 📂 screenshots/
 ┃  ┣ topology.png
 ┃  ┣ m1-show-vlan-brief.png
 ┃  ┣ m1-show-interfaces-trunk.png
 ┃  ┣ m2-show-ip-int-brief.png
 ┃  ┣ m2-show-vrrp-brief.png
 ┃  ┣ m2-subinterface-config.png
 ┃  ┣ m2-ping-fortigate.png
 ┃  ┣ m3-ip-address-print.png
 ┃  ┣ m3-vrrp-print.png
 ┃  ┣ m3-dhcp-relay-print.png
 ┃  ┣ m3-ip-route-print.png
 ┃  ┣ m3-ping-fortigate.png
 ┃  ┣ m4-ip-a.png
 ┃  ┣ m4-ip-route.png
 ┃  ┣ m4-dhcpd-conf.png
 ┃  ┣ m4-ping-internet.png
 ┃  ┣ m5-interface-physical.png
 ┃  ┣ m5-routing-table-all.png
 ┃  ┣ m5-firewall-policy.png
 ┃  ┣ m5-ping-internet.png
 ┃  ┣ m5-ping-tunnel.png
 ┃  ┣ m5-ospf-neighbor.png
 ┃  ┣ m5-routing-table-ospf.png
 ┃  ┣ m6-ip-address-print.png
 ┃  ┣ m6-ip-route-print.png
 ┃  ┣ m6-firewall-nat.png
 ┃  ┣ m6-ping-internet.png
 ┃  ┣ m6-ping-wan.png
 ┃  ┣ m7-switch-vlan-brief.png
 ┃  ┣ m7-switch-trunk.png
 ┃  ┣ m7-mt-ip-address.png
 ┃  ┣ m7-mt-dhcp-server.png
 ┃  ┣ m7-mt-ip-pool.png
 ┃  ┣ m7-mt-ip-route.png
 ┃  ┣ m7-vlan30-dhcp.png
 ┃  ┣ m7-vlan30-ping-internet.png
 ┃  ┣ m8-interface-physical.png
 ┃  ┣ m8-routing-table-all.png
 ┃  ┣ m8-firewall-policy-1.png
 ┃  ┣ m8-firewall-policy-2.png
 ┃  ┣ m8-ping-internet.png
 ┃  ┣ m8-ping-tunnel.png
 ┃  ┣ m8-ospf-neighbor.png
 ┃  ┣ m8-routing-table-ospf.png
 ┃  ┣ m9-jkt-verify.png
 ┃  ┣ m9-sby-verify.png
 ┃  ┣ m9-ping-jkt-to-sby.png
 ┃  ┣ m9-ping-sby-to-jkt.png
 ┃  ┣ m10-vlan10-dhcp.png
 ┃  ┣ m10-vlan20-dhcp.png
 ┃  ┣ m10-jkt-ping-internet.png
 ┃  ┣ m10-sby-ping-internet.png
 ┃  ┣ m10-ping-antar-site.png
 ┃  ┣ m10-webserver-access.png
 ┃  ┣ m10-ospf-table-jkt.png
 ┗  ┗ m10-ospf-table-sby.png
```

---

> **Laboratorium M-IoT — Computer Engineering**
> Platform: PNETLab | Tools: FortiOS, MikroTik RouterOS, Cisco IOS, Ubuntu Server, Tinycore Linux
