
<div dir="rtl">
<a href="https://github.com/shealtiely/ESP32_IOT_OS?tab=readme-ov-file#%D7%97%D7%99%D7%91%D7%95%D7%A8-esp-cam-%D7%9Crealtime-database">חיבור ESP-cam</a>

  
  # ESP32 IoT OS

**גשר IoT לחיבור ALTERA FPGA לענן Firebase**

---

## סקירה כללית

פרויקט זה מספק פתרון שלם לחיבור מערכת FPGA מבוססת ALTERA לענן Firebase באמצעות מיקרובקר ESP32. המערכת מאפשרת העברת נתונים דו-כיוונית בזמן אמת בין חיישנים/מפעילים המחוברים ל-FPGA לבין אפליקציית אינטרנט או מובייל.

```
ALTERA FPGA  <--Serial-->  ESP32  <--HTTPS-->  Firebase  <--API-->  App
```

---

## תכונות עיקריות

| תכונה | תיאור |
|--------|--------|
| **ניהול WiFi חכם** | תמיכה בעד 3 רשתות WiFi עם מעבר אוטומטי |
| **מצב Access Point** | יצירת נקודת גישה כשאין רשת זמינה |
| **עיבוד Dual-Core** |Core 0  ל-Core 1,  Serial2  ל-Firebase |
| **תקשורת דו-כיוונית** | ALTERA→Firebase ו-Firebase→ALTERA |
| **שמירת הגדרות** | כל ההגדרות נשמרות ב-Flash |

---



### דרישות חומרה


<div dir="rtl">
<ul>
  <li dir="rtl">ESP32 Dev Module (כל גרסה עם Dual-Core)</li>
  <li dir="rtl">תצוגת OLED SH1106 128x64 (אופציונלי)</li>
  <li dir="rtl">ALTERA FPGA עם חיבור Serial / UART</li>
</ul>
</div>



### דרישות תוכנה

<div dir="rtl">
<ul>
  <li dir="rtl">Arduino IDE 2.0+</li>
  <li dir="rtl">חשבון Firebase</li>
</ul>
</div>


---
## התקנת ספריות

### ספריות שיש להתקין ידנית (Library Manager)

ב-Arduino IDE לך ל: **Sketch → Include Library → Manage Libraries**

| ספרייה | מפתח | גרסה | תיאור |
|--------|-------|-------|--------|
| **ArduinoJson** | Benoit Blanchon | 6.x ומעלה | עיבוד JSON לתקשורת עם Firebase |
| **Adafruit SH110X** | Adafruit | 2.x ומעלה | תמיכה בתצוגת OLED |
| **Adafruit GFX Library** | Adafruit | 1.x ומעלה | ספריית גרפיקה (נדרשת עבור SH110X) |

### התקנה שלב אחר שלב:

1. פתח Arduino IDE
2. לך ל-**Sketch → Include Library → Manage Libraries**
3. חפש **"ArduinoJson"** → התקן את הגרסה האחרונה (6.x)
4. חפש **"Adafruit SH110X"** → התקן
5. כשתישאל להתקין dependencies - לחץ **"Install All"**

### ספריות מובנות (לא צריך להתקין)

הספריות הבאות מגיעות עם חבילת ESP32 ואין צורך להתקין אותן:

| ספרייה | תיאור |
|--------|--------|
| `WiFi.h` | חיבור לרשתות WiFi |
| `WebServer.h` | שרת HTTP לממשק הגדרות |
| `Preferences.h` | שמירת הגדרות ב-Flash |
| `HTTPClient.h` | שליחת בקשות HTTP ל-Firebase |
| `WiFiClientSecure.h` | תקשורת HTTPS מאובטחת |
| `Wire.h` | תקשורת I2C לתצוגת OLED |

---

## התקנת ESP32 ב-Arduino IDE
&#x202b;1. **הוסף ESP32 ל-Arduino IDE:**</br>
    
   הוסף ל - Additional Board URLs :Preferences ← File   
  ```
                                   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
  ```

&#x202b;2. **התקן את הבורד:**
   &#x202b;- Tools → Board → Boards Manager
   - חפש "ESP32" והתקן

