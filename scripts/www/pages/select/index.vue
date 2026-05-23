<template>
  <view class="page">
    <!-- 标题栏 -->
    <view class="header safe-area-top">
      <view class="header-content">
        <text class="title">{{ currentModuleName }}</text>
        <text class="subtitle">共 {{ heroCount }} 个英雄</text>
      </view>
    </view>

    <!-- 孔洞数量选择 -->
    <view class="hole-selector">
      <text class="selector-label">选择孔洞数：</text>
      <view class="selector-buttons">
        <view 
          v-for="n in 4" 
          :key="n"
          :class="['selector-btn', holeCount === n ? 'active' : '']"
          @click="setHoleCount(n)"
        >
          <text>{{ n }}孔</text>
        </view>
      </view>
    </view>

    <!-- 转盘区域 -->
    <view class="wheel-container">
      <view class="wheel-wrapper">
        <!-- 外圈装饰 -->
        <view class="outer-ring"></view>
        
        <!-- 转盘主体 -->
        <view 
          class="wheel" 
          :style="{ transform: 'rotate(' + wheelRotation + 'deg)' }"
        >
          <view class="wheel-inner">
            <!-- 珠子 -->
            <view 
              v-for="(ball, index) in displayBalls" 
              :key="index"
              class="ball"
              :style="ball.style"
            >
              <view class="ball-highlight"></view>
            </view>
          </view>
        </view>

        <!-- 中心区域 -->
        <view class="center-area">
          <!-- 孔洞 -->
          <view 
            v-for="(hole, index) in holePositions" 
            :key="'hole-' + index"
            class="hole"
            :style="{ left: hole.x + 'px', top: hole.y + 'px' }"
          ></view>
        </view>
      </view>
    </view>

    <!-- 开始按钮 -->
    <view class="action-area safe-area-bottom">
      <button 
        class="btn-start" 
        :disabled="isAnimating || !currentModule"
        @click="startSpin"
      >
        {{ isAnimating ? '选择中...' : '开始选择 (' + holeCount + '孔)' }}
      </button>
    </view>

    <!-- 结果弹窗 -->
    <view class="result-modal" v-if="showResult">
      <view class="modal-mask" @click="closeResult"></view>
      <view class="modal-content">
        <view class="result-header">
          <text class="result-title">🎉 恭喜选中！</text>
        </view>
        <view class="result-list">
          <view 
            v-for="(hero, index) in selectedHeroes" 
            :key="index"
            class="result-item"
          >
            <view class="result-ball" :style="{ background: getBallColor(index) }">
              <text class="result-icon">{{ getIcon(hero.name) }}</text>
            </view>
            <text class="result-name">{{ hero.name }}</text>
          </view>
        </view>
        <button class="btn-close" @click="closeResult">确定</button>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      currentModule: null,
      holeCount: 1,
      wheelRotation: 0,
      isAnimating: false,
      showResult: false,
      selectedHeroes: [],
      balls: [],
      animationTimer: null
    }
  },
  computed: {
    currentModuleName() {
      return this.currentModule ? this.currentModule.name : '请先选择模块'
    },
    heroCount() {
      return this.currentModule && this.currentModule.heroes ? this.currentModule.heroes.length : 0
    },
    displayBalls() {
      return this.balls.slice(0, 20)
    },
    holePositions() {
      const size = 36
      const positions = []
      for (let i = 0; i < this.holeCount; i++) {
        if (this.holeCount === 1) {
          positions.push({ x: 135, y: 160 })
        } else if (this.holeCount === 2) {
          positions.push({ x: 105, y: 160 })
          positions.push({ x: 165, y: 160 })
        } else if (this.holeCount === 3) {
          positions.push({ x: 135, y: 130 })
          positions.push({ x: 105, y: 180 })
          positions.push({ x: 165, y: 180 })
        } else {
          positions.push({ x: 105, y: 130 })
          positions.push({ x: 165, y: 130 })
          positions.push({ x: 105, y: 180 })
          positions.push({ x: 165, y: 180 })
        }
      }
      return positions
    }
  },
  onLoad() {
    this.loadData()
    this.initBalls()
  },
  onUnload() {
    if (this.animationTimer) {
      clearTimeout(this.animationTimer)
    }
  },
  methods: {
    loadData() {
      const moduleData = uni.getStorageSync('currentModule')
      if (moduleData) {
        this.currentModule = moduleData
        this.initBalls()
      }
    },
    initBalls() {
      if (!this.currentModule || !this.currentModule.heroes) {
        this.balls = []
        return
      }
      const heroes = this.currentModule.heroes
      const colors = [
        '#FF6B6B', '#4ECDC4', '#FFE66D', '#95E1D3',
        '#F38181', '#AA96DA', '#FCBAD3', '#A8D8EA',
        '#FF9F43', '#6C5CE7', '#00CEC9', '#FD79A8'
      ]
      const balls = []
      for (let i = 0; i < heroes.length; i++) {
        const angle = (i / heroes.length) * Math.PI * 2
        const radius = 60 + Math.random() * 60
        const x = 150 + Math.cos(angle) * radius - 14
        const y = 150 + Math.sin(angle) * radius - 14
        balls.push({
          hero: heroes[i],
          style: {
            left: x + 'px',
            top: y + 'px',
            background: colors[i % colors.length]
          }
        })
      }
      this.balls = balls
    },
    setHoleCount(count) {
      this.holeCount = count
    },
    getIcon(name) {
      return name && name.length > 0 ? name.charAt(0) : '?'
    },
    getBallColor(index) {
      const colors = [
        '#FF6B6B', '#4ECDC4', '#FFE66D', '#95E1D3',
        '#F38181', '#AA96DA', '#FCBAD3', '#A8D8EA'
      ]
      return colors[index % colors.length]
    },
    startSpin() {
      if (this.isAnimating || !this.currentModule) return
      
      this.isAnimating = true
      const heroes = this.currentModule.heroes
      if (!heroes || heroes.length === 0) {
        uni.showToast({ title: '没有英雄数据', icon: 'none' })
        this.isAnimating = false
        return
      }

      // 随机选择
      const selected = []
      const usedIndices = new Set()
      for (let i = 0; i < this.holeCount; i++) {
        let idx
        do {
          idx = Math.floor(Math.random() * heroes.length)
        } while (usedIndices.has(idx) && usedIndices.size < heroes.length)
        usedIndices.add(idx)
        selected.push(heroes[idx])
      }

      // 转盘动画
      const targetRotation = this.wheelRotation + 1800 + Math.random() * 720
      const duration = 5000
      const startTime = Date.now()
      const startRotation = this.wheelRotation

      const animate = () => {
        const elapsed = Date.now() - startTime
        const progress = Math.min(elapsed / duration, 1)
        const eased = 1 - Math.pow(1 - progress, 3)
        this.wheelRotation = startRotation + (targetRotation - startRotation) * eased
        
        if (progress < 1) {
          this.animationTimer = setTimeout(animate, 16)
        } else {
          this.wheelRotation = targetRotation
          setTimeout(() => {
            this.selectedHeroes = selected
            this.showResult = true
            this.isAnimating = false
          }, 500)
        }
      }
      animate()
    },
    closeResult() {
      this.showResult = false
    }
  }
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
}

