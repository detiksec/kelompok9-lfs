# System Documentation - Kelompok 9 Sisto OS (IoT Edition)

Dokumen ini menjelaskan arsitektur, komponen, dan spesifikasi teknis dari Sistem Operasi minimalis berbasis Linux From Scratch (LFS) yang dirancang khusus untuk monitoring suhu server.

## 1. Arsitektur Sistem Overview
Sistem ini adalah distribusi Linux kustom yang dibangun sepenuhnya dari kode sumber (source code). OS ini menggunakan arsitektur **AArch64 (ARM 64-bit)** dan berjalan di atas hypervisor **QEMU (Virt Machine)**. Desain sistem difokuskan pada efisiensi ekstrim untuk perangkat IoT dengan menghapus seluruh komponen yang tidak diperlukan saat runtime.

## 2. Package List Lengkap
Berikut adalah daftar paket inti hasil kompilasi mandiri yang membentuk sistem ini:

| Package | Versi | Fungsi Utama |
| :--- | :--- | :--- |
| **Linux Kernel** | 6.13.4 | Inti sistem operasi (Stable Terbaru) |
| **Glibc** | 2.40 | Library C standar untuk sistem call |
| **GCC** | 14.2.0 | Compiler C/C++ (GNU Compiler Collection) |
| **Binutils (as)** | 2.44 | Assembler dan Linker |
| **BusyBox** | 1.37.0 | Multi-call binary (ls, cat, tar, sed, gzip, dll) |
| **GNU Make** | 4.4.1 | Alat bantu otomasi build |
| **GNU Awk** | 5.3.1 | Pemrosesan teks untuk skrip monitoring |
| **Iproot2** | 6.x | Konfigurasi networking dan IP |

## 3. Kernel Configuration Choices
Kernel Linux dikonfigurasi secara manual menggunakan `make menuconfig` dengan pilihan optimasi berikut:
- **Processor:** ARMv8 / Cortex-A57 (Target QEMU Virt).
- **Filesystem:** Dukungan **Ext4** diaktifkan secara built-in (`[*]`) tanpa modul.
- **VirtIO Support:** Driver `VirtIO Block`, `VirtIO Network`, dan `VirtIO Console` diaktifkan sebagai built-in untuk performa maksimal di lingkungan virtual.
- **Minimalist:** Menonaktifkan dukungan Sound Card, Graphics Card fisik, WiFi, dan driver hardware legacy untuk mengecilkan ukuran image.

## 4. File System Layout (FHS)
Struktur direktori mengikuti standar *Filesystem Hierarchy Standard* (FHS):
- `/bin` & `/sbin`: Berisi utilitas sistem esensial (sebagian besar link ke BusyBox).
- `/etc`: File konfigurasi (passwd, group, hosts, fstab).
- `/root`: Direktori home administrator; berisi folder simulasi sensor suhu.
- `/usr`: Berisi library (`/usr/lib`) dan binary aplikasi.
- `/tmp`: Folder sementara dengan izin akses publik untuk pertukaran data antar user.

## 5. Init System Setup
Sistem ini tidak menggunakan `systemd` atau `SysVinit` yang berat. Kernel dikonfigurasi untuk langsung menjalankan **`/bin/sh`** sebagai proses pertama (PID 1). Inisialisasi aplikasi IoT dilakukan melalui eksekusi skrip shell saat login atau secara manual.

## 6. Network Configuration
- **Interface:** `eth0` (VirtIO Network Device).
- **IP Assignment:** Menggunakan klien DHCP **`udhcpc`** (bagian dari BusyBox) untuk mendapatkan alamat IP dinamis dari QEMU (default: `10.0.2.15`).
- **Tujuan:** Memungkinkan perangkat mengirimkan data log suhu ke server eksternal di masa depan.

## 7. User & Permissions
Sistem mendukung multi-user untuk keamanan:
1.  **`root`**: Akses administratif penuh. Digunakan untuk pemeliharaan sistem.
2.  **`operator`**: User terbatas untuk operasional IoT. User ini memiliki izin untuk menjalankan skrip `/bin/pantau_suhu` namun dibatasi aksesnya ke file sistem sensitif.

## 8. Services yang Running
- **Aplikasi Monitoring Suhu:** Berjalan sebagai proses latar belakang (`/bin/pantau_suhu &`) yang memantau file sensor simulasi di `/root/sensor/temp` setiap 5 detik.

## 9. Security Measures
- **Minimal Attack Surface:** Seluruh alat pengembangan (Header C, Manual Docs, Static Libs) dihapus di sistem final.
- **Binary Stripping:** Semua file eksekusi telah di-strip dari simbol debug untuk mempersulit proses *reverse engineering*.
- **Permission Hardening:** Folder `/root` dikunci rapat, hanya menyisakan izin "traverse" agar operator bisa mengakses data sensor.

## 10. Performance Benchmarks
- **Boot Time:** ~1.8 detik (dari firmware hingga prompt login).
- **Disk Usage:** **88 MB** (Berhasil memenuhi target < 100 MB).
- **Memory Usage:** ~14 MB RAM saat aplikasi monitoring berjalan.

## 11. Comparison dengan Distro Mainstream
| Fitur | Kelompok 9 IoT OS | Ubuntu Server (ARM64) |
| :--- | :--- | :--- |
| **Ukuran Image** | < 100 MB | > 1.5 GB |
| **Waktu Boot** | < 2 Detik | ~20-30 Detik |
| **Efisiensi** | Sangat Tinggi (Single Purpose) | Rendah (General Purpose) |

## 12. Diagram

### Boot Process Flowchart
```mermaid
graph TD
    A[Power On] --> B[QEMU Firmware]
    B --> C[Linux Kernel 6.13.4]
    C --> D[Mount RootFS /dev/vda]
    D --> E[Init /bin/sh]
    E --> F[User Management / Login]
    F --> G[IoT Monitoring: pantau_suhu]
