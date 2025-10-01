# บทเรียนที่ 004: ระบบ UI การ์ด (Card UI System)

## วัตถุประสงค์ (Objectives)
เรียนรู้การสร้างส่วนติดต่อผู้ใช้ (UI) สำหรับการ์ดที่สวยงามและใช้งานง่าย

## ภาพรวมระบบ (System Overview)

ระบบ UI การ์ดจะประกอบด้วย:
- การแสดงผลการ์ดในมือผู้เล่น
- การแสดงรายละเอียดการ์ดเมื่อวางเมาส์
- เอฟเฟคการลากและวาง (Drag & Drop)
- อนิเมชันการเล่นการ์ด
- การแสดงมานาและพลังชีวิต

## โครงสร้างไฟล์ (File Structure)

```
Assets/
├── Scripts/
│   └── UI/
│       ├── CardDisplay.cs          # แสดงข้อมูลการ์ด
│       ├── CardDragHandler.cs      # จัดการการลากการ์ด
│       ├── HandManager.cs          # จัดการมืการ์ด
│       └── BattleUI.cs             # จัดการ UI การต่อสู้
├── Prefabs/
│   └── CardPrefab.prefab           # Prefab การ์ด
├── Sprites/
│   └── Cards/                      # รูปภาพการ์ด
└── Scenes/
    └── CardBattle.unity            # ซีนทดสอบระบบการ์ด
```

## การติดตั้งใช้งาน (Implementation)

### 1. CardDisplay.cs - แสดงข้อมูลการ์ด

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class CardDisplay : MonoBehaviour
{
    [Header("ส่วนประกอบ UI - UI Components")]
    public Image สีพื้นหลัง;           // พื้นหลังการ์ด
    public Image ไอคอนการ์ด;           // ไอคอนการ์ด
    public TextMeshProUGUI ชื่อการ์ด;   // ชื่อการ์ด
    public TextMeshProUGUI คำอธิบาย;   // คำอธิบาย
    public TextMeshProUGUI ค่าใช้มานา;   // ค่าใช้มานา
    public TextMeshProUGUI ค่าพิเศษ;     // ค่าพิเศษ (ตัวเลข, ความแรง)

    [Header("การอ้างอิงข้อมูล - Data Reference")]
    public CardData ข้อมูลการ์ด;       // ข้อมูลการ์ดที่แสดง

    /// <summary>
    /// แสดงข้อมูลการ์ดบน UI
    /// Display card information on UI
    /// </summary>
    public void แสดงการ์ด(CardData cardData)
    {
        ข้อมูลการ์ด = cardData;

        if (cardData == null) return;

        // แสดงข้อมูลพื้นฐาน
        ชื่อการ์ด.text = cardData.ชื่อการ์ด;
        คำอธิบาย.text = cardData.คำอธิบาย;
        ค่าใช้มานา.text = cardData.ค่าใช้มานา.ToString();

        // แสดงสีพื้นหลัง
        สีพื้นหลัง.color = cardData.สีการ์ด;

        // แสดงไอคอน
        if (cardData.ไอคอนการ์ด != null)
        {
            ไอคอนการ์ด.sprite = cardData.ไอคอนการ์ด;
            ไอคอนการ์ด.gameObject.SetActive(true);
        }
        else
        {
            ไอคอนการ์ด.gameObject.SetActive(false);
        }

        // แสดงค่าพิเศษตามประเภทการ์ด
        แสดงค่าพิเศษ(cardData);
    }

    /// <summary>
    /// แสดงค่าพิเศษของการ์ดแต่ละประเภท
    /// Display special values for each card type
    /// </summary>
    private void แสดงค่าพิเศษ(CardData cardData)
    {
        switch (cardData.ประเภท)
        {
            case ประเภทการ์ด.ตัวเลข:
                ค่าพิเศษ.text = cardData.ค่าตัวเลข.ToString();
                ค่าพิเศษ.color = Color.blue;
                break;

            case ประเภทการ์ด.ตัวดำเนินการ:
                ค่าพิเศษ.text = cardData.ตัวดำเนินการ;
                ค่าพิเศษ.color = Color.red;
                break;

            case ประเภทการ์ด.ช่วยเหลือ:
                if (cardData.จำนวนรักษา > 0)
                {
                    ค่าพิเศษ.text = $"+{cardData.จำนวนรักษา} HP";
                    ค่าพิเศษ.color = Color.green;
                }
                else if (cardData.จำนวนเพิ่มมานา > 0)
                {
                    ค่าพิเศษ.text = $"+{cardData.จำนวนเพิ่มมานา} MP";
                    ค่าพิเศษ.color = Color.cyan;
                }
                break;

            case ประเภทการ์ด.กับดัก:
                ค่าพิเศษ.text = $"{cardData.ความเสียหายกับดัก} DMG";
                ค่าพิเศษ.color = Color.magenta;
                break;

            default:
                ค่าพิเศษ.text = "";
                break;
        }
    }

    /// <summary>
    /// ตรวจสอบว่าผู้เล่นมีมานาพอสำหรับการ์ดใบนี้หรือไม่
    /// Check if player has enough mana for this card
    /// </summary>
    public bool สามารถเล่นได้(int มานาปัจจุบัน)
    {
        return มานาปัจจุบัน >= ข้อมูลการ์ด.ค่าใช้มานา;
    }

    /// <summary>
    /// อัปเดตสีการ์ดตามความสามารถในการเล่น
    /// Update card color based on playability
    /// </summary>
    public void อัปเดตสถานะการเล่น(int มานาปัจจุบัน)
    {
        if (สามารถเล่นได้(มานาปัจจุบัน))
        {
            สีพื้นหลัง.color = สามารถเล่นได้สี();
        }
        else
        {
            สีพื้นหลัง.color = ไม่สามารถเล่นได้สี();
        }
    }

    private Color สามารถเล่นได้สี() => ข้อมูลการ์ด.สีการ์ด;
    private Color ไม่สามารถเล่นได้สี() => Color.gray;
}
```

### 2. CardDragHandler.cs - จัดการการลากการ์ด

```csharp
using UnityEngine;
using UnityEngine.EventSystems;

