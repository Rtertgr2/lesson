# บทเรียนที่ 002: ระบบกล้องบุคคลที่ 3 (Third-Person Camera System)

## วัตถุประสงค์ (Objectives)
เรียนรู้การสร้างระบบกล้องบุคคลที่ 3 ที่ลื่นไหลและใช้งานง่าย

## ภาพรวมระบบ (System Overview)

ระบบกล้องบุคคลที่ 3 ที่ดีควรมี:
- การติดตามผู้เล่นอย่างนุ่มนวล
- การหลบสิ่งกีดขวางอัตโนมัติ
- การควบคุมด้วยเมาส์ที่ตอบสนองดี
- การซูมเข้า-ออกได้
- มุมกล้องที่ปรับได้หลายแบบ

## โครงสร้างระบบขั้นสูง (Advanced System Structure)

```
Third-Person Camera/
├── TPCameraController.cs       # ควบคุมกล้องหลัก
├── CameraStateManager.cs       # จัดการสถานะกล้อง
├── CameraCollisionSystem.cs    # ระบบหลบสิ่งกีดขวาง
├── CameraAnimation.cs          # อนิเมชันกล้อง
└── CameraInputHandler.cs       # จัดการอินพุต
```

## การติดตั้งใช้งานขั้นสูง (Advanced Implementation)

### 1. Third-Person Camera Controller

