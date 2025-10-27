# ESP32 Learning Guide

## What is ESP32?

The ESP32 is a powerful, low-cost microcontroller with built-in Wi-Fi and Bluetooth capabilities. It's perfect for IoT projects, home automation, robotics, and sensor applications.

### Key Features:
- **Dual-core processor** (up to 240 MHz)
- **Built-in Wi-Fi** (802.11 b/g/n)
- **Bluetooth** (Classic and BLE)
- **GPIO pins** (up to 34)
- **ADC** (Analog to Digital Converter)
- **DAC** (Digital to Analog Converter)
- **PWM, I2C, SPI, UART** interfaces
- **Low power consumption** modes

---

## Getting Started

### 1. Hardware You Need:
- ESP32 development board (ESP32-DevKitC, ESP32-WROOM, etc.)
- USB cable (usually Micro-USB or USB-C depending on your board)
- Computer with USB port
- Optional: Breadboard, LEDs, resistors, sensors

### 2. Software Setup:

#### Option A: Arduino IDE (Easiest for beginners)
1. Download Arduino IDE from https://www.arduino.cc/en/software
2. Install the ESP32 board support:
   - Open Arduino IDE
   - Go to **File > Preferences**
   - Add this URL to "Additional Board Manager URLs":
     ```
     https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
     ```
   - Go to **Tools > Board > Boards Manager**
   - Search for "ESP32" and install "esp32 by Espressif Systems"
3. Select your board: **Tools > Board > ESP32 Arduino > ESP32 Dev Module**
4. Select the correct COM port: **Tools > Port**

#### Option B: PlatformIO (More advanced, better for larger projects)
1. Install VS Code
2. Install PlatformIO extension
3. Create new project and select ESP32 board

#### Option C: ESP-IDF (Professional development)
- Official Espressif development framework
- More complex but offers full control

---

## Your First ESP32 Programs

### 1. Blink LED (Hello World)

```cpp
// Built-in LED is usually on GPIO 2
#define LED_PIN 2

void setup() {
  // Initialize the LED pin as output
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);  // Turn LED on
  delay(1000);                  // Wait 1 second
  digitalWrite(LED_PIN, LOW);   // Turn LED off
  delay(1000);                  // Wait 1 second
}
```

### 2. Serial Communication

```cpp
void setup() {
  Serial.begin(115200);  // Start serial communication at 115200 baud
  delay(1000);
  Serial.println("ESP32 is ready!");
}

void loop() {
  Serial.println("Hello from ESP32!");
  delay(2000);  // Print every 2 seconds
}
```

### 3. Reading Analog Input

```cpp
#define ANALOG_PIN 34  // Use GPIO 34 (ADC1_CH6)

void setup() {
  Serial.begin(115200);
  pinMode(ANALOG_PIN, INPUT);
}

void loop() {
  int value = analogRead(ANALOG_PIN);  // Read analog value (0-4095)
  float voltage = value * (3.3 / 4095.0);  // Convert to voltage
  
  Serial.print("Raw Value: ");
  Serial.print(value);
  Serial.print(" | Voltage: ");
  Serial.println(voltage);
  
  delay(500);
}
```

### 4. Button Input with Pull-up

```cpp
#define BUTTON_PIN 4
#define LED_PIN 2

void setup() {
  Serial.begin(115200);
  pinMode(BUTTON_PIN, INPUT_PULLUP);  // Enable internal pull-up resistor
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN);
  
  if (buttonState == LOW) {  // Button pressed (active LOW)
    digitalWrite(LED_PIN, HIGH);
    Serial.println("Button Pressed!");
  } else {
    digitalWrite(LED_PIN, LOW);
  }
  
  delay(50);  // Debounce delay
}
```

### 5. PWM (Pulse Width Modulation) - LED Dimming

```cpp
#define LED_PIN 2
#define PWM_CHANNEL 0
#define PWM_FREQUENCY 5000
#define PWM_RESOLUTION 8  // 8-bit (0-255)

void setup() {
  // Configure PWM
  ledcSetup(PWM_CHANNEL, PWM_FREQUENCY, PWM_RESOLUTION);
  ledcAttachPin(LED_PIN, PWM_CHANNEL);
}

void loop() {
  // Fade in
  for (int brightness = 0; brightness <= 255; brightness++) {
    ledcWrite(PWM_CHANNEL, brightness);
    delay(10);
  }
  
  // Fade out
  for (int brightness = 255; brightness >= 0; brightness--) {
    ledcWrite(PWM_CHANNEL, brightness);
    delay(10);
  }
}
```

