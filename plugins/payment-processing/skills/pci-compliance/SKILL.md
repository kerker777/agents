---
name: pci-compliance
description: 實作 PCI DSS 合規要求，以安全處理支付卡資料及支付系統。適用於保護支付處理流程、達成 PCI 合規，或實作支付卡安全措施。
---

# PCI 合規

精通 PCI DSS（Payment Card Industry Data Security Standard，支付卡產業資料安全標準）合規，以確保安全的支付處理及持卡人資料處理。

## 何時使用此技能

- 建構支付處理系統
- 處理信用卡資訊
- 實作安全的支付流程
- 執行 PCI 合規稽核
- 縮減 PCI 合規範圍
- 實作代幣化（tokenization）和加密
- 準備 PCI DSS 評估

## PCI DSS 要求（12 項核心要求）

### 建立並維護安全網路
1. 安裝並維護防火牆配置
2. 不使用供應商預設密碼

### 保護持卡人資料
3. 保護儲存的持卡人資料
4. 加密跨公開網路傳輸的持卡人資料

### 維護漏洞管理
5. 保護系統免受惡意軟體攻擊
6. 開發並維護安全的系統和應用程式

### 實作強大的存取控制
7. 依業務需知原則限制對持卡人資料的存取
8. 識別並驗證對系統元件的存取
9. 限制對持卡人資料的實體存取

### 監控與測試網路
10. 追蹤並監控所有對網路資源和持卡人資料的存取
11. 定期測試安全系統和流程

### 維護資訊安全政策
12. 維護涵蓋資訊安全的政策

## 合規等級

**Level 1**：> 600 萬筆交易/年（需年度 ROC）
**Level 2**：100 萬至 600 萬筆交易/年（年度 SAQ）
**Level 3**：2 萬至 100 萬筆電子商務交易/年
**Level 4**：< 2 萬筆電子商務或 < 100 萬筆總交易

## 資料最小化（絕不儲存）

```python
# 絕不儲存這些資料
PROHIBITED_DATA = {
    'full_track_data': 'Magnetic stripe data',
    'cvv': 'Card verification code/value',
    'pin': 'PIN or PIN block'
}

# 可儲存（若已加密）
ALLOWED_DATA = {
    'pan': 'Primary Account Number (card number)',
    'cardholder_name': 'Name on card',
    'expiration_date': 'Card expiration',
    'service_code': 'Service code'
}

class PaymentData:
    """安全的支付資料處理。"""

    def __init__(self):
        self.prohibited_fields = ['cvv', 'cvv2', 'cvc', 'pin']

    def sanitize_log(self, data):
        """從日誌中移除敏感資料。"""
        sanitized = data.copy()

        # 遮罩 PAN
        if 'card_number' in sanitized:
            card = sanitized['card_number']
            sanitized['card_number'] = f"{card[:6]}{'*' * (len(card) - 10)}{card[-4:]}"

        # 移除禁止的資料
        for field in self.prohibited_fields:
            sanitized.pop(field, None)

        return sanitized

    def validate_no_prohibited_storage(self, data):
        """確保沒有儲存禁止的資料。"""
        for field in self.prohibited_fields:
            if field in data:
                raise SecurityError(f"Attempting to store prohibited field: {field}")
```

## 代幣化（Tokenization）

### 使用支付處理商代幣
```python
import stripe

class TokenizedPayment:
    """使用代幣處理支付（伺服器端無卡片資料）。"""

    @staticmethod
    def create_payment_method_token(card_details):
        """從卡片詳細資料建立代幣（僅限客戶端）。"""
        # 這應該僅在客戶端使用 STRIPE.JS 執行
        # 絕不將卡片詳細資料傳送到您的伺服器

        """
        // Frontend JavaScript
        const stripe = Stripe('pk_...');

        const {token, error} = await stripe.createToken({
            card: {
                number: '4242424242424242',
                exp_month: 12,
                exp_year: 2024,
                cvc: '123'
            }
        });

        // 傳送 token.id 到伺服器（不是卡片詳細資料）
        """
        pass

    @staticmethod
    def charge_with_token(token_id, amount):
        """使用代幣進行扣款（伺服器端）。"""
        # 您的伺服器只看到代幣，絕不會看到卡號
        stripe.api_key = "sk_..."

        charge = stripe.Charge.create(
            amount=amount,
            currency="usd",
            source=token_id,  # 使用代幣而非卡片詳細資料
            description="Payment"
        )

        return charge

    @staticmethod
    def store_payment_method(customer_id, payment_method_token):
        """將支付方式儲存為代幣供未來使用。"""
        stripe.Customer.modify(
            customer_id,
            source=payment_method_token
        )

        # 僅在資料庫中儲存 customer_id 和 payment_method_id
        # 絕不儲存實際的卡片詳細資料
        return {
            'customer_id': customer_id,
            'has_payment_method': True
            # 不要儲存：卡號、CVV 等
        }
```

