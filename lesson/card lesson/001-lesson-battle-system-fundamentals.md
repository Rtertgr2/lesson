# บทเรียนที่ 001: พื้นฐานระบบต่อสู้แบบผลัดกันเล่น (Battle System Fundamentals)

## วัตถุประสงค์ (Objectives)
เรียนรู้สถาปัตยกรรมพื้นฐานของระบบต่อสู้แบบผลัดกันเล่นและแนวคิดระบบการ์ด

## ภาพรวมระบบ (System Overview)

ระบบต่อสู้แบบผลัดกันเล่นประกอบด้วย:
- การจัดการสถานะการต่อสู้ (Battle States)
- คลาสพื้นฐานสำหรับการ์ด (Card Base Classes)
- หน่วยต่อสู้ (Battle Units)
- ผู้จัดการระบบต่อสู้ (Battle Manager)

## โครงสร้างระบบ (System Structure)

```
Battle System/
├── BattleManager.cs        # จัดการการไหลของเกม
├── Card.cs                 # คลาสพื้นฐานสำหรับการ์ด
├── BattleUnit.cs           # หน่วยต่อสู้ (ผู้เล่น/ศัตรู)
└── BattleState.cs          # สถานะการต่อสู้
```

## การติดตั้งใช้งาน (Implementation)

### 1. สถานะการต่อสู้ (Battle States)

```csharp
public enum BattleState
{
    กำลังเตรียม,      // กำลังตั้งค่าระบบต่อสู้ - Preparing, Setting up the battle
    เทิร์นผู้เล่น,     // เทิร์นของผู้เล่นที่จะทำการกระทำ - Player's turn to act
    เทิร์นศัตรู,       // เทิร์นของศัตรูที่จะทำการกระทำ - Enemy's turn to act
    จบการต่อสู้,      // การต่อสู้จบแล้ว (ชนะ/แพ้) - Battle finished (win/lose)
    ว่าง          // ไม่ได้อยู่ในระบบต่อสู้ - Not in battle
}
```

### 2. คลาสการ์ดพื้นฐาน (Card Base Class)

```csharp
public abstract class Card
{
    public string ชื่อการ์ด;        // ชื่อของการ์ด
    public string คำอธิบาย;       // คำอธิบายการทำงาน
    public int ค่าใช้มานา;        // มานาที่ต้องใช้
    public ประเภทการ์ด ประเภทการ์ด; // ประเภทการ์ด

    public Card(string ชื่อ, string คำอธิบาย, int ค่าใช้มานา, ประเภทการ์ด ประเภท)
    {
        ชื่อการ์ด = ชื่อ;
        this.คำอธิบาย = คำอธิบาย;
        this.ค่าใช้มานา = ค่าใช้มานา;
        this.ประเภทการ์ด = ประเภท;
    }

    /// <summary>
    /// สิ่งที่เกิดขึ้นเมื่อเล่นการ์ดใบนี้
    /// What happens when this card is played
    /// </summary>
    public abstract void เล่นการ์ด(BattleUnit ผู้ใช้, BattleUnit เป้าหมาย);
}
```

### 3. ประเภทการ์ด (Card Types)

```csharp
public enum ประเภทการ์ด
{
    ตัวเลข,         // 1.1 การ์ดตัวเลข - 1.1 Number Cards
    สลับสมการ,      // 1.2 การ์ดสลับสมการ - 1.2 Equation Switch Cards
    ตัวดำเนินการ,    // 1.3 การ์ดตัวดำเนินการ - 1.3 Operator Cards
    ช่วยเหลือ,      // 1.4 การ์ดช่วยเหลือ - 1.4 Help Cards
    กับดัก         // 1.5 การ์ดกับดัก - 1.5 Trap Cards
}
```

### 4. หน่วยต่อสู้ (Battle Unit)