---

## Wi-Fi Examples

### 6. Connect to Wi-Fi

```cpp
#include <WiFi.h>

const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

void setup() {
  Serial.begin(115200);
  delay(1000);
  
  Serial.println("Connecting to Wi-Fi...");
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nConnected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  // Your code here
}
```

### 7. Web Server

```cpp
#include <WiFi.h>
#include <WebServer.h>

const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

WebServer server(80);  // Create server on port 80

void handleRoot() {
  String html = "<html><body>";
  html += "<h1>ESP32 Web Server</h1>";
  html += "<p>Hello from ESP32!</p>";
  html += "</body></html>";
  server.send(200, "text/html", html);
}

void setup() {
  Serial.begin(115200);
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nConnected!");
  Serial.print("IP: ");
  Serial.println(WiFi.localIP());
  
  server.on("/", handleRoot);
  server.begin();
  Serial.println("Web server started");
}

void loop() {
  server.handleClient();
}
```

### 8. HTTP Request (Get data from web)

```cpp
#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected!");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    
    http.begin("http://api.github.com/");
    int httpCode = http.GET();
    
    if (httpCode > 0) {
      String payload = http.getString();
      Serial.println("Response:");
      Serial.println(payload);
    } else {
      Serial.println("Error in HTTP request");
    }
    
    http.end();
  }
  
  delay(10000);  // Request every 10 seconds
}
```

---

## ESP32 Function Generator

The ESP32 can be used as a function generator to create various waveforms using its DAC (Digital to Analog Converter) or PWM outputs.

### 9. Simple Function Generator with DAC

```cpp
// ESP32 has 2 DAC pins: GPIO 25 (DAC1) and GPIO 26 (DAC2)
#define DAC_PIN 25

void setup() {
  Serial.begin(115200);
}

void loop() {
  // Generate sine wave
  for (int i = 0; i < 360; i++) {
    float radians = i * PI / 180.0;
    int value = (sin(radians) + 1) * 127.5;  // Scale to 0-255
    dacWrite(DAC_PIN, value);
    delayMicroseconds(100);  // Controls frequency
  }
}
```

### 10. Multi-Waveform Function Generator

```cpp
#define DAC_PIN 25
#define BUTTON_PIN 4  // Button to switch waveforms

enum WaveType { SINE, SQUARE, TRIANGLE, SAWTOOTH };
WaveType currentWave = SINE;
int frequency = 100;  // Hz

void setup() {
  Serial.begin(115200);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  Serial.println("Function Generator Started");
  Serial.println("Press button to change waveform");
}

void generateSine() {
  for (int i = 0; i < 360; i++) {
    float radians = i * PI / 180.0;
    int value = (sin(radians) + 1) * 127.5;
    dacWrite(DAC_PIN, value);
    delayMicroseconds(1000000 / (frequency * 360));
  }
}

void generateSquare() {
  int halfPeriod = 500000 / frequency;  // microseconds
  dacWrite(DAC_PIN, 255);
  delayMicroseconds(halfPeriod);
  dacWrite(DAC_PIN, 0);
  delayMicroseconds(halfPeriod);
}

void generateTriangle() {
  // Rising edge
  for (int i = 0; i <= 255; i++) {
    dacWrite(DAC_PIN, i);
    delayMicroseconds(1000000 / (frequency * 512));
  }
  // Falling edge
  for (int i = 255; i >= 0; i--) {
    dacWrite(DAC_PIN, i);
    delayMicroseconds(1000000 / (frequency * 512));
  }
}

void generateSawtooth() {
  for (int i = 0; i <= 255; i++) {
    dacWrite(DAC_PIN, i);
    delayMicroseconds(1000000 / (frequency * 256));
  }
  dacWrite(DAC_PIN, 0);
}

void loop() {
  // Check button press to change waveform
  static bool lastButtonState = HIGH;
  bool buttonState = digitalRead(BUTTON_PIN);
  
  if (buttonState == LOW && lastButtonState == HIGH) {
    currentWave = (WaveType)((currentWave + 1) % 4);
    
    switch(currentWave) {
      case SINE: Serial.println("Waveform: SINE"); break;
      case SQUARE: Serial.println("Waveform: SQUARE"); break;
      case TRIANGLE: Serial.println("Waveform: TRIANGLE"); break;
      case SAWTOOTH: Serial.println("Waveform: SAWTOOTH"); break;
    }
    delay(200);  // Debounce
  }
  lastButtonState = buttonState;
  
  // Generate selected waveform
  switch(currentWave) {
    case SINE:
      generateSine();
      break;
    case SQUARE:
      generateSquare();
      break;
    case TRIANGLE:
      generateTriangle();
      break;
    case SAWTOOTH:
      generateSawtooth();
      break;
  }
}
```