```csharp
using UnityEngine;

public class TPCameraController : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public Transform player;                    // ผู้เล่นที่ติดตาม
    public CameraSettings cameraSettings;       // การตั้งค่ากล้อง
    public CameraStateManager stateManager;     // ผู้จัดการสถานะ

    [Header("ตำแหน่งกล้อง - Camera Position")]
    private Vector3 cameraOffset;               // ออฟเซ็ตกล้อง
    private float currentDistance;              // ระยะห่างปัจจุบัน
    private float targetDistance;               // ระยะห่างเป้าหมาย

    [Header("การหมุนกล้อง - Camera Rotation")]
    private float yaw;                          // การหมุนแนวนอน
    private float pitch;                        // การหมุนแนวตั้ง
    private float targetYaw;                    // เป้าหมายการหมุนแนวนอน
    private float targetPitch;                  // เป้าหมายการหมุนแนวตั้ง

    [Header("การเคลื่อนที่ - Movement")]
    private Vector3 velocity = Vector3.zero;    // ความเร็วการเคลื่อนที่
    private Vector3 angularVelocity = Vector3.zero; // ความเร็วการหมุน

    void Start()
    {
        กำหนดค่าเริ่มต้น();
        ตั้งตำแหน่งกล้องเริ่มต้น();
    }

    void LateUpdate()
    {
        จัดการอินพุต();
        อัปเดตตำแหน่งและมุมกล้อง();
        ใช้การชนเพื่อปรับตำแหน่ง();
    }

    /// <summary>
    /// กำหนดค่าต่างๆ ในตอนเริ่มต้น
    /// Initialize starting values
    /// </summary>
    private void กำหนดค่าเริ่มต้น()
    {
        currentDistance = cameraSettings.defaultDistance;
        targetDistance = currentDistance;

        // ตั้งค่ามุมเริ่มต้นจากตำแหน่งปัจจุบัน
        Vector3 angles = transform.eulerAngles;
        yaw = angles.y;
        pitch = angles.x;

        targetYaw = yaw;
        targetPitch = pitch;

        cameraOffset = new Vector3(0, cameraSettings.height, 0);
    }

    /// <summary>
    /// ตั้งตำแหน่งกล้องเริ่มต้น
    /// Set initial camera position
    /// </summary>
    private void ตั้งตำแหน่งกล้องเริ่มต้น()
    {
        if (player == null) return;

        // คำนวณตำแหน่งเริ่มต้น
        Vector3 targetPosition = player.position + cameraOffset;
        Vector3 cameraPosition = targetPosition - transform.forward * currentDistance;

        transform.position = cameraPosition;
        transform.LookAt(targetPosition);
    }

    /// <summary>
    /// จัดการอินพุตจากผู้เล่น
    /// Handle player input
    /// </summary>
    private void จัดการอินพุต()
    {
        if (cameraSettings == null) return;

        // การหมุนกล้องด้วยเมาส์
        float mouseX = Input.GetAxis("Mouse X") * cameraSettings.rotationSpeed;
        float mouseY = Input.GetAxis("Mouse Y") * cameraSettings.rotationSpeed;

        targetYaw += mouseX;
        targetPitch -= mouseY;
        targetPitch = Mathf.Clamp(targetPitch, cameraSettings.minAngle, cameraSettings.maxAngle);

        // การซูมด้วยเมาส์วีล
        float scrollWheel = Input.GetAxis("Mouse ScrollWheel");
        if (Mathf.Abs(scrollWheel) > 0.01f)
        {
            targetDistance -= scrollWheel * cameraSettings.zoomSpeed;
            targetDistance = Mathf.Clamp(targetDistance, cameraSettings.minDistance, cameraSettings.maxDistance);
        }
    }

    /// <summary>
    /// อัปเดตตำแหน่งและมุมกล้อง
    /// Update camera position and rotation
    /// </summary>
    private void อัปเดตตำแหน่งและมุมกล้อง()
    {
        // อัปเดตระยะห่างอย่างนุ่มนวล
        currentDistance = Mathf.SmoothDamp(currentDistance, targetDistance, ref velocity.z, cameraSettings.damping * Time.deltaTime);

        // อัปเดตมุมการหมุนอย่างนุ่มนวล
        yaw = Mathf.SmoothDamp(yaw, targetYaw, ref angularVelocity.y, cameraSettings.damping * Time.deltaTime);
        pitch = Mathf.SmoothDamp(pitch, targetPitch, ref angularVelocity.x, cameraSettings.damping * Time.deltaTime);

        // คำนวณตำแหน่งใหม่
        Vector3 targetPosition = player.position + cameraOffset;
        Vector3 direction = Quaternion.Euler(pitch, yaw, 0) * Vector3.back;

        Vector3 desiredPosition = targetPosition + direction * currentDistance;

        // เคลื่อนกล้องอย่างนุ่มนวล
        transform.position = Vector3.SmoothDamp(transform.position, desiredPosition, ref velocity, cameraSettings.damping * Time.deltaTime);

        // หันกล้องไปยังเป้าหมาย
        transform.LookAt(targetPosition);
    }

    /// <summary>
    /// ใช้การชนเพื่อปรับตำแหน่งกล้อง
    /// Use collision to adjust camera position
    /// </summary>
    private void ใช้การชนเพื่อปรับตำแหน่ง()
    {
        if (!cameraSettings.enableCollision || player == null) return;

        Vector3 targetPosition = player.position + cameraOffset;
        Vector3 direction = (transform.position - targetPosition).normalized;
        float distance = Vector3.Distance(transform.position, targetPosition);

        // ตรวจสอบการชน
        if (Physics.Raycast(targetPosition, direction, out RaycastHit hit, distance, cameraSettings.collisionLayers))
        {
            // ปรับตำแหน่งกล้องให้อยู่ใกล้ขึ้น
            float adjustedDistance = hit.distance - 0.1f;
            Vector3 adjustedPosition = targetPosition + direction * adjustedDistance;

            transform.position = Vector3.Lerp(transform.position, adjustedPosition, Time.deltaTime * cameraSettings.damping);
        }
    }
}
```

### 2. Camera State Manager

