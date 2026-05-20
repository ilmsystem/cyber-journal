Tahap 1 >
IP dan Interface : kita wajib tahu alamat rumah kita di jaringan
Host discovery : siapa saja yang satu jaringan tanpa memicu kecurigaan
Scanning : cek port, deteksi os , dan mencari versi aplikasi yang rentan pada target

---

jurnal ini berisi *networking reconnaissance*

## ─── 🧭 TAHAP 1: IDENTIFIKASI IP & INTERFACE LOKAL ───


Sebelum melakukan pemindaian ke target, langkah pertama yang wajib dilakukan adalah mengetahui IP Address laptop kita dan interface jaringan yang sedang aktif.


### 1. Mengecek IP Address dan Interface Aktif
Buka terminal dan jalankan salah satu perintah berikut:
```bash
ip a / ip link show.
```
cari interface nirkabel biasanya `wlan0` `wlp2s0` dibawah interface tersebut perhatikan `inet`. itulah ip address kamu 

### 2. Jalur gateway router
untuk mengetahui ip router atau gerbang utama jaringan tempat kalian berada `ip route show`
> logika analisis : ip yang diawali `default via` adalah ip router target

​## ─── 📊 TAHAP 2: PEMETAAN TETANGGA (HOST DISCOVERY) ───
​Setelah mengetahui rentang IP jaringan (misal: 192.168.100.000/24), langkah selanjutnya adalah melacak perangkat apa saja (HP, Laptop, IoT) yang sedang terhubung ke Wi-Fi tersebut.

use tools `nmap`
### 1. ping scan 
perintah ini kita gunakan di tools nmap untuk mendeteksis port yang hidup tanpa tersentuh "ramah" 
```bash
nmap -sn [ip host]
```
> `-sn` : mematikan pemindaian dan hanya mengirim paket ICMP echo request untuk melihat respon

​─── 🛠️ TAHAP 3: PEMINDAIAN TARGET SPESIFIK (DEEP SCAN) ───
​Pilih salah satu IP hidup yang kamu temukan di Tahap 2 untuk dijadikan target analisis celah keamanan (misal target: 192.168.1.50).
​1. Fast Scan (Skrining Awal)
​Memindai 100 port paling populer untuk melihat pintu masuk utama secara cepat.
```bash
nmap -F [ip host]
```
2. Syn Stealth Scan + OS Detection (Menu Wajib)
​Memindai seluruh port standar (1000 port) secara setengah terbuka (half-open) agar tidak tercatat di log aplikasi target, sekaligus menebak Sistem Operasi yang dipakai target.
```bash
nmap -sS -0 [ip host]
```
3. mencari celah
 cari tahu versi aplikasi apa saja yang berada di port terbuka . informasi ini krusial bisa kita cocokan dengan project `exploit`
```bash
nmap -sV  -v [ip host]
```
​-sV: Memaksa Nmap melakukan interaksi dengan port terbuka untuk mendapatkan banner versi (misal: SSH 7.9p1, Apache 2.4.38).
​-v: Verbose mode, menampilkan port yang terbuka secara langsung di layar tanpa menunggu proses scan selesai 100%.

4. aggresif scan
   ini saya sarankan gunakan ketika sedang simulasi saja dan ketika kamu tidak perduli firewall-nya,
```bash
nmap -A [ip host]
```
`-A` untuk gabungan deteksi os dan versi layanan, script scanning
​─── 🛑 EVASION: ATURAN KECEPATAN SCANNING ───
​[!WARNING]
Kecepatan pemindaian yang terlalu tinggi dapat memicu proteksi Firewall atau Intrusion Detection System (IDS) target yang berujung pada pemblokiran IP kamu.
​Gunakan parameter -T (Timing) untuk mengatur ritme detak jantung pemindaian Nmap:
​nmap -sS -T2 192.168.100.500 -> Polite: Lambat, memberi jeda antar paket, sangat aman untuk menghindari deteksi.
​nmap -sS -T4 192.168.100.500 -> Aggressive: Sangat cepat, cocok untuk efisiensi waktu di jaringan lab sendiri.

## ─── 🔍 PERINCIAN ISTILAH & MEKANISME KERJA TAHAP 3 ───


Untuk memahami hasil pemindaian Nmap secara mendalam, berikut adalah penjelasan detail mengenai mekanisme dan istilah teknis yang digunakan:


### 1. Istilah Status Port pada Nmap
Saat pemindaian selesai, Nmap akan mengategorikan port target ke dalam beberapa status utama:
* **`open` (Terbuka):** Aplikasi atau layanan aktif mendengarkan (*listening*) koneksi masuk pada port ini. Ini adalah target utama eksploitasi.
* **`closed` (Tertutup):** Port dapat diakses (menerima paket Nmap), tetapi tidak ada aplikasi yang berjalan atau mendengarkan di sana.
* **`filtered` (Terfilter):** Nmap tidak dapat menentukan apakah port terbuka atau tertutup karena paket data dihalangi oleh perangkat keamanan seperti *Firewall* atau aturan *iptables*.


---


### 2. Bedah Mekanisme Perintah Utama


#### A. SYN Stealth Scan (`-sS`) — Cara Kerja "Setengah Terbuka"
Disebut *stealth* karena tidak pernah menyelesaikan jabat tangan TCP secara utuh (*Three-Way Handshake*), sehingga meminimalkan pencatatan log pada sistem target.


**Mekanisme Jabat Tangan:**
1. Laptop kamu (`user@linux`) mengirim paket dengan flag **`SYN`** (Synchronization) ke port target.
2. **Jika Port Terbuka:** Target merespons dengan flag **`SYN-ACK`** (Acknowledgment).
3. Laptop kamu langsung membalas dengan flag **`RST`** (Reset) untuk memutus koneksi secara paksa, bukan **`ACK`**. Koneksi batal terjadi, tetapi Nmap sudah tahu port itu terbuka.
4. **Jika Port Tertutup:** Target langsung merespons dengan flag **`RST`**.






#### B. Service Version Detection (`-sV`) — Banner Grabbing
Nmap tidak hanya menebak port berdasarkan angka standar (misal: Port 80 pasti HTTP). Dengan `-sV`, Nmap melakukan *Banner Grabbing*.


**Mekanisme Kerja:**
Setelah port dipastikan `open`, Nmap akan mengirimkan rentetan paket interogasi (*probes*) spesifik ke port tersebut dan menganalisis balasan teks dari aplikasi. Dari balasan inilah Nmap tahu jika port 80 ternyata diisi oleh `Apache httpd 2.4.38 (Debian)`, bukan Nginx atau IIS. Informasi versi ini wajib dicatat untuk mencari kecocokan *exploit* di database CVE (*Common Vulnerabilities and Exposures*).


#### C. OS Detection (`-O`) — TCP/IP Stack Fingerprinting
Setiap Sistem Operasi (Linux, Windows, Android, iOS) memiliki karakteristik yang sedikit berbeda dalam cara mereka menyusun dan merespons paket TCP/IP mentah.


**Mekanisme Kerja:**
Nmap mengirimkan serangkaian paket data kustom (yang sengaja dibuat cacat atau tidak standar) ke port yang terbuka dan tertutup. Nmap kemudian membaca respons tersebut dan mencocokkannya dengan database *fingerprint* ribuan OS yang dimilikinya untuk menentukan apakah target menggunakan Linux Kernel sekian, atau Windows versi sekian.



