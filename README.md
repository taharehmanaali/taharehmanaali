// Libraries
#include <Wire.h>
#include <Adafruit_INA219.h>

// Sensor setup
Adafruit_INA219 ina219;

// MPPT Parameters
float voltage = 0;
float current = 0;
float power = 0;
float lastPower = 0;
float voltageStep = 0.1;  // Voltage step for P&O
float dutyCycle = 0.5;    // Starting duty cycle (0-1)
const int pwmPin = 9;     // PWM output to buck converter

void setup() {
  Serial.begin(9600);
  ina219.begin();  // Initialize the INA219 sensor

  // PWM setup
  pinMode(pwmPin, OUTPUT);
  analogWrite(pwmPin, dutyCycle * 255);  // Initial duty cycle
}

void loop() {
  // Read voltage and current from the sensor
  voltage = ina219.getBusVoltage_V();
  current = ina219.getCurrent_mA() / 1000.0;  // Convert mA to A
  power = voltage * current;                  // Calculate power

  // MPPT - Perturb & Observe algorithm
  if (power > lastPower) {
    dutyCycle += voltageStep;
  } else {
    dutyCycle -= voltageStep;
  }

  // Constrain duty cycle within 0 and 1
  dutyCycle = constrain(dutyCycle, 0, 1);

  // Apply new duty cycle to PWM
  analogWrite(pwmPin, dutyCycle * 255);

  // Print the values for monitoring
  Serial.print("Voltage: ");
  Serial.print(voltage);
  Serial.print(" V, Current: ");
  Serial.print(current);
  Serial.print(" A, Power: ");
  Serial.print(power);
  Serial.print(" W, Duty Cycle: ");
  Serial.println(dutyCycle);

  // Update la
