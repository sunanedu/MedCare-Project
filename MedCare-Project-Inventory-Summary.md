# MedCare Project Inventory Summary

วันที่สรุป: 2026-06-17  
โปรเจกต์: MedCare - เครื่องเตือนและจ่ายยาระบบ IoT พร้อม Mobile App และ Web Dashboard

## 1. สรุปภาพรวม

MedCare เป็นระบบเครื่องจ่ายยา/เตือนรับประทานยา โดยมี ESP32 เป็นตัวควบคุมอุปกรณ์จริง, Backend เป็นศูนย์กลางข้อมูลและคำสั่ง, Web Dashboard สำหรับผู้ดูแล/แอดมิน และ Mobile App สำหรับผู้ใช้หรือผู้ดูแลประจำวัน

โครงสร้างระบบหลัก:

```text
Mobile App / Web Dashboard
        |
        | REST API / WebSocket
        v
Node.js Backend + MySQL + MQTT
        |
        | MQTT / WebSocket / HTTP polling
        v
ESP32 + Servo + Button + Buzzer/Audio
```

สถานะล่าสุดจาก checklist ในโปรเจกต์:

| ส่วนงาน | สถานะโดยรวม |
|---|---:|
| Backend + Database | ประมาณ 98.5% |
| Web Dashboard | ประมาณ 98.4% |
| Mobile App | ประมาณ 98.0% |
| Hardware + Firmware | ประมาณ 86.0% |
| Overall prototype | ประมาณ 96.9% |

## 2. รายการอุปกรณ์ Hardware

### 2.1 อุปกรณ์หลักของเครื่องจ่ายยา

| รายการ | จำนวน | หน้าที่ |
|---|---:|---|
| ESP32 38-pin ESP-WROOM-32 NodeMCU CP2102 | 1 | ตัวควบคุมหลัก, Wi-Fi, BLE provisioning, รับคำสั่งจาก server |
| PCA9685 16-channel PWM Servo Driver | 1 | ขับ servo 7 ช่องผ่าน I2C |
| MG90S Servo 180 degree | 7 | เปิด/ปิดช่องจ่ายยาหมายเลข 1-7 |
| กล่อง/โครงเครื่อง ELPHECO automatic trash bin 20L | 1 | ใช้เป็น enclosure สำหรับต้นแบบเครื่องจ่ายยา |
| กล่องยา 4.5 x 4.5 cm | ตามช่องใช้งาน | บรรจุยาแต่ละรอบก่อนปล่อยลงแก้ว |
| LED push button 16mm | 1 | ปุ่มให้ผู้ใช้กดยืนยันรับยา/รับประทานยา |
| OLED SH1106/SSD1306 128x64 | 1 | แสดงสถานะอุปกรณ์และ debug โดยไม่ต้องเปิด Serial Monitor |
| Buzzer module | 1 | แจ้งเตือนเสียง fallback เมื่อ audio hardware ยังไม่สมบูรณ์ |
| Pairing/status LED | 1 | แสดงสถานะ pairing/provisioning |

### 2.2 ระบบเสียงและ Storage

| รายการ | จำนวน | หน้าที่ |
|---|---:|---|
| PCM5102A GY-PCM5102 I2S DAC | 1 | แปลงสัญญาณเสียง I2S จาก ESP32 เป็น analog audio |
| microSD card 8GB | 1 | เก็บไฟล์เสียงแจ้งเตือน |
| microSD card module for ESP32 SPI | 1 | ให้ ESP32 อ่านไฟล์เสียงจาก SD card |
| FANTECH GS733 active speaker | 1 | ลำโพงพร้อม amplifier |

### 2.3 ระบบไฟและความปลอดภัย

