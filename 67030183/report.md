# รายงานการทดลอง ใบงานที่ 5.1: การเชื่อมต่อ Wi-Fi และการค้นหาสัญญาณรอบข้าง (Wi-Fi Connection and Scanning Phase)

---

## 1. ผลการทดลอง (Experiment Results)

### 1.1 ตารางสรุปเปรียบเทียบการสแกนทั้ง 4 กรณี

| ข้อการทดลอง | เงื่อนไขการสแกน                                      | สถานะ (Success/Error Code) | จำนวน AP ที่พบ (เครือข่าย) | เวลาที่ใช้ในการสแกน (ms) |
| :---------: | :--------------------------------------------------- | :------------------------: | :------------------------: | :----------------------: |
|  **5.1.1**  | สแกนทั่วไปทุก Channel                                |        ESP_OK (0x0)        |             14             |         2,499 ms         |
|  **5.1.2**  | กำหนดสแกนเฉพาะ Channel 1                             |        ESP_OK (0x0)        |             5              |          196 ms          |
|  **5.1.3**  | กำหนดสแกน SSID ที่มีจริง ("WiFi-Test-Guest")         |        ESP_OK (0x0)        |             0              |         2,495 ms         |
|  **5.1.4**  | กำหนดสแกน SSID ที่ไม่มีจริง ("NON_EXISTENT_AP_9999") |        ESP_OK (0x0)        |             0              |         2,494 ms         |

---

### 1.2 ตารางรายละเอียด AP ที่พบจากการสแกนทั่วไป (ข้อ 5.1.1)

| ลำดับ | ชื่อเครือข่าย (SSID) | MAC Address (BSSID) | ความแรงสัญญาณ (RSSI: dBm) | ช่องความถี่ (Channel) | ประเภทการเข้ารหัส (Encryption Type) |
| :---: | :------------------- | :------------------ | :-----------------------: | :-------------------: | :---------------------------------- |
|   1   | `<Hidden SSID>`      | A6:AE:BD:24:7B:30   |          -37 dBm          |           6           | WPA2_PSK                            |
|   2   | KMITL-Legacy         | 78:17:BE:C0:7D:A0   |          -48 dBm          |           1           | WPA2_ENTERPRISE                     |
|   3   | KMITL-IoT            | 78:17:BE:C0:7D:A2   |          -49 dBm          |           1           | WPA2_PSK                            |
|   4   | KMITL-WIFI           | 78:17:BE:C0:7D:A1   |          -51 dBm          |           1           | OPEN (No Password)                  |
|   5   | 😑                   | 62:DD:AB:3C:2D:A6   |          -53 dBm          |          11           | WPA2_PSK                            |

---

## 2. คำถามท้ายการทดลอง (Post-Lab Questions)

1. **การกำหนดค่าในโครงสร้าง `wifi_scan_config_t` สำหรับสแกนเจาะจงเฉพาะช่องความถี่ (ข้อ 5.1.2) ช่วยลดเวลาในการสแกนเมื่อเทียบกับการสแกนทุกช่องความถี่ (ข้อ 5.1.1) อย่างไร และมีข้อจำกัดอย่างไร?**
   - **คำตอบ:** จากผลการทดลองใน Console Log การสแกนเจาะจงเฉพาะ Channel 1 ใช้เวลาเพียง **196 ms** ในขณะที่การสแกนทุกช่องความถี่ (Channel 1-13) ใช้เวลาถึง **2,499 ms** (ลดเวลาลงได้ถึง ~12.75 เท่า) เนื่องจาก ESP32 ไม่ต้องสลับความถี่วิทยุไปสำรวจช่องความถี่อื่น และไม่ต้องเสียเวลารอ Probe Response ในช่องที่ไม่ต้องการ
   - **ข้อจำกัด:** หาก Access Point (AP) เป้าหมายทำงานอยู่บน Channel อื่น เช่น Channel 6 หรือ 11 (เช่น SSID `<Hidden SSID>` บน Channel 6 หรือ `😑` บน Channel 11 ในข้อ 5.1.1) หรือหาก AP มีการสลับ Channel อัตโนมัติ ESP32 จะไม่สามารถค้นพบ AP นั้นได้

2. **เมื่อสังเกตผล Forensic Log ในข้อ 5.1.4 (สแกนหา SSID ที่ไม่มีอยู่จริง) ฟังก์ชัน `esp_wifi_scan_start()`, `esp_wifi_scan_get_ap_num()` และ `esp_wifi_scan_get_ap_records()` ส่งคืนค่าอย่างไร?**
   - **คำตอบ:** จาก Forensic Log ในข้อ 5.1.4:
     - `esp_wifi_scan_start()` ส่งคืนค่า **`ESP_OK (0x0)`** (ใช้เวลาสแกน 2,494 ms) เนื่องจากกระบวนการสแกนทำงานสำเร็จตามโปรโตคอล (ไม่ได้เกิด Error ระดับไดรเวอร์)
     - `esp_wifi_scan_get_ap_num()` ส่งคืนค่า **`ESP_OK (0x0)`** โดยตั้งค่าตัวแปร **`ap_count = 0`**
     - `esp_wifi_scan_get_ap_records()` **ไม่ถูกเรียกใช้งาน** เนื่องจาก `ap_count = 0` โปรแกรมจึงไม่ต้องจองหน่วยความจำเพื่อดึงระเบียน AP

3. **ค่าระดับความแรงสัญญาณ (RSSI) ที่แสดงเป็นตัวเลขติดลบ (เช่น -45 dBm กับ -80 dBm) ค่าใดแสดงถึงสัญญาณที่มีความแรงและความเสถียรมากกว่ากัน?**
   - **คำตอบ:** ค่า **`-45 dBm`** (หรือ `-37 dBm` ในข้อ 5.1.1 ลำดับที่ 1) มีความแรงและความเสถียรมากกว่า เนื่องจาก RSSI (Received Signal Strength Indicator) มีหน่วยเป็น dBm ซึ่งคิดในสเกล Logarithmic ตัวเลขติดลบยิ่งเข้าใกล้ 0 มากเท่าใด กำลังวัตต์ของสัญญาณวิทยุที่รับได้ยิ่งสูงเท่านั้น (เช่น `-37 dBm` ของ `<Hidden SSID>` ลำดับที่ 1 มีสัญญาณแรงและเสถียรกว่า `-89 dBm` ของ `RADAR1902601900480` ลำดับที่ 14 อย่างมาก)

4. **เหตุใดการดึงค่า `authmode` (`wifi_auth_mode_t`) จากโครงสร้าง `wifi_ap_record_t` จึงมีความสำคัญต่อการเตรียมการในเฟสถัดไป (Authentication & Association Phase)?**
   - **คำตอบ:** มีความสำคัญมากเพราะในเฟสถัดไป ESP32 จำเป็นต้องทราบประเภทระบบรักษาความปลอดภัยของ AP ล่วงหน้า เพื่อจัดเตรียม Handshake Protocol และ Security Credentials ให้ตรงกัน เช่น:
     - หากเป็น `OPEN (No Password)` สามารถข้ามขั้นตอนแลกเปลี่ยนรหัสผ่านแล้วเข้าสู่เฟส Association ได้ทันที
     - หากเป็น `WPA2_PSK` ต้องเตรียมรหัสผ่าน (Pre-Shared Key) สำหรับการทำ 4-Way Handshake
     - หากเป็น `WPA2_ENTERPRISE` ต้องเตรียม EAP Username/Password สำหรับส่งไปยัง RADIUS Server
     - หากกำหนด `authmode` ไม่ตรงกับ AP การเชื่อมต่อในเฟสถัดไปจะถูกปฏิเสธและล้มเหลวทันที

