# payment-analyzer

OpenClaw agent สำหรับวิเคราะห์หลักฐานการชำระเงินจากอีเมล และจับคู่กับ Google Sheet อัตโนมัติ

---

## ความสามารถ

- อ่านอีเมลที่ยังไม่อ่านและตรวจสอบว่าเป็นหลักฐานการชำระเงินหรือไม่
- ดึงข้อมูล: เลข invoice, ยอดเงิน, WHT 3%, ชื่อผู้โอน
- OCR สลิปโอนเงินที่แนบมาเป็นรูปภาพ (ภาษาไทย/อังกฤษ)
- บันทึกอีเมลพร้อม attachment เป็น PDF รายเดือน
- จับคู่กับ Google Sheet และอัปเดตสถานะอัตโนมัติ
- ย้ายอีเมลไปโฟลเดอร์ `payment` และ tag `done`
- บันทึก log รายเดือน

---

## Requirements

| เครื่องมือ | หมายเหตุ |
|-----------|---------|
| OpenClaw | latest |
| [openclaw-skills](https://github.com/saharuth-nu/openclaw-skills) | mail-inet + file-tools |
| [gog CLI](https://github.com/nicholasgasior/gog) | Google Sheets API |
| Python ≥ 3.10 | สำหรับ file-tools |
| Node.js ≥ 18 | สำหรับ mail-inet |

---

## Installation

### 1. ติดตั้ง openclaw-skills ก่อน

ทำตามขั้นตอนใน [openclaw-skills](https://github.com/saharuth-nu/openclaw-skills)

### 2. Clone agent

```bash
cd ~/.openclaw/workspace
git clone https://github.com/saharuth-nu/payment-analyzer.git
```

### 3. ตั้งค่า env

```bash
cp payment-analyzer/.env.example payment-analyzer/.env
```

แก้ไข `payment-analyzer/.env`:

```env
GSHEET_SPREADSHEET_ID=your-spreadsheet-id
GSHEET_SHEET_NAME=Sheet1
GSHEET_ACCOUNT=your@gmail.com
```

> **SPREADSHEET_ID** คือส่วนนี้ใน URL:
> `https://docs.google.com/spreadsheets/d/`**`SPREADSHEET_ID`**`/edit`

### 4. Login gog

```bash
gog auth login your@gmail.com
```

### 5. สร้าง agent dir และ models.json

```bash
mkdir -p ~/.openclaw/agents/payment-analyzer/agent
```

สร้าง `~/.openclaw/agents/payment-analyzer/agent/models.json`:

```json
{
  "providers": {
    "anthropic": {
      "models": [
        {
          "id": "claude-sonnet-4-5",
          "name": "Claude Sonnet"
        }
      ]
    }
  }
}
```

### 6. ลงทะเบียน agent ใน openclaw.json

```json
{
  "agents": {
    "list": [
      {
        "id": "payment-analyzer",
        "name": "Payment Analyzer",
        "workspace": "/Users/<username>/.openclaw/workspace/payment-analyzer",
        "agentDir": "/Users/<username>/.openclaw/agents/payment-analyzer/agent",
        "skills": ["mail-inet"]
      }
    ]
  }
}
```

### 7. ตรวจสอบ

```bash
openclaw config validate
```

---

## Google Sheet Format

Sheet ต้องมี columns A–J ตามลำดับ:

| Col | ชื่อ column |
|-----|------------|
| A | เลข Billing |
| B | ชื่อลูกค้า |
| C | BillingAmount รวมvat |
| D | ExVat |
| E | วันที่วางบิล |
| F | วิธีการวางบิล |
| G | รายละเอียดวางบิล |
| H | สถานะวางบิล |
| I | สถานะรับเงิน |
| J | ปี |

**ค่าของ สถานะรับเงิน:** `รอรับเงิน` / `ได้รับเงินแล้ว` / `ชำระบางส่วน` / `เกินกำหนด` / `ยกเลิก`

---

## Output

```
~/oneauthen-payment/
├── 2026-06/
│   ├── 2026-06-01_<uid>_<subject>.pdf
│   └── ...
└── 2026-07/
    └── ...

~/.openclaw/workspace/payment-analyzer/memory/
└── payment-2026-06.log
```

### Log format

```
[2026-06-01 10:30:48] uid=6083 subject="ส่งหลักฐานฯ" amount=13493.80 invoice=OAT-INV09 status=done
[2026-06-01 10:31:35] uid=6095 subject="ชำระเงิน" amount=53457.94 invoice=- status=รอผู้ใช้ตรวจสอบ
```

---

## การใช้งาน

เปิด OpenClaw → เลือก agent **Payment Analyzer** แล้วพิมพ์:

```
วิเคราะห์อีเมลหลักฐานการชำระเงิน
```
