---
name: helm-chart-scaffolding
description: Design, organize, and manage Helm charts for templating and packaging Kubernetes applications with reusable configurations. Use when creating Helm charts, packaging Kubernetes applications, or implementing templated deployments.
---

# Helm Chart 腳手架

建立、組織和管理 Helm chart 以封裝和部署 Kubernetes 應用程式的完整指南。

## 目的

本技能提供建構生產就緒的 Helm chart 的逐步說明，包括 chart 結構、範本模式、值管理和驗證策略。

## 何時使用此技能

當您需要：
- 從頭建立新的 Helm chart
- 封裝 Kubernetes 應用程式以供發布
- 使用 Helm 管理多環境部署
- 實施可重複使用的 Kubernetes manifest 範本
- 設定 Helm chart 儲存庫
- 遵循 Helm 最佳實踐和慣例

## Helm 概述

**Helm** 是 Kubernetes 的套件管理工具，可以：
- 將 Kubernetes manifest 範本化以便重複使用
- 管理應用程式發布和回滾
- 處理 chart 之間的相依性
- 為部署提供版本控制
- 簡化跨環境的配置管理

## 逐步工作流程

### 1. 初始化 Chart 結構

**建立新 chart：**
```bash
helm create my-app
```

**標準 chart 結構：**
```
my-app/
├── Chart.yaml           # Chart metadata
├── values.yaml          # Default configuration values
├── charts/              # Chart dependencies
├── templates/           # Kubernetes manifest templates
│   ├── NOTES.txt       # Post-install notes
│   ├── _helpers.tpl    # Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   └── tests/
│       └── test-connection.yaml
└── .helmignore         # Files to ignore
```

### 2. 配置 Chart.yaml

**Chart 元資料定義套件：**

```yaml
apiVersion: v2
name: my-app
description: A Helm chart for My Application
type: application
version: 1.0.0      # Chart version
appVersion: "2.1.0" # Application version

# Keywords for chart discovery
keywords:
  - web
  - api
  - backend

# Maintainer information
maintainers:
  - name: DevOps Team
    email: devops@example.com
    url: https://github.com/example/my-app

# Source code repository
sources:
  - https://github.com/example/my-app

# Homepage
home: https://example.com

# Chart icon
icon: https://example.com/icon.png

# Dependencies
dependencies:
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
  - name: redis
    version: "17.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
```

**參考資料：**完整範例請參閱 `assets/Chart.yaml.template`

### 3. 設計 values.yaml 結構

**階層式組織值：**

```yaml
# Image configuration
image:
  repository: myapp
  tag: "1.0.0"
  pullPolicy: IfNotPresent

# Number of replicas
replicaCount: 3

# Service configuration
service:
  type: ClusterIP
  port: 80
  targetPort: 8080

# Ingress configuration
ingress:
  enabled: false
  className: nginx
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix

# Resources
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"

# Autoscaling
autoscaling:
  enabled: false
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

# Environment variables
env:
  - name: LOG_LEVEL
    value: "info"

# ConfigMap data
configMap:
  data:
    APP_MODE: production

# Dependencies
postgresql:
  enabled: true
  auth:
    database: myapp
    username: myapp

redis:
  enabled: false
```

**參考資料：**完整結構請參閱 `assets/values.yaml.template`

### 4. 建立範本檔案

**使用 Go 範本搭配 Helm 函式：**

**templates/deployment.yaml：**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
        env:
          {{- toYaml .Values.env | nindent 12 }}
```

### 5. 建立範本輔助函式

**templates/_helpers.tpl：**
```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "my-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "my-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

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
{{- end }}

