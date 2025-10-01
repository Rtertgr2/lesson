# บทเรียนที่ 007: การจัดการฉาก (Scene Management)

## วัตถุประสงค์ (Objectives)
เรียนรู้การจัดการฉากใน Unity รวมถึงการโหลด เปลี่ยน และจัดการข้อมูลระหว่างฉาก

## ภาพรวมระบบ (System Overview)

การจัดการฉากใน Unity ประกอบด้วย:
- การโหลดและเปลี่ยนฉาก (Scene Loading & Transition)
- การจัดการข้อมูลระหว่างฉาก (Cross-Scene Data Management)
- การจัดการสถานะเกม (Game State Management)
- การแสดงหน้าจอโหลด (Loading Screen)
- การจัดการข้อผิดพลาด (Error Handling)

## โครงสร้างระบบ (System Structure)

```
Scene Management/
├── SceneManager.cs             # ผู้จัดการฉากหลัก
├── SceneTransition.cs          # จัดการการเปลี่ยนฉาก
├── LoadingScreen.cs            # หน้าจอโหลด
├── GameStateManager.cs         # จัดการสถานะเกม
└── SaveLoadManager.cs          # จัดการเซฟ/โหลด
```

## การติดตั้งใช้งาน (Implementation)

### 1. Scene Manager หลัก

