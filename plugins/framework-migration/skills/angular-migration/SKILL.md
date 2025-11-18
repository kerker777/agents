---
name: angular-migration
description: 使用混合模式、漸進式元件重寫和依賴注入更新，從 AngularJS 遷移至 Angular。適用於升級 AngularJS 應用程式、規劃框架遷移或現代化舊版 Angular 程式碼。
---

# Angular Migration

精通 AngularJS 至 Angular 的遷移，包括混合應用程式、元件轉換、依賴注入變更和路由遷移。

## 何時使用此技能

- 將 AngularJS (1.x) 應用程式遷移至 Angular (2+)
- 執行 AngularJS/Angular 混合應用程式
- 將指令轉換為元件
- 現代化依賴注入
- 遷移路由系統
- 更新至最新 Angular 版本
- 實作 Angular 最佳實務

## 遷移策略

### 1. Big Bang（完全重寫）
- 在 Angular 中重寫整個應用程式
- 平行開發
- 一次性切換
- **最適合：**小型應用程式、全新專案

### 2. Incremental（漸進式方法）
- AngularJS 和 Angular 並行執行
- 逐個功能遷移
- 使用 ngUpgrade 進行互通
- **最適合：**大型應用程式、持續交付

### 3. Vertical Slice（垂直切片）
- 完整遷移單一功能
- 新功能使用 Angular，舊功能維護於 AngularJS
- 逐步替換
- **最適合：**中型應用程式、明確的功能區分

## 混合應用程式設定

```typescript
// main.ts - Bootstrap hybrid app
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { UpgradeModule } from '@angular/upgrade/static';
import { AppModule } from './app/app.module';

platformBrowserDynamic()
  .bootstrapModule(AppModule)
  .then(platformRef => {
    const upgrade = platformRef.injector.get(UpgradeModule);
    // Bootstrap AngularJS
    upgrade.bootstrap(document.body, ['myAngularJSApp'], { strictDi: true });
  });
```

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { UpgradeModule } from '@angular/upgrade/static';

@NgModule({
  imports: [
    BrowserModule,
    UpgradeModule
  ]
})
export class AppModule {
  constructor(private upgrade: UpgradeModule) {}

  ngDoBootstrap() {
    // Bootstrapped manually in main.ts
  }
}
```

## 元件遷移

### AngularJS Controller → Angular Component
```javascript
// Before: AngularJS controller
angular.module('myApp').controller('UserController', function($scope, UserService) {
  $scope.user = {};

  $scope.loadUser = function(id) {
    UserService.getUser(id).then(function(user) {
      $scope.user = user;
    });
  };

  $scope.saveUser = function() {
    UserService.saveUser($scope.user);
  };
});
```

```typescript
// After: Angular component
import { Component, OnInit } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user',
  template: `
    <div>
      <h2>{{ user.name }}</h2>
      <button (click)="saveUser()">Save</button>
    </div>
  `
})
export class UserComponent implements OnInit {
  user: any = {};

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.loadUser(1);
  }

  loadUser(id: number) {
    this.userService.getUser(id).subscribe(user => {
      this.user = user;
    });
  }

  saveUser() {
    this.userService.saveUser(this.user);
  }
}
```

### AngularJS Directive → Angular Component
```javascript
// Before: AngularJS directive
angular.module('myApp').directive('userCard', function() {
  return {
    restrict: 'E',
    scope: {
      user: '=',
      onDelete: '&'
    },
    template: `
      <div class="card">
        <h3>{{ user.name }}</h3>
        <button ng-click="onDelete()">Delete</button>
      </div>
    `
  };
});
```

```typescript
// After: Angular component
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <button (click)="delete.emit()">Delete</button>
    </div>
  `
})
export class UserCardComponent {
  @Input() user: any;
  @Output() delete = new EventEmitter<void>();
}

// Usage: <app-user-card [user]="user" (delete)="handleDelete()"></app-user-card>
```

## 服務遷移

```javascript
// Before: AngularJS service
angular.module('myApp').factory('UserService', function($http) {
  return {
    getUser: function(id) {
      return $http.get('/api/users/' + id);
    },
    saveUser: function(user) {
      return $http.post('/api/users', user);
    }
  };
});
```

```typescript
// After: Angular service
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  constructor(private http: HttpClient) {}

  getUser(id: number): Observable<any> {
    return this.http.get(`/api/users/${id}`);
  }

  saveUser(user: any): Observable<any> {
    return this.http.post('/api/users', user);
  }
}
```

## 依賴注入變更

### Downgrading Angular → AngularJS
```typescript
// Angular service
import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class NewService {
  getData() {
    return 'data from Angular';
  }
}

// Make available to AngularJS
import { downgradeInjectable } from '@angular/upgrade/static';

angular.module('myApp')
  .factory('newService', downgradeInjectable(NewService));

// Use in AngularJS
angular.module('myApp').controller('OldController', function(newService) {
  console.log(newService.getData());
});
```

### Upgrading AngularJS → Angular
```typescript
// AngularJS service
angular.module('myApp').factory('oldService', function() {
  return {
    getData: function() {
      return 'data from AngularJS';
    }
  };
});

// Make available to Angular
import { InjectionToken } from '@angular/core';

export const OLD_SERVICE = new InjectionToken<any>('oldService');

@NgModule({
  providers: [
    {
      provide: OLD_SERVICE,
      useFactory: (i: any) => i.get('oldService'),
      deps: ['$injector']
    }
  ]
})

// Use in Angular
@Component({...})
export class NewComponent {
  constructor(@Inject(OLD_SERVICE) private oldService: any) {
    console.log(this.oldService.getData());
  }
}
```

## 路由遷移

```javascript
// Before: AngularJS routing
angular.module('myApp').config(function($routeProvider) {
  $routeProvider
    .when('/users', {
      template: '<user-list></user-list>'
    })
    .when('/users/:id', {
      template: '<user-detail></user-detail>'
    });
});
```

```typescript
// After: Angular routing
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

## 表單遷移

```html
<!-- Before: AngularJS -->
<form name="userForm" ng-submit="saveUser()">
  <input type="text" ng-model="user.name" required>
  <input type="email" ng-model="user.email" required>
  <button ng-disabled="userForm.$invalid">Save</button>
</form>
```

```typescript
// After: Angular (Template-driven)
@Component({
  template: `
    <form #userForm="ngForm" (ngSubmit)="saveUser()">
      <input type="text" [(ngModel)]="user.name" name="name" required>
      <input type="email" [(ngModel)]="user.email" name="email" required>
      <button [disabled]="userForm.invalid">Save</button>
    </form>
  `
})

// Or Reactive Forms (preferred)
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="saveUser()">
      <input formControlName="name">
      <input formControlName="email">
      <button [disabled]="userForm.invalid">Save</button>
    </form>
  `
})
export class UserFormComponent {
  userForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.userForm = this.fb.group({
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]]
    });
  }

  saveUser() {
    console.log(this.userForm.value);
  }
}
```

## 遷移時程

```
階段 1：設定 (1-2 週)
- 安裝 Angular CLI
- 建立混合應用程式
- 設定建置工具
- 設定測試環境

階段 2：基礎設施 (2-4 週)
- 遷移服務
- 遷移工具程式
- 設定路由
- 遷移共用元件

階段 3：功能遷移 (視情況而定)
- 逐個功能遷移
- 徹底測試
- 漸進式部署

階段 4：清理 (1-2 週)
- 移除 AngularJS 程式碼
- 移除 ngUpgrade
- 優化套件
- 最終測試
```

## 資源

- **references/hybrid-mode.md**：混合應用程式模式
- **references/component-migration.md**：元件轉換指南
- **references/dependency-injection.md**：依賴注入遷移策略
- **references/routing.md**：路由遷移
- **assets/hybrid-bootstrap.ts**：混合應用程式範本
- **assets/migration-timeline.md**：專案規劃
- **scripts/analyze-angular-app.sh**：應用程式分析腳本

## 最佳實務

1. **從服務開始**：優先遷移服務（較容易）
2. **漸進式方法**：逐個功能遷移
3. **持續測試**：每個步驟都要測試
4. **使用 TypeScript**：盡早遷移至 TypeScript
5. **遵循樣式指南**：從第一天開始遵循 Angular 樣式指南
6. **稍後優化**：先讓它運作，再進行優化
7. **撰寫文件**：保留遷移筆記

## 常見陷阱

- 未正確設定混合應用程式
- 在邏輯之前遷移 UI
- 忽略變更偵測的差異
- 未正確處理作用域
- 混合模式（AngularJS + Angular）
- 測試不足
