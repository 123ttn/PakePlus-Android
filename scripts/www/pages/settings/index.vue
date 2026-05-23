<template>
  <view class="page">
    <!-- 头部 -->
    <view class="header safe-area-top">
      <text class="title">设置</text>
    </view>

    <!-- 设置列表 -->
    <view class="settings-list">
      <!-- 主题设置 -->
      <view class="settings-section">
        <text class="section-title">外观</text>
        <view class="setting-item" @click="toggleTheme">
          <view class="setting-left">
            <text class="setting-icon">🎨</text>
            <text class="setting-label">深色模式</text>
          </view>
          <view :class="['toggle', darkMode ? 'active' : '']">
            <view class="toggle-dot"></view>
          </view>
        </view>
      </view>

      <!-- 动画设置 -->
      <view class="settings-section">
        <text class="section-title">动画</text>
        <view class="setting-item">
          <view class="setting-left">
            <text class="setting-icon">✨</text>
            <text class="setting-label">启用动画效果</text>
          </view>
          <view :class="['toggle', settings.enableAnimation ? 'active' : '']" @click="toggleAnimation">
            <view class="toggle-dot"></view>
          </view>
        </view>
        <view class="setting-item">
          <view class="setting-left">
            <text class="setting-icon">⏱️</text>
            <text class="setting-label">动画时长</text>
          </view>
          <view class="setting-value">
            <text>{{ settings.animationDuration }}s</text>
          </view>
        </view>
        <view class="slider-item">
          <slider 
            :value="settings.animationDuration" 
            min="1" 
            max="10" 
            step="0.5"
            activeColor="#ffd700"
            backgroundColor="rgba(255,255,255,0.2)"
            @change="onSliderChange"
          />
        </view>
      </view>

      <!-- 声音设置 -->
      <view class="settings-section">
        <text class="section-title">声音</text>
        <view class="setting-item">
          <view class="setting-left">
            <text class="setting-icon">🔊</text>
            <text class="setting-label">启用音效</text>
          </view>
          <view :class="['toggle', settings.enableSound ? 'active' : '']" @click="toggleSound">
            <view class="toggle-dot"></view>
          </view>
        </view>
      </view>

      <!-- 数据管理 -->
      <view class="settings-section">
        <text class="section-title">数据</text>
        <view class="setting-item" @click="exportAllData">
          <view class="setting-left">
            <text class="setting-icon">📤</text>
            <text class="setting-label">导出所有数据</text>
          </view>
          <text class="setting-arrow">›</text>
        </view>
        <view class="setting-item" @click="importData">
          <view class="setting-left">
            <text class="setting-icon">📥</text>
            <text class="setting-label">导入数据</text>
          </view>
          <text class="setting-arrow">›</text>
        </view>
        <view class="setting-item danger" @click="clearAllData">
          <view class="setting-left">
            <text class="setting-icon">🗑️</text>
            <text class="setting-label">清除所有数据</text>
          </view>
          <text class="setting-arrow">›</text>
        </view>
      </view>

      <!-- 关于 -->
      <view class="settings-section">
        <text class="section-title">关于</text>
        <view class="setting-item">
          <view class="setting-left">
            <text class="setting-icon">📱</text>
            <text class="setting-label">版本</text>
          </view>
          <text class="setting-value">1.0.0</text>
        </view>
        <view class="setting-item" @click="checkUpdate">
          <view class="setting-left">
            <text class="setting-icon">🔄</text>
            <text class="setting-label">检查更新</text>
          </view>
          <text class="setting-arrow">›</text>
        </view>
      </view>
    </view>

    <!-- 底部信息 -->
    <view class="footer safe-area-bottom">
      <text class="footer-text">困难选择器 v1.0.0</text>
      <text class="footer-hint">让选择不再困难</text>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      darkMode: true,
      settings: {
        enableAnimation: true,
        animationDuration: 5,
        enableSound: false
      }
    }
  },
  onLoad() {
    this.loadSettings()
  },
  methods: {
    loadSettings() {
      const savedSettings = uni.getStorageSync('appSettings')
      if (savedSettings) {
        this.settings = savedSettings
      }
    },
    saveSettings() {
      uni.setStorageSync('appSettings', this.settings)
    },
    toggleTheme() {
      this.darkMode = !this.darkMode
      // 主题切换逻辑
      uni.showToast({
        title: this.darkMode ? '深色模式' : '浅色模式',
        icon: 'success'
      })
    },
    toggleAnimation() {
      this.settings.enableAnimation = !this.settings.enableAnimation
      this.saveSettings()
    },
    toggleSound() {
      this.settings.enableSound = !this.settings.enableSound
      this.saveSettings()
      uni.showToast({
        title: this.settings.enableSound ? '音效已开启' : '音效已关闭',
        icon: 'success'
      })
    },
    onSliderChange(e) {
      this.settings.animationDuration = e.detail.value
      this.saveSettings()
    },
    exportAllData() {
      const modules = uni.getStorageSync('modules') || []
      const currentModule = uni.getStorageSync('currentModule')
      const data = {
        modules: modules,
        currentModule: currentModule,
        exportTime: new Date().toISOString()
      }
      const dataStr = JSON.stringify(data, null, 2)
      uni.setClipboardData({
        data: dataStr,
        success: () => {
          uni.showToast({
            title: '已复制到剪贴板',
            icon: 'success'
          })
        }
      })
    },
    importData() {
      uni.showModal({
        title: '导入数据',
        content: '请将数据粘贴到输入框',
        editable: true,
        placeholderText: '请输入JSON数据...',
        success: (res) => {
          if (res.content) {
            try {
              const data = JSON.parse(res.content)
              if (data.modules) {
                uni.setStorageSync('modules', data.modules)
              }
              if (data.currentModule) {
                uni.setStorageSync('currentModule', data.currentModule)
              }
              uni.showToast({
                title: '导入成功',
                icon: 'success'
              })
            } catch (e) {
              uni.showToast({
                title: '数据格式错误',
                icon: 'none'
              })
            }
          }
        }
      })
    },
    clearAllData() {
      uni.showModal({
        title: '警告',
        content: '确定要清除所有数据吗？此操作不可恢复！',
        success: (res) => {
          if (res.confirm) {
            uni.clearStorageSync()
            uni.showToast({
              title: '已清除所有数据',
              icon: 'success'
            })
          }
        }
      })
    },
    checkUpdate() {
      uni.showToast({
        title: '已是最新版本',
        icon: 'success'
      })
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

.title {
  font-size: 40rpx;
  font-weight: bold;
  color: #ffffff;
}

.settings-list {
  padding: 0 30rpx;
}

.settings-section {
  margin-bottom: 40rpx;
}

.section-title {
  display: block;
  font-size: 24rpx;
  color: #a0a0a0;
  text-transform: uppercase;
  letter-spacing: 2rpx;
  margin-bottom: 20rpx;
  padding-left: 10rpx;
}

.setting-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 30rpx;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 16rpx;
  margin-bottom: 15rpx;
}

.setting-item.danger {
  border: 1px solid rgba(255, 100, 100, 0.3);
}

.setting-left {
  display: flex;
  align-items: center;
}

.setting-icon {
  font-size: 36rpx;
  margin-right: 20rpx;
}

.setting-label {
  font-size: 28rpx;
  color: #ffffff;
}

.setting-value {
  font-size: 28rpx;
  color: #a0a0a0;
}

.setting-arrow {
  font-size: 36rpx;
  color: #a0a0a0;
}

.toggle {
  width: 100rpx;
  height: 56rpx;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 28rpx;
  padding: 4rpx;
  transition: all 0.3s;
}

.toggle.active {
  background: linear-gradient(135deg, #ffd700, #ff8c00);
}

.toggle-dot {
  width: 48rpx;
  height: 48rpx;
  background: #ffffff;
  border-radius: 50%;
  transition: all 0.3s;
  box-shadow: 0 4rpx 8rpx rgba(0, 0, 0, 0.2);
}

.toggle.active .toggle-dot {
  transform: translateX(44rpx);
}

.slider-item {
  padding: 0 30rpx;
  margin-top: -10rpx;
  margin-bottom: 20rpx;
}

.footer {
  text-align: center;
  padding: 60rpx 30rpx;
}

.footer-text {
  display: block;
  font-size: 26rpx;
  color: #a0a0a0;
  margin-bottom: 10rpx;
}

.footer-hint {
  display: block;
  font-size: 22rpx;
  color: #666;
}

.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
</style>
