# Infinity-gear-arm-module-
Infinity Gear: Engineering the Bilateral Arm Exoskeleton Module
Introduction
The evolution of wearable robotics has long been dominated by heavy, cost-prohibitive systems designed primarily for industrial or military conglomerates. The Infinity Gear project was conceived to challenge this paradigm by developing an open-source, modular wearable exoskeleton that achieves bimodal human augmentation within an accessible design framework. As the foundational phase of the platform, the bilateral arm module focuses on augmenting upper-body strength, assisting with heavy lifting, and reducing muscular fatigue through precise mechanical leverage and real-time electronic control.
Skeletal Framing and Ergonomic Integration
Designing a wearable machine requires a delicate balance between structural rigidity and human comfort. The architectural backbone of the arm module relies on 300 mm 2020 Aluminum Extrusion Rails. This specific length was chosen to match the natural anatomical span of an adult upper and lower arm, providing a parallel load-bearing skeleton that runs alongside the limbs without restricting natural joint rotation at the wrist or shoulder.
To bridge the gap between industrial aluminum and human flesh, the interior channels of the frame are lined with high-density EVA foam. Unlike soft craft foams that compress completely under pressure, high-density foam provides resilient cushioning that absorbs physical load impact and prevents pressure points during operation. The entire assembly is secured to the bicep and forearm using adjustable cinch straps equipped with industrial metal buckles, allowing for high-tension lockdown that prevents slippage when the motors engage.
Actuation and Mechanical Force Amplification
To achieve true strength enhancement, the arm module utilizes a dual-actuation strategy combining linear thrust and rotational torque:
 * Elbow Lift-Assist: Positioned laterally across the elbow joint, 12V DC Micro Linear Actuators (capable of delivering significant linear force) act as artificial muscles. When activated, they push and hold the forearm upward, bearing the brunt of heavy loads to relieve strain on the user's biceps and joints.
 * Dynamic Articulation: Digital metal-gear servo motors are integrated to handle high-speed positioning and snap extensions, ensuring the arm module can mirror rapid, fluid movements without lagging behind natural human reflexes.
Kinematic Sensing and Central Intelligence
A mechanical frame and motors are only as effective as the intelligence driving them. The arm module operates on a closed-loop feedback system centered around an ESP32 microcontroller. Serving as the brain of the unit, the ESP32 coordinates PWM control signals for the actuators while managing wireless data telemetry.
Spatial awareness is achieved via a GY-521 MPU6050 6-Axis IMU Sensor mounted directly on the forearm. By continuously tracking angular velocity, tilt angles, and movement trajectories in real time, the sensor feeds kinematic data back to the microcontroller. This allows the system to distinguish between resting states, voluntary lifts, and dynamic motions, ensuring that motor assistance engages precisely when needed.
Conclusion
The successful design and specification of the Infinity Gear arm module proves that complex wearable robotics can be engineered using accessible, modular components. By harmonizing rigid aluminum framing, high-force linear actuation, ergonomic cushioning, and intelligent sensor feedback, the arm module establishes a robust foundation for human augmentation—paving the way for seamless expansion into lower-limb jump and speed-assist modules.

Wiring Connections Guide
 1. Wire the MPU6050 IMU Sensor (I2C)
   Step 1
   Connect the GY-521 MPU6050 sensor to the ESP32 using standard I2C pins:
   * VCC \rightarrow ESP32 3.3V
   * GND \rightarrow ESP32 GND
   * SDA \rightarrow ESP32 GPIO 21
   * SCL \rightarrow ESP32 GPIO 22
   Verification: Open the Arduino Serial Monitor after uploading the code; you should see "MPU6050 connection successful!" without initialization errors.
 2. Connect the Dual Motor Driver (for 12V Actuators)
   Step 2
   Connect a dual motor driver (such as an L298N or BTS7960) to handle the high current required by the 12V micro linear actuators:
   * Driver IN1 / IN2 \rightarrow ESP32 GPIO 25 / GPIO 26 (Left Actuator Control)
   * Driver IN3 / IN4 \rightarrow ESP32 GPIO 27 / GPIO 14 (Right Actuator Control)
   * Driver VCC / Power \rightarrow 12V Battery Positive (+)
   * Driver GND \rightarrow Battery Negative (-) and tied to ESP32 GND (Common Ground)
   Verification: Send a test command via code to move the actuators; they should extend smoothly in response to logic signals.
 3. Wire the MG996R Servos
   Step 3
   Connect the digital metal-gear servos for joint articulation:
   * Left Servo Signal \rightarrow ESP32 GPIO 18
   * Right Servo Signal \rightarrow ESP32 GPIO 19
   * Servo VCC (Red) \rightarrow External 5V Regulated Rail (Do not power servos directly from the ESP32 3.3V pin)
   * Servo GND (Brown/Black) \rightarrow Common Ground
   Verification: Power on the system and observe that the servos lock into their default neutral position (90°) and respond to movement sweeps.
