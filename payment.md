# 💳 代付订单 API 说明

## 接口概述

代付订单接口用于商户发起代付请求，系统将根据商户提交的信息进行代付处理。

**接口地址前缀：** `/api/order`

**请求方式：** `POST`

**Content-Type：** `application/json`

---

## 一、创建代付订单

### 接口地址

```
POST /api/order/payment/create
```

### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| merchantNumber | string | 是 | 商户编号 |
| merchantOrderNo | string | 是 | 商户订单号（唯一，不可重复） |
| amount | number | 是 | 订单金额（USDT） |
| networkType | number | 是 | 链路类型：1-TRC20，2-ERC20，3-BEP20 |
| receiveAddress | string | 是 | 收款地址 |
| notifyUrl | string | 是 | 异步通知地址 |
| currencyType | string | 否 | 货币类型（默认 usdt） |
| extra | string | 否 | 附加字段，原样返回 |
| timestamp | number | 是 | 时间戳（秒），有效期 5 分钟 |
| sign | string | 是 | 签名，详见签名算法 |

### 请求示例

```json
{
  "merchantNumber": "M123456",
  "merchantOrderNo": "PAY_20251231_001",
  "amount": 100.00,
  "networkType": 1,
  "receiveAddress": "TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP",
  "notifyUrl": "https://your-domain.com/api/payment/notify",
  "extra": "用户ID:12345",
  "timestamp": 1735632000,
  "sign": "5A8B9C0D1E2F3A4B5C6D7E8F9A0B1C2D"
}
```

### 返回参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| code | number | 状态码，1000 表示成功 |
| message | string | 返回信息 |
| data | object | 返回数据 |

**data 字段说明：**

| 参数名 | 类型 | 说明 |
|--------|------|------|
| orderNo | string | 系统订单号 |
| merchantOrderNo | string | 商户订单号 |
| amount | number | 订单金额 |
| currencyType | string | 货币类型 |
| exchangeRate | number | 汇率 |
| payableAmount | number | 应付金额（USDT） |
| withdrawFee | number | 手续费（USDT） |
| networkType | number | 链路类型 |
| receiveAddress | string | 收款地址 |
| status | number | 订单状态 |

### 成功返回示例

```json
{
  "code": 1000,
  "message": "success",
  "data": {
    "orderNo": "P17356320001234",
    "merchantOrderNo": "PAY_20251231_001",
    "amount": 100.00,
    "currencyType": "usdt",
    "exchangeRate": 1.0000,
    "payableAmount": 100.00,
    "withdrawFee": 2.00,
    "networkType": 1,
    "receiveAddress": "TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP",
    "status": 1
  }
}
```

### 失败返回示例

```json
{
  "code": 1001,
  "message": "商户余额不足，需要 102.00，当前余额 50.00"
}
```

---

## 二、查询代付订单

### 接口地址

```
POST /api/order/payment/query
```

### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| merchantNumber | string | 是 | 商户编号 |
| orderNo | string | 否 | 系统订单号（与商户订单号二选一） |
| merchantOrderNo | string | 否 | 商户订单号（与系统订单号二选一） |
| timestamp | number | 是 | 时间戳（秒） |
| sign | string | 是 | 签名 |

### 请求示例

```json
{
  "merchantNumber": "M123456",
  "merchantOrderNo": "PAY_20251231_001",
  "timestamp": 1735632000,
  "sign": "5A8B9C0D1E2F3A4B5C6D7E8F9A0B1C2D"
}
```

### 返回参数

| 参数名 | 类型 | 说明 |
|--------|------|------|
| orderNo | string | 系统订单号 |
| merchantOrderNo | string | 商户订单号 |
| amount | number | 订单金额 |
| currencyType | string | 货币类型 |
| exchangeRate | number | 汇率 |
| payableAmount | number | 应付金额（USDT） |
| withdrawFee | number | 手续费（USDT） |
| networkType | number | 链路类型 |
| receiveAddress | string | 收款地址 |
| status | number | 订单状态 |
| isConfirmed | number | 是否已确认：0-待确认，1-已确认 |
| txHash | string | 交易哈希 |
| paidTime | string | 支付时间 |
| extra | string | 附加字段 |
| createTime | string | 创建时间 |

### 成功返回示例

```json
{
  "code": 1000,
  "message": "success",
  "data": {
    "orderNo": "P17356320001234",
    "merchantOrderNo": "PAY_20251231_001",
    "amount": 100.00,
    "currencyType": "usdt",
    "exchangeRate": 1.0000,
    "payableAmount": 100.00,
    "withdrawFee": 2.00,
    "networkType": 1,
    "receiveAddress": "TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP",
    "status": 3,
    "isConfirmed": 1,
    "txHash": "0x1234567890abcdef...",
    "paidTime": "2025-12-31 18:30:00",
    "extra": "用户ID:12345",
    "createTime": "2025-12-31 18:00:00"
  }
}
```

---

## 三、异步通知

订单状态变更为支付成功或支付失败后，系统会自动向订单关联的回调地址（notify_url）发送通知消息。

**请求方式：** `POST`

**传参方式：** `application/x-www-form-urlencoded`

### 通知参数

| 字段名 | 类型 | 说明 |
|--------|------|------|
| merchantNumber | string | 商户号 |
| orderAmount | string | 订单金额（商户提交的原始金额） |
| currencyType | string | 货币类型（如 CNY、USD） |
| exchangeRate | string | 汇率 |
| payableAmount | string | 应付金额（USDT） |
| merchantOrderNo | string | 商户订单号 |
| orderNo | string | 系统订单号 |
| status | number | 订单状态：3=支付成功，4=支付失败 |
| paidTime | string | 支付时间（格式：YYYY-MM-DD HH:mm:ss） |
| extra | string | 用户附加数据（原样返回） |
| signature | string | 数据签名，详见签名算法 |

### 通知返回

商户在收到通知信息后，需在页面输出 `OK`（OK 两个字母大写），表示已成功接收通知。

**返回示例：**

```php
// PHP
echo "OK";
```

```java
// Java
response.getWriter().write("OK");
```

```javascript
// Node.js
res.send("OK");
```

```python
# Python
return "OK"
```

> ⚠️ **注意：** 返回内容必须是 `OK`（大写），没有双引号，否则系统会认为通知失败并进行重试。

### 通知重试规则

系统向商户的 `notify_url` 发送回调通知后，如返回的不是 `OK`，则系统会触发重试机制：

| 重试次数 | 间隔时间 |
|---------|---------|
| 第 1 次 | 5 秒 |
| 第 2 次 | 10 秒 |
| 第 3 次 | 20 秒 |
| 第 4 次 | 60 秒 |
| 第 5 次 | 300 秒（5分钟） |

> 💡 超过 5 次重试后如需推送，可以在后台手动补发通知。

### 通知验签示例

**PHP:**
```php
<?php
function verifySignature($params, $merchantKey) {
    $signature = $params['signature'] ?? '';
    unset($params['signature']);
    
    $params = array_filter($params, function($v) {
        return $v !== '' && $v !== null;
    });
    
    ksort($params);
    
    $signStr = '';
    foreach ($params as $key => $value) {
        $signStr .= $key . '=' . $value . '&';
    }
    $signStr .= 'key=' . $merchantKey;
    
    $expectedSign = strtoupper(md5($signStr));
    
    return $signature === $expectedSign;
}

// 使用示例
$params = $_POST;
$merchantKey = 'your-merchant-key';

if (verifySignature($params, $merchantKey)) {
    // 签名验证通过，处理业务逻辑
    // ...
    echo "OK";
} else {
    echo "签名验证失败";
}
?>
```

**Node.js:**
```javascript
const crypto = require('crypto');

function verifySignature(params, merchantKey) {
    const signature = params.signature;
    delete params.signature;
    
    const sortedKeys = Object.keys(params)
        .filter(key => params[key] !== '' && params[key] !== null && params[key] !== undefined)
        .sort();
    
    let signStr = sortedKeys.map(key => `${key}=${params[key]}`).join('&');
    signStr += `&key=${merchantKey}`;
    
    const expectedSign = crypto.createHash('md5').update(signStr).digest('hex').toUpperCase();
    
    return signature === expectedSign;
}

// Express 示例
app.post('/notify', (req, res) => {
    const params = req.body;
    const merchantKey = 'your-merchant-key';
    
    if (verifySignature({...params}, merchantKey)) {
        // 签名验证通过，处理业务逻辑
        res.send('OK');
    } else {
        res.send('签名验证失败');
    }
});
```

**Python:**
```python
import hashlib

def verify_signature(params: dict, merchant_key: str) -> bool:
    signature = params.pop('signature', '')
    
    filtered_params = {k: v for k, v in params.items() if v not in ('', None)}
    sorted_keys = sorted(filtered_params.keys())
    
    sign_str = '&'.join([f'{k}={filtered_params[k]}' for k in sorted_keys])
    sign_str += f'&key={merchant_key}'
    
    expected_sign = hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()
    
    return signature == expected_sign

# Flask 示例
@app.route('/notify', methods=['POST'])
def notify():
    params = request.form.to_dict()
    merchant_key = 'your-merchant-key'
    
    if verify_signature(params.copy(), merchant_key):
        return 'OK'
    else:
        return '签名验证失败'
```

---

## 四、订单状态说明

| 状态值 | 说明 |
|--------|------|
| 1 | 已提交 |
| 2 | 处理中 |
| 3 | 支付成功 |
| 4 | 支付失败 |
| 5 | 商户取消 |
| 6 | 后台取消 |
| 7 | API取消 |
| 8 | 订单超时 |

---

## 五、链路类型说明

| 类型值 | 说明 | 地址格式要求 |
|--------|------|-------------|
| 1 | TRC-20 | 以 `T` 开头 |
| 2 | ERC-20 | 以 `0x` 开头 |
| 3 | BEP-20 | 以 `0x` 开头 |

---

## 六、签名算法

### 签名规则

1. 将所有参数（除 `sign`/`signature` 外）按**参数名 ASCII 码从小到大排序**
2. 使用 `key=value` 格式拼接成字符串，参数之间用 `&` 连接
3. 在拼接字符串末尾加上 `&key=商户密钥`
4. 对拼接后的字符串进行 **MD5 加密**，并转换为**大写**

### 签名示例

假设请求参数如下：

```
merchantNumber=M123456
merchantOrderNo=PAY_20251231_001
amount=100.00
networkType=1
receiveAddress=TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP
notifyUrl=https://your-domain.com/api/payment/notify
timestamp=1735632000
```

商户密钥：`your-merchant-key`

**步骤 1：按参数名排序拼接**

```
amount=100&merchantNumber=M123456&merchantOrderNo=PAY_20251231_001&networkType=1&notifyUrl=https://your-domain.com/api/payment/notify&receiveAddress=TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP&timestamp=1735632000
```

**步骤 2：加上商户密钥**

```
amount=100&merchantNumber=M123456&merchantOrderNo=PAY_20251231_001&networkType=1&notifyUrl=https://your-domain.com/api/payment/notify&receiveAddress=TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP&timestamp=1735632000&key=your-merchant-key
```

**步骤 3：MD5 加密并转大写**

```
sign=5A8B9C0D1E2F3A4B5C6D7E8F9A0B1C2D
```

---

## 七、完整示例代码

### PHP

```php
<?php
class PaymentApi {
    private $apiUrl = 'https://api-domain.com';
    private $merchantNumber;
    private $merchantKey;
    
    public function __construct($merchantNumber, $merchantKey) {
        $this->merchantNumber = $merchantNumber;
        $this->merchantKey = $merchantKey;
    }
    
    // 生成签名
    private function generateSign($params) {
        unset($params['sign']);
        $params = array_filter($params, function($v) {
            return $v !== '' && $v !== null;
        });
        ksort($params);
        
        $signStr = '';
        foreach ($params as $key => $value) {
            $signStr .= $key . '=' . $value . '&';
        }
        $signStr .= 'key=' . $this->merchantKey;
        
        return strtoupper(md5($signStr));
    }
    
    // 创建代付订单
    public function createPayment($merchantOrderNo, $amount, $networkType, $receiveAddress, $notifyUrl, $extra = '') {
        $params = [
            'merchantNumber' => $this->merchantNumber,
            'merchantOrderNo' => $merchantOrderNo,
            'amount' => $amount,
            'networkType' => $networkType,
            'receiveAddress' => $receiveAddress,
            'notifyUrl' => $notifyUrl,
            'extra' => $extra,
            'timestamp' => time(),
        ];
        
        $params['sign'] = $this->generateSign($params);
        
        return $this->post('/api/order/payment/create', $params);
    }
    
    // 查询订单
    public function queryPayment($merchantOrderNo = '', $orderNo = '') {
        $params = [
            'merchantNumber' => $this->merchantNumber,
            'merchantOrderNo' => $merchantOrderNo,
            'orderNo' => $orderNo,
            'timestamp' => time(),
        ];
        
        $params['sign'] = $this->generateSign($params);
        
        return $this->post('/api/order/payment/query', $params);
    }
    
    private function post($path, $params) {
        $ch = curl_init($this->apiUrl . $path);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($params));
        curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
        
        $response = curl_exec($ch);
        curl_close($ch);
        
        return json_decode($response, true);
    }
}

// 使用示例
$api = new PaymentApi('M123456', 'your-merchant-key');

// 创建订单
$result = $api->createPayment(
    'PAY_' . date('YmdHis') . '_' . rand(1000, 9999),
    100.00,
    1,
    'TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP',
    'https://your-domain.com/api/payment/notify'
);
print_r($result);

// 查询订单
$result = $api->queryPayment('PAY_20251231_001');
print_r($result);
?>
```

### Node.js

```javascript
const crypto = require('crypto');
const axios = require('axios');

class PaymentApi {
    constructor(merchantNumber, merchantKey, apiUrl = 'https://api-domain.com') {
        this.merchantNumber = merchantNumber;
        this.merchantKey = merchantKey;
        this.apiUrl = apiUrl;
    }
    
    generateSign(params) {
        const sortedKeys = Object.keys(params)
            .filter(key => key !== 'sign' && params[key] !== '' && params[key] !== null && params[key] !== undefined)
            .sort();
        
        let signStr = sortedKeys.map(key => `${key}=${params[key]}`).join('&');
        signStr += `&key=${this.merchantKey}`;
        
        return crypto.createHash('md5').update(signStr).digest('hex').toUpperCase();
    }
    
    async createPayment(merchantOrderNo, amount, networkType, receiveAddress, notifyUrl, extra = '') {
        const params = {
            merchantNumber: this.merchantNumber,
            merchantOrderNo,
            amount,
            networkType,
            receiveAddress,
            notifyUrl,
            extra,
            timestamp: Math.floor(Date.now() / 1000),
        };
        
        params.sign = this.generateSign(params);
        
        const response = await axios.post(`${this.apiUrl}/api/order/payment/create`, params);
        return response.data;
    }
    
    async queryPayment(merchantOrderNo = '', orderNo = '') {
        const params = {
            merchantNumber: this.merchantNumber,
            merchantOrderNo,
            orderNo,
            timestamp: Math.floor(Date.now() / 1000),
        };
        
        params.sign = this.generateSign(params);
        
        const response = await axios.post(`${this.apiUrl}/api/order/payment/query`, params);
        return response.data;
    }
}

// 使用示例
(async () => {
    const api = new PaymentApi('M123456', 'your-merchant-key');
    
    // 创建订单
    const createResult = await api.createPayment(
        `PAY_${Date.now()}_${Math.floor(Math.random() * 10000)}`,
        100.00,
        1,
        'TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP',
        'https://your-domain.com/api/payment/notify'
    );
    console.log('创建订单结果:', createResult);
    
    // 查询订单
    const queryResult = await api.queryPayment('PAY_20251231_001');
    console.log('查询订单结果:', queryResult);
})();
```

### Python

```python
import hashlib
import time
import requests

class PaymentApi:
    def __init__(self, merchant_number, merchant_key, api_url='https://api-domain.com'):
        self.merchant_number = merchant_number
        self.merchant_key = merchant_key
        self.api_url = api_url
    
    def generate_sign(self, params):
        filtered_params = {k: v for k, v in params.items() if k != 'sign' and v not in ('', None)}
        sorted_keys = sorted(filtered_params.keys())
        
        sign_str = '&'.join([f'{k}={filtered_params[k]}' for k in sorted_keys])
        sign_str += f'&key={self.merchant_key}'
        
        return hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()
    
    def create_payment(self, merchant_order_no, amount, network_type, receive_address, notify_url, extra=''):
        params = {
            'merchantNumber': self.merchant_number,
            'merchantOrderNo': merchant_order_no,
            'amount': amount,
            'networkType': network_type,
            'receiveAddress': receive_address,
            'notifyUrl': notify_url,
            'extra': extra,
            'timestamp': int(time.time()),
        }
        
        params['sign'] = self.generate_sign(params)
        
        response = requests.post(
            f'{self.api_url}/api/order/payment/create',
            json=params,
            headers={'Content-Type': 'application/json'}
        )
        return response.json()
    
    def query_payment(self, merchant_order_no='', order_no=''):
        params = {
            'merchantNumber': self.merchant_number,
            'merchantOrderNo': merchant_order_no,
            'orderNo': order_no,
            'timestamp': int(time.time()),
        }
        
        params['sign'] = self.generate_sign(params)
        
        response = requests.post(
            f'{self.api_url}/api/order/payment/query',
            json=params,
            headers={'Content-Type': 'application/json'}
        )
        return response.json()

# 使用示例
if __name__ == '__main__':
    api = PaymentApi('M123456', 'your-merchant-key')
    
    # 创建订单
    result = api.create_payment(
        f'PAY_{int(time.time())}_{hash(time.time()) % 10000}',
        100.00,
        1,
        'TDWtLxXos9pbSa9dvDCkpFufHAVmdS8iGP',
        'https://your-domain.com/api/payment/notify'
    )
    print('创建订单结果:', result)
    
    # 查询订单
    result = api.query_payment('PAY_20251231_001')
    print('查询订单结果:', result)
```

---

## 八、业务流程

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   商户平台   │      │   代付系统   │      │   区块链    │
└──────┬──────┘      └──────┬──────┘      └──────┬──────┘
       │                    │                    │
       │  1. 创建代付订单    │                    │
       │ ─────────────────> │                    │
       │                    │                    │
       │  2. 返回订单信息    │                    │
       │ <───────────────── │                    │
       │                    │                    │
       │                    │  3. 后台确认处理    │
       │                    │ ─────────────────> │
       │                    │                    │
       │                    │  4. 转账完成       │
       │                    │ <───────────────── │
       │                    │                    │
       │  5. 异步通知        │                    │
       │ <───────────────── │                    │
       │                    │                    │
       │  6. 返回 "OK"      │                    │
       │ ─────────────────> │                    │
       │                    │                    │
```

### 流程说明

1. **商户创建订单**：商户调用创建代付订单接口
2. **系统返回订单信息**：系统创建订单，扣除商户余额，返回订单信息
3. **后台确认处理**：运营人员在后台确认订单后进行代付操作
4. **转账完成**：链上转账完成
5. **异步通知**：系统向商户 `notifyUrl` 发送通知
6. **商户确认**：商户返回 `OK` 确认收到通知

---

## 九、注意事项

### 接口调用注意事项

1. **签名有效期**：时间戳必须在 5 分钟以内，超时请求将被拒绝

2. **订单号唯一**：商户订单号在商户维度下必须唯一，重复提交会返回错误

3. **余额检查**：创建订单时会检查商户余额，余额不足将返回错误

4. **地址格式**：收款地址必须符合对应链路类型的格式要求

5. **手续费说明**：手续费按笔收取，不同链路类型手续费可能不同

6. **IP 白名单**：如配置了 IP 白名单，请求 IP 必须在白名单内

### 异步通知注意事项

1. **幂等性处理**：同一订单可能会收到多次通知，商户需要根据 `orderNo` 或 `merchantOrderNo` 做幂等处理，避免重复业务处理

2. **超时设置**：系统发送通知的超时时间为 10 秒，请确保您的接口能在 10 秒内响应

3. **HTTPS 支持**：建议使用 HTTPS 协议的回调地址，确保数据传输安全

4. **日志记录**：建议记录每次收到的通知内容，便于问题排查

---

## 十、错误码说明

| 错误码 | 说明 |
|--------|------|
| 1000 | 成功 |
| 1001 | 业务错误（详见 message） |
| 400 | 请求参数错误 |
| 401 | 签名验证失败 |
| 403 | 权限不足（IP不在白名单、商户被禁用等） |

---

## 十一、常见问题

### Q: 创建订单后多久会处理？
A: 订单创建后需要后台运营人员确认，确认后会进行代付处理，具体时间视运营处理情况而定。

### Q: 如何查询订单状态？
A: 可以调用订单查询接口，或等待异步通知。

### Q: 订单失败后余额会退还吗？
A: 订单失败后，系统会自动退还代付金额和手续费到商户可用余额。

### Q: 异步通知失败怎么办？
A: 系统会自动重试 5 次（间隔 5秒、10秒、20秒、60秒、300秒），如仍失败可联系运营手动补发。

### Q: 为什么收不到通知？
A: 请检查以下几点：
- `notify_url` 是否正确填写
- 服务器是否能正常访问
- 是否有防火墙拦截
- 接口是否返回了 `OK`

### Q: 签名验证失败怎么办？
A: 请检查：
- 商户密钥是否正确
- 参数是否按照 ASCII 码排序
- 是否过滤了空值参数
- MD5 结果是否转为大写
