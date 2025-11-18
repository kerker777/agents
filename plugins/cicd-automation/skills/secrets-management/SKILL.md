---
name: secrets-management
description: Implement secure secrets management for CI/CD pipelines using Vault, AWS Secrets Manager, or native platform solutions. Use when handling sensitive credentials, rotating secrets, or securing CI/CD environments.
---

# 機敏資料管理

使用 Vault、AWS Secrets Manager 及其他工具進行 CI/CD 流水線的安全機敏資料管理實務。

## 目的

在 CI/CD 流水線中實作安全的機敏資料管理，避免硬編碼敏感資訊。

## 使用時機

- 儲存 API 金鑰與憑證
- 管理資料庫密碼
- 處理 TLS 憑證
- 自動輪替機敏資料
- 實作最小權限存取

## 機敏資料管理工具

### HashiCorp Vault
- 集中式機敏資料管理
- 動態機敏資料產生
- 機敏資料輪替
- 稽核日誌記錄
- 細緻的存取控制

### AWS Secrets Manager
- AWS 原生解決方案
- 自動輪替
- 與 RDS 整合
- CloudFormation 支援

### Azure Key Vault
- Azure 原生解決方案
- HSM 支援的金鑰
- 憑證管理
- RBAC 整合

### Google Secret Manager
- GCP 原生解決方案
- 版本控制
- IAM 整合

## HashiCorp Vault 整合

### 設定 Vault

```bash
# Start Vault dev server
vault server -dev

# Set environment
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN='root'

# Enable secrets engine
vault secrets enable -path=secret kv-v2

# Store secret
vault kv put secret/database/config username=admin password=secret
```

### GitHub Actions 與 Vault

```yaml
name: Deploy with Vault Secrets

on: [push]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4

    - name: Import Secrets from Vault
      uses: hashicorp/vault-action@v2
      with:
        url: https://vault.example.com:8200
        token: ${{ secrets.VAULT_TOKEN }}
        secrets: |
          secret/data/database username | DB_USERNAME ;
          secret/data/database password | DB_PASSWORD ;
          secret/data/api key | API_KEY

    - name: Use secrets
      run: |
        echo "Connecting to database as $DB_USERNAME"
        # Use $DB_PASSWORD, $API_KEY
```

### GitLab CI 與 Vault

```yaml
deploy:
  image: vault:latest
  before_script:
    - export VAULT_ADDR=https://vault.example.com:8200
    - export VAULT_TOKEN=$VAULT_TOKEN
    - apk add curl jq
  script:
    - |
      DB_PASSWORD=$(vault kv get -field=password secret/database/config)
      API_KEY=$(vault kv get -field=key secret/api/credentials)
      echo "Deploying with secrets..."
      # Use $DB_PASSWORD, $API_KEY
```

**參考資料：**參閱 `references/vault-setup.md`

## AWS Secrets Manager

### 儲存機敏資料

```bash
aws secretsmanager create-secret \
  --name production/database/password \
  --secret-string "super-secret-password"
```

### 在 GitHub Actions 中擷取

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: us-west-2

- name: Get secret from AWS
  run: |
    SECRET=$(aws secretsmanager get-secret-value \
      --secret-id production/database/password \
      --query SecretString \
      --output text)
    echo "::add-mask::$SECRET"
    echo "DB_PASSWORD=$SECRET" >> $GITHUB_ENV

- name: Use secret
  run: |
    # Use $DB_PASSWORD
    ./deploy.sh
```

### Terraform 與 AWS Secrets Manager

```hcl
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "production/database/password"
}

resource "aws_db_instance" "main" {
  allocated_storage    = 100
  engine              = "postgres"
  instance_class      = "db.t3.large"
  username            = "admin"
  password            = jsondecode(data.aws_secretsmanager_secret_version.db_password.secret_string)["password"]
}
```

## GitHub Secrets

### 組織/儲存庫機敏資料

```yaml
- name: Use GitHub secret
  run: |
    echo "API Key: ${{ secrets.API_KEY }}"
    echo "Database URL: ${{ secrets.DATABASE_URL }}"
```

### 環境機敏資料

```yaml
deploy:
  runs-on: ubuntu-latest
  environment: production
  steps:
  - name: Deploy
    run: |
      echo "Deploying with ${{ secrets.PROD_API_KEY }}"
```

**參考資料：**參閱 `references/github-secrets.md`

## GitLab CI/CD 變數

### 專案變數

```yaml
deploy:
  script:
    - echo "Deploying with $API_KEY"
    - echo "Database: $DATABASE_URL"
```

### 受保護與遮罩變數
- 受保護：僅在受保護分支中可用
- 遮罩：在工作日誌中隱藏
- 檔案類型：以檔案形式儲存

## 最佳實務

1. **絕不將機敏資料提交**至 Git
2. **針對不同環境使用不同機敏資料**
3. **定期輪替機敏資料**
4. **實作最小權限存取**
5. **啟用稽核日誌記錄**
6. **使用機敏資料掃描**（GitGuardian、TruffleHog）
7. **在日誌中遮罩機敏資料**
8. **靜態加密機敏資料**
9. **盡可能使用短期權杖**
10. **記錄機敏資料需求**

## 機敏資料輪替

### AWS 自動輪替

```python
import boto3
import json

def lambda_handler(event, context):
    client = boto3.client('secretsmanager')

    # Get current secret
    response = client.get_secret_value(SecretId='my-secret')
    current_secret = json.loads(response['SecretString'])

    # Generate new password
    new_password = generate_strong_password()

    # Update database password
    update_database_password(new_password)

    # Update secret
    client.put_secret_value(
        SecretId='my-secret',
        SecretString=json.dumps({
            'username': current_secret['username'],
            'password': new_password
        })
    )

    return {'statusCode': 200}
```

### 手動輪替流程

1. 產生新的機敏資料
2. 在機敏資料儲存庫中更新機敏資料
3. 更新應用程式以使用新的機敏資料
4. 驗證功能正常
5. 撤銷舊的機敏資料

## External Secrets Operator

### Kubernetes 整合

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: production
spec:
  provider:
    vault:
      server: "https://vault.example.com:8200"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "production"

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: database-credentials
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: database/config
      property: username
  - secretKey: password
    remoteRef:
      key: database/config
      property: password
```

## 機敏資料掃描

### Pre-commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit

# Check for secrets with TruffleHog
docker run --rm -v "$(pwd):/repo" \
  trufflesecurity/trufflehog:latest \
  filesystem --directory=/repo

if [ $? -ne 0 ]; then
  echo "❌ Secret detected! Commit blocked."
  exit 1
fi
```

### CI/CD 機敏資料掃描

```yaml
secret-scan:
  stage: security
  image: trufflesecurity/trufflehog:latest
  script:
    - trufflehog filesystem .
  allow_failure: false
```

## 參考檔案

- `references/vault-setup.md` - HashiCorp Vault 設定
- `references/github-secrets.md` - GitHub Secrets 最佳實務

## 相關技能

- `github-actions-templates` - 適用於 GitHub Actions 整合
- `gitlab-ci-patterns` - 適用於 GitLab CI 整合
- `deployment-pipeline-design` - 適用於流水線架構
