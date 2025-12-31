# 代收订单 API 说明

## 概述

代收订单用于商户向用户收取款项。用户支付到指定的收款地址后，系统确认收款成功后会向商户发送异步通知。

---

## 一、创建代收订单

### 请求地址

```
POST /api/order/collect/create
```

### 请求参数

| 参数名           | 类型     | 必填 | 说明                                       |
|-----------------|----------|------|-------------------------------------------|
| merchantNumber  | string   | 是   | 商户编号                                   |
| merchantOrderNo | string   | 是   | 商户订单号，商户系统内唯一                   |
| amount          | number   | 是   | 订单金额，单位：商户平台货币                 |
| currencyType    | string   | 是   | 货币类型：cny=人民币，usdt=USDT 等          |
| networkType     | number   | 是   | 链路类型：1=TRC-20，2=ERC-20，3=BEP-20      |
| notifyUrl       | string   | 是   | 异步通知地址                               |
| returnUrl       | string   | 否   | 页面跳转地址（支付完成后跳转）               |
| memberId        | string   | 否   | 商户会员ID                                 |
| extra           | string   | 否   | 附加字段，将原样返回                        |
| timestamp       | number   | 是   | 时间戳（秒）                               |
| sign            | string   | 是   | 签名                                      |

### 返回参数

| 参数名           | 类型     | 说明                          |
|-----------------|----------|-------------------------------|
| orderNo         | string   | 系统订单号                     |
| merchantOrderNo | string   | 商户订单号                     |
| amount          | number   | 订单金额                       |
| currencyType    | string   | 货币类型                       |
| payableAmount   | number   | 应付金额（USDT）               |
| networkType     | number   | 链路类型                       |
| receiveAddress  | string   | 收款地址                       |
| expireTime      | string   | 订单过期时间                    |

### 返回示例

```json
{
  "code": 1000,
  "message": "success",
  "data": {
    "orderNo": "C17356789012341234",
    "merchantOrderNo": "ORDER_20250101_001",
    "amount": 100.00,
    "currencyType": "cny",
    "payableAmount": 14.285714,
    "networkType": 1,
    "receiveAddress": "TXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "expireTime": "2025-01-01 12:30:00"
  }
}
```

---

## 二、查询代收订单

### 请求地址

```
POST /api/order/collect/query
```

### 请求参数

| 参数名           | 类型     | 必填 | 说明                                  |
|-----------------|----------|------|---------------------------------------|
| merchantNumber  | string   | 是   | 商户编号                              |
| orderNo         | string   | 否   | 系统订单号（与商户订单号二选一）        |
| merchantOrderNo | string   | 否   | 商户订单号（与系统订单号二选一）        |
| timestamp       | number   | 是   | 时间戳（秒）                          |
| sign            | string   | 是   | 签名                                  |

### 返回参数

| 参数名           | 类型     | 说明                          |
|-----------------|----------|-------------------------------|
| orderNo         | string   | 系统订单号                     |
| merchantOrderNo | string   | 商户订单号                     |
| amount          | number   | 订单金额                       |
| currencyType    | string   | 货币类型                       |
| payableAmount   | number   | 应付金额（USDT）               |
| actualAmount    | number   | 实际到账金额（USDT）           |
| fee             | number   | 手续费（USDT）                 |
| networkType     | number   | 链路类型                       |
| receiveAddress  | string   | 收款地址                       |
| status          | number   | 订单状态                       |
| paidTime        | string   | 支付时间                       |
| extra           | string   | 附加字段                       |
| createTime      | string   | 创建时间                       |

### 返回示例

```json
{
  "code": 1000,
  "message": "success",
  "data": {
    "orderNo": "C17356789012341234",
    "merchantOrderNo": "ORDER_20250101_001",
    "amount": 100.00,
    "currencyType": "cny",
    "payableAmount": 14.285714,
    "actualAmount": 14.285714,
    "fee": 0.428571,
    "networkType": 1,
    "receiveAddress": "TXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "status": 2,
    "paidTime": "2025-01-01 12:15:30",
    "extra": "自定义数据",
    "createTime": "2025-01-01 12:00:00"
  }
}
```

---

## 三、订单状态说明

| 状态值 | 状态名称   | 说明                         |
|-------|-----------|------------------------------|
| 0     | 待支付    | 订单已创建，等待用户支付       |
| 1     | 支付中    | 检测到链上转账，等待确认       |
| 2     | 支付成功  | 收款成功，金额已入账           |
| 3     | 支付失败  | 支付失败或金额不符             |
| 4     | 已取消    | 订单已被取消                   |
| 5     | 已过期    | 订单超时未支付                 |

