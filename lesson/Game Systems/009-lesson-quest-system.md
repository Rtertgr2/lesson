# บทเรียนที่ 009: ระบบภารกิจ (Quest System)

## วัตถุประสงค์ (Objectives)
เรียนรู้การสร้างและจัดการระบบภารกิจที่ยืดหยุ่นและขยายได้

## ภาพรวมระบบ (System Overview)

ระบบภารกิจที่ดีควรมี:
- การกำหนดภารกิจ (Quest Definition)
- การติดตามความคืบหน้า (Progress Tracking)
- การให้รางวัล (Reward System)
- การจัดการหลายภารกิจ (Multiple Quests)
- การแสดงผลใน UI (Quest UI)
- การเซฟความคืบหน้า (Quest Persistence)

## ประเภทภารกิจ (Quest Types)

### 1. Main Quest (ภารกิจหลัก)
- เป็นเรื่องราวหลักของเกม
- ผู้เล่นต้องทำตามลำดับ
- มีผลกับตอนจบเกม

### 2. Side Quest (ภารกิจรอง)
- ไม่จำเป็นต้องทำตามเรื่องราว
- เพิ่มเนื้อหาและรางวัล
- สามารถทำได้หลายอันพร้อมกัน

### 3. Daily Quest (ภารกิจรายวัน)
- รีเซ็ตทุกวัน
- ให้รางวัลพิเศษ
- กระตุ้นให้ผู้เล่นกลับมาเล่น

### 4. Achievement (ความสำเร็จ)
- ไม่มีเวลาและเงื่อนไขพิเศษ
- เก็บสะสมคะแนนหรือสถิติ
- แสดงความสามารถของผู้เล่น

## โครงสร้างระบบ (System Structure)

```
Quest System/
├── QuestManager.cs             # ผู้จัดการภารกิจหลัก
├── Quest.cs                    # คลาสภารกิจพื้นฐาน
├── QuestObjective.cs           # เป้าหมายของภารกิจ
├── QuestReward.cs              # รางวัลของภารกิจ
├── QuestUI.cs                  # จัดการแสดงผลภารกิจ
└── QuestDatabase.cs            # คลังภารกิจทั้งหมด
```

## การติดตั้งใช้งาน (Implementation)

### 1. Quest Class (คลาสภารกิจพื้นฐาน)