### 自訂代幣化（進階）
```python
import secrets
from cryptography.fernet import Fernet

class TokenVault:
    """卡片資料的安全代幣保管庫（如果您必須儲存）。"""

    def __init__(self, encryption_key):
        self.cipher = Fernet(encryption_key)
        self.vault = {}  # 正式環境：使用加密資料庫

    def tokenize(self, card_data):
        """將卡片資料轉換為代幣。"""
        # 產生安全的隨機代幣
        token = secrets.token_urlsafe(32)

        # 加密卡片資料
        encrypted = self.cipher.encrypt(json.dumps(card_data).encode())

        # 儲存代幣 -> 加密資料對應
        self.vault[token] = encrypted

        return token

    def detokenize(self, token):
        """從代幣取回卡片資料。"""
        encrypted = self.vault.get(token)
        if not encrypted:
            raise ValueError("Token not found")

        # 解密
        decrypted = self.cipher.decrypt(encrypted)
        return json.loads(decrypted.decode())

    def delete_token(self, token):
        """從保管庫移除代幣。"""
        self.vault.pop(token, None)
```

## 加密

### 靜態資料（Data at Rest）
```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

class EncryptedStorage:
    """使用 AES-256-GCM 加密靜態資料。"""

    def __init__(self, encryption_key):
        """使用 256-bit 金鑰初始化。"""
        self.key = encryption_key  # 必須是 32 bytes

    def encrypt(self, plaintext):
        """加密資料。"""
        # 產生隨機 nonce
        nonce = os.urandom(12)

        # 加密
        aesgcm = AESGCM(self.key)
        ciphertext = aesgcm.encrypt(nonce, plaintext.encode(), None)

        # 回傳 nonce + ciphertext
        return nonce + ciphertext

    def decrypt(self, encrypted_data):
        """解密資料。"""
        # 提取 nonce 和 ciphertext
        nonce = encrypted_data[:12]
        ciphertext = encrypted_data[12:]

        # 解密
        aesgcm = AESGCM(self.key)
        plaintext = aesgcm.decrypt(nonce, ciphertext, None)

        return plaintext.decode()

# 使用範例
storage = EncryptedStorage(os.urandom(32))
encrypted_pan = storage.encrypt("4242424242424242")
# 將 encrypted_pan 儲存在資料庫中
```

### 傳輸中資料（Data in Transit）
```python
# 始終使用 TLS 1.2 或更高版本
# Flask/Django 範例
app.config['SESSION_COOKIE_SECURE'] = True  # 僅限 HTTPS
app.config['SESSION_COOKIE_HTTPONLY'] = True
app.config['SESSION_COOKIE_SAMESITE'] = 'Strict'

# 強制使用 HTTPS
from flask_talisman import Talisman
Talisman(app, force_https=True)
```

## 存取控制

```python
from functools import wraps
from flask import session

def require_pci_access(f):
    """裝飾器，用於限制對持卡人資料的存取。"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        user = session.get('user')

        # 檢查使用者是否具有 PCI 存取角色
        if not user or 'pci_access' not in user.get('roles', []):
            return {'error': 'Unauthorized access to cardholder data'}, 403

        # 記錄存取嘗試
        audit_log(
            user=user['id'],
            action='access_cardholder_data',
            resource=f.__name__
        )

        return f(*args, **kwargs)

    return decorated_function

@app.route('/api/payment-methods')
@require_pci_access
def get_payment_methods():
    """取得支付方式（限制存取）。"""
    # 僅供具有 pci_access 角色的使用者存取
    pass
```

## 稽核日誌