.header {
  padding: 40rpx 30rpx;
  text-align: center;
}

.header-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.title {
  font-size: 40rpx;
  font-weight: bold;
  color: #ffd700;
  text-shadow: 0 0 20rpx rgba(255, 215, 0, 0.5);
}

.subtitle {
  font-size: 26rpx;
  color: #a0a0a0;
  margin-top: 10rpx;
}

.hole-selector {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 30rpx;
  gap: 20rpx;
}

.selector-label {
  font-size: 28rpx;
  color: #ffffff;
}

.selector-buttons {
  display: flex;
  gap: 15rpx;
}

.selector-btn {
  padding: 15rpx 30rpx;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 30rpx;
  border: 2px solid rgba(255, 255, 255, 0.2);
}

.selector-btn.active {
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  border-color: #ffd700;
}

.selector-btn text {
  font-size: 26rpx;
  color: #ffffff;
}

.selector-btn.active text {
  color: #1a1a2e;
  font-weight: bold;
}

.wheel-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40rpx;
}

.wheel-wrapper {
  position: relative;
  width: 300px;
  height: 300px;
}

.outer-ring {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  box-shadow: 
    0 0 30px rgba(255, 215, 0, 0.5),
    inset 0 0 20px rgba(0, 0, 0, 0.3);
}

