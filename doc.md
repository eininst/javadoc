# 墨尔本送餐智能配送计划 接口文档

## 配送员接口
**简要描述：** 获取当前可分配的快递员
   
**请求 URL：** `/api/v1/couriers`
   
**请求方式：** GET
   
**请求参数：**

| 参数名 | 必选 | 类型 | 默认值 | 单位 | 说明 |
|:----:|:---:|:-----:|:-----:|:-----:|:-----:|
| `courier_seconds` | 是 | `number` | 900 | 秒 |过滤配送员位置更新时间在最近courier_seconds内的配送员 |
| `order_seconds` | 是 | `number` | 3600 | 秒 | 过滤配送员最近order_seconds内完成的订单 |

**返回示例**
- 调用成功示例
```
{
	code: 0,
	data: [{
		driving_type: -1,
		id: 233,
		latitude: -37.80728962839981,
		longitude: 144.9662161999722,
		orders: [],
		score: 8.3974
	}, {
		driving_type: 2,
		id: 266,
		latitude: -37.85284030720391,
		longitude: 145.1517056488852,
		orders: [],
		score: 8.6077
	}]
}
```
- 调用失败示例
```
{
	code: 1001,
	message: "courier_seconds不能大于3600 (1个小时)"
}
```