```csharp
using UnityEngine;
using System.Collections.Generic;

[System.Serializable]
public class Quest
{
    [Header("ข้อมูลพื้นฐาน - Basic Information")]
    public string questID;                      // รหัสภารกิจ
    public string title;                        // ชื่อภารกิจ
    public string description;                  // คำอธิบายภารกิจ
    public string longDescription;              // คำอธิบายแบบละเอียด

    [Header("การตั้งค่า - Settings")]
    public QuestType questType;                 // ประเภทภารกิจ
    public bool isRepeatable;                   // สามารถทำซ้ำได้หรือไม่
    public int requiredLevel;                   // เลเวลขั้นต่ำที่รับภารกิจได้

    [Header("เป้าหมาย - Objectives")]
    public List<QuestObjective> objectives;     // รายการเป้าหมาย

    [Header("รางวัล - Rewards")]
    public List<QuestReward> rewards;           // รางวัลที่ได้รับ

    [Header("สถานะ - Status")]
    public QuestStatus status;                  // สถานะภารกิจ
    public Dictionary<string, int> objectiveProgress; // ความคืบหน้าของแต่ละเป้าหมาย

    [Header("ข้อมูลเพิ่มเติม - Additional Data")]
    public string npcGiver;                     // NPC ที่ให้ภารกิจ
    public string npcTurnIn;                    // NPC ที่คืนภารกิจ
    public Sprite questIcon;                    // ไอคอนภารกิจ
    public AudioClip completionSound;           // เสียงเมื่อทำเสร็จ

    /// <summary>
    /// สร้างภารกิจใหม่
    /// Create new quest
    /// </summary>
    public Quest(string id, string title, string description)
    {
        this.questID = id;
        this.title = title;
        this.description = description;

        objectives = new List<QuestObjective>();
        rewards = new List<QuestReward>();
        objectiveProgress = new Dictionary<string, int>();
        status = QuestStatus.NotStarted;

        // กำหนดค่าเริ่มต้นให้เป้าหมาย
        foreach (QuestObjective objective in objectives)
        {
            objectiveProgress[objective.objectiveID] = 0;
        }
    }

    /// <summary>
    /// เริ่มภารกิจ
    /// Start quest
    /// </summary>
    public void เริ่มภารกิจ()
    {
        if (status != QuestStatus.NotStarted) return;

        status = QuestStatus.InProgress;

        // รีเซ็ตความคืบหน้าเป้าหมาย
        foreach (QuestObjective objective in objectives)
        {
            objectiveProgress[objective.objectiveID] = 0;
        }

        Debug.Log($"เริ่มภารกิจ: {title}");
    }

    /// <summary>
    /// อัปเดตความคืบหน้าเป้าหมาย
    /// Update objective progress
    /// </summary>
    public void อัปเดตความคืบหน้า(string objectiveID, int newProgress)
    {
        if (status != QuestStatus.InProgress) return;
        if (!objectiveProgress.ContainsKey(objectiveID)) return;

        int oldProgress = objectiveProgress[objectiveID];
        objectiveProgress[objectiveID] = Mathf.Min(newProgress, ดึงเป้าหมาย(objectiveID)?.targetAmount ?? 0);

        // แจ้งการเปลี่ยนแปลง
        OnObjectiveProgressChanged?.Invoke(this, objectiveID, oldProgress, objectiveProgress[objectiveID]);

        // ตรวจสอบว่าภารกิจเสร็จหรือยัง
        ตรวจสอบสถานะภารกิจ();
    }

    /// <summary>
    /// ดึงเป้าหมายตาม ID
    /// Get objective by ID
    /// </summary>
    private QuestObjective ดึงเป้าหมาย(string objectiveID)
    {
        return objectives.Find(obj => obj.objectiveID == objectiveID);
    }

    /// <summary>
    /// ตรวจสอบสถานะภารกิจ
    /// Check quest status
    /// </summary>
    private void ตรวจสอบสถานะภารกิจ()
    {
        bool allObjectivesComplete = true;

        foreach (QuestObjective objective in objectives)
        {
            int currentProgress = objectiveProgress[objective.objectiveID];
            if (currentProgress < objective.targetAmount)
            {
                allObjectivesComplete = false;
                break;
            }
        }

        if (allObjectivesComplete && status == QuestStatus.InProgress)
        {
            เสร็จสิ้นภารกิจ();
        }
    }

    /// <summary>
    /// เสร็จสิ้นภารกิจ
    /// Complete quest
    /// </summary>
    public void เสร็จสิ้นภารกิจ()
    {
        status = QuestStatus.Completed;

        // ให้รางวัล
        ให้รางวัล();

        // แจ้งการเสร็จสิ้น
        OnQuestCompleted?.Invoke(this);

        Debug.Log($"เสร็จสิ้นภารกิจ: {title}");
    }

    /// <summary>
    /// ให้รางวัลแก่ผู้เล่น
    /// Give rewards to player
    /// </summary>
    private void ให้รางวัล()
    {
        foreach (QuestReward reward in rewards)
        {
            reward.ให้รางวัล();
        }
    }

    /// <summary>
    /// ล้มเหลวภารกิจ
    /// Fail quest
    /// </summary>
    public void ล้มเหลวภารกิจ()
    {
        status = QuestStatus.Failed;
        OnQuestFailed?.Invoke(this);
        Debug.Log($"ล้มเหลวภารกิจ: {title}");
    }

    /// <summary>
    /// ยกเลิกภารกิจ
    /// Abandon quest
    /// </summary>
    public void ยกเลิกภารกิจ()
    {
        status = QuestStatus.Abandoned;
        OnQuestAbandoned?.Invoke(this);
        Debug.Log($"ยกเลิกภารกิจ: {title}");
    }

    /// <summary>
    /// ตรวจสอบว่าภารกิจสามารถรับได้หรือไม่
    /// Check if quest can be accepted
    /// </summary>
    public bool สามารถรับได้(int playerLevel)
    {
        return playerLevel >= requiredLevel && status == QuestStatus.NotStarted;
    }

    /// <summary>
    /// ตรวจสอบว่าภารกิจสามารถคืนได้หรือไม่
    /// Check if quest can be turned in
    /// </summary>
    public bool สามารถคืนได้()
    {
        return status == QuestStatus.Completed;
    }

    /// <summary>
    /// คัดลอกภารกิจ (สำหรับสร้าง instance ใหม่)
    /// Clone quest (for creating new instance)
    /// </summary>
    public Quest คัดลอก()
    {
        Quest clone = new Quest(questID, title, description);
        clone.longDescription = this.longDescription;
        clone.questType = this.questType;
        clone.isRepeatable = this.isRepeatable;
        clone.requiredLevel = this.requiredLevel;
        clone.objectives = new List<QuestObjective>(this.objectives);
        clone.rewards = new List<QuestReward>(this.rewards);
        clone.npcGiver = this.npcGiver;
        clone.npcTurnIn = this.npcTurnIn;
        clone.questIcon = this.questIcon;
        clone.completionSound = this.completionSound;

        return clone;
    }

    // Events สำหรับการแจ้งเตือน
    public delegate void QuestEvent(Quest quest);
    public static event QuestEvent OnQuestCompleted;
    public static event QuestEvent OnQuestFailed;
    public static event QuestEvent OnQuestAbandoned;

    public delegate void ObjectiveProgressEvent(Quest quest, string objectiveID, int oldProgress, int newProgress);
    public static event ObjectiveProgressEvent OnObjectiveProgressChanged;
}

/// <summary>
/// ประเภทของภารกิจ
/// Quest types
/// </summary>
public enum QuestType
{
    Main,       // ภารกิจหลัก
    Side,       // ภารกิจรอง
    Daily,      // ภารกิจรายวัน
    Achievement // ความสำเร็จ
}

/// <summary>
/// สถานะของภารกิจ
/// Quest status
/// </summary>
public enum QuestStatus
{
    NotStarted, // ยังไม่เริ่ม
    InProgress, // กำลังทำ
    Completed,  // เสร็จแล้ว
    Failed,     // ล้มเหลว
    Abandoned   // ยกเลิก
}
```

