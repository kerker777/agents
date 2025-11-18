# 法規遵循檢查

您是一位合規專家，專精於軟體系統的法規要求，包括 GDPR、HIPAA、SOC2、PCI-DSS 及其他產業標準。請執行全面的合規稽核，並提供達成與維持合規的實作指引。

## 情境說明
使用者需要確保其應用程式符合法規要求和產業標準。請著重於合規控制的實際實作、自動化監控以及稽核軌跡的產生。

## 需求
$ARGUMENTS

## 操作指示

### 1. 合規框架分析

識別適用的法規與標準：

**法規對應關係**
```python
class ComplianceAnalyzer:
    def __init__(self):
        self.regulations = {
            'GDPR': {
                'scope': 'EU data protection',  # 歐盟資料保護
                'applies_if': [
                    'Processing EU residents data',  # 處理歐盟居民資料
                    'Offering goods/services to EU',  # 向歐盟提供商品/服務
                    'Monitoring EU residents behavior'  # 監控歐盟居民行為
                ],
                'key_requirements': [
                    'Privacy by design',  # 隱私設計
                    'Data minimization',  # 資料最小化
                    'Right to erasure',  # 刪除權
                    'Data portability',  # 資料可攜性
                    'Consent management',  # 同意管理
                    'DPO appointment',  # 資料保護長任命
                    'Privacy notices',  # 隱私聲明
                    'Data breach notification (72hrs)'  # 資料外洩通知（72小時內）
                ]
            },
            'HIPAA': {
                'scope': 'Healthcare data protection (US)',  # 醫療資料保護（美國）
                'applies_if': [
                    'Healthcare providers',  # 醫療保健提供者
                    'Health plan providers',  # 健康保險計畫提供者
                    'Healthcare clearinghouses',  # 醫療資訊交換中心
                    'Business associates'  # 業務夥伴
                ],
                'key_requirements': [
                    'PHI encryption',  # PHI 加密
                    'Access controls',  # 存取控制
                    'Audit logs',  # 稽核日誌
                    'Business Associate Agreements',  # 業務夥伴協議
                    'Risk assessments',  # 風險評估
                    'Employee training',  # 員工訓練
                    'Incident response',  # 事件回應
                    'Physical safeguards'  # 實體防護措施
                ]
            },
            'SOC2': {
                'scope': 'Service organization controls',  # 服務組織控制
                'applies_if': [
                    'SaaS providers',  # SaaS 提供者
                    'Data processors',  # 資料處理者
                    'Cloud services'  # 雲端服務
                ],
                'trust_principles': [
                    'Security',  # 安全性
                    'Availability',  # 可用性
                    'Processing integrity',  # 處理完整性
                    'Confidentiality',  # 機密性
                    'Privacy'  # 隱私性
                ]
            },
            'PCI-DSS': {
                'scope': 'Payment card data security',  # 支付卡資料安全
                'applies_if': [
                    'Accept credit/debit cards',  # 接受信用卡/簽帳卡
                    'Process card payments',  # 處理卡片支付
                    'Store card data',  # 儲存卡片資料
                    'Transmit card data'  # 傳輸卡片資料
                ],
                'compliance_levels': {
                    'Level 1': '>6M transactions/year',  # 每年超過 600 萬筆交易
                    'Level 2': '1M-6M transactions/year',  # 每年 100-600 萬筆交易
                    'Level 3': '20K-1M transactions/year',  # 每年 2 萬到 100 萬筆交易
                    'Level 4': '<20K transactions/year'  # 每年少於 2 萬筆交易
                }
            }
        }
    
    def determine_applicable_regulations(self, business_info):
        """
        根據業務情境判定適用的法規
        """
        applicable = []

        # 檢查每項法規
        for reg_name, reg_info in self.regulations.items():
            if self._check_applicability(business_info, reg_info):
                applicable.append({
                    'regulation': reg_name,
                    'reason': self._get_applicability_reason(business_info, reg_info),
                    'priority': self._calculate_priority(business_info, reg_name)
                })

        return sorted(applicable, key=lambda x: x['priority'], reverse=True)
```

