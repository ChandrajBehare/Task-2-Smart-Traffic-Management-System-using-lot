# Task-2-Smart-Traffic-Management-System-using-lot
Project Summary: Addressing the escalating challenges of road traffic, the IoT-based Smart Traffic Management System emerges as a comprehensive solution.

Below is a example of source code for a smart traffic management system using an IoT platform like Arduino.

#include <WiFi.h>
#include <PubSubClient.h>

const char* ssid = "YourWiFiSSID";
const char* password = "YourWiFiPassword";
const char* mqtt_server = "YourMQTTBrokerIPAddress";

const int RED_PIN = 13; // Red light pin
const int YELLOW_PIN = 12; // Yellow light pin
const int GREEN_PIN = 14; // Green light pin
const int EMERGENCY_SENSOR_PIN = 5; // Emergency vehicle detection sensor pin
const int VIOLATOR_SENSOR_PIN = 4; // Traffic violator detection sensor pin

WiFiClient espClient;
PubSubClient client(espClient);

void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  
  String message = "";
  for (int i = 0; i < length; i++) {
    message += (char)payload[i];
  }
  Serial.println(message);

  // Add functionality based on MQTT messages received
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP32Client")) {
      Serial.println("connected");
      // Subscribe to MQTT topics here
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  pinMode(RED_PIN, OUTPUT);
  pinMode(YELLOW_PIN, OUTPUT);
  pinMode(GREEN_PIN, OUTPUT);
  pinMode(EMERGENCY_SENSOR_PIN, INPUT);
  pinMode(VIOLATOR_SENSOR_PIN, INPUT);

  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Check for emergency vehicles
  int emergencyStatus = digitalRead(EMERGENCY_SENSOR_PIN);
  if (emergencyStatus == HIGH) {
    // Trigger emergency protocol - turn all lights red except for emergency vehicle pathway
    digitalWrite(RED_PIN, HIGH);
    digitalWrite(YELLOW_PIN, LOW);
    digitalWrite(GREEN_PIN, LOW);
    // Publish MQTT message to alert traffic management system
    client.publish("traffic/emergency", "Emergency vehicle detected! Activate emergency protocol.");
  } else {
    // Normal traffic flow
    digitalWrite(RED_PIN, LOW);
    digitalWrite(YELLOW_PIN, LOW);
    digitalWrite(GREEN_PIN, HIGH);

    // Check for traffic violators
    int violatorStatus = digitalRead(VIOLATOR_SENSOR_PIN);
    if (violatorStatus == HIGH) {
      // Publish MQTT message to alert traffic management system
      client.publish("traffic/violator", "Traffic violator detected!");
    }
  }

  delay(1000); // Adjust delay according to your requirements
}


This code sets up an ESP32-based microcontroller to connect to your Wi-Fi network and control traffic lights based on inputs from sensors (emergency vehicle detection sensor and traffic violator detection sensor). It also publishes MQTT messages to alert the traffic management system about emergency situations and traffic violators.