```csharp
using UnityEngine;

public class CameraStateManager : MonoBehaviour
{
    [Header("สถานะกล้อง - Camera States")]
    public CameraState currentState;           // สถานะปัจจุบัน
    public CameraState defaultState;           // สถานะปกติ
    public CameraState combatState;            // สถานะต่อสู้
    public CameraState cinematicState;         // สถานะภาพยนตร์

    [Header("การเปลี่ยนสถานะ - State Transition")]
    public float transitionSpeed = 2f;         // ความเร็วการเปลี่ยน
    private CameraState targetState;           // สถานะเป้าหมาย

    void Start()
    {
        currentState = defaultState;
        targetState = currentState;
    }

    void Update()
    {
        อัปเดตสถานะกล้อง();
        จัดการการเปลี่ยนสถานะ();
    }

    /// <summary>
    /// อัปเดตสถานะกล้องปัจจุบัน
    /// Update current camera state
    /// </summary>
    private void อัปเดตสถานะกล้อง()
    {
        if (currentState == null) return;

        // อัปเดตตำแหน่งและมุมกล้องตามสถานะปัจจุบัน
        currentState.UpdateCamera(transform, player);
    }

    /// <summary>
    /// จัดการการเปลี่ยนสถานะกล้อง
    /// Handle camera state transitions
    /// </summary>
    private void จัดการการเปลี่ยนสถานะ()
    {
        if (targetState == null || currentState == targetState) return;

        // เปลี่ยนสถานะอย่างนุ่มนวล
        currentState = CameraState.Lerp(currentState, targetState, Time.deltaTime * transitionSpeed);

        // ตรวจสอบว่าเปลี่ยนสถานะเสร็จหรือยัง
        if (Vector3.Distance(currentState.position, targetState.position) < 0.1f)
        {
            currentState = targetState;
        }
    }

    /// <summary>
    /// เปลี่ยนไปยังสถานะใหม่
    /// Switch to new state
    /// </summary>
    public void เปลี่ยนสถานะ(CameraState newState)
    {
        if (newState == null) return;

        targetState = newState;
        Debug.Log($"เปลี่ยนสถานะกล้องเป็น: {newState.stateName}");
    }

    /// <summary>
    /// เปลี่ยนไปยังสถานะปกติ
    /// Switch to default state
    /// </summary>
    public void เปลี่ยนเป็นสถานะปกติ()
    {
        เปลี่ยนสถานะ(defaultState);
    }

    /// <summary>
    /// เปลี่ยนไปยังสถานะต่อสู้
    /// Switch to combat state
    /// </summary>
    public void เปลี่ยนเป็นสถานะต่อสู้()
    {
        เปลี่ยนสถานะ(combatState);
    }
}

/// <summary>
/// คลาสสำหรับเก็บข้อมูลสถานะกล้อง
/// Class for storing camera state data
/// </summary>
[System.Serializable]
public class CameraState
{
    public string stateName;              // ชื่อสถานะ
    public Vector3 position;              // ตำแหน่ง
    public Vector3 rotation;              // การหมุน
    public float fieldOfView = 60f;       // มุมมอง
    public float distance = 5f;           // ระยะห่าง

    /// <summary>
    /// อัปเดตกล้องตามสถานะนี้
    /// Update camera with this state
    /// </summary>
    public void UpdateCamera(Transform cameraTransform, Transform player)
    {
        if (player == null) return;

        // คำนวณตำแหน่งใหม่
        Vector3 targetPosition = player.position + position;
        Vector3 direction = Quaternion.Euler(rotation) * Vector3.back;

        cameraTransform.position = targetPosition + direction * distance;
        cameraTransform.rotation = Quaternion.Euler(rotation);

        // อัปเดต Field of View
        Camera camera = cameraTransform.GetComponent<Camera>();
        if (camera != null)
        {
            camera.fieldOfView = Mathf.Lerp(camera.fieldOfView, fieldOfView, Time.deltaTime * 2f);
        }
    }

    /// <summary>
    /// Lerp ระหว่างสองสถานะ
    /// Lerp between two states
    /// </summary>
    public static CameraState Lerp(CameraState from, CameraState to, float t)
    {
        CameraState result = new CameraState();

        result.stateName = to.stateName;
        result.position = Vector3.Lerp(from.position, to.position, t);
        result.rotation = Vector3.Lerp(from.rotation, to.rotation, t);
        result.fieldOfView = Mathf.Lerp(from.fieldOfView, to.fieldOfView, t);
        result.distance = Mathf.Lerp(from.distance, to.distance, t);

        return result;
    }
}
```

