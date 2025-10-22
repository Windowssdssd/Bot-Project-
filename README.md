# 🎯 Diamond-Shopz Admin System (v2.0)
ระบบจัดการพอยต์และประวัติการสุ่ม — พัฒนาใหม่ทั้งหมดจากเวอร์ชันเดิม  
ออกแบบให้ **ใช้งานง่าย**, **โค้ดสะอาด**, และ **รองรับการขยายในอนาคต**

---

## ✨ ภาพรวมการอัปเดต
เวอร์ชันนี้เป็นการ **เขียนใหม่ทั้งหมด (Full Rewrite)** ของระบบแอดมินใน Diamond-Shopz  
เน้นความปลอดภัย, การใช้งานที่เข้าใจง่าย และการแสดงผลแบบมืออาชีพ

| หมวด | รายละเอียด | สถานะใหม่ |
|-------|-------------|------------|
| 🧱 โครงสร้างโค้ด | แยกเป็น Cog / Modal / View / Utils | ✅ ใช้งานง่ายและต่อยอดได้ |
| 💾 ฐานข้อมูล | ใช้ MySQL / phpMyAdmin แทน MongoDB | ✅ ปลอดภัยและเร็วขึ้น |
| 💳 ระบบพอยต์ | เพิ่ม / ลบ / โอน / ตรวจสอบยอด | ✅ มี log และ embed สรุปผล |
| 🎰 ประวัติการสุ่ม | รวม Classic + Premium | ✅ แสดงแบบ embed พร้อมปุ่มเลื่อนหน้า |
| 🖼️ การจัดการรูป | แยกไฟล์ `config/images.json` | ✅ เปลี่ยนภาพทั้งระบบได้ในที่เดียว |
| 🧾 Log แอดมิน | บันทึกทุกคำสั่งสำคัญ | ✅ แยกตามประเภทใน config |
| 🪄 UI ใหม่ | ปรับ Embed + ปุ่ม + Modal | ✅ ใช้งานสะดวกและดูเป็นระบบ |

---

## ⚙️ โครงสร้างไฟล์
```
📦 diamond-bot/
 ┣ 📂 cogs/
 ┃ ┗ 📂 admin/
 ┃    ┗ 📜 admin.py            ← ระบบจัดการพอยต์ & ประวัติการสุ่ม
 ┣ 📂 utils/
 ┃ ┗ 📜 assets.py              ← ตัวโหลดรูปจาก config/images.json
 ┣ 📂 config/
 ┃ ┣ 📜 images.json            ← ไฟล์เก็บ URL รูป icon/banner
 ┃ ┗ 📜 config.json            ← token, guild id, log channel, ฯลฯ
 ┗ 📜 main.py
```

---

## 💡 ฟีเจอร์หลัก

### 🪙 1. ระบบจัดการพอยต์
> จัดการยอดพอยต์สมาชิกได้ง่ายผ่าน Slash Command หรือ Modal UI  

#### ✅ เพิ่มพอยต์
```
/admin add-point <สมาชิก> <จำนวน>
```
- บันทึกลงฐานข้อมูล
- แสดง Embed สรุปยอดก่อน–หลัง
- ส่ง Log ไปช่องแอดมิน

#### ❌ ลบพอยต์
```
/admin remove-point <สมาชิก> <จำนวน>
```
- ป้องกันยอดติดลบด้วย `GREATEST(balance-amount, 0)`
- แสดง Embed และ Log เหมือน add-point

#### 🔍 ตรวจสอบพอยต์
```
/admin check-point <สมาชิก>
```
- แสดง Embed สรุปยอด:
  ```
  เงินคงเหลือ : 500 บาท
  ยอดเติมรวม : 2000 บาท
  ยอดสุ่มรวม : 1500 บาท
  ```

#### 🔄 โอนพอยต์ (ผ่านปุ่ม)
> มี Modal กรอก ID และจำนวน  
ตรวจสอบยอดผู้โอนก่อนทำรายการเสมอ

---

### 🎰 2. ระบบ “ประวัติการสุ่ม” (Gacha History)
```
/admin gacha-history <สมาชิก>
```
- ดึงข้อมูลจากทั้ง `db_classic` และ `db_premium`
- แสดงแบบ embed สวยงาม:
  ```
  ประวัติการสุ่มของ diamond_shopz
  สุ่มทั้งหมด 25 ครั้ง ใช้แต้มรวม 2500 บาท

  🎰 ประเภท: Price  
  💵 ใช้แต้ม : 100  
  🎁 ผลลัพธ์ : ยศ VIP 7 วัน  
  📅 วันที่ : 2/3/2025  
  ⏰ เวลา : 21:22:20
  ```
- มีปุ่ม ⬅️ / ➡️ สำหรับเลื่อนหน้า (3 รายการต่อหน้า)
- รองรับผลลัพธ์หลายแบบ (`Role`, `Point`, `Item`)

---

### 🖼️ 3. ระบบโหลดภาพกลาง (`utils/assets.py`)
> ดึงรูปทั้งหมดจาก `config/images.json` แทนการฝัง URL ในโค้ด