public class CardDragHandler : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    [Header("การอ้างอิง - References")]
    public CardDisplay cardDisplay;     // การแสดงผลการ์ด
    private RectTransform rectTransform; // ตำแหน่งการ์ด
    private Canvas canvas;              // Canvas หลัก
    private CanvasGroup canvasGroup;    // กลุ่ม Canvas

    [Header("การตั้งค่า - Settings")]
    public float ความสูงเมื่อลาก = 50f; // ความสูงเมื่อลาก
    private Vector3 ตำแหน่งเริ่มต้น;     // ตำแหน่งเริ่มต้น

    void Awake()
    {
        rectTransform = GetComponent<RectTransform>();
        canvas = GetComponentInParent<Canvas>();
        canvasGroup = GetComponent<CanvasGroup>();

        if (canvasGroup == null)
        {
            canvasGroup = gameObject.AddComponent<CanvasGroup>();
        }
    }

    /// <summary>
    /// เริ่มลากการ์ด
    /// Begin dragging card
    /// </summary>
    public void OnBeginDrag(PointerEventData eventData)
    {
        ตำแหน่งเริ่มต้น = rectTransform.position;
        canvasGroup.alpha = 0.8f;
        canvasGroup.blocksRaycasts = false;

        // ยกการ์ดขึ้นเมื่อลาก
        rectTransform.SetAsLastSibling();
    }

    /// <summary>
    /// ลากการ์ด
    /// Drag card
    /// </summary>
    public void OnDrag(PointerEventData eventData)
    {
        rectTransform.anchoredPosition += eventData.delta / canvas.scaleFactor;
    }

    /// <summary>
    /// จบการลากการ์ด
    /// End dragging card
    /// </summary>
    public void OnEndDrag(PointerEventData eventData)
    {
        canvasGroup.alpha = 1f;
        canvasGroup.blocksRaycasts = true;

        // ตรวจสอบว่าวางการ์ดในพื้นที่ที่ถูกต้องหรือไม่
        if (ตรวจสอบพื้นที่วาง(eventData))
        {
            เล่นการ์ด();
        }
        else
        {
            // กลับไปตำแหน่งเดิม
            rectTransform.position = ตำแหน่งเริ่มต้น;
        }
    }

    /// <summary>
    /// ตรวจสอบว่าวางการ์ดในพื้นที่ที่ถูกต้องหรือไม่
    /// Check if card is dropped in valid area
    /// </summary>
    private bool ตรวจสอบพื้นที่วาง(PointerEventData eventData)
    {
        // ตรวจสอบว่าวางบนพื้นที่เล่นการ์ดหรือไม่
        // ในทางปฏิบัติ คุณจะตรวจสอบกับ Collider หรือพื้นที่เฉพาะ
        return true; // สำหรับตัวอย่าง ให้ถือว่าถูกต้องเสมอ
    }

    /// <summary>
    /// เล่นการ์ด
    /// Play the card
    /// </summary>
    private void เล่นการ์ด()
    {
        if (cardDisplay != null && cardDisplay.ข้อมูลการ์ด != null)
        {
            // แจ้ง HandManager ว่าการ์ดใบนี้ถูกเล่นแล้ว
            HandManager handManager = GetComponentInParent<HandManager>();
            if (handManager != null)
            {
                handManager.เล่นการ์ด(this);
            }
        }
    }
}
```

### 3. HandManager.cs - จัดการมืการ์ด

```csharp
using UnityEngine;
using System.Collections.Generic;

