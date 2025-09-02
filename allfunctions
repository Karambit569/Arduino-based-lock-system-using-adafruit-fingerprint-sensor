#include <Arduino.h>
#include <Adafruit_Fingerprint.h>
#include <string.h>
#include <SoftwareSerial.h>

#if (defined(__AVR__) || defined(ESP8266)) && !defined(__AVR_ATmega2560__)
// Use SoftwareSerial if the board is not ATmega2560
SoftwareSerial mySerial(4, 5); // RX, TX pins for the fingerprint sensor
#else
// On boards with multiple hardware serial ports (Leonardo, Mega, etc.)
// Use hardware Serial1
#define mySerial Serial1
#endif

// Pin assignments (change according to your setup)
int Relaypin = 3;     // Relay control pin for lock/unlock
int Checkpin = 7;     // Status output pin
int Inputpin = 6;     // Input pin to check lock state

char Incoming;        // Temporary storage for incoming serial characters
String IncomingString = "";  // Buffer for incoming serial commands

Adafruit_Fingerprint finger = Adafruit_Fingerprint(&mySerial); // Fingerprint sensor object

uint8_t id; // ID for fingerprint templates

void setup() {
  pinMode(Relaypin, OUTPUT);
  pinMode(Checkpin, OUTPUT); 
  pinMode(Inputpin, INPUT);
  
  Serial.begin(9600);
  while (!Serial);  // Wait for Serial to initialize (for boards like Leonardo)
  delay(100);
  Serial.println("\n\nAdafruit finger detect test");

  // Initialize fingerprint sensor at 192200 baud
  finger.begin(192200);
  delay(5);

  // Check if fingerprint sensor is connected
  if (finger.verifyPassword()) {
    Serial.println("Found fingerprint sensor!");
  } else {
    Serial.println("Did not find fingerprint sensor :(");
    while (1) { delay(1); } // Stop execution if no sensor is found
  }

  // Print sensor parameters
  Serial.println(F("Reading sensor parameters"));
  finger.getParameters();
  Serial.print(F("Status: 0x")); Serial.println(finger.status_reg, HEX);
  Serial.print(F("Sys ID: 0x")); Serial.println(finger.system_id, HEX);
  Serial.print(F("Capacity: ")); Serial.println(finger.capacity);
  Serial.print(F("Security level: ")); Serial.println(finger.security_level);
  Serial.print(F("Device address: ")); Serial.println(finger.device_addr, HEX);
  Serial.print(F("Packet len: ")); Serial.println(finger.packet_len);
  Serial.print(F("Baud rate: ")); Serial.println(finger.baud_rate);

  finger.getTemplateCount();

  if (finger.templateCount == 0) {
    Serial.print("Sensor doesn't contain any fingerprint data. Please run the 'enroll' example.");
  }
  else {
    Serial.println("Waiting for valid finger...");
    Serial.print("Sensor contains "); Serial.print(finger.templateCount); Serial.println(" templates");
  }
}

// Helper function to read a number from Serial
uint8_t readnumber(void) {
  uint8_t num = 0;
  while (num == 0) {
    while (! Serial.available());
    num = Serial.parseInt();
  }
  return num;
}

// Function to enroll a new fingerprint
uint8_t getFingerprintEnroll() {
  int p = -1;
  Serial.print("Waiting for valid finger to enroll as #"); Serial.println(id);

  // Wait for a finger to be placed
  while (p != FINGERPRINT_OK) {
    p = finger.getImage();
    switch (p) {
      case FINGERPRINT_OK: Serial.println("Image taken"); break;
      case FINGERPRINT_NOFINGER: Serial.print("."); break;
      case FINGERPRINT_PACKETRECIEVEERR: Serial.println("Communication error"); break;
      case FINGERPRINT_IMAGEFAIL: Serial.println("Imaging error"); break;
      default: Serial.println("Unknown error"); break;
    }
  }

  // Convert image to template (first pass)
  p = finger.image2Tz(1);
  if (p != FINGERPRINT_OK) return p;

  Serial.println("Remove finger");
  delay(2000);
  
  // Wait until finger is removed
  while (p != FINGERPRINT_NOFINGER) {
    p = finger.getImage();
  }

  Serial.println("Place same finger again");
  p = -1;
  
  // Wait for finger to be placed again
  while (p != FINGERPRINT_OK) {
    p = finger.getImage();
    switch (p) {
      case FINGERPRINT_OK: Serial.println("Image taken"); break;
      case FINGERPRINT_NOFINGER: Serial.print("."); break;
      case FINGERPRINT_PACKETRECIEVEERR: Serial.println("Communication error"); break;
      case FINGERPRINT_IMAGEFAIL: Serial.println("Imaging error"); break;
      default: Serial.println("Unknown error"); break;
    }
  }

  // Convert image to template (second pass)
  p = finger.image2Tz(2);
  if (p != FINGERPRINT_OK) return p;

  // Create fingerprint model by combining both images
  p = finger.createModel();
  if (p != FINGERPRINT_OK) return p;

  // Store the new fingerprint template
  p = finger.storeModel(id);
  if (p != FINGERPRINT_OK) return p;

  delay(1000);
  return true;
}

