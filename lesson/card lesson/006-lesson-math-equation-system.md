# บทเรียนที่ 006: ระบบสมการคณิตศาสตร์ (Math Equation System)

## วัตถุประสงค์ (Objectives)
เรียนรู้การจัดการสมการคณิตศาสตร์ในระบบการ์ดต่อสู้อย่างถูกต้องและปลอดภัย

## ภาพรวมระบบ (System Overview)

ระบบสมการคณิตศาสตร์จะจัดการ:
- การคำนวณผลลัพธ์ทางคณิตศาสตร์
- การจัดการลำดับการดำเนินการ
- การป้องกันข้อผิดพลาด (เช่น การหารด้วยศูนย์)
- การแสดงผลลัพธ์ที่เข้าใจง่าย

## โครงสร้างระบบ (System Structure)

```
Math System/
├── EquationManager.cs       # จัดการสมการหลัก
├── MathValidator.cs         # ตรวจสอบความถูกต้อง
├── ResultCalculator.cs      # คำนวณผลลัพธ์
└── SafeMath.cs             # คณิตศาสตร์ที่ปลอดภัย
```

## การติดตั้งใช้งาน (Implementation)

### 1. EquationManager.cs - จัดการสมการหลัก

```csharp
using UnityEngine;
using System.Collections.Generic;
using System.Text;

public class EquationManager : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public BattleManager battleManager;     // ผู้จัดการการต่อสู้

    [Header("สถานะสมการ - Equation Status")]
    public List<int> ตัวเลขในสมการ = new List<int>();     // เก็บตัวเลขที่เล่นแล้ว
    public string ตัวดำเนินการปัจจุบัน = "+";           // ตัวดำเนินการที่ใช้งานอยู่
    public List<string> ประวัติสมการ = new List<string>(); // ประวัติการคำนวณ

    [Header("การตั้งค่า - Settings")]
    public int จำนวนตัวเลขสูงสุดในสมการ = 5;            // จำกัดจำนวนตัวเลข
    public bool แสดงขั้นตอนการคำนวณ = true;           // แสดงรายละเอียด

    /// <summary>
    /// เพิ่มตัวเลขเข้าสมการ
    /// Add number to equation
    /// </summary>
    public void เพิ่มตัวเลข(int เลข)
    {
        if (ตัวเลขในสมการ.Count >= จำนวนตัวเลขสูงสุดในสมการ)
        {
            Debug.LogWarning("สมการเต็มแล้ว - Equation is full");
            return;
        }

        ตัวเลขในสมการ.Add(เลข);

        if (แสดงขั้นตอนการคำนวณ)
        {
            Debug.Log($"เพิ่มตัวเลข: {เลข} - สมการปัจจุบัน: {แสดงสมการ()}");
        }

        // คำนวณผลลัพธ์ทันทีถ้ามีตัวเลขอย่างน้อย 2 ตัว
        if (ตัวเลขในสมการ.Count >= 2)
        {
            คำนวณผลลัพธ์();
        }
    }

    /// <summary>
    /// เปลี่ยนตัวดำเนินการ
    /// Change operator
    /// </summary>
    public void เปลี่ยนตัวดำเนินการ(string ตัวดำเนินการใหม่)
    {
        if (string.IsNullOrEmpty(ตัวดำเนินการใหม่)) return;

        ตัวดำเนินการปัจจุบัน = ตัวดำเนินการใหม่;

        if (แสดงขั้นตอนการคำนวณ && ตัวเลขในสมการ.Count > 0)
        {
            Debug.Log($"เปลี่ยนตัวดำเนินการเป็น: {ตัวดำเนินการใหม่} - สมการ: {แสดงสมการ()}");
        }

        // คำนวณใหม่ถ้ามีตัวเลขเพียงพอ
        if (ตัวเลขในสมการ.Count >= 2)
        {
            คำนวณผลลัพธ์();
        }
    }

    /// <summary>
    /// คำนวณผลลัพธ์ของสมการ
    /// Calculate equation result
    /// </summary>
    public void คำนวณผลลัพธ์()
    {
        if (ตัวเลขในสมการ.Count < 2)
        {
            Debug.LogWarning("ต้องการอย่างน้อย 2 ตัวเลขเพื่อคำนวณ - Need at least 2 numbers");
            return;
        }

        // ใช้ MathValidator เพื่อตรวจสอบความถูกต้อง
        if (!MathValidator.สามารถคำนวณได้(ตัวเลขในสมการ, ตัวดำเนินการปัจจุบัน))
        {
            Debug.LogError("ไม่สามารถคำนวณสมการนี้ได้ - Cannot calculate this equation");
            return;
        }

        // คำนวณผลลัพธ์
        int ผลลัพธ์ = ResultCalculator.คำนวณผลลัพธ์(ตัวเลขในสมการ, ตัวดำเนินการปัจจุบัน);

        if (แสดงขั้นตอนการคำนวณ)
        {
            string สมการ = แสดงสมการ();
            Debug.Log($"คำนวณ: {สมการ} = {ผลลัพธ์}");
        }

        // บันทึกประวัติ
        บันทึกประวัติสมการ(ผลลัพธ์);

        // ใช้ผลลัพธ์กับเกม
        ใช้ผลลัพธ์ในเกม(ผลลัพธ์);

        // ล้างสมการหลังคำนวณ
        ล้างสมการ();
    }

    /// <summary>
    /// แสดงสมการในรูปแบบข้อความ
    /// Display equation as text
    /// </summary>
    public string แสดงสมการ()
    {
        if (ตัวเลขในสมการ.Count == 0) return "ว่างเปล่า";

        StringBuilder สมการ = new StringBuilder();
        สมการ.Append(ตัวเลขในสมการ[0]);

        for (int i = 1; i < ตัวเลขในสมการ.Count; i++)
        {
            สมการ.Append($" {ตัวดำเนินการปัจจุบัน} {ตัวเลขในสมการ[i]}");
        }

        return สมการ.ToString();
    }

    /// <summary>
    /// ล้างสมการ
    /// Clear equation
    /// </summary>
    public void ล้างสมการ()
    {
        ตัวเลขในสมการ.Clear();
        ตัวดำเนินการปัจจุบัน = "+"; // รีเซ็ตเป็นบวก
    }

    /// <summary>
    /// ใช้ผลลัพธ์กับเกม
    /// Apply result to game
    /// </summary>
    private void ใช้ผลลัพธ์ในเกม(int ผลลัพธ์)
    {
        if (battleManager == null) return;

        // แปลงผลลัพธ์เป็นความเสียหาย
        int ความเสียหาย = Mathf.Abs(ผลลัพธ์); // ใช้ค่าสัมบูรณ์เสมอ

        // จำกัดความเสียหายสูงสุด
        ความเสียหาย = Mathf.Min(ความเสียหาย, 999);

        // สร้างความเสียหายให้ศัตรู
        battleManager.ศัตรู.รับความเสียหาย(ความเสียหาย);

        Debug.Log($"ใช้ผลลัพธ์: {ผลลัพธ์} แต้ม - สร้างความเสียหาย: {ความเสียหาย} แต้ม");
    }

    /// <summary>
    /// บันทึกประวัติสมการ
    /// Record equation history
    /// </summary>
    private void บันทึกประวัติสมการ(int ผลลัพธ์)
    {
        string สมการ = แสดงสมการ();
        ประวัติสมการ.Add($"{สมการ} = {ผลลัพธ์}");

        // เก็บประวัติแค่ 10 รายการล่าสุด
        if (ประวัติสมการ.Count > 10)
        {
            ประวัติสมการ.RemoveAt(0);
        }
    }

    /// <summary>
    /// แสดงประวัติการคำนวณทั้งหมด
    /// Display all calculation history
    /// </summary>
    public string แสดงประวัติการคำนวณ()
    {
        if (ประวัติสมการ.Count == 0) return "ไม่มีประวัติ";

        StringBuilder ประวัติ = new StringBuilder();
        ประวัติ.AppendLine("ประวัติการคำนวณ:");

        for (int i = 0; i < ประวัติสมการ.Count; i++)
        {
            ประวัติ.AppendLine($"{i + 1}. {ประวัติสมการ[i]}");
        }

        return ประวัติ.ToString();
    }
}
```