```csharp
public class BattleUnit
{
    public string ชื่อหน่วย;           // ชื่อตัวละคร
    public int พลังชีวิตปัจจุบัน;      // HP ปัจจุบัน
    public int พลังชีวิตสูงสุด;        // HP สูงสุด
    public int มานาปัจจุบัน;          // MP ปัจจุบัน
    public int มานาสูงสุด;            // MP สูงสุด
    public bool เป็นผู้เล่น;           // true = ผู้เล่น, false = ศัตรู

    public BattleUnit(string ชื่อ, int พลังชีวิต, int มานา, bool ผู้เล่น)
    {
        ชื่อหน่วย = ชื่อ;
        พลังชีวิตสูงสุด = พลังชีวิต;
        พลังชีวิตปัจจุบัน = พลังชีวิต;
        มานาสูงสุด = มานา;
        มานาปัจจุบัน = มานา;
        เป็นผู้เล่น = ผู้เล่น;
    }

    /// <summary>
    /// รับความเสียหาย - Take damage
    /// </summary>
    public void รับความเสียหาย(int ความเสียหาย)
    {
        พลังชีวิตปัจจุบัน -= ความเสียหาย;
        if (พลังชีวิตปัจจุบัน < 0) พลังชีวิตปัจจุบัน = 0;

        Debug.Log($"{ชื่อหน่วย} รับความเสียหาย {ความเสียหาย} แต้ม!");
    }

    /// <summary>
    /// ใช้มานา - Spend mana
    /// </summary>
    public bool ใช้มานา(int จำนวน)
    {
        if (มานาปัจจุบัน >= จำนวน)
        {
            มานาปัจจุบัน -= จำนวน;
            return true;
        }
        return false;
    }

    /// <summary>
    /// ตรวจสอบว่าหน่วยยังมีชีวิตอยู่ - Check if unit is alive
    /// </summary>
    public bool มีชีวิตอยู่()
    {
        return พลังชีวิตปัจจุบัน > 0;
    }
}
```

### 5. ผู้จัดการระบบต่อสู้ (Battle Manager)

