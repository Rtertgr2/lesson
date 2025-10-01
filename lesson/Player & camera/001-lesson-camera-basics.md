# บทเรียนที่ 001: พื้นฐานกล้องใน Unity (Camera Basics)

## วัตถุประสงค์ (Objectives)
เรียนรู้การทำงานพื้นฐานของกล้องใน Unity และการควบคุมมุมมองเกม

## ภาพรวมระบบ (System Overview)

กล้องใน Unity เป็นส่วนสำคัญที่:
- แสดงภาพเกมให้ผู้เล่นเห็น
- กำหนดมุมมองและมุมกล้อง
- ควบคุมการเคลื่อนไหวของกล้อง
- จัดการกับการชนและการซูม

## โครงสร้างระบบ (System Structure)

```
Camera System/
├── CameraController.cs     # ควบคุมการเคลื่อนที่กล้อง
├── CameraSettings.cs       # การตั้งค่ากล้อง
├── CameraCollision.cs      # จัดการการชนสิ่งกีดขวาง
└── CameraEffects.cs        # เอฟเฟคกล้องพิเศษ
```

## การติดตั้งใช้งาน (Implementation)

### 1. การสร้างกล้องพื้นฐาน (Basic Camera Setup)

```csharp
using UnityEngine;

public class BasicCameraController : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public Transform target;              // เป้าหมายที่กล้องจะติดตาม
    public Transform cameraTransform;     // Transform ของกล้อง

    [Header("การตั้งค่า - Settings")]
    public float distance = 5f;           // ระยะห่างจากเป้าหมาย
    public float height = 2f;             // ความสูงของกล้อง
    public float damping = 5f;            // ความนุ่มนวลในการเคลื่อนที่

    [Header("การควบคุมด้วยเมาส์ - Mouse Control")]
    public float rotationSpeed = 3f;      // ความเร็วการหมุน
    public float minAngle = -15f;         // มุมต่ำสุด
    public float maxAngle = 75f;          // มุมสูงสุด

    private float currentRotationX = 0f;
    private float currentRotationY = 0f;

    void Start()
    {
        // ตั้งค่ากล้องเริ่มต้น
        if (cameraTransform == null)
            cameraTransform = transform;

        // ตั้งมุมกล้องเริ่มต้น
        currentRotationY = cameraTransform.eulerAngles.y;
    }

    void LateUpdate()
    {
        if (target == null) return;

        // ควบคุมมุมกล้องด้วยเมาส์
        ควบคุมกล้องด้วยเมาส์();

        // คำนวณตำแหน่งกล้องใหม่
        Vector3 ตำแหน่งเป้าหมาย = คำนวณตำแหน่งกล้อง();

        // เคลื่อนกล้องอย่างนุ่มนวล
        cameraTransform.position = Vector3.Lerp(
            cameraTransform.position,
            ตำแหน่งเป้าหมาย,
            Time.deltaTime * damping
        );

        // หันกล้องไปยังเป้าหมาย
        cameraTransform.LookAt(target.position + Vector3.up * height);
    }

    /// <summary>
    /// ควบคุมมุมกล้องด้วยเมาส์
    /// Control camera angle with mouse
    /// </summary>
    private void ควบคุมกล้องด้วยเมาส์()
    {
        // การเคลื่อนที่ของเมาส์
        float mouseX = Input.GetAxis("Mouse X") * rotationSpeed;
        float mouseY = Input.GetAxis("Mouse Y") * rotationSpeed;

        // อัปเดตมุมการหมุน
        currentRotationY += mouseX;
        currentRotationX -= mouseY;

        // จำกัดมุมการหมุนแนวตั้ง
        currentRotationX = Mathf.Clamp(currentRotationX, minAngle, maxAngle);

        // หมุนกล้อง
        cameraTransform.rotation = Quaternion.Euler(currentRotationX, currentRotationY, 0);
    }

    /// <summary>
    /// คำนวณตำแหน่งที่กล้องควรอยู่
    /// Calculate where camera should be positioned
    /// </summary>
    private Vector3 คำนวณตำแหน่งกล้อง()
    {
        // คำนวณตำแหน่งจากมุมและระยะห่าง
        Vector3 ตำแหน่ง = new Vector3(0, 0, -distance);

        // หมุนตำแหน่งตามมุมกล้อง
        ตำแหน่ง = Quaternion.Euler(currentRotationX, currentRotationY, 0) * ตำแหน่ง;

        // บวกกับตำแหน่งของเป้าหมาย
        return target.position + ตำแหน่ง + Vector3.up * height;
    }
}
```

