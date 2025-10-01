# บทเรียนที่ 008: ระบบเซฟ/โหลด (Save/Load System)

## วัตถุประสงค์ (Objectives)
เรียนรู้การสร้างระบบเซฟและโหลดข้อมูลเกมที่ปลอดภัยและมีประสิทธิภาพ

## ภาพรวมระบบ (System Overview)

ระบบเซฟ/โหลดที่ดีควรมี:
- การบันทึกข้อมูลผู้เล่น (Player Data)
- การบันทึกความคืบหน้าเกม (Game Progress)
- การบันทึกการตั้งค่า (Settings)
- การเข้ารหัสข้อมูล (Data Encryption)
- การตรวจสอบความสมบูรณ์ (Data Integrity)
- การจัดการหลายเซฟ (Multiple Save Slots)

## โครงสร้างระบบ (System Structure)

```
Save/Load System/
├── SaveLoadManager.cs          # ผู้จัดการเซฟ/โหลดหลัก
├── SaveData.cs                 # โครงสร้างข้อมูลเซฟ
├── EncryptionManager.cs        # จัดการการเข้ารหัส
├── SaveSlot.cs                 # จัดการช่องเซฟ
└── AutoSave.cs                 # เซฟอัตโนมัติ
```

## ประเภทข้อมูลที่เซฟ (Save Data Types)

### 1. Player Data (ข้อมูลผู้เล่น)
- ชื่อผู้เล่น
- ระดับเลเวล
- สถิติต่าง ๆ
- สกิน/ไอเทมที่ปลดล็อก

### 2. Game Progress (ความคืบหน้าเกม)
- ด่านที่ผ่านแล้ว
- ภารกิจที่ทำแล้ว
- คะแนนสูงสุด
- ความสำเร็จที่ได้

### 3. Settings (การตั้งค่า)
- ความดังเสียง
- ความละเอียดหน้าจอ
- การควบคุม
- การตั้งค่าภาษา

## การติดตั้งใช้งาน (Implementation)

### 1. Save Data Structure

