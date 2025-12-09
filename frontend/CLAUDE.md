[根目录](../CLAUDE.md) > **frontend**

# Frontend 模块文档

> 最后更新：2025-12-09 08:27:02

## 模块职责

Frontend 模块是基于 Next.js 15 和 React 19 构建的现代化 Web 应用，为 Open Notebook 提供用户界面。它实现了响应式设计、实时交互、主题切换等功能，并与后端 API 完全集成。

## 入口与启动

### 主入口文件
- **`src/app/layout.tsx`** - 根布局组件
- **`src/app/page.tsx`** - 首页组件
- **`next.config.ts`** - Next.js 配置

### 启动方式
```bash
# 开发模式
npm run dev
# 或
make frontend

# 生产构建
npm run build

# 启动生产服务器
npm start
```

### 应用配置
- **默认端口**: 3000（开发）, 8502（生产）
- **API 代理**: 开发时代理到 `http://localhost:5055`
- **环境变量**: 见 `.env.local`

## 技术栈

### 核心框架
- **Next.js 15** - React 全栈框架
- **React 19** - UI 库
- **TypeScript 5** - 类型安全

### UI 组件
- **Tailwind CSS 4** - 样式框架
- **Radix UI** - 无头组件库
- **Lucide React** - 图标库
- **React Markdown** - Markdown 渲染

### 状态管理
- **Zustand** - 轻量级状态管理
- **TanStack Query** - 服务器状态管理
- **React Hook Form** - 表单状态

### 开发工具
- **ESLint 9** - 代码检查
- **Prettier** - 代码格式化
- **TypeScript** - 类型检查

## 目录结构

```
frontend/
├── public/              # 静态资源
├── src/
│   ├── app/            # App Router 页面
│   │   ├── (auth)/     # 认证相关页面
│   │   ├── (dashboard)/# 主应用页面
│   │   ├── api/        # API 路由
│   │   ├── layout.tsx  # 根布局
│   │   └── page.tsx    # 首页
│   ├── components/     # 通用组件
│   │   ├── ui/         # 基础 UI 组件
│   │   ├── auth/       # 认证组件
│   │   ├── common/     # 通用组件
│   │   ├── layout/     # 布局组件
│   │   ├── notebooks/  # 笔记本组件
│   │   ├── podcasts/   # 播客组件
│   │   └── sources/    # 源文件组件
│   ├── lib/            # 工具函数
│   ├── hooks/          # 自定义 Hooks
│   ├── stores/         # Zustand stores
│   └── types/          # TypeScript 类型
├── components.json     # shadcn/ui 配置
├── tailwind.config.ts  # Tailwind 配置
├── tsconfig.json       # TypeScript 配置
└── package.json        # 项目配置
```

## 页面路由

### 认证路由 `(auth)`
- **`/login`** - 登录页面

### 仪表板路由 `(dashboard)`
- **`/`** - 仪表板首页
- **`/notebooks`** - 笔记本列表
- **`/notebooks/[id]`** - 笔记本详情（三栏布局）
- **`/sources`** - 源文件管理
- **`/sources/[id]`** - 源文件详情
- **`/search`** - 搜索页面
- **`/podcasts`** - 播客管理
- **`/models`** - AI 模型配置
- **`/transformations`** - 内容转换
- **`/settings`** - 系统设置
- **`/advanced`** - 高级功能

## 核心组件

### 布局组件
- **`AppShell`** - 应用外壳
- **`AppSidebar`** - 侧边栏导航
- **ConnectionGuard`** - API 连接保护

### 笔记本组件
- **`NotebookList`** - 笔记本列表
- **`NotebookCard`** - 笔记本卡片
- **`NotebookHeader`** - 笔记本标题栏
- **`SourcesColumn`** - 源文件列
- **`NotesColumn`** - 笔记列
- **`ChatColumn`** - 对话列

### 源文件组件
- **`AddSourceDialog`** - 添加源文件对话框
- **`SourceCard`** - 源文件卡片
- **`SourceDetailContent`** - 源文件详情
- **`ChatPanel`** - 源文件对话面板

### 播客组件
- **`GeneratePodcastDialog`** - 生成播客对话框
- **`EpisodeCard`** - 节目卡片
- **`SpeakerProfilesPanel`** - 说话人配置
- **`EpisodeProfilesPanel`** - 节目配置

### 通用组件
- **`ModelSelector`** - AI 模型选择器
- **`ContextToggle`** - 上下文控制
- **`ThemeToggle`** - 主题切换
- **`CommandPalette`** - 命令面板

## 状态管理

### Zustand Stores
```typescript
// 认证状态
useAuthStore
{
  user: User | null
  isAuthenticated: boolean
  login: (credentials) => Promise<void>
  logout: () => void
}

// 应用设置
useSettingsStore
{
  theme: 'light' | 'dark' | 'system'
  apiConnected: boolean
  toggleTheme: () => void
}
```

### TanStack Query
```typescript
// 笔记本查询
useQuery({
  queryKey: ['notebooks'],
  queryFn: fetchNotebooks
})