### 2. 資料隱私合規

實作隱私控制：

**GDPR 實作**
```python
class GDPRCompliance:
    def implement_privacy_controls(self):
        """
        實作 GDPR 要求的隱私控制
        """
        controls = {}

        # 1. 同意管理
        controls['consent_management'] = '''
class ConsentManager:
    def __init__(self):
        self.consent_types = [
            'marketing_emails',  # 行銷電子郵件
            'analytics_tracking',  # 分析追蹤
            'third_party_sharing',  # 第三方分享
            'profiling'  # 用戶畫像
        ]

    def record_consent(self, user_id, consent_type, granted):
        """
        記錄使用者同意，包含完整稽核軌跡
        """
        consent_record = {
            'user_id': user_id,
            'consent_type': consent_type,
            'granted': granted,
            'timestamp': datetime.utcnow(),
            'ip_address': request.remote_addr,
            'user_agent': request.headers.get('User-Agent'),
            'version': self.get_current_privacy_policy_version(),
            'method': 'explicit_checkbox'  # 非預選
        }

        # 儲存於僅附加的稽核日誌中
        self.consent_audit_log.append(consent_record)

        # 更新目前同意狀態
        self.update_user_consents(user_id, consent_type, granted)

        return consent_record

    def verify_consent(self, user_id, consent_type):
        """
        驗證使用者是否已對特定處理給予同意
        """
        consent = self.get_user_consent(user_id, consent_type)
        return consent and consent['granted'] and not consent.get('withdrawn')
'''

        # 2. 刪除權（被遺忘權）
        controls['right_to_erasure'] = '''
class DataErasureService:
    def process_erasure_request(self, user_id, verification_token):
        """
        處理 GDPR 第 17 條刪除請求
        """
        # 驗證請求真實性
        if not self.verify_erasure_token(user_id, verification_token):
            raise ValueError("Invalid erasure request")

        erasure_log = {
            'user_id': user_id,
            'requested_at': datetime.utcnow(),
            'data_categories': []
        }

        # 1. 個人資料
        self.erase_user_profile(user_id)
        erasure_log['data_categories'].append('profile')

        # 2. 使用者生成內容（匿名化而非刪除）
        self.anonymize_user_content(user_id)
        erasure_log['data_categories'].append('content_anonymized')

        # 3. 分析資料
        self.remove_from_analytics(user_id)
        erasure_log['data_categories'].append('analytics')

        # 4. 備份資料（排程刪除）
        self.schedule_backup_deletion(user_id)
        erasure_log['data_categories'].append('backups_scheduled')

        # 5. 通知第三方
        self.notify_processors_of_erasure(user_id)

        # 保留最少記錄以符合法律規範
        self.store_erasure_record(erasure_log)

        return {
            'status': 'completed',
            'erasure_id': erasure_log['id'],
            'categories_erased': erasure_log['data_categories']
        }
'''

        # 3. 資料可攜性
        controls['data_portability'] = '''
class DataPortabilityService:
    def export_user_data(self, user_id, format='json'):
        """
        GDPR 第 20 條 - 資料可攜性
        """
        user_data = {
            'export_date': datetime.utcnow().isoformat(),
            'user_id': user_id,
            'format_version': '2.0',
            'data': {}
        }

        # 收集所有使用者資料
        user_data['data']['profile'] = self.get_user_profile(user_id)
        user_data['data']['preferences'] = self.get_user_preferences(user_id)
        user_data['data']['content'] = self.get_user_content(user_id)
        user_data['data']['activity'] = self.get_user_activity(user_id)
        user_data['data']['consents'] = self.get_consent_history(user_id)

        # 根據請求格式化
        if format == 'json':
            return json.dumps(user_data, indent=2)
        elif format == 'csv':
            return self.convert_to_csv(user_data)
        elif format == 'xml':
            return self.convert_to_xml(user_data)
'''

        return controls

**隱私設計 (Privacy by Design)**
```python
# 實作隱私設計原則
class PrivacyByDesign:
    def implement_data_minimization(self):
        """
        僅收集必要資料
        """
        # 之前（收集過多）
        bad_user_model = {
            'email': str,
            'password': str,
            'full_name': str,
            'date_of_birth': date,
            'ssn': str,  # 非必要
            'address': str,  # 對基本服務非必要
            'phone': str,  # 非必要
            'gender': str,  # 非必要
            'income': int  # 非必要
        }

        # 之後（資料最小化）
        good_user_model = {
            'email': str,  # 驗證必需
            'password_hash': str,  # 絕不儲存明文
            'display_name': str,  # 選填，使用者提供
            'created_at': datetime,
            'last_login': datetime
        }

        return good_user_model

    def implement_pseudonymization(self):
        """
        以假名取代識別欄位
        """
        def pseudonymize_record(record):
            # 產生一致的假名
            user_pseudonym = hashlib.sha256(
                f"{record['user_id']}{SECRET_SALT}".encode()
            ).hexdigest()[:16]

            return {
                'pseudonym': user_pseudonym,
                'data': {
                    # 移除直接識別資訊
                    'age_group': self._get_age_group(record['age']),
                    'region': self._get_region(record['ip_address']),
                    'activity': record['activity_data']
                }
            }