```csharp
using System;
using System.Collections.Generic;

[Serializable]
public class SaveData
{
    [Header("ข้อมูลผู้เล่น - Player Data")]
    public string playerName = "Player";
    public int level = 1;
    public int experience = 0;
    public int maxLevel = 100;

    [Header("ความคืบหน้าเกม - Game Progress")]
    public List<string> completedLevels = new List<string>();
    public List<string> completedQuests = new List<string>();
    public Dictionary<string, int> highScores = new Dictionary<string, int>();
    public List<string> unlockedAchievements = new List<string>();

    [Header("การตั้งค่า - Settings")]
    public float masterVolume = 1f;
    public float musicVolume = 0.7f;
    public float sfxVolume = 0.8f;
    public int screenResolutionIndex = 0;
    public bool fullscreen = true;

    [Header("ข้อมูลเกม - Game Data")]
    public int totalPlayTime = 0;           // เวลาเล่นทั้งหมด (วินาที)
    public int gamesPlayed = 0;             // จำนวนเกมที่เล่น
    public DateTime lastPlayDate;           // วันที่เล่นล่าสุด
    public string currentScene = "MainMenu"; // ฉากปัจจุบัน

    [Header("ข้อมูลเพิ่มเติม - Additional Data")]
    public Dictionary<string, object> customData = new Dictionary<string, object>();

    /// <summary>
    /// สร้างข้อมูลเซฟใหม่
    /// Create new save data
    /// </summary>
    public static SaveData สร้างใหม่()
    {
        SaveData newSave = new SaveData();
        newSave.lastPlayDate = DateTime.Now;
        return newSave;
    }

    /// <summary>
    /// คัดลอกข้อมูลเซฟ
    /// Clone save data
    /// </summary>
    public SaveData คัดลอก()
    {
        SaveData clone = new SaveData();

        // คัดลอกข้อมูลพื้นฐาน
        clone.playerName = this.playerName;
        clone.level = this.level;
        clone.experience = this.experience;
        clone.maxLevel = this.maxLevel;

        // คัดลอกลิสต์
        clone.completedLevels = new List<string>(this.completedLevels);
        clone.completedQuests = new List<string>(this.completedQuests);
        clone.unlockedAchievements = new List<string>(this.unlockedAchievements);

        // คัดลอกดิกชันนารี
        clone.highScores = new Dictionary<string, int>(this.highScores);

        // คัดลอกการตั้งค่า
        clone.masterVolume = this.masterVolume;
        clone.musicVolume = this.musicVolume;
        clone.sfxVolume = this.sfxVolume;
        clone.screenResolutionIndex = this.screenResolutionIndex;
        clone.fullscreen = this.fullscreen;

        // คัดลอกข้อมูลเกม
        clone.totalPlayTime = this.totalPlayTime;
        clone.gamesPlayed = this.gamesPlayed;
        clone.lastPlayDate = this.lastPlayDate;
        clone.currentScene = this.currentScene;

        // คัดลอกข้อมูลเพิ่มเติม
        foreach (var kvp in this.customData)
        {
            clone.customData[kvp.Key] = kvp.Value;
        }

        return clone;
    }

    /// <summary>
    /// รวมข้อมูลเซฟ (สำหรับอัปเดตบางส่วน)
    /// Merge save data (for partial updates)
    /// </summary>
    public void รวมกับ(SaveData other)
    {
        if (other == null) return;

        // อัปเดตข้อมูลผู้เล่น
        if (!string.IsNullOrEmpty(other.playerName))
            this.playerName = other.playerName;

        this.level = Mathf.Max(this.level, other.level);
        this.experience = Mathf.Max(this.experience, other.experience);

        // รวมลิสต์ (ไม่ซ้ำกัน)
        foreach (string level in other.completedLevels)
        {
            if (!this.completedLevels.Contains(level))
                this.completedLevels.Add(level);
        }

        foreach (string quest in other.completedQuests)
        {
            if (!this.completedQuests.Contains(quest))
                this.completedQuests.Add(quest);
        }

        foreach (string achievement in other.unlockedAchievements)
        {
            if (!this.unlockedAchievements.Contains(achievement))
                this.unlockedAchievements.Add(achievement);
        }

        // อัปเดตคะแนนสูงสุด
        foreach (var kvp in other.highScores)
        {
            if (this.highScores.ContainsKey(kvp.Key))
            {
                this.highScores[kvp.Key] = Mathf.Max(this.highScores[kvp.Key], kvp.Value);
            }
            else
            {
                this.highScores[kvp.Key] = kvp.Value;
            }
        }

        // อัปเดตการตั้งค่า
        this.masterVolume = other.masterVolume;
        this.musicVolume = other.musicVolume;
        this.sfxVolume = other.sfxVolume;
        this.screenResolutionIndex = other.screenResolutionIndex;
        this.fullscreen = other.fullscreen;

        // อัปเดตข้อมูลเกม
        this.totalPlayTime = Mathf.Max(this.totalPlayTime, other.totalPlayTime);
        this.gamesPlayed = Mathf.Max(this.gamesPlayed, other.gamesPlayed);
        this.lastPlayDate = other.lastPlayDate;

        if (!string.IsNullOrEmpty(other.currentScene))
            this.currentScene = other.currentScene;
    }
}
```

### 2. Save/Load Manager หลัก