### 2. Quest Objective (เป้าหมายภารกิจ)

```csharp
using UnityEngine;

[System.Serializable]
public class QuestObjective
{
    public string objectiveID;                  // รหัสเป้าหมาย
    public string description;                  // คำอธิบายเป้าหมาย
    public ObjectiveType objectiveType;         // ประเภทเป้าหมาย
    public int targetAmount;                    // จำนวนเป้าหมาย

    [Header("ข้อมูลเพิ่มเติม - Additional Data")]
    public string targetTag;                    // แท็กของวัตถุเป้าหมาย
    public string targetName;                   // ชื่อวัตถุเป้าหมาย
    public Vector3 targetLocation;              // ตำแหน่งเป้าหมาย

    /// <summary>
    /// สร้างเป้าหมายใหม่
    /// Create new objective
    /// </summary>
    public QuestObjective(string id, string description, ObjectiveType type, int target)
    {
        this.objectiveID = id;
        this.description = description;
        this.objectiveType = type;
        this.targetAmount = target;
    }

    /// <summary>
    /// ตรวจสอบว่าเป้าหมายนี้เสร็จแล้วหรือไม่
    /// Check if this objective is completed
    /// </summary>
    public bool เสร็จแล้ว(int currentProgress)
    {
        return currentProgress >= targetAmount;
    }

    /// <summary>
    /// คำนวณเปอร์เซ็นต์ความคืบหน้า
    /// Calculate progress percentage
    /// </summary>
    public float คำนวณเปอร์เซ็นต์(int currentProgress)
    {
        if (targetAmount <= 0) return 100f;
        return Mathf.Min((float)currentProgress / targetAmount * 100f, 100f);
    }
}

/// <summary>
/// ประเภทของเป้าหมาย
/// Objective types
/// </summary>
public enum ObjectiveType
{
    KillEnemies,        // ฆ่าศัตรู
    CollectItems,       // เก็บไอเทม
    ReachLocation,      // ไปถึงสถานที่
    TalkToNPC,          // คุยกับ NPC
    UseItem,            // ใช้ไอเทม
    CompleteLevel,      // ผ่านด่าน
    AchieveScore,       // ได้คะแนน
    SurviveTime,        // รอดชีวิต
    Custom              // กำหนดเอง
}
```

