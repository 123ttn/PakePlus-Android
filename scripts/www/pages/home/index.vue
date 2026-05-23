<template>
  <view class="page">
    <!-- 头部 -->
    <view class="header safe-area-top">
      <text class="title-lg">困难选择器</text>
      <text class="title-sm">{{ currentModuleName }}</text>
    </view>

    <!-- 当前模块卡片 -->
    <view class="current-module" v-if="currentModule">
      <view class="module-card" @click="goSelect">
        <view class="module-icon">
          <text class="icon-text">{{ moduleIcon }}</text>
        </view>
        <view class="module-info">
          <text class="module-name">{{ currentModule.name }}</text>
          <text class="module-count">共 {{ heroCount }} 个英雄</text>
        </view>
        <view class="module-arrow">
          <text class="arrow-icon">›</text>
        </view>
      </view>
    </view>

    <!-- 空状态 -->
    <view class="empty-state" v-else>
      <text class="empty-icon">🎲</text>
      <text class="empty-text">还没有选择模块</text>
      <text class="empty-hint">点击下方按钮选择或导入模块</text>
    </view>

    <!-- 快捷操作 -->
    <view class="quick-actions">
      <text class="section-title">快捷操作</text>
      <view class="action-grid">
        <view class="action-item" @click="goModules">
          <text class="action-icon">📦</text>
          <text class="action-text">模块管理</text>
        </view>
        <view class="action-item" @click="goSettings">
          <text class="action-icon">⚙️</text>
          <text class="action-text">设置</text>
        </view>
        <view class="action-item" @click="importTemplate">
          <text class="action-icon">📥</text>
          <text class="action-text">导入模板</text>
        </view>
        <view class="action-item" @click="exportData">
          <text class="action-icon">📤</text>
          <text class="action-text">导出数据</text>
        </view>
      </view>
    </view>

    <!-- 模板推荐 -->
    <view class="templates-section">
      <text class="section-title">推荐模板</text>
      <scroll-view class="template-scroll" scroll-x="true">
        <view class="template-list">
          <view 
            class="template-card" 
            v-for="(template, index) in templates" 
            :key="index"
            @click="loadTemplate(template)"
          >
            <text class="template-icon">{{ getIcon(template.name) }}</text>
            <text class="template-name">{{ template.name }}</text>
          </view>
        </view>
      </scroll-view>
    </view>

    <!-- 开始选择按钮 -->
    <view class="start-button safe-area-bottom" v-if="currentModule">
      <button class="btn-start" @click="goSelect">
        开始选择
      </button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      currentModule: null,
      templates: []
    }
  },
  computed: {
    currentModuleName() {
      return this.currentModule ? this.currentModule.name : '请选择模块'
    },
    moduleIcon() {
      return this.currentModule && this.currentModule.name ? this.currentModule.name.charAt(0) : '?'
    },
    heroCount() {
      return this.currentModule && this.currentModule.heroes ? this.currentModule.heroes.length : 0
    }
  },
  onLoad() {
    this.loadData()
    this.fetchTemplates()
  },
  onShow() {
    this.loadData()
  },
  methods: {
    loadData() {
      const moduleData = uni.getStorageSync('currentModule')
      if (moduleData) {
        this.currentModule = moduleData
      }
    },
    getIcon(name) {
      return name && name.length > 0 ? name.charAt(0) : '?'
    },
    fetchTemplates() {
      const savedModules = uni.getStorageSync('modules') || []
      this.templates = savedModules.slice(0, 5)
    },
    goSelect() {
      uni.navigateTo({
        url: '/pages/select/index'
      })
    },
    goModules() {
      uni.switchTab({
        url: '/pages/modules/index'
      })
    },
    goSettings() {
      uni.switchTab({
        url: '/pages/settings/index'
      })
    },
    loadTemplate(template) {
      this.currentModule = template
      uni.setStorageSync('currentModule', template)
      uni.showToast({
        title: '已加载: ' + template.name,
        icon: 'success'
      })
    },
    importTemplate() {
      uni.showModal({
        title: '导入模板',
        content: '请选择导入方式',
        cancelText: '从API导入',
        confirmText: '手动输入',
        success: (res) => {
          if (res.confirm) {
            this.manualImport()
          } else if (res.cancel) {
            this.apiImport()
          }
        }
      })
    },
    apiImport() {
      uni.showLoading({ title: '加载中...' })
      uni.request({
        url: 'http://localhost:9091/api/v1/templates',
        success: (res) => {
          uni.hideLoading()
          if (res.data && res.data.data) {
            this.templates = res.data.data
            uni.showToast({
              title: '已加载 ' + res.data.data.length + ' 个模板',
              icon: 'success'
            })
          }
        },
        fail: () => {
          uni.hideLoading()
          uni.showToast({
            title: '加载失败，请检查网络',
            icon: 'none'
          })
        }
      })
    },
    manualImport() {
      uni.showToast({
        title: '请在模块管理中添加',
        icon: 'none'
      })
    },
    exportData() {
      if (!this.currentModule) {
        uni.showToast({
          title: '请先选择一个模块',
          icon: 'none'
        })
        return
      }
      const dataStr = JSON.stringify(this.currentModule, null, 2)
      uni.setClipboardData({
        data: dataStr,
        success: () => {
          uni.showToast({
            title: '已复制到剪贴板',
            icon: 'success'
          })
        }
      })
    }
  }
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
  padding: 20rpx;
}

