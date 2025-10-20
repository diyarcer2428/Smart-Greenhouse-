# Smart-Greenhouse-
An IoT based system. This project details the development of an automated, Smart Greenhouse System designed to monitor and regulate environmental conditions crucial for optimal plant growth. Using a NodeMCU (ESP8266/ESP32) microcontroller, the system provides real-time data and autonomous control over critical factors like temperature,moisture. 
# Smart Greenhouse Monitoring and Control System

## Description
An IoT-based smart greenhouse system using an ESP8266 NodeMCU microcontroller. The system integrates a DHT22 temperature and humidity sensor, a soil moisture sensor, a relay to control a fan and water pump, a buzzer for alerts, and an LCD for real-time monitoring. It automates irrigation and climate control to optimize plant growth by continuously monitoring environmental conditions. Sensor data is sent to the ThingSpeak cloud platform for remote monitoring and analytics.

## Components
- ESP8266 NodeMCU
- DHT22 Temperature and Humidity Sensor
- Soil Moisture Sensor
- Relay Module (Fan and Water Pump Control)
- Buzzer (Alert System)
- LCD Display (I2C)
- Fan and Water Pump

## Wiring Details
- *DHT22:* VCC to 3V, Data to D2, GND to G
- *Soil Moisture Sensor:* VCC to 3V, AO to A0, GND to G
- *Relay:* VCC to Vin, IN to D5, GND to G
- *Buzzer:* Positive to D7, Negative to G
- *Fan and Pump:* Negative to G, Positive to NO on relay
- *LCD:* VCC to 3.3V, GND to G, SDA to D2, SCL to D1
- 
## 🔌 Wiring Connections

| Component | NodeMCU Pin | Description |
|------------|--------------|-------------|
| DHT11 Sensor | D2 | Data pin |
| Soil Moisture Sensor | A0 | Analog output |
| Buzzer | D5 | Positive terminal |
| Common Ground | G | All GND connections |

## Features
- Real-time temperature, humidity, and soil moisture monitoring
- Automatic switching of fan and pump based on sensor readings
- Audible alert on sensor thresholds breach
- Data display on LCD for easy status visualization
- Cloud data logging and remote monitoring via ThingSpeak IoT platform

## Usage Instructions 
1. Open the code below in **Arduino IDE**  
2. Install required libraries:
   - `ESP8266WiFi.h`
   - `ThingSpeak.h`
   - `DHT.h`
3. Update the WiFi credentials (`ssid`, `password`) and ThingSpeak details (`channelID`, `writeAPIKey`)  
4. Connect components according to the wiring table  
5. Upload the code to your NodeMCU board  
6. Open Serial Monitor and observe live data  
7. Check data remotely on your **ThingSpeak dashboard**

  ## 💻 Arduino Code



```cpp
#include <ESP8266WiFi.h>
#include <ThingSpeak.h>
#include "DHT.h"

// ---------- PIN DEFINITIONS ----------
#define DHTPIN D2           // DHT11 Data Pin
#define DHTTYPE DHT11
#define SOILPIN A0          // Soil Moisture Analog Pin
#define BUZZER D5           // Buzzer Pin

// ---------- WiFi & ThingSpeak ----------
const char* ssid = ""; 
const char* password = ""; 

unsigned long channelID = 3111183;            // ThingSpeak channel ID
const char* writeAPIKey = "56M44PYUELKEI77T"; // ThingSpeak read API key

WiFiClient client;
DHT dht(DHTPIN, DHTTYPE);

unsigned long lastDHTRead = 0;
unsigned long lastThingSpeakUpdate = 0;

void setup() {
  Serial.begin(115200);
  dht.begin();
  pinMode(BUZZER, OUTPUT);

  Serial.println("🌱 Smart Greenhouse System Starting...");

  // Connect to WiFi
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ WiFi Connected!");
  ThingSpeak.begin(client);
}

void loop() {
  bool buzzerState = LOW;  // Default buzzer OFF

  // ----- SOIL MOISTURE SENSOR -----
  int soilValue = analogRead(SOILPIN);
  Serial.print("🌱 Soil Moisture Value: ");
  Serial.println(soilValue);

  if (soilValue > 800) {
    Serial.println("⚠️ Soil is very dry!");
    buzzerState = HIGH;
  } else if (soilValue > 500) {
    Serial.println("🙂 Soil is moist.");
  } else {
    Serial.println("💧 Soil is wet.");
  }

  // ----- DHT11 SENSOR every 3 sec -----
  if (millis() - lastDHTRead > 3000) {
    lastDHTRead = millis();
    float h = dht.readHumidity();
    float t = dht.readTemperature();

    if (isnan(h) || isnan(t)) {
      Serial.println("❌ Failed to read from DHT sensor!");
    } else {
      Serial.print("🌡 Temperature: ");
      Serial.print(t);
      Serial.print("°C, Humidity: ");
      Serial.print(h);
      Serial.println("%");

      // ----- TEMPERATURE ALERT -----
      if (t > 32) {
        Serial.println("🔥 High Temperature Alert!");
        buzzerState = HIGH;
      } else if (t < 18) {
        Serial.println("❄️ Low Temperature Alert!");
        buzzerState = HIGH;
      }

      // ----- HUMIDITY ALERT -----
      if (h < 40) {
        Serial.println("💨 Low Humidity Alert!");
        buzzerState = HIGH;
      } else if (h > 100) {
        Serial.println("💦 High Humidity Alert!");
        buzzerState = HIGH;
      }
    }
    Serial.println("-----------------------------");
  }

  // ----- Set Buzzer State -----
  digitalWrite(BUZZER, buzzerState);

  // ----- ThingSpeak update every 20 sec -----
  if (millis() - lastThingSpeakUpdate > 20000) {
    lastThingSpeakUpdate = millis();
    if (WiFi.status() == WL_CONNECTED) {
      ThingSpeak.setField(1, dht.readTemperature());
      ThingSpeak.setField(2, dht.readHumidity());
      ThingSpeak.setField(3, soilValue);
      int response = ThingSpeak.writeFields(channelID, writeAPIKey);
      if (response == 200) {
        Serial.println("✅ ThingSpeak Updated");
      } else {
        Serial.print("❌ ThingSpeak update failed, code: ");
        Serial.println(response);
      }
    } else {
      Serial.println("❌ WiFi not connected, cannot update ThingSpeak");
    }
  }

  delay(1000); // Sensor updates every second
}

## Author
Diya Risa Chacko 