```csharp
public class BattleManager
{
    public BattleState สถานะปัจจุบัน; // currentState
    public BattleUnit ผู้เล่น; // player
    public BattleUnit ศัตรู; // enemy
    public List<Card> มือของผู้เล่น; // playerHand - การ์ดในมือของผู้เล่น

    public BattleManager()
    {
        มือของผู้เล่น = new List<Card>();
        สถานะปัจจุบัน = BattleState.ว่าง;
    }

    /// <summary>
    /// เริ่มระบบต่อสู้ใหม่ - Start a new battle
    /// </summary>
    public void เริ่มการต่อสู้()
    {
        // กำหนดค่าเริ่มต้นให้ผู้เล่นและศัตรู
        ผู้เล่น = new BattleUnit("วีรบุรุษ", 100, 50, true);
        ศัตรู = new BattleUnit("มอนสเตอร์", 80, 30, false);

        // จั่วมือเริ่มต้น (ตัวอย่าง) - Draw initial hand (example)
        จั่วมือเริ่มต้น();

        // เริ่มด้วยเทิร์นของผู้เล่น - Start with player's turn
        สถานะปัจจุบัน = BattleState.เทิร์นผู้เล่น;

        Debug.Log("เริ่มการต่อสู้แล้ว! เทิร์นของผู้เล่น - Battle started! Player's turn.");
    }

    /// <summary>
    /// จบเทิร์นปัจจุบันและเปลี่ยนไปยังผู้เล่นคนถัดไป
    /// End current turn and switch to next player
    /// </summary>
    public void จบเทิร์น()
    {
        if (สถานะปัจจุบัน == BattleState.เทิร์นผู้เล่น)
        {
            สถานะปัจจุบัน = BattleState.เทิร์นศัตรู;
            Debug.Log("เทิร์นของศัตรู! - Enemy's turn!");

            // AI ของศัตรูจะทำการกระทำที่นี่ - Enemy AI would act here
            ประมวลผลเทิร์นศัตรู();
        }
        else if (สถานะปัจจุบัน == BattleState.เทิร์นศัตรู)
        {
            สถานะปัจจุบัน = BattleState.เทิร์นผู้เล่น;
            Debug.Log("เทิร์นของผู้เล่น! - Player's turn!");

            // จั่วการ์ดใบใหม่ในแต่ละเทิร์น - Draw a new card each turn
            จั่วการ์ด();
        }
    }

    /// <summary>
    /// เล่นการ์ดจากมือของผู้เล่น - Play a card from player's hand
    /// </summary>
    public bool เล่นการ์ด(int ดัชนีการ์ด)
    {
        if (ดัชนีการ์ด < 0 || ดัชนีการ์ด >= มือของผู้เล่น.Count)
            return false;

        Card การ์ด = มือของผู้เล่น[ดัชนีการ์ด];

        // ตรวจสอบว่าผู้เล่นมีมานาพอหรือไม่ - Check if player has enough mana
        if (!ผู้เล่น.ใช้มานา(การ์ด.ค่าใช้มานา))
        {
            Debug.Log("มานาไม่พอ! - Not enough mana!");
            return false;
        }

        // เล่นการ์ด - Play the card
        การ์ด.เล่นการ์ด(ผู้เล่น, ศัตรู);

        // นำการ์ดออกจากมือ - Remove card from hand
        มือของผู้เล่น.RemoveAt(ดัชนีการ์ด);

        return true;
    }

    /// <summary>
    /// จั่วมือเริ่มต้นของการ์ด - Draw initial hand of cards
    /// </summary>
    private void จั่วมือเริ่มต้น()
    {
        // ตัวอย่าง: จั่ว 5 การ์ด - Example: Draw 5 cards
        for (int i = 0; i < 5; i++)
        {
            จั่วการ์ด();
        }
    }

    /// <summary>
    /// จั่วการ์ดใบเดียว - Draw a single card
    /// </summary>
    private void จั่วการ์ด()
    {
        // ตอนนี้สร้างการ์ดตัวเลขแบบง่าย - For now, create a simple number card
        Card การ์ดใหม่ = new การ์ดตัวเลข("5", "การ์ดตัวเลขที่มีค่า 5", 2, 5);
        มือของผู้เล่น.Add(การ์ดใหม่);

        Debug.Log($"จั่วการ์ด: {การ์ดใหม่.ชื่อการ์ด} - Drew card: {การ์ดใหม่.ชื่อการ์ด}");
    }

    /// <summary>
    /// ประมวลผลเทิร์นของศัตรู (AI แบบง่าย) - Process enemy turn (simple AI)
    /// </summary>
    private void ประมวลผลเทิร์นศัตรู()
    {
        // AI ของศัตรูแบบง่าย: โจมตีผู้เล่นโดยตรง - Simple enemy AI: attack player directly
        int ความเสียหาย = 15;
        ผู้เล่น.รับความเสียหาย(ความเสียหาย);

        // จบเทิร์นของศัตรูหลังจากหน่วงเวลา (ในเกมจริง) - End enemy turn after a delay (in real game)
        จบเทิร์น();
    }
}
```

### 6. การ์ดตัวอย่าง (Example Card Implementation)

```csharp
public class การ์ดตัวเลข : Card
{
    public int ค่าตัวเลข; // numberValue

    public การ์ดตัวเลข(string ชื่อ, string คำอธิบาย, int ค่าใช้มานา, int ค่า) :
        base(ชื่อ, คำอธิบาย, ค่าใช้มานา, ประเภทการ์ด.ตัวเลข)
    {
        ค่าตัวเลข = ค่า;
    }

    public override void เล่นการ์ด(BattleUnit ผู้ใช้, BattleUnit เป้าหมาย)
    {
        // ตอนนี้แค่สร้างความเสียหายเท่ากับตัวเลข - For now, just deal damage equal to the number
        int ความเสียหาย = ค่าตัวเลข;
        เป้าหมาย.รับความเสียหาย(ความเสียหาย);

        Debug.Log($"{ผู้ใช้.ชื่อหน่วย} เล่น {ชื่อการ์ด} สร้างความเสียหาย {ความเสียหาย} แต้ม!");
    }
}
```

## การไหลของเกม (Game Flow)

### ลำดับการทำงาน (Sequence)

1. **เริ่มการต่อสู้** → ผู้เล่นได้มืการ์ดเริ่มต้น (5 ใบ)
2. **เทิร์นผู้เล่น** → เลือกและเล่นการ์ด → จบเทิร์น
3. **เทิร์นศัตรู** → AI เล่นการ์ดหรือโจมตี → จบเทิร์น
4. **วนซ้ำ** จนกว่าฝ่ายใดฝ่ายหนึ่งจะแพ้
5. **จบเกม** → แสดงผลชนะ/แพ้

