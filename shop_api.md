# 墨尔本送餐 商家商品API 接口文档

## 获取token
**简要描述：** 通过商家的账号密码获取token
 
**请求 URL：** `/open_api/shop/v1/access_token`
   
**请求方式：** POST
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `login_name` | 是 | `string`  |商家登录账号 |
| `password` | 是 | `string`  | 商家登录密码 |

**返回示例**
- 调用成功示例
```
{
    "code": 0,
    "manager": {
        "expired_in": 31536000,
        "shop_id": 14,
        "shop_name": "墨尔本卤味工坊",
        "token": "eyJhbGciOiJIUzI1NiIsImV4cCI6MTU1NDY1OTE5NywiaWF0IjoxNTIzMTIzMTk3fQ.eyJpZCI6Mn0.LSvakghfQlTiP_JiVAcXlSA-Ob4nH-mP2NVgOINF0oU"
    }
}
```
- 调用失败示例
```
{
    "code": 100,
    "message": "登录账号不能为空"
}
```

**调用成功返回参数说明：**

| 返回参数名 | 类型 | 说明 |
|:-----:|:-----:|:-----:|
| `expired_in` | `number` | 失效时间(秒) | 
| `shop_id` | `number` | 商家ID | 
| `shop_name` | `number` | 商家名称 | 
| `token` | `number` | token请求其他接口需要携带 | 


## 获取商品分类列表
**简要描述：** 商品分类列表
 
**请求 URL：** `/open_api/shop/v1/categories`
   
**请求方式：** GET
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `token` | 是 | `string`  | 需要携带在request header(通过access_token接口获取) |

**返回示例**
- 调用成功示例
```
{
    "code": 0,
    "data": [
        {
            "create_time": "2015.03.17 20:51:09",
            "id": 26,
            "name": "全部",
            "seq": 1,
            "update_time": "2015.03.17 20:51:09"
        },
        {
            "create_time": "2015.06.07 22:58:26",
            "id": 848,
            "name": "【特色招牌菜】   ",
            "seq": 2,
            "update_time": "2015.07.08 23:56:38"
        },
        {
            "create_time": "2015.06.07 22:57:50",
            "id": 847,
            "name": "【秘制冒菜】",
            "seq": 3,
            "update_time": "2015.07.08 23:56:38"
        }
    ]
}
```

**调用成功返回参数说明：**

| 返回参数名 | 类型 | 说明 |
|:-----:|:-----:|:-----:|
| `id` | `number` | 分类ID | 
| `name` | `number` | 分类名称 | 
| `seq` | `number` | 排序序号 | 
| `create_time` | `string` | 创建时间 | 
| `update_time` | `string` | 最后修改时间 | 


## 新增商品分类
**简要描述：** 新增商品分类
 
**请求 URL：** `/open_api/shop/v1/categories`
   
**请求方式：** POST application/json
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `token` | 是 | `string`  | 需要携带在request header(通过access_token接口获取) |
| `name` | 是 | `string`  | 分类名称 |
| `seq` | 是 | `number`  | 排序序号 |
例如: 

	request body:
	
    {
        "name":"test",
        "seq":4
    }

**返回示例**
- 调用成功示例
```
{
    "code": 0
}
```
- 调用失败示例
```
{
    "code": 400,
    "message": "seq只能是数字"
}
```

## 修改商品分类
**简要描述：** 新增商品分类
 
**请求 URL：** `/open_api/shop/v1/categories/<int:category_id>`
   
**请求方式：** PUT application/json
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `token` | 是 | `string`  | 需要携带在request header(通过access_token接口获取) |
| `name` | 否 | `string`  | 分类名称 |
| `seq` | 否 | `number`  | 排序序号 |
例如: 

	request body:
	
    {
        "name":"test2",
        "seq":20
    }

**返回示例**
- 调用成功示例
```
{
    "code": 0
}
```
- 调用失败示例
```
{
    "code": 400,
    "message": "分类不存在"
}
```

## 删除商品分类
**简要描述：** 删除商品分类
 
**请求 URL：** `/open_api/shop/v1/categories/<int:category_id>`
   
**请求方式：** DELETE
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `token` | 是 | `string`  | 需要携带在request header(通过access_token接口获取) |

**返回示例**
- 调用成功示例
```
{
    "code": 0
}
```
- 调用失败示例
```
{
    "code": 400,
    "message": "分类不存在"
}
```

## 获取商品列表
**简要描述：** 获取商品列表
 
**请求 URL：** `/open_api/shop/v1/items`
   
**请求方式：** GET
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `token` | 是 | `string`  | 需要携带在request header(通过access_token接口获取) |