```csharp
using UnityEngine;
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;
using System.Collections.Generic;

public class SaveLoadManager : MonoBehaviour
{
    [Header("การตั้งค่า - Settings")]
    public string saveFolderName = "SaveData";  // ชื่อโฟลเดอร์เซฟ
    public bool useEncryption = true;           // ใช้การเข้ารหัสหรือไม่
    public int maxSaveSlots = 3;                // จำนวนช่องเซฟสูงสุด

    [Header("สถานะ - Status")]
    private string savePath;                    // พาธโฟลเดอร์เซฟ
    private EncryptionManager encryptionManager; // ผู้จัดการเข้ารหัส

    void Awake()
    {
        กำหนดพาธเซฟ();
        encryptionManager = GetComponent<EncryptionManager>();

        if (encryptionManager == null && useEncryption)
        {
            encryptionManager = gameObject.AddComponent<EncryptionManager>();
        }
    }

    /// <summary>
    /// กำหนดพาธสำหรับเก็บเซฟ
    /// Set save path
    /// </summary>
    private void กำหนดพาธเซฟ()
    {
        savePath = Path.Combine(Application.persistentDataPath, saveFolderName);

        if (!Directory.Exists(savePath))
        {
            Directory.CreateDirectory(savePath);
        }
    }

    /// <summary>
    /// เซฟเกมไปยังช่องที่กำหนด
    /// Save game to specified slot
    /// </summary>
    public bool เซฟเกม(int slotIndex, SaveData saveData)
    {
        if (slotIndex < 0 || slotIndex >= maxSaveSlots)
        {
            Debug.LogError($"ดัชนีช่องเซฟไม่ถูกต้อง: {slotIndex}");
            return false;
        }

        try
        {
            string fileName = $"save_{slotIndex}.dat";
            string filePath = Path.Combine(savePath, fileName);

            // แปลงข้อมูลเป็น JSON
            string jsonData = JsonUtility.ToJson(saveData, true);

            // เข้ารหัสข้อมูล (ถ้าเปิดใช้งาน)
            string finalData = useEncryption && encryptionManager != null
                ? encryptionManager.เข้ารหัส(jsonData)
                : jsonData;

            // บันทึกไฟล์
            File.WriteAllText(filePath, finalData);

            Debug.Log($"เซฟเกมสำเร็จ ช่อง: {slotIndex}");
            return true;
        }
        catch (System.Exception e)
        {
            Debug.LogError($"เกิดข้อผิดพลาดในการเซฟเกม: {e.Message}");
            return false;
        }
    }

    /// <summary>
    /// โหลดเกมจากช่องที่กำหนด
    /// Load game from specified slot
    /// </summary>
    public SaveData โหลดเกม(int slotIndex)
    {
        if (slotIndex < 0 || slotIndex >= maxSaveSlots)
        {
            Debug.LogError($"ดัชนีช่องเซฟไม่ถูกต้อง: {slotIndex}");
            return null;
        }

        try
        {
            string fileName = $"save_{slotIndex}.dat";
            string filePath = Path.Combine(savePath, fileName);

            if (!File.Exists(filePath))
            {
                Debug.LogWarning($"ไม่พบไฟล์เซฟ ช่อง: {slotIndex}");
                return null;
            }

            // อ่านข้อมูลจากไฟล์
            string fileData = File.ReadAllText(filePath);

            // ถอดรหัสข้อมูล (ถ้าเข้ารหัสไว้)
            string jsonData = useEncryption && encryptionManager != null
                ? encryptionManager.ถอดรหัส(fileData)
                : fileData;

            // แปลงจาก JSON เป็น SaveData
            SaveData loadedData = JsonUtility.FromJson<SaveData>(jsonData);

            if (loadedData != null)
            {
                Debug.Log($"โหลดเกมสำเร็จ ช่อง: {slotIndex}");
            }

            return loadedData;
        }
        catch (System.Exception e)
        {
            Debug.LogError($"เกิดข้อผิดพลาดในการโหลดเกม: {e.Message}");
            return null;
        }
    }

    /// <summary>
    /// ลบเซฟจากช่องที่กำหนด
    /// Delete save from specified slot
    /// </summary>
    public bool ลบเซฟ(int slotIndex)
    {
        if (slotIndex < 0 || slotIndex >= maxSaveSlots)
        {
            Debug.LogError($"ดัชนีช่องเซฟไม่ถูกต้อง: {slotIndex}");
            return false;
        }

        try
        {
            string fileName = $"save_{slotIndex}.dat";
            string filePath = Path.Combine(savePath, fileName);

            if (File.Exists(filePath))
            {
                File.Delete(filePath);
                Debug.Log($"ลบเซฟสำเร็จ ช่อง: {slotIndex}");
                return true;
            }
            else
            {
                Debug.LogWarning($"ไม่พบไฟล์เซฟที่จะลบ ช่อง: {slotIndex}");
                return false;
            }
        }
        catch (System.Exception e)
        {
            Debug.LogError($"เกิดข้อผิดพลาดในการลบเซฟ: {e.Message}");
            return false;
        }
    }

    /// <summary>
    /// ตรวจสอบว่าช่องเซฟมีข้อมูลหรือไม่
    /// Check if save slot has data
    /// </summary>
    public bool มีเซฟในช่อง(int slotIndex)
    {
        string fileName = $"save_{slotIndex}.dat";
        string filePath = Path.Combine(savePath, fileName);
        return File.Exists(filePath);
    }

    /// <summary>
    /// ดึงข้อมูลเซฟทั้งหมด (สำหรับแสดงใน UI)
    /// Get all save data (for UI display)
    /// </summary>
    public List<SaveSlotInfo> ดึงข้อมูลเซฟทั้งหมด()
    {
        List<SaveSlotInfo> slotInfos = new List<SaveSlotInfo>();

        for (int i = 0; i < maxSaveSlots; i++)
        {
            SaveSlotInfo info = new SaveSlotInfo();
            info.slotIndex = i;
            info.hasData = มีเซฟในช่อง(i);

            if (info.hasData)
            {
                SaveData data = โหลดเกม(i);
                if (data != null)
                {
                    info.playerName = data.playerName;
                    info.level = data.level;
                    info.lastPlayDate = data.lastPlayDate;
                    info.totalPlayTime = data.totalPlayTime;
                }
            }

            slotInfos.Add(info);
        }

        return slotInfos;
    }

    /// <summary>
    /// เซฟด่วน (Quick Save)
    /// Quick save
    /// </summary>
    public bool เซฟด่วน(SaveData saveData)
    {
        return เซฟเกม(0, saveData); // เซฟในช่อง 0 เสมอ
    }

    /// <summary>
    /// โหลดด่วน (Quick Load)
    /// Quick load
    /// </summary>
    public SaveData โหลดด่วน()
    {
        return โหลดเกม(0); // โหลดจากช่อง 0 เสมอ
    }

    /// <summary>
    /// เซฟอัตโนมัติ
    /// Auto save
    /// </summary>
    public void เซฟอัตโนมัติ(SaveData saveData)
    {
        เซฟเกม(maxSaveSlots, saveData); // เซฟในช่องพิเศษสำหรับเซฟอัตโนมัติ
    }
}

/// <summary>
/// ข้อมูลสำหรับแสดงใน UI
/// Information for UI display
/// </summary>
public class SaveSlotInfo
{
    public int slotIndex;
    public bool hasData;
    public string playerName = "ว่าง";
    public int level = 0;
    public System.DateTime lastPlayDate;
    public int totalPlayTime = 0;
}
```

