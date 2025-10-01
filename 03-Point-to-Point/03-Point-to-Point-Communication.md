# บทที่ 3: การสื่อสารแบบ Point-to-Point
## ESP-NOW ระหว่าง ESP32 2 ตัว

![Point to Point Communication](https://docs.espressif.com/projects/esp-idf/en/latest/_images/esp-now-one-to-one.png)

ในบทนี้เราจะเรียนรู้การสื่อสารแบบ Point-to-Point ระหว่าง ESP32 2 ตัว โดยหนึ่งตัวเป็น **Sender** และอีกตัวเป็น **Receiver**

### เป้าหมายการเรียนรู้:
- เข้าใจการทำงานของ ESP-NOW API
- สร้าง Sender และ Receiver
- ส่งข้อมูลแบบ Two-way communication
- จัดการ MAC Address และ Peer Management
- ทำความเข้าใจ Callback Functions

---

## ทฤษฎี: ESP-NOW Point-to-Point Communication

### การทำงานของ ESP-NOW

```
ESP32 A (Sender)                    ESP32 B (Receiver)
      |                                    |
      |------ Data Packet (250 bytes) -----|
      |                                    |
      |<----- ACK/Status Callback --------|
```

### ขั้นตอนการทำงาน:

1. **Initialize WiFi** - เตรียม WiFi สำหรับ ESP-NOW
2. **Initialize ESP-NOW** - เริ่มต้น ESP-NOW protocol
3. **Add Peer** - เพิ่ม MAC address ของอุปกรณ์ที่ต้องการสื่อสาร
4. **Register Callbacks** - ลงทะเบียน function สำหรับรับส่งข้อมูล
5. **Send Data** - ส่งข้อมูลผ่าน ESP-NOW
6. **Receive Data** - รับข้อมูลและประมวลผล

### Key Concepts:

- **MAC Address**: ตัวระบุเฉพาะของแต่ละ ESP32
- **Peer**: อุปกรณ์ที่สามารถสื่อสารด้วย ESP-NOW ได้
- **Callback**: Function ที่เรียกเมื่อมี Event เกิดขึ้น
- **Channel**: ช่องสัญญาณ WiFi ที่ใช้สื่อสาร

---

## การใช้งาน ESP-NOW API

### การเริ่มต้นระบบ

```c
// เริ่มต้น WiFi
void wifi_init(void) {
    ESP_ERROR_CHECK(esp_netif_init());
    ESP_ERROR_CHECK(esp_event_loop_create_default());
    
    wifi_init_config_t cfg = WIFI_INIT_CONFIG_DEFAULT();
    ESP_ERROR_CHECK(esp_wifi_init(&cfg));
    ESP_ERROR_CHECK(esp_wifi_set_storage(WIFI_STORAGE_RAM));
    ESP_ERROR_CHECK(esp_wifi_set_mode(WIFI_MODE_STA));
    ESP_ERROR_CHECK(esp_wifi_start());
}

// เริ่มต้น ESP-NOW
void espnow_init(void) {
    ESP_ERROR_CHECK(esp_now_init());
    ESP_ERROR_CHECK(esp_now_register_send_cb(on_data_sent));
    ESP_ERROR_CHECK(esp_now_register_recv_cb(on_data_recv));
}
```

### การจัดการ Peers

```c
// เพิ่ม Peer
esp_now_peer_info_t peer_info = {};
memcpy(peer_info.peer_addr, target_mac, 6);
peer_info.channel = 0;
peer_info.encrypt = false;
ESP_ERROR_CHECK(esp_now_add_peer(&peer_info));
```

### Callback Functions

```c
// Callback เมื่อส่งข้อมูลเสร็จ
void on_data_sent(const uint8_t *mac_addr, esp_now_send_status_t status) {
    if (status == ESP_NOW_SEND_SUCCESS) {
        ESP_LOGI(TAG, "✅ Data sent successfully");
    } else {
        ESP_LOGE(TAG, "❌ Failed to send data");
    }
}

// Callback เมื่อรับข้อมูล
void on_data_recv(const uint8_t *mac_addr, const uint8_t *data, int len) {
    ESP_LOGI(TAG, "📥 Received %d bytes from: %02X:%02X:%02X:%02X:%02X:%02X", 
             len, mac_addr[0], mac_addr[1], mac_addr[2], 
             mac_addr[3], mac_addr[4], mac_addr[5]);
}
```

---

## การส่งและรับข้อมูล

### โครงสร้างข้อมูล

```c
typedef struct {
    char message[200];
    int counter;
    float sensor_value;
    uint32_t timestamp;
} esp_now_data_t;
```

### การส่งข้อมูล

```c
void send_data(const uint8_t* target_mac, const char* message) {
    esp_now_data_t send_data;
    strcpy(send_data.message, message);
    send_data.counter = counter++;
    send_data.sensor_value = read_sensor();
    send_data.timestamp = esp_timer_get_time() / 1000;
    
    esp_err_t result = esp_now_send(target_mac, (uint8_t*)&send_data, sizeof(send_data));
    if (result == ESP_OK) {
        ESP_LOGI(TAG, "📤 Sending: %s", message);
    }
}
```

---

## Communication Patterns

### 1. One-Way Communication (Sender → Receiver)
- Sender ส่งข้อมูลเป็นระยะ
- Receiver รับและประมวลผล

### 2. Two-Way Communication (Bidirectional)
- ทั้งสองฝ่ายสามารถส่งและรับได้
- มีการตอบรับ (Acknowledgment)

### 3. Request-Response Pattern
- Sender ส่งคำขอ
- Receiver ตอบกลับ

---

## การจัดการ Error และ Reliability

### Error Handling
```c
esp_err_t result = esp_now_send(mac, data, len);
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
    default:
        ESP_LOGE(TAG, "Send error: %s", esp_err_to_name(result));
}
```

### Retry Mechanism
```c
#define MAX_RETRY 3
int retry_count = 0;

void send_with_retry(const uint8_t* mac, const uint8_t* data, size_t len) {
    while (retry_count < MAX_RETRY) {
        if (esp_now_send(mac, data, len) == ESP_OK) {
            break;
        }
        retry_count++;
        vTaskDelay(pdMS_TO_TICKS(100));
    }
}
```

---

## Performance Considerations

### 1. Message Size Optimization
- ข้อมูลสูงสุด 250 bytes ต่อ packet
- ใช้ข้อมูลให้มีประสิทธิภาพ

### 2. Transmission Rate
- หลีกเลี่ยงการส่งบ่อยเกินไป
- ใช้ Queue system สำหรับ message management

### 3. Power Management
- ใช้ Sleep modes เมื่อไม่ได้ใช้งาน
- ปรับ transmission interval ตามความต้องการ

---

## การแก้ไขปัญหาเบื้องต้น

### ปัญหาที่พบบ่อย:

1. **"Peer not found" Error**
   - ตรวจสอบ MAC Address
   - เพิ่ม Peer ก่อนส่งข้อมูล

2. **ไม่มีข้อมูลส่งมา**
   - ตรวจสอบ WiFi Channel
   - ระยะทางและสิ่งกีดขวาง

3. **ข้อมูลเพี้ยน**
   - ตรวจสอบขนาด struct
   - ใช้ `__attribute__((packed))`

---

## สรุป

Point-to-Point communication เป็นรูปแบบพื้นฐานของ ESP-NOW ที่ใช้ในการสื่อสารระหว่างอุปกรณ์ 2 ตัว มีข้อดี:

- **ง่ายต่อการใช้งาน**
- **ความเร็วสูง**
- **การควบคุมที่ดี**
- **เหมาะกับ Real-time applications**

---

## 📋 ใบงานปฏิบัติสำหรับบทนี้

1. **[Worksheet 3.1: Basic Sender/Receiver](Worksheets/Worksheet-3.1-Basic-Sender-Receiver.md)** - การสร้าง Sender และ Receiver พื้นฐาน
2. **[Worksheet 3.2: Two-way Communication](Worksheets/Worksheet-3.2-Two-way-Communication.md)** - การสื่อสารแบบสองทิศทาง
3. **[Worksheet 3.3: Advanced Features](Worksheets/Worksheet-3.3-Advanced-Features.md)** - การใช้งานขั้นสูงและ Optimization

**➡️ ไปที่: [04-Broadcasting](../04-Broadcasting/04-Group-Broadcasting.md)**