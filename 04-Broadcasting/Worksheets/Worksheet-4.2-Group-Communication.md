# Worksheet 4.2: Group Communication
## การจัดกลุ่มและ Message Filtering

### 🎯 วัตถุประสงค์
- เรียนรู้การสร้าง Group-based Communication
- ใช้ Group ID สำหรับ Message filtering
- จัดการ Multiple Groups
- สร้างระบบ Selective Broadcasting

---

## Group Management System

### Group Configuration
```c
#define MY_GROUP_ID 1  // เปลี่ยนในแต่ละ device

typedef struct {
    uint8_t group_id;
    char group_name[20];
    bool is_member;
} group_info_t;
```

### Message Filtering
```c
bool for_me = (recv_data->group_id == 0) || (recv_data->group_id == MY_GROUP_ID);
```

---

## การทดลอง

1. **Two-Group System**
2. **Selective Broadcasting**
3. **Dynamic Group Management**

---

**⏭️ พร้อมสำหรับ [Worksheet 4.3: Mesh Network](Worksheet-4.3-Mesh-Network.md)**