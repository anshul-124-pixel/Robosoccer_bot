# Robosoccer_bot
#include <PPMReader.h>

#define PPM_PIN 3
#define CHANNELS 6

PPMReader ppm(PPM_PIN, CHANNELS);

// Motor Pins
// Left Motor
#define L_LPWM 6
#define L_RPWM 5

// Right Motor
#define R_LPWM 10
#define R_RPWM 11

void setup() {
  Serial.begin(9600);

  pinMode(L_LPWM, OUTPUT);
  pinMode(L_RPWM, OUTPUT);

  pinMode(R_LPWM, OUTPUT);
  pinMode(R_RPWM, OUTPUT);
}

void loop() {

  // Read Channels
  int throttle = ppm.rawChannelValue(2); // Up/Down stick
  int steering = ppm.rawChannelValue(1); // Left/Right stick

  // Convert to -255 to +255
  int speed = map(throttle, 1000, 2000, -255, 255);
  int turn  = map(steering, 1000, 2000, -255, 255);
  turn = turn / 2; 

  // Deadzone near center (avoid jitter)
  if (abs(speed) < 20) speed = 0;
  if (abs(turn) < 20) turn = 0;

  // Mixing (Differential Drive)
  int leftSpeed  = speed + turn;
  int rightSpeed = speed - turn;

  // Limit values
  leftSpeed  = constrain(leftSpeed, -255, 255);
  rightSpeed = constrain(rightSpeed, -255, 255);

  // Drive Motors
  driveMotor(leftSpeed, L_LPWM, L_RPWM);
  driveMotor(rightSpeed, R_LPWM, R_RPWM);

  // Debug
  Serial.print("Throttle: "); Serial.print(throttle);
  Serial.print(" Steering: "); Serial.print(steering);
  Serial.print(" LeftPWM: "); Serial.print(leftSpeed);
  Serial.print(" RightPWM: "); Serial.println(rightSpeed);

  delay(20);
}

// Motor Control Function
void driveMotor(int pwm, int lpwmPin, int rpwmPin) {

  if (pwm > 0) {
    analogWrite(lpwmPin, pwm);
    analogWrite(rpwmPin, 0);
  }
  else if (pwm < 0) {
    analogWrite(lpwmPin, 0);
    analogWrite(rpwmPin, -pwm);
  }
  else {
    analogWrite(lpwmPin, 0);
    analogWrite(rpwmPin, 0);
  }
}
