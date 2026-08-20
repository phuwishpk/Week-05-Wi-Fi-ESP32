# รายงานการทดลอง ใบงานที่ 5.1: การเชื่อมต่อ Wi-Fi และการค้นหาสัญญาณรอบข้าง (Wi-Fi Connection and Scanning Phase)

---

## 1. ผลการทดลอง (Experiment Results)

### 1.1 ตารางสรุปเปรียบเทียบการสแกนทั้ง 4 กรณี

| ข้อการทดลอง | เงื่อนไขการสแกน | สถานะ (Success/Error Code) | จำนวน AP ที่พบ (เครือข่าย) | เวลาที่ใช้ในการสแกน (ms) |
| :---: | :--- | :---: | :---: | :---: |
| **5.1.1** | สแกนทั่วไปทุก Channel | ESP_OK (0x0) | 16 | 2,502 ms |
| **5.1.2** | กำหนดสแกนเฉพาะ Channel 1 | ESP_OK (0x0) | 8 | 196 ms |
| **5.1.3** | กำหนดสแกน SSID ที่มีจริง (`"eem_Luna"`) | ESP_OK (0x0) | 1 | 2,495 ms |
| **5.1.4** | กำหนดสแกน SSID ที่ไม่มีจริง (`"NON_EXISTENT_AP_9999"`) | ESP_OK (0x0) | 0 | 2,494 ms |

---

### 1.2 ตารางรายละเอียด AP ที่พบจากการสแกนทั่วไป (ข้อ 5.1.1)

| ลำดับ | ชื่อเครือข่าย (SSID) | MAC Address (BSSID) | ความแรงสัญญาณ (RSSI: dBm) | ช่องความถี่ (Channel) | ประเภทการเข้ารหัส (Encryption Type) |
| :---: | :--- | :--- | :---: | :---: | :--- |
| 1 | `<Hidden SSID>` | 6A:57:07:6F:73:08 | -34 dBm | 6 | WPA2_PSK |
| 2 | eem_Luna | 2E:2B:53:FB:4B:AC | -49 dBm | 1 | WPA2_WPA3_PSK |
| 3 | KMITL-WIFI | 78:17:BE:C0:7D:A1 | -50 dBm | 1 | OPEN (No Password) |
| 4 | KMITL-Legacy | 78:17:BE:C0:7D:A0 | -51 dBm | 1 | WPA2_ENTERPRISE |
| 5 | KMITL-IoT | 78:17:BE:C0:7D:A2 | -51 dBm | 1 | WPA2_PSK |

---

## 2. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **การกำหนดค่าในโครงสร้าง `wifi_scan_config_t` สำหรับสแกนเจาะจงเฉพาะช่องความถี่ (ข้อ 5.1.2) ช่วยลดเวลาในการสแกนเมื่อเทียบกับการสแกนทุกช่องความถี่ (ข้อ 5.1.1) อย่างไร และมีข้อจำกัดอย่างไร?**
   - **คำตอบ:** จากผลการทดลองใน Console Log การสแกนเจาะจงเฉพาะ Channel 1 ใช้เวลาเพียง **196 ms** ในขณะที่การสแกนทุกช่องความถี่ (Channel 1-13) ใช้เวลาถึง **2,502 ms** (ลดเวลาลงได้ถึง ~12.76 เท่า) เนื่องจาก ESP32 ไม่ต้องสลับความถี่วิทยุไปสำรวจช่องความถี่อื่น และไม่ต้องเสียเวลารอ Probe Response ในช่องที่ไม่ต้องการ
   - **ข้อจำกัด:** หาก Access Point (AP) เป้าหมายทำงานอยู่บน Channel อื่น เช่น Channel 6 หรือ 10 (เช่น SSID `<Hidden SSID>` บน Channel 6 ในข้อ 5.1.1) หรือหาก AP มีการสลับ Channel อัตโนมัติ ESP32 จะไม่สามารถค้นพบ AP นั้นได้

