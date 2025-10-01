# บทเรียนที่ 003: ระบบสร้างการ์ด (Card Creation System)

## วัตถุประสงค์ (Objectives)
เรียนรู้วิธีสร้างการ์ดจำนวนมากได้อย่างง่ายดาย โดยไม่ต้องเขียนโค้ดซ้ำๆ

## ภาพรวมระบบ (System Overview)

เราจะสร้างระบบที่ทำให้คุณสามารถ:
- สร้างการ์ดใหม่โดยไม่ต้องเขียนคลาสใหม่
- แก้ไขค่าการ์ดได้ง่ายผ่าน Unity Inspector
- จัดการคลังการ์ดทั้งหมดในที่เดียว
- โหลดและใช้การ์ดในเกมได้อย่างรวดเร็ว

## วิธีการสร้างระบบ (Implementation Approaches)

### 1. ScriptableObject (แนะนำ - Recommended)

**ข้อดี:**
- สร้างการ์ดเป็น Asset ใน Unity
- แก้ไขง่ายผ่าน Inspector
- โหลดเร็ว
- สามารถสร้างได้หลายร้อยใบ

**ไฟล์ที่ต้องสร้าง:**

#### CardData.cs
```csharp
using UnityEngine;

[CreateAssetMenu(fileName = "New Card", menuName = "Cards/Number Card")]
public class CardData : ScriptableObject
{
    public string ชื่อการ์ด;
    public string คำอธิบาย;
    public int ค่าใช้มานา;
    public ประเภทการ์ด ประเภท;

    // สำหรับการ์ดตัวเลข
    public int ค่าตัวเลข = 0;

    // สำหรับการ์ดตัวดำเนินการ
    public string ตัวดำเนินการ = "+";

    // สำหรับการ์ดช่วยเหลือ
    public int จำนวนรักษา = 0;
    public int จำนวนเพิ่มมานา = 0;

    // สำหรับการ์ดกับดัก
    public string ประเภทกับดัก = "";
    public int ความเสียหายกับดัก = 0;

    // สีของการ์ด (สำหรับ UI)
    public Color สีการ์ด = Color.white;

    // ไอคอนการ์ด
    public Sprite ไอคอนการ์ด;
}
```

#### CardManager.cs
```csharp
using UnityEngine;
using System.Collections.Generic;
using System.Linq;

public class CardManager : MonoBehaviour
{
    // คลังการ์ดทั้งหมดในเกม
    public List<CardData> การ์ดทั้งหมด;

    // โหลดการ์ดทั้งหมดจาก Resources folder
    public void โหลดการ์ดทั้งหมด()
    {
        การ์ดทั้งหมด = Resources.LoadAll<CardData>("Cards").ToList();
        Debug.Log($"โหลดการ์ดแล้ว {การ์ดทั้งหมด.Count} ใบ");
    }

    // หาการ์ดตามชื่อ
    public CardData หาการ์ดตามชื่อ(string ชื่อ)
    {
        return การ์ดทั้งหมด.Find(card => card.ชื่อการ์ด == ชื่อ);
    }

    // หาการ์ดตามประเภท
    public List<CardData> หาการ์ดตามประเภท(ประเภทการ์ด ประเภท)
    {
        return การ์ดทั้งหมด.Where(card => card.ประเภท == ประเภท).ToList();
    }

    // สุ่มการ์ด
    public CardData สุ่มการ์ด()
    {
        if (การ์ดทั้งหมด.Count == 0) return null;
        return การ์ดทั้งหมด[Random.Range(0, การ์ดทั้งหมด.Count)];
    }

    // สุ่มการ์ดตามประเภท
    public CardData สุ่มการ์ดตามประเภท(ประเภทการ์ด ประเภท)
    {
        var การ์ดที่กรองแล้ว = หาการ์ดตามประเภท(ประเภท);
        if (การ์ดที่กรองแล้ว.Count == 0) return null;
        return การ์ดที่กรองแล้ว[Random.Range(0, การ์ดที่กรองแล้ว.Count)];
    }
}
```

### 2. การสร้างการ์ดใน Inspector

