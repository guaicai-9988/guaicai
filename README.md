# 🔔 代付异步通知

用户支付完成后，系统会自动向订单关联的回调地址（notify_url）发送通知消息，告知该笔订单已支付完成。

**请求方式：** `POST`

**传参方式：** `application/x-www-form-urlencoded`

---

## 请求参数

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

---

## 订单状态说明

| 状态值 | 说明 |
|--------|------|
| 3 | 支付成功 |
| 4 | 支付失败 |

---

## 签名算法

### 签名规则

1. 将所有参数（除 `signature` 外）按**参数名 ASCII 码从小到大排序**
2. 使用 `key=value` 格式拼接成字符串，参数之间用 `&` 连接
3. 在拼接字符串末尾加上 `&key=商户密钥`
4. 对拼接后的字符串进行 **MD5 加密**，并转换为**大写**

### 签名示例

假设通知参数如下：
```
merchantNumber=M123456
orderAmount=100.00
currencyType=CNY
exchangeRate=7.2500
payableAmount=13.79
merchantOrderNo=PAY_001
orderNo=P17035555551234
status=3
paidTime=2025-12-31 18:30:00
extra=test
```

商户密钥：`your-merchant-key`

**步骤 1：按参数名排序拼接**
```
currencyType=CNY&exchangeRate=7.2500&extra=test&merchantNumber=M123456&merchantOrderNo=PAY_001&orderAmount=100.00&orderNo=P17035555551234&paidTime=2025-12-31 18:30:00&payableAmount=13.79&status=3
```

**步骤 2：加上商户密钥**
```
currencyType=CNY&exchangeRate=7.2500&extra=test&merchantNumber=M123456&merchantOrderNo=PAY_001&orderAmount=100.00&orderNo=P17035555551234&paidTime=2025-12-31 18:30:00&payableAmount=13.79&status=3&key=your-merchant-key
```

**步骤 3：MD5 加密并转大写**
```
signature=5A8B9C0D1E2F3A4B5C6D7E8F9A0B1C2D
```

---

## 通知返回

商户在收到通知信息后，需在页面输出 `OK`（OK 两个字母大写），表示已成功接收通知。

### 返回示例

**PHP:**
```php
echo "OK";
```

**Java:**
```java
response.getWriter().write("OK");
```

**Node.js:**
```javascript
res.send("OK");
```

**Python:**
```python
return "OK"
```

> ⚠️ **注意：** 返回内容必须是 `OK`（大写），没有双引号，否则系统会认为通知失败并进行重试。

---

## 通知重试

系统向商户创建订单时指定的 `notify_url` 发送回调通知后，如该 `notify_url` 返回的不是 `OK`（没有双引号，OK 两个字母大写），则系统会触发重试机制。

### 重试规则

| 重试次数 | 间隔时间 |
|---------|---------|
| 第 1 次 | 5 秒 |
| 第 2 次 | 10 秒 |
| 第 3 次 | 20 秒 |
| 第 4 次 | 60 秒 |
| 第 5 次 | 300 秒（5分钟） |

> 💡 超过 5 次重试后如需推送，可以在后台手动补发通知。

---

## 验签示例代码

### PHP
```php
<?php
function verifySignature($params, $merchantKey) {
    // 获取签名
    $signature = $params['signature'] ?? '';
    unset($params['signature']);
    
    // 过滤空值
    $params = array_filter($params, function($v) {
        return $v !== '' && $v !== null;
    });
    
    // 按参数名排序
    ksort($params);
    
    // 拼接字符串
    $signStr = '';
    foreach ($params as $key => $value) {
        $signStr .= $key . '=' . $value . '&';
    }
    $signStr .= 'key=' . $merchantKey;
    
    // MD5 加密并转大写
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

### Java
```java
import java.security.MessageDigest;
import java.util.*;

public class NotifyHandler {
    
