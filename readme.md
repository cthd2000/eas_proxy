
## EAS 代理接口

## 部署环境说明

说明       |url       
------------|-----------
正式环境 | 稍微等候
测试环境 | asked

## 关于签名
java 签名算法: 理论上就是字典序排序 key 再根据 key 拼接。
```Java
	//校验签名
   public static boolean checkSign(Map<String, Object> params, String projectKey) {
        String hash = (String) params.get("sign");
        params.remove("sign");

        String content = getSignCheckContent(params);
        content += "&projectKey=" + projectKey;
        String sign = Md5Util.md5(content);
        if (sign.equals(hash)) {
            return true;
        }
        return false;
    }
	//排序拼接
    public static String getSignCheckContent(Map<String, Object> params) {
        if (params == null) {
            return null;
        }

        StringBuffer content = new StringBuffer();
        List<String> keys = new ArrayList<>(params.keySet());
				//字典序排序
        Collections.sort(keys);

				//根据 key 获得 value 拼接
        for (int i = 0; i < keys.size(); i++) {
            String key = keys.get(i);
            String value = String.valueOf(params.get(key));
            content.append((i == 0 ? "" : "&") + key + "=" + value);
        }

        return content.toString();
    }

    public static void sign(Map<String, Object> params, String key) {
        String content = getSignCheckContent(params);
        content += "&projectKey=" + key;
        String sign = Md5Util.md5(content);
        params.put("sign", sign);
    }
```

## <span id="exchangeMsg">消息结构</span>