ตัวอย่าง `images.json`:
```json
{
  "icons": {
    "wallet": "https://media.discordapp.net/.../wallet_icon.png"
  },
  "banners": {
    "topup": "https://media.discordapp.net/.../topup_banner.png"
  }
}
```
ใช้งานในโค้ด:
```python
from utils.assets import get_asset
wallet_icon = get_asset("icons.wallet")
banner = get_asset("banners.topup")
```

---

### 🧾 4. Log แอดมิน
> ทุกคำสั่งสำคัญจะบันทึก embed ลงช่อง log ตาม `wheelConfig["log"]`

| Log ประเภท | ใช้ในคำสั่ง | ตัวอย่าง Embed |
|--------------|--------------|----------------|
| check_point | `/admin check-point` | ตรวจสอบพอยต์ |
| add_point | `/admin add-point` | เพิ่มยอด |
| remove_point | `/admin remove-point` | หักยอด |
| move_point | ปุ่มโอนพอยต์ | โอนระหว่างผู้ใช้ |

---

### 🪄 5. UI ใหม่ใน Discord
> ปรับหน้าตา Embed และปุ่มให้統一ธีม Diamond-Shopz

#### ✅ คำสั่ง `/admin setup`
สร้างเมนูแอดมิน:
- ปุ่ม: `เช็คพอย`, `ลบพอย`, `โอนพอย`
- Embed มีแบนเนอร์, ไอคอน, สีฟ้าสวยงาม
- ใส่ชื่อแอดมินผู้สร้างใน footer

---

## 🧠 สิ่งที่พัฒนาเพิ่มจากของเดิม

| หัวข้อ | เดิม | ใหม่ |
|---------|------|------|
| โค้ดโครงสร้าง | ไฟล์เดียวรวมทุกอย่าง | แยก Cog / Utils / Config |
| ฐานข้อมูล | MongoDB | MySQL / phpMyAdmin |
| รูปภาพ | URL ฝังในโค้ด | ดึงจาก `config/images.json` |
| ประวัติสุ่ม | ไม่มี | ✅ มี + ปุ่มเลื่อนหน้า |
| Embed แสดงผล | ข้อความธรรมดา | ✅ สวยงาม แบนเนอร์ + ไอคอน |
| ระบบ log | เฉพาะบางคำสั่ง | ✅ ครบทุกคำสั่ง |
| ระบบปุ่ม/Modal | ไม่มี | ✅ มีปุ่ม+Modal แยกตามฟังก์ชัน |
| โอนพอยต์ | ไม่มี | ✅ เพิ่มให้พร้อมตรวจสอบยอด |
| Audit & Error | ไม่มีตรวจสอบ | ✅ ป้องกันยอดติดลบ / ตรวจสิทธิ์ |

---

## 🔒 ความปลอดภัย & สิทธิ์
- ตรวจสิทธิ์ผ่าน `ADMIN_ROLES` ก่อนอนุญาตทุกคำสั่ง
- ตรวจไม่ให้ใช้งานใน DM
- ป้องกันยอดติดลบ
- Log ทุกการกระทำ

---

## 🧩 วิธีเพิ่มภาพใหม่
1. เปิด `config/images.json`
2. เพิ่ม key ใหม่ เช่น:
   ```json
   "banners": {
     "topup": "https://cdn.discordapp.com/attachments/.../topup.png",
     "gacha": "https://cdn.discordapp.com/attachments/.../gacha.png"
   }
   ```
3. เรียกใช้งาน:
   ```python
   banner = get_asset("banners.gacha")
   ```

---

## 🛠️ วิธีใช้งาน
1. ตรวจสอบว่า config ครบ:
   - `config/config.json` มี token / guild / log ID
   - `config/images.json` มีรูปที่ต้องการ
2. รันบอท:
   ```bash
   py main.py
   ```
3. ใช้คำสั่ง:
   - `/admin setup`
   - `/admin check-point`
   - `/admin add-point`
   - `/admin remove-point`
   - `/admin gacha-history`

---

## 📌 Roadmap (เวอร์ชันถัดไป)
- [ ] `/admin topup-history` – ประวัติการเติมเงิน
- [ ] `/admin move-history` – ประวัติการโอนแต้ม
- [ ] ระบบ Export รายงาน (PDF/CSV)
- [ ] Theme system (เปลี่ยนสี Embed ผ่าน config)

---

## 👑 Credit
**Project:** Diamond-Shopz  
**Developer:** [DIAMOND]  
**Language:** Python 3.9+ / Nextcord  
**Database:** MySQL (phpMyAdmin)  
**Version:** v2.0  
**Updated:** 22 October 2025  

---

> 💎 “เวอร์ชันนี้คือก้าวใหญ่ของระบบแอดมิน Diamond-Shopz  
> จากโค้ดรวมเดิม → สู่องค์ประกอบที่แยกชัด ปลอดภัย และสวยงาม”
