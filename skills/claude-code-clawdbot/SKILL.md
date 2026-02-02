---
name: claude-code-clawdbot
description: "Run Claude Code (Anthropic) from this host via the `claude` CLI (Agent SDK) in headless mode (`-p`) for codebase analysis, refactors, test fixing, and structured output. Use when the user asks to use Claude Code, run `claude -p`, use Plan Mode, auto-approve tools with --allowedTools, generate JSON output, or integrate Claude Code into Clawdbot workflows/cron." 
---

# Claude Code (Clawdbot)

Use the locally installed **Claude Code** CLI reliably.

This skill supports two execution styles:
- **Headless mode** (non-interactive): best for normal prompts and structured output.
- **Interactive mode (tmux)**: required for **slash commands** like `/speckit.*` (Spec Kit), which can hang or be killed when run via headless `-p`.

This skill is for **driving the Claude Code CLI**, not the Claude API directly.

## Quick checks

Verify installation:
```bash
claude --version
```

Run a minimal headless prompt (prints a single response):
```bash
./scripts/claude_code_run.py -p "Return only the single word OK."
```

## Core workflow

### 1) Run a headless prompt in a repo

```bash
cd /path/to/repo
/home/ubuntu/clawd/skills/claude-code-clawdbot/scripts/claude_code_run.py \
  -p "Summarize this project and point me to the key modules." \
  --permission-mode plan
```

### 2) Allow tools (auto-approve)

Claude Code supports tool allowlists via `--allowedTools`.
Example: allow read/edit + bash:
```bash
./scripts/claude_code_run.py \
  -p "Run the test suite and fix any failures." \
  --allowedTools "Bash,Read,Edit"
```

### 3) Get structured output

```bash
./scripts/claude_code_run.py \
  -p "Summarize this repo in 5 bullets." \
  --output-format json
```

### 4) Add extra system instructions

```bash
./scripts/claude_code_run.py \
  -p "Review the staged diff for security issues." \
  --append-system-prompt "You are a security engineer. Be strict." \
  --allowedTools "Bash(git diff *),Bash(git status *),Read"
```

## Notes (important)

- Claude Code sometimes expects a TTY.
- **Headless**: this wrapper uses `script(1)` to force a pseudo-terminal.
- **Slash commands** (e.g. `/speckit.*`) are best run in **interactive** mode; this wrapper can start an interactive Claude Code session in **tmux**.
- Use `--permission-mode plan` when you want read-only planning.
- Keep `--allowedTools` narrow (principle of least privilege), especially in automation.

## High‑leverage Claude Code tips (from the official docs)

### 1) Always give Claude a way to verify (tests/build/screenshots)

Claude performs dramatically better when it can verify its work.
Make verification explicit in the prompt, e.g.:
- “Fix the bug **and run tests**. Done when `npm test` passes.”
- “Implement UI change, **take a screenshot** and compare to this reference.”

### 2) Explore → Plan → Implement (use Plan Mode)

For multi-step work, start in plan mode to do safe, read-only analysis:
```bash
./scripts/claude_code_run.py -p "Analyze and propose a plan" --permission-mode plan
```
Then switch to execution (`acceptEdits`) once the plan is approved.

### 3) Manage context aggressively: /clear and /compact

Long, mixed-topic sessions degrade quality.
- Use `/clear` between unrelated tasks.
- Use `/compact Focus on <X>` when nearing limits to preserve the right details.

### 4) Rewind aggressively: /rewind (checkpoints)

Claude checkpoints before changes.
If an approach is wrong, use `/rewind` (or Esc Esc) to restore:
- conversation only
- code only
- both

This enables “try something risky → rewind if wrong” loops.

### 5) Prefer CLAUDE.md for durable rules; keep it short

Best practice is a concise CLAUDE.md (global or per-project) for:
- build/test commands Claude should use
- repo etiquette / style rules that differ from defaults
- non-obvious environment quirks

Overlong CLAUDE.md files get ignored.

### 6) Permissions: deny > ask > allow (and scope matters)

In `.claude/settings.json` / `~/.claude/settings.json`, rules match in order:
**deny first**, then ask, then allow.
Use deny rules to block secrets (e.g. `.env`, `secrets/**`).

### 7) Bash env vars don’t persist; use CLAUDE_ENV_FILE for persistence

Each Bash tool call runs in a fresh shell; `export FOO=bar` won’t persist.
If you need persistent env setup, set (before starting Claude Code):
```bash
export CLAUDE_ENV_FILE=/path/to/env-setup.sh
```
Claude will source it before each Bash command.

### 8) Hooks beat “please remember” instructions

