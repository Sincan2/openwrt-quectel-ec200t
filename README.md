## 💾 README.md (versi penuh siap upload)

````markdown
# 🚀 OpenWRT Quectel EC200T Integration (Auto Setup & AT Control)

> Otomatisasi deteksi dan koneksi modem **Quectel EC200T (ECM-only)** pada sistem OpenWRT / ImmortalWRT.  
> Didesain agar modem langsung online saat colok USB tanpa perlu perintah manual panjang.  
> Proyek ini dikembangkan oleh **PT Terra Net Indonesia (2025)** untuk digunakan di router **HC-G80 / MT7981** dan kompatibel juga dengan board MT7621, ARM64, dan x86.


## 🌐 Fitur Utama

✅ Deteksi otomatis modem Quectel EC200T (VID:PID = `2c7c:6026`)  
✅ Auto-setup interface `usb0` sebagai WAN DHCP  
✅ Koneksi lewat perintah **AT Command** via `socat`  
✅ Kompatibel dengan kernel **5.4 / 6.x / ImmortalWRT 23.xx**  
✅ Hotplug USB otomatis — langsung online saat modul dicolok  
✅ Tidak perlu QMI/MBIM karena EC200T hanya mendukung ECM  



## ⚙️ Prasyarat

Pastikan paket-paket berikut sudah terpasang di sistem OpenWRT kamu:

```bash
opkg update
opkg install kmod-usb-net-cdc-ether kmod-usb-serial-option usbutils socat
````

**Keterangan paket:**

* `kmod-usb-net-cdc-ether` → mendukung mode ECM (Ethernet over USB)
* `kmod-usb-serial-option` → mengenali device `/dev/ttyUSB*`
* `usbutils` → perintah `lsusb`
* `socat` → mengirimkan perintah AT command ke modem



## 🔌 Deteksi Awal Perangkat

Colokkan modul **Quectel EC200T** ke port USB router, tunggu ±30 detik, lalu jalankan:

```bash
lsusb
```

Hasil yang diharapkan:

```
Bus 002 Device 005: ID 2c7c:6026 Quectel Wireless Solutions Co., Ltd.
```

Bind driver `option` dengan perintah:

```bash
echo 2c7c 6026 ff > /sys/bus/usb-serial/drivers/option1/new_id
```

Lalu cek device serial:

```bash
ls /dev/ttyUSB*
```

Contoh hasil:

```
/dev/ttyUSB0
/dev/ttyUSB1
/dev/ttyUSB2
```

Biasanya **AT port** berada di `/dev/ttyUSB1`.

---

## 🧠 Tes Komunikasi Modem

Gunakan `socat` untuk berinteraksi langsung dengan modem:

```bash
AT_PORT=/dev/ttyUSB1
SOCAT_RUN="socat - $AT_PORT,crnl"
echo ATE0 | $SOCAT_RUN
echo ATI | $SOCAT_RUN
```

Output:

```
Quectel
EC200T
Revision: EC200TEUHAR05A01M16
```

---

## 🌍 Tambahkan Interface WAN di OpenWRT

Edit `/etc/config/network` dan tambahkan:

```bash
config interface 'QUECTEL_EC200T'
        option proto 'dhcp'
        option ifname 'usb0'
        option metric '200'
```

Kemudian restart jaringan:

```bash
/etc/init.d/network restart
```

---

## ⚡ Skrip Init.d (Full Script)

Buat file baru:

```
/etc/init.d/quectel_EC200
```

Isi lengkapnya:

```bash
#!/bin/sh /etc/rc.common

START=98

start(){

set -x
echo Start

AT_PORT=/dev/ttyUSB1
DEV=usb0
SOCAT_RUN="socat - $AT_PORT,crnl"

if test -c $AT_PORT
then echo port ready
else
	echo 2c7c 6026 ff > /sys/bus/usb-serial/drivers/option1/new_id
	echo sleep till port is ready
	sleep 30
	for i in $(seq 10);
	do
	    test -c $AT_PORT && break
	    echo .
	    sleep 10
	done
fi

test -c $AT_PORT || { echo "$AT_PORT cant be found, exit"; return 22; }

echo ATE0 | $SOCAT_RUN
echo ATI | $SOCAT_RUN
echo 'AT+QCFG="usbnet"' | $SOCAT_RUN
echo at+cgdcont? | $SOCAT_RUN
echo at+cgatt? | $SOCAT_RUN
echo at+cgact? | $SOCAT_RUN
echo at+cgact=1,1 | $SOCAT_RUN
echo AT+CGCONTRDP=1 | $SOCAT_RUN
echo AT+QNETDEVCTL=1,1,1 | $SOCAT_RUN ; sleep 1
echo AT+QNETDEVCTL=1,1,1 | $SOCAT_RUN ; sleep 1
echo AT+QNETDEVCTL=1,1,1 | $SOCAT_RUN ; sleep 1
echo AT+QNETDEVCTL? | $SOCAT_RUN

}