### 11. Function Generator with Serial Control

```cpp
#define DAC_PIN 25

String waveform = "sine";
int frequency = 100;
int amplitude = 255;  // 0-255 for DAC

void setup() {
  Serial.begin(115200);
  Serial.println("\n=== ESP32 Function Generator ===");
  Serial.println("Commands:");
  Serial.println("  wave:sine|square|triangle|sawtooth");
  Serial.println("  freq:100 (in Hz, 1-10000)");
  Serial.println("  amp:255 (0-255)");
  Serial.println("  start|stop");
}

bool running = true;

void loop() {
  // Check for serial commands
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    cmd.trim();
    
    if (cmd.startsWith("wave:")) {
      waveform = cmd.substring(5);
      Serial.println("Waveform set to: " + waveform);
    }
    else if (cmd.startsWith("freq:")) {
      frequency = cmd.substring(5).toInt();
      frequency = constrain(frequency, 1, 10000);
      Serial.println("Frequency set to: " + String(frequency) + " Hz");
    }
    else if (cmd.startsWith("amp:")) {
      amplitude = cmd.substring(4).toInt();
      amplitude = constrain(amplitude, 0, 255);
      Serial.println("Amplitude set to: " + String(amplitude));
    }
    else if (cmd == "start") {
      running = true;
      Serial.println("Started");
    }
    else if (cmd == "stop") {
      running = false;
      dacWrite(DAC_PIN, 0);
      Serial.println("Stopped");
    }
  }
  
  if (running) {
    if (waveform == "sine") {
      generateSineWave();
    } else if (waveform == "square") {
      generateSquareWave();
    } else if (waveform == "triangle") {
      generateTriangleWave();
    } else if (waveform == "sawtooth") {
      generateSawtoothWave();
    }
  }
}

void generateSineWave() {
  for (int i = 0; i < 360; i++) {
    float radians = i * PI / 180.0;
    int value = ((sin(radians) + 1) * amplitude) / 2;
    dacWrite(DAC_PIN, value);
    delayMicroseconds(1000000 / (frequency * 360));
    
    if (Serial.available()) break;  // Check for new commands
  }
}

void generateSquareWave() {
  int halfPeriod = 500000 / frequency;
  dacWrite(DAC_PIN, amplitude);
  delayMicroseconds(halfPeriod);
  dacWrite(DAC_PIN, 0);
  delayMicroseconds(halfPeriod);
}

void generateTriangleWave() {
  int steps = 256;
  int delayTime = 1000000 / (frequency * steps * 2);
  
  for (int i = 0; i <= steps; i++) {
    dacWrite(DAC_PIN, (i * amplitude) / steps);
    delayMicroseconds(delayTime);
    if (Serial.available()) break;
  }
  
  for (int i = steps; i >= 0; i--) {
    dacWrite(DAC_PIN, (i * amplitude) / steps);
    delayMicroseconds(delayTime);
    if (Serial.available()) break;
  }
}

void generateSawtoothWave() {
  int steps = 256;
  int delayTime = 1000000 / (frequency * steps);
  
  for (int i = 0; i <= steps; i++) {
    dacWrite(DAC_PIN, (i * amplitude) / steps);
    delayMicroseconds(delayTime);
    if (Serial.available()) break;
  }
  dacWrite(DAC_PIN, 0);
}
```