| รายการ | จำนวน | หน้าที่ |
|---|---:|---|
| MEAN WELL 5V 3A PSU | 1 | ไฟเลี้ยง servo rail ขั้นต่ำ |
| ESP32 5V 2A adapter | 1 | ไฟเลี้ยง ESP32 แยกจาก servo |
| Capacitor 1000uF 6.3V+ | 1 | ลด voltage drop ที่ servo rail |
| Fuse 3A-5A หรือ switch | 1 | เพิ่มความปลอดภัยของไฟเลี้ยง servo |

### 2.4 Pin Mapping สำคัญของ ESP32

| Function | GPIO |
|---|---:|
| I2C SDA สำหรับ PCA9685 + OLED | GPIO21 |
| I2C SCL สำหรับ PCA9685 + OLED | GPIO22 |
| PCM5102A DIN / DATA | GPIO25 |
| PCM5102A BCK / BCLK | GPIO26 |
| PCM5102A LCK / LRCK / WS | GPIO27 |
| SD Card SCK | GPIO18 |
| SD Card MISO | GPIO19 |
| SD Card MOSI | GPIO23 |
| SD Card CS | GPIO4 |
| ปุ่มรับยา | GPIO34 |
| LED ปุ่ม / Status LED | GPIO33 |
| Buzzer | GPIO32 |
| Pairing LED / Onboard LED | GPIO2 |

### 2.5 Servo Channel Mapping

| ช่องยา | PCA9685 Channel | Firmware Channel |
|---:|---:|---:|
| 1 | PWM 0 | 0 |
| 2 | PWM 1 | 1 |
| 3 | PWM 2 | 2 |
| 4 | PWM 3 | 3 |
| 5 | PWM 4 | 4 |
| 6 | PWM 5 | 5 |
| 7 | PWM 6 | 6 |

## 3. ซอฟต์แวร์และเทคโนโลยีที่ใช้

### 3.1 Backend

| หมวด | ใช้ |
|---|---|
| Runtime | Node.js 22 LTS |
| Framework | Express.js |
| Database driver | mysql2 |
| Validation | Joi |
| Auth | JWT, bcryptjs |
| Logging | pino, pino-http |
| Security middleware | helmet, cors |
| Realtime | WebSocket สำหรับ ESP32/device และ realtime client |
| MQTT | mqtt package, publish command ไปที่ device topic |
| Test | Node test runner, supertest |

Backend อยู่ที่:

```text
backend/
backend/src/app.js
backend/src/server.js
backend/src/modules/
backend/migrations/
```

### 3.2 Database

| หมวด | ใช้ |
|---|---|
| Database | MySQL 8.0 |
| Migration | SQL migrations ใน `backend/migrations/` |
| Schema docs | `database/schema.sql`, `Database-Design.md` |

ตารางสำคัญที่พบ:

| ตาราง | หน้าที่ |
|---|---|
| users, refresh_tokens | ผู้ใช้และ session |
| care_team_memberships | สมาชิกทีม/ผู้ดูแล |
| devices, device_configs | เครื่อง ESP32 และ config ของเครื่อง |
| device_slots | mapping ช่องยา 1-7 กับยา |
| medications | ข้อมูลยา |
| schedules, schedule_medications | ตารางยาและยาหลายรายการต่อรอบ |
| dose_events | สถานะรอบรับยา เช่น scheduled, alerting, taken, missed |
| command_queue | คิวคำสั่งไปยัง ESP32 |
| device_heartbeats | heartbeat จากอุปกรณ์ |
| dispense_logs | log การจ่ายยา |
| notifications | inbox แจ้งเตือน |
| notification_policy_settings | ตั้งค่าการแจ้งเตือน/quiet hours/escalation |
| audio_files | ไฟล์เสียงแจ้งเตือน |
| audit_logs | audit log งานสำคัญ |
| password_reset_tokens, email_verification_tokens | forgot password และ email verification |

### 3.3 Web Dashboard เดิม

| หมวด | ใช้ |
|---|---|
| Framework | React 18 + Vite |
| Routing | react-router-dom |
| Server state | TanStack Query |
| HTTP | Axios |
| UI icons | lucide-react |
| Styling | Tailwind CSS |