```csharp
using UnityEngine;
using UnityEngine.SceneManagement;
using System.Collections;
using System.Collections.Generic;

public class SceneManager : MonoBehaviour
{
    [Header("การตั้งค่า - Settings")]
    public float sceneTransitionDelay = 1f;     // ดีเลย์การเปลี่ยนฉาก
    public string loadingSceneName = "Loading"; // ชื่อฉากโหลด

    [Header("สถานะ - Status")]
    private static SceneManager instance;       // Instance เดียว
    public string currentSceneName;             // ชื่อฉากปัจจุบัน
    private Dictionary<string, object> sceneData = new Dictionary<string, object>(); // ข้อมูลฉาก

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
        currentSceneName = UnityEngine.SceneManagement.SceneManager.GetActiveScene().name;
    }

    /// <summary>
    /// โหลดฉากแบบอะซิงโครนัส
    /// Load scene asynchronously
    /// </summary>
    public void โหลดฉาก(string sceneName)
    {
        StartCoroutine(โหลดฉากแบบอะซิงโครนัส(sceneName));
    }

    /// <summary>
    /// โหลดฉากแบบอะซิงโครนัสด้วยหน้าจอโหลด
    /// Load scene with loading screen
    /// </summary>
    public void โหลดฉากด้วยหน้าจอโหลด(string sceneName)
    {
        StartCoroutine(โหลดฉากด้วยหน้าจอโหลดCoroutine(sceneName));
    }

    /// <summary>
    /// โหลดฉากแบบอะซิงโครนัส
    /// Load scene asynchronously
    /// </summary>
    private IEnumerator โหลดฉากแบบอะซิงโครนัส(string sceneName)
    {
        // แสดงหน้าจอโหลด (ถ้ามี)
        แสดงหน้าจอโหลด();

        // รอดีเลย์
        yield return new WaitForSeconds(sceneTransitionDelay);

        // โหลดฉากแบบอะซิงโครนัส
        AsyncOperation asyncLoad = UnityEngine.SceneManagement.SceneManager.LoadSceneAsync(sceneName);

        // รอให้โหลดเสร็จ
        while (!asyncLoad.isDone)
        {
            float progress = Mathf.Clamp01(asyncLoad.progress / 0.9f);
            อัปเดตแถบโหลด(progress);
            yield return null;
        }

        // อัปเดตชื่อฉากปัจจุบัน
        currentSceneName = sceneName;

        // ซ่อนหน้าจอโหลด
        ซ่อนหน้าจอโหลด();

        Debug.Log($"โหลดฉากเสร็จสิ้น: {sceneName}");
    }

    /// <summary>
    /// โหลดฉากด้วยหน้าจอโหลด
    /// Load scene with loading screen
    /// </summary>
    private IEnumerator โหลดฉากด้วยหน้าจอโหลดCoroutine(string sceneName)
    {
        // โหลดฉากโหลดก่อน
        AsyncOperation loadingSceneLoad = UnityEngine.SceneManagement.SceneManager.LoadSceneAsync(loadingSceneName);

        while (!loadingSceneLoad.isDone)
        {
            yield return null;
        }

        // จากนั้นโหลดฉากเป้าหมาย
        yield return โหลดฉากแบบอะซิงโครนัส(sceneName);
    }

    /// <summary>
    /// โหลดฉากถัดไปใน Build Settings
    /// Load next scene in build settings
    /// </summary>
    public void โหลดฉากถัดไป()
    {
        int currentIndex = UnityEngine.SceneManagement.SceneManager.GetActiveScene().buildIndex;
        int nextIndex = (currentIndex + 1) % UnityEngine.SceneManagement.SceneManager.sceneCountInBuildSettings;

        โหลดฉากด้วยชื่อจากดัชนี(nextIndex);
    }

    /// <summary>
    /// โหลดฉากก่อนหน้า
    /// Load previous scene
    /// </summary>
    public void โหลดฉากก่อนหน้า()
    {
        int currentIndex = UnityEngine.SceneManagement.SceneManager.GetActiveScene().buildIndex;
        int previousIndex = currentIndex > 0 ? currentIndex - 1 : UnityEngine.SceneManagement.SceneManager.sceneCountInBuildSettings - 1;

        โหลดฉากด้วยชื่อจากดัชนี(previousIndex);
    }

    /// <summary>
    /// โหลดฉากด้วยดัชนีใน Build Settings
    /// Load scene by build index
    /// </summary>
    private void โหลดฉากด้วยชื่อจากดัชนี(int buildIndex)
    {
        string sceneName = System.IO.Path.GetFileNameWithoutExtension(
            UnityEngine.SceneManagement.SceneUtility.GetScenePathByBuildIndex(buildIndex)
        );

        โหลดฉาก(sceneName);
    }

    /// <summary>
    /// รีโหลดฉากปัจจุบัน
    /// Reload current scene
    /// </summary>
    public void รีโหลดฉากปัจจุบัน()
    {
        โหลดฉาก(currentSceneName);
    }

    /// <summary>
    /// กลับไปยังเมนูหลัก
    /// Return to main menu
    /// </summary>
    public void กลับไปยังเมนูหลัก()
    {
        โหลดฉาก("MainMenu");
    }

    /// <summary>
    /// ออกจากเกม
    /// Quit game
    /// </summary>
    public void ออกจากเกม()
    {
        #if UNITY_EDITOR
            UnityEditor.EditorApplication.isPlaying = false;
        #else
            Application.Quit();
        #endif
    }

    /// <summary>
    /// แสดงหน้าจอโหลด
    /// Show loading screen
    /// </summary>
    private void แสดงหน้าจอโหลด()
    {
        // ค้นหาและแสดง LoadingScreen
        LoadingScreen loadingScreen = FindObjectOfType<LoadingScreen>();
        if (loadingScreen != null)
        {
            loadingScreen.Show();
        }
    }

    /// <summary>
    /// ซ่อนหน้าจอโหลด
    /// Hide loading screen
    /// </summary>
    private void ซ่อนหน้าจอโหลด()
    {
        LoadingScreen loadingScreen = FindObjectOfType<LoadingScreen>();
        if (loadingScreen != null)
        {
            loadingScreen.Hide();
        }
    }

    /// <summary>
    /// อัปเดตแถบโหลด
    /// Update loading bar
    /// </summary>
    private void อัปเดตแถบโหลด(float progress)
    {
        LoadingScreen loadingScreen = FindObjectOfType<LoadingScreen>();
        if (loadingScreen != null)
        {
            loadingScreen.UpdateProgress(progress);
        }
    }

    /// <summary>
    /// เก็บข้อมูลสำหรับฉากถัดไป
    /// Store data for next scene
    /// </summary>
    public void เก็บข้อมูลฉาก(string key, object value)
    {
        sceneData[key] = value;
    }

    /// <summary>
    /// ดึงข้อมูลจากฉากก่อนหน้า
    /// Retrieve data from previous scene
    /// </summary>
    public T ดึงข้อมูลฉาก<T>(string key)
    {
        if (sceneData.ContainsKey(key))
        {
            return (T)sceneData[key];
        }
        return default(T);
    }

    /// <summary>
    /// ล้างข้อมูลฉากทั้งหมด
    /// Clear all scene data
    /// </summary>
    public void ล้างข้อมูลฉาก()
    {
        sceneData.Clear();
    }
}
```

