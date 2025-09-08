# 前端架构设计方案（含代码规范）

## 技术栈选型

- **框架**: Vue 3.x (Composition API)
- **构建工具**: Vite
- **UI组件库**: Element Plus
- **类型系统**: TypeScript
- **CSS框架**: Tailwind CSS
- **路由**: Vue Router
- **状态管理**: Pinia
- **微前端**: micro-app
- **包管理**: pnpm
- **多包管理**: Lerna
- **代码格式化**: Prettier
- **代码检查**: ESLint

## 项目结构设计

```text
project-root/
├── packages/
│   ├── main-app/          # 主应用
│   │   ├── src/
│   │   ├── package.json
│   │   ├── vite.config.ts
│   │   ├── .eslintrc.js
│   │   └── .prettierrc
│   ├── micro-app-1/       # 微应用1
│   │   ├── src/
│   │   ├── package.json
│   │   ├── vite.config.ts
│   │   ├── .eslintrc.js
│   │   └── .prettierrc
│   └── micro-app-2/       # 微应用2
│       ├── src/
│       ├── package.json
│       ├── vite.config.ts
│       ├── .eslintrc.js
│       └── .prettierrc
├── .eslintignore
├── .prettierignore
├── lerna.json
├── package.json
└── pnpm-workspace.yaml
```



## 详细配置说明

### 1. 根目录配置

**pnpm-workspace.yaml**

```yaml
packages:
  - 'packages/*'
```



**lerna.json**

```json
{
  "$schema": "node_modules/lerna/schemas/lerna-schema.json",
  "version": "independent",
  "npmClient": "pnpm",
  "useWorkspaces": true,
  "packages": ["packages/*"]
}
```



**package.json**

```json
{
  "name": "frontend-architecture",
  "private": true,
  "scripts": {
    "dev": "lerna run dev --parallel",
    "build": "lerna run build",
    "preview": "lerna run preview --parallel",
    "lint": "lerna run lint",
    "lint:fix": "lerna run lint:fix",
    "format": "lerna run format",
    "clean": "lerna clean"
  },
  "devDependencies": {
    "lerna": "^6.0.0",
    "eslint": "^8.38.0",
    "prettier": "^2.8.7",
    "@vue/eslint-config-typescript": "^11.0.2",
    "@vue/eslint-config-prettier": "^7.1.0"
  }
}
```



### 2. ESLint 配置

**.eslintignore

```text
node_modules/
dist/
*.min.js
*.bundle.js
coverage/
```



**packages/main-app/.eslintrc.js**

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    '@vue/typescript/recommended',
    'plugin:vue/vue3-recommended',
    'prettier'
  ],
  parserOptions: {
    ecmaVersion: 2021,
    parser: '@typescript-eslint/parser',
    sourceType: 'module'
  },
  plugins: ['vue', '@typescript-eslint'],
  rules: {
    'vue/multi-word-component-names': 'off',
    '@typescript-eslint/no-explicit-any': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    'vue/html-self-closing': [
      'error',
      {
        html: {
          void: 'always',
          normal: 'never',
          component: 'always'
        }
      }
    ]
  }
}
```



### 3. Prettier 配置

**.prettierignore**

```text
node_modules/
dist/
*.min.js
*.bundle.js
coverage/
package-lock.json
yarn.lock
pnpm-lock.yaml
```



**packages/main-app/.prettierrc**

```text
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "none",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "avoid",
  "endOfLine": "auto",
  "vueIndentScriptAndStyle": true,
  "htmlWhitespaceSensitivity": "ignore"
}
```



### 4. 主应用配置

**packages/main-app/package.json**

```json
{
  "name": "main-app",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc --noEmit && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --ignore-path .gitignore",
    "lint:fix": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix --ignore-path .gitignore",
    "format": "prettier --write \"src/**/*.{js,ts,vue,html,css,scss}\""
  },
  "dependencies": {
    "@micro-zoe/micro-app": "^0.8.0",
    "element-plus": "^2.3.0",
    "pinia": "^2.0.0",
    "vue": "^3.3.0",
    "vue-router": "^4.2.0"
  },
  "devDependencies": {
    "@types/node": "^18.16.0",
    "@vitejs/plugin-vue": "^4.2.0",
    "@vue/eslint-config-typescript": "^11.0.2",
    "@vue/eslint-config-prettier": "^7.1.0",
    "autoprefixer": "^10.4.14",
    "eslint": "^8.38.0",
    "eslint-plugin-vue": "^9.9.0",
    "postcss": "^8.4.24",
    "prettier": "^2.8.7",
    "tailwindcss": "^3.3.0",
    "typescript": "^5.0.0",
    "vite": "^4.3.0",
    "vue-tsc": "^1.4.0"
  }
}
```



**packages/main-app/vite.config.ts**

```json
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  server: {
    port: 3000,
    cors: true
  }
})
```



**packages/main-app/src/main.ts**

```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import microApp from '@micro-zoe/micro-app'