---

## 3. Console Log จากการทดลองจริง (Lab 5.1)

```text
I (31) boot: ESP-IDF v5.2.3 2nd stage bootloader
I (31) boot: compile time Aug  3 2026 09:34:53
I (31) boot: Multicore bootloader
I (35) boot: chip revision: v3.1
I (39) boot.esp32: SPI Speed      : 40MHz
I (44) boot.esp32: SPI Mode       : DIO
I (48) boot.esp32: SPI Flash Size : 2MB
I (53) boot: Enabling RNG early entropy source...
I (58) boot: Partition Table:
I (62) boot: ## Label            Usage          Type ST Offset   Length
I (69) boot:  0 nvs              WiFi data        01 02 00009000 00006000
I (77) boot:  1 phy_init         RF data          01 01 0000f000 00001000
I (84) boot:  2 factory          factory app      00 00 00010000 00100000
I (92) boot: End of partition table
I (96) esp_image: segment 0: paddr=00010020 vaddr=3f400020 size=1f984h (129412) map
I (149) esp_image: segment 1: paddr=0002f9ac vaddr=3ffb0000 size=0066ch (  1644) load
I (149) esp_image: segment 2: paddr=00030020 vaddr=400d0020 size=823c4h (533444) map
I (337) esp_image: segment 3: paddr=000b23ec vaddr=3ffb066c size=03660h ( 13920) load
I (343) esp_image: segment 4: paddr=000b5a54 vaddr=40080000 size=16b48h ( 93000) load
I (391) boot: Loaded app from partition at offset 0x10000
I (391) boot: Disabling RNG early entropy source...
I (402) cpu_start: Multicore app
I (412) cpu_start: Pro cpu start user code
I (412) cpu_start: cpu freq: 160000000 Hz
I (412) cpu_start: Application information:
I (415) cpu_start: Project name:     wifi_scan_phase_lab1
I (421) cpu_start: App version:      37de524-dirty
I (427) cpu_start: Compile time:     Aug  3 2026 09:34:47
I (433) cpu_start: ELF file SHA256:  a15a1b3e1...
I (438) cpu_start: ESP-IDF:          v5.2.3
I (443) cpu_start: Min chip rev:     v0.0
I (448) cpu_start: Max chip rev:     v3.99
I (452) cpu_start: Chip rev:         v3.1
I (457) heap_init: Initializing. RAM available for dynamic allocation:
I (465) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (470) heap_init: At 3FFB7DA8 len 00028258 (160 KiB): DRAM
I (477) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (483) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (489) heap_init: At 40096B48 len 000094B8 (37 KiB): IRAM
I (497) spi_flash: detected chip: generic
I (500) spi_flash: flash io: dio
W (504) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (518) main_task: Started on CPU0
I (528) main_task: Calling app_main()
I (528) LAB_WIFI_SCAN: [FORENSIC]: Call nvs_flash_init()
I (548) LAB_WIFI_SCAN: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (548) LAB_WIFI_SCAN: [FORENSIC]: Call esp_netif_init()
I (548) LAB_WIFI_SCAN: [FORENSIC]: esp_netif_init() returned ESP_OK (0x0)
I (558) LAB_WIFI_SCAN: [FORENSIC]: Call esp_event_loop_create_default()
I (568) LAB_WIFI_SCAN: [FORENSIC]: esp_event_loop_create_default() returned ESP_OK (0x0)
I (578) LAB_WIFI_SCAN: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (588) LAB_WIFI_SCAN: [FORENSIC]: esp_netif_create_default_wifi_sta() returned pointer 0x3ffbd730
I (588) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_init(&cfg)
I (608) wifi:wifi driver task: 3ffbfd34, prio:23, stack:6656, core=0
I (618) wifi:wifi firmware version: 82f54d2
I (618) wifi:wifi certification version: v7.0
I (618) wifi:config NVS flash: enabled
I (618) wifi:config nano formating: disabled
I (628) wifi:Init data frame dynamic rx buffer num: 32
I (628) wifi:Init static rx mgmt buffer num: 5
I (628) wifi:Init management short buffer num: 32
I (638) wifi:Init dynamic tx buffer num: 32
I (638) wifi:Init static rx buffer size: 1600
I (648) wifi:Init static rx buffer num: 10
I (648) wifi:Init dynamic rx buffer num: 32
I (658) wifi_init: rx ba win: 6
I (658) wifi_init: tcpip mbox: 32
I (658) wifi_init: udp mbox: 6
I (668) wifi_init: tcp mbox: 6
I (668) wifi_init: tcp tx win: 5760
I (668) wifi_init: tcp rx win: 5760
I (678) wifi_init: tcp mss: 1440
I (678) wifi_init: WiFi IRAM OP enabled
I (688) wifi_init: WiFi RX IRAM OP enabled
I (688) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_init() returned ESP_OK (0x0)
I (698) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (708) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_set_mode() returned ESP_OK (0x0)
I (708) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_start()
I (718) phy_init: phy_version 4840,a40f0ac,Aug 16 2024,16:48:06
I (798) wifi:mode : sta (84:1f:e8:20:55:24)
I (798) wifi:enable tsf
I (808) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (808) LAB_WIFI_SCAN: ==================================================================
I (808) LAB_WIFI_SCAN:   Lab 5.1: Wi-Fi Connection and Scanning Phase (ESP-IDF Forensic)
I (818) LAB_WIFI_SCAN: ==================================================================
I (828) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (838) LAB_WIFI_SCAN: >>> Experiment 5.1.1: General AP Scan (All Channels)
I (848) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (858) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (3368) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 2499 ms]
I (3368) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (3368) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=14
I (3378) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (3388) LAB_WIFI_SCAN: [AP COUNT]: 14 network(s) found
I (3388) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_records(&number, ap_info)
I (3398) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_records() returned ESP_OK (0x0), records=14

--------------------------------------------------------------------------------------------------
No.  | SSID                     | MAC Address (BSSID) | RSSI   | Chan | Encryption Type
--------------------------------------------------------------------------------------------------
1    | <Hidden SSID>            | A6:AE:BD:24:7B:30 | -37  dBm | 6    | WPA2_PSK
2    | KMITL-Legacy             | 78:17:BE:C0:7D:A0 | -48  dBm | 1    | WPA2_ENTERPRISE
3    | KMITL-IoT                | 78:17:BE:C0:7D:A2 | -49  dBm | 1    | WPA2_PSK
4    | KMITL-WIFI               | 78:17:BE:C0:7D:A1 | -51  dBm | 1    | OPEN (No Password)
5    | 😑                     | 62:DD:AB:3C:2D:A6 | -53  dBm | 11   | WPA2_PSK
6    | KMITL-WIFI               | 78:17:BE:C0:66:21 | -72  dBm | 1    | OPEN (No Password)
7    | KMITL-Legacy             | 78:17:BE:C0:66:20 | -75  dBm | 1    | WPA2_ENTERPRISE
8    | KMITL-IoT                | 78:17:BE:C0:66:22 | -75  dBm | 1    | WPA2_PSK
9    | KMITL-Legacy             | 78:17:BE:C0:72:60 | -75  dBm | 11   | WPA2_ENTERPRISE
10   | KMITL-IoT                | 78:17:BE:C0:72:62 | -75  dBm | 11   | WPA2_PSK
11   | KMITL-Legacy             | 78:17:BE:C0:66:60 | -78  dBm | 11   | WPA2_ENTERPRISE
12   | KMITL-WIFI               | 78:17:BE:C0:72:61 | -79  dBm | 11   | OPEN (No Password)
13   | KMITL-WIFI               | 78:17:BE:C0:66:61 | -82  dBm | 11   | OPEN (No Password)
14   | RADAR1902601900480       | DA:BC:38:B3:75:51 | -89  dBm | 1    | WPA_WPA2_PSK
--------------------------------------------------------------------------------------------------

I (4558) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (4558) LAB_WIFI_SCAN: >>> Experiment 5.1.2: Channel-Specific Scan (Channel 1)
I (4558) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (4568) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (4778) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 196 ms]
I (4778) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (4778) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=5
I (4788) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (4798) LAB_WIFI_SCAN: [AP COUNT]: 5 network(s) found
I (4798) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_records(&number, ap_info)
I (4808) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_records() returned ESP_OK (0x0), records=5

--------------------------------------------------------------------------------------------------
No.  | SSID                     | MAC Address (BSSID) | RSSI   | Chan | Encryption Type
--------------------------------------------------------------------------------------------------
1    | KMITL-Legacy             | 78:17:BE:C0:7D:A0 | -49  dBm | 1    | WPA2_ENTERPRISE
2    | KMITL-IoT                | 78:17:BE:C0:7D:A2 | -50  dBm | 1    | WPA2_PSK
3    | KMITL-WIFI               | 78:17:BE:C0:7D:A1 | -52  dBm | 1    | OPEN (No Password)
4    | KMITL-WIFI               | 78:17:BE:C0:66:21 | -73  dBm | 1    | OPEN (No Password)
5    | KMITL-Legacy             | 78:17:BE:C0:65:E0 | -92  dBm | 1    | WPA2_ENTERPRISE
--------------------------------------------------------------------------------------------------

I (5898) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (5898) LAB_WIFI_SCAN: >>> Experiment 5.1.3: Targeted SSID Scan - Existing ("WiFi-Test-Guest")
I (5898) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (5908) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (8418) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 2495 ms]
I (8418) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (8418) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=0
I (8428) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (8438) LAB_WIFI_SCAN: [AP COUNT]: 0 network(s) found
W (8438) LAB_WIFI_SCAN: [NOTE]: No Access Point found matching the criteria.
I (9448) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (9448) LAB_WIFI_SCAN: >>> Experiment 5.1.4: Targeted SSID Scan - Non-Existent ("NON_EXISTENT_AP_9999")
I (9448) LAB_WIFI_SCAN: ------------------------------------------------------------------
I (9458) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_start(scan_config, block=true)
I (11968) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_start() returned ESP_OK (0x0) [Duration: 2494 ms]
I (11968) LAB_WIFI_SCAN: [FORENSIC]: Call esp_wifi_scan_get_ap_num(&ap_count)
I (11968) LAB_WIFI_SCAN: [FORENSIC]: esp_wifi_scan_get_ap_num() returned ESP_OK (0x0), ap_count=0
I (11978) LAB_WIFI_SCAN: [STATUS]: Scan SUCCESS
I (11988) LAB_WIFI_SCAN: [AP COUNT]: 0 network(s) found
W (11988) LAB_WIFI_SCAN: [NOTE]: No Access Point found matching the criteria.
I (11998) LAB_WIFI_SCAN: ==================================================================
I (12008) LAB_WIFI_SCAN:   [Phase 1 Completed: Wi-Fi Scan Finished]
I (12018) LAB_WIFI_SCAN:   Program stopped after scanning. Auth/Assoc Phase not started.
I (12028) LAB_WIFI_SCAN: ==================================================================
I (12028) main_task: Returned from app_main()
```

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