// Function to get fingerprint ID (detailed check)
uint8_t getFingerprintID() {
  uint8_t p = finger.getImage();
  if (p != FINGERPRINT_OK) return p;

  p = finger.image2Tz();
  if (p != FINGERPRINT_OK) return p;

  p = finger.fingerSearch();
  if (p != FINGERPRINT_OK) return p;

  // Fingerprint matched
  Serial.print("Found ID #"); Serial.print(finger.fingerID);
  Serial.print(" with confidence of "); Serial.println(finger.confidence);

  // Unlock relay
  digitalWrite(Relaypin, HIGH);
  Serial.print("UNLOCKED\n");  
  delay(5000);
  Serial.print("LOCKED\n");
  digitalWrite(Relaypin, LOW);

  return finger.fingerID;
}

// Simplified fingerprint check function
int getFingerprintIDez() {
  uint8_t p = finger.getImage();
  if (p != FINGERPRINT_OK) return -1;

  p = finger.image2Tz();
  if (p != FINGERPRINT_OK) return -1;

  p = finger.fingerFastSearch();
  if (p != FINGERPRINT_OK) return -1;

  // Fingerprint matched
  Serial.print("Found ID #"); Serial.print(finger.fingerID);
  Serial.print(" with confidence of "); Serial.println(finger.confidence);

  return finger.fingerID;
}

// Delete a fingerprint template by ID
uint8_t deleteFingerprint(uint8_t id) {
  uint8_t p = finger.deleteModel(id);
  if (p == FINGERPRINT_OK) {
    Serial.println("Deleted!");
  } else {
    Serial.print("Error: 0x"); Serial.println(p, HEX);
  }
  return p;
}

// Send data over Serial
void sendData(String data) {
  char *buffer;
  data.toCharArray(buffer, 100);
  Serial.write(buffer);
}

void loop() {
  // Check if a serial command is received
  if (Serial.available()>0) {
    Incoming = Serial.read();
    IncomingString += Incoming;

    // Enroll new fingerprint
    if (IncomingString.endsWith("enroll")) {
      Serial.println("Enroll command detected");
      id = readnumber();
      if (id != 0) {
        while (!getFingerprintEnroll());
      }
    }

    // Delete fingerprint
    if (IncomingString.endsWith("delete")) {
      uint8_t id = readnumber();
      if (id != 0) {
        deleteFingerprint(id);
      }
    }

    // Manually open relay
    if (IncomingString.endsWith("open")) {
      digitalWrite(Relaypin, HIGH);
      Serial.print("UNLOCKED\n");  
      delay(5000);
    }
  }
  else {
    // Continuously check fingerprint sensor
    getFingerprintID();
    delay(50);
  }

  // Monitor input pin for lock state
  if (digitalRead(Inputpin)==HIGH) {
    Serial.print("Locked\n");
    sendData("Locked\n");
    delay(50);
  } else if (digitalRead(Inputpin)==LOW) {
    Serial.print("Unlocked\n");
    sendData("Unlocked\n");
    delay(50);
  }

  // Check for remote "open" command
  if (Serial.available()) {
    String in_from_server = Serial.readStringUntil('\n');
    Serial.read();
    if(in_from_server.equals("open")){
      digitalWrite(Relaypin,HIGH);
      Serial.print("UNLOCKED\n");  
      delay(5000);
    }
  }
}
