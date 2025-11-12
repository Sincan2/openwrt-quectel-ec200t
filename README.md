# OpenWRT - Quectel EC200T Setup

Modul Quectel EC200T (ECM-only) auto setup untuk OpenWRT.  
Menyediakan skrip init.d dan hotplug agar modem langsung online saat colok.

## Fitur:
- Deteksi otomatis modem EC200T (VID:PID = 2c7c:6026)
- Setup via AT-command melalui socat
- DHCP interface di `usb0`