## 3. Console Log จากการทดลองจริง (Lab 5.2)

```text
I (529) LAB_WIFI_CONN: [FORENSIC]: Call nvs_flash_init()
I (549) LAB_WIFI_CONN: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (549) LAB_WIFI_CONN: [FORENSIC]: Call esp_netif_init()
I (549) LAB_WIFI_CONN: [FORENSIC]: Call esp_event_loop_create_default()
I (559) LAB_WIFI_CONN: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (569) LAB_WIFI_CONN: [FORENSIC]: esp_netif_create_default_wifi_sta() returned 0x3ffbd754
I (579) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_init(&cfg)
I (589) wifi:wifi driver task: 3ffbfd58, prio:23, stack:6656, core=0
I (599) wifi:wifi firmware version: 82f54d2
I (599) wifi:wifi certification version: v7.0
I (599) wifi:config NVS flash: enabled
I (599) wifi:config nano formating: disabled
I (609) wifi:Init data frame dynamic rx buffer num: 32
I (609) wifi:Init static rx mgmt buffer num: 5
I (619) wifi:Init management short buffer num: 32
I (619) wifi:Init dynamic tx buffer num: 32
I (619) wifi:Init static rx buffer size: 1600
I (629) wifi:Init static rx buffer num: 10
I (629) wifi:Init dynamic rx buffer num: 32
I (639) wifi_init: rx ba win: 6
I (639) wifi_init: tcpip mbox: 32
I (639) wifi_init: udp mbox: 6
I (649) wifi_init: tcp mbox: 6
I (649) wifi_init: tcp tx win: 5760
I (659) wifi_init: tcp rx win: 5760
I (659) wifi_init: tcp mss: 1440
I (659) wifi_init: WiFi IRAM OP enabled
I (669) wifi_init: WiFi RX IRAM OP enabled
I (669) LAB_WIFI_CONN: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (679) LAB_WIFI_CONN: [FORENSIC]: Call esp_event_handler_instance_register(IP_EVENT)
I (689) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (699) LAB_WIFI_CONN: ==================================================================
I (709) LAB_WIFI_CONN:   Lab 5.2: Wi-Fi Connection & IP Assignment (ESP-IDF Forensic)
I (709) LAB_WIFI_CONN: ==================================================================
I (719) LAB_WIFI_CONN:

I (729) LAB_WIFI_CONN: ------------------------------------------------------------------
I (739) LAB_WIFI_CONN: >>> Experiment 5.2.1: Connection Test - Correct Credentials
I (739) LAB_WIFI_CONN: ------------------------------------------------------------------
I (749) LAB_WIFI_CONN:   Target SSID: "PhuwishP"
I (759) LAB_WIFI_CONN:   Target Password: "123456789"
I (759) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_stop()
I (769) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
I (819) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (819) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_start()
I (829) phy_init: phy_version 4840,a40f0ac,Aug 16 2024,16:48:06
I (909) wifi:mode : sta (84:1f:e8:20:55:24)
I (909) wifi:enable tsf
I (909) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (909) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (909) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (919) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_connect()
I (929) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (1799) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1
I (1799) wifi:state: init -> auth (b0)
I (1809) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (1819) wifi:state: auth -> assoc (0)
I (1829) wifi:state: assoc -> run (10)
I (1879) wifi:connected with PhuwishP, aid = 2, channel 6, BW20, bssid = a6:ae:bd:24:7b:30
I (1879) wifi:security: WPA2-PSK, phy: bgn, rssi: -38
I (1919) wifi:pm start, type: 1

I (1919) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (1919) LAB_WIFI_CONN: =======================================================
I (1919) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_CONNECTED received!
I (1929) LAB_WIFI_CONN:   -> Connected to SSID : PhuwishP
I (1939) LAB_WIFI_CONN:   -> BSSID            : A6:AE:BD:24:7B:30
I (1939) LAB_WIFI_CONN:   -> Channel          : 6
I (1949) LAB_WIFI_CONN:   -> Auth Mode        : 3
I (1959) LAB_WIFI_CONN: =======================================================
I (1949) wifi:AP's beacon interval = 102400 us, DTIM period = 1
I (2919) esp_netif_handlers: sta ip: 172.20.10.2, mask: 255.255.255.240, gw: 172.20.10.1
I (2919) LAB_WIFI_CONN: =======================================================
I (2919) LAB_WIFI_CONN: [EVENT FORENSIC]: IP_EVENT_STA_GOT_IP received!
I (2929) LAB_WIFI_CONN:   -> IP Address : 172.20.10.2
I (2939) LAB_WIFI_CONN:   -> Netmask    : 255.255.255.240
I (2939) LAB_WIFI_CONN:   -> Gateway    : 172.20.10.1
I (2949) LAB_WIFI_CONN: =======================================================
I (2959) LAB_WIFI_CONN: [RESULT]: TEST PASSED - Connected to AP successfully!
I (4959) LAB_WIFI_CONN:

I (4959) LAB_WIFI_CONN: ------------------------------------------------------------------
I (4959) LAB_WIFI_CONN: >>> Experiment 5.2.2: Connection Test - Wrong SSID (No AP Found)
I (4969) LAB_WIFI_CONN: ------------------------------------------------------------------
I (4969) LAB_WIFI_CONN:   Target SSID: "NON_EXISTENT_SSID_9999"
I (4979) LAB_WIFI_CONN:   Target Password: "12345678"
I (4989) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_stop()
I (4989) wifi:state: run -> init (0)
I (5009) wifi:pm stop, total sleep time: 1846091 us / 3090222 us

I (5009) wifi:new:<6,0>, old:<6,0>, ap:<255,255>, sta:<6,0>, prof:1
E (5009) wifi:NAN WiFi stop
W (5009) LAB_WIFI_CONN: =======================================================
W (5019) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (5029) LAB_WIFI_CONN:   -> Target SSID          : PhuwishP
W (5029) LAB_WIFI_CONN:   -> Reason Code (Decimal): 8
W (5039) LAB_WIFI_CONN:   -> Reason Code (Hex)    : 0x08
W (5039) LAB_WIFI_CONN:   -> Reason Description   : OTHER_DISCONNECT_REASON
W (5049) LAB_WIFI_CONN: =======================================================
I (5059) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (5069) wifi:flush txq
I (5069) wifi:stop sw txq
I (5069) wifi:lmac stop hw txq
I (5069) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
I (5109) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (5109) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_start()
I (5119) wifi:mode : sta (84:1f:e8:20:55:24)
I (5119) wifi:enable tsf
I (5119) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (5129) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (5129) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (5139) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_connect()
I (5149) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
W (5159) LAB_WIFI_CONN: [RESULT]: TEST FAILED - Disconnected event captured.
I (7159) LAB_WIFI_CONN:

I (7159) LAB_WIFI_CONN: ------------------------------------------------------------------
I (7159) LAB_WIFI_CONN: >>> Experiment 5.2.3: Connection Test - Wrong Password (Auth/Handshake Fail)
I (7169) LAB_WIFI_CONN: ------------------------------------------------------------------
I (7179) LAB_WIFI_CONN:   Target SSID: "PhuwishP"
I (7179) LAB_WIFI_CONN:   Target Password: "WRONG_PASS_9999"
I (7189) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_stop()
E (7189) wifi:NAN WiFi stop
I (7199) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (7199) wifi:flush txq
I (7199) wifi:stop sw txq
I (7209) wifi:lmac stop hw txq
I (7209) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
I (7259) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (7259) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_start()
I (7269) wifi:mode : sta (84:1f:e8:20:55:24)
I (7269) wifi:enable tsf
I (7269) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (7279) LAB_WIFI_CONN: [FORENSIC]: Call esp_wifi_connect()
I (7289) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (7269) LAB_WIFI_CONN: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (7559) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1
I (7559) wifi:state: init -> auth (b0)
I (7569) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (7569) wifi:state: auth -> assoc (0)
I (7579) wifi:state: assoc -> run (10)
I (7629) wifi:state: run -> init (2c0)
I (7639) wifi:new:<6,0>, old:<6,0>, ap:<255,255>, sta:<6,0>, prof:1
W (7639) LAB_WIFI_CONN: =======================================================
W (7639) LAB_WIFI_CONN: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (7649) LAB_WIFI_CONN:   -> Target SSID          : PhuwishP
W (7649) LAB_WIFI_CONN:   -> Reason Code (Decimal): 2
W (7659) LAB_WIFI_CONN:   -> Reason Code (Hex)    : 0x02
W (7669) LAB_WIFI_CONN:   -> Reason Description   : WIFI_REASON_AUTH_EXPIRE (2)
W (7669) LAB_WIFI_CONN: =======================================================
W (7679) LAB_WIFI_CONN: [RESULT]: TEST FAILED - Disconnected event captured.
I (7689) LAB_WIFI_CONN: ==================================================================
I (7699) LAB_WIFI_CONN:   [Phase 2/3/4/5 Completed: Wi-Fi Connection Lab Finished]
I (7709) LAB_WIFI_CONN: ==================================================================
I (7719) main_task: Returned from app_main()
```

