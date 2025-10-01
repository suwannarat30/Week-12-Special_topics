# บทที่ 2: การติดตั้งและตั้งค่า ESP-IDF
## เตรียมพร้อมสำหรับการพัฒนา ESP-NOW

![ESP-IDF Logo](https://docs.espressif.com/projects/esp-idf/en/latest/_static/esp-idf-logo.svg)

ในบทนี้เราจะเรียนรู้การติดตั้งและตั้งค่า ESP-IDF (Espressif IoT Development Framework) เพื่อเตรียมพร้อมสำหรับการพัฒนา ESP-NOW

### เป้าหมายการเรียนรู้:
- ติดตั้ง ESP-IDF บนระบบ Windows และ macOS
- ตั้งค่า Command Line Environment
- เข้าใจโครงสร้างโปรเจค ESP-IDF
- ทดสอบการเชื่อมต่อกับ ESP32

---

## การติดตั้ง ESP-IDF

### วิธีที่ 1: ใช้ ESP-IDF Installer (แนะนำสำหรับผู้เริ่มต้น)

1. **ดาวน์โหลด ESP-IDF Installer**
   - ไปที่: https://dl.espressif.com/dl/esp-idf/
   - เลือก Windows Installer (.exe)
   - เวอร์ชันที่แนะนำ: ESP-IDF v5.1+

2. **รันไฟล์ Installer**
   - คลิกขวาแล้วเลือก "Run as administrator"
   - เลือก Installation path (แนะนำ: `C:\Espressif`)
   - เลือก Components ที่ต้องการ:
     - ✅ ESP-IDF
     - ✅ ESP-IDF Tools
     - ✅ Python
     - ✅ Git

### วิธีที่ 2: ติดตั้งด้วย Command Line (สำหรับผู้ใช้งานขั้นสูง)

```bash
# Clone ESP-IDF repository
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
git checkout v5.1

# Install tools
./install.bat    # Windows
./install.sh     # macOS/Linux

# Setup environment
./export.bat     # Windows  
./export.sh      # macOS/Linux
```

---

## การตั้งค่า Command Line Environment

### 1. เปิด ESP-IDF Command Prompt

**สำหรับ Windows:**
- เปิด "ESP-IDF Command Prompt" จาก Start Menu
- หรือเปิด Command Prompt แล้วรันคำสั่ง:
  ```cmd
  %USERPROFILE%\esp\esp-idf\export.bat
  ```

**สำหรับ macOS/Linux:**
- เปิด Terminal
- รันคำสั่ง:
  ```bash
  . $HOME/esp/esp-idf/export.sh
  ```

### 2. ทดสอบการติดตั้ง
รันคำสั่งต่อไปนี้เพื่อตรวจสอบ:

```bash
idf.py --version
python --version
git --version
```

**ผลลัพธ์ที่คาดหวัง:**
```
ESP-IDF v5.1.x
Python 3.8.x
git version 2.x.x
```

### 3. ตั้งค่า Serial Port (Optional)

**Windows:** ตรวจสอบ Device Manager → Ports (COM & LPT)
**macOS:** รันคำสั่ง `ls /dev/cu.*`
**Linux:** รันคำสั่ง `ls /dev/ttyUSB*`

---

## โครงสร้างโปรเจค ESP-IDF

### การสร้างโปรเจคใหม่

```bash
# สร้างโปรเจคจาก template
idf.py create-project my_esp_now_project

# หรือ copy จาก example
cp -r $IDF_PATH/examples/get-started/hello_world my_project
```

### โครงสร้างโปรเจค

```
my_esp_now_project/
├── main/
│   ├── main.c              # ไฟล์โค้ดหลัก
│   └── CMakeLists.txt      # Build configuration
├── CMakeLists.txt          # Project configuration
├── sdkconfig               # ESP-IDF configuration
└── build/                  # Build output
```

### การกำหนดค่าสำคัญ

**main/CMakeLists.txt สำหรับ ESP-NOW:**
```cmake
idf_component_register(
    SRCS "main.c"
    INCLUDE_DIRS "."
    REQUIRES esp_wifi esp_now nvs_flash
)
```

---

## การ Build และ Flash โปรเจค

### ขั้นตอนพื้นฐาน

1. **เชื่อมต่อ ESP32**
   - เสียบสาย USB ระหว่าง ESP32 และคอมพิวเตอร์
   - ตรวจสอบ Port ที่เชื่อมต่อ

2. **ตั้งค่าเป้าหมาย (Target)**
   ```bash
   idf.py set-target esp32
   ```

3. **Build โปรเจค**
   ```bash
   idf.py build
   ```

4. **Flash โปรแกรม**
   ```bash
   idf.py -p COM3 flash       # Windows
   idf.py -p /dev/ttyUSB0 flash  # Linux
   ```

5. **Monitor Serial Output**
   ```bash
   idf.py monitor
   ```

### คำสั่งรวม (All-in-One)

```bash
# Build, Flash และ Monitor ในคำสั่งเดียว
idf.py build flash monitor
```

---

## การใช้งาน idf.py Commands

### คำสั่งพื้นฐาน

| คำสั่ง | หน้าที่ |
|--------|---------|
| `idf.py create-project <name>` | สร้างโปรเจคใหม่ |
| `idf.py build` | Build โปรเจค |
| `idf.py flash` | Flash โปรแกรมลง ESP32 |
| `idf.py monitor` | แสดง Serial output |
| `idf.py clean` | ลบไฟล์ build |
| `idf.py fullclean` | ลบไฟล์ build ทั้งหมด |
| `idf.py menuconfig` | เปิด Configuration menu |

### คำสั่งขั้นสูง

```bash
# ตรวจสอบข้อมูล ESP32
idf.py chip_id

# Erase flash ทั้งหมด
idf.py erase_flash

# ดู partition table
idf.py partition_table

# Monitor พร้อม filter
idf.py monitor --print_filter="ESP_NOW*"
```

---

## การตั้งค่า Configuration (menuconfig)

### เปิด Configuration Menu

```bash
idf.py menuconfig
```

### การตั้งค่าสำคัญสำหรับ ESP-NOW

1. **WiFi Configuration:**
   - Component config → ESP32-specific → WiFi
   - WiFi Task Core ID: Core 0 (แนะนำ)
   - WiFi RX Task Core ID: Core 0

2. **Log Configuration:**
   - Component config → Log output
   - Default log verbosity: Info (หรือ Debug สำหรับ development)

3. **FreeRTOS Configuration:**
   - Component config → FreeRTOS
   - Tick rate (Hz): 1000 (แนะนำ)

---

## การแก้ไขปัญหาเบื้องต้น

### ปัญหาที่พบบ่อย

#### 1. "idf.py: command not found"
**วิธีแก้:**
```bash
# รัน export script
%USERPROFILE%\esp\esp-idf\export.bat  # Windows
. $HOME/esp/esp-idf/export.sh        # macOS/Linux
```

#### 2. "Serial port not found"
**วิธีแก้:**
- ตรวจสอบสาย USB
- ติดตั้ง Driver สำหรับ ESP32
- ตรวจสอบ Device Manager (Windows)

#### 3. "Permission denied" (Linux/macOS)
**วิธีแก้:**
```bash
sudo usermod -a -G dialout $USER  # Linux
sudo chmod 666 /dev/ttyUSB0       # Temporary fix
```

#### 4. Build errors
**วิธีแก้:**
```bash
# Clean และ build ใหม่
idf.py fullclean
idf.py build
```

---

## การเตรียม ESP-NOW Components

### ตัวอย่างโค้ดพื้นฐาน

```c
#include <stdio.h>
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"
#include "esp_now.h"

static const char* TAG = "ESP_NOW_SETUP";

void wifi_init(void)
{
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_start());
    
    ESP_LOGI(TAG, "WiFi initialized for ESP-NOW");
}

void app_main(void)
{
    ESP_ERROR_CHECK(nvs_flash_init());
    wifi_init();
    
    ESP_LOGI(TAG, "ESP-NOW Setup Complete!");
    
    while(1) {
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

---

## เครื่องมือและ Extensions ที่มีประโยชน์

### VS Code Extensions

1. **ESP-IDF Extension**
   - ติดตั้งจาก VS Code Marketplace
   - รองรับ IntelliSense และ Debugging
   - Build และ Flash ผ่าน GUI

2. **C/C++ Extension**
   - Microsoft C/C++ extension
   - รองรับ Code completion

### Serial Terminal Tools

1. **PuTTY** (Windows)
2. **CoolTerm** (Cross-platform)
3. **Arduino IDE Serial Monitor**
4. **VS Code Terminal**

---

## สรุปและขั้นตอนต่อไป

### ✅ Checklist การเตรียมความพร้อม:
- [ ] ESP-IDF ติดตั้งเรียบร้อย
- [ ] `idf.py --version` แสดงเวอร์ชั่นถูกต้อง
- [ ] `idf.py build` ทำงานได้สำเร็จ
- [ ] `idf.py flash monitor` ทำงานได้
- [ ] ESP32 อย่างน้อย 2 ตัว พร้อมใช้งาน

### 🔧 คำสั่งสำคัญที่ควรจำ:

**สำหรับ Windows:**
```cmd
# เปิด ESP-IDF environment
%USERPROFILE%\esp\esp-idf\export.bat

# สร้างโปรเจคและ build
idf.py create-project my_project
cd my_project
idf.py build flash monitor
```

**สำหรับ macOS/Linux:**
```bash
# เปิด ESP-IDF environment
. $HOME/esp/esp-idf/export.sh

# สร้างโปรเจคและ build
idf.py create-project my_project
cd my_project
idf.py build flash monitor
```

### 📖 ขั้นตอนต่อไป:
ในบทต่อไป เราจะเรียนรู้การสื่อสารแบบ Point-to-Point ด้วย ESP-NOW

**➡️ ไปที่: [03-Point-to-Point](../03-Point-to-Point/03-Point-to-Point-Communication.md)**

---

## 📋 ใบงานปฏิบัติสำหรับบทนี้

1. **[Worksheet 2.1: การติดตั้ง ESP-IDF](Worksheets/Worksheet-2.1-Installation.md)** - การติดตั้งแบบ step-by-step
2. **[Worksheet 2.2: การตั้งค่าและ Configuration](Worksheets/Worksheet-2.2-Configuration.md)** - การใช้ menuconfig และการตั้งค่า
3. **[Worksheet 2.3: การสร้างโปรเจคแรก](Worksheets/Worksheet-2.3-First-Project.md)** - การสร้างและทดสอบโปรเจค ESP-NOW แรก

---
*หมายเหตุ: หากมีปัญหาในการติดตั้ง สามารถดู documentation ได้ที่ https://docs.espressif.com/projects/esp-idf/*