字段 |字段类型 |字段说明
---|---|---
index | long | 消息下标
msgCode | string | 消息唯一业务编码
[bizType](#bizType) | string | 业务类型
originProject | string | 来源系统
[msgJson](#msgJson) | json | 来源业务 json
originCallbackUrl | string | 来源回调 url
createDate | date | 创建日期
[resultJson](#resultJson) | json | 结果 json
msgState | integer | 消息状态
exchangeUrl | string | 代理回调交换 url
exchangeState | integer | 交换状态
exchangeDate | date | 交换日期
updateDate | date | 更新日期

## <span id="bizType">业务类型</span>


编码	| 说明
---|---
[PO](#PO) | 采购订单
[PI](#PI) | 采购入库
PR | 采购退货
[SO](#SO) | 销售订单
[SS](#SS) | 销售出库
SR | 销售退货
IT | 二方调拨单
ST | 库存调拨单
TO | 调拨出库单
TI | 调拨入库单
AO | 其他出库单
AI | 其他入库单
AR | 应收单
AP | 应付单
CO | 收款单
BP | 付款单
LM | 库位移动单
CS | 确认签收日期
VO | 凭证
SN | 无来源销售出库单
IS | 库存状态调整单
CU | 客户
SU | 供应商
PP | 采购计划单

## <span id="http">Http 响应</span>

字段 |字段类型 |字段说明
---|---|---
code | integer | 编码(0 成功 其它失败)
data | object | 根据不同的请求返回
msg | string | 描述


## 推送消息接口

> 请求方式：POST<br>
请求URL ：[/msg/exchange](#)


字段 |字段类型 |字段说明
---|---|---
[bizType](#bizType) | string | 业务类型
msgCode | string | 消息唯一业务编码
originCallbackUrl | string | 来源回调 url
originProject | string | 来源系统
sign | string | 签名(通过参数自然排序/字典序拼接而成的 md5( key1=val1&key2=val2&...&projectKey=xxx))

### 请求例子
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

### 返回参数

[Http 响应](#http)

#### data 参数

[完整消息类型](#exchangeMsg)

### 返回例子
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

## 消息查询接口

> 请求方式：POST<br>
请求URL ：[/msg/status](#)

字段 |字段类型 |字段说明
---|---|---
index | long | 消息游标(跟 msgCode 可以二选一，也可以全带 AND)
msgCode | string | 消息唯一业务编码(跟 index 可以二选一，也可以全带 AND)
originProject | string | 来源系统
sign | string | 签名(通过参数自然排序拼接而成的 md5(key1=val1&key2=val2&...&projectKey=xxx))

### 请求例子
```json
{
	"index": 1,
	"originProject": "xps",
	"sign": "743d0fdcd3539903268581648358b093"
}
```

### 返回参数

[Http 响应](#http)

#### data 参数

[完整消息类型](#exchangeMsg)

### 返回例子
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

## 消息拉取接口

> 请求方式：POST<br>
请求URL ：[/msg/pull](#)

字段 |字段类型 |字段说明
---|---|---
index | long | 开始抓取的消息下标(>=)
limit | long | 抓取的条数
originProject | string | 来源系统
sign | string | 签名(通过参数自然排序拼接而成的 md5( key1=val1&key2=val2&...&projectKey=xxx))

### 请求例子
```json
{
	"index": 0,
	"limit": 23,
	"originProject": "eas",
	"sign": "16c5303a42a50b3936870f0492e0a575"
}
```

### 返回参数

[Http 响应](#http)

#### data 参数

字段 |字段类型 |字段说明
---|---|---
maxIndex | long | 目前消息最大下标
msgList | array | 拉取的消息集合

#### msgList 参数

[完整消息类型](#exchangeMsg)

### 返回例子
```json
{
    "code": 0,
    "data": {
        "maxIndex": 24,
        "msgList": [
            {
                "bizType": "SO",
                "createDate": "2019-01-11 03:01:49",
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
            }
        ]
    },
    "desc": "调用成功"
}
```
## EAS 通知 Proxy 接口

> 请求方式：POST<br>
请求URL ：[/msg/notify](#)

### 请求参数

字段 |字段类型 |字段说明
---|---|---
index | long | 消息下标
msgCode | string | 消息唯一业务编码
originProject | string | 来源系统
sign | string | 签名(通过参数自然排序拼接而成的 md5( key1=val1&key2=val2&...&projectKey=xxx))

### 请求例子
```json
{
	"index": 3,
	"msgCode": "SQ20180901003",
	"originProject": "eas",
	"resultJson": "{}",
	"sign": "bab99b8e38b56736077b90ca53e07327"
}
```

### 返回参数

[Http 响应](#http)

## 消息 callback 接口

该接口由 Proxy 获得 EAS 答复以后回调 originCallbackUrl

### 回调参数

[完整消息类型](#exchangeMsg)

### 回调例子
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
### 需要应答参数

字段 |字段类型 |字段说明
---|---|---
state | integer | 消息接收状态，0 代表成功接收
desc | string | 说明

### 应答例子

注意此状态为消息的「接收」状态，成功接收就为 0，跟业务成功与否没有关系

```json
{
  "desc": "success",
  "state": 0
}
```

***
***
***

## <span id="PO">采购订单</span>

### 1.字段定义
#### 单据头字段

字段 | 字段类型 | 字段说明 | 备注
--- | --- | --- | ---
baseStatus | String | 单据状态 | 审核状态：4
billTypeID | String | 单据类型 | 采购订单：220
bizDate | Date | 业务日期 | 毫秒
createTime | Date | 制单日期 | 毫秒
bizType | String | 业务类型 | 普通采购：110
creator | String | 制单人 | EAS用户编码
currency | String | 币别 | EAS币别编码或ISO货币代码，人民币：BB01或RMB
exchangeRate | BigDecimal | 汇率 | 
inTax | boolean | 是否含税 | true
paymentType | String | 付款方式 | 赊购：004
purchaseOrgUnit | String | 采购组织 | EAS采购组织编码
supplier | String | 供应商 | EAS供应商编码
totalAmount | BigDecimal | 总金额 | 保留2位小数
totalTax | BigDecimal | 总税额 | 保留2位小数
totalTaxAmount | BigDecimal | 总价税合计 | 保留2位小数
localTotalAmount | BigDecimal | 本位币总金额 | 保留2位小数
localTotalTaxAmount | BigDecimal | 本位币总加税合计 | 保留2位小数
fromSystem | String | 来源系统 | 讯宇：XY
sourceFunction | String | 来源功能 | 预留字段
entries | List | 分录 | 

#### 分录字段

字段 | 字段类型 | 字段说明 | 备注
--- | --- | --- | ---
seq | int | 行号 | 回告报文中会携带
orderNumber | String | 供应商订单号 | 
bizTraceNumber | String | 业务跟踪号 | 
taxPrice | BigDecimal | 含税单价 | 保留6位小数
actualPrice | BigDecimal | 实际单价 | 保留6位小数
actualTaxPrice | BigDecimal | 实际含税单价 | 保留6位小数
price | BigDecimal | 单价 | 保留6位小数
qty | BigDecimal | 数量 | 
baseQty | BigDecimal | 基本数量 | 
discountRate | BigDecimal | 折扣率 | 
discountAmount | BigDecimal | 折扣额 | 保留2位小数
taxRate | BigDecimal | 税率 | 
taxAmount | BigDecimal | 价税合计 | 保留2位小数
amount | BigDecimal | 金额 | 保留2位小数
tax | BigDecimal | 税额 | 保留2位小数
localTaxAmount | BigDecimal | 本位币价税合计 | 保留2位小数
localAmount | BigDecimal | 本位币金额 | 保留2位小数
localTax | BigDecimal | 本位币税额 | 保留2位小数
material | String | 物料 | EAS物料编码
present | boolean | 是否赠品 | 非赠品：false；赠品：true

***
### <span id="msgJson">2.报文</span>
```json
{
	"baseStatus": "4",
	"billTypeID": "220",
	"bizDate": "1616947200000",
	"bizType": "110",
	"createTime": "1616947200000",
	"creator": "InterfaceUser",
	"currency": "BB01",
	"exchangeRate": 1.0,
	"inTax": true,
	"localTotalAmount": 1415.93,
	"localTotalTaxAmount": 1600.0,
	"paymentType": "004",
	"purchaseOrgUnit": "02029901",
	"supplier": "01.02.01.0012",
	"totalAmount": 1415.93,
	"totalTax": 184.07,
	"totalTaxAmount": 1600.00,
	"sourceFunction": "",
	"fromSystem": "XY",
	"entries": [
		{
			"seq": 1,
			"actualPrice": 176.99115,
			"actualTaxPrice": 200.0,
			"amount": 1415.93,
			"baseQty": 8,
			"orderNumber": "",
			"bizTraceNumber": "PO_20210329_01",
			"discountAmount": 0.0,
			"discountRate": 0.0,
			"localAmount": 1415.93,
			"localTax": 184.07,
			"localTaxAmount": 1600.0,
			"material": "04.01.01.01.178",
			"present": false,
			"price": 176.99115,
			"qty": 8,
			"tax": 184.07,
			"taxAmount": 1600.0,
			"taxPrice": 200.0,
			"taxRate": 13.0
		}
	]
}
```
***
### <span id="resultJson">3. 响应</span>
3.1 成功
```json
{
	"error": "",
	"success": true,
	"data": {
		"id": "wEQAAAS+0Dgxcb+t",
		"number": "POSQ2021030005",
		"entries": [
			{
				"seq": 1,
				"id": "wEQAAAS+0DkmBBzF"
			}
		]
	}
}
```
3.2 失败
```json
{
	"data": "",
	"error": "解析异常：采购组织不能为空",
	"success": false
}
```
***
***
## <span id="PI">采购入库单</span>
###  1.字段定义
#### 单据头字段

字段 | 字段类型 | 字段说明 | 备注
--- | --- | --- | ---
baseStatus | String | 单据状态 | 审核状态：4
billTypeID | String | 单据类型 | 采购入库单：103
bizDate | Date | 业务日期 | 毫秒
bizType | String | 业务类型 | 普通采购：110
transactionType | String | 事务类型 | 普通采购：001
creator | String | 制单人 | EAS用户编码
currency | String | 币别 | EAS币别编码或ISO货币代码，人民币：BB01或RMB
exchangeRate | BigDecimal | 汇率 | 
id | String | 来源单据id | EAS的采购订单id
inTax | boolean | 是否含税 | 含税：true；不含税：false
paymentType | String | 付款方式 | 赊购：004
sourceFunction | String | 来源功能 | 预留字段
fromSystem | String | 来源系统 | XY
storageOrgUnit | String | 库存组织 | EAS库存组织编码
supplier | String | 供应商 | EAS供应商编码
totalActualCost | BigDecimal | 总实际成本 | 保留2位小数
totalAmount | BigDecimal | 总金额 | 保留2位小数
totalLocalAmount | BigDecimal | 总本位币金额 | 保留2位小数
totalQty | BigDecimal | 总数量 | 
entries | List | 分录 | 

#### 分录字段

| 字段    | 字段类型 | 字段说明 | 备注 |
| --- | --- | --- | --- |
| seq | int | 行号 | 会携带在回告报文中 |
| id | String | 来源单据明细id | EAS采购订单分录id |
| bizTraceNumber | String | 业务跟踪号 |  |
| orderNumber | String | 供应商订单号 |  |
| warehouse | String | 仓库 | 备注 |
| material | String | 物料 | EAS物料编码 |
| location | String | 库位 | 良品区：01；机损区：02；箱损区：03；待报废：04；超保区：05；售后区 |
| lot | String | 批次 |  |
| taxPrice | BigDecimal | 含税单价 | 保留6位小数 |
| actualTaxPrice | BigDecimal | 实际含税单价 | 保留6位小数 |
| actualPrice | BigDecimal | 实际单价 | 保留6位小数 |
| price | BigDecimal | 单价 | 保留6位小数 |
| unitActualCost | BigDecimal | 单位实际成本 | 保留6位小数 |
| unitPurchaseCost | BigDecimal | 单位采购成本 | 保留6位小数 |
| discountRate | BigDecimal | 折扣率 |  |
| discountAmount | BigDecimal | 折扣 | 保留2位小数 |
| taxRate | BigDecimal | 税率 |  |
| tax | BigDecimal | 税额 | 保留2位小数 |
| taxAmount | BigDecimal | 价税合计 | 保留2位小数 |
| actualCost | BigDecimal | 实际成本 | 保留2位小数 |
| amount | BigDecimal | 金额 | 保留2位小数 |
| qty | BigDecimal | 数量 |  |
| baseQty | BigDecimal | 数量 |  |
| localAmount | BigDecimal | 本位币金额 | 保留2位小数 |
| localPrice | String | 本位币单价 | 保留6位小数 |
| localTax | String | 本位币税额 | 保留2位小数 |
| localTaxAmount | String | 本位币价税合计 | 保留2位小数 |
| purchaseCost | BigDecimal | 采购成本 | 保留2位小数 |
| present | boolean | 是否赠品 | 赠品：true; 非赠品：false。默认false |

### 2.报文
```json
{
	"baseStatus": "4",
	"billTypeID": "103",
	"bizDate": 1616947200000,
	"bizType": "312",
	"transactionType": "001",
	"creator": "InterfaceUser",
	"currency": "BB01",
	"exchangeRate": 1.0,
	"id": "wEQAAAS+0Dgxcb+t",
	"inTax": true,
	"paymentType": "004",
	"sourceFunction": "",
	"fromSystem": "XY",
	"storageOrgUnit": "02029901",
	"supplier": "01.02.01.0012",
	"totalActualCost": 1415.93,
	"totalAmount": 1415.93,
	"totalLocalAmount": 1600.0,
	"totalQty": 8,
	"entries": [
		{
			"seq": 1234,
			"qty": 8,
			"actualCost": 1415.93,
			"actualPrice": 6.637168,
			"actualTaxPrice": 7.5,
			"amount": 1415.93,
			"baseQty": 8,
			"bizTraceNumber": "PI_20210329_01",
			"orderNumber": "",
			"discountAmount": 0,
			"discountRate": 0,
			"id": "wEQAAAS+0DkmBBzF",
			"localAmount": 1415.93,
			"localPrice": 176.99115,
			"localTax": 184.07,
			"localTaxAmount": 1600.0,
			"location": "01",
			"lot": "SQ20210329002",
			"material": "04.01.01.01.178",
			"present": false,
			"price": 176.99115,
			"purchaseCost": 1415.93,
			"tax": 184.07,
			"taxAmount": 1600.0,
			"taxPrice": 200.0,
			"taxRate": 13.0,
			"unitActualCost": 176.99115,
			"unitPurchaseCost": 176.99115,
			"warehouse": "001"
		}
	]
}
```
### 3.响应
3.1 成功
```json
{
	"error": "",
	"success": true,
	"data": {
		"id": "wEQAAAS+0Dp4MGHj",
		"number": "PUI02021030000002",
		"entries": [
			{
				"seq": 1234,
				"id": "wEQAAAS+0DuOCIYW"
			}
		]
	}
}
```
3.2 失败
```json
{
	"data": "",
	"error": "解析异常：库存组织不能为空",
	"success": false
}
```
***
***
## <span id="SO">销售订单</span>

### 1、字段定义
#### 单据头字段

| 字段 | 字段类型 | 字段说明 | 备注 |
| --- | --- | --- | --- |
| baseStatus | String | 单据状态 | 审核状态：4 |
| billType | String | 单据类型 | 销售订单：310 |
| bizDate | Date | 业务日期 | 毫秒 |
| creator | String | 制单人 | EAS用户编码 |
| currency | String | 币别 | EAS币别编码或ISO货币代码，人民币：BB01或RMB |
| customerOrderNumber | String | 客户订单号 | 作为重复判断依据 |
| deliveryType | String | 交货方式 | 送货：SEND；自提：CARRY |
| fromSystem | String | 来源系统 | 讯宇：XY |
| orderCustomer | String | 订货客户 | EAS客户编码 |
| paymentType | String | 付款方式 | 赊销：002 |
| saleOrgUnit | String | 销售组织 | EAS销售组织编码 |
| localTotalAmount | BigDecimal | 总金额本位币 | 保留2位小数 |
| localTotalTaxAmount | BigDecimal | 总本位币价税合计 | 保留2位小数 |
| totalAmount | BigDecimal | 总金额 | 保留2位小数 |
| totalTax | BigDecimal | 总税额 | 保留2位小数 |
| totalTaxAmount | BigDecimal | 总价税合计 | 保留2位小数 |
| isInTax | int | 是否含税 | 不含税：0；含税：1 |
| sourceFunction | String | 来源功能 | 预留字段 |
| entries | List | 分录 |  |


#### 分录字段

| 字段 | 字段类型 | 字段说明 | 备注 |
| --- | --- | --- | --- |
| seq | int | 行号 | 会携带在回告报文中 |
| orderNumber | String | 客户订单号 |  |
| bizTraceNumber | String | 业务跟踪号 | 备注 |
| taxPrice | BigDecimal | 含税单价 | 保留6位小数 |
| actualTaxPrice | BigDecimal | 实际含税单价 | 保留6位小数 |
| actualPrice | BigDecimal | 实际单价 | 保留6位小数 |
| price | BigDecimal | 单价 | 保留6位小数 |
| qty | BigDecimal | 数量 |  |
| baseQty | BigDecimal | 基本数量 |  |
| discount | BigDecimal | 折扣率 |  |
| discountAmount | BigDecimal | 折扣金额 | 保留2位小数 |
| taxRate | BigDecimal | 税率 |  |
| taxAmount | BigDecimal | 价税合计 | 保留2位小数 |
| tax | BigDecimal | 税额 | 保留2位小数 |
| amount | BigDecimal | 金额 | 保留2位小数 |
| localAmount | BigDecimal | 本位币金额 | 保留2位小数 |
| localTax | BigDecimal | 本位币税额 | 保留2位小数 |
| localTaxAmount | BigDecimal | 价税合计本位币 | 保留2位小数 |
| warehouse | String | 仓库 | EAS仓库编码 |
| isPresent | int | 是否赠品 | 非赠品：0；赠品：1 |
| material | String | 物料 | EAS物料编码 |

### 2、报文
```json
{
	"baseStatus": "4",
	"billType": "310",
	"bizDate": 1616947200000,
	"creator": "InterfaceUser",
	"currency": "BB01",
	"customerOrderNumber": "SO_20210329_01",
	"deliveryType": "SEND",
	"fromSystem": "XY",
	"isInTax": 1,
	"localTotalAmount": 158.41,
	"localTotalTaxAmount": 179.0,
	"orderCustomer": "09.01.0273",
	"paymentType": "002",
	"saleOrgUnit": "02029932",
	"sourceFunction": "",
	"totalAmount": 158.41,
	"totalTax": 20.59,
	"totalTaxAmount": 179.0,
	"entries": [
		{
			"seq": 42764,
			"actualPrice": 158.40708,
			"actualTaxPrice": 179.0,
			"amount": 158.41,
			"baseQty": 1,
			"orderNumber": "ORDERNUMBER",
			"bizTraceNumber": "BIZTRACENUM",
			"isPresent": 0,
			"localAmount": 158.41,
			"localTax": 20.59,
			"localTaxAmount": 179.0,
			"material": "04.01.01.01.178",
			"price": 158.40708,
			"qty": 1,
			"tax": 20.59,
			"taxAmount": 179.0,
			"taxPrice": 179.0,
			"taxRate": 13.0,
			"warehouse": "099"
		}
	]
}
```
### 3、响应
3.1 成功
```json
{
	"error": "",
	"success": true,
	"data": {
		"id": "wEQAAAS+0Dgxcb+t",
		"number": "POSQ2021030005",
		"entries": [
			{
				"seq": 12345,
				"id": "wEQAAAS+0DkmBBzF"
			}
		]
	}
}
```
3.2 失败
```json
{
	"data": "",
	"error": "转换报文时异常：销售组织不能为空",
	"success": false
}
```
***
***
## <span id="SS">销售出库单</span>

### 1.字段定义
#### 单据头字段

| 字段 | 字段类型 | 字段说明 | 备注 |
| --- | --- | --- | --- |
| id | String | 来源单据id | EAS销售出库单id |
| baseStatus | String | 单据状态 | 审核状态：“4” |
| sourceFunction | String | 来源功能 | 重复推单校验依据 |
| creator | String | 制单人 | EAS用户编码 |
| bizType | String | 业务类型 | 普通销售：“210” |
| transactionType | String | 事务类型 | 普通销售出库：“010” |
| bizDate | Date | 业务日期 | 毫秒 |
| currency | String | 币别 | EAS币别编码或ISO货币代码，人民币：BB01或RMB |
| customer | String | 客户 | EAS客户编码 |
| exchangeRate | BigDecimal | 汇率 |  |
| isInTax | boolean | 是否含税 | 不含税：false；含税：true |
| storageOrgUnit | String | 库存组织 | EAS库存组织编码 |
| totalQty | BigDecimal | 总数量 |  |
| totalAmount | BigDecimal | 总金额 | 保留2位小数 |
| totalLocalAmount | BigDecimal | 价税总合计本位币 | 保留2位小数 |

#### 分录字段

| 字段 | 字段类型 | 字段说明 | 备注 |
| --- | --- | --- | --- |
| seq | int | 行号 | 会携带在回告报文中 |
| qty | BigDecimal | 数量 |  |
| id | String | 来源单据分录id | EAS销售订单分录id |
| invUpdateType | String | 更新类型 | 普通出库：“002” |
| material | String | 物料 | EAS物料编码 |
| lot | String | 批次 |  |
| warehouse | String | 仓库 | EAS仓库编码 |
| location | String | 库位 | 良品区：01；机损区：02；箱损区：03；待报废：04；超保区：05；售后区 |
| isPresent | int | 是否赠品 | 非赠品：0；赠品：1 |
| taxRate | BigDecimal | 税率 |  |
| taxPrice | BigDecimal | 含税单价 | 保留6位小数 |
| salePrice | BigDecimal | 单价 | 保留6位小数 |
| price | BigDecimal | 实际含税单价 | 保留6位小数 |
| actualPrice | BigDecimal | 实际单价 | 保留6位小数 |
| amount | BigDecimal | 价税合计 | 保留2位小数 |
| tax | BigDecimal | 税额 | 保留2位小数 |
| nonTaxAmount | BigDecimal | 金额 | 保留2位小数 |
| localPrice | BigDecimal | 本位币单价 | 保留6位小数 |
| localNonTaxAmount | BigDecimal | 金额本位币 | 保留2位小数 |
| localAmount | BigDecimal | 价税合计本位币 | 保留2位小数 |
| localTax | BigDecimal | 本位币税额 | 保留2位小数 |
| orderNumber | String | 客户订单号 |  |
| bizTraceNumber | String | 业务跟踪号 |  |
| unitActualCost | BigDecimal | 单位实际成本 | 保留6位小数 |
| baseUnitActualcost | BigDecimal | 基本单位实际成本 | 保留6位小数 |
| actualCost | BigDecimal | 实际成本 | 保留2位小数 |
| discountType | String | 折扣方式 | 折扣比率：“0” |
| discount | BigDecimal | 折扣率 |  |
| discountAmount | BigDecimal | 折扣金额 | 保留2位小数 |


### 2、报文
```json
{
	"id": "wEQAAALKWwBcKh8M",
	"sourceFunction": "XSCK_DB_TEST_2020070701",
	"creator": "user",
	"bizType": "210",
	"transactionType": "010",
	"bizDate": "2020-07-07",
	"currency": "BB01",
	"customer": "09.01.0273",
	"exchangeRate": 1,
	"isInTax": true,
	"storageOrgUnit": "02029901",
	"totalQty": 10,
	"totalAmount": 1097.35,
	"totalLocalAmount": 1097.35,
	"baseStatus": "4",
	"entries": [
		{
			"seq": 3,
			"qty": 10,
			"id": "wEQAAALKWwGsy4DG",
			"invUpdateType": "002",
			"material": "04.01.01.01.178",
			"lot": "SQ20191017001",
			"warehouse": "003",
			"location": "002.01",
			"taxRate": 13.0,
			"taxPrice": 124.0,
			"salePrice": 109.734513,
			"price": 124.0,
			"actualPrice": 109.734513,
			"amount": 1240.0,
			"localAmount": 1240.0,
			"tax": 142.65,
			"localTax": 142.65,
			"localNonTaxAmount": 1097.35,
			"localPrice": 109.734513,
			"nonTaxAmount": 1097.35,
			"bizTraceNumber": "DBDD_TEST_2020070701",
			"actualCost": 0,
			"baseUnitActualcost": 0,
			"unitActualCost": 0,
			"discount": 0,
			"discountAmount": 0,
			"discountType": "0",
			"isPresent": 0
		}
	]
}
```
### 3、响应
3.1 成功
```json
{
	"error": "",
	"success": true,
	"data": {
		"id": "wEQAAAS+0Dgxcb+t",
		"number": "POSQ2021030005",
		"entries": [
			{
				"seq": 1,
				"id": "wEQAAAS+0DkmBBzF"
			}
		]
	}
}
```
3.2 失败
```json
{
	"data": "",
	"error": "转换报文时异常：采购组织不能为空",
	"success": false
}
```