Path:

```text
web/
web/src/App.jsx
web/src/pages/
web/src/lib/api.js
```

### 3.4 Dashboard Web V2

| หมวด | ใช้ |
|---|---|
| Framework | React 18 + Vite |
| Language | TypeScript + JSX บางส่วน |
| Routing | react-router-dom |
| Server state | TanStack Query |
| HTTP | Axios |
| UI icons | lucide-react |
| Styling | Tailwind CSS / CSS |

Path:

```text
dashboard/
dashboard/src/App.tsx
dashboard/src/pages/
dashboard/src/lib/api.js
```

### 3.5 Mobile App

| หมวด | ใช้ |
|---|---|
| Framework | React Native CLI |
| Language | TypeScript |
| React | React 19 |
| Navigation | React Navigation |
| State | Zustand |
| Server state | TanStack Query |
| Form | React Hook Form + Zod |
| HTTP | Axios |
| BLE | react-native-ble-plx |
| UI icons | lucide-react-native |
| Storage | AsyncStorage |
| Build | Android APK ผ่าน Gradle / script `android:apk` |
| Test | Jest, TypeScript typecheck |

Path:

```text
mobile/
mobile/src/navigation/
mobile/src/screens/
mobile/src/lib/api.ts
```

### 3.6 ESP32 Firmware

| หมวด | ใช้ |
|---|---|
| Framework | PlatformIO + Arduino |
| Language | C++ |
| Board | esp32dev |
| Filesystem | SPIFFS |
| JSON | ArduinoJson |
| Servo driver | Adafruit PWM Servo Driver Library |
| Display | Adafruit SSD1306, Adafruit SH110x, Adafruit GFX |
| WebSocket | links2004/WebSockets |
| MQTT | PubSubClient |
| Storage | NVS Preferences + SPIFFS schedule cache |
| Provisioning | BLE GATT |

Path:

```text
firmware/
firmware/platformio.ini
firmware/src/main.cpp
firmware/include/config.example.h
firmware/data/schedules.json
```

### 3.7 เครื่องมือเสริม

