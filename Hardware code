#include <SPI.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Servo.h>
#include <MFRC522.h> // RFID library
TwoWire Wire(0);
SPIClass SPI(0);

// OLED display configuration
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 32
#define OLED_RESET -1
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

// Pin definitions
#define TRIG_PIN 3          // Ultrasonic trigger pin GPIO
#define ECHO_PIN 4          // Ultrasonic echo pin GPIO
#define SERVO_PIN1 0        // Servo for door 1 on PWM0 of Aries V2
#define SERVO_PIN2 1        // Servo for door 2 on GPIO 5 PMW1
#define RYG_RED 6           // RGB LED Red
#define RYG_YELLOW 7        // RGB LED Yellow (not used for door status)
#define RYG_GREEN 8         // RGB LED Green
#define BUZZER_PIN 9        // Buzzer on GPIO 9

// RFID configuration
#define SS_PIN 10           // Slave Select pin for RFID
#define RST_PIN 2           // Reset pin for RFID
MFRC522 rfid(SS_PIN, RST_PIN); // Initializing MFRC522 object

// Servo motor objects
Servo doorServo1;           // Servo for the first door
Servo doorServo2;           // Servo for the second door

// Valid employee UID in hexadecimal format
byte validUID[4] = {0x83, 0x64, 0x4B, 0x97};  

long duration;
int distance;
bool door1Open = false; // Flag to track the state of the first door

// Function to print UID in hexadecimal format
void printHex(byte *buffer, byte bufferSize) {
  for (byte i = 0; i < bufferSize; i++) {
    Serial.print(buffer[i] < 0x10 ? " 0" : " ");
    Serial.print(buffer[i], HEX);
  }
}

// Checking if UID matches the valid UID
bool checkValidUID(byte *uid, byte size) {
  if (size != 4) return false; 
  for (byte i = 0; i < 4; i++) {
    if (uid[i] != validUID[i]) return false;
  }
  return true;
}

void setup() {
  Serial.begin(115200);
  SPI.begin();              // Start SPI bus for RFID
  Wire.begin();             // Start I2C bus for OLED
  
  // Initialize the OLED display
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { // Check if 0x3C is correct address
    Serial.println(F("SSD1306 allocation failed"));
    while (true); // Halt if OLED does not initialize
  }
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 10);
  display.println(F("Access Control System Initialized"));
  display.display();
  delay(2000);

  // Initialize RFID reader
  rfid.PCD_Init();

  doorServo1.attach(SERVO_PIN1);
  doorServo2.attach(SERVO_PIN2);


  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(RYG_RED, OUTPUT);
  pinMode(RYG_YELLOW, OUTPUT);
  pinMode(RYG_GREEN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);


  digitalWrite(RYG_RED, HIGH);   // Red LED on (door closed)
  digitalWrite(RYG_GREEN, LOW);  // Green LED off
  digitalWrite(BUZZER_PIN, LOW); // Ensure buzzer is off

  Serial.println("Setup complete. System ready.");
}

void loop() {
  // distance from ultrasonic sensor for the first door
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  duration = pulseIn(ECHO_PIN, HIGH);
  distance = duration * 0.034 / 2;

  if (distance <= 50 && !door1Open) {
    // If person is close and door is closed, open the first door
    doorServo1.write(90);  // Open position (90 degrees)
    delay(500);            // Give servo time to move
    door1Open = true;      // Update the door state to open

    // Set LEDs: turn off red, turn on green
    digitalWrite(RYG_RED, LOW);
    digitalWrite(RYG_GREEN, HIGH);

    Serial.println("Gate 1 opened");  // Print message
  } else if (distance > 50 && door1Open) {
    // If person is not close and door is open, close the first door
    doorServo1.write(0);   // Closed position (0 degrees)
    delay(500);            // Give servo time to move
    door1Open = false;     // Update the door state to closed

    // Set LEDs: turn on red, turn off green
    digitalWrite(RYG_RED, HIGH);
    digitalWrite(RYG_GREEN, LOW);

    Serial.println("Gate 1 closed");  
  }

  // Check RFID for the second door
  if (rfid.PICC_IsNewCardPresent() && rfid.PICC_ReadCardSerial()) {
    Serial.print("Detected UID: ");
    printHex(rfid.uid.uidByte, rfid.uid.size);
    Serial.println();

    if (checkValidUID(rfid.uid.uidByte, rfid.uid.size)) {
      // Valid entry - Open Gate 2
      Serial.println("Valid RFID detected. Opening Gate 2...");
      doorServo2.write(90);  // Open the second door (90 degrees)
      delay(1000);           // Give servo time to move
      digitalWrite(RYG_GREEN, HIGH); // Green LED for valid entry
      display.clearDisplay();
      display.setCursor(0, 10);
      display.println("Access Granted");
      display.display();
      delay(3000);           // Keep the door open for 3 seconds
      doorServo2.write(0);   // Close the door (0 degrees)
      delay(1000);           // Allow servo to return
      digitalWrite(RYG_GREEN, LOW);
      Serial.println("Gate 2 closed"); 
    } else {
      // Invalid entry - Sound buzzer and display error
      Serial.println("Invalid RFID detected.");
      digitalWrite(RYG_RED, HIGH);  // Red LED for invalid entry
      display.clearDisplay();
      display.setCursor(0, 10);
      display.println("Entry restricted");
      display.println("for Outsiders");
      display.display();
      
      // Activate buzzer for 1 second
      digitalWrite(BUZZER_PIN, HIGH);
      delay(1000);  // Buzzer ON for 1 second
      digitalWrite(BUZZER_PIN, LOW);  // Turn off buzzer after 1 second
      
      delay(2000);  // Display error for 2 seconds
      digitalWrite(RYG_RED, LOW);    // Turn off red LED
    }
    rfid.PICC_HaltA();  // Halt PICC
    rfid.PCD_StopCrypto1(); // Stop encryption on PCD
  }


  display.clearDisplay();
  delay(100);
}