#### CardCreator.cs
```csharp
using UnityEngine;
using System.Collections.Generic;

public class CardCreator : MonoBehaviour
{
    [Header("การตั้งค่าการสร้างการ์ด")]
    public List<CardTemplate> การ์ดที่ต้องการสร้าง;

    [Header("ผลลัพธ์")]
    public List<CardData> การ์ดที่สร้างแล้ว;

    [System.Serializable]
    public class CardTemplate
    {
        public string ชื่อการ์ด;
        public ประเภทการ์ด ประเภท;
        public int มานา;

        // สำหรับแต่ละประเภท
        public int ค่าตัวเลข = 0;
        public string ตัวดำเนินการ = "+";
        public int จำนวนรักษา = 0;
        public string ประเภทกับดัก = "";
        public int ความเสียหายกับดัก = 0;
    }

    [ContextMenu("สร้างการ์ด")]
    public void สร้างการ์ด()
    {
        การ์ดที่สร้างแล้ว.Clear();

        foreach (var template in การ์ดที่ต้องการสร้าง)
        {
            CardData การ์ดใหม่ = ScriptableObject.CreateInstance<CardData>();

            การ์ดใหม่.ชื่อการ์ด = template.ชื่อการ์ด;
            การ์ดใหม่.ประเภท = template.ประเภท;
            การ์ดใหม่.ค่าใช้มานา = template.มานา;

            // กำหนดค่าเฉพาะประเภท
            switch (template.ประเภท)
            {
                case ประเภทการ์ด.ตัวเลข:
                    การ์ดใหม่.คำอธิบาย = $"ตัวเลข {template.ชื่อการ์ด}";
                    การ์ดใหม่.ค่าตัวเลข = template.ค่าตัวเลข;
                    break;

                case ประเภทการ์ด.ตัวดำเนินการ:
                    การ์ดใหม่.คำอธิบาย = $"ตัวดำเนินการ {template.ชื่อการ์ด}";
                    การ์ดใหม่.ตัวดำเนินการ = template.ตัวดำเนินการ;
                    break;

                case ประเภทการ์ด.ช่วยเหลือ:
                    การ์ดใหม่.คำอธิบาย = $"รักษา {template.จำนวนรักษา} HP";
                    การ์ดใหม่.จำนวนรักษา = template.จำนวนรักษา;
                    break;

                case ประเภทการ์ด.กับดัก:
                    การ์ดใหม่.คำอธิบาย = $"กับดัก {template.ประเภทกับดัก}";
                    การ์ดใหม่.ประเภทกับดัก = template.ประเภทกับดัก;
                    การ์ดใหม่.ความเสียหายกับดัก = template.ความเสียหายกับดัก;
                    break;
            }

            การ์ดที่สร้างแล้ว.Add(การ์ดใหม่);
        }

        Debug.Log($"สร้างการ์ดแล้ว {การ์ดที่สร้างแล้ว.Count} ใบ");
    }
}
```

### 3. โฟลเดอร์โครงสร้าง (Folder Structure)

```
Assets/
├── Cards/                          # คลังการ์ดทั้งหมด
│   ├── NumberCards/               # การ์ดตัวเลข
│   ├── OperatorCards/             # การ์ดตัวดำเนินการ
│   ├── HelpCards/                 # การ์ดช่วยเหลือ
│   └── TrapCards/                 # การ์ดกับดัก
├── Scripts/
│   └── CardSystem/
│       ├── CardData.cs            # ScriptableObject สำหรับการ์ด
│       ├── CardManager.cs         # จัดการคลังการ์ด
│       └── CardCreator.cs         # สร้างการ์ดจาก Inspector
└── Resources/
    └── Cards/                     # การ์ดที่โหลดในเกม
```

## การใช้งานจริง (Practical Usage)

### การสร้างการ์ดทีละใบ

1. **คลิกขวาใน Project window**
2. **เลือก Create → Cards → Number Card**
3. **กรอกข้อมูลใน Inspector:**
   - ชื่อการ์ด: "เลข 5"
   - คำอธิบาย: "ตัวเลขห้า"
   - ค่าใช้มานา: 2
   - ประเภท: ตัวเลข
   - ค่าตัวเลข: 5

### การสร้างการ์ดหลายใบรวดเดียว

ใช้ CardCreator component:

1. **แนบ CardCreator กับ GameObject**
2. **เพิ่ม CardTemplate ใน Inspector**
3. **คลิกปุ่ม "สร้างการ์ด"**

### การโหลดและใช้การ์ดในเกม

```csharp
public class GameManager : MonoBehaviour
{
    public CardManager cardManager;

    void Start()
    {
        // โหลดการ์ดทั้งหมด
        cardManager.โหลดการ์ดทั้งหมด();

        // หาการ์ดเลข 5
        CardData เลขห้า = cardManager.หาการ์ดตามชื่อ("เลข 5");

        // สุ่มมืการ์ด
        List<CardData> มือใหม่ = new List<CardData>();
        for (int i = 0; i < 5; i++)
        {
            มือใหม่.Add(cardManager.สุ่มการ์ด());
        }

        Debug.Log($"ได้มืการ์ด: {มือใหม่.Count} ใบ");
    }
}
```

## ประโยชน์ของระบบนี้ (Benefits)

### ✅ ประหยัดเวลา
- สร้างการ์ด 100 ใบใน 10 นาที
- ไม่ต้องเขียนโค้ดซ้ำๆ

### ✅ แก้ไขง่าย
- แก้ไขค่าการ์ดได้ทันทีใน Inspector
- ไม่ต้อง rebuild โปรเจค

### ✅ จัดการสะดวก
- ค้นหาการ์ดตามชื่อ/ประเภทได้ง่าย
- สุ่มการ์ดอัตโนมัติ

### ✅ ขยายระบบได้
- เพิ่มประเภทการ์ดใหม่ได้ง่าย
- เพิ่มข้อมูลการ์ดได้ไม่จำกัด

## แบบฝึกหัด (Exercise)

1. **สร้างการ์ดตัวเลข 1-10**
2. **สร้างการ์ดตัวดำเนินการทั้ง 4 ตัว (+, -, ×, ÷)**
3. **สร้างการ์ดช่วยเหลือ 2-3 ใบ**
4. **สร้างการ์ดกับดัก 2-3 ใบ**

**ลองสร้างการ์ดตามแบบฝึกหัด แล้วคุณเข้าใจระบบนี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปบทเรียนถัดไปกัน! 🚀
