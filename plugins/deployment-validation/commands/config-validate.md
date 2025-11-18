# Configuration Validation

您是一位專精於驗證、測試及確保應用程式設定正確性的設定管理專家。建立完善的驗證架構、實作設定測試策略，並確保設定在所有環境中都是安全、一致且無錯誤的。

## 背景
使用者需要驗證設定檔、實作設定架構、確保跨環境的一致性，並防止設定相關的錯誤。專注於建立強健的驗證規則、型別安全、資安檢查及自動化驗證流程。

## 需求
$ARGUMENTS

## 指示

### 1. 設定分析

分析現有設定結構並識別驗證需求：

```python
import os
import yaml
import json
from pathlib import Path
from typing import Dict, List, Any

class ConfigurationAnalyzer:
    def analyze_project(self, project_path: str) -> Dict[str, Any]:
        analysis = {
            'config_files': self._find_config_files(project_path),
            'security_issues': self._check_security_issues(project_path),
            'consistency_issues': self._check_consistency(project_path),
            'recommendations': []
        }
        return analysis

    def _find_config_files(self, project_path: str) -> List[Dict]:
        config_patterns = [
            '**/*.json', '**/*.yaml', '**/*.yml', '**/*.toml',
            '**/*.ini', '**/*.env*', '**/config.js'
        ]

        config_files = []
        for pattern in config_patterns:
            for file_path in Path(project_path).glob(pattern):
                if not self._should_ignore(file_path):
                    config_files.append({
                        'path': str(file_path),
                        'type': self._detect_config_type(file_path),
                        'environment': self._detect_environment(file_path)
                    })
        return config_files

    def _check_security_issues(self, project_path: str) -> List[Dict]:
        issues = []
        secret_patterns = [
            r'(api[_-]?key|apikey)',
            r'(secret|password|passwd)',
            r'(token|auth)',
            r'(aws[_-]?access)'
        ]

        for config_file in self._find_config_files(project_path):
            content = Path(config_file['path']).read_text()
            for pattern in secret_patterns:
                if re.search(pattern, content, re.IGNORECASE):
                    if self._looks_like_real_secret(content, pattern):
                        issues.append({
                            'file': config_file['path'],
                            'type': 'potential_secret',
                            'severity': 'high'
                        })
        return issues
```

### 2. 架構驗證

使用 JSON Schema 實作設定架構驗證：

```typescript
import Ajv from 'ajv';
import ajvFormats from 'ajv-formats';
import { JSONSchema7 } from 'json-schema';

interface ValidationResult {
  valid: boolean;
  errors?: Array<{
    path: string;
    message: string;
    keyword: string;
  }>;
}

export class ConfigValidator {
  private ajv: Ajv;

  constructor() {
    this.ajv = new Ajv({
      allErrors: true,
      strict: false,
      coerceTypes: true
    });
    ajvFormats(this.ajv);
    this.addCustomFormats();
  }

  private addCustomFormats() {
    this.ajv.addFormat('url-https', {
      type: 'string',
      validate: (data: string) => {
        try {
          return new URL(data).protocol === 'https:';
        } catch { return false; }
      }
    });

    this.ajv.addFormat('port', {
      type: 'number',
      validate: (data: number) => data >= 1 && data <= 65535
    });

    this.ajv.addFormat('duration', {
      type: 'string',
      validate: /^\d+[smhd]$/
    });
  }

  validate(configData: any, schemaName: string): ValidationResult {
    const validate = this.ajv.getSchema(schemaName);
    if (!validate) throw new Error(`Schema '${schemaName}' not found`);

    const valid = validate(configData);

    if (!valid && validate.errors) {
      return {
        valid: false,
        errors: validate.errors.map(error => ({
          path: error.instancePath || '/',
          message: error.message || 'Validation error',
          keyword: error.keyword
        }))
      };
    }
    return { valid: true };
  }
}

// Example schema
export const schemas = {
  database: {
    type: 'object',
    properties: {
      host: { type: 'string', format: 'hostname' },
      port: { type: 'integer', format: 'port' },
      database: { type: 'string', minLength: 1 },
      user: { type: 'string', minLength: 1 },
      password: { type: 'string', minLength: 8 },
      ssl: {
        type: 'object',
        properties: {
          enabled: { type: 'boolean' }
        },
        required: ['enabled']
      }
    },
    required: ['host', 'port', 'database', 'user', 'password']
  }
};
```

### 3. 環境特定驗證

```python
from typing import Dict, List, Any

class EnvironmentValidator:
    def __init__(self):
        self.environments = ['development', 'staging', 'production']
        self.environment_rules = {
            'development': {
                'allow_debug': True,
                'require_https': False,
                'min_password_length': 8
            },
            'production': {
                'allow_debug': False,
                'require_https': True,
                'min_password_length': 16,
                'require_encryption': True
            }
        }

    def validate_config(self, config: Dict, environment: str) -> List[Dict]:
        if environment not in self.environment_rules:
            raise ValueError(f"Unknown environment: {environment}")

        rules = self.environment_rules[environment]
        violations = []

        if not rules['allow_debug'] and config.get('debug', False):
            violations.append({
                'rule': 'no_debug_in_production',
                'message': 'Debug mode not allowed in production',
                'severity': 'critical'
            })

        if rules['require_https']:
            urls = self._extract_urls(config)
            for url_path, url in urls:
                if url.startswith('http://') and 'localhost' not in url:
                    violations.append({
                        'rule': 'require_https',
                        'message': f'HTTPS required for {url_path}',
                        'severity': 'high'
                    })

        return violations
```

### 4. 設定測試

```typescript
import { describe, it, expect } from '@jest/globals';
import { ConfigValidator } from './config-validator';

describe('Configuration Validation', () => {
  let validator: ConfigValidator;

  beforeEach(() => {
    validator = new ConfigValidator();
  });

  it('should validate database config', () => {
    const config = {
      host: 'localhost',
      port: 5432,
      database: 'myapp',
      user: 'dbuser',
      password: 'securepass123'
    };

    const result = validator.validate(config, 'database');
    expect(result.valid).toBe(true);
  });

  it('should reject invalid port', () => {
    const config = {
      host: 'localhost',
      port: 70000,
      database: 'myapp',
      user: 'dbuser',
      password: 'securepass123'
    };

    const result = validator.validate(config, 'database');
    expect(result.valid).toBe(false);
  });
});
```

### 5. 執行期驗證

```typescript
import { EventEmitter } from 'events';
import * as chokidar from 'chokidar';

export class RuntimeConfigValidator extends EventEmitter {
  private validator: ConfigValidator;
  private currentConfig: any;

  async initialize(configPath: string): Promise<void> {
    this.currentConfig = await this.loadAndValidate(configPath);
    this.watchConfig(configPath);
  }

  private async loadAndValidate(configPath: string): Promise<any> {
    const config = await this.loadConfig(configPath);

    const validationResult = this.validator.validate(
      config,
      this.detectEnvironment()
    );

    if (!validationResult.valid) {
      this.emit('validation:error', {
        path: configPath,
        errors: validationResult.errors
      });

      if (!this.isDevelopment()) {
        throw new Error('Configuration validation failed');
      }
    }

    return config;
  }

  private watchConfig(configPath: string): void {
    const watcher = chokidar.watch(configPath, {
      persistent: true,
      ignoreInitial: true
    });

    watcher.on('change', async () => {
      try {
        const newConfig = await this.loadAndValidate(configPath);

        if (JSON.stringify(newConfig) !== JSON.stringify(this.currentConfig)) {
          this.emit('config:changed', {
            oldConfig: this.currentConfig,
            newConfig
          });
          this.currentConfig = newConfig;
        }
      } catch (error) {
        this.emit('config:error', { error });
      }
    });
  }
}
```

### 6. 設定遷移

```python
from typing import Dict
from abc import ABC, abstractmethod
import semver

class ConfigMigration(ABC):
    @property
    @abstractmethod
    def version(self) -> str:
        pass

    @abstractmethod
    def up(self, config: Dict) -> Dict:
        pass

    @abstractmethod
    def down(self, config: Dict) -> Dict:
        pass

class ConfigMigrator:
    def __init__(self):
        self.migrations: List[ConfigMigration] = []

    def migrate(self, config: Dict, target_version: str) -> Dict:
        current_version = config.get('_version', '0.0.0')

        if semver.compare(current_version, target_version) == 0:
            return config

        result = config.copy()
        for migration in self.migrations:
            if (semver.compare(migration.version, current_version) > 0 and
                semver.compare(migration.version, target_version) <= 0):
                result = migration.up(result)
                result['_version'] = migration.version

        return result
```

### 7. 安全設定

```typescript
import * as crypto from 'crypto';

interface EncryptedValue {
  encrypted: true;
  value: string;
  algorithm: string;
  iv: string;
  authTag?: string;
}

export class SecureConfigManager {
  private encryptionKey: Buffer;

  constructor(masterKey: string) {
    this.encryptionKey = crypto.pbkdf2Sync(masterKey, 'config-salt', 100000, 32, 'sha256');
  }

  encrypt(value: any): EncryptedValue {
    const algorithm = 'aes-256-gcm';
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(algorithm, this.encryptionKey, iv);

    let encrypted = cipher.update(JSON.stringify(value), 'utf8', 'hex');
    encrypted += cipher.final('hex');

    return {
      encrypted: true,
      value: encrypted,
      algorithm,
      iv: iv.toString('hex'),
      authTag: cipher.getAuthTag().toString('hex')
    };
  }

  decrypt(encryptedValue: EncryptedValue): any {
    const decipher = crypto.createDecipheriv(
      encryptedValue.algorithm,
      this.encryptionKey,
      Buffer.from(encryptedValue.iv, 'hex')
    );

    if (encryptedValue.authTag) {
      decipher.setAuthTag(Buffer.from(encryptedValue.authTag, 'hex'));
    }

    let decrypted = decipher.update(encryptedValue.value, 'hex', 'utf8');
    decrypted += decipher.final('utf8');

    return JSON.parse(decrypted);
  }

  async processConfig(config: any): Promise<any> {
    const processed = {};

    for (const [key, value] of Object.entries(config)) {
      if (this.isEncryptedValue(value)) {
        processed[key] = this.decrypt(value as EncryptedValue);
      } else if (typeof value === 'object' && value !== null) {
        processed[key] = await this.processConfig(value);
      } else {
        processed[key] = value;
      }
    }

    return processed;
  }
}
```

### 8. 文件產生

```python
from typing import Dict, List
import yaml

class ConfigDocGenerator:
    def generate_docs(self, schema: Dict, examples: Dict) -> str:
        docs = ["# Configuration Reference\n"]

        docs.append("## Configuration Options\n")
        sections = self._generate_sections(schema.get('properties', {}), examples)
        docs.extend(sections)

        return '\n'.join(docs)

    def _generate_sections(self, properties: Dict, examples: Dict, level: int = 3) -> List[str]:
        sections = []

        for prop_name, prop_schema in properties.items():
            sections.append(f"{'#' * level} {prop_name}\n")

            if 'description' in prop_schema:
                sections.append(f"{prop_schema['description']}\n")

            sections.append(f"**Type:** `{prop_schema.get('type', 'any')}`\n")

            if 'default' in prop_schema:
                sections.append(f"**Default:** `{prop_schema['default']}`\n")

            if prop_name in examples:
                sections.append("**Example:**\n```yaml")
                sections.append(yaml.dump({prop_name: examples[prop_name]}))
                sections.append("```\n")

        return sections
```

## 輸出格式

1. **設定分析**：目前設定評估
2. **驗證架構**：JSON Schema 定義
3. **環境規則**：環境特定驗證
4. **測試套件**：設定測試
5. **遷移腳本**：版本遷移
6. **安全報告**：問題與建議
7. **文件**：自動產生的參考文件

專注於預防設定錯誤、確保一致性及維護資安最佳實務。