| รายการ | หน้าที่ |
|---|---|
| local-lan-server | server mock/development สำหรับ LAN และทดสอบ ESP32 |
| scripts/*.ps1 | start dev stack, check services, apply profile |
| tools/ffmpeg | ใช้จัดการ/แปลงไฟล์เสียง |
| backend/scripts | seed, migrate, QA, relay, test utilities |

## 4. Environment / Profile ที่มี

### 4.1 Profile หลัก

| Profile | ใช้สำหรับ | Backend | Web | ESP32 |
|---|---|---|---|---|
| dev / Home | ทดสอบในบ้านหรือห้อง dev | `http://192.168.1.121:3001/api/v1` | `http://192.168.1.121:5173` | `esp32dev` |
| office | ทดสอบ hardware ที่ office LAN | `http://192.168.20.225:3001/api/v1` | `http://192.168.20.225:5173` | `esp32dev-office-full` |
| production | Cloud VM จริง | `https://medcare.i-esaan.com/api/v1` | `https://medcare.i-esaan.com` | `esp32dev-production-full` |

หมายเหตุ: มี `esp32dev-office-standalone` สำหรับทดสอบบอร์ดเปล่าหรือ fallback profile

### 4.2 Port ที่ใช้บ่อย

| Service | Port |
|---|---:|
| Backend dev/office | 3001 |
| Backend production app port | 3000 |
| Web dev server | 5173 |
| Mobile Metro | 8081 |
| MQTT broker | 1883 |
| MySQL | 3306 |

### 4.3 Backend Environment Variables สำคัญ

| กลุ่ม | ตัวแปร |
|---|---|
| App | `NODE_ENV`, `PORT`, `APP_URL`, `DEVICE_API_BASE_URL`, `WEB_DIST_DIR` |
| Database | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` |
| Auth | `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`, `JWT_ACCESS_EXPIRES`, `JWT_REFRESH_EXPIRES` |
| Device security | `DEVICE_TOKEN_PEPPER` |
| Dose lifecycle | `DOSE_EVENT_LIFECYCLE_ENABLED`, `DOSE_EVENT_SWEEP_INTERVAL_MS`, `DOSE_EVENT_MISSED_GRACE_MINUTES` |
| Device offline monitor | `DEVICE_OFFLINE_MONITOR_ENABLED`, `DEVICE_OFFLINE_SWEEP_INTERVAL_MS`, `DEVICE_OFFLINE_AFTER_MINUTES` |
| Telegram | `TELEGRAM_ENABLED`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` |
| SMTP Email | `SMTP_ENABLED`, `SMTP_HOST`, `SMTP_PORT`, `SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_FROM` |
| Google AI TTS | `GOOGLE_AI_API_KEY`, `GOOGLE_AI_TTS_MODEL`, `GOOGLE_AI_TTS_VOICE` |
| MQTT | `MQTT_ENABLED`, `MQTT_URL`, `MQTT_USERNAME`, `MQTT_PASSWORD`, `MQTT_TOPIC_PREFIX`, `MQTT_QOS` |

### 4.4 Frontend Environment

| App | ตัวแปร |
|---|---|
| web | `VITE_API_BASE_URL`, `VITE_APP_NAME` |
| dashboard | ใช้ API profile selector และ config ใน app |
| mobile | ใช้ API profile ใน app เช่น Production, Office, Home, USB debug |

## 5. ฟังก์ชันที่มีในระบบ

### 5.1 Auth, User, Team

- Register / Login / Refresh token / Logout
- Forgot password และ reset password
- Email verification / resend verification
- JWT auth
- Role-based access: admin, owner, helper, viewer
- Profile API: ดูและแก้ไขข้อมูลผู้ใช้
- Care team/member management
- Audit log สำหรับ action สำคัญ

### 5.2 Device Management

- เพิ่ม/แก้ไข/ลบอุปกรณ์
- Provision device จาก MAC address เพื่อสร้าง device token
- Regenerate device token
- ดู device config
- แก้ไข device config
- ดูสถานะ online/offline, heartbeat, IP, firmware
- ดู command history
- ดู dispense logs
- ลบ logs
- Device offline monitor สร้าง notification เมื่ออุปกรณ์ heartbeat หาย

### 5.3 Device Slots และ Servo

- รองรับช่องยา 1-7
- Mapping slot กับ medication
- Enable/disable slot
- Servo calibration รายช่อง
- คำสั่งทดสอบ servo:
  - `test_servo`
  - `test_servo_cycle`
  - `dispense`
  - `reload_schedule`
  - `play_audio`
  - `reboot`
- รองรับ servo direction กลับทิศได้ผ่าน lock/release angle

### 5.4 Medication

- Medication CRUD
- เพิ่ม/แก้ไข/ลบข้อมูลยา
- รองรับ medication image upload ผ่าน base64 JSON
- รองรับข้อมูลยา เช่น ชื่อยา, dose, note, active state

### 5.5 Schedule

- Schedule CRUD
- หนึ่ง schedule ผูกกับหนึ่ง device และหนึ่ง slot
- หนึ่ง schedule เลือกยาได้หลายรายการผ่าน `schedule_medications`
- รองรับวันในสัปดาห์, เวลา, meal timing, active/inactive
- Validate ว่ามี device, medication และ slot ที่ใช้งานได้
- ESP32 ดึง schedule และ cache ไว้ local

### 5.6 Dose Events และ Care Workflow

- สร้าง event ของรอบรับยาวันนี้จาก active schedules
- Lifecycle job เปลี่ยน scheduled เป็น alerting เมื่อถึงเวลา
- Missed job เปลี่ยนเป็น missed เมื่อเลย grace time
- Mark status ได้ เช่น taken, missed, cancelled
- เก็บ log การจ่ายยาและผลลัพธ์จาก ESP32
- สรุป adherence วันนี้ผ่าน stats endpoint

Care loop เป้าหมาย:

```text
ถึงเวลาเตือน
  -> ESP32 แจ้งเตือนด้วย buzzer/LED/audio
  -> รอผู้ใช้กดปุ่มรับยา
  -> จ่ายยาช่องที่ถูกต้อง
  -> ส่ง log / dose event กลับ backend
  -> แจ้งผู้ดูแลถ้ารับยาแล้วหรือพลาดรับยา
```

### 5.7 Notifications

- Notifications inbox API
- Mark read / mark all read
- Important/unread state
- Notification settings:
  - escalation
  - repeat alerts
  - quiet hours
  - delivery channels
- Telegram adapter สำหรับ missed dose
- Email SMTP adapter มีโครงสร้างแล้ว
- Device offline notification

### 5.8 Audio Files

- List audio files
- Upload audio file
- Generate audio ผ่าน Google AI TTS config
- Update/delete audio metadata
- Device endpoint สำหรับดึง audio files
- Firmware มี command `play_audio` และมี buzzer fallback

### 5.9 Web Dashboard Functions

หน้าหลักที่มีใน `web`:

- Login
- Dashboard
- Medications
- Schedules
- Devices
- Device slots
- Device alert settings
- Escalation settings
- Dose events
- MQTT pipeline diagnostics
- Server/API settings
- Servo calibration
- History
- Notifications
- Audit logs
- Settings

หน้าหลักที่มีใน `dashboard` V2:

- Login / Register / Forgot password
- Dashboard overview
- Devices / Add device wizard / Device detail
- Device slots
- Schedules
- Medications
- Dose events/history
- Notifications
- Caregivers
- Settings hub
- Profile settings
- Notification settings
- Device alert settings
- Server/API settings
- Diagnostics
- Servo calibration
- Audit logs
- Audio files
- Help

### 5.10 Mobile App Functions

- Login / Register / Forgot password
- Home dashboard
- Schedule tab
- Notifications tab พร้อม unread badge
- Settings tab
- Medication list/editor
- Schedule list/editor
- Dose event history
- Profile settings
- Theme settings
- Family members
- Device management
- Device onboarding ผ่าน BLE
- Device slots
- Audio files
- Servo calibration
- Device alert settings
- Notification settings
- Help
- Server/API profile selection
- MQTT/status diagnostics
- Audit logs
- Offline/cached session handling
- Multi-language structure: Thai, English, Chinese

### 5.11 Firmware Functions

- Wi-Fi connection
- BLE provisioning ชื่ออุปกรณ์ `MedCare-XXXXXX`
- รับ `ssid`, `password`, `api_base_url`, `device_token`
- Config fallback chain:
  1. compile-time config
  2. saved NVS config
  3. server config
- Fetch device config
- Fetch schedules
- Cache schedules ใน SPIFFS
- Execute schedules แม้ server unavailable
- Heartbeat
- Command queue ผ่าน MQTT / WebSocket / HTTP polling fallback
- Duplicate command guard
- Servo control ผ่าน PCA9685
- Button confirm state machine
- Buzzer/LED alert fallback
- OLED status display
- Queue failed logs แล้ว retry
- Serial debug commands เช่น `help`, `config`, `ws-status`, `i2c-scan`, `wifi-scan`, `servo`

## 6. API Module Summary

| Module | Base path | ฟังก์ชัน |
|---|---|---|
| auth | `/api/v1/auth` | register, login, refresh, logout, forgot/reset password, email verification |
| users | `/api/v1/users` | profile, access, team management |
| devices | `/api/v1/devices` | device CRUD, provision, config, slots, commands, logs |
| device | `/api/v1/device` | endpoint สำหรับ ESP32: config, schedules, commands, heartbeat, logs |
| medications | `/api/v1/medications` | medication CRUD, image upload |
| schedules | `/api/v1/schedules` | schedule CRUD |
| dose-events | `/api/v1/dose-events` | list dose events, update status |
| notifications | `/api/v1/notifications` | inbox, read state, notification settings |
| audio-files | `/api/v1/audio-files` | list/upload/generate/update/delete audio |
| stats | `/api/v1/stats` | adherence summary |
| system | `/api/v1/system` | MQTT status |
| audit-logs | `/api/v1/audit-logs` | audit log list/export |
| docs | `/api/v1/openapi.json`, `/api/v1/docs` | OpenAPI JSON/docs |

## 7. สิ่งที่ยังต้องทำ / Future Work

### 7.1 Hardware/Firmware

- Calibrate servo angle จริงรายช่อง 1-7 กับกล่องยาจริง
- ทดสอบ flow จริง: alert -> กดปุ่ม -> จ่ายยา -> log -> notification
- ทดสอบ missed-dose timeout
- ทดสอบ offline/reconnect recovery
- Validate BLE provisioning กับ ESP32 จริงให้จบครบ flow
- Validate audio hardware PCM5102A + SD card + speaker ใน final flow
- Run 24-hour soak test
- พิจารณา MQTT over TLS port 8883 หลัง flow หลักนิ่งแล้ว

### 7.2 Mobile

- Validate APK ล่าสุดบนมือถือจริงให้ครบทุก flow
- ทดสอบ BLE onboarding กับเครื่องจริง
- ทำ/ยืนยัน push notification หรือ FCM สำหรับ missed dose/offline
- ทดสอบ session/token storage หลังปิดเปิด app หลายรอบ
- ทดสอบ online/offline mode กับ Production profile

### 7.3 Web/Dashboard

- ทดสอบ production polish ระยะยาว:
  - login
  - refresh token
  - route reload
  - offline session
  - responsive layout
- ตรวจ CRUD หลักกับข้อมูลจริง
- ตรวจ diagnostics/MQTT no-motion command หลายรอบ
- ย้ายหรือยืนยันว่า Dashboard V2 จะเป็นตัวหลักแทน `web` เดิม

### 7.4 Notification/Operation

- ตั้งค่า Telegram chat id จริง
- เติม/เปิดใช้งาน Email, LINE, FCM ตามช่องทางที่ต้องการใช้จริง
- ตรวจ escalation policy และ quiet hours กับเคสจริง
- เพิ่ม export/report ที่จำเป็น เช่น audit log หรือ adherence report

## 8. ไฟล์อ้างอิงสำคัญ

| ไฟล์/โฟลเดอร์ | ใช้ดูเรื่อง |
|---|---|
| `README.md` | ภาพรวมโปรเจกต์และ folder map |
| `System-Architecture.md` | architecture, stack, data flow |
| `Hardware-Specification.md` | BOM, wiring, power |
| `ESP32-Pinout-Current.md` | pinout ปัจจุบันของ ESP32 |
| `Firmware-Specification.md` | firmware behavior และ command |
| `App-Feature-Checklist.md` | สถานะงานและ checklist |
| `Remaining_To_100_Percent_Web_Mobile_ESP32.md` | งานที่เหลือก่อน 100% |
| `Dashboard-Web-V2-Implementation-Spec.md` | spec dashboard ใหม่ |
| `Mobile-App-React-Native-Spec-v2.md` | spec mobile app |
| `backend/package.json` | backend dependencies/scripts |
| `mobile/package.json` | mobile dependencies/scripts |
| `web/package.json` | web dependencies/scripts |
| `dashboard/package.json` | dashboard dependencies/scripts |
| `firmware/platformio.ini` | firmware environments/libraries |
| `config/service-profiles.json` | dev/office/production profiles |
| `backend/.env.example` | backend env template |
| `web/.env.example` | web env template |