```

### 3. 安全合規

實作各種標準的安全控制：

**SOC2 安全控制**
```python
class SOC2SecurityControls:
    def implement_access_controls(self):
        """
        SOC2 CC6.1 - 邏輯與實體存取控制
        """
        controls = {
            'authentication': '''
# 多因素驗證
class MFAEnforcement:
    def enforce_mfa(self, user, resource_sensitivity):
        if resource_sensitivity == 'high':
            return self.require_mfa(user)
        elif resource_sensitivity == 'medium' and user.is_admin:
            return self.require_mfa(user)
        return self.standard_auth(user)

    def require_mfa(self, user):
        factors = []

        # 因素 1：密碼（你知道的）
        factors.append(self.verify_password(user))

        # 因素 2：TOTP/SMS（你擁有的）
        if user.mfa_method == 'totp':
            factors.append(self.verify_totp(user))
        elif user.mfa_method == 'sms':
            factors.append(self.verify_sms_code(user))

        # 因素 3：生物辨識（你是誰）- 選填
        if user.biometric_enabled:
            factors.append(self.verify_biometric(user))

        return all(factors)
''',
            'authorization': '''
# 角色型存取控制
class RBACAuthorization:
    def __init__(self):
        self.roles = {
            'admin': ['read', 'write', 'delete', 'admin'],
            'user': ['read', 'write:own'],
            'viewer': ['read']
        }

    def check_permission(self, user, resource, action):
        user_permissions = self.get_user_permissions(user)

        # 檢查明確權限
        if action in user_permissions:
            return True

        # 檢查基於所有權的權限
        if f"{action}:own" in user_permissions:
            return self.user_owns_resource(user, resource)

        # 記錄被拒絕的存取嘗試
        self.log_access_denied(user, resource, action)
        return False
''',
            'encryption': '''
# 靜態與傳輸中加密
class EncryptionControls:
    def __init__(self):
        self.kms = KeyManagementService()

    def encrypt_at_rest(self, data, classification):
        if classification == 'sensitive':
            # 使用信封加密
            dek = self.kms.generate_data_encryption_key()
            encrypted_data = self.encrypt_with_key(data, dek)
            encrypted_dek = self.kms.encrypt_key(dek)

            return {
                'data': encrypted_data,
                'encrypted_key': encrypted_dek,
                'algorithm': 'AES-256-GCM',
                'key_id': self.kms.get_current_key_id()
            }

    def configure_tls(self):
        return {
            'min_version': 'TLS1.2',
            'ciphers': [
                'ECDHE-RSA-AES256-GCM-SHA384',
                'ECDHE-RSA-AES128-GCM-SHA256'
            ],
            'hsts': 'max-age=31536000; includeSubDomains',
            'certificate_pinning': True
        }
'''
        }

        return controls