2. **เมื่อสังเกตผล Forensic Log ในข้อ 5.1.4 (สแกนหา SSID ที่ไม่มีอยู่จริง) ฟังก์ชัน `esp_wifi_scan_start()`, `esp_wifi_scan_get_ap_num()` และ `esp_wifi_scan_get_ap_records()` ส่งคืนค่าอย่างไร?**
   - **คำตอบ:** จาก Forensic Log ในข้อ 5.1.4:
     - `esp_wifi_scan_start()` ส่งคืนค่า **`ESP_OK (0x0)`** (ใช้เวลาสแกน 2,494 ms) เนื่องจากกระบวนการสแกนทำงานสำเร็จตามโปรโตคอล (ไม่ได้เกิด Error ระดับไดรเวอร์)
     - `esp_wifi_scan_get_ap_num()` ส่งคืนค่า **`ESP_OK (0x0)`** โดยตั้งค่าตัวแปร **`ap_count = 0`**
     - `esp_wifi_scan_get_ap_records()` **ไม่ถูกเรียกใช้งาน** เนื่องจาก `ap_count = 0` โปรแกรมจึงไม่ต้องจองหน่วยความจำเพื่อดึงระเบียน AP

3. **ค่าระดับความแรงสัญญาณ (RSSI) ที่แสดงเป็นตัวเลขติดลบ (เช่น -45 dBm กับ -80 dBm) ค่าใดแสดงถึงสัญญาณที่มีความแรงและความเสถียรมากกว่ากัน?**
   - **คำตอบ:** ค่า **`-45 dBm`** (หรือ `-34 dBm` ในข้อ 5.1.1 ลำดับที่ 1) มีความแรงและความเสถียรมากกว่า เนื่องจาก RSSI (Received Signal Strength Indicator) มีหน่วยเป็น dBm ซึ่งคิดในสเกล Logarithmic ตัวเลขติดลบยิ่งเข้าใกล้ 0 มากเท่าใด กำลังวัตต์ของสัญญาณวิทยุที่รับได้ยิ่งสูงเท่านั้น (เช่น `-34 dBm` ของ `<Hidden SSID>` มีสัญญาณแรงและเสถียรกว่า `-77 dBm` ของ `KMITL-WIFI` อย่างมาก)

4. **เหตุใดการดึงค่า `authmode` (`wifi_auth_mode_t`) จากโครงสร้าง `wifi_ap_record_t` จึงมีความสำคัญต่อการเตรียมการในเฟสถัดไป (Authentication & Association Phase)?**
   - **คำตอบ:** มีความสำคัญมากเพราะในเฟสถัดไป ESP32 จำเป็นต้องทราบประเภทระบบรักษาความปลอดภัยของ AP ล่วงหน้า เพื่อจัดเตรียม Handshake Protocol และ Security Credentials ให้ตรงกัน เช่น:
     - หากเป็น `OPEN (No Password)` สามารถข้ามขั้นตอนแลกเปลี่ยนรหัสผ่านแล้วเข้าสู่เฟส Association ได้ทันที
     - หากเป็น `WPA2_PSK` หรือ `WPA2_WPA3_PSK` ต้องเตรียมรหัสผ่าน (Pre-Shared Key) สำหรับการทำ 4-Way Handshake
     - หากเป็น `WPA2_ENTERPRISE` ต้องเตรียม EAP Username/Password สำหรับส่งไปยัง RADIUS Server
     - หากกำหนด `authmode` ไม่ตรงกับ AP การเชื่อมต่อในเฟสถัดไปจะถูกปฏิเสธและล้มเหลวทันที

---

# รายงานการทดลอง ใบงานที่ 5.2: การยืนยันตัวตน การสถาปนาการเชื่อมต่อ และการรับหมายเลข IP Address (Wi-Fi Connection & IP Assignment)

## 1. ผลการทดลอง (Experiment Results)

### 1.1 ตารางสรุปเปรียบเทียบผลการทดลองทั้ง 3 สถานการณ์

| ข้อการทดลอง | สถานการณ์ทดสอบ                     |    Event สุดท้ายที่ได้รับ     | ผลลัพธ์ (Passed/Failed) | Reason Code (Decimal / Hex) | คำอธิบาย Reason Code                                                 |
| :---------: | :--------------------------------- | :---------------------------: | :---------------------: | :-------------------------: | :------------------------------------------------------------------- |
|  **5.2.1**  | SSID และ Password ถูกต้อง          |     `IP_EVENT_STA_GOT_IP`     |         Passed          |             N/A             | เชื่อมต่อและได้รับ IP Address สำเร็จ                                 |
|  **5.2.2**  | ระบุ SSID ผิด (ไม่มีในระบบ)        | `WIFI_EVENT_STA_DISCONNECTED` |         Failed          |       `201` (`0xC9`)        | `WIFI_REASON_NO_AP_FOUND` (ไม่พบ AP ที่มี SSID ดังกล่าว)             |
|  **5.2.3**  | ระบุ SSID ถูกต้อง แต่ Password ผิด | `WIFI_EVENT_STA_DISCONNECTED` |         Failed          |        `2` (`0x02`)         | `WIFI_REASON_AUTH_EXPIRE` (ยืนยันตัวตนล้มเหลวเนื่องจาก Password ผิด) |

---

### 1.2 บันทึกข้อมูลเครือข่ายจากการเชื่อมต่อสำเร็จ (ข้อ 5.2.1)

| พารามิเตอร์เครือข่าย    | ค่าที่ได้รับจริงจาก DHCP / Event |
| :---------------------- | :------------------------------- |
| **SSID**                | `PhuwishP`                       |
| **BSSID (MAC Address)** | `A6:AE:BD:24:7B:30`              |
| **Channel**             | `6`                              |
| **IP Address**          | `172.20.10.2`                    |
| **Subnet Mask**         | `255.255.255.240`                |
| **Default Gateway**     | `172.20.10.1`                    |

---

## 2. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **เหตุใดการระบุ SSID ผิด (ข้อ 5.2.2) จึงส่งผลให้เกิด Disconnect Event ด้วย Reason Code `201` (`WIFI_REASON_NO_AP_FOUND`) ตั้งแต่เฟส Scan?**
   - **คำตอบ:** เมื่อระบุ SSID ที่ไม่มีอยู่จริง ESP32 จะทำการส่ง Probe Request ในช่วง Scan Phase เพื่อค้นหา AP ที่ตรงกับ SSID นั้น เมื่อสแกนทุก Channel แล้วไม่ได้รับ Probe Response ตอบกลับมา ระบบจึงไม่สามารถเข้าสู่เฟส Authentication ได้ ไดรเวอร์ Wi-Fi จึงแจ้ง Event `WIFI_EVENT_STA_DISCONNECTED` พร้อมระบุ Reason Code `201` (`WIFI_REASON_NO_AP_FOUND`) ทันที

2. **เหตุใดการพิมพ์ Password ผิด (ข้อ 5.2.3) จึงผ่านเฟส Auth และ Assoc มาได้ แต่มาล้มเหลวในเฟส 4-Way Handshake (Reason Code `15` หรือ `204`)?**
   - **คำตอบ:** ในมาตรฐาน IEEE 802.11i (WPA2/WPA3):
     - **Authentication Phase (เฟส 2):** เป็นเพียงการยืนยันตัวตนระดับ Open System ลิงก์เบื้องต้น ไม่มีการส่ง/ตรวจเช็กรหัสผ่าน Wi-Fi
     - **Association Phase (เฟส 3):** เป็นการตกลงพารามิเตอร์เครือข่ายและรับค่า Association ID (AID) จาก AP
     - **4-Way Handshake Phase (เฟส 4):** เป็นขั้นตอนที่ AP และ ESP32 ใช้ Pre-Shared Key (PSK) คำนวณ Pairwise Master Key (PMK) เพื่อสร้างและยืนยัน Pairwise Transient Key (PTK) ผ่านแพ็กเกจ EAPOL หาก Password ผิด ทั้งสองฝั่งจะไม่สามารถถอดรหัสและพิสูจน์ทราบ PTK ที่ตรงกันได้ จึงเกิด Timeout และตัดการเชื่อมต่อด้วย Reason Code `15` (`WIFI_REASON_HANDSHAKE_TIMEOUT`) หรือ `204` (`WIFI_REASON_4WAY_HANDSHAKE_TIMEOUT`)

3. **ลำดับการเกิด Event ระหว่าง `WIFI_EVENT_STA_CONNECTED` กับ `IP_EVENT_STA_GOT_IP` Event ใดเกิดขึ้นก่อนกัน และมีความหมายทางกายภาพของ Layer Network ต่างกันอย่างไร?**
   - **คำตอบ:**
     - **`WIFI_EVENT_STA_CONNECTED` เกิดขึ้นก่อน:** หมายถึงการสถาปนาการเชื่อมต่อสำเร็จในระดับ **Data Link Layer (Layer 2 - Wi-Fi Protocol / 802.11)** โดย ESP32 กับ AP ผ่านกระบวนการ Auth, Assoc และ 4-Way Handshake เรียบร้อยแล้ว
     - **`IP_EVENT_STA_GOT_IP` เกิดขึ้นทีหลัง:** หมายถึงการสถาปนาการเชื่อมต่อสำเร็จในระดับ **Network Layer (Layer 3 - TCP/IP Protocol)** โดย ESP32 สื่อสารกับ DHCP Server เพื่อขอและได้รับหมายเลข IP Address, Subnet Mask และ Gateway เรียบร้อยแล้ว

4. **สมาชิกตัวแปร `reason` ในโครงสร้าง `wifi_event_sta_disconnected_t` มีประโยชน์อย่างไรต่อการออกแบบระบบค้นหาสาเหตุและกู้คืนการเชื่อมต่อ (Auto-Reconnection Mechanism) ในแอปพลิเคชัน IoT?**
   - **คำตอบ:** ช่วยให้นักพัฒนาสร้างกลไกการแก้ไขปัญหาและกู้คืนการเชื่อมต่อแบบชาญฉลาด (Smart Auto-Reconnection Strategy) ตามสาเหตุจริง เช่น:
     - หากเกิดจาก `WIFI_REASON_NO_AP_FOUND` (201) หรือ `BEACON_TIMEOUT` (200) อาจเป็นเพราะบอร์ดอยู่นอกระยะสัญญาณ ให้ทำการชะลอเวลา (Exponential Backoff) แล้วลองสแกน/เชื่อมต่อใหม่
     - หากเกิดจาก `WIFI_REASON_HANDSHAKE_TIMEOUT` (15) หรือ `AUTH_FAIL` (202) แสดงว่า Password ผิดพลาด ให้หยุดการพยายามลองเชื่อมต่อซ้ำ และแจ้งเตือนผู้ใช้ให้กำหนดค่า Credentials ใหม่ผ่าน Captive Portal หรือ Bluetooth
     - หากเกิดจาก `WIFI_REASON_ASSOC_FAIL` หรือหลุดชั่วคราว ให้สั่ง `esp_wifi_connect()` ใหม่ได้ทันที

---

# รายงานการทดลอง ใบงานที่ 5.3: การยืนยันตัวตนและการผูกสัมพันธ์ในระดับ Link Layer (Authentication & Association Phase)

---

## 1. ผลการทดลอง (Experiment Results)

### 1.1 ตารางสรุปเปรียบเทียบผลการทดลองในระดับ Link Layer

| ข้อการทดลอง | สถานการณ์ทดสอบ | Event ที่ได้รับ | ผลการผูกสัมพันธ์ Link Layer | ค่า Association ID (AID) ที่ได้ | Reason Code (ถ้ามี) |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.3.1** | ร้องขอ Auth & Assoc กับ AP มีอยู่จริง | `WIFI_EVENT_STA_CONNECTED` | สำเร็จ (Link Layer Connected) | 3 | N/A |
| **5.3.2** | ร้องขอ Auth & Assoc กับ AP ไม่มีอยู่จริง | `WIFI_EVENT_STA_DISCONNECTED` | ล้มเหลว (SSID Not Found) | N/A | 201 (`0xC9`, `WIFI_REASON_NO_AP_FOUND`) |

---

### 1.2 บันทึกข้อมูล Link Layer จาก Event `WIFI_EVENT_STA_CONNECTED` (ข้อ 5.3.1)