### 3. Encryption Manager

```csharp
using UnityEngine;
using System.Security.Cryptography;
using System.Text;
using System;

public class EncryptionManager : MonoBehaviour
{
    [Header("การตั้งค่าเข้ารหัส - Encryption Settings")]
    public string encryptionKey = "YourSecretKey123"; // คีย์เข้ารหัส

    /// <summary>
    /// เข้ารหัสข้อความ
    /// Encrypt text
    /// </summary>
    public string เข้ารหัส(string plainText)
    {
        try
        {
            byte[] plainBytes = Encoding.UTF8.GetBytes(plainText);
            byte[] keyBytes = Encoding.UTF8.GetBytes(encryptionKey);

            // ใช้ AES encryption
            using (Aes aes = Aes.Create())
            {
                aes.Key = keyBytes;
                aes.IV = new byte[16]; // Initialization Vector

                using (var encryptor = aes.CreateEncryptor(aes.Key, aes.IV))
                {
                    byte[] encryptedBytes = encryptor.TransformFinalBlock(plainBytes, 0, plainBytes.Length);
                    return Convert.ToBase64String(encryptedBytes);
                }
            }
        }
        catch (Exception e)
        {
            Debug.LogError($"เกิดข้อผิดพลาดในการเข้ารหัส: {e.Message}");
            return plainText;
        }
    }

    /// <summary>
    /// ถอดรหัสข้อความ
    /// Decrypt text
    /// </summary>
    public string ถอดรหัส(string encryptedText)
    {
        try
        {
            byte[] encryptedBytes = Convert.FromBase64String(encryptedText);
            byte[] keyBytes = Encoding.UTF8.GetBytes(encryptionKey);

            using (Aes aes = Aes.Create())
            {
                aes.Key = keyBytes;
                aes.IV = new byte[16];

                using (var decryptor = aes.CreateDecryptor(aes.Key, aes.IV))
                {
                    byte[] decryptedBytes = decryptor.TransformFinalBlock(encryptedBytes, 0, encryptedBytes.Length);
                    return Encoding.UTF8.GetString(decryptedBytes);
                }
            }
        }
        catch (Exception e)
        {
            Debug.LogError($"เกิดข้อผิดพลาดในการถอดรหัส: {e.Message}");
            return encryptedText;
        }
    }

    /// <summary>
    /// สร้างคีย์เข้ารหัสแบบสุ่ม
    /// Generate random encryption key
    /// </summary>
    public string สร้างคีย์สุ่ม()
    {
        const string chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        var random = new System.Random();
        string key = new string(Enumerable.Repeat(chars, 16)
            .Select(s => s[random.Next(s.Length)]).ToArray());

        encryptionKey = key;
        return key;
    }

    /// <summary>
    /// ตรวจสอบความสมบูรณ์ของข้อมูล
    /// Check data integrity
    /// </summary>
    public bool ตรวจสอบความสมบูรณ์(string data)
    {
        if (string.IsNullOrEmpty(data))
            return false;

        try
        {
            // ลองถอดรหัสเพื่อตรวจสอบ
            ถอดรหัส(data);
            return true;
        }
        catch
        {
            return false;
        }
    }
}
```

