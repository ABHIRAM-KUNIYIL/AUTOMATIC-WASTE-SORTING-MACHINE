# Automatic Waste Sorting Machine

This project, developed for Advanced Electronics course, is an automated system for sorting common waste items into three categories: metal, plastic, and glass.

## How It Works

The machine uses a combination of sensors controlled by an Arduino Uno to identify and sort waste:

1.  An **Ultrasonic Sensor** first detects if an object has been placed in the machine.
2.  An **Inductive Proximity Sensor** checks if the object is metallic.
3.  A **Capacitive Proximity Sensor** differentiates between plastic and glass. It's calibrated to detect glass but ignore plastic.
4.  Based on the sensor readings, two **Servo Motors** control a gate and a rotating pipe to direct the waste object into the correct collection bin.

## Core Components

* **Controller:** Arduino Uno
* **Sensors:**
    * Inductive Proximity Sensor
    * Capacitive Proximity Sensor
    * Ultrasonic Sensor
* **Actuators:** 2x TowerPro MG995 Servo Motors
* **Power:** 6V Power Supply

## Arduino Code

The logic for the sorting process is handled by the following Arduino code.

```cpp
#include <Servo.h>

// Declare servo motor objects
Servo Pipe_Servo;
Servo Gate_Servo;

// Define sensor pin variables
int sensorInd = A0; // Inductive sensor (analog)
int sensorpin = 2;  // Capacitive sensor (digital)
const int trigPin = 7; // Ultrasonic sensor trigger
const int echoPin = 8; // Ultrasonic sensor echo

// Declare variables for sensor readings and servo positions
int indValue;
long duration;
int distance;
float Pipe_Pos = 0.0;
float Gate_Pos = 0.0;

void setup() {
  // Initialize serial communication for debugging
  Serial.begin(9600);

  // Set pin modes for ultrasonic sensor
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  // Attach servos to their respective pins
  Pipe_Servo.attach(11);
  Gate_Servo.attach(10);

  // --- System Initialization ---
  Serial.println("System Initializing...");
  delay(5000); // Wait for components to stabilize

  // Move Pipe Servo to initial (zero) position (90 degrees)
  Pipe_Pos = Pipe_Servo.read();
  if (Pipe_Pos < 90) {
    for (int i = Pipe_Pos; i <= 90; i = i + 1) {
      Pipe_Servo.write(i);
      delay(15);
    }
  } else {
    for (int i = Pipe_Pos; i >= 90; i = i - 1) {
      Pipe_Servo.write(i);
      delay(15);
    }
  }
  delay(1000);

  // Cycle the Gate Servo to ensure it's ready
  for (int n = 0; n <= 45; n = n + 1) {
    Gate_Servo.write(n);
    delay(20);
  }
  delay(1000);
  for (int n = 45; n >= 0; n = n - 1) {
    Gate_Servo.write(n);
    delay(25);
  }
  
  Serial.println("System Ready.");
  delay(5000);
}

void loop() {
  delay(3000); // Wait before next scan

  // --- Ultrasonic Sensor Reading ---
  digitalWrite(trigPin, LOW);
  delayMicroseconds(10);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(30);
  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  distance = duration * 0.034 / 2;
  
  Serial.print("Distance: ");
  Serial.println(distance);

  // --- Read Inductive and Capacitive Sensors ---
  indValue = analogRead(sensorInd);
  int sensorstate = digitalRead(sensorpin);
  delay(500);

  // --- Main Sorting Logic ---
  
  // Condition for METAL
  if (indValue >= 250 && sensorstate == 1 && distance <= 14) {
    Serial.println("Metal Detected");
    moveServosToBin(90); // Metal Bin Position
  }
  // Condition for PLASTIC
  else if (indValue <= 250 && sensorstate == 1 && distance <= 15) {
    Serial.println("Plastic Detected");
    moveServosToBin(145); // Plastic Bin Position
  }
  // Condition for GLASS
  else if (indValue <= 250 && sensorstate != 1 && distance <= 14) {
    Serial.println("Glass Detected");
    moveServosToBin(25); // Glass Bin Position
  }
}

// Function to move servos to the target bin and operate the gate
void moveServosToBin(int targetPosition) {
  
  // Move Pipe Servo to the target bin position
  int currentPipePos = Pipe_Servo.read();
  if (currentPipePos < targetPosition) {
    for (int i = currentPipePos; i <= targetPosition; i++) {
      Pipe_Servo.write(i);
      delay(15);
    }
  } else {
    for (int i = currentPipePos; i >= targetPosition; i--) {
      Pipe_Servo.write(i);
      delay(15);
    }
  }
  delay(1000); // Wait for pipe to settle

  // Open and close the gate to drop the object
  for (int n = 0; n <= 45; n++) {
    Gate_Servo.write(n);
    delay(20);
  }
  delay(1000);
  for (int n = 45; n >= 0; n--) {
    Gate_Servo.write(n);
    delay(25);
  }
  delay(1000); // Wait before next cycle
}
