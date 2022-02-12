# MobileNet Image Classification on ESP32-CAM and Edge Impulse (TinyML)

![41Ub2S0SjXL _AC_](https://user-images.githubusercontent.com/44191076/153631624-e13576b3-b440-4cd0-8a42-fd29cbe25a2d.jpg)

This example is specifically for Ai-Thinker's ESP32-CAM board, preferably with a ESP32-CAM-MB programmer board (or use an external USB-TTL module, see [here](https://randomnerdtutorials.com/program-upload-code-esp32-cam/)). You also need to install [Arduino-ESP32](https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json) board support and choose ```Ai Thinker ESP32-CAM```.

This is modified from [ESP32 Cam and Edge Impulse](https://github.com/edgeimpulse/example-esp32-cam) with simplified code and copied necessary libraries from Espressif's [esp-face](https://github.com/Yuri-R-Studio/esp-face). ```esp-face``` had been changed a lot into [esp-dl](https://github.com/espressif/esp-dl) and thus broke the original example. The original example also requires WiFi and the user has to refresh the webpage to classify a new image.

> See the original example repo or [this article](https://www.survivingwithandroid.com/tinyml-esp32-cam-edge-image-classification-with-edge-impulse/) about how to generate your own model on Edge Impulse. You can also still run the original example by copy every libraries in this example to the project directory, then re-open the .ino script.

This example does not output image via a web server, it can be modified to run in anyway you like, as long as you can point the camera to the images's direction. Be noted that you won't be able to read anything in the serial port monitor if you use Arduino IDE 2.0!

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
