# ESP

{% hint style="info" %}
**乐鑫芯片**
{% endhint %}

## 环境搭建

参考[https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html)



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

Windows

```
#powershell
./install.ps1
./export.ps1

$env:ESPPORT=COM3
```

## 呼吸灯



## 中断按键

{% embed url="https://blog.csdn.net/qq_38113006/article/details/105450618" %}
参考
{% endembed %}

{% embed url="https://blog.csdn.net/qq_38113006/article/details/105475390" %}

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

**复位配对：**

附件配对信息存储在NVS（非易失性存储）分区中。一旦配对，附件将无法再次配对，除非先清除此配对信息。可以通过以下方式完成：

```bash
$ esptool.py -p $ESPPORT erase_region 0x10000 0x6000
```

**学会使用HomeKit**

例子：[https://github.com/HanChenen/esp32-homekit](https://github.com/HanChenen/esp32-homekit)&#x20;

hap.h是重中之重。 它含有三个枚举、两个结构体和若干个函数。这三个枚举分别是配件、服务和特性。配件类型有19个，有开关、风扇和车库等等。服务有35个，相比于配件就划分的更细了，主要是对传感器做了更多的扩展。特性有106个，有名字，状态，大小等等。这三者的关系是配件>服务>特性，比如SWITCH例程中的注册流程是这样的，首先注册一个HAP\_ACCESSORY\_CATEGORY\_OTHER附件，然后再给其添加服务类型HAP\_SERVICE\_ACCESSORY\_INFORMATION和HAP\_SERVICE\_SWITCHS，第一个服务类型是配件信息，用来说明一些必要的产品信息，如生产商，型号，版本号等等，总共六项，这六项就是第三个枚举中列出的特性；第二个是开关服务，开关只用了一个特性来描述，主要描述开关状态。 我们回到hap.h文件中来，继续看结构体。&#x20;

ESP32 能耗

deepsleep 12mA,GIPO推挽电流40mA  &#x20;

| 状态            |    能耗 |    功率 |
| ------------- | ----: | ----: |
| Deep Sleep    |  12mA | 0.06W |
| wifi search   | 120mA |       |
| WiFi 稳定工作     |  40mA |       |
| AOD4184关 LED  | 0.8mA |       |
| AOD4184关 100欧 |   2mA |       |
| AOD4184开 LED  |  16mA |       |
|               |       |       |
|               |       |       |

AOD4184 d极串LED+100欧，

先电阻后Led&#x20;

5V->1K 4.61V-> Led 1.8V->NMOSds 0V, led 0.1mA pass 2mA

5V->NMOSdg 1.38V ->47K->0V

先Led后电阻

5V->Led 3.1V->1K 1.6V->NMOSds 0V

5V->NMOSdg->1.52V->10K->0V

## 灯控制器

需求：

记忆功能

开关控制电源，可控3档

语言可控每盏灯1-8号，A组黄白，B组黄白

语言有情景模式

自动工作模式（根据时段变光）

无人自动关闭（30分钟）

```
电子侧B    电脑侧A
5||6       7||8
4||3       2||1

```
