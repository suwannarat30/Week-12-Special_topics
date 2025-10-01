# บทที่ 4: การสื่อสารแบบ Broadcasting และ Group Communication
## ESP-NOW One-to-Many และ Many-to-Many

![Broadcasting Communication](https://docs.espressif.com/projects/esp-idf/en/latest/_images/esp-now-one-to-many.png)

ในบทนี้เราจะเรียนรู้การสื่อสารแบบ **Broadcasting** และ **Group Communication** ที่ ESP32 หนึ่งตัวสามารถส่งข้อมูลไปยังหลายตัวพร้อมกัน

### เป้าหมายการเรียนรู้:
- สร้างระบบ One-to-Many Communication  
- ใช้งาน Broadcast Address
- สร้างระบบ Group Communication with IDs
- จัดการ Multiple Peers
- ส่งข้อมูลแบบ Multicast

---

## ทฤษฎี: Broadcasting vs Group Communication

### 1. Broadcasting (One-to-Many)
```
       Broadcaster (Master)
            |
    +-------+-------+
    |       |       |
Receiver1 Receiver2 Receiver3
```

- **Broadcast Address**: `FF:FF:FF:FF:FF:FF`
- ส่งข้อมูลไปยังทุกอุปกรณ์ที่อยู่ในระยะ
- ไม่ต้องรู้ MAC Address ของ Receiver

### 2. Group Communication (Many-to-Many)
```
    Node A ←→ Node B
      ↕        ↕
    Node D ←→ Node C
```

- ใช้ **Group ID** หรือ **Channel ID**
- แต่ละโหนดสามารถส่งและรับได้
- จัดการ Peers แบบ Dynamic

### 3. Multicast (Selective Broadcasting)
```
Controller → [Group 1: A, B]
          → [Group 2: C, D]
```

- ส่งไปยังกลุ่มเฉพาะ
- ใช้ Group ID หรือ Message Type filtering

---

## Broadcasting Implementation

### การใช้ Broadcast Address

```c
// Broadcast Address (ส่งให้ทุกคน)
static uint8_t broadcast_mac[6] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF};

// เพิ่ม Broadcast Peer
esp_now_peer_info_t peer_info = {};
memcpy(peer_info.peer_addr, broadcast_mac, 6);
peer_info.channel = 0;
peer_info.encrypt = false;
ESP_ERROR_CHECK(esp_now_add_peer(&peer_info));
```

### โครงสร้างข้อมูล Broadcasting

```c
typedef struct {
    char sender_id[20];
    char message[180];
    uint8_t message_type;  // 1=Info, 2=Command, 3=Alert
    uint8_t group_id;      // 0=All, 1=Group1, 2=Group2
    uint32_t sequence_num;
    uint32_t timestamp;
} broadcast_data_t;
```

### การส่ง Broadcast Message

```c
void send_broadcast(const char* message, uint8_t msg_type, uint8_t group_id) {
    broadcast_data_t broadcast_data;
    
    strcpy(broadcast_data.sender_id, "MASTER_001");
    strncpy(broadcast_data.message, message, sizeof(broadcast_data.message) - 1);
    broadcast_data.message_type = msg_type;
    broadcast_data.group_id = group_id;
    broadcast_data.sequence_num = ++sequence_counter;
    broadcast_data.timestamp = esp_timer_get_time() / 1000;
    
    ESP_LOGI(TAG, "📡 Broadcasting [Type:%d, Group:%d]: %s", 
             msg_type, group_id, message);
    
    esp_now_send(broadcast_mac, (uint8_t*)&broadcast_data, sizeof(broadcast_data));
}
```

---

## Group Communication

### Group Filtering

```c
#define MY_GROUP_ID 1  // แต่ละ node มี Group ID ต่างกัน

void on_data_recv(const uint8_t *mac_addr, const uint8_t *data, int len) {
    broadcast_data_t *recv_data = (broadcast_data_t*)data;
    
    // ตรวจสอบว่าข้อความนี้สำหรับเราหรือไม่
    bool for_me = (recv_data->group_id == 0) || (recv_data->group_id == MY_GROUP_ID);
    
    if (!for_me) {
        ESP_LOGI(TAG, "📋 Message for Group %d (not for me)", recv_data->group_id);
        return;
    }
    
    // ประมวลผลข้อความ
    process_message(recv_data);
}
```

### การจัดการ Multiple Groups

```c
typedef struct {
    uint8_t group_id;
    char group_name[20];
    bool is_member;
} group_info_t;

group_info_t groups[] = {
    {1, "Sensors", true},
    {2, "Controllers", false},
    {3, "Displays", true}
};

bool is_group_member(uint8_t group_id) {
    for (int i = 0; i < sizeof(groups)/sizeof(groups[0]); i++) {
        if (groups[i].group_id == group_id && groups[i].is_member) {
            return true;
        }
    }
    return false;
}
```

---

## Mesh Network Implementation

### Basic Mesh Node

```c
typedef struct {
    uint8_t mac[6];
    char node_id[20];
    bool is_active;
    uint32_t last_seen;
} peer_info_t;

#define MAX_PEERS 10
static peer_info_t known_peers[MAX_PEERS];
static int peer_count = 0;

// เพิ่ม Peer แบบ Dynamic
void add_peer(const uint8_t* mac, const char* node_id) {
    // ตรวจสอบว่า peer นี้มีอยู่แล้วหรือไม่
    int index = find_peer_by_mac(mac);
    
    if (index >= 0) {
        // Update existing peer
        known_peers[index].last_seen = esp_timer_get_time() / 1000;
    } else if (peer_count < MAX_PEERS) {
        // Add new peer
        memcpy(known_peers[peer_count].mac, mac, 6);
        strcpy(known_peers[peer_count].node_id, node_id);
        known_peers[peer_count].is_active = true;
        known_peers[peer_count].last_seen = esp_timer_get_time() / 1000;
        
        // เพิ่มใน ESP-NOW peer list
        esp_now_peer_info_t peer_info = {};
        memcpy(peer_info.peer_addr, mac, 6);
        peer_info.channel = 0;
        peer_info.encrypt = false;
        esp_now_add_peer(&peer_info);
        
        peer_count++;
    }
}
```

### Message Forwarding

```c
typedef struct {
    char sender_id[20];
    char target_id[20];     // "ALL" สำหรับ broadcast
    char message[150];
    uint8_t hop_count;      // นับจำนวน hop
    uint32_t sequence_num;
} mesh_data_t;

void on_mesh_recv(const uint8_t *mac_addr, const uint8_t *data, int len) {
    mesh_data_t *recv_data = (mesh_data_t*)data;
    
    // เพิ่ม sender เป็น peer
    add_peer(mac_addr, recv_data->sender_id);
    
    // ตรวจสอบว่าข้อความนี้สำหรับเราหรือไม่
    bool for_me = (strcmp(recv_data->target_id, "ALL") == 0) || 
                  (strcmp(recv_data->target_id, MY_NODE_ID) == 0);
    
    if (for_me) {
        // ประมวลผลข้อความ
        process_mesh_message(recv_data);
    } else {
        // Forward message (ถ้า hop count ยังไม่เกินขมีด)
        if (recv_data->hop_count < MAX_HOPS) {
            recv_data->hop_count++;
            esp_now_send(broadcast_mac, (uint8_t*)recv_data, sizeof(mesh_data_t));
        }
    }
}
```

---

## Performance และ Optimization

### Message Queue System

```c
#define MAX_QUEUE_SIZE 20

typedef struct {
    uint8_t mac[6];
    uint8_t data[250];
    size_t data_len;
    uint32_t timestamp;
    uint8_t retry_count;
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

### Duplicate Detection

```c
#define SEQUENCE_CACHE_SIZE 50
static uint32_t received_sequences[SEQUENCE_CACHE_SIZE];
static int sequence_index = 0;

bool is_duplicate_message(uint32_t sequence) {
    for (int i = 0; i < SEQUENCE_CACHE_SIZE; i++) {
        if (received_sequences[i] == sequence) {
            return true;
        }
    }
    
    // เพิ่ม sequence ใหม่
    received_sequences[sequence_index] = sequence;
    sequence_index = (sequence_index + 1) % SEQUENCE_CACHE_SIZE;
    return false;
}
```

---

## Use Cases และ Applications

### 1. Smart Home Automation
```
Gateway → [Living Room Group]  → Lights, AC, TV
        → [Bedroom Group]      → Lights, Fan
        → [Kitchen Group]      → Appliances
```

### 2. Industrial IoT
```
Master Controller → [Zone A Sensors] → Temperature, Pressure
                 → [Zone B Sensors] → Vibration, Flow
                 → [All Zones]     → Emergency Stop
```

### 3. Vehicle Communication
```
Lead Vehicle → [Convoy] → Speed, Direction
            → [Safety] → Emergency Brake
```

---

## การแก้ไขปัญหา Broadcasting

### ปัญหาที่พบบ่อย:

1. **Message Flooding**
   - ใช้ Sequence number
   - จำกัด Broadcast rate
   - ใช้ Time-to-Live (TTL)

2. **Collision และ Interference**
   - ใช้ Random delay
   - จำกัดจำนวน nodes
   - เลือก Channel ที่เหมาะสม

3. **Power Consumption**
   - ใช้ Sleep modes
   - ปรับ Broadcast interval
   - เลือกเฉพาะ nodes ที่จำเป็น

---

## สรุป

Broadcasting และ Group Communication เป็นเทคนิคที่ทรงพลังสำหรับ:

- **การสื่อสารแบบ One-to-Many**
- **การสร้าง Mesh Networks**
- **การจัดกลุ่มอุปกรณ์**
- **การสร้าง Scalable systems**

สำคัญคือต้องจัดการ Performance และ Reliability อย่างเหมาะสม

---

## 📋 ใบงานปฏิบัติสำหรับบทนี้

1. **[Worksheet 4.1: Basic Broadcasting](Worksheets/Worksheet-4.1-Basic-Broadcasting.md)** - การสร้างระบบ Broadcasting พื้นฐาน
2. **[Worksheet 4.2: Group Communication](Worksheets/Worksheet-4.2-Group-Communication.md)** - การจัดกลุ่มและ Message filtering
3. **[Worksheet 4.3: Mesh Network](Worksheets/Worksheet-4.3-Mesh-Network.md)** - การสร้าง Mesh network และ Multi-hop communication

**➡️ ไปที่: [05-Troubleshooting](../05-Troubleshooting/05-Advanced-Troubleshooting.md)**