---

## 四、货币类型说明

| 类型值  | 说明                                    |
|--------|----------------------------------------|
| cny    | 人民币                                  |
| usdt   | USDT                                   |

> 注：具体支持的货币类型以系统数据字典 `currency_type` 配置为准

---

## 五、链路类型说明

| 类型值 | 说明                                    |
|-------|----------------------------------------|
| 1     | TRC-20 (波场链)                         |
| 2     | ERC-20 (以太坊链)                       |
| 3     | BEP-20 (币安智能链)                     |

---

## 六、异步通知

### 通知地址

商户在创建订单时提供的 `notifyUrl`

### 通知参数

| 参数名           | 类型     | 说明                              |
|-----------------|----------|----------------------------------|
| merchantNumber  | string   | 商户号                            |
| orderAmount     | string   | 订单金额（商户提交的原始金额）      |
| actualAmount    | string   | 实际到账金额（USDT）              |
| fee             | string   | 手续费（USDT）                    |
| merchantOrderNo | string   | 商户订单号                        |
| orderNo         | string   | 系统订单号                        |
| status          | number   | 订单状态（2=成功，3=失败）         |
| paidTime        | string   | 支付时间                          |
| extra           | string   | 用户附加数据（原样返回）           |
| signature       | string   | 数据签名                          |

### 通知规则

1. 订单状态变更为 **支付成功(2)** 或 **支付失败(3)** 时，系统会自动发送通知
2. 商户需要返回 **OK**（大写）表示接收成功
3. 如果商户返回其他内容，系统将进行重试
4. 重试间隔：5秒、10秒、20秒、1分钟、5分钟
5. 最多重试 **5次**

### 签名验证

商户收到通知后，应验证签名以确保通知来源可靠。

签名规则：
1. 将所有参数（除 `signature` 外）按字母顺序排序
2. 拼接为 `key1=value1&key2=value2&...` 格式
3. 最后追加 `&key=商户密钥`
4. 对整个字符串进行 MD5 加密，转为大写

---

## 七、签名算法

### 签名规则

1. 将所有非空参数（不含 `sign` 字段）按参数名 ASCII 码从小到大排序
2. 使用 URL 键值对的格式（即 `key1=value1&key2=value2…`）拼接成字符串
3. 在拼接的字符串最后追加 `&key=商户密钥`
4. 对整个字符串进行 MD5 运算
5. 将得到的 MD5 值转为 **大写**

### 签名示例

假设参数如下：
```
merchantNumber=M123456
merchantOrderNo=ORDER_001
amount=100.00
currencyType=cny
networkType=1
notifyUrl=https://example.com/notify
timestamp=1703555555
```

商户密钥：`abcdefghijk123456`

**Step 1: 按字母排序并拼接**
```
amount=100.00&currencyType=cny&merchantNumber=M123456&merchantOrderNo=ORDER_001&networkType=1&notifyUrl=https://example.com/notify&timestamp=1703555555
```

**Step 2: 追加密钥**
```
amount=100.00&currencyType=cny&merchantNumber=M123456&merchantOrderNo=ORDER_001&networkType=1&notifyUrl=https://example.com/notify&timestamp=1703555555&key=abcdefghijk123456
```

**Step 3: MD5 加密并转大写**
```
sign = MD5(上述字符串).toUpperCase()
```

---

## 八、代码示例

### PHP 示例

