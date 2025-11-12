
## 📂 **STRUKTUR DIREKTORI**

```
openwrt-quectel-ec200t/
├── README.md
├── LICENSE
├── docs/
│   └── tutorial-id.md
└── files/
    ├── init.d/
    │   └── quectel_EC200
    └── hotplug.d/
        └── usb/
            └── 10-quectel_ec200.sh
```


## 🧾 **README.md**

````markdown
# 🚀 OpenWRT Quectel EC200T Integration

> Otomatisasi deteksi & koneksi modem **Quectel EC200T (ECM mode)** pada sistem OpenWRT / ImmortalWRT.  
> Proyek ini dibuat untuk mempermudah pengguna mengaktifkan modul EC200T tanpa konfigurasi manual panjang.

---

## 🌐 Fitur Utama

✅ Deteksi otomatis modem EC200T (VID:PID = `2c7c:6026`)  
✅ Auto-setup interface `usb0` sebagai koneksi WAN DHCP  
✅ Menggunakan perintah **AT Command** via `socat`  
✅ Kompatibel dengan kernel **5.x / 6.x** OpenWRT  
✅ Hotplug USB otomatis — langsung online saat colok  

---

## ⚙️ Prasyarat

Sebelum mulai, pastikan paket-paket berikut sudah terpasang:

```bash
opkg update
opkg install kmod-usb-net-cdc-ether kmod-usb-serial-option usbutils socat
````

* `kmod-usb-net-cdc-ether` → driver untuk mode ECM
* `kmod-usb-serial-option` → mendeteksi `/dev/ttyUSB*`
* `usbutils` → menyediakan perintah `lsusb`
* `socat` → mengirim perintah AT ke modem

---

## 🧩 Instalasi Manual

1. **Salin skrip init.d**

   ```bash
   cp files/init.d/quectel_EC200 /etc/init.d/
   chmod +x /etc/init.d/quectel_EC200
   ```

2. **Salin skrip hotplug USB**

   ```bash
   mkdir -p /etc/hotplug.d/usb/
   cp files/hotplug.d/usb/10-quectel_ec200.sh /etc/hotplug.d/usb/
   chmod +x /etc/hotplug.d/usb/10-quectel_ec200.sh
   ```

3. **Tambahkan interface jaringan ke `/etc/config/network`**

   ```bash
   config interface 'QUECTEL_EC200T'
           option proto 'dhcp'
           option ifname 'usb0'
           option metric '200'
   ```

4. **Restart jaringan dan jalankan skrip**

   ```bash
   /etc/init.d/network restart
   /etc/init.d/quectel_EC200 start
   ```

Jika berhasil, cek IP address di interface `usb0`:

```bash
ip ro
```

---

## 🔁 Mekanisme Hotplug Otomatis

Begitu modul EC200T dicolok ke USB, OpenWRT akan:

1. Menjalankan `hotplug.d` untuk mendeteksi VID:PID `2c7c:6026`
2. Menjalankan `/etc/init.d/quectel_EC200 restart`
3. Menginisialisasi koneksi data dan DHCP client di `usb0`

---

## 📄 Dokumentasi Lengkap

📘 Baca panduan versi Bahasa Indonesia di:
[`docs/tutorial-id.md`](docs/tutorial-id.md)

---

## 🧠 Pengujian & Kompatibilitas

| Platform        | Status      | Catatan                                            |
| --------------- | ----------- | -------------------------------------------------- |
| HC-G80 (MT7981) | ✅ Tested    | ImmortalWRT 23.05, kernel 5.4                      |
| MT7621 Router   | ✅ Tested    | OpenWRT 22.03                                      |
| Generic ARM64   | ⚠️ Untested | Harus pakai kernel dengan driver `cdc_ether` aktif |

---

## 🤝 Kontribusi

Kontribusi sangat diterima!
Silakan buat **Pull Request** atau buka **Issue** jika menemukan bug.

Langkah dasar:

```bash
git clone https://github.com/Sincan2/openwrt-quectel-ec200t.git
cd openwrt-quectel-ec200t
```

---

## 🪪 Lisensi

Proyek ini dirilis di bawah lisensi **MIT License**
Lihat [LICENSE](LICENSE) untuk detail.

---

## 🧑‍💻 Pembuat

**PT Terra Net Indonesia** – 2025
Dikelola oleh [Sincan2 / TerraNet Developer](https://github.com/Sincan2)

---

> *“Simple, stable, and smart LTE integration for OpenWRT — by TerraNet.”*

````

---

## 📄 **LICENSE (MIT License)**

```text
MIT License

Copyright (c) 2025 PT Terra Net

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the “Software”), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
````

---

## 📘 **docs/tutorial-id.md**

Gunakan hasil terjemahan panjang dari sebelumnya (yang sudah kamu minta).
Letakkan seluruh isi artikel lengkap di sini agar pengguna punya panduan penuh.

---

Kalau kamu mau, saya bisa bantu **buat ZIP siap upload** ke GitHub berisi:

* Struktur folder ini
* README, LICENSE, docs, dan scripts semua sudah rapi
* Nama repositori: `openwrt-quectel-ec200t`

Mau saya buatkan langsung file `.zip`-nya biar kamu tinggal upload ke repo itu?
