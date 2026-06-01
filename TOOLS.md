# TOOLS.md — Payment Analyzer Tools

## Email Tools (mail-inet skill)

```bash
MAIL_SKILL=~/.openclaw/workspace/skills/mail-inet

# List unread emails
node $MAIL_SKILL/scripts/list-emails.js --unread --limit 10

# Get full email content + attachments list
node $MAIL_SKILL/scripts/get-email.js <uid>

# Download attachment
node $MAIL_SKILL/scripts/download-attachment.js <uid> \
  --filename "<name>" \
  --output "/tmp/payment-analyzer/<uid>_<name>"

# Mark email as read
node $MAIL_SKILL/scripts/mark-email.js <uid>
```

## File Analysis Tools (file-tools skill)

```bash
FILE_SKILL=~/.openclaw/workspace/skills/file-tools

# Extract text from PDF/DOCX/XLSX/PPTX
python3 $FILE_SKILL/scripts/extract-text.py "<path>"

# Extract text from image via OCR (Thai + English)
python3 $FILE_SKILL/scripts/extract-text.py "<path>" --ocr --lang eng+tha
```

## Google Sheets (gog skill)

```bash
# อ่านข้อมูลทั้งหมดจาก sheet (header + data rows)
gog -j sheets get "$GSHEET_SPREADSHEET_ID" "${GSHEET_SHEET_NAME}!A:J" \
  -a "$GSHEET_ACCOUNT"

# อ่านเฉพาะ header row เพื่อดูลำดับ column
gog -j sheets get "$GSHEET_SPREADSHEET_ID" "${GSHEET_SHEET_NAME}!A1:J1" \
  -a "$GSHEET_ACCOUNT"

# อัปเดตค่าในเซลล์ (เมื่อผู้ใช้สั่งเท่านั้น)
gog -j sheets update "$GSHEET_SPREADSHEET_ID" "${GSHEET_SHEET_NAME}!I<row>" "ได้รับเงินแล้ว" \
  -a "$GSHEET_ACCOUNT"
```

## Env Vars

เก็บใน `~/.openclaw/workspace/payment-analyzer/.env` — โหลดก่อนใช้ gog:

```bash
set -a && source ~/.openclaw/workspace/payment-analyzer/.env && set +a
```

| Variable | ความหมาย |
|----------|----------|
| `GSHEET_SPREADSHEET_ID` | ID จาก URL ของ Google Sheet |
| `GSHEET_SHEET_NAME` | ชื่อ tab (default: Sheet1) |
| `GSHEET_ACCOUNT` | Google account ที่ login ไว้ใน gog |

## Save Email as PDF (file-tools skill)

```bash
FILE_SKILL=~/.openclaw/workspace/skills/file-tools

python3 $FILE_SKILL/scripts/email-to-pdf.py \
  --uid      "<uid>" \
  --date     "<YYYY-MM-DD>" \
  --subject  "<subject>" \
  --from     "<sender_email>" \
  --body     "<email_body_text>" \
  --attach   "<attachment_path_1>" \
  --attach   "<attachment_path_2>" \
  --output-dir ~/oneauthen-payment
```

- ชื่อไฟล์: `YYYY-MM-DD_<uid>_<subject>.pdf`
- สร้างโฟลเดอร์ `~/oneauthen-payment/` อัตโนมัติถ้ายังไม่มี
- รวมเนื้อหาอีเมล + ทุก attachment ไว้ใน PDF เดียว
- รูปภาพ → embed เป็น image page
- PDF attachment → merge ต่อท้าย
- ไฟล์อื่น (DOCX/XLSX) → extract text แล้ว render เป็น text page

## Temp Directory

```bash
mkdir -p /tmp/payment-analyzer
# ล้างหลัง run เสร็จ (optional)
rm -rf /tmp/payment-analyzer/*
```

## Memory Location

```
~/.openclaw/workspace/payment-analyzer/memory/
├── payment-results-YYYY-MM.md   ← monthly payment log
└── YYYY-MM-DD.md                ← daily run log
```
