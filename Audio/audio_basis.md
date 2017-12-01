# Audio basis
- 声音的基本属性：响度（Loudness），音调（Pitck），音色（Quality）
    - 响度：声音大小，跟声波的振幅有关；
    - 音调：跟声音的频率有关，频率越大，音调越高；
    - 音色：不同的发音材质表现出的效果的不同；

- 采样：采样深度、采样速率
    - 采样深度：采样是一个将连续信号离散化的过程，用最高多大的数来表示采样结果会影响到采样的准确性；一般为16bit，32bit；
    - 采样速率（频率）：采样速率越高，越能准确还原真是的波形；人耳所能听到的声音范围20-20KHz，所以一般采用44.1K、48K、96K的采样率；

- Nyquist–Shannon采样定律：（奈奎斯特-香农采样定律）当对被采样的模拟信号进行还原时，其最高频率只有采样率的一半；

- Weber-Fechner law：（韦伯-费希纳定律）`差别值 / 原先刺激量 = C`；也是就说，能引起感官变化的刺激差别量与原先的刺激量比值是恒定的，当刺激量较大时，需要更大的差别值才会引起感官上的同等刺激；改进：刺激强度以几何级增长，感知强度则以算术级数增加，`S = C×logR


## 常见缩写

- SRC：Sample-rate conversion，取样频率转换；

- AEC： Acoustic Echo Cancelation，回音消除

- NS：Noise Suppression，噪音抑制

- BVE：Bright Voice Enhancement，声音增强；

- FFP：Far-Field Pickup，远场拾音；

- BF：Beam-forming，波束成形；

- SLIMbus：Serial Low-power Inter-chip Media Bus 串行低功耗芯片间媒体总线，是基带或应用处理器和在移动终端的外围组件之间的标准接口。它是由ARM，诺基亚，意法半导体和德州仪器共同创建的MIPI联盟开发的。该接口同时支持许多数字音频组件，并以不同的采样率和位宽传输多个数字音频数据。