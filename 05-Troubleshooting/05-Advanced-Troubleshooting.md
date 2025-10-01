# บทที่ 5: Advanced Troubleshooting และ Optimization
## การแก้ปัญหาและปรับปรุงประสิทธิภาพ ESP-NOW

![ESP-NOW Optimization](https://docs.espressif.com/projects/esp-idf/en/latest/_images/esp-now-data-flow.png)

ในบทสุดท้ายนี้เราจะเรียนรู้เทคนิคขั้นสูงในการ **Debug**, **แก้ปัญหา** และ **ปรับปรุงประสิทธิภาพ** ของระบบ ESP-NOW

### เป้าหมายการเรียนรู้:
- วิเคราะห์และแก้ไขปัญหาที่พบบ่อย
- ใช้เครื่องมือ Debug และ Monitoring
- เพิ่มประสิทธิภาพการส่งข้อมูล
- จัดการ Power Management
- ปรับปรุงความเสถียรของระบบ

---

## การวิเคราะห์ปัญหาที่พบบ่อย

### 1. Connection Issues

| ปัญหา | สาเหตุ | วิธีแก้ไข |
|--------|--------|-----------|
| `ESP_ERR_ESPNOW_NOT_INIT` | ยังไม่เรียก `esp_now_init()` | เรียก `esp_now_init()` ก่อน |
| `ESP_ERR_ESPNOW_ARG` | MAC Address ผิด | ตรวจสอบ MAC Address |
| `ESP_ERR_ESPNOW_FULL` | Peer List เต็ม | ลบ Peer ที่ไม่ใช้ |
| `ESP_ERR_ESPNOW_NOT_FOUND` | ไม่พบ Peer | เพิ่ม Peer ก่อนส่ง |

### 2. Performance Issues

```c
// การตรวจสอบประสิทธิภาพ
typedef struct {
    uint32_t packets_sent;
    uint32_t packets_received;
    uint32_t send_success;
    uint32_t send_failed;
    uint32_t total_bytes_sent;
    float success_rate;
} espnow_stats_t;

void update_statistics(bool success, size_t data_len) {
    stats.packets_sent++;
    stats.total_bytes_sent += data_len;
    
    if (success) {
        stats.send_success++;
    } else {
        stats.send_failed++;
    }
    
    stats.success_rate = (float)stats.send_success / stats.packets_sent * 100;
}
```

---

## เครื่องมือ Debug และ Monitoring

### 1. Advanced Logging System

```c
#include "esp_log.h"

static const char* TAG_MAIN = "ESP_NOW_MAIN";
static const char* TAG_SEND = "ESP_NOW_SEND";
static const char* TAG_RECV = "ESP_NOW_RECV";
static const char* TAG_DEBUG = "ESP_NOW_DEBUG";

// ฟังก์ชันแสดงสถิติระบบ
void print_system_stats(void) {
    ESP_LOGI(TAG_DEBUG, "💾 Free Heap: %lu bytes", esp_get_free_heap_size());
    ESP_LOGI(TAG_DEBUG, "💾 Min Free Heap: %lu bytes", esp_get_minimum_free_heap_size());
    ESP_LOGI(TAG_DEBUG, "⚡ CPU Frequency: %lu MHz", esp_clk_cpu_freq() / 1000000);
    
    // แสดงจำนวน Peers
    esp_now_peer_num_t peer_num;
    esp_now_get_peer_num(&peer_num);
    ESP_LOGI(TAG_DEBUG, "👥 Total Peers: %d", peer_num.total_num);
}
```

### 2. Packet Statistics

```c
void print_packet_statistics(void) {
    uint32_t uptime = esp_timer_get_time() / 1000000;
    
    ESP_LOGI(TAG_DEBUG, "📊 ESP-NOW Statistics (Uptime: %lu sec):", uptime);
    ESP_LOGI(TAG_DEBUG, "   📤 Sent: %lu packets (%lu bytes)", 
             stats.packets_sent, stats.total_bytes_sent);
    ESP_LOGI(TAG_DEBUG, "   ✅ Success Rate: %.1f%%", stats.success_rate);
    
    if (uptime > 0) {
        ESP_LOGI(TAG_DEBUG, "   ⚡ TX Rate: %.1f pkt/sec", 
                 (float)stats.packets_sent / uptime);
    }
}
```

### 3. Network Diagnostics

```c
// ฟังก์ชันตรวจสอบการเชื่อมต่อ
void ping_peer(const uint8_t* mac_addr) {
    typedef struct {
        char message[20];
        uint32_t timestamp;
        uint32_t sequence;
    } ping_data_t;
    
    static uint32_t ping_sequence = 0;
    ping_data_t ping_data;
    
    strcpy(ping_data.message, "PING");
    ping_data.timestamp = esp_timer_get_time() / 1000;
    ping_data.sequence = ++ping_sequence;
    
    ESP_LOGI(TAG_DEBUG, "🏓 Pinging peer...");
    esp_err_t result = esp_now_send(mac_addr, (uint8_t*)&ping_data, sizeof(ping_data));
}
```

---

## เทคนิคเพิ่มประสิทธิภาพ

### 1. Message Queuing และ Retry Mechanism

```c
#define MAX_QUEUE_SIZE 20
#define MAX_RETRY_COUNT 3

typedef struct {
    uint8_t mac[6];
    uint8_t data[250];
    size_t data_len;
    uint32_t timestamp;
    uint8_t retry_count;
    bool in_use;
} message_queue_item_t;

static message_queue_item_t send_queue[MAX_QUEUE_SIZE];

bool queue_message(const uint8_t* mac, const uint8_t* data, size_t len) {
    // หา slot ว่าง
    for (int i = 0; i < MAX_QUEUE_SIZE; i++) {
        if (!send_queue[i].in_use) {
            memcpy(send_queue[i].mac, mac, 6);
            memcpy(send_queue[i].data, data, len);
            send_queue[i].data_len = len;
            send_queue[i].timestamp = esp_timer_get_time() / 1000;
            send_queue[i].retry_count = 0;
            send_queue[i].in_use = true;
            return true;
        }
    }
    return false; // Queue full
}
```

### 2. Adaptive Transmission Rate

```c
typedef struct {
    uint8_t mac[6];
    uint32_t success_count;
    uint32_t fail_count;
    uint32_t transmission_interval; // ms
    int8_t rssi;
} peer_performance_t;

void update_peer_performance(const uint8_t* mac, bool success, int8_t rssi) {
    peer_performance_t *peer = find_peer_performance(mac);
    
    if (peer) {
        peer->rssi = rssi;
        
        if (success) {
            peer->success_count++;
            // ลดช่วง transmission ถ้าสัญญาณดี
            if (rssi > -50 && peer->transmission_interval > 500) {
                peer->transmission_interval -= 100;
            }
        } else {
            peer->fail_count++;
            // เพิ่มช่วง transmission ถ้าสัญญาณแย่
            if (rssi < -70 && peer->transmission_interval < 5000) {
                peer->transmission_interval += 200;
            }
        }
    }
}
```

### 3. Power Management

```c
#include "esp_pm.h"
#include "esp_sleep.h"

// การตั้งค่า Power Management
void init_power_management(void) {
    esp_pm_config_esp32_t pm_config = {
        .max_freq_mhz = 240,
        .min_freq_mhz = 80,
        .light_sleep_enable = true
    };
    
    esp_err_t result = esp_pm_configure(&pm_config);
    if (result == ESP_OK) {
        ESP_LOGI(TAG_MAIN, "🔋 Power management enabled");
    }
}

// Deep Sleep สำหรับ Battery-powered devices
void enter_deep_sleep(uint32_t sleep_time_sec) {
    ESP_LOGI(TAG_MAIN, "💤 Entering deep sleep for %lu seconds", sleep_time_sec);
    
    esp_sleep_enable_timer_wakeup(sleep_time_sec * 1000000ULL);
    esp_wifi_stop();
    esp_deep_sleep_start();
}
```

---

## การจัดการ Security

### 1. Message Authentication

```c
#include "mbedtls/md.h"

// การสร้าง Hash สำหรับ Authentication
void calculate_message_hash(const uint8_t* data, size_t data_len, uint8_t* hash) {
    mbedtls_md_context_t ctx;
    const mbedtls_md_info_t* info = mbedtls_md_info_from_type(MBEDTLS_MD_SHA256);
    
    mbedtls_md_init(&ctx);
    mbedtls_md_setup(&ctx, info, 1); // HMAC mode
    mbedtls_md_hmac_starts(&ctx, (const unsigned char*)AUTH_KEY, strlen(AUTH_KEY));
    mbedtls_md_hmac_update(&ctx, data, data_len);
    mbedtls_md_hmac_finish(&ctx, hash);
    mbedtls_md_free(&ctx);
}

// โครงสร้างข้อมูลที่มี Authentication
typedef struct {
    char sender_id[20];
    char message[180];
    uint32_t timestamp;
    uint8_t hash[32]; // SHA256 hash
} secure_message_t;
```

### 2. Simple Encryption

```c
// XOR encryption สำหรับการป้องกันพื้นฐาน
void encrypt_decrypt_xor(uint8_t* data, size_t len, const char* key) {
    size_t key_len = strlen(key);
    
    for (size_t i = 0; i < len; i++) {
        data[i] ^= key[i % key_len];
    }
}
```

---

## การ Debug ขั้นสูง

### 1. Packet Analyzer

```c
// ฟังก์ชันแสดงข้อมูล Raw ของ Packet
void dump_packet_hex(const uint8_t* data, size_t len, const char* label) {
    ESP_LOGI(TAG_DEBUG, "📦 %s (%d bytes):", label, len);
    
    for (size_t i = 0; i < len; i += 16) {
        char hex_line[64] = {0};
        char ascii_line[17] = {0};
        
        for (size_t j = 0; j < 16 && (i + j) < len; j++) {
            sprintf(hex_line + (j * 3), "%02X ", data[i + j]);
            
            if (data[i + j] >= 32 && data[i + j] <= 126) {
                ascii_line[j] = data[i + j];
            } else {
                ascii_line[j] = '.';
            }
        }
        
        ESP_LOGI(TAG_DEBUG, "   %04X: %-48s |%s|", i, hex_line, ascii_line);
    }
}
```

### 2. Performance Profiling

```c
#include "esp_timer.h"

#define PROFILE_START(name) \
    uint64_t prof_start_##name = esp_timer_get_time();

#define PROFILE_END(name) \
    do { \
        uint64_t prof_duration = esp_timer_get_time() - prof_start_##name; \
        ESP_LOGI(TAG_DEBUG, "⚡ %s took %llu μs", #name, prof_duration); \
    } while(0)

// ตัวอย่างการใช้งาน
void example_profiled_function(void) {
    PROFILE_START(send_data);
    
    // ทำงานที่ต้องการวัดประสิทธิภาพ
    esp_now_send(mac, data, len);
    
    PROFILE_END(send_data);
}
```

---

## การแก้ไขปัญหาเฉพาะ

### 1. Range และ Signal Issues

```c
// การตรวจสอบคุณภาพสัญญาณ
void monitor_signal_quality(void) {
    wifi_ap_record_t ap_info;
    if (esp_wifi_sta_get_ap_info(&ap_info) == ESP_OK) {
        ESP_LOGI(TAG_DEBUG, "📶 WiFi RSSI: %d dBm", ap_info.rssi);
        ESP_LOGI(TAG_DEBUG, "📡 Channel: %d", ap_info.primary);
        
        if (ap_info.rssi < -70) {
            ESP_LOGW(TAG_DEBUG, "⚠️ Weak signal detected!");
        }
    }
}
```

### 2. Memory Management

```c
// การตรวจสอบ Memory usage
void monitor_memory_usage(void) {
    uint32_t free_heap = esp_get_free_heap_size();
    uint32_t min_free_heap = esp_get_minimum_free_heap_size();
    
    ESP_LOGI(TAG_DEBUG, "💾 Current Free: %lu bytes", free_heap);
    ESP_LOGI(TAG_DEBUG, "💾 Minimum Free: %lu bytes", min_free_heap);
    
    if (free_heap < 10000) {
        ESP_LOGW(TAG_DEBUG, "⚠️ Low memory warning!");
    }
}
```

---

## เครื่องมือและคำสั่งที่มีประโยชน์

### 1. idf.py Commands สำหรับ Debug

```bash
# แสดง Log แบบ Filtered
idf.py monitor --print_filter="ESP_NOW*"

# Monitor พร้อม Timestamp
idf.py monitor --timestamp

# Save log ลงไฟล์
idf.py monitor | tee esp32_debug.log

# ตั้งค่า Log level
idf.py menuconfig
# Component config -> Log output -> Default log verbosity
```

### 2. การวิเคราะห์ Performance

```bash
# ดู Memory usage
idf.py size

# วิเคราะห์ Binary
idf.py size-components

# ดู Partition table
idf.py partition_table
```

---

## Best Practices และคำแนะนำ

### 1. Code Structure

```c
// ใช้ Log levels ที่เหมาะสม
ESP_LOGE() // สำหรับ Error
ESP_LOGW() // สำหรับ Warning  
ESP_LOGI() // สำหรับ Information
ESP_LOGD() // สำหรับ Debug (ปิดใน Production)
```

### 2. Error Handling

```c
esp_err_t result = esp_now_send(mac, data, len);
if (result != ESP_OK) {
    ESP_LOGE(TAG, "Send failed: %s", esp_err_to_name(result));
    // จัดการ error
}
```

### 3. Resource Management

```c
// ตรวจสอบ Memory อยู่เสมอ
if (esp_get_free_heap_size() < 10000) {
    ESP_LOGW(TAG, "Low memory warning");
}
```

---

## สรุป

การ Troubleshooting และ Optimization ของ ESP-NOW ต้องอาศัย:

1. **การใช้เครื่องมือ Debug ที่เหมาะสม**
2. **การตรวจสอบ Performance อย่างสม่ำเสมอ**
3. **การจัดการ Memory และ Power อย่างมีประสิทธิภาพ**
4. **การเพิ่ม Security ตามความเหมาะสม**
5. **การทดสอบในสภาพแวดล้อมจริง**

---

## 📋 ใบงานปฏิบัติสำหรับบทนี้

1. **[Worksheet 5.1: Debugging Tools](Worksheets/Worksheet-5.1-Debugging-Tools.md)** - การใช้เครื่องมือ Debug และ Monitoring
2. **[Worksheet 5.2: Performance Optimization](Worksheets/Worksheet-5.2-Performance-Optimization.md)** - การปรับปรุงประสิทธิภาพ
3. **[Worksheet 5.3: Security Implementation](Worksheets/Worksheet-5.3-Security.md)** - การเพิ่ม Security features

---

**🎉 ยินดีด้วย! คุณได้เสร็จสิ้นการเรียนรู้ ESP-NOW ทั้ง 5 บทแล้ว**

*พร้อมที่จะพัฒนาโครงการ ESP-NOW ของคุณเองได้แล้ว!*