### 3. Quest Reward (รางวัลภารกิจ)

```csharp
using UnityEngine;

[System.Serializable]
public class QuestReward
{
    public RewardType rewardType;               // ประเภทรางวัล
    public string itemID;                       // รหัสไอเทม
    public int amount;                          // จำนวน
    public int experience;                      // ประสบการณ์
    public int gold;                           // ทอง

    [Header("ข้อมูลเพิ่มเติม - Additional Data")]
    public string description;                  // คำอธิบายรางวัล

    /// <summary>
    /// ให้รางวัลแก่ผู้เล่น
    /// Give reward to player
    /// </summary>
    public void ให้รางวัล()
    {
        switch (rewardType)
        {
            case RewardType.Item:
                ให้ไอเทม();
                break;
            case RewardType.Experience:
                ให้ประสบการณ์();
                break;
            case RewardType.Gold:
                ให้ทอง();
                break;
            case RewardType.UnlockContent:
                ปลดล็อคเนื้อหา();
                break;
        }

        Debug.Log($"ให้รางวัล: {description}");
    }

    private void ให้ไอเทม()
    {
        // เพิ่มไอเทมให้ผู้เล่น
        InventoryManager inventory = FindObjectOfType<InventoryManager>();
        if (inventory != null)
        {
            inventory.AddItem(itemID, amount);
        }
    }

    private void ให้ประสบการณ์()
    {
        // เพิ่มประสบการณ์ให้ผู้เล่น
        PlayerStats playerStats = FindObjectOfType<PlayerStats>();
        if (playerStats != null)
        {
            playerStats.AddExperience(experience);
        }
    }

    private void ให้ทอง()
    {
        // เพิ่มทองให้ผู้เล่น
        PlayerStats playerStats = FindObjectOfType<PlayerStats>();
        if (playerStats != null)
        {
            playerStats.AddGold(gold);
        }
    }

    private void ปลดล็อคเนื้อหา()
    {
        // ปลดล็อคเนื้อหาใหม่
        ContentManager contentManager = FindObjectOfType<ContentManager>();
        if (contentManager != null)
        {
            contentManager.UnlockContent(itemID);
        }
    }
}

/// <summary>
/// ประเภทรางวัล
/// Reward types
/// </summary>
public enum RewardType
{
    Item,           // ไอเทม
    Experience,     // ประสบการณ์
    Gold,           // ทอง
    UnlockContent   // ปลดล็อคเนื้อหา
}
```

### 4. Quest Manager (ผู้จัดการภารกิจ)