// 源文件变更
useMutation({
  mutationFn: uploadSource,
  onSuccess: () => queryClient.invalidateQueries()
})
```

## API 集成

### API 客户端
```typescript
// lib/api.ts
const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL || '/api'
})

// 请求拦截器
api.interceptors.request.use(config => {
  // 添加认证头
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})
```

### 类型定义
```typescript
// types/api.ts
interface Notebook {
  id: string
  name: string
  description?: string
  created_at: string
  sources: Source[]
}

interface Source {
  id: string
  title: string
  type: string
  status: 'processing' | 'completed' | 'error'
}
```

## 主题系统

### 主题配置
- **Light/Dark 主题** - 使用 next-themes
- **系统主题** - 跟随系统设置
- **主题切换** - 平滑过渡动画

### 使用示例
```typescript
import { useTheme } from 'next-themes'

function ThemeToggle() {
  const { theme, setTheme } = useTheme()

  return (
    <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
      切换主题
    </button>
  )
}
```

## 表单处理

### React Hook Form
```typescript
// 添加笔记本表单
const {
  register,
  handleSubmit,
  formState: { errors }
} = useForm<NotebookFormData>()

// 提交处理
const onSubmit = async (data: NotebookFormData) => {
  await createNotebook(data)
}
```

### Zod 验证
```typescript
import { z } from 'zod'

const notebookSchema = z.object({
  name: z.string().min(1, '名称不能为空'),
  description: z.string().optional()
})
```

## 错误处理

### 错误边界
- **`ErrorBoundary`** - 捕获 React 错误
- **`ConnectionErrorOverlay`** - API 连接错误

### 错误显示
```typescript
// 使用 Sonner toast
import { toast } from 'sonner'

// 成功提示
toast.success('操作成功')

// 错误提示
toast.error('操作失败：', { description: error.message })
```

## 性能优化

### 代码分割
- 路由级别的自动代码分割
- 组件懒加载
```typescript
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <div>加载中...</div>
})
```

### 图片优化
- Next.js Image 组件
- 自动格式转换和尺寸优化

### 缓存策略
- TanStack Query 缓存
- SWR 模式用于实时数据

## 测试

### 测试工具
- **Jest** - 测试框架
- **React Testing Library** - 组件测试
- **Playwright** - E2E 测试（计划中）

### 测试示例
```typescript
import { render, screen } from '@testing-library/react'
import { NotebookCard } from './NotebookCard'

test('渲染笔记本卡片', () => {
  render(<NotebookCard notebook={mockNotebook} />)
  expect(screen.getByText('测试笔记本')).toBeInTheDocument()
})
```

## 部署

### 构建命令
```bash
# 开发构建
npm run build

# 生产构建
npm run build && npm start
```

### 环境变量
```bash
NEXT_PUBLIC_API_URL=http://localhost:5055
NEXT_PUBLIC_WS_URL=ws://localhost:5055
NEXT_PUBLIC_APP_NAME=Open Notebook
```

## 开发指南

### 添加新页面
1. 在 `src/app/` 下创建路由文件夹
2. 添加 `page.tsx` 文件
3. 可选：添加 `layout.tsx` 自定义布局

### 创建新组件
1. 在 `src/components/` 对应目录创建
2. 使用 TypeScript 编写
3. 添加 Props 类型定义
4. 编写测试文件

### 样式指南
1. 使用 Tailwind CSS 类
2. 响应式设计（sm:、md:、lg:、xl:）
3. 组件变体使用 cva
4. 保持一致的设计系统

## 常见问题 (FAQ)

### Q: 如何添加新的 API 端点？
A: 在 `lib/api.ts` 中添加函数，使用 TanStack Query 调用。

### Q: 如何实现实时更新？
A: 使用 WebSocket 或轮询，配合 TanStack Query 的 invalidateQueries。

### Q: 如何优化大列表渲染？
A: 使用虚拟化，如 react-window 或 react-virtualized。

### Q: 如何处理文件上传？
A: 使用 FormData 和 axios，配合进度条组件。

## 相关文件清单

### 配置文件
- `next.config.ts` - Next.js 配置
- `tailwind.config.ts` - Tailwind 配置
- `tsconfig.json` - TypeScript 配置
- `components.json` - shadcn/ui 配置

### 核心页面
- `src/app/layout.tsx` - 根布局
- `src/app/page.tsx` - 首页
- `src/app/(dashboard)/notebooks/[id]/page.tsx` - 笔记本详情

### 核心组件
- `src/components/layout/AppShell.tsx` - 应用布局
- `src/components/notebooks/NotebookList.tsx` - 笔记本列表
- `src/components/sources/AddSourceDialog.tsx` - 添加源文件

### 工具和配置
- `src/lib/api.ts` - API 客户端
- `src/types/index.ts` - 类型定义
- `src/hooks/useNotebooks.ts` - 自定义 Hook

## 变更记录 (Changelog)

### 2025-12-09 08:27:02
- 📝 创建前端模块文档
- 🏗️ 整理组件结构
- 📊 添加路由说明
- 🔧 补充开发指南

---

*此文档由 AI 自动生成，如需更新请参考项目贡献指南*