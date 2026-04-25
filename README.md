# Arduino-parking-system-in-ASTU
ASTU Smart Parking System  . An Arduino-based gate controller for parking buildings. Uses an MFRC522 RFID scanner to verify UIDs and a Servo motor for the barrier.  Features:  Access Control: Only authorized IDs open the gate.  Capacity Logic: Tracks car count; denies entry when full.  Simulation: Built tested in Wokwi.
Built by Eyob.

#include <SPI.h>
#include <MFRC522.h>
#include <Servo.h>

// Pin Definitions for MFRC522
#define SS_PIN 10
#define RST_PIN 9

// Pin Definition for Servo
#define SERVO_PIN 3

MFRC522 rfid(SS_PIN, RST_PIN);
Servo gateServo;

// --- CONFIGURATION ---
// In Wokwi, the default tag often has UID: "DE AD BE EF" or similar.
// Run the simulation, scan a card, and update this string with what you see!
String allowedID = "DE AD BE EF"; 

int carCount = 0;
const int MAX_CAPACITY = 5;

void setup() {
  Serial.begin(9600);
  SPI.begin();           // Start SPI Communication
  rfid.PCD_Init();       // Start RFID Module
  
  gateServo.attach(SERVO_PIN);
  gateServo.write(0);    // Ensure gate is closed on startup
  
  Serial.println("--- ASTU Parking System Ready ---");
  Serial.print("Current Capacity: ");
  Serial.print(carCount);
  Serial.println("/");
  Serial.println(MAX_CAPACITY);
  Serial.println("Scan your ID card at the entrance...");
}

void loop() {
  // Check if a new card is present
  if (!rfid.PICC_IsNewCardPresent() || !rfid.PICC_ReadCardSerial()) {
    return;
  }

  // Read the UID from the scanned card
  String scannedID = "";
  for (byte i = 0; i < rfid.uid.size; i++) {
    scannedID.concat(String(rfid.uid.uidByte[i] < 0x10 ? " 0" : " "));
    scannedID.concat(String(rfid.uid.uidByte[i], HEX));
  }
  scannedID.toUpperCase();
  scannedID = scannedID.substring(1); // Remove the leading space

  Serial.print("ID Scanned: [");
  Serial.print(scannedID);
  Serial.println("]");

  // Access Control Logic
  if (scannedID == allowedID) {
    if (carCount < MAX_CAPACITY) {
      Serial.println("ACCESS GRANTED: Welcome!");
      carCount++;
      openGate();
      displayStatus();
    } else {
      Serial.println("ACCESS DENIED: Parking Lot Full!");
    }
  } else {
    Serial.println("ACCESS DENIED: Unknown ID Card.");
  }

  // Halt the RFID card to prevent multiple reads
  rfid.PICC_HaltA();
  rfid.PCD_StopCrypto1();
}

void openGate() {
  Serial.println("Opening Gate...");
  gateServo.write(90);  // Rotate to 90 degrees
  delay(3000);          // Wait 3 seconds for car to pass
  Serial.println("Closing Gate...");
  gateServo.write(0);   // Return to 0 degrees
}

void displayStatus() {
  Serial.println("---------------------------------");
  Serial.print("Cars Inside: ");
  Serial.println(carCount);
  Serial.print("Available Spots: ");
  Serial.println(MAX_CAPACITY - carCount);
  Serial.println("---------------------------------");
}
