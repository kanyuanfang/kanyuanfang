<template>
  <div class="snow-container" v-if="enabled">
    <canvas
      ref="snowCanvas"
      class="snow-canvas"
      :width="canvasWidth"
      :height="canvasHeight"
    ></canvas>
  </div>
</template>

<script>
export default {
  name: 'SnowEffect',
  props: {
    // 雪花数量
    maxFlake: {
      type: Number,
      default: 80
    },
    // 雪花大小
    flakeSize: {
      type: Number,
      default: 4
    },
    // 下落速度
    fallSpeed: {
      type: Number,
      default: 0.3
    },
    // 是否启用雪花效果
    enabled: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      canvas: null,
      ctx: null,
      flakes: [],
      animationId: null,
      canvasWidth: 0,
      canvasHeight: 0,
      // 鼠标位置跟踪
      mouseX: -1000,
      mouseY: -1000,
      mouseActive: false
    }
  },
  mounted() {
    console.log('SnowEffect mounted, enabled:', this.enabled)
    if (this.enabled) {
      this.initCanvas()
      this.createFlakes()
      this.startSnowfall()
    }

    // 监听窗口大小变化
    window.addEventListener('resize', this.handleResize)

    // 监听鼠标移动
    window.addEventListener('mousemove', this.handleMouseMove)
    window.addEventListener('mouseenter', this.handleMouseEnter)
    window.addEventListener('mouseleave', this.handleMouseLeave)
  },
  beforeUnmount() {
    this.stopSnowfall()
    window.removeEventListener('resize', this.handleResize)
    window.removeEventListener('mousemove', this.handleMouseMove)
    window.removeEventListener('mouseenter', this.handleMouseEnter)
    window.removeEventListener('mouseleave', this.handleMouseLeave)
  },
  watch: {
    enabled(newVal) {
      if (newVal) {
        this.initCanvas()
        this.createFlakes()
        this.startSnowfall()
      } else {
        this.stopSnowfall()
      }
    }
  },
  methods: {
    // 初始化Canvas
    initCanvas() {
      this.canvas = this.$refs.snowCanvas
      if (!this.canvas) return

      this.ctx = this.canvas.getContext('2d')
      this.updateCanvasSize()
    },

    // 更新Canvas尺寸
    updateCanvasSize() {
      this.canvasWidth = window.innerWidth
      this.canvasHeight = window.innerHeight
      if (this.canvas) {
        this.canvas.width = this.canvasWidth
        this.canvas.height = this.canvasHeight
      }
    },

    // 窗口大小变化处理
    handleResize() {
      this.updateCanvasSize()
    },

    // 鼠标移动处理
    handleMouseMove(event) {
      this.mouseX = event.clientX
      this.mouseY = event.clientY
      this.mouseActive = true
    },

    // 鼠标进入页面
    handleMouseEnter() {
      this.mouseActive = true
    },

    // 鼠标离开页面
    handleMouseLeave() {
      this.mouseActive = false
      this.mouseX = -1000
      this.mouseY = -1000
    },

    // 创建雪花
    createFlakes() {
      this.flakes = []
      for (let i = 0; i < this.maxFlake; i++) {
        this.flakes.push(this.createFlake())
      }
      console.log(`Created ${this.flakes.length} snowflakes`)
    },

    // 创建单个雪花对象
    createFlake() {
      return {
        x: Math.floor(Math.random() * this.canvasWidth),
        y: Math.floor(Math.random() * this.canvasHeight),
        size: Math.random() * this.flakeSize + 2, // 雪花大小 (2-6px)
        maxSize: this.flakeSize,
        speed: Math.random() * 0.3 + this.fallSpeed, // 更慢的速度
        fallSpeed: this.fallSpeed,
        velY: Math.random() * 0.3 + this.fallSpeed, // 垂直速度
        velX: (Math.random() - 0.5) * 0.1, // 更小的初始水平速度
        rotation: Math.random() * Math.PI * 2, // 初始旋转角度
        rotationSpeed: (Math.random() - 0.5) * 0.01, // 更慢的旋转速度
        opacity: Math.random() * 0.4 + 0.6, // 透明度 (0.6-1.0)
        swayAmplitude: Math.random() * 0.2 + 0.1, // 更小的摆动幅度
        swaySpeed: Math.random() * 0.01 + 0.005, // 更慢的摆动速度
        swayOffset: Math.random() * Math.PI * 2, // 摆动相位偏移
        // 鼠标躲避相关属性
        avoidanceForceX: 0,
        avoidanceForceY: 0,
        avoidanceRadius: 80 // 躲避半径
      }
    },

    // 更新雪花位置 - 真实物理运动
    updateFlake(flake) {
      // 重力影响 - 垂直下落
      flake.velY += 0.01 // 重力加速度
      if (flake.velY > flake.speed * 2) {
        flake.velY = flake.speed * 2 // 限制最大下落速度
      }

      // 空气阻力
      flake.velX *= 0.99 // 更小的阻力，保持飘逸感
      flake.velY *= 0.999

      // 风的影响 - 更轻柔的水平摆动
      flake.swayOffset += flake.swaySpeed
      const windForce = Math.sin(flake.swayOffset) * flake.swayAmplitude
      flake.velX += windForce * 0.05 // 减小风力影响

      // 随机微风扰动 - 减少频率和强度
      if (Math.random() < 0.005) {
        flake.velX += (Math.random() - 0.5) * 0.05
      }

      // 鼠标躲避效果
      if (this.mouseActive) {
        this.applyMouseAvoidance(flake)
      } else {
        // 逐渐减少躲避力
        flake.avoidanceForceX *= 0.95
        flake.avoidanceForceY *= 0.95
      }

      // 应用躲避力
      flake.velX += flake.avoidanceForceX
      flake.velY += flake.avoidanceForceY

      // 旋转
      flake.rotation += flake.rotationSpeed

      // 更新位置
      flake.x += flake.velX
      flake.y += flake.velY

      // 边界处理
      if (flake.x >= this.canvasWidth + 10 || flake.x <= -10 || flake.y >= this.canvasHeight + 10) {
        this.resetFlake(flake)
      }
    },

    // 鼠标躲避算法
    applyMouseAvoidance(flake) {
      const dx = flake.x - this.mouseX
      const dy = flake.y - this.mouseY
      const distance = Math.sqrt(dx * dx + dy * dy)

      // 如果雪花在躲避半径内
      if (distance < flake.avoidanceRadius && distance > 0) {
        // 计算躲避强度（距离越近，躲避力越强）
        const avoidanceStrength = (flake.avoidanceRadius - distance) / flake.avoidanceRadius
        const force = avoidanceStrength * 0.02 // 躲避力强度

        // 计算躲避方向（远离鼠标）
        const normalizedDx = dx / distance
        const normalizedDy = dy / distance

        // 应用躲避力
        flake.avoidanceForceX = normalizedDx * force
        flake.avoidanceForceY = normalizedDy * force

        // 增加旋转速度（表现紧张感）
        flake.rotationSpeed += (Math.random() - 0.5) * 0.005
      } else {
        // 逐渐减少躲避力
        flake.avoidanceForceX *= 0.9
        flake.avoidanceForceY *= 0.9
      }
    },

    // 重置雪花位置
    resetFlake(flake) {
      // 从屏幕上方随机位置开始
      flake.x = Math.random() * (this.canvasWidth + 200) - 100 // 稍微超出边界
      flake.y = -20

      // 重新设置雪花属性
      flake.size = Math.random() * flake.maxSize + 2
      flake.speed = Math.random() * 0.3 + flake.fallSpeed
      flake.velY = Math.random() * 0.2 + 0.1 // 初始较慢的下落速度
      flake.velX = (Math.random() - 0.5) * 0.1 // 更小的初始水平速度
      flake.rotation = Math.random() * Math.PI * 2
      flake.rotationSpeed = (Math.random() - 0.5) * 0.01 // 更慢的旋转
      flake.opacity = Math.random() * 0.4 + 0.6
      flake.swayAmplitude = Math.random() * 0.2 + 0.1 // 更小的摆动幅度
      flake.swaySpeed = Math.random() * 0.01 + 0.005 // 更慢的摆动
      flake.swayOffset = Math.random() * Math.PI * 2

      // 重置躲避相关属性
      flake.avoidanceForceX = 0
      flake.avoidanceForceY = 0
      flake.avoidanceRadius = 60 + Math.random() * 40 // 60-100px的随机躲避半径
    },

    // 渲染雪花 - 真实雪花形状
    renderFlake(flake) {
      if (!this.ctx) return

      this.ctx.save()
      this.ctx.translate(flake.x, flake.y)
      this.ctx.rotate(flake.rotation)

      // 设置雪花颜色和透明度
      this.ctx.strokeStyle = `rgba(255, 255, 255, ${flake.opacity})`
      this.ctx.fillStyle = `rgba(255, 255, 255, ${flake.opacity * 0.3})`
      this.ctx.lineWidth = 0.5

      // 绘制六角雪花
      this.drawSnowflakeShape(flake.size)

      this.ctx.restore()
    },

    // 绘制真实的雪花形状
    drawSnowflakeShape(size) {
      const arms = 6 // 六个分支
      const armLength = size
      const branchLength = size * 0.3

      this.ctx.beginPath()

      // 绘制六个主要分支
      for (let i = 0; i < arms; i++) {
        const angle = (i * Math.PI * 2) / arms
        const x1 = Math.cos(angle) * armLength
        const y1 = Math.sin(angle) * armLength

        // 主分支
        this.ctx.moveTo(0, 0)
        this.ctx.lineTo(x1, y1)

        // 在主分支上添加小分支
        for (let j = 0.3; j <= 0.8; j += 0.25) {
          const branchX = Math.cos(angle) * armLength * j
          const branchY = Math.sin(angle) * armLength * j

          // 左侧小分支
          const leftAngle = angle - Math.PI / 6
          const leftX = branchX + Math.cos(leftAngle) * branchLength
          const leftY = branchY + Math.sin(leftAngle) * branchLength
          this.ctx.moveTo(branchX, branchY)
          this.ctx.lineTo(leftX, leftY)

          // 右侧小分支
          const rightAngle = angle + Math.PI / 6
          const rightX = branchX + Math.cos(rightAngle) * branchLength
          const rightY = branchY + Math.sin(rightAngle) * branchLength
          this.ctx.moveTo(branchX, branchY)
          this.ctx.lineTo(rightX, rightY)
        }
      }

      this.ctx.stroke()

      // 中心小圆
      this.ctx.beginPath()
      this.ctx.arc(0, 0, size * 0.1, 0, Math.PI * 2)
      this.ctx.fill()
    },

    // 开始下雪动画
    startSnowfall() {
      if (!this.ctx) return

      const animate = () => {
        // 清空画布
        this.ctx.clearRect(0, 0, this.canvasWidth, this.canvasHeight)

        // 更新和渲染每个雪花
        this.flakes.forEach(flake => {
          this.updateFlake(flake)
          this.renderFlake(flake)
        })

        // 继续动画
        this.animationId = requestAnimationFrame(animate)
      }

      animate()
    },

    // 停止下雪动画
    stopSnowfall() {
      if (this.animationId) {
        cancelAnimationFrame(this.animationId)
        this.animationId = null
      }
    }
  }
}
</script>

<style scoped>
.snow-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 10;
  overflow: hidden;
}

.snow-canvas {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 10;
}

</style>