### 4. Auto-Save System

```csharp
using UnityEngine;
using System.Collections;

public class AutoSave : MonoBehaviour
{
    [Header("การตั้งค่า - Settings")]
    public float autoSaveInterval = 300f;     // เซฟทุก 5 นาที
    public bool saveOnLevelComplete = true;   // เซฟเมื่อผ่านด่าน
    public bool saveOnApplicationQuit = true; // เซฟเมื่อออกเกม

    [Header("การอ้างอิง - References")]
    public SaveLoadManager saveLoadManager;   // ผู้จัดการเซฟ/โหลด

    private float lastSaveTime = 0f;          // เวลาเซฟล่าสุด

    void Start()
    {
        lastSaveTime = Time.time;

        if (saveOnApplicationQuit)
        {
            Application.quitting += OnApplicationQuit;
        }
    }

    void Update()
    {
        // เซฟอัตโนมัติตามเวลา
        if (Time.time - lastSaveTime >= autoSaveInterval)
        {
            เซฟอัตโนมัติ();
        }
    }

    /// <summary>
    /// เซฟอัตโนมัติ
    /// Auto save
    /// </summary>
    public void เซฟอัตโนมัติ()
    {
        if (saveLoadManager == null) return;

        // รวบรวมข้อมูลเกมปัจจุบัน
        SaveData currentData = รวบรวมข้อมูลเกมปัจจุบัน();

        // เซฟอัตโนมัติ
        saveLoadManager.เซฟอัตโนมัติ(currentData);

        lastSaveTime = Time.time;
        Debug.Log("เซฟอัตโนมัติสำเร็จ");
    }

    /// <summary>
    /// เซฟเมื่อผ่านด่าน
    /// Save on level complete
    /// </summary>
    public void เซฟเมื่อผ่านด่าน(string levelName)
    {
        if (!saveOnLevelComplete || saveLoadManager == null) return;

        SaveData currentData = รวบรวมข้อมูลเกมปัจจุบัน();

        // เพิ่มด่านที่ผ่านแล้ว
        if (!currentData.completedLevels.Contains(levelName))
        {
            currentData.completedLevels.Add(levelName);
        }

        // เซฟในช่องปกติ (ไม่ใช่เซฟอัตโนมัติ)
        saveLoadManager.เซฟเกม(0, currentData);

        Debug.Log($"เซฟเมื่อผ่านด่าน: {levelName}");
    }

    /// <summary>
    /// รวบรวมข้อมูลเกมปัจจุบัน
    /// Collect current game data
    /// </summary>
    private SaveData รวบรวมข้อมูลเกมปัจจุบัน()
    {
        SaveData data = SaveData.สร้างใหม่();

        // รวบรวมข้อมูลจาก GameManager หรือ Player
        GameManager gameManager = FindObjectOfType<GameManager>();
        if (gameManager != null)
        {
            data.playerName = gameManager.playerName;
            data.level = gameManager.playerLevel;
            data.experience = gameManager.playerExperience;
            data.completedLevels = gameManager.completedLevels;
            data.totalPlayTime = gameManager.totalPlayTime;
        }

        return data;
    }

    /// <summary>
    /// จัดการเมื่อออกจากเกม
    /// Handle application quit
    /// </summary>
    private void OnApplicationQuit()
    {
        เซฟอัตโนมัติ();
    }

    /// <summary>
    /// จัดการเมื่อเกมหยุดชั่วคราว
    /// Handle application pause
    /// </summary>
    private void OnApplicationPause(bool pause)
    {
        if (pause)
        {
            เซฟอัตโนมัติ();
        }
    }
}
```

