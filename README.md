# การข้าม SSL Pinning สำหรับ iOS (SSL Pinning Bypass) — iptables

ชุดสคริปต์ Bash สำหรับการตั้งค่าเซิร์ฟเวอร์ OpenVPN พร้อมกฎ iptables เพื่อดักจับและเปลี่ยนเส้นทางการจราจร (Traffic) ของ iOS เพื่อใช้ในการข้าม SSL Pinning ระหว่างการวิจัยด้านความปลอดภัยและการทดสอบการเจาะระบบ (Penetration Testing)

## 📜 สคริปต์ที่มีให้

### 1. `openvpn-install.sh`
ตัวติดตั้งเซิร์ฟเวอร์ OpenVPN แบบอัตโนมัติ อ้างอิงจาก [Nyr/openvpn-install](https://github.com/Nyr/openvpn-install)

**รองรับ:** Ubuntu 22.04+, Debian 11+, AlmaLinux/Rocky/CentOS 9+, Fedora

**คุณสมบัติ:**
- ตั้งค่าเซิร์ฟเวอร์ OpenVPN แบบเต็มรูปแบบพร้อม PKI (ผ่าน easy-rsa)
- รองรับทั้งโปรโตคอล UDP และ TCP
- กำหนดค่า DNS ได้ (Google, Cloudflare, OpenDNS, Quad9, AdGuard, หรือกำหนดเอง)
- จัดการกฎ iptables/firewalld โดยอัตโนมัติ
- เพิ่ม/ยกเลิกการเข้าถึงของเครื่องลูกข่าย (Client) ได้โดยไม่ต้องติดตั้งใหม่

**วิธีใช้งาน:**
```bash
chmod +x openvpn-install.sh
sudo bash openvpn-install.sh
```

---

### 2. `iptables-setup.sh`
ตั้งค่ากฎ NAT ของ iptables เพื่อเปลี่ยนเส้นทางการจราจร HTTP/HTTPS จากอุโมงค์ VPN (`tun0`) ไปยัง Proxy ท้องถิ่น (พอร์ต `8080`) — มีประโยชน์มากสำหรับเครื่องมืออย่าง Burp Suite หรือ mitmproxy

**การทำงาน:**
- เปลี่ยนเส้นทางพอร์ต `80` → `8080` บน `tun0`
- เปลี่ยนเส้นทางพอร์ต `443` → `8080` บน `tun0`
- เพิ่มกฎ MASQUERADE สำหรับ Subnet ที่กำหนดบน `eth0`

**วิธีใช้งาน:**
```bash
chmod +x iptables-setup.sh
sudo bash iptables-setup.sh <VPN_SERVER_IP>

# ตัวอย่าง:
sudo bash iptables-setup.sh 10.8.0.1
```

---

## 🛠️ ขั้นตอนการตั้งค่าทั่วไป (Typical Setup Flow)

1. รัน `openvpn-install.sh` เพื่อตั้งค่าเซิร์ฟเวอร์ VPN ของคุณ
2. ตั้งค่าเครื่องมือ Proxy (เช่น Burp Suite) ให้รอรับการเชื่อมต่อที่พอร์ต `8080` ด้วย IP ของ VPN Interface (เช่น `10.x.x.1`)
   - ใน Burp Suite: **Proxy → Options → Interface IP → Request Handling → Enable Support for Invisible Proxying**
3. เชื่อมต่ออุปกรณ์ iOS ของคุณเข้ากับ VPN
4. รัน `iptables-setup.sh` พร้อมระบุ IP ของอุปกรณ์ iOS (Client IP) เพื่อเปลี่ยนเส้นทาง Traffic ไปยัง Proxy
   ```bash
   sudo bash iptables-setup.sh <iOS_VPN_CLIENT_IP>
   # ตัวอย่าง: sudo bash iptables-setup.sh 10.8.0.2
   ```
5. ติดตั้ง CA Certificate ของ Proxy บนอุปกรณ์ iOS
   - ส่งออกไฟล์ CA จาก Burp Suite → โอนเข้าเครื่อง iOS → **Settings → General → VPN & Device Management → Install**

---

## 📋 ความต้องการของระบบ (Requirements)

- เซิร์ฟเวอร์ Linux (สิทธิ์ root)
- `iptables`
- OpenVPN (ติดตั้งผ่าน `openvpn-install.sh`)
- เครื่องมือ Proxy (Burp Suite, mitmproxy ฯลฯ) ที่รอรับการเชื่อมต่อที่พอร์ต `8080`

---