Use hooks to enforce deterministic actions (format-on-edit, block writes to sensitive dirs, etc.)
when you need guarantees.

### 9) Use subagents for heavy investigation / independent review

Subagents can read many files without polluting the main context.
Use them for broad codebase research or post-implementation review.

### 10) Treat Claude as a Unix utility (headless, pipes, structured output)

Examples:
```bash
cat build-error.txt | claude -p "Explain root cause" 
claude -p "List endpoints" --output-format json
```
This is ideal for CI and automation.

## Interactive mode (tmux)

If your prompt contains lines starting with `/` (slash commands), the wrapper defaults to **auto → interactive**.

Example:

```bash
./scripts/claude_code_run.py \
  --mode auto \
  --permission-mode acceptEdits \
  --allowedTools "Bash,Read,Edit,Write" \
  -p $'/speckit.constitution ...\n/speckit.specify ...\n/speckit.plan ...\n/speckit.tasks\n/speckit.implement'
```

It will print tmux attach/capture commands so you can monitor progress.

## Spec Kit end-to-end workflow (tips that prevent hangs)

When you want Claude Code to drive **Spec Kit** end-to-end via `/speckit.*`, do **not** use headless `-p` for the whole flow.
Use **interactive tmux mode** because:
- Spec Kit runs multiple steps (Bash + file writes + git) and may pause for confirmations.
- Headless runs can appear idle and be killed (SIGKILL) by supervisors.

### Prerequisites (important)

1) **Initialize Spec Kit** (once per repo)
```bash
specify init . --ai claude
```

2) Ensure the folder is a real git repo (Spec Kit uses git branches/scripts):
```bash
git init
git add -A
git commit -m "chore: init"
```

3) Recommended: set an `origin` remote (can be a local bare repo) so `git fetch --all --prune` won’t behave oddly:
```bash
git init --bare ../origin.git
git remote add origin ../origin.git
git push -u origin main || git push -u origin master
```

4) Give Claude Code enough tool permissions for the workflow:
- Spec creation/tasks/implement need file writes, so include **Write**.
- Implementation often needs Bash.

Recommended:
```bash
--permission-mode acceptEdits --allowedTools "Bash,Read,Edit,Write"
```

### Run the full Spec Kit pipeline

```bash
./scripts/claude_code_run.py \
  --mode interactive \
  --tmux-session cc-speckit \
  --permission-mode acceptEdits \
  --allowedTools "Bash,Read,Edit,Write" \
  -p $'/speckit.constitution Create project principles for quality, accessibility, and security.\n/speckit.specify <your feature description>\n/speckit.plan I am building with <your stack/constraints>\n/speckit.tasks\n/speckit.implement'
```

### Monitoring / interacting

The wrapper prints commands like:
- `tmux ... attach -t <session>` to watch in real time
- `tmux ... capture-pane ...` to snapshot output

If Claude Code asks a question mid-run (e.g., “Proceed?”), attach and answer.

## Operational gotchas (learned in practice)

### 1) Vite + ngrok: "Blocked request. This host (...) is not allowed"

If you expose a Vite dev server through ngrok, Vite will block unknown Host headers unless configured.

- **Vite 7** expects `server.allowedHosts` to be `true` or `string[]`.
  - ✅ Allow all hosts (quick):
    ```ts
    server: { host: true, allowedHosts: true }
    ```
  - ✅ Allow just your ngrok host (safer):
    ```ts
    server: { host: true, allowedHosts: ['xxxx.ngrok-free.app'] }
    ```
  - ❌ Do **not** set `allowedHosts: 'all'` (won't work in Vite 7).

After changing `vite.config.*`, restart the dev server.

### 2) Don’t accidentally let your *shell* eat your prompt

When you drive tmux via a shell command (e.g. `tmux send-keys ...`), avoid unescaped **backticks** and shell substitutions in the text you pass.
They can be interpreted by your shell before the text even reaches Claude Code.

Practical rule:
- Prefer sending prompts from a file, or ensure the wrapper/script quotes prompt text safely.

### 3) Long-running dev servers should run in a persistent session

In automation environments, backgrounded `vite` / `ngrok` processes can get SIGKILL.
Prefer running them in a managed background session (Clawdbot exec background) or tmux, and explicitly stop them when done.

## OpenSpec workflow (opsx)

OpenSpec is another spec-driven workflow (like Spec Kit) powered by slash commands (e.g. `/opsx:*`).
In practice it has the same reliability constraints:
- Prefer **interactive tmux mode** for `/opsx:*` commands (avoid headless `-p` for the whole flow).

### Setup (per machine)

Install CLI:
```bash
npm install -g @fission-ai/openspec@latest
```

### Setup (per project)

Initialize OpenSpec **with tool selection** (required):
```bash
openspec init --tools claude
```

Tip: disable telemetry if desired:
```bash
export OPENSPEC_TELEMETRY=0
```

### Recommended end-to-end command sequence

Inside Claude Code (interactive):
1) `/opsx:onboard`
2) `/opsx:new <change-name>`
3) `/opsx:ff` (fast-forward: generates proposal/design/specs/tasks)
4) `/opsx:apply` (implements tasks)
5) `/opsx:archive` (optional: archive finished change)