## การใช้งานจริง (Practical Usage)

### การเซฟและโหลดในโค้ด (Code Usage)

```csharp
public class GameManager : MonoBehaviour
{
    private SaveLoadManager saveLoadManager;
    private SaveData currentSaveData;

    void Start()
    {
        saveLoadManager = FindObjectOfType<SaveLoadManager>();
        โหลดเกมเมื่อเริ่มต้น();
    }

    /// <summary>
    /// โหลดเกมเมื่อเริ่มต้นเกม
    /// Load game on game start
    /// </summary>
    private void โหลดเกมเมื่อเริ่มต้น()
    {
        currentSaveData = saveLoadManager.โหลดด่วน();

        if (currentSaveData != null)
        {
            // นำเข้าข้อมูลเซฟ
            นำเข้าข้อมูลเซฟ(currentSaveData);
        }
        else
        {
            // สร้างข้อมูลใหม่ถ้าไม่มีเซฟ
            currentSaveData = SaveData.สร้างใหม่();
        }
    }

    /// <summary>
    /// นำเข้าข้อมูลเซฟ
    /// Import save data
    /// </summary>
    private void นำเข้าข้อมูลเซฟ(SaveData saveData)
    {
        playerName = saveData.playerName;
        playerLevel = saveData.level;
        playerExperience = saveData.experience;
        completedLevels = saveData.completedLevels;
        // ... นำเข้าข้อมูลอื่นๆ
    }

    /// <summary>
    /// เซฟเกมปัจจุบัน
    /// Save current game
    /// </summary>
    public void เซฟเกมปัจจุบัน(int slotIndex)
    {
        // อัปเดตข้อมูลเซฟปัจจุบัน
        อัปเดตข้อมูลเซฟ();

        // เซฟเกม
        saveLoadManager.เซฟเกม(slotIndex, currentSaveData);
    }

    /// <summary>
    /// อัปเดตข้อมูลเซฟจากสถานะเกมปัจจุบัน
    /// Update save data from current game state
    /// </summary>
    private void อัปเดตข้อมูลเซฟ()
    {
        currentSaveData.playerName = playerName;
        currentSaveData.level = playerLevel;
        currentSaveData.experience = playerExperience;
        currentSaveData.completedLevels = completedLevels;
        currentSaveData.totalPlayTime += Mathf.RoundToInt(Time.time);
        currentSaveData.lastPlayDate = DateTime.Now;
    }
}
```

## เทคนิคขั้นสูง (Advanced Techniques)

### 1. Binary Serialization

