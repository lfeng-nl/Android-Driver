# GPIO系统

GPIO硬件上存在各种差异；SOC中存在大量的引脚配置寄存器，这些寄存器按功能可以分为：
- 引脚功能和特性配置
- GPIO方向，电平控制
- 中断控制

## 如何通过软件抽象来掩盖赢家差异？
传统GPIO driver是负责上面三类的控制，新的Linux内核中的GPIO subsystem则用三个模块来对应上面三类硬件功能：
- （1）pin control subsystem，驱动pin controller硬件的软件子系统；
- （2）GPIO subsystem，驱动GPIO controller硬件的软件子系统；
- （3）GPIO interrupt chip driver，此模块是做为interrupt subsystem中的底层硬件驱动模块存在；

