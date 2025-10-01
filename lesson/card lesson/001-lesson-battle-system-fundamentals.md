using UnityEngine;
using System.Collections.Generic;

/// <summary>
/// บทเรียนที่ 001: พื้นฐานระบบต่อสู้แบบผลัดกันเล่น
/// บทนำเกี่ยวกับสถาปัตยกรรมระบบต่อสู้ขั้นพื้นฐานและแนวคิดระบบการ์ด
/// LESSON 001: Turn-Based Battle System Fundamentals
/// Introduction to basic battle system architecture and card system concepts
/// </summary>
public class Lesson001_BattleSystemFundamentals : MonoBehaviour
{
    // ========================================
    // สถานะของระบบต่อสู้ - BATTLE SYSTEM STATES
    // ========================================
    public enum BattleState
    {
        กำลังเตรียม,      // กำลังตั้งค่าระบบต่อสู้ - Preparing, Setting up the battle
        เทิร์นผู้เล่น,     // เทิร์นของผู้เล่นที่จะทำการกระทำ - Player's turn to act
        เทิร์นศัตรู,       // เทิร์นของศัตรูที่จะทำการกระทำ - Enemy's turn to act
        จบการต่อสู้,      // การต่อสู้จบแล้ว (ชนะ/แพ้) - Battle finished (win/lose)
        ว่าง          // ไม่ได้อยู่ในระบบต่อสู้ - Not in battle
    }

    // ========================================
    // คลาสพื้นฐานของระบบการ์ด - CARD SYSTEM BASE CLASSES
    // ========================================

    /// <summary>
    /// คลาสพื้นฐานสำหรับการ์ดทุกใบในระบบ
    /// Base class for all cards in the system
    /// </summary>
    public abstract class Card
    {
        public string ชื่อการ์ด; // cardName
        public string คำอธิบาย; // description
        public int ค่าใช้มานา; // manaCost
        public ประเภทการ์ด ประเภทการ์ด; // cardType

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

    /// <summary>
    /// ประเภทของการ์ดในระบบของคุณ - Types of cards in your system
    /// </summary>
    public enum ประเภทการ์ด
    {
        ตัวเลข,         // 1.1 การ์ดตัวเลข - 1.1 Number Cards
        สลับสมการ,      // 1.2 การ์ดสลับสมการ - 1.2 Equation Switch Cards
        ตัวดำเนินการ,    // 1.3 การ์ดตัวดำเนินการ - 1.3 Operator Cards
        ช่วยเหลือ,      // 1.4 การ์ดช่วยเหลือ - 1.4 Help Cards
        กับดัก         // 1.5 การ์ดกับดัก - 1.5 Trap Cards
    }

    // ========================================
    // หน่วยต่อสู้ (ผู้เล่น/ศัตรู) - BATTLE UNIT (PLAYER/ENEMY)
    // ========================================

    /// <summary>
    /// แทนหน่วยในระบบต่อสู้ (ผู้เล่นหรือศัตรู)
    /// Represents a unit in battle (player or enemy)
    /// </summary>
    public class BattleUnit
    {
        public string ชื่อหน่วย; // unitName
        public int พลังชีวิตปัจจุบัน; // currentHealth
        public int พลังชีวิตสูงสุด; // maxHealth
        public int มานาปัจจุบัน; // currentMana
        public int มานาสูงสุด; // maxMana
        public bool เป็นผู้เล่น; // isPlayer - true สำหรับผู้เล่น, false สำหรับศัตรู

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

            Debug.Log($"{ชื่อหน่วย} รับความเสียหาย {ความเสียหาย} แต้ม! พลังชีวิต: {พลังชีวิตปัจจุบัน}/{พลังชีวิตสูงสุด}");
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

    // ========================================
    // ผู้จัดการระบบต่อสู้ - BATTLE MANAGER
    // ========================================

    /// <summary>
    /// จัดการสถานะโดยรวมและการไหลของระบบต่อสู้
    /// Manages the overall battle state and flow
    /// </summary>
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

    // ========================================
    // การติดตั้งใช้งานการ์ดตัวอย่าง - EXAMPLE CARD IMPLEMENTATIONS
    // ========================================

    /// <summary>
    /// การติดตั้งใช้งานการ์ดตัวเลขตัวอย่าง
    /// Example Number Card implementation
    /// </summary>
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

            Debug.Log($"{ผู้ใช้.ชื่อหน่วย} เล่น {ชื่อการ์ด} สร้างความเสียหาย {ความเสียหาย} แต้ม! - {ผู้ใช้.ชื่อหน่วย} plays {ชื่อการ์ด} dealing {ความเสียหาย} damage!");
        }
    }

    // ========================================
    // ตัวแปรบทเรียนสำหรับการทดสอบ - LESSON VARIABLES FOR TESTING
    // ========================================

    private BattleManager ผู้จัดการต่อสู้; // battleManager

    void Start()
    {
        Debug.Log("=== บทเรียนที่ 001: พื้นฐานระบบต่อสู้แบบผลัดกันเล่น ===");
        Debug.Log("=== LESSON 001: Battle System Fundamentals ===");
        Debug.Log("บทเรียนนี้สาธิต: - This lesson demonstrates:");
        Debug.Log("1. สถานะและการไหลของระบบต่อสู้ - 1. Battle states and flow");
        Debug.Log("2. คลาสพื้นฐานของระบบการ์ด - 2. Card system base classes");
        Debug.Log("3. การจัดการหน่วยต่อสู้ - 3. Battle unit management");
        Debug.Log("4. ผู้จัดการระบบต่อสู้ขั้นพื้นฐาน - 4. Basic battle manager");
        Debug.Log("");

        // กำหนดค่าเริ่มต้นและเริ่มการต่อสู้สาธิต - Initialize and start a demo battle
        ผู้จัดการต่อสู้ = new BattleManager();
        ผู้จัดการต่อสู้.เริ่มการต่อสู้();

        // สาธิต: เล่นการ์ด - Demo: Play a card
        Debug.Log("สาธิต: กำลังเล่นการ์ดใบแรก... - Demo: Playing first card...");
        ผู้จัดการต่อสู้.เล่นการ์ด(0);
    }

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
            if (ผู้จัดการต่อสู่ != null && ผู้จัดการต่อสู้.มือของผู้เล่น.Count > 0)
            {
                ผู้จัดการต่อสู้.เล่นการ์ด(0);
            }
        }
    }
}