| พารามิเตอร์ Link Layer | ค่าที่อ่านได้จริงจาก Forensic Log |
| :--- | :--- |
| **SSID** | `PhuwishP` |
| **BSSID (MAC Address)** | `82:CA:F7:29:29:6C` |
| **Channel** | `1` |
| **Auth Mode Enum** | `3` (`WIFI_AUTH_WPA2_PSK`) |
| **Association ID (AID)** | `3` |

---

## 2. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **Association ID (AID) คืออะไร มีบทบาทอย่างไรใน Phase 3 และส่งคืนมาในโครงสร้างข้อมูลตัวแปรใด?**
   - **คำตอบ:**
     - **ความหมายและบทบาท:** AID คือหมายเลขระบุตัวตนชั่วคราว (16-bit integer ช่วง 1-2007) ที่ Access Point (AP) กำหนดให้แก่ Station (STA) ในขั้นตอน 802.11 Association Response (Phase 3) เพื่อใช้เป็นดัชนีระบุตัวเครื่อง ESP32 ในตาราง Association Table ของ AP และใช้ในการจัดการพลังงาน Power Save Mode (Listen Interval / TIM Bitmap)
     - **ตัวแปรที่ส่งคืน:** ถูกส่งคืนมาในโครงสร้างข้อมูล **`wifi_event_sta_connected_t`** ในฟิลด์ **`event->aid`** (ผ่าน Event `WIFI_EVENT_STA_CONNECTED`)

2. **เหตุใดการเชื่อมต่อ Wi-Fi ความปลอดภัยแบบ WPA2-PSK จึงสามารถผ่าน Phase 2 (Authentication) และ Phase 3 (Association) จนเกิด Event `WIFI_EVENT_STA_CONNECTED` ได้สำเร็จ แม้ผู้ใช้จะป้อนรหัสผ่าน (Password) ผิด?**
   - **คำตอบ:** เพราะตามมาตรฐาน IEEE 802.11 กระบวนการใน **Phase 2 (Authentication)** ของ WPA2 ใช้รูปแบบ **Open System Authentication** (แลกเปลี่ยนเพียง 2 เฟรมเพื่อตกลงว่าไม่ได้ใช้ระบบรักษาความปลอดภัยแบบเก่าอย่าง WEP) และ **Phase 3 (Association)** เป็นการตกลงคุณสมบัติฮาร์ดแวร์/อัตราบิต (Capabilities & Supported Rates) ซึ่งทั้ง Phase 2 และ Phase 3 **ยังไม่ได้มีการตรวจสอบรหัสผ่าน Pre-Shared Key (PSK) หรือทำ 4-Way Handshake** แต่อย่างใด (การตรวจสอบรหัสผ่านจะเกิดขึ้นใน **Phase 4: 4-Way Handshake** ภายหลังการผูกสัมพันธ์ระดับ Link Layer เสร็จสิ้นแล้วเท่านั้น)

3. **หาก Router มีการตั้งค่า MAC Address Filtering (อนุญาตเฉพาะ MAC ที่ลงทะเบียน) ESP32 จะล้มเหลวในเฟสใด และจะส่ง Disconnect Reason Code ใดออกมา?**
   - **คำตอบ:** ESP32 จะล้มเหลวใน **Phase 2 (Authentication Phase)** หรือ **Phase 3 (Association Phase)** เนื่องจาก AP จะปฏิเสธเฟรม Auth Request หรือ Assoc Request ของ MAC ที่ไม่อนุญาต โดยส่ง Disconnect Reason Code:
     - **`WIFI_REASON_AUTH_FAIL` (Reason 1 หรือ 202)** ในเฟส Authentication หรือ
     - **`WIFI_REASON_ASSOC_FAIL` (Reason 3 หรือ 203)** ในเฟส Association

