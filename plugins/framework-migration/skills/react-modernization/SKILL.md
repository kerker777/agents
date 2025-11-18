---
name: react-modernization
description: 升級 React 應用程式至最新版本、從類別元件遷移至 hooks，以及採用並行功能。適用於現代化 React 程式碼庫、遷移至 React Hooks 或升級至最新 React 版本。
---

# React Modernization

精通 React 版本升級、類別元件轉 hooks 遷移、並行功能採用，以及用於自動轉換的 codemods。

## 何時使用此技能

- 升級 React 應用程式至最新版本
- 將類別元件遷移至函式元件與 hooks
- 採用並行 React 功能（Suspense、transitions）
- 應用 codemods 進行自動重構
- 現代化狀態管理模式
- 更新至 TypeScript
- 使用 React 18+ 功能改善效能

## 版本升級路徑

### React 16 → 17 → 18

**各版本的破壞性變更：**

**React 17：**
- 事件委派變更
- 移除事件池
- Effect 清理時機
- JSX 轉換（不需要匯入 React）

**React 18：**
- 自動批次處理
- 並行渲染
- Strict Mode 變更（雙重調用）
- 新的 root API
- 伺服器端 Suspense

## 類別元件轉 Hooks 遷移

### 狀態管理
```javascript
// Before: Class component
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      name: ''
    };
  }

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}

// After: Functional component with hooks
function Counter() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  const increment = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
}
```

### 生命週期方法轉 Hooks
```javascript
// Before: Lifecycle methods
class DataFetcher extends React.Component {
  state = { data: null, loading: true };

  componentDidMount() {
    this.fetchData();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData();
    }
  }

  componentWillUnmount() {
    this.cancelRequest();
  }

  fetchData = async () => {
    const data = await fetch(`/api/${this.props.id}`);
    this.setState({ data, loading: false });
  };

  cancelRequest = () => {
    // Cleanup
  };

  render() {
    if (this.state.loading) return <div>Loading...</div>;
    return <div>{this.state.data}</div>;
  }
}

// After: useEffect hook
function DataFetcher({ id }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      try {
        const response = await fetch(`/api/${id}`);
        const result = await response.json();

        if (!cancelled) {
          setData(result);
          setLoading(false);
        }
      } catch (error) {
        if (!cancelled) {
          console.error(error);
        }
      }
    };

    fetchData();

    // Cleanup function
    return () => {
      cancelled = true;
    };
  }, [id]); // Re-run when id changes

  if (loading) return <div>Loading...</div>;
  return <div>{data}</div>;
}
```

### Context 和 HOCs 轉 Hooks
```javascript
// Before: Context consumer and HOC
const ThemeContext = React.createContext();

class ThemedButton extends React.Component {
  static contextType = ThemeContext;

  render() {
    return (
      <button style={{ background: this.context.theme }}>
        {this.props.children}
      </button>
    );
  }
}

// After: useContext hook
function ThemedButton({ children }) {
  const { theme } = useContext(ThemeContext);

  return (
    <button style={{ background: theme }}>
      {children}
    </button>
  );
}

// Before: HOC for data fetching
function withUser(Component) {
  return class extends React.Component {
    state = { user: null };

    componentDidMount() {
      fetchUser().then(user => this.setState({ user }));
    }

    render() {
      return <Component {...this.props} user={this.state.user} />;
    }
  };
}

// After: Custom hook
function useUser() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser().then(setUser);
  }, []);

  return user;
}

function UserProfile() {
  const user = useUser();
  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

## React 18 並行功能

### 新的 Root API
```javascript
// Before: React 17
import ReactDOM from 'react-dom';

ReactDOM.render(<App />, document.getElementById('root'));

// After: React 18
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

### 自動批次處理
```javascript
// React 18: All updates are batched
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // Only one re-render (batched)
}

// Even in async:
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // Still batched in React 18!
}, 1000);

// Opt out if needed
import { flushSync } from 'react-dom';

flushSync(() => {
  setCount(c => c + 1);
});
// Re-render happens here
setFlag(f => !f);
// Another re-render
```

### Transitions
```javascript
import { useState, useTransition } from 'react';

function SearchResults() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    // Urgent: Update input immediately
    setQuery(e.target.value);

    // Non-urgent: Update results (can be interrupted)
    startTransition(() => {
      setResults(searchResults(e.target.value));
    });
  };

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <Results data={results} />
    </>
  );
}
```

### Suspense for Data Fetching
```javascript
import { Suspense } from 'react';

// Resource-based data fetching (with React 18)
const resource = fetchProfileData();

function ProfilePage() {
  return (
    <Suspense fallback={<Loading />}>
      <ProfileDetails />
      <Suspense fallback={<Loading />}>
        <ProfileTimeline />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails() {
  // This will suspend if data not ready
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
  const posts = resource.posts.read();
  return <Timeline posts={posts} />;
}
```