### 2. Loading Screen System

```csharp
using UnityEngine;
using UnityEngine.UI;
using TMPro;

public class LoadingScreen : MonoBehaviour
{
    [Header("การอ้างอิง UI - UI References")]
    public GameObject loadingPanel;           // แผงหน้าจอโหลด
    public Slider loadingBar;                 // แถบโหลด
    public TextMeshProUGUI loadingText;       // ข้อความโหลด
    public TextMeshProUGUI progressText;      // ข้อความเปอร์เซ็นต์

    [Header("เอฟเฟค - Effects")]
    public Animator loadingAnimator;          // อนิเมเตอร์
    public string showTrigger = "Show";       // ทริกเกอร์แสดง
    public string hideTrigger = "Hide";       // ทริกเกอร์ซ่อน

    void Start()
    {
        if (loadingPanel != null)
        {
            loadingPanel.SetActive(false);
        }
    }

    /// <summary>
    /// แสดงหน้าจอโหลด
    /// Show loading screen
    /// </summary>
    public void Show()
    {
        if (loadingPanel != null)
        {
            loadingPanel.SetActive(true);
        }

        if (loadingAnimator != null)
        {
            loadingAnimator.SetTrigger(showTrigger);
        }

        // รีเซ็ตแถบโหลด
        if (loadingBar != null)
        {
            loadingBar.value = 0f;
        }

        if (loadingText != null)
        {
            loadingText.text = "กำลังโหลด...";
        }
    }

    /// <summary>
    /// ซ่อนหน้าจอโหลด
    /// Hide loading screen
    /// </summary>
    public void Hide()
    {
        if (loadingAnimator != null)
        {
            loadingAnimator.SetTrigger(hideTrigger);
        }
        else
        {
            // ซ่อนทันทีถ้าไม่มีอนิเมเตอร์
            if (loadingPanel != null)
            {
                loadingPanel.SetActive(false);
            }
        }
    }

    /// <summary>
    /// อัปเดตความคืบหน้า
    /// Update progress
    /// </summary>
    public void UpdateProgress(float progress)
    {
        if (loadingBar != null)
        {
            loadingBar.value = progress;
        }

        if (progressText != null)
        {
            progressText.text = $"{Mathf.RoundToInt(progress * 100)}%";
        }

        if (loadingText != null)
        {
            if (progress < 0.3f)
                loadingText.text = "กำลังเตรียมข้อมูล...";
            else if (progress < 0.7f)
                loadingText.text = "กำลังโหลดฉาก...";
            else
                loadingText.text = "เกือบเสร็จแล้ว...";
        }
    }

    /// <summary>
    /// ตั้งค่าข้อความโหลด
    /// Set loading text
    /// </summary>
    public void ตั้งค่าข้อความโหลด(string text)
    {
        if (loadingText != null)
        {
            loadingText.text = text;
        }
    }
}
```

### 3. Scene Transition Manager

```csharp
using UnityEngine;
using System.Collections;

public class SceneTransition : MonoBehaviour
{
    [Header("การอ้างอิง - References")]
    public Animator transitionAnimator;       // อนิเมเตอร์การเปลี่ยนฉาก
    public SceneManager sceneManager;         // ผู้จัดการฉาก

    [Header("การตั้งค่า - Settings")]
    public float transitionDuration = 1f;     // ระยะเวลาการเปลี่ยน

    /// <summary>
    /// เปลี่ยนฉากด้วยอนิเมชัน
    /// Transition to scene with animation
    /// </summary>
    public void เปลี่ยนฉาก(string sceneName)
    {
        StartCoroutine(เปลี่ยนฉากCoroutine(sceneName));
    }

    /// <summary>
    /// Coroutine สำหรับการเปลี่ยนฉาก
    /// Coroutine for scene transition
    /// </summary>
    private IEnumerator เปลี่ยนฉากCoroutine(string sceneName)
    {
        // เล่นอนิเมชันออก
        if (transitionAnimator != null)
        {
            transitionAnimator.SetTrigger("Start");
            yield return new WaitForSeconds(transitionDuration / 2);
        }

        // โหลดฉาก
        if (sceneManager != null)
        {
            sceneManager.โหลดฉาก(sceneName);
        }

        // รอให้โหลดเสร็จ
        while (UnityEngine.SceneManagement.SceneManager.GetActiveScene().name != sceneName)
        {
            yield return null;
        }

        // เล่นอนิเมชันเข้า
        if (transitionAnimator != null)
        {
            transitionAnimator.SetTrigger("End");
        }
    }
}
```

