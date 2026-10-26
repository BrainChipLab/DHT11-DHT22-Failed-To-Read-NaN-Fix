# "Failed to Read from DHT Sensor!" (NaN) — Test Sketch & Wiring Guide

Test sketch with automatic retry, wiring tables (4-pin bare sensor vs. 3-pin module), and ESP32 pin checklist for fixing `"Failed to read from DHT sensor!"` and `NaN` readings on **DHT11** and **DHT22 (AM2302)**.

---

### 1. Required Libraries (Arduino IDE Library Manager)
1. **DHT sensor library** by Adafruit
2. **Adafruit Unified Sensor**

---

### 2. Test Sketch with Automatic Retry (Arduino Uno & ESP32 Core 3.x)
Waits 2 seconds on boot for the sensor to stabilize, and retries up to 3 times (waiting 2.1 seconds between attempts) if a single read fails:

```cpp
#include "DHT.h"

#define DHTPIN  4       // ESP32: GPIO 4  |  Arduino Uno: use 2
#define DHTTYPE DHT22   // Change to DHT11 if using a DHT11 sensor

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
  delay(2000);                         // let the sensor wake up
}

void loop() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  for (int i = 0; i < 3 && (isnan(h) || isnan(t)); i++) {
    delay(2100);                       // library reads at most every 2 s
    h = dht.readHumidity();
    t = dht.readTemperature();
  }

  if (isnan(h) || isnan(t)) {
    Serial.println("Failed to read from DHT sensor!");
  } else {
    Serial.print(t); Serial.print(" C   ");
    Serial.print(h); Serial.println(" %");
  }
  delay(2000);
}
```

---

### 3. Wiring Checklist (4-Pin Bare Sensor vs. 3-Pin Breakout Module)

#### A. 4-Pin Bare Sensor (Grille Facing You, Pins Pointing Down, Left -> Right)
| Sensor Pin | Function | **ESP32 DevKit** | **Arduino Uno / Nano** |
| :--- | :--- | :--- | :--- |
| **Pin 1** (Left) | `VCC` | `3V3` *(never 5V)* | `5V` |
| **Pin 2** | `DATA` | `GPIO 4` | `D2` |
| **Pin 3** | `NC` (Not Connected) | Leave unconnected | Leave unconnected |
| **Pin 4** (Right) | `GND` | `GND` | `GND` |
| **Pull-Up Resistor** | **10 kΩ** (4.7 kΩ – 10 kΩ) | Between `Pin 2 (DATA)` and `3V3` | Between `Pin 2 (DATA)` and `5V` |

#### B. 3-Pin Module (Built-in Pull-Up Resistor)
* Follow the printed labels on your specific PCB (`+ / out / -`, `VCC / DATA / GND`, or `S / + / -`).
* Connect `+` (`VCC`) to **`3V3`** on ESP32 or **`5V`** on Arduino Uno, `out` (`DATA` / `S`) to your GPIO pin, and `-` (`GND`) to `GND`.

---

### 4. ESP32 Pins to Avoid for DHT Sensors
The DHT protocol uses a single bidirectional data wire—the microcontroller must pull the line LOW to start a reading, then switch to input mode to listen:
* **Do NOT use `GPIO 34`, `35`, `36`, `39`:** These are **input-only** pins on classic ESP32 and cannot pull the data line low!
* **Do NOT use `GPIO 6–11`:** Connected to internal SPI flash.
* **Avoid Strapping Pins (`0`, `2`, `5`, `12`, `15`):** A pull-up on `GPIO 12` or `GPIO 2` can prevent booting or uploading.
* **Safe Pins:** `GPIO 4`, `16`, `17`, `18`, `19`, `21`, `22`, `23`, `25`, `26`, `27`, `32`, `33`.
