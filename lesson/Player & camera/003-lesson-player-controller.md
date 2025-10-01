# บทเรียนที่ 003: ตัวควบคุมผู้เล่น (Player Controller)

## วัตถุประสงค์ (Objectives)
เรียนรู้การสร้างตัวควบคุมผู้เล่นที่ตอบสนองดีและเคลื่อนที่อย่างเป็นธรรมชาติ

## ภาพรวมระบบ (System Overview)

ตัวควบคุมผู้เล่นที่ดีควรมี:
- การเคลื่อนที่บนพื้น (Ground Movement)
- การกระโดด (Jumping)
- การวิ่ง (Running)
- การหมุนตัว (Rotation)
- การตรวจสอบพื้น (Ground Check)
- การป้องกันการทะลุพื้น (Collision Prevention)

## โครงสร้างระบบ (System Structure)

```
Player Controller/
├── PlayerMovement.cs           # การเคลื่อนที่หลัก
├── GroundCheck.cs              # ตรวจสอบพื้น
├── JumpController.cs           # ควบคุมการกระโดด
├── RotationController.cs       # ควบคุมการหมุน
└── MovementSettings.cs         # การตั้งค่าเคลื่อนที่
```

## การติดตั้งใช้งาน (Implementation)

### 1. Player Movement Controller

```csharp
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public CharacterController controller;      // Character Controller
    public Transform cameraTransform;          // กล้องของผู้เล่น
    public GroundCheck groundCheck;            // ตัวตรวจสอบพื้น

    [Header("การตั้งค่าเคลื่อนที่ - Movement Settings")]
    public MovementSettings movementSettings;   // การตั้งค่าเคลื่อนที่

    [Header("สถานะผู้เล่น - Player Status")]
    private Vector3 velocity;                   // ความเร็วปัจจุบัน
    private float speed;                        // ความเร็วปัจจุบัน
    private bool isGrounded;                    // อยู่บนพื้นหรือไม่
    private bool isRunning;                     // กำลังวิ่งหรือไม่

    void Start()
    {
        if (controller == null)
            controller = GetComponent<CharacterController>();

        if (groundCheck == null)
            groundCheck = GetComponentInChildren<GroundCheck>();

        velocity = Vector3.zero;
        speed = movementSettings.walkSpeed;
    }

    void Update()
    {
        อัปเดตสถานะผู้เล่น();
        จัดการอินพุต();
        ใช้แรงโน้มถ่วง();
        อัปเดตการเคลื่อนที่();
    }

    /// <summary>
    /// อัปเดตสถานะผู้เล่น
    /// Update player status
    /// </summary>
    private void อัปเดตสถานะผู้เล่น()
    {
        isGrounded = groundCheck != null ? groundCheck.IsGrounded() : controller.isGrounded;
        isRunning = Input.GetKey(movementSettings.runKey);

        // ปรับความเร็วตามสถานะ
        if (isRunning && isGrounded)
            speed = movementSettings.runSpeed;
        else
            speed = movementSettings.walkSpeed;
    }

    /// <summary>
    /// จัดการอินพุตจากผู้เล่น
    /// Handle player input
    /// </summary>
    private void จัดการอินพุต()
    {
        if (cameraTransform == null) return;

        // อินพุตการเคลื่อนที่
        float moveX = Input.GetAxis("Horizontal");
        float moveZ = Input.GetAxis("Vertical");

        // คำนวณทิศทางเคลื่อนที่สัมพัทธ์กับกล้อง
        Vector3 forward = cameraTransform.forward;
        Vector3 right = cameraTransform.right;

        forward.y = 0f;
        right.y = 0f;

        forward.Normalize();
        right.Normalize();

        // คำนวณเวกเตอร์เคลื่อนที่
        Vector3 moveDirection = (forward * moveZ + right * moveX).normalized;

        // หมุนผู้เล่นไปทางที่เคลื่อนที่
        if (moveDirection != Vector3.zero)
        {
            Quaternion targetRotation = Quaternion.LookRotation(moveDirection);
            transform.rotation = Quaternion.Slerp(transform.rotation, targetRotation, movementSettings.rotationSpeed * Time.deltaTime);
        }

        // เคลื่อนที่ผู้เล่น
        Vector3 moveVelocity = moveDirection * speed;
        velocity.x = moveVelocity.x;
        velocity.z = moveVelocity.z;
    }

    /// <summary>
    /// ใช้แรงโน้มถ่วง
    /// Apply gravity
    /// </summary>
    private void ใช้แรงโน้มถ่วง()
    {
        if (isGrounded && velocity.y < 0)
        {
            velocity.y = -2f; // ติดพื้น
        }
        else
        {
            velocity.y += movementSettings.gravity * Time.deltaTime;
        }
    }

    /// <summary>
    /// อัปเดตการเคลื่อนที่ของ Character Controller
    /// Update Character Controller movement
    /// </summary>
    private void อัปเดตการเคลื่อนที่()
    {
        if (controller != null)
        {
            controller.Move(velocity * Time.deltaTime);
        }
        else
        {
            // สำหรับ GameObject ปกติ (ไม่ใช้ Character Controller)
            transform.position += velocity * Time.deltaTime;
        }
    }

    /// <summary>
    /// ทำให้ผู้เล่นกระโดด
    /// Make player jump
    /// </summary>
    public void กระโดด()
    {
        if (isGrounded)
        {
            velocity.y = Mathf.Sqrt(movementSettings.jumpHeight * -2f * movementSettings.gravity);
            Debug.Log("ผู้เล่นกระโดด!");
        }
    }

    /// <summary>
    /// หยุดการเคลื่อนที่ทั้งหมด
    /// Stop all movement
    /// </summary>
    public void หยุดเคลื่อนที่()
    {
        velocity = Vector3.zero;
    }
}
```