```php
<?php
class CollectOrderApi
{
    private $baseUrl = 'https://api.example.com';
    private $merchantNumber = 'M123456';
    private $merchantKey = 'your_merchant_key';

    /**
     * 创建代收订单
     * @param string $currencyType 货币类型：cny, usdt 等
     */
    public function createOrder($merchantOrderNo, $amount, $currencyType, $networkType, $notifyUrl, $extra = '')
    {
        $params = [
            'merchantNumber' => $this->merchantNumber,
            'merchantOrderNo' => $merchantOrderNo,
            'amount' => $amount,
            'currencyType' => $currencyType, // 字符串类型：cny, usdt
            'networkType' => $networkType,
            'notifyUrl' => $notifyUrl,
            'extra' => $extra,
            'timestamp' => time(),
        ];

        $params['sign'] = $this->generateSign($params);

        return $this->post('/api/order/collect/create', $params);
    }

    /**
     * 查询订单
     */
    public function queryOrder($merchantOrderNo = '', $orderNo = '')
    {
        $params = [
            'merchantNumber' => $this->merchantNumber,
            'timestamp' => time(),
        ];

        if ($merchantOrderNo) {
            $params['merchantOrderNo'] = $merchantOrderNo;
        }
        if ($orderNo) {
            $params['orderNo'] = $orderNo;
        }

        $params['sign'] = $this->generateSign($params);

        return $this->post('/api/order/collect/query', $params);
    }

    /**
     * 验证通知签名
     */
    public function verifyNotify($params)
    {
        $signature = $params['signature'] ?? '';
        unset($params['signature']);
        
        $calculatedSign = $this->generateSign($params);
        
        return $signature === $calculatedSign;
    }

    /**
     * 生成签名
     */
    private function generateSign($params)
    {
        // 过滤空值
        $params = array_filter($params, function ($value) {
            return $value !== '' && $value !== null;
        });

        // 按字母排序
        ksort($params);

        // 拼接字符串
        $signStr = '';
        foreach ($params as $key => $value) {
            $signStr .= "{$key}={$value}&";
        }
        $signStr .= "key={$this->merchantKey}";

        // MD5加密
        return strtoupper(md5($signStr));
    }

    /**
     * POST 请求
     */
    private function post($path, $params)
    {
        $url = $this->baseUrl . $path;
        
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($params));
        curl_setopt($ch, CURLOPT_HTTPHEADER, ['Content-Type: application/json']);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_TIMEOUT, 30);
        
        $response = curl_exec($ch);
        curl_close($ch);
        
        return json_decode($response, true);
    }
}

// 使用示例
$api = new CollectOrderApi();

// 创建代收订单
$result = $api->createOrder(
    'ORDER_' . time(),
    100.00,
    'cny', // 货币类型
    1, // TRC-20
    'https://your-domain.com/notify',
    '附加数据'
);
print_r($result);

// 查询订单
$result = $api->queryOrder('ORDER_123456');
print_r($result);
?>
```

### Java 示例

```java
import java.security.MessageDigest;
import java.util.*;
import java.net.http.*;
import java.net.URI;

public class CollectOrderApi {
    private String baseUrl = "https://api.example.com";
    private String merchantNumber = "M123456";
    private String merchantKey = "your_merchant_key";

    /**
     * 创建代收订单
     * @param currencyType 货币类型：cny, usdt 等
     */
    public String createOrder(String merchantOrderNo, double amount, String currencyType,
                              int networkType, String notifyUrl, String extra) throws Exception {
        Map<String, Object> params = new TreeMap<>();
        params.put("merchantNumber", merchantNumber);
        params.put("merchantOrderNo", merchantOrderNo);
        params.put("amount", amount);
        params.put("currencyType", currencyType); // 字符串类型：cny, usdt
        params.put("networkType", networkType);
        params.put("notifyUrl", notifyUrl);
        if (extra != null && !extra.isEmpty()) {
            params.put("extra", extra);
        }
        params.put("timestamp", System.currentTimeMillis() / 1000);
        
        params.put("sign", generateSign(params));
        
        return post("/api/order/collect/create", params);
    }

    /**
     * 验证通知签名
     */
    public boolean verifyNotify(Map<String, Object> params) {
        String signature = (String) params.remove("signature");
        String calculatedSign = generateSign(params);
        return signature != null && signature.equals(calculatedSign);
    }

    /**
     * 生成签名
     */
    private String generateSign(Map<String, Object> params) {
        // 过滤空值并排序
        TreeMap<String, Object> sortedParams = new TreeMap<>();
        for (Map.Entry<String, Object> entry : params.entrySet()) {
            if (entry.getValue() != null && !entry.getValue().toString().isEmpty()) {
                sortedParams.put(entry.getKey(), entry.getValue());
            }
        }
        
        // 拼接字符串
        StringBuilder sb = new StringBuilder();
        for (Map.Entry<String, Object> entry : sortedParams.entrySet()) {
            sb.append(entry.getKey()).append("=").append(entry.getValue()).append("&");
        }
        sb.append("key=").append(merchantKey);
        
        // MD5加密
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] digest = md.digest(sb.toString().getBytes("UTF-8"));
            StringBuilder hexString = new StringBuilder();
            for (byte b : digest) {
                String hex = Integer.toHexString(0xff & b);
                if (hex.length() == 1) hexString.append('0');
                hexString.append(hex);
            }
            return hexString.toString().toUpperCase();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private String post(String path, Map<String, Object> params) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        String json = new com.google.gson.Gson().toJson(params);
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(baseUrl + path))
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(json))
            .build();
            
        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
        return response.body();
    }
}
```