// 初始化微前端
microApp.start()

const app = createApp(App)
app.use(createPinia())
app.use(router)
app.use(ElementPlus)
app.mount('#app')
```



### 5. 微应用配置示例

**packages/micro-app-1/vite.config.ts**

```typescript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src')
    }
  },
  server: {
    port: 3001,
    cors: true,
    headers: {
      'Access-Control-Allow-Origin': '*'
    }
  },
  base: '/micro-app-1/'
})
```



**packages/micro-app-1/src/main.ts**

```typescript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import router from './router'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'

// 微前端环境判断
if (window.__MICRO_APP_ENVIRONMENT__) {
  // 微前端环境下动态设置路由base
  const base = window.__MICRO_APP_BASE_ROUTE__ || '/micro-app-1'
  router.base = base
}

const app = createApp(App)
app.use(createPinia())
app.use(router)
app.use(ElementPlus)
app.mount('#app')
```



## 代码规范配置

### 1. ESLint 与 Prettier 集成

为了确保 ESLint 和 Prettier 协同工作，我们需要安装以下插件：

```bash
# 在根目录执行
pnpm add -D eslint-plugin-prettier eslint-config-prettier
```



### 2. 编辑器配置

**.vscode/settings.json** (推荐在根目录创建)

```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact",
    "vue",
    "html"
  ],
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[vue]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```



### 3. Git Hooks 配置

**package.json** (在根目录添加)

```json
{
  "scripts": {
    "prepare": "husky install",
    "pre-commit": "lint-staged"
  },
  "devDependencies": {
    "husky": "^8.0.0",
    "lint-staged": "^13.0.0"
  },
  "lint-staged": {
    "*.{js,ts,vue}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```



安装并设置 Git Hooks:

```bash
# 安装 husky
pnpm add -D husky lint-staged

# 初始化 husky
pnpm run prepare

# 添加 pre-commit hook
npx husky add .husky/pre-commit "pnpm run pre-commit"
```



## 微前端配置

### 主应用路由配置

```typescript
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue')
  },
  {
    path: '/micro-app-1/*',
    name: 'MicroApp1',
    component: () => import('@/views/MicroAppContainer.vue')
  },
  {
    path: '/micro-app-2/*',
    name: 'MicroApp2',
    component: () => import('@/views/MicroAppContainer.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

export default router
```



### 微前端容器组件

```vue
<template>
  <div id="micro-app-container">
    <micro-app
      :name="appName"
      :url="appUrl"
      :baseroute="baseRoute"
      @created="handleCreate"
      @beforemount="handleBeforeMount"
      @mounted="handleMounted"
      @unmount="handleUnmount"
      @error="handleError"
    ></micro-app>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

const appName = computed(() => {
  const path = route.path.split('/')[1]
  return path
})

const appUrl = computed(() => {
  return `http://localhost:3001/${appName.value}/`
})

const baseRoute = computed(() => {
  return `/${appName.value}`
})

const handleCreate = () => {
  console.log('微应用创建了')
}

const handleBeforeMount = () => {
  console.log('微应用即将渲染')
}

const handleMounted = () => {
  console.log('微应用已经渲染完成')
}

const handleUnmount = () => {
  console.log('微应用已经卸载')
}

const handleError = () => {
  console.log('微应用加载出错')
}
</script>
```



## Tailwind CSS 配置

**tailwind.config.js**

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```



**postcss.config.js**

```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```



## TypeScript 配置

**tsconfig.json**

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
    "jsx": "preserve",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```



## 开发脚本说明

### 安装依赖

```bash
# 安装根目录依赖
pnpm install

# 安装所有包的依赖
lerna bootstrap
```



### 开发命令

```bash
# 启动所有应用
pnpm run dev

# 启动特定应用
lerna run dev --scope=main-app
lerna run dev --scope=micro-app-1

# 检查代码规范
pnpm run lint

# 自动修复代码规范问题
pnpm run lint:fix

# 格式化代码
pnpm run format

# 构建所有应用
pnpm run build

# 清理所有依赖
pnpm run clean
```



## 部署策略

1. **独立部署**: 每个微应用可以独立构建和部署
2. **主应用部署**: 主应用负责集成各个微应用
3. **环境隔离**: 开发、测试、生产环境使用不同的配置
4. **CDN部署**: 静态资源部署到CDN加速访问
5. **代码检查**: 在CI/CD流程中加入代码规范检查

## 注意事项

1. **样式隔离**: 微前端应用间使用CSS Modules或Scoped CSS避免样式冲突
2. **状态管理**: 使用Pinia进行状态管理，避免全局状态污染
3. **依赖共享**: 通过webpack externals或import-map共享公共依赖
4. **通信机制**: 使用customEvent或redux进行微应用间通信
5. **错误边界**: 实现错误边界处理微应用加载失败的情况
6. **代码规范**: 确保所有开发人员配置相同的编辑器设置和Git Hooks