```csharp
using UnityEngine;
using System.Collections.Generic;
using System.Linq;

public class QuestManager : MonoBehaviour
{
    [Header("ภารกิจที่รับแล้ว - Accepted Quests")]
    public List<Quest> activeQuests;            // ภารกิจที่กำลังทำ
    public List<Quest> completedQuests;         // ภารกิจที่ทำเสร็จแล้ว

    [Header("การตั้งค่า - Settings")]
    public int maxActiveQuests = 5;             // จำนวนภารกิจสูงสุดที่รับได้พร้อมกัน

    private static QuestManager instance;       // Instance เดียว

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    void Start()
    {
        activeQuests = new List<Quest>();
        completedQuests = new List<Quest>();
    }

    /// <summary>
    /// รับภารกิจใหม่
    /// Accept new quest
    /// </summary>
    public bool รับภารกิจ(Quest quest)
    {
        if (activeQuests.Count >= maxActiveQuests)
        {
            Debug.LogWarning("มีภารกิจมากเกินไปแล้ว");
            return false;
        }

        if (quest.สามารถรับได้(ดึงเลเวลผู้เล่น()))
        {
            activeQuests.Add(quest);
            quest.เริ่มภารกิจ();

            OnQuestAccepted?.Invoke(quest);
            return true;
        }

        return false;
    }

    /// <summary>
    /// คืนภารกิจที่เสร็จแล้ว
    /// Turn in completed quest
    /// </summary>
    public void คืนภารกิจ(Quest quest)
    {
        if (quest.สามารถคืนได้())
        {
            activeQuests.Remove(quest);
            completedQuests.Add(quest);

            OnQuestTurnedIn?.Invoke(quest);
        }
    }

    /// <summary>
    /// อัปเดตความคืบหน้าภารกิจ
    /// Update quest progress
    /// </summary>
    public void อัปเดตความคืบหน้า(string objectiveID, int progress)
    {
        foreach (Quest quest in activeQuests)
        {
            quest.อัปเดตความคืบหน้า(objectiveID, progress);
        }
    }

    /// <summary>
    /// อัปเดตความคืบหน้าตามประเภทเป้าหมาย
    /// Update progress by objective type
    /// </summary>
    public void อัปเดตความคืบหน้าตามประเภท(ObjectiveType objectiveType, string targetID, int amount = 1)
    {
        foreach (Quest quest in activeQuests)
        {
            foreach (QuestObjective objective in quest.objectives)
            {
                if (objective.objectiveType == objectiveType)
                {
                    bool matchesTarget = false;

                    switch (objectiveType)
                    {
                        case ObjectiveType.KillEnemies:
                            matchesTarget = objective.targetTag == targetID;
                            break;
                        case ObjectiveType.CollectItems:
                            matchesTarget = objective.targetName == targetID;
                            break;
                        case ObjectiveType.ReachLocation:
                            matchesTarget = Vector3.Distance(objective.targetLocation, ดึงตำแหน่งปัจจุบัน()) < 5f;
                            break;
                    }

                    if (matchesTarget)
                    {
                        quest.อัปเดตความคืบหน้า(objective.objectiveID, amount);
                    }
                }
            }
        }
    }

    /// <summary>
    /// ดึงเลเวลผู้เล่น (จำลอง)
    /// Get player level (simulated)
    /// </summary>
    private int ดึงเลเวลผู้เล่น()
    {
        PlayerStats playerStats = FindObjectOfType<PlayerStats>();
        return playerStats != null ? playerStats.level : 1;
    }

    /// <summary>
    /// ดึงตำแหน่งผู้เล่นปัจจุบัน (จำลอง)
    /// Get current player position (simulated)
    /// </summary>
    private Vector3 ดึงตำแหน่งปัจจุบัน()
    {
        GameObject player = GameObject.FindGameObjectWithTag("Player");
        return player != null ? player.transform.position : Vector3.zero;
    }

    /// <summary>
    /// ดึงภารกิจที่กำลังทำอยู่
    /// Get active quests
    /// </summary>
    public List<Quest> ดึงภารกิจที่กำลังทำ()
    {
        return activeQuests;
    }

    /// <summary>
    /// ดึงภารกิจที่เสร็จแล้ว
    /// Get completed quests
    /// </summary>
    public List<Quest> ดึงภารกิจที่เสร็จแล้ว()
    {
        return completedQuests;
    }

    /// <summary>
    /// ดึงภารกิจตาม ID
    /// Get quest by ID
    /// </summary>
    public Quest ดึงภารกิจตามID(string questID)
    {
        return activeQuests.Find(q => q.questID == questID) ??
               completedQuests.Find(q => q.questID == questID);
    }

    /// <summary>
    /// ดึงภารกิจตามประเภท
    /// Get quests by type
    /// </summary>
    public List<Quest> ดึงภารกิจตามประเภท(QuestType questType)
    {
        return activeQuests.Where(q => q.questType == questType).ToList();
    }

    /// <summary>
    /// ล้างภารกิจทั้งหมด (สำหรับเริ่มเกมใหม่)
    /// Clear all quests (for new game)
    /// </summary>
    public void ล้างภารกิจทั้งหมด()
    {
        activeQuests.Clear();
        completedQuests.Clear();
    }

    // Events สำหรับการแจ้งเตือน
    public delegate void QuestListEvent(List<Quest> quests);
    public static event QuestListEvent OnActiveQuestsChanged;

    public delegate void SingleQuestEvent(Quest quest);
    public static event SingleQuestEvent OnQuestAccepted;
    public static event SingleQuestEvent OnQuestTurnedIn;
}
```