.header {
  padding: 40rpx 20rpx;
  text-align: center;
}

.title-lg {
  display: block;
  font-size: 48rpx;
  font-weight: bold;
  color: #ffd700;
  text-shadow: 0 0 20rpx rgba(255, 215, 0, 0.5);
  margin-bottom: 10rpx;
}

.title-sm {
  display: block;
  font-size: 28rpx;
  color: #a0a0a0;
}

.current-module {
  margin: 30rpx 0;
}

.module-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 24rpx;
  padding: 30rpx;
  display: flex;
  align-items: center;
  border: 1px solid rgba(255, 215, 0, 0.3);
  box-shadow: 0 10rpx 40rpx rgba(0, 0, 0, 0.3);
}

.module-icon {
  width: 100rpx;
  height: 100rpx;
  background: linear-gradient(135deg, #ffd700, #ff8c00);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-right: 20rpx;
}

.icon-text {
  font-size: 40rpx;
  font-weight: bold;
  color: #1a1a2e;
}

.module-info {
  flex: 1;
}

.module-name {
  display: block;
  font-size: 32rpx;
  color: #ffffff;
  font-weight: bold;
  margin-bottom: 8rpx;
}

.module-count {
  display: block;
  font-size: 24rpx;
  color: #a0a0a0;
}

.module-arrow {
  padding: 20rpx;
}

.arrow-icon {
  font-size: 48rpx;
  color: #ffd700;
}

.empty-state {
  text-align: center;
  padding: 100rpx 40rpx;
}

.empty-icon {
  font-size: 120rpx;
  display: block;
  margin-bottom: 30rpx;
}

.empty-text {
  display: block;
  font-size: 32rpx;
  color: #ffffff;
  margin-bottom: 15rpx;
}

.empty-hint {
  display: block;
  font-size: 26rpx;
  color: #a0a0a0;
}

.quick-actions {
  margin: 40rpx 0;
}

.section-title {
  display: block;
  font-size: 30rpx;
  color: #ffffff;
  font-weight: bold;
  margin-bottom: 25rpx;
  padding-left: 10rpx;
}

.action-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20rpx;
}

.action-item {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20rpx;
  padding: 30rpx 15rpx;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.action-icon {
  display: block;
  font-size: 50rpx;
  margin-bottom: 10rpx;
}

.action-text {
  display: block;
  font-size: 22rpx;
  color: #d0d0d0;
}

.templates-section {
  margin: 40rpx 0;
}

.template-scroll {
  white-space: nowrap;
}

.template-list {
  display: inline-flex;
  gap: 20rpx;
  padding: 10rpx 0;
}

.template-card {
  width: 180rpx;
  background: rgba(255, 255, 255, 0.08);
  border-radius: 20rpx;
  padding: 30rpx 20rpx;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.template-icon {
  display: block;
  width: 80rpx;
  height: 80rpx;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border-radius: 50%;
  line-height: 80rpx;
  margin: 0 auto 15rpx;
  font-size: 32rpx;
  color: #ffffff;
}

.template-name {
  display: block;
  font-size: 24rpx;
  color: #d0d0d0;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.start-button {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 30rpx;
  background: rgba(26, 26, 46, 0.95);
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

.safe-area-bottom {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
</style>
