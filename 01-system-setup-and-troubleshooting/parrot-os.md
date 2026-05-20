# Modul 01: Log Sistem, Arsitektur Hardware & Troubleshooting Storage

Jurnal ini berfungsi sebagai cetak biru (blueprint) konfigurasi sistem operasi Parrot OS pada laptop utama, mencakup pemetaan partisi disk secara riil dan prosedur penanganan ruang penyimpanan yang terbatas.

---

## ─── SPECIFICATION & BASE SYSTEM ───

* **Processor:** Intel Pentium P6100
* **Operating System:** Parrot OS (Home Edition / Security Edition)
* **Environment:** single boot from dual boot [kali linux x windows 7]
* **Tujuan Sistem:** Lingkungan belajar mandiri Network Hacking, Python Development, dan Pentesting Lab.

---

## ─── DISK PARTITION MAP (ARSITEKTUR PENYIMPANAN) ───

Sistem penyimpanan dibagi secara spesifik untuk memisahkan ruang sistem operasi Linux, data umum,serta file root. Berikut adalah peta partisi riil pada harddisk:

| Kode Partisi | Kapasitas | Mount Point / Fungsi | Karakteristik & Kondisi |
| :--- | :--- | :--- | :--- |
| `/dev/sda1` | **98 GB** | `/` (Sebagai root system parrot os) | Sangat aman. Menampung system core, tools dari parrot, lingkungan dekstop, dan cachea paket. |
| `/dev/sda2` | **4 GB** | `swap` sebagai virtual memori cadangan agar prossesor dan ram tidak ngos ngosan |
| `/dev/sda3` | **149 GB** | `data` sebagai partisi data menyimpan file data seperti biasa |
| `/dev/sda4` | **46 GB** | `/home` sebagai tempat penyimpanan personal user ` name `, termasuk berkas konfigurasi lokal dan repo project |

---

## ─── LOG INSTALASI & TROUBLESHOOTING STORAGES ───

#### **Solusi Taktis 1: Pembersihan Cache Rutin**
Setiap selesai melakukan instalasi tools baru lewat terminal, wajib menjalankan perintah pembersihan untuk menghapus file instalasi `.deb` yang masih tertinggal di cache:
```bash
sudo apt update && upgrade
sudo apt clean
sudo apt autoremove
