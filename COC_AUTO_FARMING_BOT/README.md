# CoC Auto Farming Bot - Flow Logic

**⚠️ DISCLAIMER**: Bot ini hanya untuk **pembelajaran automation logic**. Penggunaan pada akun real CoC **berisiko ban permanen**. Gunakan dengan akun test/burner saja.

---

## 📋 Daftar Isi
1. [Flow Logic Dasar](#flow-logic-dasar)
2. [Attack Strategy](#attack-strategy)
3. [Farming Logic](#farming-logic)
4. [Implementation](#implementation)
5. [Setup di Android](#setup-di-android)

---

## Flow Logic Dasar

```
┌─────────────────────────────────┐
│   START BOT                     │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  Check if in Multiplayer Map    │
│  (HOME BASE / VILLAGE)          │
└──��─────────┬────────────────────┘
             │
       ┌─────┴─────┐
       NO          YES
       │           │
       ▼           ▼
    WAIT      PROCEED
             │
             ▼
┌─────────────────────────────────┐
│  Open Attack Menu               │
│  (Tap "FIND OPPONENT" / "1v1")  │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  Wait for Matchmaking           │
│  (Timeout: 30s)                 │
└────────────┬────────────────────┘
             │
       ┌─────┴──────┐
       FAIL         SUCCESS
       │            │
       ▼            ▼
    RETRY       SCAN BASE
    (3x)            │
       │            ▼
       │    ┌─────────────────────────┐
       │    │ Detect Base Layout      │
       │    │ - Identify THall        │
       │    │ - Detect Resource       │
       │    │ - Map Defenses          │
       │    └────────────┬────────────┘
       │                 │
       │                 ▼
       │    ┌─────────────────────────┐
       │    │ Calculate Best Troop    │
       │    │ Strategy                │
       │    │ - Farming Troops:       │
       │    │   * Barbarian + Archer  │
       │    │   * Giant + Goblin      │
       │    │   * Dragon (if higher)  │
       │    └────────────┬────────────┘
       │                 │
       │                 ▼
       │    ┌─────────────────────────┐
       │    │ Deploy Troops Phase 1   │
       │    │ - Tank (Giant)          │
       │    │ - Melee (Barbarian)     │
       │    └────────────┬────────────┘
       │                 │
       │                 ▼
       │    ┌─────────────────────────┐
       │    │ Deploy Troops Phase 2   │
       │    │ - Ranged (Archer)       │
       │    │ - Goblin (Gold focused) │
       │    └────────────┬────────────┘
       │                 │
       │                 ▼
       │    ┌─────────────────────────┐
       │    │ Cast Spells (Optional)  │
       │    │ - Healing Spell         │
       │    │ - Rage Spell (if high)  │
       │    └────────────┬────────────┘
       │                 │
       │                 ▼
       │    ┌─────────────────────────┐
       │    │ Battle Auto-Play        │
       │    │ - Wait for completion   │
       │    │ - Check 80%+ damage     │
       │    │ (30-60 seconds)         │
       │    └────────────┬────────────┘
       │                 │
       │                 ▼
       │    ┌─────────────────────────┐
       │    │ Collect Loot            │
       │    │ - Gold                  │
       │    │ - Elixir                │
       │    │ - DE (if available)     │
       │    └────────────┬────────────┘
       │                 │
       └─────────────────┴──────────┐
                                    │
                                    ▼
                        ┌─────────────────────────┐
                        │ Check Cooldown          │
                        │ - War preparation: NO   │
                        │ - Raid cooldown OK?     │
                        │ - Army trained?         │
                        └────────────┬────────────┘
                                     │
                                ┌────┴────┐
                                YES      NO
                                │        │
                        ┌───────┘        └─────┐
                        │                      │
                        ▼                      ▼
                    REPEAT              WAIT TRAIN
                    (Loop)              (Timeout)
                        │                      │
                        └──────────┬───────────┘
                                   │
                                   ▼
                        ┌─────────────────────────┐
                        │ Check if Stop Condition │
                        │ - Max attacks/day?      │
                        │ - Time limit?           │
                        │ - Out of army?          │
                        └────────────┬────────────┘
                                     │
                                ┌────┴────┐
                                YES      NO
                                │        │
                                ▼        │
                            STOP &      │
                            LOG         │
                                        └─→ BACK TO "OPEN ATTACK MENU"

```

---

## Attack Strategy

### **Strategy 1: Goblin Farming (Elixir Focus)**
```
Troop Composition:
- 50% Goblins
- 30% Barbarians  
- 20% Archers

Spell:
- Healing Spell (2-3x) untuk tank

Target Priority:
1. Gold Storage (PRIMARY)
2. Elixir Storage (SECONDARY)
3. Open Resources
4. Townhall (LAST - untuk star)

Risk: LOW (goblins cepat)
Loot: MEDIUM-HIGH
Durasi: 2-3 menit per raid
```

### **Strategy 2: Giant + Goblin (Balanced)**
```
Troop Composition:
- 30% Giants (tank)
- 40% Goblins (damage)
- 30% Archers (support)

Spell:
- Healing Spell (3-4x)
- Rage Spell (1-2x) opsional

Target Priority:
1. Nearest Defense
2. Gold Storage
3. Elixir Storage
4. Resources

Risk: MEDIUM
Loot: HIGH
Durasi: 3-4 menit per raid
```

### **Strategy 3: Dragon (High TH)**
```
Troop Composition:
- 8-12 Dragons
- Atau mix: Dragons + Hog Riders

Spell:
- Healing Spell
- Rage Spell

Target Priority:
1. Destroy all defenses
2. Collect all resources
3. Townhall for full 3-star

Risk: MEDIUM-HIGH (mahal elixir)
Loot: VERY HIGH
Durasi: 4-5 menit per raid
```

---

## Farming Logic

### **Key Decision Points:**

#### **1. Should Attack?**
```
if (in_home_base && not_in_war && can_attack):
    return TRUE
else:
    return FALSE
```

#### **2. Which Strategy?**
```
if (TH_level <= 7):
    strategy = GOBLIN_FARMING
elif (TH_level <= 9):
    strategy = GIANT_GOBLIN
else:
    strategy = DRAGON
```

#### **3. Troop Deployment Timing:**
```
Phase 1 (0s - 5s):
  - Deploy ALL tank troops (Giant)
  - Let them walk toward defenses
  
Phase 2 (5s - 15s):
  - Deploy damage dealers (Goblins)
  - Focus on resource buildings
  
Phase 3 (15s - 25s):
  - Deploy support (Archers)
  - Clean up remaining buildings
  
Phase 4 (25s - 45s):
  - Cast healing spell if troops low HP
  - Cast rage spell on clustered troops
```

#### **4. Stop Conditions:**
```
STOP if:
  - All troops deployed & idle (battle end)
  - OR Townhall destroyed
  - OR 80%+ base destroyed
  - OR timer >= 3 minutes (auto-surrender)
  
NEXT ATTACK after:
  - Collection time: 30 seconds
  - Troop training check: loop until ready
  - Matchmaking cooldown: 10-15 seconds
```

---

## Implementation

### **Pseudocode (Python/Logic)**

```python
class CoCAutoFarmBot:
    
    def __init__(self):
        self.max_attacks_per_day = 30
        self.attack_count = 0
        self.farming_strategy = "GOBLIN_FARMING"
        self.stop_on_error = False
        
    def main_loop(self):
        """Main farming loop"""
        while self.attack_count < self.max_attacks_per_day:
            
            # Step 1: Check if can attack
            if not self.is_in_home_base():
                self.wait(5)
                continue
                
            if not self.is_army_ready():
                self.wait(30)  # Wait for training
                continue
            
            # Step 2: Open attack menu
            self.tap_find_opponent()
            self.wait(2)
            
            # Step 3: Matchmake
            if not self.wait_for_match(timeout=30):
                self.logger("Matchmaking failed, retry...")
                self.tap_back()
                continue
            
            # Step 4: Analyze base
            base_info = self.analyze_base()
            if not base_info:
                self.logger("Base analysis failed")
                self.tap_back()
                continue
            
            # Step 5: Execute attack
            success = self.execute_attack(base_info)
            
            # Step 6: Collect loot
            self.wait_for_battle_end()
            self.collect_loot()
            
            # Step 7: Log stats
            self.attack_count += 1
            self.logger(f"Attack {self.attack_count} completed")
            
            # Step 8: Cooldown
            self.wait(15)
    
    def is_in_home_base(self):
        """Check if CoC app is open and in home base"""
        # Screenshot & detect "Find Opponent" button
        # Return True if visible
        pass
    
    def is_army_ready(self):
        """Check if troops are fully trained"""
        # Screenshot & check training queue
        # Return True if army full
        pass
    
    def tap_find_opponent(self):
        """Tap the 'Find Opponent' button"""
        # Coordinate: Usually center-bottom of screen
        # tap(x=540, y=900)
        pass
    
    def wait_for_match(self, timeout=30):
        """Wait for opponent found"""
        # Loop for N seconds
        # Check if battle scene loaded
        # Return True if matched, False if timeout
        pass
    
    def analyze_base(self):
        """Detect base layout"""
        # Screenshot
        # Use image recognition to find:
        #   - Townhall position
        #   - Resource building positions
        #   - Defense positions
        # Return dict with coordinates
        pass
    
    def execute_attack(self, base_info):
        """Deploy troops according to strategy"""
        
        # Phase 1: Deploy tanks
        self.deploy_troops_phase1(base_info)
        self.wait(5)
        
        # Phase 2: Deploy damage
        self.deploy_troops_phase2(base_info)
        self.wait(10)
        
        # Phase 3: Deploy support
        self.deploy_troops_phase3(base_info)
        self.wait(10)
        
        # Phase 4: Cast spells if needed
        if self.check_troops_low_hp():
            self.cast_healing_spell()
            self.wait(5)
        
        return True
    
    def deploy_troops_phase1(self, base_info):
        """Deploy tank troops (Giant)"""
        # Find troop in army
        # Tap multiple times to deploy around perimeter
        # Start from defensive corner
        pass
    
    def deploy_troops_phase2(self, base_info):
        """Deploy damage dealers (Goblin)"""
        # Deploy toward resource buildings
        # Follow behind giants
        pass
    
    def deploy_troops_phase3(self, base_info):
        """Deploy support (Archer)"""
        # Deploy to cover all areas
        # Focus on backline damage
        pass
    
    def wait_for_battle_end(self):
        """Wait for battle to finish or timeout"""
        # Loop until:
        # - All troops idle/dead
        # - OR 3 minutes passed
        # - OR Return to results screen
        self.wait(60)  # Battle usually 1-3 min
    
    def collect_loot(self):
        """Tap 'next' to go to loot screen"""
        # Tap next button
        # Wait for loot animation
        self.tap(x=540, y=800)
        self.wait(3)
    
    def logger(self, msg):
        """Log message"""
        print(f"[{datetime.now()}] {msg}")
    
    def wait(self, seconds):
        """Wait N seconds"""
        time.sleep(seconds)
    
    def tap(self, x, y):
        """Tap screen at (x, y)"""
        # Use accessibility service or adb
        pass
    
    def check_troops_low_hp(self):
        """Check if main troops have low health"""
        # Rough check: if 30% of troops died
        return True
    
    def cast_healing_spell(self):
        """Cast healing spell on troops"""
        # Tap spell icon
        # Tap area with troops
        pass

# Run bot
if __name__ == "__main__":
    bot = CoCAutoFarmBot()
    bot.main_loop()
```

---

## Setup di Android

### **Method 1: Menggunakan Tasker (Recommended untuk Phone-Only)**

**Step 1: Install Tasker**
- Download dari Play Store
- Grant accessibility permissions

**Step 2: Create Task dalam Tasker**
```
Task: CoC_Farm

1. Loop (30 times - untuk 30 attacks)
   - Wait: 2 seconds
   - Tap Screen: 540, 900  [Find Opponent]
   - Wait: 30 seconds [matchmaking]
   - Tap Screen: 300, 300  [Deploy Giant 1]
   - Tap Screen: 500, 300  [Deploy Giant 2]
   - Tap Screen: 700, 300  [Deploy Giant 3]
   - Wait: 5 seconds
   - Tap Screen: 250, 600  [Deploy Goblin area]
   - Tap Screen: 550, 600  [Deploy Goblin area]
   - Wait: 60 seconds [battle play]
   - Tap Screen: 540, 800  [Next/Collect]
   - Wait: 15 seconds [cooldown]
   
2. End Loop
```

**Step 3: Create Profile untuk Auto-Start**
```
Profile: CoC_Farm_Schedule
- Time: 09:00 (morning)
- Action: Run Task "CoC_Farm"
```

---

### **Method 2: Menggunakan Macrodroid**

**Step 1: Install Macrodroid**
```
Play Store → Macrodroid
```

**Step 2: Create Macro**
```
Macro Name: CoC Auto Farm

Trigger:
  - App Opened: Clash of Clans

Actions (in sequence):
  1. Repeat (30 times):
     - Tap at: 540, 900 (Find Opponent)
     - Delay: 30s
     - Tap at: 300, 300
     - Tap at: 500, 300
     - Delay: 5s
     - Tap at: 250, 600
     - Delay: 60s
     - Tap at: 540, 800
     - Delay: 15s
```

---

### **Method 3: Menggunakan ADB Script (Advanced)**

**Setup:**
1. Enable Developer Mode di Android
2. Connect ke laptop via ADB (atau via Android Terminal)
3. Run script Python

**Script minimal:**
```bash
#!/bin/bash

# Farm 30 kali
for i in {1..30}; do
  echo "Attack $i"
  
  # Tap Find Opponent
  adb shell input tap 540 900
  sleep 30
  
  # Deploy troops
  adb shell input tap 300 300
  adb shell input tap 500 300
  sleep 5
  
  # Deploy damage
  adb shell input tap 250 600
  sleep 60
  
  # Next
  adb shell input tap 540 800
  sleep 15
done
```

---

## ⚠️ Anti-Detection Tips

1. **Add Random Delays**
   ```python
   delay = random.randint(5, 15)
   self.wait(delay)
   ```

2. **Vary Tap Positions**
   ```python
   x_offset = random.randint(-10, 10)
   y_offset = random.randint(-10, 10)
   self.tap(x + x_offset, y + y_offset)
   ```

3. **Human-like Behavior**
   - Jangan attack 24/7
   - Pakai schedule (pagi, siang, malam)
   - Jeda antar attack: 15-30 detik

4. **Don't Farm Continuously**
   - Max 50-60 attacks per hari
   - Spread throughout the day
   - Jangan exact same pattern

---

## 🚨 Risk Warning

| Risk Level | Action | Result |
|---|---|---|
| LOW | 1-2x attack/hari | Maybe safe (1-2 bulan) |
| MEDIUM | 10-20x attack/hari | Warning dalam 1-2 minggu |
| HIGH | 30+ attack/hari 24/7 | **BAN dalam 3-7 hari** |
| CRITICAL | Bot logic yang obvious | **BAN INSTANT** |

---

## 📊 Expected Farming Rate

| Strategy | Per Attack | Per 10 Attack | Per 100 Attack |
|---|---|---|---|
| Goblin | 150-200K elixir | 1.5-2M elixir | 15-20M elixir |
| G&G | 200-300K mixed | 2-3M mixed | 20-30M mixed |
| Dragon | 300-500K all | 3-5M all | 30-50M all |

---

## Next Steps

1. **Buat script Python full** dengan image recognition
2. **Integrate OpenCV** untuk base analysis
3. **Setup error handling** & retry logic
4. **Add logging/stats** dashboard
5. **Deploy ke Android** via accessibility service

Mau saya lanjutin dengan salah satu ini?
