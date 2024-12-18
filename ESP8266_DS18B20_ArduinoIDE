#include <Arduino.h>
#if defined(ESP32)
  #include <WiFi.h>
#elif defined(ESP8266)
  #include <ESP8266WiFi.h>
#endif
#include <Firebase_ESP_Client.h>
#include <OneWire.h>
#include <DallasTemperature.h>
#include <U8g2lib.h>

U8G2_SSD1306_128X64_NONAME_F_SW_I2C
u8g2(U8G2_R0, /* clock=*/ 12, /* data=*/ 14, /* reset=*/ U8X8_PIN_NONE); 

// Provide the token generation process info.
#include "addons/TokenHelper.h"
// Provide the RTDB payload printing info and other helper functions.
#include "addons/RTDBHelper.h"

// Insert your network credentials
#define WIFI_SSID "MSWifi"
#define WIFI_PASSWORD "m12345678"

// Insert Firebase project API Key
#define API_KEY "AIzaSyCm-HmgnmGyfeBlXdZy0AZuDN7HMHzmOVY"

// Insert RTDB URL
#define DATABASE_URL "https://esp32-database-7b3e1-default-rtdb.asia-southeast1.firebasedatabase.app/" 

// Define DS18B20 sensor data pin
#define ONE_WIRE_BUS D2 // GPIO pin connected to DS18B20 sensor

// Set up OneWire and DallasTemperature instances
OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

// Define Firebase Data object
FirebaseData fbdo;
FirebaseAuth auth;
FirebaseConfig config;

unsigned long sendDataPrevMillis = 0;
bool signupOK = false;

void setup() {
  u8g2.begin();

  Serial.begin(115200);
  
  // Start the DS18B20 sensor
  sensors.begin();

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(300);
  }
  Serial.println();
  Serial.print("Connected with IP: ");
  Serial.println(WiFi.localIP());
  Serial.println();

  // Assign the API key (required)
  config.api_key = API_KEY;

  // Assign the RTDB URL (required)
  config.database_url = DATABASE_URL;

  // Sign up
  if (Firebase.signUp(&config, &auth, "", "")) {
    Serial.println("Firebase Sign-Up OK");
    signupOK = true;
  } else {
    Serial.printf("Firebase Sign-Up Error: %s\n", config.signer.signupError.message.c_str());
  }

  // Assign the callback function for token status
  config.token_status_callback = tokenStatusCallback; // See addons/TokenHelper.h

  Firebase.begin(&config, &auth);
  Firebase.reconnectWiFi(true);
}

void loop() {
  if (Firebase.ready() && signupOK && (millis() - sendDataPrevMillis > 2000 || sendDataPrevMillis == 0)) {
    sendDataPrevMillis = millis();

    // Request temperatures from DS18B20
    sensors.requestTemperatures();

    // Read temperature from the first sensor (if you have multiple DS18B20s, you can change the index)
    float temperature = sensors.getTempCByIndex(0);

    // Check if reading is valid
    if (temperature == DEVICE_DISCONNECTED_C) {
      Serial.println("Failed to read from DS18B20 sensor!");
      return;
    }
    u8g2.clearBuffer();
    u8g2.setFont(u8g2_font_7x14B_tr);
    u8g2.drawStr(0, 8, "temperature: ");

    // Chuyển đổi giá trị float sang chuỗi
    char tempStr[10]; // Dành không gian cho chuỗi (số chẵn, dấu chấm, và một vài chữ số sau dấu thập phân)
    dtostrf(temperature, 4, 2, tempStr); // Chuyển đổi float thành chuỗi, 4 ký tự rộng, 2 chữ số thập phân

    u8g2.drawStr(0, 20, tempStr); // Hiển thị nhiệt độ
    u8g2.sendBuffer();
    


    Serial.print("Temperature: ");
    Serial.print(temperature);
    Serial.println(" °C");

    // Upload temperature to Firebase
    if (Firebase.RTDB.setFloat(&fbdo, "sensor/temperature", temperature)) {
      Serial.println("Temperature uploaded successfully!");
    } else {
      Serial.println("Failed to upload temperature!");
      Serial.println(fbdo.errorReason());
    }
  }
}
