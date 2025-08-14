<template>
  <div class="main-layout">
    <!-- 头部组件 -->
    <Header />
    
    <!-- 主要内容区域 -->
    <main class="main-content">
      <router-view />
    </main>
    
    <!-- 底部组件 -->
    <Footer />
    
    <!-- 高级音乐播放器 -->
    <AdvancedMusicPlayer ref="musicPlayer" />
  </div>
</template>

<script>
import Header from './Header.vue'
import Footer from './Footer.vue'
import AdvancedMusicPlayer from '../common/AdvancedMusicPlayer.vue'

export default {
  name: 'MainLayout',
  components: {
    Header,
    Footer,
    AdvancedMusicPlayer
  }
}
</script>

<style scoped>
.main-layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
  padding: 20px;
}

@media (max-width: 768px) {
  .main-content {
    padding: 10px;
  }
}
</style>