### 2. การตั้งค่ากล้อง (Camera Settings)

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "CameraSettings", menuName = "Settings/Camera Settings")]
public class CameraSettings : ScriptableObject
{
    [Header("การเคลื่อนที่ - Movement")]
    public float followSpeed = 5f;        // ความเร็วในการติดตาม
    public float rotationSpeed = 3f;      // ความเร็วการหมุน
    public float damping = 5f;            // ความนุ่มนวล

    [Header("ระยะห่างและมุม - Distance & Angles")]
    public float defaultDistance = 5f;    // ระยะห่างปกติ
    public float minDistance = 2f;        // ระยะห่างต่ำสุด
    public float maxDistance = 10f;       // ระยะห่างสูงสุด
    public float height = 2f;             // ความสูง
    public float minAngle = -15f;         // มุมต่ำสุด
    public float maxAngle = 75f;          // มุมสูงสุด

    [Header("การซูม - Zoom")]
    public float zoomSpeed = 2f;          // ความเร็วการซูม
    public bool enableZoom = true;        // เปิดใช้งานการซูม

    [Header("การหลบสิ่งกีดขวาง - Collision Avoidance")]
    public bool enableCollision = true;   // เปิดใช้งานการหลบสิ่งกีดขวาง
    public float collisionRadius = 0.3f;  // รัศมีการตรวจสอบการชน
    public LayerMask collisionLayers;     // เลเยอร์ที่ตรวจสอบการชน
}
```

### 3. การจัดการการชน (Camera Collision)

```csharp
using UnityEngine;

public class CameraCollisionHandler : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public Transform target;              // เป้าหมายที่ติดตาม
    public CameraSettings settings;       // การตั้งค่ากล้อง

    [Header("สถานะ - Status")]
    private float adjustedDistance;       // ระยะห่างที่ปรับแล้ว
    private Vector3 desiredPosition;      // ตำแหน่งที่ต้องการ

    void Start()
    {
        adjustedDistance = settings.defaultDistance;
    }

    void LateUpdate()
    {
        if (target == null || !settings.enableCollision) return;

        // คำนวณตำแหน่งที่ต้องการ
        desiredPosition = คำนวณตำแหน่งที่ต้องการ();

        // ตรวจสอบการชนและปรับระยะห่าง
        adjustedDistance = ตรวจสอบและปรับระยะห่าง(desiredPosition, settings.defaultDistance);

        // ตั้งตำแหน่งกล้องใหม่
        transform.position = desiredPosition.normalized * adjustedDistance;
        transform.LookAt(target.position + Vector3.up * settings.height);
    }

    /// <summary>
    /// คำนวณตำแหน่งที่กล้องต้องการอยู่
    /// Calculate desired camera position
    /// </summary>
    private Vector3 คำนวณตำแหน่งที่ต้องการ()
    {
        // คำนวณทิศทางจากเป้าหมายไปยังกล้อง
        Vector3 direction = (transform.position - target.position).normalized;
        return direction * settings.defaultDistance;
    }

    /// <summary>
    /// ตรวจสอบและปรับระยะห่างกล้อง
    /// Check and adjust camera distance
    /// </summary>
    private float ตรวจสอบและปรับระยะห่าง(Vector3 desiredPosition, float originalDistance)
    {
        float adjustedDistance = originalDistance;

        // สร้าง Ray จากเป้าหมายไปยังตำแหน่งที่ต้องการ
        Ray ray = new Ray(target.position, desiredPosition.normalized);

        // ตรวจสอบการชน
        if (Physics.SphereCast(ray, settings.collisionRadius, out RaycastHit hit, originalDistance, settings.collisionLayers))
        {
            // ปรับระยะห่างให้ใกล้ขึ้นเมื่อชนสิ่งกีดขวาง
            adjustedDistance = hit.distance - settings.collisionRadius;

            // จำกัดระยะห่างไม่ให้ต่ำกว่าค่าต่ำสุด
            adjustedDistance = Mathf.Max(adjustedDistance, settings.minDistance);
        }

        // กลับมาระยะห่างปกติอย่างนุ่มนวลเมื่อไม่ชน
        if (adjustedDistance >= originalDistance - 0.1f)
        {
            adjustedDistance = Mathf.Lerp(adjustedDistance, originalDistance, Time.deltaTime * 2f);
        }

        return adjustedDistance;
    }
}
```

### 4. การควบคุมการซูม (Zoom Control)

```csharp
using UnityEngine;

