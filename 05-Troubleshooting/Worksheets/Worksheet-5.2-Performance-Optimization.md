# Worksheet 5.2: Performance Optimization
## การปรับปรุงประสิทธิภาพ ESP-NOW

### 🎯 วัตถุประสงค์
- เรียนรู้เทคนิค Performance optimization
- ปรับปรุง Throughput และ Latency
- จัดการ Power consumption
- เพิ่มความเสถียรของระบบ

---

## Message Queue System

### Queue Implementation
```c
#define MAX_QUEUE_SIZE 20

typedef struct {
    uint8_t mac[6];
    uint8_t data[250];
    size_t data_len;
    uint8_t retry_count;
    bool in_use;
} message_queue_item_t;

static message_queue_item_t send_queue[MAX_QUEUE_SIZE];
```

### Adaptive Transmission
```c
void update_transmission_rate(const uint8_t* mac, bool success, int8_t rssi) {
    if (success && rssi > -50) {
        // เพิ่มความถี่การส่ง
        decrease_transmission_interval(mac);
    } else if (!success || rssi < -70) {
        // ลดความถี่การส่ง
        increase_transmission_interval(mac);
    }
}
```

---

## Power Management

### Sleep Modes
```c
// Light Sleep
void enter_light_sleep(uint32_t sleep_time_ms) {
    esp_sleep_enable_timer_wakeup(sleep_time_ms * 1000);
    esp_light_sleep_start();
}

// Deep Sleep
void enter_deep_sleep(uint32_t sleep_time_sec) {
    esp_sleep_enable_timer_wakeup(sleep_time_sec * 1000000ULL);
    esp_deep_sleep_start();
}
```

### Battery Monitoring
```c
void monitor_battery_level(void) {
    // อ่านค่าแบตเตอรี่จาก ADC
    int adc_value = adc1_get_raw(ADC1_CHANNEL_0);
    float battery_voltage = (adc_value * 3.3) / 4095.0;
    
    if (battery_voltage < 3.2) {
        ESP_LOGW(TAG, "Low battery: %.2fV", battery_voltage);
        enter_power_save_mode();
    }
}
```

---

## การทดลอง

### การทดลองที่ 1: Throughput Optimization
1. วัด baseline performance
2. ใช้ Message queue
3. เปรียบเทียบผลลัพธ์

### การทดลองที่ 2: Power Optimization
1. วัด current consumption
2. ใช้ Sleep modes
3. คำนวณ battery life

### การทดลองที่ 3: Reliability Enhancement
1. ทดสอบ Retry mechanism
2. วัด packet loss rate
3. ปรับปรุง Error recovery

---

**⏭️ พร้อมสำหรับ [Worksheet 5.3: Security Implementation](Worksheet-5.3-Security.md)**