public class HandManager : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public GameObject cardPrefab;           // Prefab การ์ด
    public Transform handContainer;         // ที่วางมืการ์ด
    public BattleManager battleManager;     // ผู้จัดการการต่อสู้

    [Header("การตั้งค่า - Settings")]
    public int จำนวนการ์ดสูงสุดในมือ = 7;  // จำนวนการ์ดสูงสุดในมือ
    public float ระยะห่างระหว่างการ์ด = 100f; // ระยะห่างระหว่างการ์ด

    [Header("สถานะ - Status")]
    public List<CardDisplay> การ์ดในมือ;     // การ์ดในมือ
    private CardManager cardManager;        // ผู้จัดการการ์ด

    void Start()
    {
        การ์ดในมือ = new List<CardDisplay>();
        cardManager = FindObjectOfType<CardManager>();

        if (cardManager != null)
        {
            จั่วมือเริ่มต้น();
        }
    }

    /// <summary>
    /// จั่วมือเริ่มต้น
    /// Draw initial hand
    /// </summary>
    public void จั่วมือเริ่มต้น()
    {
        if (cardManager == null) return;

        List<CardData> มือใหม่ = cardManager.สุ่มมืการ์ด(5);

        foreach (CardData cardData in มือใหม่)
        {
            เพิ่มการ์ดในมือ(cardData);
        }

        จัดเรียงการ์ดในมือ();
    }

    /// <summary>
    /// เพิ่มการ์ดในมือ
    /// Add card to hand
    /// </summary>
    public void เพิ่มการ์ดในมือ(CardData cardData)
    {
        if (การ์ดในมือ.Count >= จำนวนการ์ดสูงสุดในมือ) return;

        // สร้างการ์ดจาก Prefab
        GameObject cardObject = Instantiate(cardPrefab, handContainer);
        CardDisplay cardDisplay = cardObject.GetComponent<CardDisplay>();

        if (cardDisplay != null)
        {
            cardDisplay.แสดงการ์ด(cardData);
            การ์ดในมือ.Add(cardDisplay);
        }
    }

    /// <summary>
    /// จัดเรียงการ์ดในมือ
    /// Arrange cards in hand
    /// </summary>
    public void จัดเรียงการ์ดในมือ()
    {
        int จำนวนการ์ด = การ์ดในมือ.Count;
        float ความกว้างทั้งหมด = (จำนวนการ์ด - 1) * ระยะห่างระหว่างการ์ด;
        float ตำแหน่งเริ่มต้น = -ความกว้างทั้งหมด / 2f;

        for (int i = 0; i < จำนวนการ์ด; i++)
        {
            CardDisplay card = การ์ดในมือ[i];
            float ตำแหน่งX = ตำแหน่งเริ่มต้น + (i * ระยะห่างระหว่างการ์ด);

            card.GetComponent<RectTransform>().anchoredPosition =
                new Vector2(ตำแหน่งX, 0);
        }
    }

    /// <summary>
    /// เล่นการ์ด
    /// Play card
    /// </summary>
    public void เล่นการ์ด(CardDragHandler cardDragHandler)
    {
        CardDisplay cardDisplay = cardDragHandler.cardDisplay;

        if (cardDisplay != null && battleManager != null)
        {
            // แปลง CardData เป็น Card และเล่น
            Card การ์ดที่เล่นได้ = cardDisplay.ข้อมูลการ์ด.แปลงเป็นการ์ด();

            if (battleManager.เล่นการ์ด(การ์ดที่เล่นได้))
            {
                // นำการ์ดออกจากมือ
                การ์ดในมือ.Remove(cardDisplay);
                Destroy(cardDragHandler.gameObject);

                // จัดเรียงการ์ดใหม่
                จัดเรียงการ์ดในมือ();

                Debug.Log($"เล่นการ์ด: {cardDisplay.ข้อมูลการ์ด.ชื่อการ์ด}");
            }
        }
    }

    /// <summary>
    /// จั่วการ์ดใหม่
    /// Draw new card
    /// </summary>
    public void จั่วการ์ด()
    {
        if (cardManager != null)
        {
            CardData การ์ดใหม่ = cardManager.สุ่มการ์ด();
            เพิ่มการ์ดในมือ(การ์ดใหม่);
            จัดเรียงการ์ดในมือ();
        }
    }

    /// <summary>
    /// อัปเดตสถานะการเล่นการ์ดทั้งหมด
    /// Update playability status of all cards
    /// </summary>
    public void อัปเดตสถานะการ์ดทั้งหมด()
    {
        if (battleManager == null) return;

        int มานาปัจจุบัน = battleManager.ผู้เล่น.มานาปัจจุบัน;

        foreach (CardDisplay card in การ์ดในมือ)
        {
            card.อัปเดตสถานะการเล่น(มานาปัจจุบัน);
        }
    }
}
```

### 4. BattleUI.cs - จัดการ UI การต่อสู้

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class BattleUI : MonoBehaviour
{
    [Header("การอ้างอิง UI - UI References")]
    public TextMeshProUGUI ผู้เล่นHP;        // HP ผู้เล่น
    public TextMeshProUGUI ผู้เล่นMP;        // MP ผู้เล่น
    public TextMeshProUGUI ศัตรูHP;          // HP ศัตรู
    public TextMeshProUGUI เทิร์นปัจจุบัน;    // เทิร์นปัจจุบัน
    public Button จบเทิร์นButton;          // ปุ่มจบเทิร์น

    [Header("การอ้างอิงระบบ - System References")]
    public BattleManager battleManager;     // ผู้จัดการการต่อสู้
    public HandManager handManager;         // ผู้จัดการมืการ์ด

    void Start()
    {
        if (จบเทิร์นButton != null)
        {
            จบเทิร์นButton.onClick.AddListener(จบเทิร์น);
        }

        อัปเดตUI();
    }

    void Update()
    {
        // อัปเดตสถานะการ์ดทุกเฟรม
        if (handManager != null)
        {
            handManager.อัปเดตสถานะการ์ดทั้งหมด();
        }
    }

    /// <summary>
    /// อัปเดตข้อมูล UI ทั้งหมด
    /// Update all UI information
    /// </summary>
    public void อัปเดตUI()
    {
        if (battleManager == null) return;

        // อัปเดตข้อมูลผู้เล่น
        if (ผู้เล่นHP != null)
            ผู้เล่นHP.text = $"HP: {battleManager.ผู้เล่น.พลังชีวิตปัจจุบัน}/{battleManager.ผู้เล่น.พลังชีวิตสูงสุด}";

        if (ผู้เล่นMP != null)
            ผู้เล่นMP.text = $"MP: {battleManager.ผู้เล่น.มานาปัจจุบัน}/{battleManager.ผู้เล่น.มานาสูงสุด}";

        // อัปเดตข้อมูลศัตรู
        if (ศัตรูHP != null)
            ศัตรูHP.text = $"HP: {battleManager.ศัตรู.พลังชีวิตปัจจุบัน}/{battleManager.ศัตรู.พลังชีวิตสูงสุด}";

        // อัปเดตเทิร์นปัจจุบัน
        if (เทิร์นปัจจุบัน != null)
        {
            string เทิร์นText = battleManager.สถานะปัจจุบัน == BattleState.เทิร์นผู้เล่น ? "ผู้เล่น" : "ศัตรู";
            เทิร์นปัจจุบัน.text = $"เทิร์น: {เทิร์นText}";
        }

        // อัปเดตปุ่มจบเทิร์น
        if (จบเทิร์นButton != null)
        {
            จบเทิร์นButton.interactable = battleManager.สถานะปัจจุบัน == BattleState.เทิร์นผู้เล่น;
        }
    }

    /// <summary>
    /// จบเทิร์น
    /// End turn
    /// </summary>
    public void จบเทิร์น()
    {
        if (battleManager != null)
        {
            battleManager.จบเทิร์น();
            อัปเดตUI();

            // จั่วการ์ดใหม่สำหรับผู้เล่น
            if (handManager != null)
            {
                handManager.จั่วการ์ด();
            }
        }
    }

    /// <summary>
    /// แสดงเอฟเฟคการเล่นการ์ด
    /// Show card play effect
    /// </summary>
    public void แสดงเอฟเฟคการเล่นการ์ด(Card การ์ดที่เล่น)
    {
        // แสดงอนิเมชันหรือเอฟเฟคพิเศษเมื่อเล่นการ์ด
        Debug.Log($"กำลังเล่นเอฟเฟคสำหรับการ์ด: {การ์ดที่เล่น.ชื่อการ์ด}");

        // ในทางปฏิบัติ คุณอาจจะ:
        // - เล่นอนิเมชัน
        // - แสดง particle effects
        // - สร้างเสียง
        // - เขย่าหน้าจอ
    }
}
```

