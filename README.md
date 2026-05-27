# Dokumentasi Virtual Home Lab Active Directory Domain Services (AD DS) Menggunakan VirtualBox

## 1. Pendahuluan
Dokumentasi ini menjelaskan langkah-langkah membangun lingkungan virtual home lab Active Directory Domain Services (AD DS) menggunakan Oracle VirtualBox.

Dokumentasi mencakup:
- Instalasi dan konfigurasi Active Directory Domain Services
- Pembuatan Organizational Unit (OU)
- Pembuatan User dan Group
- Konfigurasi Group Policy Object (GPO)
- Konfigurasi File Sharing
- Join Client ke Domain

---

## 2. Topologi Lab

### Server dan Client
<div align="center">
<pre>
┌────────────────────┐        ┌────────────────────┐
│        DC01        │        │      CLIENT01      │
│ Windows Server 2019│        │   Windows 10/11    │
│   AD DS + DNS      │◄──────►│    Domain Client   │
│   192.168.10.10    │        │    192.168.10.20   │
└────────────────────┘        └────────────────────┘
</pre>
</div>

### Domain
| Item | Value |
|---|---|
| Domain Name | homelab.local |
| NetBIOS Name | HOMELAB |

---

## 3. Persiapan Virtual Machine
### Install Oracle VirtualBox
Unduh dan install Oracle VirtualBox dari: https://www.virtualbox.org

### Install Windows Server 2019
Unduh dan install Windows 2019 dari: https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019

### Install Windows 10/11
Unduh dan install Windows 10/11 dari: https://www.microsoft.com/software-download/windows10 atau https://www.microsoft.com/software-download/windows11

### Langkah Membuat VM di VirtualBox
1. Buka **Oracle VirtualBox**, klik **New** untuk membuat VM baru.
2. Masukkan nama VM sesuai peran:
   - `DC01` untuk Domain Controller
   - `CLIENT01` untuk Client
3. Pilih tipe OS:
   - **Microsoft Windows**, versi sesuai ISO (Server 2019 atau Windows 10/11).
4. Atur **Memory Size**:
   - DC01: 1536 MB
   - CLIENT01: 1536 MB
5. Buat virtual hard disk baru:
   - Format: VDI
   - Storage: Dynamically allocated
   - Size: 20 GB
6. Setelah VM dibuat, buka **Settings**:
   - **System → Processor**: set sesuai spesifikasi (DC01: 2 core, CLIENT01: 1 core).
   - **Network → Adapter 1**: pilih **Internal Network / Host-Only Adapter**.
7. Masukkan ISO OS ke **Storage → Optical Drive**.
8. Klik **Start** untuk memulai instalasi OS.
9. Ikuti wizard instalasi Windows Server 2019 atau Windows 10/11 hingga selesai.

<p align="center">
  <img src="image/windows-server-setup.png" width="75%">
</p>
<p align="center">
  <img src="image/windows-client-setup.png" width="75%">
</p>

---

## 4. Instalasi Active Directory Domain Services

### Konfigurasi IP Address Server
1. Buka **Control Panel** → **Network & Internet** → **Change adapter settings**.
2. Klik kanan **Ethernet** → **Properties** → **IPv4** → **Properties**.
3. Pilih **Use the following IP address**, lalu isi:

   | Setting | Value |
   |---|---|
   | IP Address | 192.168.10.10 |
   | Subnet Mask | 255.255.255.0 |
   | Gateway | 192.168.10.1 |
   | Preferred DNS | 192.168.10.10 |

   <p align="center">
     <img src="image/ip-add-setup.png" width="75%">
   </p>

### Install Role AD DS
1. Buka **Server Manager** → **Add Roles and Features**.  
2. Pilih:  
   - Active Directory Domain Services  
   - DNS Server  
3. Ikuti intruksi, lalu klik **Install**.

<p align="center">
  <img src="image/install-ad-ds.png" width="75%">
</p>

