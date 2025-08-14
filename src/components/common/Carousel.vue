<template>
  <div class="carousel-container">
    <div class="carousel-wrapper">
      <div 
        class="carousel-slide" 
        v-for="(slide, index) in slides" 
        :key="index"
        :class="{ active: currentSlide === index }"
        @click="goToSlide(index)"
      >
        <div class="slide-content">
          <img :src="slide.image" :alt="slide.title" class="slide-image" />
          <div class="slide-overlay">
            <h3 class="slide-title">{{ slide.title }}</h3>
            <p class="slide-description">{{ slide.description }}</p>
            <el-button type="primary" @click.stop="handleSlideClick(slide)">
              了解更多
            </el-button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 指示器 -->
    <div class="carousel-indicators">
      <span 
        v-for="(slide, index) in slides" 
        :key="index"
        class="indicator"
        :class="{ active: currentSlide === index }"
        @click="goToSlide(index)"
      ></span>
    </div>
    
    <!-- 导航按钮 -->
    <button class="carousel-btn prev" @click="prevSlide">
      <el-icon><ArrowLeft /></el-icon>
    </button>
    <button class="carousel-btn next" @click="nextSlide">
      <el-icon><ArrowRight /></el-icon>
    </button>
  </div>
</template>

<script>
import { ArrowLeft, ArrowRight } from '@element-plus/icons-vue'

export default {
  name: 'CarouselComponent',
  components: {
    ArrowLeft,
    ArrowRight
  },
  data() {
    return {
      currentSlide: 0,
      autoPlayTimer: null,
      slides: [
        {
          id: 1,
          title: '探索自然之美',
          description: '发现世界各地的壮丽风景',
          image: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?ixlib=rb-4.0.3&w=800&q=80',
          link: '/recommendations'
        },
        {
          id: 2,
          title: '音乐的力量',
          description: '聆听来自世界的美妙旋律',
          image: 'https://images.unsplash.com/photo-1493225457124-a3eb161ffa5f?ixlib=rb-4.0.3&w=800&q=80',
          link: '/recommendations'
        },
        {
          id: 3,
          title: '科技前沿',
          description: '探索最新的技术趋势和工具',
          image: 'https://images.unsplash.com/photo-1518709268805-4e9042af2176?ixlib=rb-4.0.3&w=800&q=80',
          link: '/recommendations'
        },
        {
          id: 4,
          title: '文化艺术',
          description: '感受不同文化的艺术魅力',
          image: 'https://images.unsplash.com/photo-1541961017774-22349e4a1262?ixlib=rb-4.0.3&w=800&q=80',
          link: '/blog'
        }
      ]
    }
  },
  mounted() {
    this.startAutoPlay()
  },
  beforeUnmount() {
    this.stopAutoPlay()
  },
  methods: {
    goToSlide(index) {
      this.currentSlide = index
      this.resetAutoPlay()
    },
    nextSlide() {
      this.currentSlide = (this.currentSlide + 1) % this.slides.length
      this.resetAutoPlay()
    },
    prevSlide() {
      this.currentSlide = this.currentSlide === 0 ? this.slides.length - 1 : this.currentSlide - 1
      this.resetAutoPlay()
    },
    startAutoPlay() {
      this.autoPlayTimer = setInterval(() => {
        this.nextSlide()
      }, 5000)
    },
    stopAutoPlay() {
      if (this.autoPlayTimer) {
        clearInterval(this.autoPlayTimer)
        this.autoPlayTimer = null
      }
    },
    resetAutoPlay() {
      this.stopAutoPlay()
      this.startAutoPlay()
    },
    handleSlideClick(slide) {
      this.$router.push(slide.link)
    }
  }
}
</script>

<style scoped>
.carousel-container {
  position: relative;
  width: 100%;
  height: 300px;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.carousel-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
}

.carousel-slide {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: opacity 0.5s ease-in-out;
  cursor: pointer;
}

.carousel-slide.active {
  opacity: 1;
}

.slide-content {
  position: relative;
  width: 100%;
  height: 100%;
}

.slide-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.slide-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.8));
  color: white;
  padding: 2rem;
  text-align: left;
}

.slide-title {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.5);
}

.slide-description {
  font-size: 1rem;
  margin-bottom: 1rem;
  opacity: 0.9;
}

.carousel-indicators {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
}

.indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: all 0.3s ease;
}

.indicator.active {
  background: white;
  transform: scale(1.2);
}

.carousel-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(255, 255, 255, 0.9);
  border: none;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 10;
}

.carousel-btn:hover {
  background: white;
  transform: translateY(-50%) scale(1.1);
}

.carousel-btn.prev {
  left: 15px;
}

.carousel-btn.next {
  right: 15px;
}

@media (max-width: 768px) {
  .carousel-container {
    height: 250px;
  }
  
  .slide-overlay {
    padding: 1rem;
  }
  
  .slide-title {
    font-size: 1.2rem;
  }
  
  .slide-description {
    font-size: 0.9rem;
  }
  
  .carousel-btn {
    width: 35px;
    height: 35px;
  }
}
</style>
