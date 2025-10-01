# บทเรียนที่ 3: การสร้างระบบผู้เล่น (Player Controller)

## สวัสดีครับ Mate!

ตอนนี้เราเข้าสู่บทเรียนที่ 3 เกี่ยวกับระบบผู้เล่นกันแล้ว เราจะสร้าง Player Controller ที่สมบูรณ์แบบขึ้นแบบ step by step พร้อมกับโค้ดตัวอย่างที่ใช้งานได้จริงเลย

## ภาพรวมของระบบผู้เล่น

ระบบผู้เล่นที่ดีควรมีส่วนประกอบหลักดังนี้:

- **การเคลื่อนที่**: WASD หรือ Arrow Keys
- **การกระโดด**: Spacebar หรือปุ่มกระโดด
- **การวิ่ง**: Left Shift เพื่อเพิ่มความเร็ว
- **การหมุน**: หมุนตามทิศทางการเคลื่อนที่หรือเมาส์
- **ตรวจสอบพื้น**: เพื่อให้กระโดดได้เฉพาะเมื่ออยู่บนพื้น
- **แรงโน้มถ่วง**: ทำให้ผู้เล่นตกลงมาอย่างเป็นธรรมชาติ

## ขั้นตอนการสร้าง (Step by Step)

### ขั้นตอนที่ 1: เตรียม Player Object

1. สร้าง Capsule หรือ 3D Model สำหรับเป็นตัวแทนผู้เล่น
2. ตั้งชื่อว่า "Player"
3. เพิ่ม Rigidbody Component
4. เพิ่ม Collider Component (Capsule Collider)

### ขั้นตอนที่ 2: สร้างโค้ด Player Controller

เราจะสร้าง Script ชื่อ `PlayerController` ด้วยโค้ดดังนี้:

```csharp
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    [Header("Movement Settings")]
    [SerializeField] private float walkSpeed = 5f;
    [SerializeField] private float runSpeed = 8f;
    [SerializeField] private float jumpForce = 5f;
    [SerializeField] private float rotationSpeed = 10f;

    [Header("Ground Check")]
    [SerializeField] private Transform groundCheck;
    [SerializeField] private float groundDistance = 0.4f;
    [SerializeField] private LayerMask groundMask;

    private Rigidbody rb;
    private bool isGrounded;
    private float currentSpeed;

    void Start()
    {
        rb = GetComponent<Rigidbody>();
        currentSpeed = walkSpeed;
    }

    void Update()
    {
        // ตรวจสอบว่าอยู่บนพื้นหรือไม่
        isGrounded = Physics.CheckSphere(groundCheck.position, groundDistance, groundMask);

        // รับ input การเคลื่อนที่
        float moveX = Input.GetAxis("Horizontal");
        float moveZ = Input.GetAxis("Vertical");

        // คำนวณทิศทางการเคลื่อนที่
        Vector3 movement = transform.right * moveX + transform.forward * moveZ;

        // ปรับความเร็วตามการวิ่ง
        if (Input.GetKey(KeyCode.LeftShift) && isGrounded)
        {
            currentSpeed = runSpeed;
        }
        else
        {
            currentSpeed = walkSpeed;
        }

        // เคลื่อนที่ผู้เล่น
        Vector3 velocity = movement * currentSpeed;
        velocity.y = rb.velocity.y; // เก็บความเร็วแนวตั้งเดิม
        rb.velocity = velocity;

        // การกระโดด
        if (Input.GetButtonDown("Jump") && isGrounded)
        {
            rb.AddForce(Vector3.up * jumpForce, ForceMode.Impulse);
        }

        // หมุนผู้เล่นตามทิศทางการเคลื่อนที่
        if (movement != Vector3.zero)
        {
            Quaternion targetRotation = Quaternion.LookRotation(movement);
            transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, rotationSpeed * Time.deltaTime);
        }
    }

    // แสดง Gizmo สำหรับ Ground Check ใน Scene View
    void OnDrawGizmosSelected()
    {
        if (groundCheck != null)
        {
            Gizmos.color = Color.red;
            Gizmos.DrawWireSphere(groundCheck.position, groundDistance);
        }
    }
}
```

### ขั้นตอนที่ 3: ตั้งค่าใน Inspector

1. แนบ Script ให้กับ Player GameObject
2. กำหนดค่าต่างๆ ใน Inspector:
   - Walk Speed: 5
   - Run Speed: 8
   - Jump Force: 5
   - Rotation Speed: 10
   - ลาก Ground Check Transform เข้ามา (สร้าง Empty GameObject ที่เท้าผู้เล่น)
   - กำหนด Ground Layer ให้กับ groundMask

### ขั้นตอนที่ 4: เพิ่มพื้นและสิ่งกีดขวาง

1. สร้าง Plane สำหรับเป็นพื้น
2. กำหนด Layer เป็น "Ground"
3. เพิ่ม Cube หรือสิ่งกีดขวางเพื่อทดสอบการเคลื่อนที่

## การควบคุม

- **WASD**: เคลื่อนที่
- **Space**: กระโดด
- **Left Shift**: วิ่ง
- **เมาส์**: หมุนกล้อง (ถ้ามี Camera Controller)

## คำถามสำหรับตรวจสอบความเข้าใจ

1. โค้ดส่วนไหนที่ตรวจสอบว่าผู้เล่นอยู่บนพื้น?
2. โค้ดส่วนไหนที่ทำให้ผู้เล่นกระโดด?
3. ถ้าอยากเพิ่มการหมุนด้วยเมาส์ต้องแก้ไขส่วนไหน?

---

**คุณเข้าใจบทเรียนนี้ดีไหมครับ?** ให้คะแนน 1-3 นะครับ:
- 1 = ยังงงๆ อยู่ อยากให้อธิบายเพิ่มเติม
- 2 = เข้าใจบ้างแต่ยังไม่ชัดทั้งหมด
- 3 = เข้าใจดีแล้ว พร้อมไปต่อ!

ถ้าพร้อม เราจะเพิ่มฟีเจอร์ขั้นสูงอย่าง Double Jump หรือ Wall Jump ในบทเรียนถัดไปครับ!
