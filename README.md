# 电子实验

{% hint style="info" %}
**Good to know:** providing a brief overview of your product and its core use cases is a great place to start with product docs. Your product might seem obvious to you – you made it! However, to others, even folks who are trying your product after reading your site or getting a sales demo, it can still be unclear. This is your chance to clarify your product and set the right expectations!
{% endhint %}

Apple HomeKit

[https://github.com/RavenSystem/esp-homekit-devices/wiki](https://github.com/RavenSystem/esp-homekit-devices/wiki)&#x20;

> Loom is a video messaging tool that helps you get your message across through instantly shareable videos.
>
> With Loom, you can record your camera, microphone, and desktop simultaneously. Your video is then instantly available to share through Loom's patented technology.
>
> — From the [Loom Docs](https://support.loom.com/hc/en-us/articles/360002158057-What-is-Loom-)

## MCU对比

| 芯片      | 指令                       |                | 功耗                                                   |
| ------- | ------------------------ | -------------- | ---------------------------------------------------- |
| ESP32   | 32RISC 160-240Mhz  2Core | 自带2.4G Wifi 蓝牙 | 活动：80-240mA ,调制解调睡眠:3-20mA,轻度睡眠 0.8mA， 深度睡眠 10-150uA |
|         |                          |                |                                                      |
| ESP8266 | 32RISC 80Mhz 1Core       |                |                                                      |
| MSP430  | 16RISC 1-8Mhz 1Core      |                | 活动:200-400uA,睡眠:0.1uA                                |
| STM32   | 32ARM 8-72Mhz            |                |                                                      |
|         |                          |                |                                                      |

## &#x20;ESP8266

功耗

> T[https://www.yiboard.com/thread-1550-1-1.html#:\~:text=为什么ESP8266功耗很重要？%20ESP8266的功耗在15µA至400mA之间，具体取决于不同的使用情况%E3%80%82,在使用WiFi的空闲状态下，NodeMCU%20V2的电流消耗约为70mA%E3%80%](https://www.yiboard.com/thread-1550-1-1.html)

ESP8266的功耗在15µA至400mA之间，具体取决于不同的使用情况。

在使用WiFi的空闲状态下，NodeMCU V2的电流消耗约为70mA。在3.3V的工作电压下，NodeMCU V2在空闲状态下需要以下功耗：

_W= U \* I = 3.3V \* 70mA = 231mW_

因此，NodeMCU每年的能耗为：E = W \* 365 \* 24 = 2.024 kWh

假设每千瓦时的成本约为0.8元。如果我想用一个NodeMCU建造一个检测站，那么每年的电费将超过1.6元，几乎忽悠不计。那么，为什么需要关注功耗呢？如何设置才能将每年的成本降至低于1元？

ESP8266具有几种睡眠模式以及与这些模式相关的电源解决方案。总共有三种不同的睡眠模式。下表的来源是ESP8266 Espressif生产商的手册。

|              | 调制解调器睡眠 | 轻度睡眠    | 深度睡眠        |
| ------------ | ------- | ------- | ----------- |
| Wi-Fi        | OFF     | OFF     | OFF         |
| 系统时钟         | ON      | OFF     | OFF         |
| RTC          | ON      | ON      | ON          |
| CPU          | ON      | 挂起      | OFF         |
| 基板电流         | 15 mA   | 0.4 mA  | 20 uA       |
| 平均电流DTIM = 1 | 16.2 mA | 1.8 mA  | <p><br></p> |
| 平均电流DTIM = 2 | 15.4 mA | 0.9 mA  | <p><br></p> |
| 平均电流DTIM = 3 | 15.2 mA | 0.55 mA |             |

\
**轻度睡眠**

轻度睡眠模式与调制解调器睡眠模式非常相似。仅有的两个区别是：

1\.  ESP8266系统时钟的电源

2\.  内部CPU处于挂起状态

因此，开发板将不会响应来自其他硬件接口的信号和中断。 ESP8266需要从外部GPIO唤醒。唤醒过程需要少于3ms。通过GPIO唤醒ESP8266的接口为：void gpio\_pin\_wakeup\_enable(uint32 i, GPIO\_INT\_TYPE intr\_state);

这两个差异导致功耗从15mA大幅下降至0.4mA。

以下示例将NodeMCU连接和断开与本地WiFi的连接，并将睡眠模式设置为浅睡眠。

```c
void setup() {
  Serial.begin(115200);
}
void loop() {
  // Disable light sleep
  wifi_set_sleep_type(NONE_SLEEP_T);
  delay(2000);
  // Enable light sleep
  wifi_set_sleep_type(LIGHT_SLEEP_T);
  delay(2000);
}
```

**深度睡眠**

如果要使用外部电源并节省尽可能多的能量，请进入深度睡眠模式！从理论上讲，ESP8266的功耗降至20µA \* 3.3V = 66µW。您的1000mAh外接电池续航时间更长。

T = 1000mAh / 20µA = 50,000h = 2083天= 5.7年

与其他两种模式不同，系统无法自动进入深度睡眠模式。用户可以调用接口函数system\_deep\_sleep立即启用深度睡眠。在这种模式下，芯片将关闭Wi-Fi连接和数据连接。只有RTC模块仍在工作，负责定期唤醒。

要启用深度睡眠模式，您只需要使用以下代码行：ESP.deepSleep(uint32 time\_in\_us);在此处定义一个时间，ESP8266从深度睡眠模式醒来的时间。深度睡眠的最大时间为4,294,967,295 µs，约为71分钟。

在定义的时间之后，ESP8266将被来自D0引脚（GPIO16）的复位脉冲唤醒，该引脚被定义为唤醒线路。因此，您必须将D0与RST物理连接。

下面图片显示了如何在深度睡眠阶段之后将D0引脚与RST连接以唤醒ESP8266 NodeMCU和WeMos D1 Mini微控制器。

{% content-ref url="guides/creating-your-first-project.md" %}
[creating-your-first-project.md](guides/creating-your-first-project.md)
{% endcontent-ref %}

{% content-ref url="guides/creating-your-first-task.md" %}
[creating-your-first-task.md](guides/creating-your-first-task.md)
{% endcontent-ref %}

{% content-ref url="guides/advanced-permissions.md" %}
[advanced-permissions.md](guides/advanced-permissions.md)
{% endcontent-ref %}

{% hint style="info" %}
**Good to know:** your product docs aren't just a reference of all your features! use them to encourage folks to perform certain actions and discover the value in your product.
{% endhint %}

### 大众CD端子

![](<.gitbook/assets/截屏2022-05-25 17.51.26.png>)

n the fundamentals of MyProduct to get a deeper understanding of our main features:

{% content-ref url="fundamentals/projects.md" %}
[projects.md](fundamentals/projects.md)
{% endcontent-ref %}

{% content-ref url="fundamentals/members.md" %}
[members.md](fundamentals/members.md)
{% endcontent-ref %}

{% content-ref url="fundamentals/task-lists.md" %}
[task-lists.md](fundamentals/task-lists.md)
{% endcontent-ref %}

{% content-ref url="fundamentals/tasks.md" %}
[tasks.md](fundamentals/tasks.md)
{% endcontent-ref %}

{% hint style="info" %}
**Good to know:** Splitting your product into fundamental concepts, objects, or areas can be a great way to let readers deep dive into the concepts that matter most to them. Combine guides with this approach to 'fundamentals' and you're well on your way to great documentation!
{% endhint %}
