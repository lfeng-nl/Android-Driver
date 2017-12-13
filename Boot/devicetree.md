

# 本文介绍devicetree相关知识

## 1.The Devicetree

### 1.设备树结构和约定

- 节点名：一般形式：node_name@node_address；例如`fm2018@60`
- path name：一个节点可以被一个从根节点开始的路径标示：`/node_name1/node_name2/node_name3`
- 属性：节点都有描述他特性的属性，属性由name和value组成；

### 2.标准属性

- **compatible**：值为`<stringlist>`；用于驱动程序查找，一般为“manufacturer，model”例如：`compatible = “fsl,mpc8641-uart”,"ns16550"`；会先尝试`fsl,mpc8641-uart”`，失败则找`"ns16550"`；

- **model**：值为`<stringlist>`；格式为`“manufacturer，model”`

- **phandle**：值为`<u32>`；指定了该节点在设备树中唯一的数值标识符；

- **status**：值为`<string>`；指示设备运行状态，”okay“，”disabled“，”fail“，”fail-sss“

- **#address-cells and #size-cells**：值为`<u32>`；描述子节点如何编址；`#address-cells`：子节点`reg`属性中，几个cell标示地址；`#size-cells`：子节点`reg`属性中，几个cell表示长度 ；例如：下面例子中，0x4600代表地址，0x100代表所站空间；

  ```
  soc{
    #address-cells  = <1>;
    #size-cells = <1>;
    serial{
      reg = <0x4600 0x100>;
    }
  }
  ```

- **reg**：值为`<prop-encoded-array>`；描述了设备在其父总线定义空间内的地址资源；`reg = <address length>`；例如

  ```
  // a 32(0x20)-byte block at offset 0x3000, and a 256(0x100)-byte block offset 0xfe00;
  #address-cells = <1>;
  #size-cells = <1>;
  {
  	reg = <0x3000 0x20 0xfe00 0x100>;
  }
  ```

- **ranges**：值为`<empty> or <prop-encoded-array>`；定义了子节点地址空间和父节点地址空间的映射关系；格式一般为`<child-bus-address patent-bus-address length>`；address长度由各自的`#address-cells`决定，length长度由ranges所在节点的`#size-cells`决定；例如：子节点地址 0x0 映射到父节点地址 0xe0000000，范围为0x00100000；

  ```
  #address-cells = <1>;
  #size-cells = <1>;
  ranges = <0x0 0xe0000000 0x00100000>;
  ```

- **name**：值为`<string> `；表示节点名称；目前已不建议使用；

### 3.中断和中断映射

设备树采用中断树模型来表示中断；

- **interrupts**：值为`<prop-encoded-array> `；定义了由设备产生的中断，值为任意数量的中断说明符（ interrupt specifiers)，一般cells数量由`#interrupt-cells`决定，具体每个cell的作用，根据驱动程序来，

  ```
  interrupts = <0xa 8>;
  ```

- **interrupt-parent**：值为`<phandle> `；中断树的层级和设备树的层级可能不匹配，该值指明父中断；

- **#interrupt-cells**：值为`<u32> `；该中断域中，需要几个cell来表示中断表示符号；

- **interrupt-controller**：值为`<empty>`；定义了一个中断控制节点，该节点做为接收中断信号设备；