ESP32 Firmware Code (esp32_arm_controller.ino)
Copy and paste the following code into your Arduino IDE (ensure you have installed the MPU6050 and ESP32Servo libraries via the Library Manager):
#include <Wire.h>
#include <MPU6050.h>
#include <ESP32Servo.h>

// Initialize MPU6050 Sensor
MPU6050 mpu;

// Initialize Servos
Servo leftServo;
Servo rightServo;

// Motor Driver Pins (L298N / Dual DC Motor Driver)
const int act1Pin1 = 25;
const int act1Pin2 = 26;
const int act2Pin1 = 27;
const int act2Pin2 = 14;

// Servo Pins
const int leftServoPin = 18;
const int rightServoPin = 19;

void setup() {
  Serial.begin(115200);
  Wire.begin();

  // Initialize MPU6050
  mpu.initialize();
  Serial.println(mpu.testConnection() ? "MPU6050 connection successful!" : "MPU6050 connection failed!");

  // Configure Motor Control Pins
  pinMode(act1Pin1, OUTPUT);
  pinMode(act1Pin2, OUTPUT);
  pinMode(act2Pin1, OUTPUT);
  pinMode(act2Pin2, OUTPUT);

  // Attach Servos
  leftServo.attach(leftServoPin);
  rightServo.attach(rightServoPin);

  // Set initial neutral positions
  leftServo.write(90);
  rightServo.write(90);
  stopActuators();
}

void loop() {
  // Read sensor data (Accelerometer & Gyroscope)
  int16_t ax, ay, az, gx, gy, gz;
  mpu.getMotion6(&ax, &ay, &az, &gx, &gy, &gz);

  // Map acceleration values to determine arm tilt angle (simple threshold logic)
  // When arm tilts forward/backward, trigger assistance
  if (ax > 10000) {
    // Lift / Extend action
    extendActuators();
    leftServo.write(135);
    rightServo.write(135);
  } else if (ax < -10000) {
    // Retract action
    retractActuators();
    leftServo.write(45);
    rightServo.write(45);
  } else {
    // Hold position / Idle
    stopActuators();
    leftServo.write(90);
    rightServo.write(90);
  }

  delay(50);
}

void extendActuators() {
  digitalWrite(act1Pin1, HIGH);
  digitalWrite(act1Pin2, LOW);
  digitalWrite(act2Pin1, HIGH);
  digitalWrite(act2Pin2, LOW);
}

void retractActuators() {
  digitalWrite(act1Pin1, LOW);
  digitalWrite(act1Pin2, HIGH);
  digitalWrite(act2Pin1, LOW);
  digitalWrite(act2Pin2, HIGH);
}

void stopActuators() {
  digitalWrite(act1Pin1, LOW);
  digitalWrite(act1Pin2, LOW);
  digitalWrite(act2Pin1, LOW);
  digitalWrite(act2Pin2, LOW);
}

<img width="1408" height="768" alt="52456" src="https://github.com/user-attachments/assets/2c78f5d1-f3d4-4b05-bfe9-572f29be5181" />
<img width="1672" height="941" alt="52445" src="https://github.com/user-attachments/assets/646c6c20-b057-4fc4-a397-8cde4123c15a" />