### Promote Server Menjadi Domain Controller
1. Klik notifikasi bendera di Server Manager.  
2. Pilih **Promote this server to a domain controller**.
   
   <p align="center">
     <img src="image/promote-server.png" width="75%">
   </p>
   
3. Pilih **Add a new forest**.  
4. Isi root domain: `homelab.local`.

   <p align="center">
     <img src="image/domain-name.png" width="75%">
   </p>
   
5. Tentukan password DSRM.  
6. Klik **Install** → server akan restart otomatis.
   
   <p align="center">
     <img src="image/install-domain.png" width="75%">
   </p>

---

## 5. Membuat Organizational Unit (OU)

### Membuka Active Directory Users and Computers
Jalankan:
```text
dsa.msc
```

<p align="center">
  <img src="image/ou.png" width="75%">
</p>
   
### Struktur OU
```text
Company
├── Computers
│   ├── HR
│   ├── IT
│   └── Finance
├── Users
│   ├── HR
│   ├── IT
│   └── Finance
└── Servers
```

### Langkah Membuat OU
1. Klik kanan domain ``homelab.local``.
2. Pilih **New → Organizational Unit**.
3. Buat OU utama: ``Company``.
4. Di dalam OU Company, buat OU: ``Computers``, ``Users``, dan ``Servers``.
5. Tambahkan sub-OU sesuai divisi (HR, IT, Finance) di dalam OU ``Computers`` dan ``Users``.

<p align="center">
  <img src="image/ou-added.png" width="75%">
</p>

---

## 6. Membuat User dan Group
### Membuat User
1. Masuk ke OU Users → OU divisi.
2. Klik kanan → **New** → **User**.
3. Isi data user dan password.
   <p align="center">
     <img src="image/make-user.png" width="75%">
   </p>   
4. Hilangkan centang **User must change password at next logon**.
   <p align="center">
     <img src="image/user-password.png" width="75%">
   </p>
5. Klik **Finish**. Maka user baru telah ditambahkan.
   <p align="center">
     <img src="image/user-added.png" width="75%">
   </p>

### Membuat Group
1. Masuk ke OU Users → OU divisi.
2. Klik kanan → **New** → **Group**.
   <p align="center">
     <img src="image/make-group.png" width="75%">
   </p>
3. Contoh group:
   - HR-Staff
   - IT-Staff
   - Finance-Staff

### Menambahkan User ke Group
1. Klik kanan user → **Add to a group**.
2. Masukkan nama group (misalnya ``IT-Staff``).
3. Klik **OK**.
   <p align="center">
     <img src="image/add-to-group.png" width="75%">
   </p>
   <p align="center">
     <img src="image/add-to-group2.png" width="75%">
   </p>
---

## 7. Join Client ke Domain

### Konfigurasi IP Client
| Setting | Value |
|---|---|
| IP Address | 192.168.10.20 |
| Subnet Mask | 255.255.255.0 |
| Gateway | 192.168.10.1 |
| DNS Server | 192.168.10.10 |

<p align="center">
  <img src="image/ip-add-setup-client.png" width="75%">
</p>

### Join Domain
1. Buka **Settings** → **System** → **About**.
2. Klik **Rename this PC (Advanced)** → **Change**.
4. Ubah Computer Name: ``CLIENT01``.
5. Pilih Domain, isi: ``homelab.local``.
6. Masukkan credential: ``HOMELAB\Administrator``.
   <p align="center">
     <img src="image/join-domain.png" width="75%">
   </p>
7. Restart komputer

### Login Menggunakan User Domain
1. Pada login screen, pilih **Other User**.
2. Masukkan username dan password yang sudah terdaftar di Active Directory
3. Contoh:
   - Username: budi
   - Password: ********
   <p align="center">
     <img src="image/logon-client.png" width="75%">
   </p>
---

## 8. Membuat Group Policy Object (GPO)

