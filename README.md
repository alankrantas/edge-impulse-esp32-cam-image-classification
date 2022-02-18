# Live Image Classification on ESP32-CAM and ST7735 TFT using MobileNet v1 from Edge Impulse (TinyML)

![41Ub2S0SjXL _AC_](https://user-images.githubusercontent.com/44191076/153631624-e13576b3-b440-4cd0-8a42-fd29cbe25a2d.jpg)

This example is for running a micro neural network model on the 10-dollar Ai-Thinker ESP32-CAM board and show the image classification results on a small TFT LCD display.

This is modified from [ESP32 Cam and Edge Impulse](https://github.com/edgeimpulse/example-esp32-cam) with simplified code, TFT support and copied necessary libraries from Espressif's [esp-face](https://github.com/Yuri-R-Studio/esp-face). ```esp-face``` had been changed a lot into [esp-dl](https://github.com/espressif/esp-dl) and thus broke the original example. The original example requires WiFi and has image lagging problems.

> See the original example repo or [this article](https://www.survivingwithandroid.com/tinyml-esp32-cam-edge-image-classification-with-edge-impulse/) about how to generate your own model on Edge Impulse. You can also still run the original example by copy every libraries in this example to the project directory, then re-open the .ino script.

![demo](https://user-images.githubusercontent.com/44191076/154735134-12b59e38-79d6-4890-945c-db0604b0444e.JPG)

## Setup

The following is needed in your Arduino IDE:

* [Arduino-ESP32 board support](https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json) (select ```Ai Thinker ESP32-CAM```)
* [Adafruit GFX Library](https://github.com/adafruit/Adafruit-GFX-Library)
* [Adafruit ST7735 and ST7789 Library](https://github.com/adafruit/Adafruit-ST7735-Library)
* All library files in this repo (simply open the .ino to import them)

Be noted that you won't be able to read any serial output if you use Arduino IDE 2.0!

## Wiring

![pinout](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/raw/main/ESP32-CAM-pinout-new.png)

![wiring](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/raw/main/esp32-cam-edge-impulse.png)

The whole system is powered from a power module that can output both 5V and 3.3V. The ESP32-CAM is powered by 5V and TFT by 3.3V. I use a 7.5V 1A charger (power modules require 6V+) to provide stable power. My power module can output 500 mA max (you don't need a lot since we don't use WiFi).

| USB-TTL pins | ESP32-CAM |
| --- | --- |
| Tx | GPIO 3 (UOR) |
| Rx | GPIO 1 (UOT) |
| GND | GND |

The USB-TTL's GND should be connected to the breadboard, not the ESP32-CAM itself. If you want to upload code, disconnect power then connect GPIO 0 to GND (also should be on the breadboard), then power it up. It would be in flash mode. (The alternative way is remove the ESP32-CAM itself and use the ESP32-CAM-MB programmer board.)

| TFT pins | ESP32-CAM |
| --- | --- |
| SCK (SCL) | GPIO 14 |
| MOST (SDA) | GPIO 13 |
| RESET (RST) | GPIO 12 |
| DC | GPIO 2 |
| CS | GPIO 15 |
| BL (back light) | 3V3 |

The script will display a 120x120 image on the TFT, so any 160x128 or 128x128 versions can be used. But you might want to change the parameter in ```tft.initR(INITR_GREENTAB);``` to ```INITR_REDTAB``` or ```INITR_BLACKTAB``` to get correct text colors.

| Button | ESP32-CAM |
| --- | --- |
| BTN | 3V3 |
| BTN | GPIO 4 |

Be noted that since the button pin is shared with the flash LED (this is the available pin left; GPIO 16 is camera-related), the button has to be **pulled down** with two 10 KΩ resistors.

## The Example Model - Cat & Dog Classification

I used Microsoft's [Kaggle Cats and Dogs Dataset](https://www.microsoft.com/en-us/download/details.aspx?id=54765) which has 12,500 cats and 12,500 dogs. 24,969 photos had successfully uploaded and split into 80-20% training/test sets.

The model I choose was ```MobileNetV1 96x96 0.25 (no final dense layer, 0.1 dropout)``` with transfer learning. Since free Edge Impulse accounts has a training time limit of 20 minutes per job, I can only train the model for 5 cycles. (You can go [ask for more](https://forum.edgeimpulse.com/t/err-deadlineexceeded-ways-to-fix-this/2354/2) though...) I imagine if you have only a dozen images per class, you can try better models or longer training cycles.

Anyway, I got ```89.8%``` accuracy for training set and ```86.97%``` for test set, which seems to be decent enough.

![1](https://user-images.githubusercontent.com/44191076/153631673-96b90c0b-5745-43b9-9e5f-9a426d8bfe61.png)

Also, ESP32-CAM is not yet an officially supported board, so I cannot use EON Tuner for futher find-tuning.

You can find my published Edge Impulse project here: [esp32-cam-cat-dog](https://studio.edgeimpulse.com/public/76904/latest).

[ei-esp32-cam-cat-dog-arduino-1.0.4.zip](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/blob/main/ei-esp32-cam-cat-dog-arduino-1.0.4.zip) is the downloaded Arduino library which can be imported into Ardiono IDE.

The camera captures 240x240 images and resize them into 96x96. The inference time is 2607 ms (2.6 secs) per image, which is not very fast,  with mostly good results. I don't know yet if different image sets or models may effect the result.

![bogdan-farca-CEx86maLUSc-unsplash](https://user-images.githubusercontent.com/44191076/153636524-9b2edab9-7c50-4aa1-9d6e-74477d67011f.jpg)

![richard-brutyo-Sg3XwuEpybU-unsplash](https://user-images.githubusercontent.com/44191076/153636561-16f7fb47-dcfc-4988-8772-85dcc5acfdac.jpg)