### 2. MathValidator.cs - ตรวจสอบความถูกต้อง

```csharp
using UnityEngine;
using System.Collections.Generic;

public static class MathValidator
{
    /// <summary>
    /// ตรวจสอบว่าสามารถคำนวณสมการนี้ได้หรือไม่
    /// Check if equation can be calculated
    /// </summary>
    public static bool สามารถคำนวณได้(List<int> ตัวเลข, string ตัวดำเนินการ)
    {
        // ตรวจสอบจำนวนตัวเลข
        if (ตัวเลข.Count < 2)
        {
            Debug.LogWarning("ต้องการอย่างน้อย 2 ตัวเลข - Need at least 2 numbers");
            return false;
        }

        // ตรวจสอบตัวดำเนินการที่รองรับ
        if (!ตัวดำเนินการที่รองรับ(ตัวดำเนินการ))
        {
            Debug.LogError($"ตัวดำเนินการไม่รองรับ: {ตัวดำเนินการ} - Unsupported operator");
            return false;
        }

        // ตรวจสอบการหารด้วยศูนย์
        if (ตัวดำเนินการ == "÷" || ตัวดำเนินการ == "/")
        {
            foreach (int เลข in ตัวเลข)
            {
                if (เลข == 0)
                {
                    Debug.LogError("ไม่สามารถหารด้วยศูนย์ได้ - Cannot divide by zero");
                    return false;
                }
            }
        }

        // ตรวจสอบเลขที่ใหญ่เกินไป
        foreach (int เลข in ตัวเลข)
        {
            if (เลข > 1000000 || เลข < -1000000)
            {
                Debug.LogWarning($"ตัวเลขใหญ่เกินไป: {เลข} - Number too large");
                return false;
            }
        }

        return true;
    }

    /// <summary>
    /// ตรวจสอบว่าตัวดำเนินการรองรับหรือไม่
    /// Check if operator is supported
    /// </summary>
    private static bool ตัวดำเนินการที่รองรับ(string ตัวดำเนินการ)
    {
        string[] ตัวดำเนินการที่รองรับ = { "+", "-", "×", "*", "÷", "/" };
        return System.Array.Exists(ตัวดำเนินการที่รองรับ, op => op == ตัวดำเนินการ);
    }

    /// <summary>
    /// ตรวจสอบความปลอดภัยของสมการ
    /// Validate equation safety
    /// </summary>
    public static bool สมการปลอดภัย(List<int> ตัวเลข, string ตัวดำเนินการ)
    {
        // ตรวจสอบไม่ให้ผลลัพธ์ติดลบถ้าไม่ต้องการ
        if (!อนุญาตผลลัพธ์ติดลบ() && อาจให้ผลลัพธ์ติดลบ(ตัวเลข, ตัวดำเนินการ))
        {
            return false;
        }

        // ตรวจสอบไม่ให้ผลลัพธ์เป็นศูนย์
        if (!อนุญาตผลลัพธ์เป็นศูนย์() && อาจให้ผลลัพธ์เป็นศูนย์(ตัวเลข, ตัวดำเนินการ))
        {
            return false;
        }

        return true;
    }

    private static bool อนุญาตผลลัพธ์ติดลบ() => true; // สามารถปรับได้
    private static bool อนุญาตผลลัพธ์เป็นศูนย์() => true; // สามารถปรับได้

    private static bool อาจให้ผลลัพธ์ติดลบ(List<int> ตัวเลข, string ตัวดำเนินการ)
    {
        // ตรวจสอบกรณีที่อาจให้ผลลัพธ์ติดลบ
        return ตัวเลข.Any(n => n < 0) || ตัวดำเนินการ == "-";
    }

    private static bool อาจให้ผลลัพธ์เป็นศูนย์(List<int> ตัวเลข, string ตัวดำเนินการ)
    {
        // ตรวจสอบกรณีที่อาจให้ผลลัพธ์เป็นศูนย์
        return ตัวเลข.Contains(0) && (ตัวดำเนินการ == "×" || ตัวดำเนินการ == "*");
    }
}
```

