# GCPAY 商户 API 对接文档

> 版本：v1.0  
> 更新时间：2026-01-01

---

## 目录

- [1. 接入说明](#1-接入说明)
- [2. 签名规则](#2-签名规则)
- [3. 代收接口](#3-代收接口)
- [4. 代付接口](#4-代付接口)
- [5. 异步通知](#5-异步通知)
- [6. 附录](#6-附录)

---

## 1. 接入说明

### 1.1 基本信息

| 项目 | 说明 |
|------|------|
| 请求协议 | HTTPS |
| 请求方式 | POST |
| 数据格式 | application/x-www-form-urlencoded |
| 字符编码 | UTF-8 |

### 1.2 商户凭证

接入前需获取以下凭证：

| 参数 | 说明 |
|------|------|
| `appid` | 商户唯一标识 |
| `appsecret` | 商户密钥（用于签名，请妥善保管） |

### 1.3 链类型说明

| chain_type | 链类型 |
|------------|--------|
| 1 | TRC20（波场链） |
| 2 | ERC20（以太坊链） |

---

## 2. 签名规则

### 2.1 签名步骤

1. **过滤参数**：去除 `signature` 参数和空值参数
2. **参数排序**：将剩余参数按照 key 的 ASCII 码升序排序
3. **拼接字符串**：使用 `key=value&` 格式拼接，末尾追加 `&appsecret=您的密钥`
4. **MD5加密**：对拼接后的字符串进行 MD5 加密，结果转为大写

### 2.2 签名示例

**请求参数：**
```json
{
    "appid": "your_appid",
    "order_sn": "M202601010001",
    "pay_money": "100.00",
    "timestamp": "1735689600"
}
```

**签名过程：**
```
1. 按 key 排序: appid, order_sn, pay_money, timestamp
2. 拼接字符串: appid=your_appid&order_sn=M202601010001&pay_money=100.00&timestamp=1735689600&appsecret=您的密钥
3. MD5加密并转大写: 9A8B7C6D5E4F3A2B1C0D...
```

### 2.3 PHP 签名代码示例

```php
function get_signature($data, $appsecret) {
    // 过滤空值和签名字段
    foreach ($data as $key => $value) {
        if ($key == "signature" || $value === '') {
            unset($data[$key]);
        }
    }
    // 按 key 排序
    ksort($data);
    // 拼接并加密
    $sign = strtoupper(md5(urldecode(http_build_query($data)) . '&appsecret=' . $appsecret));
    return $sign;
}
```

### 2.4 Java 签名代码示例

```java
import java.security.MessageDigest;
import java.util.*;

public class SignUtil {
    public static String getSignature(Map<String, String> data, String appsecret) {
        // 过滤空值和签名字段
        Map<String, String> filteredData = new TreeMap<>();
        for (Map.Entry<String, String> entry : data.entrySet()) {
            if (!"signature".equals(entry.getKey()) && entry.getValue() != null && !entry.getValue().isEmpty()) {
                filteredData.put(entry.getKey(), entry.getValue());
            }
        }
        
        // 拼接字符串
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<String, String> entry : filteredData.entrySet()) {
            sb.append(entry.getKey()).append("=").append(entry.getValue()).append("&");
        }
        sb.append("appsecret=").append(appsecret);
        
        // MD5加密并转大写
        return md5(sb.toString()).toUpperCase();
    }
    
    private static String md5(String str) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] bytes = md.digest(str.getBytes("UTF-8"));
            StringBuilder sb = new StringBuilder();
            for (byte b : bytes) {
                sb.append(String.format("%02x", b));
            }
            return sb.toString();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 2.5 Python 签名代码示例

```python
import hashlib
from urllib.parse import urlencode

def get_signature(data: dict, appsecret: str) -> str:
    # 过滤空值和签名字段
    filtered_data = {k: v for k, v in data.items() if k != 'signature' and v}
    
    # 按 key 排序
    sorted_data = dict(sorted(filtered_data.items()))
    
    # 拼接字符串
    query_string = urlencode(sorted_data) + '&appsecret=' + appsecret
    
    # MD5加密并转大写
    return hashlib.md5(query_string.encode('utf-8')).hexdigest().upper()
```

### 2.6 timestamp 说明

- `timestamp` 为当前 Unix 时间戳（秒级）
- 服务器验证签名有效期为 **5分钟**，请确保服务器时间准确

---

## 3. 代收接口

### 3.1 统一下单

**接口地址：** `/api/pay/unifiedorder`

**请求参数：**

| 参数 | 必填 | 类型 | 说明 |
|------|------|------|------|
| appid | 是 | string | 商户ID |
| order_sn | 是 | string | 商户订单号（唯一） |
| pay_money | 是 | string | 支付金额（USDT） |
| chain_type | 是 | int | 链类型：1=TRC20，2=ERC20 |
| notify_url | 是 | string | 异步通知地址 |
| callback_url | 否 | string | 同步回调地址 |
| product_name | 否 | string | 商品名称 |
| product_desc | 否 | string | 商品描述 |
| product_num | 否 | int | 商品数量 |
| pay_username | 是 | string | 付款用户标识（用于会员独立地址分配） |
| attach | 否 | string | 附加数据（原样返回） |
| timestamp | 是 | int | 时间戳 |
| signature | 是 | string | 签名 |

**响应参数：**

```json
{
    "code": 1,
    "msg": "success",
    "data": {
        "appid": "your_appid",
        "order_sn": "M202601010001",
        "pay_usdt": 100.00,
        "address": "TXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "chain_type": 1,
        "pay_url": "https://域名/payment/index/order/id/M202601010001",
        "en_pay_url": "https://域名/payment/index/en_order/id/M202601010001",
        "exchange_rate": 7.25,
        "time_out": 1735693200,
        "signature": "9A8B7C6D5E4F3A2B..."
    }
}
```

**响应字段说明：**

| 字段 | 说明 |
|------|------|
| pay_usdt | 实际需支付的 USDT 金额 |
| address | 收款地址 |
| pay_url | 中文支付页面 URL |
| en_pay_url | 英文支付页面 URL |
| time_out | 订单超时时间戳 |

**PHP 请求示例：**

```php
$data = [
    'appid' => 'your_appid',
    'order_sn' => 'M' . date('YmdHis') . rand(1000, 9999),
    'pay_money' => '100.00',
    'chain_type' => 1,
    'notify_url' => 'https://your-domain.com/notify',
    'product_name' => '测试商品',
    'timestamp' => time(),
];
$data['signature'] = get_signature($data, 'your_appsecret');

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://api-domain.com/api/pay/unifiedorder');
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);

$result = json_decode($response, true);
```

---

### 3.2 订单查询

**接口地址：** `/api/pay/search`

**请求参数：**

| 参数 | 必填 | 类型 | 说明 |
|------|------|------|------|
| appid | 是 | string | 商户ID |
| order_sn | 是 | string | 商户订单号 |
| timestamp | 是 | int | 时间戳 |
| signature | 是 | string | 签名 |

**响应参数：**

```json
{
    "code": 1,
    "msg": "success",
    "data": {
        "appid": "your_appid",
        "order_sn": "M202601010001",
        "pay_usdt": 100.00,
        "pay_money": 725.00,
        "status": 1,
        "success_time": 1735689900,
        "attach": "",
        "signature": "xxx"
    }
}
```

**订单状态说明：**

| status | 说明 |
|--------|------|
| 0 | 未支付 |
| 1 | 支付成功 |
| 2 | 已超时 |

---

### 3.3 汇率查询

**接口地址：** `/api/pay/exchange_rate`

**请求方式：** GET/POST

**响应示例：**

```json
{
    "code": 1,
    "msg": "success",
    "data": 7.25
}
```

---

## 4. 代付接口

### 4.1 代付申请

**接口地址：** `/api/df/apply`

**请求参数：**

| 参数 | 必填 | 类型 | 说明 |
|------|------|------|------|
| appid | 是 | string | 商户ID |
| df_sn | 是 | string | 商户代付单号（唯一） |
| money | 是 | string | 代付金额（USDT） |
| receive_address | 是 | string | 收款地址 |
| chain_type | 是 | int | 链类型：1=TRC20，2=ERC20 |
| notify_url | 是 | string | 异步通知地址 |
| attach | 否 | string | 附加数据 |
| timestamp | 是 | int | 时间戳 |
| signature | 是 | string | 签名 |

**响应参数：**

```json
{
    "code": 1,
    "msg": "申请代付成功",
    "data": null
}
```

**注意事项：**

- 代付金额将从商户余额中扣除
- 代付手续费另计，从余额中额外扣除
- 请确保商户余额充足（代付金额 + 手续费）
- 收款地址必须与链类型匹配（TRC20 地址以 T 开头，ERC20 地址以 0x 开头）

**PHP 请求示例：**

```php
$data = [
    'appid' => 'your_appid',
    'df_sn' => 'DF' . date('YmdHis') . rand(1000, 9999),
    'money' => '50.00',
    'receive_address' => 'TXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    'chain_type' => 1,
    'notify_url' => 'https://your-domain.com/df_notify',
    'timestamp' => time(),
];
$data['signature'] = get_signature($data, 'your_appsecret');

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, 'https://api-domain.com/api/df/apply');
curl_setopt($ch, CURLOPT_POST, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data));
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($ch);
curl_close($ch);

$result = json_decode($response, true);
```

---

### 4.2 代付查询

**接口地址：** `/api/df/search`

**请求参数：**

| 参数 | 必填 | 类型 | 说明 |
|------|------|------|------|
| appid | 是 | string | 商户ID |
| df_sn | 是 | string | 商户代付单号 |
| timestamp | 是 | int | 时间戳 |
| signature | 是 | string | 签名 |

**响应参数：**

```json
{
    "code": 1,
    "msg": "success",
    "data": {
        "appid": "your_appid",
        "df_sn": "DF202601010001",
        "money": "100.00",
        "status": 1,
        "success_time": 1735689900,
        "signature": "xxx"
    }
}
```

**代付状态说明：**

| status | 说明 |
|--------|------|
| 0 | 待处理 |
| 1 | 已完成 |
| 2 | 已拒绝/失败 |

---

### 4.3 余额查询

**接口地址：** `/api/df/balance`

**请求参数：**

| 参数 | 必填 | 类型 | 说明 |
|------|------|------|------|
| appid | 是 | string | 商户ID |
| timestamp | 是 | int | 时间戳 |
| signature | 是 | string | 签名 |

**响应参数：**

```json
{
    "code": 1,
    "msg": "success",
    "data": {
        "appid": "your_appid",
        "usdt_balance": "1000.00"
    }
}
```

---

## 5. 异步通知

### 5.1 代收回调通知

当订单支付成功后，系统会向商户的 `notify_url` 发送 POST 请求。

**通知参数：**

| 参数 | 说明 |
|------|------|
| appid | 商户ID |
| order_sn | 商户订单号 |
| pay_money | 支付金额（法币） |
| pay_usdt | 实际支付 USDT 金额 |
| status | 订单状态（1=成功） |
| success_time | 支付成功时间戳 |
| attach | 附加数据 |
| signature | 签名 |

**响应要求：**

- 收到通知后，请返回纯文本 `OK`（大写）
- 返回 `OK` 后，系统将停止通知
- 未返回 `OK`，系统将重试通知（最多 5 次）

**重试策略：**

| 次数 | 间隔 |
|------|------|
| 第1次 | 立即 |
| 第2次 | 20秒后 |
| 第3次 | 30秒后 |
| 第4次 | 1分钟后 |
| 第5次 | 5分钟后 |

---

### 5.2 代付回调通知

当代付完成后，系统会向商户的 `notify_url` 发送 POST 请求。

**通知参数：**

| 参数 | 说明 |
|------|------|
| appid | 商户ID |
| df_sn | 商户代付单号 |
| money | 代付金额 |
| status | 订单状态（1=成功） |
| success_time | 完成时间戳 |
| signature | 签名 |

**响应要求：** 同代收回调

---

### 5.3 回调处理示例（PHP）

```php
<?php
// 接收回调通知
$postData = $_POST;

// 获取签名
$receivedSign = $postData['signature'] ?? '';

// 您的 appsecret
$appsecret = 'your_appsecret';

// 验证签名
$expectedSign = get_signature($postData, $appsecret);

if ($receivedSign !== $expectedSign) {
    // 签名验证失败
    echo 'FAIL';
    exit;
}

// 签名验证通过，处理业务逻辑
$orderSn = $postData['order_sn'] ?? $postData['df_sn'];
$status = $postData['status'];

if ($status == 1) {
    // 支付/代付成功，更新本地订单状态
    // TODO: 更新数据库订单状态
    
    // 注意：请做好幂等处理，避免重复处理
}

// 返回 OK 停止通知
echo 'OK';

// 签名函数
function get_signature($data, $appsecret) {
    foreach ($data as $key => $value) {
        if ($key == "signature" || $value === '') {
            unset($data[$key]);
        }
    }
    ksort($data);
    return strtoupper(md5(urldecode(http_build_query($data)) . '&appsecret=' . $appsecret));
}
```

---

## 6. 附录

### 6.1 错误码说明

| 错误码 | 说明 |
|--------|------|
| 1 | 成功 |
| -1 | 通用错误 |
| 10001 | 商户不存在 |
| 10003 | 订单入库失败 |
| 10005 | 商户不存在 |
| 10006 | 订单不存在 |
| 10010 | 代付订单不存在 |

### 6.2 常见错误信息

| 错误信息 | 说明 |
|----------|------|
| appid必须 | 缺少 appid 参数 |
| 商户不存在 | appid 无效或商户被禁用 |
| 缺少timestamp | 缺少时间戳参数 |
| 请求已过期 | 时间戳超过有效期（5分钟） |
| 缺少签名 | 缺少 signature 参数 |
| 签名错误 | 签名验证失败 |
| 商户订单号重复 | 订单号已存在 |
| 金额不能低于X | 金额低于最小限额 |
| 单笔金额不能超过X | 金额超过最大限额 |
| 请求过于频繁 | 触发接口限频 |
| 商户TRC的USDT余额不足 | 代付时余额不足 |
| 该链路目前不支持 | chain_type 参数错误 |
| 收款地址不是TRC地址 | TRC20 地址格式错误 |
| 收款地址不是ETH地址 | ERC20 地址格式错误 |

### 6.3 接口限频

| 接口 | 限制 |
|------|------|
| 统一下单 | 10次/分钟/IP |
| 订单查询 | 20次/分钟/IP |
| 创建地址 | 5次/分钟/IP |

### 6.4 注意事项

1. **签名验证**：所有请求必须携带有效签名，签名错误将被拒绝
2. **时间戳**：请确保服务器时间准确，时间戳有效期为 5 分钟
3. **订单号**：商户订单号必须唯一，重复订单号将被拒绝
4. **回调处理**：请做好幂等处理，同一笔订单可能收到多次回调通知
5. **HTTPS**：生产环境请使用 HTTPS 协议
6. **地址格式**：
   - TRC20 地址以 `T` 开头，长度 34 位
   - ERC20 地址以 `0x` 开头，长度 42 位

### 6.5 测试环境

| 项目 | 说明 |
|------|------|
| 商户后台 | `https://域名/merchant` |
| 测试账号 | 请联系客服获取 |

### 6.6 联系方式

如有技术问题，请联系：
- Telegram: @qy6566

---

## 更新日志

| 版本 | 日期 | 内容 |
|------|------|------|
| v1.0 | 2026-01-01 | 初始版本 |