## การสร้าง Card Prefab

### ขั้นตอนการสร้าง (Creation Steps)

1. **สร้าง Canvas**
   - UI → Canvas (Screen Space - Overlay)

2. **สร้าง Card Prefab**
   - สร้าง Panel เป็นพื้นฐานการ์ด
   - เพิ่ม Image เป็นพื้นหลัง
   - เพิ่ม Text สำหรับชื่อ, คำอธิบาย, มานา

3. **เพิ่ม Components**
   - CardDisplay.cs
   - CardDragHandler.cs
   - CanvasGroup (สำหรับการลาก)

4. **ปรับแต่งรูปร่าง**
   - ขนาด: 120x160 pixels
   - มุมมน: 10px radius
   - เงา: Drop Shadow

## การทดสอบระบบ (Testing the System)

### การตั้งค่าในซีน (Scene Setup)

1. **สร้าง BattleManager** ในซีน
2. **สร้าง HandManager** และกำหนด Card Prefab
3. **สร้าง BattleUI** และเชื่อมต่อกับ BattleManager
4. **สร้าง CardManager** และโหลดการ์ด

### การควบคุมการทดสอบ (Test Controls)

- **เมาส์ซ้าย** - ลากการ์ด
- **เมาส์ขวา** - แสดงรายละเอียดการ์ด
- **Spacebar** - จบเทิร์น
- **Tab** - จั่วการ์ดใหม่

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **ใช้งานง่าย** - ลากวางการ์ด
- **ข้อมูลชัดเจน** - แสดงมานาและเอฟเฟค
- **ปรับแต่งได้** - เปลี่ยนสีและอนิเมชันได้ง่าย
- **ตอบสนองดี** - อัปเดตสถานะแบบเรียลไทม์

### ⚠️ ข้อควรระวัง
- **ประสิทธิภาพ** - การ์ดจำนวนมากอาจทำให้เฟรมเรตต่ำ
- **การตรวจสอบการชน** - ต้องตรวจสอบพื้นที่วางการ์ดให้ถูกต้อง
- **ความเข้ากันได้** - ทดสอบกับหน้าจอขนาดต่างๆ

## แบบฝึกหัด (Exercise)

1. **สร้าง Card Prefab** ที่สวยงาม
2. **เพิ่มอนิเมชัน** เมื่อวางเมาส์บนการ์ด
3. **สร้างเอฟเฟค** เมื่อเล่นการ์ด
4. **เพิ่มเสียง** สำหรับการกระทำต่างๆ

**คุณเข้าใจระบบ UI การ์ดนี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนเรื่อง Enemy AI กันต่อ! 🚀
