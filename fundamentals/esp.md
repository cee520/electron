# ESP

{% hint style="info" %}
**Good to know:** Splitting your product into fundamental concepts, objects, or areas can be a great way to let readers deep dive into the concepts that matter most to them.
{% endhint %}

环境搭建

参考 [https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/linux-macos-setup.html)



MacOS

```
sudo easy_install pip
brew install cmake ninja dfu-util
git clone --recursive https://github.com/espressif/esp-idf.git

cd ~/esp/esp-idf
./install.sh <esp32 esp32s2> #setup  tools
. $HOME/esp/esp-idf/export.sh    #setup enviroment 
```

下载错误就用wget手动下载&#x20;