4. **สรุปความแตกต่างสำคัญระหว่างจุดสิ้นสุดของ Phase 3 (Link-Layer Connected) กับจุดสิ้นสุดของ Phase 5 (IP Address Assigned)**
   - **คำตอบ:**
     - **Phase 3 (Link-Layer Connected / `WIFI_EVENT_STA_CONNECTED`):**
       - ทำงานในระดับ **Data Link Layer (L2)** ตามมาตรฐาน IEEE 802.11
       - บอร์ด ESP32 และ AP สามารถสื่อสารเฟรมวิทยุกันได้แล้ว มี AID และ BSSID ประจำตัว
       - **ยังไม่สามารถส่งข้อมูลแพ็กเก็ต IP / Internet ได้** และยังไม่ได้ผ่านการตรวจสอบรหัสผ่าน WPA2 (ถ้ามี)
     - **Phase 5 (IP Address Assigned / `IP_EVENT_STA_GOT_IP`):**
       - ทำงานในระดับ **Network Layer (L3)** ตามมาตรฐาน TCP/IP
       - บอร์ด ESP32 ผ่านกระบวนการแลกเปลี่ยนคีย์ความปลอดภัย WPA2 (Phase 4) และขอรับหมายเลข IP Address, Subnet Mask, Gateway จาก DHCP Server บน AP เรียบร้อยแล้ว
       - **พร้อมสำหรับการรับส่งข้อมูลบนเครือข่ายอินเทอร์เน็ต (Sockets, HTTP, MQTT ฯลฯ)**

---

# รายงานการทดลอง ใบงานที่ 5.4: กระบวนการแลกเปลี่ยนคีย์ความปลอดภัยและการจัดสรรหมายเลข IP Address (4-Way Handshake & IP Assignment Phase)


---

## 1. ผลการทดลอง (Experiment Results)

### 1.1 ตารางสรุปเปรียบเทียบผลการทดลองใน Handshake & IP Phase

| ข้อการทดลอง | สถานการณ์ทดสอบ                         | Event `WIFI_EVENT_STA_CONNECTED` (เกิด/ไม่เกิด) | Event `IP_EVENT_STA_GOT_IP` (เกิด/ไม่เกิด) | ผลการทดลอง | Disconnect Reason Code (ถ้ามี)                                                              |
| :-----------:| :---------------------------------------| :-----------------------------------------------:| :------------------------------------------:| :----------:| :--------------------------------------------------------------------------------------------|
| **5.4.1**   | Password ถูกต้อง (`"123456789"`)       | เกิด                                            | เกิด                                       | Passed     | N/A                                                                                         |
| **5.4.2**   | Password ผิด (`"WRONG_PASSWORD_1234"`) | เกิด                                            | ไม่เกิด                                    | Failed     | `2` (`0x02`, `WIFI_REASON_AUTH_EXPIRE`) หรือ `15` (`0x0F`, `WIFI_REASON_HANDSHAKE_TIMEOUT`) |

---

### 1.2 บันทึกข้อมูล IP Network จาก Event `IP_EVENT_STA_GOT_IP` (ข้อ 5.4.1)

| พารามิเตอร์ Network Layer | ค่าที่จัดสรรได้จริงจาก DHCP Server |
| :------------------------ | :--------------------------------- |
| **IP Address**            | `172.20.10.2`                      |
| **Subnet Mask**           | `255.255.255.240`                  |
| **Default Gateway**       | `172.20.10.1`                      |

---

## 2. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **เหตุใดกระบวนการ 4-Way Handshake จึงพิสูจน์ทราบรหัสผ่าน Wi-Fi ได้โดยไม่ต้องส่งรหัสผ่าน (Passphrase) ลอยไปในอากาศเลยแม้แต่แพ็กเกจเดียว?**
   - **คำตอบ:** เพราะ 4-Way Handshake ใช้หลักการ **Challenge-Response และ Message Integrity Code (MIC)** โดยทั้ง AP และ ESP32 จะนำ Passphrase และ SSID มาคำนวณล่วงหน้าเป็น **PMK (Pairwise Master Key)** จากนั้นแลกเปลี่ยนค่าสุ่ม **ANonce** (จาก AP) และ **SNonce** (จาก ESP32) เพื่อสร้าง **PTK (Pairwise Transient Key)** แยกกันคนละฝั่ง จากนั้นใช้ส่วนหนึ่งของ PTK มาคำนวณค่าแฮช **MIC** สรุปแนบไปกับเฟรม EAPOL-Key หากรหัสผ่านตรงกัน ค่า MIC ที่คำนวณได้จากทั้งสองฝั่งจะตรงกัน 100% จึงพิสูจน์ทราบรหัสผ่านได้โดยไม่ต้องส่ง Passphrase จริงผ่านอากาศ

