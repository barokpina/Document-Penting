1. download Aplikasi arduino IDE

```
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

/* ================== KONFIGURASI ================== */

// WiFi
const char* ssid     = "NAMA_WIFI";
const char* password = "PASSWORD_WIFI";

// MQTT
const char* mqtt_server = "broker.hivemq.com"; // boleh ganti
const int   mqtt_port   = 1883;

// GANTI DEVICE ID BIAR UNIK
const char* DEVICE_ID = "citra_relay_01";

// MQTT Topic
String topic_cmd    = String("smartrelay/") + DEVICE_ID + "/cmd";
String topic_status = String("smartrelay/") + DEVICE_ID + "/status";

// Relay
#define RELAY_PIN D1   // GPIO5
#define RELAY_ON  LOW  // ACTIVE LOW
#define RELAY_OFF HIGH

// Waktu
unsigned long wifiCheckInterval = 15000; // 15 detik
unsigned long restartDelay      = 8000;  // relay off time
unsigned long lastWifiCheck     = 0;

/* ================================================= */

WiFiClient espClient;
PubSubClient client(espClient);

/* ================== FUNGSI ================== */

void relayRestart(String source) {
  client.publish(topic_status.c_str(), "MODEM_OFF");
  digitalWrite(RELAY_PIN, RELAY_ON);
  delay(restartDelay);
  digitalWrite(RELAY_PIN, RELAY_OFF);
  client.publish(topic_status.c_str(), "MODEM_ON");
  client.publish(topic_status.c_str(), source.c_str());
}

void callback(char* topic, byte* payload, unsigned int length) {
  String msg = "";
  for (int i = 0; i < length; i++) {
    msg += (char)payload[i];
  }

  msg.trim();

  if (msg == "RESTART") {
    relayRestart("RESTART_BY_MQTT");
  }
}

void reconnectMQTT() {
  while (!client.connected()) {
    if (client.connect(DEVICE_ID)) {
      client.subscribe(topic_cmd.c_str());
      client.publish(topic_status.c_str(), "MQTT_CONNECTED");
    } else {
      delay(5000);
    }
  }
}

void connectWiFi() {
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  unsigned long startAttempt = millis();
  while (WiFi.status() != WL_CONNECTED && millis() - startAttempt < 20000) {
    delay(500);
  }
}

/* ================== SETUP ================== */

void setup() {
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, RELAY_OFF);

  Serial.begin(9600);
  connectWiFi();

  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);

  if (WiFi.status() == WL_CONNECTED) {
    client.publish(topic_status.c_str(), "WIFI_CONNECTED");
  }
}

/* ================== LOOP ================== */

void loop() {
  if (WiFi.status() != WL_CONNECTED) {
    connectWiFi();
  }

  if (!client.connected()) {
    reconnectMQTT();
  }

  client.loop();

  // Cek WiFi periodik
  if (millis() - lastWifiCheck > wifiCheckInterval) {
    lastWifiCheck = millis();

    if (WiFi.status() != WL_CONNECTED) {
      relayRestart("AUTO_RESTART_WIFI_LOST");
    }
  }
}
```
