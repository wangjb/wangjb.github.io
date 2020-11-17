---
layout: post
title: 在ARM Cortex-M上部屬KWS應用
---

{% include image.html url="/assets/img/mcu_kws_yes.jpg" %}

如何訓練模型是一件事，在現實場景中應用模型又是另一回事。物聯網科技透過IOT技術連結各種電子儀器，若再賦予其AI演算法就朝智慧物聯網邁進一步了，而微控制器(Microcontroller)則是物聯網應用所乘載的關鍵實體元件之一，如何將訓練好的AI演算法有效部屬在MCU上，關乎實際運行時的效能。

ARM這篇2017年文章[Hello Edge: Keyword Spotting on Microcontrollers](https://arxiv.org/abs/1711.07128)以部屬KWS(語音喚醒)技術在MCU為例，探討在MCU上部屬AI演算法時可能遭遇到資源侷限與運算速度等實際問題，也對當時的KWS技術做了一些review，是可以學習到綜觀觀點的一篇優質文章。

這裡我嘗試依照ARM這篇教學文件[Build Arm Cortex-M voice assistant with Google TensorFlow Lite](https://developer.arm.com/solutions/machine-learning-on-arm/developer-material/how-to-guides/build-arm-cortex-m-voice-assistant-with-google-tensorflow-lite/single-page)，在colab上將預先訓練好的KWS模型部屬在STM32f746NG開發版上。

## 準備開發環境

> 安裝gcc-arm與相關套件

```bash
!sudo apt-get -qq update -y
!sudo apt-get -qq install -y xxd
!sudo pip -q install --upgrade mbed-cli
!sudo apt-get -qq install mercurial
!sudo apt-get -qq install -y gcc-arm-none-eabi
```

> 下載tensorflow repo

```bash
!git clone https://github.com/tensorflow/tensorflow.git
```

## 使用 mbed-cli + gcc_arm 編譯部屬

> 產出編譯MCU所需相關原始碼

```bash
%cd tensorflow
!make -f tensorflow/lite/micro/tools/make/Makefile TARGET=mbed TAGS="disco_f746ng" generate_micro_speech_mbed_project
```

> 下載MCU所需相關libray

```bash
%cd tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/micro_speech/mbed
!mbed config root .
!mbed deploy
```

> 使用gcc-arm-none-eabi開始編譯相關檔案

```bash
!mbed compile -m DISCO_F746NG -t GCC_ARM 
```

> 部屬執行檔到MCU，看看成果如何

```bash
cp ./BUILD/DISCO_F746NG/GCC_ARM/mbed.bin /你的MCU/
```

{% include image.html url="/assets/img/mcu_kws_naive.jpg" description="LCD顯示MCU的KWS偵測結果一直為'unknown'。:(" %}

實際執行發現，即使在沒有什麼背景噪音的情況下，會一直產出"Heard unknown"的訊息，而非應該產出的"Heard silence"，可能是演算法過於sensitive，又或者是MCU的的microphone有問題!? 而且必須靠近MCU版的microphone念出"no"字才有機會偵測到，並且偵測成功率很差，keyword "yes"則更慘，幾乎沒成功偵測幾個，之後debug方向，要嘛是MCU的micro有問題，要嘛是用的model真的太爛...

## 另一個安裝版本...

後來發現arm教學文件似乎參考的是[這個github](https://github.com/uTensor/tf_microspeech)，照著readme.md上的說明也的確可以產出執行檔，部屬到MCU上後，發現反而才是教學文件中所使用的code...

> 一樣安裝相關套件

```bash
!sudo apt-get -qq update -y
!sudo apt-get -qq install -y xxd
!sudo pip -q install --upgrade mbed-cli
!sudo apt-get -qq install mercurial
!sudo apt-get -qq install -y gcc-arm-none-eabi
```
> import repo

```bash
!mbed import https://github.com/uTensor/tf_microspeech
```

> 編譯、部屬

```bash
%cd tf_microspeech/
!mbed compile -m DISCO_F746NG -t GCC_ARM --profile=tflm/tensorflow/lite/build_profiles/release.json 
```

> 這個版本中，偵測結果訊號只走usb回傳，LCD螢幕不會顯示偵測結果，可以用gtkterm以baud rate 9600讀取

{% include image.html url="/assets/img/mcu_kws_msg.png" description="透過usb讀取MCU回傳KWS模型偵測結果。" %}

結果這個版本的偵測效果非常的好！輕聲說出yes或no也能偵測的出來 :D 看來可能是model的問題比較大，但因為是部屬到mcu上，從聲音訊號的擷取到最後模型偵測結果，中間任何library使用錯誤都可能導致結果不如預期，但至少可以確定的是，我的MCU上的microphone沒有問題 :D

{% include image.html url="/assets/img/mcu_kws_yes.jpg" description="透過usb讀取MCU回傳KWS模型偵測結果'yes'。" %}
{% include image.html url="/assets/img/mcu_kws_no.jpg" description="透過usb讀取MCU回傳KWS模型偵測結果'no'。" %}

> 若要加上螢幕顯示偵測結果可以參考ARM教學文件章節"Extend the program"去修改程式碼`command_responder.cc`，並將LCD相關libray與header複製到目錄`tf_microspeech`底下

```bash
%cp -rf /content/tensorflow/tensorflow/lite/micro/tools/make/gen/mbed_cortex-m4/prj/micro_speech/mbed/LCD_DISCO_F746NG* /content/tf_microspeech/.
```

> 重新編譯後部屬即可！

```bash
%cd /content/tf_microspeech/
!mbed config root .
!mbed compile -m DISCO_F746NG -t GCC_ARM --profile=tflm/tensorflow/lite/build_profiles/release.json  -v
```


enjoy! :)

