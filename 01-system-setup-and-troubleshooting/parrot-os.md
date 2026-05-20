# Log Sistem & Konfigurasi Parrot OS

## 1. Spesifikasi & Manajemen Penyimpanan Hardware
* **Device:** Laptop Intel Pentium P6100.
* **Kendala Ruang Disk:** Partisi root `/dev/sda5` hanya berkapasitas 19GB dan cepat penuh saat instalasi tools/update, sementara partisi data `/dev/sda3` masih sangat lega (150GB).
* **Solusi/Mitigasi:** Rutin membersihkan cache instalasi dengan perintah `sudo apt clean`. Ke depan, folder besar yang memakan ruang akan dialihkan menggunakan metode *symbolic link* (symlink) ke `/dev/sda3`.

## 2. Troubleshooting Repositori & Git
* **Error Repositori:** Sempat terjadi kegagalan pencarian package karena kesalahan konfigurasi list repositori. Solusinya adalah mengoreksi jalur mirror ke repositori resmi Parrot OS yang stabil.
* **Keamanan Kredensial Git (Lokal):** Untuk menjaga privasi, identitas Git dikonfigurasi secara lokal (hanya berlaku di folder ini) menggunakan email samaran GitHub agar email utama tidak terekspos ke publik.
