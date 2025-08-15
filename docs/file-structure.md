# 项目文件结构说明

## 整体结构

```
qinglan-share-website/
├── public/                          # 静态资源目录
│   ├── index.html                   # 主HTML模板
│   ├── favicon.ico                  # 网站图标
│   └── lyrics/                      # LRC歌词文件目录
│       ├── wo-ji-de.lrc            # 我记得 - 歌词文件
│       ├── cheng-ai-ying.lrc       # 程艾影 - 歌词文件
│       ├── jue-bie-shu.lrc         # 诀别书 - 歌词文件
│       ├── hua-ri.lrc              # 花日 - 歌词文件
│       ├── life-time.lrc           # Life Time - 歌词文件
│       ├── don.lrc                 # どん - 歌词文件
│       └── april-front.lrc         # エイプリル・フロント - 歌词文件
├── src/                             # 源代码目录
│   ├── assets/                      # 资源文件
│   │   └── music-organized/         # 重组后的音乐文件
│   │       ├── wo-ji-de.mp3        # 音频文件
│   │       ├── wo-ji-de.jpg        # 封面图片
│   │       ├── cheng-ai-ying.mp3   # 音频文件
│   │       ├── cheng-ai-ying.jpg   # 封面图片
│   │       └── ...                 # 其他音乐文件
│   ├── components/                  # Vue组件
│   │   ├── common/                  # 通用组件
│   │   │   └── AdvancedMusicPlayer.vue  # 高级音乐播放器组件
│   │   └── layout/                  # 布局组件
│   │       └── MainLayout.vue       # 主布局组件
│   ├── router/                      # 路由配置
│   │   └── index.js                # 路由定义
│   ├── views/                       # 页面组件
│   │   └── Home.vue                # 首页组件
│   ├── App.vue                      # 根组件
│   └── main.js                      # 应用入口文件
├── docs/                            # 项目文档
│   ├── README.md                    # 文档总览
│   ├── development-process.md       # 开发过程记录
│   ├── music-player-features.md     # 播放器功能说明
│   ├── technical-implementation.md  # 技术实现文档
│   ├── deployment-guide.md          # 部署指南
│   └── file-structure.md           # 文件结构说明（本文件）
├── package.json                     # 项目依赖配置
├── package-lock.json               # 依赖锁定文件
├── vue.config.js                   # Vue CLI配置
├── babel.config.js                 # Babel配置
├── .gitignore                      # Git忽略文件
└── README.md                       # 项目说明文件
```

## 核心文件详解

### 1. 主要组件文件

#### 1.1 AdvancedMusicPlayer.vue
**路径**: `src/components/common/AdvancedMusicPlayer.vue`
**功能**: 核心音乐播放器组件
**结构**:
```vue
<template>
  <!-- 播放器UI模板 -->
  <div class="advanced-music-player">
    <!-- 最小化播放器 -->
    <div v-if="isMinimized" class="minimized-player">
      <!-- 基础控制界面 -->
    </div>
    
    <!-- 完整播放器 -->
    <div v-else class="player-warp">
      <!-- 播放列表面板 -->
      <div v-if="showPlaylistModal" class="playlist-panel">
        <!-- 播放列表内容 -->
      </div>
      
      <!-- 歌曲信息卡片 -->
      <div class="player-info">
        <!-- 歌曲信息和歌词显示 -->
      </div>
      
      <!-- 播放控制器 -->
      <div class="player-control">
        <!-- 详细控制界面 -->
      </div>
    </div>
  </div>
</template>

<script>
  // 组件逻辑
  export default {
    name: 'AdvancedMusicPlayer',
    data() {
      // 组件数据
    },
    computed: {
      // 计算属性
    },
    methods: {
      // 组件方法
    }
  }
</script>

<style scoped>
  /* 组件样式 */
</style>
```

#### 1.2 MainLayout.vue
**路径**: `src/components/layout/MainLayout.vue`
**功能**: 主布局组件，包含音乐播放器
**特点**:
- 响应式布局设计
- 集成音乐播放器组件
- 全局样式定义

### 2. 配置文件

#### 2.1 package.json
**功能**: 项目依赖和脚本配置
**主要依赖**:
```json
{
  "dependencies": {
    "vue": "^3.0.0",
    "vue-router": "^4.0.0",
    "element-plus": "^2.0.0"
  },
  "devDependencies": {
    "@vue/cli-service": "^5.0.0",
    "babel-loader": "^8.0.0"
  },
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
  }
}
```

