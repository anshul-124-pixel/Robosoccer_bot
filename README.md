# Robosoccer_bot
#include <PPMReader.h>

#define PPM_PIN 3# RoboSoccer Bot

A wireless differential-drive RoboSoccer bot designed for fast movement, ball control, and competitive gameplay in a controlled arena. This project integrates embedded systems, motor control, wireless communication, and mechanical design using Arduino-based architecture.

Developed at Indian Institute of Technology Ropar.

# Project Overview

The RoboSoccer Bot is a remote-controlled robotic vehicle built to play robotic soccer matches. The bot is controlled wirelessly using an RF transmitter-receiver setup and is capable of:

* Forward and backward motion
* Sharp directional turning
* In-place rotation
* Fast response movement
* Ball pushing and control during gameplay

The system uses a differential drive mechanism, where independent motor control on the left and right side allows agile navigation and quick directional changes.


#  System Architecture

The complete system consists of:

* Wireless transmitter and receiver communication
* Arduino-based control unit
* Motor driver modules for high-current motor control
* DC geared motors for wheel actuation
* Voltage regulation circuitry
* Metal chassis with optimized front plate for ball handling


#  Working Mechanism

The RoboSoccer Bot operates using a battery-powered control and drive system.

## Power Distribution

* A 12V LiPo battery powers the entire bot.
* The LM2596 buck converter reduces voltage from 12V to 5V for safe microcontroller operation.
* Motor drivers receive direct battery power for high-current motor operation.


## Wireless Control

The user controls the bot through an RF transmitter.
The RF receiver mounted on the robot sends these commands to the Arduino Nano.

The Arduino:

1. Reads incoming control signals
2. Processes movement instructions
3. Generates motor control outputs
4. Sends PWM signals to motor drivers


## Motor Control

The robot uses:

* 4 DC motors
* 2 BTS7960 motor drivers

Each driver controls motors on one side of the chassis.

Using differential drive logic:

* Same direction on both sides → Forward / Backward
* Opposite directions → Rotation
* Speed variation between sides → Turning

PWM (Pulse Width Modulation) is used for:

* Speed control
* Smooth acceleration
* Better maneuverability

#  Code Overview

The Arduino code handles:

* RF receiver signal processing
* PWM motor speed control
* Direction logic
* Differential drive implementation
* Real-time response handling

## Core Functionalities

// Receiver input mapping
// Motor direction control
// PWM speed adjustment
// Left-right drive synchronization
// Turning and rotation logic

#  Project Structure

RoboSoccer-Bot/
│
├── Arduino_Code/
│   └── robosoccer_bot.ino
│
├── Circuit_Diagrams/
│
├── Images/
│
├── Documentation/
│   └── ROBOSOCCER BOT.pdf
│
└── README.md


#  Features

* Wireless remote operation
* Differential drive steering
* High torque motor system
* Fast directional response
* Compact and durable chassis
* Stable traction using rubber wheels
* Modular electronics architecture



#  Future Improvements

Potential future upgrades include:

* Autonomous navigation
* Computer vision integration
* Ball detection algorithms
* Dribbler and kicker mechanisms
* PID-based speed stabilization
* Bluetooth or WiFi control system
* Sensor-assisted gameplay


# Applications

* RoboSoccer competitions
* Embedded systems learning
* Robotics projects
* Differential drive experimentation
* Wireless control systems


#  Conclusion

The RoboSoccer Bot demonstrates the integration of:

* Embedded programming
* Wireless communication
* Motor control systems
* Mechanical design
* Real-time robotics control

This project provides a strong foundation for advanced robotics applications and competitive robotic gameplay.


# License

This project is open-source and available under the MIT License.
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