### 3. ResultCalculator.cs - คำนวณผลลัพธ์

```csharp
using UnityEngine;
using System.Collections.Generic;

public static class ResultCalculator
{
    /// <summary>
    /// คำนวณผลลัพธ์ของสมการ
    /// Calculate equation result
    /// </summary>
    public static int คำนวณผลลัพธ์(List<int> ตัวเลข, string ตัวดำเนินการ)
    {
        if (ตัวเลข.Count == 0) return 0;

        int ผลลัพธ์ = ตัวเลข[0];

        for (int i = 1; i < ตัวเลข.Count; i++)
        {
            ผลลัพธ์ = ใช้ตัวดำเนินการ(ผลลัพธ์, ตัวเลข[i], ตัวดำเนินการ);
        }

        return ผลลัพธ์;
    }

    /// <summary>
    /// คำนวณผลลัพธ์ของตัวเลขสองตัว
    /// Calculate result of two numbers
    /// </summary>
    private static int ใช้ตัวดำเนินการ(int ตัวเลข1, int ตัวเลข2, string ตัวดำเนินการ)
    {
        switch (ตัวดำเนินการ)
        {
            case "+": return SafeMath.บวก(ตัวเลข1, ตัวเลข2);
            case "-": return SafeMath.ลบ(ตัวเลข1, ตัวเลข2);
            case "×":
            case "*": return SafeMath.คูณ(ตัวเลข1, ตัวเลข2);
            case "÷":
            case "/": return SafeMath.หาร(ตัวเลข1, ตัวเลข2);
            default:
                Debug.LogError($"ตัวดำเนินการไม่รองรับ: {ตัวดำเนินการ}");
                return ตัวเลข1;
        }
    }

    /// <summary>
    /// คำนวณตามลำดับความสำคัญของตัวดำเนินการ
    /// Calculate with operator precedence
    /// </summary>
    public static int คำนวณด้วยลำดับความสำคัญ(List<int> ตัวเลข, List<string> ตัวดำเนินการ)
    {
        // จัดการ × และ ÷ ก่อน (+ และ - ทีหลัง)
        return คำนวณด้วยลำดับ(ตัวเลข, ตัวดำเนินการ, new[] { "×", "*" }, new[] { "÷", "/" });
    }

    private static int คำนวณด้วยลำดับ(List<int> ตัวเลข, List<string> ตัวดำเนินการ,
                                       string[] ตัวดำเนินการลำดับแรก, string[] ตัวดำเนินการลำดับสอง)
    {
        // คำนวณ × และ ÷ ก่อน
        for (int i = ตัวดำเนินการ.Count - 1; i >= 0; i--)
        {
            if (System.Array.Exists(ตัวดำเนินการลำดับแรก, op => op == ตัวดำเนินการ[i]))
            {
                int ผลลัพธ์ = ใช้ตัวดำเนินการ(ตัวเลข[i], ตัวเลข[i + 1], ตัวดำเนินการ[i]);

                // แทนที่ผลลัพธ์ในลิสต์
                ตัวเลข[i] = ผลลัพธ์;
                ตัวเลข.RemoveAt(i + 1);
                ตัวดำเนินการ.RemoveAt(i);
            }
        }

        // คำนวณ + และ - ทีหลัง
        for (int i = ตัวดำเนินการ.Count - 1; i >= 0; i--)
        {
            if (System.Array.Exists(ตัวดำเนินการลำดับสอง, op => op == ตัวดำเนินการ[i]))
            {
                int ผลลัพธ์ = ใช้ตัวดำเนินการ(ตัวเลข[i], ตัวเลข[i + 1], ตัวดำเนินการ[i]);

                ตัวเลข[i] = ผลลัพธ์;
                ตัวเลข.RemoveAt(i + 1);
                ตัวดำเนินการ.RemoveAt(i);
            }
        }

        return ตัวเลข[0];
    }
}
```