public class CameraZoomController : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public Camera mainCamera;             // กล้องหลัก
    public CameraSettings settings;       // การตั้งค่ากล้อง

    [Header("สถานะการซูม - Zoom Status")]
    private float currentDistance;        // ระยะห่างปัจจุบัน
    private float targetDistance;         // ระยะห่างเป้าหมาย

    void Start()
    {
        currentDistance = settings.defaultDistance;
        targetDistance = currentDistance;
    }

    void Update()
    {
        if (!settings.enableZoom) return;

        // ควบคุมการซูมด้วยเมาส์วีล
        float scrollInput = Input.GetAxis("Mouse ScrollWheel");

        if (Mathf.Abs(scrollInput) > 0.01f)
        {
            targetDistance -= scrollInput * settings.zoomSpeed;
            targetDistance = Mathf.Clamp(targetDistance, settings.minDistance, settings.maxDistance);
        }

        // ปรับระยะห่างอย่างนุ่มนวล
        currentDistance = Mathf.Lerp(currentDistance, targetDistance, Time.deltaTime * settings.zoomSpeed);

        // อัปเดต Field of View หรือระยะห่างของกล้อง
        if (mainCamera.orthographic)
        {
            mainCamera.orthographicSize = currentDistance;
        }
        else
        {
            // สำหรับกล้อง Perspective ให้ปรับตำแหน่งแทน
            Vector3 direction = (transform.position - mainCamera.transform.position).normalized;
            mainCamera.transform.position = transform.position - direction * currentDistance;
        }
    }
}
```

## ประเภทของกล้องใน Unity (Camera Types)

### 1. Perspective Camera (กล้องแบบมุมมอง)
- เหมือนตา manusia
- มี Field of View
- วัตถุที่อยู่ไกลจะดูเล็กลง

### 2. Orthographic Camera (กล้องแบบตรง)
- มุมมองแบบ 2D
- ไม่มี Perspective
- ขนาดวัตถุคงที่ไม่ว่าอยู่ใกล้ไกล

## การควบคุมกล้องขั้นสูง (Advanced Camera Controls)

### การติดตามผู้เล่น (Player Following)

```csharp
public class CameraFollow : MonoBehaviour
{
    public Transform player;              // ผู้เล่นที่ติดตาม
    public Vector3 offset = new Vector3(0, 5, -10); // ออฟเซ็ตจากผู้เล่น
    public float smoothSpeed = 0.125f;    // ความนุ่มนวล

    void LateUpdate()
    {
        if (player == null) return;

        // คำนวณตำแหน่งเป้าหมาย
        Vector3 desiredPosition = player.position + offset;

        // ใช้ Lerp สำหรับการเคลื่อนที่ที่นุ่มนวล
        Vector3 smoothedPosition = Vector3.Lerp(transform.position, desiredPosition, smoothSpeed);

        transform.position = smoothedPosition;
        transform.LookAt(player.position);
    }
}
```

### การสลับมุมกล้อง (Camera Switching)

```csharp
public class CameraSwitcher : MonoBehaviour
{
    public Camera[] cameras;              // กล้องทั้งหมดในเกม
    public int currentCameraIndex = 0;    // กล้องที่ใช้งานอยู่

