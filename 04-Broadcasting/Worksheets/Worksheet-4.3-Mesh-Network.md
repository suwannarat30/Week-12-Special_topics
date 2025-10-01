# Worksheet 4.3: Mesh Network
## การสร้าง Mesh Network และ Multi-hop Communication

### 🎯 วัตถุประสงค์
- เรียนรู้การสร้าง Mesh Network
- ทำความเข้าใจ Multi-hop Routing
- จัดการ Dynamic Peer Discovery
- สร้างระบบ Self-healing Network

---

## Mesh Network Architecture

### Node Structure
```c
typedef struct {
    uint8_t mac[6];
    char node_id[20];
    bool is_active;
    uint32_t last_seen;
} peer_info_t;
```

### Message Forwarding
```c
void forward_message(mesh_data_t *msg) {
    if (msg->hop_count < MAX_HOPS) {
        msg->hop_count++;
        esp_now_send(broadcast_mac, (uint8_t*)msg, sizeof(mesh_data_t));
    }
}
```

---

## การทดลอง

1. **3-Node Mesh Network**
2. **Message Routing**
3. **Network Self-healing**
4. **Performance Analysis**

---

**✅ เสร็จสิ้นการเรียนรู้บทที่ 4**