If the UI prompts you for project type/stack, answer explicitly (e.g. “Web app (HTML/JS) with localStorage”).

## Bundled script

- `scripts/claude_code_run.py`: wrapper that runs the local `claude` binary with a pseudo-terminal and forwards flags.

---

# React + TypeScript 项目开发实战经验

这些经验来自于实际开发中遇到的问题和解决方案，可作为快速参考。

## 1. 常见 React Hooks 导入问题

**问题症状**：
- 页面完全空白
- 浏览器控制台报错：`ReferenceError: useState is not defined`
- Vite HMR 无法正常更新

**根本原因**：
使用了 React hooks（如 `useState`, `useEffect`, `useReducer`）但忘记从 `react` 导入。

**快速修复**：
```typescript
// ❌ 错误
import { useEffect } from 'react';

function App() {
  const [state, setState] = useState(null);  // ReferenceError
  // ...
}

// ✅ 正确
import { useEffect, useState } from 'react';

function App() {
  const [state, setState] = useState(null);  // 正常工作
  // ...
}
```

**预防措施**：
1. 使用 TypeScript 时，类型检查会提示未定义的变量
2. 配置 ESLint 规则检测未导入的 hooks
3. 使用 IDE 的自动导入功能

## 2. Vite 开发服务器常见问题

### 问题：Vite HMR 不生效

**症状**：修改代码后浏览器没有自动更新，需要手动刷新。

**解决方案**：
1. 检查 `vite.config.ts` 配置是否正确
2. 确保没有禁用 HMR
3. 检查浏览器控制台是否有 WebSocket 连接错误
4. 尝试重启开发服务器

### 问题：Vite + ngrok "Blocked request" 错误

**症状**：
```
Blocked request. This host (xxxx.ngrok-free.app) is not allowed
```

**解决方案**（Vite 7+）：
```typescript
// vite.config.ts
export default defineConfig({
  server: {
    host: true,  // 监听所有网络接口
    allowedHosts: true,  // 允许所有主机（快速方案）
    // 或更安全的方式：
    // allowedHosts: ['xxxx.ngrok-free.app']
  }
})
```

**注意**：
- ❌ `allowedHosts: 'all'` 在 Vite 7 中不起作用
- ✅ 使用 `true` 或字符串数组

### 问题：开发服务器端口被占用

**症状**：
```
Error: Port 3000 is already in use
```

**解决方案**：
```bash
# 方案 1: 使用其他端口
npm run dev -- --port 3001

# 方案 2: 杀死占用端口的进程
# Windows
netstat -ano | findstr :3000
taskkill /PID <进程ID> /F

# macOS/Linux
lsof -ti:3000 | xargs kill -9
```

## 3. React 项目状态管理最佳实践

### 简单状态：使用 useState
```typescript
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### 复杂状态：使用 useReducer
适用于有多个相关操作的状态管理。

```typescript
type Action =
  | { type: 'ADD_ITEM'; payload: Item }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_ITEM'; payload: { id: string; item: Partial<Item> } };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD_ITEM':
      return { ...state, items: [...state.items, action.payload] };
    case 'REMOVE_ITEM':
      return { ...state, items: state.items.filter(i => i.id !== action.payload) };
    case 'UPDATE_ITEM':
      return {
        ...state,
        items: state.items.map(i =>
          i.id === action.payload.id ? { ...i, ...action.payload.item } : i
        ),
      };
    default:
      return state;
  }
}

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  // ...
}
```

**何时使用 useReducer**：
- 状态逻辑复杂，有多个子值
- 下一个状态依赖于前一个状态
- 需要为复杂状态编写可测试的 reducer 函数

## 4. API 集成最佳实践

### 使用免费 API 代替需要 Key 的服务

**案例：私人日记项目的天气集成**

**原始方案**（OpenWeatherMap）：
- ❌ 需要用户手动输入 API Key
- ❌ 有请求限制
- ❌ 可能需要付费

**优化方案**（Open-Meteo）：
- ✅ 完全免费
- ✅ 不需要 API Key
- ✅ 无请求限制
- ✅ 支持全球位置

```typescript
// 使用 Open-Meteo 获取天气
async function getWeather(lat: string, lon: string) {
  const response = await fetch(
    `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true`
  );
  const data = await response.json();
  return data.current_weather;
}
```

**推荐免费 API**：
- 天气：Open-Meteo, wttr.in
- 图片：Unsplash, Pexels
- 地图：Leaflet + OpenStreetMap
- 文本：本地 LLM（Ollama）

### API 缓存策略

减少 API 调用，提升性能：

```typescript
const CACHE_DURATION = 60 * 60 * 1000; // 1 小时

