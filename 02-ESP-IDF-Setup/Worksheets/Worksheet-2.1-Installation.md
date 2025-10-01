# Worksheet 2.1: การติดตั้ง ESP-IDF
## การติดตั้งและตั้งค่า ESP-IDF แบบ Step-by-Step

### 🎯 วัตถุประสงค์
- ติดตั้ง ESP-IDF อย่างถูกต้องและสมบูรณ์
- ตั้งค่า Development Environment
- ทดสอบการทำงานของระบบ
- แก้ไขปัญหาที่พบบ่อย

### 📋 ความต้องการของระบบ
- **OS:** Windows 10/11, macOS 10.14+, หรือ Ubuntu 18.04+
- **RAM:** อย่างน้อย 4GB (แนะนำ 8GB+)
- **Storage:** อย่างน้อย 2GB ว่าง
- **Internet:** สำหรับดาวน์โหลด Components

---

## ส่วนที่ 1: การเตรียมความพร้อม

### 1.1 ตรวจสอบระบบ

**สำหรับ Windows:**
- [ ] Windows 10/11 (64-bit)
- [ ] สิทธิ์ Administrator
- [ ] Windows PowerShell หรือ Command Prompt

**สำหรับ macOS:**
- [ ] macOS 10.14 หรือใหม่กว่า
- [ ] Xcode Command Line Tools
- [ ] Homebrew (แนะนำ)

**สำหรับ Ubuntu/Linux:**
- [ ] Ubuntu 18.04+ หรือ Linux distro อื่น
- [ ] sudo access
- [ ] Git, Python 3.8+

### 1.2 ตรวจสอบ Python

รันคำสั่งต่อไปนี้:
```bash
python3 --version
pip3 --version
```

**ผลลัพธ์ที่ต้องการ:**
```
Python 3.8.x หรือใหม่กว่า
pip 20.x.x หรือใหม่กว่า
```

**หาก Python ไม่มี:** ให้ติดตั้งจาก https://python.org

---

## ส่วนที่ 2: การติดตั้ง ESP-IDF (Method 1: Installer)

### 2.1 ดาวน์โหลด ESP-IDF Installer

1. **ไปที่:** https://dl.espressif.com/dl/esp-idf/
2. **เลือกเวอร์ชัน:** ESP-IDF v5.1.x (Online Installer)
3. **ดาวน์โหลด:** ไฟล์ .exe (Windows) หรือ .dmg (macOS)

### 2.2 รัน Installer (Windows)

1. **คลิกขวาที่ไฟล์ .exe** → "Run as administrator"
2. **เลือก Installation Options:**
   ```
   ✅ ESP-IDF v5.1.x
   ✅ ESP-IDF Tools  
   ✅ Python 3.x
   ✅ Git for Windows
   ✅ VS Code Extension (Optional)
   ```

3. **เลือก Installation Path:**
   ```
   แนะนำ: C:\Espressif
   หลีกเลี่ยง: เส้นทางที่มีช่องว่างหรือตัวอักษรพิเศษ
   ```

4. **รอการติดตั้ง:** ประมาณ 10-20 นาที (ขึ้นอยู่กับ Internet speed)

### 2.3 การติดตั้งใน macOS

```bash
# ติดตั้ง Homebrew (ถ้ายังไม่มี)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# ติดตั้ง dependencies
brew install cmake ninja dfu-util

# Clone ESP-IDF
mkdir -p ~/esp
cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
git checkout v5.1.2

# Run install script
./install.sh esp32
```

### 2.4 การติดตั้งใน Ubuntu/Linux

```bash
# ติดตั้ง dependencies
sudo apt-get update
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0

# Clone ESP-IDF
mkdir -p ~/esp
cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
git checkout v5.1.2

# Run install script
./install.sh esp32
```

---

## ส่วนที่ 3: การตั้งค่า Environment

### 3.1 ตั้งค่า ESP-IDF Environment (Windows)

