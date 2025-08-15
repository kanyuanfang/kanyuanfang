<template>
  <div class="advanced-music-player" :class="{ minimized: isMinimized }">
    <!-- 欢迎提示 -->
    <div v-if="showWelcomePrompt" class="welcome-prompt" @click="handleWelcomeBackgroundClick">
      <div class="welcome-content" @click.stop>
        <div class="welcome-icon">🎵</div>
        <h3>“锦水汤汤，与君长绝。” </h3>
        <p>选择你的浏览方式</p>
        <div class="welcome-options">
          <el-button type="primary" @click="handleMusicMode" class="welcome-btn music-btn">
            <el-icon><VideoPlay /></el-icon>
            聆听音乐
            <span class="btn-subtitle">开启《诀别书》，感受古韵悠长</span>
          </el-button>
          <el-button @click="handleSilentMode" class="welcome-btn silent-btn">
            <el-icon><Mute /></el-icon>
            静音浏览
            <span class="btn-subtitle">安静地探索网站内容</span>
          </el-button>
        </div>
        <div class="welcome-note">
          <p>💡 你可以随时在右下角控制音乐播放</p>
        </div>
      </div>
    </div>
    <!-- 最小化时的简单控制条 -->
    <div v-if="isMinimized" class="minimized-player" @click="toggleMinimize">
      <div class="mini-info">
        <img :src="currentSong.cover" alt="封面" class="mini-cover" />
        <div class="mini-text">
          <div class="mini-name">{{ currentSong.name }}</div>
          <div class="mini-singer">{{ currentSong.singer }}</div>
        </div>
      </div>
      <div class="mini-controls">
        <el-icon @click.stop="togglePlay" class="mini-play-btn">
          <VideoPlay v-if="!isPlaying" />
          <VideoPause v-else />
        </el-icon>
        <el-icon @click.stop="nextSong" class="mini-next-btn">
          <DArrowRight />
        </el-icon>
      </div>
    </div>

    <!-- 完整播放器 -->
    <div v-else class="player-warp">
      

      <!-- 歌曲信息卡片 -->
      <div class="player-info" :class="{ show: isPlaying }">
        <div class="info">
          <div class="name">{{ currentSong.name }}</div>
          <div class="singer-album">{{ currentSong.singer }} - {{ currentSong.album }}</div>
          <!-- 歌词显示 -->
          <div class="lyrics-container" v-if="parsedLyrics.length > 0">
            <div class="current-lyric">{{ currentLyric }}</div>
          </div>
        </div>
      </div>

      <!-- 播放列表面板 -->
      <div v-if="showPlaylistModal" class="playlist-panel" :class="{ 'panel-show': showPlaylistModal }">
        <div class="playlist-header">
          <h3>播放列表</h3>
          <el-icon @click="showPlaylistModal = false" class="close-btn">
            <Close />
          </el-icon>
        </div>
        <div class="playlist-content">
          <div class="local-playlist">
            <div
              v-for="(song, index) in musicList"
              :key="index"
              class="song-item"
              :class="{ active: index === currentIndex }"
              @click="playSong(index)"
            >
              <img :src="song.cover" alt="封面" class="song-cover" />
              <div class="song-info">
                <div class="song-name">{{ song.name }}</div>
                <div class="song-artist">{{ song.singer }} - {{ song.album }}</div>
              </div>
              <div class="song-duration">{{ song.time }}</div>
              <el-icon class="play-icon" :class="{ loading: loadingSongIndex === index }">
                <VideoPause v-if="index === currentIndex && isPlaying" />
                <VideoPlay v-else-if="loadingSongIndex !== index" />
                <span v-else class="loading-spinner">⟳</span>
              </el-icon>
            </div>
          </div>
        </div>
      </div>

      <!-- 音乐控制器 -->
      <div class="player-control">
        <!-- 封面唱片 -->
        <div class="cover" :class="{ rotating: isPlaying, paused: !isPlaying }" @click="toggleMinimize">
          <img :src="currentSong.cover" alt="封面" />
        </div>
        <!-- 右侧控制区域 -->
        <div class="control-area">
          <!-- 进度条 -->
          <div class="music_progress">
            <div class="music_progress_top">
              <span class="current-time">{{ formatTime(currentTime) }}</span>
              <span class="time">{{ formatTime(duration) }}</span>
            </div>
            <div class="music_progress_bar" @click.stop="seekTo">
              <div class="music_progress_line" :style="{ width: progressPercent + '%' }"></div>
            </div>
          </div>
          <!-- 控制按钮 -->
          <div class="control">
            <!-- <el-icon @click.stop="toggleRandomMode" class="control-btn random-btn" :class="{ active: isRandomMode }">
              <span class="random-icon">🔀</span>
            </el-icon> -->
            <el-icon @click.stop="prevSong" class="control-btn">
              <DArrowLeft />
            </el-icon>
            <el-icon @click.stop="togglePlay" class="control-btn play-btn">
              <VideoPlay v-if="!isPlaying" />
              <VideoPause v-else />
            </el-icon>
            <el-icon @click.stop="nextSong" class="control-btn">
              <DArrowRight />
            </el-icon>
            <el-icon @click.stop="showPlaylist" class="control-btn">
              <List />
            </el-icon>
          </div>
        </div>
      </div>

      <!-- 背景 -->
      <!-- <div class="mask_bg" :style="{ backgroundImage: `url(${currentSong.cover})` }"></div> -->
    </div>



    <!-- 音频元素 -->
    <audio
      ref="audioPlayer"
      @timeupdate="updateProgress"
      @ended="onSongEnd"
      @loadedmetadata="onMetadataLoaded"
      preload="metadata"
    ></audio>
  </div>