export async function getWeatherWithCache(date: string) {
  const cacheKey = `weather_${date}`;
  const cached = localStorage.getItem(cacheKey);

  if (cached) {
    const { data, timestamp } = JSON.parse(cached);
    if (Date.now() - timestamp < CACHE_DURATION) {
      return data; // 返回缓存数据
    }
  }

  const data = await fetchWeatherFromAPI();
  localStorage.setItem(cacheKey, JSON.stringify({
    data,
    timestamp: Date.now(),
  }));

  return data;
}
```

## 5. LocalStorage 使用注意事项

### 数据容量限制

LocalStorage 通常限制在 5-10MB。

**最佳实践**：
1. 压缩大型数据（JSON + gzip）
2. 分片存储（多个键）
3. 使用 IndexedDB 存储大量数据

```typescript
// 检查容量
function checkStorageSpace() {
  const test = 'x'.repeat(1024 * 1024); // 1MB
  try {
    localStorage.setItem('test', test);
    localStorage.removeItem('test');
    return true;
  } catch (e) {
    console.error('Storage limit exceeded');
    return false;
  }
}
```

### 数据序列化

LocalStorage 只能存储字符串：

```typescript
// ✅ 正确：存储前序列化，读取后解析
const entries = [{ id: 1, title: 'Test' }];
localStorage.setItem('entries', JSON.stringify(entries));
const loaded = JSON.parse(localStorage.getItem('entries') || '[]');

// ❌ 错误：直接存储对象
localStorage.setItem('entries', entries); // 存储为 "[object Object]"
```

### 错误处理

LocalStorage 可能被禁用（隐私模式）：

```typescript
function safeSetItem(key: string, value: string) {
  try {
    localStorage.setItem(key, value);
    return true;
  } catch (e) {
    console.error('LocalStorage access failed:', e);
    alert('无法保存数据，请检查浏览器设置');
    return false;
  }
}
```

## 6. Tailwind CSS 响应式设计

### 移动优先策略

```tsx
<div className="
  // 手机（默认）
  p-4 text-sm

  // 平板（≥768px）
  md:p-6 md:text-base

  // 桌面（≥1024px）
  lg:p-8 lg:text-lg
">
  响应式内容
</div>
```

### 深色模式

**配置**（`tailwind.config.js`）：
```js
module.exports = {
  darkMode: 'class', // 使用 class 策略
  // ...
};
```

**使用**：
```tsx
function App() {
  const [darkMode, setDarkMode] = useState(false);

  useEffect(() => {
    if (darkMode) {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
  }, [darkMode]);

  return (
    <div className={darkMode ? 'dark bg-gray-900 text-white' : 'bg-white text-gray-900'}>
      <button onClick={() => setDarkMode(!darkMode)}>
        {darkMode ? '🌙' : '☀️'}
      </button>
    </div>
  );
}
```

## 7. TypeScript 类型定义最佳实践

### 定义清晰的接口

```typescript
// ✅ 好：明确的类型定义
interface DiaryEntry {
  id: string;
  date: string;  // ISO 格式 YYYY-MM-DD
  content: string;
  mood?: 'happy' | 'neutral' | 'sad' | 'excited' | 'thoughtful';
  tags: string[];
  isMemory: boolean;
  weather?: WeatherData;
  createdAt: string;  // ISO timestamp
  updatedAt: string;
}

interface WeatherData {
  temp: number;
  description: string;
  icon: string;
  humidity: number;
  windSpeed: number;
}
```

### 类型守卫

```typescript
function isDiaryEntry(obj: unknown): obj is DiaryEntry {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'content' in obj &&
    'date' in obj
  );
}
```

## 8. 项目初始化检查清单

使用 Claude Code + Spec Kit 开发 React 项目时，确保：

### 文件结构
```
project/
├── src/
│   ├── components/      # 可复用组件
│   ├── services/       # 外部服务（API）
│   ├── utils/          # 工具函数
│   ├── types/          # TypeScript 类型
│   ├── App.tsx        # 主应用
│   ├── main.tsx       # 入口
│   └── index.css      # 全局样式
├── package.json
├── tsconfig.json
├── vite.config.ts
└── tailwind.config.js
```

### 依赖检查
```json
{
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0",
    "tailwindcss": "^3.0.0",
    "postcss": "^8.0.0",
    "autoprefixer": "^10.0.0"
  }
}
```

### 配置检查

**tsconfig.json**：
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

**vite.config.ts**：
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    host: true,
    port: 3000,
    allowedHosts: true,
  },
});
```

