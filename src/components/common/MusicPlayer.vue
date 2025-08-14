<template>
  <div class="music-player" :class="{ minimized: isMinimized }">
    <div class="player-header" @click="toggleMinimize">
      <span class="player-title">🎵 音乐播放器</span>
      <el-icon class="minimize-icon">
        <ArrowUp v-if="!isMinimized" />
        <ArrowDown v-if="isMinimized" />
      </el-icon>
    </div>

    <div class="player-content" v-show="!isMinimized">
      <!-- 网易云音乐外链播放器 -->
      <div class="netease-player">
        <iframe
          frameborder="no"
          border="0"
          marginwidth="0"
          marginheight="0"
          width="100%"
          height="380"
          :src="musicUrl"
          class="music-iframe"
        ></iframe>
      </div>

      <!-- 播放器控制选项 -->
      <div class="player-controls">
        <el-select v-model="selectedPlaylist" @change="changePlaylist" size="small" class="playlist-select">
          <el-option
            v-for="playlist in playlists"
            :key="playlist.id"
            :label="playlist.name"
            :value="playlist.id"
          />
        </el-select>
      </div>
    </div>
  </div>
</template>

<script>
import { ArrowUp, ArrowDown } from '@element-plus/icons-vue'

export default {
  name: 'MusicPlayer',
  components: {
    ArrowUp,
    ArrowDown
  },
  data() {
    return {
      isMinimized: false,
      selectedPlaylist: '13629151489',
      playlists: [
        { id: '514947114', name: '默认歌单' },
        { id: '2884035', name: '华语流行' },
        { id: '19723756', name: '轻音乐' },
        { id: '3779629', name: '民谣' },
        { id: '2250011882', name: '古风' },
        { id: '13629151489', name: '我的音乐' }
      ]
    }
  },
  computed: {
    musicUrl() {
      return `//music.163.com/outchain/player?type=0&id=${this.selectedPlaylist}&auto=1&height=380`
    }
  },
  methods: {
    toggleMinimize() {
      this.isMinimized = !this.isMinimized
    },
    changePlaylist(playlistId) {
      this.selectedPlaylist = playlistId
      // 保存用户选择到本地存储
      localStorage.setItem('selectedPlaylist', playlistId)
    }
  },
  mounted() {
    // 从本地存储恢复用户选择
    const savedPlaylist = localStorage.getItem('selectedPlaylist')
    if (savedPlaylist) {
      this.selectedPlaylist = savedPlaylist
    }
  }
}
</script>

<style scoped>
.music-player {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 280px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
  z-index: 1000;
  transition: all 0.3s ease;
  overflow: hidden;
}

.music-player.minimized {
  width: 180px;
  height: 50px;
}

.player-header {
  padding: 0.8rem 1rem;
  background: linear-gradient(135deg, #8B4513 0%, #A0522D 50%, #CD853F 100%);
  color: white;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.9rem;
}

.player-title {
  font-weight: 600;
  font-size: 0.85rem;
}

.minimize-icon {
  transition: transform 0.3s ease;
  font-size: 16px;
}

.player-content {
  background: #fafafa;
}

.netease-player {
  width: 100%;
  height: 380px;
  overflow: hidden;
}

.music-iframe {
  border: none;
  border-radius: 0;
  background: transparent;
}

.player-controls {
  padding: 0.8rem;
  background: white;
  border-top: 1px solid #f0f0f0;
}

.playlist-select {
  width: 100%;
}

.playlist-select .el-input__inner {
  font-size: 0.8rem;
  height: 32px;
  line-height: 32px;
}

@media (max-width: 768px) {
  .music-player {
    bottom: 10px;
    right: 10px;
    left: 10px;
    width: auto;
    max-width: 300px;
    margin: 0 auto;
  }

  .music-player.minimized {
    width: 160px;
    left: auto;
    right: 10px;
  }

  .netease-player {
    height: 350px;
  }

  .player-header {
    padding: 0.6rem 0.8rem;
  }

  .player-title {
    font-size: 0.8rem;
  }
}

@media (max-width: 480px) {
  .music-player {
    bottom: 80px; /* 避免与移动端底部导航冲突 */
  }

  .netease-player {
    height: 320px;
  }
}
</style>
