
# struktur folder linux
| Folder [3, 4, 5, 6, 7] | Fungsi Utama |
|---|---|
| / (Root) | Tingkat tertinggi. Semua folder dan file berada di bawah direktori ini. |
| /bin | Berisi perintah dasar sistem (binary) yang bisa digunakan semua user (contoh: ls, cd, cp). |
| /sbin | Berisi perintah khusus untuk administrator/root (system binary), seperti fdisk, reboot, iptables. |
| /etc | Tempat menyimpan file konfigurasi sistem dan aplikasi (analoginya seperti Registry di Windows). |
| /home | Folder pribadi untuk pengguna biasa. Setiap user punya sub-folder di sini (contoh: /home/budi). |
| /root | Folder rumah (home directory) khusus untuk pengguna admin tertinggi (Root User). |
| /var | Menyimpan data yang dinamis dan terus berubah, seperti file log sistem, database, dan email. |
| /tmp | Tempat penyimpanan file sementara (temporary). Biasanya otomatis terhapus saat komputer restart. |
| /usr | Berisi program, dokumentasi, dan library yang diinstal oleh pengguna (user). |
| /opt | Tempat instalasi aplikasi tambahan dari pihak ketiga yang mandiri (contoh: Google Chrome, Slack). |
| /boot | Berisi file penting untuk proses menyalakan komputer, termasuk Kernel Linux dan bootloader. |
| /dev | Berisi file representasi dari perangkat keras (periferal) seperti harddisk (sda), USB, atau terminal. |
| /proc & /sys | Folder virtual yang menyediakan informasi tentang sistem, memori, dan proses yang sedang berjalan. |
| /mnt & /media | Tempat untuk menempelkan (mounting) media penyimpanan luar seperti Flashdisk atau harddisk eksternal. |
| vlog | mencatat log journal atau aktifitas yang ada di linux |
| vaud | mencatat log yang kritikal seperti kernel dan semua aplikasi yang kritikal |
| vtmp | untuk menyimpan data sementara aplikasi yang tidak akan dihapus 
gunakan tmpsf untuk mounting tmp. tmpfs digunakan untuk menyimpan file tmp langsung kedalam RAM jadi tidak ada proses write ke hardisk dan performa menjadi lebih cepat |

# perbedaan vfat dan fat
Secara fungsional dalam perintah mkfs di Linux, tidak ada perbedaan hasil format antara mkfs.vfat dan mkfs.fat karena keduanya adalah program yang sama. Di dalam sistem Linux, perintah mkfs.vfat hanyalah sebuah symbolic link (jalan pintas) yang mengarah langsung ke berkas biner utama, yaitu mkfs.fat. [1, 2, 3] 
Meskipun perkakas eksekusinya sama, kedua istilah ini merujuk pada konsep historis dan dukungan teknis yang berbeda di sisi kernel Linux: [2, 4] 
## Perbedaan Konseptual (Tabel Perbandingan)

| Aspek [1, 2, 3, 5, 6, 7, 8] | mkfs.fat (Sistem FAT Asli) | mkfs.vfat (Virtual FAT) |
|---|---|---|
| Status File | Berkas biner utama / program asli di Linux. | Symbolic link (pintasan) ke mkfs.fat. |
| Nama File Panjang | Dulu dibatasi format standar 8.3 (maksimal 8 karakter nama + 3 karakter ekstensi). | Mendukung Nama File Panjang (Long Filename / LFN) hingga 255 karakter. |
| Sensitivitas Huruf | Menyimpan nama file hanya dalam huruf besar (UPPERCASE). | Mendukung penyimpanan format huruf besar dan kecil (case-preserving). |
| Varian FAT | Otomatis menentukan jenis FAT12, FAT16, atau FAT32 berdasarkan ukuran partisi. | Secara otomatis mengaktifkan ekstensi penamaan modern di atas struktur FAT tersebut. |

## Mengapa Keduanya Dipisah di Linux?
Pemisahan nama ini terjadi karena alasan historis driver kernel Linux: [2] 

   1. msdos / fat: Dulu digunakan untuk mendiskripsikan sistem berkas DOS jadul yang kaku dan tidak bisa membuat nama file panjang.
   2. vfat: Diperkenalkan sejak era Windows 95 sebagai lapisan ekstensi agar sistem FAT lama bisa membaca dan menyimpan nama file yang panjang serta menggunakan huruf kecil. Anda bisa membaca detail historisnya di forum diskusi atau arsip 

# option fstab atau mounting
| Opsi | Pengertian | Fungsi Utama & Keuntungan |
|---|---|---|
| nodev | Melarang interpretasi berkas perangkat khusus (character/block devices) di partisi tersebut. | Keamanan: Mencegah pembuatan replika perangkat keras palsu yang bisa disalahgunakan. |
| nosuid | Menonaktifkan fungsi bit Set-User-ID (SUID) dan Set-Group-ID (SGID). | Keamanan: Mencegah pengguna biasa menjalankan program dengan hak akses root. |
| rw | Memberikan hak akses Baca dan Tulis (Read-Write) pada sistem berkas. | Fungsionalitas: Mengizinkan pembuatan, modifikasi, dan penghapusan file/folder. |
| noexec | Melarang eksekusi file biner, program, atau skrip langsung dari partisi tersebut. | Keamanan: Memblokir eksekusi virus, malware, atau skrip berbahaya yang terunduh. |
| relatime | Memperbarui waktu akses file (atime) hanya jika file berubah atau setelah 24 jam. | Performa: Mempercepat proses baca disk dan menghemat umur pakai SSD. |
