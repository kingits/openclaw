# การเรียกใช้ GitHub Actions ด้วย Pipeline CI

## สิ่งที่เปลี่ยนแปลง

### 1. เพิ่ม workflow_dispatch trigger
ตอนนี้คุณสามารถเรียกใช้ GitHub Actions ได้แบบ manual จาก GitHub UI:

**วิธีการ:**
1. ไปที่แท็บ **Actions** ในรีโพสิทอรี
2. เลือก workflow ที่ต้องการ (เช่น "CI")
3. คลิกปุ่ม **Run workflow**
4. เลือก branch ที่ต้องการรัน
5. คลิก **Run workflow** อีกครั้ง

### 2. เพิ่ม branch filters
CI จะรันโดยอัตโนมัติเมื่อ:
- Push ไปที่ `main` branch
- Push ไปที่ `copilot/**` branches
- สร้าง Pull Request

### 3. เอกสารประกอบ
สร้างเอกสารครบถ้วนเกี่ยวกับ CI pipeline ที่ `docs/reference/ci.md` ซึ่งอธิบาย:
- วิธีการรัน workflows
- โครงสร้างของ test matrix
- วิธีการ debug failures
- การตั้งค่า runners
- แนวทางการเพิ่ม checks ใหม่

## ไฟล์ที่แก้ไข

1. `.github/workflows/ci.yml`
   - เพิ่ม `workflow_dispatch` trigger สำหรับการรันแบบ manual
   - เพิ่ม branch filters (`main`, `copilot/**`)

2. `.github/workflows/workflow-sanity.yml`
   - เพิ่ม `workflow_dispatch` trigger

3. `docs/reference/ci.md`
   - เอกสารใหม่ที่อธิบายทุกอย่างเกี่ยวกับ CI pipeline

4. `docs/docs.json`
   - เพิ่มลิงก์ไปยังเอกสาร CI ใน navigation

## การใช้งาน

### รัน CI แบบอัตโนมัติ
CI จะรันโดยอัตโนมัติเมื่อคุณ:
- Push code ไปที่ `main` หรือ `copilot/**` branches
- สร้าง Pull Request

### รัน CI แบบ Manual
1. ไปที่ https://github.com/kingits/openclaw/actions
2. เลือก workflow "CI"
3. คลิก "Run workflow"
4. เลือก branch
5. คลิก "Run workflow" เพื่อเริ่มการรัน

### ตรวจสอบผลการรัน
- ไปที่ https://github.com/kingits/openclaw/actions
- คลิกที่ workflow run ที่ต้องการดู
- ดู logs ของแต่ละ job

## เอกสารเพิ่มเติม

อ่านเอกสารฉบับเต็มได้ที่:
- [CI Documentation](docs/reference/ci.md) - ภาษาอังกฤษ
- [Testing Documentation](docs/testing.md) - สำหรับ developers

## การทดสอบ Local

ก่อน push code ควรรัน checks เหล่านี้:

```bash
# Full gate (แนะนำก่อน push)
pnpm lint && pnpm build && pnpm test

# Individual checks
pnpm lint              # Code linting
pnpm format            # Format checking
pnpm build             # TypeScript build
pnpm test              # Unit tests
```

## ปัญหาที่แก้ไข

ตอนนี้ GitHub Actions สามารถ:
- ✅ รันแบบอัตโนมัติเมื่อ push/PR
- ✅ รันแบบ manual ได้จาก GitHub UI
- ✅ กรอง branches ที่ไม่จำเป็น
- ✅ มีเอกสารครบถ้วนสำหรับการใช้งาน
