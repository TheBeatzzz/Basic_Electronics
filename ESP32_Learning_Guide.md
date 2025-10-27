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
## Servo Motor Control

Servo motors are commonly used for precise angular positioning. The ESP32 can control servos using PWM signals.

### Hardware Setup:
- **Servo Wire Colors:**
  - Brown/Black: GND
  - Red: VCC (4.8V - 6V)
  - Orange/Yellow/White: Signal (PWM)

- **Important:** Power servos from external 5V supply (not ESP32's 3.3V pin) for better performance
- Connect grounds together (ESP32 GND and external power GND)

### 13. Basic Servo Control

```cpp
#include <ESP32Servo.h>

Servo myServo;
#define SERVO_PIN 18

void setup() {
  Serial.begin(115200);
  
  // Allow allocation of all timers for servo library
  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  ESP32PWM::allocateTimer(2);
  ESP32PWM::allocateTimer(3);
  
  myServo.attach(SERVO_PIN);  // Attach servo to pin 18
  Serial.println("Servo Ready!");
}

void loop() {
  // Sweep from 0 to 180 degrees
  for (int angle = 0; angle <= 180; angle++) {
    myServo.write(angle);
    Serial.print("Angle: ");
    Serial.println(angle);
    delay(15);  // Wait for servo to reach position
  }
  
  delay(1000);
  
  // Sweep from 180 to 0 degrees
  for (int angle = 180; angle >= 0; angle--) {
    myServo.write(angle);
    Serial.print("Angle: ");
    Serial.println(angle);
    delay(15);
  }
  
  delay(1000);
}
```

### 14. Servo Control with Serial Commands

```cpp
#include <ESP32Servo.h>

Servo myServo;
#define SERVO_PIN 18

void setup() {
  Serial.begin(115200);
  
  ESP32PWM::allocateTimer(0);
  myServo.attach(SERVO_PIN);
  
  Serial.println("=== ESP32 Servo Control ===");
  Serial.println("Commands:");
  Serial.println("  angle:<0-180> - Set specific angle");
  Serial.println("  sweep - Sweep 0 to 180");
  Serial.println("  center - Move to center (90°)");
  Serial.println("  min - Move to minimum (0°)");
  Serial.println("  max - Move to maximum (180°)");
}

void loop() {
  if (Serial.available()) {
    String command = Serial.readStringUntil('\n');
    command.trim();
    
    if (command.startsWith("angle:")) {
      int angle = command.substring(6).toInt();
      angle = constrain(angle, 0, 180);
      myServo.write(angle);
      Serial.print("Moved to: ");
      Serial.print(angle);
      Serial.println("°");
    }
    else if (command == "sweep") {
      Serial.println("Sweeping...");
      for (int i = 0; i <= 180; i++) {
        myServo.write(i);
        delay(15);
      }
      for (int i = 180; i >= 0; i--) {
        myServo.write(i);
        delay(15);
      }
      Serial.println("Sweep complete");
    }
    else if (command == "center") {
      myServo.write(90);
      Serial.println("Moved to center (90°)");
    }
    else if (command == "min") {
      myServo.write(0);
      Serial.println("Moved to minimum (0°)");
    }
    else if (command == "max") {
      myServo.write(180);
      Serial.println("Moved to maximum (180°)");
    }
    else {
      Serial.println("Unknown command");
    }
  }
}
```

### 15. Multi-Servo Control

```cpp
#include <ESP32Servo.h>

Servo servo1;
Servo servo2;
Servo servo3;

#define SERVO1_PIN 18
#define SERVO2_PIN 19
#define SERVO3_PIN 21

void setup() {
  Serial.begin(115200);
  
  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  ESP32PWM::allocateTimer(2);
  
  servo1.attach(SERVO1_PIN);
  servo2.attach(SERVO2_PIN);
  servo3.attach(SERVO3_PIN);
  
  // Initialize all servos to center position
  servo1.write(90);
  servo2.write(90);
  servo3.write(90);
  
  Serial.println("3-Servo Controller Ready");
  Serial.println("Format: servo_number,angle (e.g., 1,90)");
}

void loop() {
  if (Serial.available()) {
    String input = Serial.readStringUntil('\n');
    input.trim();
    
    int commaIndex = input.indexOf(',');
    if (commaIndex > 0) {
      int servoNum = input.substring(0, commaIndex).toInt();
      int angle = input.substring(commaIndex + 1).toInt();
      angle = constrain(angle, 0, 180);
      
      switch(servoNum) {
        case 1:
          servo1.write(angle);
          Serial.print("Servo 1 -> ");
          break;
        case 2:
          servo2.write(angle);
          Serial.print("Servo 2 -> ");
          break;
        case 3:
          servo3.write(angle);
          Serial.print("Servo 3 -> ");
          break;
        default:
          Serial.println("Invalid servo number (1-3)");
          return;
      }
      Serial.print(angle);
      Serial.println("°");
    }
  }
}
```

### 16. Servo Control with Potentiometer

```cpp
#include <ESP32Servo.h>

Servo myServo;
#define SERVO_PIN 18
#define POT_PIN 34  // Potentiometer on ADC1

void setup() {
  Serial.begin(115200);
  
  ESP32PWM::allocateTimer(0);
  myServo.attach(SERVO_PIN);
  pinMode(POT_PIN, INPUT);
  
  Serial.println("Servo controlled by potentiometer");
}

void loop() {
  // Read potentiometer value (0-4095)
  int potValue = analogRead(POT_PIN);
  
  // Map to servo angle (0-180)
  int angle = map(potValue, 0, 4095, 0, 180);
  
  // Move servo
  myServo.write(angle);
  
  // Print values
  Serial.print("Pot: ");
  Serial.print(potValue);
  Serial.print(" | Angle: ");
  Serial.println(angle);
  
  delay(15);  // Small delay for servo stability
}
```

### 17. Servo Control with Buttons

```cpp
#include <ESP32Servo.h>

Servo myServo;
#define SERVO_PIN 18
#define BUTTON_LEFT 4
#define BUTTON_RIGHT 5
#define BUTTON_CENTER 19

int currentAngle = 90;
int stepSize = 5;  // Degrees per button press

void setup() {
  Serial.begin(115200);
  
  ESP32PWM::allocateTimer(0);
  myServo.attach(SERVO_PIN);
  
  pinMode(BUTTON_LEFT, INPUT_PULLUP);
  pinMode(BUTTON_RIGHT, INPUT_PULLUP);
  pinMode(BUTTON_CENTER, INPUT_PULLUP);
  
  myServo.write(currentAngle);
  Serial.println("Servo Button Control");
  Serial.println("Left: Decrease | Right: Increase | Center: Reset to 90°");
}

void loop() {
  // Read buttons (active LOW)
  bool leftPressed = (digitalRead(BUTTON_LEFT) == LOW);
  bool rightPressed = (digitalRead(BUTTON_RIGHT) == LOW);
  bool centerPressed = (digitalRead(BUTTON_CENTER) == LOW);
  
  if (leftPressed) {
    currentAngle -= stepSize;
    currentAngle = constrain(currentAngle, 0, 180);
    myServo.write(currentAngle);
    Serial.print("← Angle: ");
    Serial.println(currentAngle);
    delay(200);  // Debounce
  }
  
  if (rightPressed) {
    currentAngle += stepSize;
    currentAngle = constrain(currentAngle, 0, 180);
    myServo.write(currentAngle);
    Serial.print("→ Angle: ");
    Serial.println(currentAngle);
    delay(200);  // Debounce
  }
  
  if (centerPressed) {
    currentAngle = 90;
    myServo.write(currentAngle);
    Serial.println("↺ Reset to 90°");
    delay(200);  // Debounce
  }
}
```

### 18. Servo Control with Web Interface

```cpp
#include <WiFi.h>
#include <WebServer.h>
#include <ESP32Servo.h>

const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

WebServer server(80);
Servo myServo;

#define SERVO_PIN 18
int currentAngle = 90;

void handleRoot() {
  String html = R"(
<!DOCTYPE html>
<html>
<head>
  <meta name='viewport' content='width=device-width, initial-scale=1'>
  <title>ESP32 Servo Control</title>
  <style>
    body { font-family: Arial; text-align: center; margin: 20px; }
    h1 { color: #0066cc; }
    .slider { width: 80%; max-width: 400px; }
    .angle { font-size: 48px; color: #00aa00; margin: 20px; }
    button { 
      padding: 15px 30px; 
      font-size: 18px; 
      margin: 10px; 
      cursor: pointer;
      border: none;
      border-radius: 5px;
      background: #0066cc;
      color: white;
    }
    button:hover { background: #0052a3; }
  </style>
</head>
<body>
  <h1>🎛️ ESP32 Servo Control</h1>
  <div class='angle' id='angleDisplay'>90°</div>
  <input type='range' min='0' max='180' value='90' class='slider' id='angleSlider'>
  <br><br>
  <button onclick='setAngle(0)'>0°</button>
  <button onclick='setAngle(45)'>45°</button>
  <button onclick='setAngle(90)'>90°</button>
  <button onclick='setAngle(135)'>135°</button>
  <button onclick='setAngle(180)'>180°</button>
  
  <script>
    var slider = document.getElementById('angleSlider');
    var display = document.getElementById('angleDisplay');
    
    slider.oninput = function() {
      setAngle(this.value);
    }
    
    function setAngle(angle) {
      fetch('/servo?angle=' + angle)
        .then(response => response.text())
        .then(data => {
          display.innerHTML = angle + '°';
          slider.value = angle;
        });
    }
  </script>
</body>
</html>
  )";
  server.send(200, "text/html", html);
}

void handleServo() {
  if (server.hasArg("angle")) {
    currentAngle = server.arg("angle").toInt();
    currentAngle = constrain(currentAngle, 0, 180);
    myServo.write(currentAngle);
    server.send(200, "text/plain", "OK");
    Serial.print("Servo moved to: ");
    Serial.println(currentAngle);
  } else {
    server.send(400, "text/plain", "Missing angle parameter");
  }
}

void setup() {
  Serial.begin(115200);
  
  // Setup servo
  ESP32PWM::allocateTimer(0);
  myServo.attach(SERVO_PIN);
  myServo.write(currentAngle);
  
  // Connect to Wi-Fi
  Serial.println("Connecting to Wi-Fi...");
  WiFi.begin(ssid, password);
  
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  Serial.println("\nConnected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
  
  // Setup web server
  server.on("/", handleRoot);
  server.on("/servo", handleServo);
  server.begin();
  
  Serial.println("Web server started");
  Serial.println("Open your browser and go to: http://" + WiFi.localIP().toString());
}

void loop() {
  server.handleClient();
}
```

### 19. Robotic Arm / Pan-Tilt Control

```cpp
#include <ESP32Servo.h>

// Define servos
Servo panServo;   // Horizontal rotation
Servo tiltServo;  // Vertical tilt

#define PAN_PIN 18
#define TILT_PIN 19

int panAngle = 90;
int tiltAngle = 90;

void setup() {
  Serial.begin(115200);
  
  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  
  panServo.attach(PAN_PIN);
  tiltServo.attach(TILT_PIN);
  
  // Center both servos
  panServo.write(panAngle);
  tiltServo.write(tiltAngle);
  
  Serial.println("=== Pan-Tilt Control ===");
  Serial.println("Commands:");
  Serial.println("  w/s - Tilt up/down");
  Serial.println("  a/d - Pan left/right");
  Serial.println("  c - Center both");
  Serial.println("  pan:<angle> - Set pan angle");
  Serial.println("  tilt:<angle> - Set tilt angle");
}

void loop() {
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    cmd.trim();
    cmd.toLowerCase();
    
    if (cmd == "w") {
      tiltAngle = constrain(tiltAngle + 5, 0, 180);
      tiltServo.write(tiltAngle);
      Serial.println("Tilt up: " + String(tiltAngle));
    }
    else if (cmd == "s") {
      tiltAngle = constrain(tiltAngle - 5, 0, 180);
      tiltServo.write(tiltAngle);
      Serial.println("Tilt down: " + String(tiltAngle));
    }
    else if (cmd == "a") {
      panAngle = constrain(panAngle - 5, 0, 180);
      panServo.write(panAngle);
      Serial.println("Pan left: " + String(panAngle));
    }
    else if (cmd == "d") {
      panAngle = constrain(panAngle + 5, 0, 180);
      panServo.write(panAngle);
      Serial.println("Pan right: " + String(panAngle));
    }
    else if (cmd == "c") {
      panAngle = 90;
      tiltAngle = 90;
      panServo.write(panAngle);
      tiltServo.write(tiltAngle);
      Serial.println("Centered: Pan=90°, Tilt=90°");
    }
    else if (cmd.startsWith("pan:")) {
      panAngle = cmd.substring(4).toInt();
      panAngle = constrain(panAngle, 0, 180);
      panServo.write(panAngle);
      Serial.println("Pan set to: " + String(panAngle));
    }
    else if (cmd.startsWith("tilt:")) {
      tiltAngle = cmd.substring(5).toInt();
      tiltAngle = constrain(tiltAngle, 0, 180);
      tiltServo.write(tiltAngle);
      Serial.println("Tilt set to: " + String(tiltAngle));
    }
    else {
      Serial.println("Unknown command");
    }
  }
}
```

### 20. Servo Choreography / Animation Sequences

```cpp
#include <ESP32Servo.h>

Servo myServo;
#define SERVO_PIN 18

// Structure to hold animation steps
struct AnimationStep {
  int angle;
  int delayMs;
};

// Define animation sequences
AnimationStep waveAnimation[] = {
  {90, 500}, {120, 300}, {90, 300}, {120, 300}, {90, 500}
};

AnimationStep scanAnimation[] = {
  {0, 100}, {30, 100}, {60, 100}, {90, 100}, 
  {120, 100}, {150, 100}, {180, 100}, {150, 100},
  {120, 100}, {90, 100}, {60, 100}, {30, 100}, {0, 100}
};

AnimationStep nodAnimation[] = {
  {90, 300}, {60, 400}, {90, 300}, {60, 400}, {90, 500}
};

void setup() {
  Serial.begin(115200);
  
  ESP32PWM::allocateTimer(0);
  myServo.attach(SERVO_PIN);
  myServo.write(90);
  
  Serial.println("=== Servo Animation Controller ===");
  Serial.println("Commands:");
  Serial.println("  wave - Wave motion");
  Serial.println("  scan - Scanning motion");
  Serial.println("  nod - Nodding motion");
  Serial.println("  loop - Continuous loop");
}

void playAnimation(AnimationStep animation[], int steps) {
  for (int i = 0; i < steps; i++) {
    myServo.write(animation[i].angle);
    Serial.print("Step ");
    Serial.print(i + 1);
    Serial.print(": ");
    Serial.print(animation[i].angle);
    Serial.println("°");
    delay(animation[i].delayMs);
  }
  Serial.println("Animation complete");
}

void loop() {
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    cmd.trim();
    
    if (cmd == "wave") {
      Serial.println("Playing wave animation...");
      playAnimation(waveAnimation, 5);
    }
    else if (cmd == "scan") {
      Serial.println("Playing scan animation...");
      playAnimation(scanAnimation, 13);
    }
    else if (cmd == "nod") {
      Serial.println("Playing nod animation...");
      playAnimation(nodAnimation, 5);
    }
    else if (cmd == "loop") {
      Serial.println("Continuous loop mode (press any key to stop)");
      while (!Serial.available()) {
        playAnimation(waveAnimation, 5);
        delay(500);
      }
      Serial.read();  // Clear buffer
      Serial.println("Loop stopped");
    }
    else {
      Serial.println("Unknown command");
    }
  }
}
```

### Servo Motor Tips & Best Practices:

**Power Supply:**
- Use external 5V power supply (1A+ for standard servos)
- Connect ESP32 GND to power supply GND (common ground)
- Don't power servos from ESP32's 3.3V or 5V pins

**Signal Connection:**
- Servo signal wire connects to ESP32 GPIO (3.3V tolerant)
- Most servos work fine with 3.3V logic signals
- If issues occur, use a level shifter (3.3V → 5V)

**PWM Settings:**
- Standard servo: 50 Hz (20ms period)
- Pulse width: 1ms (0°) to 2ms (180°)
- ESP32Servo library handles this automatically

**Calibration:**
- Different servos may have slightly different ranges
- Some servos: 500-2500 μs instead of standard 1000-2000 μs
- Use `myServo.attach(pin, min, max)` for custom ranges

**Common Issues:**
1. **Servo jittering**: Add capacitor (100-470μF) across power supply
2. **ESP32 resets**: Insufficient power supply current
3. **Servo doesn't move**: Check wiring, especially ground connection
4. **Servo buzzing at position**: Normal for most servos

**Installing ESP32Servo Library:**
1. Open Arduino IDE
2. Go to **Sketch > Include Library > Manage Libraries**
3. Search for "ESP32Servo"
4. Install "ESP32Servo by Kevin Harrington"
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
### Installing USB-to-Serial Drivers (CP2102 / CH340)

Most ESP32 boards use either CP2102 or CH340 USB-to-serial chips. If your computer doesn't recognize the ESP32, you need to install the appropriate driver.

#### How to Identify Which Driver You Need:

1. **Check your ESP32 board** - Look for a small chip near the USB port
   - If it says "CP2102" or "CP2104" → You need CP2102 drivers
   - If it says "CH340G" or "CH340C" → You need CH340 drivers

2. **Check Device Manager (Windows)**:
   - Press `Win + X` and select "Device Manager"
   - Look under "Ports (COM & LPT)" or "Other devices"
   - If you see "CP210x" → CP2102 driver needed
   - If you see "CH340" or "USB2.0-Serial" → CH340 driver needed
   - If you see a device with a yellow warning icon → Driver needed

#### Installing CP2102 Drivers:

**Windows:**
1. Download from: https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers
2. Click "Downloads" tab
3. Download "CP210x Universal Windows Driver"
4. Extract the ZIP file
5. Run `CP210xVCPInstaller_x64.exe` (or x86 for 32-bit Windows)
6. Follow the installation wizard
7. Restart your computer
8. Reconnect ESP32 - should now appear as COM port

**macOS:**
1. Download from: https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers
2. Download "CP210x VCP Mac OSX Driver"
3. Open the .dmg file
4. Run the installer package
5. Go to System Preferences → Security & Privacy
6. Click "Allow" if installation was blocked
7. Restart your Mac
8. Reconnect ESP32

**Linux:**
- Most Linux distributions include CP210x drivers by default
- Check with: `lsmod | grep cp210x`
- If needed, run: `sudo modprobe cp210x`
- Add user to dialout group: `sudo usermod -a -G dialout $USER`
- Log out and log back in

#### Installing CH340 Drivers:

**Windows:**
1. Download from: http://www.wch-ic.com/downloads/CH341SER_EXE.html
2. Or use this direct link: http://www.wch.cn/downloads/CH341SER_EXE.html
3. Extract the ZIP file
4. Run `CH341SER.EXE`
5. Click "Install" button
6. Wait for "Driver installed successfully" message
7. Restart your computer
8. Reconnect ESP32

**Alternative Windows Method:**
1. Download from SparkFun: https://learn.sparkfun.com/tutorials/how-to-install-ch340-drivers/all
2. Follow their detailed guide

**macOS:**
1. Download from: https://github.com/adrianmihalko/ch340g-ch34g-ch34x-mac-os-x-driver
2. Download the latest `.pkg` file
3. Open the .pkg file
4. Follow installation instructions
5. **Important for macOS Catalina and later:**
   - Go to System Preferences → Security & Privacy
   - Click the lock to make changes
   - Click "Allow" next to the blocked extension
6. Restart your Mac
7. Reconnect ESP32

**Linux:**
1. CH340 drivers are usually built into the kernel (kernel 2.6.24 or later)
2. Check with: `lsmod | grep ch34x`
3. If not loaded: `sudo modprobe ch341`
4. Add user to dialout group: `sudo usermod -a -G dialout $USER`
5. Log out and log back in

#### Verifying Driver Installation:

**Windows:**
1. Open Device Manager (`Win + X` → Device Manager)
2. Expand "Ports (COM & LPT)"
3. Look for "Silicon Labs CP210x USB to UART Bridge (COM#)" or "USB-SERIAL CH340 (COM#)"
4. Note the COM port number (e.g., COM3, COM4)

**macOS:**
1. Open Terminal
2. Type: `ls /dev/cu.*`
3. Look for `/dev/cu.usbserial-####` (CP2102) or `/dev/cu.wchusbserial####` (CH340)

**Linux:**
1. Open Terminal
2. Type: `ls /dev/ttyUSB*` or `dmesg | grep tty`
3. Look for `/dev/ttyUSB0` or similar

#### Configuring Arduino IDE:

After driver installation:
1. Restart Arduino IDE
2. Go to **Tools → Port**
3. Select the COM port (Windows) or /dev/cu.* (macOS) or /dev/ttyUSB* (Linux)
4. If port doesn't appear:
   - Try a different USB cable (some cables are power-only)
   - Try a different USB port
   - Restart computer

#### Common Driver Issues:

**Issue: "Port is not available" or "Access Denied"**
- **Windows**: Close any Serial Monitor or other programs using the port
- **macOS/Linux**: Grant permissions with `sudo chmod 666 /dev/ttyUSB0` (adjust port name)
- **Linux**: Add user to dialout group (see Linux instructions above)

**Issue: Driver won't install on macOS**
- Disable System Integrity Protection (SIP) temporarily
- Go to Recovery Mode (Restart + Cmd+R)
- Open Terminal in Recovery Mode
- Run: `csrutil disable`
- Restart and install driver
- Re-enable SIP: Boot to Recovery, run `csrutil enable`

**Issue: Multiple drivers causing conflicts**
- Uninstall all USB-serial drivers
- Restart computer
- Install only the driver you need

**Issue: "Device not recognized" on Windows**
- Try different USB ports (USB 2.0 ports work better than 3.0 sometimes)
- Update Windows USB drivers
- Check USB cable (use data cable, not charging-only cable)

---

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

