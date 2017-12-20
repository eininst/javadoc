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
		orders: [1594249,1594265],
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

**调用成功返回参数说明：**

| 返回参数名 | 类型 | 说明 |
|:-----:|:-----:|:-----:|
| `id` | `number` | 配送员ID | 
| `longitude` | `number` | 配送员经度 | 
| `latitude` | `number` | 配送员纬度 | 
| `driving_type` | `number` | -1(未设定),0(电动自行车),1(摩托车),2(汽车),3(脚踏自行车),4(面包车),5(货车) | 
| `score` | `number` | 配送员评分(满分10) | 
| `orders` | `array` | 配送员最近order_seconds内完成的订单 | 



## 订单接口

**简要描述：** 获取当前可分配的快递员
   
**请求 URL：** `/api/v1/orders`
   
**请求方式：** GET
   

**返回示例**

- 调用成功示例
```
{
	code: 0,
	data: [{
		city_id: 1,
		create_time: "2017.11.20 20:46:43",
		delivery_time: "2017.11.20 尽快送货",
		id: 1594265,
		items: [{
			count: 1,
			item_id: 111298,
			name: "三秦套餐",
			price: 12.5
		}],
		location: "-37.805744,144.962790",
		pickup_minutes: -1,
		shop_id: 898,
		shop_location: "-37.80671299999999,144.96536100000003",
		status: 1,
		user_id: 14128
	},
	{
		city_id: 1,
		create_time: "2017.11.20 20:44:42",
		delivery_time: "2017.11.20 尽快送货",
		id: 1594249,
		items: [{
			count: 1,
			item_id: 372334,
			name: "湖南小炒肉",
			price: 19.8
		}, {
			count: 1,
			item_id: 372322,
			name: "烟笋炒腊肉",
			price: 26.8
		}, {
			count: 1,
			item_id: 372382,
			name: "皇族香米饭",
			price: 2
		}],
		location: "-37.806893,145.012289",
		pickup_minutes: -1,
		shop_id: 2544,
		shop_location: "-37.8201379,145.12363119999998",
		status: 1,
		user_id: 9614
	}]
}

```

**调用成功返回参数说明：**

| 返回参数名 | 类型 | 说明 |
|:-----:|:-----:|:-----:|
| `id` | `number` | 订单ID | 
| `city_id` | `number` | 固定为1 代表墨尔本 | 
| `location` | `string` | 订单经纬度 | 
| `create_time` | `string` | 订单创建时间 | 
| `delivery_time` | `string` | 客户选择的收货时间 | 
| `status` | `number` | 订单状态 1(准备中)  | 
| `user_id` | `number` | 客户ID | 
| `shop_id` | `number` | 餐馆ID | 
| `shop_location` | `number` | 餐馆经纬度| 
| `pickup_minutes` | `number` | 餐厅预估时间 -1(代表没有预估时间) | 
| `items` | `array` | 订单商品信息 | 

## 订单查询接口
**简要描述：** 通过id查询订单信息
   
**请求 URL：** `/api/v1/order/query`
   
**请求方式：** GET



**请求参数：**

| 参数名 | 必选 | 类型 | 默认值 | 说明 |
|:----:|:---:|:-----:|:-----:|:-----:|
| `ids` | 是 | `string` | 无 | 例如: 1594249,1594265 (以,分割订单号) 最多200个|
- 调用成功示例

返回格式参考  [订单接口](#订单接口)