### 5. Quest Database (คลังภารกิจ)

```csharp
using UnityEngine;
using System.Collections.Generic;
using System.Linq;

public class QuestDatabase : MonoBehaviour
{
    [Header("ภารกิจทั้งหมด - All Quests")]
    public List<Quest> allQuests;               // ภารกิจทั้งหมดในเกม

    private static QuestDatabase instance;      // Instance เดียว

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
        }
        else
        {
            Destroy(gameObject);
        }
    }

    /// <summary>
    /// ดึงภารกิจตาม ID
    /// Get quest by ID
    /// </summary>
    public Quest ดึงภารกิจตามID(string questID)
    {
        return allQuests.Find(q => q.questID == questID)?.คัดลอก();
    }

    /// <summary>
    /// ดึงภารกิจทั้งหมดที่รับได้ตามเลเวล
    /// Get all available quests for player level
    /// </summary>
    public List<Quest> ดึงภารกิจที่รับได้(int playerLevel)
    {
        return allQuests.Where(q => q.สามารถรับได้(playerLevel)).ToList();
    }

    /// <summary>
    /// ดึงภารกิจตามประเภท
    /// Get quests by type
    /// </summary>
    public List<Quest> ดึงภารกิจตามประเภท(QuestType questType)
    {
        return allQuests.Where(q => q.questType == questType).ToList();
    }

    /// <summary>
    /// ดึงภารกิจจาก NPC
    /// Get quests from NPC
    /// </summary>
    public List<Quest> ดึงภารกิจจากNPC(string npcName)
    {
        return allQuests.Where(q => q.npcGiver == npcName).ToList();
    }

    /// <summary>
    /// เพิ่มภารกิจใหม่เข้าไปในฐานข้อมูล
    /// Add new quest to database
    /// </summary>
    public void เพิ่มภารกิจ(Quest quest)
    {
        if (!allQuests.Exists(q => q.questID == quest.questID))
        {
            allQuests.Add(quest);
        }
    }

    /// <summary>
    /// ลบภารกิจออกจากฐานข้อมูล
    /// Remove quest from database
    /// </summary>
    public void ลบภารกิจ(string questID)
    {
        allQuests.RemoveAll(q => q.questID == questID);
    }
}
```

## การใช้งานจริง (Practical Usage)

### การสร้างภารกิจตัวอย่าง (Example Quest Creation)

```csharp
public class QuestExample : MonoBehaviour
{
    private QuestManager questManager;
    private QuestDatabase questDatabase;

    void Start()
    {
        questManager = FindObjectOfType<QuestManager>();
        questDatabase = FindObjectOfType<QuestDatabase>();

        สร้างภารกิจตัวอย่าง();
    }

    /// <summary>
    /// สร้างภารกิจตัวอย่าง
    /// Create example quests
    /// </summary>
    private void สร้างภารกิจตัวอย่าง()
    {
        // สร้างภารกิจหลัก
        Quest mainQuest = new Quest("MQ_001", "ช่วยเหลือหมู่บ้าน", "ช่วยชาวบ้านจากมอนสเตอร์ร้าย");

        // เพิ่มเป้าหมาย
        mainQuest.objectives.Add(new QuestObjective("OBJ_001", "ฆ่ามอนสเตอร์ 10 ตัว", ObjectiveType.KillEnemies, 10));
        mainQuest.objectives.Add(new QuestObjective("OBJ_002", "คุยกับหัวหน้าหมู่บ้าน", ObjectiveType.TalkToNPC, 1));

        // เพิ่มรางวัล
        mainQuest.rewards.Add(new QuestReward { rewardType = RewardType.Experience, experience = 1000 });
        mainQuest.rewards.Add(new QuestReward { rewardType = RewardType.Gold, gold = 500 });

        // ตั้งค่าอื่นๆ
        mainQuest.questType = QuestType.Main;
        mainQuest.requiredLevel = 1;
        mainQuest.npcGiver = "VillageChief";

        // เพิ่มเข้าไปในฐานข้อมูล
        if (questDatabase != null)
        {
            questDatabase.เพิ่มภารกิจ(mainQuest);
        }

        // สร้างภารกิจรอง
        Quest sideQuest = new Quest("SQ_001", "เก็บเห็ดวิเศษ", "เก็บเห็ดวิเศษ 5 ดอกให้กับนักเล่นแร่แปรธาตุ");

        sideQuest.objectives.Add(new QuestObjective("OBJ_003", "เก็บเห็ดวิเศษ 5 ดอก", ObjectiveType.CollectItems, 5));
        sideQuest.rewards.Add(new QuestReward { rewardType = RewardType.Item, itemID = "MagicHerb", amount = 1 });
        sideQuest.questType = QuestType.Side;
        sideQuest.requiredLevel = 3;

        questDatabase?.เพิ่มภารกิจ(sideQuest);
    }
}
```

