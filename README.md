# Analisis Wireshark: Dekripsi TLS & Deteksi Trojan Dryex

**Tanggal:** 11/November/2025
**Analis:** Xzavis
**Sumber Analisis:** Latihan ini didasarkan pada file PCAP dan video tutorial "Decrypting HTTPS Traffic With Wireshark".

---

## Ringkasan Eksekutif (Summary)

Analisis ini dilakukan pada file capture paket (`.pcap`) yang berisi lalu lintas jaringan terenkripsi (HTTPS/TLS). Dengan menggunakan kunci SSL yang disediakan (memungkinkan simulasi serangan Man-in-the-Middle), lalu lintas berhasil didekripsi menggunakan Wireshark.

Investigasi ini mengungkap alur serangan lengkap dari malware **Dryex**, yang dikenal menargetkan institusi keuangan. Alur infeksi dimulai dari dokumen Office ber-makro, yang kemudian mengunduh *payload* DLL berbahaya (`invest20.dll`), dan akhirnya membangun komunikasi persisten ke server Command and Control (C2).

## Tools yang Digunakan

* **Wireshark:** Untuk analisis lalu lintas jaringan, dekripsi TLS, dan ekstraksi objek/file.
* **VirusTotal:** Digunakan (secara konseptual) untuk menganalisis file malware yang diekspor.

## Analisis & Temuan (Findings)

Berdasarkan analisis file `.pcap` yang telah didekripsi, berikut adalah temuan-temuan kunci:

### 1. Dekripsi Lalu Lintas TLS
* Lalu lintas yang awalnya terenkripsi berhasil didekripsi dengan memasukkan file kunci SSL ke Wireshark (`Edit > Preferences > Protocols > TLS`).
* Ini mengubah paket "Application Data" yang tidak dapat dibaca menjadi protokol yang jelas, seperti **HTTP**.

### 2. Unduhan Payload Malware
* Setelah dekripsi, filter `http.request` segera menunjukkan aktivitas berbahaya.
* Ditemukan sebuah `GET` request yang sangat mencurigakan dari *host* korban untuk mengunduh file bernama **`invest20.dll`**.
* File ini diekstraksi dari Wireshark (`File > Export Objects > HTTP`) untuk analisis lebih lanjut.
* Analisis VirusTotal (seperti yang dideskripsikan di video) mengkonfirmasi file ini sebagai **Win32 DLL (Portable Executable)**, yang merupakan *payload* inti dari Trojan Dryex.

### 3. Komunikasi Command and Control (C2)
* Setelah infeksi, malware segera mencoba menghubungi server C2.
* Teramati adanya beberapa `POST` request yang diarahkan ke file **`docs.php`** di server penyerang.
* Komunikasi ini (dilihat menggunakan "Follow TLS Stream") adalah cara malware menerima perintah dan mungkin mengirimkan data curian.
## Indicators of Compromise (IoCs)

Berikut adalah artefak forensik yang dikumpulkan dari analisis ini, yang dapat digunakan untuk deteksi dan pemblokiran (Blue Team).

### 1. Host yang Terinfeksi
* **IP Address:** `10.4.11.101`
* **Hostname:** `desktop-u54aj8k`

### 2. File Malware (Payload)
* **Nama File:** `invest20.dll` 
* **SHA256 Hash:** `[31cf42b2a7c5c558f44cfc67684cc344c17d4946d3a1e0b2cecb8eb58173cb2f]`
* * **SHA1 Hash:** `[aac940f9906034938cd657ed2ba21bc675e6ae20 ]`

### 3. Jaringan (C2)
* * **Filter Wireshark:** `165	2020-04-01 21:02:49.397114	10.4.1.101	50074	94.103.84.245	443	HTTP	251	GET /invest_20.dll HTTP/1.1 `


![Bukti Unduhan Payload](image_bf049a.png)

## Kesimpulan & Rekomendasi

Analisis ini berhasil mengidentifikasi alur serangan Trojan Dryex secara *end-to-end*. Dengan mendekripsi lalu lintas TLS, kita dapat melihat "di balik layar" bagaimana malware dikirimkan (`invest20.dll`) dan bagaimana ia berkomunikasi (`docs.php`).

**Rekomendasi (Simulasi Blue Team):**
1.  **Isolasi** *host* `desktop-u54aj8k` (IP `10.4.11.101`) dari jaringan.
2.  **Blokir** IP/Domain server C2 di *Firewall* atau *Proxy*.
3.  **Buat Aturan Deteksi** (misal: di SIEM/IDS) untuk memantau permintaan file `invest20.dll` atau komunikasi ke `docs.php`.
4.  **Lakukan *Forensik* ** pada *host* korban untuk menemukan dokumen Office ber-makro yang menjadi titik awal infeksi.

---

> ## ⚠️ Peringatan Keamanan
>
> Repositori ini berisi file `.pcap` dan artefak (nama/hash) dari analisis malware untuk tujuan edukasi.
>
> * File `Wireshark-tutorial-on-decrypting-HTTPS-SSL-TLS-traffic.pcap` berisi lalu lintas jaringan berbahaya.
> * File `invest_20.dll` adalah **payload malware aktif**.
>
> **JANGAN MENGUNDUH ATAU MENGEKSEKUSI FILE `invest_20.dll` PADA SISTEM APAPUN.**