### 2. Ground Check System

```csharp
using UnityEngine;

public class GroundCheck : MonoBehaviour
{
    [Header("การตั้งค่า - Settings")]
    public float groundDistance = 0.4f;       // ระยะตรวจสอบพื้น
    public LayerMask groundMask;              // เลเยอร์ที่ถือว่าเป็นพื้น

    [Header("สถานะ - Status")]
    private bool isGrounded = false;          // อยู่บนพื้นหรือไม่

    void FixedUpdate()
    {
        ตรวจสอบพื้น();
    }

    /// <summary>
    /// ตรวจสอบว่าผู้เล่นอยู่บนพื้นหรือไม่
    /// Check if player is on ground
    /// </summary>
    private void ตรวจสอบพื้น()
    {
        // สร้าง Ray จากตำแหน่งของ GroundCheck ลงด้านล่าง
        Ray groundRay = new Ray(transform.position, Vector3.down);

        // ตรวจสอบการชนกับพื้น
        if (Physics.Raycast(groundRay, out RaycastHit hit, groundDistance, groundMask))
        {
            isGrounded = true;
        }
        else
        {
            isGrounded = false;
        }
    }

    /// <summary>
    /// ตรวจสอบว่าอยู่บนพื้นหรือไม่
    /// Check if grounded
    /// </summary>
    public bool IsGrounded()
    {
        return isGrounded;
    }

    /// <summary>
    /// แสดง Gizmo สำหรับการดีบัก
    /// Show gizmo for debugging
    /// </summary>
    void OnDrawGizmos()
    {
        Gizmos.color = isGrounded ? Color.green : Color.red;

        // วาดเส้น Ray ที่ใช้ตรวจสอบพื้น
        Gizmos.DrawRay(transform.position, Vector3.down * groundDistance);

        // วาดสเฟียร์ที่จุดสิ้นสุดของ Ray
        Gizmos.DrawWireSphere(transform.position + Vector3.down * groundDistance, 0.1f);
    }
}
```

### 3. Jump Controller

