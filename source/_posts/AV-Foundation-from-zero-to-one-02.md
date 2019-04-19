---
title: AV Foundation 从0到1-播放和录制音频
date: 2017-05-27 16:30:52
categories:
 - 技术
tags:
 - AV Foundation
---

## 播放和录制音频

AV Foundation定义了7种分类来描述应用程序所使用的音频行为：

| 分类               | 作用               | 是否允许混音 | 音频输入 | 音频输出 |
| ---------------- | ---------------- | ------ | ---- | ---- |
| Ambient          | 游戏、效率应用程序        | √      |      | √    |
| Solo Ambient（默认） | 游戏、效率应用程序        |        |      | √    |
| Playback         | 音频和视频播放器         | 可选     |      | √    |
| Record           | 录音机、音频捕捉         |        | √    |      |
| Play and Record  | VoIP、语音聊天        | 可选     | √    | √    |
| Audio Processing | 离线会话处理           |        |      |      |
| Multi-Route      | 使用外部硬件的高级A/V应用程序 |        | √    | √    |

所有的iOS应用程序都自动带有一个默认的音频会话，分类名称为Solo Ambient。

针对音频的播放和录制，一共有两个比较重要的类 `AVAudioPlayer` 和 `AVAudioRecorder` 。

### AVAudioPlayer

> 可以播放本地的音频文件，不可以用来播放网络音频

#### 播放中的控制

**修改播放器的音量：**播放器的音量独立于系统的音量，可以对 AVAudioPlayer 的 volume 属性值进行0.0（静音）~1.0（最大音量）的设置。

**修改播放器的Pan值：**我们可以对 AVAudioPlayer 的 pan 属性值进行-1.0（极左）~1.0（极右）的设置，例如你在耳机中听到一辆车从你的左侧飞驰到右侧，那么就是这个值在起作用了。

**调整播放率：**我们可以对 AVAudioPlayer 的 rate 属性值进行0.5（半速）~2.0（2倍速）的设置，即加速播放和减速播放。

**调整循环模式：**我们可以对 AVAudioPlayer 的 numberOfLoops 值进行任意整数的设置，该属性值如果设置为大于0的数时则可以实现n此的循环播放，如果设置为-1则会导致无限循环。

**音频计量：**我们可以通过设置 AVAudioPlayer 的 meteringEnabled 属性值的YES或NO，来决定是否读取当期播放的音频的level metering。

等等...

前面我们说过，所有的iOS应用默认的音频会话分类为 Solo Ambient ，那么，很显然如果我们要做一款基于音频的应用的话，默认的分类是满足不了我们的要求的。所以我们最好在 didFinishedLaunch 方法中设置音频会话类型。即 AVAudioSessionCategoryXXX。然后记得激活会话。

其次如果我们需要在应用程序在后台运行的时候依然可以播放音频的话，我们需要在应用程序的 Info.plist 文件中添加一个新的 Required background modes 类型的数组，然后在数组中添加一个 App plays audio or streams audio/video using AirPlay 的选项。

#### 处理中断事件

有时候我们在播放音频的时候可能会一些事情打断，比如说有电话呼入、有闹钟响起、有微信视频邀请等等...所以，处理中段事件时必须的，我们需要确保当前的音频播放被打断之后最好有一种比较友好的响应方式。

首先，我们要注册中断通知 `AVAudioSessionInterruptionNotification` ，通过判断 notification.userInfo 中的 AVAudioSessionInterruptionTypeKey 所对应的键值来确认当前的中断操作是开始还是结束。然后需要根据对应的操作来做出相应的处理。

#### 处理线路改变

假如说我们正在用手机播放音频，这时如果我插入了耳机那证明我这时候想切换到耳机来播放音频。反之，如果我正在使用耳机播放音频，这时我断开耳机的连接之后预期的结果应该是暂停播放。

同样，系统默认并不会帮我们做这些操作，这时需要注册线路变化通知 `AVAudioSessionRouteChangeNotification` ，该通知的userInfo字典中包含线路被切换的原因的 AVAudioSessionRouteChangeReasonKey 值。该值代表一个无符号的整数。