---

# รายงานการทดลอง ใบงานที่ 5.3: การยืนยันตัวตนและการผูกสัมพันธ์ในระดับ Link Layer (Authentication & Association Phase)

---

## 1. ผลการทดลอง (Experiment Results)

### 1.1 ตารางสรุปเปรียบเทียบผลการทดลองในระดับ Link Layer

| ข้อการทดลอง | สถานการณ์ทดสอบ                           |        Event ที่ได้รับ        |  ผลการผูกสัมพันธ์ Link Layer  | ค่า Association ID (AID) ที่ได้ | Reason Code (ถ้ามี)                     |
| :---------: | :--------------------------------------- | :---------------------------: | :---------------------------: | :-----------------------------: | :-------------------------------------- |
|  **5.3.1**  | ร้องขอ Auth & Assoc กับ AP มีอยู่จริง    |  `WIFI_EVENT_STA_CONNECTED`   | สำเร็จ (Link Layer Connected) |                3                | N/A                                     |
|  **5.3.2**  | ร้องขอ Auth & Assoc กับ AP ไม่มีอยู่จริง | `WIFI_EVENT_STA_DISCONNECTED` |   ล้มเหลว (SSID Not Found)    |               N/A               | 201 (`0xC9`, `WIFI_REASON_NO_AP_FOUND`) |

