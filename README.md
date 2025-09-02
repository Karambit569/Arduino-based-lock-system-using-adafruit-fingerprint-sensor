#readme

This project is an **Arduino-based  lock system** using the [Adafruit Fingerprint Sensor].  
It allows you to **enroll, delete, and authenticate fingerprints** to control a relay (e.g., door lock).

### This project is made for only test/education purpose. Not recommended for actual usage. Memory/buffer problem may occur.

---

##  Features
- Enroll and store up to 127 fingerprint templates
- Delete fingerprint by ID
- Authenticate fingerprints and trigger a relay (lock/unlock)
- Manual serial commands (`enroll`, `delete`, `open`)
- Lock status monitoring via input pin

---

##  Hardware Requirements
- Arduino
- [Adafruit Fingerprint Sensor]
- Relay module 
- Jumper wires
- Power supply

### Example Wiring (can be altered)
| Component              | Arduino Pin |
|-------------------------|-------------|
| Fingerprint TX         | D4 |
| Fingerprint RX         | D5 |
| Relay Signal           | D3 |
| Input Switch           | D6 |
| Check Output           | D7 |

>  Pin numbers can be changed in the code.


##  Software Requirements
- [Arduino IDE]
- Adafruit Fingerprint Library  
  Install via Arduino Library Manager:  
  `Sketch > Include Library > Manage Libraries... > Search "Adafruit Fingerprint"`

The default baud rate for Adafruit Fingerprint Sensor is usually 57600,
but the code currently uses 192200. If your sensor fails, adjust this line:

finger.begin(57600);

If buffer overrun occurs, change the sendData function:

void sendData(String data) {
    char buffer[100];
    data.toCharArray(buffer, sizeof(buffer));
    Serial.write(buffer);
}

