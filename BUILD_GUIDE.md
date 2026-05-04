 Build Guide - Kelompok 9 IoT OS

Panduan ini merinci langkah-langkah untuk membangun ulang Sistem Operasi IoT Mo>

## System Requirements
- **Host OS:** Ubuntu 22.04 LTS
- **RAM:** 8 GB
- **Disk:** 55GB ruang kosong
- **CPU:** 5 cores
- **Software Tambahan:** `build-essential`, `bison`, `flex`, `gawk`, `texinfo`,>

## Preparation Steps

### 1. Host System Setup
Pastikan semua paket yang dibutuhkan terpasang di sistem host Ubuntu.
```bash
sudo apt update
sudo apt install build-essential bison flex gawk texinfo qemu-system-aarch64

```
Setelah menyiapkan Sistem sistem tambahan yang dibutuhkan pembaca dapat langsun>

### 2. Partition Preparation
Pembaca dapat membagi penyimpanan virtul menjadi 25 GB Untuk Host Ubuntu, dan 3>
Dikarenakan LFS pada pembuatan kali ini menggunakan RAM yang terbilang kecil ma>
Pembaca dapat menggunakan Gparted untuk melakukan setting pada partisi
Pembaca dapat menyiapkan partisi build dengan
```bash
export LFS=/mnt/lfs
sudo mkdir -p $LFS
sudo mount /dev/sda3 $LFS
```

### 3. Download sources.
Pembaca dapat mengunduh dokumen langkah langkah membuat LFS yang sudah ada di i>
pembaca juga dapat mengunduh source code paket LFS.
```
mkdir -v $LFS/sources
chmod -v a+wt $LFS/sources
wget https://www.linuxfromscratch.org/lfs/view/stable/wget-list
```

## Build Process
### Phase 1: Toolchain
Membangun cross-compiler sementara untuk target aarch64.
Binutils (v2.44): Assembler dan Linker.
GCC (v14.2.0) pass 1: Kompilasi compiler awal.
Linux headers: Menyediakan API kernel.
Glibc: Library C standar.
GCC pass 2: Finalisasi compiler untuk membangun system base.

### Phase 2: System Base
Membangun komponen inti sistem operasi.
Kernel compilation: Menggunakan kernel versi 6.13.4 dengan konfigurasi minimal >
Essential packages: Instalasi Bash, Coreutils, dan BusyBox untuk menggantikan u>
System configuration: Setting /etc/fstab, /etc/passwd, dan hostname.

### Phase 3: Theme-Specific
Custom configurations: Implementasi skrip /bin/pantau_suhu di dalam folder /bin.
1. Jalankan QEMU
QEMU akan digunakan untuk menjalankan LFS yang memiliki arsitektur aarch64.

sudo qemu-system-aarch64 -M virt -cpu cortex-a57 -m 1024 \
-kernel /home/vboxuserpw123/lfs-kernel-arm64 \
-drive file=/mnt/storage/lfs_disk.img,if=virtio,format=raw \
-append "root=/dev/vda rw console=tty0 console=ttyAMA0 rootwait" \
-device virtio-gpu-pci -device virtio-keyboard-pci -device virtio-mouse-pci

2. Login sebagai operator
#masuk ke user
su - operator
#cek apakah user sudah menjadi operator atau masih root
whoami
3. Jalankan /bin/pantau_suhu dan pastikan output suhu muncul
#cek bin/sh
ls -l /usr/bin/sh
#jalankan program pantau_suhu
sh /bin/pantau_suhu
