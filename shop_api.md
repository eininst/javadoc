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
