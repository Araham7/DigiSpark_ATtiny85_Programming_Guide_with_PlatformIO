# **Complete DigiSpark ATtiny85 Programming Guide with PlatformIO** 🚀

## **Overview**
DigiSpark is a tiny USB development board based on ATtiny85 microcontroller. PlatformIO makes it easy to program DigiSpark with professional development tools. This comprehensive guide covers setup, configuration, and troubleshooting for seamless DigiSpark programming.

---

## **Complete Setup Guide**

### **Step 1: Install PlatformIO**
#### **Method 1: VS Code Extension (Recommended)**
1. Install Visual Studio Code
2. Open Extensions Marketplace (Ctrl+Shift+X)
3. Search for "PlatformIO IDE"
4. Click Install

#### **Method 2: Command Line Installation**
```bash
# Install PlatformIO Core
pip install platformio

# Or use pipx (recommended for isolation)
pip install pipx
pipx ensurepath
pipx install platformio
```

### **Step 2: Create New Project**
```bash
# Create project directory
mkdir digispark-project
cd digispark-project

# Initialize PlatformIO project for DigiSpark
pio project init --board digispark-tiny
```

### **Step 3: Configure platformio.ini (CRITICAL SETTINGS)**
```ini
; PlatformIO Project Configuration File
;
;   Build options: build flags, source filter
;   Upload options: custom upload port, speed and extra flags
;   Library options: dependencies, extra library storages
;   Advanced options: extra scripting
;
; Please visit documentation for the other options and examples
; https://docs.platformio.org/page/projectconf.html

[env:digispark-tiny]
platform = atmelavr
board = digispark-tiny
framework = arduino

; # CRITICAL: Add these lines for DigiSpark
upload_protocol = micronucleus
upload_port = usb

; # Optional but helpful
board_build.f_cpu = 16500000L
monitor_speed = 9600
```

---

## **Troubleshooting & Essential Setup**

### **Step 4: Install Micronucleus (Required for Upload)**
```bash
# (a) First install required packages
sudo apt-get update
sudo apt-get install libusb-dev gcc make git

# (b) Clone micronucleus repository
git clone https://github.com/micronucleus/micronucleus.git

# (c) Build micronucleus
cd micronucleus/commandline
make clean
make

# (d) Install micronucleus globally
sudo cp micronucleus /usr/local/bin/

# Verify installation
micronucleus --help
```

### **Step 5: Set USB Permissions for PlatformIO**
```bash
# Create udev rules file for DigiSpark
sudo nano /etc/udev/rules.d/49-micronucleus.rules
```

Add this line to the file:
```text
SUBSYSTEM=="usb", ATTRS{idVendor}=="16d0", ATTRS{idProduct}=="0753", MODE="0660", GROUP="plugdev"
```

Then apply the changes:
```bash
# Reload udev rules
sudo udevadm control --reload-rules
sudo udevadm trigger

# Add user to necessary groups
sudo usermod -a -G plugdev $USER
sudo usermod -a -G dialout $USER

# IMPORTANT: Logout and login again or reboot
# Or run: newgrp plugdev
```

### **Step 6: Test the Setup**
```bash
# First build the project
pio run

# Then upload to DigiSpark
pio run -t upload

# IMPORTANT UPLOAD PROCESS:
# 1. Run the upload command first
# 2. Wait for "Please plug in the device" message
# 3. THEN connect your DigiSpark
# 4. Wait for upload to complete (5-10 seconds)
```

---

## **Sample Programs for Testing**

### **Basic Blink Example (src/main.cpp)**
```cpp
#include <Arduino.h>

#define LED_PIN 1  // Built-in LED on DigiSpark (Pin 1)

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, HIGH);
  delay(1000);
  digitalWrite(LED_PIN, LOW);
  delay(1000);
}
```

### **Button Input Example**
```cpp
#include <Arduino.h>

#define LED_PIN 1
#define BUTTON_PIN 2

void setup() {
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);
}

void loop() {
  if (digitalRead(BUTTON_PIN) == LOW) {
    digitalWrite(LED_PIN, HIGH);  // LED ON when button pressed
  } else {
    digitalWrite(LED_PIN, LOW);   // LED OFF when button released
  }
}
```

---

## **Expected Successful Output**

When everything is properly configured, you should see output similar to this:

```bash
# First build the project
pio run

Processing digispark-tiny (platform: atmelavr; board: digispark-tiny; framework: arduino)
--------------------------------------------------------------------------------------------------------
Verbose mode can be enabled via `-v, --verbose` option
CONFIGURATION: https://docs.platformio.org/page/boards/atmelavr/digispark-tiny.html
PLATFORM: Atmel AVR (5.1.0) > Digispark USB
HARDWARE: ATTINY85 16MHz, 512B RAM, 5.87KB Flash
DEBUG: Current (simavr) External (simavr)
PACKAGES: 
 - framework-arduino-avr-digistump @ 1.7.2 
 - toolchain-atmelavr @ 1.70300.191015 (7.3.0)
LDF: Library Dependency Finder -> https://bit.ly/configure-pio-ldf
LDF Modes: Finder ~ chain, Compatibility ~ soft
Found 24 compatible libraries
Scanning dependencies...
No dependencies
Building in release mode
...
===================================== [SUCCESS] Took 4.55 seconds =====================================

# Then upload
pio run -t upload

Processing digispark-tiny (platform: atmelavr; board: digispark-tiny; framework: arduino)
--------------------------------------------------------------------------------------------------------
...
Configuring upload protocol...
AVAILABLE: micronucleus
CURRENT: upload_protocol = micronucleus
Uploading .pio/build/digispark-tiny/firmware.hex
> Please plug in the device (will time out in 60 seconds) ... 
> Device is found!
...
> Starting the user app ...
running: 100% complete
>> Micronucleus done. Thank you!
===================================== [SUCCESS] Took 4.43 seconds =====================================
```

---

## **Common Issues & Solutions**

### **Issue 1: "Command not found: micronucleus"**
```bash
# Reinstall micronucleus
cd micronucleus/commandline
sudo make uninstall
make clean
make
sudo make install
```

### **Issue 2: "Permission denied" on USB**
```bash
# Check your user groups
groups $USER

# If plugdev not listed, add it
sudo usermod -a -G plugdev $USER

# Check USB device
lsusb | grep 16d0

# Should show: Bus 001 Device 008: ID 16d0:0753
```

### **Issue 3: Upload Timeout**
1. Make sure DigiSpark is **NOT** connected when you start upload
2. Wait for "Please plug in the device" message
3. Then connect DigiSpark within 60 seconds
4. Try different USB port

### **Issue 4: PlatformIO not detecting board**
```bash
# List all available boards
pio boards | grep -i digispark

# Should show: digispark-tiny
```

---

## **Advanced Configuration**

### **Alternative platformio.ini with More Options**
```ini
[env:digispark-tiny]
platform = atmelavr
board = digispark-tiny
framework = arduino

; Upload configuration
upload_protocol = micronucleus
upload_port = usb
upload_flags = 
    -t120

; Build configuration
board_build.f_cpu = 16500000L
build_flags = 
    -Os
    -Wl,--gc-sections
    -ffunction-sections
    -fdata-sections

; Serial monitor
monitor_speed = 9600
monitor_rts = 0
monitor_dtr = 0

; Libraries
lib_deps = 
    ; Add libraries here
    ; https://platformio.org/lib/search?query=digispark
```

### **Using External Libraries**
```bash
# Search for DigiSpark compatible libraries
pio lib search "ATTiny85"

# Install a library
pio lib install "DigiKeyboard"
```

---

## **Quick Reference Commands**

```bash
# Build project
pio run

# Upload to DigiSpark
pio run -t upload

# Clean build files
pio run -t clean

# Serial monitor
pio device monitor

# List connected devices
pio device list

# Update PlatformIO
pio upgrade

# Check PlatformIO version
pio --version

# PlatformIO home dashboard
pio home
```

---

## **DigiSpark Pin Reference**

| Pin | ATtiny85 | Arduino | Function              |
|-----|----------|---------|-----------------------|
| P0  | PB0      | 0       | Digital I/O, ADC      |
| P1  | PB1      | 1       | Built-in LED, PWM     |
| P2  | PB2      | 2       | Digital I/O, ADC      |
| P3  | PB3      | 3       | Digital I/O, ADC      |
| P4  | PB4      | 4       | Digital I/O           |
| P5  | PB5      | 5       | Digital I/O, Reset    |
| USB | -        | -       | Power + Data          |

---

## **Tips for Success**

1. **Always** connect DigiSpark **AFTER** starting upload command
2. Use short USB cable (some long cables cause issues)
3. If upload fails, unplug and try again
4. For first-time setup, reboot after adding user to groups
5. Keep PlatformIO updated: `pio upgrade`
6. Use PlatformIO's serial monitor for debugging
7. Check `~/.platformio/platformio.ini` for global settings

---

## **Conclusion**

With this complete guide, you should be able to:
✅ Install and configure PlatformIO for DigiSpark  
✅ Set up micronucleus for uploading  
✅ Configure proper USB permissions  
✅ Build and upload programs successfully  
✅ Troubleshoot common issues  

The DigiSpark ATtiny85 is a powerful tiny microcontroller that's perfect for small projects, and PlatformIO provides a professional development environment for it.

Happy coding! 🎯