    void Start()
    {
        // ปิดกล้องทั้งหมดยกเว้นกล้องแรก
        for (int i = 0; i < cameras.Length; i++)
        {
            cameras[i].gameObject.SetActive(i == currentCameraIndex);
        }
    }

    void Update()
    {
        // สลับกล้องด้วยปุ่ม C
        if (Input.GetKeyDown(KeyCode.C))
        {
            สลับกล้องถัดไป();
        }
    }

    /// <summary>
    /// สลับไปยังกล้องถัดไป
    /// Switch to next camera
    /// </summary>
    public void สลับกล้องถัดไป()
    {
        // ปิดกล้องปัจจุบัน
        cameras[currentCameraIndex].gameObject.SetActive(false);

        // เปลี่ยนไปกล้องถัดไป
        currentCameraIndex = (currentCameraIndex + 1) % cameras.Length;

        // เปิดกล้องใหม่
        cameras[currentCameraIndex].gameObject.SetActive(true);

        Debug.Log($"สลับกล้องเป็น: {cameras[currentCameraIndex].name}");
    }
}
```

## การตั้งค่าใน Unity Editor (Editor Setup)

### ขั้นตอนการสร้าง (Creation Steps)

1. **สร้าง Camera** ในซีน
2. **ตั้งค่า Projection**:
   - Perspective สำหรับ 3D
   - Orthographic สำหรับ 2D
3. **ปรับ Clipping Planes**:
   - Near: 0.1
   - Far: 1000
4. **เพิ่ม Components**:
   - CameraController.cs
   - CameraCollisionHandler.cs
   - CameraZoomController.cs

## การทดสอบระบบ (Testing the System)

### การควบคุมการทดสอบ (Test Controls)

- **เมาส์** - หมุนมุมกล้อง
- **Mouse Wheel** - ซูมเข้า/ออก
- **WASD** - เคลื่อนที่ผู้เล่น (ถ้ามี)
- **C** - สลับกล้อง (ถ้ามีหลายกล้อง)

### สิ่งที่ควรสังเกต (What to Observe)

- **การติดตามผู้เล่น** - กล้องเคลื่อนที่ตามผู้เล่น
- **การหลบสิ่งกีดขวาง** - กล้องปรับระยะห่างอัตโนมัติ
- **การซูม** - สามารถซูมเข้าใกล้/ออกห่างได้
- **มุมมอง** - มองเห็นเกมจากมุมที่เหมาะสม

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **ควบคุมง่าย** - ใช้เมาส์และคีย์บอร์ด
- **ปรับแต่งได้** - เปลี่ยนค่าต่างๆ ได้ง่าย
- **หลบสิ่งกีดขวาง** - ไม่ติดกับสิ่งกีดขวาง
- **นุ่มนวล** - การเคลื่อนที่ไม่กระตุก

### ⚠️ ข้อควรระวัง
- **ประสิทธิภาพ** - การตรวจสอบการชนอาจกินทรัพยากร
- **การตั้งค่า** - ต้องปรับค่าต่างๆ ให้เหมาะกับเกม
- **ความเข้ากันได้** - ทดสอบกับอุปกรณ์และหน้าจอต่างๆ

## แบบฝึกหัด (Exercise)

1. **สร้างกล้องติดตามผู้เล่น** ที่นุ่มนวล
2. **เพิ่มระบบซูม** ด้วยเมาส์วีล
3. **สร้างระบบหลบสิ่งกีดขวาง** ให้กล้องไม่ติดกำแพง
4. **เพิ่มการสลับมุมกล้อง** สำหรับมุมมองต่างๆ

**คุณเข้าใจพื้นฐานกล้องใน Unity นี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนเรื่องกล้องบุคคลที่ 3 กันต่อ! 🚀