## Codemods 自動化

### 執行 React Codemods
```bash
# Install jscodeshift
npm install -g jscodeshift

# React 16.9 codemod (rename unsafe lifecycle methods)
npx react-codeshift <transform> <path>

# Example: Rename UNSAFE_ methods
npx react-codeshift --parser=tsx \
  --transform=react-codeshift/transforms/rename-unsafe-lifecycles.js \
  src/

# Update to new JSX Transform (React 17+)
npx react-codeshift --parser=tsx \
  --transform=react-codeshift/transforms/new-jsx-transform.js \
  src/

# Class to Hooks (third-party)
npx codemod react/hooks/convert-class-to-function src/
```

### 自訂 Codemod 範例
```javascript
// custom-codemod.js
module.exports = function(file, api) {
  const j = api.jscodeshift;
  const root = j(file.source);

  // Find setState calls
  root.find(j.CallExpression, {
    callee: {
      type: 'MemberExpression',
      property: { name: 'setState' }
    }
  }).forEach(path => {
    // Transform to useState
    // ... transformation logic
  });

  return root.toSource();
};

// Run: jscodeshift -t custom-codemod.js src/
```

## 效能優化

### useMemo 和 useCallback
```javascript
function ExpensiveComponent({ items, filter }) {
  // Memoize expensive calculation
  const filteredItems = useMemo(() => {
    return items.filter(item => item.category === filter);
  }, [items, filter]);

  // Memoize callback to prevent child re-renders
  const handleClick = useCallback((id) => {
    console.log('Clicked:', id);
  }, []); // No dependencies, never changes

  return (
    <List items={filteredItems} onClick={handleClick} />
  );
}

// Child component with memo
const List = React.memo(({ items, onClick }) => {
  return items.map(item => (
    <Item key={item.id} item={item} onClick={onClick} />
  ));
});
```

### 程式碼分割
```javascript
import { lazy, Suspense } from 'react';

// Lazy load components
const Dashboard = lazy(() => import('./Dashboard'));
const Settings = lazy(() => import('./Settings'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

## TypeScript 遷移

```typescript
// Before: JavaScript
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

// After: TypeScript
interface ButtonProps {
  onClick: () => void;
  children: React.ReactNode;
}

function Button({ onClick, children }: ButtonProps) {
  return <button onClick={onClick}>{children}</button>;
}

// Generic components
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return <>{items.map(renderItem)}</>;
}
```

## 遷移檢查清單

```markdown
### 遷移前
- [ ] 漸進式更新依賴項（不要一次全部更新）
- [ ] 檢視發行說明中的破壞性變更
- [ ] 設定測試套件
- [ ] 建立功能分支

### 類別元件 → Hooks 遷移
- [ ] 識別要遷移的類別元件
- [ ] 從葉節點元件開始（無子元件）
- [ ] 將 state 轉換為 useState
- [ ] 將生命週期轉換為 useEffect
- [ ] 將 context 轉換為 useContext
- [ ] 提取自訂 hooks
- [ ] 徹底測試

### React 18 升級
- [ ] 先更新至 React 17（如需要）
- [ ] 更新 react 和 react-dom 至 18
- [ ] 更新 @types/react（如使用 TypeScript）
- [ ] 變更為 createRoot API
- [ ] 使用 StrictMode 測試（雙重調用）
- [ ] 處理並行渲染問題
- [ ] 在有益處的地方採用 Suspense/Transitions

### 效能
- [ ] 識別效能瓶頸
- [ ] 在適當的地方加入 React.memo
- [ ] 對耗時運算使用 useMemo/useCallback
- [ ] 實作程式碼分割
- [ ] 優化重新渲染

### 測試
- [ ] 更新測試工具（React Testing Library）
- [ ] 使用 React 18 功能測試
- [ ] 檢查控制台警告
- [ ] 效能測試
```

## 資源

- **references/breaking-changes.md**：版本特定的破壞性變更
- **references/codemods.md**：Codemod 使用指南
- **references/hooks-migration.md**：全面的 hooks 模式
- **references/concurrent-features.md**：React 18 並行功能
- **assets/codemod-config.json**：Codemod 設定
- **assets/migration-checklist.md**：逐步檢查清單
- **scripts/apply-codemods.sh**：自動化 codemod 腳本

## 最佳實務

1. **漸進式遷移**：不要一次遷移所有內容
2. **徹底測試**：每個步驟都要全面測試
3. **使用 Codemods**：自動化重複性轉換
4. **從簡單開始**：從葉節點元件開始
5. **善用 StrictMode**：及早發現問題
6. **監控效能**：測量前後效能
7. **撰寫文件**：保留遷移日誌

## 常見陷阱

- 忘記 useEffect 依賴項
- 過度使用 useMemo/useCallback
- 未在 useEffect 中處理清理
- 混合類別和函式模式
- 忽略 StrictMode 警告
- 假設破壞性變更
