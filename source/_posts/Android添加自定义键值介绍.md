---
title: Android添加自定义键值介绍
date: 2016-05-16 17:47:57
tags: 自定义键值 KeyEvent
---
Android添加自定义键值介绍
在Android中，上层可使用的键值默认情况下是92个，从0-91；一般情况下，这些键值是够用的，但是如果想扩充的话，还是需要添加新的键值的，那么如何将一个新的键值从驱动的设置映射到上层，使应用可以对我们自定义的键值进行相应的处理呢？
<!--more-->

** 接下来我们一步一步进行自定义键值的添加**
根据功能需求，我们需要自定义一个键值，用来实现特定功能。
举例如下：
客户需要实现灭屏双击唤醒屏幕功能
灭屏时，双击屏幕，BSP驱动同事上传一个自定义的Linux键值
经过键值转换，将Linux码键转换为上层KeyEvent中自定义的键值
获取到经过转换后的键值，完成唤醒操作
上述中简单的介绍了一个自定义键值，从底层驱动触发、上传、转换到被用户使用的一个流程，接下来我们讲讲具体的实现。
## 涉及到的文件
``` java
kernel\include\uapi\linux\Input.h ----------------------------------- 底层Linux码值定义
kernel\drivers\input\touchscreen\atmel\Atmel_mxt_ts.c ---- 底层Linux码值上报
framework\base\core\java\android\view\KeyEvent.java ------ framework层Android键值定义
frameworks\base\core\res\res\values\attrs.xml --------------- framework层Android键值资源定义
frameworks\base\data\keyboards\Generic.kl ------------------ Linux码值转换成Android键值转换表
frameworks\native\include\android\keycodes.h -------------- Android自定义键值nativ声明
frameworks\native\include\input\InputEventLabels.h ------- Android自定义键值nativ声明
frameworks\base\policy\src\com\android\internal\policy\impl\PhoneWindowManager.java -------- 本例中处理自定义键值的地方
```

## 实现步骤及细节
根据上述的文件，我们依次介绍下如何添加。

    kernel\include\uapi\linux\Input.h
这个文件是kernel定义Linux码值的地方，我们这里我们可以看到很多类似“#define KEY_*** ###”的定义，我们做如下定义：

	#define KEY_DOUBLE_TAP 195
这个"DOUBLE_TAP"命名要从下至上保持一致，且195这个数字要没有被定义过。

	2.kernel\drivers\input\touchscreen\atmel\Atmel_mxt_ts.c
这个文件主要是对上面的码值进行引用，具体细节上如何操作，可询问BSP同事，这里不再描述。
上述两个文件修改完成后，编译bootimage，推送到手机即可验证。接下来我们讲讲Framework层涉及到的修改。

	3.framework\base\core\java\android\view\KeyEvent.java
Android的所有键值都是在这个地方有相应的定义的，所以我们也需要添加我们的修改到这个文件。
在完成本例时，Android的最后一个键值为“HELP”,数值是259，所以我们在此基础上定义我们的自定义键值的数值大小为260。

{% img [class names] /img/keyevent/KeyEvent_1.jpeg %}
从上图中我们可以看见，在我们添加260数值下面有一段注释，这个就是添加一个自定义键值在Framework层需要做的修改。
其中isSystem(),isWakeKey()并不是必改项，可以根据需要添加，其中修改了isWakeKey()后，每次有改键值上报上来，系统都会自动唤醒屏幕。
接下来我们根据这个注释依次修改相应文件，基本上就是我们之前提到的涉及到的文件。

	4.frameworks\native\include\android\keycodes.h
这个文件的修改如下：
{% img [class names] /img/keyevent/KeyEvent_2.jpeg %}
	5.frameworks\native\include\input\InputEventLabels.h
这个文件的修改如下：
{% img [class names] /img/keyevent/KeyEvent_3.jpeg %}
	6.frameworks\base\core\res\res\values\attrs.xml
Android键值资源声明：
{% img [class names] /img/keyevent/KeyEvent_6.jpeg %}
到此为止，自定义键值的底层码值定义、上层键值声明都已全部完成，此时我们还不能在上层获取到正确的键值，因为我们还差最重要的一步：Linux码值转换为Android键值。
这个转换要如何实现呢？Android中其实是存在一张键值转换表的，根据相同的键值命名，把Linux码值与Framework层的键值一一对应起来。点我了解更多

	7.frameworks\base\data\keyboards\Generic.kl
此文件为本例项目中使用的键值转换表，在表中做如下添加：
{% img [class names] /img/keyevent/KeyEvent_4.jpeg %}

195为我们在Linux下定义的码值，对应我们Framework中声明的“DOUBLE_TAP”键。

	8.frameworks\base\policy\src\com\android\internal\policy\impl\PhoneWindowManager.java
本例中，我们最终获取并使用自定义键值的地方是在PhoneWindowManager的interceptKeyBeforeQueueing()方法中：
{% img [class names] /img/keyevent/KeyEvent_5.jpeg %}

## 编译及验证

在第三部分的添加修改完成后， 我们需要对应编译相应的模块并将生成的东西push到手机中验证。
对于kernel中的修改，编译bootimage后fastboot boot.img即可
对于Framework里面的编译对应模块将其生成的产物放到对应目录下重启即可
[*]: 如果framework里面模块编译后，对应push到手机后无效果，可以试试make snod将重新打包的system.img fastboot到手机。
对于Generic.kl文件，看其模块make文件可知，其执行的操作就是将Generic.kl文件拷贝到/system/usr/keylayout目录下，
所以我们可以直接将修改好的文件push到手机：adb push Generic.kl /system/usr/keylayout