```

### 4. 稽核日誌與監控

實作全面的稽核軌跡：

**稽核日誌系統**
```python
class ComplianceAuditLogger:
    def __init__(self):
        self.required_events = {
            'authentication': [
                'login_success',  # 登入成功
                'login_failure',  # 登入失敗
                'logout',  # 登出
                'password_change',  # 密碼變更
                'mfa_enabled',  # MFA 啟用
                'mfa_disabled'  # MFA 停用
            ],
            'authorization': [
                'access_granted',  # 存取授予
                'access_denied',  # 存取拒絕
                'permission_changed',  # 權限變更
                'role_assigned',  # 角色指派
                'role_revoked'  # 角色撤銷
            ],
            'data_access': [
                'data_viewed',  # 資料檢視
                'data_exported',  # 資料匯出
                'data_modified',  # 資料修改
                'data_deleted',  # 資料刪除
                'bulk_operation'  # 批次作業
            ],
            'compliance': [
                'consent_given',  # 同意給予
                'consent_withdrawn',  # 同意撤回
                'data_request',  # 資料請求
                'data_erasure',  # 資料刪除
                'privacy_settings_changed'  # 隱私設定變更
            ]
        }

    def log_event(self, event_type, details):
        """
        建立防竄改的稽核日誌項目
        """
        log_entry = {
            'id': str(uuid.uuid4()),
            'timestamp': datetime.utcnow().isoformat(),
            'event_type': event_type,
            'user_id': details.get('user_id'),
            'ip_address': self._get_ip_address(),
            'user_agent': request.headers.get('User-Agent'),
            'session_id': session.get('id'),
            'details': details,
            'compliance_flags': self._get_compliance_flags(event_type)
        }

        # 新增完整性檢查
        log_entry['checksum'] = self._calculate_checksum(log_entry)

        # 儲存於不可變日誌中
        self._store_audit_log(log_entry)

        # 針對關鍵事件進行即時告警
        if self._is_critical_event(event_type):
            self._send_security_alert(log_entry)

        return log_entry

    def _calculate_checksum(self, entry):
        """
        建立防竄改檢查碼
        """
        # 包含前一個項目的雜湊值以達到類區塊鏈的完整性
        previous_hash = self._get_previous_entry_hash()

        content = json.dumps(entry, sort_keys=True)
        return hashlib.sha256(
            f"{previous_hash}{content}{SECRET_KEY}".encode()
        ).hexdigest()
```

**合規報告**
```python
def generate_compliance_report(self, regulation, period):
    """
    為稽核人員產生合規報告
    """
    report = {
        'regulation': regulation,
        'period': period,
        'generated_at': datetime.utcnow(),
        'sections': {}
    }

    if regulation == 'GDPR':
        report['sections'] = {
            'data_processing_activities': self._get_processing_activities(period),
            'consent_metrics': self._get_consent_metrics(period),
            'data_requests': {
                'access_requests': self._count_access_requests(period),
                'erasure_requests': self._count_erasure_requests(period),
                'portability_requests': self._count_portability_requests(period),
                'response_times': self._calculate_response_times(period)
            },
            'data_breaches': self._get_breach_reports(period),
            'third_party_processors': self._list_processors(),
            'privacy_impact_assessments': self._get_dpias(period)
        }

    elif regulation == 'HIPAA':
        report['sections'] = {
            'access_controls': self._audit_access_controls(period),
            'phi_access_log': self._get_phi_access_log(period),
            'risk_assessments': self._get_risk_assessments(period),
            'training_records': self._get_training_compliance(period),
            'business_associates': self._list_bas_with_agreements(),
            'incident_response': self._get_incident_reports(period)
        }

    return report