```python
import logging
from datetime import datetime

class PCIAuditLogger:
    """符合 PCI 規範的稽核日誌。"""

    def __init__(self):
        self.logger = logging.getLogger('pci_audit')
        # 配置為寫入安全的只附加日誌

    def log_access(self, user_id, resource, action, result):
        """記錄對持卡人資料的存取。"""
        entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'user_id': user_id,
            'resource': resource,
            'action': action,
            'result': result,
            'ip_address': request.remote_addr
        }

        self.logger.info(json.dumps(entry))

    def log_authentication(self, user_id, success, method):
        """記錄驗證嘗試。"""
        entry = {
            'timestamp': datetime.utcnow().isoformat(),
            'user_id': user_id,
            'event': 'authentication',
            'success': success,
            'method': method,
            'ip_address': request.remote_addr
        }

        self.logger.info(json.dumps(entry))

# 使用範例
audit = PCIAuditLogger()
audit.log_access(user_id=123, resource='payment_methods', action='read', result='success')
```

## 安全最佳實踐

### 輸入驗證
```python
import re

def validate_card_number(card_number):
    """驗證卡號格式（Luhn 演算法）。"""
    # 移除空格和破折號
    card_number = re.sub(r'[\s-]', '', card_number)

    # 檢查是否全為數字
    if not card_number.isdigit():
        return False

    # Luhn 演算法
    def luhn_checksum(card_num):
        def digits_of(n):
            return [int(d) for d in str(n)]

        digits = digits_of(card_num)
        odd_digits = digits[-1::-2]
        even_digits = digits[-2::-2]
        checksum = sum(odd_digits)
        for d in even_digits:
            checksum += sum(digits_of(d * 2))
        return checksum % 10

    return luhn_checksum(card_number) == 0

def sanitize_input(user_input):
    """清理使用者輸入以防止注入攻擊。"""
    # 移除特殊字元
    # 依預期格式驗證
    # 為資料庫查詢進行跳脫
    pass
```

## PCI DSS SAQ（自我評估問卷）

### SAQ A（最少要求）
- 使用託管支付頁面的電子商務
- 您的系統上沒有卡片資料
- ~20 個問題

### SAQ A-EP
- 使用嵌入式支付表單的電子商務
- 使用 JavaScript 處理卡片資料
- ~180 個問題

### SAQ D（最多要求）
- 儲存、處理或傳輸卡片資料
- 完整的 PCI DSS 要求
- ~300 個問題

## 合規檢查清單

```python
PCI_COMPLIANCE_CHECKLIST = {
    'network_security': [
        'Firewall configured and maintained',
        'No vendor default passwords',
        'Network segmentation implemented'
    ],
    'data_protection': [
        'No storage of CVV, track data, or PIN',
        'PAN encrypted when stored',
        'PAN masked when displayed',
        'Encryption keys properly managed'
    ],
    'vulnerability_management': [
        'Anti-virus installed and updated',
        'Secure development practices',
        'Regular security patches',
        'Vulnerability scanning performed'
    ],
    'access_control': [
        'Access restricted by role',
        'Unique IDs for all users',
        'Multi-factor authentication',
        'Physical security measures'
    ],
    'monitoring': [
        'Audit logs enabled',
        'Log review process',
        'File integrity monitoring',
        'Regular security testing'
    ],
    'policy': [
        'Security policy documented',
        'Risk assessment performed',
        'Security awareness training',
        'Incident response plan'
    ]
}
```

## 資源

- **references/data-minimization.md**：絕不儲存禁止的資料
- **references/tokenization.md**：代幣化策略
- **references/encryption.md**：加密要求
- **references/access-control.md**：基於角色的存取控制
- **references/audit-logging.md**：全面的日誌記錄
- **assets/pci-compliance-checklist.md**：完整檢查清單
- **assets/encrypted-storage.py**：加密工具
- **scripts/audit-payment-system.sh**：合規稽核腳本

## 常見違規

1. **儲存 CVV**：絕不儲存卡片驗證碼
2. **未加密的 PAN**：卡號在靜態時必須加密
3. **弱加密**：使用 AES-256 或同等級加密
4. **無存取控制**：限制誰可以存取持卡人資料
5. **缺少稽核日誌**：必須記錄所有對支付資料的存取
6. **不安全的傳輸**：始終使用 TLS 1.2+
7. **預設密碼**：變更所有預設憑證
8. **無安全測試**：需要定期滲透測試

## 縮減 PCI 範圍

1. **使用託管支付**：Stripe Checkout、PayPal 等
2. **代幣化**：以代幣取代卡片資料
3. **網路區隔**：隔離持卡人資料環境
4. **外包**：使用符合 PCI 規範的支付處理商
5. **不儲存**：絕不儲存完整的卡片詳細資料

透過最小化接觸卡片資料的系統，可以大幅降低合規負擔。
