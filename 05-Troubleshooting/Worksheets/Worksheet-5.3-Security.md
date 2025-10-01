# Worksheet 5.3: Security Implementation
## การเพิ่ม Security Features ให้กับ ESP-NOW

### 🎯 วัตถุประสงค์
- เรียนรู้การเพิ่ม Security ให้กับ ESP-NOW
- ใช้ Message Authentication
- ทำ Encryption/Decryption
- ป้องกัน Security threats

---

## Message Authentication

### HMAC Implementation
```c
#include "mbedtls/md.h"

static const char* AUTH_KEY = "MySecretKey2024!";

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
```

### Secure Message Structure
```c
typedef struct {
    char sender_id[20];
    char message[180];
    uint32_t timestamp;
    uint32_t sequence;
    uint8_t hash[32]; // SHA256 hash
} secure_message_t;
```

---

## Simple Encryption

### XOR Cipher
```c
void encrypt_decrypt_xor(uint8_t* data, size_t len, const char* key) {
    size_t key_len = strlen(key);
    
    for (size_t i = 0; i < len; i++) {
        data[i] ^= key[i % key_len];
    }
}

void send_encrypted_message(const uint8_t* target_mac, const char* message) {
    char encrypted_msg[200];
    strcpy(encrypted_msg, message);
    
    // เข้ารหัสข้อความ
    encrypt_decrypt_xor((uint8_t*)encrypted_msg, strlen(encrypted_msg), AUTH_KEY);
    
    esp_now_send(target_mac, (uint8_t*)encrypted_msg, strlen(message) + 1);
}
```

---

## Security Best Practices

### 1. Key Management
- ใช้ Unique keys สำหรับแต่ละ device
- เปลี่ยน keys เป็นระยะ
- เก็บ keys ใน secure storage

### 2. Message Validation
- ตรวจสอบ timestamp
- ใช้ sequence numbers
- Validate message format

### 3. Access Control
- ใช้ whitelist MAC addresses
- จำกัดสิทธิ์การเข้าถึง
- Monitor unauthorized access

---

## การทดลอง

### การทดลองที่ 1: Message Authentication
1. สร้างระบบ HMAC
2. ทดสอบ Authentication
3. วัดผลกระทบต่อ Performance

### การทดลองที่ 2: Encryption System
1. ใช้ XOR cipher
2. ทดสอบ Encryption/Decryption
3. วิเคราะห์ Security level

### การทดลองที่ 3: Security Monitoring
1. ตรวจจับ unauthorized messages
2. Log security events
3. สร้าง Alert system

---

**✅ เสร็จสิ้นการเรียนรู้ ESP-NOW ทั้งหมด!**

**🎉 ยินดีด้วย! คุณพร้อมที่จะพัฒนาโครงการ ESP-NOW ของคุณเองแล้ว**