<template>
  <div class="home">
    <!-- 轮播图区域 -->
    <section class="hero-section">
      <div class="hero-content">
        <h2 class="hero-title">欢迎来到青岚</h2>
        <p class="hero-subtitle">在这里发现美好，分享生活</p>
      </div>
    </section>

    <!-- 主要内容区域 -->
    <div class="main-container">
      <!-- 左侧内容 -->
      <div class="left-content">
        <!-- 轮播图组件 -->
        <div class="carousel-section">
          <h3>精选推荐</h3>
          <Carousel />
        </div>

        <!-- 推荐模块 -->
        <div class="recommendations-grid">
          <div class="recommendation-card" @click="goToRecommendations('websites')">
            <div class="card-icon">🌐</div>
            <h4>网站推荐</h4>
            <p>精选优质网站，发现互联网的精彩</p>
            <div class="card-stats">
              <span>120+ 网站</span>
            </div>
          </div>
          <div class="recommendation-card" @click="goToRecommendations('music')">
            <div class="card-icon">🎵</div>
            <h4>音乐推荐</h4>
            <p>好听的音乐分享，治愈你的心灵</p>
            <div class="card-stats">
              <span>300+ 歌曲</span>
            </div>
          </div>
          <div class="recommendation-card" @click="goToRecommendations('movies')">
            <div class="card-icon">🎬</div>
            <h4>影视推荐</h4>
            <p>精彩电影电视剧，享受视觉盛宴</p>
            <div class="card-stats">
              <span>80+ 影片</span>
            </div>
          </div>
          <div class="recommendation-card" @click="goToRecommendations('software')">
            <div class="card-icon">💻</div>
            <h4>软件推荐</h4>
            <p>实用工具软件，提升工作效率</p>
            <div class="card-stats">
              <span>50+ 工具</span>
            </div>
          </div>
          <div class="recommendation-card" @click="goToRecommendations('ai')">
            <div class="card-icon">🤖</div>
            <h4>AI工具</h4>
            <p>大模型工具推荐，拥抱AI时代</p>
            <div class="card-stats">
              <span>30+ 工具</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 右侧边栏 -->
      <div class="right-sidebar">
        <!-- 资讯模块 -->
        <div class="news-section">
          <h3>今日资讯</h3>
          <div class="news-item">
            <h5>📰 今日新鲜事</h5>
            <p>最新热点资讯</p>
          </div>
          <div class="news-item">
            <h5>🔒 网安事件</h5>
            <p>网络安全动态</p>
          </div>
          <div class="news-item">
            <h5>🍉 吃瓜事件</h5>
            <p>热门话题讨论</p>
          </div>
          <div class="news-item">
            <h5>📅 历史上的今天</h5>
            <p>历史事件回顾</p>
          </div>
        </div>

        <!-- 世界地图 -->
        <div class="map-section">
          <h3>🗺️ 世界地图</h3>
          <div class="map-placeholder">
            <p>世界地图组件</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Carousel from '../components/common/Carousel.vue'

export default {
  name: 'HomePage',
  components: {
    Carousel
  },
  data() {
    return {
      // 数据将在后续添加
    }
  },
  methods: {
    goToRecommendations(category) {
      this.$router.push({
        path: '/recommendations',
        query: { category }
      })
    }
  },
  mounted() {
    // 组件挂载后的逻辑
  }
}
</script>

<style scoped>
.home {
  min-height: 100vh;
}

.hero-section {
  background: linear-gradient(rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0.6)),
              url('https://images.unsplash.com/photo-1544551763-46a013bb70d5?ixlib=rb-4.0.3&w=1920&q=80') center/cover;
  height: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  text-align: center;
  margin-bottom: 1.5rem;
  position: relative;
  overflow: hidden;
}

.hero-section::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, rgba(139, 69, 19, 0.15), rgba(205, 133, 63, 0.15));
  animation: shimmer 4s ease-in-out infinite alternate;
}