### 12. High-Frequency PWM Function Generator

For higher frequencies, use PWM instead of DAC:

```cpp
#define PWM_PIN 2
#define PWM_CHANNEL 0
#define PWM_FREQUENCY 50000  // 50 kHz PWM frequency
#define PWM_RESOLUTION 8     // 8-bit resolution

void setup() {
  Serial.begin(115200);
  
  // Setup PWM
  ledcSetup(PWM_CHANNEL, PWM_FREQUENCY, PWM_RESOLUTION);
  ledcAttachPin(PWM_PIN, PWM_CHANNEL);
  
  Serial.println("High-frequency PWM generator ready");
}

void loop() {
  // Generate a 1 kHz sine wave using PWM
  for (int i = 0; i < 360; i++) {
    float radians = i * PI / 180.0;
    int dutyCycle = (sin(radians) + 1) * 127.5;
    ledcWrite(PWM_CHANNEL, dutyCycle);
    delayMicroseconds(2778);  // 1 kHz = 1000 us per cycle, 360 steps
  }
}
```

### Function Generator Specifications:

**Using DAC (GPIO 25, 26):**
- Output voltage: 0 - 3.3V
- Resolution: 8-bit (256 levels)
- Maximum practical frequency: ~10 kHz
- Output impedance: ~1kΩ
- Best for: Audio signals, low-frequency signals

**Using PWM:**
- Output: Digital pulses (requires low-pass filter for analog)
- PWM frequency: Up to 40 MHz
- Resolution: 1-16 bit (configurable)
- Best for: Motor control, LED dimming, high-frequency signals

### Adding a Low-Pass Filter:

For cleaner analog output from PWM, add a simple RC low-pass filter:

```
PWM Pin ----[10kΩ]---- Output
                |
              [1µF]
                |
               GND
```

Cutoff frequency = 1 / (2π × R × C) ≈ 15.9 Hz

For different frequencies, adjust R and C values:
- **1 kHz cutoff**: R=1.6kΩ, C=0.1µF
- **10 kHz cutoff**: R=1.6kΩ, C=0.01µF

### Function Generator Project Enhancements:

1. **OLED Display**: Show current waveform, frequency, and amplitude
2. **Rotary Encoder**: Adjust frequency and amplitude
3. **Multiple Outputs**: Use both DAC channels for dual output
4. **Web Interface**: Control via Wi-Fi
5. **Sweep Mode**: Frequency sweep from start to end
6. **Modulation**: AM/FM modulation capabilities
7. **Preset Storage**: Save and recall settings

### Example: Function Generator with OLED Display

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

#define DAC_PIN 25
#define BUTTON_WAVE 4
#define BUTTON_FREQ_UP 5
#define BUTTON_FREQ_DOWN 18

int waveType = 0;  // 0=sine, 1=square, 2=triangle, 3=sawtooth
int frequency = 100;

void setup() {
  Serial.begin(115200);
  
  pinMode(BUTTON_WAVE, INPUT_PULLUP);
  pinMode(BUTTON_FREQ_UP, INPUT_PULLUP);
  pinMode(BUTTON_FREQ_DOWN, INPUT_PULLUP);
  
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println("SSD1306 allocation failed");
  }
  
  updateDisplay();
}

void updateDisplay() {
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);
  
  display.setCursor(0, 0);
  display.println("Function Generator");
  
  display.setCursor(0, 20);
  display.print("Wave: ");
  switch(waveType) {
    case 0: display.println("Sine"); break;
    case 1: display.println("Square"); break;
    case 2: display.println("Triangle"); break;
    case 3: display.println("Sawtooth"); break;
  }
  
  display.setCursor(0, 35);
  display.print("Freq: ");
  display.print(frequency);
  display.println(" Hz");
  
  display.display();
}

void loop() {
  // Button handling
  if (digitalRead(BUTTON_WAVE) == LOW) {
    waveType = (waveType + 1) % 4;
    updateDisplay();
    delay(200);
  }
  
  if (digitalRead(BUTTON_FREQ_UP) == LOW) {
    frequency += 10;
    if (frequency > 1000) frequency = 1000;
    updateDisplay();
    delay(100);
  }
  
  if (digitalRead(BUTTON_FREQ_DOWN) == LOW) {
    frequency -= 10;
    if (frequency < 10) frequency = 10;
    updateDisplay();
    delay(100);
  }
  
  // Generate waveform (simplified for example)
  generateWave();
}

