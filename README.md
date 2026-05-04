# Connected-Embedded-Systems-Assignment-1
# Assignment 1: DS3231 RTC Application
# Grade: 93%
## Source code available upon request

## Overview

This project is a C++ application for interacting with a **DS3231 Real-Time Clock (RTC)** module over **I2C** on a Linux-based embedded platform. In addition to basic RTC access, the project integrates **GPIO-based LEDs**, the **SQW/INT alarm output**, a **snooze button**, and a custom **"own button"** feature.

The application demonstrates:

- Reading and displaying the current time
- Reading and displaying the current date
- Reading and displaying the RTC temperature sensor
- Configuring Alarm 1 and Alarm 2 on the DS3231
- Reacting to RTC alarm interrupts via the SQW/INT pin
- Driving LEDs to indicate alarm state and alarm trigger events
- Handling a snooze button for active alarms
- Handling a custom button that checks the current date and triggers a custom LED pattern

## Project Contents

This folder contains:

- `application.cpp` - main entry point and demo/test harness
- `DS3231.h` - DS3231 class interface
- `DS3231.cpp` - DS3231 implementation, alarm logic, GPIO logic, and test helpers
- `I2CDevice.h` - generic I2C device abstraction
- `I2CDevice.cpp` - Linux I2C implementation using `/dev/i2c-*`
- `build` - shell script used to compile the application
- `rtc` - compiled executable artifact
- `.vscode/` - local editor settings

## Architecture

### 1. `I2CDevice`

`I2CDevice` is a reusable base class that wraps low-level Linux I2C access. It provides functions to:

- open an I2C bus
- write a byte
- write a register
- read a single register
- read a block of registers
- dump register contents for debugging

The DS3231 driver inherits from this class.

### 2. `DS3231`

`DS3231` extends `I2CDevice` and implements all RTC-specific behavior, including:

- time/date formatting and decoding from BCD
- writing time/date values back to RTC registers
- configuring Alarm 1 and Alarm 2
- clearing alarm flags and disabling alarms
- enabling interrupt mode through the DS3231 control register
- interacting with GPIO lines through `libgpiod`

### 3. `application.cpp`

The main application currently acts as a demonstration harness. It:

1. creates a `DS3231` instance on bus `1` at device address `0x68`
2. opens the RTC device
3. initializes LED GPIO lines
4. clears existing alarms
5. displays the current time, date, and temperature
6. sets Alarm 1 and Alarm 2 with hardcoded test values
7. provides commented test hooks for alarm, SQW, snooze, and custom button features
8. turns off LEDs, resets alarms, releases GPIO lines, and closes the device before exiting

## Current Runtime Behavior

With the current `application.cpp`, the program will:

- print a banner to the terminal
- display current RTC time
- display current RTC date
- display current RTC temperature
- enable Alarm 1 and Alarm 2 with sample values
- light the corresponding alarm LEDs when alarms are enabled
- clean up before exit

Several extra tests are present in the code but commented out. Uncomment them in `application.cpp` to exercise more of the assignment functionality.

## Implemented Features

### RTC Display Functions

- `displayTime()` reads registers `0x00-0x02` and prints the current time
- `displayDate()` reads registers `0x03-0x06` and prints weekday, date, month, and year
- `displayTemp()` reads registers `0x11-0x12` and prints the internal DS3231 temperature

### RTC Write Functions

- `setTime(hours, minutes, seconds)` writes the time in BCD format
- `setDate(day, date, month, year)` writes day/date/month/year in BCD format

### Alarm Features

- `setAlarm1(hours, minutes, seconds, day)` configures Alarm 1 as a day-based alarm
- `setAlarm2(hours, minutes, date)` configures Alarm 2 as a date-based alarm
- `ReadAlarm(alarmNumber)` prints the stored alarm configuration
- `clearFlag(alarmNumber)` clears the DS3231 alarm flag
- `clearAlarmEnable(alarmNumber)` disables the selected alarm interrupt
- `resetAlarms()` disables both alarms and clears both alarm flags

### LED Features

Three LEDs are used:

- Alarm 1 active LED
- Alarm 2 active LED
- Alarm triggered/status LED

Implemented LED behavior includes:

- turning on the alarm-specific LED when an alarm is enabled
- blinking the triggered LED when an alarm interrupt occurs
- flashing LEDs during snooze indication
- flashing LEDs for the custom own-button feature

### Interrupt and Button Features

- `initSQWPin()` configures the DS3231 SQW/INT pin as a GPIO input with falling-edge detection
- `readSQWPin()` waits for alarm interrupts and determines whether Alarm 1 or Alarm 2 fired
- `initSnoozeButton()` and `readSnoozeButton()` handle a hardware snooze button
- `snoozeAlarm()` temporarily shifts the next alarm event
- `initOwnButton()` and `readOwnButton()` handle a custom button input
- `ownFunction()` checks whether the current date is `27 May` and prints a custom message