那么耳机断开这个事件对应的原因为 AVAudioSessionRouteChangeReasonOldDeviceUnavailable 。事实上，从这个原因的字面意思可以看出，这个通知其实告诉监听者的是旧的设备不可用而导致的线路改变，这也就是说其实耳机断开的事件相对于这个通知来说是充分不必要条件，什么意思呢，就是耳机断开会导致 AVAudioSession 发送该通知，但 AVAudioSession 发出该通知并不一定是因为耳机被断开。所以，你懂了？

这样的话，我们在接收到该通知之后，必须再次向 notification.userInfo 提出请求，以获得其中用于描述前一个线路的 AVAudioSessionRouteDescription 。该描述信息整合了一个输入的NSArray和一个输出的NSArray。数组中的元素都是 AVAudioSessionPortDescription 的实例，用于描述不同的I/O接口属性。

在AVFoundation框架的AVAudioSession.h文件中详细定义了 `input port types` 和 `output port types` 以及同时代表input和output的 `port types that refer to either input or output` 各种类型。

整个处理逻辑：

```objc
- (void)handleRouteChange:(NSNotification *)notification {
    NSLog(@"%s %@", __func__, notification.userInfo);

    NSDictionary *info = notification.userInfo;
    AVAudioSessionRouteChangeReason reason = [info[AVAudioSessionRouteChangeReasonKey] unsignedIntegerValue];

    if (reason == AVAudioSessionRouteChangeReasonOldDeviceUnavailable) {
        AVAudioSessionRouteDescription *previousRoute = info[AVAudioSessionRouteChangePreviousRouteKey];
        AVAudioSessionPortDescription *previousOutput = previousRoute.outputs[0];
        NSString *portType = previousOutput.portType;

        if ([portType isEqualToString:AVAudioSessionPortHeadphones]) {
            [self stop];
            if (self.delegate) {
                [self.delegate playbackStopped];
            }
        }
    }
}
```

### AVAudioRecorder

相对于 AVAudioPlayer 的兄弟类。 AVAudioRecorder 可以使用设备内置的麦克风或外部音频设备（比如数字音频接口或USB麦克风等）进行录制。

创建 AVAudioRecorder 实例时需要为其提供数据的一些信息，分别是：

* 用于表示音频流写入文件的本地URL。
* 包含用于配置录音会话键值信息的NSDictionary对象。
* 用于捕捉初始化阶段各种错误的NSError指针。

同样，在初始化完成之后，立即调用 prepareToRecord 方法，会将录制启动时的延时降到最小。

在上边第二条，配置录音会话键值信息的NSDictionary对象这一步中的键值信息也值得讨论一番。我们只简要的来看一些基本的键值信息。

**音频格式：**AVFormatKey

**采样率：**AVSampleRateKey 标准的采样率一般有8000、16000、22050或44100几种。

**通道数：**AVNumberOfChannelKey 指定默认值1意味着单声道录制，设置2意味着使用立体声录制。除非使用外部硬件进行录制，否则通常应该创建单声道录音。

**位深度：**AVEncoderBitDepthHintKey 可以理解为计算机声卡处理声音的解析度。这个数值越大，解析度就越高，录制和回放的声音就越真实。

等等...

先回到创建 AVAudioRecorder 实例时我们提供的音频流的本地URL，一般改URL最好是在tmp文件夹。在录制完成之后，通常将该文件复制到Documents文件夹中保存。音频录制同样支持暂停，调用 AVAudioRecorder 的pause方法即可暂停录制，再次调用record方法即可继续录制。通常我们在录制完成时最好能给用户一个友好的提示告诉他录制完成，并请求用户完成保存前的一些操作（命名）。

相对于音频播放来说，音频录制可能要说的东西并没有很多。接下来我们看一下Audio Metering。

### Audio Metering

Audio Metering 堪称 AVAudioRecorder 和 AVAudioPlayer 中最强大和最实用的功能。Audio Metering 可以让开发者读取音频的平均分贝和峰值分贝数据。开发者可以根据这些数据以可视化的方式将声音的大小波动效果呈献给用户。

两个类使用的方法都是 averagePowerForChannel: 和 peakPowerForChannel: 。两个方法都会返回一个表示分贝(dB)等级的浮点值。这个范围从表示最大分贝的0dB(fullscale)到表示最小分贝或静音的-160dB。