void generateWave() {
  // Add your waveform generation code here
  // Similar to previous examples
}
```

---

## Important ESP32 GPIO Notes

### ADC (Analog Input) Pins:
- **ADC1**: GPIO 32, 33, 34, 35, 36, 39 (Can use with Wi-Fi)
- **ADC2**: GPIO 0, 2, 4, 12-15, 25-27 (Cannot use when Wi-Fi is active)

### Special Pins to Avoid:
- **GPIO 0**: Boot mode selection (avoid using)
- **GPIO 2**: Built-in LED, also used for boot
- **GPIO 12**: Boot voltage selection (avoid)
- **GPIO 6-11**: Connected to flash memory (DO NOT USE)
- **GPIO 34-39**: Input only (no pull-up/pull-down resistors)

### Safe Pins for General Use:
- GPIO 13, 14, 15, 16, 17, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33

---

## Common ESP32 Libraries

### Essential Libraries:
```cpp
#include <WiFi.h>           // Wi-Fi functionality
#include <WebServer.h>      // Web server
#include <HTTPClient.h>     // HTTP client
#include <Wire.h>           // I2C communication
#include <SPI.h>            // SPI communication
#include <BluetoothSerial.h> // Bluetooth
#include <SPIFFS.h>         // File system
#include <Preferences.h>    // Non-volatile storage
```

---

## Troubleshooting

### Can't Upload Code:
1. Hold the **BOOT** button while uploading
2. Check the correct COM port is selected
3. Try a different USB cable
4. Install CP2102 or CH340 drivers
5. Lower upload speed (Tools > Upload Speed > 115200)

### ESP32 Keeps Restarting:
1. Check power supply (needs stable 5V, at least 500mA)
2. Add capacitor (10µF-100µF) between VIN and GND
3. Check for short circuits

### Wi-Fi Not Connecting:
1. Verify SSID and password
2. Check if Wi-Fi is 2.4 GHz (ESP32 doesn't support 5 GHz)
3. Move closer to router
4. Check if network is hidden

---

## Project Ideas for Learning

### Beginner:
1. LED blink patterns
2. Temperature sensor reader (DHT11/DHT22)
3. Button-controlled LED
4. Light sensor (photoresistor)
5. Serial monitor calculator

### Intermediate:
1. Wi-Fi weather station
2. Web-controlled LED
3. Motion detector with notification
4. OLED display with sensor data
5. Bluetooth serial terminal

### Advanced:
1. Home automation system
2. IoT data logger with cloud storage
3. ESP-NOW wireless communication
4. Camera streaming (ESP32-CAM)
5. Voice-controlled devices

---

## Useful Resources

### Official Documentation:
- ESP32 Arduino Core: https://docs.espressif.com/projects/arduino-esp32/
- ESP-IDF Programming Guide: https://docs.espressif.com/projects/esp-idf/

### Community & Tutorials:
- Random Nerd Tutorials: https://randomnerdtutorials.com/
- ESP32.com Forum: https://www.esp32.com/
- Arduino Forum ESP32 Section

### Pin Reference:
- Always check your specific board's pinout diagram
- Different ESP32 boards may have different pin layouts

---

## Next Steps

1. **Start Simple**: Begin with blink and serial examples
2. **Understand GPIO**: Learn how to read and write digital/analog signals
3. **Add Sensors**: Connect temperature, light, or motion sensors
4. **Go Wireless**: Experiment with Wi-Fi and Bluetooth
5. **Build Projects**: Apply what you learned in real projects
6. **Join Community**: Ask questions, share projects

---

## Safety Notes

⚠️ **Important:**
- ESP32 operates at **3.3V logic level**
- GPIO pins can handle max **12mA** each
- Total GPIO current should not exceed **40mA**
- Use resistors with LEDs (220Ω - 1kΩ)
- Don't connect 5V directly to GPIO pins
- Use level shifters for 5V sensors

---

Good luck with your ESP32 journey! Start with simple examples and gradually move to more complex projects. 🚀
