# ENGCE301 Final Lab — Set 1: Microservices + HTTPS + Lightweight Logging

## Group Information

- **Section:** —
- **Group No:** PZMT
- **Repository:** `engce301-final-lab1-675432060050-675432060688`

---

## Members

| # | ชื่อ-นามสกุล | รหัสนักศึกษา |
|---|---|---|
| 1 | นายชาคริตส์ แก้วมูลเมือง | 675432060050 |
| 2 | นายพีสิรวิชญ์ ชัยวัชรนนท์ | 675432060688 |

- **งาน Set 1:** https://github.com/Zag5A6167/engce301-final-lab1-675432060050-675432060688
- **งาน Set 2:** https://github.com/Zag5A6167/engce301-final-lab2-675432060050-675432060688

---

## Responsibility Mapping

| Component | Responsible Member |
|-----------|--------------------|
| Nginx / HTTPS / Gateway | นายชาคริตส์ แก้วมูลเมือง |
| Auth Service | นายชาคริตส์ แก้วมูลเมือง |
| Task Service | นายพีสิรวิชญ์ ชัยวัชรนนท์ |
| User Service (Set 2) | นายพีสิรวิชญ์ ชัยวัชรนนท์ |
| Database / Databases | นายชาคริตส์ แก้วมูลเมือง |
| Frontend Main App | นายพีสิรวิชญ์ ชัยวัชรนนท์ |
| Frontend Log Viewer | นายพีสิรวิชญ์ ชัยวัชรนนท์ |
| Logging Integration | นายชาคริตส์ แก้วมูลเมือง |
| Documentation / Testing | ทั้งสองคน |

---

## Architecture

```
Browser / Postman
       │
       │ HTTPS :443  (HTTP :80 → redirect HTTPS)
       ▼
┌─────────────────────────────────────────────────────┐
│  🛡️  Nginx  (API Gateway + TLS + Rate Limiter)       │
│                                                     │
│  /api/auth/*  → auth-service:3001                   │
│  /api/tasks/* → task-service:3002  [JWT required]   │
│  /api/logs/*  → log-service:3003   [JWT required]   │
│  /            → frontend:80        (Static HTML)    │
└────┬──────────────────┬──────────────────┬──────────┘
     │                  │                  │
     ▼                  ▼                  ▼
┌──────────┐   ┌──────────────┐   ┌────────────────┐
│ Auth Svc │   │  Task Svc    │   │  Log Service   │
│  :3001   │   │   :3002      │   │   :3003        │
└────┬─────┘   └──────┬───────┘   └───────┬────────┘
     │                │                   │
     └────────┬────────┘                  │
              ▼                           │
     ┌──────────────────┐                 │
     │   PostgreSQL      │◄────────────────┘
     │  users / tasks /  │
     │  logs  tables     │
     └──────────────────┘
```

---

## วิธีรัน

```bash
# 1. สร้าง Self-Signed Certificate (รันครั้งเดียว)
chmod +x scripts/gen-certs.sh
./scripts/gen-certs.sh

# 2. ตั้งค่า environment
cp .env.example .env

# 3. รัน Docker
docker compose up --build -d

# 4. เข้าใช้งาน
open https://localhost
```

---

## Seed Users

| Username | Email | Password | Role |
|---|---|---|---|
| alice | alice@lab.local | `alice123` | member |
| bob | bob@lab.local | `bob456` | member |
| admin | admin@lab.local | `adminpass` | admin |

---

## HTTPS Flow

1. **Browser** ส่ง request มาที่ `http://localhost:80`
2. **Nginx** ตอบด้วย `301 Redirect` → `https://localhost:443`
3. **TLS Handshake** — Nginx ใช้ Self-Signed Certificate (`nginx/certs/cert.pem`)
4. **Nginx** รับ encrypted request แล้ว **decrypt** (TLS Termination)
5. **Nginx** forward ไปยัง service ที่ถูกต้องผ่าน Docker internal network (HTTP ธรรมดา)
6. Response กลับมาถูก **encrypt** อีกครั้งก่อนส่งไปยัง Browser

> ⚠️ Browser จะแจ้งเตือน "Not Secure" เพราะเป็น Self-Signed Cert — กด **Advanced → Proceed** ได้เลย (ปกติสำหรับ development)

---

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/auth/login` | — | เข้าสู่ระบบ รับ JWT |
| GET | `/api/auth/verify` | Bearer | ตรวจสอบ JWT |
| GET | `/api/auth/me` | Bearer | ข้อมูลผู้ใช้ปัจจุบัน |
| GET | `/api/tasks/` | Bearer | ดึง task ทั้งหมด |
| POST | `/api/tasks/` | Bearer | สร้าง task |
| PUT | `/api/tasks/:id` | Bearer | แก้ไข task |
| DELETE | `/api/tasks/:id` | Bearer | ลบ task |
| GET | `/api/logs/` | Bearer | ดึง log ทั้งหมด |
| GET | `/api/logs/stats` | Bearer | สถิติ log |

---

*ENGCE301 Software Design and Development | มหาวิทยาลัยเทคโนโลยีราชมงคลล้านนา*