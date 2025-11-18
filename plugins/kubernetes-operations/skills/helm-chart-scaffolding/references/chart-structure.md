# Helm Chart 結構參考

Helm chart 組織、檔案慣例和最佳實踐的完整指南。

## 標準 Chart 目錄結構

```
my-app/
├── Chart.yaml              # Chart metadata (required)
├── Chart.lock              # Dependency lock file (generated)
├── values.yaml             # Default configuration values (required)
├── values.schema.json      # JSON schema for values validation
├── .helmignore             # Patterns to ignore when packaging
├── README.md               # Chart documentation
├── LICENSE                 # Chart license
├── charts/                 # Chart dependencies (bundled)
│   └── postgresql-12.0.0.tgz
├── crds/                   # Custom Resource Definitions
│   └── my-crd.yaml
├── templates/              # Kubernetes manifest templates (required)
│   ├── NOTES.txt          # Post-install instructions
│   ├── _helpers.tpl       # Template helper functions
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── networkpolicy.yaml
│   └── tests/
│       └── test-connection.yaml
└── files/                  # Additional files to include
    └── config/
        └── app.conf
```

## Chart.yaml 規格

### API 版本 v2（Helm 3+）

```yaml
apiVersion: v2                    # Required: API version
name: my-application              # Required: Chart name
version: 1.2.3                    # Required: Chart version (SemVer)
appVersion: "2.5.0"              # Application version
description: A Helm chart for my application  # Required
type: application                 # Chart type: application or library
keywords:                         # Search keywords
  - web
  - api
  - backend
home: https://example.com         # Project home page
sources:                          # Source code URLs
  - https://github.com/example/my-app
maintainers:                      # Maintainer list
  - name: John Doe
    email: john@example.com
    url: https://github.com/johndoe
icon: https://example.com/icon.png  # Chart icon URL
kubeVersion: ">=1.24.0"          # Compatible Kubernetes versions
deprecated: false                 # Mark chart as deprecated
annotations:                      # Arbitrary annotations
  example.com/release-notes: https://example.com/releases/v1.2.3
dependencies:                     # Chart dependencies
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
    tags:
      - database
    import-values:
      - child: database
        parent: database
    alias: db
```

## Chart 類型

### Application Chart
```yaml
type: application
```
- 標準 Kubernetes 應用程式
- 可安裝和管理
- 包含 K8s 資源的範本

### Library Chart
```yaml
type: library
```
- 共用範本輔助函式
- 無法直接安裝
- 作為其他 chart 的相依性使用
- 沒有 templates/ 目錄

## Values 檔案組織

### values.yaml（預設值）
```yaml
# Global values (shared with subcharts)
global:
  imageRegistry: docker.io
  imagePullSecrets: []

# Image configuration
image:
  registry: docker.io
  repository: myapp/web
  tag: ""  # Defaults to .Chart.AppVersion
  pullPolicy: IfNotPresent

# Deployment settings
replicaCount: 1
revisionHistoryLimit: 10

# Pod configuration
podAnnotations: {}
podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 1000

# Container security
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
    - ALL

# Service
service:
  type: ClusterIP
  port: 80
  targetPort: http
  annotations: {}

# Resources
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

# Autoscaling
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80

# Node selection
nodeSelector: {}
tolerations: []
affinity: {}

# Monitoring
serviceMonitor:
  enabled: false
  interval: 30s
```

### values.schema.json（驗證）
```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1
    },
    "image": {
      "type": "object",
      "required": ["repository"],
      "properties": {
        "repository": {
          "type": "string"
        },
        "tag": {
          "type": "string"
        },
        "pullPolicy": {
          "type": "string",
          "enum": ["Always", "IfNotPresent", "Never"]
        }
      }
    }
  },
  "required": ["image"]
}
```

## 範本檔案

### 範本命名慣例

- **小寫加連字號**：`deployment.yaml`、`service-account.yaml`
- **部分範本**：以底線為前綴 `_helpers.tpl`
- **測試**：放在 `templates/tests/`
- **CRD**：放在 `crds/`（不範本化）

### 常見範本

#### _helpers.tpl
```yaml
{{/*
Standard naming helpers
*/}}
{{- define "my-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{- define "my-app.fullname" -}}
{{- if .Values.fullnameOverride -}}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" -}}
{{- else -}}
{{- $name := default .Chart.Name .Values.nameOverride -}}
{{- if contains $name .Release.Name -}}
{{- .Release.Name | trunc 63 | trimSuffix "-" -}}
{{- else -}}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" -}}
{{- end -}}
{{- end -}}
{{- end -}}

{{- define "my-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{/*
Common labels
*/}}
{{- define "my-app.labels" -}}
helm.sh/chart: {{ include "my-app.chart" . }}
{{ include "my-app.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end -}}

{{- define "my-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}

{{/*
Image name helper
*/}}
{{- define "my-app.image" -}}
{{- $registry := .Values.global.imageRegistry | default .Values.image.registry -}}
{{- $repository := .Values.image.repository -}}
{{- $tag := .Values.image.tag | default .Chart.AppVersion -}}
{{- printf "%s/%s:%s" $registry $repository $tag -}}
{{- end -}}
```