### 3. Camera Animation System

```csharp
using UnityEngine;
using System.Collections;

public class CameraAnimation : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public TPCameraController cameraController;
    public CameraStateManager stateManager;

    [Header("อนิเมชันที่มีอยู่ - Available Animations")]
    public AnimationClip enterCombatAnimation;
    public AnimationClip victoryAnimation;
    public AnimationClip deathAnimation;

    /// <summary>
    /// เล่นอนิเมชันเข้าสู่การต่อสู้
    /// Play enter combat animation
    /// </summary>
    public void เล่นอนิเมชันเข้าต่อสู้()
    {
        StartCoroutine(EnterCombatSequence());
    }

    /// <summary>
    /// เล่นอนิเมชันชัยชนะ
    /// Play victory animation
    /// </summary>
    public void เล่นอนิเมชันชัยชนะ()
    {
        StartCoroutine(VictorySequence());
    }

    /// <summary>
    /// ลำดับอนิเมชันเข้าสู่การต่อสู้
    /// Enter combat animation sequence
    /// </summary>
    private IEnumerator EnterCombatSequence()
    {
        if (stateManager == null) yield break;

        // เปลี่ยนไปสถานะต่อสู้
        stateManager.เปลี่ยนเป็นสถานะต่อสู้();

        // รอให้กล้องเคลื่อนที่ไปตำแหน่ง
        yield return new WaitForSeconds(1f);

        // เพิ่มเอฟเฟคสั่นกล้อง
        StartCoroutine(CameraShake(0.5f, 0.1f));

        // รอให้เอฟเฟคจบ
        yield return new WaitForSeconds(0.5f);
    }

    /// <summary>
    /// ลำดับอนิเมชันชัยชนะ
    /// Victory animation sequence
    /// </summary>
    private IEnumerator VictorySequence()
    {
        // ซูมเข้าใกล้ผู้เล่น
        float originalDistance = cameraController.targetDistance;
        cameraController.targetDistance = 2f;

        yield return new WaitForSeconds(1f);

        // หมุนรอบผู้เล่น
        StartCoroutine(RotateAroundPlayer(3f, 360f));

        yield return new WaitForSeconds(3f);

        // ซูมออกกลับระยะห่างปกติ
        cameraController.targetDistance = originalDistance;
    }

    /// <summary>
    /// สั่นกล้อง
    /// Shake camera
    /// </summary>
    public IEnumerator CameraShake(float duration, float magnitude)
    {
        Vector3 originalPosition = transform.localPosition;
        float elapsed = 0f;

        while (elapsed < duration)
        {
            float x = Random.Range(-1f, 1f) * magnitude;
            float y = Random.Range(-1f, 1f) * magnitude;

            transform.localPosition = originalPosition + new Vector3(x, y, 0);

            elapsed += Time.deltaTime;
            yield return null;
        }

        transform.localPosition = originalPosition;
    }

    /// <summary>
    /// หมุนกล้องรอบผู้เล่น
    /// Rotate camera around player
    /// </summary>
    private IEnumerator RotateAroundPlayer(float duration, float angle)
    {
        float elapsed = 0f;
        float startYaw = cameraController.yaw;

        while (elapsed < duration)
        {
            float progress = elapsed / duration;
            cameraController.targetYaw = startYaw + (angle * progress);

            elapsed += Time.deltaTime;
            yield return null;
        }
    }
}
```

### 4. Smart Camera Collision System