## GPIO Mapping

The following GPIO line offsets are hardcoded in `DS3231.cpp`:

| Signal | GPIO line offset |
| --- | ---: |
| SQW / INT input | `5` |
| Alarm 1 LED | `23` |
| Alarm 2 LED | `24` |
| Alarm triggered LED | `25` |
| Snooze button | `27` |
| Own button | `22` |

The GPIO chip path is hardcoded as:

```text
/dev/gpiochip0
```

If your hardware uses different GPIO line numbers or a different GPIO chip, update those constants in `DS3231.cpp`.

## I2C Configuration

The application uses:

- I2C bus: `/dev/i2c-1`
- DS3231 device address: `0x68`

These values come from:

- `I2CDevice.h`, which defines `I2C_1` as `/dev/i2c-1`
- `application.cpp`, which constructs `DS3231 rtc(1, 0x68)`

## Build Instructions

### Provided build script

The project already includes a `build` shell script containing:

```bash
g++ application.cpp I2CDevice.cpp DS3231.cpp -lgpiod -o rtc
```

### Manual build

To compile manually on the target Linux device:

```bash
g++ application.cpp I2CDevice.cpp DS3231.cpp -lgpiod -o rtc
```

### Using the build script

Make the script executable if needed, then run it:

```bash
chmod +x build
./build
```

## Run Instructions

Run the compiled program with:

```bash
./rtc
```

Because the program accesses `/dev/i2c-1` and `/dev/gpiochip0`, it will normally need appropriate permissions. On many systems this means running it with elevated privileges:

```bash
sudo ./rtc
```

## Dependencies

This project is intended for a Linux embedded environment with:

- `g++`
- Linux I2C userspace support
- `libgpiod`
- access to `/dev/i2c-1`
- access to `/dev/gpiochip0`

Typical package requirements on Debian-based systems are:

```bash
sudo apt update
sudo apt install g++ libgpiod-dev i2c-tools
```

Depending on the platform image, GPIO and I2C kernel support may also need to be enabled.

## Hardware Requirements

This project assumes the following hardware is connected:

- a DS3231 RTC module connected to the system I2C bus
- the DS3231 interrupt output connected to the selected GPIO line for SQW/INT
- three LEDs connected to the configured GPIO output lines
- one push button for snooze
- one push button for the custom own-button feature

## Suggested Demo Flow

If you want to demonstrate all assignment requirements, a typical flow is:

1. verify the RTC is connected and visible on I2C bus 1
2. compile the project
3. run the basic display functions for time/date/temperature
4. test `testDateTimeTemp()` by uncommenting it in `application.cpp`
5. test Alarm 1 and Alarm 2 individually
6. test SQW interrupt handling with `testSQW()`
7. test snooze behavior with `testSnooze()`
8. test the custom own-button behavior with `testOwnButton()`

## Notes on the Test Helpers

The DS3231 class includes several helper methods specifically for assignment demonstration:

- `testDateTimeTemp()`
- `testAlarm1()`
- `testAlarm2()`
- `testSQW()`
- `testSnooze()`
- `testOwnButton()`

Most of these are currently commented out in `application.cpp`. Uncomment only the test path you want to run at a given time to avoid conflicting behaviors.

## Known Constraints and Assumptions

- The code is written for a Linux target, not for native execution on Windows.
- GPIO line numbers are platform-specific and may need adjustment.
- The DS3231 weekday handling in this project uses numeric day values `0-6` for Monday-Sunday.
- Alarm and button logic is largely demo-oriented and driven from hardcoded values in `application.cpp`.
- The application is designed for direct hardware access and cannot be fully exercised without the target device.

## Example Output

Expected terminal output will look similar to:

```text
DS3231 RTC Code for Assignment 1 EEN1071
=================================================
Time, Date, and Temperature
Current Time: HH:MM:SS
Day and Date: Wednesday 27/05/2026
Current Temperature: 23.25 C
=================================================
Alarms
=================================================
Own Button
```

The exact values depend on the RTC contents and which tests are enabled.

## Possible Improvements

If you continue developing the project, useful next steps would be:

- add a `Makefile`
- accept alarm/time settings from command-line arguments
- separate demo code from the reusable DS3231 driver
- improve error reporting for invalid input values
- add debounce and state-reset logic refinement for repeated button usage
- document the exact hardware wiring in a schematic or wiring table

## Summary

This project implements a DS3231 RTC interface in C++ using Linux I2C and `libgpiod`. It supports time/date/temperature access, two alarms, SQW interrupt handling, LED status indication, snooze functionality, and a custom date-check button feature. The code is structured so that `I2CDevice` handles generic bus access while `DS3231` implements device-specific logic and assignment-specific behavior.

