# DIAMOND Gacha Bot — Full README

> ระบบบอทกาชา (Classic/Premium) + จัดการพอยต์ + เติมเงิน TrueMoney + แผงควบคุมแอดมิน + ระบบตั๋ว + ฟิลเตอร์ลิงก์ + OCR + ข้อความ/อีเวนต์ ยูทิลิตี้  
> สร้างด้วย **Python / nextcord** และ **SQL**

---

## สารบัญ
- [สรุประบบทั้งหมด (High-level)](#สรุประบบทั้งหมด-high-level)
- [โครงสร้างโปรเจค](#โครงสร้างโปรเจค)
- [การติดตั้ง & รัน](#การติดตั้ง--รัน)
- [การตั้งค่า (config)](#การตั้งค่า-config)
- [ฐานข้อมูล (MySQL Schema)](#ฐานข้อมูล-mysql-schema)
- [คำสั่งทั้งหมด (Slash / UI)](#คำสั่งทั้งหมด-slash--ui)
- [คำอธิบายระบบหลัก](#คำอธิบายระบบหลัก)
  - [1) ระบบกาชา (Classic / Premium)](#1-ระบบกาชา-classic--premium)
  - [2) ระบบแอดมิน (จัดการพอยต์/ประวัติ/ตรวจสอบ)](#2-ระบบแอดมิน-จัดการพอยต์ประวัติตรวจสอบ)
  - [3) ระบบเติมเงิน TrueMoney](#3-ระบบเติมเงิน-truemoney)
  - [4) ระบบตั๋วช่วยเหลือ (Tickets)](#4-ระบบตั๋วช่วยเหลือ-tickets)
  - [5) ฟิลเตอร์ลิงก์ต้องห้าม / ข้อความ / OCR](#5-ฟิลเตอร์ลิงก์ต้องห้าม--ข้อความ--ocr)
- [ไฟล์ยูทิลิตี้และคอมโพเนนต์](#ไฟล์ยูทิลิตี้และคอมโพเนนต์)
- [แนวทางดีไซน์ Embed & รูปภาพ (assets)](#แนวทางดีไซน์-embed--รูปภาพ-assets)
- [แนวทางความปลอดภัย](#แนวทางความปลอดภัย)
- [Changelog (สิ่งที่ใหม่/ปรับปรุงจากเวอร์ชั่นเดิม)](#changelog-สิ่งที่ใหม่ปรับปรุงจากเวอร์ชั่นเดิม)
- [License](#license)

---

## สรุประบบทั้งหมด (High-level)

**Core**
- กาชา 2 โหมด: **Classic** และ **Premium (Fix / Price)** พร้อมปุ่ม UI เลือกจำนวนครั้ง
- เติมเงินผ่านลิงก์ซอง **TrueMoney** (ระบบ Redeem อัตโนมัติ + ป้องกันซ้ำ)
- ฐานข้อมูล **MySQL** พร้อมตารางผู้ใช้/ประวัติสุ่ม/เติมเงิน/บันทึกการกระทำ
- จัดการพอยต์โดยแอดมิน (เพิ่ม / ลบ / โอน / ตรวจสอบยอด)
- ตรวจสอบ **ประวัติการสุ่ม** (รวม Classic+Premium) พร้อม **ปุ่มเลื่อนหน้า** (pagination)

**Support**
- ระบบตั๋ว (Ticket) สำหรับเรื่อง **เติมเงินไม่เข้า** ฯลฯ
- ฟิลเตอร์ลิงก์ต้องห้าม (giftlink filter)
- ฟิลเตอร์ข้อความ (message filter)
- OCR อ่านข้อความจากรูปภาพ

**UI/UX**
- ใช้ **nextcord.ui.View / Modal / Button** ทุกจุดสำคัญ
- ใช้ **Embed** สวยงาม พร้อม **ภาพ banner + icon** จาก `config/images.json`
- มี **Leaderboard** รวมยอดเติม Top 10

---

## โครงสร้างโปรเจค

> *ไฟล์/โฟลเดอร์ที่สำคัญ*

```
.
├── cogs/
│   ├── admin/
│   │   └── admin.py              # ระบบแอดมิน: add/remove/check/move point + ประวัติ + เมนูปุ่ม
│   ├── wheel/
│   │   └── wheel.py              # ระบบกาชา + เติมเงิน Wallet + Leaderboard
│   ├── tickets/
│   │   └── __init__.py           # Modal/ระบบตั๋ว (เช่น เติมเงินไม่เข้า)
│   ├── tools/
│   │   └── voucher_embed.py      # Text command: 'voucher_embed' (ตัวอย่าง utility)
│   ├── ban/
│   │   └── giftlink_filter.py    # ฟิลเตอร์ลิงก์ต้องห้าม
│   ├── message/
│   │   └── message_filter.py     # ฟิลเตอร์ข้อความ
│   └── ocr_cog/
│       └── ocr_cog.py            # OCR อ่านข้อความจากภาพ
├── components.py                  # Embed / Wheel utils (logEmbed, waitingEmbed, successEmbed, …)
├── database/
│   └── db_classic.py             # MySQL adapter (MongoDatabase-compatible interface)
│   └── db_premium.py             # (ถ้ามี) โครงสร้างคล้าย classic สำหรับ premium
├── utils/
│   └── assets.py                 # ดึงรูปจาก config/images.json (get_asset / IMAGES)
├── config/
│   ├── images.json               # ไอคอน/แบนเนอร์ที่ใช้ใน Embed
│   ├── database.json             # ENGINE, host, port, user, password, database
│   └── wheel.json / others       # ค่าคอนฟิกระบบกาชา/ล้อ/ช่อง log
├── data/
│   └── leaderboard.json          # เก็บ channel/message id สำหรับ Leaderboard
└── main.py                        # จุดเริ่มรันบอท
```

> *ชื่อไฟล์อาจต่างเล็กน้อยตามโปรเจคจริงของคุณ แต่แนวคิด/การจัดวางนี้รองรับระบบทั้งหมดที่กล่าวถึง*

---

## การติดตั้ง & รัน

### 1) Python & Dependencies
- Python 3.9 (รองรับ 3.9)
- ติดตั้ง library ที่ต้องใช้:
```bash
pip install nextcord mysql-connector-python aiohttp requests
```

### 2) ตั้งค่า Discord Bot
- สร้าง bot ใน Developer Portal
- เพิ่ม TOKEN ในตัวรัน (เช่น `main.py`) และเชิญเข้ากิลด์
- เปิดใช้ **Message Content Intent** หากต้องการอ่านข้อความ

### 3) ตั้งค่า MySQL
- สร้าง database
- ใส่ค่าการเชื่อมต่อใน `config/database.json` (ENGINE ต้องเป็น `"mysql"`)

### 4) ตั้งค่า assets (ภาพ)
แก้ `config/images.json` เช่น:
```json
{
  "icons": {
    "wallet": "https://media.discordapp.net/attachments/1022553398685470783/1022563370811002880/8527952.png?format=webp&quality=lossless"
  },
  "banners": {
    "topup": "https://media.discordapp.net/attachments/1017690797719683113/1166692807495389294/Top_up3.png?format=webp&quality=lossless&width=1872&height=343"
  }
}
```

### 5) รันบอท
```bash
python main.py
```
คุณจะเห็นบันทึก log คล้าย:
```
Successfully to load cogs.tickets
Successfully to load cogs.wheel.wheel
Successfully to load cogs.tools.voucher_embed
Successfully to load cogs.ban.giftlink_filter
Successfully to load cogs.event.event
Successfully to load cogs.message.message_filter
Successfully to load cogs.ocr_cog.ocr_cog
Connected to Discord API
```

---

## การตั้งค่า (config)

**สำคัญ** (ใช้ในหลายจุด):
- `GUILD_IDS`: รายชื่อ guild สำหรับลงทะเบียน slash command
- `OWNER_IDS`: รายชื่อ owner ที่มีสิทธิ์รันคำสั่งบางอย่าง (เช่น `/admin setup`)
- `ADMIN_ROLES`: role id สำหรับอนุญาตคำสั่งแอดมิน
- `wheelConfig`: ช่อง log สำหรับ check/add/remove/move point
- `WHEEL_CLASSIC`, `WHEEL_PREMIUM`, `WHEEL_GUILD`: ราคาสุ่ม, ช่อง log, guild สำหรับแจก role
- `topupConfig`: เบอร์ wallet (main.classic / main.premium), channel log เติม

> ตรวจสอบว่ามี key ครบและค่าเป็น `int/str` ที่ถูกต้อง (เช่น channel ID เป็น int หรือ cast เป็น int ตอนใช้งาน)

---

## ฐานข้อมูล (MySQL Schema)

ระบบใช้ MySQL Schema

ตารางหลัก:

### `users`
| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `discord_id` | VARCHAR(32) PK | ไอดีผู้ใช้ |
| `balance` | INT | แต้มคงเหลือ |
| `total` | INT | ยอดเติมสะสม |
| `random_amount` | INT | จำนวนครั้งสุ่ม (รอใช้กับระบบ Fix) |
| `all_random_amount` | INT | จำนวนครั้งสุ่มรวม |
| `created_at` | DATETIME | |
| `updated_at` | DATETIME | |

### `topups`
| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `id` | BIGINT AI PK | |
| `discord_id` | VARCHAR(32) | FK → users |
| `method` | VARCHAR(32) | เช่น "Wallet" |
| `amount` | INT | จำนวนเงินที่เติม |
| `ref` | VARCHAR(255) UNIQUE | ป้องกันลิงก์ซองซ้ำ |
| `ts` | DATETIME | เวลา |

### `random_history`
| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `id` | BIGINT AI PK | |
| `discord_id` | VARCHAR(32) | FK → users |
| `method` | VARCHAR(16) | `"Price"` หรือ `"Fix"` |
| `amount` | INT | แต้มที่ใช้ |
| `result_json` | TEXT | เก็บ `list[dict]` ผลลัพธ์ (Role/Point/Name) |
| `ts` | DATETIME | เวลา |

### `actions`
| คอลัมน์ | ชนิด | หมายเหตุ |
|---|---|---|
| `id` | BIGINT AI PK | |
| `discord_id` | VARCHAR(32) | FK → users |
| `type` | VARCHAR(32) | AddPoint / RemovePoint / MovePoint / Topup / … |
| `meta` | TEXT | ข้อมูลเพิ่มเติม (เช่น “100 by 1234567890”) |
| `ts` | DATETIME | เวลา |

> นอกจากนี้ wheel.py มี helper เขียน `topup_logs` (external log) เก็บ `mobile, amount, voucher_url, receiver_type, timestamp` สำหรับ auditing เพิ่มเติม (ไม่บังคับใช้)

---

## คำสั่งทั้งหมด (Slash / UI)

### กลุ่ม `/wheel`
- `/wheel classic` — ส่ง Embed + ปุ่มสุ่ม/เช็ค point/เติมเงิน (Classic)
- `/wheel premium` — ส่ง Embed + ปุ่มสุ่ม/เช็ค point/เติมเงิน (Premium)
- `/wheel leaderboard` — สร้าง Leaderboard (Top 10 ตาม `users.total`) และบันทึกลง `data/leaderboard.json`

**ปุ่ม (ตัวอย่าง Classic)**  
- **🎁 สุ่ม 1/10/25 ครั้ง** — ตัดแต้ม (price = จำนวน × `WHEEL_CLASSIC['price']`), สุ่มผล, แจก role/คืนแต้มถ้าเป็นพอยต์
- **🧧 เติมเงิน** — เปิด Modal ให้กรอกลิงก์ซอง → Redeem อัตโนมัติ → บันทึก DB → Log แอดมิน → อัปเดต Leaderboard
- **📃 เช็ค Point** — แสดง embed ยอดคงเหลือ/ยอดเติม/จำนวนสุ่ม (ใช้ภาพ/ไอคอนจาก `images.json`)
- **🎫 เติมเงินไม่เข้า** — เปิด Ticket Modal เพื่อส่งข้อมูลให้ทีมงาน

### กลุ่ม `/admin`
- `/admin setup` — สร้างเมนูแอดมิน (ปุ่ม: เช็คพอย / ลบพอย / โอน Point)
- `/admin check-point <member>` — แสดง embed:
  - เงินคงเหลือ, ยอดเติมรวม, **ยอดสุ่มรวม** (เพิ่มใหม่), mention ผู้ใช้
  - รูป thumbnail / banner จาก `config/images.json`
- `/admin add-point <member> <amount>` — เพิ่มแต้ม พร้อม **embed แบบสรุปผล** (ก่อน/หลัง)
- `/admin remove-point <member> <amount>` — ลบแต้ม พร้อม **embed แบบสรุปผล** (ก่อน/หลัง)
- `/admin gacha-history <member>` — **ใหม่**: ดู *ประวัติการสุ่ม* (ดึงจาก `random_history` ทั้ง Classic+Premium), มี **ปุ่มเลื่อนหน้า**:
  - ส่วนหัว: `สุ่มทั้งหมด X ครั้ง ใช้แต้มรวม Y บาท`
  - แต่ละรายการ: `ประเภท, ใช้แต้ม, ผลลัพธ์, วันที่, เวลา`
- (มี `MoveBalanceModal` ใน UI สำหรับโอนแต้มด้วยปุ่ม)

> ทุกคำสั่งแอดมินมีการตรวจสอบ **ADMIN_ROLES** / **OWNER_IDS** และส่ง log ไปยังช่องที่กำหนดใน `wheelConfig["log"][...]`

### อื่น ๆ
- `Text commands: ['voucher_embed']` — ยูทิลิตี้แสดง embed voucher (ขึ้นกับ cogs/tools/voucher_embed.py)
- `giftlink_filter`, `message_filter`, `ocr_cog` — utility cogs ตามชื่อ

---

## คำอธิบายระบบหลัก

### 1) ระบบกาชา (Classic / Premium)
- สุ่มผ่านปุ่ม (1/10/25 ครั้ง Classic, 1/5/10 ครั้ง Premium Fix)
- ตัดแต้มจาก `users.balance` และบันทึก `random_history`
- ผลลัพธ์ (Role/Point) ถูกแปลงเป็น action: ให้ role หรือคืนแต้มบางส่วน/เต็มจำนวน
- มี log ไปช่องสำหรับติดตาม

### 2) ระบบแอดมิน (จัดการพอยต์/ประวัติ/ตรวจสอบ)
- ฝั่งเพิ่ม/ลบพอยต์: สร้าง embed สรุป **ผู้ใช้ / จำนวนที่เพิ่มหรือลบ / ยอดก่อน / ยอดหลัง** และใช้ **banner + icon** เดียวกับเช็คพอยต์เพื่อความเป็นเอกภาพ
- เช็คพอยต์: แสดงยอด **เงินคงเหลือ / ยอดเติมรวม / ยอดสุ่มรวม**
- ประวัติการสุ่ม: รวม Classic+Premium แสดงผลแบบหน้า มีปุ่ม `< ก่อนหน้า / ถัดไป >`

### 3) ระบบเติมเงิน TrueMoney
- กรอกลิงก์ซอง ➜ Redeem ผ่าน API ➜ ถ้าสำเร็จ:
  - บันทึก `topups` (unique `ref`) + อัปเดต `users.total` และ `users.balance`
  - ส่ง log แอดมิน + อัปเดต Leaderboard
- ป้องกันลิงก์ซ้ำด้วย `INSERT IGNORE` (ดู `db_classic.addTopup`)

### 4) ระบบตั๋วช่วยเหลือ (Tickets)
- มี Modal สำหรับส่ง **ลิงก์ซอง/รายละเอียด** กรณี “เติมเงินไม่เข้า”
- ส่งข้อมูลเข้า Webhook/Channel ทีมงาน

### 5) ฟิลเตอร์ลิงก์ต้องห้าม / ข้อความ / OCR
- cogs แยกอิสระ: บล็อก giftlink ผิดเงื่อนไข, ทำความสะอาดข้อความ, OCR ข้อความจากรูป
- เปิด/ปิดและตั้งค่าได้ในแต่ละ cog

---

## ไฟล์ยูทิลิตี้และคอมโพเนนต์

### `components.py`
- รวม **Embed class** ที่ใช้บ่อย: `logEmbed`, `waitingEmbed`, `successEmbed`, `balanceEmbed`, `topupWalletEmbed`, …
- ใช้ร่วมกันทั้งฝั่งกาชา/แอดมิน
- ลดการซ้ำซ้อนของโค้ด embed

### `utils/assets.py`
- โหลดรูปจาก `config/images.json` และให้ฟังก์ชัน `get_asset("icons.wallet")`, `get_asset("banners.topup")`
- เปลี่ยนรูปได้โดย **ไม่ต้องแก้โค้ด** — เพียงอัปเดต URL ใน `images.json`

---

## แนวทางดีไซน์ Embed & รูปภาพ (assets)
- **Consistency**: ใช้ icon/banner เดียวกันทุกที่ (เช็คพอยต์, add/remove point, ประวัติ) เพื่อประสบการณ์ผู้ใช้ที่สอดคล้อง
- **Config-driven**: ดึงรูปจาก `images.json` ผ่าน `get_asset` ปรับเปลี่ยนได้ง่าย
- **Short & Clear**: ตัวอธิบายภายใน embed กระชับ อ่านง่าย มีหัวข้อ/อีโมจิช่วยนำสายตา

---

## แนวทางความปลอดภัย
- ป้องกัน SQL injection ด้วย **parameterized queries**
- `topups.ref` เป็น **UNIQUE** ป้องกันเติมซ้ำ
- ตรวจสอบสิทธิ์ **ADMIN_ROLES / OWNER_IDS** ทุกคำสั่งสำคัญ
- ตรวจสอบรูปแบบลิงก์ซองด้วย regex ก่อนเรียก API
- การย้ายแต้ม/ตัดแต้มใช้ `GREATEST(balance - x, 0)` ป้องกันค่าติดลบ
- ใช้ `START TRANSACTION` + `COMMIT/ROLLBACK` สำหรับการโอนแต้ม (atomicity)

---

## Changelog (สิ่งที่ใหม่/ปรับปรุงจากเวอร์ชั่นเดิม)

**ใหม่**
- `/admin gacha-history` พร้อม **pagination** และรูปแบบข้อความตามตัวอย่างที่ผู้ใช้ต้องการ
- แก้ `/admin check-point` แสดง **ยอดสุ่มรวม** เพิ่ม
- แสดง **ผลลัพธ์ add/remove point** เป็น embed สรุป (ผู้ใช้/จำนวน/ก่อน/หลัง) + รูป icon/banner จาก `images.json`
- ระบบ **assets** (`utils/assets.py`) + `config/images.json` ให้เปลี่ยนรูปได้โดยไม่ต้องแก้โค้ด
- ปรับโค้ดให้ใช้ `interaction.followup.send` / `response.defer` อย่างถูกที่ เพื่อหลีกเลี่ยง Timeout
- เพิ่ม **validation**: ห้ามจำนวนติดลบ, ตรวจสอบรูปแบบลิงก์, ตรวจ role/สิทธิ์

**ปรับปรุง**
- **MySQL Adapter** (drop-in แทน Mongo): เพิ่มเมธอดรองรับพารามิเตอร์ `action_by`, ป้องกัน duplicate ref, ใช้ธุรกรรมตอน `movePoint`
- Leaderboard อัปเดตอัตโนมัติหลังเติมเงินสำเร็จ
- โครงสร้าง embed/ข้อความสม่ำเสมอ อ่านง่ายขึ้น

ผลลัพธ์: โค้ด **อ่านง่ายขึ้น**, **เสถียรกว่าเดิม**, **ปรับแต่งง่าย** และมี **เครื่องมือแอดมินครบ** ในที่เดียว

---

## License
โปรเจคนี้ไม่มีการระบุสัญญาอนุญาต (proprietary) — โปรดใช้งานภายใต้เงื่อนไขของผู้พัฒนาเซิร์ฟเวอร์ของคุณ
