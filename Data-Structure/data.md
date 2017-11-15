## 1.platform_set_drvdata：将 data 放到`platform_device -> device dev --> device_private *p --> void *driver_data;;`
```c
void platform_set_drvdata(struct platform_device *pdev, void *data)
{
	dev_set_drvdata(&pdev->dev, data);
}
```
## 2.platform_get_drvdata：参见 `platform_set_drvdata`，反方向获取；
## 3.dev_set_drvdata：将data放到dev的私有数据中 driver_data
```c
int dev_set_drvdata(struct device *dev, void *data)
{
	int error;

	if (!dev->p) {
		error = device_private_init(dev);
		if (error)
			return error;
	}
	dev->p->driver_data = data;
	return 0;
}
```