&#x202b;3. **העלה את הקוד:**
   - פתח את `ESP32_IoT_Bridge.ino`
   - בחר בורד: ESP32 Dev Module
   - לחץ Upload

&#x202b;4. **הגדרות ראשוניות:**
   - התחבר לרשת WiFi: `ESP32_IOT_OS`
   - גש לכתובת: `192.168.4.1`
   - הזן פרטי WiFi ו-Firebase (עד 3 רשתות שונות)

---

## חיבורי חומרה

| פין ESP32 | חיבור | תיאור |
|-----------|--------|--------|
| GPIO 16 (RX2) | TX של ALTERA | קבלת נתונים |
| GPIO 17 (TX2) | RX של ALTERA | שליחת נתונים |
| GND | GND | אדמה משותפת |

> **שים לב:** השתמש ב-Level Shifter אם ה-FPGA עובד ב-5V

---

## פורמט נתונים

### ALTERA → Firebase
```json
{
  "fromAltera": {
    "A": 25,
    "B": 100,
    "C": 42
  }
}
```

### Firebase → ALTERA
```json
{
  "toAltera": 1
}
```

---

## הגדרת Firebase

1. צור פרויקט ב-[Firebase Console](https://console.firebase.google.com)
2. צור Realtime Database
3. הגדר Rules (לפיתוח):
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```
4. העתק את ה-Database URL והזן בממשק ההגדרות

---

## ארכיטקטורה

```
ESP32 Dual-Core Architecture
============================

Core 0 (Protocol CPU):
├── Task: serialTask
└── אחריות: קריאה מ-ALTERA

Core 1 (Application CPU):
├── Task: firebaseTask
└── אחריות: תקשורת עם Firebase
```

---

## פתרון בעיות

| בעיה | פתרון |
|------|--------|
| לא מתחבר ל-WiFi | בדוק שם רשת וסיסמה, וודא רשת 2.4GHz |
| לא מתחבר ל-Firebase | בדוק URL ו-Rules |
| נתונים לא מגיעים | בדוק חיבורי Serial ו-Baud Rate (9600) |
| לא מצליח להעלות קוד | לחץ על כפתור BOOT בזמן ההעלאה |

---

## קבצים

| קובץ | תיאור |
|------|--------|
| `ESP32_IoT_Bridge.ino` | קוד ה-ESP32 המלא |
| `docs/index.html` | דף מידע  |

---


## חיבור ESP-cam לRealtime Database
בכדי לחבר את הESP-cam למסד נתונים בfirebase תחילה נוריד את הספריות הבאות לArduinoIDE: <br>
את הספרייה Firebase-ESP32 על-ידי mobizt <br>
ואת הספרייה ArduinoJson על-ידי Benoit Blanchon (אם חיברתם את הלוח פיתוח אז זה מותקן לכם)

## פתיחת קובץ
בIDE פתחו את הדוגמה CameraWebServer 
```
- files → examples → ESP32 → Camera → CameraWebServer
```
 ותשימו בקובץ ino הראשי שנפתח את התוכנית בתיקיית ESPcam בקבצים כאן.


## קונפיגורציה של המצלמה
תשנו את החלקים הבאים לפי הפרוייקט שלכם: <br>
כתובת מסד נתונים ואת הנקודת גישה(WiFi) **מומלץ לשים נקודה חמה**
```
#define DATABASE_URL "robotic-laser-tank-default-rtdb.firebaseio.com"
...
...
const char *ssid = "oplus_co_apofwi";
const char *password = "hkum7108";
```
אחרי זה צריבה

## פיד המצלמה
במסד הנתונים ייפתח branch של כתתובת IP של המצלמה העתיקו אותו לשורת החיפוש בדפדן שלכם במחשב <br> 
**חשוב שהמחשב יהיה מחובר לאותו רשת כמו המצלמה**
גוללים למטה ולוחצים על הכפתור של Start stream והפיד ייפתח למטה




## רישיון

ESP IOT OS © 2025 by Shealtiel Yaish is licensed under CC BY-NC 4.0
</div>
