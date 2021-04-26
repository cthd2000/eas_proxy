## eas_proxy文档

## url

| 说明     | url                             |
| -------- | ------------------------------- |
| 正式环境 | http://easproxy.api.yhglobal.cn |
| 测试环境 | http://47.104.65.97:58029       |



## 1、推送消息接口

外部系统调用proxy，向代理推送数据

> 请求方式：POST<br>
> 请求URL ：[/msg/exchange](#)

### 1.1、请求
#### 请求参数

| 字段                | 字段类型 | 字段说明                                                     |
| ------------------- | -------- | ------------------------------------------------------------ |
| [bizType](#bizType) | string   | 业务类型                                                     |
| msgCode             | string   | 消息唯一业务编码                                             |
| originCallbackUrl   | string   | 来源回调 url                                                 |
| originProject       | string   | 来源系统                                                     |
| sign                | string   | 签名(通过参数自然排序/字典序拼接而成的 md5( key1=val1&key2=val2&...&projectKey=xxx))，详情见，eas_proxy/util/sign.md |

#### 报文样例
```json
{
	"bizType":"SO",
	"msgCode":"SQ20180901022",
	"msgJson":"{}",
	"originCallbackUrl":"http://127.0.0.1:8080/callback/test",
	"originProject":"XPS",
	"sign": "855b600cf5b8cc0fcc6b794181cd2878"
}
```
### 1.2、返回
#### 返回参数
[Http 响应](#http)

#### data 参数

[完整消息类型](#exchangeMsg)

#### 报文样例
```json
{
    "code": 0,
    "data": {
        "bizType": "SO",
        "createDate": "2019-01-11 03:01:48",
        "exchangeDate": null,
        "exchangeState": 0,
        "exchangeUrl": "http://127.0.0.1:8080/msg/notify",
        "index": 24,
        "msgCode": "SQ20180901023",
        "msgJson": "{}",
        "msgState": 0,
        "originCallbackUrl": "http://127.0.0.1:8080/callback/test",
        "originProject": "XPS",
        "resultJson": null,
        "updateDate": null
    },
    "desc": "调用成功"
}
```



## 2、消息查询接口

外部系统调用proxy，查询某条消息的状态

> 请求方式：POST<br>
> 请求URL ：[/msg/status](#)

### 2.1、请求
#### 请求参数

| 字段          | 字段类型 | 字段说明                                                     |
| ------------- | -------- | ------------------------------------------------------------ |
| index         | long     | 消息游标(调用“推送消息接口”的返回参数)                       |
| originProject | string   | 来源系统                                                     |
| sign          | string   | 签名(通过参数自然排序拼接而成的 md5(key1=val1&key2=val2&...&projectKey=xxx)) |

#### 请求样例
```json
{
	"index": 1,
	"originProject": "xps",
	"sign": "743d0fdcd3539903268581648358b093"
}
```
### 2.2、返回
####  返回参数
[Http 响应](#http)

#### data 参数

[完整消息类型](#exchangeMsg)

#### 返回样例
```json
{
    "code": 0,
    "data": {
        "bizType": "SO",
        "createDate": "2019-01-11 02:01:10",
        "exchangeDate": "2019-01-11 04:01:11",
        "exchangeState": 1,
        "exchangeUrl": "http://47.104.154.165:58029/msg/notify",
        "index": 1,
        "msgCode": "SQ20180901001",
        "msgJson": "{}",
        "msgState": 1,
        "originCallbackUrl": "http://47.104.154.165:58029/callback/test",
        "originProject": "XPS",
        "resultJson": "{}",
        "updateDate": "2019-01-16 05:01:08"
    },
    "desc": "调用成功"
}
```



## 3、消息回告来源系统

proxy调用来源系统（originCallbackUrl），回告消息的处理结果

### 3.1、回调
#### 回调参数

[完整消息类型](#exchangeMsg)

#### 报文样例
```json
{
  "resultJson": "{}",
  "exchangeDate": "2019-01-11 04:01:21",
  "updateDate": "2019-01-12 12:01:21",
  "bizType": "SO",
  "msgState": 0,
  "originCallbackUrl": "http://127.0.0.1:8080/callback/test",
  "exchangeUrl": "http://127.0.0.1:8080/msg/notify",
  "sign": "d0cbc1dbeacd4c32ee934aa36dabc722",
  "index": 4,
  "msgJson": "{}",
  "originProject": "XPS",
  "exchangeState": 2,
  "msgCode": "SQ20180901004",
  "createDate": "2019-01-11 10:01:38"
}
```
### 3.2、响应
#### 响应参数

| 字段  | 字段类型 | 字段说明                     |
| ----- | -------- | ---------------------------- |
| state | integer  | 消息接收状态，0 代表成功接收 |
| desc  | string   | 说明                         |

### 响应样例

注意此状态为消息的「接收」状态，成功接收就为 0，跟业务成功与否没有关系

```json
{
  "desc": "success",
  "state": 0
}
```

## <span id="exchangeMsg">消息结构</span>

| 字段                      | 字段类型 | 字段说明         |
| ------------------------- | -------- | ---------------- |
| index                     | long     | 消息下标         |
| msgCode                   | string   | 消息唯一业务编码 |
| [bizType](#bizType)       | string   | 业务类型         |
| originProject             | string   | 来源系统         |
| [msgJson](#msgJson)       | json     | 来源业务 json    |
| originCallbackUrl         | string   | 来源回调 url     |
| createDate                | date     | 创建日期         |
| [resultJson](#resultJson) | json     | 结果 json        |
| msgState                  | integer  | 消息状态         |
| exchangeUrl               | string   | 代理回调交换 url |
| exchangeState             | integer  | 交换状态         |
| exchangeDate              | date     | 交换日期         |
| updateDate                | date     | 更新日期         |

## <span id="bizType">业务类型</span>
业务报文详情见：eas_proxy/model目录

| 编码      | 说明             |
| --------- | ---------------- |
| [PO](#PO) | 采购订单         |
| [PI](#PI) | 采购入库         |
| PR        | 采购退货         |
| [SO](#SO) | 销售订单         |
| [SS](#SS) | 销售出库         |
| SR        | 销售退货         |
| IT        | 二方调拨单       |
| ST        | 库存调拨单       |
| TO        | 调拨出库单       |
| TI        | 调拨入库单       |
| AO        | 其他出库单       |
| AI        | 其他入库单       |
| AR        | 应收单           |
| AP        | 应付单           |
| CO        | 收款单           |
| BP        | 付款单           |
| LM        | 库位移动单       |
| CS        | 确认签收日期     |
| VO        | 凭证             |
| SN        | 无来源销售出库单 |
| IS        | 库存状态调整单   |
| CU        | 客户             |
| SU        | 供应商           |
| PP        | 采购计划单       |

## <span id="http">Http 响应</span>

| 字段 | 字段类型 | 字段说明              |
| ---- | -------- | --------------------- |
| code | integer  | 编码(0 成功 其它失败) |
| data | object   | 根据不同的请求返回    |
| msg  | string   | 描述                  |