#### 2.2 vue.config.js
**功能**: Vue CLI构建配置
```javascript
module.exports = {
  publicPath: process.env.NODE_ENV === 'production' ? '/' : '/',
  outputDir: 'dist',
  assetsDir: 'static',
  productionSourceMap: false,
  
  configureWebpack: {
    optimization: {
      splitChunks: {
        chunks: 'all'
      }
    }
  }
}
```

### 3. 资源文件组织

#### 3.1 音乐文件结构
**路径**: `src/assets/music-organized/`
**命名规范**:
- 音频文件: `{song-id}.mp3`
- 封面图片: `{song-id}.jpg`
- 歌词文件: `{song-id}.lrc` (存放在public/lyrics/)

**示例**:
```
wo-ji-de.mp3     # 我记得 - 音频文件
wo-ji-de.jpg     # 我记得 - 封面图片
wo-ji-de.lrc     # 我记得 - 歌词文件（在public/lyrics/目录）
```

#### 3.2 歌词文件结构
**路径**: `public/lyrics/`
**格式**: LRC标准格式
**示例内容**:
```
[00:00.00] 作词 : 赵雷
[00:01.00] 作曲 : 赵雷
[00:28.15]我带着比身体重的行李 游入尼罗河底
[00:41.78]我看到几个人站在一起
```

## 数据流向

### 1. 音乐数据流
```
音乐列表配置 (musicList)
    ↓
当前歌曲选择 (currentIndex)
    ↓
音频文件加载 (audio_url)
    ↓
播放控制 (play/pause)
    ↓
进度更新 (currentTime)
    ↓
歌词同步显示 (currentLyric)
```

### 2. 歌词数据流
```
LRC文件路径 (lrcFile)
    ↓
网络请求加载 (fetch)
    ↓
LRC文本解析 (parseLrcText)
    ↓
时间轴数组 (parsedLyrics)
    ↓
二分查找匹配 (binarySearchLyric)
    ↓
当前歌词显示 (currentLyric)
```

### 3. 用户交互流
```
用户操作 (点击/拖拽)
    ↓
事件处理 (methods)
    ↓
状态更新 (data)
    ↓
视图重渲染 (template)
    ↓
样式变化 (CSS)
```

## 构建输出结构

### 1. 开发环境
```
开发服务器 (localhost:8080)
├── /                    # 主页面
├── /static/             # 静态资源
├── /lyrics/             # 歌词文件
└── /src/assets/         # 音乐文件
```

### 2. 生产环境
```
dist/                    # 构建输出目录
├── index.html          # 主页面
├── static/             # 静态资源
│   ├── css/           # 样式文件
│   ├── js/            # JavaScript文件
│   ├── img/           # 图片文件
│   └── media/         # 音频文件
└── lyrics/            # 歌词文件（需手动复制）
```

## 开发工作流

### 1. 本地开发
```bash
# 启动开发服务器
npm run serve

# 文件监听和热重载
# 修改源文件 → 自动编译 → 浏览器刷新
```

### 2. 生产构建
```bash
# 构建生产版本
npm run build

# 输出优化
# 代码压缩 → 资源优化 → 生成dist目录
```

### 3. 部署流程
```bash
# 上传构建文件
scp -r dist/* server:/var/www/html/

# 上传歌词文件
scp -r public/lyrics server:/var/www/html/

# 配置服务器
# Nginx配置 → SSL证书 → 域名解析
```

## 文件命名规范

### 1. 组件文件
- **格式**: PascalCase
- **示例**: `AdvancedMusicPlayer.vue`, `MainLayout.vue`

### 2. 音乐文件
- **格式**: kebab-case
- **示例**: `wo-ji-de.mp3`, `cheng-ai-ying.jpg`

### 3. 配置文件
- **格式**: kebab-case 或 camelCase
- **示例**: `vue.config.js`, `babel.config.js`

### 4. 文档文件
- **格式**: kebab-case
- **示例**: `development-process.md`, `file-structure.md`

## 版本控制

### 1. Git忽略文件
```gitignore
# 依赖目录
node_modules/

# 构建输出
dist/

# 环境配置
.env.local
.env.*.local

# 编辑器配置
.vscode/
.idea/

# 系统文件
.DS_Store
Thumbs.db
```

### 2. 分支策略
- **main**: 主分支，稳定版本
- **develop**: 开发分支，功能集成
- **feature/***: 功能分支，新功能开发

## 性能考虑

### 1. 文件大小优化
- **音频文件**: 适当压缩，保持音质
- **图片文件**: 压缩优化，使用WebP格式
- **代码文件**: 生产环境自动压缩

### 2. 加载策略
- **懒加载**: 歌词文件按需加载
- **预加载**: 下一首歌曲预加载
- **缓存**: 合理的缓存策略

---

*本文档详细说明了项目的文件组织结构和各文件的作用，便于理解和维护项目。*
