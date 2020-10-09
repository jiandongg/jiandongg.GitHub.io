## 问题：思博伦标识包大小=wireshark包大小+4byte

## 原因：
wireshark无法捕获数据包的fcs信息，因为这个信息的数据包在网卡校验成功之后，便被抹去了（因操作系统而定），保证了网络数据包一致性核上层应用的简便性。

思博伦关于数据包FCS字段的说明：
```
How can I capture entire frames, including the FCS?
Wireshark can only capture data that the packet capture library - libpcap on UNIX-flavored OSes, and the Npcap port to Windows of libpcap on Windows - can capture, and libpcap/Npcap can capture only the data that the OS’s raw packet capture mechanism (or the Npcap driver, and the underlying OS networking code and network interface drivers, on Windows) will allow it to capture.
For any particular link-layer network type, unless the OS supplies the FCS of a frame as part of the frame, or can be configured to do so, Wireshark - and other programs that capture raw packets, such as tcpdump - cannot capture the FCS of a frame. You will have to determine whether your OS needs to be so configured and, if so, can be so configured, configure it if necessary and possible, and make whatever changes to libpcap and the packet capture program you’re using are necessary, if any, to support capturing the FCS of a frame.
Most OSes do not support capturing the FCS of a frame on Ethernet, and probably do not support it on most other link-layer types. Some drivers on some OSes do support it, such as some (all?) Ethernet drivers on NetBSD and possibly the driver for Apple’s gigabit Ethernet interface in macOS; in those OSes, you might always get the FCS, or you might only get the FCS if you capture in promiscuous mode (you’d have to determine which is the case).
Versions of Wireshark prior to 0.9.15 will not treat an Ethernet FCS in a captured packet as an FCS. 0.9.15 and later will attempt to determine whether there’s an FCS at the end of the frame and, if it thinks there is, will display it as such, and will check whether it’s the correct CRC-32 value or not.
https://www.wireshark.org/faq.html#q6.10
```

结论：真实包大小=思博伦包大小，wireshark包大小+4byte的fcs=真实包大小

## 发现：思博伦的流的edit取消勾选 include signature field之后，数据包大小可以更小。这样应该是取消了Spirent signature field

### 附录：发送过小包时的思博伦信息：
```
10/9/2020 11:19:54 .4 	Error 	Other 	Start Traffic 4 failed with error: There were 1 validation error(s); aborting apply.
Stream block (StreamBlock 4):  Frame length (64 bytes) is not large enough to include header and Spirent signature field and FCS. At least 78 bytes are needed..
```