```csharp
using UnityEngine;

public class JumpController : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public PlayerMovement playerMovement;       // ตัวควบคุมการเคลื่อนที่
    public GroundCheck groundCheck;             // ตัวตรวจสอบพื้น

    [Header("การตั้งค่าการกระโดด - Jump Settings")]
    public KeyCode jumpKey = KeyCode.Space;     // ปุ่มกระโดด
    public float jumpCooldown = 0.2f;           // ระยะห่างการกระโดด
    public int maxJumps = 2;                    // จำนวนครั้งกระโดดสูงสุด

    [Header("สถานะการกระโดด - Jump Status")]
    private float lastJumpTime = 0f;            // เวลากระโดดล่าสุด
    private int currentJumps = 0;               // จำนวนครั้งที่กระโดดแล้ว

    void Update()
    {
        จัดการอินพุตการกระโดด();
    }

    /// <summary>
    /// จัดการอินพุตการกระโดด
    /// Handle jump input
    /// </summary>
    private void จัดการอินพุตการกระโดด()
    {
        if (Input.GetKeyDown(jumpKey))
        {
            if (สามารถกระโดดได้())
            {
                กระโดด();
            }
        }
    }

    /// <summary>
    /// ตรวจสอบว่าสามารถกระโดดได้หรือไม่
    /// Check if can jump
    /// </summary>
    private bool สามารถกระโดดได้()
    {
        // ตรวจสอบ Cooldown
        if (Time.time - lastJumpTime < jumpCooldown)
            return false;

        // ตรวจสอบจำนวนครั้งที่กระโดด
        if (groundCheck != null && groundCheck.IsGrounded())
        {
            currentJumps = 0; // รีเซ็ตเมื่ออยู่บนพื้น
            return true;
        }
        else if (currentJumps < maxJumps - 1)
        {
            return true; // กระโดดกลางอากาศได้
        }

        return false;
    }

    /// <summary>
    /// ทำการกระโดด
    /// Perform jump
    /// </summary>
    private void กระโดด()
    {
        if (playerMovement != null)
        {
            playerMovement.กระโดด();
            currentJumps++;
            lastJumpTime = Time.time;

            Debug.Log($"กระโดดครั้งที่ {currentJumps}");
        }
    }

    /// <summary>
    /// รีเซ็ตจำนวนการกระโดด
    /// Reset jump count
    /// </summary>
    public void รีเซ็ตการกระโดด()
    {
        currentJumps = 0;
        lastJumpTime = 0f;
    }
}
```

### 4. Movement Settings

```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "MovementSettings", menuName = "Settings/Movement Settings")]
public class MovementSettings : ScriptableObject
{
    [Header("ความเร็ว - Speeds")]
    public float walkSpeed = 6f;              // ความเร็วเดิน
    public float runSpeed = 12f;              // ความเร็ววิ่ง
    public float rotationSpeed = 10f;         // ความเร็วการหมุน

    [Header("การกระโดด - Jumping")]
    public float jumpHeight = 3f;             // ความสูงกระโดด
    public float gravity = -9.81f;            // แรงโน้มถ่วง

    [Header("การควบคุม - Controls")]
    public KeyCode runKey = KeyCode.LeftShift; // ปุ่มวิ่ง
    public KeyCode jumpKey = KeyCode.Space;   // ปุ่มกระโดด

    [Header("การจำกัด - Limits")]
    public float maxFallSpeed = 20f;          // ความเร็วตกสูงสุด
    public float airControl = 0.5f;           // การควบคุมในอากาศ
}
```

## การเคลื่อนที่ขั้นสูง (Advanced Movement)

### 1. Smooth Rotation System

```csharp
public class SmoothRotation : MonoBehaviour
{
    [Header("การตั้งค่า - Settings")]
    public float rotationSmoothTime = 0.1f;   // เวลาในการหมุนที่นุ่มนวล
    public Transform cameraTransform;         // กล้องอ้างอิง

    private float rotationVelocity;           // ความเร็วการหมุน

    /// <summary>
    /// หมุนผู้เล่นไปยังทิศทางที่ต้องการอย่างนุ่มนวล
    /// Smoothly rotate player to desired direction
    /// </summary>
    public void หมุนไปยังทิศทาง(Vector3 direction, float deltaTime)
    {
        if (direction == Vector3.zero) return;

        // คำนวณมุมเป้าหมาย
        float targetAngle = Mathf.Atan2(direction.x, direction.z) * Mathf.Rad2Deg + cameraTransform.eulerAngles.y;

        // หมุนอย่างนุ่มนวล
        float angle = Mathf.SmoothDampAngle(transform.eulerAngles.y, targetAngle, ref rotationVelocity, rotationSmoothTime);

        transform.rotation = Quaternion.Euler(0f, angle, 0f);
    }
}
```

### 2. Wall Running System

