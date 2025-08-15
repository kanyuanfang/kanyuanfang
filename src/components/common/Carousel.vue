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
          title: '山水如画',
          description: '青山绿水，诗意江南',
          image: 'https://images.unsplash.com/photo-1547036967-23d11aacaee0?ixlib=rb-4.0.3&w=1200&q=80',
          link: '/recommendations'
        },
        {
          id: 2,
          title: '古韵悠长',
          description: '千年古刹，梵音袅袅',
          image: 'https://images.unsplash.com/photo-1578662996442-48f60103fc96?ixlib=rb-4.0.3&w=1200&q=80',
          link: '/recommendations'
        },
        {
          id: 3,
          title: '竹林深处',
          description: '翠竹摇曳，清风徐来',
          image: 'https://images.unsplash.com/photo-1544551763-46a013bb70d5?ixlib=rb-4.0.3&w=1200&q=80',
          link: '/recommendations'
        },
        {
          id: 4,
          title: '湖光山色',
          description: '碧波荡漾，远山如黛',
          image: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?ixlib=rb-4.0.3&w=1200&q=80',
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
  height: 350px;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 12px 35px rgba(139, 69, 19, 0.2);
  border: 2px solid rgba(205, 133, 63, 0.3);
  background: linear-gradient(45deg, rgba(139, 69, 19, 0.05), rgba(205, 133, 63, 0.05));
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
  transition: all 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  cursor: pointer;
  transform: scale(1.05);
}

.carousel-slide.active {
  opacity: 1;
  transform: scale(1);
}

.carousel-slide::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, rgba(139, 69, 19, 0.1), rgba(205, 133, 63, 0.1));
  z-index: 1;
  pointer-events: none;
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
  background: linear-gradient(transparent, rgba(139, 69, 19, 0.9), rgba(0, 0, 0, 0.8));
  color: white;
  padding: 2.5rem;
  text-align: left;
  z-index: 2;
  backdrop-filter: blur(2px);
}

.slide-title {
  font-size: 1.8rem;
  font-weight: 600;
  margin-bottom: 0.8rem;
  text-shadow: 2px 2px 6px rgba(0, 0, 0, 0.7);
  font-family: 'STKaiti', '楷体', serif;
  letter-spacing: 2px;
  color: #F5F5DC;
}

.slide-description {
  font-size: 1.1rem;
  margin-bottom: 1.5rem;
  opacity: 0.95;
  font-family: 'STKaiti', '楷体', serif;
  letter-spacing: 1px;
  line-height: 1.6;
  color: #F5F5DC;
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
  width: 14px;
  height: 14px;
  border-radius: 50%;
  background: rgba(205, 133, 63, 0.6);
  border: 2px solid rgba(139, 69, 19, 0.4);
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(2px);
}

.indicator.active {
  background: rgba(139, 69, 19, 0.9);
  border-color: rgba(205, 133, 63, 0.8);
  transform: scale(1.3);
  box-shadow: 0 2px 8px rgba(139, 69, 19, 0.4);
}

.carousel-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(139, 69, 19, 0.8);
  border: 2px solid rgba(205, 133, 63, 0.6);
  border-radius: 50%;
  width: 45px;
  height: 45px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 10;
  color: #F5F5DC;
  backdrop-filter: blur(4px);
  box-shadow: 0 4px 15px rgba(139, 69, 19, 0.3);
}

.carousel-btn:hover {
  background: rgba(205, 133, 63, 0.9);
  border-color: rgba(139, 69, 19, 0.8);
  transform: translateY(-50%) scale(1.15);
  box-shadow: 0 6px 20px rgba(139, 69, 19, 0.4);
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