{{/*
Selector labels
*/}}
{{- define "my-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### 6. 管理相依性

**在 Chart.yaml 中新增相依性：**
```yaml
dependencies:
  - name: postgresql
    version: "12.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
```

**更新相依性：**
```bash
helm dependency update
helm dependency build
```

**覆寫相依性值：**
```yaml
# values.yaml
postgresql:
  enabled: true
  auth:
    database: myapp
    username: myapp
    password: changeme
  primary:
    persistence:
      enabled: true
      size: 10Gi
```

### 7. 測試與驗證

**驗證命令：**
```bash
# Lint the chart
helm lint my-app/

# Dry-run installation
helm install my-app ./my-app --dry-run --debug

# Template rendering
helm template my-app ./my-app

# Template with values
helm template my-app ./my-app -f values-prod.yaml

# Show computed values
helm show values ./my-app
```

**驗證腳本：**
```bash
#!/bin/bash
set -e

echo "Linting chart..."
helm lint .

echo "Testing template rendering..."
helm template test-release . --dry-run

echo "Checking for required values..."
helm template test-release . --validate

echo "All validations passed!"
```

**參考資料：**請參閱 `scripts/validate-chart.sh`

### 8. 封裝與發布

**封裝 chart：**
```bash
helm package my-app/
# Creates: my-app-1.0.0.tgz
```

**建立 chart 儲存庫：**
```bash
# Create index
helm repo index .

# Upload to repository
# AWS S3 example
aws s3 sync . s3://my-helm-charts/ --exclude "*" --include "*.tgz" --include "index.yaml"
```

**使用 chart：**
```bash
helm repo add my-repo https://charts.example.com
helm repo update
helm install my-app my-repo/my-app
```

### 9. 多環境配置

**環境特定值檔案：**

```
my-app/
├── values.yaml          # Defaults
├── values-dev.yaml      # Development
├── values-staging.yaml  # Staging
└── values-prod.yaml     # Production
```

**values-prod.yaml：**
```yaml
replicaCount: 5

image:
  tag: "2.1.0"

resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20

ingress:
  enabled: true
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix

postgresql:
  enabled: true
  primary:
    persistence:
      size: 100Gi
```

**使用環境配置安裝：**
```bash
helm install my-app ./my-app -f values-prod.yaml --namespace production
```

### 10. 實施 Hooks 與測試

**預安裝 hook：**
```yaml
# templates/pre-install-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-app.fullname" . }}-db-setup
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: db-setup
        image: postgres:15
        command: ["psql", "-c", "CREATE DATABASE myapp"]
      restartPolicy: Never
```

**測試連線：**
```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-app.fullname" . }}-test-connection"
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "my-app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

**執行測試：**
```bash
helm test my-app
```

## 常見模式

### 模式 1：條件式資源

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "my-app.fullname" . }}
spec:
  # ...
{{- end }}
```

### 模式 2：迭代清單

```yaml
env:
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}
```

### 模式 3：包含檔案

```yaml
data:
  config.yaml: |
    {{- .Files.Get "config/application.yaml" | nindent 4 }}
```

### 模式 4：全域值

```yaml
global:
  imageRegistry: docker.io
  imagePullSecrets:
    - name: regcred

# Use in templates:
image: {{ .Values.global.imageRegistry }}/{{ .Values.image.repository }}
```

## 最佳實踐

1. **對 chart 和應用程式版本使用語意化版本控制**
2. **在 values.yaml 中以註解記錄所有值**
3. **使用範本輔助函式** 處理重複邏輯
4. **在封裝前驗證 chart**
5. **明確固定相依性版本**
6. **對可選資源使用條件**
7. **遵循命名慣例**（小寫、連字號）
8. **包含 NOTES.txt** 提供使用說明
9. **使用輔助函式一致地新增標籤**
10. **在所有環境測試安裝**

## 疑難排解

**範本轉譯錯誤：**
```bash
helm template my-app ./my-app --debug
```

**相依性問題：**
```bash
helm dependency update
helm dependency list
```

**安裝失敗：**
```bash
helm install my-app ./my-app --dry-run --debug
kubectl get events --sort-by='.lastTimestamp'
```

## 參考檔案

- `assets/Chart.yaml.template` - Chart 元資料範本
- `assets/values.yaml.template` - 值結構範本
- `scripts/validate-chart.sh` - 驗證腳本
- `references/chart-structure.md` - 詳細 chart 組織

## 相關技能

- `k8s-manifest-generator` - 用於建立基礎 Kubernetes manifest
- `gitops-workflow` - 用於自動化 Helm chart 部署