### 4. SafeMath.cs - คณิตศาสตร์ที่ปลอดภัย

```csharp
using UnityEngine;

public static class SafeMath
{
    /// <summary>
    /// บวกตัวเลขอย่างปลอดภัย
    /// Safe addition
    /// </summary>
    public static int บวก(int a, int b)
    {
        try
        {
            checked { return a + b; }
        }
        catch (System.OverflowException)
        {
            Debug.LogWarning($"เกิดการล้นจำนวนในการบวก: {a} + {b} - Overflow in addition");
            return int.MaxValue;
        }
    }

    /// <summary>
    /// ลบตัวเลขอย่างปลอดภัย
    /// Safe subtraction
    /// </summary>
    public static int ลบ(int a, int b)
    {
        try
        {
            checked { return a - b; }
        }
        catch (System.OverflowException)
        {
            Debug.LogWarning($"เกิดการล้นจำนวนในการลบ: {a} - {b} - Overflow in subtraction");
            return int.MinValue;
        }
    }

    /// <summary>
    /// คูณตัวเลขอย่างปลอดภัย
    /// Safe multiplication
    /// </summary>
    public static int คูณ(int a, int b)
    {
        try
        {
            checked { return a * b; }
        }
        catch (System.OverflowException)
        {
            Debug.LogWarning($"เกิดการล้นจำนวนในการคูณ: {a} × {b} - Overflow in multiplication");
            return int.MaxValue;
        }
    }

    /// <summary>
    /// หารตัวเลขอย่างปลอดภัย
    /// Safe division
    /// </summary>
    public static int หาร(int a, int b)
    {
        if (b == 0)
        {
            Debug.LogError("ไม่สามารถหารด้วยศูนย์ได้ - Cannot divide by zero");
            return 0;
        }

        try
        {
            checked { return a / b; }
        }
        catch (System.OverflowException)
        {
            Debug.LogWarning($"เกิดการล้นจำนวนในการหาร: {a} ÷ {b} - Overflow in division");
            return int.MaxValue;
        }
    }

    /// <summary>
    /// คำนวณกำลังสองอย่างปลอดภัย
    /// Safe power calculation
    /// </summary>
    public static int กำลังสอง(int ฐาน)
    {
        try
        {
            checked { return ฐาน * ฐาน; }
        }
        catch (System.OverflowException)
        {
            Debug.LogWarning($"เกิดการล้นจำนวนในการยกกำลัง: {ฐาน}² - Overflow in power");
            return int.MaxValue;
        }
    }

    /// <summary>
    /// คำนวณรากที่สองอย่างปลอดภัย
    /// Safe square root
    /// </summary>
    public static double รากที่สอง(double จำนวน)
    {
        if (จำนวน < 0)
        {
            Debug.LogWarning($"ไม่สามารถคำนวณรากที่สองของจำนวนติดลบ: {จำนวน} - Cannot calculate square root of negative number");
            return 0;
        }

        return System.Math.Sqrt(จำนวน);
    }

    /// <summary>
    /// คำนวณค่าสัมบูรณ์
    /// Absolute value
    /// </summary>
    public static int ค่าสัมบูรณ์(int จำนวน)
    {
        return System.Math.Abs(จำนวน);
    }

    /// <summary>
    /// จำกัดค่าตัวเลขในช่วงที่กำหนด
    /// Clamp number to range
    /// </summary>
    public static int จำกัดช่วง(int ค่า, int ต่ำสุด, int สูงสุด)
    {
        return Mathf.Clamp(ค่า, ต่ำสุด, สูงสุด);
    }
}
```