## การแสดงผลใน UI (Quest UI)

### Quest UI Manager

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;
using System.Collections.Generic;

public class QuestUI : MonoBehaviour
{
    [Header("การอ้างอิง UI - UI References")]
    public GameObject questPanel;               // แผงภารกิจหลัก
    public GameObject questItemPrefab;          // Prefab รายการภารกิจ
    public Transform questListContainer;        // ที่วางรายการภารกิจ
    public TextMeshProUGUI questTitle;          // ชื่อภารกิจที่เลือก
    public TextMeshProUGUI questDescription;    // คำอธิบายภารกิจ
    public Slider questProgressBar;             // แถบความคืบหน้า
    public TextMeshProUGUI progressText;        // ข้อความความคืบหน้า

    [Header("ปุ่ม - Buttons")]
    public Button acceptQuestButton;            // ปุ่มรับภารกิจ
    public Button abandonQuestButton;           // ปุ่มยกเลิกภารกิจ
    public Button trackQuestButton;             // ปุ่มติดตามภารกิจ

    private Quest selectedQuest;                // ภารกิจที่เลือก
    private List<GameObject> questItems = new List<GameObject>(); // รายการ UI ภารกิจ

    void Start()
    {
        อัปเดตรายการภารกิจ();
    }

    /// <summary>
    /// อัปเดตแสดงรายการภารกิจ
    /// Update quest list display
    /// </summary>
    public void อัปเดตรายการภารกิจ()
    {
        // ล้างรายการเดิม
        foreach (GameObject item in questItems)
        {
            Destroy(item);
        }
        questItems.Clear();

        // สร้างรายการใหม่
        QuestManager questManager = FindObjectOfType<QuestManager>();
        if (questManager != null)
        {
            foreach (Quest quest in questManager.ดึงภารกิจที่กำลังทำ())
            {
                สร้างรายการภารกิจ(quest);
            }
        }
    }

    /// <summary>
    /// สร้างรายการภารกิจใน UI
    /// Create quest item in UI
    /// </summary>
    private void สร้างรายการภารกิจ(Quest quest)
    {
        GameObject questItem = Instantiate(questItemPrefab, questListContainer);
        QuestItemUI itemUI = questItem.GetComponent<QuestItemUI>();

        if (itemUI != null)
        {
            itemUI.แสดงภารกิจ(quest);
            itemUI.onQuestSelected += เลือกภารกิจ;
        }

        questItems.Add(questItem);
    }

    /// <summary>
    /// เลือกภารกิจ
    /// Select quest
    /// </summary>
    private void เลือกภารกิจ(Quest quest)
    {
        selectedQuest = quest;
        แสดงรายละเอียดภารกิจ(quest);
    }

    /// <summary>
    /// แสดงรายละเอียดภารกิจ
    /// Show quest details
    /// </summary>
    private void แสดงรายละเอียดภารกิจ(Quest quest)
    {
        if (questTitle != null) questTitle.text = quest.title;
        if (questDescription != null) questDescription.text = quest.longDescription;

        // คำนวณความคืบหน้ารวม
        float totalProgress = คำนวณความคืบหน้ารวม(quest);

        if (questProgressBar != null)
        {
            questProgressBar.value = totalProgress / 100f;
        }

        if (progressText != null)
        {
            progressText.text = $"{Mathf.RoundToInt(totalProgress)}%";
        }

        // อัปเดตปุ่ม
        อัปเดตปุ่ม(quest);
    }

