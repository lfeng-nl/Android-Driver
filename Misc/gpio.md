# 本文介绍Linux gpio相关知识
## 内核中gpio的使用
- 测试gpio是否合法`bool gpio_is_valid(int gpio);`
- 申请某个gpio端口，`int gpio_request(unsigned gpio, const char *label)`；需要gpio号和名字
- 标记gpio的使用方向，包括输入还是输出；`int gpio_direction_input(unsigned gpio); int gpio_direction_output(unsigned gpio, int value);`
- 获得gpio引脚的值或设置gpio引脚的值：`int gpio_get_value(unsigned gpio); int gpio_set_value(unsigned gpio, int value);`
> `gpio_direction_output` vs `gpio_set_value`: 前者置为输出同时更新端口值，后者仅仅改变端口值；
- gpio当做中断使用：`int gpio_to_irq(unsigned gpio);`返回中断编号
- 导出gpio端口到用户空间：`int gpio_export(unsigned gpio, bool direction_may_change)`;`direction_may_change`:表示是否允许用户程序修改gpio方向；
## 用户空间gpio的使用
通过sysfs接口访问gpio，`/sys/class/gpio/`
- export/unexport 文件接口：只能写不能读，用户程序通过写入gpio编号来向内核申请/释放某个gpio的控制权，例如：`echo 19 > export`上述操作会建立一个gpio19的节点
- `/sys/class/gpio/gpioN`：某个具体gpio端口，里面有一下属性文件；
    - direction：表示gpio方向，可以是in/out；如果内核代码不支持或不愿意修改，则不存在此属性，例如`gpio_export(n, 0);`
    - value：表示gpio引脚电平高低；
- `/sys/class/gpio/gpiochipN`：用来管理和控制一组gpio端口的控制器，
    - base：表示控制器管理的最小端口号
    - lable：诊断使用的标志，
    - ngpio：控制器管理的gpio端口数量（端口范围是：N~N+ngpio-1)
## GPIO三态
- 高阻态：输入和输出电阻相当大，相当于隔离状态，不会对外界电路产生任何影响，但还可以读取端口上的电平，一般高阻态都是做为模拟量的输入，因为高阻态不会影响到输入的电平，可以准确的读取模拟量。
- 推挽输出:push-pull,输出电阻小，自身能够输出高低电平；
- 开漏输出:oc，输出端相当于三极管的集电极，要得到高电平状态需要上拉电阻；吸收电流能力强相对强（20mA）；