## การขยาย BattleUnit สำหรับคณิตศาสตร์

### MathBattleUnit.cs (อัปเดต)

```csharp
public class MathBattleUnit : BattleUnit
{
    [Header("ระบบสมการ - Equation System")]
    public EquationManager equationManager;     // ผู้จัดการสมการ

    public MathBattleUnit(string ชื่อ, int พลังชีวิต, int มานา, bool ผู้เล่น)
        : base(ชื่อ, พลังชีวิต, มานา, ผู้เล่น)
    {
        equationManager = new EquationManager();
    }

    /// <summary>
    /// เพิ่มตัวเลขเข้าสมการ
    /// Add number to equation
    /// </summary>
    public void เพิ่มตัวเลขเข้าสมการ(int เลข)
    {
        if (equationManager != null)
        {
            equationManager.เพิ่มตัวเลข(เลข);
        }
    }

    /// <summary>
    /// เปลี่ยนตัวดำเนินการ
    /// Change operator
    /// </summary>
    public void เปลี่ยนตัวดำเนินการ(string ตัวดำเนินการใหม่)
    {
        if (equationManager != null)
        {
            equationManager.เปลี่ยนตัวดำเนินการ(ตัวดำเนินการใหม่);
        }
    }

    /// <summary>
    /// แสดงสมการปัจจุบัน
    /// Display current equation
    /// </summary>
    public string แสดงสมการปัจจุบัน()
    {
        return equationManager?.แสดงสมการ() ?? "ไม่มีสมการ";
    }

    /// <summary>
    /// แสดงประวัติการคำนวณ
    /// Display calculation history
    /// </summary>
    public string แสดงประวัติการคำนวณ()
    {
        return equationManager?.แสดงประวัติการคำนวณ() ?? "ไม่มีประวัติ";
    }
}
```

