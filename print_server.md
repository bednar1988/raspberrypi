# Raspberry Pi 4B Print Server – Samsung ML-1665

Turn your Raspberry Pi into a network print server so every computer and phone at home can print wirelessly — no manufacturer drivers needed on the client side.

---

## Requirements

- Raspberry Pi 4B running Raspberry Pi OS (Lite is enough)
- USB cable (Pi → printer)
- Samsung ML-1665 connected and powered on
- SSH access or keyboard/monitor connected to the Pi

---

## Step 1 – Update the System

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 2 – Install CUPS

CUPS (Common Unix Printing System) is the core of the print server.

```bash
sudo apt install cups -y
```

Add the `pi` user to the `lpadmin` group so you can manage printers without `sudo`:

```bash
sudo usermod -aG lpadmin pi
```

---

## Step 3 – Configure CUPS for Network Access

Open the CUPS configuration file:

```bash
sudo nano /etc/cups/cupsd.conf
```

Make the following changes:

```
# Listen on all interfaces, not just localhost
Listen 0.0.0.0:631

# Allow access from the local network
<Location />
  Order allow,deny
  Allow @LOCAL
</Location>

<Location /admin>
  Order allow,deny
  Allow @LOCAL
</Location>
```

Save with `Ctrl+O` → `Enter` → `Ctrl+X`, then restart and enable CUPS:

```bash
sudo systemctl restart cups
sudo systemctl enable cups
```

---

## Step 4 – Install the Samsung ML-1665 Driver

The ML-1665 uses the **SpliX** driver. Install it:

```bash
sudo apt install printer-driver-splix -y
```

If SpliX doesn't work, try the alternative:

```bash
sudo apt install printer-driver-foo2zjs -y
```

Plug in the printer via USB, then verify the Pi detects it:

```bash
lsusb
# You should see something like: Samsung Electronics ML-1660 Series
```

---

## Step 5 – Add the Printer via the CUPS Web UI

Find your Pi's IP address:

```bash
hostname -I
```

Then open a browser on any computer on your network and go to:

```
http://[RASPBERRY_PI_IP]:631
```

1. Click **Administration → Add Printer**
2. Log in with the `pi` username and password
3. Select the printer from the list (it should appear as a USB device)
4. When choosing a driver, select **Samsung ML-1660 Series** — it is fully compatible with the ML-1665
5. Check **Share This Printer** before saving

Test the printer from the CUPS interface using **Print Test Page**.

---

## Step 6 – AirPrint for iPhone, iPad & macOS

Install **Avahi** (mDNS daemon) so Apple devices discover the printer automatically:

```bash
sudo apt install avahi-daemon -y
sudo systemctl enable avahi-daemon
sudo systemctl start avahi-daemon
```

CUPS 2.x automatically advertises shared printers as AirPrint when Avahi is running. On macOS and iOS the printer will appear in the system printer list with no additional setup required.

---

## Step 7 – Connecting from Windows

Windows 10/11 should detect the printer automatically via the network. If it doesn't:

1. Go to **Settings → Bluetooth & devices → Printers & scanners**
2. Click **Add a printer or scanner**
3. If the printer isn't listed, click **The printer that I want isn't listed**
4. Choose **Add a printer using a TCP/IP address or hostname**
5. Enter your Raspberry Pi's IP address

---

## Step 8 – Connecting from Android

1. Open **Settings → Connected devices → Printing**
2. Enable **Default Print Service**
3. Android will discover CUPS printers on the local network automatically

Alternatively, install the **CUPS printing** app from the Play Store for manual setup.

---

## Quick Reference

| Task | Command / Address |
|---|---|
| CUPS web panel | `http://[PI_IP]:631` |
| Check printer status | `lpstat -p -d` |
| Send a test print | `echo "test" \| lp` |
| View CUPS error logs | `sudo journalctl -u cups -f` |
| Restart CUPS | `sudo systemctl restart cups` |

---

## Troubleshooting

**Printer not detected by `lsusb`**
Try a different USB cable or port. Run `dmesg | tail -20` right after plugging in to see kernel messages.

**CUPS web UI not accessible from other devices**
Double-check the `cupsd.conf` changes in Step 3 and that port 631 is not blocked by a firewall (`sudo ufw allow 631`).

**Print jobs stuck in queue**
Cancel all jobs and restart the printer and CUPS:
```bash
cancel -a
sudo systemctl restart cups
```

**macOS doesn't see the printer**
Make sure `avahi-daemon` is running (`sudo systemctl status avahi-daemon`) and that your Mac and Pi are on the same Wi-Fi network.

---

> **Note:** The Samsung ML-1665 is nearly identical to the ML-1660 internally. Always choose the **ML-1660** driver in CUPS if the exact model isn't listed — it works perfectly.