### 4. Game State Manager

```csharp
using UnityEngine;
using System.Collections.Generic;

public class GameStateManager : MonoBehaviour
{
    [Header("สถานะเกม - Game State")]
    public GameState currentState;            // สถานะปัจจุบัน
    private static GameStateManager instance; // Instance เดียว

    [Header("ข้อมูลเกม - Game Data")]
    private Dictionary<string, object> gameData = new Dictionary<string, object>();

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
            DontDestroyOnLoad(gameObject);
            โหลดสถานะเกม();
        }
        else
        {
            Destroy(gameObject);
        }
    }

    /// <summary>
    /// เปลี่ยนสถานะเกม
    /// Change game state
    /// </summary>
    public void เปลี่ยนสถานะ(GameState newState)
    {
        if (currentState == newState) return;

        GameState previousState = currentState;
        currentState = newState;

        Debug.Log($"เปลี่ยนสถานะเกมจาก {previousState} เป็น {currentState}");

        // แจ้งผู้สังเกตการณ์
        แจ้งการเปลี่ยนสถานะ(previousState, currentState);
    }

    /// <summary>
    /// เก็บข้อมูลเกม
    /// Store game data
    /// </summary>
    public void เก็บข้อมูล(string key, object value)
    {
        gameData[key] = value;
    }

    /// <summary>
    /// ดึงข้อมูลเกม
    /// Retrieve game data
    /// </summary>
    public T ดึงข้อมูล<T>(string key)
    {
        if (gameData.ContainsKey(key))
        {
            return (T)gameData[key];
        }
        return default(T);
    }

    /// <summary>
    /// ล้างข้อมูลทั้งหมด
    /// Clear all data
    /// </summary>
    public void ล้างข้อมูลทั้งหมด()
    {
        gameData.Clear();
    }

    /// <summary>
    /// แจ้งการเปลี่ยนสถานะให้ผู้สังเกตการณ์
    /// Notify observers of state change
    /// </summary>
    private void แจ้งการเปลี่ยนสถานะ(GameState oldState, GameState newState)
    {
        // ส่ง Event หรือเรียกใช้ Delegate ที่ลงทะเบียนไว้
        OnGameStateChanged?.Invoke(oldState, newState);
    }

    /// <summary>
    /// โหลดสถานะเกมจากเซฟ
    /// Load game state from save
    /// </summary>
    private void โหลดสถานะเกม()
    {
        // โหลดจาก SaveLoadManager หรือ PlayerPrefs
        currentState = GameState.MainMenu;
    }

    /// <summary>
    /// บันทึกสถานะเกม
    /// Save game state
    /// </summary>
    public void บันทึกสถานะเกม()
    {
        // บันทึกไปยัง SaveLoadManager หรือ PlayerPrefs
    }

    // Event สำหรับการเปลี่ยนสถานะ
    public delegate void GameStateChanged(GameState oldState, GameState newState);
    public static event GameStateChanged OnGameStateChanged;
}

/// <summary>
/// สถานะเกมทั้งหมด
/// All game states
/// </summary>
public enum GameState
{
    MainMenu,      // เมนูหลัก
    Playing,       // กำลังเล่น
    Paused,        // หยุดชั่วคราว
    GameOver,      // เกมโอเวอร์
    Victory,       // ชนะ
    Loading,       // กำลังโหลด
    Cutscene       // คัทซีน
}
```

## การใช้งานจริง (Practical Usage)

### การตั้งค่าใน Unity Editor

1. **สร้าง Scene Manager** ในทุกฉากหรือใช้ DontDestroyOnLoad
2. **สร้าง Loading Scene** สำหรับหน้าจอโหลด
3. **ตั้งค่า Build Settings** ให้รวมทุกฉากที่ใช้
4. **สร้าง Transition Canvas** สำหรับอนิเมชันการเปลี่ยนฉาก

### การเรียกใช้ในโค้ด (Code Usage)

