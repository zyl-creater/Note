AVMUTE 是 HDMI 规范中的一个术语，表示"Audio-Video Mute"（音视频静音）。AVMUTE 通常与 HDMI 设备的音频和视频功能以及设备之间的通信相关。

AVMUTE 的主要功能是在 HDMI 设备之间通信，以控制音频和视频信号的静音（Mute）状态。这是有用的，因为当需要在 HDMI 设备之间传递信号时，可能需要在某些情况下临时关闭音频或视频。例如，当用户暂停视频播放时，视频源可以发送 AVMUTE 命令，将视频信号静音，同时保持音频信号开启。

AVMUTE 是通过 HDMI 控制命令和协议实现的，以确保各种 HDMI 设备可以协调并在需要时控制音频和视频的状态。具体的 AVMUTE 命令和信号可能会因 HDMI 设备的类型和制造商而有所不同，但通常包括音频和视频的静音/恢复状态指令。

如avmute介绍，可以理解为暂时停止音视频输出，等待合适的时候再输出音视频。该功能等同开关hdmi，该开关不会影响到上层Android display逻辑。

HDMI AVMute是HDMI TMDS数据岛周期General Control Packet中的一个标示。
AVMute字面意思是Audio Video Mute声音图像消隐，简单来说就是电视的声音至于静音状态、图像至于黑屏状态。
此信号是HDMI输出源（如DVD）发起的，传输给电视。为解决一些问题提出的方案。比如DVD在切换分辨率、Color Space、开关机等操作时，电视可能会看到一些过度花屏的现象，所以，DVD在进行相应操作之前，发送给电视一个AVMute信号，让电视至于黑屏静音状态，等待DVD切换好后，再发送一个Clear AVMute信号，让电视再开机，这样，切换过程中的花屏现象就会被屏蔽掉。

## 发送avmute
```
echo 1 > /sys/class/amhdmitx/amhdmitx0/avmute
```

## 清除avmute
```
echo -1 > /sys/class/amhdmitx/amhdmitx0/avmute
```

AVMUTE 的使用
对应的切点：/sys/class/amhdmitx/amhdmitx0/avmute
往其写入 1，表示进行 SET_AVMUTE
往其写入-1，表示进行 CLEAR_AVMUTE