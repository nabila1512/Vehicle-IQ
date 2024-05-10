#include <Wire.h>
#include <MPU6050.h>
#include <SoftwareSerial.h>
#include <TinyGPS++.h>

#define VIBRATION_PIN 2
#define GSM_RX_PIN 3
#define GSM_TX_PIN 4
#define GPS_RX_PIN 5
#define GPS_TX_PIN 6

MPU6050 mpu;
SoftwareSerial gsmSerial(GSM_RX_PIN, GSM_TX_PIN);
SoftwareSerial gpsSerial(GPS_RX_PIN, GPS_TX_PIN);

TinyGPSPlus gps;

void setup() {
  pinMode(VIBRATION_PIN, INPUT);
  Serial.begin(9600);
  gsmSerial.begin(9600);
  gpsSerial.begin(9600);
  
  Wire.begin(); // Initialize I2C communication
  mpu.initialize();
  mpu.setFullScaleAccelRange(MPU6050_ACCEL_FS_16);
}

void loop() {
  if (digitalRead(VIBRATION_PIN) == HIGH || accidentDetected()) {
    sendSMS("Accident detected! Location: " + getGPSLocation());
    delay(5000); // Delay to prevent multiple alerts due to vibration
  }
}

bool accidentDetected() {
  int16_t ax, ay, az;
  mpu.getAcceleration(&ax, &ay, &az);
  
  if (abs(ax) > 10000 || abs(ay) > 10000 || abs(az) > 10000) {
    return true;
  }
  return false;
}

String getGPSLocation() {
  while (gpsSerial.available() > 0) {
    if (gps.encode(gpsSerial.read())) {
      if (gps.location.isValid()) {
        return String(gps.location.lat(), 6) + ", " + String(gps.location.lng(), 6);
      }
    }
  }
  return "Location not available";
}

void sendSMS(String message) {
  gsmSerial.println("AT+CMGF=1");
  delay(100);
  gsmSerial.println("AT+CMGS=\"+1234567890\""); // Replace with your phone number
  delay(100);
  gsmSerial.println(message);
  delay(100);
  gsmSerial.println((char)26);
  delay(100);
  gsmSerial.println();
}