## การทดสอบระบบ (Testing the System)

### การตั้งค่าในซีน (Scene Setup)

1. **สร้าง EquationManager** ในซีน
2. **เชื่อมต่อกับ BattleManager**
3. **สร้าง MathBattleUnit** แทน BattleUnit ปกติ

### การควบคุมการทดสอบ (Test Controls)

```csharp
void Update()
{
    // ทดสอบการเพิ่มตัวเลข
    if (Input.GetKeyDown(KeyCode.Alpha1)) เพิ่มตัวเลขเข้าสมการ(1);
    if (Input.GetKeyDown(KeyCode.Alpha2)) เพิ่มตัวเลขเข้าสมการ(2);
    if (Input.GetKeyDown(KeyCode.Alpha3)) เพิ่มตัวเลขเข้าสมการ(3);

    // ทดสอบการเปลี่ยนตัวดำเนินการ
    if (Input.GetKeyDown(KeyCode.P)) เปลี่ยนตัวดำเนินการ("+");
    if (Input.GetKeyDown(KeyCode.M)) เปลี่ยนตัวดำเนินการ("×");

    // แสดงข้อมูล
    if (Input.GetKeyDown(KeyCode.H)) Debug.Log(แสดงประวัติการคำนวณ());
}
```

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **คำนวณถูกต้อง** - จัดการกับทุกกรณี
- **ป้องกันข้อผิดพลาด** - หารด้วยศูนย์ ล้นจำนวน
- **แสดงผลชัดเจน** - เห็นขั้นตอนการคำนวณ
- **ขยายได้** - เพิ่มตัวดำเนินการใหม่ได้ง่าย

### ⚠️ ข้อควรระวัง
- **ประสิทธิภาพ** - การคำนวณที่ซับซ้อน
- **ความซับซ้อน** - ผู้เล่นต้องเข้าใจระบบ
- **สมดุล** - ผลลัพธ์ต้องสมดุลกับเกม

## แบบฝึกหัด (Exercise)

1. **สร้างสมการง่ายๆ** (เช่น 2 + 3 = 5)
2. **ทดสอบตัวดำเนินการ** ทั้ง 4 ตัว (+, -, ×, ÷)
3. **จัดการข้อผิดพลาด** (หารด้วยศูนย์)
4. **สร้างประวัติการคำนวณ** และแสดงผล

**คุณเข้าใจระบบสมการคณิตศาสตร์นี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนหัวข้อถัดไปกัน! 🚀