**返回示例**
- 调用成功示例
```
{
    "code": 0,
    "data": [
        {
            "category": {
                "create_time": "2015.06.07 22:58:26",
                "id": 848,
                "name": "【特色招牌菜】   ",
                "seq": 2,
                "update_time": "2015.07.08 23:56:38"
            },
            "desc": "本店冷吃兔 人气畅销 麻辣鲜香 嚼劲十足 川味典范～独道的正宗四川手艺！",
            "group_buy": 0,
            "group_price": 0,
            "id": 355,
            "image": "p_20150317230401289019.jpg",
            "name": "冷吃兔",
            "option_groups": [],
            "price": 20,
            "seq": 2,
            "shop_id": 14,
            "stock": 999,
            "unit": 0,
            "update_time": "2015.10.15 22:00:03"
        },
        {
            "category": {
                "create_time": "2015.06.07 22:58:26",
                "id": 848,
                "name": "【特色招牌菜】   ",
                "seq": 2,
                "update_time": "2015.07.08 23:56:38"
            },
            "desc": "冷吃牛肉是四川特色之一，刀工精细 做工讲究 风味独特 鲜味悠长～",
            "group_buy": 0,
            "group_price": 30,
            "id": 357,
            "image": "p_20150318000158603580.jpg",
            "name": "冷吃牛肉",
            "option_groups": [],
            "price": 20,
            "seq": 4,
            "shop_id": 14,
            "stock": 999,
            "unit": 0,
            "update_time": "2015.10.15 22:00:03"
        }
    ]
}
```

## 新增商品
**简要描述：** 新增商品
 
**请求 URL：** `/open_api/shop/v1/items`
   
**请求方式：** POST
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `token` | 是 | `string`  | 需要携带在request header(通过access_token接口获取) |
| `category_id` | 是 | `string`  | 商品分类ID |
| `name` | 是 | `string`  | 商品名称,字数限制10个 |
| `price` | 是 | `string`  | 商品价格 |
| `stock` | 是 | `string`  | 商品库存 |
| `seq` | 是 | `string`  | 商品排序序号 |
| `image` | 是 | `string`  | 商品图片 |
| `group_buy` | 否 | `number`  | 是否开启团购价(0不开启，1开启,默认0) |
| `group_price` | 否 | `number`  | 团购价格(group_buy为1时生效) |
| `desc` | 否 | `string`  | 商品描述,展示在商品详情界面上 |
| `option_group` | 否 | `json`  | 商品选项组 |


```
"option_group":[
	{
	    "name":"规格口味", //选项组名称,必填
	    "mode": 0, //模式(单选0，多选1),必填
	    "limit": 0, //数量限制(0为不限制),非必填
	    "min": 0, //最少选择数量(mode为1时生效,0为不限制),非必填
	    "options":[
	        {
	            "name": "香锅小龙虾", //选项名称,必填
	            "price": 70.8 //价格,必填
	        },
	        {
	            "name": "蒜泥小龙虾", //选项名称,必填
	            "price": 68 //价格,必填
	        }
	    ]
	}
]
```

**请求示例**
```
{
    "category_id": 42908,
    "price": 68,
    "stock": 99,
    "seq": 1,
    "name": "精品小龙虾2",
    "desc": "汤面不宜久放，请及时食用",
    "option_group":[
	    {
	        "name":"规格口味",
	        "mode": 0,
	        "limit": 0, 
	        "min": 0, 
	        "options":[
	            {
	                "name": "香锅小龙虾", 
	                "price": 70.8 
	            },
	            {
	                "name": "蒜泥小龙虾", 
	                "price": 68 
	            }
	        ]
	    },
	    	    {
	    	"name": "辣度",
	    	"mode":0,
	    	"options":[
	    		{	
	    			"name": "不辣",
	    			"price": 0
	    		},
	    		{	
	    			"name": "不辣",
	    			"price": 0
	    		},
	    		{	
	    			"name": "不辣",
	    			"price": 0
	    		}
	    	]
	    },
	    {
	    	"name": "加菜",
	    	"mode": 1,
	    	"options":[
	    		{
	    			"name": "年糕",
	    			"price": 4
	    		},
	    		{
	    			"name": "午餐肉",
	    			"price": 12
	    		}
	    	]
	    }
	]
}
```

- 调用成功示例
```
{
    "code": 0
}
```
- 调用失败示例
```
{
    "code": 400,
    "message": "name不能为空"
}
```



## 修改商品
**简要描述：** 新增商品
 
**请求 URL：** `/open_api/shop/v1/items`
   
**请求方式：** POST
   
**请求参数：**

| 参数名 | 必选 | 类型 | 说明 |
|:----:|:---:|:-----:|:-----:|
| `token` | 是 | `string`  | 需要携带在request header(通过access_token接口获取) |
| `category_id` | 是 | `string`  | 商品分类ID |
| `name` | 是 | `string`  | 商品名称,字数限制10个 |
| `price` | 是 | `string`  | 商品价格 |
| `stock` | 是 | `string`  | 商品库存 |
| `seq` | 是 | `string`  | 商品排序序号 |
| `image` | 是 | `string`  | 商品图片 |
| `group_buy` | 否 | `number`  | 是否开启团购价(0不开启，1开启,默认0) |
| `group_price` | 否 | `number`  | 团购价格(group_buy为1时生效) |
| `desc` | 否 | `string`  | 商品描述,展示在商品详情界面上 |
| `option_group` | 否 | `json`  | 商品选项组 |


**请求示例**
```
返回格式参考  [新增商品](#新增商品)
```

- 调用成功示例
```
{
    "code": 0
}
```
- 调用失败示例
```
{
    "code": 400,
    "message": "name不能为空"
}
```