### Membuka Group Policy Management
Tekan **Windwows** + **r**, lalu jalankan:
```text
gpmc.msc
```
<p align="center">
  <img src="image/gpo.png" width="75%">
</p>

### Membuat GPO Baru
Contoh: **Disable Control Panel**
1. Klik kanan OU Users → **Create a GPO in this domain, and Link it here**.
2. Nama GPO: ``Disable Control Panel``.

<p align="center">
  <img src="image/make-gpo1.png" width="75%">
</p>

<p align="center">
  <img src="image/make-gpo2.png" width="75%">
</p>

### Konfigurasi Policy
Klik kanan GPO yang sudah dibuat, lalu pilih **Edit**. Masuk ke:
```text
User Configuration → Policies → Administrative Templates → Control Panel
```

Aktifkan policy:
- **Prohibit access to Control Panel and PC settings** → Enabled

<p align="center">
  <img src="image/make-gpo3.png" width="75%">
</p>

<p align="center">
  <img src="image/make-gpo4.png" width="75%">
</p>

### Update GPO pada Client
Jalankan CMD sebagai Administrator:
```text
gpupdate /force
```

<p align="center">
  <img src="image/gpupdate.png" width="75%">
</p>

---

## 9. Konfigurasi File Sharing
### Membuat Folder Sharing
Contoh folder:
```text
C:\SharedData
```

<p align="center">
  <img src="image/sharedfolder.png" width="75%">
</p>

### Share Folder
1. Klik kanan folder → **Properties** → **Sharing** → **Advanced Sharing**.
2. Centang **Share this folder** → **Permissions**.

<p align="center">
  <img src="image/sharedfolder2.png" width="75%">
</p>

### Konfigurasi Permission
1. Klik **Add** lalu masukan nama group yang sudah dibuat di Active Directory (misal: HR-Staff)
   <p align="center">
     <img src="image/permission.png" width="75%">
   </p>
2. Pilih jenis Permission yang ingin diterapkan pada group tersebut.
   <p align="center">
     <img src="image/permission2.png" width="75%">
   </p>
   
### Akses Shared Folder dari Client
Pada File Explorer masuk ke:
```text
\\DC01\\SharedData
```

<p align="center">
     <img src="image/akses-sharedfolder.png" width="75%">
   </p>

Atau menggunakan IP:
```text
\\192.168.10.10\\SharedData
```
---

## 10. Verifikasi dan Testing
### Testing DNS
Pada CMD di client, jalankan:
```text
ping dc01
nslookup homelab.local
```

<p align="center">
  <img src="image/test-dns.png" width="75%">
</p>

### Testing GPO
Coba buka Control Panel. Jika akses diblok → GPO berhasil.
<p align="center">
  <img src="image/test-gpo.png" width="75%">
</p>

### Testing File Sharing
Akses ``\\DC01\SharedData`` → cek permission sesuai group.
<p align="center">
  <img src="image/test-filesharing.png" width="75%">
</p>

---

## 11. Troubleshooting

### Client Gagal Join Domain
Periksa:
- DNS client harus mengarah ke Domain Controller
- Pastikan waktu server dan client sinkron
- Pastikan network adapter berada pada network yang sama

### GPO Tidak Berjalan
Periksa dengan:
```text
gpupdate /force
gpresult /r
```

### Shared Folder Tidak Bisa Diakses
Periksa:
- Sharing Permission
- NTFS Permission
- Firewall Windows

---

## 12. Kesimpulan
Dengan mengikuti dokumentasi ini, lingkungan virtual home lab Active Directory dapat digunakan untuk:
- Simulasi administrasi jaringan Windows
- Manajemen user dan group
- Implementasi Group Policy
- Simulasi file server
- Pembelajaran domain environment menggunakan VirtualBox

---

## 13. Teknologi yang Digunakan
- Windows Server 2019
- Windows 10/11
- Oracle VirtualBox
- Active Directory Domain Services (AD DS)
- DNS Server
- Group Policy Management
