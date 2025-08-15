# 快速开始指南

## 5分钟快速体验

### 1. 环境准备
确保您的系统已安装：
- Node.js 14.0.0+
- npm 6.0.0+

### 2. 项目启动
```bash
# 克隆项目
git clone <repository-url>
cd qinglan-share-website

# 安装依赖
npm install

# 启动开发服务器
npm run serve
```

### 3. 访问应用
打开浏览器访问：`http://localhost:8080`

## 核心功能体验

### 🎵 音乐播放器
1. **基础播放**：点击播放按钮开始播放音乐
2. **模式切换**：点击播放器展开完整模式
3. **歌曲切换**：使用上一曲/下一曲按钮
4. **进度控制**：拖拽进度条跳转播放位置

### 🎤 歌词同步
1. **实时显示**：歌词随音乐播放实时更新
2. **精确同步**：毫秒级精度的时间匹配
3. **智能过滤**：自动过滤制作信息

### 📋 播放列表
1. **打开列表**：点击播放列表按钮
2. **选择歌曲**：点击任意歌曲直接播放
3. **当前标识**：正在播放的歌曲会高亮显示

## 自定义配置

### 添加新音乐
1. **准备文件**：
   - 音频文件：`song-name.mp3`
   - 封面图片：`song-name.jpg`
   - 歌词文件：`song-name.lrc`

2. **放置文件**：
   ```
   src/assets/music-organized/song-name.mp3
   src/assets/music-organized/song-name.jpg
   public/lyrics/song-name.lrc
   ```

3. **更新配置**：
   ```javascript
   // 在 AdvancedMusicPlayer.vue 的 musicList 中添加
   {
     name: "歌曲名称",
     audio_url: require("@/assets/music-organized/song-name.mp3"),
     singer: "歌手名",
     album: "专辑名",
     cover: require("@/assets/music-organized/song-name.jpg"),
     time: "03:45",
     lrcFile: "song-name.lrc"
   }
   ```

### LRC歌词格式
```
[00:00.00] 作词 : 作词人
[00:01.00] 作曲 : 作曲人
[00:15.30]第一行歌词
[00:30.60]第二行歌词
[00:45.90]第三行歌词
```

## 常见问题

### Q: 音乐文件无法播放？
**A**: 检查文件路径和格式：
- 确保音频文件在正确位置
- 支持的格式：MP3, WAV, OGG
- 检查文件权限

### Q: 歌词不显示？
**A**: 检查歌词文件：
- 确保LRC文件在 `public/lyrics/` 目录
- 检查LRC格式是否正确
- 查看浏览器控制台错误信息

### Q: 播放列表为空？
**A**: 检查音乐配置：
- 确保 `musicList` 数组不为空
- 检查音频文件路径是否正确
- 验证require语法

## 开发调试

### 启用调试模式
在浏览器开发者工具中查看详细日志：
1. 打开 F12 开发者工具
2. 切换到 Console 标签页
3. 播放音乐查看日志输出

### 常用调试命令
```javascript
// 在浏览器控制台中执行

// 查看当前播放状态
console.log(this.$refs.musicPlayer.isPlaying)

// 查看当前歌词数据
console.log(this.$refs.musicPlayer.parsedLyrics)

// 测试歌词文件加载
fetch('/lyrics/wo-ji-de.lrc').then(r => r.text()).then(console.log)
```

## 性能优化建议

### 1. 音频文件优化
- **文件大小**：建议单个文件 < 10MB
- **音质设置**：128-320 kbps
- **格式选择**：优先使用 MP3 格式

### 2. 图片优化
- **封面尺寸**：建议 300x300 像素
- **文件格式**：JPEG 或 WebP
- **压缩质量**：80-90%

### 3. 歌词文件
- **编码格式**：UTF-8
- **文件大小**：通常 < 10KB
- **时间精度**：建议精确到 0.1 秒

## 部署预览

### 本地构建测试
```bash
# 构建生产版本
npm run build

# 使用简单HTTP服务器预览
npx serve dist

# 访问 http://localhost:5000
```

### 部署检查清单
- [ ] 音频文件路径正确
- [ ] 歌词文件可访问
- [ ] 图片资源加载正常
- [ ] 响应式设计测试
- [ ] 浏览器兼容性测试

## 下一步

### 深入了解
- 阅读 [技术实现文档](./technical-implementation.md) 了解核心算法
- 查看 [开发过程记录](./development-process.md) 了解开发历程
- 参考 [部署指南](./deployment-guide.md) 进行生产部署

### 功能扩展
- 添加更多音乐文件
- 自定义播放器主题
- 集成在线音乐API
- 添加播放历史功能

### 社区参与
- 提交问题和建议
- 贡献代码改进
- 分享使用经验

---

*本指南帮助您快速上手项目，如有问题请查看详细文档或联系开发团队。*
