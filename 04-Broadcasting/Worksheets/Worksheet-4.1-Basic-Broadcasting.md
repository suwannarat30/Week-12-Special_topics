# Worksheet 4.1: Basic Broadcasting
## การสร้างระบบ Broadcasting พื้นฐาน

### 🎯 วัตถุประสงค์
- เรียนรู้การใช้ Broadcast Address
- สร้าง One-to-Many Communication
- ทดลองการส่งข้อมูลแบบ Broadcasting
- จัดการ Multiple Receivers

### 🔧 อุปกรณ์ที่ต้องใช้
- ESP32 Development Board x 3-4 ตัว
- 1 ตัว = Broadcaster (Master)
- 2-3 ตัว = Receivers (Clients)

---

## การสร้าง Broadcaster

### Broadcast Address
```c
// ใช้ FF:FF:FF:FF:FF:FF สำหรับ broadcast
static uint8_t broadcast_mac[6] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};
```

### โครงสร้างข้อมูล Broadcasting
```c
typedef struct {
    char sender_id[20];
    char message[180];
    uint8_t message_type;  // 1=Info, 2=Command, 3=Alert
    uint32_t sequence_num;
    uint32_t timestamp;
} broadcast_data_t;
```

---

## การสร้าง Receivers

### Message Filtering
```c
void on_data_recv(const uint8_t *mac_addr, const uint8_t *data, int len) {
    broadcast_data_t *recv_data = (broadcast_data_t*)data;
    
    // ตรวจสอบ sequence number (ป้องกันการรับซ้ำ)
    if (is_duplicate_message(recv_data->sequence_num)) {
        return;
    }
    
    // ประมวลผลข้อความ
    process_broadcast_message(recv_data);
}
```

---

## การทดลอง

1. **Basic Broadcasting Test**
2. **Multiple Message Types**
3. **Range Testing**
4. **Performance Analysis**

---

**⏭️ พร้อมสำหรับ [Worksheet 4.2: Group Communication](Worksheet-4.2-Group-Communication.md)**