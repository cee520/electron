# ESP

{% hint style="info" %}
**Good to know:** Splitting your product into fundamental concepts, objects, or areas can be a great way to let readers deep dive into the concepts that matter most to them.
{% endhint %}

## 环境搭建

参考 [https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html)



MacOS

```
sudo easy_install pip
brew install cmake ninja dfu-util
git clone --recursive https://github.com/espressif/esp-idf.git -b v4.4.1

cd ~/esp/esp-idf
./install.sh <esp32 esp32s2> #setup  tools
source $HOME/esp/esp-idf/export.sh    #setup enviroment 
```

下载错误就用wget手动下载&#x20;

## HomeKit实验

[https://github.com/espressif/esp-apple-homekit-adk](https://github.com/espressif/esp-apple-homekit-adk)

#### Set up MbedTLS

```
cd $IDF_PATH/components/mbedtls/mbedtls
$ git pull
$ git checkout -b mbedtls-2.16.6-adk origin/mbedtls-2.16.6-adk
```

#### Get esp-apple-homekit-adk

```
git clone --recursive https://github.com/espressif/esp-apple-homekit-adk.git
```

#### Compile and Flash

```
$ cd /path/to/esp-apple-homekit-adk/examples/Lightbulb
$ export ESPPORT=/dev/tty.SLAB_USBtoUART #Set your board's serial port here
$ idf.py set-target esp32
$ idf.py menuconfig # Set Example Configuration -> WiFi SSID/Password
$ idf.py flash
$ esptool.py -p $ESPPORT write_flash 0x340000 accessory_setup.bin
$ idf.py monitor
```