#### NOTES.txt
```
Thank you for installing {{ .Chart.Name }}.

Your release is named {{ .Release.Name }}.

To learn more about the release, try:

  $ helm status {{ .Release.Name }}
  $ helm get all {{ .Release.Name }}

{{- if .Values.ingress.enabled }}

Application URL:
{{- range .Values.ingress.hosts }}
  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ .host }}{{ .path }}
{{- end }}
{{- else }}

Get the application URL by running:
  export POD_NAME=$(kubectl get pods --namespace {{ .Release.Namespace }} -l "app.kubernetes.io/name={{ include "my-app.name" . }}" -o jsonpath="{.items[0].metadata.name}")
  kubectl port-forward $POD_NAME 8080:80
  echo "Visit http://127.0.0.1:8080"
{{- end }}
```

## 相依性管理

### 宣告相依性

```yaml
# Chart.yaml
dependencies:
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled  # Enable/disable via values
    tags:                          # Group dependencies
      - database
    import-values:                 # Import values from subchart
      - child: database
        parent: database
    alias: db                      # Reference as .Values.db
```

### 管理相依性

```bash
# Update dependencies
helm dependency update

# List dependencies
helm dependency list

# Build dependencies
helm dependency build
```

### Chart.lock

由 `helm dependency update` 自動產生：

```yaml
dependencies:
- name: postgresql
  repository: https://charts.bitnami.com/bitnami
  version: 12.0.0
digest: sha256:abcd1234...
generated: "2024-01-01T00:00:00Z"
```

## .helmignore

從 chart 套件中排除檔案：

```
# Development files
.git/
.gitignore
*.md
docs/

# Build artifacts
*.swp
*.bak
*.tmp
*.orig

# CI/CD
.travis.yml
.gitlab-ci.yml
Jenkinsfile

# Testing
test/
*.test

# IDE
.vscode/
.idea/
*.iml
```

## 自訂資源定義（CRD）

將 CRD 放在 `crds/` 目錄：

```
crds/
├── my-app-crd.yaml
└── another-crd.yaml
```

**重要 CRD 注意事項：**
- CRD 在任何範本之前安裝
- CRD 不會範本化（沒有 `{{ }}` 語法）
- CRD 不會隨 chart 升級或刪除
- 使用 `helm install --skip-crds` 跳過安裝

## Chart 版本控制

### 語意化版本控制

- **Chart 版本**：chart 變更時遞增
  - MAJOR：破壞性變更
  - MINOR：新功能，向後相容
  - PATCH：錯誤修正

- **應用程式版本**：正在部署的應用程式版本
  - 可以是任何字串
  - 不需要遵循 SemVer

```yaml
version: 2.3.1      # Chart version
appVersion: "1.5.0" # Application version
```

## Chart 測試

### 測試檔案

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-app.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "my-app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

### 執行測試

```bash
helm test my-release
helm test my-release --logs
```

## Hooks

Helm hooks 允許在特定時間點介入：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-app.fullname" . }}-migration
  annotations:
    "helm.sh/hook": pre-upgrade,pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

### Hook 類型

- `pre-install`：範本轉譯前
- `post-install`：所有資源載入後
- `pre-delete`：任何資源刪除前
- `post-delete`：所有資源刪除後
- `pre-upgrade`：升級前
- `post-upgrade`：升級後
- `pre-rollback`：回滾前
- `post-rollback`：回滾後
- `test`：使用 `helm test` 執行

### Hook 權重

控制 hook 執行順序（-5 到 5，數字小的先執行）

### Hook 刪除策略

- `before-hook-creation`：建立新 hook 前刪除先前的 hook
- `hook-succeeded`：成功執行後刪除
- `hook-failed`：hook 失敗時刪除

## 最佳實踐

1. **對重複的範本邏輯使用輔助函式**
2. **在範本中引用字串**：`{{ .Values.name | quote }}`
3. **使用 values.schema.json 驗證值**
4. **在 values.yaml 中記錄所有值**
5. **對 chart 版本使用語意化版本控制**
6. **精確固定相依性版本**
7. **包含 NOTES.txt** 提供使用說明
8. **為關鍵功能新增測試**
9. **使用 hooks** 進行資料庫遷移
10. **保持 chart 專注** - 每個 chart 一個應用程式

## Chart 儲存庫結構

```
helm-charts/
├── index.yaml
├── my-app-1.0.0.tgz
├── my-app-1.1.0.tgz
├── my-app-1.2.0.tgz
└── another-chart-2.0.0.tgz
```

### 建立儲存庫索引

```bash
helm repo index . --url https://charts.example.com
```

## 相關資源

- [Helm Documentation](https://helm.sh/docs/)
- [Chart Template Guide](https://helm.sh/docs/chart_template_guide/)
- [Best Practices](https://helm.sh/docs/chart_best_practices/)
