# Worksheet 3.1: Basic Sender/Receiver
## การสร้าง Sender และ Receiver พื้นฐาน

### 🎯 วัตถุประสงค์
- สร้าง ESP-NOW Sender และ Receiver
- ทำความเข้าใจ MAC Address management
- ทดลองการส่งข้อมูลแบบต่างๆ
- เรียนรู้ Callback functions

### 🔧 อุปกรณ์ที่ต้องใช้
- ESP32 Development Board x 2 ตัว
- สาย USB x 2 เส้น
- คอมพิวเตอร์ที่ติดตั้ง ESP-IDF

---

## ส่วนที่ 1: การสร้าง Receiver

### ขั้นตอนที่ 1: สร้างโปรเจค Receiver
```bash
idf.py create-project espnow_receiver
cd espnow_receiver
```

### ขั้นตอนที่ 2: แก้ไข CMakeLists.txt
```cmake
idf_component_register(
    SRCS "main.c"
    INCLUDE_DIRS "."
    REQUIRES esp_wifi esp_now nvs_flash
)
```

### ขั้นตอนที่ 3: โค้ด Receiver
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

static const char* TAG = "ESP_NOW_RECEIVER";

typedef struct {
    char message[200];
    int counter;
    float sensor_value;
} esp_now_data_t;

// Callback เมื่อรับข้อมูล
void on_data_recv(const uint8_t *mac_addr, const uint8_t *data, int len) {
    esp_now_data_t *recv_data = (esp_now_data_t*)data;
    
    ESP_LOGI(TAG, "📥 Received from: %02X:%02X:%02X:%02X:%02X:%02X", 
             mac_addr[0], mac_addr[1], mac_addr[2], 
             mac_addr[3], mac_addr[4], mac_addr[5]);
    
    ESP_LOGI(TAG, "📨 Message: %s", recv_data->message);
    ESP_LOGI(TAG, "🔢 Counter: %d", recv_data->counter);
    ESP_LOGI(TAG, "🌡️ Sensor Value: %.2f", recv_data->sensor_value);
    ESP_LOGI(TAG, "📦 Data Length: %d bytes", len);
    ESP_LOGI(TAG, "--------------------------------");
}

void wifi_init(void) {
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_start());
    
    ESP_LOGI(TAG, "WiFi initialized");
}

void espnow_init(void) {
    ESP_ERROR_CHECK(esp_now_init());
    ESP_ERROR_CHECK(esp_now_register_recv_cb(on_data_recv));
    
    ESP_LOGI(TAG, "ESP-NOW initialized and ready to receive");
}

void print_mac_address(void) {
    uint8_t mac[6];
    esp_wifi_get_mac(WIFI_IF_STA, mac);
    ESP_LOGI(TAG, "📍 My MAC Address: %02X:%02X:%02X:%02X:%02X:%02X", 
             mac[0], mac[1], mac[2], mac[3], mac[4], mac[5]);
    ESP_LOGI(TAG, "⚠️ Copy this MAC to Sender code!");
}