    /// <summary>
    /// คำนวณความคืบหน้ารวมของภารกิจ
    /// Calculate total quest progress
    /// </summary>
    private float คำนวณความคืบหน้ารวม(Quest quest)
    {
        if (quest.objectives.Count == 0) return 0f;

        float totalProgress = 0f;

        foreach (QuestObjective objective in quest.objectives)
        {
            int currentProgress = quest.objectiveProgress[objective.objectiveID];
            totalProgress += objective.คำนวณเปอร์เซ็นต์(currentProgress);
        }

        return totalProgress / quest.objectives.Count;
    }

    /// <summary>
    /// อัปเดตปุ่มตามสถานะภารกิจ
    /// Update buttons based on quest status
    /// </summary>
    private void อัปเดตปุ่ม(Quest quest)
    {
        if (acceptQuestButton != null)
            acceptQuestButton.gameObject.SetActive(quest.status == QuestStatus.NotStarted);

        if (abandonQuestButton != null)
            abandonQuestButton.gameObject.SetActive(quest.status == QuestStatus.InProgress);

        if (trackQuestButton != null)
            trackQuestButton.gameObject.SetActive(quest.status == QuestStatus.InProgress);
    }

    /// <summary>
    /// รับภารกิจที่เลือก
    /// Accept selected quest
    /// </summary>
    public void รับภารกิจที่เลือก()
    {
        if (selectedQuest != null)
        {
            QuestManager questManager = FindObjectOfType<QuestManager>();
            if (questManager != null)
            {
                questManager.รับภารกิจ(selectedQuest);
                อัปเดตรายการภารกิจ();
            }
        }
    }

    /// <summary>
    /// ยกเลิกภารกิจที่เลือก
    /// Abandon selected quest
    /// </summary>
    public void ยกเลิกภารกิจที่เลือก()
    {
        if (selectedQuest != null)
        {
            selectedQuest.ยกเลิกภารกิจ();
            อัปเดตรายการภารกิจ();
        }
    }
}
```

## การทดสอบระบบ (Testing the System)

### การควบคุมการทดสอบ (Test Controls)

- **NPC Interaction** - รับภารกิจจาก NPC
- **Quest Journal** - ดูภารกิจทั้งหมด
- **Objective Updates** - อัปเดตความคืบหน้า

### สิ่งที่ควรสังเกต (What to Observe)

- **การรับภารกิจ** - สถานะเปลี่ยนจาก NotStarted เป็น InProgress
- **การอัปเดตเป้าหมาย** - ความคืบหน้าเพิ่มขึ้น
- **การเสร็จสิ้นภารกิจ** - ได้รับรางวัลและสถานะเป็น Completed
- **การแสดงผล UI** - ข้อมูลภารกิจแสดงถูกต้อง

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **ยืดหยุ่น** - รองรับหลายประเภทภารกิจ
- **ขยายได้** - เพิ่มเป้าหมายและรางวัลได้ง่าย
- **ติดตามได้** - รู้ความคืบหน้าทุกเป้าหมาย
- **เซฟได้** - ความคืบหน้ายังอยู่เมื่อโหลดเกม

### ⚠️ ข้อควรระวัง
- **สมดุล** - ภารกิจไม่ยากหรือง่ายเกินไป
- **ความชัดเจน** - ผู้เล่นเข้าใจเป้าหมาย
- **ประสิทธิภาพ** - การอัปเดตหลายภารกิจ

## แบบฝึกหัด (Exercise)

1. **สร้างภารกิจหลัก** ที่มีหลายเป้าหมาย
2. **สร้างภารกิจรอง** ที่ไม่เกี่ยวข้องกับเรื่องหลัก
3. **สร้างระบบรางวัล** ที่ให้ไอเทมและประสบการณ์
4. **สร้าง UI แสดงภารกิจ** ที่สวยงามและใช้งานง่าย

**คุณเข้าใจระบบภารกิจนี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนหัวข้อถัดไปกัน! 🚀
