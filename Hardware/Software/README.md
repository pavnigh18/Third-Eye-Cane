# Software 
Arduino and ESp source code.
#include <SoftwareSerial.h>
#include <TinyGPS++.h>

// Pin definitions for HC-SR04 Ultrasonic Sensor
const int trigPin = 9;
const int echoPin = 8;

// Pin definitions for 3-Pin Buzzer
const int buzzerPin = 5;
const int buzzerPowerPin = 7; // Powers the buzzer module

// Pin definitions for Moisture Sensor
const int moisturePin = A0;
const int moisturePowerPin = 6; // Powers the moisture sensor

// GPS Module Pins (Software Serial)
const int RXPin = 4; 
const int TXPin = 3; 
const uint32_t GPSBaud = 9600;

// Objects
TinyGPSPlus gps;
SoftwareSerial ss(RXPin, TXPin);

// Variables
int distanceCm;
int moistureValue;

void setup() {
  Serial.begin(9600);
  ss.begin(GPSBaud);

  // Set up Ultrasonic pins
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  
  // Set up Buzzer pins
  pinMode(buzzerPin, OUTPUT);
  pinMode(buzzerPowerPin, OUTPUT);
  digitalWrite(buzzerPowerPin, HIGH); // Turn on power to buzzer

  // Set up Moisture Sensor power pin
  pinMode(moisturePowerPin, OUTPUT);
  digitalWrite(moisturePowerPin, HIGH); // Turn on power to moisture sensor

  Serial.println("All Systems Active (Direct Setup without Breadboard)...");
}

void loop() {
  // --- 1. READ GPS MODULE ---
  ss.listen();
  unsigned long start = millis();
  while (millis() - start < 60) { 
    while (ss.available() > 0) {
      gps.encode(ss.read());
    }
  }

  // --- 2. READ ULTRASONIC SENSOR ---
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  long duration = pulseIn(echoPin, HIGH, 20000); 
  distanceCm = duration * 0.034 / 2;

  // --- 3. READ MOISTURE SENSOR ---
  moistureValue = analogRead(moisturePin);

  // --- 4. PRINT DATA TO SERIAL MONITOR ---
  printData();

  // --- 5. BUZZER LOGIC (80 cm Threshold) ---
  if (distanceCm >= 3 && distanceCm <= 80) {
    int pitch = map(distanceCm, 3, 80, 1500, 300);
    pitch = constrain(pitch, 300, 1500); 
    tone(buzzerPin, pitch); 
  } else {
    noTone(buzzerPin); 
  }

  delay(40); 
}

void printData() {
  Serial.print("Dist: "); Serial.print(distanceCm); Serial.print("cm | ");
  Serial.print("Moist: "); Serial.print(moistureValue); Serial.print(" | ");
  Serial.print("GPS: ");
  if (gps.location.isValid()) {
    Serial.print(gps.location.lat(), 6);
    Serial.print(",");
    Serial.println(gps.location.lng(), 6);
  } else {
    Serial.println("Waiting for Lock...");
  }
}
