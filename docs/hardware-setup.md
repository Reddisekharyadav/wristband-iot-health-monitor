# Hardware Setup Guide

## Required Components

### Main Components
- **Microcontroller**: ESP32 or Arduino IoT compatible board with WiFi
- **Heart Rate & SpO2**: MAX30102 sensor module
- **Motion Sensor**: MPU6050 (Accelerometer + Gyroscope)
- **GSR Sensor**: Galvanic Skin Response sensor
- **LED**: Status indicator LED
- **Resistors**: 220Ω for LED, 10kΩ pull-up resistors for I2C (if not built-in)

### Additional Materials
- Breadboard or custom PCB
- Jumper wires (Male-to-Male, Male-to-Female)
- USB cable for programming
- Power supply (3.7V LiPo battery or USB power)
- Wristband enclosure or 3D printed case

## Assembly Instructions

### Step 1: Prepare the Breadboard
1. Place the ESP32/Arduino board on the breadboard
2. Ensure power rails are connected properly

### Step 2: Connect Power Rails
```
ESP32 3.3V → Positive Rail (Red)
ESP32 GND  → Negative Rail (Blue/Black)
```

### Step 3: Connect GSR Sensor
```
GSR Signal → Arduino A0
GSR VCC   → 3.3V
GSR GND   → GND
```

### Step 4: Connect MAX30102 (I2C)
```
MAX30102 VIN → 3.3V
MAX30102 GND → GND
MAX30102 SDA → Arduino SDA (Pin 21 on ESP32)
MAX30102 SCL → Arduino SCL (Pin 22 on ESP32)
```

**Note**: If using 5V board, connect VIN to 5V instead.

### Step 5: Connect MPU6050 (I2C)
```
MPU6050 VCC → 5V (or 3.3V depending on module)
MPU6050 GND → GND
MPU6050 SDA → Arduino SDA (Pin 21 on ESP32)
MPU6050 SCL → Arduino SCL (Pin 22 on ESP32)
```

**Important**: Both I2C devices share the same SDA/SCL lines.

### Step 6: Connect Status LED
```
LED Anode (+) → Arduino Pin 13
LED Cathode (-) → 220Ω Resistor → GND
```

### Step 7: Add Pull-up Resistors (If Needed)
Some boards have built-in I2C pull-ups. If you experience communication issues:
```
10kΩ Resistor: SDA → 3.3V
10kΩ Resistor: SCL → 3.3V
```

## Wiring Diagram

```
                  ESP32/Arduino
                  ┌───────────┐
                  │           │
    GSR Sensor ───┤ A0        │
                  │           │
    MAX30102  ────┤ SDA   GND ├─── GND Rail
    MPU6050   ────┤ SCL   3V3 ├─── 3.3V Rail
                  │           │
    LED ──────────┤ D13       │
                  │           │
                  └───────────┘
```

## Testing Hardware

### Test 1: Power Check
1. Connect USB cable
2. Check if power LED lights up on all sensors
3. Verify voltage levels with multimeter:
   - 3.3V rail: 3.2-3.4V
   - 5V rail: 4.8-5.2V

### Test 2: I2C Device Detection
Upload and run this I2C scanner sketch:

```cpp
#include <Wire.h>

void setup() {
  Wire.begin();
  Serial.begin(9600);
  Serial.println("I2C Scanner");
}

void loop() {
  byte error, address;
  int nDevices = 0;
  
  Serial.println("Scanning...");
  
  for(address = 1; address < 127; address++) {
    Wire.beginTransmission(address);
    error = Wire.endTransmission();
    
    if (error == 0) {
      Serial.print("Device found at 0x");
      if (address < 16) Serial.print("0");
      Serial.println(address, HEX);
      nDevices++;
    }
  }
  
  if (nDevices == 0)
    Serial.println("No I2C devices found");
  else
    Serial.println("Scan complete");
  
  delay(5000);
}
```

**Expected Output**:
```
Scanning...
Device found at 0x57  (MAX30102)
Device found at 0x68  (MPU6050)
Scan complete
```

### Test 3: Individual Sensor Tests

#### GSR Sensor Test
```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  int value = analogRead(A0);
  Serial.print("GSR: ");
  Serial.println(value);
  delay(100);
}
```

Expected: Values between 0-1023, changing when you touch sensor

#### MAX30102 Test
Use the example from DFRobot_MAX30102 library:
`File → Examples → DFRobot_MAX30102 → HeartRate`

#### MPU6050 Test
Use the example from MPU6050 library:
`File → Examples → MPU6050 → MPU6050_raw`

## Enclosure Design

### Wristband Mounting
1. Design or purchase a wristband case
2. Position MAX30102 to contact skin on wrist
3. Mount GSR sensors on two finger contact points
4. Secure MPU6050 flat against wrist
5. Add battery compartment
6. Include charging port access

### 3D Printing Files
*(Coming Soon - Add STL files to `/hardware` folder)*

## Power Management

### Battery Option
- **Recommended**: 3.7V 1000mAh LiPo battery
- **Charging**: Add TP4056 charging module
- **Runtime**: ~8-12 hours depending on WiFi usage

### USB Power
- Suitable for testing and development
- Not portable but unlimited runtime

## Safety Considerations

⚠️ **Important Safety Notes**:
- Do not use on medical-grade applications without proper certification
- Ensure proper insulation of all connections
- Do not submerge in water (unless using waterproof components)
- Discontinue use if skin irritation occurs
- This device is for educational/experimental purposes only

## Troubleshooting Hardware

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| No power to sensors | Loose connection | Check power rail connections |
| I2C device not found | Wrong address/wiring | Verify SDA/SCL connections |
| Erratic readings | Electrical noise | Add decoupling capacitors (0.1µF) |
| GSR not responding | Poor contact | Ensure clean, dry skin contact |
| MAX30102 no pulse | Not on skin | Position sensor directly on wrist |

## Next Steps

After hardware assembly:
1. Proceed to [Software Configuration](software-config.md)
2. Set up [Arduino IoT Cloud](arduino-cloud-setup.md)
3. Upload the main sketch

---

For questions, open an issue on GitHub.