    public static boolean verifySignature(Map<String, String> params, String merchantKey) {
        String signature = params.get("signature");
        params.remove("signature");
        
        // 过滤空值并排序
        TreeMap<String, String> sortedParams = new TreeMap<>();
        for (Map.Entry<String, String> entry : params.entrySet()) {
            if (entry.getValue() != null && !entry.getValue().isEmpty()) {
                sortedParams.put(entry.getKey(), entry.getValue());
            }
        }
        
        // 拼接字符串
        StringBuilder signStr = new StringBuilder();
        for (Map.Entry<String, String> entry : sortedParams.entrySet()) {
            signStr.append(entry.getKey()).append("=").append(entry.getValue()).append("&");
        }
        signStr.append("key=").append(merchantKey);
        
        // MD5 加密
        String expectedSign = md5(signStr.toString()).toUpperCase();
        
        return signature.equals(expectedSign);
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

### Node.js
```javascript
const crypto = require('crypto');

function verifySignature(params, merchantKey) {
    const signature = params.signature;
    delete params.signature;
    
    // 过滤空值并排序
    const sortedKeys = Object.keys(params)
        .filter(key => params[key] !== '' && params[key] !== null && params[key] !== undefined)
        .sort();
    
    // 拼接字符串
    let signStr = sortedKeys.map(key => `${key}=${params[key]}`).join('&');
    signStr += `&key=${merchantKey}`;
    
    // MD5 加密并转大写
    const expectedSign = crypto.createHash('md5').update(signStr).digest('hex').toUpperCase();
    
    return signature === expectedSign;
}

// Express 示例
app.post('/notify', (req, res) => {
    const params = req.body;
    const merchantKey = 'your-merchant-key';
    
    if (verifySignature({...params}, merchantKey)) {
        // 签名验证通过，处理业务逻辑
        // ...
        res.send('OK');
    } else {
        res.send('签名验证失败');
    }
});
```

### Python
```python
import hashlib

def verify_signature(params: dict, merchant_key: str) -> bool:
    signature = params.pop('signature', '')
    
    # 过滤空值并排序
    filtered_params = {k: v for k, v in params.items() if v not in ('', None)}
    sorted_keys = sorted(filtered_params.keys())
    
    # 拼接字符串
    sign_str = '&'.join([f'{k}={filtered_params[k]}' for k in sorted_keys])
    sign_str += f'&key={merchant_key}'
    
    # MD5 加密并转大写
    expected_sign = hashlib.md5(sign_str.encode('utf-8')).hexdigest().upper()
    
    return signature == expected_sign

# Flask 示例
@app.route('/notify', methods=['POST'])
def notify():
    params = request.form.to_dict()
    merchant_key = 'your-merchant-key'
    
    if verify_signature(params.copy(), merchant_key):
        # 签名验证通过，处理业务逻辑
        # ...
        return 'OK'
    else:
        return '签名验证失败'
```

---

## 注意事项

1. **幂等性处理**：同一订单可能会收到多次通知，商户需要根据 `orderNo` 或 `merchantOrderNo` 做幂等处理，避免重复业务处理。

2. **超时设置**：系统发送通知的超时时间为 10 秒，请确保您的接口能在 10 秒内响应。

3. **HTTPS 支持**：建议使用 HTTPS 协议的回调地址，确保数据传输安全。

4. **IP 白名单**：如需配置 IP 白名单，请联系客服获取通知服务器 IP 列表。

5. **日志记录**：建议记录每次收到的通知内容，便于问题排查。

---

## 常见问题

### Q: 为什么收不到通知？
A: 请检查以下几点：
- `notify_url` 是否正确填写
- 服务器是否能正常访问
- 是否有防火墙拦截
- 接口是否返回了 `OK`

### Q: 如何手动获取通知？
A: 可以联系客服在后台进行手动补发通知操作。

### Q: 签名验证失败怎么办？
A: 请检查：
- 商户密钥是否正确
- 参数是否按照 ASCII 码排序
- 是否过滤了空值参数
- MD5 结果是否转为大写
