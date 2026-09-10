+++
author = "Luan Pham"
title = "Giám sát nhiệt độ và độ ẩm bằng ESP32 với MQTT"
date = "2026-09-10"
description = "Xây dựng thiết bị ESP32 đọc nhiệt độ, độ ẩm không khí và độ ẩm đất, sau đó gửi dữ liệu qua MQTT."
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
featureImageAlt = "Giám sát nhiệt độ và độ ẩm bằng ESP32 với MQTT"
featureImageCap = "ESP32 gửi dữ liệu nhiệt độ, độ ẩm không khí và độ ẩm đất qua MQTT."
draft = false
+++

Chỉ với một ESP32, vài cảm biến và MQTT, bạn đã có thể xây dựng một thiết bị giám sát hữu ích cho ngôi nhà, nhà kính hoặc hệ thống IoT nhỏ. Trong dự án này, ESP32 đọc nhiệt độ, độ ẩm không khí và độ ẩm đất, sau đó gửi các phép đo đến MQTT broker thông qua Wi-Fi.

Kiến trúc này có thể mở rộng thêm dashboard, cảnh báo, lưu trữ dữ liệu hoặc tưới cây tự động.

## Chúng ta sẽ xây dựng gì?

Thiết bị sẽ:

- Kết nối vào mạng Wi-Fi
- Đọc nhiệt độ và độ ẩm không khí từ cảm biến DHT22
- Đọc độ ẩm đất từ cảm biến điện dung analog
- Gửi các phép đo dưới dạng JSON qua MQTT
- Tự động kết nối lại khi Wi-Fi hoặc MQTT tạm thời bị mất

Dữ liệu sẽ được gửi đến topic:

```text
home/garden/esp32-01/telemetry
```

Ví dụ payload:

```json
{
  "temperature": 27.4,
  "humidity": 68.2,
  "soil_moisture": 54
}
```

## Phần cứng

Bạn cần:

- Bo mạch ESP32
- Cảm biến nhiệt độ và độ ẩm DHT22 hoặc AM2302
- Cảm biến độ ẩm đất điện dung có ngõ ra analog
- Điện trở 10 kOhm nếu module DHT22 chưa có điện trở kéo lên
- Dây nối và breadboard

### Kết nối tham khảo

| Linh kiện | ESP32 |
| --- | --- |
| DHT22 VCC | 3.3V |
| DHT22 GND | GND |
| DHT22 DATA | GPIO 4 |
| Cảm biến đất VCC | 3.3V |
| Cảm biến đất GND | GND |
| Cảm biến đất AO | GPIO 34 |

GPIO 34 chỉ dùng làm input, phù hợp cho cảm biến analog trên nhiều bo ESP32. Hãy kiểm tra sơ đồ chân của đúng bo mạch bạn đang sử dụng trước khi đấu nối.

## Kiến trúc MQTT

MQTT sử dụng mô hình publish/subscribe:

1. ESP32 đóng vai trò MQTT publisher.
2. Broker nhận dữ liệu telemetry.
3. Dashboard, dịch vụ tự động hóa hoặc thiết bị khác subscribe vào topic.

Nhờ vậy firmware của cảm biến không phụ thuộc vào ứng dụng sử dụng dữ liệu. ESP32 không cần biết dữ liệu được hiển thị bằng Node-RED, Home Assistant hay backend tự xây dựng.

Khi phát triển ở local, bạn có thể dùng Mosquitto làm broker:

```bash
mosquitto_sub -h localhost -t 'home/garden/esp32-01/telemetry' -v
```

## Thư viện Arduino

Cài các thư viện sau từ Library Manager của Arduino IDE:

- DHT sensor library
- Adafruit Unified Sensor
- PubSubClient
- ArduinoJson

Thư viện Wi-Fi cho ESP32 đã có sẵn trong ESP32 Arduino core.

## Mã nguồn ví dụ cho ESP32

Sketch sau đọc cảm biến mỗi 30 giây và gửi một JSON payload:

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

  // Hãy hiệu chuẩn các giới hạn này theo cảm biến và ứng dụng thực tế.
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

Sketch này sử dụng `ArduinoJson`, vì vậy bạn cũng cần cài thư viện đó. Hãy thay thông tin Wi-Fi và địa chỉ broker trước khi upload chương trình.

## Hiệu chuẩn cảm biến độ ẩm đất

Cảm biến đất analog không có sẵn một giá trị phần trăm dùng chung cho mọi trường hợp. Giá trị raw phụ thuộc vào cảm biến, loại đất, điện áp cấp và độ sâu cắm đầu dò.

Hãy ghi lại hai giá trị:

1. Giá trị khi cảm biến nằm trong đất khô.
2. Giá trị khi cảm biến nằm trong đất ướt.

Sau đó thay `3000` và `1200` trong hàm `map()` bằng giá trị đo được từ cảm biến của bạn. Hàm `constrain()` giữ phần trăm được gửi trong khoảng từ 0 đến 100.

## Kiểm tra dữ liệu

Subscribe vào MQTT topic để xem message:

```bash
mosquitto_sub -h 192.168.1.100 \
  -t 'home/garden/esp32-01/telemetry' \
  -v
```

Bạn sẽ nhận được một JSON message mới sau mỗi 30 giây. Nếu không thấy message, hãy kiểm tra Serial Monitor của ESP32, địa chỉ Wi-Fi, địa chỉ broker và firewall.

## Một số cải tiến về độ tin cậy

Ví dụ này được viết ngắn gọn, nhưng thiết bị thực tế nên cân nhắc thêm:

- Username và password cho MQTT
- Mã hóa TLS khi dữ liệu đi qua mạng không tin cậy
- Last-will và retained message
- Gửi thông báo khi đọc cảm biến lỗi
- Logic reconnect không block
- Deep sleep nếu thiết bị dùng pin
- MQTT client ID riêng cho từng thiết bị

Không nên đưa một MQTT broker không có xác thực trực tiếp lên internet công cộng.

## Kết luận

Dự án này minh họa một mô hình IoT hữu ích: ESP32 phụ trách đọc cảm biến, MQTT phụ trách truyền dữ liệu, còn một dịch vụ khác phụ trách hiển thị hoặc tự động hóa.

Thiết kế đủ đơn giản cho dự án đầu tiên nhưng vẫn có thể mở rộng. Bạn có thể thêm cảm biến, gửi dữ liệu đến các topic riêng, subscribe lệnh điều khiển hoặc bật máy bơm khi độ ẩm đất giảm dưới một ngưỡng.

Ba việc quan trọng nhất là hiệu chuẩn cảm biến, xử lý reconnect và bảo mật kết nối MQTT trước khi triển khai trong môi trường thực tế.