```csharp
public class WallRunController : MonoBehaviour
{
    [Header("การตั้งค่า - Settings")]
    public LayerMask wallLayer;               // เลเยอร์กำแพง
    public float wallRunGravity = 1f;         // แรงโน้มถ่วงเมื่อวิ่งกำแพง
    public float wallRunSpeed = 8f;           // ความเร็ววิ่งกำแพง

    private bool isWallRunning = false;       // กำลังวิ่งกำแพงอยู่หรือไม่

    void Update()
    {
        ตรวจสอบการวิ่งกำแพง();
    }

    /// <summary>
    /// ตรวจสอบว่าสามารถวิ่งกำแพงได้หรือไม่
    /// Check if can wall run
    /// </summary>
    private void ตรวจสอบการวิ่งกำแพง()
    {
        // ตรวจสอบว่าอยู่ใกล้กำแพงและกำลังเคลื่อนที่ไปด้านข้าง
        RaycastHit hit;
        if (Physics.Raycast(transform.position, transform.right, out hit, 1f, wallLayer))
        {
            if (!isGrounded && Input.GetKey(KeyCode.D))
            {
                เริ่มวิ่งกำแพง(hit.normal);
            }
        }
        else if (Physics.Raycast(transform.position, -transform.right, out hit, 1f, wallLayer))
        {
            if (!isGrounded && Input.GetKey(KeyCode.A))
            {
                เริ่มวิ่งกำแพง(hit.normal);
            }
        }
        else
        {
            หยุดวิ่งกำแพง();
        }
    }

    /// <summary>
    /// เริ่มวิ่งกำแพง
    /// Start wall running
    /// </summary>
    private void เริ่มวิ่งกำแพง(Vector3 wallNormal)
    {
        isWallRunning = true;

        // ปรับแรงโน้มถ่วง
        if (playerMovement != null)
        {
            playerMovement.velocity.y = 0f; // หยุดการตก
            // ปรับความเร็วการเคลื่อนที่
        }
    }

    /// <summary>
    /// หยุดวิ่งกำแพง
    /// Stop wall running
    /// </summary>
    private void หยุดวิ่งกำแพง()
    {
        isWallRunning = false;

        // คืนค่าแรงโน้มถ่วงปกติ
        if (playerMovement != null)
        {
            // คืนค่า gravity ปกติ
        }
    }
}
```

### 3. Dash System

```csharp
using UnityEngine;

public class DashController : MonoBehaviour
{
    [Header("การตั้งค่าแดช - Dash Settings")]
    public float dashSpeed = 20f;             // ความเร็วแดช
    public float dashDuration = 0.2f;         // ระยะเวลาแดช
    public float dashCooldown = 1f;           // ระยะห่างแดช
    public KeyCode dashKey = KeyCode.LeftShift; // ปุ่มแดช

    [Header("สถานะแดช - Dash Status")]
    private bool isDashing = false;           // กำลังแดชอยู่หรือไม่
    private float dashStartTime = 0f;         // เวลาเริ่มแดช
    private float lastDashTime = 0f;          // เวลาแดชล่าสุด
    private Vector3 dashDirection;            // ทิศทางแดช

    void Update()
    {
        จัดการอินพุตแดช();
        อัปเดตสถานะแดช();
    }

    /// <summary>
    /// จัดการอินพุตแดช
    /// Handle dash input
    /// </summary>
    private void จัดการอินพุตแดช()
    {
        if (Input.GetKeyDown(dashKey) && สามารถแดชได้())
        {
            เริ่มแดช();
        }
    }

    /// <summary>
    /// ตรวจสอบว่าสามารถแดชได้หรือไม่
    /// Check if can dash
    /// </summary>
    private bool สามารถแดชได้()
    {
        return Time.time - lastDashTime >= dashCooldown;
    }

    /// <summary>
    /// เริ่มแดช
    /// Start dashing
    /// </summary>
    private void เริ่มแดช()
    {
        isDashing = true;
        dashStartTime = Time.time;
        lastDashTime = Time.time;

        // คำนวณทิศทางแดช
        Vector3 moveInput = new Vector3(Input.GetAxis("Horizontal"), 0f, Input.GetAxis("Vertical"));
        if (moveInput == Vector3.zero)
        {
            // ถ้าไม่มีอินพุต ให้แดชไปข้างหน้า
            dashDirection = transform.forward;
        }
        else
        {
            dashDirection = moveInput.normalized;
        }

        if (playerMovement != null)
        {
            playerMovement.velocity = dashDirection * dashSpeed;
        }

        Debug.Log("เริ่มแดช!");
    }

    /// <summary>
    /// อัปเดตสถานะแดช
    /// Update dash status
    /// </summary>
    private void อัปเดตสถานะแดช()
    {
        if (isDashing)
        {
            if (Time.time - dashStartTime >= dashDuration)
            {
                สิ้นสุดแดช();
            }
            else
            {
                // ระหว่างแดช ให้รักษาความเร็ว
                if (playerMovement != null)
                {
                    playerMovement.velocity = dashDirection * dashSpeed;
                }
            }
        }
    }

    /// <summary>
    /// สิ้นสุดแดช
    /// End dashing
    /// </summary>
    private void สิ้นสุดแดช()
    {
        isDashing = false;

        if (playerMovement != null)
        {
            // คืนค่าการเคลื่อนที่ปกติ
            Vector3 moveInput = new Vector3(Input.GetAxis("Horizontal"), 0f, Input.GetAxis("Vertical"));
            playerMovement.velocity = moveInput.normalized * playerMovement.speed;
        }

        Debug.Log("สิ้นสุดแดช");
    }
}
```

