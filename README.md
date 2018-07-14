# meeting-assistant

***Volume bars***
白色的音量显示器，图像化音量大小。当音量超过警示音时，音量条会变成红色。

***Basic Setup Panel***  
Threshold: 通过输入不同分贝数字/使用"+","-"按钮调整警示音量上限  
Warning: 是否在超过警示音量时播放提示声音（关闭则为静音）  
Trigger Mode: 本应用共有两种模式: trigger mode & smooth mode.   

- Trigger mode: 当环境音量持续大于警示音量至少0.3s后，会播放提示声音直到环境音持续低于警示音量至少0.3s。
- Smooth mode: 只要环境音量大于警示音量，应用就会播放提示音；提示音的音量大小由（环境音量 - 警示音量）决定。
- Trigger mode默认为off, 所以默认模式是smooth mode。