```

### 5. 醫療保健合規 (HIPAA)

實作 HIPAA 特定控制：

**PHI 保護**
```python
class HIPAACompliance:
    def protect_phi(self):
        """
        實作 HIPAA 保護健康資訊 (PHI) 的防護措施
        """
        # 技術防護措施
        technical_controls = {
            'access_control': '''
class PHIAccessControl:
    def __init__(self):
        self.minimum_necessary_rule = True

    def grant_phi_access(self, user, patient_id, purpose):
        """
        實作最小必要標準
        """
        # 驗證合法目的
        if not self._verify_treatment_relationship(user, patient_id, purpose):
            self._log_denied_access(user, patient_id, purpose)
            raise PermissionError("No treatment relationship")

        # 根據角色與目的授予有限存取權
        access_scope = self._determine_access_scope(user.role, purpose)

        # 時限性存取
        access_token = {
            'user_id': user.id,
            'patient_id': patient_id,
            'scope': access_scope,
            'purpose': purpose,
            'expires_at': datetime.utcnow() + timedelta(hours=24),
            'audit_id': str(uuid.uuid4())
        }

        # 記錄所有存取
        self._log_phi_access(access_token)

        return access_token
''',
            'encryption': '''
class PHIEncryption:
    def encrypt_phi_at_rest(self, phi_data):
        """
        符合 HIPAA 的 PHI 加密
        """
        # 使用 FIPS 140-2 驗證的加密
        encryption_config = {
            'algorithm': 'AES-256-CBC',
            'key_derivation': 'PBKDF2',
            'iterations': 100000,
            'validation': 'FIPS-140-2-Level-2'
        }

        # 加密 PHI 欄位
        encrypted_phi = {}
        for field, value in phi_data.items():
            if self._is_phi_field(field):
                encrypted_phi[field] = self._encrypt_field(value, encryption_config)
            else:
                encrypted_phi[field] = value

        return encrypted_phi

    def secure_phi_transmission(self):
        """
        確保 PHI 傳輸安全
        """
        return {
            'protocols': ['TLS 1.2+'],
            'vpn_required': True,
            'email_encryption': 'S/MIME or PGP required',
            'fax_alternative': 'Secure messaging portal'
        }
'''
        }

        # 行政防護措施
        admin_controls = {
            'workforce_training': '''
class HIPAATraining:
    def track_training_compliance(self, employee):
        """
        確保員工 HIPAA 訓練合規
        """
        required_modules = [
            'HIPAA Privacy Rule',  # HIPAA 隱私規則
            'HIPAA Security Rule',  # HIPAA 安全規則
            'PHI Handling Procedures',  # PHI 處理程序
            'Breach Notification',  # 外洩通知
            'Patient Rights',  # 病患權利
            'Minimum Necessary Standard'  # 最小必要標準
        ]

        training_status = {
            'employee_id': employee.id,
            'completed_modules': [],
            'pending_modules': [],
            'last_training_date': None,
            'next_due_date': None
        }

        for module in required_modules:
            completion = self._check_module_completion(employee.id, module)
            if completion and completion['date'] > datetime.now() - timedelta(days=365):
                training_status['completed_modules'].append(module)
            else:
                training_status['pending_modules'].append(module)

        return training_status
'''
        }

        return {
            'technical': technical_controls,
            'administrative': admin_controls
        }
