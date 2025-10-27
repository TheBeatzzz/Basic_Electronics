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