.wheel {
  position: absolute;
  width: 100%;
  height: 100%;
  transition: transform 0.05s linear;
}

.wheel-inner {
  position: absolute;
  width: 280px;
  height: 280px;
  left: 10px;
  top: 10px;
  border-radius: 50%;
  background: linear-gradient(135deg, #2d2d4e, #1a1a3e);
  overflow: hidden;
}

.ball {
  position: absolute;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  box-shadow: 
    inset -4px -4px 8px rgba(0, 0, 0, 0.4),
    inset 4px 4px 8px rgba(255, 255, 255, 0.3),
    2px 2px 4px rgba(0, 0, 0, 0.3);
}

.ball-highlight {
  position: absolute;
  width: 8px;
  height: 8px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 50%;
  top: 4px;
  left: 4px;
}

.center-area {
  position: absolute;
  width: 100%;
  height: 100%;
}

.hole {
  position: absolute;
  width: 36px;
  height: 36px;
  background: radial-gradient(circle, #000000 0%, #1a1a1a 100%);
  border-radius: 50%;
  border: 2px solid rgba(255, 215, 0, 0.5);
  box-shadow: 
    inset 0 4px 8px rgba(0, 0, 0, 0.8),
    0 0 10px rgba(0, 0, 0, 0.5);
  transform: translate(-50%, -50%);
}

.action-area {
  padding: 40rpx 30rpx;
}

.btn-start {
  width: 100%;
  height: 100rpx;
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  border: none;
  border-radius: 50rpx;
  font-size: 34rpx;
  font-weight: bold;
  color: #1a1a2e;
  box-shadow: 0 10rpx 30rpx rgba(255, 215, 0, 0.4);
}

.btn-start[disabled] {
  background: #555;
  color: #999;
  box-shadow: none;
}

.result-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 999;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-mask {
  position: absolute;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.8);
}

.modal-content {
  position: relative;
  width: 80%;
  background: linear-gradient(135deg, #2d2d4e, #1a1a3e);
  border-radius: 30rpx;
  padding: 50rpx;
  border: 2px solid rgba(255, 215, 0, 0.5);
}

.result-header {
  text-align: center;
  margin-bottom: 40rpx;
}

.result-title {
  font-size: 40rpx;
  font-weight: bold;
  color: #ffd700;
}

.result-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 30rpx;
  margin-bottom: 40rpx;
}

.result-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15rpx;
}

.result-ball {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 
    inset -6px -6px 12px rgba(0, 0, 0, 0.4),
    inset 6px 6px 12px rgba(255, 255, 255, 0.3),
    4px 4px 8px rgba(0, 0, 0, 0.3);
}

.result-icon {
  font-size: 48rpx;
  color: #ffffff;
  font-weight: bold;
}

.result-name {
  font-size: 28rpx;
  color: #ffffff;
  text-align: center;
}

.btn-close {
  width: 100%;
  height: 90rpx;
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  border: none;
  border-radius: 45rpx;
  font-size: 32rpx;
  font-weight: bold;
  color: #1a1a2e;
}

.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
</style>