```

### 6. 支付卡合規 (PCI-DSS)

實作 PCI-DSS 要求：

**PCI-DSS 控制**
```python
class PCIDSSCompliance:
    def implement_pci_controls(self):
        """
        實作 PCI-DSS v4.0 要求
        """
        controls = {
            'cardholder_data_protection': '''
class CardDataProtection:
    def __init__(self):
        # 絕不儲存這些資料
        self.prohibited_data = ['cvv', 'cvv2', 'cvc2', 'cid', 'pin', 'pin_block']

    def handle_card_data(self, card_info):
        """
        符合 PCI-DSS 的卡片資料處理
        """
        # 立即代碼化
        token = self.tokenize_card(card_info)

        # 若必須儲存，僅儲存允許的欄位
        stored_data = {
            'token': token,
            'last_four': card_info['number'][-4:],
            'exp_month': card_info['exp_month'],
            'exp_year': card_info['exp_year'],
            'cardholder_name': self._encrypt(card_info['name'])
        }

        # 絕不記錄完整卡號
        self._log_transaction(token, 'XXXX-XXXX-XXXX-' + stored_data['last_four'])

        return stored_data

    def tokenize_card(self, card_info):
        """
        以代碼取代 PAN（主帳號）
        """
        # 使用支付處理商的代碼化
        response = payment_processor.tokenize({
            'number': card_info['number'],
            'exp_month': card_info['exp_month'],
            'exp_year': card_info['exp_year']
        })

        return response['token']
''',
            'network_segmentation': '''
# PCI 合規的網路區段
class PCINetworkSegmentation:
    def configure_network_zones(self):
        """
        實作網路區段
        """
        zones = {
            'cde': {  # 持卡人資料環境 (Cardholder Data Environment)
                'description': 'Systems that process, store, or transmit CHD',  # 處理、儲存或傳輸 CHD 的系統
                'controls': [
                    'Firewall required',  # 需要防火牆
                    'IDS/IPS monitoring',  # IDS/IPS 監控
                    'No direct internet access',  # 無直接網際網路存取
                    'Quarterly vulnerability scans',  # 季度漏洞掃描
                    'Annual penetration testing'  # 年度滲透測試
                ]
            },
            'dmz': {
                'description': 'Public-facing systems',  # 對外系統
                'controls': [
                    'Web application firewall',  # 網站應用程式防火牆
                    'No CHD storage allowed',  # 不允許儲存 CHD
                    'Regular security scanning'  # 定期安全掃描
                ]
            },
            'internal': {
                'description': 'Internal corporate network',  # 內部企業網路
                'controls': [
                    'Segmented from CDE',  # 與 CDE 區隔
                    'Limited CDE access',  # 限制 CDE 存取
                    'Standard security controls'  # 標準安全控制
                ]
            }
        }

        return zones
''',
            'vulnerability_management': '''
class PCIVulnerabilityManagement:
    def quarterly_scan_requirements(self):
        """
        PCI-DSS 季度掃描要求
        """
        scan_config = {
            'internal_scans': {
                'frequency': 'quarterly',  # 季度
                'scope': 'all CDE systems',  # 所有 CDE 系統
                'tool': 'PCI-approved scanning vendor',  # PCI 核准的掃描廠商
                'passing_criteria': 'No high-risk vulnerabilities'  # 無高風險漏洞
            },
            'external_scans': {
                'frequency': 'quarterly',  # 季度
                'performed_by': 'ASV (Approved Scanning Vendor)',  # 核准掃描廠商
                'scope': 'All external-facing IP addresses',  # 所有對外 IP 位址
                'passing_criteria': 'Clean scan with no failures'  # 乾淨掃描無失敗
            },
            'remediation_timeline': {
                'critical': '24 hours',  # 嚴重：24 小時
                'high': '7 days',  # 高：7 天
                'medium': '30 days',  # 中：30 天
                'low': '90 days'  # 低：90 天
            }
        }

        return scan_config
'''
        }

        return controls
