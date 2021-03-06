To use as a component of ESP-IDF
=================================================

## Installation

- Download and install [esp-idf v4.2 release](https://github.com/espressif/arduino-esp32/tree/idf-release/v4.2)
- Create blank idf project (from one of the examples)
- in the project folder, create a folder called components and clone this repository inside

    ```bash
    mkdir -p components && \
    cd components && \
    git clone -single-branch --branch mod-idf-v4.2 https://github.com/summivox/arduino-esp32.git arduino && \
    cd arduino && \
    git submodule update --init --recursive && \
    cd ../.. && \
    make menuconfig
  ```
  
  **If the project is already in a git repo, use these instead:**

  ```bash
  mkdir -p components && \
  git submodule add https://github.com/summivox/arduino-esp32 arduino --branch mod-idf-v4.2 --recursive
  ```

- ```make menuconfig``` has some Arduino options
    - **important: you MUST enable these to build**
        - mbedTLS: Enable pre-shared-key ciphersuites
        - bluetooth: At least enable BLE and Bluedroid
        
    - "Autostart Arduino setup and loop on boot"
        - If you enable this options, your main.cpp should be formated like any other sketch

          ```arduino
          //file: main.cpp
          #include "Arduino.h"

          void setup(){
            Serial.begin(115200);
          }

          void loop(){
            Serial.println("loop");
            delay(1000);
          }
          ```

        - Else you need to implement ```app_main()``` and call ```initArduino();``` in it.

          Keep in mind that setup() and loop() will not be called in this case.
          If you plan to base your code on examples provided in [esp-idf](https://github.com/espressif/esp-idf/tree/master/examples), please make sure move the app_main() function in main.cpp from the files in the example.

          ```arduino
          //file: main.cpp
          #include "Arduino.h"

          extern "C" void app_main()
          {
              initArduino();
              pinMode(4, OUTPUT);
              digitalWrite(4, HIGH);
              //do your own thing
          }
          ```
    - "Disable mutex locks for HAL"
        - If enabled, there will be no protection on the drivers from concurently accessing them from another thread/interrupt/core
    - "Autoconnect WiFi on boot"
        - If enabled, WiFi will start with the last known configuration
        - Else it will wait for WiFi.begin
- ```make flash monitor``` will build, upload and open serial monitor to your board

## Logging To Serial

If you are writing code that does not require Arduino to compile and you want your `ESP_LOGx` macros to work in Arduino IDE, you can enable the compatibility by adding the following lines after your includes:

```cpp
#ifdef ARDUINO_ARCH_ESP32
#include "esp32-hal-log.h"
#endif
```

## FreeRTOS Tick Rate (Hz)

You might notice that Arduino-esp32's `delay()` function will only work in multiples of 10ms. That is because, by default, esp-idf handles task events 100 times per second.
To fix that behavior you need to set FreeRTOS tick rate to 1000Hz in `make menuconfig` -> `Component config` -> `FreeRTOS` -> `Tick rate`.

## FreeRTOS task watchdog time out

In `menuconfig`, disable "Watch CPU1 Idle Task".

## Compilation Errors

As commits are made to esp-idf and submodules, the codebases can develop incompatibilities which cause compilation errors.  If you have problems compiling, follow the instructions in [Issue #1142](https://github.com/espressif/arduino-esp32/issues/1142) to roll esp-idf back to a known good version.