</template>

<script>
import { VideoPlay, VideoPause, DArrowLeft, DArrowRight, List, Close, Mute } from '@element-plus/icons-vue'

export default {
  name: 'AdvancedMusicPlayer',
  components: {
    VideoPlay,
    VideoPause,
    DArrowLeft,
    DArrowRight,
    List,
    Close,
    Mute
  },
  data() {
    return {
      isMinimized: true,
      isPlaying: false,
      currentIndex: 0,
      currentTime: 0,
      duration: 0,
      showPlaylistModal: false,
      activeTab: 'local',
      selectedPlaylist: '514947114',
      showWelcomePrompt: true,
      hasUserInteracted: false,
      currentLyricIndex: 0,
      loadingSongIndex: -1, // 正在加载的歌曲索引
      // 歌词同步相关
      parsedLyrics: [], // 解析后的歌词数组
      currentLyricLine: 0, // 当前歌词行
      // 随机播放相关
      isRandomMode: true, // 默认开启随机播放
      playedSongs: [], // 已播放的歌曲索引记录
      isFirstPlay: true, // 是否是首次播放
      neteasePlaylists: [
        { id: '514947114', name: '默认歌单' },
        { id: '2884035', name: '华语流行' },
        { id: '19723756', name: '轻音乐' },
        { id: '3779629', name: '民谣' },
        { id: '2250011882', name: '古风' },
        { id: '5059642708', name: '电子音乐' }
      ],
      musicList: [
        {
          name: "诀别书",
          audio_url: require("@/assets/music-organized/jue-bie-shu.mp3"),
          singer: "邓垚",
          album: "诀别书",
          cover: require("@/assets/music-organized/jue-bie-shu.jpg"),
          time: "05:00",
          lrcFile: "jue-bie-shu.lrc"
        },
        {
          name: "我记得",
          audio_url: require("@/assets/music-organized/wo-ji-de.mp3"),
          singer: "赵雷",
          album: "署前街少年",
          cover: require("@/assets/music-organized/wo-ji-de.jpg"),
          time: "05:29",
          lrcFile: "wo-ji-de.lrc"
        },
        {
          name: "程艾影",
          audio_url: require("@/assets/music-organized/cheng-ai-ying.mp3"),
          singer: "赵雷",
          album: "程艾影",
          cover: require("@/assets/music-organized/cheng-ai-ying.jpg"),
          time: "05:00",
          lrcFile: "cheng-ai-ying.lrc"
        },
        {
          name: "花日（手风琴版）",
          audio_url: require("@/assets/music-organized/hua-ri.mp3"),
          singer: "王耳德",
          album: "花日",
          cover: require("@/assets/music-organized/hua-ri.jpg"),
          time: "04:30",
          lrcFile: "hua-ri.lrc"
        },
        {
          name: "Life Time",
          audio_url: require("@/assets/music-organized/life-time.mp3"),
          singer: "William King",
          album: "Acoustic Guitar",
          cover: require("@/assets/music-organized/life-time.jpg"),
          time: "03:30",
          lrcFile: "life-time.lrc"
        },
        {
          name: "どん",
          audio_url: require("@/assets/music-organized/don.mp3"),
          singer: "秋山羊子",
          album: "どん",
          cover: require("@/assets/music-organized/don.jpg"),
          time: "04:00",
          lrcFile: "don.lrc"
        },
        {
          name: "エイプリル・フロント",
          audio_url: require("@/assets/music-organized/april-front.mp3"),
          singer: "松たか子",
          album: "エイプリル・フロント",
          cover: require("@/assets/music-organized/april-front.jpg"),
          time: "04:15",
          lrcFile: "april-front.lrc"
        }
      ]
    }
  },
  computed: {
    currentSong() {
      return this.musicList[this.currentIndex] || this.musicList[0]
    },
    progressPercent() {
      return this.duration > 0 ? (this.currentTime / this.duration) * 100 : 0
    },
    neteasePlayerUrl() {
      return `//music.163.com/outchain/player?type=0&id=${this.selectedPlaylist}&auto=0&height=200`
    },
    currentLyric() {
      // 如果没有解析到歌词，根据歌曲类型显示不同的默认文本
      if (!this.parsedLyrics || this.parsedLyrics.length === 0) {
        const song = this.currentSong
        // 检查是否是纯音乐
        if (song.name.includes('纯音乐') || song.name.includes('Instrumental') ||
            song.name.includes('Life Time') || song.name.includes('花日') ||
            song.name.includes('诀别书')) {
          return '♪ 纯音乐，请欣赏 ♪'
        }
        return '♪ 享受音乐 ♪'
      }

      // 使用二分查找找到当前时间对应的歌词
      const index = this.binarySearchLyric(this.parsedLyrics, this.currentTime)
      const lyricItem = this.parsedLyrics[index]

      if (!lyricItem) {
        return '♪ 享受音乐 ♪'
      }

      // 如果歌词为空（间奏），显示音乐符号
      let lyric = lyricItem.text || '♪ ♪ ♪'

      // 如果歌词包含"纯音乐"，添加音乐符号装饰
      if (lyric.includes('纯音乐') || lyric.includes('请欣赏') || lyric.includes('请静心聆听')) {
        lyric = `♪ ${lyric} ♪`
      }

      // 调试信息（只在开发环境显示，且每5秒输出一次）
      if (process.env.NODE_ENV === 'development' &&
          this.currentTime > 0 &&
          Math.floor(this.currentTime) % 5 === 0 &&
          this.currentTime % 1 < 0.1) {
        console.log(`🎵 当前时间: ${this.currentTime.toFixed(1)}s, 歌词索引: ${index}/${this.parsedLyrics.length}, 歌词: "${lyric}"`)
      }

      return lyric
    }
  },
  mounted() {
    // 从本地存储恢复状态
    const savedIndex = localStorage.getItem('currentSongIndex')
    const savedPlaylist = localStorage.getItem('selectedPlaylist')
    const savedTab = localStorage.getItem('activeTab')
    const hasInteracted = localStorage.getItem('musicPlayerInteracted')
    const userPreference = localStorage.getItem('userPreference')

    // 检查用户是否之前有过选择
    if (hasInteracted && hasInteracted !== 'false') {
      this.showWelcomePrompt = false

      if (hasInteracted === 'true' || userPreference === 'music') {
        this.hasUserInteracted = true
        console.log('🎵 恢复音乐模式')
      } else if (hasInteracted === 'silent' || userPreference === 'silent') {
        this.hasUserInteracted = false
        this.isMinimized = true
        console.log('🔇 恢复静音模式')
      }
    } else {
      // 首次访问，显示欢迎页面
      this.showWelcomePrompt = true
      this.hasUserInteracted = false
      console.log('👋 首次访问，显示欢迎页面')
    }

    // 恢复播放状态
    if (savedIndex) {
      this.currentIndex = parseInt(savedIndex)
      console.log(`🎵 恢复上次播放: ${this.musicList[this.currentIndex]?.name}`)
    } else {
      this.currentIndex = 0 // 默认诀别书
    }

    if (savedPlaylist) this.selectedPlaylist = savedPlaylist
    if (savedTab) this.activeTab = savedTab

    this.loadCurrentSong()

    // 添加用户交互监听器
    this.addAutoPlayListener()
  },
  methods: {
    toggleMinimize() {
      this.isMinimized = !this.isMinimized
    },
    togglePlay() {
      if (this.isPlaying) {
        this.$refs.audioPlayer.pause()
        this.isPlaying = false
        return Promise.resolve()
      } else {
        const playPromise = this.$refs.audioPlayer.play()
        if (playPromise !== undefined) {
          return playPromise.then(() => {
            this.isPlaying = true
          }).catch((error) => {
            console.log('播放失败:', error)
            this.isPlaying = false
            throw error
          })
        } else {
          this.isPlaying = true
          return Promise.resolve()
        }
      }
    },
    prevSong() {
      this.currentIndex = this.currentIndex > 0 ? this.currentIndex - 1 : this.musicList.length - 1
      this.loadCurrentSong()
      if (this.isPlaying) {
        this.$nextTick(() => {
          this.$refs.audioPlayer.play().catch(() => {
            this.isPlaying = false
          })
        })
      }
    },
    nextSong() {
      if (this.isRandomMode && !this.isFirstPlay) {
        this.playRandomSong()
      } else {
        this.currentIndex = this.currentIndex < this.musicList.length - 1 ? this.currentIndex + 1 : 0
      }

      this.isFirstPlay = false
      this.loadCurrentSong()

      if (this.isPlaying) {
        this.$nextTick(() => {
          this.$refs.audioPlayer.play().catch(() => {
            this.isPlaying = false
          })
        })
      }
    },
    playSong(index) {
      // 如果点击的是当前正在播放的歌曲，则切换播放/暂停状态
      if (index === this.currentIndex) {
        this.togglePlay().catch(() => {
          console.log('播放切换失败')
        })
        return
      }

      // 设置加载状态
      this.loadingSongIndex = index

      // 切换到新歌曲
      this.currentIndex = index
      this.loadCurrentSong()

      // 确保音频加载完成后开始播放
      this.$nextTick(() => {
        const audio = this.$refs.audioPlayer

        // 监听音频可以播放事件
        const playWhenReady = () => {
          audio.play().then(() => {
            this.isPlaying = true
            this.loadingSongIndex = -1 // 清除加载状态
            // 关闭播放列表面板
            // this.showPlaylistModal = false
          }).catch((error) => {
            console.log('播放失败:', error)
            this.isPlaying = false
            this.loadingSongIndex = -1 // 清除加载状态
          })
          audio.removeEventListener('canplay', playWhenReady)
        }

        // 监听加载错误
        const handleError = () => {
          console.log('音频加载失败')
          this.loadingSongIndex = -1 // 清除加载状态
          audio.removeEventListener('error', handleError)
        }

        audio.addEventListener('error', handleError)

        // 如果音频已经可以播放，直接播放
        if (audio.readyState >= 3) {
          playWhenReady()
        } else {
          // 否则等待音频加载完成
          audio.addEventListener('canplay', playWhenReady)
        }
      })
    },
    loadCurrentSong() {
      const song = this.currentSong
      this.$refs.audioPlayer.src = song.audio_url
      localStorage.setItem('currentSongIndex', this.currentIndex.toString())

      // 加载歌词
      this.loadLyrics(song.lrcFile)
    },
    showPlaylist() {
      this.showPlaylistModal = this.showPlaylistModal ? false : true
    },
    handleTabChange(tab) {
      this.activeTab = tab
      localStorage.setItem('activeTab', tab)
    },
    loadNeteasePlaylist() {
      localStorage.setItem('selectedPlaylist', this.selectedPlaylist)
    },
    updateProgress() {
      this.currentTime = this.$refs.audioPlayer.currentTime
    },
    onMetadataLoaded() {
      this.duration = this.$refs.audioPlayer.duration
    },
    onSongEnd() {
      this.isPlaying = false
      // 记录当前歌曲已播放
      if (!this.playedSongs.includes(this.currentIndex)) {
        this.playedSongs.push(this.currentIndex)
      }

      // 自动播放下一首
      this.autoPlayNext()
    },
    seekTo(event) {
      const progressBar = event.currentTarget
      const clickX = event.offsetX
      const width = progressBar.offsetWidth
      const newTime = (clickX / width) * this.duration
      this.$refs.audioPlayer.currentTime = newTime
    },
    formatTime(time) {
      if (!time || isNaN(time)) return '00:00'
      const minutes = Math.floor(time / 60)
      const seconds = Math.floor(time % 60)
      return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
    },
    // 处理音乐模式选择
    handleMusicMode() {
      this.showWelcomePrompt = false
      this.hasUserInteracted = true
      localStorage.setItem('musicPlayerInteracted', 'true')
      localStorage.setItem('userPreference', 'music')
      console.log('🎵 用户选择音乐模式')

      // 自动开始播放第一首歌（诀别书）
      this.startAutoPlay()
    },

    // 处理静音模式选择
    handleSilentMode() {
      this.showWelcomePrompt = false
      this.hasUserInteracted = false // 保持为false，不启用自动播放
      this.isPlaying = false
      localStorage.setItem('musicPlayerInteracted', 'silent')
      localStorage.setItem('userPreference', 'silent')
      console.log('🔇 用户选择静音模式')

      // 最小化播放器
      this.isMinimized = true
    },

    // 处理欢迎页面背景点击 - 进入静音模式
    handleWelcomeBackgroundClick() {
      console.log('🔇 点击欢迎页面背景，进入静音模式')
      this.handleSilentMode()
    },

    // 兼容旧的方法名（如果其他地方还在使用）
    handleWelcomeClick() {
      this.handleMusicMode()
    },
    addAutoPlayListener() {
      // 监听用户的第一次交互
      const startAutoPlay = () => {
        // 检查用户偏好，如果是静音模式则不自动播放
        const userPreference = localStorage.getItem('userPreference')
        if (userPreference === 'silent') {
          console.log('🔇 用户选择了静音模式，跳过自动播放监听器')
          // 移除监听器
          document.removeEventListener('click', startAutoPlay)
          document.removeEventListener('keydown', startAutoPlay)
          document.removeEventListener('touchstart', startAutoPlay)
          return
        }

        if (!this.hasUserInteracted) {
          this.showWelcomePrompt = false
          this.hasUserInteracted = true
          localStorage.setItem('musicPlayerInteracted', 'true')

          // 尝试自动播放
          this.$nextTick(() => {
            setTimeout(() => {
              if (!this.isPlaying) {
                this.togglePlay().catch(() => {
                  // 如果自动播放失败，静默处理
                  console.log('自动播放被浏览器阻止，等待用户交互')
                })
              }
            }, 300)
          })

          // 移除监听器
          document.removeEventListener('click', startAutoPlay)
          document.removeEventListener('keydown', startAutoPlay)
          document.removeEventListener('touchstart', startAutoPlay)
        }
      }

      // 添加多种交互事件监听
      document.addEventListener('click', startAutoPlay, { once: true })
      document.addEventListener('keydown', startAutoPlay, { once: true })
      document.addEventListener('touchstart', startAutoPlay, { once: true })
    },

    // 加载歌词文件
    async loadLyrics(lrcFileName) {
      if (!lrcFileName) {
        console.log('没有歌词文件名，跳过加载')
        this.parsedLyrics = []
        return
      }

      console.log('开始加载歌词文件:', lrcFileName)

      try {
        // 尝试从public目录加载LRC文件
        const url = `/lyrics/${lrcFileName}`
        console.log('请求URL:', url)

        const response = await fetch(url)
        console.log('响应状态:', response.status, response.statusText)

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`)
        }

        const lrcText = await response.text()
        console.log('LRC文件加载成功:', lrcFileName)
        console.log('文件内容长度:', lrcText.length)

        // 检查文件是否为空或只有空白字符
        if (!lrcText.trim()) {
          console.warn('LRC文件为空:', lrcFileName)
          this.parsedLyrics = []
          return
        }

        console.log('文件内容预览:', lrcText.substring(0, 300))

        this.parsedLyrics = this.parseLrcText(lrcText)
        console.log('解析后的歌词数量:', this.parsedLyrics.length)

        if (this.parsedLyrics.length > 0) {
          console.log('前几行解析结果:', this.parsedLyrics.slice(0, 5))
        } else {
          console.warn('没有解析到有效歌词，可能是纯音乐或格式问题')
        }
      } catch (error) {
        console.error('加载LRC文件失败:', error)
        console.error('请确保LRC文件存在于public/lyrics/目录中')
        this.parsedLyrics = []
      }
    },

    // 解析LRC歌词文本
    parseLrcText(lrcText) {
      console.log('开始解析LRC歌词文本')
      const lines = lrcText.split('\n')
      const lyrics = []
      let lineCount = 0

      for (const line of lines) {
        lineCount++
        const trimmedLine = line.trim()
        if (!trimmedLine) continue

        // 更强健的LRC时间标签匹配，支持多种格式
        // [mm:ss.xx] [mm:ss.xxx] [mm:ss] [mm:ss.x] 等格式
        const match = trimmedLine.match(/^\[(\d{1,2}):(\d{2})(?:\.(\d{1,3}))?\](.*)$/)
        if (match) {
          const minutes = parseInt(match[1])
          const seconds = parseInt(match[2])
          let milliseconds = 0

          // 处理毫秒部分，支持1-3位数字
          if (match[3]) {
            const msStr = match[3].padEnd(3, '0') // 补齐到3位
            milliseconds = parseInt(msStr.substring(0, 3)) // 取前3位
          }

          const text = match[4].trim()

          // 计算时间（秒），毫秒转换为小数
          const time = minutes * 60 + seconds + milliseconds / 1000

          console.log(`第${lineCount}行: [${minutes}:${seconds}.${milliseconds}] "${text}"`)

          // 过滤掉制作信息，但保留所有实际歌词内容
          if (text &&
              !text.includes('作词') &&
              !text.includes('作曲') &&
              !text.includes('编曲') &&
              !text.includes('制作人') &&
              !text.includes('录音') &&
              !text.includes('混音') &&
              !text.includes('母带') &&
              !text.includes('封面设计') &&
              !text.includes('吉他') &&
              !text.includes('贝斯') &&
              !text.includes('鼓') &&
              !text.includes('键盘') &&
              !text.includes('电吉他') &&
              !text.includes('钢琴') &&
              !text.includes('打击乐') &&
              !text.includes('Organ') &&
              !text.includes('口琴') &&
              !text.includes('和声') &&
              !text.includes('录音师') &&
              !text.includes('混音师') &&
              !text.includes('录音室') &&
              !text.includes('录音助理') &&
              !text.includes('母带工程') &&
              !text.includes('工程师') &&
              !text.includes('Sterling Sound') &&
              !text.includes('Randy Merrill')) {

            lyrics.push({ time, text })
            console.log(`✓ 添加歌词: ${time.toFixed(3)}s - "${text}"`)
          } else if (text) {
            console.log(`✗ 跳过制作信息: "${text}"`)
          } else {
            // 空歌词行，可能是间奏
            lyrics.push({ time, text: '' })
            console.log(`✓ 添加空行: ${time.toFixed(3)}s`)
          }
        } else {
          // 尝试匹配其他可能的格式，如带有负数或其他特殊字符的时间标签
          const altMatch = trimmedLine.match(/^\[(\d{1,2}):(\d{2})(?:\.(\d{1,3}))?.*?\](.*)$/)
          if (altMatch) {
            console.log(`✗ 发现格式异常的时间标签，跳过第${lineCount}行: "${trimmedLine}"`)
          } else {
            console.log(`✗ 无法解析第${lineCount}行: "${trimmedLine}"`)
          }
        }
      }

      // 按时间排序
      lyrics.sort((a, b) => a.time - b.time)

      console.log(`解析完成！总共${lineCount}行，有效歌词${lyrics.length}行`)
      console.log('前5行歌词:', lyrics.slice(0, 5))

      return lyrics
    },

    // 二分查找歌词
    binarySearchLyric(lyrics, currentTime) {
      let left = 0
      let right = lyrics.length - 1
      let result = 0

      while (left <= right) {
        const mid = Math.floor((left + right) / 2)
        if (lyrics[mid].time <= currentTime) {
          result = mid
          left = mid + 1
        } else {
          right = mid - 1
        }
      }

      return result
    },

    // 随机播放歌曲
    playRandomSong() {
      // 如果所有歌曲都播放过了，重置播放记录（除了当前歌曲）
      if (this.playedSongs.length >= this.musicList.length - 1) {
        this.playedSongs = [this.currentIndex]
      }

      // 获取未播放的歌曲列表
      const unplayedSongs = []
      for (let i = 0; i < this.musicList.length; i++) {
        if (!this.playedSongs.includes(i)) {
          unplayedSongs.push(i)
        }
      }

      // 如果有未播放的歌曲，随机选择一首
      if (unplayedSongs.length > 0) {
        const randomIndex = Math.floor(Math.random() * unplayedSongs.length)
        this.currentIndex = unplayedSongs[randomIndex]
      } else {
        // 如果没有未播放的歌曲，随机选择一首（排除当前歌曲）
        let newIndex
        do {
          newIndex = Math.floor(Math.random() * this.musicList.length)
        } while (newIndex === this.currentIndex && this.musicList.length > 1)
        this.currentIndex = newIndex
      }

      console.log(`🎵 随机播放: ${this.musicList[this.currentIndex].name}`)
    },

    // 自动播放下一首
    autoPlayNext() {
      console.log('🎵 歌曲播放完毕，自动播放下一首')
      this.nextSong()

      // 延迟一点时间后自动开始播放
      this.$nextTick(() => {
        setTimeout(() => {
          this.togglePlay().catch(() => {
            console.log('自动播放失败')
          })
        }, 500)
      })
    },

    // 开始自动播放
    startAutoPlay() {
      // 检查用户偏好，如果是静音模式则不自动播放
      const userPreference = localStorage.getItem('userPreference')
      if (userPreference === 'silent') {
        console.log('🔇 用户选择了静音模式，跳过自动播放')
        return
      }

      console.log('🎵 开始自动播放')
      this.$nextTick(() => {
        setTimeout(() => {
          this.togglePlay().catch(() => {
            console.log('自动播放失败，等待用户交互')
          })
        }, 300)
      })
    },

    // 切换随机播放模式
    toggleRandomMode() {
      this.isRandomMode = !this.isRandomMode
      console.log(`🎵 随机播放模式: ${this.isRandomMode ? '开启' : '关闭'}`)

      // 重置播放记录
      if (this.isRandomMode) {
        this.playedSongs = [this.currentIndex]
      }
    },

    // 重置播放器状态（用于测试）
    resetPlayerState() {
      localStorage.removeItem('isFirstVisit')
      localStorage.removeItem('currentSongIndex')
      localStorage.removeItem('musicPlayerInteracted')
      localStorage.removeItem('userPreference')
      console.log('🎵 播放器状态已重置，刷新页面将显示欢迎页面')

      // 立即重置当前状态
      this.showWelcomePrompt = true
      this.hasUserInteracted = false
      this.isMinimized = true
    }
  }
}
</script>

<style scoped>
.advanced-music-player {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000;
  user-select: none;
}

/* 欢迎提示 */
.welcome-prompt {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, rgba(139, 69, 19, 0.95) 0%, rgba(160, 82, 45, 0.95) 50%, rgba(205, 133, 63, 0.95) 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  cursor: pointer;
  animation: fadeIn 0.5s ease;
}

.welcome-content {
  text-align: center;
  color: white;
  max-width: 400px;
  padding: 2rem;
}

.welcome-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  animation: bounce 2s infinite;
}

.welcome-content h3 {
  font-size: 2rem;
  margin-bottom: 1rem;
  font-weight: 600;
}

.welcome-content p {
  font-size: 1.1rem;
  margin-bottom: 2rem;
  opacity: 0.9;
}

.welcome-options {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-top: 1.5rem;
  width: 100%;
}

.welcome-btn {
  padding: 1rem 2rem;
  font-size: 1rem;
  border-radius: 15px;
  transition: all 0.3s ease;
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  min-height: 80px;
  justify-content: center;
  border: none;
}

.welcome-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 25px rgba(0, 0, 0, 0.15);
}

.music-btn {
  background: linear-gradient(135deg, #4682B4, #87CEEB);
  color: white;
}

.music-btn:hover {
  background: linear-gradient(135deg, #5a9bd4, #a0d8ef);
}

.silent-btn {
  background: rgba(255, 255, 255, 0.9);
  border: 2px solid #ddd;
  color: #666;
}

.silent-btn:hover {
  background: rgba(248, 249, 250, 1);
  border-color: #bbb;
  color: #555;
}

.btn-subtitle {
  font-size: 0.8rem;
  opacity: 0.8;
  font-weight: normal;
  line-height: 1.3;
}

.welcome-note {
  margin-top: 1.5rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

.welcome-note p {
  font-size: 0.85rem;
  opacity: 0.7;
  margin: 0;
  color: #666;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-10px); }
  60% { transform: translateY(-5px); }
}

/* 最小化状态 */
.minimized-player {
  width: 300px;
  height: 60px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 30px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 16px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  /* transition: all 0.3s ease; */
}



.mini-info {
  display: flex;
  align-items: center;
  gap: 12px;
  flex: 1;
  min-width: 0;
}

.advanced-music-player .mini-cover {
  width: 44px !important;
  height: 44px !important;
  border-radius: 50% !important;
  object-fit: cover !important;
  /* 强制确保最小化封面也是完美圆形 */
  min-width: 44px !important;
  max-width: 44px !important;
  min-height: 44px !important;
  max-height: 44px !important;
  flex-shrink: 0 !important;
  display: block !important;
}

.mini-text {
  flex: 1;
  min-width: 0;
}

.mini-name {
  font-size: 14px;
  font-weight: 600;
  color: #333;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.mini-singer {
  font-size: 12px;
  color: #666;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.mini-controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

.mini-play-btn, .mini-next-btn {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: #8B4513;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
}

.mini-play-btn:hover, .mini-next-btn:hover {
  background: #A0522D;
  transform: scale(1.1);
}

/* 完整播放器 */
.player-warp {
  position: relative;
  width: 380px;
  cursor: pointer;
  /* transition: all 0.3s ease; */
}



.player-control {
  width: 100%;
  height: 90px;
  padding: 15px 25px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  /* transition: all 0.3s ease; */
  margin-top: -20px;
  position: relative;
  z-index: 2;
}



.advanced-music-player .cover {
  width: 110px !important;
  height: 110px !important;
  border-radius: 50% !important;
  background: white !important;
  margin-top: -65px !important;
  margin-left: -5px !important;
  padding: 6px !important;
  position: relative !important;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1) !important;
  /* 强制确保是完美圆形 */
  min-width: 110px !important;
  max-width: 110px !important;
  min-height: 110px !important;
  max-height: 110px !important;
  overflow: hidden !important;
  flex-shrink: 0 !important;
  display: flex !important;
  align-items: center !important;
  justify-content: center !important;
  z-index: 3 !important;
}

.advanced-music-player .cover::before {
  content: "";
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background: white;
  z-index: 2;
}

.advanced-music-player .cover img {
  width: 98px !important;
  height: 98px !important;
  border-radius: 50% !important;
  object-fit: cover !important;
  /* 强制确保图片是完美圆形 */
  display: block !important;
  min-width: 98px !important;
  max-width: 98px !important;
  min-height: 98px !important;
  max-height: 98px !important;
  flex-shrink: 0 !important;
}

.advanced-music-player .cover.rotating {
  animation: rotate 10s linear infinite;
  animation-play-state: running;
}

.advanced-music-player .cover.paused {
  animation-play-state: paused;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.control-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 10px;
  justify-content: center;
}

.control {
  display: flex;
  align-items: center;
  gap: 15px;
  justify-content: center;
}

.control-btn {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  color: #8B4513;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 20px;
}

.control-btn:hover {
  background: rgba(139, 69, 19, 0.1);
  transform: scale(1.1);
}

.play-btn {
  background: #8B4513;
  color: white;
  border-radius: 50%;
}

.play-btn:hover {
  background: #A0522D;
}

.random-btn {
  position: relative;
}

.random-btn.active {
  background: rgba(139, 69, 19, 0.2);
  color: #8B4513;
}

.random-btn.active::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 50%;
  transform: translateX(-50%);
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: #8B4513;
}

.random-icon {
  font-size: 16px;
  display: inline-block;
}

/* 歌曲信息卡片 */
.player-info {
  position: absolute;
  top: 0;
  right: 0; /* 改为靠右对齐 */
  width: 320px; /* 固定宽度，确保一致性 */
  min-width: 320px; /* 最小宽度 */
  max-width: 320px; /* 最大宽度 */
  padding: 15px;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  border-radius: 10px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  z-index: 1;
  opacity: 0;
  transition: all 0.5s ease;
  box-sizing: border-box; /* 确保padding包含在宽度内 */
}

.player-info.show {
  top: -130px;
  opacity: 1;
  z-index: 1;
}

.info {
  text-align: right;
  width: 100%; /* 确保内容区域占满整个卡片 */
}

.name {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 4px;
  white-space: nowrap; /* 防止换行 */
  overflow: hidden; /* 隐藏溢出 */
  text-overflow: ellipsis; /* 显示省略号 */
}

.singer-album {
  font-size: 12px;
  color: #666;
  margin-bottom: 8px;
  white-space: nowrap; /* 防止换行 */
  overflow: hidden; /* 隐藏溢出 */
  text-overflow: ellipsis; /* 显示省略号 */
}

.lyrics-container {
  margin-top: 8px;
  height: 40px; /* 固定歌词容器高度 */
  display: flex;
  align-items: center;
  justify-content: center;
}

.current-lyric {
  font-size: 13px;
  color: #8B4513;
  text-align: center;
  font-style: italic;
  line-height: 1.4;
  min-height: 18px;
  max-height: 36px; /* 限制最大高度，最多两行 */
  transition: all 0.3s ease;
  opacity: 0.9;
  overflow: hidden; /* 隐藏溢出的文字 */
  display: -webkit-box;
  -webkit-line-clamp: 2; /* 最多显示两行 */
  line-clamp: 2; /* 标准属性，用于兼容性 */
  -webkit-box-orient: vertical;
  word-break: break-word; /* 允许单词内换行 */
  width: 100%; /* 占满容器宽度 */
}

.music_progress {
  width: 100%;
  margin-bottom: 8px;
  margin-left: 15px;
  margin-right: 5px;
}

.music_progress_top {
  display: flex;
  justify-content: space-between;
  font-size: 10px;
  color: #8B4513;
  margin-bottom: 4px;
  opacity: 0.8;
}

.music_progress_bar {
  width: 100%;
  height: 3px;
  background: #e0e0e0;
  border-radius: 2px;
  cursor: pointer;
  position: relative;
  transition: height 0.2s ease;
}

.music_progress_bar:hover {
  height: 4px;
}

.music_progress_line {
  height: 100%;
  background: linear-gradient(90deg, #8B4513, #A0522D);
  border-radius: 2px;
  transition: width 0.1s ease;
}

/* 背景 */
.mask_bg {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-size: cover;
  background-position: center;
  filter: blur(50px);
  opacity: 0.3;
  z-index: -10;
  transition: all 1s ease;
  pointer-events: none;
}

/* 播放列表 */
.playlist-select {
  width: 100%;
  margin-bottom: 16px;
}

.netease-player-container {
  margin-top: 16px;
}

.netease-iframe {
  border-radius: 8px;
  overflow: hidden;
}

.music-list {
  list-style: none;
  padding: 0;
  margin: 0;
  max-height: 300px;
  overflow-y: auto;
}

.music-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
  transition: all 0.3s ease;
}

.music-list li:hover {
  background: rgba(139, 69, 19, 0.05);
}

.music-list li.playing {
  background: rgba(139, 69, 19, 0.1);
  color: #8B4513;
}

.play-circle {
  color: #8B4513;
  cursor: pointer;
  transition: all 0.3s ease;
}

.play-circle:hover {
  transform: scale(1.2);
}

/* 播放列表面板样式 */
.playlist-panel {
  position: absolute;
  top: -460px;
  left: 0;
  width: 100%;
  height: 320px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  z-index: 10;
  overflow: hidden;
  transform: translateY(-20px);
  opacity: 0;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  pointer-events: none;
}

.playlist-panel.panel-show {
  transform: translateY(0);
  opacity: 1;
  pointer-events: auto;
}

.playlist-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  border-bottom: 1px solid rgba(139, 69, 19, 0.1);
  background: rgba(139, 69, 19, 0.05);
}

.playlist-header h3 {
  margin: 0;
  color: #8B4513;
  font-size: 16px;
  font-weight: 600;
}

.close-btn {
  font-size: 18px;
  color: #8B4513;
  cursor: pointer;
  transition: all 0.3s ease;
  padding: 4px;
  border-radius: 50%;
}

.close-btn:hover {
  color: #A0522D;
  background: rgba(139, 69, 19, 0.1);
  transform: rotate(90deg);
}

.playlist-content {
  height: calc(100% - 60px);
  overflow-y: auto;
  padding: 10px;
}

.local-playlist {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.song-item {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid transparent;
  animation: slideInUp 0.3s ease forwards;
}

.song-item:nth-child(1) { animation-delay: 0.1s; }
.song-item:nth-child(2) { animation-delay: 0.15s; }
.song-item:nth-child(3) { animation-delay: 0.2s; }
.song-item:nth-child(4) { animation-delay: 0.25s; }
.song-item:nth-child(5) { animation-delay: 0.3s; }
.song-item:nth-child(6) { animation-delay: 0.35s; }
.song-item:nth-child(7) { animation-delay: 0.4s; }
.song-item:nth-child(8) { animation-delay: 0.45s; }

@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.song-item:hover {
  background: rgba(139, 69, 19, 0.1);
  border-color: rgba(139, 69, 19, 0.2);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.song-item:active {
  transform: translateY(0);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
  background: rgba(139, 69, 19, 0.2);
}

.song-item.active {
  background: rgba(139, 69, 19, 0.15);
  border-color: #8B4513;
  box-shadow: 0 2px 8px rgba(139, 69, 19, 0.2);
  position: relative;
}

.song-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 3px;
  background: #8B4513;
  border-radius: 0 2px 2px 0;
}

.song-item.active .song-name {
  color: #8B4513;
  font-weight: 600;
}

.song-item.active .play-icon {
  color: #8B4513;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.6;
  }
}

.song-cover {
  width: 35px;
  height: 35px;
  border-radius: 6px;
  object-fit: cover;
  margin-right: 10px;
  flex-shrink: 0;
}

.song-info {
  flex: 1;
  min-width: 0;
}

.song-name {
  font-size: 13px;
  font-weight: 500;
  color: #333;
  margin-bottom: 2px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.song-artist {
  font-size: 11px;
  color: #666;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.song-duration {
  font-size: 11px;
  color: #999;
  margin-right: 8px;
  flex-shrink: 0;
}

.play-icon {
  font-size: 14px;
  color: #8B4513;
  flex-shrink: 0;
  transition: all 0.3s ease;
}

.play-icon:hover {
  transform: scale(1.2);
}

.play-icon.loading {
  color: #8B4513;
}

.loading-spinner {
  display: inline-block;
  animation: spin 1s linear infinite;
  font-size: 14px;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .advanced-music-player {
    bottom: 10px;
    right: 10px;
    left: 10px;
  }

  .minimized-player {
    width: 100%;
    max-width: 350px;
    margin: 0 auto;
  }

  .player-warp {
    width: 100%;
    max-width: 350px;
  }

  .player-control {
    padding: 15px 20px;
  }

  .control {
    gap: 10px;
  }

  .control-btn {
    width: 36px;
    height: 36px;
    font-size: 18px;
  }
}

@media (max-width: 480px) {
  .advanced-music-player {
    bottom: 80px;
  }

  .advanced-music-player .cover {
    width: 80px !important;
    height: 80px !important;
    margin-top: -45px !important;
    margin-left: -3px !important;
    /* 强制确保移动端封面也是完美圆形 */
    min-width: 80px !important;
    max-width: 80px !important;
    min-height: 80px !important;
    max-height: 80px !important;
  }

  .control {
    gap: 8px;
  }

  .control-btn {
    width: 32px;
    height: 32px;
    font-size: 16px;
  }

  /* 移动端播放列表面板适配 */
  .playlist-panel {
    top: -240px;
    height: 220px;
  }

  .song-item {
    padding: 6px 10px;
  }

  .song-cover {
    width: 30px;
    height: 30px;
    margin-right: 8px;
  }

  .song-name {
    font-size: 12px;
  }

  .song-artist {
    font-size: 10px;
  }

  .song-duration {
    font-size: 10px;
  }

  .play-icon {
    font-size: 12px;
  }
}
</style>