```

### 7. 持續合規監控

設定自動化合規監控：

**合規儀表板**
```python
class ComplianceDashboard:
    def generate_realtime_dashboard(self):
        """
        即時合規狀態儀表板
        """
        dashboard = {
            'timestamp': datetime.utcnow(),
            'overall_compliance_score': 0,
            'regulations': {}
        }

        # GDPR 合規指標
        dashboard['regulations']['GDPR'] = {
            'score': self.calculate_gdpr_score(),
            'status': 'COMPLIANT',
            'metrics': {
                'consent_rate': '87%',
                'data_requests_sla': '98% within 30 days',
                'privacy_policy_version': '2.1',
                'last_dpia': '2025-06-15',
                'encryption_coverage': '100%',
                'third_party_agreements': '12/12 signed'
            },
            'issues': [
                {
                    'severity': 'medium',
                    'issue': 'Cookie consent banner update needed',
                    'due_date': '2025-08-01'
                }
            ]
        }

        # HIPAA 合規指標
        dashboard['regulations']['HIPAA'] = {
            'score': self.calculate_hipaa_score(),
            'status': 'NEEDS_ATTENTION',
            'metrics': {
                'risk_assessment_current': True,
                'workforce_training_compliance': '94%',
                'baa_agreements': '8/8 current',
                'encryption_status': 'All PHI encrypted',
                'access_reviews': 'Completed 2025-06-30',
                'incident_response_tested': '2025-05-15'
            },
            'issues': [
                {
                    'severity': 'high',
                    'issue': '3 employees overdue for training',
                    'due_date': '2025-07-25'
                }
            ]
        }

        return dashboard
```

**自動化合規檢查**
```yaml
# .github/workflows/compliance-check.yml
name: Compliance Checks

on:
  push:
    branches: [main, develop]
  pull_request:
  schedule:
    - cron: '0 0 * * *'  # 每日合規檢查

jobs:
  compliance-scan:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: GDPR Compliance Check
      run: |
        python scripts/compliance/gdpr_checker.py

    - name: Security Headers Check
      run: |
        python scripts/compliance/security_headers.py

    - name: Dependency License Check
      run: |
        license-checker --onlyAllow 'MIT;Apache-2.0;BSD-3-Clause;ISC'

    - name: PII Detection Scan
      run: |
        # 掃描硬編碼的 PII
        python scripts/compliance/pii_scanner.py

    - name: Encryption Verification
      run: |
        # 驗證所有敏感資料已加密
        python scripts/compliance/encryption_checker.py

    - name: Generate Compliance Report
      if: always()
      run: |
        python scripts/compliance/generate_report.py > compliance-report.json

    - name: Upload Compliance Report
      uses: actions/upload-artifact@v3
      with:
        name: compliance-report
        path: compliance-report.json
```

### 8. 合規文件

產生必要文件：

**隱私政策產生器**
```python
def generate_privacy_policy(company_info, data_practices):
    """
    產生符合 GDPR 的隱私政策
    """
    policy = f"""
# 隱私政策

**最後更新**：{datetime.now().strftime('%B %d, %Y')}

## 1. 資料控制者
{company_info['name']}
{company_info['address']}
Email: {company_info['privacy_email']}
DPO: {company_info.get('dpo_contact', 'privacy@company.com')}

## 2. 我們收集的資料
{generate_data_collection_section(data_practices['data_types'])}

## 3. 處理的法律依據
{generate_legal_basis_section(data_practices['purposes'])}

## 4. 您的權利
根據 GDPR，您擁有以下權利：
- 存取您個人資料的權利
- 更正權
- 刪除權（「被遺忘權」）
- 限制處理的權利
- 資料可攜性權利
- 反對權
- 與自動化決策相關的權利

## 5. 資料保留
{generate_retention_policy(data_practices['retention_periods'])}

## 6. 國際傳輸
{generate_transfer_section(data_practices['international_transfers'])}

## 7. 聯絡我們
若要行使您的權利，請聯絡：{company_info['privacy_email']}
"""

    return policy
```

## 輸出格式

1. **合規評估**：所有適用法規的目前合規狀態
2. **差距分析**：需要注意的特定領域及嚴重性評級
3. **實作計畫**：達成合規的優先順序路徑圖
4. **技術控制**：所需控制的程式碼實作
5. **政策範本**：隱私政策、同意表單和聲明
6. **稽核程序**：持續合規監控的腳本
7. **文件**：稽核人員所需的記錄與證據
8. **訓練教材**：員工合規訓練資源

請著重於在合規要求、業務營運與使用者體驗之間取得平衡的實際實作。