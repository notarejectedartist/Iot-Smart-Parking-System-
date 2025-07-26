#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <Servo.h>
// LCD setup (0x27 is common I2C address for 16x2 LCD)
LiquidCrystal_I2C lcd(0x27, 16, 2);
// Servo setup for gate control
Servo gate;
const int gatePin = 4;
// IR sensor pins
const int entryPin = 5; // Entry sensor
const int exitPin = 6; // Exit sensor
const int slot1Pin = 2; // Parking slot 1
const int slot2Pin = 3; // Parking slot 2
// Parking system variables
int freeSpots = 2; // Start with 2 free spots
bool gateMoving = false; // To prevent multiple gate operations
unsigned long gateTimer = 0;
void setup() {
Serial.begin(9600);
// Initialize LCD
lcd.init();
lcd.backlight();
// Setup servo
gate.attach(gatePin);
closeGate(); // Start with gate closed
// Setup sensors
pinMode(entryPin, INPUT_PULLUP);
pinMode(exitPin, INPUT_PULLUP);
pinMode(slot1Pin, INPUT_PULLUP);
pinMode(slot2Pin, INPUT_PULLUP);
// Show welcome message
lcd.setCursor(0, 0);
lcd.print(" SMART PARKING ");
lcd.setCursor(0, 1);
lcd.print(" SYSTEM ");
delay(2000);
updateDisplay();
}
void loop() {
// Check if car is entering
if (digitalRead(entryPin) == LOW && !gateMoving) {
if (freeSpots > 0) {
openGate();
freeSpots--;
updateDisplay();
} else {
showFullMessage();
}
}
// Check if car is exiting
if (digitalRead(exitPin) == LOW && !gateMoving) {
openGate();
freeSpots = min(2, freeSpots + 1); // Never exceed 2 spots
updateDisplay();
}
// Handle automatic gate closing
if (gateMoving && millis() - gateTimer >= 5000) {
closeGate();
gateMoving = false;
}
// Update display every second
static unsigned long lastUpdate = 0;
if (millis() - lastUpdate >= 1000) {
updateDisplay();
lastUpdate = millis();
}
}
void openGate() {
gate.write(180); // Open gate (180° position)
gateMoving = true;
gateTimer = millis();
}
void closeGate() {
gate.write(90); // Close gate (90° position)
}
void updateDisplay() {
lcd.clear();
// Show free spots
lcd.setCursor(0, 0);
lcd.print("Free: ");
lcd.print(freeSpots);
lcd.print("/2");
// Show slot status
lcd.setCursor(0, 1);
lcd.print("S1:");
lcd.print(digitalRead(slot1Pin) == HIGH ? "Empty" : "Full ");
lcd.print(" S2:");
lcd.print(digitalRead(slot2Pin) == HIGH ? "Empty" : "Full");
}
void showFullMessage() {
lcd.clear();
lcd.setCursor(0, 0);
lcd.print(" PARKING FULL ");
lcd.setCursor(0, 1);
lcd.print(" TRY LATER ");
delay(3000);
updateDisplay();
}
#include Wire.h.txt