## 9. 实战案例：私人日记项目

### 使用 Spec Kit + Claude Code 完整工作流

**步骤 1：初始化项目**
```bash
cd /path/to/project
specify init . --ai claude
git init
git add -A
git commit -m "chore: init"
```

**步骤 2：运行 Spec Kit**
```bash
./scripts/claude_code_run.py \
  --mode interactive \
  --tmux-session diary-speckit \
  --permission-mode acceptEdits \
  --allowedTools "Bash,Read,Edit,Write" \
  -p $'/speckit.constitution ...\n/speckit.specify ...\n/speckit.plan ...\n/speckit.tasks\n/speckit.implement'
```

**步骤 3：常见问题修复**

#### 问题 1：React hooks 未导入
**症状**：页面空白，控制台报错
**修复**：
```typescript
// 在 App.tsx 顶部添加
import { useEffect, useReducer, useState } from 'react';
```

#### 问题 2：天气 API 需要 Key
**症状**：需要用户手动配置才能使用
**优化**：改用 Open-Meteo 免费服务
```typescript
// 原来：需要 API Key
const response = await fetch(
  `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}`
);

// 优化后：不需要 Key
const response = await fetch(
  `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current_weather=true`
);
```

#### 问题 3：类型定义不完整
**症状**：TypeScript 报错缺少属性
**修复**：
```typescript
// 更新类型定义，移除不再需要的字段
export interface DiaryState {
  entries: DiaryEntry[];
  selectedDate: string;
  searchQuery: string;
  showMemoriesOnly: boolean;
  darkMode: boolean;
  // 移除 weatherApiKey
}
```

### 项目运行检查

```bash
# 1. 安装依赖
npm install

# 2. 启动开发服务器
npm run dev

# 3. 访问 http://localhost:3000

# 4. 检查控制台是否有错误

# 5. 测试功能
#    - 创建日记
#    - 查看天气
#    - 切换深色模式
#    - 搜索功能
```

## 10. 调试技巧

### 浏览器开发者工具

**控制台错误**：
- 按 F12 打开开发者工具
- 查看 Console 标签页的红色错误
- 常见错误：
  - `ReferenceError`: 变量未定义
  - `TypeError`: 类型错误（如 `.map` 用于非数组）
  - `SyntaxError`: 语法错误

**网络请求**：
- Network 标签页查看 API 请求
- 检查状态码（200, 404, 500）
- 查看 Response 数据格式

**React DevTools**：
- 安装 React Developer Tools 浏览器扩展
- 查看组件树和状态
- 调试 props 和 hooks

### Vite 日志

```bash
# 启动时检查警告
npm run dev

# 常见警告：
# - "! Some chunks are larger than 500kB"
#   解决：代码分割（lazy loading）
# - "Circular dependency"
#   解决：重构代码结构
```

### TypeScript 类型检查

```bash
# 运行类型检查
npx tsc --noEmit

# 修复所有类型错误后再提交
```

## 11. Git 工作流

### 提交前检查
```bash
# 1. 查看修改
git status

# 2. 检查差异
git diff

# 3. 运行测试
npm test

# 4. 类型检查
npx tsc --noEmit

# 5. 提交
git add .
git commit -m "feat: add feature description"
```

### 提交消息规范（Conventional Commits）
```
feat: add weather widget with Open-Meteo API
fix: resolve useState import error
docs: update README with setup instructions
style: format code with Prettier
refactor: simplify reducer logic
test: add unit tests for storage utils
chore: update dependencies
```

---

## 总结

这些实战经验涵盖了：
- ✅ React 常见问题（hooks 导入、状态管理）
- ✅ Vite 配置（HMR、ngrok 兼容）
- ✅ API 集成（免费服务、缓存策略）
- ✅ LocalStorage 最佳实践（容量限制、错误处理）
- ✅ TypeScript 类型定义
- ✅ 响应式设计和深色模式
- ✅ 完整项目开发流程（Spec Kit + Claude Code）
- ✅ 调试技巧和 Git 工作流

在开发 React + TypeScript 项目时，参考这些经验可以快速定位和解决问题。
