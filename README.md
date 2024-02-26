# Live Image Classification on ESP32-CAM and ST7735 TFT using MobileNet v1 from Edge Impulse (TinyML)

![41Ub2S0SjXL _AC_](https://user-images.githubusercontent.com/44191076/153631624-e13576b3-b440-4cd0-8a42-fd29cbe25a2d.jpg)

This example is for running a micro neural network model on the 10-dollar Ai-Thinker ESP32-CAM board and show the image classification results on a small TFT LCD display.

> Be noted that I am not testing and improving this any further. This is simply a proof-of-concept and demonstration that you can make simple and practical edge AI devices without making them overly complicated.

This is modified from [ESP32 Cam and Edge Impulse](https://github.com/edgeimpulse/example-esp32-cam) with simplified code, TFT support and copied necessary libraries from Espressif's [esp-face](https://github.com/Yuri-R-Studio/esp-face). ```esp-face``` had been refactored into [esp-dl](https://github.com/espressif/esp-dl) to support their other products and thus broke the original example. The original example also requires WiFi connection and has image lagging problems, which makes it difficult to use. My version works more like a hand-held point and shoot camera.

> See the original example repo or [this article](https://www.survivingwithandroid.com/tinyml-esp32-cam-edge-image-classification-with-edge-impulse/) about how to generate your own model on Edge Impulse. You can also still run the original example by copy every libraries in this example to the project directory, then re-open the .ino script.

![demo](https://user-images.githubusercontent.com/44191076/154735134-12b59e38-79d6-4890-945c-db0604b0444e.JPG)

See the [video demonstration](https://www.youtube.com/watch?v=UoWfiEZE0Y4)

[中文版介紹](https://alankrantas.medium.com/tinyml-%E5%BD%B1%E5%83%8F%E8%BE%A8%E8%AD%98%E5%BE%AE%E5%9E%8B%E5%8C%96-%E5%9C%A8%E6%88%90%E6%9C%AC-10-%E7%BE%8E%E5%85%83%E7%9A%84-esp32-cam-%E9%96%8B%E7%99%BC%E6%9D%BF%E4%B8%8A%E5%8D%B3%E6%99%82%E5%88%86%E9%A1%9E%E8%B2%93%E7%8B%97-%E5%BE%9E%E6%AD%A4%E8%B7%9F%E9%BA%BB%E7%85%A9%E7%9A%84-wifi-%E9%80%A3%E7%B7%9A%E8%AA%AA%E6%8B%9C%E6%8B%9C-%E4%BD%BF%E7%94%A8-mobilenet-v1-%E6%A8%A1%E5%9E%8B%E8%88%87%E9%81%B7%E7%A7%BB%E5%AD%B8%E7%BF%92-10fb02da83e9)

## Setup

The following is needed in your Arduino IDE:

* [Arduino-ESP32 board support](https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json) (select ```Ai Thinker ESP32-CAM```)
* [Adafruit GFX Library](https://github.com/adafruit/Adafruit-GFX-Library)
* [Adafruit ST7735 and ST7789 Library](https://github.com/adafruit/Adafruit-ST7735-Library)
* [Import](https://docs.arduino.cc/software/ide-v1/tutorials/installing-libraries) the model library you've generated from Edge Impulse Studio
* Download [edge-impulse-esp32-cam](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/tree/main/edge-impulse-esp32-cam) from this repo and open the ```.ino``` file in the directory.

Be noted that you won't be able to read any serial output if you use Arduino IDE 2.0!

## Wiring

![pinout](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/raw/main/ESP32-CAM-pinout-new.png)

![wiring](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/raw/main/esp32-cam-edge-impulse.png)

For the ESP32-CAM, the side with the reset button is "up". The whole system is powered from a power module that can output both 5V and 3.3V. The ESP32-CAM is powered by 5V and TFT by 3.3V. I use a 7.5V 1A charger (power modules require 6.5V+ to provide stable 5V). The power module I use only output 500 mA max - but you don't need a lot since we don't use WiFi.

| USB-TTL pins | ESP32-CAM |
| --- | --- |
| Tx | GPIO 3 (UOR) |
| Rx | GPIO 1 (UOT) |
| GND | GND |

The USB-TTL's GND should be connected to the breadboard, not the ESP32-CAM itself. If you want to upload code, disconnect power then connect GPIO 0 to GND (also should be on the breadboard), then power it up. It would be in flash mode. (The alternative way is remove the ESP32-CAM itself and use the ESP32-CAM-MB programmer board.)

| TFT pins | ESP32-CAM |
| --- | --- |
| SCK (SCL) | GPIO 14 |
| MOSI (SDA) | GPIO 13 |
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

My demo model used Microsoft's [Kaggle Cats and Dogs Dataset](https://www.microsoft.com/en-us/download/details.aspx?id=54765) which has 12,500 cats and 12,500 dogs. 24,969 photos had successfully uploaded and split into 80-20% training/test sets. The variety of the images is perfect since we are not doing YOLO- or SSD- style object detection.

![下載](https://user-images.githubusercontent.com/44191076/154785876-b65de5e1-acba-4c2a-9c25-01d02e9b7a2b.png)

The model I choose was ```MobileNetV1 96x96 0.25 (no final dense layer, 0.1 dropout)``` with transfer learning. Since free Edge Impulse accounts has a training time limit of 20 minutes per job, I can only train the model for 5 cycles. (You can go [ask for more](https://forum.edgeimpulse.com/t/err-deadlineexceeded-ways-to-fix-this/2354/2) though...) I imagine if you have only a dozen images per class, you can try better models or longer training cycles.

Anyway, I got ```89.8%``` accuracy for training set and ```86.97%``` for test set, which seems decent enough.

![1](https://user-images.githubusercontent.com/44191076/153631673-96b90c0b-5745-43b9-9e5f-9a426d8bfe61.png)

Also, ESP32-CAM is not yet an officially supported board when I created this project, so I cannot use EON Tuner for futher find-tuning.

You can find my published Edge Impulse project here: [esp32-cam-cat-dog](https://studio.edgeimpulse.com/public/76904/latest). [ei-esp32-cam-cat-dog-arduino-1.0.4.zip](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/blob/main/ei-esp32-cam-cat-dog-arduino-1.0.4.zip) is the downloaded Arduino library which can be imported into Ardiono IDE.

The camera captures 240x240 images and resize them into 96x96 for the model input, and again resize the original image to 120x120 for the TFT display. The model inference time (prediction time) is 2607 ms (2.6 secs) per image, which is not very fast, with mostly good results. I don't know yet if different image sets or models may effect the result.

> Note: the demo model has only two classes - dog and cat - thus it will try "predict" whatever it sees to either dogs or cats. A better model should have a third class of "not dogs nor cats" to avoid invalid responses.

## Boilerplate Version

The [edge-impulse-esp32-cam-bare](https://github.com/alankrantas/edge-impulse-esp32-cam-image-classification/tree/main/edge-impulse-esp32-cam-bare) is the version that dosen't use any external devices. The model would be running in a non-stop loop. You can try to point the camera to the images and read the prediction via serial port (use Arduino IDE 1.x).

![bogdan-farca-CEx86maLUSc-unsplash](https://user-images.githubusercontent.com/44191076/153636524-9b2edab9-7c50-4aa1-9d6e-74477d67011f.jpg)

![richard-brutyo-Sg3XwuEpybU-unsplash](https://user-images.githubusercontent.com/44191076/153636561-16f7fb47-dcfc-4988-8772-85dcc5acfdac.jpg)