### การควบคุมการทดสอบ (Test Controls)

```csharp
void Update()
{
    // กด Spacebar เพื่อจบเทิร์น (สำหรับการทดสอบ) - Press Space to end turn (for testing)
    if (Input.GetKeyDown(KeyCode.Space))
    {
        if (ผู้จัดการต่อสู้ != null)
        {
            ผู้จัดการต่อสู้.จบเทิร์น();
        }
    }

    // กด Tab เพื่อเล่นการ์ด (สำหรับการทดสอบ) - Press Tab to play a card (for testing)
    if (Input.GetKeyDown(KeyCode.Tab))
    {
        if (ผู้จัดการต่อสู้ != null && ผู้จัดการต่อสู้.มือของผู้เล่น.Count > 0)
        {
            ผู้จัดการต่อสู้.เล่นการ์ด(0);
        }
    }
}
```

## จุดสำคัญที่ควรรู้ (Key Concepts)

### ✅ สถาปัตยกรรมที่ดี (Good Architecture)
- **แยกความรับผิดชอบ** - แต่ละคลาสมีหน้าที่เฉพาะ
- **สืบทอดคลาส** - การ์ดแต่ละประเภทสืบทอดจาก Card
- **สถานะการจัดการ** - ใช้ Enum จัดการสถานะเกม

### ✅ การออกแบบการ์ด (Card Design)
- **แต่ละการ์ดมีเอกลักษณ์** - แต่ละประเภทมีพฤติกรรมต่างกัน
- **ใช้มานาเป็นทรัพยากร** - จัดสมดุลการใช้การ์ด
- **เอฟเฟคชัดเจน** - ผู้เล่นเข้าใจการ์ดแต่ละใบ

### ✅ การจัดการเทิร์น (Turn Management)
- **ผลัดกันเล่น** - ผู้เล่นและศัตรูผลัดกันเล่น
- **ตรวจสอบเงื่อนไข** - มานาพอไหม ถึงเทิร์นหรือยัง
- **เปลี่ยนสถานะ** - อัปเดตสถานะเกมตามการกระทำ

## แบบฝึกหัด (Exercise)

1. **สร้าง BattleUnit เพิ่มเติม** (เช่น มอนสเตอร์ประเภทต่างๆ)
2. **เพิ่มประเภทการ์ดใหม่** (ตามระบบของคุณ)
3. **ปรับสมดุลเกม** (HP, MP, ความเสียหาย)
4. **เพิ่มเงื่อนไขชนะ** (เช่น เก็บคะแนนให้ถึง 1000)

## การทดสอบระบบ (Testing the System)

### ขั้นตอนการทดสอบ (Testing Steps)

1. **แนบสคริปต์** กับ GameObject ในซีน
2. **กด Play** เพื่อรันเกม
3. **เปิด Console Window** เพื่อดู Debug.Log
4. **ทดสอบการควบคุม**:
   - **Spacebar** - จบเทิร์น
   - **Tab** - เล่นการ์ดใบแรก

### สิ่งที่ควรสังเกต (What to Observe)

- **สถานะการต่อสู้** เปลี่ยนตามการกระทำ
- **มานา** ลดลงเมื่อเล่นการ์ด
- **พลังชีวิต** ลดลงเมื่อรับความเสียหาย
- **เทิร์น** เปลี่ยนระหว่างผู้เล่นและศัตรู

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **โครงสร้างชัดเจน** - เข้าใจง่าย ปรับแต่งได้
- **ขยายได้** - เพิ่มการ์ดและกฎใหม่ได้ง่าย
- **ทดสอบง่าย** - Debug.Log แสดงการทำงาน

### ⚠️ ข้อควรปรับปรุง
- **AI ศัตรู** - ยังเป็นแบบง่ายมาก
- **UI การ์ด** - ยังไม่มีส่วนติดต่อผู้ใช้
- **สมการคณิตศาสตร์** - ยังไม่มีระบบคำนวณ

**คุณเข้าใจพื้นฐานระบบต่อสู้นี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนเรื่องการติดตั้งใช้งานการ์ดแต่ละประเภทกันต่อ! 🚀