### Node.js 示例

```javascript
const crypto = require('crypto');
const axios = require('axios');

class CollectOrderApi {
    constructor() {
        this.baseUrl = 'https://api.example.com';
        this.merchantNumber = 'M123456';
        this.merchantKey = 'your_merchant_key';
    }

    /**
     * 创建代收订单
     * @param {string} currencyType 货币类型：cny, usdt 等
     */
    async createOrder(merchantOrderNo, amount, currencyType, networkType, notifyUrl, extra = '') {
        const params = {
            merchantNumber: this.merchantNumber,
            merchantOrderNo,
            amount,
            currencyType, // 字符串类型：cny, usdt
            networkType,
            notifyUrl,
            timestamp: Math.floor(Date.now() / 1000),
        };
        
        if (extra) {
            params.extra = extra;
        }
        
        params.sign = this.generateSign(params);
        
        return this.post('/api/order/collect/create', params);
    }

    /**
     * 查询订单
     */
    async queryOrder(merchantOrderNo = '', orderNo = '') {
        const params = {
            merchantNumber: this.merchantNumber,
            timestamp: Math.floor(Date.now() / 1000),
        };
        
        if (merchantOrderNo) params.merchantOrderNo = merchantOrderNo;
        if (orderNo) params.orderNo = orderNo;
        
        params.sign = this.generateSign(params);
        
        return this.post('/api/order/collect/query', params);
    }

    /**
     * 验证通知签名
     */
    verifyNotify(params) {
        const signature = params.signature;
        delete params.signature;
        
        const calculatedSign = this.generateSign(params);
        return signature === calculatedSign;
    }

    /**
     * 生成签名
     */
    generateSign(params) {
        // 过滤空值
        const filteredParams = {};
        for (const key of Object.keys(params)) {
            if (params[key] !== '' && params[key] !== null && params[key] !== undefined) {
                filteredParams[key] = params[key];
            }
        }
        
        // 按字母排序
        const sortedKeys = Object.keys(filteredParams).sort();
        
        // 拼接字符串
        let signStr = sortedKeys.map(key => `${key}=${filteredParams[key]}`).join('&');
        signStr += `&key=${this.merchantKey}`;
        
        // MD5加密
        return crypto.createHash('md5').update(signStr).digest('hex').toUpperCase();
    }

    async post(path, params) {
        try {
            const response = await axios.post(this.baseUrl + path, params, {
                headers: { 'Content-Type': 'application/json' },
                timeout: 30000,
            });
            return response.data;
        } catch (error) {
            throw error;
        }
    }
}

// 使用示例
const api = new CollectOrderApi();

// 创建代收订单
api.createOrder(
    'ORDER_' + Date.now(),
    100.00,
    'cny', // 货币类型
    1, // TRC-20
    'https://your-domain.com/notify',
    '附加数据'
).then(console.log).catch(console.error);

// 处理异步通知
// Express 示例
app.post('/notify', (req, res) => {
    const params = req.body;
    
    if (api.verifyNotify({...params})) {
        console.log('签名验证成功');
        console.log('订单状态:', params.status);
        console.log('实际到账:', params.actualAmount);
        
        // 处理业务逻辑...
        
        res.send('OK'); // 返回 OK 表示接收成功
    } else {
        console.log('签名验证失败');
        res.send('FAIL');
    }
});
```

### Python 示例