## การตั้งค่าใน Unity Editor (Editor Setup)

### ขั้นตอนการสร้าง (Creation Steps)

1. **สร้าง Capsule** สำหรับตัวผู้เล่น
2. **เพิ่ม Character Controller Component**
3. **แนบ Scripts**:
   - PlayerMovement.cs
   - GroundCheck.cs
   - JumpController.cs
   - DashController.cs (ถ้าต้องการ)
4. **สร้าง Empty GameObject** สำหรับ GroundCheck
5. **กำหนด Camera Transform** ใน Inspector

## การควบคุมขั้นสูง (Advanced Controls)

### การตั้งค่าระบบอินพุต (Input System Setup)

```csharp
using UnityEngine.InputSystem;

public class PlayerInputHandler : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public PlayerMovement playerMovement;

    [Header("Input Actions")]
    public InputAction moveAction;
    public InputAction jumpAction;
    public InputAction runAction;
    public InputAction dashAction;

    void OnEnable()
    {
        moveAction.Enable();
        jumpAction.Enable();
        runAction.Enable();
        dashAction.Enable();
    }

    void OnDisable()
    {
        moveAction.Disable();
        jumpAction.Disable();
        runAction.Disable();
        dashAction.Disable();
    }

    void Update()
    {
        Vector2 moveInput = moveAction.ReadValue<Vector2>();
        bool jumpPressed = jumpAction.WasPressedThisFrame();
        bool runPressed = runAction.IsPressed();
        bool dashPressed = dashAction.WasPressedThisFrame();

        if (playerMovement != null)
        {
            playerMovement.SetMoveInput(moveInput);

            if (jumpPressed)
                playerMovement.Jump();

            if (runPressed)
                playerMovement.SetRunning(true);

            if (dashPressed)
                playerMovement.Dash();
        }
    }
}
```

## การทดสอบระบบ (Testing the System)

### การควบคุมการทดสอบ (Test Controls)

- **WASD** - เคลื่อนที่ผู้เล่น
- **Space** - กระโดด
- **Left Shift** - วิ่ง
- **Left Shift** (ใน Movement Settings) - แดช

### สิ่งที่ควรสังเกต (What to Observe)

- **การเคลื่อนที่บนพื้น** - ลื่นไหลและตอบสนองดี
- **การกระโดด** - สูงตามที่ตั้งค่าไว้
- **การตรวจสอบพื้น** - ไม่ทะลุพื้นหรือลอย
- **การหมุนตัว** - หมุนตามทิศทางการเคลื่อนที่

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **เคลื่อนที่เป็นธรรมชาติ** - ใช้ Character Controller
- **ตรวจสอบพื้นแม่นยำ** - ไม่มีปัญหากระตุก
- **ขยายได้** - เพิ่มความสามารถใหม่ได้ง่าย
- **ปรับแต่งได้** - ScriptableObject สำหรับการตั้งค่า

### ⚠️ ข้อควรระวัง
- **การชนกัน** - Character Controller กับสิ่งกีดขวาง
- **ประสิทธิภาพ** - การตรวจสอบพื้นทุกเฟรม
- **การตั้งค่า** - ต้องปรับให้เหมาะกับเกม

## แบบฝึกหัด (Exercise)

1. **สร้างตัวควบคุมผู้เล่น** ที่เคลื่อนที่ WASD
2. **เพิ่มระบบกระโดด** ที่ทำงานอย่างถูกต้อง
3. **สร้างระบบวิ่ง** เมื่อกดปุ่มวิ่ง
4. **เพิ่มระบบแดช** สำหรับเคลื่อนที่เร็วพิเศษ

**คุณเข้าใจตัวควบคุมผู้เล่นนี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนหัวข้อถัดไปกัน! 🚀
