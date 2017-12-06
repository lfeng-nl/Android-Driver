# 本文介绍Linux i2c相关驱动

## 1.驱动的注册过程

`i2c_add_driver( )`--> `i2c_register_driver( )` --> `driver_register( )`

- `i2c_add_driver( )`：宏，仅仅调用`i2c_register_driver( )`
- `i2c_register_driver( )`：定义总线类型`bus_type`，`driver->driver.bus = &i2c_bus_type;` ；调用`driver_register( )`
- `driver_register( )`：
  - |--> ` driver_find()`：按名字在总线上查找驱动是否已经注册
  - |--> `bus_add_driver()`：把驱动添加到总线上：
    - |--> `bus_get()`得到`bus_type`
    - |--> `driver_private`实例化
    - |--> `klist_init()`：初始化klist structure
    - |--> `kboject_init_and_add()`：初始化kobject 结构并添加到kobject等级结构中
    - |--> `klist_add_tail()`：初始化klist_node 并添加到尾部
    - |--> `driver_attach()`：尝试绑定设备和驱动
      - 调用`bus_for_each_dev()`遍历bus，调用`__driver_attach( )`；
      - 根据bus_type中的 `i2c_device_match()`按照名字匹配设备和驱动；
        - |-->`of_driver_match_device()`
        - |-->``
        - |-->``
      - 调用`driver_probe_device()`--> `really_probe()`--> `dev->bus->probe()` --> `i2c_driver->probe()`
    - |--> `module_add_driver()`
  - |--> ` driver_add_greups()`：将驱动加入对应的组中
  - |--> ` kobject_uevent()`：注册uevent事件