```csharp
/// <summary>
/// เซฟด้วย Binary Serialization (สำหรับข้อมูลขนาดใหญ่)
/// Save with binary serialization (for large data)
/// </summary>
public bool เซฟแบบBinary(int slotIndex, object data)
{
    try
    {
        string fileName = $"save_{slotIndex}.bin";
        string filePath = Path.Combine(savePath, fileName);

        BinaryFormatter formatter = new BinaryFormatter();
        using (FileStream stream = new FileStream(filePath, FileMode.Create))
        {
            formatter.Serialize(stream, data);
        }

        return true;
    }
    catch (System.Exception e)
    {
        Debug.LogError($"เกิดข้อผิดพลาดในการเซฟแบบ Binary: {e.Message}");
        return false;
    }
}

/// <summary>
/// โหลดด้วย Binary Serialization
/// Load with binary serialization
/// </summary>
public object โหลดแบบBinary(int slotIndex)
{
    try
    {
        string fileName = $"save_{slotIndex}.bin";
        string filePath = Path.Combine(savePath, fileName);

        if (!File.Exists(filePath)) return null;

        BinaryFormatter formatter = new BinaryFormatter();
        using (FileStream stream = new FileStream(filePath, FileMode.Open))
        {
            return formatter.Deserialize(stream);
        }
    }
    catch (System.Exception e)
    {
        Debug.LogError($"เกิดข้อผิดพลาดในการโหลดแบบ Binary: {e.Message}");
        return null;
    }
}
```

### 2. Cloud Save (สำหรับเกมออนไลน์)

```csharp
public class CloudSaveManager : MonoBehaviour
{
    /// <summary>
    /// เซฟไปยังคลาวด์
    /// Save to cloud
    /// </summary>
    public void เซฟไปยังคลาวด์(SaveData saveData)
    {
        // ใช้ Unity Services หรือ PlayFab เพื่อเซฟออนไลน์
        // Implementation depends on cloud service used
    }

    /// <summary>
    /// โหลดจากคลาวด์
    /// Load from cloud
    /// </summary>
    public SaveData โหลดจากคลาวด์()
    {
        // โหลดข้อมูลจากคลาวด์
        return null;
    }
}
```

## การทดสอบระบบ (Testing the System)

### การควบคุมการทดสอบ (Test Controls)

- **ปุ่มเซฟ** ใน UI สำหรับทดสอบการเซฟ
- **ปุ่มโหลด** สำหรับทดสอบการโหลด
- **Console** สำหรับดูสถานะการเซฟ/โหลด

### สิ่งที่ควรสังเกต (What to Observe)

- **ข้อมูลถูกบันทึก** และโหลดได้ถูกต้อง
- **การเข้ารหัส** ทำงานอย่างปกติ
- **หลายช่องเซฟ** ทำงานได้ดี
- **เซฟอัตโนมัติ** ทำงานตามเวลา

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **ปลอดภัย** - มีการเข้ารหัสข้อมูล
- **ยืดหยุ่น** - รองรับหลายรูปแบบการเซฟ
- **มีประสิทธิภาพ** - JSON และ Binary แล้วแต่สถานการณ์
- **ขยายได้** - เพิ่มประเภทข้อมูลได้ง่าย

### ⚠️ ข้อควรระวัง
- **พื้นที่เก็บข้อมูล** - ขนาดไฟล์เซฟ
- **ความเข้ากันได้** - เวอร์ชันเกมที่ต่างกัน
- **ความปลอดภัย** - คีย์เข้ารหัสต้องเก็บเป็นความลับ

## แบบฝึกหัด (Exercise)

1. **สร้างระบบเซฟ/โหลด** สำหรับข้อมูลผู้เล่น
2. **เพิ่มการเข้ารหัส** ให้ข้อมูลเซฟปลอดภัย
3. **สร้างหลายช่องเซฟ** ให้ผู้เล่นเลือกได้
4. **เพิ่มเซฟอัตโนมัติ** ทุก 5 นาที

**คุณเข้าใจระบบเซฟ/โหลดนี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนเรื่อง Quest System กันต่อ! 🚀