**วิธีที่ 1: ใช้ ESP-IDF Command Prompt**
- เปิด Start Menu
- ค้นหา "ESP-IDF Command Prompt"
- เปิดโปรแกรม

**วิธีที่ 2: ตั้งค่าใน PowerShell/CMD**
```cmd
# รันคำสั่งนี้ทุกครั้งที่เปิด terminal ใหม่
%USERPROFILE%\esp\esp-idf\export.bat

# หรือเพิ่มใน PATH permanently
```

### 3.2 ตั้งค่าใน macOS/Linux

```bash
# เพิ่มในไฟล์ ~/.bashrc หรือ ~/.zshrc
echo 'alias get_idf=". $HOME/esp/esp-idf/export.sh"' >> ~/.bashrc

# Reload shell
source ~/.bashrc

# ใช้งาน
get_idf
```

### 3.3 ทดสอบการติดตั้ง

รันคำสั่งต่อไปนี้:
```bash
idf.py --version
python --version
git --version
```

**ผลลัพธ์ที่คาดหวัง:**
```
ESP-IDF v5.1.x-xxx-gxxxxxxx
Python 3.8.x
git version 2.x.x
```

---

## ส่วนที่ 4: การสร้างโปรเจคทดสอบ

### 4.1 สร้างโปรเจคแรก

```bash
# สร้างโปรเจคใหม่
idf.py create-project hello_esp_idf

# เข้าไปในโฟลเดอร์
cd hello_esp_idf

# ตรวจสอบโครงสร้าง
ls -la
```

**โครงสร้างที่ควรเห็น:**
```
hello_esp_idf/
├── CMakeLists.txt
├── main/
│   ├── CMakeLists.txt
│   └── hello_esp_idf.c
└── README.md
```

### 4.2 แก้ไขโค้ดทดสอบ

แก้ไขไฟล์ `main/hello_esp_idf.c`:

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_log.h"

static const char* TAG = "ESP_IDF_TEST";