```csharp
using UnityEngine;

public class SmartCameraCollision : MonoBehaviour
{
    [Header("การตั้งค่า - Settings")]
    public LayerMask collisionLayers = -1;     // เลเยอร์ที่ตรวจสอบการชน
    public float collisionRadius = 0.3f;       // รัศมีการตรวจสอบ
    public float predictionDistance = 0.5f;    // ระยะทางคาดการณ์
    public int maxIterations = 10;              // จำนวนการวนซ้ำสูงสุด

    [Header("การอ้างอิง - References")]
    public Transform player;                   // ผู้เล่น
    public Camera mainCamera;                  // กล้องหลัก

    private Vector3[] collisionPoints;         // จุดตรวจสอบการชน
    private bool isColliding = false;          // กำลังชนอยู่หรือไม่

    void Start()
    {
        กำหนดจุดตรวจสอบการชน();
    }

    void LateUpdate()
    {
        if (player == null || mainCamera == null) return;

        ปรับตำแหน่งกล้องเพื่อหลบสิ่งกีดขวาง();
    }

    /// <summary>
    /// กำหนดจุดที่ใช้ตรวจสอบการชน
    /// Define collision checking points
    /// </summary>
    private void กำหนดจุดตรวจสอบการชน()
    {
        collisionPoints = new Vector3[]
        {
            Vector3.zero,                    // จุดศูนย์กลาง
            Vector3.up * 0.5f,              // จุดบน
            Vector3.down * 0.5f,            // จุดล่าง
            Vector3.left * 0.5f,            // จุดซ้าย
            Vector3.right * 0.5f,           // จุดขวา
            Vector3.up * 0.3f + Vector3.right * 0.3f,    // จุดบนขวา
            Vector3.up * 0.3f + Vector3.left * 0.3f,     // จุดบนซ้าย
            Vector3.down * 0.3f + Vector3.right * 0.3f,  // จุดล่างขวา
            Vector3.down * 0.3f + Vector3.left * 0.3f    // จุดล่างซ้าย
        };
    }

    /// <summary>
    /// ปรับตำแหน่งกล้องเพื่อหลบสิ่งกีดขวาง
    /// Adjust camera position to avoid obstacles
    /// </summary>
    private void ปรับตำแหน่งกล้องเพื่อหลบสิ่งกีดขวาง()
    {
        Vector3 playerPosition = player.position + Vector3.up * 1.5f; // ความสูงตา
        Vector3 cameraToPlayer = transform.position - playerPosition;

        float originalDistance = cameraToPlayer.magnitude;
        Vector3 desiredDirection = cameraToPlayer.normalized;

        // คำนวณทิศทางที่ปรับแล้ว
        Vector3 adjustedDirection = คำนวณทิศทางที่ปรับแล้ว(playerPosition, desiredDirection, originalDistance);

        // ตั้งตำแหน่งกล้องใหม่
        Vector3 newPosition = playerPosition + adjustedDirection * originalDistance;
        transform.position = Vector3.Lerp(transform.position, newPosition, Time.deltaTime * 5f);

        // หันกล้องไปยังผู้เล่น
        transform.LookAt(playerPosition);
    }

    /// <summary>
    /// คำนวณทิศทางที่ปรับแล้วเพื่อหลบสิ่งกีดขวาง
    /// Calculate adjusted direction to avoid obstacles
    /// </summary>
    private Vector3 คำนวณทิศทางที่ปรับแล้ว(Vector3 playerPosition, Vector3 desiredDirection, float distance)
    {
        Vector3 adjustedDirection = desiredDirection;
        bool foundCollision = false;

        // ตรวจสอบการชนจากหลายจุด
        foreach (Vector3 offset in collisionPoints)
        {
            Vector3 checkPosition = playerPosition + desiredDirection * distance + offset;
            Vector3 checkDirection = (checkPosition - playerPosition).normalized;

            if (Physics.Raycast(playerPosition, checkDirection, out RaycastHit hit, distance, collisionLayers))
            {
                foundCollision = true;

                // คำนวณทิศทางใหม่ที่หลบสิ่งกีดขวาง
                Vector3 hitNormal = hit.normal;
                Vector3 newDirection = Vector3.Reflect(desiredDirection, hitNormal).normalized;

                // รวมทิศทางใหม่กับทิศทางเดิม
                adjustedDirection = Vector3.Slerp(adjustedDirection, newDirection, 0.3f).normalized;

                // เพิ่มการคาดการณ์
                adjustedDirection += hitNormal * predictionDistance;
                adjustedDirection = adjustedDirection.normalized;

                break;
            }
        }

        // หากไม่พบการชน ให้ใช้ทิศทางเดิม
        if (!foundCollision)
        {
            return desiredDirection;
        }

        return adjustedDirection;
    }
}
```

