# Worksheet 5.1: Debugging Tools
## การใช้เครื่องมือ Debug และ Monitoring

### 🎯 วัตถุประสงค์
- เรียนรู้การใช้เครื่องมือ Debug ต่างๆ
- สร้างระบบ Monitoring และ Statistics
- วิเคราะห์ปัญหาและหาสาเหตุ
- ใช้ Log system อย่างมีประสิทธิภาพ

---

## Advanced Logging System

### Multi-level Logging
```c
static const char* TAG_MAIN = "ESP_NOW_MAIN";
static const char* TAG_SEND = "ESP_NOW_SEND";
static const char* TAG_RECV = "ESP_NOW_RECV";
static const char* TAG_DEBUG = "ESP_NOW_DEBUG";

// ตัวอย่างการใช้งาน
ESP_LOGI(TAG_MAIN, "System started");
ESP_LOGD(TAG_DEBUG, "Debug info: value = %d", value);
ESP_LOGW(TAG_SEND, "Warning: retry count = %d", retry);
ESP_LOGE(TAG_RECV, "Error: invalid data received");
```

### Statistics System
```c
typedef struct {
    uint32_t packets_sent;
    uint32_t packets_received;
    uint32_t send_success;
    uint32_t send_failed;
    float success_rate;
    uint32_t total_bytes_sent;
    uint32_t total_bytes_received;
} espnow_stats_t;

void print_statistics(void) {
    ESP_LOGI(TAG_DEBUG, "📊 ESP-NOW Statistics:");
    ESP_LOGI(TAG_DEBUG, "   📤 Sent: %lu packets", stats.packets_sent);
    ESP_LOGI(TAG_DEBUG, "   📥 Received: %lu packets", stats.packets_received);
    ESP_LOGI(TAG_DEBUG, "   ✅ Success Rate: %.1f%%", stats.success_rate);
}
```

---

## การทดลอง

### การทดลองที่ 1: System Monitoring
1. สร้างระบบ Statistics
2. ติดตาม Memory usage
3. วัด Performance metrics

### การทดลองที่ 2: Network Diagnostics
1. Ping system
2. Signal strength monitoring
3. Packet loss analysis

### การทดลองที่ 3: Error Analysis
1. Error code classification
2. Failure pattern analysis
3. Recovery mechanisms

---

## Debugging Commands

```bash
# Monitor with filters
idf.py monitor --print_filter="ESP_NOW*"

# Save debug logs
idf.py monitor | tee debug.log

# Memory analysis
idf.py size-components
```

---

**⏭️ พร้อมสำหรับ [Worksheet 5.2: Performance Optimization](Worksheet-5.2-Performance-Optimization.md)**