void app_main(void)
{
    ESP_LOGI(TAG, "🚀 ESP-IDF Installation Test Started!");
    ESP_LOGI(TAG, "📋 ESP-IDF Version: %s", esp_get_idf_version());
    ESP_LOGI(TAG, "💾 Free heap: %lu bytes", esp_get_free_heap_size());
    ESP_LOGI(TAG, "⚡ CPU frequency: %lu MHz", esp_clk_cpu_freq() / 1000000);
    
    int counter = 0;
    while (1) {
        ESP_LOGI(TAG, "⏰ System running: %d seconds", counter++);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

### 4.3 Build โปรเจค

```bash
# ตั้งค่า target (ESP32)
idf.py set-target esp32

# Build โปรเจค
idf.py build
```

**ผลลัพธ์ที่คาดหวัง:**
```
Project build complete. To flash, run this command:
idf.py -p (PORT) flash
```

---

## ส่วนที่ 5: การเชื่อมต่อและ Flash ESP32

### 5.1 เชื่อมต่อ ESP32

1. **เสียบสาย USB** ระหว่าง ESP32 และคอมพิวเตอร์
2. **ตรวจสอบ Serial Port:**

**Windows:**
```cmd
# ตรวจสอบใน Device Manager
# หรือใช้ PowerShell
Get-WmiObject -Class Win32_SerialPort | Select-Object Name,DeviceID
```

**macOS:**
```bash
ls /dev/cu.*
# ผลลัพธ์: /dev/cu.usbserial-xxxxxxxx
```

**Linux:**
```bash
ls /dev/ttyUSB*
# ผลลัพธ์: /dev/ttyUSB0
```

### 5.2 Flash โปรแกรม

```bash
# Flash พร้อมระบุ port
idf.py -p COM3 flash              # Windows
idf.py -p /dev/cu.usbserial-* flash  # macOS
idf.py -p /dev/ttyUSB0 flash      # Linux

# หรือใช้ auto-detect
idf.py flash
```

### 5.3 Monitor Serial Output

```bash
# เปิด Serial Monitor
idf.py monitor

# หรือ Flash และ Monitor พร้อมกัน
idf.py flash monitor
```

**ผลลัพธ์ที่คาดหวัง:**
```
I (xxx) ESP_IDF_TEST: 🚀 ESP-IDF Installation Test Started!
I (xxx) ESP_IDF_TEST: 📋 ESP-IDF Version: v5.1.x-xxx-gxxxxxxx
I (xxx) ESP_IDF_TEST: 💾 Free heap: 289234 bytes
I (xxx) ESP_IDF_TEST: ⚡ CPU frequency: 240 MHz
I (xxx) ESP_IDF_TEST: ⏰ System running: 1 seconds
```

---

## ส่วนที่ 6: การแก้ไขปัญหา

### 6.1 ปัญหาการติดตั้ง

**Problem:** "idf.py: command not found"
```bash
# Solution:
# Windows
%USERPROFILE%\esp\esp-idf\export.bat

# macOS/Linux  
. $HOME/esp/esp-idf/export.sh
```

**Problem:** "Permission denied" (Linux/macOS)
```bash
# Solution:
sudo usermod -a -G dialout $USER
# จากนั้น logout และ login ใหม่

# หรือใช้ชั่วคราว
sudo chmod 666 /dev/ttyUSB0
```

**Problem:** "Serial port not found"
```bash
# Solution:
# 1. ตรวจสอบสาย USB
# 2. ติดตั้ง CP210x หรือ FTDI driver
# 3. ลอง port อื่น
# 4. ตรวจสอบ Device Manager (Windows)
```

### 6.2 ปัญหา Build

**Problem:** Build errors
```bash
# Solution:
idf.py fullclean
idf.py build

# หรือลบ build folder
rm -rf build
idf.py build
```

**Problem:** "Target not set"
```bash
# Solution:
idf.py set-target esp32
```

### 6.3 ปัญหา Flash

**Problem:** "Failed to connect to ESP32"
```bash
# Solution:
# 1. กดปุ่ม BOOT บน ESP32 ขณะ flash
# 2. ลด baud rate
idf.py -p COM3 -b 115200 flash

# 3. ใช้ force mode
idf.py -p COM3 --before default_reset --after hard_reset flash
```

---

## ส่วนที่ 7: การตั้งค่าเพิ่มเติม

### 7.1 VS Code Integration

1. **ติดตั้ง VS Code Extension:**
   - เปิด VS Code
   - ไปที่ Extensions (Ctrl+Shift+X)
   - ค้นหา "ESP-IDF"
   - ติดตั้ง extension โดย Espressif

2. **กำหนดค่า ESP-IDF Path:**
   - Ctrl+Shift+P → "ESP-IDF: Configure ESP-IDF Extension"
   - เลือก "Use existing setup"
   - ระบุ path ของ ESP-IDF

### 7.2 การตั้งค่า Serial Monitor

```bash
# ตั้งค่า baud rate
idf.py -p COM3 -b 115200 monitor

# Filter messages
idf.py monitor --print_filter="ESP_IDF_TEST*"

# Save log to file
idf.py monitor | tee esp32_log.txt
```

### 7.3 Advanced Configuration

```bash
# เปิด menuconfig
idf.py menuconfig

# การตั้งค่าสำคัญ:
# - Serial flasher config → Flash size
# - Component config → Log output → Default log verbosity
# - Component config → FreeRTOS → Tick rate (Hz)
```

---

## ส่วนที่ 8: การทดสอบ ESP-NOW Components

### 8.1 แก้ไข CMakeLists.txt

แก้ไขไฟล์ `main/CMakeLists.txt`:
```cmake
idf_component_register(
    SRCS "hello_esp_idf.c"
    INCLUDE_DIRS "."
    REQUIRES esp_wifi esp_now nvs_flash
)
```

### 8.2 ทดสอบ ESP-NOW Initialization

แก้ไขไฟล์ `main/hello_esp_idf.c`:
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_system.h"
#include "esp_wifi.h"
#include "esp_event.h"
#include "esp_log.h"
#include "nvs_flash.h"
#include "esp_now.h"

static const char* TAG = "ESP_NOW_TEST";

void wifi_init(void) {
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_start());
    
    ESP_LOGI(TAG, "✅ WiFi initialized");
}

void espnow_init(void) {
    ESP_ERROR_CHECK(esp_now_init());
    ESP_LOGI(TAG, "✅ ESP-NOW initialized");
}

void app_main(void)
{
    ESP_ERROR_CHECK(nvs_flash_init());
    
    ESP_LOGI(TAG, "🚀 ESP-IDF + ESP-NOW Test Started!");
    
    wifi_init();
    espnow_init();
    
    // แสดง MAC Address
    uint8_t mac[6];
    esp_wifi_get_mac(WIFI_IF_STA, mac);
    ESP_LOGI(TAG, "📍 MAC Address: %02X:%02X:%02X:%02X:%02X:%02X", 
             mac[0], mac[1], mac[2], mac[3], mac[4], mac[5]);
    
    ESP_LOGI(TAG, "✅ All systems ready for ESP-NOW development!");
    
    while (1) {
        vTaskDelay(pdMS_TO_TICKS(5000));
    }
}
```

### 8.3 Build และทดสอบ

```bash
idf.py build flash monitor
```

**ผลลัพธ์ที่คาดหวัง:**
```
I (xxx) ESP_NOW_TEST: 🚀 ESP-IDF + ESP-NOW Test Started!
I (xxx) ESP_NOW_TEST: ✅ WiFi initialized
I (xxx) ESP_NOW_TEST: ✅ ESP-NOW initialized  
I (xxx) ESP_NOW_TEST: 📍 MAC Address: AA:BB:CC:DD:EE:FF
I (xxx) ESP_NOW_TEST: ✅ All systems ready for ESP-NOW development!
```

---

## ส่วนที่ 9: Checklist และการยืนยัน

### 9.1 Installation Checklist

- [ ] ESP-IDF ติดตั้งสำเร็จ
- [ ] `idf.py --version` แสดงเวอร์ชั่นถูกต้อง
- [ ] Python และ Git ทำงานได้
- [ ] สร้างโปรเจคใหม่ได้
- [ ] Build โปรเจคสำเร็จ
- [ ] Flash ลง ESP32 ได้
- [ ] Serial Monitor แสดงผลถูกต้อง
- [ ] ESP-NOW components ใช้งานได้

### 9.2 Performance Test

รันคำสั่งต่อไปนี้และบันทึกผล:

```bash
# ทดสอบ build time
time idf.py build

# ทดสอบ flash time  
time idf.py flash

# ตรวจสอบ memory usage
idf.py size
```

**บันทึกผล:**
- Build time: _______ seconds
- Flash time: _______ seconds  
- Binary size: _______ bytes
- RAM usage: _______ bytes

---

## ✅ สรุปการเรียนรู้

หลังจากทำ Worksheet นี้เสร็จ นักเรียนควรสามารถ:

- [ ] ติดตั้ง ESP-IDF ได้สำเร็จ
- [ ] ใช้คำสั่ง idf.py ได้
- [ ] สร้างและ build โปรเจคได้
- [ ] Flash โปรแกรมลง ESP32 ได้
- [ ] ใช้ Serial Monitor ได้
- [ ] แก้ไขปัญหาเบื้องต้นได้
- [ ] เตรียม ESP-NOW development environment ได้

### 🎯 ระดับความเข้าใจ:
- [ ] เริ่มต้น (1-3) - ติดตั้งและใช้งานพื้นฐานได้
- [ ] ปานกลาง (4-6) - แก้ปัญหาและปรับแต่งได้
- [ ] ขั้นสูง (7-10) - เข้าใจ architecture และ optimization ได้

---

**⏭️ พร้อมสำหรับ [Worksheet 2.2: การตั้งค่าและ Configuration](Worksheet-2.2-Configuration.md)**