stop(){
    echo Stop
}
```

Jadikan executable:

```bash
chmod +x /etc/init.d/quectel_EC200
```

Jalankan:

```bash
/etc/init.d/quectel_EC200 start
```

---

## 🔁 Skrip Hotplug USB (Full Script)

Buat file:

```
/etc/hotplug.d/usb/10-quectel_ec200.sh
```

Isi lengkapnya:

```bash
#!/bin/sh

[ "$ACTION" = add -a "$DEVTYPE" = usb_device ] || exit 0

vid=$(cat /sys$DEVPATH/idVendor)
pid=$(cat /sys$DEVPATH/idProduct)

if [[ $vid == 2c7c ]] && [[ $pid == 6026 ]]
then	true
else	exit
fi

sleep 30 
/etc/init.d/quectel_EC200 restart 2>&1 | logger -t quectel_EC200
```

Jadikan executable:

```bash
chmod +x /etc/hotplug.d/usb/10-quectel_ec200.sh
```

---

## 🔄 Proses Kerja Otomatis

1. Saat EC200T dicolok → sistem mengenali VID:PID `2c7c:6026`
2. Hotplug menjalankan `/etc/init.d/quectel_EC200 restart`
3. Script mengirim perintah AT ke modem dan mengaktifkan koneksi data
4. OpenWRT menjalankan DHCP client di `usb0`
5. Modem terhubung ke internet

---

## 🌐 Cek Routing

Setelah modem terkoneksi:

```bash
ip ro
```

Contoh output:

```
default via 10.56.99.192 dev usb0  src 10.56.99.63  metric 200
```

---

## 🧩 Troubleshooting

**Q:** Tidak muncul `/dev/ttyUSB1`
**A:** Jalankan ulang perintah bind driver:

```bash
echo 2c7c 6026 ff > /sys/bus/usb-serial/drivers/option1/new_id
```

**Q:** DHCP tidak dapat IP
**A:** Jalankan ulang:

```bash
/etc/init.d/quectel_EC200 start
/etc/init.d/network restart
```

**Q:** Mau pakai interface custom (misal `wwan0`)?
**A:** Ganti `option ifname 'usb0'` pada `/etc/config/network`.

---

## 🧠 Kompatibilitas Diuji

| Platform / Board           | Status         | Catatan                      |
| -------------------------- | -------------- | ---------------------------- |
| HC-G80 (MT7981)            | ✅ Berhasil     | ImmortalWRT 23.05 kernel 5.4 |
| MikroTik RB750Gr3 (MT7621) | ✅ Berhasil     | OpenWRT 22.03                |
| Raspberry Pi 4 (ARM64)     | ⚠️ Belum diuji | Harus aktifkan `cdc_ether`   |
| x86 Generic                | ⚠️ Belum diuji | Perlu USB port langsung      |

---

## 🧩 Kontribusi

Ingin bantu? Kamu bisa:

* Menambah dukungan modem Quectel lain (EC25, EC200U)
* Tambah deteksi otomatis port AT (`ttyUSB1`/`ttyUSB2`)
* Tambah skrip LuCI GUI kecil untuk tombol *Connect / Disconnect*

Langkah dasar:

```bash
git clone https://github.com/Sincan2/openwrt-quectel-ec200t.git
cd openwrt-quectel-ec200t
```

---

## 📘 Referensi Resmi

* 🔗 [OpenWRT Hotplug Documentation](https://openwrt.org/docs/guide-user/base-system/hotplug)
* 🔗 [LTE Dongle Setup (ECM/NCM)](https://openwrt.org/docs/guide-user/network/wan/wwan/ltedongle)
* 🔗 [Ethernet over USB NCM Guide](https://openwrt.org/docs/guide-user/network/wan/wwan/ethernetoverusb_ncm)

---

## 🪪 Lisensi

```text
MIT License

Copyright (c) 2025 PT
Terra Net

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
```

---