```python
import hashlib
import time
import requests
import json

class CollectOrderApi:
    def __init__(self):
        self.base_url = 'https://api.example.com'
        self.merchant_number = 'M123456'
        self.merchant_key = 'your_merchant_key'
    
    def create_order(self, merchant_order_no, amount, currency_type, network_type, notify_url, extra=''):
        """
        创建代收订单
        :param currency_type: 货币类型，字符串：cny, usdt 等
        """
        params = {
            'merchantNumber': self.merchant_number,
            'merchantOrderNo': merchant_order_no,
            'amount': amount,
            'currencyType': currency_type,  # 字符串类型：cny, usdt
            'networkType': network_type,
            'notifyUrl': notify_url,
            'timestamp': int(time.time()),
        }
        
        if extra:
            params['extra'] = extra
        
        params['sign'] = self.generate_sign(params)
        
        return self.post('/api/order/collect/create', params)
    
    def query_order(self, merchant_order_no='', order_no=''):
        """查询订单"""
        params = {
            'merchantNumber': self.merchant_number,
            'timestamp': int(time.time()),
        }
        
        if merchant_order_no:
            params['merchantOrderNo'] = merchant_order_no
        if order_no:
            params['orderNo'] = order_no
        
        params['sign'] = self.generate_sign(params)
        
        return self.post('/api/order/collect/query', params)
    
    def verify_notify(self, params):
        """验证通知签名"""
        signature = params.pop('signature', '')
        calculated_sign = self.generate_sign(params)
        return signature == calculated_sign
    
    def generate_sign(self, params):
        """生成签名"""
        # 过滤空值
        filtered_params = {k: v for k, v in params.items() if v is not None and v != ''}
        
        # 按字母排序
        sorted_keys = sorted(filtered_params.keys())
        
        # 拼接字符串
        sign_str = '&'.join([f'{k}={filtered_params[k]}' for k in sorted_keys])
        sign_str += f'&key={self.merchant_key}'
        
        # MD5加密
        return hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()
    
    def post(self, path, params):
        """发送POST请求"""
        url = self.base_url + path
        headers = {'Content-Type': 'application/json'}
        
        response = requests.post(url, json=params, headers=headers, timeout=30)
        return response.json()


# 使用示例
if __name__ == '__main__':
    api = CollectOrderApi()
    
    # 创建代收订单
    result = api.create_order(
        f'ORDER_{int(time.time())}',
        100.00,
        'cny',  # 货币类型
        1,  # TRC-20
        'https://your-domain.com/notify',
        '附加数据'
    )
    print(result)
    
    # 查询订单
    result = api.query_order(merchant_order_no='ORDER_123456')
    print(result)


# Flask 处理异步通知示例
from flask import Flask, request

app = Flask(__name__)

@app.route('/notify', methods=['POST'])
def notify():
    params = request.json or request.form.to_dict()
    api = CollectOrderApi()
    
    if api.verify_notify(dict(params)):
        print('签名验证成功')
        print(f'订单状态: {params.get("status")}')
        print(f'实际到账: {params.get("actualAmount")}')
        
        # 处理业务逻辑...
        
        return 'OK'  # 返回 OK 表示接收成功
    else:
        print('签名验证失败')
        return 'FAIL'
```

---

## 九、业务流程

```
1. 商户系统调用 [创建代收订单] 接口
   ↓
2. 系统返回收款地址和订单信息
   ↓
3. 商户展示收款地址给用户
   ↓
4. 用户向收款地址转账 USDT
   ↓
5. 系统监控链上交易
   ↓
6. 检测到转账后，更新订单状态
   ↓
7. 系统向商户发送异步通知
   ↓
8. 商户收到通知，处理业务逻辑
   ↓
9. 商户返回 "OK"，完成交易
```

---

## 十、常见问题

### Q: 收款地址有效期多长？

A: 默认30分钟。超时未支付的订单将自动标记为"已过期"。

### Q: 用户转账金额与订单金额不符怎么办？

A: 系统会记录实际到账金额。如果金额不符，可能需要人工处理。

### Q: 通知失败怎么办？

A: 系统会自动重试5次（间隔5秒、10秒、20秒、1分钟、5分钟）。商户也可以主动调用查询接口获取订单状态。

### Q: 如何处理订单超时？

A: 建议商户在用户支付页面显示倒计时，超时后引导用户重新发起支付。

### Q: 如何测试接口？

A: 可以联系技术支持获取测试环境账号，在测试环境进行接口调试。

### Q: 手续费如何计算？

A: 手续费根据商户费率配置计算。分为固定费用和费率两种模式：
- 金额 ≤ 分界值：收取固定费用
- 金额 > 分界值：按费率百分比计算

---

## 十一、错误码说明

| 错误码 | 说明                     |
|-------|-------------------------|
| 1000  | 成功                     |
| 1001  | 参数错误                 |
| 1002  | 签名错误                 |
| 1003  | 商户不存在               |
| 1004  | 商户已禁用               |
| 1005  | 商户代收权限未开启        |
| 1006  | 商户订单号已存在          |
| 1007  | 暂无可用收款地址          |
| 1008  | 订单不存在               |
| 1009  | 金额超出限制              |

---

## 十二、联系方式

如有技术问题，请联系：
- 技术支持邮箱：tech@example.com
- 在线客服：https://example.com/support
