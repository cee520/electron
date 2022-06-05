# HomeKit实验

{% hint style="info" %}
**Good to know:** depending on the product you're building, it can be useful to explicitly document use cases. Got a product that can be used by a bunch of people in different ways? Maybe consider splitting it out!
{% endhint %}

## ESP32

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
