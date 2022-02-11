# MobileNet Image Classification on ESP32-CAM and Edge Impulse (TinyML)

![41Ub2S0SjXL _AC_](https://user-images.githubusercontent.com/44191076/153631624-e13576b3-b440-4cd0-8a42-fd29cbe25a2d.jpg)

This is modified from [ESP32 Cam and Edge Impulse](https://github.com/edgeimpulse/example-esp32-cam) with simplified code and copied necessary libraries from Espressif's [esp-face](https://github.com/Yuri-R-Studio/esp-face).

(```esp-face``` had been changed a lot into [esp-dl](https://github.com/espressif/esp-dl) and thus broke the original example.)

This example is for Ai-Thinker's ESP32-CAM board, preferably with a ESP32-CAM-MB programmer board. This code does not output image via a web server, but you can try to point the camera to the images's direction and read the labels on serial output. Be noted that you won't be able to read anything if you use Arduino IDE 2.0!

## The Example Model - Cat & Dog

I used Microsoft's [Kaggle Cats and Dogs Dataset](https://www.microsoft.com/en-us/download/details.aspx?id=54765) which has 12,500 cats and 12,500 dogs. 24,969 photos had successfully uploaded and split into 80-20% training/test sets.

The model I choose was ```MobileNetV1 96x96 0.25 (no final dense layer, 0.1 dropout)```. Edge Impulse use transfer learning for its image classification models. Since in free accounts the training time has a limit of 20 minutes per job, I can only train the model for 5 cycles. You can go ask more on the forum though... Anyway, I got 89.8% accuracy for training set and 86.97% for test set, which seems to be decent enough.

![1](https://user-images.githubusercontent.com/44191076/153631673-96b90c0b-5745-43b9-9e5f-9a426d8bfe61.png)

Since ESP32-CAM is not an officially supported board, I cannot use EON Tuner to improve it further.

You can find my published project here: https://studio.edgeimpulse.com/public/76904/latest

```ei-esp32-cam-cat-dog-arduino-1.0.4.zip``` is the downloaded Arduino library which can be imported into yout Ardiono IDE.

The inference time is 2607 ms, which is not very fast,  with mostly good results. I don't know yet if different image sets or models may effect the result.

![bogdan-farca-CEx86maLUSc-unsplash](https://user-images.githubusercontent.com/44191076/153636524-9b2edab9-7c50-4aa1-9d6e-74477d67011f.jpg)

![richard-brutyo-Sg3XwuEpybU-unsplash](https://user-images.githubusercontent.com/44191076/153636561-16f7fb47-dcfc-4988-8772-85dcc5acfdac.jpg)