void app_main(void) {
    ESP_ERROR_CHECK(nvs_flash_init());
    
    wifi_init();
    print_mac_address();
    espnow_init();
    
    ESP_LOGI(TAG, "🎯 ESP-NOW Receiver started - Waiting for data...");
    
    while (1) {
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}
```

### ขั้นตอนที่ 4: Build และ Flash
```bash
idf.py build flash monitor
```

**📝 บันทึก MAC Address ของ Receiver:**
```
MAC Address: __ : __ : __ : __ : __ : __
```

---

## ส่วนที่ 2: การสร้าง Sender

### ขั้นตอนที่ 1: สร้างโปรเจค Sender
```bash
idf.py create-project espnow_sender
cd espnow_sender
```

### ขั้นตอนที่ 2: โค้ด Sender
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

static const char* TAG = "ESP_NOW_SENDER";

// MAC Address ของ Receiver (แก้ไขให้ตรงกับของจริง)
static uint8_t receiver_mac[6] = {0x24, 0x6F, 0x28, 0xAA, 0xBB, 0xCC};

typedef struct {
    char message[200];
    int counter;
    float sensor_value;
} esp_now_data_t;

// Callback เมื่อส่งข้อมูลเสร็จ
void on_data_sent(const uint8_t *mac_addr, esp_now_send_status_t status) {
    if (status == ESP_NOW_SEND_SUCCESS) {
        ESP_LOGI(TAG, "✅ Data sent successfully");
    } else {
        ESP_LOGE(TAG, "❌ Failed to send data");
    }
}

void wifi_init(void) {
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_start());
    
    ESP_LOGI(TAG, "WiFi initialized");
}

void espnow_init(void) {
    ESP_ERROR_CHECK(esp_now_init());
    ESP_ERROR_CHECK(esp_now_register_send_cb(on_data_sent));
    
    // เพิ่ม Peer (Receiver)
    esp_now_peer_info_t peer_info = {};
    memcpy(peer_info.peer_addr, receiver_mac, 6);
    peer_info.channel = 0;
    peer_info.encrypt = false;
    
    ESP_ERROR_CHECK(esp_now_add_peer(&peer_info));
    ESP_LOGI(TAG, "ESP-NOW initialized and peer added");
}

void app_main(void) {
    ESP_ERROR_CHECK(nvs_flash_init());
    
    wifi_init();
    espnow_init();
    
    esp_now_data_t send_data;
    int counter = 0;
    
    ESP_LOGI(TAG, "🚀 ESP-NOW Sender started");
    
    while (1) {
        // เตรียมข้อมูลที่จะส่ง
        sprintf(send_data.message, "Hello from Sender! Time: %d", counter);
        send_data.counter = counter++;
        send_data.sensor_value = 25.5 + (float)(counter % 10);
        
        // ส่งข้อมูล
        esp_err_t result = esp_now_send(receiver_mac, (uint8_t*)&send_data, sizeof(send_data));
        if (result == ESP_OK) {
            ESP_LOGI(TAG, "📤 Sending: %s (Counter: %d)", send_data.message, send_data.counter);
        } else {
            ESP_LOGE(TAG, "❌ Error sending data");
        }
        
        vTaskDelay(pdMS_TO_TICKS(2000)); // ส่งทุก 2 วินาที
    }
}
```

### ขั้นตอนที่ 3: แก้ไข MAC Address
⚠️ **สำคัญ:** แก้ไข `receiver_mac` ในโค้ด Sender ให้ตรงกับ MAC Address ของ Receiver

### ขั้นตอนที่ 4: Build และ Flash
```bash
idf.py build flash monitor
```

---

## ส่วนที่ 3: การทดสอบและบันทึกผล

### การทดลองที่ 1: การสื่อสารพื้นฐาน

**ผลลัพธ์ที่คาดหวัง:**

**Sender Output:**
```
I (2234) ESP_NOW_SENDER: 📤 Sending: Hello from Sender! Time: 0 (Counter: 0)
I (2334) ESP_NOW_SENDER: ✅ Data sent successfully
I (4234) ESP_NOW_SENDER: 📤 Sending: Hello from Sender! Time: 1 (Counter: 1)
```

**Receiver Output:**
```
I (2334) ESP_NOW_RECEIVER: 📥 Received from: 24:6F:28:AA:BB:CC
I (2334) ESP_NOW_RECEIVER: 📨 Message: Hello from Sender! Time: 0
I (2334) ESP_NOW_RECEIVER: 🔢 Counter: 0
I (2334) ESP_NOW_RECEIVER: 🌡️ Sensor Value: 25.50
```

**📝 บันทึกผลการทดลอง:**
- การส่งข้อมูลสำเร็จ: ✅ / ❌
- ความถี่การส่ง: ทุก _____ วินาที
- ข้อมูลที่รับได้ถูกต้อง: ✅ / ❌

---

## ส่วนที่ 4: การทดลองข้อมูลประเภทต่างๆ

### การทดลองที่ 2: ส่งข้อมูล Sensor

แก้ไขโครงสร้างข้อมูล:
```c
typedef struct {
    char device_id[20];
    float temperature;
    float humidity;
    int light_level;
    bool motion_detected;
    uint32_t timestamp;
} sensor_data_t;
```

**แบบฝึกหัด:**
1. สร้างโครงสร้างข้อมูลใหม่
2. แก้ไขโค้ด Sender และ Receiver
3. ทดสอบการส่งข้อมูล sensor

---

## ส่วนที่ 5: การจัดการ Error

### การทดลองที่ 3: Error Handling

เพิ่ม Error handling ในโค้ด:
```c
esp_err_t result = esp_now_send(receiver_mac, (uint8_t*)&send_data, sizeof(send_data));
switch (result) {
    case ESP_OK:
        ESP_LOGI(TAG, "Send initiated");
        break;
    case ESP_ERR_ESPNOW_NOT_INIT:
        ESP_LOGE(TAG, "ESP-NOW not init");
        break;
    case ESP_ERR_ESPNOW_ARG:
        ESP_LOGE(TAG, "Invalid argument");
        break;
    case ESP_ERR_ESPNOW_INTERNAL:
        ESP_LOGE(TAG, "Internal error");
        break;
    case ESP_ERR_ESPNOW_NO_MEM:
        ESP_LOGE(TAG, "No memory");
        break;
    default:
        ESP_LOGE(TAG, "Unknown error: %s", esp_err_to_name(result));
}
```

**คำถาม:**
1. Error ไหนที่เกิดขึ้นบ่อยที่สุด?
2. วิธีการป้องกัน Error แต่ละประเภท?

---

## ส่วนที่ 6: การทดสอบ Range และ Performance

### การทดลองที่ 4: ทดสอบระยะทาง

**ขั้นตอน:**
1. ทดสอบการสื่อสารในระยะ 1 เมตร
2. เพิ่มระยะทางเป็น 5, 10, 20 เมตร
3. ทดสอบกับสิ่งกีดขวาง (กำแพง, ประตู)

**📝 บันทึกผล:**
| ระยะทาง | สภาพแวดล้อม | สถานะการส่ง | หมายเหตุ |
|---------|-------------|-------------|---------|
| 1m | ห้องปิด | ✅ / ❌ | |
| 5m | ห้องปิด | ✅ / ❌ | |
| 10m | กลางแจ้ง | ✅ / ❌ | |
| 20m | มีสิ่งกีดขวาง | ✅ / ❌ | |

---

## ส่วนที่ 7: การปรับปรุงโค้ด

### การทดลองที่ 5: เพิ่ม Features

1. **Message Counter:** นับจำนวนข้อความที่ส่งและรับ
2. **RSSI Monitoring:** แสดงความแรงสัญญาณ
3. **Timestamp:** เพิ่มเวลาในการส่งข้อมูล

**ตัวอย่างการเพิ่ม Message Counter:**
```c
static int sent_count = 0;
static int received_count = 0;

// ใน Sender
sent_count++;
ESP_LOGI(TAG, "📊 Total sent: %d", sent_count);

// ใน Receiver  
received_count++;
ESP_LOGI(TAG, "📊 Total received: %d", received_count);
```

---

## ส่วนที่ 8: Troubleshooting

### ปัญหาที่พบบ่อยและวิธีแก้ไข

| ปัญหา | สาเหตุ | วิธีแก้ไข |
|--------|--------|-----------|
| ไม่ได้รับข้อมูล | MAC Address ผิด | ตรวจสอบ MAC Address |
| ส่งไม่สำเร็จ | Peer ไม่ได้เพิ่ม | เรียก esp_now_add_peer() |
| ข้อมูลผิดเพี้ยน | Struct size ไม่ตรง | ใช้ __packed__ attribute |
| การเชื่อมต่อขาดๆ หายๆ | ระยะทางไกล | ลดระยะทางหรือเพิ่ม antenna |

---

## ✅ สรุปการเรียนรู้

หลังจากทำ Worksheet นี้เสร็จ นักเรียนควรสามารถ:

- [ ] สร้าง ESP-NOW Sender และ Receiver ได้
- [ ] จัดการ MAC Address และ Peer ได้  
- [ ] ใช้ Callback functions ได้
- [ ] ส่งข้อมูลประเภทต่างๆ ได้
- [ ] จัดการ Error เบื้องต้นได้
- [ ] ทดสอบ Range และ Performance ได้

### 🎯 ระดับความเข้าใจ:
- [ ] เริ่มต้น (1-3) - ส่งข้อมูลพื้นฐานได้
- [ ] ปานกลาง (4-6) - จัดการ Error และปรับปรุงได้
- [ ] ขั้นสูง (7-10) - เข้าใจ Performance และ Optimization

---

**⏭️ พร้อมสำหรับ [Worksheet 3.2: Two-way Communication](Worksheet-3.2-Two-way-Communication.md)**