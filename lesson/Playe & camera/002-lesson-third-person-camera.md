# บทเรียนที่ 2: การสร้างกล้องบุคคลที่ 3 (Third Person Camera)

## สวัสดีครับ Mate!

ตอนนี้เราเข้าใจพื้นฐานกล้องแล้ว เรามาเริ่มสร้างระบบกล้องบุคคลที่ 3 กันจริงๆ เลยครับ บทเรียนนี้เราจะเรียนรู้ทีละขั้นตอนพร้อมกับโค้ดตัวอย่างที่คุณสามารถนำไปใช้ได้เลย

## ภาพรวมของระบบกล้องบุคคลที่ 3

กล้องบุคคลที่ 3 ที่ดีควรมีฟีเจอร์หลักๆ ดังนี้:
- **ติดตามผู้เล่น**: กล้องเคลื่อนตามตำแหน่งผู้เล่น
- **หมุนด้วยเมาส์**: ผู้เล่นสามารถหมุนกล้องรอบตัวละครได้
- **ระยะห่างคงที่**: กล้องรักษาระยะห่างจากผู้เล่น
- **หลบสิ่งกีดขวาง**: กล้องปรับตำแหน่งอัตโนมัติเมื่อมีสิ่งกีดขวาง

## ขั้นตอนการสร้าง (Step by Step)

### ขั้นตอนที่ 1: เตรียม Scene

1. สร้าง GameObject ว่างเปล่า ตั้งชื่อว่า "ThirdPersonCamera"
2. สร้าง Capsule หรือ 3D Model สำหรับเป็นตัวแทนผู้เล่น ตั้งชื่อว่า "Player"
3. สร้างพื้น (Plane) ให้ผู้เล่นยืน

### ขั้นตอนที่ 2: สร้างโค้ดกล้อง

เราจะสร้าง Script ชื่อ `ThirdPersonCamera` ด้วยโค้ดดังนี้:

```csharp
using UnityEngine;

public class ThirdPersonCamera : MonoBehaviour
{
    // ตัวแปรสำหรับอ้างอิงผู้เล่น
    [SerializeField] private Transform target;

    // ตัวแปรสำหรับปรับแต่งกล้อง
    [SerializeField] private float distance = 5f;        // ระยะห่างจากผู้เล่น
    [SerializeField] private float height = 2f;          // ความสูงของกล้อง
    [SerializeField] private float damping = 5f;         // ความนุ่มนวลในการเคลื่อนที่
    [SerializeField] private float rotationSpeed = 3f;   // ความเร็วในการหมุน

    // ตัวแปรภายในสำหรับเก็บข้อมูลการหมุน
    private float currentRotationAngle = 0f;
    private float currentHeight = 0f;

    void Update()
    {
        // รับ input จากเมาส์สำหรับหมุนกล้อง
        currentRotationAngle += Input.GetAxis("Mouse X") * rotationSpeed;

        // คำนวณตำแหน่งและการหมุนของกล้อง
        CalculateCameraPosition();
    }

    void CalculateCameraPosition()
    {
        if (target == null) return;

        // คำนวณตำแหน่งที่ต้องการให้กล้องอยู่
        Vector3 desiredPosition = target.position;
        desiredPosition.y += height;

        // คำนวณการหมุนรอบผู้เล่น
        Quaternion currentRotation = Quaternion.Euler(0, currentRotationAngle, 0);
        desiredPosition = target.position + (currentRotation * Vector3.back * distance);

        // เคลื่อนกล้องไปยังตำแหน่งที่ต้องการแบบนุ่มนวล
        transform.position = Vector3.Lerp(transform.position, desiredPosition, Time.deltaTime * damping);
        transform.LookAt(target.position + Vector3.up * height);
    }
}
```

### ขั้นตอนที่ 3: นำโค้ดไปใช้

1. แนบ Script นี้ให้กับ GameObject "ThirdPersonCamera"
2. ใน Inspector กำหนด "Player" ให้กับช่อง Target
3. ปรับค่าต่างๆ ตามต้องการ:
   - Distance: ระยะห่างจากผู้เล่น (เริ่มจาก 5)
   - Height: ความสูงของกล้อง (เริ่มจาก 2)
   - Damping: ความนุ่มนวล (เริ่มจาก 5)
   - Rotation Speed: ความเร็วในการหมุน (เริ่มจาก 3)

### ขั้นตอนที่ 4: เพิ่มการเคลื่อนที่ให้ผู้เล่น

สร้าง Script ชื่อ `PlayerMovement` สำหรับให้ผู้เล่นเคลื่อนที่ได้:

```csharp
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    [SerializeField] private float speed = 5f;

    void Update()
    {
        float moveHorizontal = Input.GetAxis("Horizontal");
        float moveVertical = Input.GetAxis("Vertical");

        Vector3 movement = new Vector3(moveHorizontal, 0f, moveVertical);
        transform.Translate(movement * speed * Time.deltaTime);
    }
}
```

## การทดสอบ

1. กด Play ใน Unity
2. ใช้ WASD เพื่อเคลื่อนที่ผู้เล่น
3. ใช้เมาส์เพื่อหมุนกล้องรอบตัวละคร
4. ลองปรับค่าต่างๆ ใน Inspector เพื่อดูผลที่แตกต่าง

## คำถามสำหรับตรวจสอบความเข้าใจ

หลังจากที่คุณลองทำตามแล้ว ลองตอบคำถามเหล่านี้ดูครับ:

1. โค้ดส่วนไหนที่รับ input จากเมาส์?
2. โค้ดส่วนไหนที่คำนวณตำแหน่งกล้อง?
3. ถ้าอยากให้กล้องซูมเข้า-ออกได้ ต้องแก้ไขส่วนไหน?

---

**คุณเข้าใจบทเรียนนี้ดีไหมครับ?** ให้คะแนน 1-3 นะครับ:
- 1 = ยังงงๆ อยู่ อยากให้อธิบายเพิ่มเติม
- 2 = เข้าใจบ้างแต่ยังไม่ชัดทั้งหมด
- 3 = เข้าใจดีแล้ว พร้อมไปต่อ!

ถ้าพร้อม เราจะเพิ่มฟีเจอร์ขั้นสูงอย่างการหลบสิ่งกีดขวางในบทเรียนถัดไปครับ!
