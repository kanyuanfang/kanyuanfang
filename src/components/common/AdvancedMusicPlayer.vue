<template>
  <div class="advanced-music-player" :class="{ minimized: isMinimized }">
    <!-- 欢迎提示 -->
    <div v-if="showWelcomePrompt" class="welcome-prompt" @click="handleWelcomeClick">
      <div class="welcome-content">
        <div class="welcome-icon">🎵</div>
        <h3>欢迎来到青岚音乐</h3>
        <p>点击开始享受美妙的音乐之旅</p>
        <el-button type="primary" @click="handleWelcomeClick" class="welcome-btn">
          <el-icon><VideoPlay /></el-icon>
          开始播放
        </el-button>
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
          <div class="lyrics-container" v-if="currentSong.lyrics">
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
import { VideoPlay, VideoPause, DArrowLeft, DArrowRight, List, Close } from '@element-plus/icons-vue'

export default {
  name: 'AdvancedMusicPlayer',
  components: {
    VideoPlay,
    VideoPause,
    DArrowLeft,
    DArrowRight,
    List,
    Close
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
          name: "我记得",
          audio_url: "/music-player-demo-master/audios/我记得.mp3",
          singer: "赵雷",
          album: "署前街少年",
          cover: "http://p2.music.126.net/FCWD6ibS2JK2B3QAnXuzwQ==/109951167805892385.jpg",
          time: "05:29",
          lyrics: [
            { time: 0, text: "♪ 音乐开始 ♪" },
            { time: 15, text: "那些年的时光" },
            { time: 30, text: "如今还在心上" },
            { time: 45, text: "记忆中的模样" },
            { time: 60, text: "永远不会遗忘" },
            { time: 75, text: "青春的岁月里" },
            { time: 90, text: "有你陪伴身旁" },
            { time: 105, text: "那些美好时光" },
            { time: 120, text: "我记得..." }
          ]
        },
        {
          name: "成都",
          audio_url: "/music-player-demo-master/audios/成都.mp3",
          singer: "赵雷",
          album: "成都",
          cover: "http://p2.music.126.net/34YW1QtKxJ_3YnX9ZzKhzw==/2946691234868155.jpg",
          time: "05:28",
          lyrics: [
            { time: 0, text: "♪ 音乐开始 ♪" },
            { time: 20, text: "让我掉下眼泪的" },
            { time: 35, text: "不止昨夜的酒" },
            { time: 50, text: "让我依依不舍的" },
            { time: 65, text: "不止你的温柔" },
            { time: 80, text: "余路还要走多久" },
            { time: 95, text: "你攥着我的手" },
            { time: 110, text: "让我感到为难的" },
            { time: 125, text: "是挣扎的自由" }
          ]
        },
        {
          name: "南方姑娘",
          audio_url: "/music-player-demo-master/audios/南方姑娘.mp3",
          singer: "赵雷",
          album: "赵小雷",
          cover: "http://p2.music.126.net/wldFtES1Cjnbqr5bjlqQbg==/18876415625841069.jpg",
          time: "05:32",
          lyrics: [
            { time: 0, text: "♪ 音乐开始 ♪" },
            { time: 18, text: "南方的艳阳里" },
            { time: 33, text: "大雪纷飞" },
            { time: 48, text: "北方的寒夜里" },
            { time: 63, text: "四季如春" },
            { time: 78, text: "如果天黑之前来得及" },
            { time: 93, text: "我要忘了你的眼睛" },
            { time: 108, text: "穷极一生做不完一场梦" }
          ]
        },
        {
          name: "阴天快乐",
          audio_url: "/music-player-demo-master/audios/阴天快乐.mp3",
          singer: "陈奕迅",
          album: "Rice & Shine",
          cover: "http://p2.music.126.net/itkdsMFR8nYzaTiDdHO3tA==/109951165995320408.jpg",
          time: "04:20",
          lyrics: [
            { time: 0, text: "♪ 音乐开始 ♪" },
            { time: 15, text: "阴天快乐" },
            { time: 30, text: "心情如天气" },
            { time: 45, text: "时晴时雨" },
            { time: 60, text: "但总会放晴" },
            { time: 75, text: "就像这首歌" },
            { time: 90, text: "带给你温暖" },
            { time: 105, text: "阴天也快乐" }
          ]
        },
        {
          name: "爱情转移",
          audio_url: "/music-player-demo-master/audios/爱情转移.mp3",
          singer: "陈奕迅",
          album: "认了吧",
          cover: "http://p2.music.126.net/o_OjL_NZNoeog9fIjBXAyw==/18782957139233959.jpg",
          time: "04:20",
          lyrics: [
            { time: 0, text: "♪ 音乐开始 ♪" },
            { time: 20, text: "爱情转移" },
            { time: 35, text: "像季节更替" },
            { time: 50, text: "时间在流逝" },
            { time: 65, text: "感情在变迁" },
            { time: 80, text: "但美好的回忆" },
            { time: 95, text: "永远不会消失" },
            { time: 110, text: "在心中闪闪发光" }
          ]
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
      if (!this.currentSong.lyrics || this.currentSong.lyrics.length === 0) {
        return '♪ 享受音乐 ♪'
      }

      // 根据当前播放时间找到对应的歌词
      let lyricIndex = 0
      for (let i = 0; i < this.currentSong.lyrics.length; i++) {
        if (this.currentTime >= this.currentSong.lyrics[i].time) {
          lyricIndex = i
        } else {
          break
        }
      }

      return this.currentSong.lyrics[lyricIndex]?.text || '♪ 享受音乐 ♪'
    }
  },
  mounted() {
    // 从本地存储恢复状态
    const savedIndex = localStorage.getItem('currentSongIndex')
    const savedPlaylist = localStorage.getItem('selectedPlaylist')
    const savedTab = localStorage.getItem('activeTab')
    const hasInteracted = localStorage.getItem('musicPlayerInteracted')

    if (savedIndex) this.currentIndex = parseInt(savedIndex)
    if (savedPlaylist) this.selectedPlaylist = savedPlaylist
    if (savedTab) this.activeTab = savedTab
    if (hasInteracted === 'true') {
      this.showWelcomePrompt = false
      this.hasUserInteracted = true
    }

    this.loadCurrentSong()

    // 添加用户交互监听器来自动播放
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
      this.currentIndex = this.currentIndex < this.musicList.length - 1 ? this.currentIndex + 1 : 0
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
      this.nextSong()
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
    handleWelcomeClick() {
      this.showWelcomePrompt = false
      this.hasUserInteracted = true
      localStorage.setItem('musicPlayerInteracted', 'true')
      // 自动开始播放第一首歌
      this.$nextTick(() => {
        this.togglePlay().catch(() => {
          // 播放失败时静默处理
        })
      })
    },
    addAutoPlayListener() {
      // 监听用户的第一次交互
      const startAutoPlay = () => {
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

.welcome-btn {
  font-size: 1.1rem;
  padding: 12px 24px;
  border-radius: 25px;
  background: white;
  color: #8B4513;
  border: none;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
}

.welcome-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
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

/* 歌曲信息卡片 */
.player-info {
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 90%;
  padding: 15px;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  border-radius: 10px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  z-index: 1;
  opacity: 0;
  transition: all 0.5s ease;
}

.player-info.show {
  top: -110px;
  opacity: 1;
  z-index: 1;
}

.info {
  text-align: right;
}

.name {
  font-size: 16px;
  font-weight: bold;
  color: #333;
  margin-bottom: 4px;
}

.singer-album {
  font-size: 12px;
  color: #666;
  margin-bottom: 8px;
}

.lyrics-container {
  margin-top: 8px;
}

.current-lyric {
  font-size: 13px;
  color: #8B4513;
  text-align: center;
  font-style: italic;
  line-height: 1.4;
  min-height: 18px;
  transition: all 0.3s ease;
  opacity: 0.9;
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