```csharp
public class GameController : MonoBehaviour
{
    private SceneManager sceneManager;

    void Start()
    {
        sceneManager = FindObjectOfType<SceneManager>();
    }

    /// <summary>
    /// เริ่มเกมใหม่
    /// Start new game
    /// </summary>
    public void เริ่มเกมใหม่()
    {
        sceneManager.เก็บข้อมูล("NewGame", true);
        sceneManager.โหลดฉาก("GameScene");
    }

    /// <summary>
    /// โหลดด่านถัดไป
    /// Load next level
    /// </summary>
    public void โหลดด่านถัดไป()
    {
        sceneManager.โหลดฉากถัดไป();
    }

    /// <summary>
    /// กลับไปยังเมนูหลัก
    /// Return to main menu
    /// </summary>
    public void กลับไปยังเมนูหลัก()
    {
        sceneManager.กลับไปยังเมนูหลัก();
    }
}
```

## เทคนิคขั้นสูง (Advanced Techniques)

### 1. Additive Scene Loading

```csharp
/// <summary>
/// โหลดฉากแบบ Addictive (ไม่แทนที่ฉากเดิม)
/// Load scene additively
/// </summary>
public void โหลดฉากแบบAdditive(string sceneName)
{
    StartCoroutine(โหลดฉากแบบAdditiveCoroutine(sceneName));
}

private IEnumerator โหลดฉากแบบAdditiveCoroutine(string sceneName)
{
    AsyncOperation asyncLoad = UnityEngine.SceneManagement.SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);

    while (!asyncLoad.isDone)
    {
        yield return null;
    }

    Debug.Log($"โหลดฉากแบบ Additive เสร็จสิ้น: {sceneName}");
}
```

### 2. Scene Unloading

```csharp
/// <summary>
/// อันโหลดฉาก
/// Unload scene
/// </summary>
public void อันโหลดฉาก(string sceneName)
{
    StartCoroutine(อันโหลดฉากCoroutine(sceneName));
}

private IEnumerator อันโหลดฉากCoroutine(string sceneName)
{
    AsyncOperation asyncUnload = UnityEngine.SceneManagement.SceneManager.UnloadSceneAsync(sceneName);

    while (!asyncUnload.isDone)
    {
        yield return null;
    }

    Debug.Log($"อันโหลดฉากเสร็จสิ้น: {sceneName}");
}
```

## การทดสอบระบบ (Testing the System)

### การควบคุมการทดสอบ (Test Controls)

- **ปุ่มทดสอบ** ใน UI สำหรับเปลี่ยนฉาก
- **Console** สำหรับดูสถานะการโหลด
- **Profiler** สำหรับวัดประสิทธิภาพ

### สิ่งที่ควรสังเกต (What to Observe)

- **ความลื่นไหล** ของการเปลี่ยนฉาก
- **หน้าจอโหลด** ที่ทำงานถูกต้อง
- **ข้อมูลที่คงอยู่** ระหว่างฉาก
- **ไม่มี Memory Leak** เมื่อเปลี่ยนฉาก

## ข้อควรพิจารณา (Considerations)

### ✅ จุดแข็งของระบบนี้
- **จัดการได้ง่าย** - โครงสร้างชัดเจน
- **ยืดหยุ่น** - รองรับหลายสถานการณ์
- **มีประสิทธิภาพ** - โหลดแบบอะซิงโครนัส
- **ขยายได้** - เพิ่มฟีเจอร์ใหม่ได้ง่าย

### ⚠️ ข้อควรระวัง
- **การจัดการ Memory** - อันโหลดฉากที่ไม่ใช้
- **การอ้างอิง Object** - ตรวจสอบอายุการใช้งาน
- **ประสิทธิภาพ** - โหลดฉากขนาดใหญ่

## แบบฝึกหัด (Exercise)

1. **สร้างระบบเปลี่ยนฉาก** ระหว่างเมนูกับเกม
2. **เพิ่มหน้าจอโหลด** ที่แสดงความคืบหน้า
3. **สร้างระบบเซฟด่าน** ที่ผู้เล่นกลับมาเล่นต่อได้
4. **เพิ่มอนิเมชันการเปลี่ยนฉาก** ที่สวยงาม

**คุณเข้าใจการจัดการฉากนี้ในระดับไหน (1-3)?**

เมื่อพร้อม เราจะไปเรียนเรื่อง Save/Load System กันต่อ! 🚀