---

### 1.2 บันทึกข้อมูล Link Layer จาก Event `WIFI_EVENT_STA_CONNECTED` (ข้อ 5.3.1)

| พารามิเตอร์ Link Layer   | ค่าที่อ่านได้จริงจาก Forensic Log |
| :----------------------- | :-------------------------------- |
| **SSID**                 | `PhuwishP`                        |
| **BSSID (MAC Address)**  | `82:CA:F7:29:29:6C`               |
| **Channel**              | `1`                               |
| **Auth Mode Enum**       | `3` (`WIFI_AUTH_WPA2_PSK`)        |
| **Association ID (AID)** | `3`                               |

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

## 3. Console Log จากการทดลองจริง (Lab 5.3)

````text
I (392) boot: Loaded app from partition at offset 0x10000
I (392) boot: Disabling RNG early entropy source...
I (403) cpu_start: Multicore app
I (413) cpu_start: Pro cpu start user code
I (413) cpu_start: cpu freq: 160000000 Hz
I (413) cpu_start: Application information:
I (416) cpu_start: Project name:     wifi_auth_assoc_lab3
I (422) cpu_start: App version:      a12ebc6-dirty
I (427) cpu_start: Compile time:     Aug  3 2026 10:12:10
I (434) cpu_start: ELF file SHA256:  1bd51fbd7...
I (439) cpu_start: ESP-IDF:          v5.2.3
I (444) cpu_start: Min chip rev:     v0.0
I (448) cpu_start: Max chip rev:     v3.99
I (453) cpu_start: Chip rev:         v3.1
I (458) heap_init: Initializing. RAM available for dynamic allocation:
I (465) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (471) heap_init: At 3FFB7DB0 len 00028250 (160 KiB): DRAM
I (478) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (484) heap_init: At 40096B48 len 0001BCB0 (111 KiB): D/IRAM
I (490) heap_init: At 40096B48 len 000094B8 (37 KiB): IRAM
I (498) spi_flash: detected chip: generic
I (501) spi_flash: flash io: dio
W (505) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (519) main_task: Started on CPU0
I (529) main_task: Calling app_main()
I (529) LAB_AUTH_ASSOC: [FORENSIC]: Call nvs_flash_init()
I (549) LAB_AUTH_ASSOC: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (549) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_netif_init()
I (549) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_event_loop_create_default()
I (559) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (569) LAB_AUTH_ASSOC: [FORENSIC]: esp_netif_create_default_wifi_sta() returned 0x3ffbd7d8
I (579) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_init(&cfg)
I (589) wifi:wifi driver task: 3ffbfddc, prio:23, stack:6656, core=0
I (599) wifi:wifi firmware version: 82f54d2
I (599) wifi:wifi certification version: v7.0
I (599) wifi:config NVS flash: enabled
I (599) wifi:config nano formating: disabled
I (609) wifi:Init data frame dynamic rx buffer num: 32
I (609) wifi:Init static rx mgmt buffer num: 5
I (619) wifi:Init management short buffer num: 32
I (619) wifi:Init dynamic tx buffer num: 32
I (629) wifi:Init static rx buffer size: 1600
I (629) wifi:Init static rx buffer num: 10
I (629) wifi:Init dynamic rx buffer num: 32
I (639) wifi_init: rx ba win: 6
I (639) wifi_init: tcpip mbox: 32
I (649) wifi_init: udp mbox: 6
I (649) wifi_init: tcp mbox: 6
I (659) wifi_init: tcp tx win: 5760
I (659) wifi_init: tcp rx win: 5760
I (669) wifi_init: tcp mss: 1440
I (669) wifi_init: WiFi IRAM OP enabled
I (669) wifi_init: WiFi RX IRAM OP enabled
I (669) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (679) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (689) LAB_AUTH_ASSOC: ==================================================================
I (699) LAB_AUTH_ASSOC:   Lab 5.3: Wi-Fi Authentication & Association Phase (ESP-IDF Forensic)
I (709) LAB_AUTH_ASSOC: ==================================================================
I (719) LAB_AUTH_ASSOC:

I (719) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (729) LAB_AUTH_ASSOC: >>> Experiment 5.3.1: Link-Layer Auth & Assoc Phase Test
I (739) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (749) LAB_AUTH_ASSOC:   Target SSID    : "PhuwishP"
I (759) LAB_AUTH_ASSOC:   Target Password: "123456789"
I (759) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_stop()
I (769) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (779) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (819) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (819) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_start()
I (819) phy_init: phy_version 4840,a40f0ac,Aug 16 2024,16:48:06
I (909) wifi:mode : sta (84:1f:e8:20:55:24)
I (909) wifi:enable tsf
I (909) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (909) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (909) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (919) LAB_AUTH_ASSOC: [FORENSIC]: Initiating 802.11 Link-Layer Connection (Auth & Assoc)...
I (929) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect()
I (939) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (1069) wifi:new:<1,0>, old:<1,0>, ap:<255,255>, sta:<1,0>, prof:1
I (1069) wifi:state: init -> auth (b0)
I (1089) wifi:state: auth -> assoc (0)
I (1099) wifi:state: assoc -> run (10)
I (1219) wifi:connected with PhuwishP, aid = 2, channel 1, BW20, bssid = 82:ca:f7:29:29:6c
I (1219) wifi:security: WPA2-PSK, phy: bgn, rssi: -37
I (1239) wifi:pm start, type: 1

I (1239) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (1239) LAB_AUTH_ASSOC: =======================================================
I (1239) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_CONNECTED received!
I (1249) LAB_AUTH_ASSOC:   [SUCCESS]: Phase 2 (Auth) & Phase 3 (Assoc) COMPLETED!
I (1259) LAB_AUTH_ASSOC:   -> Connected SSID        : PhuwishP
I (1269) LAB_AUTH_ASSOC:   -> BSSID (MAC Address)   : 82:CA:F7:29:29:6C
I (1269) LAB_AUTH_ASSOC:   -> Channel               : 1
I (1279) LAB_AUTH_ASSOC:   -> Auth Mode             : 3
I (1279) LAB_AUTH_ASSOC:   -> Association ID (AID)  : 3
I (1289) LAB_AUTH_ASSOC: =======================================================
I (1299) LAB_AUTH_ASSOC: [RESULT]: TEST PASSED - Phase 2 (Auth) & Phase 3 (Assoc) Successful!
I (1319) wifi:AP's beacon interval = 102400 us, DTIM period = 1
I (2239) esp_netif_handlers: sta ip: 172.20.10.2, mask: 255.255.255.240, gw: 172.20.10.1
I (3309) LAB_AUTH_ASSOC:

I (3309) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (3309) LAB_AUTH_ASSOC: >>> Experiment 5.3.2: Link-Layer Test - Non-Existent AP
I (3309) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (3319) LAB_AUTH_ASSOC:   Target SSID    : "NON_EXISTENT_AP_9999"
I (3329) LAB_AUTH_ASSOC:   Target Password: "12345678"
I (3339) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_stop()
I (3339) wifi:state: run -> init (0)
I (3359) wifi:pm stop, total sleep time: 1508952 us / 2121221 us

I (3359) wifi:new:<1,0>, old:<1,0>, ap:<255,255>, sta:<1,0>, prof:1
E (3359) wifi:NAN WiFi stop
W (3359) LAB_AUTH_ASSOC: =======================================================
W (3369) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (3379) LAB_AUTH_ASSOC:   -> Target SSID          : PhuwishP
W (3379) LAB_AUTH_ASSOC:   -> Reason Code (Decimal): 8
W (3389) LAB_AUTH_ASSOC:   -> Reason Code (Hex)    : 0x08
W (3389) LAB_AUTH_ASSOC:   -> Reason Diagnosis     : OTHER_DISCONNECT_REASON
W (3399) LAB_AUTH_ASSOC: =======================================================
I (3409) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (3419) wifi:flush txq
I (3419) wifi:stop sw txq
I (3419) wifi:lmac stop hw txq
I (3429) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (3429) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (3489) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (3489) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_start()
I (3489) wifi:mode : sta (84:1f:e8:20:55:24)
I (3489) wifi:enable tsf
I (3499) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (3499) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (3509) LAB_AUTH_ASSOC: [FORENSIC]: Initiating 802.11 Link-Layer Connection (Auth & Assoc)...
I (3519) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect()
I (3529) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
W (3529) LAB_AUTH_ASSOC: [RESULT]: TEST FAILED - Disconnected event captured in Link-Layer.
I (3539) LAB_AUTH_ASSOC: ==================================================================
I (3549) LAB_AUTH_ASSOC:   [Phase 2 & Phase 3 Completed: Link-Layer Auth & Assoc Finished]
I (3559) LAB_AUTH_ASSOC: ==================================================================
I (3569) main_task: Returned from app_main()
W (5939) LAB_AUTH_ASSOC: =======================================================
W (5939) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (5949) LAB_AUTH_ASSOC:   -> Target SSID          : NON_EXISTENT_AP_9999
W (5949) LAB_AUTH_ASSOC:   -> Reason Code (Decimal): 201
W (5959) LAB_AUTH_ASSOC:   -> Reason Code (Hex)    : 0xC9
W (5969) LAB_AUTH_ASSOC:   -> Reason Diagnosis     : WIFI_REASON_NO_AP_FOUND (201) [Phase 1: SSID Not Found]
W (5979) LAB_AUTH_ASSOC: =======================================================
```eason Code:
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

## 3. Console Log จากการทดลองจริง (Lab 5.3)

```text
I (392) boot: Loaded app from partition at offset 0x10000
I (392) boot: Disabling RNG early entropy source...
I (403) cpu_start: Multicore app
I (413) cpu_start: Pro cpu start user code
I (413) cpu_start: cpu freq: 160000000 Hz
I (413) cpu_start: Application information:
I (416) cpu_start: Project name:     wifi_auth_assoc_lab3
I (422) cpu_start: App version:      37de524-dirty
I (427) cpu_start: Compile time:     Aug  3 2026 10:08:12
I (434) cpu_start: ELF file SHA256:  a26ca9393...
I (439) cpu_start: ESP-IDF:          v5.2.3
I (444) cpu_start: Min chip rev:     v0.0
I (448) cpu_start: Max chip rev:     v3.99
I (453) cpu_start: Chip rev:         v3.1
I (458) heap_init: Initializing. RAM available for dynamic allocation:
I (465) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (471) heap_init: At 3FFB7DB0 len 00028250 (160 KiB): DRAM
I (478) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (484) heap_init: At 40096B48 len 0001BCB0 (111 KiB): D/IRAM
I (490) heap_init: At 40096B48 len 000094B8 (37 KiB): IRAM
I (498) spi_flash: detected chip: generic
I (501) spi_flash: flash io: dio
W (505) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (519) main_task: Started on CPU0
I (529) main_task: Calling app_main()
I (529) LAB_AUTH_ASSOC: [FORENSIC]: Call nvs_flash_init()
I (549) LAB_AUTH_ASSOC: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (549) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_netif_init()
I (549) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_event_loop_create_default()
I (559) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (569) LAB_AUTH_ASSOC: [FORENSIC]: esp_netif_create_default_wifi_sta() returned 0x3ffbd7d8
I (579) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_init(&cfg)
I (589) wifi:wifi driver task: 3ffbfddc, prio:23, stack:6656, core=0
I (599) wifi:wifi firmware version: 82f54d2
I (599) wifi:wifi certification version: v7.0
I (599) wifi:config NVS flash: enabled
I (599) wifi:config nano formating: disabled
I (609) wifi:Init data frame dynamic rx buffer num: 32
I (609) wifi:Init static rx mgmt buffer num: 5
I (619) wifi:Init management short buffer num: 32
I (619) wifi:Init dynamic tx buffer num: 32
I (629) wifi:Init static rx buffer size: 1600
I (629) wifi:Init static rx buffer num: 10
I (629) wifi:Init dynamic rx buffer num: 32
I (639) wifi_init: rx ba win: 6
I (639) wifi_init: tcpip mbox: 32
I (649) wifi_init: udp mbox: 6
I (649) wifi_init: tcp mbox: 6
I (659) wifi_init: tcp tx win: 5760
I (659) wifi_init: tcp rx win: 5760
I (669) wifi_init: tcp mss: 1440
I (669) wifi_init: WiFi IRAM OP enabled
I (669) wifi_init: WiFi RX IRAM OP enabled
I (669) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (679) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (689) LAB_AUTH_ASSOC: ==================================================================
I (699) LAB_AUTH_ASSOC:   Lab 5.3: Wi-Fi Authentication & Association Phase (ESP-IDF Forensic)
I (709) LAB_AUTH_ASSOC: ==================================================================
I (719) LAB_AUTH_ASSOC:

I (719) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (729) LAB_AUTH_ASSOC: >>> Experiment 5.3.1: Link-Layer Auth & Assoc Phase Test
I (739) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (749) LAB_AUTH_ASSOC:   Target SSID    : "PhuwishP"
I (749) LAB_AUTH_ASSOC:   Target Password: "123456789"
I (759) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_stop()
I (769) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (769) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (809) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (809) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_start()
I (809) phy_init: phy_version 4840,a40f0ac,Aug 16 2024,16:48:06
I (889) wifi:mode : sta (84:1f:e8:20:55:24)
I (899) wifi:enable tsf
I (899) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (899) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (899) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (909) LAB_AUTH_ASSOC: [FORENSIC]: Initiating 802.11 Link-Layer Connection (Auth & Assoc)...
I (919) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect()
I (929) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (939) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1
I (939) wifi:state: init -> auth (b0)
I (949) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (969) wifi:state: auth -> assoc (0)
I (979) wifi:state: assoc -> run (10)
E (8939) LAB_AUTH_ASSOC: [RESULT]: TEST TIMEOUT - Response timeout from AP.
I (10939) LAB_AUTH_ASSOC:

I (10939) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (10939) LAB_AUTH_ASSOC: >>> Experiment 5.3.2: Link-Layer Test - Non-Existent AP
I (10939) LAB_AUTH_ASSOC: ------------------------------------------------------------------
I (10949) LAB_AUTH_ASSOC:   Target SSID    : "NON_EXISTENT_AP_9999"
I (10959) LAB_AUTH_ASSOC:   Target Password: "12345678"
I (10969) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_stop()
I (10969) wifi:state: run -> init (0)
I (10989) wifi:new:<6,0>, old:<6,0>, ap:<255,255>, sta:<6,0>, prof:1
E (10989) wifi:NAN WiFi stop
I (10989) wifi:ieee80211_handshake mode switch cause sta destroyed
W (10989) LAB_AUTH_ASSOC: =======================================================
W (10999) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (11009) LAB_AUTH_ASSOC:   -> Target SSID          : PhuwishP
W (11009) LAB_AUTH_ASSOC:   -> Reason Code (Decimal): 8
W (11019) LAB_AUTH_ASSOC:   -> Reason Code (Hex)    : 0x08
W (11029) LAB_AUTH_ASSOC:   -> Reason Diagnosis     : OTHER_DISCONNECT_REASON
W (11029) LAB_AUTH_ASSOC: =======================================================
I (11039) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (11049) wifi:flush txq
I (11049) wifi:stop sw txq
I (11049) wifi:lmac stop hw txq
I (11059) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
W (11069) wifi:Password length matches WPA2 standards, authmode threshold changes from OPEN to WPA2
I (11109) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (11109) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_start()
I (11119) wifi:mode : sta (84:1f:e8:20:55:24)
I (11119) wifi:enable tsf
I (11119) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (11129) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (11129) LAB_AUTH_ASSOC: [FORENSIC]: Initiating 802.11 Link-Layer Connection (Auth & Assoc)...
I (11139) LAB_AUTH_ASSOC: [FORENSIC]: Call esp_wifi_connect()
I (11149) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (11129) LAB_AUTH_ASSOC: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
W (11169) LAB_AUTH_ASSOC: [RESULT]: TEST FAILED - Disconnected event captured in Link-Layer.
I (11169) LAB_AUTH_ASSOC: ==================================================================
I (11179) LAB_AUTH_ASSOC:   [Phase 2 & Phase 3 Completed: Link-Layer Auth & Assoc Finished]
I (11189) LAB_AUTH_ASSOC: ==================================================================
I (11199) main_task: Returned from app_main()
W (13559) LAB_AUTH_ASSOC: =======================================================
W (13559) LAB_AUTH_ASSOC: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (13569) LAB_AUTH_ASSOC:   -> Target SSID          : NON_EXISTENT_AP_9999
W (13579) LAB_AUTH_ASSOC:   -> Reason Code (Decimal): 201
W (13579) LAB_AUTH_ASSOC:   -> Reason Code (Hex)    : 0xC9
W (13589) LAB_AUTH_ASSOC:   -> Reason Diagnosis     : WIFI_REASON_NO_AP_FOUND (201) [Phase 1: SSID Not Found]
W (13599) LAB_AUTH_ASSOC: =======================================================
```

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

---

## 3. Console Log จากการทดลองจริง (Lab 5.4)

```text
I (392) boot: Loaded app from partition at offset 0x10000
I (392) boot: Disabling RNG early entropy source...
I (403) cpu_start: Multicore app
I (413) cpu_start: Pro cpu start user code
I (413) cpu_start: cpu freq: 160000000 Hz
I (413) cpu_start: Application information:
I (416) cpu_start: Project name:     wifi_handshake_ip_lab4
I (422) cpu_start: App version:      a12ebc6-dirty
I (428) cpu_start: Compile time:     Aug  3 2026 10:41:50
I (434) cpu_start: ELF file SHA256:  bd5c18467...
I (439) cpu_start: ESP-IDF:          v5.2.3
I (444) cpu_start: Min chip rev:     v0.0
I (449) cpu_start: Max chip rev:     v3.99
I (454) cpu_start: Chip rev:         v3.1
I (459) heap_init: Initializing. RAM available for dynamic allocation:
I (466) heap_init: At 3FFAE6E0 len 00001920 (6 KiB): DRAM
I (472) heap_init: At 3FFB7DB0 len 00028250 (160 KiB): DRAM
I (478) heap_init: At 3FFE0440 len 00003AE0 (14 KiB): D/IRAM
I (484) heap_init: At 3FFE4350 len 0001BCB0 (111 KiB): D/IRAM
I (491) heap_init: At 40096B48 len 000094B8 (37 KiB): IRAM
I (498) spi_flash: detected chip: generic
I (502) spi_flash: flash io: dio
W (505) spi_flash: Detected size(4096k) larger than the size in the binary image header(2048k). Using the size in the binary image header.
I (520) main_task: Started on CPU0
I (530) main_task: Calling app_main()
I (530) LAB_HANDSHAKE_IP: [FORENSIC]: Call nvs_flash_init()
I (550) LAB_HANDSHAKE_IP: [FORENSIC]: nvs_flash_init() returned ESP_OK (0x0)
I (550) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_netif_init()
I (560) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_event_loop_create_default()
I (560) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_netif_create_default_wifi_sta()
I (570) LAB_HANDSHAKE_IP: [FORENSIC]: esp_netif_create_default_wifi_sta() returned 0x3ffbd7d8
I (580) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_init(&cfg)
I (600) wifi:wifi driver task: 3ffbfddc, prio:23, stack:6656, core=0
I (610) wifi:wifi firmware version: 82f54d2
I (610) wifi:wifi certification version: v7.0
I (610) wifi:config NVS flash: enabled
I (610) wifi:config nano formating: disabled
I (610) wifi:Init data frame dynamic rx buffer num: 32
I (620) wifi:Init static rx mgmt buffer num: 5
I (620) wifi:Init management short buffer num: 32
I (630) wifi:Init dynamic tx buffer num: 32
I (630) wifi:Init static rx buffer size: 1600
I (640) wifi:Init static rx buffer num: 10
I (640) wifi:Init dynamic rx buffer num: 32
I (640) wifi_init: rx ba win: 6
I (650) wifi_init: tcpip mbox: 32
I (650) wifi_init: udp mbox: 6
I (660) wifi_init: tcp mbox: 6
I (660) wifi_init: tcp tx win: 5760
I (660) wifi_init: tcp rx win: 5760
I (670) wifi_init: tcp mss: 1440
I (670) wifi_init: WiFi IRAM OP enabled
I (680) wifi_init: WiFi RX IRAM OP enabled
I (680) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_event_handler_instance_register(WIFI_EVENT)
I (690) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_event_handler_instance_register(IP_EVENT)
I (700) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_set_mode(WIFI_MODE_STA)
I (710) LAB_HANDSHAKE_IP: ==================================================================
I (710) LAB_HANDSHAKE_IP:   Lab 5.4: 4-Way Handshake & IP Assignment Phase (ESP-IDF Forensic)
I (720) LAB_HANDSHAKE_IP: ==================================================================
I (730) LAB_HANDSHAKE_IP:

I (740) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (750) LAB_HANDSHAKE_IP: >>> Experiment 5.4.1: Handshake & IP Test - Correct Password
I (750) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (760) LAB_HANDSHAKE_IP:   Target SSID    : "PhuwishP"
I (770) LAB_HANDSHAKE_IP:   Target Password: "123456789"
I (780) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_stop()
I (780) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
I (810) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (810) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_start()
I (820) phy_init: phy_version 4840,a40f0ac,Aug 16 2024,16:48:06
I (900) wifi:mode : sta (84:1f:e8:20:55:24)
I (900) wifi:enable tsf
I (900) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (900) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (900) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (910) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_connect()
I (920) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
I (1540) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1
I (1540) wifi:state: init -> auth (b0)
I (1540) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (1550) wifi:state: auth -> assoc (0)
I (1560) wifi:state: assoc -> run (10)
I (1700) wifi:connected with PhuwishP, aid = 2, channel 6, BW20, bssid = 6a:57:07:6f:73:08
I (1700) wifi:security: WPA2-PSK, phy: bgn, rssi: -42
I (1720) wifi:pm start, type: 1

I (1720) wifi:dp: 1, bi: 102400, li: 3, scale listen interval from 307200 us to 307200 us
I (1730) LAB_HANDSHAKE_IP: =======================================================
I (1730) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_CONNECTED received!
I (1740) LAB_HANDSHAKE_IP:   -> Phase 2 (Auth) & Phase 3 (Assoc) PASSED
I (1750) LAB_HANDSHAKE_IP:   -> Connected SSID  : PhuwishP
I (1750) LAB_HANDSHAKE_IP:   -> BSSID           : 6A:57:07:6F:73:08
I (1760) LAB_HANDSHAKE_IP:   -> Channel         : 6
I (1770) LAB_HANDSHAKE_IP:   -> Association ID  : 3
I (1770) LAB_HANDSHAKE_IP: [FORENSIC]: Entering Phase 4: 4-Way EAPOL Key Exchange...
I (1780) LAB_HANDSHAKE_IP: =======================================================
I (1780) wifi:AP's beacon interval = 102400 us, DTIM period = 1
I (2730) esp_netif_handlers: sta ip: 172.20.10.2, mask: 255.255.255.240, gw: 172.20.10.1
I (2730) LAB_HANDSHAKE_IP: =======================================================
I (2730) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: IP_EVENT_STA_GOT_IP received!
I (2740) LAB_HANDSHAKE_IP:   [SUCCESS]: Phase 4 (4-Way Handshake) & Phase 5 (DHCP IP) COMPLETED!
I (2750) LAB_HANDSHAKE_IP:   -> Allocated IP Address : 172.20.10.2
I (2760) LAB_HANDSHAKE_IP:   -> Subnet Mask          : 255.255.255.240
I (2770) LAB_HANDSHAKE_IP:   -> Default Gateway      : 172.20.10.1
I (2770) LAB_HANDSHAKE_IP: =======================================================
I (2780) LAB_HANDSHAKE_IP: [RESULT]: TEST PASSED - 4-Way Handshake & DHCP IP Assignment Successful!
I (4790) LAB_HANDSHAKE_IP:

I (4790) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (4790) LAB_HANDSHAKE_IP: >>> Experiment 5.4.2: Handshake Test - Incorrect Password
I (4790) LAB_HANDSHAKE_IP: ------------------------------------------------------------------
I (4800) LAB_HANDSHAKE_IP:   Target SSID    : "PhuwishP"
I (4810) LAB_HANDSHAKE_IP:   Target Password: "WRONG_PASSWORD_1234"
I (4820) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_stop()
I (4830) wifi:state: run -> init (0)
I (4840) wifi:pm stop, total sleep time: 1935637 us / 3118063 us

I (4840) wifi:new:<6,0>, old:<6,0>, ap:<255,255>, sta:<6,0>, prof:1
E (4840) wifi:NAN WiFi stop
W (4840) LAB_HANDSHAKE_IP: =======================================================
W (4850) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (4860) LAB_HANDSHAKE_IP:   -> Target SSID          : PhuwishP
W (4860) LAB_HANDSHAKE_IP:   -> Reason Code (Decimal): 8
W (4870) LAB_HANDSHAKE_IP:   -> Reason Code (Hex)    : 0x08
W (4880) LAB_HANDSHAKE_IP:   -> Reason Diagnosis     : OTHER_DISCONNECT_REASON
W (4890) LAB_HANDSHAKE_IP: =======================================================
I (4890) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 3 received
I (4900) wifi:flush txq
I (4900) wifi:stop sw txq
I (4900) wifi:lmac stop hw txq
I (4910) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_set_config(WIFI_IF_STA, &wifi_config)
I (4940) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_set_config() returned ESP_OK (0x0)
I (4940) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_start()
I (4950) wifi:mode : sta (84:1f:e8:20:55:24)
I (4950) wifi:enable tsf
I (4960) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (4960) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_start() returned ESP_OK (0x0)
I (4960) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_START received
I (4980) LAB_HANDSHAKE_IP: [FORENSIC]: Call esp_wifi_connect()
I (4990) LAB_HANDSHAKE_IP: [FORENSIC]: esp_wifi_connect() returned ESP_OK (0x0)
W (5250) LAB_HANDSHAKE_IP: [RESULT]: TEST FAILED - Disconnected during Handshake or Auth.
I (5250) LAB_HANDSHAKE_IP: ==================================================================
I (5260) wifi:new:<6,0>, old:<1,0>, ap:<255,255>, sta:<6,0>, prof:1
I (5260) wifi:state: init -> auth (b0)
I (5270) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT ID 43 received
I (5270) LAB_HANDSHAKE_IP:   [Phase 4 & Phase 5 Completed: Wi-Fi Handshake & IP Lab Finished]
I (5280) LAB_HANDSHAKE_IP: ==================================================================
I (5280) wifi:state: auth -> assoc (0)
I (5300) main_task: Returned from app_main()
I (5320) wifi:state: assoc -> run (10)
I (5810) wifi:state: run -> init (2c0)
I (5820) wifi:new:<6,0>, old:<6,0>, ap:<255,255>, sta:<6,0>, prof:1
W (5820) LAB_HANDSHAKE_IP: =======================================================
W (5820) LAB_HANDSHAKE_IP: [EVENT FORENSIC]: WIFI_EVENT_STA_DISCONNECTED received!
W (5830) LAB_HANDSHAKE_IP:   -> Target SSID          : PhuwishP
W (5840) LAB_HANDSHAKE_IP:   -> Reason Code (Decimal): 2
W (5840) LAB_HANDSHAKE_IP:   -> Reason Code (Hex)    : 0x02
W (5850) LAB_HANDSHAKE_IP:   -> Reason Diagnosis     : WIFI_REASON_AUTH_EXPIRE (2)
W (5860) LAB_HANDSHAKE_IP: =======================================================
```


````
