## eas_proxy文档

## url

| 说明     | url                             |
| -------- | ------------------------------- |
| 正式环境 | http://easproxy.api.yhglobal.cn |
| 测试环境 | http://47.104.65.97:58029       |

## <span id="sign">签名</span>
其它系统调用eas_proxy的任何接口，需带上签名（sign）参数。[签名算法链接](https://github.com/cthd2000/eas_proxy/blob/master/util/sign.md)

## 1、推送消息接口
> 用途：其它系统调用eas_proxy， 向eas_proxy推送数据<br>
> 调用链条:  由其它系统调用eas_proxy接口<br>
> 请求方式：POST<br>
> 请求URL ：[/msg/exchange](#)


### 1.1、请求
#### 请求参数

| 字段                | 字段类型 | 字段说明                     |
| ------------------- | -------- | ------------------------- |
| [bizType](#bizType) | string   | 业务类型 |
| msgCode             | string   | 消息唯一业务编码 |
| originCallbackUrl   | string   | 来源回调 url |
| originProject       | string   | 来源系统 |
| sign                | string   | [签名](#sign) |

### 1.2、响应
#### 响应参数
[Http 响应](#http)

#### data 参数
[完整消息类型](#exchangeMsg)

### 1.3、报文样例
> 推送不同的业务单据，需要匹配对应的bizType（业务类型）和msgJson（业务报文）<br>
> 以下以销售订单为例

#### 请求报文样例
> 业务单据:  销售订单<br>
> bizType：SO<br>
> msgJson ：[销售订单报文链接](https://github.com/cthd2000/eas_proxy/blob/master/model/销售订单.md)

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
#### 响应报文样例
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
> 用途：其它系统调用eas_proxy，查询某条消息的状态<br>
> 调用链条:  由其它系统调用eas_proxy接口<br>
> 请求方式：POST<br>
> 请求URL ：[/msg/status](#)<br>

### 2.1、请求
#### 请求参数

| 字段          | 字段类型 | 字段说明 |
| ------------- | -------- | ------------- |
| index         | long     | 消息游标(调用“推送消息接口”的返回参数) |
| originProject | string   | 来源系统 |
| sign          | string   | [签名](#sign) |

### 2.2、响应
####  响应参数
[Http 响应](#http)

#### data 参数

[完整消息类型](#exchangeMsg)

### 2.3、报文样例
#### 请求报文样例
```json
{
	"index": 1,
	"originProject": "xps",
	"sign": "743d0fdcd3539903268581648358b093"
}
```
#### 响应报文样例
> 注意，resultJson（结果json）为EAS处理该条数据的结果，<br>
> 其报文结构并非固定，具体需查看bizType（业务类型）对应的业务单据文档

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

> 调用链条:  由eas_proxy调用其它系统（originCallbackUrl）的服务

### 3.1、回调
#### 回调参数

[完整消息类型](#exchangeMsg)

### 3.2、响应
#### 响应参数

| 字段  | 字段类型 | 字段说明                     |
| ----- | -------- | ---------------------------- |
| state | integer  | 消息接收状态，0 代表成功接收 |
| desc  | string   | 说明                         |

### 3.3、报文样例
#### 回调报文样例
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
#### 响应报文样例

> 注意此状态为消息的「接收」状态，成功接收就为 0，跟业务成功与否没有关系

```json
{
  "desc": "success",
  "state": 0
}
```

## 附录
### <span id="exchangeMsg">1、消息结构</span>

| 字段                      | 字段类型 | 字段说明         |
| ------------------------- | -------- | ---------------- |
| index                     | long     | 消息下标         |
| msgCode                   | string   | 消息唯一业务编码 |
| [bizType](#bizType)       | string   | 业务类型         |
| originProject             | string   | 来源系统         |
| [msgJson](#msgJson)       | json     | 来源业务 json    |
| originCallbackUrl         | string   | 来源回调 url     |
| createDate                | date     | 创建日期         |
| [resultJson](#resultJson) | json     | 结果 json [链接](https://github.com/cthd2000/eas_proxy/tree/master/model) |
| msgState                  | integer  | 消息状态         |
| exchangeUrl               | string   | 代理回调交换 url |
| exchangeState             | integer  | 交换状态         |
| exchangeDate              | date     | 交换日期         |
| updateDate                | date     | 更新日期         |

### <span id="bizType">2、业务类型</span>
业务报文详情见：eas_proxy/model目录

| 编码 | 名称 | 备注 |
| --------- | --------- | --------- |
| PO | 采购订单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/采购订单.md) |
| PI | 采购入库 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/采购入库单.md) |
| PR        | 采购退货 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/采购退货单.md) |
| SO | 销售订单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/销售订单.md) |
| SS | 销售出库 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/销售出库单.md) |
| SR        | 销售退货 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/销售退货单.md) |
| IT        | 二方调拨单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/二方调拨单.md) |
| ST        | 库存调拨单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/库存调拨单.md) |
| TO        | 调拨出库单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/调拨出库单.md) |
| TI        | 调拨入库单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/调拨入库单.md) |
| AO        | 其他出库单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/其他出库单.md) |
| AI        | 其他入库单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/其他出库单.md) |
| AR        | 应收单 |  |
| AP        | 应付单 |  |
| CO        | 收款单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/财务会计/收款单.md) |
| BP        | 付款单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/财务会计/付款单.md) |
| LM        | 库位移动单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/库位移动单.md) |
| CS        | 确认签收日期 |  |
| VO        | 凭证 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/财务会计/凭证.md) |
| VO_SEQ        | 凭证-单线程 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/财务会计/凭证.md) |
| SN        | 无来源销售出库单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/无来源销售出库单.md)  |
| IS        | 库存状态调整单 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/供应链管理/库存状态调整单.md) |
| CU        | 客户 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/基础资料/客户-创建.md) |
| SU        | 供应商 | [文档链接](https://github.com/cthd2000/eas_proxy/blob/master/model/基础资料/供应商-创建.md) |
| PP        | 采购计划单 |  |
| ARIV        | 应收发票单 |  |
| ARIVQ        | 应收发票单-查询 |  |
| ARIVC        | 应收发票单-取消 |  |

### <span id="http">3、Http 响应</span>

| 字段 | 字段类型 | 字段说明              |
| ---- | -------- | --------------------- |
| code | integer  | 编码(0 成功 其它失败) |
| data | object   | 根据不同的请求返回    |
| msg  | string   | 描述                  |