2. **อธิบายบทบาทและที่มาของคีย์ PMK (Pairwise Master Key) และ PTK (Pairwise Transient Key) ว่ามีความสัมพันธ์กันอย่างไรในการเข้ารหัสเฟรมข้อมูล?**
   - **คำตอบ:**
     - **PMK (Pairwise Master Key):** เป็นคีย์หลักที่เกิดจากการนำ Passphrase และ SSID ผ่านฟังก์ชัน PBKDF2 (SHA-1 HMAC 4096 รอบ) คีย์นี้จะคงที่ตราบใดที่รหัสผ่าน Wi-Fi ไม่เปลี่ยน
     - **PTK (Pairwise Transient Key):** เป็นคีย์ชั่วคราวที่สร้างขึ้นใหม่ทุกครั้งที่มีการเชื่อมต่อ เกิดจากฟังก์ชัน PRF(PMK + ANonce + SNonce + AP_MAC + STA_MAC)
     - **ความสัมพันธ์:** **PMK ทำหน้าที่เป็นต้นกำเนิดความลับ (Master Secret)** ในขณะที่ **PTK คือคีย์ใช้งานจริง (Operational Keys)** ที่แบ่งออกเป็น KCK (ตรวจ MIC), KEK (ถอดรหัส GTK) และ TK (ใช้เข้ารหัสแพ็กเก็ตข้อมูล Unicast Data Packets ผ่าน AES-CCMP จริงบนอากาศ)

3. **เหตุใดเมื่อเราพิมพ์ Password ผิด (ข้อ 5.4.2) ESP32 จึงยังคงได้รับ Event `WIFI_EVENT_STA_CONNECTED` ก่อนที่จะเกิด Event `WIFI_EVENT_STA_DISCONNECTED` ตามมาในภายหลัง?**
   - **คำตอบ:** เพราะ Event **`WIFI_EVENT_STA_CONNECTED`** ถูกส่งออกมาเมื่อสิ้นสุด **Phase 3 (Association Phase)** ซึ่งเป็นเพียงขั้นตอนการตกลงคุณสมบัติและรับค่า Association ID (AID) ในระดับ Link Layer (802.11) เท่านั้น **โดยยังไม่มีการตรวจสอบรหัสผ่าน** แต่เมื่อเข้าสู่ **Phase 4 (4-Way Handshake)** ESP32 คำนวณค่า MIC ส่งไปให้ AP ตรวจสอบ แล้ว AP พบว่า MIC ไม่ตรงกัน (เนื่องจากใช้ Password ผิด) AP หรือ ESP32 จึงส่ง Disconnect Frame ออกมา และเกิด Event **`WIFI_EVENT_STA_DISCONNECTED`** พร้อม Reason Code ตามหลังมาในภายหลัง

4. **หากเครือข่าย Wi-Fi ไม่มี DHCP Server (ไม่มีการแจก IP อัตโนมัติ) ผลการทดลองในข้อ 5.4.1 จะหยุดอยู่ที่ขั้นตอนใด และจะไม่เกิด Event ใดขึ้น?**
   - **คำตอบ:**
     - **ขั้นตอนที่หยุด:** การทดลองจะหยุดอยู่ที่ **สิ้นสุด Phase 4 (4-Way Handshake)** และค้างอยู่ในสถานะรอคอยการตอบกลับจาก DHCP Server ใน **Phase 5 (DHCP Request)**
     - **Event ที่จะไม่เกิดขึ้น:** จะ**ไม่เกิด Event `IP_EVENT_STA_GOT_IP`** เนื่องจาก ESP32 ไม่ได้รับข้อเสนอ (DHCP Offer/ACK) หมายเลข IP Address, Subnet Mask และ Gateway จาก DHCP Server ทำให้ฝั่ง Network Layer (Layer 3) ไม่สมบูรณ์และไม่สามารถสื่อสารผ่านโปรโตคอล TCP/IP ได้