@keyframes shimmer {
  0% { opacity: 0.1; }
  100% { opacity: 0.25; }
}

.hero-content {
  position: relative;
  z-index: 2;
  animation: fadeInUp 1s ease-out;
}

.hero-content h2 {
  font-size: 2.8rem;
  margin-bottom: 0.8rem;
  text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.7);
  font-weight: 600;
  letter-spacing: 3px;
  font-family: 'STKaiti', '楷体', serif;
}

.hero-content p {
  font-size: 1.1rem;
  opacity: 0.95;
  text-shadow: 1px 1px 4px rgba(0, 0, 0, 0.5);
  font-weight: 300;
  letter-spacing: 2px;
  font-family: 'STKaiti', '楷体', serif;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.main-container {
  max-width: 1100px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 1.5rem;
  padding: 0 1rem;
}

.left-content {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.carousel-section {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  padding: 1.2rem;
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.1);
  border: 1px solid rgba(205, 133, 63, 0.2);
}

.carousel-section h3 {
  margin-bottom: 1rem;
  color: #8B4513;
  font-family: 'STKaiti', '楷体', serif;
  font-weight: 600;
}



.recommendations-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
}

.recommendation-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.1);
  border: 1px solid rgba(205, 133, 63, 0.2);
  transition: all 0.3s ease;
  cursor: pointer;
  text-align: center;
  position: relative;
  overflow: hidden;
}

.recommendation-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s ease;
}

.recommendation-card:hover::before {
  left: 100%;
}

.recommendation-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 8px 20px rgba(139, 69, 19, 0.2);
  border-color: rgba(205, 133, 63, 0.4);
}

.card-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
  display: block;
}

.recommendation-card h4 {
  margin-bottom: 0.8rem;
  color: #2c3e50;
  font-size: 1.2rem;
  font-weight: 600;
}

.recommendation-card p {
  color: #7f8c8d;
  font-size: 0.95rem;
  line-height: 1.5;
  margin-bottom: 1rem;
}

.card-stats {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid #ecf0f1;
}

.card-stats span {
  background: linear-gradient(135deg, #8B4513 0%, #CD853F 100%);
  color: white;
  padding: 0.3rem 0.8rem;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 500;
  box-shadow: 0 2px 8px rgba(139, 69, 19, 0.3);
}

.right-sidebar {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.news-section,
.map-section {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 12px;
  padding: 1.2rem;
  box-shadow: 0 4px 12px rgba(139, 69, 19, 0.1);
  border: 1px solid rgba(205, 133, 63, 0.2);
}

.news-section h3,
.map-section h3 {
  margin-bottom: 1rem;
  color: #8B4513;
  font-family: 'STKaiti', '楷体', serif;
  font-weight: 600;
}

.news-item {
  padding: 0.8rem;
  border-left: 3px solid #CD853F;
  margin-bottom: 0.8rem;
  background: rgba(250, 248, 243, 0.8);
  border-radius: 0 8px 8px 0;
  transition: all 0.3s ease;
}

.news-item:hover {
  background: rgba(205, 133, 63, 0.1);
  border-left-color: #8B4513;
}

.news-item h5 {
  margin-bottom: 0.4rem;
  color: #8B4513;
  font-size: 0.9rem;
}

.news-item p {
  color: #A0522D;
  font-size: 0.8rem;
}

.map-placeholder {
  height: 300px;
  background: #f8f9fa;
  border: 2px dashed #dee2e6;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  color: #6c757d;
}

@media (max-width: 768px) {
  .hero-section {
    height: 300px;
  }

  .hero-content h2 {
    font-size: 2rem;
  }

  .hero-content p {
    font-size: 1rem;
  }

  .main-container {
    grid-template-columns: 1fr;
    padding: 0 0.5rem;
    gap: 1rem;
  }

  .recommendations-grid {
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 0.8rem;
  }

  .recommendation-card {
    padding: 1rem;
  }

  .carousel-section,
  .news-section,
  .map-section {
    padding: 1rem;
  }
}
</style>