## การสร้าง Third-Person Camera ใน Unity

### ขั้นตอนการสร้าง (Creation Steps)

1. **สร้าง Empty GameObject** ชื่อ "ThirdPersonCamera"
2. **เพิ่ม Camera Component**
3. **แนบ Scripts**:
   - TPCameraController.cs
   - CameraStateManager.cs
   - CameraAnimation.cs
   - SmartCameraCollision.cs
4. **กำหนด Player Transform** ใน Inspector
5. **ปรับ Camera Settings**

## การควบคุมขั้นสูง (Advanced Controls)

### การตั้งค่าระบบอินพุต (Input System Setup)

```csharp
using UnityEngine.InputSystem;

public class CameraInputHandler : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public TPCameraController cameraController;

    [Header("Input Actions - การกระทำอินพุต")]
    public InputAction lookAction;        // มองรอบ
    public InputAction zoomAction;        // ซูม
    public InputAction switchStateAction; // สลับสถานะ

    void OnEnable()
    {
        lookAction.Enable();
        zoomAction.Enable();
        switchStateAction.Enable();
    }

    void OnDisable()
    {
        lookAction.Disable();
        zoomAction.Disable();
        switchStateAction.Disable();
    }

    void Update()
    {
        Vector2 lookInput = lookAction.ReadValue<Vector2>();
        float zoomInput = zoomAction.ReadValue<float>();

        if (cameraController != null)
        {
            cameraController.UpdateRotation(lookInput);
            cameraController.UpdateDistance(zoomInput);
        }
    }
}
```

## การทดสอบระบบ (Testing the System)

### การควบคุมการทดสอบ (Test Controls)

- **เมาส์ขวา** - มองรอบตัวละคร
- **Mouse Wheel** - ซูมเข้า/ออก
- **Space** - สลับสถานะกล้อง
- **WASD** - เคลื่อนที่ตัวละคร

### สิ่งที่ควรสังเกต (What to Observe)

- **การติดตามที่นุ่มนวล** - กล้องเคลื่อนที่ตามผู้เล่นอย่างลื่นไหล
- **การหลบสิ่งกีดขวาง** - กล้องปรับตำแหน่งอัตโนมัติเมื่อมีสิ่งกีดขวาง
- **การซูม** - สามารถซูมเข้าใกล้และออกห่างได้
- **การเปลี่ยนสถานะ** - กล้องเปลี่ยนมุมมองตามสถานการณ์

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **เคลื่อนที่ลื่นไหล** - SmoothDamp ทำให้การเคลื่อนที่เป็นธรรมชาติ
- **หลบสิ่งกีดขวางอัจฉริยะ** - ใช้หลายจุดตรวจสอบการชน
- **หลายสถานะ** - สามารถเปลี่ยนมุมกล้องตามสถานการณ์
- **อนิเมชัน** - มีเอฟเฟคกล้องสำหรับเหตุการณ์พิเศษ

### ⚠️ ข้อควรระวัง
- **ความซับซ้อน** - มีหลายระบบทำงานร่วมกัน
- **ประสิทธิภาพ** - การตรวจสอบการชนอาจกินทรัพยากร
- **การปรับสมดุล** - ต้องปรับค่าต่างๆ ให้เหมาะกับเกม

## แบบฝึกหัด (Exercise)

1. **สร้างกล้องบุคคลที่ 3** ที่ติดตามผู้เล่นอย่างนุ่มนวล
2. **เพิ่มระบบหลบสิ่งกีดขวาง** ให้กล้องไม่ติดกับวัตถุ
3. **สร้างหลายสถานะกล้อง** (ปกติ, ต่อสู้, ตาย)
4. **เพิ่มอนิเมชันกล้อง** สำหรับเหตุการณ์พิเศษ

**คุณเข้าใจระบบกล้องบุคคลที่ 3 นี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนเรื่องตัวควบคุมผู้เล่นกันต่อ! 🚀
