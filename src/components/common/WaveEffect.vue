<template>
  <div class="wave-container" v-if="enabled">
    <div class="wave wave1"></div>
    <div class="wave wave2"></div>
    <div class="wave wave3"></div>
  </div>
</template>

<script>
export default {
  name: 'WaveEffect',
  mounted() {
    console.log('WaveEffect mounted, enabled:', this.enabled, 'theme:', this.theme)
  },
  props: {
    // 是否启用波浪效果
    enabled: {
      type: Boolean,
      default: true
    },
    // 波浪颜色主题
    theme: {
      type: String,
      default: 'winter', // winter, ocean, aurora
      validator: value => ['winter', 'ocean', 'aurora'].includes(value)
    },
    // 波浪透明度
    opacity: {
      type: Number,
      default: 0.1,
      validator: value => value >= 0 && value <= 1
    }
  },
  computed: {
    waveColors() {
      const themes = {
        winter: {
          wave1: 'rgba(176, 196, 222, 0.8)', // 淡钢蓝色
          wave2: 'rgba(230, 230, 250, 0.6)', // 淡紫色
          wave3: 'rgba(255, 255, 255, 0.4)'  // 白色
        },
        ocean: {
          wave1: 'rgba(70, 130, 180, 0.8)',  // 钢蓝色
          wave2: 'rgba(135, 206, 235, 0.6)', // 天蓝色
          wave3: 'rgba(173, 216, 230, 0.4)'  // 淡蓝色
        },
        aurora: {
          wave1: 'rgba(138, 43, 226, 0.8)',  // 紫色
          wave2: 'rgba(72, 61, 139, 0.6)',   // 深蓝紫色
          wave3: 'rgba(147, 112, 219, 0.4)'  // 中紫色
        }
      }
      return themes[this.theme]
    }
  }
}
</script>

<style scoped>
.wave-container {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 5;
  overflow: hidden;
  background: transparent;
}

.wave {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 200%;
  height: 200px;
  background: linear-gradient(
    45deg,
    v-bind('waveColors.wave1') 0%,
    v-bind('waveColors.wave2') 50%,
    v-bind('waveColors.wave3') 100%
  );
  border-radius: 50%;
  animation-iteration-count: infinite;
  animation-timing-function: ease-in-out;
}

.wave1 {
  animation: wave1 8s infinite;
  opacity: v-bind('opacity * 1.2');
  transform: translateX(-50%) translateY(50%);
}

.wave2 {
  animation: wave2 12s infinite;
  opacity: v-bind('opacity');
  transform: translateX(-50%) translateY(60%);
  animation-delay: -2s;
}

.wave3 {
  animation: wave3 16s infinite;
  opacity: v-bind('opacity * 0.8');
  transform: translateX(-50%) translateY(70%);
  animation-delay: -4s;
}

/* 波浪动画 */
@keyframes wave1 {
  0%, 100% {
    transform: translateX(-50%) translateY(50%) rotate(0deg);
  }
  25% {
    transform: translateX(-45%) translateY(45%) rotate(1deg);
  }
  50% {
    transform: translateX(-50%) translateY(40%) rotate(0deg);
  }
  75% {
    transform: translateX(-55%) translateY(45%) rotate(-1deg);
  }
}

@keyframes wave2 {
  0%, 100% {
    transform: translateX(-50%) translateY(60%) rotate(0deg);
  }
  25% {
    transform: translateX(-55%) translateY(55%) rotate(-1deg);
  }
  50% {
    transform: translateX(-50%) translateY(50%) rotate(0deg);
  }
  75% {
    transform: translateX(-45%) translateY(55%) rotate(1deg);
  }
}

@keyframes wave3 {
  0%, 100% {
    transform: translateX(-50%) translateY(70%) rotate(0deg);
  }
  25% {
    transform: translateX(-45%) translateY(65%) rotate(1deg);
  }
  50% {
    transform: translateX(-50%) translateY(60%) rotate(0deg);
  }
  75% {
    transform: translateX(-55%) translateY(65%) rotate(-1deg);
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .wave {
    height: 150px;
  }
}

@media (max-width: 480px) {
  .wave {
    height: 100px;
  }
}

/* 减少动画在低性能设备上的影响 */
@media (prefers-reduced-motion: reduce) {
  .wave {
    animation-duration: 20s;
  }
}
</style>