- **interrupt-map**：值为`<prop-encoded-array> `；定义中断映射入口，从前向后包括：中断子设备地址（cell由所在总线的address-cells属性决定），子中断标识符(interrupt specifier)，中断父设备，父中断设备地址，父中断标识符(interrupt specifier)五部分。例如：

  ```
  soc {
    compatible = "simple-bus";
    #address-cells = <1>;
    #size-cells = <1>;
    open-pic {
          clock-frequency = <0>;
          interrupt-controller;
          #address-cells = <0>;
          #interrupt-cells = <2>;
    };
    pci {
          #interrupt-cells = <1>;
          #size-cells = <2>;
          #address-cells = <3>;
          interrupt-map-mask = <0xf800 0 0 7>;
          interrupt-map = <
          /*IDSEL 0x11 - PCI slot 1 */
          0x8800 0 0 1 &open-pic 2 1  /*INTA*/
          0x8800 0 0 2 &open-pic 3 1 /*INTB*/
          0x8800 0 0 3 &open-pic 4 1 /*INTC*/
          0x8800 0 0 4 &open-pic 1 1 /*INTD*/

          /*IDSEL 0x12 - PCI slot 2*/
          0x9000 0 0 1 &open-pic 3 1 /*INTA*/
          0x9000 0 0 2 &open-pic 4 1 /*INTB*/
          0x9000 0 0 3 &open-pic 1 1 /*INTC*/
          0x9000 0 0 4 &open-pic 2 1 /*INTD*/
          >;
        };
  };
  /****************************************************
  说明：
  child unit address: 0x8800 0 0
  child interrupt specifier: 1
  interrupt parent:&open-pic
  parent unit address: (empty because #address-cells = <0> in the open-pic node)
  parent interrupt specifier: 2 1
  *****************************************************/
  ```

  ​

- **interrupt-map-mask**：值为`<prop-encoded-array> `

> 查看[我眼中的Linux设备树](http://blog.csdn.net/loongembedded/article/details/51453499)

-----------

## 2.设备节点要求

### 1.`root node`

- 所有的设备树必须有root节点`/`，

### 2.`/aliases node`

- 此节点定义别名；

  ```
  aliases{
    selrial0 = "/simple-bus@fe0000000/serial@11c500";
  }
  /********************************************
  设置serial0的别名；
  ********************************************/

  ```

### 3.`/memory node`

### 4.`/chosen node`

### 5.`/cpus node`

--------

## 3.设备绑定

### 1.绑定规则

- 通用规则：

### 2.serial  devices
### 3.Network devices
-------

## 4.Devicetree source format 

### 1.节点和属性定义:`[]为非必须`

```
[label:] node-name[@unit-address] {
  [properties definitions]
  [child nodes]
}
```
属性由名称和值组成，值可以为空；属性值可以为32位数，字符串，bytestring，或他们的组合；

```
[label:] property-name = value;
[label:] property-name;
interrupts = <17 0xc>;

//string 带一个NULL结尾的字符
compatible = "simple-bus";

// bytestring
local-mac-address = [00 00 12 34 56 78]；
//相当于
local-mac-address = [000012345678];
```

### 2.文件layout

```
/dts-v1/;
[memory reservations]
/ {
    [property definitions]
    [child nodes]
};
```

## 5.pinctrl相关

定义引脚复用或配置参数（上下拉，三态，驱动强度等）及不同工况下的状态，例如：active,suspend；

> 参考：`kernel/Documentation/devicetree/bindings/pinctrl/`下相关文档；

### 1.相关属性

- **compatible**：匹配名称；
- **gpio-controller**：值为`<empty>`；表明该节点为gpio控制器；
- **#gpio-cell**：值为`<u32>`，与`#address-cells`相似，表明gpio指示符cell个个数；具体每位代表含义，参考相关文档，例如高通：必须为2位，用于指定pin number和flags，

### 2.子节点相关属性

- **pins**：值为`<string-arrar>`；必须，受此子节点指定属性影响的gpio引脚列表。
- **function**：值为<string>；必须，指定引脚功能。
- **bias-disable**：值为<none>；可选，指定引脚必须配置为no pull.
- **bias-pull-down：值为<none>；可选，指定引脚为下拉。
- **bias-pull-up**：值为<none>；指定引脚为上拉。
- **output-high**：值为<none>；引脚为输出模式，驱动为高电平。
- **output-low**：值为<none>；引脚为输出模式，驱动为低电平。
- **drive-strength**：值为<u32>；驱动强度，mA，值为：2,4,6,8,10,12,14,16.