+++
author = "Luan Pham"
title = "ESP32 Temperature and Moisture Monitoring with MQTT"
date = "2026-09-10"
description = "Build an ESP32 monitoring device that reads temperature, air humidity, and soil moisture, then publishes telemetry over MQTT."
tags = [
    "ESP32",
    "MQTT",
    "IoT",
    "Arduino",
]
categories = [
    "Technology",
]
thumbnail = "images/post_content/esp32_temperature_mqtt.png"
featureImage = "images/post_content/esp32_temperature_mqtt.png"
featureImageAlt = "ESP32 temperature and moisture monitoring with MQTT"
featureImageCap = "An ESP32 publishes temperature, humidity, and soil moisture telemetry over MQTT."
draft = false
+++

An ESP32, a few sensors, and MQTT are enough to build a useful monitoring device for a home, greenhouse, or small IoT system. In this project, the ESP32 reads temperature, air humidity, and soil moisture, then publishes the measurements to an MQTT broker over Wi-Fi.

The same architecture can later be extended with dashboards, alerts, data storage, or automatic irrigation.

## What we will build

The device will:

- Connect to a Wi-Fi network
- Read temperature and air humidity from a DHT22 sensor
- Read soil moisture from an analog capacitive sensor
- Publish measurements as a JSON message over MQTT
- Reconnect automatically when Wi-Fi or MQTT is temporarily unavailable

The data will be published to:

```text
home/garden/esp32-01/telemetry
```

An example payload is:

```json
{
  "temperature": 27.4,
  "humidity": 68.2,
  "soil_moisture": 54
}
```

## Hardware

You need:

- ESP32 development board
- DHT22 or AM2302 temperature and humidity sensor
- Capacitive soil moisture sensor with an analog output
- 10 kOhm resistor if your DHT22 module does not include a pull-up resistor
- Jumper wires and a breadboard

### Example connections

| Component | ESP32 |
| --- | --- |
| DHT22 VCC | 3.3V |
| DHT22 GND | GND |
| DHT22 DATA | GPIO 4 |
| Soil sensor VCC | 3.3V |
| Soil sensor GND | GND |
| Soil sensor AO | GPIO 34 |

GPIO 34 is input-only, which makes it a suitable pin for an analog sensor on many ESP32 boards. Check the pinout of your specific board before wiring it.

## MQTT architecture

MQTT uses a publish/subscribe model:

1. The ESP32 acts as an MQTT publisher.
2. The broker receives the telemetry.
3. A dashboard, automation service, or another device subscribes to the topic.

This keeps the sensor firmware independent from the application consuming the data. The ESP32 does not need to know whether the data is displayed in Node-RED, Home Assistant, or a custom backend.

For local development, you can use Mosquitto as the broker:

```bash
mosquitto_sub -h localhost -t 'home/garden/esp32-01/telemetry' -v
```

## Arduino libraries

Install these libraries from the Arduino IDE Library Manager:

- DHT sensor library
- Adafruit Unified Sensor
- PubSubClient

The ESP32 Wi-Fi library is included with the ESP32 Arduino core.

## ESP32 example code

The following sketch reads the sensors every 30 seconds and publishes a JSON payload:

```cpp
#include <ArduinoJson.h>
#include <DHT.h>
#include <PubSubClient.h>
#include <WiFi.h>

const char *WIFI_SSID = "your-wifi-ssid";
const char *WIFI_PASSWORD = "your-wifi-password";
const char *MQTT_HOST = "192.168.1.100";
const int MQTT_PORT = 1883;
const char *MQTT_TOPIC = "home/garden/esp32-01/telemetry";

constexpr uint8_t DHT_PIN = 4;
constexpr uint8_t DHT_TYPE = DHT22;
constexpr uint8_t SOIL_MOISTURE_PIN = 34;
constexpr unsigned long PUBLISH_INTERVAL_MS = 30000;

DHT dht(DHT_PIN, DHT_TYPE);
WiFiClient wifiClient;
PubSubClient mqttClient(wifiClient);
unsigned long lastPublish = 0;

void connectWiFi() {
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }
}

void connectMQTT() {
  while (!mqttClient.connected()) {
    String clientId = "esp32-" + String((uint32_t)ESP.getEfuseMac(), HEX);
    if (!mqttClient.connect(clientId.c_str())) {
      delay(2000);
    }
  }
}

void publishTelemetry() {
  const float temperature = dht.readTemperature();
  const float humidity = dht.readHumidity();
  const int soilRaw = analogRead(SOIL_MOISTURE_PIN);

  if (isnan(temperature) || isnan(humidity)) {
    return;
  }

  // Calibrate these limits for the specific soil sensor and application.
  const int soilMoisture = constrain(map(soilRaw, 3000, 1200, 0, 100), 0, 100);

  StaticJsonDocument<192> document;
  document["temperature"] = round(temperature * 10) / 10.0;
  document["humidity"] = round(humidity * 10) / 10.0;
  document["soil_moisture"] = soilMoisture;

  char payload[192];
  serializeJson(document, payload, sizeof(payload));
  mqttClient.publish(MQTT_TOPIC, payload);
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  connectWiFi();
  mqttClient.setServer(MQTT_HOST, MQTT_PORT);
}

void loop() {
  if (WiFi.status() != WL_CONNECTED) {
    connectWiFi();
  }

  if (!mqttClient.connected()) {
    connectMQTT();
  }
  mqttClient.loop();

  if (millis() - lastPublish >= PUBLISH_INTERVAL_MS) {
    lastPublish = millis();
    publishTelemetry();
  }
}
```

The example uses `ArduinoJson`, so install that library as well. Replace the Wi-Fi credentials and broker address before uploading the sketch.

## Calibrating the soil moisture sensor

Analog soil sensors do not produce a universal percentage. The raw value depends on the sensor, soil type, supply voltage, and how deeply the probe is inserted.

Record two values:

1. The reading with the sensor in dry soil.
2. The reading with the sensor in wet soil.

Then replace `3000` and `1200` in the `map()` call with the values measured by your sensor. The `constrain()` call keeps the published percentage between 0 and 100.

## Testing the data

Subscribe to the MQTT topic and watch the messages:

```bash
mosquitto_sub -h 192.168.1.100 \
  -t 'home/garden/esp32-01/telemetry' \
  -v
```

You should receive a new JSON message every 30 seconds. If no message appears, check the ESP32 serial monitor, Wi-Fi address, broker address, and firewall rules.

## Important reliability improvements

This example is intentionally small, but a production device should also consider:

- MQTT username and password
- TLS encryption when data crosses an untrusted network
- Last-will and retained messages
- Sensor read error reporting
- Non-blocking reconnect logic
- Deep sleep if the device is battery powered
- A unique MQTT client ID for every device

Do not expose an unauthenticated MQTT broker directly to the public internet.

## Conclusion

This project demonstrates a useful IoT pattern: the ESP32 handles sensing, MQTT handles transport, and another service handles visualization or automation.

The design is simple enough for a first project but flexible enough to grow. You can add more sensors, publish separate topics, subscribe to control commands, or trigger an irrigation pump when soil moisture falls below a threshold.

The most important steps are to calibrate the sensor, handle reconnects, and secure the MQTT connection before deploying the device in a real environment.
